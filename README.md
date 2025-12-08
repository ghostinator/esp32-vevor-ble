# ESPHome Vevor Diesel Heater BLE Controller

This project provides an **ESP32-based Bluetooth Low Energy (BLE)
bridge** between a **Vevor diesel heater** and **Home Assistant**.
It enables full control and telemetry via ESPHome while decoding the
proprietary BLE frames emitted by the heater.

This repository contains the ESPHome YAML configuration that handles:

-   BLE connection and automatic polling
-   Full telemetry decoding (temperatures, voltage, altitude, modes, errors, etc.)
-   Power control, mode selection, setpoint control (temperature or level)
-   Publishing all values and states to Home Assistant

------------------------------------------------------------------------

# 🔧 Build & Installation Instructions

## 1. Install ESPHome

You can use: - ESPHome Dashboard
- The `esphome` CLI
- Home Assistant's ESPHome add-on


## 2. Create `secrets.yaml`

A `secrets.example.yaml` file is included. Create your own `secrets.yaml` in the same folder:

``` yaml
mac_address: ""
passkey: "1234"
wifi_ssid: ""
wifi_password: ""
api_encryption_key: "vHF/emJMoAZz3q8k4JySuSZnYEa43IujpwN9+fQkwYk="
ota_password: "09ec3c09b5847b5a4eeef2c15ce8fb00"
```

### Secret values:

| Field                  | Description                                                                                       |
| -----------------------|---------------------------------------------------------------------------------------------------|
| **mac_address**        | Bluetooth MAC address of the heater. You can obtain this using any BLE scanner app on your phone. |
| **passkey**            | The heater passkey set in the Vevor app. Default is **1234** unless changed.                      |
| **wifi_ssid**          | Credentials for the network the ESP32 should join.                                                |
| **wifi_password**      | Credentials for the network the ESP32 should join.                                                | 
| **api_encryption_key** | ESPHome API encryption key used by Home Assistant.                                                |
| **ota_password**       | Password required for over-the-air firmware updates.                                              |

### ⚠️ Important: Change Your API Encryption Key and OTA Password

The values in `secrets.example.yaml` come from the original project this was forked from.
They remain in place for ease of setup, but **should be replaced before deploying your device**.

#### Generate a new API key:

``` bash
openssl rand -base64 32
```

#### Choose any strong OTA password:

``` bash
openssl rand -hex 16
```

Update your `secrets.yaml` accordingly.

## 4. Update the Device Name (Optional)

Inside the YAML:

``` yaml
substitutions:
  name: bt-vevor_ble
  friendly_name: Diesel_Air_Heater
```

Modify these if you want your device to appear differently in Home Assistant.

## 5. Compile & Flash

### First-time USB flash:

``` bash
esphome run vevor_ble.yaml
```

After the ESP32 is flashed once, all future updates can be done **OTA** via WIFI.

# 📡 Features & Home Assistant Integration

This ESPHome configuration performs **bidirectional BLE communication** with the heater.

## ✓ Controls Available

| Control                     | Description                                     |
|-----------------------------|-------------------------------------------------|
| **Heater Power (On/Off)**   | Heater On/Off                                   |
| **Mode Selection**          | Level mode or Automatic (temperature) mode.     |
| **Level Setpoint**          | Sets output level 0--10 (Level mode).           |
| **Temperature Setpoint**    | Sets target temperature in °C (Automatic mode). |

# 📊 Telemetry Decoding (Published to Home Assistant)

### Status Sensors

-   **Heater Running** (binary_sensor)
-   **Heater Mode** ("Level" / "Automatic")
-   **Glow Plug Status** ("Heating", "Running", "Cooling Down", etc.)
-   **Error Code**

### Environmental / System Sensors

-   **Cabin Temperature (°C)**
-   **Heater Core Temperature (°C)**
-   **Battery Voltage (V)**
-   **Altitude (m)**
-   **Uptime**

### Device Metadata

-   Part number
-   Motherboard firmware version
-   CO sensor active + PPM (if present)
-   Temperature & altitude units
-   Language broadcast by the controller

### Raw BLE Data

One raw characteristic sensor exposes the **full decrypted frame** for
debugging.

# 🙏 Credits & Related Projects

Huge thanks to the pioneers who decoded the heater protocol for
older models:

-   https://github.com/spin877/Bruciatore_BLE
-   https://github.com/Knutnoh/Bruciatore_BLE
-   https://community.home-assistant.io/t/vevor-diesel-heater-control-development-in-progress/832159/14
