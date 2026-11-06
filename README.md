# salt-measure

A water softener salt-level monitor for Home Assistant: an ESP32-C3 with an ultrasonic
distance sensor (HC-SR04 style) looks down into the brine tank, ESPHome reports the
distance, and Home Assistant turns it into salt level, kilograms, daily consumption,
days remaining, low-salt alerts and a usage dashboard with period-over-period comparisons.

## Layout

| Path | What it is |
|---|---|
| `esphome/salt-sensor.yaml` | ESPHome firmware config (ESP32-C3, ultrasonic on GPIO0 trigger / GPIO1 echo) |
| `homeassistant/salt_softener.yaml` | Template sensors: level, mass, daily drop, consumed total, days remaining |
| `homeassistant/helpers.yaml` | `input_select` helpers used by the alert automation and dashboard |
| `homeassistant/salt_level_alerts.yaml` | Automation: warning below 50%, critical below 25% |
| `homeassistant/dashboard/water-softener.json` | Lovelace dashboard (raw config) |

## Setup

1. **Firmware.** Add `salt_level_sensor__encryption_key`, `wifi_ssid` and `wifi_password`
   to your ESPHome `secrets.yaml`, then flash `esphome/salt-sensor.yaml`.
2. **Calibrate.** In `salt_softener.yaml`, set `empty_cm` (sensor reading with an empty
   tank), `full_cm` and `full_kg` for your tank. The example values are 85.3 cm, 17.0 cm
   and 160 kg. The entity names (`sensor.salt_level_sensor_salt_tank_distance_raw`)
   depend on your ESPHome device name.
3. **Template sensors.** Include the file from `configuration.yaml`:
   `template: !include salt_softener.yaml`
4. **Helpers.** Create the three dropdowns in `helpers.yaml`.
5. **Alerts.** Add `salt_level_alerts.yaml` to your automations and replace
   `notify.mobile_app_YOUR_PHONE` with your own notify service.
6. **Dashboard.** Install [apexcharts-card](https://github.com/RomRider/apexcharts-card)
   through HACS, create a new dashboard, open the raw configuration editor and paste in
   `water-softener.json`.

## Notes

- `sensor.salt_softener_consumed` is a `total_increasing` sensor, so the dashboard can use
  long-term statistics (`change`) for weekly/monthly comparisons. It only starts counting
  from the first 23:55 run after you add it.
- `sensor.salt_softener_settled_distance` is expected to be a Statistics helper
  (median, 60 samples, 30 min) on `sensor.salt_softener_distance`; create it in
  Settings > Helpers.
- The ultrasonic sensor can be thrown off by condensation or a lid change, so readings
  beyond the empty floor are discarded and daily drops outside 0-8 cm are skipped.
