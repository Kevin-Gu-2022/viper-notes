This folder will provide all relevant documentation for the Viper quad-copter system

# Directory Structure
- Architecture - Describes the overall architecture of system, both software and hardware
- Components - Tips, datasheets, and other relevant info regarding Viper's individual components



# Quick Start

## Simulation

## Actual Flight

### WARNINGS
- This is a quadcopter with parts moving at high speed, especially when propellers are attached
- When battery plugged in, make sure the two banana leads do not touch
	- This WILL short the battery, and MAY lead to FIRE / thermal runaway!


`ros2 launch viper viper-quad.py`
- This will launch the program, but it will not launch the `joy_node` and `teleop_node` needed for wireless control
- Launch this separately on your host machine
- By connecting to Pika Spark's WiFi hotspot, the messages created from the PS3 controller should be shared with the flight controller 