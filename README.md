# T300 Tweaks

Klipper configuration tweaks and custom macros for the **Comgrow T300** 3D printer.

Includes tuned configs for stock T300 and T300 with the **3MS multi-material system** running **Happy Hare**.

---

## What's In Here

| Directory | Contents |
|---|---|
| `configs/base/` | Full config for T300 without MMU |
| `configs/with-3ms/` | Full config for T300 + 3MS + Happy Hare |
| `macros/` | Individual macros to cherry-pick |
| `images/` | Photos and diagrams |

---

## Usage Options

### Option 1: Cherry-pick individual macros

Copy specific `.cfg` files from `macros/` into your config directory and add an `[include]` line in your `printer.cfg`.

```ini
[include macros/m108_fan_control.cfg]
[include macros/improved_z_leveling.cfg]
[include macros/blob_purge.cfg]       # requires Happy Hare MMU
[include macros/endless_spool.cfg]    # requires Happy Hare MMU
```

### Option 2: Use the full config files

Copy the contents of `configs/base/` or `configs/with-3ms/` to your printer's config directory (`~/printer_data/config/`).

**Before you start:**
1. Copy `MCU_ID.cfg.example` to `MCU_ID.cfg`
2. Edit `MCU_ID.cfg` and replace the `TODO:` placeholder with your actual MCU serial:
   ```
   ls /dev/serial/by-id/
   ```
3. Run your own calibration (see below)

---

## Required Calibration

> **Do not print without calibrating first.** The SAVE_CONFIG values in these files are from our specific machine and will not match yours.

| Command | What it does |
|---|---|
| `PROBE_CALIBRATE` | Sets your z_offset (nozzle-to-bed distance) |
| `PID_CALIBRATE HEATER=extruder TARGET=200` | Extruder PID tuning |
| `PID_CALIBRATE HEATER=heater_bed TARGET=60` | Bed PID tuning |
| `BED_MESH_CALIBRATE` | Generates your bed mesh |

---

## What's Different From Stock

### Input Shaper
Stock firmware ships with incorrect input shaper values that cause over-smoothing. These configs include values tuned with an MPU-6050 accelerometer:

| Axis | Stock (wrong) | Tuned |
|---|---|---|
| X | `3hump_ei @ 58 Hz` | `mzv @ 196.6 Hz` |
| Y | `2hump_ei @ 40 Hz` | `mzv @ 16.0 Hz` |

Run `INPUT_SHAPER_TEST_X` / `INPUT_SHAPER_TEST_Y` from `input_shaper_calibration.cfg` to calibrate for your machine.

### Extruder Rotation Distance
Stock value is `3.55`. Corrected to `3.586` after calibration.

### M108 Fan Percentage Control
Adds `M108 P<0.0-1.0>` to scale fan speed without changing slicer output.
Example: `M108 P0.7` makes M106 S255 behave like M106 S179.

### Improved Z Leveling
Three alternative macros for Z-axis leveling: `GENTLE_Z_TILT`, `MANUAL_Z_SYNC`, and `QUICK_Z_CHECK`.

### 3MS + Happy Hare (with-3ms only)

- **Blob purge** at back-left corner (X=10, Y=290) — builds a tall blob and knocks it off the bed edge
- **Endless spool** — automatic gate switching on runout with stub ejection, pre-print gate remapping
- **Tip forming** tuned for PLA/PETG/ASA at 200°C with 0 cooling moves

---

## Macros Reference

### Base (no MMU)

| Macro | Description |
|---|---|
| `START_PRINT` | Full print start with adaptive bed mesh |
| `END_PRINT` | Print end with park and heaters off |
| `PAUSE` / `RESUME` / `CANCEL_PRINT` | Standard print control |
| `GENTLE_Z_TILT` | Quiet automated Z leveling |
| `MANUAL_Z_SYNC` | Interactive Z screw adjustment guide |
| `QUICK_Z_CHECK` | Measure and report gantry tilt |
| `INPUT_SHAPER_TEST_X` / `_Y` | Full resonance test at 3 Z heights |
| `M108 P<value>` | Set fan speed percentage (0.0–1.0) |

### 3MS / MMU additions

| Macro | Description |
|---|---|
| `_MMU_BLOB_PURGE` | Blob purge on toolchange (called by Happy Hare) |
| `_MMU_ENDLESS_SPOOL_STUB_PURGE` | Eject old stub on runout (called by Happy Hare) |
| `ENDLESS_SPOOL_TOGGLE` | Enable/disable endless spool |
| `ENDLESS_SPOOL_STATUS` | Show current groups and state |
| `ENDLESS_SPOOL_SET_GROUPS G0=0 G1=0 G2=1 G3=1` | Set gate groups |
| `ENDLESS_SPOOL_GROUPS_ALL` | Put all gates in one group |

---

## Hardware

- **Printer:** Comgrow T300 (300×300×370mm)
- **MMU:** 3MS (3-Material System), 4 gates
- **MMU Software:** Happy Hare v3.42
- **Probe:** CR Touch (x_offset: 20, y_offset: -24)
- **Extruder:** Stock direct drive, 1.75mm filament
- **Hotend:** Dragon-style (cooling_tube_position: 35mm)
- **Accelerometer:** ESP32 + MPU-6050 (for input shaper)
