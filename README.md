
# Autonomous Decision-Making Robot — PIC18F4550

An autonomous mobile robot built on the Microchip PIC18F4550 microcontroller that goes beyond simple line following: it navigates a track, stops at checkpoints, detects obstacles, identifies target colors, and acts on that information — all in a single closed decision loop.

Built as a Complex Engineering Problem (CEP) project at GIKI (Ghulam Ishaq Khan Institute of Engineering Sciences and Technology).

<img width="231" height="232" alt="image" src="https://github.com/user-attachments/assets/d9302fbf-c903-4f0a-9f85-f3d79f0c519b" />


## Overview

Unlike a basic Line Follower Robot (LFR) that only tracks a line, this platform uses the line as a navigation backbone while simultaneously reading ultrasonic and color sensor data to make real decisions at each checkpoint:

1. Follow a black line on a white background to a checkpoint.
2. Detect an obstacle/box via ultrasonic ranging and halt if it's within threshold range.
3. Identify the object's color (red or blue).
4. Rotate a servo clockwise for red, counter-clockwise for blue.
5. Resume line following to the next checkpoint, repeating until the track ends.

## Hardware

| Component | Role |
|---|---|
| PIC18F4550 | Main controller — runs all decision-making firmware |
| L298N | Dual H-bridge motor driver for the two DC gear motors |
| HC-SR04 | Ultrasonic sensor for obstacle detection at checkpoints |
| Color Sensor (TCS3200-type) | Discriminates red vs. blue targets |
| SG90 Micro Servo | Executes the color-conditional rotation |
| IR Reflectance Modules ×5 | Line-detection array (LM393 comparator based) |
| 12V Li-ion Battery Pack | Power source (3 × 4V cells in series) |

**Architecture:** the system is split into a sensing layer (IR array, ultrasonic, color sensor), a control layer (PIC18F4550 firmware), an actuation layer (L298N + servo), and a power layer (12V battery + on-board regulation). All sensor data converges on the microcontroller, which is the sole source of actuator commands.

## Software / Firmware

Developed in **C** using **MPLAB X IDE** and the **XC8 compiler**, built up in three milestones:

**Milestone 1 — Motor Control (PWM)**
CCP1/CCP2 modules generate PWM to drive both motors independently through the L298N; direction is set via GPIO to the IN1–IN4 inputs. Verified with bench tests (wheels off the ground) for linear speed control and reliable direction reversal.

**Milestone 2 — Line-Follower Algorithm**
The 5-element IR array produces a 5-bit pattern (black = HIGH, white = LOW), decoded into a discrete steering law:
- Centered on line → drive straight
- Drifting left/right → reduce duty cycle on the corresponding motor
- Junction/T-intersection → execute a pre-programmed turn
- Off the line entirely → stop and flag an error

**Milestone 3 — Full Autonomous Decision Loop**
Integrates obstacle and color detection into the line-following loop:
1. Follow line → 2. Detect checkpoint via ultrasonic ranging → 3. Halt if obstacle within threshold → 4. Read color sensor (red/blue) → 5. Rotate servo accordingly → 6. Resume line following.

Distance from the ultrasonic sensor is computed as:

```
d = (t · v) / 2
```
where `t` = measured echo pulse time, `v` ≈ 343 m/s (speed of sound).

## Results

- **Motor control:** smooth, monotonic speed response to PWM duty cycle; no stalling or overheating.
- **Line following:** consistent tracking on straights and curves; junctions handled correctly; automatic recovery from brief off-line excursions.
- **Obstacle detection:** repeatable, accurate halting distance at every checkpoint.
- **Color detection + servo response:** correct clockwise/counter-clockwise actuation for red/blue targets across multiple end-to-end runs.

## Known Limitations & Future Work

- Steering is a discrete state-based law, not PID — tighter curves would benefit from a PID controller.
- Color sensor is sensitive to ambient lighting; requires threshold recalibration per environment. Adding active white-LED illumination and normalized RGB ratios would harden this.
- IR array only supports a fixed 5-bit line pattern; a wider array or downward camera would allow more complex track topologies.
- No wheel encoders yet — adding them would enable closed-loop odometry and let the robot bridge gaps where the line is missing.

## Team
- Muhammad Daniyal Chishty

Dept. of Electrical Engineering, GIKI, Topi, Pakistan

## Acknowledgment

Thanks to the course instructor and lab staff at GIKI for support with hardware procurement, debugging, and testing throughout this project.
