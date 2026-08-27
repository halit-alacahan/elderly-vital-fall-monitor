<div align="center">

# 🫀 Leonardo V1.0 — Wearable Fall Detection & Biometric Telemetry System

**An ultra-low-power, RISC-V edge-computing IoT wearable for geriatric care, autonomous fall detection, and continuous vital signs monitoring.**

[![Hardware](https://img.shields.io/badge/Hardware-Altium%20Designer-blue.svg?style=for-the-badge&logo=altiumdesigner)](#)
[![MCU](https://img.shields.io/badge/MCU-ESP32--C6%20(RISC--V)-red.svg?style=for-the-badge&logo=espressif)](#)
[![Connectivity](https://img.shields.io/badge/Wireless-Wi--Fi%206%20%7C%20BLE%205.0%20%7C%20802.15.4-green.svg?style=for-the-badge)](#)
[![Sensors](https://img.shields.io/badge/Sensors-BMI270%20%7C%20MAX30102-orange.svg?style=for-the-badge)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

<br/>

| Top View (MCU, USB-C, Power Path) | Bottom View (Optical PPG, IMU, Actuators) |
| :---: | :---: |
| <img src="top.jpg" alt="Leonardo V1.0 Top View" width="400"/> | <img src="bottom.jpg" alt="Leonardo V1.0 Bottom View" width="400"/> |

</div>

---

## 📋 Table of Contents
- [Project Overview](#-project-overview)
- [System Architecture](#-system-architecture)
- [Key Hardware Features](#-key-hardware-features)
- [Hardware Subsystems Breakdown](#-hardware-subsystems-breakdown)
- [Pinout & Interconnect Mapping](#-pinout--interconnect-mapping)
- [I2C Address Map](#-i2c-address-map)
- [Fall Detection State Machine](#-fall-detection-state-machine)
- [Embedded Firmware Implementation](#-embedded-firmware-implementation)
- [Repository Structure](#-repository-structure)
- [Getting Started](#-getting-started)
- [License](#-license)

---

## 🌟 Project Overview

**Leonardo V1.0** is an open-source, custom-engineered medical IoT wearable designed for real-time geriatric safety. Elderly falls represent a critical health risk often accompanied by immediate cardiovascular complications. 

This board integrates **high-frequency 6-DoF inertial measurement** with **photoplethysmography (PPG) optical pulse oximetry** in an ultra-compact wrist/chest-worn form factor.

### Primary Objectives:
* **Zero-Latency Fall Detection:** Low-power interrupt-driven inertial monitoring that differentiates daily living activities (ADL) from traumatic impact events.
* **Continuous Photoplethysmography (PPG):** High-precision SpO₂ (Blood Oxygen) and Heart Rate (BPM) measurement placed on the PCB's bottom side for optimal dermal contact.
* **Autonomous Emergency Response:** Instantaneous dual-alert actuation (Haptic ERM + Acoustic Piezo Buzzer) with a hardware panic cancel switch.
* **Next-Gen Connectivity:** Powered by the **ESP32-C6** RISC-V SoC, supporting **Wi-Fi 6 (802.11ax)**, **Bluetooth 5 (LE)**, and **IEEE 802.15.4 (Zigbee/Thread)** for smart hospital and home-care telemetry.

---

## 🏗️ System Architecture

```text
+===================================================================================================+
|                                    LEONARDO V1.0 HARDWARE ARCHITECTURE                            |
+===================================================================================================+
|                                                                                                   |
|  [USB Type-C (5V)] ---> [ESD Clamp (ESDA6V1)]                                                     |
|           |                                                                                       |
|           +---------> [TI BQ24040 Charger (100mA)] <=======> [Li-Po Battery Connector (2-Pin)]    |
|           |                       |                                      |                        |
|           |                       +------------+                         |                        |
|           v                                    |                         v                        |
|   [V_BUS Detection]                     [Charge Status]        [PJA3441 P-MOSFET Switch]          |
|           |                                                              |                        |
|           +--------------------------------------------------------------+                        |
|                                          |                                                        |
|                                          v [V_SYS Rail]                                           |
|                         +---------------------------------+                                       |
|                         |                                 |                                       |
|                         v                                 v                                       |
|           [MIC5504-3.3YM5 LDO (3.3V)]        [TLV70018 LDO (1.8V)]                                |
|                         |                                 |                                       |
|                         +---------------+                 +---------------+                       |
|                                         |                                 |                       |
+-----------------------------------------|---------------------------------|-----------------------+
| 3.3V SYSTEM DOMAIN                      v                                 v 1.8V ANALOG DOMAIN    |
|                               +-------------------+             +-------------------+             |
|                               |   ESP32-C6 MCU    |             | MAX30102 Biosense |             |
|                               |   (RISC-V Core)   |             | (PPG SpO2 & HR)   |             |
|                               +-------------------+             +-------------------+             |
|                                 |   |   |   |   |                         |  |                    |
|       +-------------------------+   |   |   |   +------+                  |  |                    |
|       |                             |   |   |          |                  |  |                    |
|       v                             v   v   v          v                  v  v                    |
|  [BMI270 6-DoF IMU] <=================[PCA9306 Level Translator]==========+==+                    |
|  (Accel + Gyro @ 3.3V)                 (3.3V <---> 1.8V I2C Bus)                                  |
|                                                                                                   |
|  OUTPUT ACTUATION & IO:                                                                           |
|  - Coin ERM Vibration Motor (AO3400 N-MOSFET Driver)                                              |
|  - Micro Piezo Buzzer (Transistor Driven)                                                         |
|  - Emergency SOS / Alarm Cancel Switch (RC Debounced)                                             |
|  - 32.768 kHz External RTC Oscillator (Deep Sleep Wakeup)                                         |
|  - Native USB CDC / JTAG Interface (D+ / D-)                                                      |
+===================================================================================================+
```

---

## ⚡ Key Hardware Features

| Specification | Implementation | Benefits |
| :--- | :--- | :--- |
| **Core Processor** | **Espressif ESP32-C6-WROOM-1** | 160 MHz 32-bit RISC-V single-core, integrated Wi-Fi 6, BLE 5, Thread/Matter |
| **RTC Crystal** | **ECS-.327-6-16R-TR (32.768 kHz)** | Ultra-low power deep sleep timing and precise real-time timestamping |
| **Motion Tracking** | **Bosch Sensortec BMI270** | 16-bit 6-axis IMU, built-in FIFO, and programmable motion interrupts |
| **Vitals Sensor** | **Maxim Integrated MAX30102** | Integrated Red + IR LEDs and photodetector for non-invasive SpO₂ and HR |
| **Bus Translation** | **TI PCA9306DQER** | Bidirectional $3.3\text{V} \leftrightarrow 1.8\text{V}$ voltage-level translation without bus degradation |
| **Battery Charger** | **TI BQ24040DSQR** | Linear Li-Ion/Li-Po charger with thermal NTC regulation ($I_{\text{CHG}} = 100\text{ mA}$) |
| **Power Path** | **PJA3441 P-Channel MOSFET** | Seamless auto-switching between battery and USB power without brownouts |
| **Haptic Driver** | **AO3400 N-MOSFET** | High-efficiency drive stage with flyback protection for the ERM coin motor |
| **Acoustic Driver** | **PBSS Series BJT Switch** | High-output resonant driver for the onboard alarm buzzer |
| **USB Protection** | **ESDA6V1BC6 Array + Ferrite** | Full ESD clamp on differential data lines and RF noise suppression |

---

## 🔍 Hardware Subsystems Breakdown

### 1. Power Management & Dynamic Power Path
* **Charge Controller:** Utilizes the **TI BQ24040**, programmed with a $5.49\text{ k}\Omega$ resistor on `ISET` to provide an exact **100 mA** charge rate—ideal for small form-factor single-cell Li-Po batteries ($150\text{ mAh} - 300\text{ mAh}$). A $10\text{ k}\Omega$ NTC thermistor input (`TS`) prevents charging under extreme temperatures.
* **Ideal Diode Power Path:** A P-MOSFET (`PJA3441`) paired with a Schottky diode topology dynamically isolates the battery when 5V $V_{\text{BUS}}$ is connected via USB-C. This prevents charging cycle interruptions and directly powers $V_{\text{SYS}}$.
* **Dual Independent LDO Regulators:**
  * **$3.3\text{V}$ System Rail:** Powered by a low-dropout, high-PSRR `MIC5504-3.3YM5-TR` supplying the ESP32-C6, BMI270, actuators, and pull-up networks.
  * **$1.8\text{V}$ Biosensor Core Rail:** Dedicated ultra-low-noise `TLV70018DCKR` LDO feeding the analog front-end of the MAX30102 to eliminate digital power-rail switching noise from PPG readings.

### 2. Inertial Motion Tracking (BMI270)
* Ultra-low power 6-axis inertial measurement unit (IMU).
* Connected directly to the 3.3V native I2C bus (`BMI_SDA`, `BMI_SCK`) with dedicated hardware interrupts:
  * `INT1` (IO5): Watermark and Free-Fall / Significant Motion trigger.
  * `INT2` (IO4): High-G shock / posture orientation detection.

### 3. Photoplethysmography Biosensing (MAX30102)
* High-sensitivity optical sensor designed for pulse oximetry and heart rate tracking.
* Uses dual power supplies: $1.8\text{V}$ for internal logic/photodiode ADC and $3.3\text{V}$ for the optical LED drive current (`VLED`).
* Placed directly on the PCB bottom layer for direct dermal contact.
* `MAX_INT` (IO2) triggers an active-low interrupt on sample FIFO saturation.

### 4. Dual-Voltage I2C Translation (PCA9306)
* The system utilizes the **TI PCA9306** bidirectional voltage-level translator.
* Converts the $3.3\text{V}$ I2C lines (`BMI_SDA` / `BMI_SCK`) from the ESP32-C6 to the $1.8\text{V}$ logic levels (`MAX_SDA` / `MAX_SCK`) required by the MAX30102, isolated with $4.7\text{ k}\Omega$ pull-up resistors on both domains.

### 5. Feedback Actuation & Protection
* **Haptic Feedback:** Driven by a low $R_{\text{DS(on)}}$ **AO3400** N-MOSFET via `MOTOR_EN` (IO19) with a 100k gate pull-down and flyback diode protection.
* **Audible Alarm:** Driven via `BUZZER_EN` (IO21) to generate loud acoustic alert sweeps ($2.7\text{ kHz} - 4\text{ kHz}$) when a fall is detected.
* **Emergency Input:** Hardware tactile switch (`EVQ-P7A01P`) on `IO15` with an RC filter ($1\text{ k}\Omega + 100\text{ nF}$) for bounce-free user cancellation of false alarms or manual SOS triggers.

---

## 📌 Pinout & Interconnect Mapping

| ESP32-C6 Pin | Net Label | Peripheral Destination | Domain | Description / Function |
| :--- | :--- | :--- | :--- | :--- |
| **IO0** | `XTAL_IN` | ECS-.327-6-16R-TR | 3.3V | 32.768 kHz External Crystal Input |
| **IO1** | `XTAL_OUT` | ECS-.327-6-16R-TR | 3.3V | 32.768 kHz External Crystal Output |
| **IO2** | `MAX_INT` | MAX30102 Interrupt | 3.3V (Shifted) | Active-Low PPG Sample Ready Interrupt |
| **IO4** | `INT2` | BMI270 Interrupt 2 | 3.3V | High-G Shock / Step Interrupt |
| **IO5** | `INT1` | BMI270 Interrupt 1 | 3.3V | Free-fall / Motion Wakeup Interrupt |
| **IO6** | `BMI_SDA` | PCA9306 / BMI270 | 3.3V | Master I2C Data Line (4.7k Pull-up) |
| **IO7** | `BMI_SCK` | PCA9306 / BMI270 | 3.3V | Master I2C Clock Line (4.7k Pull-up) |
| **IO12** | `USB_D_N` | USB Type-C | 3.3V Diff | Native USB 2.0 Differential Data Minus |
| **IO13** | `USB_D_P` | USB Type-C | 3.3V Diff | Native USB 2.0 Differential Data Plus |
| **IO15** | `ALARM_SW` | EVQ-P7A01P Switch | 3.3V | Hardware Panic / SOS Input (Debounced) |
| **IO16** | `TXD0` | Test Point / Header | 3.3V | UART0 Serial Transmit (Debug Console) |
| **IO17** | `RXD0` | Test Point / Header | 3.3V | UART0 Serial Receive (Debug Console) |
| **IO19** | `MOTOR_EN` | AO3400 Gate | 3.3V | Haptic Coin Motor PWM / Drive Trigger |
| **IO21** | `BUZZER_EN`| Transistor Driver | 3.3V | Audible Alarm Piezo Drive Trigger |

---

## 🗺️ I2C Address Map

| Device | 7-bit Address | Bus Voltage | Functionality |
| :--- | :--- | :--- | :--- |
| **Bosch BMI270** | `0x68` | 3.3V (Primary) | 6-DoF Accelerometer & Gyroscope (SDO tied to GND) |
| **Maxim MAX30102**| `0x57` | 1.8V (Translated) | High-Resolution Pulse Oximetry & Heart Rate |

---

## 🔄 Fall Detection State Machine

```text
+-----------------------------------------------------------------------------+
|                           FALL DETECTION STATE MACHINE                      |
+-----------------------------------------------------------------------------+
|                                                                             |
|   +-------------------+                                                     |
|   |    STATE 0:       |  Low-Power Sleep / Step & Motion Tracking           |
|   |    MONITORING     |  (BMI270 running low-ODR inertial checks)           |
|   +-------------------+                                                     |
|             |                                                               |
|             | Free-Fall Acceleration Dip Detected (|a| < 0.5g for > 80ms)   |
|             v                                                               |
|   +-------------------+                                                     |
|   |    STATE 1:       |  High-rate sampling buffer (200 Hz)                 |
|   |   IMPACT CHECK    |                                                     |
|   +-------------------+                                                     |
|             |                                                               |
|             | High-G Impact Spike Recorded (|a| > 3.0g within 400ms)        |
|             v                                                               |
|   +-------------------+                                                     |
|   |    STATE 2:       |  Post-fall stillness analysis                       |
|   |   POST-IMPACT     |  (Monitoring angular velocity for 5 - 10 seconds)   |
|   +-------------------+                                                     |
|             |                                                               |
|             | Inactivity Confirmed + Abnormal Pulse Rate / Hypoxia          |
|             v                                                               |
|   +-------------------+                                                     |
|   |    STATE 3:       |  Haptic vibration + Warning buzzer activated        |
|   |  LOCAL WARNING    |  User given 20s window to press ALARM_SW            |
|   +-------------------+                                                     |
|             |                                                               |
|             | Timeout Expired (No User Cancellation via Button)             |
|             v                                                               |
|   +-------------------+                                                     |
|   |    STATE 4:       |  Telemetry Packet Dispatched via Wi-Fi 6 / BLE      |
|   | EMERGENCY DISPATCH|  (Includes Device ID, HR, SpO2 & Impact Vector)     |
|   +-------------------+                                                     |
|                                                                             |
+-----------------------------------------------------------------------------+
```
## 📂 Repository Structure

```text
├── hardware/
│   ├── schematics/           # Altium Designer schematics (.SchDoc, PDF export)
│   ├── pcb_layout/           # Multi-layer layout files (.PcbDoc)
│   ├── gerber/               # Production-ready RS-274X Gerber & Drill files
│   ├── bom/                  # Bill of Materials (BOM) with LCSC / Digikey PN
│   └── 3d_models/            # STEP / 3D models and enclosure files
├── firmware/                 # ESP-IDF C/C++ firmware source code
├── docs/                     # Datasheets, calculation sheets, and test data
├── top.jpg                   # PCB Top 3D Render
├── bottom.jpg                # PCB Bottom 3D Render
├── LICENSE                   # MIT License
└── README.md                 # Project Documentation
```

---

## 🚀 Getting Started

### Hardware Prerequisites
* 1x **Leonardo V1.0 PCB Assembly**
* 1x **3.7V Li-Po Battery** (JST-PH 2.0mm connector)
* 1x **USB Type-C Cable** (Data + Power)

### Firmware Compilation & Flashing

1. **Install ESP-IDF (v5.1 or later):**
   ```bash
   git clone --recursive [https://github.com/espressif/esp-idf.git](https://github.com/espressif/esp-idf.git)
   cd esp-idf
   ./install.sh
   . ./export.sh
   ```

2. **Clone this repository:**
   ```bash
   git clone [https://github.com/your-username/leonardo-v1-fall-detection.git](https://github.com/your-username/leonardo-v1-fall-detection.git)
   cd leonardo-v1-fall-detection/firmware
   ```

3. **Set target to ESP32-C6:**
   ```bash
   idf.py set-target esp32c6
   ```

4. **Configure Project (Wi-Fi, MQTT Endpoint, Sensor Thresholds):**
   ```bash
   idf.py menuconfig
   ```

5. **Build, Flash, and Monitor:**
   ```bash
   idf.py build
   idf.py -p /dev/ttyACM0 flash monitor
   ```

---

## 📜 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for complete details.

---

<div align="center">
  <sub>Engineered with precision for advanced geriatric telemetry and emergency response.</sub>
</div>
