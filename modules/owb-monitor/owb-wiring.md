---
title: Wiring
parent: OWB Monitor
nav_order: 1
---

# OWB Monitor — Wiring

> **TODO:** Wiring diagram to be added after first site visit. GPIO pin assignments to be confirmed against LILYGO T-SIM7080-S3 pinout.

---

## Temperature Sensors — DS18B20

Four DS18B20 waterproof probes mounted on pipes using hose clamps with thermal paste. All four sensors share a single 1-Wire data line.

| Sensor | Location | GPIO |
|---|---|---|
| House loop supply | Supply pipe — house loop | GPIO4 (1-Wire bus) |
| House loop return | Return pipe — house loop | GPIO4 (1-Wire bus) |
| Barn loop supply | Supply pipe — barn loop | GPIO4 (1-Wire bus) |
| Barn loop return | Return pipe — barn loop | GPIO4 (1-Wire bus) |

> **TODO:** Confirm GPIO4 against LILYGO T-SIM7080-S3 pinout before flashing.

### Mounting

- Apply thermal paste to the pipe at each sensor location
- Place the DS18B20 probe flat against the paste
- Secure with a hose clamp sized to the pipe OD

> **TODO:** Measure pipe OD at all four locations on site — needed for hose clamp sizing.

### 1-Wire Pull-up Resistor

The 1-Wire bus requires a 4.7kΩ pull-up resistor between the data line and 3.3V. Place it at the node end of the wire run, not at the sensor.

---

## Current Sensors — SCT-013-030

Two CT clamps, one per pump. Each clamp goes around the **hot conductor only** — not both conductors in the cable. Clamping both conductors cancels the magnetic field and produces a null reading.

The pumps are plug-in. A custom monitored outlet box routes each pump's hot conductor through a CT clamp before the outlet. The box plugs into a standard 120V outlet — no panel work required.

| Sensor | Pump | ADS1115 Channel |
|---|---|---|
| House pump current | House loop pump | A0 (differential A0_GND) |
| Barn pump current | Barn loop pump | A1 (differential A1_GND) |

### Burden Resistor

The SCT-013-030 requires a burden resistor across its output to convert current to voltage for the ADS1115. Without it the output will rail.

> **TODO:** Confirm burden resistor value against ADS1115 input range (0–3.3V) before building the outlet box.

---

## ADS1115 — I2C

| ADS1115 Pin | LILYGO T-SIM7080-S3 Pin |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SDA | GPIO8 |
| SCL | GPIO9 |
| ADDR | GND (default address 0x48) |

> **TODO:** Confirm GPIO8/9 for I2C against LILYGO T-SIM7080-S3 pinout.

---

## Power

USB-C for the initial deployment. LiPo and solar input planned for the permanent install.
