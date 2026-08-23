# toio_gazebo

## Introduction

This is a ROS 2 Package to develop package of [toio](https://toio.io/) using Gazebo.

<video style="width: 100%" muted="" controls="" alt="type:video">
   <source src="../videos/toio_gazebo.mp4" type="video/mp4">
</video>


<video style="width: 100%" muted="" controls="" alt="type:video">
   <source src="../videos/toio_navigation_with_gazebo.mp4" type="video/mp4">
</video>

## Feature

`toio_gazebo` provides Gazebo environment for toio. It aims for **interface
parity with the real cube([toio_ros2](toio_ros2.md))**, so the same topics work
against the simulation and the hardware.

- Control using the `/cmd_vel` topic
- Publish `/tf`(`map -> odom -> center`, or `map -> center`) and `/odom` wheel odometry
- Indicator LED and sound: `/toio/led`, `/toio/sound`, and the `/toio/led_timed`, `/toio/led_pattern`, `/toio/melody` topics([toio_msgs](toio_msgs.md))
- `/toio/imu`(IMU sensor) and `/toio/motion`(fixed stub for parity)
- **Opt-in simulated battery**(`/toio/battery_state`, `publish_battery`) so Open-RMF `ChargeBattery` can fire in sim
- Multi-robot simulation separated by ROS namespace and TF frame prefix
- Spawn an additional robot into a running simulation

> The full ROS 2 interface(topics and launch arguments) is in
> [docs/topics.md](https://github.com/atinfinity/toio_gazebo/blob/main/docs/topics.md),
> and which toio_ros2 interfaces the simulation supports, stubs or leaves out is
> in [docs/toio_ros2_support.md](https://github.com/atinfinity/toio_gazebo/blob/main/docs/toio_ros2_support.md).
> The LED and sound behavior is in
> [docs/led_and_sound.md](https://github.com/atinfinity/toio_gazebo/blob/main/docs/led_and_sound.md).

## Requirements

- Ubuntu 24.04
- ROS 2 Jazzy
- Gazebo Harmonic

## Worlds

World files are stored in [worlds](https://github.com/atinfinity/toio_gazebo/tree/main/worlds).

|world|Description|
|:---|:---|
|toio_a4_map.sdf|A4 play mat|
|toio_a3_map.sdf|A3 play mat|
|depot.sdf|depot environment|

[launch/simulation.launch.py](https://github.com/atinfinity/toio_gazebo/blob/main/launch/simulation.launch.py) loads `toio_a4_map.sdf` as default, and the world can be switched using the `world` and `world_frame` arguments.

```bash
ros2 launch toio_gazebo simulation.launch.py world:=<WORLD_SDF_FILEPATH> world_frame:=<WORLD_FRAME_NAME>
```

## Multi-robot simulation

[launch/toio_multi_simulation.launch.py](https://github.com/atinfinity/toio_gazebo/blob/main/launch/toio_multi_simulation.launch.py) spawns two toio robots(`toio1` and `toio2`).

```bash
ros2 launch toio_gazebo toio_multi_simulation.launch.py
```

Each robot is separated by ROS namespace and TF frame prefix. The other topics
above(`/toio/led`, `/toio/imu`, `/odom`, ...) are namespaced the same way
(`/toio1/toio/led`, `/toio2/odom`, ...).

|robot|cmd_vel|joint_states|TF|
|:---|:---|:---|:---|
|toio1|`/toio1/cmd_vel`|`/toio1/joint_states`|`map -> toio1/center`|
|toio2|`/toio2/cmd_vel`|`/toio2/joint_states`|`map -> toio2/center`|

To spawn an additional robot into a running simulation, use [launch/spawn_toio.launch.py](https://github.com/atinfinity/toio_gazebo/blob/main/launch/spawn_toio.launch.py).

```bash
ros2 launch toio_gazebo spawn_toio.launch.py namespace:=toio3 robot_name:=toio3 frame_prefix:=toio3/ x_pose:=0.145 y_pose:=-0.095
```

## Launch arguments

On the real cube these are parameters of the `toio_ros2` node; in the simulation
they are launch arguments, because the LED is a Gazebo plugin and the sound is a
separate node. The main ones:

|argument|default|description|
|:---|:---|:---|
|`led_duration_ms`|`0`|lighting time of `/toio/led`(0 keeps it lit until the next command)|
|`led_light_intensity`|`1.0`|intensity of the light cast on the mat by the indicator|
|`sound_volume`|`255`|volume of `/toio/sound`(0 mutes; any other value is full volume)|
|`imu_interval_ms`|`100`|notification interval of `/toio/imu`(update rate `1000/imu_interval_ms` Hz)|
|`publish_odom`|`True`|publish `/odom` and the `map -> odom -> center` TF tree|
|`publish_battery`|`False`|publish the simulated `/toio/battery_state`(off by default; on makes Open-RMF `ChargeBattery` fire in sim)|

The battery model(`discharge_rate`, `charge_rate`, `quantize_steps`, `chargers`,
...) and full details are in
[docs/topics.md](https://github.com/atinfinity/toio_gazebo/blob/main/docs/topics.md).
