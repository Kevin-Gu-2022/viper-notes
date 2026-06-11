- Blue Controller MAC: 41427A8A33F3
- Lenovo MAC: 18:1D:EA:33:CD:26

- The PS3 controller provided is a clone that switches itself to a Xbox when it detects it is connected to a PC. Generally Xbox controllers have better support
- Use the black controller with Ubuntu 22.04
## Wake-up Controller

>[!Note]
> Following section only needed in Ubuntu 20.04
- Not responsive until a sequence of codes are sent to wake it up

```python
import usb.core

# Find device (Xbox 260 ID 045e:028e)
dev = usb.core.find(idVendor=0x045e, idProduct=0x028e)

if dev is None:
	print("Controller not found. Check lsusb")
else:
	# Send initialisation packet
	dev.ctrl_transfer(0xc1, 0x01, 0x0100, 0x00, 0x14)
	print("Sent wake-up command! Check output of jstest /dev/input/js0")
```
- Dependency: `pyusb` which is wrapper for `libusb`
```bash
sudo apt update
sudo apt install libusb-dev
python3 -m pip install pyusb
# Or if it doesn't work
sudo apt install python3-usb
```
- - To automate this script every time controller is connected:
	- Move this script to `/usr/local/bin/fix_controller.py`
	- Add `udev` rule by adding the following to a new file called `99-xbox-clone-fix.rules` at `/etc/udev/rules.d`:

```
SUBSYTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="028e", ACTION=="add", RUN+="/usr/bin/python3 /usr/local/bin/flix_controller.py"
```

- Run the following as well after creating this:
```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```
- Also make sure the Python script has the necessary executing permissions
- Finally, reboot if still not working

## Connecting Controller
- Stores one host MAC address in its EEPROM and only connects to that one
- Because we are using a clone, there aren't really any existing tools out there to change the host MAC address stored
- So, use the following 2 scripts to read and set MAC addresses
- Make sure to also edit the config file to allow for un-bonded access 
- A successful connection = a solid red LED on Player 1
- Flashing LED means it is attempting to connect
### `controller_read.py`
- Dependency: `sudo apt install python3-usb`

```python
import usb.core
import usb.util

# PS3 Controller IDs (Common for originals and most knockoffs)
VENDOR_ID = 0x045E
PRODUCT_ID = 0x028E

def get_master_mac():
    # Find the device
    dev = usb.core.find(idVendor=VENDOR_ID, idProduct=PRODUCT_ID)

    if dev is None:
        print("Controller not found. Check the USB connection.")
        return

    # Detach kernel driver if Ubuntu is already using it
    if dev.is_kernel_driver_active(0):
        dev.detach_kernel_driver(0)

    try:
        # Request Feature Report 0xf5 (where the MAC is stored)
        # bmRequestType: 0xa1 (Interface, Class, Device-to-Host)
        # bRequest: 0x01 (GET_REPORT)
        # wValue: 0x03f5 (Feature Report 0xf5)
        msg = dev.ctrl_transfer(0xa1, 0x01, 0x03f5, 0, 8)
        
        # The Master MAC is usually in bytes [2:8]
        master_mac = ":".join([f"{b:02X}" for b in msg[2:8]])
        print(f"Stored Master MAC Address: {master_mac}")

    except Exception as e:
        print(f"Error reading from device: {e}")
    finally:
        # Reattach the driver so the OS can use it again
        usb.util.dispose_resources(dev)
        try:
            dev.attach_kernel_driver(0)
        except:
            pass

if __name__ == "__main__":
    get_master_mac()
```

### `set_controller.py`
```python
import usb.core
import usb.util

# Common PS3 Controller IDs
VENDOR_ID = 0x045E
PRODUCT_ID = 0x028E

def set_master_mac(new_master_mac_str):
    # Convert "AA:BB:CC:DD:EE:FF" to a list of bytes
    bytes_mac = [int(x, 16) for x in new_master_mac_str.split(':')]
    
    # The PS3 'Set Master' packet structure for Feature Report 0xf5:
    # [Report ID (0xf5), Padding (0x00), MAC_0, MAC_1, MAC_2, MAC_3, MAC_4, MAC_5]
    payload = [0xf5, 0x00] + bytes_mac

    dev = usb.core.find(idVendor=VENDOR_ID, idProduct=PRODUCT_ID)
    if dev is None:
        print("Controller not found.")
        return

    if dev.is_kernel_driver_active(0):
        dev.detach_kernel_driver(0)

    try:
        # bmRequestType: 0x21 (Interface, Class, Host-to-Device)
        # bRequest: 0x09 (SET_REPORT)
        # wValue: 0x03f5 (Feature Report 0xf5)
        dev.ctrl_transfer(0x21, 0x09, 0x03f5, 0, payload)
        print(f"Successfully wrote {new_master_mac_str} to controller!")
        print("Now unplug the USB and press the PS button to pair via Bluetooth.")
    except Exception as e:
        print(f"Failed to write MAC address: {e}")
    finally:
        usb.util.dispose_resources(dev)

if __name__ == "__main__":
    # REPLACE THIS with your PC's Bluetooth MAC
    my_pc_mac = "18:1D:EA:33:CD:26" 
    set_master_mac(my_pc_mac)
```

### Fixing Connection Issues
Edit the config file:

```bash
sudo vim /etc/bluetooth/input.conf
```  
    
- Find the line `#ClassicBondedOnly=true` (it might be commented out) and change to false.
- Restart PC
- This is needed because the PS3 controller doesn't implement typical Bluetooth bonding handshakes on subsequent connections

