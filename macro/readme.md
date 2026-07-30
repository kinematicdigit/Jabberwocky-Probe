WIP do not use



# 📖 Universal Switchy / Klicky & Auto-Z Calibration Template

The universal Klipper configuration template (JWProbe.cfg) is designed for 3D printers utilizing a **Physical Auto-Z Switch** paired with Jabberwocky Probe.

It features dynamic low-torque homing current adjustments, a highly optimized high-speed levelling profile, and an isolated calibration routine that eliminates premature runaway travel moves to `X0`.

---

## 🚀 Features

*   **Centralized Configuration:** Tune all physical coordinates, dock positions, and boundary margins in a single variable block at the top of the file without messing with complex macro logic.
*   **Prematurity-Free Calibration Workflow:** Automatically keeps the probe attached across nozzle, switch, and bed probing steps before executing a single safe return trip to the dock.
*   **High-Speed Levelling Profile:** Lowers transit Z-hop paths to `2mm` and `3mm` to reduce printing preparation time by up to 50%.
*   **Universal Compatibility:** Includes active `[quad_gantry_level]` setups alongside drop-in, fully commented `[z_tilt]` layouts for dual or triple Z-axis machines.
*   **Synchronized Z-Axis Floor:** Keeps global hardware axis safety margins and calibration search depths linked to eliminate `Move out of range` errors.

---

## 🛠️ Prerequisites & Hardware Setup

Before installing this template, verify that your machine meets the following structural requirements:
1.  **Klipper Z-Calibration Plugin:** The `protoloft/klipper_z_calibration` extension must be installed on your host system via KIAUH or direct terminal input.
2.  **Physical Frame Endstop Switch:** A dedicated microswitch button must be permanently mounted to your frame (typically near the rear-right of the bed limits). Your physical nozzle tip must be able to push directly down on this pin.
3.  **Dockable Probe:** Your toolhead probe mechanism must function correctly electrically when verified manually using the `QUERY_PROBE` console command.

---

## ⚙️ Step 1: Mapping Real-World Coordinates

Before deploying the configuration file, use your web interface (Mainsail/Fluidd) to manually jog your toolhead to record your exact machine geometry coordinates:

1.  **Nozzle On Switch (`variable_x_homing_pos` / `variable_y_homing_pos`):** Jog the toolhead until the **tip of the nozzle** is perfectly centered on top of your frame-mounted Z-switch pin button. Record the `X` and `Y` values.
2.  **Probe On Switch (`switch_xy_position` under `[z_calibration]`):** Manually attach your probe. Jog the toolhead until the **center of the probe body button** is perfectly aligned over that exact same frame Z-switch pin. Record the `X` and `Y` values.
3.  **Probe Pickup Dock (`variable_deploy_pos`):** Jog the toolhead to the exact entry `X` coordinate where it smoothly slides into the dock cradle to attach the probe.
4.  **Probe Separation Station (`variable_retract_pos`):** Jog the toolhead to the exact `X` coordinate where it slides or strips to break the magnet bond and detach the probe (typically `X0`).
5.  **Bed Center Midpoint (`variable_safe_x` / `variable_safe_y`):** Locate the geometric absolute center of your print surface (e.g., `90, 90` for a standard 180x180mm print bed).

---

## 📝 Step 2: Deployment & Variable Setup

Create a new file named `jw_probe.cfg` inside your Klipper configuration directory, paste the template contents into it, and add `[include jw_probe.cfg]` to your primary `printer.cfg` file.

Navigate to **`SECTION 1. PRINTER-SPECIFIC VARIABLE STORES`** at the top of the file and insert your recorded metrics:

