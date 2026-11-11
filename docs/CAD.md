# CAD plan (Fusion 360, Ultimaker S3/S5/S7)

## Design idea: two-part body

A 1.5 in display has only about 27 mm of visible screen, so on a body big enough to hold a Pi 5 it would look tiny and lose the BMO feel. So the robot is two parts:

- **Torso** (the character): small BMO-style rounded box with the display as the face, camera above it, arms on the sides, and the Nano inside.
- **Base** (the plinth): wider, flatter box that holds the Pi 5, active cooler, power jacks and wiring. The torso sits on top and cables run up through a slot.

```
        +-----------+
   arm -|   (cam)   |- arm        Torso   ~100 W x 115 H x 55 D mm
        |  +-----+  |             Display window with raised bezel
        |  |face |  |             3 optional button pockets under the screen
        |  +-----+  |
        |  o  o  +  |
        +-----------+
     +-----------------+
     |   Pi 5 / power  |          Base    ~140 W x 50 H x 100 D mm
     +-----------------+
```

All dimensions are **starting values. Measure the real parts with calipers before locking the model.**

## Print volume rule

Ultimaker S3 build volume is about 230 x 190 x 200 mm (about 215 x 190 x 200 with both print cores). S5 and S7 are larger. **Design every printed part to fit within 200 x 180 x 190 mm** and it will print on any of them.

## Parts to model

| Part | Approx. size (mm) | Key features |
|---|---|---|
| Torso front shell | 100 x 115 x 25 | Display window, camera window, button holes, front bezel |
| Torso rear shell | 100 x 115 x 32 | Nano mount, cable exit, screw posts |
| Display bracket | 40 x 40 x 8 | Holds the OLED module flush behind the window |
| Servo mounts (x2) | 30 x 15 x 15 | Pockets sized to the MG90S body, screw tabs |
| Arms (x2) | 60 long x 8 wide x 4 thick | Slot or hole for servo horn. Light, so the servo is not loaded |
| Camera holder | 30 x 30 x 12 | Fits Camera Module 3 board, lens opening, sliding privacy shutter |
| Base lower | 140 x 50 x 100 | Pi 5 standoffs, ventilation slots, jack cutouts |
| Base upper plate | 140 x 5 x 100 | Torso seat, cable slot |
| Optional parts | | Button caps, sensor plug, lens cover |

## Fusion 360 workflow (for a first model)

1. **Set up user parameters** (Modify, Change Parameters) before drawing anything. Use these so you can tune fits without remodelling:

| Parameter | Start value | Meaning |
|---|---|---|
| `wall` | 2.4 mm | Six 0.4 mm lines |
| `fit_clear` | 0.2 mm | Clearance for parts that must slide or fit |
| `oled_w`, `oled_h` | measure | OLED module outline |
| `oled_win` | 27 mm square | Visible window, slightly under the active area |
| `servo_w`, `servo_d`, `servo_h` | about 23 x 12.5 x 22.5 | MG90S body, measure yours |
| `torso_w`, `torso_h`, `torso_d` | 100, 115, 55 | Overall torso |
| `corner_r` | 10 mm | BMO-like rounded corners |

2. **Torso:** sketch a rounded rectangle, extrude to `torso_d`, then Shell inward by `wall`. Split the body in two with a plane at about 45 percent depth (front and rear shells).
3. **Display window:** sketch a square of `oled_win`, cut through the front shell, add a 1.5 mm raised bezel around it.
4. **Camera and sensor windows:** above the display, a 10 mm round hole for the lens and a second 10 mm plugged hole beside it for the future temperature sensor.
5. **Screw posts:** four M2.5 heat-set insert posts in the rear shell and matching holes in the front.
6. **Servo pockets:** model a box cut using the servo dimensions plus `fit_clear`, on each side wall, with the shaft facing outward through a hole.
7. **Arms:** a simple rounded bar with a hole sized for the servo horn. Make a second mirrored copy.
8. **Base:** a box shell, standoffs for the Pi 5 (mounting holes 58 x 49 mm, M2.5), ventilation slots on the sides and top, a cutout for the USB-C supply and the servo DC jack.
9. **Export:** right-click each body, Save as Mesh, 3MF or STL, then slice in Cura.

## Print settings (Cura, Ultimaker profile)

| Setting | Value |
|---|---|
| Material | PLA (teal for shell). PETG is an option for the arms if PLA snaps |
| Nozzle | 0.4 mm AA core |
| Layer height | 0.15 mm for shells and bezel, 0.2 mm for the base |
| Wall line count | 4 |
| Infill | 15 to 20 percent gyroid |
| Supports | Avoid by orienting well. Use tree supports only where needed (window overhangs). The S3 can use PVA or breakaway in its second core |
| Orientation | Front shell face-down for a clean screen side. Rear shell open side up. Arms flat |
| Adhesion | Brim 5 mm on tall thin parts |

## Fit-check approach

Print the cheap parts first, before any full shell:

1. Display bracket plus a 30 mm test frame (about 15 minutes)
2. Servo pocket test block (about 20 minutes)
3. Arm and horn test (about 20 minutes)
4. Torso front shell only, to check the window and camera hole
5. Full torso and base

Expect two or three iterations. Keep tolerances in parameters so changes are quick.

## Arm design for future ball joints

Keep the arm design modular: the shoulder is a separate small part that holds the servo horn, and the arm plugs into it. For ball joints later, only the shoulder bracket and the arm socket change; the torso side wall keeps the same screw pattern.

## Assembly order

1. Fit the display and camera into the front shell
2. Fit the servos into their pockets, attach arms last (after centring the servos at 90 degrees)
3. Mount the Nano in the rear shell and wire up
4. Close the torso, run the camera ribbon and USB cable down through the base slot
5. Fit the Pi, cooler and jacks in the base, then seat the torso

## Open checks

- Torso depth: confirm the OLED module, Nano and wiring fit in about 55 mm. If not, go to 65 mm.
- Camera cable bend radius through the base slot.
- Pi 5 cooling with the vents closed, test under load.
