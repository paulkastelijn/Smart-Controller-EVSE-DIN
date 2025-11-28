# ESPHome Smart EVSE Controller for Waveshare ESP32-S3

![GitHub release (latest by date)](https://img.shields.io/github/v/release/paulkastelijn/Smart-Controller-EVSE-DIN)
![GitHub stars](https://img.shields.io/github/stars/paulkastelijn/Smart-Controller-EVSE-DIN)
![GitHub license](https://img.shields.io/github/license/paulkastelijn/Smart-Controller-EVSE-DIN)

Maintained by [Paul Kastelijn](https://github.com/paulkastelijn) — last updated on **28 November 2025**.

This project provides a complete [ESPHome](https://esphome.io/) configuration to build a smart Electric Vehicle Supply Equipment (EVSE) controller using a Waveshare ESP32-S3 board (like the Waveshare ESP32-S3-Relay-CH6 relay board). It turns a standard EV charger into a smart device, fully integrable with [Home Assistant](https://www.home-assistant.io/).

Before assembling the hardware or flashing ESPHome, make sure you already run [EVCC](https://evcc.io/). EVCC is the **brain** of the solution: it orchestrates charging sessions, optimizes them against tariffs or PV surplus, and exposes the high-level logic that this project relies on. Without EVCC there is no smart control layer to talk to the ESP32 firmware.

Think of the overall system as:

- **EVCC — the Brain:** Provides the decision making, schedules, surplus logic, and talks Modbus/MQTT to control charging targets.
- **ESP32 (this project) — the Bridge:** Translates EVCC commands into GPIO and Modbus actions, exposes sensors back to EVCC/Home Assistant, and keeps the communication reliable.
- **EVSE DIN Controller — the Muscle:** The physical hardware that enforces the requested current limits and safely enables or disables power delivery to the car.

The EV side hardware is the [EVSE DIN controller](https://www.evracing.cz/evse-din/); this firmware uses its Modbus register map to issue commands and read status information.

See AI generated video with about this project, the intentions and goals: https://youtu.be/nh9oXZltK5U

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
secrets.yaml                # Your private credentials (WiFi, passwords, etc.)
evse/
├── substitutions.yaml      # User-specific values (device name, etc.)
├── device.yaml             # Core device definition (board type, project info)
├── base.yaml               # Base components (WiFi, API, Web Server, Logger)
├── modbus.yaml             # Modbus component setup for EVSE controller
├── globals.yaml            # Global variables for internal state
├── sensors.yaml            # Sensor definitions (from Modbus, etc.)
├── text_sensors.yaml       # Text sensor definitions
├── binary_sensors.yaml     # Binary sensor definitions (e.g., charger status)
├── switches.yaml           # Switch entities for Home Assistant
├── outputs.yaml            # GPIO output definitions (LEDs)
├── numbers.yaml            # Number entities for setting values (e.g., charge current)
├── buttons.yaml            # Button entities for triggering actions
├── scripts.yaml            # Reusable automation scripts
└── intervals.yaml          # Timed interval components
```

## Wiring and Connections

Properly wiring the components is critical for the system to function correctly and safely. The pin assignments are defined across several YAML files in the `evse/` directory.

### ESP32 to EVSE DIN Controller (Modbus RTU)

Communication with the EVSE DIN controller is handled via Modbus RTU over an RS485 serial connection. You will need an **RS485 to TTL converter module** to interface between the ESP32 and the controller.

1.  **Connect the ESP32 to the RS485 to TTL module:**
    -   ESP32 `TX` pin → RS485 Module `DI` (Data Input)
    -   ESP32 `RX` pin → RS485 Module `RO` (Receiver Output)
    -   ESP32 `GPIO` pin for flow control → RS485 Module `DE` and `RE` (Driver/Receiver Enable, often jumpered together)

    *To find the exact GPIO pins used for `TX`, `RX`, and flow control, check the `uart:` section within the `evse/modbus.yaml` file.*

2.  **Connect the RS485 module to the Modbus bus:**
    -   RS485 Module `A` → EVSE Controller Modbus `A` terminal
    -   RS485 Module `B` → EVSE Controller Modbus `B` terminal

    *Note: If you are also using a Modbus energy meter, it should be connected to the same A/B bus lines in parallel.*

### Relays for Phase Switching

This project is designed for the **Waveshare ESP32-S3-Relay-CH6**, which has 6 on-board relays. These are used to control the main contactor and switch between 1-phase and 3-phase charging.

-   **Relay 1 (Contactor):** Controls the main power of the secondary contactor of the EVSE, controling phase 2 and 3. The first contactor should be connected to the DIN EVSE and controlls Phases 1, 2 and 3. The First and second contactor should be connected in series for phases 2 and 3, so that the secondary contactor will swithch phase 2 and 3. Phase swithching is in compliance with the safety standards for EV charging, ensure switching will only occur of no power is on the releays. 

### Status LED
-   **Relay 2, 3 and 4 (for external collored led ring powering):** Used to switch the led ring of an evbox. in this project this wil switch a 12 volts ledring Green, Blue and Red

*The specific GPIO pins connected to these relays are defined in the `switch:` section of the `evse/switches.yaml` file. Verify your wiring matches these definitions.*

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
    git clone https://github.com/paulkastelijn/Smart-Controller-EVSE-DIN.git
    cd Smart-Controller-EVSE-DIN
    ```

2.  **Configure Substitutions:**
    Open `secrets.yaml` and fill in your network credentials, passwords, and any other private information. This file is ignored by Git to protect your data.
    Then, open `evse/substitutions.yaml` to set non-sensitive, device-specific parameters like `device_name`.

3.  **Review Pin Assignments:**
    Before flashing, carefully review the pin assignments in `evse/modbus.yaml`, `evse/switches.yaml`, and `evse/outputs.yaml` to ensure they match your physical wiring.

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
