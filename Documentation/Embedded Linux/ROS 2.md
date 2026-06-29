# Start Workspace
- Create a new directory that contains the workspace, then in it, add the `src` directory:
```bash
mkdir -p ~/ros2_ws/src && cd ~/ros2_ws/src
git clone <repo_url>
```

- Install relevant dependencies depending on what `README.md` says
- Build the project
```bash
cd ~/ros2_ws
colcon build --packages-select <package_name>
```

- Every time start new terminal, the following must be sourced:
```bash
# The main ROS 2 installation (can be added to .bashrc)
source /opt/ros/$ROS_DISTRO/setup.bash
# The local workspace
source install/setup.bash
```

- Now that the environment is set up, you can execute your code.
	- **To run a single node:** `ros2 run <package_name> <executable_name>`
	- **To run a launch file (multiple nodes):** `ros2 launch <package_name> <launch_file_name>`

# QoS
#### Liveliness
- Tracks 'health' of node, i.e. is node still there?
- Normally just done by the DDS ROS 2 middleware
- LIVELINESS_MANUAL_BY_TOPIC - The signal that establishes a Topic is alive is at the Topic level. Only publishing a message on the Topic or an explicit signal from the application to assert liveliness on the Topic will mark the Topic as being alive.
- Publisher an be explicit by calling the assert_liveliness operations, or implicit by writing some data
- lease time is the maximum time before node declared dead
- When 0, it is infinite
#### Deadline
- Threshold for the time between messages
- A deadline time of 0 will disable the deadline tracking

More info [here](https://design.ros2.org/articles/qos_deadline_liveliness_lifespan.html)

## Parameters
- ROS 2 supports adding multiple parameters into node, allowing for runtime configuration of certain parameters
- Via CLI: `ros2 param set /node_name parameter_name value`
- Or better through GUI: `rqt` directly into terminal
- In Humble, best way is to subscribe to `/parameter_events` and filter for relevant parameter changes. No `add_post_set_parameter_callback` which actually changes the values.
	- Tried to change the values using `add_on_set_parameter_callback` but doesn't appear to work
- https://github.com/ros2/ros2_documentation/issues/2979


# package.xml
Make sure to include this line, otherwise, package won't be found
```xml
<export>
  <build_type>ament_cmake</build_type>
</export>
```