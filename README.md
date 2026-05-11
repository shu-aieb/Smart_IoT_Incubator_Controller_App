# Smart Incubator App - IoT Control System

An Android based application designed to remotely monitor, manage and automate industrial poultry incubators. 

This project is a connection between complex embedded hardware (ESP8266) and end users, with a strong emphasis on fault-tolerant communication in an environment of low connectivity in rural areas.

## 📱 Interface Showcase
![UI Showcase](https://github.com/user-attachments/assets/6e1ac113-4508-4673-9297-9b520f54dedb)

## 🎯 The Problem & Purpose
Control of environmental parameters is essential for industrial incubators. Thousands of eggs can be killed by a 2-degree drop in temperature. The farmers had to have a remote monitoring solution for their hardware. But in the rural farm area, there are many times when there is no internet connectivity, so regular IoT solutions are not particularly useful. 

**The Solution:**
Designed a three-phase communication structure. The default option is to sync with a cloud by the Firebase option. In the case of an unreachable cloud, users on-site can directly connect to the Local Access Point of the hardware. When the user is not at site and there is no Internet connection, the app switches to a custom two-way GSM/SMS command protocol, which keeps the hardware within reach.

## 🏗️ Architecture & Tech Stack

This is a native Android application built in Java and XML.It's a native Android (Java, XML) solution.
### Backend / Bridge
This module is compatible with the ESP8266 Microcontroller.This module can be used with the ESP8266 Microcontroller.
- HTTPS (REST): Communicates with the web via an HTTPS connection.
Shared Preferences/SQLite (Local caching of batch parameters):

### Standard Android Directory Structure
```text
app/src/main/java/com/iramelectronics/incubator/
├── core/                 # App-wide utilities, Base Activities
├── network/              # Firebase listeners, WiFi socket clients, SMS broadcast receivers
├── models/               # Data models for Incubators, Batches, and Sensor Telemetry
├── services/             # Background services for alarm triggers
└── ui/
    ├── dashboard/        # Live telemetry parsing and gauge rendering
    ├── batch_manager/    # Date/Time calculation logic for poultry species
    ├── settings/         # PIN management, Router configuration
    └── warranty/         # Hardware sync verification
```
## ⚡ Key Engineering Decisions & Features

### 1. Triple-Redundancy Communication

The app routes commands dynamically depending on network availability, to address the rural connectivity issue. The most complex implementation was the Remote USSD Execution. Farmers had to make sure the carrier balance was in the device as the SMS fallback feature is hardware dependent. I created my own protocol that I can send a ussd code to the hardware and the hardware dials the code and processes the response to get the balance and passes it back to the app UI.

### 2. Autonomous Batch Computation Engine

The App makes users' life easier by giving them the option of having multiple batches running concurrently within an incubator rather than having to use manual calendars. The app chooses a particular type of poultry bird, and it sets the exact temperature, humidity, and durations for the motors to turn on and off. It maintains a background track of the days, and lights local device alarms as the candling/hatching stage starts.

### 3. These are UI Updates or Alarm States that are sent asynchronously.

The dashboard listens to continuous streams of sensor data. Updates to the sensors (Temperature, Humidity, Motor Countdown) are removed from "heavy" UI redraws to prevent UI freezing. When such limits are exceeded, the app activates high priority audio alarms (if device is not silent) and activates visual states of blinking.

### 4. Anti-Fraud Smart Warranty

This warranty logic is embedded in the hardware, eliminating the need for a paper warranty card. A second time stamp is permanently fixed to the microcontroller when the device is first booted up physically. The app extracts this hash and can determine how many years it has left, depending on the hardware model, and show a tamperproof digital warranty card.

### 5. Multi-Tier Security & Localization

The app distinguishes between Owner controls and User controls. Owners can set up to 6 different PINs for farm workers, which will limit access to critical temperature limits and enable owners to monitor dashboard statistics.

* **Accessibility:** Full bilingual support (English and Bengali) with toggling dynamically at runtime (without restarting the Activity). The hardware troubleshooting guides and connection diagrams are dynamically pulled from the app depending on the hardware model that is registered to the application.

## 🐛 Edge Cases Handled
* **Standard SMS Payloads:** Standardised SMS payloads to ensure commands fit into 160 characters to minimise packet loss and carrier charges.

* **Enhanced connection timeouts:** For the Local Wi-fi mode, so that the app does not hang if the walking distance exceeds the range of the hardware.

* **Timer Desync:** No longer caused by the app's countdown timer drifting from the hardware's physical motor relays by using the hardware as the single source of truth and polling for changes instead of trying to calculate.
