# 🦎 Jabberwocky Probe (JWPROBE) — Master Configuration Suite

The **Jabberwocky Probe** is an ultra-lightweight, high-fidelity mechanical toolhead slider probe designed specifically for the **LDO Jabberwocky Toolhead**. While optimized out-of-the-box for this specific toolhead architecture, it can be adapted to other carriage setups with modifications. A master CAD design file is included in this repository to allow the community to develop and remix their own mounting configurations.

---

## ⚡ The Jabberwocky Advantage

Compared to traditional 3D printer sensing technology, the Jabberwocky Probe offers clear mechanical and electrical advantages:

* **No Docks or Pickup Procedures**: Unlike Klicky or Clicky-style probes, the Jabberwocky mechanism is completely toolhead-mounted. It utilizes physical left and right X-axis frame strikes to mechanically slide between stowed and deployed states—eliminating dropped probes, alignment drift, and wasted bed space due to physical docking stations.
* **Universal Surface Compatibility**: Inductive or eddy-current sensors (such as Beacon or Cartographer) rely on metal proximity and fail or miscalculate on glass, thick textured surfaces, or composite build plates. Because the Jabberwocky Probe relies on a mechanical microswitch touch, it works flawlessly on any bed surface.
* **Ultra-Simple Single Circuit Wiring**: While devices like the BL-Touch require complex, multi-wire logic arrays (servo PWM signals, power, ground, and endstop lines), the Jabberwocky Probe acts as a smart series loop. It operates using a **single simple 2-wire circuit layout**, drastically cutting down toolhead cable chain bulk and freeing up mainboard IO pins.

---

## 📦 Third-Party Software Dependencies

This suite relies on two open-source extensions to function. They must be installed on your host system via an SSH terminal session before booting Klipper with these configuration files:

### 1. Protoloft's Auto-Z Calibration Plugin (klipper_z_calibration)
* **Purpose**: Manages nozzle-contact and switch-body micro-taps for dynamic runtime Z-offset calculations.
* **Installation Terminal Commands**:


```bash
cd ~
git clone https://github.com
./klipper_z_calibration/install.sh
```

### 2. Julian Schill's Klipper LED Effects Extension (klipper-led_effect)
* **Purpose**: Drives non-blocking, asynchronous visual tracking and hardware state animations across your toolhead indicator arrays.
* **Installation Terminal Commands**:
```bash
cd ~
git clone https://github.com
./klipper-led_effect/install.sh
```

---

## 📂 Sub-Directory Routing Layout

To maintain a clean configuration directory, all supporting logic is separated into an isolated `JWPROBE/` module directory. The master controller wrapper handles inclusions automatically:

* **printer.cfg** (Your main configuration file)
  * **[include JWProbe.cfg]** (Master Wrapper Control Hub)
    * **[include JWPROBE/jwprobe_variables.cfg]** (Configuration parameters & stroke limits)
    * **[include JWPROBE/jwprobe_hardware.cfg]** (Pin definitions, mesh boundaries, QGL constants)
    * **[include JWPROBE/jwprobe_core_movements.cfg]** (Sensorless homing overrides & leveling loops)
    * **[include JWPROBE/jwprobe_actuation_safety.cfg]** (Low-level X-strike mechanics & verification gates)
    * **[include JWPROBE/jwprobe_led_effects.cfg]** (Compact animation arrays and power decay ticks)
    * **[include JWPROBE/jwprobe_test_macros.cfg]** (G-code calibration patterns & live tuning panels)
    * **[include JWPROBE/jwprobe_install_wizard.cfg]** (Automatic calibration configuration script)

---

## 🔬 Mandatory Verification Protocol

Before powering on your stepper motors or executing any movement macros, you **MUST** open and follow the step-by-step diagnostic verification guide:

👉 [**CLICK HERE TO OPEN TESTING_PROTOCOL.md**](./TESTING_PROTOCOL.md)

### 🚨 What you will verify in the protocol:
* **Stage 1**: Live sensor state queries (`probe: open` vs `probe: TRIGGERED`) to verify electrical series loop wiring continuity.
* **Stage 2**: Safe low-torque sensorless homing overrides to ensure the toolhead backs out safely from frame edges.
* **Stage 3**: Micro-tuning your left and right physical X-limit strikes to perfectly actuate the mechanical slider carriage.
* **Stage 4**: Validating software protection interlocks to confirm the machine safely aborts if a fault state is detected.

---

## 🛠️ First-Time Installation Procedures

Before launching your very first automated print, you **MUST** follow these exact mechanical configuration steps to prevent hardware collisions:

### Step 1: Physical Height Alignment
The top face of your uncompressed Auto-Z switch pin body **MUST** be physically adjusted to sit exactly **3.0mm ABOVE your bed plane**.

