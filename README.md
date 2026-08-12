# 📡 RodosWX3 – Bresser Weather Sensor to APRS Gateway

**RodosWX3** is advanced, versatile software for weather stations, integrating external Bresser radio sensors (5-in-1, 6-in-1, 7-in-1) with the amateur APRS network. The project was created with maximum reliability, wireless configuration (Plug & Play), and user convenience in mind.

The program is multi-platform – it automatically adapts to microcontrollers from the **ESP8266** and **ESP32** families (including integrated boards such as Heltec V2/V3, TTGO LoRa32, and LilyGo).

---

## ✨ Main Features

*   **Asynchronous Web Panel:** A clear, live-refreshing dashboard running in the background without blocking radio reception.
*   **Wireless Configuration:** All parameters (WiFi Network, APRS, Location, Operating Mode) are changed via a web browser. Settings are saved in the non-volatile **LittleFS** memory.
*   **Two APRS Transmission Modes:**
    *   **APRS-IS:** Classic TCP connection to the worldwide network (e.g., `rotate.aprs2.net`).
    *   **KISS over TCP:** Sending AX.25 standard frames to local TNC servers (e.g., nodes of the free iGate/Share-TNC network).
*   **Station ID Filtering:** Protects against random data from neighboring stations. The system listens only to the assigned transmitter.
*   **Hardware Watchdog:** Automatic microcontroller restart with a rescue message (ALARM) sent to the APRS network if the external station goes silent for more than 5 minutes.
*   **Advanced Rain Buffer:** Automatic calculation of the rainfall sum from the last hour (`r`) and 24 hours (`p`).
*   **Fallback Mode (Access Point):** In case of no WiFi coverage, the station creates its own `RodosWX3_Setup` network for initial configuration.
*   **OLED Carousel:** Support for SSD1306 (128x64) screens. Automatic switching every 4 seconds between 4 screens (Status, Weather, Wind, Rain/Sun).

---

## 🛠 Hardware Requirements

1.  **Microcontroller:** Any ESP8266 (e.g., Wemos D1 Mini) or ESP32 board (recommended integrated boards like Heltec WiFi LoRa 32 V3 or TTGO).
2.  **Radio Receiver:** 868 MHz module – CC1101, SX1276, or SX1262 (supported natively by the library).
3.  **Pressure and Temperature Sensor:** BME280 or BMP280 (I2C bus).
4.  **Display:** OLED SSD1306 128x64 (often built into development boards).

---

## 🔌 Wiring and Connections

### 1. Radio Module (LoRa / CC1101)
If you are using a dedicated board with an integrated radio (e.g., Heltec, TTGO), **you do not need to solder any additional wires or change pins in the code**. Simply select your board in the Arduino IDE, and the software will automatically map the appropriate pins (SPI, CS, IRQ, RST).
If you are using an external CC1101 module, connect it to the hardware SPI bus of the microcontroller.

### 2. BME280 / BMP280 Sensor (I2C)
The sensor must be operated in **I2C** mode. If you have a 6-pin module, make the following bridges:

| BME280 Pin | ESP Connection | Description / Action |
| :--- | :--- | :--- |
| **VCC** | 3.3V | Power supply |
| **GND** | GND | Ground |
| **SCL / SCK** | SCL Pin | I2C Clock |
| **SDA / SDI** | SDA Pin | I2C Data line |
| **CSB** | 3.3V (VCC) | Bridge with VCC – forces I2C mode |
| **SDO** | GND | Bridge with Ground – sets I2C address to `0x76` |

*   *ESP8266 Tip:* The program uses pins **D1 (SCL)** and **D2 (SDA)** by default.
*   *ESP32 Tip:* The program will automatically assign I2C pins according to your board manufacturer's schematic. Just connect to the hardware SCL/SDA.

---

## 💻 Installation and Libraries

To correctly compile the code in the Arduino IDE, download and install the following from the Library Manager:

*   `BresserWeatherSensorReceiver` (radio decoder)
*   `Adafruit BME280 Library` and `Adafruit Unified Sensor` (BME/BMP280 support)
*   `Adafruit GFX` and `Adafruit SSD1306` (OLED screen support)
*   `ESPAsyncWebServer` (asynchronous web server)
*   `AsyncTCP` (for ESP32) or `ESPAsyncTCP` (for ESP8266)

Before uploading the code, make sure you have selected the correct device variant from the *Tools -> Board* menu.

---

## 🚀 First Run (Configuration)

Since the device does not know your WiFi password right after uploading the software, it will enter configuration mode (Access Point).

1.  Power on the microcontroller.
2.  On your phone or computer, find the WiFi network named **`RodosWX3_Setup`** and connect to it.
3.  Open your browser and go to the address: `http://192.168.4.1`
4.  In the Web Panel, fill in the Main Station Configuration (router SSID and password, CallSign, APRS mode, geographical coordinates).
5.  Click **Save configuration and restart**.
6.  The device will restart, connect to your home network, and its assigned IP address will be displayed on the OLED screen.

---

## 🔒 ID Filtering (Station Pairing)

Bresser stations always generate a new, random ID number after inserting new batteries. For the base station to ignore interference and data from the neighborhood:

1.  Log in to the base station's Web Panel.
2.  Insert batteries into the external unit on the roof.
3.  In the station panel, drop down the "Detected nearby" list. The system will display the collected hardware identifiers.
4.  Select the new ID and save the setting. From this moment on, the hardware is strictly assigned and protected by the logic of the built-in watchdog.
