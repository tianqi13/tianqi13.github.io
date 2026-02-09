---
layout: page
title: Lab 2
---
## Lab 2:

### Task 1: Set up IMU 

The IMU used in this lab is the ICM-20948, with a three axis gyroscope, accelerometer and magnetometer. 

<div style="text-align: center;">
  <img src="assets/img_lab2/imu.png" alt="ICM-20948 IMU" width="400"/>
</div>

AD0_VAL specifies the least significant bit of the IMU’s 7-bit I2C address. The default value is 1 because the AD0 pin is pulled high. It should be set to 0 only if the ADR jumper is soldered, which ties the pin to ground and changes the device address.

Below is the Serial Monitor output from the Example1_Basic sketch:

<div style="text-align: center;">
  <img src="assets/img_lab2/data.png" alt="ICM-20948 IMU" width="800"/>
</div>

3 sets of data corresponding to gyroscope, accelerometer and magnetometer are printed. This screenshot was taken when the IMU was laid flat on the table, which corresponds to the near 0 values for _a_x_ and _a_y_. The measured _a_z_ value is approximately +1g, which is the upward normal force on the sensor when it is stationary on the table. 

Similarly, the gyroscope values indicate near-zero angular velocity about all three axes, consistent with the IMU being at rest.

### Task 2: Accelerometer 

Pitch and Roll are calculated with these equations: 
<div style="text-align: center;">
  <img src="assets/img_lab2/atan.png" alt="ICM-20948 IMU" width="400"/>
</div>

We use atan2() as it uses the signs of both readings to determine the correct quadrant that the angle lies in. 

Pitch and Roll values at -90, 0, 90 are:
<div style="display: flex; justify-content: center;">

<table>
  <tr>
    <td colspan="2" align="center">
      <img src="assets/img_lab2/0deg.png" width="300"><br>
      <sub>0 degree Pitch and Roll</sub>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="assets/img_lab2/pitch_90.png" width="300"><br>
      <sub>90 degree Pitch</sub>
    </td>
    <td align="center">
      <img src="assets/img_lab2/pitch_n90.png" width="300"><br>
      <sub>-90 degree Pitch</sub>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="assets/img_lab2/roll_90.png" width="300"><br>
      <sub>90 degree Roll</sub>
    </td>
    <td align="center">
      <img src="assets/img_lab2/roll_n90.png" width="300"><br>
      <sub>-90 degree Roll</sub>
    </td>
  </tr>
</table>

</div>

There are small discrepancies in the calculated pitch and roll values (not exactly 0/90/-90), so we perform two point calibration to reduce this error. 

We measure the mean pitch and roll values recorded at -90° and +90°, then compute a 2-point linear calibration of the form:

    angle_cal = scale * angle_raw + offset

Measured values:
- Pitch: $r_{-90}$ = -89.14 , $r_{+90}$  = 86.53
- Roll:  $r_{-90}$ = -88.38 , $r_{+90}$  = 85.223

Scale and Offset:

    pitch_cal = 1.0246 * pitch_raw + 1.34
    roll_cal = 1.0368 * roll_raw + 1.64

Now we observe more accurate pitch and roll readings:

<div style="text-align: center;">
  <iframe width="560" height="315"
    src="https://youtu.be/UTVPtDu3RzI?si=3wIHFu2NvPdfUONE"
    title="Stunt"
    frameborder="0"
    allowfullscreen>
  </iframe>
</div>

Similar to the buffering method implemented in Lab 1, we store 1000 sensor readings and send the data to the computer over BLE. 

<div style="text-align: center;">
  <img src="assets/img_lab2/raw_roll_pit.png" alt="ICM-20948 IMU" width="500"/>
</div>

Pitch and roll values calculated from raw accelerometer data is noisy. We perform a frequency spectrum analysis to investigate the noise.

<div style="text-align: center;">
  <img src="assets/img_lab2/fft_code.png" alt="ICM-20948 IMU" width="500"/>
</div>

<div style="text-align: center;">
  <img src="assets/img_lab2/fft.png" alt="ICM-20948 IMU" width="500"/>
</div>

From the spectrum, most of the energy associated with noise appears at higher frequencies, while the useful signal is primarily within **0 to 5 Hz**, suggesting that a low-pass filter with cutoff frequency 5Hz would be a good choice. 

We calculate α using the sampling and cut-off frequency, and use it to generate a digital low pass filter.

<div style="text-align: center;">
  <img src="assets/img_lab2/lpf.png" alt="ICM-20948 IMU" width="500"/>
</div>

When we plot the raw vs filtered Pitch and Roll values, we observe a substantial decrease in noise. The angle plots are much smoother and the random peaks (due to noise) are ignored. 

<div style="text-align: center;">
  <img src="assets/img_lab2/filtered.png" alt="ICM-20948 IMU" width="500"/>
</div>

We introduce vibrational noise by gently tapping the table and observe the resulting FFT. Noticeable peaks appear at higher frequencies.

<div style="text-align: center;">
  <img src="assets/img_lab2/vib_noise_freq.png" alt="ICM-20948 IMU" width="500"/>
</div>

