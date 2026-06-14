- Inertial measurement unit of Pika Spark is BNO085
	- Already performs sensor fusion
	- Runs proprietary `SH-2` firmware onboard
		- say it's not accurate, but I think it should be more than good enough for Viper considering its large size, i.e. higher inertia
- Driver uses the now deprecated `SysGPIO` functions. No longer part of Linux
	- if you use older versions of image from Arduino website, the ROS 2 IMU driver will still work, but the latest `pika-spark-ros-jazzy-image` will not run. The `/sys/class/gpio` directories don't even exist
	- Driver could be ported to use `gpiod`, ~~but latest image does not support `/dev/spi0.0`. Apparently Alex has fixed the device tree that includes this. Still waiting for new image. My Yocto image does not seem to work.~~
		- `/dev/spidev1.0` works now thanks to the device tree blob

> [!Note]
If you reflash Pika Spark, the device tree blob must be added again: 
`su`, then, `mv ov_carrier_pika_spark_base.dtbo /boot/devicetree/`. Then, reboot
# Changes to Driver
- Don't think you need to touch any of the pins, other than making sure the nBOOT pin is tied high
- `gpioset` does not keep pin down/up once returned, so need to keep running in background
```bash
gpioset -c gpiochip2 23=1 &  # Needs to be running in background
```
- nIRQ pin is definitely needed