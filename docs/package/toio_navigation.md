# toio_navigation

## Introduction

`toio_navigation` is ROS 2 package for navigation2 using [toio](https://toio.io/).

<video style="width: 100%" muted="" controls="" alt="type:video">
   <source src="../videos/toio_ros2_navigation_demo.mp4" type="video/mp4">
</video>

## Feature

`toio_navigation` supports navigation using [navigation2](https://nav2.org/).
And, `toio_navigation` bundled nav2 parameters tuned for toio.

- Navigation of a single toio using nav2
- Multi-robot navigation(one nav2 stack per robot)
- Avoidance of the peer robots using `peer_robot_costmap_publisher`
- Maps of the A4/A3 play mat(with and without obstacle)

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
- toio_ros2 <https://github.com/atinfinity/toio_ros2>
- toio_description <https://github.com/atinfinity/toio_description>

## Maps

Maps for the play mat are stored in [maps](https://github.com/atinfinity/toio_navigation/tree/jazzy/maps).

|map|Description|
|:---|:---|
|toio_a4_map.yaml|A4 play mat|
|toio_a4_map_with_obstacle.yaml|A4 play mat with obstacle|
|toio_a3_map.yaml|A3 play mat|
|toio_a3_map_with_obstacle.yaml|A3 play mat with obstacle|

If you use Gazebo simulator, please add `use_sim_time:=True` to the launch command.

## Multi-robot navigation

[launch/toio_multi_navigation.launch.py](https://github.com/atinfinity/toio_navigation/blob/jazzy/launch/toio_multi_navigation.launch.py) launches one nav2 stack per robot.
The robot list defaults to `toio1,toio2` and can be changed with the `robots` argument(e.g. `robots:=toio1,toio2,toio3`), and every robot gets all the other robots as peers for `peer_robot_costmap_publisher`.

Each stack is separated by ROS namespace, and the TF frames are separated by `frame_prefix`(e.g. `toio1/base_link`) on the shared `/tf`.
The `map` frame is shared by all robots.

One RViz instance is launched per robot([rviz/nav2_multi.rviz](https://github.com/atinfinity/toio_navigation/blob/jazzy/rviz/nav2_multi.rviz)).
Each RViz shows both robot models on the shared map, while the map, paths and costmaps belong to its own robot.
You can send an independent goal to each robot from its own RViz("Nav2 Goal" tool), or via the `navigate_to_pose` action of each namespace.

### Avoidance of the peer robots

`peer_robot_costmap_publisher`(launched per robot) looks up the poses of the peer robots from TF and publishes an `OccupancyGrid`(`peer_robots_costmap`) which marks each peer as a filled rectangle(rotated by the yaw of the peer) on top of a copy of the static map.
The `peer_robot_layer`(a `nav2_costmap_2d::StaticLayer`) of the local/global costmaps consumes it, so the planner and the controller avoid the other robots.

Parameters of `peer_robot_costmap_publisher` are stored in [params/nav2_params.yaml](https://github.com/atinfinity/toio_navigation/blob/jazzy/params/nav2_params.yaml).

|name|Type|Default|Description|
|:---|:---|:---|:---|
|footprint_length|double|0.032|length of the rectangular footprint of a peer robot(meter)|
|footprint_width|double|0.032|width of the rectangular footprint of a peer robot(meter)|
|update_rate|double|10.0|publish rate(Hz)|
|peer_base_frames|string|-|comma-separated TF frames of the peer robots(e.g. `toio2/base_footprint`). Set automatically by `toio_multi_navigation.launch.py`|

The node is launched as part of `navigation.launch.py`.
And, it can also be launched standalone using [launch/peer_robot_costmap_publisher.launch.py](https://github.com/atinfinity/toio_navigation/blob/jazzy/launch/peer_robot_costmap_publisher.launch.py).
