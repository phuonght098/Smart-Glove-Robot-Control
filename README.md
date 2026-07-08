# Robotic Arm Control via Sensor Glove

# Table of Content
1. Introduction
2. Features
3. Hardware
4. Software
5. Getting Started
6. Results
7. Future Work
8. Contact
9. License

# 1. Introduction
This project focuses on the design and fabrication of a 5-degree-of-freedom (5 DOF) robotic arm equipped with a gripper, capable of imitating human arm movements in real-time. Tilt angles and motion data are acquired via a sensor glove equipped with the ICM20948 IMU module.

The project is developed with the aim of researching robotic applications in hazardous environments, enabling remote pick-and-place operations without direct physical intervention.

# 2. Features
🎯 Real-time Control: Low latency operation, accurately tracking and imitating human hand movements.

🧭 Digital Processing with DMP: Utilizes the ICM20948's onboard Digital Motion Processor to calculate Quaternions directly on the sensor, significantly reducing the processing load on the main microcontroller.

⚡ Power Optimization: Features a customized voltage divider circuit and optimized current consumption for better efficiency.

🛠️ Fully Open-source: Provides complete source code, PCB design files, and 3D printing models.

# 3. Hardware

## 3.1 Bill of Materials - BOM
Here is the list of main components used in this project.

| Component  | Qty | 
| :--- | ---: | 
| ESP32 | 2 | 
| ICM20948 | 3 | 
| Servo MG996R | 3 | 
| Servo MG90S | 3 |
| PCA9685 | 1 |
| Flex Sensor | 1 |
| Resistor 4.7k | 2 |
| Resistor 10k | 1 |
| Resistor 100k | 1 |
| Resistor 200k | 1 |
| LED | 1 |
| Toggle Switch | 1 |
| Lipo 2s 420mAh | 1 |
| Mini360 | 1 |
| Ball Bearing 6806ZZ (30x42x7) | 1 |

📌 Note: For the mechanical structure, all .STL files required for 3D printing the robotic arm are provided in the /3D_Models directory.
<img width="468" height="674" alt="image" src="https://github.com/user-attachments/assets/d0c0fb10-26f1-4c1f-9d8d-6707b45ed179" />

## 3.2 Wiring & PCB
The schematic and PCB layout files, designed using EasyEDA.com, are located in the /Hardware folder. Due to the I2C address conflict among the ICM20948 sensors (which have a default address of 0x68), we need to connect the AD0 pin of each sensor to Vcc or GND to fix their addresses to either 0x68 or 0x69, as illustrated below.

<img width="395" height="622" alt="image" src="https://github.com/user-attachments/assets/b454093c-5fd3-4e43-abee-58911b8b81de" />

Below is the wiring diagram of the sensor glove circuit.

<img width="677" height="420" alt="image" src="https://github.com/user-attachments/assets/53cb7cd3-c1b8-44ae-a9c9-7d3bc5710e85" />

A voltage divider circuit is implemented for the flex sensor to read the ADC value on pin D34 of the ESP32. When the flex sensor bends, its resistance changes, which in turn alters the voltage at the junction between the flex sensor and the 10k resistor.

Image of the sensor glove's PCB layout:

<img width="1142" height="537" alt="image" src="https://github.com/user-attachments/assets/c19a23ac-2bc5-4c76-a716-27035e868c17" />

## 3.3 3D Printing
Recommended print settings:

Material: PLA / PETG

Infill: > 30%

Layer height: 0.2mm

# 4. Software

## 4.1 Overall System Flowchart
<details>
<summary>📊 System Flowchart</summary>

```mermaid
graph TD
  START([START]) --> INIT1[Initialize ESP32]
  INIT1 --> INIT2[Initialize I2C Bus]
  INIT2 --> INIT3[Initialize 3x ICM20948 IMUs]
  INIT3 --> INIT4[Initialize DMP Quaternion]
  INIT4 --> INIT5[Initialize ESP-NOW]
  INIT5 --> WAIT[Wait for DMP convergence 60s]
  WAIT --> COND{PCA Calibration?}
  
  COND -- Yes --> PCA[PCA Calibration]
  PCA --> ZERO[Zero Calibration]
  ZERO --> MAIN[Main Loop]
  
  COND -- No --> IDLE[Wait for Command]
  IDLE --> MAIN
  
  MAIN --> READ[Read Quaternions from 3 IMUs]
  READ --> SYNC[Synchronize Timestamps]
  SYNC --> NLERP[NLERP Filtering]
  NLERP --> ANAT[Convert to Anatomical Frame]
  ANAT --> CALC[Calculate Joint Angles S1-S5]
  CALC --> MAP[Map to Servo PWM]
  MAP --> SEND[Send via ESP-NOW]
  SEND --> EXEC[Robot Execution]
  EXEC --> READ
```

