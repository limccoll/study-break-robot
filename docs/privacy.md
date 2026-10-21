# Privacy

The robot points a camera at someone's face, so privacy is a core design rule, not an add-on.

## What happens to the camera feed
- Frames are processed in memory on the Raspberry Pi and thrown away straight after.
- No images or video are ever written to disk or sent anywhere.
- A physical lens cover lets the user block the camera.

## What leaves the robot
Only session summaries, sent to the phone app over Bluetooth Low Energy:
- session start and end time
- number of breaks suggested and taken
- a simple tiredness level over time, not raw face data

## Accuracy
Fatigue detection will be tested honestly on real study sessions and the results reported here, including where it gets things wrong (for example glasses, poor lighting, or looking down at notes).
