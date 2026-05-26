- To communicate between ROS 2 and Gazebo topics, run this:
  ```bash
ros2 run ros_gz_bridge parameter_bridge /TOPIC@ROS_MSG@IGN_MSG
  ```
  - `ros_gz` supports YAML file configuration
  - [Source](https://github.com/gazebosim/ros_gz/blob/ros2/ros_gz_bridge/README.md#example-1a-ignition-transport-talker-and-ros-2-listener)