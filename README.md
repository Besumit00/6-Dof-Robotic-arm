# 6-DOF Robotic Arm — Build Project

A DIY 6 degrees-of-freedom robotic arm: 3D-printed, MG996R servo-driven, designed in Fusion 360.

## Contents

- `docs/6DOF_Robotic_Arm_Fusion360_Guide.md` — full component-by-component build guide, including a dedicated Fusion 360 sketching/auto-constraint troubleshooting section.
- `media/` — reference renders and a turntable video generated from an early STL pass.
- `cad-exports/` — put your STEP/STL exports from Fusion 360 here as you finish each component (see note below).

## Specs

- 6 DOF: base, shoulder, elbow, wrist pitch, wrist roll, gripper
- Servo: MG996R (x6)
- Max reach: ~450mm | Height: ~430mm
- Material: PLA/ABS, 3D printed
- Plate/arm thickness: 6mm (arms: 10mm)

## Keeping this repo in sync with Fusion 360

Fusion's cloud files aren't git-trackable directly. To version your progress:

1. In Fusion 360: Data Panel → right-click your design → **Export**.
2. Export as **STEP** (full parametric geometry) and/or **STL** (for printing).
3. Save into `cad-exports/`, named per component, e.g. `cad-exports/base_v1.step`.
4. Commit after each milestone (see commands below).

## Build Log

- [x] Base (bore + bearing seat + bolt pattern)
- [x] Shoulder Plate (boss + servo horn holes)
- [ ] Upper Arm
- [ ] Elbow Link
- [ ] Forearm
- [ ] Wrist Housing + Wrist Link
- [ ] Gripper
- [ ] Full assembly + joints
