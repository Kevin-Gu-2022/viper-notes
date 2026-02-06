# Setup
Set-up same for Yakut and Yukon
- One time only
- Accessing Babel through `slcan0` allow use of other `can-utils` like `candump`
- `slacand` (Serial Line CAN Daemon) is a program that acts as a bridge between serial-based CAN adapters and the Linux SocketCAN networking stack

```bash
# First time only
sudo apt install can-utils

# Plug in Babel and determine serial port (ls /dev/ttyACM*)
sudo slcand -o -c -s8 /dev/ttyACM0 slcan0  # s8 is 1Mbps
sudo ip link set slcan0 up
```

## Initialise DSDL
First time only
```bash
mkdir -p ~/.cyphal      # Ensure the directory actually exists.

# Add all namespaces from the public regulated data types repository:
wget https://github.com/OpenCyphal/public_regulated_data_types/archive/refs/heads/master.zip -O dsdl.zip
unzip dsdl.zip -d ~/.cyphal
mv -f ~/.cyphal/public_regulated_data_types*/* ~/.cyphal
# There will be some garbage left in the destination directory, but it's mostly harmless.

# Add vendor-specific namespaces the same way, if you need any:
wget https://github.com/Zubax/zubax_dsdl/archive/refs/heads/master.zip -O dsdl.zip
unzip dsdl.zip -d ~/.cyphal
mv -f ~/.cyphal/zubax_dsdl*/* ~/.cyphal
```
---

## Setting up Environment Variables for Babel
Create a file called `env_slcan.sh`
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
 - Do not place the downloaded script in `~/.local/bin` as `sudo` can't read it. I suggest `/usr/local/bin`
 - *Source* `env_sclan.sh`. Do not execute it with `./` as this would not save the environment variables defined 
 - Don't forget to set path for `.cyphal`


# Operation
- Yakut/Yukon opens its own Cyphal node to communicate
- Storing of values using the registers will only update after the node is power cycled, storing configuration in its persistent memory
- if publisher not active, its ID will be 65535. Need to set it to something else for it to start publishing
- Yakut split into request/response on top and publisher/subscribers on bottom 