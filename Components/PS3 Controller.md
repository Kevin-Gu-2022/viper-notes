- Stores one host MAC address in its EEPRROM and only connects to that one
- [Repo](https://github.com/user-none/sixaxispairer) to view and set Bluetooth address of controller

- Blue Controller MAC: 41427A8A33F3
- Lenovo MAC: 18:1D:EA:33:CD:26
## PS3 Controller Clone

- The PS3 controller provided is a clone that switches itself to a Xbox when it detects it is connected to a PC. Generally Xbox controllers have better support


### Wakeup Controller

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
- Use the following 2 scripts to read and set MAC addresses
- Not very robust. Still disconnects after a while, in which case just forget the device and reconnect via Bluetooth. I suspect it is a battery issue...


### `controller_read.py`

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