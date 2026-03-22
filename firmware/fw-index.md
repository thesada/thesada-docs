---
title: Firmware
nav_order: 5
has_children: true
---

# Firmware

Thesada nodes run one of two firmware approaches depending on the use case:

**ESPHome** is used for simpler sensor nodes where its built-in components cover the requirements. Configuration lives in `thesada-cfg/esphome/`. See the [Getting Started](../getting-started/esphome.md) guide and each module's ESPHome Config page.

**Custom firmware** (`thesada-fw`) is used where ESPHome lacks coverage — cellular fallback, Lua scripting, complex inter-module logic, or full control over the build. This section documents the custom firmware.

---

## Quick links

| | |
|---|---|
| [Architecture](fw-architecture.md) | Module design, boot sequence, Shell, Lua, OTA, connectivity |
| [Testing](fw-testing.md) | Test script, manual checks, section-by-section verification |
| [Source](https://github.com/thesada/thesada-fw) | GitHub — GPL-3.0-only |

---

## Feature overview

| Feature | Notes |
|---|---|
| WiFi multi-SSID | Ranked by RSSI; NTP synced on connect |
| LTE-M/NB-IoT fallback | SIM7080G modem-native MQTT over TLS; reverts to WiFi automatically |
| TLS MQTT | CA cert from LittleFS (`/ca.crt`); ISRG Root X1 works for Let's Encrypt brokers |
| DS18B20 temperature | OneWire, multi-sensor, auto-discovery, configurable interval |
| ADS1115 current sensing | Differential measurement, configurable gain |
| SD card logging | CSV, per-boot files, configurable max file size with auto-rotation |
| Lua scripting | Lua 5.3 runtime; hot-reloadable rules, EventBus + MQTT bindings |
| Shell CLI | 27 commands over serial, WebSocket (`/ws/serial`), and HTTP (`POST /api/cmd`) |
| OTA — push | Upload `.bin` via web dashboard or curl |
| OTA — pull | Fetch manifest JSON, SHA256 verify, auto-install; works with GitHub Releases |
| Heartbeat LED | Blue CHGLED pulse via AXP2101 at configurable interval |
| NTP log timestamps | ISO 8601 UTC in log lines once clock is synced |
| Temperature alerts | Threshold rules with hysteresis, MQTT + HTTP webhook |

---

## Hardware

Target board: **LILYGO T-SIM7080-S3** (ESP32-S3, SIM7080G modem, AXP2101 PMU)

Build target: `esp32-s3-dev` in PlatformIO (`base/platformio.ini`).
