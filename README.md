# MagLev LQR: Electromagnetic Suspension Platform

![Control: LQR](https://img.shields.io/badge/Control Theory-LQR-red) ![Hardware: KiCad](https://img.shields.io/badge/EDA-KiCad-blue) ![Firmware: FreeRTOS](https://img.shields.io/badge/MCU-ESP32-orange)

An inherently unstable electromagnetic levitation system designed to suspend a lightweight object (a desktop mini-satellite model) in mid-air. This project focuses heavily on state-space control theory, custom PCB layout, and power electronics, utilizing an ESP32 for deterministic real-time control.

![MagLev Prototype](./docs/maglev_front.jpg)

## 🧠 Control Theory: The Math Behind the Magic

Electromagnetic levitation is a non-linear, open-loop unstable physical system (it contains a right-half-plane pole). If the electromagnet pulls too hard, the object crashes into it; if it pulls too weakly, gravity drops the object.

To solve this:
1. **Plant Linearization:** The non-linear magnetic force equation was linearized around an equilibrium operating point (the target levitation gap).
2. **State-Space Modeling:** The system dynamics were modeled in a discrete-time state-space representation.
3. **LQR Optimization:** A Linear-Quadratic Regulator was designed in MATLAB to calculate the optimal feedback gain matrix ($K$). The $Q$ and $R$ matrices were iteratively tuned to balance the levitation stability vs. the PWM control effort.

## ⚡ Hardware Architecture

Since software is only as good as the hardware running it, a custom PCB was designed in KiCad to handle the high currents and minimize sensor noise.
* **Microcontroller:** ESP32 handling the control loop inside a FreeRTOS high-priority hardware timer.
* **Power Stage:** Logic-level MOSFETs / Half-Bridge topology driving the main electromagnet coil with high-frequency PWM to avoid acoustic whine and optimize current smoothing.
* **Sensors:** Analog Hall-Effect sensor (or IR, *ajustar según tu diseño*) placed directly below the coil to measure the air gap in real-time.
* **Connectivity:** The ESP32 leverages an MQTT client to broadcast telemetry (current draw, distance error) to a local network for real-time plotting without blocking the main control loop.

---

## 🚀 Repository Guide & Execution

Since this is a hardware-centric project, running it requires a sequential approach: from math, to hardware, to firmware.

### Phase 1: MATLAB Simulation & Gain Calculation
Before flashing any code, the optimal control gains must be calculated based on your physical coil and magnet mass.
1. Open MATLAB and navigate to the `/simulation` folder.
2. Run the `maglev_sysid_lqr.m` script.
3. The script will output the State Feedback Gain Matrix ($K$) to the console. Copy these values.

### Phase 2: Hardware Assembly
1. The Gerber files are located in `/hardware/fabrication/gerbers.zip`. Send these to your preferred manufacturer (e.g., JLCPCB or PCBWay).
2. Solder the BOM components. **Warning:** Ensure the flyback diode across the electromagnet is properly soldered to prevent inductive voltage spikes from frying the MOSFET.
3. Assemble the 3D printed mechanical frame (files in `/mechanical`).

### Phase 3: Firmware Flashing
1. Navigate to `/firmware/src/main.cpp`.
2. Paste your MATLAB-calculated $K$ matrix into the designated constant variables.
3. Build and flash the ESP32 using PlatformIO:
\`\`\`bash
cd firmware
pio run -t upload
\`\`\`

## Tuning Note
If the object oscillates rapidly before dropping, your $R$ penalty matrix in MATLAB is likely too low (the controller is too aggressive). Increase $R$, recalculate $K$, and re-flash.

---
*Developed by [Tu Nombre/Usuario]*
