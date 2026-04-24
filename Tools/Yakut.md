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
- Yakut/Yukon opens its own Cyphal node to communicate (
	- You'll be able to see this if you open both Yukon and yakut monitor at same time. Make sure parameters are the same, i.e., interface, MTU (size of frame), etc. However, it looks like Yukon will not display itself; Yakut will though
- Storing of values using the registers will only update after the node is power cycled, storing configuration in its persistent memory
- if publisher not active, its ID will be 65535. Need to set it to something else for it to start publishing
- Yakut split into request/response on top and publisher/subscribers on bottom 

# Yakut Monitor Output Interpretation

The `yakut monitor` command provides a real-time "top-like" view of the **OpenCyphal** (formerly UAVCAN) network.

## 1. Node Status (Header Section)

This section lists all active devices currently detected on the bus.

| **Column** | **Meaning**                                                  |
| ---------- | ------------------------------------------------------------ |
| **NodID**  | The unique Node ID (0–127 for CAN).                          |
| **Mode**   | `oper` (Operational) is the standard healthy state.          |
| **Health** | `nomina` (Nominal) indicates no internal errors.             |
| **VSSC**   | Vendor-Specific Status Code (usually `0`).                   |
| **Uptime** | Time since power-on (**Days:Hours:Minutes:Seconds**).        |
| **Name**   | The human-readable string identifying the hardware/software. |

**Current Nodes:**
- **63:** `org.opencyphal.yakut.monitor` (The monitor tool itself).
- **100:** `107-systems.viper` (Your target hardware device).
## 2. Message Traffic (MESSG Section)

This matrix displays **Subject IDs** (data topics) and their publication frequencies.

- **Rows (Left IDs):** These are the **Subject IDs** (e.g., 113, 114, 116).
    
- **Columns (63, 100):** Data mapped to specific Node IDs.
    
- **Cell Values:** Represent the frequency in **Hertz (Hz)**.
    
    - _Example:_ Node 100 is publishing Subjects 113, 114, and 116 at **100 Hz**.
        
    - _Example:_ `0` indicates the node knows the port but isn't currently sending/receiving.
        
### Standard Subject IDs

- **7509 / 7510:** Standard Cyphal Heartbeat and Port List messages. These are mandatory for a healthy node.
	- 7509 is the heartbeat. At least 1 Hz
	- 7510 is the port list. Publishes list of all subjects it is publishing/ (wishes to) subscribed to, as well as all RPC (remote procedure calls) services it is consuming/providing. Must publish at least once every 10 seconds

## 3. Request/Response (RQ+RS Section)

This section tracks **Service** interactions (Client-Server) rather than asynchronous messages.
- **IDs 384, 385, 430:** These are specific Service IDs (likely for configuration, diagnostics, or file transfer).
- **Values (0):** Currently, no requests or responses are active on these services.
    
## 4. Network Statistics (Totals Section)

The bottom section provides an aggregate health check of the bus.

|**Metric**|**Description**|
|---|---|
|**∑t/s**|Total Transfers per second. Node 100 is pushing **401 t/s**.|
|**∑B/s**|Total Bandwidth used. Node 100 is using approx **3 KB/s**.|
|**Transport Errors**|**Critical Metric.** Should stay at **0**. If this increases, check your $120 \Omega$ termination or wiring.|

## Tips for Interpretation

> [!TIP]
> 
> **Activity Indicators:** A `.` or blinking character indicates sporadic/low-frequency activity.
> 
> **The "Missing" Bytes:** The `TOTAL` bandwidth (3 KB/s) is higher than the `MESSG` bandwidth (2 KB/s) because it accounts for protocol overhead: CAN headers, tail bytes, CRCs, and padding.
