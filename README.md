# Toilet-Odor-Detector-and-Alarm-system-using-MQ137-sensor-with-EEPROM-calibration-and-web-server
This project is an **odor detection and alarm system** for toilets, utilizing an **MQ137 gas sensor** to monitor ammonia (NH₃) levels. The system uses **EEPROM for calibration storage**, **serial communication** for user inputs, and a **web server** to monitor the odor status remotely.

### Features
- **MQ137 Sensor**: Detects ammonia (NH₃) gas for toilet odor.
- **EEPROM Calibration**: Stores the calibration value of the sensor to persist across reboots.
- **Web Server**: Allows remote monitoring of the current gas levels and odor status.
- **Alarm System**: Activates a buzzer and LED when a threshold value is exceeded.
- **Serial Commands**: Allows recalibration via serial interface.

## Hardware Components
- **ESP8266** (NodeMCU/WeMos) Wi-Fi module
- **MQ137**: Ammonia gas sensor
- **Buzzer**: Alarm sound
- **LED**: Visual alarm indicator
- **Jumper wires**
- **Breadboard (optional)**

### Wiring Diagram
+---------------------+ | ESP8266/NodeMCU | +---------------------+ | Pin D1 -> MQ137 A0 | | Pin D2 -> Buzzer D5 | | Pin D3 -> LED D6 | +---------------------+

> You can use the following circuit diagram as a guide. Ensure the MQ137 sensor is connected properly to the **analog input** and the buzzer and LED are connected to **digital outputs**.

### Code Description
The system consists of:
1. **Wi-Fi Connection**: Connects to a Wi-Fi network using your SSID and password.
2. **Sensor Calibration**: Reads the MQ137 sensor value and calibrates it by reading the "clean air" value stored in EEPROM or recalibrates it on first use.
3. **Threshold Check**: The system monitors the gas levels and triggers an alarm (buzzer and LED) if the gas concentration exceeds a predefined threshold.
4. **Web Server**: Displays the current odor status (gas concentration) on a webpage.
5. **Serial Command Interface**: Allows recalibration via serial commands.

### Calibration
- The system automatically **calibrates the sensor** on the first startup or if no valid calibration value is stored in EEPROM.
- You can also manually trigger recalibration using the command: recal
- from the serial monitor.

### Web Server Endpoint
Once the system connects to Wi-Fi, it hosts a local web server on the ESP8266. To monitor the odor status, open a browser and visit the following IP:
http://<ESP8266_IP_ADDRESS>

This page will display:
- **Current Gas Concentration (ppm)**
- **Odor Status (Threshold Exceeded or Safe)**

### Serial Commands
1. **Recalibrate the sensor**: Type `recal` in the Serial Monitor to trigger a recalibration process and store the new value in EEPROM.

2. **View stored calibration**: The current calibration value is printed when the system starts.

### Installation

1. **Install Arduino IDE**: Ensure you have the latest Arduino IDE installed and configure it for ESP8266 boards.
2. **Install Required Libraries**:
   - **ESP8266WiFi**: For Wi-Fi connectivity.
   - **ESP_Mail_Client**: For sending email alerts (if applicable).
   - **ESP8266WebServer**: For hosting the web server.

3. **Upload the Code**:
   - Select the correct board and port in the **Tools** menu (NodeMCU or similar ESP8266 board).
   - Upload the provided code to the ESP8266.

### Code Snippet
```cpp
#include <ESP8266WiFi.h>
#include <ESP_Mail_Client.h>
#include <ESP8266WebServer.h>
#include <EEPROM.h>

// Wi-Fi Credentials
#define WIFI_SSID "YOUR_SSID"
#define WIFI_PASSWORD "YOUR_PASSWORD"

// Email Configuration (Optional)
#define SMTP_HOST "smtp.yourprovider.com"
#define SMTP_PORT 465
#define AUTHOR_EMAIL "your_email@example.com"
#define AUTHOR_PASSWORD "your_email_password"
#define RECIPIENT_EMAIL "recipient@example.com"

// MQ137 Pin
#define MQ137_PIN A0

// EEPROM Address
#define EEPROM_ADDR 0

// Thresholds
#define GAS_THRESHOLD 100  // Adjust for your sensor

// Calibration Variables
float RO_CLEAN_AIR = 10.0;  // Default calibration value

// Web Server
ESP8266WebServer server(80);

void setup() {
  Serial.begin(115200);
  EEPROM.begin(512);

  // Wi-Fi connection
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("WiFi connected");

  // Retrieve stored calibration value from EEPROM
  EEPROM.get(EEPROM_ADDR, RO_CLEAN_AIR);

  if (isnan(RO_CLEAN_AIR) || RO_CLEAN_AIR < 1.0 || RO_CLEAN_AIR > 1000.0) {
    RO_CLEAN_AIR = calibrateMQ137();
    EEPROM.put(EEPROM_ADDR, RO_CLEAN_AIR);
    EEPROM.commit();
    Serial.println("Calibration value stored in EEPROM");
  }

  // Set up the web server
  server.on("/", HTTP_GET, handleRoot);
  server.begin();
  Serial.println("Web server started");
}

void loop() {
  server.handleClient();
}

float calibrateMQ137() {
  float sensorValue = 0;
  for (int i = 0; i < 50; i++) {
    sensorValue += analogRead(MQ137_PIN);
    delay(100);
  }
  sensorValue = sensorValue / 50.0;
  return sensorValue;
}

void handleRoot() {
  float gasConcentration = readGasConcentration();
  String message = "Current Gas Concentration: " + String(gasConcentration) + " ppm";
  message += "\nThreshold: " + String(GAS_THRESHOLD) + " ppm";
  message += "\nStatus: " + (gasConcentration > GAS_THRESHOLD ? "Odor Detected" : "Safe");

  server.send(200, "text/plain", message);
}

float readGasConcentration() {
  int rawValue = analogRead(MQ137_PIN);
  float sensorVoltage = rawValue * (5.0 / 1023.0);
  float RS_GAS = ((5.0 * 1.0) / sensorVoltage) - 1.0;
  float ratio = RS_GAS / RO_CLEAN_AIR;
  return 1000 * pow(ratio, -2.95);
}
Troubleshooting
NaN in Gas Reading: If the sensor reading shows NaN, check the wiring and ensure the sensor is correctly connected and powered.

Wi-Fi Connection Issues: Double-check your Wi-Fi credentials and ensure the ESP8266 is in range.

License
This project is licensed under the MIT License - see the LICENSE file for details.
