# 6-DOF Educational Robotic Arm — Complete Fusion 360 Build Guide

**Goal:** Design a parametric 6-DOF robotic arm from scratch, 3D-printable, ready for later ROS2 integration.

---

## PART 0 — Project Setup (do this first)

1. **New Project**: Data Panel → New Project → name it `RoboticArm_6DOF`.
2. **Units**: Utilities → Document Settings → set to `mm`. Never mix units mid-project.
3. **Design history**: Make sure you're in **Parametric** modeling (default), not Direct Modeling — check the timeline bar is visible at the bottom. This is what lets you edit dimensions later.
4. **Naming discipline**: Rename every sketch and feature as you make it (double-click in timeline). "Sketch14" six months from now tells you nothing.
5. **Component structure**: Create each of the 10 parts as its own **Component** (not just a body) inside one master assembly file. Right-click root → New Component. This is what lets you assign joints later.

*Alternative*: Some designers build each part in a **separate file** and use "Insert > Insert into Current Design" to assemble. This is cleaner for version control and sharing files individually, but harder to keep parametrically linked (e.g. hole patterns matching between mating parts). For a 10-part educational project, **single multi-component file** is recommended — easier joint setup, one file to manage.

---

## PART 1 — SKETCHING MASTERCLASS (your main pain point)

This is the section to read twice. Almost everyone hits the exact wall you're describing: you draw a line, Fusion silently snaps on a horizontal/vertical/coincident constraint, and now when you try to drag or dimension something, geometry jumps in unexpected directions because it's fighting invisible constraints.

### Why this happens
Fusion's sketch engine has **auto-constrain / auto-project** turned on by default. As you sketch, it guesses your intent:
- Draw a line roughly horizontal → it snaps a **Horizontal** constraint.
- Endpoint near another endpoint → **Coincident** constraint.
- Line near tangent to an arc → **Tangent** constraint.
- Two lines similar length → sometimes **Equal**.

These are invisible unless you turn on constraint markers, so five auto-constraints can pile onto a shape you thought was "just a rough sketch," and now it won't move the way you expect.

### Fix #1 — See what's actually constrained
- Sketch → bottom toolbar → click the **constraint visibility icon** (small icon that looks like a padlock/marks), or press **Sketch Palette → Show Constraints**.
- Every little yellow/green icon next to a line (⊥ ‖ — | ⊙ etc.) is a constraint. Hover over one to see its type; click to select it; press **Delete** to remove it.
- Fully constrained geometry turns **black** (or dark blue in some themes). Under-constrained geometry stays **blue**. This color, not "does it look right," is your source of truth.

### Fix #2 — Suppress auto-constraints *while drawing*
- **Hold Ctrl (Windows) / Cmd (Mac)** while dragging a line/arc — this temporarily disables auto-constrain snapping for that action. This is the single most useful habit to build.
- Watch the little constraint glyphs that pop up near your cursor as you draw (a small horizontal/vertical/coincident icon appears right before it locks in) — that's Fusion telling you what it's about to snap. If you don't want it, tap Ctrl/Cmd before releasing the click.

### Fix #3 — Turn off the specific auto-constraints you don't want globally
- Go to **Preferences (the gear icon, or Fusion menu → Preferences on Mac) → Sketch**.
- Uncheck: **"Automatic constraints"** section — you can toggle off horizontal/vertical, coincident, tangent individually.
- Also consider unchecking **"Autoproject edges for sketch creation"** if you find edges from other geometry are getting pulled into your new sketch unexpectedly.
- This is a global setting, so if you like auto-constrain for quick sketches but hate it for precision parts, you'll be toggling this — that's normal and fine.

### Fix #4 — Build sketches in the right order (this prevents 90% of fights)
1. **Draw construction geometry first** (centerlines, reference points) — set line type to "Construction" (keyboard shortcut `X`, or the icon in the sketch toolbar). Construction lines don't extrude but they give dimensioning anchors and don't confuse auto-constrain logic with real geometry.
2. **Rough in the shape loosely** — don't worry about it looking exact. Use Ctrl/Cmd while drawing to avoid unwanted snaps.
3. **Add dimensions before adding manual constraints.** A dimension (press `D`) pins geometry in a very predictable way. Once dimensioned, add only the constraints you're missing (symmetry, tangency, equal) deliberately by selecting entities and clicking the constraint icon — never let auto-constrain do it for you on final geometry.
4. **Check the constraint count vs. degrees of freedom.** Fusion tells you at the bottom status bar: "Fully constrained" or "X DOF remaining." Don't extrude until it says fully constrained (unless you intentionally want an adjustable sketch, which is rare for machine parts).

