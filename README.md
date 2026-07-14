# 6-DOF Robotic Arm — Build Project

My first CAD project: a 6 degrees-of-freedom robotic arm, designed from scratch and modeled in Fusion 360 — 3D-printed, servo-driven.

I started with zero prior CAD experience. This repo tracks the build from a rough paper concept through to a full parametric assembly.

## Specs

- **DOF:** 6 (base rotation, shoulder, elbow, wrist pitch, wrist roll, end effector)
- **Servo:** MG996R (x6)
- **Max reach:** ~450mm | **Height:** ~430mm
- **Material:** PLA/ABS, 3D printed
- **Plate/arm thickness:** 6mm (arms: 10mm)

## Current status: end effector is a placeholder

The gripper in the current assembly is a **rigid, non-actuated end effector** — not a functional gripper yet. A proper actuated jaw mechanism is planned as the next iteration. Flagging this clearly so the repo doesn't overstate where the project actually is.

## Repo structure

```
docs/           Build guide (component-by-component instructions,
                including Fusion 360 sketch/constraint troubleshooting)
media/          Reference renders + motion turntable video
cad-exports/    STEP files exported from Fusion 360, one per component
```

## Build log

| # | Component | Status |
|---|---|---|
| 00 | Full Assembly | Done |
| 01 | Base | Done |
| 02 | Shoulder Plate | Done |
| 03 | Upper Arm | Done |
| 04 | Elbow Link | Done |
| 05 | Forearm | Pending |
| 06 | Wrist Housing | Done (built as `link4`, needs export/rename) |
| 07 | Wrist Link | Done (built as `Wrist`, needs export/rename) |
| 08 | End Effector (rigid placeholder) | Done (functional gripper planned) |
| - | Joints + motion simulation | Done |

## Keeping this synced with Fusion 360

Fusion's cloud files aren't git-trackable directly. Workflow for updates:

1. In Fusion 360: Data Panel -> right-click design -> Export -> STEP.
2. Save into `cad-exports/`, named `NN_ComponentName.step` to keep build order.
3. Commit with a short message describing what changed.
