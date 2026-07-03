# User Input Latency Test
Used a quick package to perform latency tests between host computer and Pika Spark.
This package sends a message to Pika Spark on `/latency_ping` and the node on Pika Spark responds on `/latency_pong`. Used to test latency of physical medium, as well as ROS 2's DDS.
# Build
```bash
colcon build --packages-select latency_initiator
```
# Run
```bash
. install/setup.bash
ros2 run latency_initiator latency_pong
```

- RTT average eyeballing it is around 11 to 13 ms


# Jitter in Control Loop
- Jitter in motor_speed commands
- 0.0101 s for mean
- 1.99 ms for std dev
- Not too bad, but still quite significant
# Jitter in IMU Publishing
- This is tested in old image
- Characterises the jitter in `/imu` topic
- Basically how quickly my state estimator within node is updating
- Mean: 0.0053 s when it is supposed to be 5 ms
- Std deviation: 5.50 ms. Quite significant
# Jitter Compensation
- IMU sampling and control loop are decoupled. Control loop just uses the latest IMU packet
- Not yet implemented, but this would be extremely helpful:
	- Adding a predicted_angle based on age of the last IMU packet and its gyro_rate, i.e. the rad/s
	- First order approximation should help to alleviate effects of jitter
	- Though, best is still to characterise this measurement age first
```bash
predicted_angle =
measured_angle +
gyro_rate * measurement_age;
```

- I am also using the dt from last loop for the PID. I am not assuming constant loop rate.

# Literature
### Effect of sampling jitter and control jitter on positioning error in motion control systems
- Control jitter and sampling jitter
- Control jitter is jitter in the control loop, or equivalently in the motor_speed ZOH
- Sampling jitter is the jitter in the sampling
- Basically, can inject $1-z^{-1}$ or $z^{-1}-1$ plus a normalised jitter function after the ideal ZOH and ideal samplers
- Converts a time variant digital control system into a *time invariant* digital control system
- High-speed systems that require faster sampling rates will be more susceptible to jitter
- sampling jitter has a negligible effect on regulation error
- control jitter operates primarily on the high frequency controller gain to produce a low frequency disturbance, which is counteracted by the controller’s disturbance rejection response
- the presence of control jitter will contribute additional regulation error to the digital control system
- Solutions to jitter effect on regulation error
	- one method is to increase the controller disturbance rejection capability
		- stability constraints will impose limits on the attainable disturbance rejection of the controller
	- attenuate the controller gain near the system Nyquist frequency ωN = /T0
		- involves adding a jitter compensator
	- Or, just find better hardware and design to minimise jitter lol
- the previously discussed jitter compensator Cg (z) in Eq. (42) is generally not helpful in reducing jitter’s effect on tracking error because the frequencies of the reference command are usually far less than the system Nyquist frequency
	- Suggestion is get better hardware
- an experimentally measured plant frequency response is sufficient to calculate the effect of jitter

### The Jitter Margin and Its Application in the Design of Real-Time Control Systems
- The jitter margin is defined as a function of the amount of constant delay in the control loop, and it describes how much additional time-varying delay can be tolerated before the loop goes unstable
- For continuous-time control systems, the delay margin can be computed as Lm = ϕm/ωc, where ϕm is the phase margin and ωc is the crossover frequency of the system. Due to aliasing effects, the exact computation is more complicated for computer-controlled systems
- Delay is the constant bit and jitter is the extra variation
	- The input-output delay experienced by the controller can be divided into two parts: a constant part, L ≥ 0, and a timevarying part (the jitter), J ≥ 0, see Figure 2. The minimum possible delay is hence given by L, and the maximum possible delay is given by L+J.
- This paper mainly proposes the delay and jitter margins, similar to the phase margins, where it describes how much room a system has to play around before going unstable