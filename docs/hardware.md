# Hardware

| Part | Notes | Source |
|---|---|---|
| Raspberry Pi 5 | Runs MediaPipe face tracking | Makerspace loan or buy |
| Camera Module 3 | Needs a test in real evening light | Makerspace loan or buy |
| 2.42" OLED display (SSD1309, I²C) | Animated face | Buy |
| 3 × micro servos (SG90 or similar) | Dance movements | Makerspace loan or buy |
| PCA9685 16-channel servo driver | Runs the servos from I²C | Buy |
| Push button | Start or stop a session without the phone | Buy |
| USB-C power supply (27 W, official Pi 5) | Powers the Pi and servos | Buy |
| 3D-printed enclosure | Vented shell, lens cover, servo mounts | Makerspace printers |

**Estimated cost:** about £180 bought outright, or £55–65 if the makerspace lends the Pi, camera and servos.

## Wiring plan (draft)

- OLED and PCA9685 share the I²C bus (GPIO 2 SDA, GPIO 3 SCL).
- Servos are powered from the PCA9685 V+ rail, not from the Pi's 5 V pin.
- Button on a GPIO pin with an internal pull-up.

*To be confirmed during the December build.*
