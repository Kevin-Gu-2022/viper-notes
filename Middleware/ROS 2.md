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