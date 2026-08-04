# 🔬 Jabberwocky Probe (JWPROBE) — Validation & Testing Protocol

This protocol guides you through safely verifying the electrical continuity, physical slider actuation, and software safety interlocks of your toolhead-mounted Jabberwocky Probe before running your first print.

---

## ⚠️ Pre-Flight Safety Rules

* **Hover Over Power**: Keep your hand physically hovering over your printer's **Main Power Switch** during every initial movement test.
* **Clear Bed**: Ensure your build plate surface is completely clear of debris, clips, or tools.
* **Center Start**: Manually position your toolhead roughly in the absolute center of your X and Y axes before powering on the machine for Stage 1.

---

## 🛑 STAGE 1: Electrical Slider States Test
**Objective**: Verify the hardware circuit configuration and toolhead board pin mapping are reading electrical states accurately as you manually slide the mechanism.

1. Reach into the printer and manually slide the Jabberwocky carriage up into its **fully retracted (stowed)** position.
2. Open your web interface (Mainsail/Fluidd) and execute the following command in the console:
   ```gcode
   QUERY_PROBE
   ```
3. **Verify Return State**: The console must report `probe: open` (indicating the loop is open/stowed).
4. Now, manually click the slider mechanism down/out into its **fully deployed** position in mid-air.
5. Execute the query command again:
   ```gcode
   QUERY_PROBE
   ```
6. **Verify Return State**: The console must now report `probe: TRIGGERED` (indicating a closed, electrically continuous path ready to probe).

> 💡 **Troubleshooting State Inversions**: If your states are reversed or do not toggle, open `JWPROBE/jwprobe_hardware.cfg`. Check your `pin:` definition. If your physical switch wiring uses a different normal state layout, you may need to add or remove an exclamation mark (`!`) from the pin declaration to align it with the software.
---

## 🧭 STAGE 2: Safe Axis Boundary Homing Override Test
**Objective**: Verify sensorless homing limits and uniform torque reduction steps execute smoothly without slamming structural frame elements.

1. Ensure the probe slider is manually retracted, then send a system home instruction through the terminal deck:
   ```gcode
   G28
   ```
2. Closely monitor the sequence for the following expected behaviors:
   * **Z-Lift**: The machine executes an immediate, soft vertical clearance separation lift.
   * **X-Homing**: The toolhead travels right toward the X-Max limit frame boundary under reduced stall current (`0.550A`). It should tap the mechanical stop gently, resolve its zero tracking point (`X=176.0`), and back off softly.
   * **Y-Homing**: The toolhead travels back toward the Y-Max limit boundary slowly (`F1200` / 20mm/s), touches the rear stall block smoothly, and backs out to establish `Y=190.0`.
   * **Z-Homing**: The toolhead sets native kinematic reference limits, drives directly over your defined Auto-Z switch pin coordinates (`variable_x_homing_pos`, `variable_y_homing_pos`), and executes a bare nozzle touch strike downward.
   * **Circuit Verification**: Immediately after homing Z, the toolhead will automatically deploy the slider to test circuit health. Because the loop will read `TRIGGERED` in mid-air, the context-split helper macro will verify electrical continuity, immediately call `RETRACT_PROBE` to click the switch safely closed, and park perfectly over the center coordinates of your bed canvas.
