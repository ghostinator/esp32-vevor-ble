# ESPHome Vevor/Hcalory Diesel Heater BLE Controller

This project provides an **ESP32-based Bluetooth Low Energy (BLE) bridge** between a **Vevor/Hcalory diesel heater** and **Home Assistant**. It enables full control and telemetry via ESPHome while decoding the proprietary BLE frames emitted by the heater.

**Significant Fork Note:** This version has been specifically updated to support **newer "Black/Red" controller boards** that emit **20-byte Bluetooth packets**. The original code (and many other forks) expects 45-byte packets and fails to read data from these newer units (often showing 0 values). This version also fixes voltage/temperature decoding errors caused by Little Endian byte ordering.

This repository contains the ESPHome YAML configuration that handles:

-   BLE connection and automatic polling using 16-bit UUIDs for stability
-   **Universal Telemetry Decoding:** Supports newer 20-byte frames (fixing the "Zero Data" bug) and standard 45-byte frames
-   **Correct Math:** Little Endian decoding for Voltage, Temperature, and Altitude
-   Power control, mode selection, setpoint control (temperature or level)
-   Publishing all values and states to Home Assistant

---

# 🔧 Build & Installation Instructions

## 1. Install ESPHome

You can use:
- ESPHome Dashboard
- The `esphome` CLI
- Home Assistant's ESPHome add-on


## 2. Create `secrets.yaml`

A `secrets.example.yaml` file is included. Create your own `secrets.yaml` in the same folder:

```yaml
mac_address: ""
passkey: "1234"
wifi_ssid: ""
wifi_password: ""
api_encryption_key: "vHF/emJMoAZz3q8k4JySuSZnYEa43IujpwN9+fQkwYk="
ota_password: "09ec3c09b5847b5a4eeef2c15ce8fb00"
```

### Secret values:

| Field | Description |
| :--- | :--- |
| **mac_address** | Bluetooth MAC address of the heater. You can obtain this using any BLE scanner app on your phone. |
| **passkey** | The heater passkey set in the Vevor app. Default is **1234** unless changed. |
| **wifi_ssid** | Credentials for the network the ESP32 should join. |
| **wifi_password** | Credentials for the network the ESP32 should join. |
| **api_encryption_key** | ESPHome API encryption key used by Home Assistant. |
| **ota_password** | Password required for over-the-air firmware updates. |

### ⚠️ Important: Change Your API Encryption Key and OTA Password

The values in `secrets.example.yaml` come from the original project this was forked from.
They remain in place for ease of setup, but **should be replaced before deploying your device**.

#### Generate a new API key:

```bash
openssl rand -base64 32
```

#### Choose any strong OTA password:

```bash
openssl rand -hex 16
```

Update your `secrets.yaml` accordingly.

## 4. Update the Device Name (Optional)

Inside the YAML:

```yaml
substitutions:
  name: bt-vevor_ble
  friendly_name: Diesel_Air_Heater
```

Modify these if you want your device to appear differently in Home Assistant.

## 5. Compile & Flash

### First-time USB flash:

```bash
esphome run vevor_ble.yaml
```

After the ESP32 is flashed once, all future updates can be done **OTA** via WIFI.

---

# 📡 Features & Home Assistant Integration

This ESPHome configuration performs **bidirectional BLE communication** with the heater.

## ✓ Controls Available

| Control | Description |
| :--- | :--- |
| **Heater Power (On/Off)** | Heater On/Off |
| **Mode Selection** | Level mode or Automatic (temperature) mode. |
| **Level Setpoint** | Sets output level 1–10 (Level mode). |
| **Temperature Setpoint** | Sets target temperature in °C (Automatic mode). |

# 📊 Telemetry Decoding (Published to Home Assistant)

### Status Sensors

-   **Heater Running** (binary_sensor)
-   **Heater Mode** ("Level" / "Automatic" / "Standby")
-   **Glow Plug Status** ("Heating", "Running", "Cooling Down", "Idle")
-   **Error Code**

### Environmental / System Sensors

-   **Cabin Temperature (°C)**
-   **Heater Core Temperature (°C)**
-   **Battery Voltage (V)**
-   **Altitude (m)**
-   **Uptime**
-   **WiFi Signal**

### Debugging Sensors

-   **Last Hex Packet**: Shows the raw hex string of the last received Bluetooth packet. Useful for debugging new controller variants.

---

# 🛠 Technical Details (Why this fork exists)

Many Vevor/Hcalory heaters have switched to a newer controller board (often Red or Black casing) that behaves differently than the older "Blue" boards.

1.  **Packet Size:** The old boards sent 45+ bytes of data. The new boards send concise **20-byte** packets. Most existing scripts ignore these "short" packets, resulting in no data. This repo accepts `x.size() >= 20`.
2.  **Ghost Packets (The "Home Assistant Crash" Fix):** Some controllers occasionally emit a massive **128-byte** packet filled with zeros. If passed to Home Assistant, this string exceeds the 255-character state limit, causing the sensor integration to crash with an error. This repo explicitly filters out any packet larger than 25 bytes to prevent this.
3.  **Byte Order:** The new boards send multi-byte data (Voltage, Temp, Altitude) in **Little Endian** format (Low Byte First). Older scripts read this as Big Endian, resulting in massive, incorrect values (e.g., 31,000 Volts). This repo correctly calculates `Low + (High * 256)`.
4.  **Connection Stability:** This configuration uses Short UUIDs (`0xFFE0` / `0xFFE1`) rather than 128-bit UUIDs for the service definition, which improves connection reliability with these specific Chinese BLE chips.
5.  **Decoder Method:** The decoding logic is placed inside a `sensor` lambda using `type: characteristic`. This ensures that every notification is processed immediately, bypassing some of the standard ESPHome sensor filters that can block non-compliant BLE frames.

# 🙏 Credits & Related Projects

Huge thanks to the pioneers who decoded the heater protocol for older models:

-   [https://github.com/spin877/Bruciatore_BLE](https://github.com/spin877/Bruciatore_BLE)
-   [https://github.com/Knutnoh/Bruciatore_BLE](https://github.com/Knutnoh/Bruciatore_BLE)
-   [https://community.home-assistant.io/t/vevor-diesel-heater-control-development-in-progress/832159/14](https://community.home-assistant.io/t/vevor-diesel-heater-control-development-in-progress/832159/14)

## Support and Donations

If you find these scripts helpful and would like to support further development or simply show your appreciation, you can use the following donation addresses:

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/H2H25YLCS)
* **PayPal:** brandon@ghostinator.co
* **SOL:** AUyf3DxRL9uKWf7ePEjY5GhSL3XJtAqQZ2w7kUMLi1xS
* **ETH, Monad, Base, Polygon:** 0x521E4D25212180B5f8A9f0261D00835EA3bd7fa6
* **Sui:** 0x89844c9e0c509115e8d2747b02d230ceb2bf96a1246707416177457d0e1bb593e
* **Bitcoin:** bc1qkljmuw3mexksnk0wg06p35xvhpkakgnhyp63la
