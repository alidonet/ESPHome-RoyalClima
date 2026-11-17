<p align="center">
  <img src="images/royalclima.png" width="380" alt="Royal Clima">
  <img src="images/esp32.jpg" width="120" alt="ESP32">
</p>

[Русский](readme.md)

# ESPHome for Royal Clima

Firmware for connecting a **Royal Clima SOFFIO Primo** heat recovery ventilation unit to an ESP32. Control ventilation and view readings through Home Assistant or a web browser.

The series includes **RCS-250-P, RCS-350-P, RCS-500-P and RCS-650-P**. Tested on **RCS-650-P**. Compatibility with **Holtop XHBQ** units is possible but has not been tested.

- [Firmware for initial USB installation](bin/royalclima-firmware.factory.bin).
- [Firmware for Wi-Fi updates](bin/royalclima-firmware.ota.bin).
- [ESPHome configuration](RoyalClima.yaml).
- [Manufacturer documentation](docs/).

<p align="center">
  <img src="images/soffio.jpg" style="max-width: 100%; height: auto;" alt="Royal Clima SOFFIO Primo ventilation unit">
</p>

## What you need

- An **ESP32-WROOM-32** board, such as ESP32-DevKitC or DevKit V1.
- A **UART ↔ RS-485 transceiver with 3.3 V logic**, such as a MAX3485 module with DI, RO, DE and /RE pins.
- A USB data cable, an ESP32 power supply and connecting wires.
- A Wi-Fi network.

ESP32-C3, S3 and boards with different pins require corresponding configuration changes. The wiring below is for a standard ESP32-WROOM-32 board.

## Wiring

Disconnect the ventilation unit from power before connecting wires.

```text
+--------------------+        +------------------------+        +--------------------+
|  Ventilation unit  |        |    UART <-> RS-485     |        |       ESP32        |
|                GND | [1]    |                        |        |                    |
|    RS-485 port   B |<------>| B                   DI |<-------| GPIO17 / TX        |
|                  A |<------>| A                   RO |------->| GPIO16 / RX        |
|                12V | [1]    |               DE + /RE |<-------| GPIO18 / EN        |
|                    |        |                    GND |--------| GND                |
|                    |        |                    VCC |<-------| 3V3 [2]            |
+--------------------+        +------------------------+        |                    |
                                                                | USB / power        |
                                                                +--------------------+
```

[1] +12 V and GND on the unit's connector are for the DC-DC power option described below.

[2] 3.3 V power for a compatible MAX3485 module.

| ESP32 | RS-485 transceiver |
|---|---|
| GPIO17 | DI |
| GPIO16 | RO |
| GPIO18 | DE and /RE connected together |
| GND | GND |
| 3V3 | VCC of a 3.3 V module |

Connect transceiver terminals **A and B** to **A and B** on the unit's RS-485 connector. Power the ESP32 through USB or the board's designated power input. Check the transceiver's pin labels.

![Ventilation unit board wiring diagram](images/scheme_en.png)

Set **SW4-4 to ON for 9600 baud** on the ventilation unit's board. The default unit address is **1** (parameter 08 on the wired controller). If you change it on the unit, set the same `modbus_address` in `substitutions` and rebuild the firmware; `baud_rate` is configured there too. Both polling groups use the same address.

### Power from the ventilation unit

One option is an inexpensive **12 V → 5 V step-down DC-DC converter** connected to **+12 V and GND** on the same RS-485 connector. Feed its 5 V output into an ESP32 input rated for 5 V. This connects the module to the unit using four wires: **+12 V, GND, A and B**.

**This option has not been tested in practice.** Check the permissible load of the 12 V output and whether it can supply enough power for the ESP32 and transceiver. Do not apply 12 V directly to the ESP32. Do not connect USB and DC-DC power simultaneously unless the board supports it.

## Installing the firmware

### Prebuilt firmware

**The files in `bin/` have not yet been rebuilt for the new Wi-Fi provisioning configuration.** Wi-Fi setup through Improv Serial or `royalclima Setup`, described below, applies to builds from the updated YAML; older binaries may behave differently.

