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
![Robot Arm Blueprint]<img width="468" height="674" alt="image" src="https://github.com/user-attachments/assets/d0c0fb10-26f1-4c1f-9d8d-6707b45ed179" />

## 3.2 Wiring & PCB
