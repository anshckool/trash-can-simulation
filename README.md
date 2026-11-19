# Autonomous Target-Tracking Smart Trash Can (5 m/s Sweet Spot Hybrid Build)

[![Live Demo](https://img.shields.io/badge/Live_Demo-Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white)](https://trash-can-simulation.vercel.app)
[![GitHub Repo](https://img.shields.io/badge/GitHub-Repository-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/anshckool/trash-can-simulation)

An interactive, high-fidelity 3D CAD and real-world Newtonian physics simulation of an autonomous, target-tracking omnidirectional trash can. Built using **Three.js (r128)**, **OrbitControls**, and **Tailwind CSS**.

🔗 **Live Production URL:** [https://trash-can-simulation.vercel.app](https://trash-can-simulation.vercel.app)

Live simulation features strict Newtonian projectile dynamics, quadratic air drag, 120 FPS camera latency pipelines, 3-wheel Kiwi holonomic kinematics, traction clamping, and dynamic chassis inertia.

![Autonomous Smart Trash Can Simulation](https://img.shields.io/badge/Physics-120Hz_Newtonian-00d2ff)
![Kinematics](https://img.shields.io/badge/Kinematics-3--Wheel_Kiwi_120%C2%B0-6366f1)
![Top Speed](https://img.shields.io/badge/Top_Speed-5.0_m%2Fs_(1%2C012_RPM)-10b981)
![Max Accel](https://img.shields.io/badge/Max_Accel-5.5_m%2Fs%C2%B2_(Tile_Traction)-f59e0b)
![Vision](https://img.shields.io/badge/Vision-120_FPS_OV9281-a855f7)

---

## 🚀 Key Engineering & Physical Features

### 1. Mechanical Assembly & 3D CAD Model
- **Wide Anti-Tip Chassis:**
  - 36 cm diameter CNC circular lower deck ($R = 0.18\text{ m}$, $y = 0.065\text{ m}$) + upper collar deck ($y = 0.155\text{ m}$) fastened by 4 brass hex standoffs.
  - Center of Gravity maintained strictly below $0.14\text{ m}$.
- **3-Wheel 120° Kiwi Holonomic Drive:**
  - 3x **100mm Double Aluminium Omni Wheels** with visible dual-row rubber perimeter rollers (20 rollers per wheel).
  - 3x **5010 360KV Brushless Outrunner Motors** coupled via **3:1 HTD timing belts** and 15T/45T pulleys to wheel drive shafts.
  - 3x **VESC Mini FOC Controllers** running field-oriented sinusoidal commutation.
  - **ESP32-WROOM-32 #2 Receiver**, **LM2596S 5V DC-DC Buck**, and **1.5KE27A TVS Diode + 1000µF 35V low-ESR Capacitor** for regenerative braking spike absorption.
- **Direct 6S LiPo Power:**
  - 22.2V 2200mAh 45C orange LiPo battery mounted flat on the bottom center of the lower deck.
- **Catch Basket & ArUco Rim Marker:**
  - Tapered frosted polycarbonate bin (30 cm mouth diameter, 22 cm base, rim height $y = 0.568\text{ m}$) with an ArUco 4x4 fiducial marker decal for overhead 6-DOF tracking.
- **Exploded View & X-Ray Shell:**
  - Interactive slider ($0\%\rightarrow 100\%$) smoothly separates all chassis tiers and drive units.
  - X-Ray toggle converts outer bin into a translucent wireframe shell to view internal electronics and belts in real-time.
- **Ceiling Vision Station:**
  - Overhead 120 FPS OV9281 Monochrome Global-Shutter Camera ($y = 3.2\text{ m}$) with planar homography projection cone over the 3m x 3m tile arena.
  - Side vision desk with laptop running OpenCV MOG2 optical tracking and ESP32 #1 USB dongle transmitting visible purple ESP-NOW RF pulses.

---

### 2. Strict Real-World Physics Engine (No Guaranteed Catches)
- **Quadratic Aerodynamic Fluid Drag:**
  - Models a $4.5\text{ g}$ crumpled paper ball ($r = 0.035\text{ m}$, $C_d = 0.80$, $\rho = 1.225\text{ kg/m}^3$).
  - Integrates $\vec{F}_d = -0.5 \cdot \rho \cdot C_d \cdot A \cdot |\vec{v}| \cdot \vec{v}$ alongside gravity $g = 9.81\text{ m/s}^2$ at 120 Hz.
  - Renders both the **grey dashed vacuum parabola** and **cyan drag-decayed trajectory**, demonstrating the genuine 15%–35% aerodynamic range drop.
- **Realistic Vision Latency Pipeline:**
  - **95 ms Initial Latency:**
    - 7 optical frames @ 120 FPS ($58\text{ ms}$)
    - OS UVC & MJPEG decode ($25\text{ ms}$)
    - OpenCV MOG2 contour & trajectory fitting ($4\text{ ms}$)
    - USB serial to ESP32 #1 dongle ($4\text{ ms}$)
    - ESP-NOW 2.4GHz RF packet ($3\text{ ms}$)
    - VESC FOC current controller ramp ($5\text{ ms}$)
  - During $0–95\text{ ms}$, the robot remains stationary while the amber target ring jitters with $\pm 5.5\text{ cm}$ estimation noise.
  - **210 ms Refinement:** Second ESP-NOW packet updates the target ring to green with $\pm 1.6\text{ cm}$ noise, enabling mid-flight trajectory corrections.
- **Actuator Limits & Traction Clamping:**
  - $v_{max} = 5.0\text{ m/s}$ ($1,012\text{ RPM}$ at 100mm wheels with 6% lateral scrub).
  - $a_{max} = 5.5\text{ m/s}^2$ clamped by tile friction ($\mu \approx 0.56$), below the $a_{tip} = 6.3\text{ m/s}^2$ tipping threshold ($g \cdot \frac{0.09}{0.14}$).
  - Dynamic chassis tilt inertia ($\pm 0.08\text{ rad}$ max) reacting to linear acceleration and braking.
  - 5%–10% stochastic floor slip perturbation corrected via closed-loop PD visual servoing.
- **True 3D Rim Collision Physics (at $y = 0.568\text{ m}$):**
  - **$err \le 0.125\text{ m}$:** Clean Catch — ball enters basket, dissipates velocity, and settles at the bottom.
  - **$0.125\text{ m} < err \le 0.175\text{ m}$:** Rim Bounce — ball strikes the circular rim, deflects outward/upward ($0.45$ restitution), and bounces onto the floor.
  - **$err > 0.175\text{ m}$:** Late Miss — robot cannot reach the landing zone in time; ball hits the floor.

---

### 3. Interactive UI & Controls
- **Camera Presets:** Isometric, Under-Chassis, 120 FPS Overhead POV, Kiwi Top-Down, Vision Desk.
- **Throw Controls:** Normal Toss, Extreme Far Toss, Auto-Toss loop, Center Bin reset, and manual Throw Speed ($2.5–6.0\text{ m/s}$) & Distance ($0.4–1.8\text{ m}$) sliders.
- **Live Telemetry HUD:** Speed, Accel, Kiwi M1/M2/M3 RPM bars ($0–1,012\text{ RPM}$), Chassis Tilt, Impact Error, and Catch / Bounce / Miss tallies.
- **Interactive Inspector:** Click on any 3D part in the viewport or sidebar to inspect CAD parameters and tolerances.
- **Synthesized Audio:** Built-in procedural Web Audio API sound effects for throws, radio pulses, catches, and rim bounces.

---

## 🛠️ Local Development & Viewing
Double-click `index.html` or run:
```bash
python -m http.server 8080
```
Open [http://localhost:8080](http://localhost:8080) in your browser.
