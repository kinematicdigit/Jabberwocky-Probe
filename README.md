# Switchy Probe for Jabberwocky Toolhead  
*Exclusive for Multimaterial Printing*

---

## Introduction

The **Switchy Probe** is a reliable, stationary switch-based Z-probe designed exclusively for the **Jabberwocky Toolhead**, optimized for multimaterial 3D printing. It enables precise bed leveling, automatic Z-axis calibration, and hands-free probe deployment and retraction without mechanical complexity.

---

## Features

- **Simple Stationary Switch Design**  
  No moving parts except the Z-axis, reducing wear and failure points compared to servo-based probes.

- **Tailored for Jabberwocky Toolhead**  
  Seamless integration with multimaterial setups requiring frequent, accurate toolhead switching.

- **Automatic Deploy and Retract**  
  Probe deploys at `XMAX` and retracts at `XMIN` automatically during calibration routines.

- **Precision Bed Leveling**  
  Supports 4-point probing, quad gantry leveling, and mesh calibration.

- **Full Klipper Compatibility**  
  Overrides native Klipper commands for seamless automation.

- **Easy to Customize**  
  Adjustable deploy/retract positions, Z heights, and bed dimensions.

---

## Configuration Variables

Define these in your `printer.cfg` or an included config file:

```
[variables]
deploy_position: 250           # XMAX - deploy position (right edge)
retract_position: 0            # XMIN - retract position (left edge)
deploy_z_height: 10            # Z height when deploying the probe
probe_trigger_position: 0      # Z height when the switch triggers
XMIN: 0
YMIN: 0
XMAX: 250
YMAX: 250
```
Core Macros
DEPLOY_PROBE
Deploy the probe by moving to XMAX and raising to the deploy height.

```[gcode_macro DEPLOY_PROBE]
gcode:
    {% if probe_state != "triggered" %}
        M117 "Deploying Switchy Probe..."
        G90
        G1 X{deploy_position} F6000
        G1 Z{deploy_z_height} F300
    {% else %}
        M117 "Probe already deployed."
    {% endif %}
```

RETRACT_PROBE
Retract the probe by moving to XMIN and raising Z safely. 
```[gcode_macro RETRACT_PROBE]
gcode:
    {% if probe_state == "triggered" %}
        M117 "Retracting Switchy Probe..."
        G90
        G1 X{retract_position} F6000
        G1 Z10 F300
        G4 P2000
    {% else %}
        M117 "Probe already retracted."
    {% endif %}
```

SWITCHY_PROBE
Perform a 4-point bed leveling cycle using the Switchy Probe.

```[gcode_macro SWITCHY_PROBE]
gcode:
    DEPLOY_PROBE

    G90
    G1 X{XMIN+7} Y{YMIN+7} Z5 F6000
    G1 Z0 F300
    G30

    G1 X{XMAX-7} Y{YMIN+7} Z5 F6000
    G1 Z0 F300
    G30

    G1 X{XMAX-7} Y{YMAX-7} Z5 F6000
    G1 Z0 F300
    G30

    G1 X{XMIN+7} Y{YMAX-7} Z5 F6000
    G1 Z0 F300
    G30

    RETRACT_PROBE

    G1 X{XMAX/2} Y{YMAX/2} Z5 F6000
    G1 Z5 F300
```

Overridden Klipper Commands
BED_MESH_CALIBRATE
Automatically deploys the probe, runs Klipper's mesh calibration, then retracts.
```[gcode_macro BED_MESH_CALIBRATE]
gcode:
    DEPLOY_PROBE
    M117 "Running custom BED_MESH_CALIBRATE..."
    BED_MESH_CALIBRATE
    RETRACT_PROBE
    G1 X{XMAX/2} Y{YMAX/2} Z5 F6000
```

QUAD_GANTRY_LEVEL
Automatically deploys the probe, runs gantry leveling, then retracts.
```[gcode_macro QUAD_GANTRY_LEVEL]
gcode:
    DEPLOY_PROBE
    M117 "Running custom QUAD_GANTRY_LEVEL..."
    QUAD_GANTRY_LEVEL
    RETRACT_PROBE
    G1 X{XMAX/2} Y{YMAX/2} Z5 F6000
```

How It Works
Deploy: Probe moves to XMAX and lifts to deploy_z_height.

Probe: Runs 4-point probing, gantry leveling, or mesh calibration.

Retract: Probe moves to XMIN, lifts, and waits briefly.

Ready: Moves to bed center for next operation.

Conclusion
The Switchy Probe offers a simple, robust automatic probing solution tailored for multimaterial printing with the Jabberwocky Toolhead. Its Klipper macro integration ensures reliable bed leveling, gantry leveling, and mesh calibration — fully automated and easy to customize.
