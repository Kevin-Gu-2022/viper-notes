- USB-CAN and UART-CAN adapter
- [Datasheet](https://files.zubax.com/products/com.zubax.babel/Zubax_Babel_Datasheet.pdf)
- Need SocketCAN to work. See setup with [[Yakut]]
- Basically, this device translates the binary of CAN into an ASCII based protocol called SLCAN. This protocol is used to transfer the CAN data to the PC. 
- On the PC, SLCAND converts it back to binary, exposing it via SocketCAN networking stack (accessible over `can0` network interface). Yakut then reads from this `can0` (if choose to use SocketCAN, with the benefit that other processes can read from this source simultaneously). 
## CLI
- Access using `screen /dev/ttyACM0 115200`
- No echo, so just start typing
- All commands end with a `\r\n` where newline is just an enter
- Communicates using same SLCAN interface that is sending over ASCII'd CAN data