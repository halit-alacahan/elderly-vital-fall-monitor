<div align="center">

# 🫀 Leonardo V1.0 — Smart Wearable Fall Detection & Biometric Telemetry

**An ultra-low-power, RISC-V edge-computing IoT wearable for geriatric care, autonomous fall detection, and continuous vital signs monitoring.**

[![Hardware](https://img.shields.io/badge/Hardware-Altium%20Designer-blue.svg?style=for-the-badge&logo=altiumdesigner)](#)
[![MCU](https://img.shields.io/badge/MCU-ESP32--C6%20(RISC--V)-red.svg?style=for-the-badge&logo=espressif)](#)
[![Connectivity](https://img.shields.io/badge/Wireless-Wi--Fi%206%20%7C%20BLE%205.0%20%7C%20802.15.4-green.svg?style=for-the-badge)](#)
[![Sensors](https://img.shields.io/badge/Sensors-BMI270%20%7C%20MAX30102-orange.svg?style=for-the-badge)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

<br/>

<p align="center">
  <img src="top.jpg" alt="Leonardo V1.0 - Top View" width="48%" style="border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
  &nbsp;&nbsp;
  <img src="bottom.jpg" alt="Leonardo V1.0 - Bottom View" width="48%" style="border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
</p>

*Figure 1: Leonardo V1.0 3D PCB Renders (Top: ESP32-C6, USB-C, Power Management | Bottom: MAX30102 Optical Sensor, BMI270, Coin Vibration Motor, Level Shifter)*

</div>

---

## 📋 Table of Contents
- [Project Overview](#-project-overview)
- [System Architecture](#-system-architecture)
- [Key Hardware Features](#-key-hardware-features)
- [Hardware Subsystems Breakdown](#-hardware-subsystems-breakdown)
  - [Power Management & Dynamic Power Path](#1-power-management--dynamic-power-path)
  - [Inertial Motion & Fall Detection (BMI270)](#2-inertial-motion--fall-detection-bmi270)
  - [Photoplethysmography Biosensing (MAX30102)](#3-photoplethysmography-biosensing-max30102)
  - [Dual-Voltage I2C Bus Translation (PCA9306)](#4-dual-voltage-i2c-bus-translation-pca9306)
  - [Actuation & Multi-Modal Feedback](#5-actuation--multi-modal-feedback)
- [Pinout & Interconnect Mapping](#-pinout--interconnect-mapping)
- [I2C Bus Address Map](#-i2c-bus-address-map)
- [Fall Detection & State Machine Flow](#-fall-detection--state-machine-flow)
- [Firmware Architecture](#-firmware-architecture)
- [Repository Structure](#-repository-structure)
- [Getting Started](#-getting-started)
- [License](#-license)

---

## 🌟 Project Overview

**Leonardo V1.0** is an open-source, custom-engineered medical IoT wearable designed for real-time geriatric safety. Elderly falls represent a critical health risk often accompanied by immediate cardiovascular complications. 

This board integrates **high-frequency 6-DoF inertial measurement** with **photoplethysmography (PPG) optical pulse oximetry** in an ultra-compact wrist/chest-worn form factor.

### Primary Objectives:
* **Zero-Latency Fall Detection:** Low-power interrupt-driven inertial monitoring that differentiates daily living activities (ADL) from traumatic impact events.
* **Continuous Photoplethysmography (PPG):** High-precision SpO₂ (Blood Oxygen) and Heart Rate (BPM) measurement placed on the PCB's bottom side for optimal skin contact.
* **Autonomous Emergency Response:** Instantaneous dual-alert actuation (Haptic ERM + Acoustic Piezo Buzzer) with a hardware panic cancel switch.
* **Next-Gen Connectivity:** Powered by the **ESP32-C6** RISC-V SoC, supporting **Wi-Fi 6 (802.11ax)**, **Bluetooth 5 (LE)**, and **IEEE 802.15.4 (Zigbee/Thread)** for smart hospital and home-care telemetry.

---

## 🏗️ System Architecture