</details>

## 4.2 Flowchart PCA Calibration
This is the most unique core algorithm of the project for coordinate axis calibration.

<details>
<summary>📊 PCA Calibration </summary>

```mermaid
graph TD
  START([START PCA]) --> NPOSE[N-Pose Posture]
  NPOSE --> GRAV[Measure Gravity Vectors<br>gUpper, gForearm, gHand]
  GRAV --> PCA_S[Shoulder PCA <br> Shoulder Motion]
  PCA_S --> THU_S[Collect Quaternions for 3s]
  THU_S --> DELTA_S[Calculate ΔQuaternion]
  DELTA_S --> COV_S[Create Covariance Matrix]
  COV_S --> POW_S[Power Iteration]
  POW_S --> AXIS_S[Shoulder PCA Axis]
  
  AXIS_S --> PCA_E[Elbow PCA <br> Elbow Motion]
  PCA_E --> THU_E[Collect Quaternions for 3s]
  THU_E --> DELTA_E[Calculate ΔQuaternion]
  DELTA_E --> COV_E[Create Covariance Matrix]
  COV_E --> POW_E[Power Iteration]
  POW_E --> AXIS_E[Elbow PCA Axis]
  
  AXIS_E --> PCA_W[Wrist PCA <br> Wrist Motion]
  PCA_W --> THU_W[Collect Quaternions for 3s]
  THU_W --> DELTA_W[Calculate ΔQuaternion]
  DELTA_W --> COV_W[Create Covariance Matrix]
  COV_W --> POW_W[Power Iteration]
  POW_W --> AXIS_W[Wrist PCA Axis]
  
  AXIS_W --> GRAM[Gram-Schmidt Orthogonalization]
  GRAM --> MOUNT[Create Mount Quaternion]
  MOUNT --> SAVE[Save Q_Mount]
  SAVE --> END([END])
```
</details>
## 4.3 Quaternion Processing Pipeline
This is the main continuous data processing pipeline in the system loop.


<details>
<summary>📊 PCA Calibration </summary>

```mermaid
graph TD
  Q1[/Quaternion Upper/] --> SYNC[Time Synchronization]
  Q2[/Quaternion Forearm/] --> SYNC
  Q3[/Quaternion Hand/] --> SYNC
  
  SYNC --> NLERP[NLERP Filter]
  NLERP --> F1[Q_Upper_Filtered]
  NLERP --> F2[Q_Forearm_Filtered]
  NLERP --> F3[Q_Hand_Filtered]
  
  F1 --> MOUNT[Mount Calibration]
  F2 --> MOUNT
  F3 --> MOUNT
  
  MOUNT --> ANAT[Anatomical Quaternion]
  ANAT --> DRIFT[Drift Compensation]
  DRIFT --> REL[Relative Quaternion]
  
  REL --> J1[Shoulder]
  REL --> J2[Elbow]
  REL --> J3[Wrist]
  
  J1 --> ANGLES[/Joint Angles S1 S2 S3 S4 S5/]
  J2 --> ANGLES
  J3 --> ANGLES

```
</details>

## 4.4 Shoulder Angles
<details>
  
```mermaid
graph TD
  IN[/Q_Shoulder/] --> MAT[Quaternion → Matrix]
  MAT --> R[R_Shoulder Matrix]
  
  R --> S1[S1: atan2<br>-R01, R00]
  R --> S2[S2: asin<br>R02]
  
  S1 --> OUT[/Result: Shoulder Angles/]
  S2 --> OUT


```
</details>
## 4.5 Elbow Angle
<details>

```mermaid
graph TD
  U[/Anat_Upper/] --> INV[Inverse]
  INV --> MUL{X}
  F[/Anat_Forearm/] --> MUL
  
  MUL --> QE[Q_Elbow]
  QE --> DECOMP[Decompose Swing-Twist]
  DECOMP --> TWIST[Twist Component]
  TWIST --> CALC[Calculate Twist Angle<br>2 * atan2]
  
  CALC --> OUT[/S3 Angle <br> Flexion / Extension/]

```
</details>

## 4.6 Wrist Angles
<details>
  
```mermaid
graph TD
  F[/Anat_Forearm/] --> REL[Relative Quaternion]
  H[/Anat_Hand/] --> REL
  
  REL --> QW[Q_Wrist]
  QW --> ST[Swing-Twist Decomposition]
  
  ST --> TWIST[Twist Component]
  TWIST --> S4[/S4 Angle/]
  
  ST --> SWING[Swing Component]
  SWING --> ROT[Rotation Matrix]
  ROT --> ATAN[atan2]
  ATAN --> S5[/S5 Angle/]

```
</details>

# 5. Getting Started
