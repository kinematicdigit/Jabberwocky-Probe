# 🦎 Jabberwocky Probe (JWPROBE) v1.2 — Master Documentation Manual

The **Jabberwocky Probe (JWPROBE)** is an ultra-lightweight mechanical, toolhead-deployable probe designed for the LDO Jabberwocky Toolhead, offering high-precision, surface-independent calibration using a 2-wire, Normally Closed (NC) loop.

## ⚡ The Jabberwocky Advantage

*   **Zero Docks or Servos**: Actuated by physical X-axis frame strikes, reducing toolhead mass.
*   **Surface Independent**: Operates flawlessly on PEI, glass, or garolite.
*   **Continuous-Series Loop Failsafe**: Uses a NC loop (`nhk:gpio10`) for immediate stoppage upon wire break or connection failure.
*   **Thermal Immunity**: Impervious to high chamber/bed temperatures.

---

## ⚖️ Comparative Analysis (Advantages Over Competitive Probes)

| Feature | JWPROBE v1.2 | BLTouch/Antclabs | Klicky/Euclid | Voron Tap | Inductive |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Toolhead Mass** | **Extremely Low** | Medium | Low | **Very High** | Low |
| **Thermal Res.** | **High** | Low | Medium | High | **Extremely Low** |
| **Fail-Safe** | **NC Loop** | None | Variable | High | Variable |
| **Build Efficiency**| **Maximum** | High | Reduced | Maximum | Reduced |
| **Wear Components** | **Minimal** | High | Medium | High | **Zero** |

---

### 🌍 Build Surface Compatibility Matrix

*   **Glass/Borosilicate**: **Flawless** (Physical trigger vs. Inductive failure).
*   **Garolite/G10**: **Flawless** (Direct surface probing vs. Inconsistent sensing).
*   **Textured PEI/Steel**: **Flawless** (True physical map vs. Texture variance noise).

---

## 🛠 Bill of Materials (BOM)

To assemble the Jabberwocky Probe v1.2, you will need the following hardware components:

### 🔩 Fasteners & Hardware
*   **M2x8 SHCS**: 2 pieces (Used as adjustable set screws to fine-tune physical trigger points).
*   **M2x10 SHCS**: 7 pieces (Primary structural assembly fasteners).
*   **M3x20 SHCS**: 2 pieces (Main mounting fasteners).
*   **M3 Nuts**: 2 pieces.
*   **N52 Magnets**: 6 pieces (Ensures crisp deployment snapping and secure stowing retention).

### 🔌 Electronics & Switches
*   **Omron D2HW-C201H v2**: 1 piece (The strike switch mounted to the sliding carriage).
*   **Omron D2F-L**: 1 piece (The touchdown microswitch for direct nozzle tracing).
*   **Wiring/Connectors**: Various lengths of premium wire. Small male and female JST inline connectors are strongly recommended to establish a modular, quick-disconnect breakpoint for painless toolhead servicing.

---

## 🔌 Electrical Blueprint & Wiring Topology

The mechanical architecture of the JWPROBE relies completely on your printer's X-axis idler blocks to physically actuate the carriage. 
*   **Deployment**: The slider hits the X-Minimum boundary wall to swing the Omron D2HW-C201H switch into its active probing stance.
*   **Stowing**: Bringing the toolhead to the X-Maximum boundary wall strikes the opposing frame face, sliding the mechanism back into its stowed position.

### ⚡ The Continuous-Series Loop Logic
To inform Klipper of its operational status without requiring complex multi-channel logic, the Jabberwocky Probe routes both microswitches into a single **continuous-series loop** utilizing **Normally Closed (NC)** configuration paths.

![Jabberwocky Probe Electrical Schematic](https://github.com/kinematicdigit/Jabberwocky-Probe/raw/main/version1.2/images/electrical_schematic.png)

```text
  [ Toolhead MCU / Mainboard Pin: nhk:gpio10 ]
                       │
                       ▼
         ┌───────────────────────────┐
         │    Omron D2HW-C201H       │  (Carriage Engagement Switch)
         │   [Terminal: NC Loop]     │
         └─────────────┬─────────────┘
                       │
                       ▼  <-- Inline JST Quick-Disconnect Breakpoint
                       │
         ┌─────────────┴─────────────┐
         │       Omron D2F-L         │  (Nozzle Touchdown Switch)
         │   [Terminal: NC Loop]     │
         └─────────────┬─────────────┘
                       │
                       ▼
             [ Ground / Return Pin ]
```

### 🔒 Hardware Failsafe Operation
Because the circuit relies on a closed electrical path while idle or searching, polarity does not matter. This layout acts as an un-bypassable hardware fail-safe: if a wire breaks mid-print, a JST plug unseats, or a terminal connection fails, the loop instantly opens. Klipper will register an immediate probe trigger event, preventing the toolhead from executing a catastrophic bed crash.


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



