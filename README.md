 ESP32-S3 Mini FC And Controller With ESP-NOW — Brushed + Optical Flow/ToF

**Low-cost indoor hover / position-hold platform based on ESP32-S3-WROOM-1.**

Target use: experimentation with brushed propulsion + optical flow/ToF stabilization.

---

<p align="center">
  <img src="docs/img/fc_v1_top.png" alt="FC v1 top" width="520"/>
</p>

<p align="center">
  <b>Status:</b> Controller (FC) PCB v1 completed • PDB & Flow/ToF boards in design
</p>

---

## ✨ Key Features

- **Flight MCU:** ESP32-S3-WROOM-1 (FreeRTOS / ESP-IDF)
- **IMU:** MPU-9250 (I²C)
- **Barometer:** BMP280 (I²C)
- **Range / Flow (planned):** VL53L0X/L1X (I²C, 2.8 V) + PMW3901 (SPI module)
- **Power Telemetry:** INA219 high-side current/voltage + VBAT ADC
- **Brushed Drive:** 2× DRV8833 → 4× 8520 coreless motors (LEDC 16–24 kHz)
- **Connectivity / UI:** ESP-NOW RX, USB‑C DFU, low‑voltage buzzer (failsafe)
- **Form factor:** 3‑board stack — Top = PDB, Middle = FC, Bottom = Flow/ToF
- **Props (brushed):** 55 mm bi‑blade (1.0 mm hub) recommended. 65 mm is heavy for 8520; use with throttle limiting + thermal checks.

---

## 🧱 Hardware Overview

### Current Board Status

| Board | Function | Rev | State |
|---|---|---:|:--|
  **FC (Controller)** | ESP32‑S3 + ST773 + Analog Sticks + Battery Management | v1.0 | ✅ Raeady to Fabrication |
| **FC (Controller)** | ESP32‑S3 + MPU‑9250 + BMP280 + 2×DRV8833 + USB‑C + telemetry headers | v1.0 | 🛠️ In design  |
| **PDB (Top)** | VBAT in, TVS + bulk caps, INA219 (0.01 Ω shunt, Kelvin), star power distribution | v0.x | 🛠️ In design |
| **Flow/ToF (Bottom)** | PMW3901 (SPI) + VL53L0X/L1X (I²C, 2.8 V LDO), downward apertures | v0.x | 🛠️ In design |

### Electrical & Grounding

- **Split grounds:** `GND_PWR` (PDB/drivers) and `GND_LOG` (MCU/sensors) joined at a single‑point **0 Ω bridge** near the regulator.
- **IMU "quiet island":** continuous ground below IMU; `3V3_IMU` fed through ferrite with local **10 µF + 1 µF + 0.1 µF** decoupling.
- **PDB entry:** SMBJ14A TVS and **680–1000 µF** low‑ESR bulk at battery entry; star branches to ESCs/drivers.

### Mechanical

- **Mounting:** 20×20 mm (M2); target outline ≈ 26×26 mm per board.
- **Stack:** Top = **PDB** → 6–8 mm → Middle = **FC** → 5–6 mm → Bottom = **Flow/ToF**.
- **Optics:** optical modules face downward through **matte‑black apertures** to reduce stray light.

---

## 📌 FC v1 Pinout (as built)

**SPI (shared bus)**

```
SCK  = GPIO36
MOSI = GPIO35
MISO = GPIO37
CS(IMU) = GPIO34
INT(IMU)= GPIO33
```

**I²C (INA219 + BMP280 + future VL53)**

```
SCL = GPIO9
SDA = GPIO8
Pull‑ups: 4.7 kΩ; for VL53, pull to 2.8 V — ESP32 reads HIGH.
```

**Brushed PWM (LEDC, 16–24 kHz, 10–11‑bit)**

```
M1 = GPIO3
M2 = GPIO4
M3 = GPIO5
M4 = GPIO6
DRV8833 single‑ended: IN1 = PWM, IN2 = GND; OUT1/OUT2 → motor.
```

**Misc**

```
VBAT_ADC = GPIO1 (divider 1.00 MΩ / 470 kΩ + 0.1 µF)
BUZZER   = GPIO7 (NPN)
UART TX  = GPIO43
UART RX  = GPIO44
BOOT     = GPIO0
EN       = module EN
```

> Headers for future Flow (PMW3901) and ToF (VL53) are broken out on the FC. Final mapping for Flow/ToF is set on the bottom board; FC headers are pinned to match.

---

## 🧰 Core BOM

*(FC completed • others planned)*

- **MCU:** ESP32‑S3‑WROOM‑1
- **IMU / Baro:** MPU‑9250 (SPI) + BMP280 (I²C)
- **Brushed Drivers:** 2× DRV8833, **0.1 µF** at each motor; **snubber pads (100 Ω + 10 nF)** per phase (optional)
- **Telemetry:** INA219 + **0.01 Ω 2512 (1–2 W)** high‑side shunt (Kelvin) · VBAT divider
- **Regulator:** 3V3 ≥ 1 A — 1S supported via buck‑boost (**TPS63070** / **MPM3610‑3V3**)
- **Protection / IO:** USB‑C (2×5.1 kΩ CC, D± ESD), **TVS on VBAT**, bulk caps on PDB
- **Flow / Range (planned):** PMW3901 module (SPI) · VL53L0X/L1X + **2.8 V LDO** (TLV70228 / AP7331‑2.8)

> **Estimated full stack cost (brushed + flow + ToF):** **€45–60**

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
