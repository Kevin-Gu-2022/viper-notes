Implements the position node by communicating with MTF-01 flow sensor

# Setup
Follow this guide to flash CAN firmware using `arduino-cli`: https://github.com/107-systems/CyphalPicoBase-CAN-firmware/tree/main
- May need to set up some `udev` [rules](https://docs.platformio.org/en/latest/core/installation/udev-rules.html):
```bash
curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core/develop/platformio/assets/system/99-platformio-udev.rules | sudo tee /etc/udev/rules.d/99-platformio-udev.rules
```
- First time flashing, hold down the BOOTSEL button to put in flash mode
	- After that, can directly flash without holding down the button
# Notes
- LED_2 on GP21 is D5 (red) on board
	- Blinking signifies heartbeat of Cyphal
- LED_3 on GP22 is D4 (yellow) on board
	- Blinking signifies receive buffer full, i.e. receiving data
	- Brighter it is, higher the CAN traffic
- LED_1 (green) is the one on the Raspberry Pi Pico dev board
- Even though `y cmd <node_id> store` and `y cmd <node_id> restart` may return an error, it is still doing it correctly, and is needed for changes to be permanently saved to EEPROM

## Platform IO
- Just install extension and everything should just work
- Allows one-click build and flash, as well as `.pio` directory where you can see built binaries
	- I think it uses its own build system and not `arduino-cli`

# Changes
- Removed neopixel
- Removed servo