1. Connect the ESP32 to your computer using USB.
2. Open [ESPHome Web](https://web.esphome.io/) in Chrome or Edge, connect to the board and install the **factory.bin** file linked above.
3. If the installer offers Wi-Fi setup over USB (Improv Serial), enter your network details. Otherwise, connect your phone or computer to the open **royalclima Setup** network and open `http://192.168.4.1/`: select your 2.4 GHz Wi-Fi network and enter its password.
4. Open `http://royalclima.local/` or the ESP32's IP address on your network.

For subsequent Wi-Fi updates, use **ota.bin** through ESPHome.

### Installation through ESPHome

1. Add [RoyalClima.yaml](RoyalClima.yaml) to ESPHome Device Builder.
2. Adjust `tx_pin`, `rx_pin`, `en_pin`, `baud_rate` and `modbus_address` in `substitutions` if necessary.
3. Select **Install**. Use USB for the first installation and Wi-Fi for subsequent updates. Configure Wi-Fi after flashing, through USB or **royalclima Setup**, as described above; Wi-Fi does not require `secrets.yaml`.

Wi-Fi credentials are stored on the ESP32. Normal OTA updates preserve them as long as the YAML does not specify an SSID and password. Erasing flash or performing a factory reset clears the settings; USB flashing that erases flash requires provisioning again. If the saved network is unavailable, the device returns to setup mode (fallback AP timeout: 30 seconds). See [ESPHome Wi-Fi credential persistence](https://esphome.io/components/wifi/#saving-wifi-credentials).

## Features

- Turn ventilation on and off.
- Set supply and exhaust fan speeds independently: **steps 0–10**, where 0 means stopped.
- Four temperatures: outdoor air, return air from the room, supply air and exhaust air discharged outside.
- Bypass, defrost, fire input and boost states.
- Fan runtime and the configured filter reminder interval.
- Read and change humidity and heater temperature setpoints.
- Text status and error messages in Russian or English, selected through **Message language**.

Sensor boost depends on the unit's settings: **CO₂ or humidity**. Separate external boost is activated by closing the External switch input on the ventilation unit's board.

## Sensors and their purpose

Names are listed as they appear in the firmware. Ventilation entities belong to **Ventilation System**; ESP32 diagnostics belong to **Royal Clima**.

### Ventilation readings

| Name | Purpose |
|---|---|
| Return air temperature | Temperature of air extracted from the room (RA). |
| Outdoor air temperature | Outdoor air temperature (OA). |
| Supply Air Temperature | Temperature of air supplied to the room (SA). |
| Exhaust air temperature | Temperature of air discharged outside (FR). |
| Supply fan speed | Current supply fan step, from 0 to 10; 0 means stopped. |
| Exhaust fan speed | Current exhaust fan step, from 0 to 10; 0 means stopped. |
| Humidity setpoint | Configured humidity, not measured room humidity. |
| Heater temperature setpoint | Configured heater temperature. |
| Filter reminder interval | Configured filter reminder period in days, not days remaining. |
| Fan runtime | Fan runtime in hours. |
| Filter usage | Runtime as a percentage of the configured filter interval: hours / (days × 24) × 100. Can exceed 100%; unknown when source data is missing or stale. |

Speeds are steps, not RPM or percentages. Humidity is a setpoint, not a measurement. The filter interval is a configured period, not time remaining until maintenance.

### Ventilation states

| Name | Purpose |
|---|---|
| Vent system Online | Up-to-date data is available from the ventilation unit. |
| Ventilation power | The unit is on or off. |
| Bypass open | The bypass is open or closed. |
| Bypass closed * | An additional bypass-closed indicator. |
| Defrost active | Defrost mode is active. |
| Fire alarm | The unit's fire input is active. |
| Ventilation error | The unit reports an error. |
| External input boost active | Boost from the external input is active. |
| Sensor boost active | Boost from the CO₂ or humidity sensor is active, depending on the unit's settings. |
| Ventilation status | Text description of the unit's status in the selected language. |
| Error description | Description of the unit's errors in the selected language. |

### ESP32 and communication diagnostics

| Name | Purpose |
|---|---|
| WiFi signal | ESP32 Wi-Fi signal level. |
| Uptime | ESP32 time since reboot, in seconds. |
| ESP32 connection | The ESP32 is connected to Home Assistant or another ESPHome client. |
| Project version | Installed project firmware version. |
| Seconds since last data | Seconds since the last successful read of the unit's main data. |
| Offline events | Number of communication failures since ESP32 startup. |

### Additional diagnostic sensors

These values are intended for diagnostics and are usually hidden. An asterisk (*) marks entities disabled by default; enable them in the ESPHome integration settings if needed.

| Name | Purpose |
|---|---|
| Power state raw * | Raw unit power state. |
| Supply fan speed code * | Raw supply fan speed code. |
| Exhaust fan speed code * | Raw exhaust fan speed code. |
| Status flags raw * | Raw unit status flags. |
| Error flags raw * | Raw unit error code. |
| External input boost signal raw * | Raw boost signal from the external input. |
| Sensor boost signal raw * | Raw boost signal from the CO₂ or humidity sensor. |
| Filter interval code * | Raw code for the configured filter reminder period. |

### Controls and settings

| Name | Purpose |
|---|---|
| Ventilation control | Turn the unit on and off. |
| Supply speed control | Select the supply fan step: 0–10. |
| Exhaust speed control | Select the exhaust fan step: 0–10. |
| Humidity setpoint control * | Change the humidity setpoint: 1–99 %. |
| Heater temperature setpoint control * | Change the heater temperature setpoint: 10–25 °C. |
| Message language | Message language: ru for Russian, en for English. Defaults to en; the selection is saved. |
| ESP32 Reboot | Restart only the ESP32, without restarting the ventilation controller. |

## Connecting to Home Assistant

In Home Assistant, open **Settings → Devices & services** and add the discovered ESPHome device. If it is not discovered automatically, add the **ESPHome** integration and enter the ESP32's IP address.

For convenient control, use the separate [Royal Clima ERV Card](https://github.com/alidonet/ha-royal-clima-card) project. Installation and configuration instructions are available in its repository.
Example card views in Home Assistant (Russian labels):

![Royal Clima control panel in Home Assistant](images/control_work.png)

![Royal Clima airflow view in Home Assistant](images/flow_work.png)

Readings and controls are also available in a browser at the ESP32's address. The firmware supports MQTT after configuring the broker connection in YAML.
