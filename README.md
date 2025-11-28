# ESPHome Smart EVSE Controller for Waveshare ESP32-S3

![GitHub release (latest by date)](https://img.shields.io/github/v/release/paulkastelijn/Wavesare-ESP32-S3-Relay_6CH)
![GitHub stars](https://img.shields.io/github/stars/paulkastelijn/Wavesare-ESP32-S3-Relay_6CH)
![GitHub license](https://img.shields.io/github/license/paulkastelijn/Wavesare-ESP32-S3-Relay_6CH)

Maintained by [Paul Kastelijn](https://github.com/paulkastelijn) — last updated on **28 November 2025**.

This project provides a complete [ESPHome](https://esphome.io/) configuration to build a smart Electric Vehicle Supply Equipment (EVSE) controller using a Waveshare ESP32-S3 board (like the Waveshare ESP32-S3-Relay-CH6 relay board). It turns a standard EV charger into a smart device, fully integrable with [Home Assistant](https://www.home-assistant.io/).

The primary goal is to enable intelligent EV charging, allowing for features like solar surplus charging, scheduled charging based on electricity tariffs, and remote monitoring and control.

## Features

- **Smart Charging Control:** Start, stop, and pause charging remotely.
- **Dynamic Current Adjustment:** Set the maximum charging current via Home Assistant.
- **Modbus Integration:** Reads data from a Modbus-enabled energy meter to monitor charging power, voltage, and current.
- **Home Assistant Integration:** Exposes all controls and sensors to Home Assistant for automation and dashboarding.
- **Standalone Web UI:** Access basic controls and sensor readings directly from the device's web interface.
- **Safety Interlocks:** Uses binary sensors to monitor the charger's state (e.g., vehicle connected, charging active).
- **Modular Configuration:** The ESPHome code is split into logical files, making it easy to understand and customize.

## Hardware Requirements

1.  **ESP32 Board:** A [Waveshare ESP32-S3 development board](https://www.waveshare.com/wiki/ESP32-S3-DEV-KIT-N8R8). The configuration is tailored for this but can be adapted for other ESP32-S3 models.
2.  **EVSE Hardware:** A basic EV charger (contactor, CP/PP signal generator). This firmware is designed to control the low-level components.
3.  **Modbus Energy Meter:** A meter (like a PZEM-004T or similar) to measure charging power. This is required for power monitoring features.
4.  **Power Supply:** A suitable power supply for the ESP32 board and any associated relays.

## Software Requirements

- **ESPHome:** You need a running instance of ESPHome to compile and flash this firmware. This can be the Home Assistant Add-on, a standalone Docker container, or a Python command-line installation.

## Project Structure

The ESPHome configuration is broken down into multiple YAML files for clarity and maintainability. They are all located in the `evse/` directory and included by the main `waveshare-esp32s3.yaml` file.

```
waveshare-esp32s3.yaml      # Main configuration file, includes all others
evse/
├── substitutions.yaml      # User-specific values (WiFi, device name, etc.)
├── device.yaml             # Core device definition (board type, project info)
├── base.yaml               # Base components (WiFi, API, Web Server, Logger)
├── modbus.yaml             # Modbus component setup
├── globals.yaml            # Global variables for internal state
├── sensors.yaml            # Sensor definitions (from Modbus, etc.)
├── text_sensors.yaml       # Text sensor definitions
├── binary_sensors.yaml     # Binary sensor definitions (e.g., charger status)
├── switches.yaml           # Switch entities for Home Assistant
├── outputs.yaml            # GPIO output definitions
├── numbers.yaml            # Number entities for setting values (e.g., charge current)
├── buttons.yaml            # Button entities for triggering actions
├── scripts.yaml            # Reusable automation scripts
└── intervals.yaml          # Timed interval components
```

## Installation and Configuration

1.  **Clone the Repository:**
    ```bash
    git clone https://github.com/paulkastelijn/Wavesare-ESP32-S3-Relay_6CH.git
    cd Wavesare-ESP32-S3-Relay_6CH
    ```

2.  **Configure Substitutions:**
    Open the `evse/substitutions.yaml` file. This is the **only file** you should need to edit for a basic setup. Fill in the following details:
    - `device_name`: A unique name for your EVSE controller (e.g., `garage-evse`).
    - `wifi_ssid`: Your Wi-Fi network name.
    - `wifi_password`: Your Wi-Fi password.
    - `ota_password`: A password for Over-The-Air updates.

3.  **Review Pin Assignments:**
    The default GPIO pin assignments are defined in files like `outputs.yaml`, `switches.yaml`, and `modbus.yaml`. If your wiring differs from the intended setup for the Waveshare board, you will need to adjust the `pin:` values in these files accordingly.

4.  **Compile and Upload:**
    - **Using Home Assistant ESPHome Add-on:**
        - Copy the entire project folder (containing `waveshare-esp32s3.yaml` and the `evse` directory) into your Home Assistant `esphome` configuration directory.
        - Open the ESPHome dashboard in Home Assistant.
        - Click "+ NEW DEVICE" and then "Continue".
        - Select "Install from an existing configuration file" and choose `waveshare-esp32s3.yaml`.
        - Follow the wizard to compile and upload the firmware. The first upload must be done over USB. Subsequent updates can be done wirelessly (OTA).
    - **Using ESPHome Command Line:**
        ```bash
        # Install firmware for the first time via USB
        esphome run waveshare-esp32s3.yaml

        # For subsequent wireless updates
        esphome run waveshare-esp32s3.yaml
        ```

5.  **Add to Home Assistant:**
    Once the device is online, Home Assistant should automatically discover it. A new device will appear in the "Integrations" page. Click "Configure" to add it to your setup.

## Usage

After adding the device to Home Assistant, you will have access to several new entities, including:

- **Switches:** To enable/disable charging.
- **Sensors:** To monitor voltage, current, power, and energy consumption.
- **Number Inputs:** To set the maximum charging current.
- **Binary Sensors:** To show the status of the charger (e.g., `Car Connected`).

You can add these entities to your Home Assistant dashboards or use them in automations to create your own smart charging logic.

## Contributing

Contributions are welcome! If you have ideas for improvements, find a bug, or want to add a new feature, please feel free to:

1.  Open an issue to discuss the change.
2.  Fork the repository and create a new branch.
3.  Submit a pull request with your changes.

## License

This project is released under the MIT License. See the LICENSE file for details.

---
**Author Links:** Follow project updates on [GitHub @paulkastelijn](https://github.com/paulkastelijn).
