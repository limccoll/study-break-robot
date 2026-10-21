# Study-Break Robot 🤖

A small desk robot that notices when you're getting tired while studying and nudges you to take a break, like a pet with a face.

Built for the **Edinburgh Award** at the University of Edinburgh School of Informatics Makerspace (Oct 2026 – summer 2027).

## What it does
- Watches for signs of study fatigue, such as eye closure and yawning, using **on-device computer vision**.
- Shows its mood on an animated OLED face and does a short dance when it's time for a break.
- Syncs session summaries to a phone app over **Bluetooth Low Energy**.

## Privacy by design
Video is processed on the robot and **never stored or sent anywhere**. Only session summaries (start time, length, breaks) leave the device.

## Hardware
| Part | Purpose |
|---|---|
| Raspberry Pi 5 + Camera Module 3 | Face tracking on the robot itself |
| 2.42" OLED display | Animated expressions |
| 3 micro servos + PCA9685 driver | Break-time dance |
| 3D-printed body | Vented shell, lens cover, servo mounts |

## Tech
Python · MediaPipe · OpenCV · Raspberry Pi OS · BLE · Flutter (companion app)

## Roadmap
- [ ] Nov 2026: camera test in evening light, fatigue detection working, accuracy check
- [ ] Dec 2026: OLED face and moods, button, idle / watching / break logic
- [ ] Jan–Feb 2027: Bluetooth link and a basic app (start, stop, end-of-day summary)
- [ ] Mar 2027: servo dance and first printed body, then feature freeze
- [ ] Apr–May 2027: tuning, final body, demo video

## Build log
See [`BUILD_LOG.md`](BUILD_LOG.md).
