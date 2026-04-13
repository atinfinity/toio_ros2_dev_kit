# toio_ros2

## Introduction

`toio_ros2` is ROS 2 package for using [toio](https://toio.io/).

![](https://raw.githubusercontent.com/atinfinity/toio_ros2/refs/heads/jazzy/image/toio_ros2_rviz2.png)

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
|/goal_pose|[geometry_msgs/msg/PoseStamped](https://docs.ros2.org/foxy/api/geometry_msgs/msg/PoseStamped.html)|desired robot pose|

## Published topics

|topic name|Type|Description|
|:---|:---|:---|
|/toio/pose|[geometry_msgs/msg/PoseStamped](https://docs.ros2.org/foxy/api/geometry_msgs/msg/PoseStamped.html)|toio pose in map frame|
|/toio/battery_level|[std_msgs/msg/Float32](https://docs.ros2.org/foxy/api/std_msgs/msg/Float32.html)|battery level of toio|
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

Parameter files is stored in [params](https://github.com/atinfinity/toio_ros2/tree/jazzy/params).
And, [launch/toio_ros2_bringup.launch.py](https://github.com/atinfinity/toio_ros2/blob/jazzy/launch/toio_ros2_bringup.launch.py) load [params/toio_a4_play_mat_params.yaml](https://github.com/atinfinity/toio_ros2/blob/jazzy/params/toio_a4_play_mat_params.yaml) as default.


```python
declare_params_file_cmd = DeclareLaunchArgument(
    'params_file',
    default_value=os.path.join(toio_ros2_dir, 'params', 'toio_a4_play_mat_params.yaml'),
    description='Full path to the ROS2 parameters file to use toio_ros2 node')
```

## Frame

![](https://raw.githubusercontent.com/atinfinity/toio_ros2/refs/heads/jazzy/image/frames.png)