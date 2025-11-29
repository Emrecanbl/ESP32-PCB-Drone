# ESP32-S3 Micro-Drone FC & ESP-NOW Controller  
### Brushed 1S Test Rig → 4″ BLDC Platform with Optical Flow / ToF

**Staged micro-drone platform based on ESP32-S3-WROOM-1 and a custom ESP-NOW handheld controller.**

Target use:  
1. **Phase 1:** Low-cost 1S brushed micro-drone with PCB-frame for PID tuning, basic IMU/baro fusion and ESP-NOW link validation.  
2. **Phase 2:** Full sensor suite (BMI088, PMW3901, VL53L1, BMP280, GPS) and porting the same flight stack to a 4″ brushless quad with a CM5 + ROS2 companion.

---

## ✨ High-Level Features

### Phase 1 – 1S Brushed Micro-Drone (PCB = Frame)

- **Flight MCU:** ESP32-S3-WROOM-1 (FreeRTOS)
- **Sensors (current / initial focus):**
  - MPU-6050 or Bosch **BMI088** (IMU – SPI/I²C)
  - **BMP280** barometer (I²C)
- **Propulsion:**  
  - 4× **8520 coreless brushed** motors  
  - Driven via 2× **DRV8833** (or similar dual H-bridge drivers)
- **Power:**
  - 1S LiPo (e.g. 3.7 V, 650–1000 mAh, 25C)  
  - Optional current/voltage telemetry (INA219 + VBAT ADC)
- **Radio / RC:**
  - **ESP-NOW** handheld transmitter (separate PCB, already fabricated)
  - Custom RC packet: roll, pitch, yaw, throttle + AUX, failsafe
- **Form factor:**
  - **PCB as frame**, motor pads in the corners  
  - Tello-class layout (75 mm props possible), target AUW ≈ 50–65 g

Goal of this phase:  
Implement + tune **PID control, basic IMU fusion, baro-assisted Z control and ESP-NOW RX/failsafe** on a cheap, safe lab platform before moving to BLDC.

---

### Phase 2 – Full Sensor Suite & 4″ BLDC Platform

Once the basic flight stack is stable:

- **IMU:** Bosch **BMI088** (low-noise gyro, drone-grade)
- **Barometer:** **BMP280**
- **Range / Flow:**  
  - **VL53L1** ToF (downward)  
  - **PMW3901** optical flow (SPI)
- **GPS:** u-blox **SAM-M10Q** (MAX-M10S family), 1PPS → MCU
- **BLDC Drive (planned):**
  - 4× 1404/1504 brushless motors  
  - 4-in-1 BLHeli ESC, **DShot** output from ESP32  
  - 2S / 2S2P 18650 Li-ion pack for longer hover
- **Companion (planned):**
  - **CM5** compute module running **ROS2**  
  - Visual odometry / SLAM + high-level autonomy  
  - MAVLink-like link between FC and companion

---

## 🧱 Hardware Overview

### Boards & Phases

| Board / Stack | Function | Phase | State |
|---|---|---|:--|
| **Handheld Controller** | ESP32-S3 + display + analog sticks + battery management, ESP-NOW TX | 1 | ✅ PCB fabricated / in bring-up |
| **Micro-Drone FC (PCB-Frame)** | ESP32-S3 + IMU + BMP280 + DRV88xx drivers + 1S power | 1 | 🛠 In design / bring-up |
| **Flow/ToF Board** | PMW3901 (SPI) + VL53L1 (I²C, 2.8 V LDO), downward aperture | 2 | 📝 Planned |
| **4″ BLDC FC + PDB Stack** | ESP32-S3/BMI088/BMP280/GPS, 4-in-1 ESC, power distribution | 2 | 📝 Planned |
| **CM5 Carrier (Companion)** | CM5 + 5 V supply + camera + UART to FC | 3 | 📝 Planned |

---

## 📸 Gallery (placeholder)

> NOTE: Images below refer to earlier BLDC stack concepts and will be updated as the brushed PCB-frame design stabilises.

<p align="center">
  <img src="PCB_ESP_3D_Model.png" width="40%">&nbsp;&nbsp;
  <img src="Photos/PCB_ESP_Schematic.png" width="40%">
  <img src="Photos/PCB_ESP_Tracing.png" width="40%"><br>
  <em> UP_FC schematic/placement (early concept) </em>
</p>

<p align="center">
  <img src="PCB_Bottom_3D_Model.png" width="40%">&nbsp;&nbsp;
  <img src="PCB_Bottom_Schematic.png" width="40%"><br>
  <img src="PCB_Bottom_Tracing.png" width="40%"><br>
  <em> UP_FC schematic/placement (early concept) </em>
</p>
<p align="center">
  <img src="PCB_Controller_3D_Model.png" width="40%">&nbsp;&nbsp;
  <img src="PCB_Controller_Schematic.png" width="40%"><br>
  <img src="PCB_Controller_Tracing.png" width="40%"><br>
   <em> FC 3D model (concept) </em>
</p>

<p align="center">
  <img src="Photos/PCB_Controller_Schematic.png" width="40%">&nbsp;&nbsp;
  <img src="Photos/PCB_Controller_Tracing.png" width="40%"><br>
  <em> CONTROLLER_PCB </em>
</p>

<p align="center">
  <img src="Photos/PCB_Controller_3D_Model.png" width="80%"><br>
   <em> CONTROLLER_PCB 3D model </em>
</p>

---

## 🧰 Core BOM (by phase)

### Phase 1 – 1S Brushed Micro-Drone

