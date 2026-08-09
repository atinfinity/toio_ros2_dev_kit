# toio_ros2

## Introduction

`toio_ros2` is ROS 2 package for using [toio](https://toio.io/).

![](https://raw.githubusercontent.com/atinfinity/toio_ros2/refs/heads/jazzy/image/toio_ros2_rviz2.png)

If you are interested in this work, please read <https://qiita.com/dandelion1124/items/150ec284d8d85769f745>.

## Feature

- Control using the `cmd_vel` topic
- Navigation using the `goal_pose` topic(cube built-in target motion)
- Publish `/toio/pose` topic and `tf` topic(position and orientation of toio)
- Publish `/toio/battery_state` as battery state information
- Connect to a specific cube using the `cube_id` or `cube_address` parameter
- Multiple cubes using ROS namespace and TF frame prefix

## Requirements

### Hardware

- toio Core Cube
- toio play mat
    - Please see <https://toio.github.io/toio-spec/en/docs/hardware_position_id>.

### Software

I checked this package on the following environment.

- Ubuntu 24.04
- ROS 2 Jazzy
- toio.py 1.10.0

## Subscribed topics

|topic name|Type|Description|
|:---|:---|:---|
|/cmd_vel|[geometry_msgs/msg/Twist](https://docs.ros2.org/foxy/api/geometry_msgs/msg/Twist.html)|desired robot velocity|
|/goal_pose|[geometry_msgs/msg/PoseStamped](https://docs.ros2.org/foxy/api/geometry_msgs/msg/PoseStamped.html)|desired robot pose (cube built-in target motion; disabled when `enable_goal_pose_motion` is false)|

## Published topics

|topic name|Type|Description|
|:---|:---|:---|
|/toio/pose|[geometry_msgs/msg/PoseStamped](https://docs.ros2.org/foxy/api/geometry_msgs/msg/PoseStamped.html)|toio pose in map frame|
|/toio/battery_state|[sensor_msgs/msg/BatteryState](https://docs.ros2.org/foxy/api/sensor_msgs/msg/BatteryState.html)|battery level of toio (`percentage` is 0.0-1.0). The cube notifies it in 10% steps, see <https://toio.github.io/toio-spec/docs/ble_battery>|
|/tf|-|a valid transform from `map` to `center`|

## Parameters

Default is a param for A4 mat. 
Please see <https://toio.github.io/toio-spec/docs/hardware_position_id> in detail.

|name|Type|Default|Description|
|:---|:---|:---|:---|
|field_min_x|double|98.0|minimum of `x` in field|
|field_max_x|double|402.0|maximum of `x` in field|
|field_min_y|double|142.0|minimum of `y` in field|
|field_max_y|double|358.0|maximum of `y` in field|
|field_width_meter|double|0.297|width of field(meter)|
|field_height_meter|double|0.210|height of field(meter)|
|goal_max_speed|int|30|maximum motor speed for `goal_pose` motion|
|goal_timeout|int|60|timeout(second) for `goal_pose` motion|
|goal_boundary_margin|int|10|margin(Position ID units) kept between a clamped goal and the mat boundary|
|cube_id|string|''|connect only to the cube whose BLE local name contains `cube_id`|
|cube_address|string|''|connect only to the cube with this BLE address|
|frame_prefix|string|''|prefix of the TF child frame(`<frame_prefix>center`) for multi-cube setups|
|enable_goal_pose_motion|bool|true|subscribe `goal_pose` and use the cube built-in target motion. Set to false when an external traffic authority(e.g. Open-RMF) owns the motion plan and all movement must go through Nav2 `cmd_vel`|

Parameter files is stored in [params](https://github.com/atinfinity/toio_ros2/tree/jazzy/params).
And, [launch/toio_ros2_bringup.launch.py](https://github.com/atinfinity/toio_ros2/blob/jazzy/launch/toio_ros2_bringup.launch.py) load [params/toio_a4_play_mat_params.yaml](https://github.com/atinfinity/toio_ros2/blob/jazzy/params/toio_a4_play_mat_params.yaml) as default.


```python
declare_params_file_cmd = DeclareLaunchArgument(
    'params_file',
    default_value=os.path.join(toio_ros2_dir, 'params', 'toio_a4_play_mat_params.yaml'),
    description='Full path to the ROS2 parameters file to use toio_ros2 node')
```

Parameter files use the `/**/toio_ros2_node:` wildcard key so that they apply to the node in any namespace(both the plain single-cube launch and the per-robot namespaces of the multi-cube launch).
A bare `toio_ros2_node:` key would only match the root namespace and be silently ignored by the namespaced nodes.

## Connecting to a specific cube

By default(`cube_id` and `cube_address` are empty), the node connects to the nearest cube found by the BLE scan.
This is convenient when you have a single cube, but with other cubes around you may connect to somebody else's cube.

To connect only to your own cube, set the `cube_id` parameter to the identifier contained in the cube's BLE local name.
The name format depends on the cube, so take the `<cube_id>` part of whichever form your cube advertises.

- `toio Core Cube-<cube_id>` (e.g. `toio Core Cube-C7f` -> `C7f`)
- `toio-<cube_id> (toio Core Cube)` (e.g. `toio-a7D (toio Core Cube)` -> `a7D`)

The name of every cube found by the scan is printed in the node log at startup, so you can find your `cube_id` there.

```bash
ros2 run toio_ros2 toio_ros2_node --ros-args -p cube_id:=a7D
```

`cube_id` is matched as a substring of the BLE local name, so use enough characters to identify one cube.
A short `cube_id` may match several cubes, and the nearest match is used.

`cube_address` can be used instead to specify the BLE address directly, but note that it is platform dependent: a MAC address on Linux/Windows and a CoreBluetooth UUID on macOS.
`cube_id` takes precedence when both are set.

## Using multiple cubes

Each cube is handled by its own node instance separated by a ROS namespace.
[launch/toio_multi_bringup.launch.py](https://github.com/atinfinity/toio_ros2/blob/jazzy/launch/toio_multi_bringup.launch.py) brings up one cube per namespace listed in the `robots` argument(default `toio1,toio2`).
Specifying `cube_id` of every cube is mandatory here, because without it the nodes would race for the same cube.

```bash
ros2 launch toio_ros2 toio_multi_bringup.launch.py cube1_id:=a7D cube2_id:=A8e
```

For three or more cubes, pass the `robots` and `cube_ids` arguments.

```bash
ros2 launch toio_ros2 toio_multi_bringup.launch.py robots:=toio1,toio2,toio3 cube_ids:=a7D,A8e,B9f
```

Topics are namespaced(`/toio1/cmd_vel`, `/toio1/toio/pose`, ...) and TF uses one tree with the shared `map` frame and per-cube prefixed frames(`toio1/center`, `toio2/center`, ...).
RViz2 starts with [rviz/toio_multi.rviz](https://github.com/atinfinity/toio_ros2/blob/jazzy/rviz/toio_multi.rviz), which shows the pose and robot model of every cube.

## Frame

![](https://raw.githubusercontent.com/atinfinity/toio_ros2/refs/heads/jazzy/image/frames.png)
