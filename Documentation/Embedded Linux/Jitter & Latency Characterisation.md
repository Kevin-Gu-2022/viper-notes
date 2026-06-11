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
# Jitter in IMU Publishing
- This will need to be in old image
- Characterises the jitter in `/imu` topic
- Basically how quickly my state estimator within node is updating

# Jitter in 