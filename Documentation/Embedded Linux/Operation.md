> [!Note] 
> Stuff here is mostly irrelevant to current image, but the notes here are still useful if encounter similar issues
# Custom Yocto Image
## ROS 2 Image
- After flashing my own custom image with ROS 2 onto board, it seems like the only way to access the shell is via UART debug port. Follow this [guide](https://docs.lxrobotics.com/en/products/pika-spark/tutorials/serial-shell)
>[!Note] 
>New image does not seem to appear on `dmesg -wH`
>Also, no green light shown when connected to WiFi (this was case with default Arduino image)
- Use `screen /dev/ttyUSB0 115200` to view the shell
- Follow this [guide](https://docs.lxrobotics.com/en/products/pika-spark/tutorials/wifi) to get WiFi connected
```bash
# List the available networks
nmcli dev wifi list
# Connect to WiFi
nmcli --ask dev wifi connect network-ssid
```
- According to the official Arduino website, a USBC to USBC cable apparently is buggy. I tried multiple times before realising UART was working, so have not verified this

> [!Tip]
> Flashing blue light indicates successful boot

## Original Image
**Annoyingly, the custom image does not seem to expose CAN interface**. CAN is technically wired to the STM32, so additional drivers are needed, which I believe got removed somewhere along the way.

>[!Warning]
Flashing custom Yocto builds does not use this method

To return to original image by Arduino, follow this [guide](https://docs.arduino.cc/tutorials/portenta-x8/image-flashing/#:~:text=Update%20Using%20uuu%20Flashing%20Tool). Remember to flip the switches to ON side when flashing and then flip back when finished.

TLDR: Extract the 2 `gz` compressed files: One is the flash tool `uuu` and the other is the image. Then, run:
```bash
cd mfgtool-files-portenta-x8
sudo uuu full_image.uuu
```

Multiple old versions available. OS version 746 seems to work: https://docs.arduino.cc/tutorials/portenta-x8/x8-firmware-release-notes/#:~:text=OS%20Image%20746
>[!Note]
> CAN bus was not working with the newer versions (maybe...) The Arduino Portenta might've just not been physically connected to board properly. Haven't had time to flash back to newer versions to test

### Arduino Image Versions
- Version 746 works both ways, but times out occasionally.
- Version 861: Cansend from Pika Spark works, but can't receive anything
- Latest Version 934: No sending or receive
- Version 881: Works both ways, but times out more often than Version 746
# Docker Containers
- If you want to re-enter container after exiting a container, just do:
	```bash
docker start name_of_container # docker ps -a
docker exec -it name_of_container bash
	```
	
>[!Warning]
If you `docker run`, this starts a NEW instance, and you lose all previous dev work within container

>[!Terminology]
>Container is the instance of the image. Image is the 'compiled' framework for how containers should operate; includes the multiple layers layered on top of OS that implements 'containerisation' concept
## Docker Container for Dev
Simple container with CAN tools and git
```bash
# Use the slim version to save space. Will be latest debian. 
# Alternatively: FROM docker.io/arm64v8/ros:humble-ros-base
FROM debian:stable-slim

# Install only the essentials without recommended "bloat"
RUN apt-get update && apt-get install -y --no-install-recommends \
    iproute2 \
    can-utils \
    git \
    kmod \
    ca-certificates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

CMD ["/bin/bash"]

```

Build image first, then `run` for first time. After that, do `docker start` & `docker exec -it container_name bash`
```bash
# Build image
docker build -t x8-slim-dev .

docker run -it \
    --privileged \
    --net=host \
    --name container_name \
    -v /home/shell/my_project:/app \
    x8-dev-container
```

Don't worry if encounter this error, just means you've just connected to WiFi and system time hasn't synced yet. Just run command again and it will work.
```bash
Sending build context to Docker daemon  6.144kB
Step 1/10 : FROM docker.io/arm64v8/ros:humble-ros-base
Get "https://registry-1.docker.io/v2/": x509: certificate has expired or is not yet valid: current time 2022-04-28T17:46:01Z is before 2025-09-24T00:00:00Z
```
### Exiting
- Docker container continues to run if used `exec`
	- To stop do `docker stop name_of_container`
	- This command essentially attaches another shell session to the container
- If `docker run`, Ctrl+D stops  the container, since the process with PID 1 (i.e. bash) ends
