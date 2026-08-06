# 🦎 Jabberwocky Probe (JWPROBE) v1.2 — Master Configuration Suite

The **Jabberwocky Probe** is an ultra-lightweight mechanical toolhead-deployable probe designed natively for any 3D printer running the **LDO Jabberwocky Toolhead**. Utilizing a 2-wire, Normally Closed (NC) continuous-series loop, this suite delivers surface-independent calibration accuracy down to sub-micron windows.

---

## ⚡ The Jabberwocky Advantage

*   **Zero Docks or Servos**: Actuated entirely by physical X-axis frame wall strikes.
*   **Surface Independent**: Pure mechanical switch contact ensures flawless tracing across PEI, glass, or garolite.
*   **Continuous-Series Loop**: Bridges carriage engagement and nozzle touchdown into a single robust safety input channel (`nhk:gpio10`).

---

## 📦 Third-Party Extensions (SSH Install Required)

This configuration requires two external open-source packages to compile correctly. Execute these installation scripts via your host terminal before initializing:

1. **Protoloft's Auto-Z Calibration Plugin (`klipper_z_calibration`)**
   * *Purpose*: Manages nozzle-versus-body switch tracking math dynamically.
2. **Julian Schill's Klipper LED Effects (`klipper-led_effect`)**
   * *Purpose*: Triggers real-time toolhead state strobes (`JW_LOGO_HOMING`, `JW_LOGO_LEVELING`).

---

## 🔬 Step-by-Step Pre-Flight Verification Checklist

Before attempting automated motion or launching your first print preparation loop, you **MUST** run through these verification checklist operations in order:

*   [ ] **1. Manual State Verification (`QUERY_PROBE`)**
    *   Physically verify your probe carriage is slid all the way to the **RIGHT (STOWED)** position.
    *   In your terminal console, type `QUERY_PROBE` and hit Enter. Expect: `probe: TRIGGERED`.
*   [ ] **2. Manual Deployment Verification**
    *   Physically extend the probe slider mechanism out to the **LEFT (DEPLOYED)** position.
    *   In your terminal console, type `QUERY_PROBE` and hit Enter. Expect: `probe: open`.
*   [ ] **3. Safe Axis Boundary Homing (`G28`)**
    *   Manually push your gantry up high into free air workspace clearance (at least 70mm above the bed).
    *   Hover your hand directly over your physical machine emergency power cutoff toggle switch.
    *   Execute a raw `G28` in your console terminal bar and verify that the toolhead performs its native single-pass downward touchdown pass smoothly onto your Auto-Z switch pin center face (`PC3`) without drifting or head crashing.

---

## 🛠️ First-Time Installation Procedures

1. **Physical Pin Proximity**: Position your physical Auto-Z switch pin so the top face sits exactly 3.0mm above your bed canvas.
2. **Variable Alignment**: Open `JWPROBE/jwprobe_variables.cfg` and enter your true axis striking boundaries (`deploy_x_strike`, `retract_x_strike`).
3. **Core Mapping**: Ensure your master `[stepper_z] endstop_pin` references the true, physical macroswitch channel (`PC3`).

---

## 🏎️ Slicer Integration Blueprint

Copy and paste these parameter macros directly into your slicer's profile sheets to automate your machine operations:

### 🏁 Start G-Code (OrcaSlicer / Bambu Studio / SuperSlicer / PrusaSlicer):
```text
PRINT_START BED=[first_layer_bed_temperature] EXTRUDER=[first_layer_temperature]
```

### 🏁 Start G-Code (Cura):
```text
PRINT_START BED={material_bed_temperature_layer_0} EXTRUDER={material_print_temperature_layer_0}
```

### 🛑 End G-Code (Universal):
```text
PRINT_END
```

---

## 🎨 Dashboard Customization & Button Grouping Guide

Because Klipper web managers store card filters locally via browser cookie profiles, users should customize their interactive option tiles on first-time setup:

### ⛵ For Mainsail Users
1. Click the **Interface Settings Gear Icon** at the top-right corner of your browser screen.
2. Navigate down to the **Macros** tab card options.
3. Click **+ Add Group`, name it `JWProbe Options`, and check the selection tokens for `TOGGLE_AUTO_Z_HARDWARE`, `MODE_AUTO_INTELLIGENCE`, and `MODE_ALWAYS_CALIBRATE`.
4. Click the color drop-down menus directly next to each macro token:
   * Set `MODE_AUTO_INTELLIGENCE` to **Primary / Blue** (🏎️ Performance Mode).
   * Set `MODE_ALWAYS_CALIBRATE` to **Danger / Red** (🚨 Heavy Overhaul Pass).
   * Set `TOGGLE_AUTO_Z_HARDWARE` to **Warning / Orange** (⚙️ Hardware Switch).

### 💧 For Fluidd (Fluiddr) Users
1. Click the **Three Vertical Dots Menu** on the top-right corner of your existing default Macros Card.
2. Select **Manage Groups** to pop the layout customizer window.
3. Create a fresh section label named `JWProbe Options` and check your three `JWPROBE` utility macros.
4. Use Fluidd's color chip palette to lock in your custom theme styling (**Blue** for Auto Intelligence, **Red** for Always Calibrate, and **Orange** for Hardware Bypass).

---

## 📂 Repository Sub-Directory Architecture

To install this package, place the `[include JWProbe.cfg]` directive inside your root `printer.cfg`. All core codebase dependencies resolve inside the modular sub-folder:

```text
📁 printer.cfg (Root configuration file - Includes [include JWProbe.cfg])
└── 📄 JWProbe.cfg (Master directory hub mapping open-source dependencies)
    └── 📁 JWPROBE/
        ├── 📄 jwprobe_variables.cfg      (Master tracking register & Mainsail buttons)
        ├── 📄 jwprobe_hardware.cfg       (Z steppers, [probe], QGL constraints)
        ├── 📄 jwprobe_homing.cfg         (Sensorless X/Y sweeps & Auto-Z pin tracking)
        ├── 📄 jwprobe_leveling_mesh.cfg   (FIRST_TIME_QGL & COMPUTE_OFFSET_DIFFERENCE)
        ├── 📄 jwprobe_slicer_hooks.cfg   (Slicer interface engine - PRINT_START hooks)
        ├── 📄 jwprobe_print_end.cfg      (Controlled shutdown manager - PRINT_END hooks)
        └── 📄 jwprobe_test_macros.cfg    (Live tuning utilities & diagnostic panels)
```



