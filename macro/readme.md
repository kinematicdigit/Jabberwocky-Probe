**WIP do not use**

# 📖 Universal Switchy / Klicky & Auto-Z Calibration Template

The universal Klipper configuration template (JWProbe.cfg) is designed for 3D printers utilizing a **Physical Auto-Z Switch** paired with Jabberwocky Probe.

It features dynamic low-torque homing current adjustments, a highly optimized high-speed levelling profile, and an isolated calibration routine that eliminates premature runaway travel moves to `X0`.

---

## 🚀 Features

*   **Centralized Configuration:** Tune all physical coordinates and boundary safety margins inside a single variables block at the top of your config.
*   **High-Speed Leveling Alignment:** Lowers horizontal transit Z-hop paths to minimize vertical travel time during multi-point sweeps.
*   **Synchronized Z-Axis Floor:** Connects global hardware axis boundaries directly with calibration searching depths to eliminate `Move out of range` errors.
*   **Universal Compatibility:** Works seamlessly with `[quad_gantry_level]` setups or drop-in `[z_tilt]` layouts for dual or triple Z-axis machines.

---

## 🛠️ Step 1: Collect Your Hardware Coordinates

Use the manual control arrows in your web interface (Mainsail/Fluidd) to jog your printer. Record the exact coordinates for the following positions:

1.  **Nozzle Center on Z-Switch:** Jog the toolhead until the bare **nozzle tip** is perfectly centered on top of your frame-mounted Z-switch pin. Note the `X` and `Y` values.
2.  **Probe Center on Z-Switch:** Jog the toolhead until the **center of the probe body button** is aligned over that exact same Z-switch pin. Note these `X` and `Y` values.
3.  **Print Bed Center and Boundaries:** Note your total usable print area dimensions (e.g., `180` x `180` mm) and its absolute midpoint center (e.g., `90, 90`).

---

## 📝 Step 2: Update the Variable Blocks

Open your configuration file. Update **`[gcode_macro Homing_Variables]`** and **`[gcode_macro switchy_variables]`** using your recorded metrics.

### 1. Configure Homing & Switch Positions
Input your Nozzle-on-Pin coordinates into `variable_x_homing_pos` and `variable_y_homing_pos`:
```ini
[gcode_macro Homing_Variables]
variable_lift_z:                5.0  ; Z-hop height before homing
variable_x_homing_pos:          125  ; <--- YOUR NOZZLE-ON-PIN X HERE
variable_y_homing_pos:          188  ; <--- YOUR NOZZLE-ON-PIN Y HERE
variable_z_end_pos:             5.0  ; Post-homing Z retract height
variable_z_min_floor:          -3.0  ; Maximum allowable probing depth below 0.0mm
```

### 2. Configure Safe Transit Zones & Bed Center
Input the absolute center coordinate of your bed (e.g., `90, 90` for a 180x180 bed):
```ini
[gcode_macro switchy_variables]
variable_safe_x:                 90  ; <--- YOUR BED CENTER X HERE
variable_safe_y:                 90  ; <--- YOUR BED CENTER Y HERE
```

### 3. Calculate Safety Boundary Margins
To keep the toolhead from striking bed clips or falling off edges, calculate a safety margin envelope (e.g., 20mm inward from the physical edges). 
*   *Crucial rule:* You must factor your probe's physical `y_offset` (e.g., `26.0`) into the **Y variables** so the gantry does not travel too far back during automatic probing routines.

```ini
# Example for a 180x180 bed with a 20mm margin and a 26mm rear probe offset:
variable_mesh_min_x:             20  ; 20mm left edge margin
variable_mesh_min_y:             46  ; 20mm front margin + 26mm probe offset
variable_mesh_max_x:            160  ; 180mm max minus 20mm right margin
variable_mesh_max_y:            144  ; 180mm max Y minus 20mm back margin minus 16mm clearance

# Mirror these exact boundary limits down into your QGL coordinate variables:
variable_qgl_point1_x:           20  ; Front-Left X
variable_qgl_point1_y:           20  ; Front-Left Y
variable_qgl_point2_x:           20  ; Rear-Left X
variable_qgl_point2_y:          134  ; Rear-Left Y (Matches max mesh Y constraint)
variable_qgl_point3_x:          160  ; Rear-Right X
variable_qgl_point3_y:          134  ; Rear-Right Y (Matches max mesh Y constraint)
variable_qgl_point4_x:          160  ; Front-Right X
variable_qgl_point4_y:           20  ; Front-Right Y
```

---

## 🔬 Step 3: Align Hardware Sections

Verify that your physical stepper, probe, mesh, and QGL blocks reference your variables instead of raw hardcoded values.

*   **`[stepper_z] position_min`:** Must match your `variable_z_min_floor` (set to `-3.0`) to avoid software limits blocking a successful switch click.
*   **`[probe] y_offset`:** Must match the physical distance from your nozzle tip to the center of your probe body (set to `26.0`).
*   **`[bed_mesh]` & `[quad_gantry_level]`:** Ensure the layout coordinates for `mesh_min`, `mesh_max`, and `points` match the safety boundary targets established in your variables block.

---

## 🧪 Step 4: First-Run Safety Verification

> ⚠️ **WARNING:** Keep your hand directly over the software **Emergency Stop** button or the printer's physical power switch during this initial verification pass.

1.  Issue a **`RESTART`** command in your console terminal dashboard.
2.  Run **`G28`** to test homing overrides. The toolhead must home X and Y, transition directly to your Z-switch pin (`X125, Y188`), tap the button with the nozzle, and retract up 5mm.
3.  Test your leveling macros (`QUAD_GANTRY_LEVEL` or `Z_TILT_ADJUST`) to ensure movements stay well within the inner boundary limits without wandering off the bed surface.
4.  Run **`CALIBRATE_Z`** to check accuracy before launching an actual print job.