### Step 2: Validate Machine Variables
Open `JWPROBE/jwprobe_variables.cfg` and verify that `variable_x_homing_pos` and `variable_y_homing_pos` align with your switch pin center. Check that `variable_bed_max_x` and `variable_bed_max_y` precisely match your mechanical envelope limits.

### Step 3: Run the Installation Wizard
Type `JBW_INITIAL_INSTALL_WIZARD` into your interface console deck and press Enter. Follow the printed instructions to jog the toolhead down until the nozzle just touches the pin face. Upon clicking **ACCEPT**, the system will run the sign-corrected math automatically and print your precise offset value. Enter this result into `variable_switch_offset` inside your variables file.

### Step 4: Clear Out Saved Overrides
Ensure that `[stepper_z] position_endstop` inside `JWPROBE/jwprobe_hardware.cfg` is set to your baseline calibration height (e.g., `5.943`). Check the very bottom auto-generated block of your master `printer.cfg` file and completely remove any stale auto-saved `position_endstop` or `z_offset` lines.

### Step 5: Run the Verification Protocol
Review and execute the testing procedures outlined inside `TESTING_PROTOCOL.md` to confirm your sensorless limits and safety gates respond as intended.

---

## 🎛️ Central Configuration Options

Users can toggle between dynamic Auto-Z tracking or traditional static offsets cleanly by modifying a single flag in `JWPROBE/jwprobe_variables.cfg`:

```ini
[gcode_macro JWProbe_Variables]
# Set to True to run the dynamic Auto-Z plugin nozzle taps
# Set to False to bypass the plugin and apply a standard static offset ceiling fallback
variable_use_auto_z: False  
```
### 📊 Verbose Logging & Debug Traffic Controls

To prevent terminal clutter during stable production runs or to activate deep tracing parameters during initial troubleshooting, you can adjust the diagnostic reporting depth via `variable_verbose_level` inside `JWPROBE/jwprobe_variables.cfg`:

```ini
[gcode_macro JWProbe_Variables]
# 0 = Silent Mode: Turns off all regular background informational console echoes.
# 1 = Standard Production Mode (Default): Logs critical pipeline steps and calculations.
# 2 = Deep Debug Mode: Outputs granular trace telemetry blocks for all macro functions.
variable_verbose_level: 1
```

> 💡 **Live Runtime Adjustments**: If you want to change the logging depth on the fly during an active printing session without restarting your entire firmware configuration, you can send this raw override string directly to your interface console deck:
> ```gcode
> SET_GCODE_VARIABLE MACRO=JWProbe_Variables VARIABLE=verbose_level VALUE=2
> ```

### ⚠️ Hardware Pin Choice Reminder
If you toggle `variable_use_auto_z: False`, Klipper requires you to open `JWPROBE/jwprobe_hardware.cfg` and comment out the physical pin line to tell the system stepper drivers to route through virtual endstop flags instead:

```ini
[stepper_z]
# OPTION A: If using Protoloft Auto-Z Calibration, uncomment these lines:
endstop_pin: PC3                     
position_endstop: 5.943             

# OPTION B: If using standard manual Z-offset calibration, uncomment these lines:
# endstop_pin: probe:z_virtual_endstop
# homing_retract_dist: 0
```

---

## 🏎️ Slicer Integration Blueprint

To automate your Jabberwocky Probe routine during every print pass, update your slicer's **Start G-code** text field (OrcaSlicer, PrusaSlicer, SuperSlicer, etc.) to call the custom preparation pipeline wrapper after hitting thermal targets:

```gcode
; --- SLICER START G-CODE REFERENCE BLOCK ---
M140 S[first_layer_bed_temperature] ; Start heating bed
M104 S150                            ; Set standby toolhead temp to mitigate ooze
M190 S[first_layer_bed_temperature] ; Wait for bed temp to stabilize

COMPUTE_OFFSET_DIFFERENCE            ; Executes homing, dynamic probe calibration, QGL, and adaptive mesh

M109 S[first_layer_temperature]     ; Ramp hotend core up to true printing temperature
M118 Jabberwocky Pipeline Clear. Launching print initialization tracking.
```

---

## 🩺 Diagnostic Support Panels

The repository includes explicit, built-in macro control panels inside `jwprobe_test_macros.cfg` to adjust your first layer on the fly:

* `CALIBRATE_FIRST_LAYER_PLA` / `CALIBRATE_FIRST_LAYER_ABS`: Generates a specialized, native 40x40 tracking patch to let you observe layer adhesion live without needing to slice diagnostic STLs.
* `SQUISH_MORE_CLOSE` / `SQUISH_LESS_AWAY`: Volatile runtime adjustment buttons to bump your operational offset by `0.02mm` steps mid-print.
* `EMERGENCY_CLEAR_LED_ALARM`: Instantly terminates diagnostic alarm patterns, clears system error flashes, and restores your lighting blocks back to generic ready flags.

