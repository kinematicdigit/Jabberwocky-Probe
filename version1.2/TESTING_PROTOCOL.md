# 🔬 Jabberwocky Probe (JWPROBE) — Validation & Testing Protocol

This protocol outlines the essential steps to verify your custom toolhead-mounted probe, including physical, electrical, and software checks.

## ⚠ Pre-Flight Safety
*   **Power Control**: Maintain hand-over-switch control during all movement tests.
*   **Clear Area**: Ensure the bed is empty.
*   **Center Start**: Manually position the toolhead at the center.

---

## 🛑 STAGE 1: Electrical Slider States Test
1.  **Stow Probe**: Manually set to retracted; run `QUERY_PROBE`. Expect `probe: open`.
2.  **Deploy Probe**: Manually set to deployed; run `QUERY_PROBE`. Expect `probe: TRIGGERED`.
*   *Troubleshooting*: Adjust the `!` in `jwprobe_hardware.cfg` if states are inverted.

---

## 🧭 STAGE 2: Safe Axis Boundary Homing
1.  Ensure probe is retracted.
2.  Run `G28` and observe the following:
    *   **Z-Lift**: Soft clearance move.
    *   **X/Y Homing**: Gentle contact with high-speed retraction checks for structural safety.
    *   **Z-Homing**: Accurate touchdown on the Auto-Z switch.
    *   **Final Parking**: Head moves to the center, safe from boundaries.

---

## 🚀 STAGE 3: Production & Alignment
### 🦎 Frame Recovery (`FIRST_TIME_QGL`)
Runs a 4-point bed check to calculate adjustments (up to `30mm`) and stows the probe.

### 🏭 Daily Prep (`COMPUTE_OFFSET_DIFFERENCE`)
Executes the full 2-minute cycle: Safe Home -> Deploy -> QGL -> Calibrate Z -> Mesh -> Stow -> Park.

---

## 🚀 AI Session Handshake
Use this summary for future development:
> "We have a 9-file repo with 2-minute auto-prep (`COMPUTE_OFFSET_DIFFERENCE`), sub-3-micron precision, using NC circuit on `nhk:gpio10`/`PC3`. Retain existing macros/parameters."
