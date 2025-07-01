# Switchy
Switchy Probe
<img src="https://github.com/kinematicdigit/Switchy/blob/main/images/mainimage.png" />

A simple probe that relies strictly on a single endstop port to operate. No dock is needed, and it is adjustable for different printer environments.

<img src="https://github.com/kinematicdigit/Switchy/blob/main/images/backside.png" />

Switchy relies on idler blocks to actuate the probe, sliding the Omron D2HW-C201H switch to swing into place. Once probing is complete, the toolhead is stowed, but the toolhead is brought to the X-Minimum location, and the probe is returned to its stored state. The probe utilized two switches wired in series to act as a mechanism to inform Klipper of the probe's state.

A basic macro set has been included (I welcome anyone to elaborate or refine the macro). Adjust the variables to suit your printer parameters. Thanks to @mdrifterx3 for assistance in creating the macros. 

# BOM
| Hardware / Component                             | Quantity |
| ------------------------------------------------ | -------- |
| M3 Nuts                                          | 2        |
| M2x8 SHCS                                        | 2        |
| M2x10 SHCS                                       | 7        |
| M3x16 SHCS                                       | 2        |
| N52 Magnets                                      | 4        |
| Omron D2HW-C201H v2                              | 1        |
| Omron D2F-L                                      | 1        |
| Wiring & Connector                               | various  |