#### Black Controller
`dmesg -wH` output for a successful connection event.
```bash
[  +8.388488] hid-generic 0005:1949:0402.0013: unknown main item tag 0x0
[  +0.000292] input: Gamepad Consumer Control as /devices/pci0000:00/0000:00:14.0/usb1/1-7/1-7:1.0/bluetooth/hci0/hci0:256/0005:1949:0402.0013/input/input57
[  +0.000693] input: Gamepad as /devices/pci0000:00/0000:00:14.0/usb1/1-7/1-7:1.0/bluetooth/hci0/hci0:256/0005:1949:0402.0013/input/input58
[  +0.000992] hid-generic 0005:1949:0402.0013: input,hidraw5: BLUETOOTH HID v1.1b Gamepad [Gamepad] on 18:1d:ea:33:cd:26

```
#### Blue Controller
`dmesg -wH` output for a unsuccessful connection event.
```bash
[  +1.404045] playstation 0005:054C:09CC.0012: unknown main item tag 0x0
[  +0.004638] playstation 0005:054C:09CC.0012: hidraw5: BLUETOOTH HID v1.00 Gamepad [Wireless Controller] on 18:1d:ea:33:cd:26
[  +1.810268] playstation 0005:054C:09CC.0012: Failed to retrieve feature with reportID 163: -5
[  +0.000013] playstation 0005:054C:09CC.0012: Failed to retrieve DualShock4 firmware info: -5
[  +0.000003] playstation 0005:054C:09CC.0012: Failed to get firmware info from DualShock4
[  +0.000002] playstation 0005:054C:09CC.0012: Failed to create dualshock4.
[  +0.000355] playstation: probe of 0005:054C:09CC.0012 failed with error -5

```

Appears to be a problem with `hid-playstation` loading instead of the generic  driver. I've tried blacklisting `hid_playstation` but still doesn't work. Tried adding `udev` rule to load the `hid-generic` too, but doesn't work. 
# Interfacing with ROS2
## ROS 2 Setup
- Dependencies:
```bash
sudo apt install ros-$ROS_DISTRO-joy ros-$ROS_DISTRO-teleop-twist-joy
```
- Two main nodes offered by `joy` package:
	- `game_controller_node`: Uses SDL2, which has an additional abstraction to map to typical controller outputs
	- `joy_node`: Raw data from joystick. Message order dependent on manufacturer
- `joy` package gives the raw input, whereas the `teleop-twist-joy` gives the velocities

## `/joy` Topic
- Commands:
```bash
ros2 run joy joy_node
# Start another terminal
ros2 topic echo /joy_node
```
- Need to have the joy_node running for the velocity to read
- No need for remapping here
- `ros2 topic hz /joy` gives the frequency at which commands are published
	- Black controller uses a variable rate of between 15 to 20 Hz
	- When joystick used, publishing rate increases up to 23 Hz
## `/cmd_vel` Topic

Remap using yaml file. Only for `/cmd_vel`

```yaml
# Use /**: if want to use with launcher.py as Viper uses its own namespace
teleop_twist_joy_node:
  ros__parameters:
    # 1. Axis Mapping (Sticks)
    # Standard Mode 2: Left stick is Thrust/Yaw, Right stick is Pitch/Roll
    axis_linear:
      x: 3                # Pitch (Forward/Backward)
      y: 2                # Roll (Left/Right)
      z: 1                # Thrust (Up/Down) - usually a trigger or stick

    axis_angular:
      yaw: 0              # Yaw (Rotate Left/Right)

    # 2. Scaling (Sensitivity)
    # These values multiply the joystick input (-1 to 1) 
    # to reach your desired velocity in m/s or rad/s.
    scale_linear:
      x: 2.0              # Max pitch speed
      y: 2.0              # Max roll speed
      z: 1.5              # Max climb rate

    scale_angular:
      yaw: 1.2            # Max rotation speed

    # 3. Safety & Enable Buttons
    # The node will NOT publish commands unless this button is held (values zeroed out too)
    enable_button: 7      # e.g., Right Bumper (R1)
    require_enable_button: true

```
- This follows the **FLU coordinate system**, where x-axis points forward, y-axis to left, z-axis up. So, moving right joystick left will give positive y and vice versa for moving it to right
- Applying right hand rule, obvious that pushing left joystick right will give a negative yaw
- Don't forget to hold down dead-man's switch (configured to right bumper)

Commands:
```bash
# Take mapping from yaml file: /home/kevin/dev/viper_ws/src/viper/config
ros2 run teleop_twist_joy teleop_node --ros-args --params-file joystick_params.yaml
# Start new terminal
ros2 topic echo /cmd_vel
```
In the real thing, we'd use the `launch.py` script

Annoyingly, wired and wireless connections have different mappings

### [geometry_msgs](https://docs.ros.org/en/noetic/api/geometry_msgs/html/index-msg.html)/Twist Message
| Field     | Joystick Position         | Meaning                   |
| --------- | ------------------------- | ------------------------- |
| linear.x  | Right Joystick Front/Back | Strafe forward/backward   |
| linear.y  | Right Joystick Left/Right | Strafe left/right         |
| linear.z  | Left Joystick Front/Back  | Throttle to climb/descend |
| angular.x | ignored                   | unused                    |
| angular.y | ingnored                  | unused                    |
| angular.z | Left Joystick Left/Right  | turning CW/CCW            |
## Disconnection Events
From tests on Ubuntu 22.04 and ROS Humble, it appears that disconnecting controller (whether that be from loss of connection or inactivity) will just stop publishing messages on `/joy`. So, after a period of inactivity, the controller can simply be reactivated by pressing P3 button. This 're-registers' the node seamlessly without needing to restart whole embedded application.
