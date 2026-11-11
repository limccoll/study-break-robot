# Handoff: build spec for Claude Code

Read this file first, then [PLAN.md](PLAN.md), [PARTS.md](PARTS.md) and [PROTOCOL.md](PROTOCOL.md). They are the source of truth. If code and docs disagree, stop and ask.

## Project in one paragraph

Study Buddy is a BMO-inspired desk robot. A Raspberry Pi 5 does camera vision (blink rate, eye closure, yawns, gaze) and sends derived metrics over USB serial to an Arduino Nano ESP32. The Nano runs the Pomodoro timer and state machine, draws faces on a 1.5 in colour SSD1351 OLED, moves two arm servos, and speaks Bluetooth LE to a native SwiftUI iPhone app. The app controls colours, expressions and arm poses, reads HealthKit and Screen Time threshold events, and sends a mood score to the robot.

## The user

A student who is new to hardware and has some coding experience. Explain wiring and tool steps plainly. Never assume a tool is installed. The user has a paid Apple Developer account, Xcode on a Mac, access to an Ultimaker S3/S5/S7, and will model in Fusion 360. Hardware is tested by the user, not by you: when a step needs physical hardware, write the test steps and wait.

## Target repo layout

```
firmware/nano-esp32/     Arduino sketch (PlatformIO or Arduino IDE project)
vision/pi/               Python vision service
ios/StudyBuddy/          Xcode project (SwiftUI)
cad/                     Fusion 360 exports and parameter notes (STL/3MF only, no huge binaries)
docs/                    Specs (these files)
tools/                   Small helper scripts (serial monitor, BLE test)
```

## Global rules

- Follow [PROTOCOL.md](PROTOCOL.md) exactly. If it must change, update the doc in the same commit and bump nothing silently.
- Only make changes directly requested by the current milestone. Do not add features, extra abstractions or new dependencies without asking.
- **Ask before:** adding any dependency or library, changing the BLE UUIDs or JSON fields, deleting files, touching signing or entitlements, running anything that needs elevated privileges.
- Never commit secrets, Apple team IDs, provisioning profiles or video/image captures. `*.jpg` and `*.mp4` are already in `.gitignore`.
- **Privacy rule:** the Pi must never save, log or transmit camera frames. Only derived numbers leave the vision process.
- Keep each milestone to one pull-request-sized change. After each step, print `DONE: <what was completed>`.
- Write a short note in [BUILD_LOG.md](../BUILD_LOG.md) at the end of each milestone.
- Dates: today is 2026-11-11. The feature freeze is March 2027.
- Do not create a pull request unless asked.

## Libraries (confirm before adding)

| Area | Proposed | Why |
|---|---|---|
| Pi vision | `mediapipe`, `opencv-python`, `pyserial`, `picamera2` | Face landmarks, capture, serial |
| Nano display | `Adafruit_GFX` + `Adafruit_SSD1351` (or `Arduino_GFX`) | SSD1351 support; use a 16-bit canvas for flicker-free drawing |
| Nano servos | `ESP32Servo` | Servo PWM on ESP32 |
| Nano BLE | `NimBLE-Arduino` or `ArduinoBLE` | GATT server. Verify which one the Nano ESP32 core supports well |
| Nano JSON | `ArduinoJson` | Parsing commands |
| iOS | CoreBluetooth, HealthKit, FamilyControls, DeviceActivity, ManagedSettings, SwiftData, Swift Charts | All Apple first-party |

## Milestones

Each milestone lists the starting state, the target, what you may touch, the stop conditions and how the user verifies it.

### M0: Scaffold

- Start: repo has docs only.
- Target: the layout above, a `.gitignore` update, and a short `README` in each folder describing how to build or run it.
- Touch: new folders and READMEs only.
- Stop and ask if: you want to pick PlatformIO versus Arduino IDE. Recommend Arduino IDE for a beginner unless there is a reason.
- Done when: folders exist and each README says how to set up its toolchain.

### M1: Vision service on the Pi

- Start: empty `vision/pi/`.
- Target: a Python service that opens the camera, runs MediaPipe Face Landmarker, and computes EAR, blink count, blink rate (60 s window), PERCLOS, yawn (MAR), head yaw/pitch and `attn`, then classifies `ok`, `tired`, `distracted` or `absent`. Includes a `--debug` window that draws landmarks and numbers, and a `--dry-run` that prints the JSON line instead of using serial. A 2-minute calibration mode stores personal baselines in a local JSON file.
- Touch: `vision/pi/**` only.
- Forbidden: writing frames to disk, network calls, cloud models.
- Stop and ask if: Pi performance is below about 15 fps, or accuracy looks poor in low light.
- Done when: the user can see values change when closing eyes, yawning and looking away, and the console prints valid `vision` lines from [PROTOCOL.md](PROTOCOL.md).
- Also produce: a `vision/pi/ACCURACY.md` template for logging test results (lighting, glasses, outcome).

