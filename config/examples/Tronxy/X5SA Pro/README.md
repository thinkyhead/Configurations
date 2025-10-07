# Table of Contents

- [General X5SA Series Info](#general-x5sa-series-info)
- [Configure Marlin for Your Printer](#configure-marlin-for-your-printer)
  - [Motherboard CXY‑V6‑191017](#motherboard-cxy‑v6‑191017)
- [Flash Marlin Using SD Card](#flash-marlin-using-sd-card)
- [Known Issues](#known-issues)
- [Suggested Printing Workflow](#suggested-printing-workflow)
  - [Slicer start G-Code Changes](#slicer-start-g-code-changes)
  - [Normal Workflow](#normal-workflow)

---

## General X5SA Series Info

For general information about the X5SA series printers and its predecessor, see the
[X5SA Readme on GitHub](//github.com/MarlinFirmware/Configurations/tree/import-2.1.x/config/examples/Tronxy/X5SA). Including how to backup your existing settings and firmware.

---

## Configure Marlin for Your Printer

### Motherboard CXY‑V6‑191017

These `Configuration.h` and `Configuration_adv.h` files are designed for the Tronxy X5SA Pro with:
- motherboard: CXY‑V6‑191017
- CPU: STM32F103ZET6
- TMC2225 stepper motor drivers
- Titan extruder
- Bed probe for Z-axis homing and bed levelling.
- 330x330x400 mm build volume with heated bed

Standard Marlin features that have been enabled with these configuration files include:
- Default feedrate: { 150, 150, 20, 40 } mm/s for { X, Y, Z, E }.
- Default accelleration: { 500, 500, 100, 800 } mm/s/s for  { X, Y, Z, E }.
- The bed probe offset from the nozzle is set to for the standard location with the factory hotend/probe. If you have moved your probe you'll need to adjust to match your machine.

---

## Flash Marlin Using SD Card

You can now update Marlin directly from an SD card.

1. Compile Marlin with the settings above. The build output will be `YOUR-MARLIN-DIR/.pio/build/chitu_f103/update.cbd`.
2. Power off the printer.
3. Copy `update.cbd` to an SD card and insert it.
4. Turn the printer on. You'll hear a series of beeps, then Marlin will begin the update.

That's all—no need to open the case or use a programmer.

---

## Known Issues

If you are using Marlin 2.1.3-beta3 or earlier, the required pull request [28059](//github.com/MarlinFirmware/Marlin/pull/28059) had not yet been merged. If you are not using newer code, you must manually override the Z‑stop pin in `pins_CHITU3D_V6.h` because the CXY‑V6‑191017 board in the X5SA Pro uses PG9 instead of PA14 through its 30-pin connector.

In `pins_CHITU3D_V6.h` replace:

```cpp
#define Z_STOP_PIN PA14
```

with:

```cpp
#ifndef Z_STOP_PIN
  #define Z_STOP_PIN PA14
#endif
```

---

## Suggested Printing Workflow

### Slicer start G-Code Changes

Because the bed support structure can physically wobble, these configurations were designed to run the `G29 P1` Auto Unified Bed Leveling (UBL) at the start of every print, instead of saving the data. This is why the UBL mesh is limited to a 3×3 grid to save time. It is inteded for you to adjust your slicer settings to include the `G29 P1` command to make it do this bed probe at the start of every print.

In your slicer's machine‑start G‑code replace the `G28` line, that homes all axis, with :

   ```text
   G28 ; home all axes
   G29 P1 ; Probe the bed
   G29 A F10.0 ; activate UBL and set fade height to 10mm
   ```

(Do not save the mesh; the bed support structure's wobble when removing prints or cleaning the bed makes it unreliable.)


### Normal Workflow

Because the inductive/capacitive/hall effect probes read differently at different temperatures, it is best to preheat the bed and nozzle before tramming the bed or setting the Z offset. To help ensure consitent sensor temperature, auto home all axis while it heats up.

For those coming from the Tronxy firmware interface, the Tronxy "Auto Leveling" feature is replaced with the menu item “Probe and Level → Tramming Wizard”. For the Marlin interface, measure and adjust each point in the wizard one at a time starting at the front-left, rather than probing the whole bed then adjusting and re-probing as Tronxy had you doing.

Use the menu item “Probe and Level → Z Probe Wizard” to set the zero height. This can be saved to EEPROM with the menu “Configuration → Store Settings.” Once you start your print, If the first layer looks like the nozzle is too high or low on the first layer, use the tune menu babystepping to adjust the Z zero height position.

Enjoy a smoother printing experience!