```ini
[gcode_macro Homing_Variables]
variable_x_homing_pos:          124  ; <--- Insert Nozzle-On-Pin X coordinate
variable_y_homing_pos:          188  ; <--- Insert Nozzle-On-Pin Y coordinate
variable_z_min_floor:          -3.0  ; Allowed depth margin below 0.0mm limit

[gcode_macro switchy_variables]
variable_deploy_pos:            176  ; <--- Insert Pickup Dock X coordinate
variable_retract_pos:             0  ; <--- Insert Drop-off Station X coordinate
variable_safe_x:                 90  ; <--- Insert Bed Center X coordinate
variable_safe_y:                 90  ; <--- Insert Bed Center Y coordinate
```

---

## 🛡️ Step 3: Configuring Boundary Safety Margins

To prevent your probe from running into frame components or hitting bed leveling clips, adjust your mesh and alignment boundaries within the geometric variable constants section:

*   **For a 10mm Boundary Margin:** Subtract 10 from your max size, add 10 to your min size.
*   **For a 20mm Boundary Margin:** Subtract 20 from your max size, add 20 to your min size.
*   *Note:* Ensure you add your probe's physical `y_offset` (e.g., `26.0`) to your **Y boundaries** to keep your toolhead from driving too far back.

```ini
# Example configuration for a 180x180 bed with a 20mm margin and a 26mm rear probe offset:
variable_mesh_min_x:             20  ; Edge safety buffer
variable_mesh_min_y:             46  ; 20mm margin + 26mm probe offset
variable_mesh_max_x:            160  ; Edge safety buffer
variable_mesh_max_y:            144  ; 160 max Y travel minus probe offset factor
```

---

## 🧪 Step 4: First-Run Safety Verification Testing

> ⚠️ **WARNING:** Do not leave the printer unattended during this test phase. Keep your hand placed directly over your web interface **Emergency Stop** button or your machine's physical power switch.

1.  Send a **`RESTART`** instruction to reload your configuration.
2.  Run **`G28`** to verify core homing. The toolhead must home X and Y, move directly to your Z-switch pin (`X124, Y188`), tap the button with the nozzle, and lift back up `5mm`.
3.  Execute **`DEPLOY_PROBE`**. The toolhead must travel smoothly to your dock, attach the probe cleanly, and return back to the center of the bed (`X90, Y90`) at a safe `Z5` height.
4.  Execute **`RETRACT_PROBE`**. The toolhead must slide to your stripping station (`X0`), break the magnetic connection cleanly, and return to the center of the bed completely empty.

---

## 🎯 Step 5: Running the Auto-Z Calibration

Once the individual travel sequences function as expected, execute the automated verification macro command:

```gcode
CALIBRATE_Z
```

### Automation Sequence Path Breakdown:
1.  The machine fires `DEPLOY_PROBE` to pick up and attach the hardware sensor.
2.  The bare **nozzle tip** references the physical frame-mounted switch pin.
3.  The toolhead re-aligns to tap the switch pin button using the center of the attached **probe switch body**.
4.  The toolhead travels smoothly to **X10** (or your centered mesh position) to sample the bed surface plane.
5.  After the math calculation resolves, `RETRACT_PROBE` is triggered, safely sliding the probe off at your `X0` station.
6.  The dynamic offset value is locked into Klipper memory and printed to the terminal dashboard console.

---

## 🔄 Step 6: Print Start Integration

To fully automate this calibration sequence before every print job, replace the standard homing steps inside your `[gcode_macro PRINT_START]` block with this operational logic chain:

```ini
[gcode_macro PRINT_START]
gcode:
    M104 S150                    ; Preheat nozzle to a safe non-oozing temperature
    M140 S[bed_temperature]      ; Heat the print surface completely
    G28                          ; Safe low-torque home for all axes
    QUAD_GANTRY_LEVEL            ; Balance gantry frame (or use Z_TILT_ADJUST)
    G28 Z                        ; Re-home Z axis following geometric frame leveling
    CALIBRATE_Z                  ; Run dynamic Auto-Z calibration workflow template
    BED_MESH_CALIBRATE           ; Generate or load surface compensation profile map
```