- **MCU:** ESP32-S3-WROOM-1
- **Sensors:**
  - IMU: MPU-6050 or Bosch BMI088 (final target)  
  - Baro: BMP280 (I²C)
- **Motor Drivers:**
  - 2× **DRV8833** (each drives 2× 8520 brushed motors)  
  - Optional: 0.1 µF near each motor, bulk cap on VBAT
- **Power / Telemetry:**
  - 1S LiPo: 3.7 V, 650–1000 mAh, ≈25C  
  - Optional: INA219 high-side shunt (0.01 Ω, 1–2 W, Kelvin) + VBAT divider → ADC
- **Regulator:**
  - 3V3 ≥ 1 A LDO or buck from VBAT
- **Protection / IO:**
  - USB-C for DFU/serial (2× 5.1 kΩ CC, ESD on D±)  
  - TVS on VBAT, input bulk caps

---

### Phase 2 – Flow / ToF / BLDC

- **Flow / Range:**
  - **PMW3901** optical flow module (SPI)  
  - **VL53L1** ToF + 2.8 V LDO (e.g. TLV70228 / AP7331-2.8), local decoupling
- **GPS:**
  - **SAM-M10Q** with integrated patch, 1PPS to MCU
- **BLDC (future 4″ platform):**
  - 4× 1404/1504 motors  
  - 4-in-1 ESC (BLHeli_S/32, DShot)  
  - 2S / 2S2P 18650 pack
- **Regulation:**
  - 5 V BEC for CM5  
  - 3V3 buck for FC (e.g. TPS63070 / MPM3610-3V3)

---

## 🧪 Firmware Targets (FreeRTOS)

Applies to both brushed micro-rig and later BLDC platform; only motor backend changes.

- `imu_task` **1 kHz**  
  - SPI/I²C reads, gyro/accel pre-filtering, DLPF config
- `attitude_task` **500–1000 Hz**  
  - Mahony/Madgwick or complementary filter
- `alt_hold_task` **50–100 Hz** (Phase 2)  
  - VL53L1 + baro fusion → Z PID
- `flow_task` **100–200 Hz** (Phase 2)  
  - PMW3901 → body-frame vx, vy (flow_rate × height)
- `pos_xy_task` **50–100 Hz** (Phase 2)  
  - XY PID, low-pass on flow (5–10 Hz)
- `mixer_task` **500 Hz**  
  - Brushed: PWM to DRV8833  
  - BLDC: DShot / PWM to ESC (future)
- `rx_task` **200–300 Hz**  
  - ESP-NOW RC, failsafe 50–100 ms → motor cut
- `power_task` **20–50 Hz**  
  - INA219 + VBAT → sag / cutoff logic

---

## 🚀 Getting Started (Phase 1 – 1S Brushed Rig)

### Hardware

1. Assemble the **ESP32-S3 Micro-FC PCB-frame**.
2. Solder **4× 8520 motors** to the corner pads.
3. Mount the **ESP32-S3**, IMU and BMP280; add DRV8833 motor drivers.
4. Connect a **1S LiPo** (e.g. 3.7 V, 1000 mAh) via VBAT pads.
5. Power up **without props** and verify:
   - USB serial output  
   - IMU + BMP280 readings  
   - Motor PWM / DRV8833 outputs (scope or LED)

### Bring-up Checklist

- Calibrate **IMU** (acc/gyro offsets) and **baro**.
- Validate **ESP-NOW RC** reception and failsafe timing.
- Implement basic **angle/rate PID** with motor test (no props / tethered).
- Attach props, enable **throttle limit**, perform short **tethered hover**.
- Iterate PID + filter tuning until stable hover is achieved.

---

## 🛠 Build & Layout Notes

- **EDA:** KiCad 7/8 (`/hardware`)
- **Fabrication:** 1.2–1.6 mm FR-4; for future BLDC FC consider 2 oz copper or 1 oz with ground pours + via stitching.
- **PCB-Frame:**  
  - Keep IMU close to CoG  
  - Battery centered on IMU on the opposite side if possible
- **Optics (Phase 2):**  
  - Flow + ToF modules must face downward  
  - Use **matte-black apertures** to reduce stray light
- **Cabling:**  
  - Keep motor power traces short and grouped  
  - Solid ground under IMU and digital traces  
  - Decouple each IC locally

---

## ✅ Test Plan (Summary)

**Phase 1 – Brushed Micro-Rig**

- **No-prop smoke test:**  
  - Check 3V3 ripple, VBAT behavior under small throttle  
  - Verify DRV8833 temperature at sustained low throttle
- **IMU sanity:**  
  - Stationary noise, bias drift, accel magnitude ≈ 1 g
- **RC link:**  
  - Latency, packet loss, failsafe trigger < 100 ms
- **Tethered step response:**  
  - ±5–10° pitch/roll commands  
  - Observe overshoot, settling, and coupling

**Phase 2 – Flow / ToF / BLDC**

- **Alt-hold:**  
  - ToF + baro fusion, hover at fixed height  
- **XY-hold:**  
  - Enable flow-based hold, verify drift reduction indoor
- **Thermals:**  
  - 60 s at 50 % throttle; log ESC / driver / regulator temps
- **EMI:**  
  - IMU spectrum under motor load  
  - Try PWM frequency adjustments and snubbers if needed

---

This repository tracks the evolution from a **safe, low-cost 1S brushed lab platform** to a **full 4″ brushless drone with optical flow, ToF, GPS and a ROS2 companion** reusing the same ESP32-based flight stack.
