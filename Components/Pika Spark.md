[Pika Spark](https://pika-spark.io/) is a small high-performance micro-robot control system with high amounts of computing power and interfaces. Docs [here](https://docs.lxrobotics.com/).
![[PikaSparkScreenshot.png]]

## Initial Setup
- Need to add `udev` rule to allow `adb shell` to read/write to Arduino Portenta
- Add `SUBSYSTEM=="usb", ATTRS{idVendor}=="2341", MODE="0666"` to `/etc/udev/rules.d/60-arduino.rules`. This basically gives read/write permissions to the Arduino Portenta X8 (the 2341 is the vendor ID you see when you type `lsusb`)