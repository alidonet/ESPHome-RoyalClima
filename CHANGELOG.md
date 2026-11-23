# Changelog

## v1.0.0

- Added a saved 3/10 fan-speed selection in the device web interface. ESP32 restarts after a successful settings flush to apply the selected mode.
- Reject power, fan-speed and setpoint writes when the main Modbus readings are missing or stale.
- Continue tracking power and speed commands after mismatched reads until confirmation or timeout.
- Added a Filter due indicator when filter usage reaches 100%.
- Added Read requests and Read success diagnostics, disabled by default, and disabled Offline count by default.
- Added a MAC suffix to the ESP32 network name to prevent MQTT Discovery collisions between multiple adapters. Connections using the former hostname need updating.

## v0.9.x

- Added MQTT configuration through the device web interface.
- Disabled MQTT settings, target controls and fault bits by default in Home Assistant.
- Removed duplicate read-only entities for power, fan speeds and targets; controls display values read from the unit.
- Raised the minimum ESPHome version to 2026.11.0 and linked the Vent device to the ESP32 as its gateway in Home Assistant.
- Allowed more time for the unit to report its state after a power or fan-speed command. The firmware can confirm a command from a later reading.
- Retained the last successful readings for fan runtime, filter interval, and heater setpoint until new data arrives or the ESP32 reboots. Filter usage now updates when the runtime or filter interval is read.
- Added support for 0–3 and 0–10 fan speeds;

## v0.8.x

- Added readback tracking for power and fan-speed commands.
- Shortened entity names. After upgrading, check entity IDs in Home Assistant and update automations or cards that reference them.

## v0.7.0

- Added a filter usage sensor based on fan runtime and the configured filter interval.
- Try to improve commands reply.

## v0.6.0

- Added Wi-Fi setup through Improv Serial or the ESP32 setup access point.
- Retained the last readings through short communication interruptions.
- Renamed the boost indicators to distinguish an external input from a CO₂ or humidity sensor, depending on the unit configuration. Renamed the exhaust-air temperature sensor.

## v0.5.0

- Try to fix looped disconnect issue,  reduce queries for rare-sensors (slow reading);
- Add language option (en/ru); return translated messages;
- Add version for tracking changes;

## v0.4.0

- Added a fan runtime sensor and filter reminders in the bundled Home Assistant example.

## v0.3.0

- Added the setup guide for ESP32 and Royal Clima Soffio Primo. This release included monitoring, power and fan-speed controls, and prebuilt factory and OTA firmware.

## v0.2.0

- update timeous for modbus, all available sensors (R/O);

## v0.1.0
- Initial version with basic sensors (R/O).
