# Protocols

Version 1. Small, readable JSON so it is easy to debug on a serial console or a BLE tester app (nRF Connect, LightBlue). Binary can replace it later if needed.

## 1. Pi to Nano (USB serial)

- Pi opens `/dev/ttyACM0` (Nano ESP32 over USB-C), 115200 baud (ignored by USB CDC but set it anyway).
- One JSON object per line, UTF-8, `\n` terminated, under 200 bytes.
- Every object has `"v":1` and `"type"`.

### Pi to Nano

```json
{"v":1,"type":"vision","face":true,"state":"tired","blink_pm":9,"perclos":0.18,"ear":0.17,"yawn":false,"attn":0.92,"away_s":0}
```

| Field | Meaning |
|---|---|
| `face` | A face is currently found |
| `state` | `ok`, `tired`, `distracted`, `absent` |
| `blink_pm` | Blinks per minute (60 s window) |
| `perclos` | Fraction of the last 60 s with eyes mostly closed (0 to 1) |
| `ear` | Current eye aspect ratio |
| `yawn` | Yawn in the last 5 s |
| `attn` | Attention score, 1 = looking at screen, 0 = away |
| `away_s` | Seconds continuously looking away |

Sent at 2 Hz while a session is active, once per 5 s otherwise. Also sends `{"v":1,"type":"hello","fw":"0.1.0"}` on startup.

### Nano to Pi

```json
{"v":1,"type":"cfg","cam":true,"notes":false,"thr":{"perclos":0.15,"away_s":8,"ear":0.2}}
{"v":1,"type":"session","active":true}
{"v":1,"type":"calibrate"}
```

The Pi applies `cfg` immediately, and shuts the camera off when `cam` is false.

## 2. Nano to iPhone (BLE GATT)

Device name: `StudyBuddy`. One custom service, four characteristics. Replace placeholder UUIDs with generated ones once and keep them in `docs/PROTOCOL.md` and in both code bases.

Service UUID: `5B1D0001-6F0A-4C1E-9A3E-2F7D8A1C0B00`

| Name | UUID | Properties | Content |
|---|---|---|---|
| State | `5B1D0002-...` | Read, Notify | Current robot state |
| Command | `5B1D0003-...` | Write without response | Commands from the phone |
| Config | `5B1D0004-...` | Read, Write | Colours and thresholds, saved on the Nano |
| Stats | `5B1D0005-...` | Notify | One summary per minute |

Request an MTU of at least 185 on iOS (it negotiates automatically). Keep each message under 180 bytes so it fits one packet.

### State (notify on every change, and at least every 5 s)

```json
{"phase":"study","expr":"focus","remaining_s":1180,"round":2,"mood":72,"fatigue":0.2,"attn":0.9,"pi":true}
```

`phase`: `idle`, `study`, `break`, `longbreak`, `paused`. `pi`: whether the Pi is connected.

### Command (write)

```json
{"cmd":"start","study_s":1500,"break_s":300,"long_s":900,"rounds":4}
{"cmd":"pause"}
{"cmd":"resume"}
{"cmd":"skip"}
{"cmd":"stop"}
{"cmd":"expr","name":"happy"}
{"cmd":"arm","l":90,"r":90}
{"cmd":"pose","name":"wave"}
{"cmd":"mood","v":72}
{"cmd":"alert","kind":"screentime","tier":2}
{"cmd":"calibrate"}
```

Arm angles are degrees 0 to 180 and are clamped to the configured min and max. Poses: `rest`, `wave`, `cheer`, `shrug`, `point`.

### Config (read/write, persisted in flash)

```json
{"face":"#7FD6C2","eye":"#101010","mouth":"#101010","bg":"#000000","accent":"#FFFFFF","bright":80,"arm_min":10,"arm_max":170,"perclos":0.15,"away_s":8,"ear":0.2,"cam":true,"notes":false}
```

Colours are `#RRGGBB`, converted to RGB565 on the Nano. `bright` is 0 to 100.

### Stats (notify, once per minute during a session)

```json
{"t":1783000000,"blinks":14,"perclos":0.08,"attn":0.87,"yawn":0,"phase":"study"}
```

`t` is unix seconds. The phone sets the Nano's clock with `{"cmd":"time","t":...}` on connect.

## 3. Behaviour rules

- The Nano owns the timer and the expression. Commands from the phone override the automatic expression for 10 s, then it returns to automatic.
- If no Pi message arrives for 10 s, `pi` is false and the robot runs on timer and phone input only.
- If BLE disconnects, the robot continues autonomously and the phone resyncs on reconnect.
- Alerts have a cooldown (default 5 min for tired, 2 min for distracted) so the robot does not nag.
