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

`toio_gazebo` provides Gazebo environment for toio.

- Control using the `/cmd_vel` topic
- Publish `/tf` topic(position and orientation of toio)
- Multi-robot simulation separated by ROS namespace and TF frame prefix
- Spawn an additional robot into a running simulation

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

Each robot is separated by ROS namespace and TF frame prefix.

|robot|cmd_vel|joint_states|TF|
|:---|:---|:---|:---|
|toio1|`/toio1/cmd_vel`|`/toio1/joint_states`|`map -> toio1/center`|
|toio2|`/toio2/cmd_vel`|`/toio2/joint_states`|`map -> toio2/center`|

To spawn an additional robot into a running simulation, use [launch/spawn_toio.launch.py](https://github.com/atinfinity/toio_gazebo/blob/main/launch/spawn_toio.launch.py).

```bash
ros2 launch toio_gazebo spawn_toio.launch.py namespace:=toio3 robot_name:=toio3 frame_prefix:=toio3/ x_pose:=0.145 y_pose:=-0.095
```
