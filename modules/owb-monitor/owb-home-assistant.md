---
title: Home Assistant
parent: OWB Monitor
nav_order: 3
---

# OWB Monitor — Home Assistant

The OWB monitor publishes sensor data via MQTT. Home Assistant auto-discovers all entities via MQTT discovery — no manual entity configuration needed.

---

## Prerequisites

- OWB node flashed and online
- Mosquitto MQTT broker running (see [Mosquitto MQTT Broker](../../getting-started/mosquitto.md))
- Telegram bot set up in Home Assistant (see [Telegram](../../getting-started/telegram.md))

---

## 1. Verify MQTT Entities

Once the node is online and publishing, entities will appear automatically in Home Assistant.

**Settings → Devices & Services → MQTT**

You should see six entities:

| Entity | Description |
|---|---|
| `sensor.house_loop_supply_temp` | House loop supply temperature (°C) |
| `sensor.house_loop_return_temp` | House loop return temperature (°C) |
| `sensor.barn_loop_supply_temp` | Barn loop supply temperature (°C) |
| `sensor.barn_loop_return_temp` | Barn loop return temperature (°C) |
| `sensor.house_pump_current` | House loop pump current draw (A) |
| `sensor.barn_pump_current` | Barn loop pump current draw (A) |

If entities do not appear, confirm the node is publishing by subscribing to the MQTT topic:

```bash
mosquitto_sub -h YOUR_HA_IP -p 1883 \
  -u mqtt-user -P YOUR_PASSWORD \
  -t 'thesada/owb/#' -v
```

---

## 2. Alert Automations

Three automations cover the critical failure conditions. Create each via **Settings → Automations → Create Automation → Edit in YAML**.

Replace `notify.YOUR_NOTIFY_ENTITY` with your actual Telegram notify entity ID.

### Supply Temperature Low

Fires when the boiler supply temperature drops below 40°C — boiler going out or not heating.

```yaml
alias: OWB Supply Temp Low
description: Alert when boiler supply temperature drops below 40°C
trigger:
  - platform: numeric_state
    entity_id: sensor.house_loop_supply_temp
    below: 40
action:
  - action: notify.YOUR_NOTIFY_ENTITY
    data:
      message: |-
        Thesada OWB Alert:
        Supply temp low — {{ states('sensor.house_loop_supply_temp') | round(1) }}°C
        Check fire.
mode: single
```

### Loop Delta Collapsed

Fires when the difference between supply and return temperature drops below 5°C — possible flow blockage or pump issue.

```yaml
alias: OWB Loop Delta Low
description: Alert when supply/return delta collapses below 5°C
trigger:
  - platform: template
    value_template: >
      {{ (states('sensor.house_loop_supply_temp') | float -
          states('sensor.house_loop_return_temp') | float) < 5 }}
action:
  - action: notify.YOUR_NOTIFY_ENTITY
    data:
      message: |-
        Thesada OWB Alert:
        House loop delta low.
        Supply: {{ states('sensor.house_loop_supply_temp') | round(1) }}°C
        Return: {{ states('sensor.house_loop_return_temp') | round(1) }}°C
mode: single
```

### Pump Current Zero

Fires when a pump stops drawing current during heating hours — possible pump failure.

```yaml
alias: OWB Pump Current Zero
description: Alert when pump current drops to zero during heating hours
trigger:
  - platform: numeric_state
    entity_id: sensor.house_pump_current
    below: 0.1
  - platform: numeric_state
    entity_id: sensor.barn_pump_current
    below: 0.1
condition:
  - condition: time
    after: "06:00:00"
    before: "23:00:00"
action:
  - action: notify.YOUR_NOTIFY_ENTITY
    data:
      message: |-
        Thesada OWB Alert:
        {{ trigger.to_state.attributes.friendly_name }} not drawing current.
        Possible pump failure.
mode: single
```

---

## 3. Dashboard

> **TODO:** OWB dashboard YAML to be added after first real data is collected on site.
