# Jabberwocky Probe
Jabberwocky Probe (formerly Switchy)
<img src="https://github.com/kinematicdigit/Switchy/blob/main/images/mainimage.png" />

A simple probe that relies strictly on a single endstop port to operate. No dock is needed, and it is adjustable for different printer environments.

<img src="https://github.com/kinematicdigit/Switchy/blob/main/images/backside.png" />

> [!INSTALLATION NOTE] 
> I have not made full installation instructions for this yet. It is mostly intuitive, but the two primary things to consider are ensuring the slide moves freely and that the magnets do not have the same polarity from one bank to the other. They should be opposite of each other, or the slide will not work as well. When assembling, you may need to do a small amount of sanding, or I recommend printing the parts on the smoothest plate you have. This probably will not work on CNC-based idlers without some sort of extenders that protrude down to hit the X-min and X-max triggers. 

Jabberwocky Probe relies on idler blocks to actuate the probe (X-min and X-max), sliding the Omron D2HW-C201H switch into place to swing into position (the X-min, which is shared by homing the X-axis of your printer). Once probing is complete, the probe is stowed by bringing it to the X-Maximum location. The probe utilizes two switches wired in series to inform Klipper of the probe's state.

Wiring involves connecting the two switches in series with NC on connections and a connector inline with one of the leads. I recommend using small male and female JST connectors for easier installation. Polarity does not matter.

<img src="https://github.com/kinematicdigit/Switchy/blob/main/images/wiring.png" />

>[!IMPORTANT]
You will need to do Z-offset manually using the `PROBE_CALIBRATE` in your console, or initiate it from KlipperScreen. Once you trigger the macro, it will deploy the probe and check Z. Once the probe is parked, manually push the probe to the side to retract the probe tip and then do your paper offset calibration. Once it is done, either `save_config` or hit the accept on KlipperScreen and restart. You should only need to do this on nozzle changes or plate changes.

**A basic macro set has been included, which is still experimental** (I welcome anyone to elaborate or refine the macro - use at your own risk). Adjust the variables to suit your printer parameters and in your printer config ```[stepper_z]``` settings you need to change your ```endstop_pin``` to ```endtop_pin: probe:z_virtual_endstop```, ```homing_retract_dist: 0``` and remove or comment out ```position_endstop:``` Thanks to @mdrifterx3 for assistance in creating the macros. 

> [!NOTE] 
> I am not certain if this is the case, but the probe might only work with sensorless X-endstop homing. I have not installed a switch for the X-endstop to verify this. Because the probe uses the X-min to trigger the probe, it shares that action. There are two set screws on the probe (M2x8 SHCS) to adjust the probe's trigger points. So in the case of a physical switch, it might be possible to adjust those set screw so it works with a physical switch. This needs to be confirmed in testing.
>
> To-do (stuff to still flush out or goals to reach that might need help from other more knowledgeable people):
> 1. Better homing override (better control over G28 and other standard homing sequences)
> 2. ~~Adaptive Bed mesh is not working currently (can only do a full plate)~~ thanks to igiannakas for helping me figure this out.
> 3. Allow for Auto-Z calibration (currently can't get it to work because it seems to conflict with the same defined Z-stop)


# BOM
| Hardware / Component                             | Quantity |
| ------------------------------------------------ | -------- |
| M3 Nuts                                          | 2        |
| M2x8 SHCS                                        | 2        |
| M2x10 SHCS                                       | 7        |
| M3x20 SHCS                                       | 2        |
| N52 Magnets                                      | 6        |
| Omron D2HW-C201H v2                              | 1        |
| Omron D2F-L                                      | 1        |
| Wiring & Connector                               | various  |


Coffee fund donations are always welcome to help support my ongoing efforts. [Click here](https://www.paypal.com/donate/?business=CXCABTX8BCAUY&no_recurring=0&item_name=Thank+you+for+your+support%21&currency_code=CAD) to donate. Thank-you!
