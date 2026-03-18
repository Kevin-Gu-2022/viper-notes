- Blue Controller MAC: 41427A8A33F3
- Lenovo MAC: 18:1D:EA:33:CD:26

- The PS3 controller provided is a clone that switches itself to a Xbox when it detects it is connected to a PC. Generally Xbox controllers have better support
## Wakeup Controller

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
- Stores one host MAC address in its EEPRROM and only connects to that one
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

# Interfacing with ROS2
- Dependencies:
```bash
sudo apt install ros-$ROS_DISTRO-joy ros-$ROS_DISTRO-teleop-twist-joy
```
- Two main nodes offered by `joy` package:
	- `game_controller_node`: Uses SDL2, which has an additional abstraction to map to typical controller outputs
	- `joy_node`: Raw data from joystick. Message order dependent on manufacturer


- Commands:
```bash
ros2 run joy joy_node
# Start another terminal
ros2 topic echo /joy_node
```