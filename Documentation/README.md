This folder will provide all relevant documentation for the Viper quad-copter system

# Directory Structure
- Architecture - Describes the overall architecture of system, both software and hardware
- Components - Tips, datasheets, and other relevant info regarding Viper's individual components
- Embedded Linux - Tips on things to do with the embedded Linux platform on Pika Spark
- Tooling - guide on how to use tools useful in Viper project
- Simulation - Gazebo related docs

# Quick Start
## Running It
```shell
cd colcon_ws
. install/setup.bash
ros2 launch viper [launch-file.py]
```
### Launch file overview
- `viper-quad.py` - run the flight controller node directly on the Pika Spark hardware.
- `viper-sim.py` - run Gazebo simulation locally with the `viper` node, `joy`, `teleop_twist_joy`, and `ros_gz_bridge`. More details on Gazebo implementation [here](https://github.com/Kevin-Gu-2022/viper/blob/gazebo/gazebo/README.md).
- `viper-cosim.py` - co-simulation setup that runs the joystick/Gazebo side locally while the flight controller remains on the Pika Spark.
### Selecting a CAN Interface
A CAN interface must be set up by running the relevant [script](https://github.com/Kevin-Gu-2022/viper/blob/gazebo/scripts) before the flight controller node is run. There are 2 options:
- `vcan`: Setup by running `setup_vcan.sh` in `scripts` directory
- `can`: This can be the physical interface on Pika Spark (`setup_can.sh`) or `slcan` (`source setup_yakut_slcan.sh`) on host PC. Scripts register under `can` for simplicity. For more details on which script to run, see [here](https://github.com/Kevin-Gu-2022/viper/blob/gazebo/scripts/README.md).
	
If you use the Zubax Babel as a USB-CAN device you need to run `. setup_yakut_slcan.sh` first.

### Wireless Comms
- Need to set up PS3 controller too. See [[PS3 Controller]]
- By connecting to Pika Spark's WiFi hotspot, the messages created from the PS3 controller should be shared with the flight controller 
- See [[WiFi Connection]] for more details

## Building
- Follow the concise build instructions [here](https://github.com/Kevin-Gu-2022/viper/blob/gazebo/docs/build.md)
- Documented in [[Cross Compilation]]
# WARNING
- This is a quadcopter with parts moving at high speed, especially when propellers are attached
- When battery plugged in, make sure the two banana leads do not touch
	- This WILL short the battery, and MAY lead to FIRE / thermal runaway!