### Fix #5 — When a sketch is already a tangled mess
- Select all sketch geometry → right-click → **"Remove all constraints"** — nukes everything back to raw unconstrained geometry so you can restart the constraining process cleanly without redrawing the shapes.
- Alternatively, just delete the sketch and start over using the Ctrl/Cmd-while-drawing habit — often faster than untangling than fixing an existing sketch, especially early on while you're building the habit.

### Fix #6 — Use "Sketch Dimensions first" workflow (alternative philosophy)
Some engineers avoid the auto-constrain fight entirely by **drawing everything as disconnected line segments** (no snapping to endpoints at all — draw each line in open space), then manually applying Coincident constraints between endpoints by selecting two points and pressing `C`, then dimensioning. Slower to draw, but you have 100% control and no surprise constraints. This is worth doing for your first 2-3 parts until Ctrl/Cmd-drag becomes muscle memory, then switch back to the faster auto-assisted flow.

**Recommended for you specifically:** Do your next 3 sketches with auto-constraints OFF entirely (Fix #3), and manually add every constraint and dimension. This rebuilds the mental model of what's happening. Turn auto-constrain back on once it stops surprising you.

---

## PART 2 — Suggested Parameters (from your brief)

Set these up FIRST as **User Parameters** (Modify → Change Parameters → User Parameters), not hardcoded numbers in sketches. This is the core of "parametric modeling" — change one number, whole model updates.

| Parameter | Value |
|---|---|
| `base_plate_L` / `base_plate_W` | 120 x 120 mm |
| `base_plate_T` | 6 mm |
| `base_cyl_D` | 100 mm |
| `base_cyl_H` | 70 mm |
| `arm_thickness` | 10–12 mm |
| `upper_arm_L` | 170 mm |
| `forearm_L` | 140 mm |
| `wrist_link_L` | 90 mm |
| `pivot_hole_D` | 8 mm |
| `servo_hole_D` | 3.2 mm |

*Alternative*: instead of User Parameters, some people just dimension sketches with hard numbers and edit each one when needed. This works for a one-off model but breaks the "change once, updates everywhere" benefit — with 10 mating components sharing hole spacing and pivot diameters, **use User Parameters**, no real alternative here that scales.

---

## PART 3 — Component-by-Component Workflow

For every component below, follow this loop (from your brief, expanded):

1. **Analyze function** — what loads does it see, what does it connect to, what needs to rotate.
2. **Create sketch** — on the correct plane (usually XY for flat parts, or Right/Front for profiles that get revolved).
3. **Fully constrain** — using Part 1's method. Confirm "Fully constrained" in status bar.
4. **Extrude/Revolve using parameters** — reference the User Parameters, not raw numbers.
5. **Fillets/chamfers** — add last, after the base shape is validated. (Fillets applied too early make later sketch edits fail — "failed to compute" errors — because the fillet geometry depends on edges that move.)
6. **Assemble with joints** — covered in Part 4.
7. **Check motion** — drag-test the joint in the assembly.
8. **Prepare for manufacturing** — wall thickness check (min ~1.5–2mm for PLA/PETG), fillet sharp internal corners, orient for printing.

### 1. Base
- Sketch: circle Ø`base_cyl_D` on XY plane, centered at origin (use the origin point directly — free coincident constraint, not a snap-guess).
- Extrude `base_cyl_H`.
- Second sketch on top face: bolt circle pattern for servo mount + central bearing bore.
- *Alternative*: solid base cylinder vs. a **ribbed/hollow base** (shell it, add internal ribs) — hollow saves ~40% filament and print time with negligible strength loss for an educational arm. Recommended if you have a home printer with limited budget/time.

### 2. Rotating Shoulder Plate
- Sits on the base's central bearing/bushing (608 bearing, 8mm bore, 22mm OD — check this matches your `pivot_hole_D`... if using a 608 bearing, actually set the pivot bore parameter to 8mm shaft-side and add a separate 22mm bearing seat, don't conflate the two).
- Sketch: plate profile with mounting holes for the shoulder servo horn.
- *Alternative*: direct **servo horn screw-mount** (light, simple, cheaper) vs. **bearing-supported plate** (608 bearing) which takes the radial/axial load off the servo shaft — strongly recommended for the shoulder joint specifically, since it carries the whole arm's weight. Use the bearing here even if you skip bearings elsewhere.

