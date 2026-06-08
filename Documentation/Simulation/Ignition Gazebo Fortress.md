
- Red is the x-axis, green is the y-axis and blue is the z-axis.
- Gazebo Fortress has slightly different syntax to current Gazebo E.g.:
```bash
ign gazebo ... # Gazebo Fortress
gz sim ... # Newer versions
```

## Paths
- Environment variable where Gazebo will look for local models: `IGN_GAZEBO_RESOURCE_PATH`: `export IGN_GAZEBO_RESOURCE_PATH="/home/kevin/dev/viper_ws/src/viper/gazebo/models"`
	- `model.config` is there for the Gazebo to find the correct file
- The `mode://` bit gets replaced with the IGN_GAZEBO_RESOURCE_PATH
- Could add this to `package.xml`, this way when you `source install/setup.bash`, it will initialise the variables
	- Apparently, exporting the env variables within `package.xml` is [buggy](https://github.com/gazebosim/ros_gz/issues/205) in Gazebo Fortress, so just use environment hooks, following answer in linked GitHub issue


## Topics within Gazebo
-   ~~Known [issue](https://github.com/gazebosim/gz-transport/issues/263) that Gazebo cannot list the active publishers~~ Check your YAML file and launch commands are correct
	- Try running straight from command line if not work:
	```bash
ros2 run ros_gz_bridge parameter_bridge  X3/gazebo/command/motor_speed@actuator_msgs/msg/Actuators]gz.msgs.Actuators
	```
- Make sure the message types between ROS 2 and Gazebo match this [table](https://github.com/gazebosim/ros_gz/blob/ros2/ros_gz_bridge/README.md#example-1a-ignition-transport-talker-and-ros-2-listener)
- When using with YAML and launch file, the topics only become visible in `ign topic -l` when play is pressed on the simulation 


## Resetting Pose
```bash
ign service -s /world/quadcopter_teleop/set_pose --reqtype ignition.msgs.Pose   --reptype ignition.msgs.Boolean   --req "name: 'viper', position: {x: 0, y: 0, z: 3}, orientation: {x: 0, y: 0, z: 0, w: 1}" --timeout 2000
```
where the `/world/quadcopter_teleop/set_pose` must be what world name is.

>[!Tip]
>Use `ign service -l | grep pose` to find correct string


# Running with Launch File
- Note that when the run with launch file, i.e. `viper-sim.py`, it isn't running from the dev `gazebo` directory
	- It gets copied to `install/viper/share` folder
	- Make sure to `colcon build` after any changes to update the `sdf` file used