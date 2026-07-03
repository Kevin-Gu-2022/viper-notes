There may be better and faster build options with using docker's buildx. Checkout this [guide](https://www.docker.com/blog/faster-multi-platform-builds-dockerfile-cross-compilation-guide/)
## Workflow Option 1 (EASIEST) - Compiling using Temporary Container
This method requires QEMU to emulate the arm64 arcitecture. Install using this command:
```bash
docker run --privileged --rm tonistiigi/binfmt --install all
```
- `docker buildx ls` to see supported architectures
- This command will need to be run with every reboot
> [!Tip]
> Check architecture of current system using `uname -m`
- `amd64` and `x86_64` are same thing
- `aarch64` and `amd64` are equivalent

1. Build Docker image for a minimal ROS 2 Humble container using Dockerfile described below:
```Dockerfile
# Will throw warning about hardcoded platform when building, can safely ignore
FROM --platform=linux/arm64 docker.io/arm64v8/ros:humble-ros-base

# Basic dev tools + colcon + ROS tools
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    python3-pip \
    python3-colcon-common-extensions \
    python3-rosdep \
    python3-vcstool \
    ros-humble-actuator-msgs \
    vim \
    can-utils \
    iproute2 \
    sudo \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/ws
```
- Force correct platform either through Dockerfile: 
- ```Dockerfile
FROM --platform=linux/arm64 ros:humble-ros-base
  ```
  Or in command line: 
  ```bash
docker build --platform linux/arm64 -t your_image_name .
  ```


2. Build:
```bash
sudo docker build -t ros-humble-aarch64 .
```

3. Mount the volume to Docker when running the image. This command starts a container, builds the program, then deletes the container.
```bash
# Must be run in root of ROS 2 workspace
docker run --rm \
    -v $PWD:/tmp/ws \
    ros-humble-aarch64 \
    bash -c "cd /tmp/ws && source /opt/ros/humble/setup.bash && colcon build [--packages-select package-name]"
```

4. After build, because volume is mounted, you can just push the install directory onto Pika Spark:
```bash
adb push install /home/pika
```
>[!Tip]
>Adding `/` after the directory will copy the stuff within that directory. No `/` is just the entire directory including `install`

> [!Note] 
> This may mess up any existing installs that use the x86 architecture. Probably a good idea to delete any existing `build` or `install` directories to avoid path issues.

5. If Pika Spark image has been reflashed, make sure it also has the same image on it:
```bash
# This saves the image as a file we can load onto Pika Spark
docker save ros-humble-aarch64 -o ros-humble-aarch64.tar  
# Load onto Pika Spark
sudo adb push ros-humble-aarch64.tar /home/pika

# Log into Pika Spark
adb shell
su  # Switch to root
# Load image
docker load -i ros-humble-aarch64.tar
# Start container for first time
docker run -it \  
	--name ros-humble-aarch64-container \  
	--net=host \  
	--privileged \  
	ros-humble-aarch64
```
6. Start the container if not already started: `docker start ros-humble-aarch64`
7. Copy the install directory into the container
```bash
docker cp install/ <container_name>:/tmp/colcon_ws/install
```
8. Attach to container and start the program
```bash
docker exec -it ros-humble-aarch64-container bash
# cd into the root of ROS 2 workspace
. /opt/ros/humble/setup.bash
. install/setup.bash
ros2 launch viper viper-quad.py
```

## Workflow Option 2 - Sending Entire Image
1. Setup a base Dockerfile for development, something like this:
```Dockerfile
# This Dockerfile is only the base dev workspace

# Will throw warning about hardcoded platform when building, just ignore
FROM --platform=linux/arm64 docker.io/arm64v8/ros:humble-ros-base

# Basic dev tools + colcon + ROS tools
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    python3-pip \
    python3-colcon-common-extensions \
    python3-rosdep \
    python3-vcstool \
    vim \
    can-utils \
    sudo \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /tmp/colcon_ws/src
WORKDIR /tmp/colcon_ws/src
RUN git clone -b canbus-test --recursive https://github.com/Kevin-Gu-2022/viper
```

2. Build it
```bash
sudo docker build -t <tag_name> .
```
- `tag_name`: name given to the image, e.g. `viper-dev-image`

3.  Run on host machine
```bash
sudo docker run -it \
  --name <container_name> \
  --network host \
  --privileged \
  <image_name> \
  bash
```
- `container_name`: The name of the container instance, e.g. `viper-dev-container`
- `image_name`: The name of the image you want to create container out of, e.g. `viper-dev-image`

4. Once inside, do your development, e.g building workspace, making changes etc.
5. When finished, exit the container using `exit` or Ctrl+D
6. Commit the container, i.e. save a local copy of current state as a new image
```bash
sudo docker commit <container_name> <new_image_name:tag>
```
- `container_name`: The name of the container for which you want the snapshot of, e.g. `viper-dev-container`
- `new_image_name:tag`: The name f the new image that we are creating, e.g. `viper-dev-image:v1`
	- Highly recommended to make use of the tag naming convention to keep track of images

7. Save the image to a file
```bash
sudo docker save <new_image_name:tag> -o <image_file_name.tar>
```
- `new_image_name:tag`: The name of the new image created in previous step. Make sure it is the same, e.g. `viper-dev-image:v1`
- `image_file_name.tar`: Name of the compressed image file, e.g. `viper-dev-image-v1.tar`

8. Push onto target device
```bash
sudo adb push <compressed_image_name.tar> /home/pika
```

9. Log onto Pika Spark and load the image
```bash
su  # To change to user with docker permissions
docker load -i <compressed_image_name.tar>
```

10. Run the container
```bash
docker run -it \  
--name viper-dev-container-v1 \  
--net=host \  
--privileged \  
viper-dev-image:v1
```
Make sure the tag matches what you gave it on the host computer.

> [!Note]
> The images you see in `docker images` are only the ones created directly from Dockerfile or from `docker commit`. They are ***templates*** for a container!!

- Note that you'd be unable to run any CAN related stuff within the container that is running on x86 architecture. I suspect it's to do with the emulation layer messing stuff up