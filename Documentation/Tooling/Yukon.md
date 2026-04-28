# Setup
- Follow initial set-up (same as [[Yakut]])
- Don't necessarily have to setup SocketCAN on `can0`, but it is still useful
- Can just access the serial port directly using GUI
- [Video tutorial](https://www.youtube.com/watch?v=_nGi3y3FqvU)
- Viper uses MTU of 8 and 1Mbps data rate. Make sure these are set under CAN > SocketCAN for the transport, otherwise parsing issues may occur

---

# Use
- Click on lines then right click to subscribe/publish


>[! Warning]
>Datatypes in Yukon/Yakut use . instead of the underscores you see in the code
>E.g. `zubax::primitive::real16::Vector4_1_0` in Yukon/Yakut is `zubax.primitive.real16.Vector4.1.0`


> [!Tip]
> SocketCAN not SLCAN in Yukon menu for transport