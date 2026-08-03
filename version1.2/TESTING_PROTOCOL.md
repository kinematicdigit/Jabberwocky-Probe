# 🔬 Jabberwocky Probe (JWPROBE) — Validation & Testing Protocol

This methodical testing protocol guides you through safely verifying electrical continuity, mechanical axis limit strikes, physical actuation, and software safety interlocks before launching your first automated print.

---

## ⚠️ Pre-Flight Safety Rules

* **Hover Over Power**: Keep your hand physically hovering over your printer's **Main Power Switch** during every initial movement test.
* **Clear Bed**: Ensure your build plate surface is completely clear of debris, clips, or tools.
* **Center Start**: Manually position your toolhead roughly in the absolute center of your X and Y axes before powering on the machine for Stage 1.

---

## 🛑 STAGE 1: Electrical Loop & Continuity Test
**Objective**: Verify the hardware circuit configuration and Nitehawk toolhead board pin mapping are reading electrical states accurately before executing automated movements.

1. Keep the Jabberwocky Probe carriage **manually detached** and sitting inside its physical storage dock slot.
2. Open your web interface (Mainsail/Fluidd) and execute the following command in the console:
   ```gcode
   QUERY_PROBE
   ```
3. **Verify Return State**: The console must report `probe: open` (because the toolhead series loop is broken/open).
4. Manually pick up the slider carriage mechanism and snap it onto the toolhead carriage magnets.
5. Execute the query command again:
   ```gcode
   QUERY_PROBE
   ```
6. **Verify Return State**: The console must now report `probe: TRIGGERED` (indicating a closed, electrically continuous path).

> 💡 **Troubleshooting State Inversions**: If your states are reversed or do not toggle, open `JWPROBE/jwprobe_hardware.cfg`. Check your `pin: ^nhk:gpio10` definition. If your physical switch wiring uses a different normal state layout, you may need to add or remove an exclamation mark (`!`) from the pin declaration.

---

## 🧭 STAGE 2: Safe Axis Boundary Homing Override Test
**Objective**: Verify sensorless homing limits and uniform torque reduction steps execute smoothly without slamming structural frame elements.

1. Ensure your toolhead is clear, then send a system home instruction through the terminal deck:
   ```gcode
   G28
   ```
2. Closely monitor the sequence for the following expected behaviors:
   * **Z-Lift**: The machine executes an immediate, soft vertical clearance separation lift.
   * **X-Homing**: The toolhead travels left toward the X-Min limit frame boundary under reduced stall current (`0.550A`). It should tap the mechanical stop gently, resolve its zero tracking point, and back off softly.
   * **Y-Homing**: The toolhead travels back toward the Y-Max limit boundary slowly (`F1200` / 20mm/s), touches the stall block smoothly, and backs out.
   * **Z-Homing**: The toolhead sets native kinematic reference limits, drives directly over your defined Auto-Z switch pin coordinates (`variable_x_homing_pos`, `variable_y_homing_pos`), and executes a bare nozzle touch strike.
   * **Circuit Verification**: Immediately after homing Z, the toolhead will automatically sweep to the left frame boundary, pick up the probe carriage, run a fast electrical continuity query loop, return the probe safely back to the dock, and park perfectly over the center coordinates of your 180x180 bed canvas.

---

## 🔧 STAGE 3: Low-Level Physical Actuation Verification
**Objective**: Test individual limit strikes independently to validate your absolute physical position offsets.

1. Ensure the machine is successfully homed (`G28`).
2. Run the toolhead deployment macro manually from your dashboard terminal:
   ```gcode
   DEPLOY_PROBE
   ```
   * **Pass Criteria**: Toolhead travels back to the rear lane alignment coordinate, drives cleanly past your normal left travel envelope to your `variable_deploy_x_strike` limit to engage the slider tab, snaps up the carriage, pulls out instantly to `variable_safe_clear_x`, and echoes `Safety Guard: Jabberwocky Probe attached successfully.` to the console.
3. Run the retraction macro manually from the terminal:
   ```gcode
   RETRACT_PROBE
   ```
   * **Pass Criteria**: Toolhead returns back to the rear alignment wall lane, drives right past your maximum printable envelope to your `variable_retract_x_strike` limit to push the slider back into its storage seat, steps away safely into the printable area, and centers itself on the bed.

> 🛠️ **Micro-Tuning Strikes**: If your toolhead hits the frame too loudly or fails to grip/drop the slider tab reliably, adjust `variable_deploy_x_strike` or `variable_retract_x_strike` inside `JWPROBE/jwprobe_variables.cfg` in micro-steps of `0.2mm` to `0.5mm` until the mechanical engagement feels completely fluid.

---

## 🎯 STAGE 4: Automated Calibration & Safety-Interlock Pass
**Objective**: Confirm your automated preparation pipelines and software protection overrides block execution during fault states to prevent nozzle/bed damage.

1. Run your primary master print preparation script:
   ```gcode
   COMPUTE_OFFSET_DIFFERENCE
   ```
2. Observe the full hands-free routine:
   * The toolhead must pick up the slider effortlessly.
   * If `variable_use_auto_z` is toggled `True`, it must process your automatic micro-taps using Protoloft’s engine. If `False`, it applies your manual static fallback baseline.
   * The toolhead executes its leveling passes (`QUAD_GANTRY_LEVEL`) and surface scans (`BED_MESH_CALIBRATE`) using the active probe.
   * The toolhead drops the carriage off at its storage station automatically upon completion.
3. **The Ultimate Failure Interlock Check**: Manually attach the probe carriage to your toolhead magnets while the machine is sitting idle. Run the post-calibration guard macro:
   ```gcode
   ASSERT_PROBE_DOCKED
   ```
   * **Pass Criteria**: The machine must immediately read the anomalous open signal, interrupt its background tracking queues, flash your toolhead lighting indicator arrays red, execute an automated emergency recovery retraction strike to throw the probe back into the dock safely, or raise an explicit terminal error command that completely locks out further machine movements.
