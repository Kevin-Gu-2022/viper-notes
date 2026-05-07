# Custom Yocto Image
## ROS 2 Image
- After flashing custom image with ROS 2 onto board, it seems like the only way to access the shell is via UART debug port. Follow this [guide](https://docs.lxrobotics.com/en/products/pika-spark/tutorials/serial-shell)
>[!Note] 
>New image does not seem to appear on `dmesg -wH`
>Also, no green light shown when connected to WiFi (this was case with default Arduino image)
- Use `screen /dev/ttyUSB0 115200` to view the shell
- Follow this [guide](https://docs.lxrobotics.com/en/products/pika-spark/tutorials/wifi) to get WiFi connected
- According to the official Arduino website, a USBC to USBC cable apparently is buggy. I tried multiple times before realising UART was working, so have not verified this

> [!Tip]
> Flashing blue light indicates successful boot

## Original Image
**Annoyingly, the custom image does not seem to expose CAN interface**. CAN is technically wired to the STM32, so additional drivers are needed, which I believe got removed somewhere along the way.

To return to original image by Arduino, follow this [guide](https://docs.arduino.cc/tutorials/portenta-x8/image-flashing/#:~:text=Update%20Using%20uuu%20Flashing%20Tool). Remember to flip the switches to ON side when flashing and then flip back when finished.


# Docker Containers
- If you want to preserve something after exiting a container, just do:
	```bash
docker start name_of_container # docker ps -a
docker exec -it name_of_container bash
	```
	If you `docker run`, this starts a NEW instance