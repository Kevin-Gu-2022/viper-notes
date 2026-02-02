
[Understanding Sensor Fusion and Tracking, Part 1: What Is Sensor Fusion?](https://www.youtube.com/watch?v=6qV3YjFppuc)
- Low pass filters add delays
- Using multiple of same sensors, and averaging or voting to see which is more accurate 
- Combining sensors with physical world: Kalman filters 
- Using different sensors good too, e.g. gyro and accelerometer


[Understanding Sensor Fusion and Tracking, Part 2: Fusing a Mag, Accel, & Gyro Estimate](https://www.youtube.com/watch?v=0rlvvYgmTvI)
- Accelerometer + magnetometer together: can use cross products to determine coordinate frames
	- Accelerometer sensitive to linear acceleration
	- Also sensitive to rotational acceleration if IMU not in centre of mass
- Combine with system model to subtract the acceleration due to linear acceleration i.e. from motors
	- Doesn't really work with large disturbances
- Alternatively, ignore acceleration measurements that are outside some threshold of 1g, though lose state of system when outside threshold
- Gyro: multiplying measured angular rate by sample time $\Delta t$ gives the delta angle. Basically integrating the gyro, though this integrates the bias and noise too, so drift happens
![[gyro_drift.png]]
- - Filters:
	- Complementary: a trust line with a slider between accelerometer/magnetometer and gyro, hence the 'complement' of each other. This constant is manually selected
	- Kalman: Similar, but constant is calculated based on noise, system model accuracy/trust
	- Madgwick
	- Mahony
- All first initialise attitude then use mag field and gravity to correct gyro drift
[Drone Control and the Complementary Filter](https://www.youtube.com/watch?v=whSw42XddsU)
- Flix uses complementary filter
- Bias and random noise are summed (think of integrating a constant)
- Integrating gyro is more accurate in short term, but susceptible to drift in long term. Integration is equivalent to low pass filter and reduces noise (remember, a capacitor is an integrator, so think about how a capacitor is unable to change charge quickly at high frequencies)
- Accelerometer has absolute measurement of where down is, but is noisy and susceptible to linear and rotational accelerations, so unreliable in short term
- A basic overview of how the complementary filter works
![[basic_complementary_filter.png]]
- Practical implementation of the filter![[practical complementary filter.png]]