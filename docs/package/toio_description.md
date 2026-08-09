# toio_description

## Introduction

`toio_description` provides a 3D model of the [toio](https://toio.io/) for visualization.

![](https://raw.githubusercontent.com/atinfinity/toio_description/refs/heads/main/image/toio_description.png)

## Feature

- 3D model of toio described in [robot/toio.urdf.xacro](https://github.com/atinfinity/toio_description/blob/main/robot/toio.urdf.xacro)
- Publish `robot_description` topic and `tf` topic using [launch/robot_description.launch.py](https://github.com/atinfinity/toio_description/blob/main/launch/robot_description.launch.py)
- `namespace` and `frame_prefix` arguments for multi-robot setups

## Requirements

I checked this package on the following environment.

- Ubuntu 24.04
- ROS 2 Jazzy

## License

The 3D model is distributed under a different license from the source code.
Please see [License](../about/license.md) in detail.
