# Car Presence Detection System

This project utilizes an ESPHome configuration to detect the presence of a car on a parking spot using a Wemos D1 Mini (ESP8266) board and an ultrasonic sensor. The system determines the car's presence based on measured distances and provides occupancy feedback.

---

## Features
- **Ultrasonic Distance Measurement**: Uses an ultrasonic sensor to measure distances and identify if a car is present within a defined range.
- **Wi-Fi Connectivity**: Connects to your Wi-Fi network for remote monitoring via Home Assistant.
- **Fallback AP Mode**: Provides a fallback access point for configuration in case of Wi-Fi issues.
- **OTA Updates**: Enables over-the-air firmware updates for easy maintenance.
- **Real-Time Status**: Displays IP, MAC address, and firmware version on the dashboard.
- **Custom Logic for Presence Detection**: Tracks if the distance is within the desired range for a prolonged period.

---

## Hardware Requirements
- **Wemos D1 Mini (ESP8266)**
- **Ultrasonic Sensor (e.g., HC-SR04)**
- **Connecting Wires**
- **Power Source (5V)**

---

## Connections
### Ultrasonic Sensor
- **Trigger Pin**: Connect to D1 (GPIO5) on the Wemos D1 Mini.
- **Echo Pin**: Connect to D2 (GPIO4) on the Wemos D1 Mini.
- **VCC**: Connect to the 5V pin on the Wemos D1 Mini.
- **GND**: Connect to the GND pin on the Wemos D1 Mini.

---

## Configuration Overview
### Substitutions
```yaml
substitutions:
  name: car-presence-workshop
  friendly_name: "Car Presence Workshop"
  board: d1_mini
```
- `name`: Internal name for the ESPHome node.
- `friendly_name`: Display name for the node.
- `board`: Specifies the ESP8266 board type.

---

## Main Components
### Wi-Fi Settings
```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: ${friendly_name} Fallback AP
    password: !secret car_presence_workshop_ap_password
```
- Connects to your Wi-Fi network using credentials stored in the `secrets.yaml` file.
- Enables a fallback access point for troubleshooting.

### Ultrasonic Sensor Configuration
```yaml
sensor:
  - platform: ultrasonic
    trigger_pin: D1
    echo_pin: D2
    name: "Ultrasonic Distance"
    update_interval: 10s
    id: ultrasonic_sensor
```
- Measures distance every 10 seconds using the HC-SR04 sensor.
- `trigger_pin` and `echo_pin`: Define GPIO pins connected to the sensor.

### Binary Sensors
#### Car Presence
```yaml
binary_sensor:
  - platform: template
    name: "Car Presence"
    id: car_presence
    device_class: occupancy
    lambda: |-
      if (id(distance_in_range).state) {
        return true;
      } else {
        return false;
      }
```
- Determines if a car is present based on the `distance_in_range` sensor.

#### Distance In Range
```yaml
binary_sensor:
  - platform: template
    name: "Distance In Range"
    id: distance_in_range
    internal: true
    lambda: |-
      return false;  # Default state
```
- Internal sensor to monitor if the measured distance falls within the specified range.

#### Interval for Presence Detection
```yaml
interval:
  - interval: 10s
    then:
      - lambda: |-
          static int check_counter = 0;
          float dist = id(ultrasonic_sensor).state;
          if (dist >= 50 && dist <= 200) {  // Replace with your desired range
            check_counter++;
          } else {
            check_counter = 0;
          }

          if (check_counter >= 12) {  // Car present for 2 minutes
            id(distance_in_range).publish_state(true);
          } else {
            id(distance_in_range).publish_state(false);
          }
```
- Detects the car’s presence if the distance remains in range for at least 2 minutes (12 cycles of 10 seconds).

---

## Additional Features
### Text Sensors
```yaml
text_sensor:
  - platform: version
    name: ESPHome Version - ${friendly_name}
  - platform: wifi_info
    ip_address: 
      name: IP - ${friendly_name}
    mac_addres:
      name: MAC - ${friendly_name}
```
- Displays firmware version, IP address, and MAC address on the Home Assistant dashboard.

### Logger
```yaml
logger:
```
- Provides debugging logs.

### API and OTA Updates
```yaml
api:
  encryption:
    key: !secret car_presence_workshop_api_encryption_key

ota:
  - platform: esphome
    password: !secret car_presence_workshop_ota_password
```
- **API**: Allows secure communication with Home Assistant.
- **OTA**: Enables remote firmware updates.

---

## Usage
1. Flashing the Firmware:
- Compile and upload the ESPHome configuration to the Wemos D1 Mini.
2. Wi-Fi Setup:
- If preconfigured, the device will connect automatically. Otherwise, use the fallback AP to configure Wi-Fi.
3. Dashboard Integration:
- View distance readings and car presence status in Home Assistant.
4. Calibrate Distance Range:
- Adjust the `50` to `200` range in the interval section based on your setup.

---

## Secrets
Ensure the following are defined in your `secrets.yaml` file:
- `wifi_ssid`
- `wifi_password`
- `car_presence_workshop_ap_password`
- `car_presence_workshop_api_encryption_key`
- `car_presence_workshop_ota_password`