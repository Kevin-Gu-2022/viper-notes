- USB-CAN and UART-CAN adapter
- [Datasheet](https://files.zubax.com/products/com.zubax.babel/Zubax_Babel_Datasheet.pdf)
- Need SocketCAN to work. See setup with [[Yakut]]
- Basically, this device translates the binary of CAN into an ASCII based protocol called SLCAN. This protocol is used to transfer the CAN data to the PC. 
- On the PC, SLCAND converts it back to binary, exposing it via SocketCAN networking stack (accessible over `can0` network interface). Yakut then reads from this `can0` (if choose to use SocketCAN, with the benefit that other processes can read from this source simultaneously). 

# Usage
If you are using Cyphal/CAN with an SLCAN-compatible USB-CAN adapter, such as [Zubax Babel](https://zubax.com/babel), paste the following snippet into a file named `env_slcan.sh`:
```bash
if ! [ -e /sys/class/net/slcan0 ]; then
    # Install this script to your PATH: https://gist.github.com/pavel-kirienko/32e395683e8b7f49e71413aebf5e1a89
    sudo setup_slcan -r /dev/serial/by-id/usb-Zubax*Babel*
fi
export UAVCAN__CAN__IFACE='socketcan:slcan0'
if [ -e /sys/class/net/slcan1 ]; then
    export UAVCAN__CAN__IFACE="$UAVCAN__CAN__IFACE socketcan:slcan1"
fi
export UAVCAN__CAN__MTU=8
export UAVCAN__CAN__BITRATE='1000000 1000000'   # Arbitration and data segment bit rates.
export UAVCAN__NODE__ID=$(yakut accommodate)    # Pick a node-ID for Yakut automatically.
echo "Auto-selected node-ID for this session: $UAVCAN__NODE__ID"
```
Then source the file into your shell session as follows:

```bash
. env_slcan.sh
```
- Sourcing, as opposed to execution, will make the current shell session ingest the environment variables defined in the file. Whenever you start a new shell session for use with Yakut, be sure to **always source this file** first.
- You can have several `env_*.sh` files for different network configurations that you use often. People familiar with languages like Python will find it somewhat analogous to virtual environments.
- If you get an error from Yakut that goes along the lines of the transport not being configured, that likely means that you forgot to source the environment file.
- Source the `.env_slcan.sh` to open Zubax interface

[Source](https://telega.zubax.com/tutorials/yakut.html): Yakut tutorial from Telega Zubax. Very comprehensive.
# Babel CLI
- Access using `screen /dev/ttyACM0 115200`
- No echo, so just start typing
- All commands end with a `\r\n` where newline is just an enter
- Communicates using same SLCAN interface that is sending over ASCII'd CAN data
