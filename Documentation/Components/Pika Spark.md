- [Pika Spark](https://pika-spark.io/) is a small high-performance micro-robot control system with high amounts of computing power and interfaces. 
- Docs [here](https://docs.lxrobotics.com/)
- [Pika Spark Datasheet](https://datasheet.pika-spark.io/)
![[PikaSparkScreenshot.png|600]]

# Schematic
![[Pika-Spark-Rev-1-1-BNO085.pdf]]
# Arduino Portenta X8
- [Datasheet](https://docs.arduino.cc/resources/datasheets/ABX00049-datasheet.pdf) 
## Initial Setup
- Need to add `udev` rule to allow `adb shell` to read/write to Arduino Portenta
- Add `SUBSYSTEM=="usb", ATTRS{idVendor}=="2341", MODE="0666"` to `/etc/udev/rules.d/60-arduino.rules`. This basically gives read/write permissions to the Arduino Portenta X8 (the 2341 is the vendor ID you see when you type `lsusb`)

# Power
- In PoDL power class 12
- See description of what PoDL is in [datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/ltc9111.pdf) for LTC9111 used on Pika Spark

# Wifi & Bluetooth
- Uses the Murata 1DX chip. More info here:
 https://www.murata.com/en-GB/products/connectivitymodule/wi-fi-bluetooth/overview/lineup/type1dx
- 65 Mbps for WiFi and 3 Mbps for Bluetooth