- Stores one host MAC address in its EEPRROM and only connects to that one
- [Repo](https://github.com/user-none/sixaxispairer) to view and set Bluetooth address of controller
### PS3 Controller Clone
>[!Note]
> Only needed in Ubuntu 20.04
- The PS3 controller provided is a clone that switches itself to a Xbox when it detects it is connected to a PC. Generally Xbox controllers have better support
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