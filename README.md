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

## 4.1 