Our LPF is able to successfully attenuate high-frequency noise, and the filtered pitch and roll converge to approximately 0°.

<div style="text-align: center;">
  <img src="assets/img_lab2/vib_noise.png" alt="ICM-20948 IMU" width="500"/>
</div>

### Task 3: Gyroscope
Pitch, Roll and Yaw angle values from the Gyroscope readings are calculated using:

<div style="text-align: center;">
  <img src="assets/img_lab2/gyro_eq.png" alt="ICM-20948 IMU" width="200"/>
</div>

Implemented in code:
<div style="text-align: center;">
  <img src="assets/img_lab2/gyro.png" alt="ICM-20948 IMU" width="500"/>
</div>

Plot of pitch, roll, and yaw as IMU was rotated in order of Z, X, Y axis:
<div style="text-align: center;">
  <img src="assets/img_lab2/gyro_pry.png" alt="ICM-20948 IMU" width="500"/>
</div>

The angles derived from gyroscope readings accurately track rapid changes in orientation with minimal noise. In the roll graph, the accelerometer shows a negative dip while the gyroscope remains near zero because the IMU is rotating about the Y axis (pitch), which introduces linear acceleration that corrupts the accelerometer-based roll estimate. 

However, the gyroscope suffers from drift. Despite the IMU bring stationary, the angles calculated increase with time due to the accumulation of small bias errors during integration.

<div style="text-align: center;">
  <img src="assets/img_lab2/drift.png" alt="ICM-20948 IMU" width="500"/>
</div>

The above plots demonstrate that the accelerometer and gyroscope exhibit complementary characteristics. The accelerometer provides a stable long-term reference but becomes unreliable during dynamic motion, while the gyroscope accurately captures short-term rotational changes yet accumulates drift over time. 

As such, we design a complementary filter:
<div style="text-align: center;">
  <img src="assets/img_lab2/comp_filter_eq.png" alt="ICM-20948 IMU" width="200"/>
</div>

<div style="text-align: center;">
  <img src="assets/img_lab2/comp_filter.png" alt="ICM-20948 IMU" width="500"/>
</div>

Increasing α means that we place greater weight on the accelerometer measurement and less on the gyro. After playing around with different values of α, I found that a value of _0.05_ gave a plot that follows the gyroscope during rapid motion while still converging back toward zero when the IMU is at rest.  

<div style="text-align: center;">
  <img src="assets/img_lab2/comp_ss.png" alt="ICM-20948 IMU" width="500"/>
</div>


### Task 4: Sampling

A new case SET_RECORD controls the flag to start recording (record_main).

<div style="text-align: center;">
  <img src="assets/img_lab2/set_record.png" alt="ICM-20948 IMU" width="500"/>
</div>

Data collection is handled in the main loop. Instead of waiting for data to be ready, the code skips over the current loop if data is not ready. Extra logic is used to ensure data is only recorded when record_main is true, and when the array is not yet full.

<div style="text-align: center;">
  <img src="assets/img_lab2/main.png" alt="ICM-20948 IMU" width="500"/>
</div>

A second GET_READINGS command starts data transmission from board to computer. Data is sampled at a rate of roughly 329.5Hz.

A running count _total_loop_ is incremented each time the main loop is run. The measurement loop starts when the board receives the SET_RECORD command. At the end of 5 seconds, we calculate the frequency of the main loop using loops_elapsed/5.0. 

<div style="text-align: center;">
  <img src="assets/img_lab2/measure_loop.png" alt="ICM-20948 IMU" width="500"/>
</div>

The main loop of the Arduino runs at roughly 2383.0Hz, 7x faster than the IMU produces new values. 

In this lab, I chose to have separate arrays for storing Accelerometer and Gyroscope data. This method is more organized and code is more readable, especially when sending data over BLE.

Floats were used because the IMU returns floating point values, and the -$\pi$ to $\pi$ angle range requires sufficient precision to avoid quantization errors. Floats are also 4 bytes, which are memory efficient when compared to strings and doubles. 

My program uses 6 arrays: time, pitch, roll (from accelerometer), omegaX, Y, Z (from gyroscope). Setting each of them to length 5000 gives a compile time message:

<div style="text-align: center;">
  <img src="assets/img_lab2/size.png" alt="ICM-20948 IMU"/>
</div>

Assuming all remaining 227920 bytes are all allocated to these arrays, each array could be ~9496 values larger, which means a maximum array length of 14496. A sampling rate of 329Hz means it would take ~43.86 seconds to fill these arrays.

Recording 5000 readings corresponds to approximately 18 seconds of raw sensor data, which the plot below shows:

<div style="text-align: center;">
  <img src="assets/img_lab2/17seconds.png" alt="ICM-20948 IMU"/>
</div>

### Task 5: Stunts

<div style="text-align: center;">
  <iframe width="560" height="315"
    src="https://www.youtube.com/embed/9JmRxtrYF88"
    title="Stunt"
    frameborder="0"
    allowfullscreen>
  </iframe>
</div>

The car reacts quickly and accelerates rapidly once commanded by the controller. It is able to turn on the spot if the right button is held down, but it often catches an edge and flips. 



