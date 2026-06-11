# Overview
- Serial communication with CAN high and low lines
- Bus topology so no masters. Also less wires
- Half duplex!
- Asynchronous
- Built in CRC
- Each message contains an ID + data similar to I2C
- Differential signal between 2 lines means no bit flips due to interference
	- Wires next to each other so interference affects both in same way
	- **Recessive (Logical 1):** Both wires are at approximately 2.5V (Differential voltage $\approx 0V$).
	- **Dominant (Logical 0):** CAN_H goes up (~3.5V) and CAN_L goes down (~1.5V).
	- A dominant bit always "wins" over a recessive bit on the bus.

# Usage
- Control CAN interface through `ip` command, part of `iproute2`
```bash
ip link set can0 type can bitrate 1000000 # Bitrate needs to be set first
ip link set can0 up/down # To turn on/off interface
```

To see **summary** of statistics:
```bash
ip -s link [show can0]
```

To see details about CAN bus:
```bash
# Detailed CAN statistics including error counters
ip -details -statistics link show can0
```

- Can set `restart-ms` to automatically bring interface back up after errors

> [!Warning]
>  Make sure to set TX buffer higher otherwise will run into errors when traffic is bursty, especially when sharing node info. Included in `docker-run.sh`
>  ```bash
>  sudo ip link set can0 txqueuelen 256
>  ```

- Unfortunately, buffer space still seems to run out after 40-ish minutes