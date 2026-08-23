# toio_msgs

## Introduction

`toio_msgs` provides the custom ROS 2 message types used by the toio interface.
They cover the parts of the cube that the standard messages do not: an indicator
color with its own lighting time, blink patterns and melodies the cube plays on
its own, and the cube's motion-detection notification.

Both [toio_ros2](toio_ros2.md) (the real cube) and [toio_gazebo](toio_gazebo.md)
(the simulation) publish and subscribe these messages, so the same commands work
against hardware and against the simulator.

## Messages

| message | used on | summary |
|---|---|---|
| `Led` | `/toio/led_timed` | An indicator color (`std_msgs/ColorRGBA`, 0.0-1.0 per channel) plus a `duration_ms` lighting time. `duration_ms` is taken in 10 ms units; `DURATION_UNLIMITED` (0) stays lit until the next command, up to `DURATION_MAX_MS` (2550) |
| `LedPattern` | `/toio/led_pattern` | A blink sequence the cube plays itself: `Led[] steps` (up to `STEPS_MAX` = 29) and `repeat`. `REPEAT_INFINITE` (0) repeats until the next indicator command |
| `Melody` | `/toio/melody` | A melody the cube plays itself: `MidiNote[] notes` (up to `NOTES_MAX` = 59) and `repeat` |
| `MidiNote` | (in `Melody`) | One note: `duration_ms` (10 ms units), `note` (`NOTE_MIN` 0 - `NOTE_MAX` 128, `NOTE_REST` = 128 for silence) and `volume` (0 mutes, any other value is full volume) |
| `MotionDetection` | `/toio/motion` | Motion-detection state: `horizontal`, `collision`, `double_tap`, `posture` (`POSTURE_*`) and `shake`. `collision` / `double_tap` are momentary events (true only in the notification that detects them) |

Full field-by-field definitions with the toio specification references are in the
message files: <https://github.com/atinfinity/toio_msgs/tree/main/msg>.

## Notes

- On the real cube ([toio_ros2](toio_ros2.md)) every message maps to a
  corresponding BLE characteristic of the cube.
- In the simulation ([toio_gazebo](toio_gazebo.md)) the LED and sound messages
  are honored (`ToioLedSystem` plugin and a host-side sound node), while
  `/toio/motion` is published as a fixed stub because Gazebo has no
  motion-detection equivalent. See
  [toio_gazebo interface support](https://github.com/atinfinity/toio_gazebo/blob/main/docs/toio_ros2_support.md).