### 3. Upper Arm
- Length `upper_arm_L` = 170mm, thickness `arm_thickness`.
- Sketch as a construction centerline first (170mm long) with pivot circles (Ø`pivot_hole_D`) at each end, THEN sketch the arm outline referencing those circles tangentially. This is the construction-geometry-first method from Part 1.
- *Alternative*: straight rectangular arm (simplest, easiest to print/sketch) vs. **I-beam / truss profile** (lighter, stiffer per gram, but much harder to sketch and constrain, and harder to print without supports). For your first pass, go rectangular with rounded ends; revisit truss profile later once comfortable with sketching.

### 4. Elbow Link
- Connects upper arm to forearm. Short link, two pivot bores.
- *Alternative*: single-link elbow vs. **double-plate (clevis) elbow** — clevis (two parallel plates sandwiching the forearm) is more rigid and avoids side-loading the servo shaft. Recommended given servo torque here (MG996R has real, non-trivial backlash/side-load sensitivity).

### 5. Forearm
- Same method as Upper Arm, length `forearm_L` = 140mm.

### 6. Wrist Housing
- Encloses wrist servo(s). This is the most geometrically complex part — likely 2-3 sketches on different planes plus a shell operation.
- *Alternative*: single-DOF wrist pitch only vs. **wrist pitch + roll (2 DOF)** to hit your "6-DOF" total honestly (Base rotate, shoulder, elbow, wrist pitch, wrist roll, gripper = 6). Confirm your DOF budget across components now, before committing hole locations — this is the part most likely to force a redesign later if planned wrong.

### 7. Wrist Link
- `wrist_link_L` = 90mm, connects wrist housing to gripper.

### 8. Gripper
- *Alternative*: **parallel-jaw gripper** (simple rack-and-pinion or linkage, easy to sketch, reliable) vs. **underactuated/tendon gripper** (more human-like, much harder to design and tune). For an educational build, parallel-jaw is the correct first choice.

### 9. Servo Horn Adapters
- Small parts mating servo spline to your arm bores. Get exact horn spline dimensions from the MG996R/DS3218 datasheet (they differ — 25T spline vs 25T but different tooth angle in some clones) — physically measure your actual servo horn rather than trusting a generic online dimension, clone servos vary.

### 10. Final Assembly
- Bring all components in, apply joints (Part 4), run motion study.

---

## PART 4 — Assembly & Joints

- Use **Revolute joints** for every rotating pair (as your brief specifies) — Assemble → Joint → select the two components → choose Revolute → pick the axis (usually the pivot bore's cylindrical face gives you the axis automatically).
- Set **joint limits** (Angle limits in the joint dialog) to match real servo travel — typically 0–180° for standard hobby servos. This is what lets your "Check motion" step catch collisions before printing.
- *Alternative*: **As-built joints** (position first, then convert to joint) vs. **joints-first** (define joint, then position) — for a from-scratch design, joints-first is more parametric and predictable; as-built is faster when importing pre-existing STLs you didn't model.

---

## PART 5 — Manufacturing Prep

- Fillet internal corners ≥1mm to reduce print stress concentrations.
- Orient tall/thin parts (arms) flat on the print bed to avoid weak layer-line snapping under bending load — this matters more than infill % for these parts.
- Add **0.2–0.3mm clearance** on all bearing/bushing press-fits and pivot bores before exporting STL (printed holes typically print undersized) — verify with a test coupon print, not just assumption.
- Export: File → Export → STEP (for engineering drawings/archival) and STL (per-component, for printing) separately.

---

## PART 6 — Deliverables Checklist

- [ ] Fusion 360 project with all 10 components
- [ ] Individual component files (or components within the master, exported individually)
- [ ] Full assembly with working revolute joints
- [ ] Engineering drawings (Create → Drawing, one per critical part, with dimensions)
- [ ] Exploded view (Assembly → Create Exploded View, animate and capture)
- [ ] Rendered images (Render workspace, or Shaded with Edges + decent lighting for a quick version)
- [ ] STL exports per component

---

## Suggested Order to Actually Do This

1. Set up parameters (Part 2).
2. Build the Base + Shoulder Plate first — pair with a joint immediately and test rotation before building the rest of the arm on top of an unverified joint.
3. Upper Arm → Elbow → Forearm, testing the joint chain after each addition.
4. Wrist Housing → Wrist Link (decide DOF split now).
5. Gripper last — it's the most iterated-on part typically, so leave room in the schedule.
6. Servo adapters as needed throughout, not saved to the end.
7. Drawings, exploded view, renders, STL export.

---

**Next step suggestion:** tell me which component you want to start sketching first (I'd recommend the Base), and I'll walk through that exact sketch step-by-step, including which specific dimensions/constraints to add in what order.
