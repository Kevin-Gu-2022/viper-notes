- Real time visualisation software
- Easier if download through ROS, comes with the ROS plugins

# Dependency Issues
If on ROS 2 Humble and Ubuntu 22.04, you may run into a dependency issue with mismatched versions for a package.
```bash
kevin@kev-ubuntu:~$ sudo apt-get install ros-humble-ros-gz
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
comerr-dev : Depends: libcom-err2 (= 1.46.5-2ubuntu1.1) but 1.46.5-2ubuntu1.2 is to be installed
E: Unable to correct problems, you have held broken packages.
```

Downgrade the offending package (I'm pretty sure this is what I did. Of course, try the usual `sudo apt update && sudo apt upgrade` first)
```bash
sudo apt-get install libcom-err2=1.46.5-2ubuntu1.1
```

# Usage
- Launch with: 
- ```bash
  ros2 run plotjuggler plotjuggler
  ```
Alternatively, add a desktop icon to `~/.local/share/applications/plotjuggler.desktop:
```ini
[Desktop Entry]
Name=PlotJuggler
Comment=Real-time time series visualization tool
Exec=bash -c "source /opt/ros/humble/setup.bash && ros2 run plotjuggler plotjuggler"
Terminal=false
Type=Application
Categories=Utility;Application;
```
## Usage with Yakut
- Subscribe to the relevant topic ID, and pipe it directly to `netcat` over UDP
```bash
y sub 1000:zubax.telega.dq | nc -u localhost 9870
```
- Set PlotJuggler to stream from UDP server on port 9870