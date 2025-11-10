 ESP32-S3 Mini FC And Controller With ESP-NOW — BLDC + Optical Flow/ToF

**Low-cost indoor hover / position-hold platform based on ESP32-S3-WROOM-1.**

Target use: experimentation with brushed propulsion + optical flow/ToF stabilization.

---

## ✨ Key Features

- **Flight MCU:** ESP32-S3-WROOM-1 (FreeRTOS)
- **IMU:** MPU-9250 (I²C)
- **Barometer:** BMP280 (I²C)
- **Range / Flow :** VL53L0X/L1X (I²C, 2.8 V) + PMW3901 (SPI module)
- **Power Telemetry:** INA219 high-side current/voltage + VBAT ADC
- **Brushed Drive:** 4× ESC → 4× BLDC motor 
- **Connectivity / UI:** ESP-NOW RX, USB‑C DFU
- **Form factor:** 3‑board stack — Top = PDB, Middle = FC, Bottom = Flow/ToF

---

## 🧱 Hardware Overview

### Current Board Status

| Board | Function | Rev | State |
|---|---|---:|:--|
  **Controller** | ESP32‑S3 + ST773 + Analog Sticks + Battery Management | v1.0 | ✅ Raeady to Fabrication |
| **FC (UP)** | ESP32‑S3 + MPU‑9250 + BMP280 + SAM-M10Q + USB‑C | v1.0 | ✅ Raeady to Fabrication |
| **PDB (Bottom)** | VBAT in, TVS + bulk caps, INA219 (0.01 Ω shunt, Kelvin), star power distribution | v0.x | ✅ Raeady to Fabrication|
| **Flow/ToF (Bottom)** | PMW3901 (SPI) + VL53L0X/L1X (I²C, 2.8 V LDO), downward apertures | v0.x | ✅ Raeady to Fabrication |

## Gallery

<p align="center">
  <img src="Photos/PCB_ESP_Schematic.png" width="40%">&nbsp;&nbsp;
  <img src="Photos/PCB_ESP_Tracing.png" width="40%"><br>
  <em> UP_PCB </em>
</p>

<p align="center">
  <img src="Photos/PCB_ESP_3D_Model.png" width="80%"><br>
   <em> TOP_PCB </em>
</p>

<p align="center">
  <img src="Photos/PCB_Bottom_Schematic.png" width="40%">&nbsp;&nbsp;
  <img src="Photos/PCB_Bottom_Tracing.png" width="40%"><br>
   <em> BASE_PCB </em>
</p>

<p align="center">
  <img src="Photos/PCB_Bottom_3D_Model.png" width="80%"><br>
  <em> BASE_PCB </em>
</p>

<p align="center">
  <img src="Photos/PCB_Controller_Schematic.png" width="40%">&nbsp;&nbsp;
  <img src="Photos/PCB_Controller_Tracing.png" width="40%"><br>
  <em> CONTROLLER_PCB </em>
</p>

<p align="center">
  <img src="Photos/PCB_Controller_3D_Model.png" width="80%"><br>
   <em> CONTROLLER_PCB </em>
</p>

### Mechanical

- **Mounting:** 20×20 mm (M2); target outline ≈ 26×26 mm per board.
- **Stack:** Up = **FC** → 5–6 mm → Bottom = **Flow/ToF**.
- **Optics:** optical modules face downward through **matte‑black apertures** to reduce stray light.

## 🧰 Core BOM

*(FC completed • others planned)*

- **MCU:** ESP32‑S3‑WROOM‑1
- **IMU / Baro:** MPU‑9250 (SPI) + BMP280 (I²C)
- **Brushed Drivers:** 2× DRV8833, **0.1 µF** at each motor; **snubber pads (100 Ω + 10 nF)** per phase (optional)
- **Telemetry:** INA219 + **0.01 Ω 2512 (1–2 W)** high‑side shunt (Kelvin) · VBAT divider
- **Regulator:** 3V3 ≥ 1 A — 1S supported via buck‑boost (**TPS63070** / **MPM3610‑3V3**)
- **Protection / IO:** USB‑C (2×5.1 kΩ CC, D± ESD), **TVS on VBAT**, bulk caps on PDB
- **Flow / Range (planned):** PMW3901 module (SPI) · VL53L0X/L1X + **2.8 V LDO** (TLV70228 / AP7331‑2.8)
- **GPS: u-blox MAX-M10S:**  SAM-M10Q (integrated patch); 1PPS→MCU

---

## 🧪 Firmware Targets (FreeRTOS)

- `imu_task` **1 kHz** (SPI DMA, DLPF)
- `attitude_task` **500–1000 Hz** (Madgwick/Mahony)
- `alt_hold_task` **100 Hz** (VL53 → Z PID)
- `flow_task` **100–200 Hz** (PMW3901 → vx, vy ≈ flow_rate × height)
- `pos_xy_task` **50–100 Hz** (XY PID, 5–10 Hz LPF on flow)
- `mixer_task` **500 Hz** (LEDC PWM)
- `rx_task` **200–300 Hz** (ESP‑NOW), failsafe **50–100 ms** → motor cut
- `power_task` **50–100 Hz** (INA219 + VBAT)

## 🚀 Getting Started

### Hardware

1. Assemble the **FC v1** board.
2. Connect **4× 8520** motors to **DRV8833** outputs.
3. Wire **VBAT** through the **PDB** (or use a bench supply).

### Bring-up Checklist

- Power‑up **without props**; verify USB serial, IMU/BMP/INA219 I²C, and PWM outputs.
- Calibrate **IMU & baro**; confirm **VBAT** reading.
- Attach props, set **throttle limit**, perform **short tethered hover** test.
- Position‑hold (when bottom board is ready): enable **ToF altitude‑hold**, then add **flow‑based XY‑hold**.

---

## 🛠 Build & Layout Notes

- **EDA:** KiCad 7/8 (`/hardware`)
- **Fabrication:** 1.6 mm FR‑4; prefer **2 oz copper** or **1 oz** with top/bottom pours + dense via‑stitch
- **Optics (bottom):** matte‑black apertures; clear downward line‑of‑sight
- **Cabling:** twist motor power pairs; keep signal lines short over solid ground
- **Calibration:** mag hard/soft‑iron (figure‑8), ToF offset, flow scale vs height

---

## ✅ Test Plan (summary)

- **No‑prop smoke:** 3V3 ripple, PWM edges (pre/post series‑R), TVS/bulk behavior
- **Tethered step:** ±10° command, capture overshoot/settling
- **Alt‑hold (ToF)** → enable **XY‑hold (Flow)**
- **Thermals:** 60 s @ 50 % throttle; log DRV8833/bulk temps
- **EMI:** IMU spectrum; try PWM **16↔24 kHz**, adjust snubbers if needed

---
