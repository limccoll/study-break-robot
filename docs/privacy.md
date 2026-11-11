# Privacy

The robot points a camera at someone's face and the phone app reads health and usage data, so privacy is a core design rule, not an add-on.

## What happens to the camera feed

- Frames are processed in memory on the Raspberry Pi and thrown away straight after.
- No images or video are ever written to disk or sent anywhere, and the code must not log them.
- A sliding physical shutter on the camera holder lets the user block the lens.
- The app has a camera on/off switch that tells the Pi to stop capturing.

## What leaves the Pi

Only derived numbers go over the USB cable to the Nano: blink rate, eye-closure fraction, yawn flag, an attention score and a state word (`ok`, `tired`, `distracted`, `absent`). No face data.

## What leaves the robot

Over Bluetooth LE, to the paired phone only:
- current robot state and timer
- one summary per minute during a session: blinks, eye closure, attention, yawns

No cloud, no accounts, no internet needed.

## Phone data

- **Apple Health** data (steps, sleep, heart rate) is read on the device and used only to compute a mood score. It is not uploaded or stored on the robot, which only receives the 0 to 100 score.
- **Screen Time** data stays inside Apple's sandbox. The app only learns that a threshold was crossed, not which app or exact minutes.
- Session stats are stored locally on the phone and can be deleted in the app.

## Accuracy

Fatigue and distraction detection will be tested honestly on real study sessions and the results reported, including where it gets things wrong (for example glasses, poor lighting, or looking down at notes). The robot is a study aid, not a medical device, and its tiredness signal is not a health measurement.
