# AC400 IR Controller – Sender Configuration

This repository contains the `sender.yaml` configuration file used to transmit infrared (IR) commands for controlling an **AC400 air cleaner / air purifier**.

The file defines IR command sequences that replicate the functions of the original remote control and can be used with microcontrollers and IR transmitter hardware.

---

## 📄 File Overview

### `sender.yaml`

This YAML file contains:
- Infrared protocol definitions
- Raw or encoded IR command data
- Command mappings for air cleaner functions such as:
  - Power on/off
  - Fan speed levels
  - Operating modes
  - Additional device-specific features (e.g. night mode, ionizer, timer)

The file is designed to be included in a larger configuration, for example when using ESPHome.

---

## 🔧 Requirements

Typical hardware and software requirements:
- ESP8266 or ESP32 microcontroller
- IR LED (with appropriate resistor)
- Firmware or framework that supports IR transmission (e.g. ESPHome)
- Configured GPIO pin for the IR transmitter

---

## 🚀 Usage Example (ESPHome)

Example of including `sender.yaml` in an ESPHome configuration:

```yaml
packages:
  ac400_ir_sender: !include sender.yaml
