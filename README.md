# 🚁 Micro-Drone Flight Stack (STM32H7)

Multi-stage development of a custom embedded flight controller.

> **Status:** In Progress

This repository documents the development of a custom flight controller and flight stack based on the **STM32H743**, combined with an **ESP32 handheld transmitter** for low-latency wireless control using **ESP-NOW**.

The goal is to build a **modular platform** that grows from a simple stabilized micro-drone to a fully equipped system with optical flow, ToF range sensing, GPS, and later a **ROS2/SLAM companion computer**.

---

## 🔧 1. Overview

The flight system is built around:

- A high-performance **STM32H743** microcontroller running **FreeRTOS**
- An **ESP32-S3 handheld transmitter** that provides all RC inputs via **ESP-NOW**

The project is structured to gradually cover:

- IMU filtering & rate/angle estimation  
- PID control of attitude & altitude  
- Sensor fusion (IMU + baro + ToF + optical flow)  
- Indoor position hold  
- Obstacle detection & local path planning  
- GPS-assisted outdoor flight  
- High-level autonomy via **ROS2 + CM5 companion computer**

Each stage introduces new sensing, control, and autonomy capabilities.

---

## 🚀 2. Current Development Stage — 3S Micro-Drone

**Platform:** 3-cell micro-drone with STM32H743-based flight controller.

### Hardware

- **Flight MCU:** STM32H743  
- **Motors:** 4× 1103 BLDC  
- **Frame:** Lightweight PCB-based frame  

### Sensors

- **Bosch BMI088** – low-noise gyro/accelerometer  
- **BMP280** – barometer  
- **VL53L1** – ToF range sensor  
- **PMW3901** – optical flow (vx/vy estimation)  

### Remote Control

- **ESP32 handheld transmitter**
- **ESP-NOW** (custom RC protocol)
- Features:
  - Arming
  - Throttle limit
  - Failsafe
  - Low-voltage scaling
  - Buzzer control

### Current focus areas

- Hardware bring-up of STM32H7 board + sensor modules  
- Real-time task structure under **FreeRTOS**  
- PID controllers (rate, angle, vertical control)  
- IMU pre-filtering and basic sensor fusion  
- ESP-NOW RC link reliability / failsafe  
- Early tethered hover tests and vibration analysis  

---
## Gallery

<p align="center">
  <img src="Photos/PCB_ESP_3D_Model.png" width="40%">&nbsp;&nbsp;
  <img src="Photos/PCB_ESP_Schematic.png" width="40%"><br>
  <img src="Photos/PCB_ESP_Tracing.png" width="40%"><br>
  <em> ESP32 </em>
</p>
<p align="center">
  <img src="Photos/PCB_Bottom_3D_Model.png" width="40%">&nbsp;&nbsp;
  <img src="Photos/PCB_Bottom_Schematic.png" width="40%"><br>
  <img src="Photos/PCB_Bottom_Tracing.png" width="40%"><br>
  <em> Bottom  </em>
</p>
<p align="center">
  <img src="Photos/PCB_Controller_3D_Model.png" width="40%">&nbsp;&nbsp;
  <img src="Photos/PCB_Controller_Schematic.png" width="40%"><br>
  <img src="Photos/PCB_Controller_Tracing.png" width="40%"><br>
  <em> Bottom  </em>
</p>
## 🧭 3. Roadmap (Next Stages)

### 🔸 GPS Integration (u-blox M10Q)

- UART + 1PPS sync  
- Outdoor hold / basic navigation  
- Integration into global state estimator  

### 🔸 Companion Computer (CM5 + ROS2)

- High-level navigation  
- SLAM / Visual Odometry  
- Object detection / landing markers  
- MAVLink-style communication between FC and companion  

### 🔸 Obstacle Detection & Local Planning

Before adding the CM5, a local perception module will be developed:

- 8–10× **VL53L7** ToF sensors in a 360° ring  
- Local 3D bubble mapping  
- Simple reactive path planning  
- Direction-dependent speed limiting  

### 🔸 Early Vision Experiments (ESP32-CAM)

- Feature tracking  
- Visual flow  
- Basic onboard image processing without CM5  

---

## 🏗️ 4. Technology Stack

### Languages

- C  
- C++  

### Microcontrollers / Hardware

- **STM32H743**  
- **ESP32-S3** (transmitter + early vision tests)  
- **1103 BLDC motors / 3S battery**  
- **4-in-1 ESC** (DShot)  

### RTOS / Frameworks

- **FreeRTOS**  
- **STM32 HAL / LL**  
- Custom motor mixer / DShot backend  

### Sensors

- **BMI088** (SPI)  
- **PMW3901** (SPI)  
- **VL53L1** (I²C)  
- **BMP280** (I²C)  
- **u-blox M10Q** (UART + PPS)  

### Algorithms

- PID (rate / angle / altitude)  
- Complementary / Madgwick filter  
- Multi-sensor fusion  
- Failsafe handling  
- Flow-based XY velocity estimation  
- Altitude estimation (baro + ToF)  

---

## 📦 5. Project Status

| Module                      | Status        |
|-----------------------------|--------------|
| STM32H7 flight controller PCB | 🟡 In progress |
| ESP32 handheld transmitter  | 🟢 Working (Rev1 tested) |
| IMU + barometer integration | 🟢 Working    |
| ToF + optical flow          | 🟡 Partial    |
| PID controllers             | 🟡 Tuning     |
| GPS integration             | 🔜 Planned    |
| CM5 companion               | 🔜 Planned    |
| ROS2 / SLAM                 | 🔜 Planned    |

---

## 📚 6. Project Purpose

This system is designed as:

- A realistic **embedded control & sensor-fusion platform**  
- A testbed for **autonomy on small aerial systems**  
- A hands-on environment for **motion control, perception, and state estimation**  
- A modular base that grows from a simple micro-drone to a **ROS2-capable autonomous platform**

The long-term objective is a **fully custom, research-grade flight stack** capable of autonomy on compact hardware.

---

## 📎 7. License

License will be added once the codebase is published.
