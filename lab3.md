---
layout: page
title: Lab 3
---

## Lab 3:

### Prelab: 

In this lab we evaluate a VL53L1X Time-of-Flight sensor. This sensor has a default I2C address of 0x52. 2 ToF sensors are used to record distances from the car to its surroundings. Each sensor has a range of 4 meters and field-of-view of 27 degrees. 

Given these constraints, I chose to mount one sensor at the front of the car to detect obstacles directly ahead, and place the second on the side to provide lateral distance measurements for trajectory control. A potential drawback of this sensor placement is that the robot is unable to detect any obstacles behind it. 

To use both ToF sensors at the same time, we have to change the address of one of the sensors. Following [this](https://community.st.com/t5/imaging-sensors/can-we-change-the-vl53l1x-address/td-p/310169) forum discussion, we run the following:




<div style="text-align: center;">
  <img src="assets/img_lab2/imu.png" alt="ICM-20948 IMU" width="400"/>
</div>

Prelab
Briefly discuss the approach to using 2 ToF sensors
Sketch of wiring diagram (with brief explanation if you want)

### Task 1:

Picture of your ToF sensor connected to your QWIIC breakout board
Screenshot of Artemis scanning for I2C device (and discussion on I2C address)
Discussion and pictures of sensor data with chosen mode
2 ToF sensors and the IMU: Discussion and screenshot/video of sensors working in parallel
Tof sensor speed: Discussion on speed and limiting factor; include code snippet of how you do this
Time v Distance: Include graph of data sent over bluetooth (2 sensors)
Time v Angle: Include graph of data sent over bluetooth
(5000) Discussion on infrared transmission based sensors
(5000) Sensitivity of sensors to colors and textures