### M2: Nano face and arms

- Start: empty `firmware/nano-esp32/`.
- Target: a sketch that initialises the SSD1351 and two servos, renders the six faces (`happy`, `focus`, `excited`, `sleepy`, `worried`, `lockin`) from a colour palette struct, handles blinking, animates arm poses (`rest`, `wave`, `cheer`, `shrug`, `point`) with smooth easing, and accepts commands over the USB serial console using the Command JSON from the protocol.
- Touch: `firmware/nano-esp32/**`.
- Rules: pins from [PARTS.md](PARTS.md) as named constants at the top of one file; clamp servo angles to config min/max; detach servos when idle; no blocking `delay()` in the main loop.
- Stop and ask if: a pin in PARTS.md conflicts with the Nano ESP32 pinout.
- Done when: typing `{"cmd":"expr","name":"sleepy"}` in the serial monitor changes the face, and `{"cmd":"pose","name":"wave"}` waves the arms.

### M3: Pomodoro and state machine

- Target: the Nano runs the study/break/long-break timer, automatic expression selection, alert cooldowns, optional button handling (pins D2 to D4, safe if no buttons are fitted), and saves config in flash.
- Done when: a full 25/5 cycle runs with expressions and a break wave, and settings survive a reboot.

### M4: Pi to Nano link

- Target: the Pi writes `vision` lines to the Nano, the Nano parses them and reacts: `tired` leads to `sleepy` plus a break suggestion, `distracted` for more than `away_s` leads to `lockin` plus a pointing arm. Nano sends `cfg` and `session` back. Degrades cleanly when the Pi is absent.
- Done when: closing eyes for about 3 s sleepifies the robot, and unplugging the Pi returns it to timer-only mode within 10 s.

### M5: BLE

- Target: the GATT service from [PROTOCOL.md](PROTOCOL.md), with State, Command, Config and Stats working. Test with a generic BLE app before any iOS code.
- Stop and ask if: UUIDs need to change or MTU limits force a message format change.
- Done when: nRF Connect can change colours and trigger poses, and State notifications arrive.

### M6: iPhone app, control

- Target: SwiftUI app with a connection screen, timer screen, colour pickers, expression buttons, arm sliders, saved poses, settings, and a stats screen with Swift Charts. Auto-reconnect and state restoration.
- Rules: app code needs no network access and no accounts. Camera on/off is a visible switch.
- Stop and ask before: changing capabilities, bundle ID, signing or Info.plist usage strings.
- Done when: every control works from the phone, and the app reconnects after the robot is power-cycled.

### M7: Health and mood score

- Target: HealthKit read access for steps, sleep analysis and heart rate with proper usage descriptions, the mood formula from [PLAN.md](PLAN.md) with adjustable weights, and sending `mood` commands.
- Done when: changing the step goal or sleep input visibly changes the baseline face.

### M8: Screen Time

- Target: Family Controls authorisation (`.individual`), `FamilyActivityPicker`, a DeviceActivityMonitor extension with threshold events, App Group handoff to the main app, and `alert` commands to the robot. Include a manual "screen time tier" debug slider so the robot reaction works even before Apple approves distribution.
- Reminder: raw usage minutes are **not** readable by the app. Use threshold tiers only. Do not attempt workarounds.
- Stop and ask before: anything involving entitlements or requesting Apple approval.
- Done when: crossing a configured threshold makes the robot show `worried`.

### M9: CAD support (optional)

- The user models in Fusion 360. You may write: a parameters table for [CAD.md](CAD.md), a printable fit-check list, and a Python or OpenSCAD test jig only if asked. Do not generate final shell models unprompted.

## Test checklist

- [ ] Vision lines valid JSON, under 200 bytes, 2 Hz
- [ ] No image files created anywhere on the Pi after a session
- [ ] All faces render at the 128x128 resolution without flicker
- [ ] Servos never hit their end stops and are detached when idle
- [ ] Robot works with no Pi, no phone, or neither
- [ ] BLE reconnects after phone sleep, robot power cycle and app relaunch
- [ ] Config survives a reboot
- [ ] Two-hour unattended session with no crash or Pi overheating

## First prompt to give Claude Code

> Read `docs/HANDOFF.md`, `docs/PLAN.md`, `docs/PARTS.md` and `docs/PROTOCOL.md`. Do milestone M0 only. Report what you created and ask me any questions before starting M1.
