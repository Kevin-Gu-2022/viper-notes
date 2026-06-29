# Docker Method
- This method is good for testing if code runs correctly on target architecture, i.e. arm64
- You'd need to compress image and `adb push` it then `docker load` on target machine
- Work with Docker with simulated arm64 via QEMU on host machine, which is typically x86_64

## Setup
Install emulator on host machine via Docker:
```bash
docker run --privileged --rm tonistiigi/binfmt --install all
```
- `docker buildx ls` to see supported architectures
- In Dockerfile, force correct platform: 
- ```Dockerfile
FROM --platform=linux/arm64 ros:humble-ros-base
  ```
  Or specify in command line: 
  ```bash
docker build --platform linux/arm64 -t your_image_name .
  ```

> [!Tip]
> Check architecture of current system using `uname -m`
- `amd64` and `x86_64` are same thing
- `aarch64` and `amd64` are equivalent

## Workflow Option 1 - Sending Entire Image
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

WORKDIR /tmp

RUN git clone https://github.com/gsl-lite/gsl-lite && cd gsl-lite && \
    mkdir build && cd build && \
    cmake .. && make -j8 && \
    sudo make install

RUN git clone https://github.com/catchorg/Catch2 && cd Catch2 && \
    mkdir build && cd build && \
    cmake .. && make -j8 && \
    sudo make install

RUN git clone https://github.com/fmtlib/fmt && cd fmt && \
    mkdir build && cd build && \
    cmake -DFMT_TEST=OFF .. && \
    make -j8 && \
    sudo make install

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

- Note that you'd be unable to run any CAN related stuff within the container. I suspect it's to do with the emulation layer messing stuff up


## Workflow Option 2 - Manually Compiling in Container and Sharing `install` Directory
This method requires QEMU to emulate the arm64 arcitecture. Install using this command:
```bash
docker run --privileged --rm tonistiigi/binfmt --install all
```

1. Build Docker image and run it and copy data into the Docker container described below
2. Compile in container using QEMU emulation
3. Copy out the `install` directory from ARM Docker container using `docker cp`
4. `adb push` the `install` directory directly to Pika Spark
	`docker cp install/ <container_name>:/tmp/colcon_ws/install`
>[!Tip]
>Adding `/` after the directory will copy the stuff within that directory. No `/` is just the entire directory including `install`

5. On Pika Spark, start the ROS Humble container from image
	`docker run -it --privileged --net=host arm64v8/ros:humble-ros-base`
6. Run node as normal after sourcing environments. 

Use the following minimal ROS 2 Humble container:
```Dockerfile
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

## Workflow Option 3 (EASIEST) - Compiling using Temporary Container

Even better would be to mount the volume to Docker when running the image for first time. This command starts a container, builds the program, then deletes the container.
```bash
# Must be run in root of ROS 2 workspace
docker run --rm \
    -v $PWD:/tmp/ws \
    ros-humble-aarch64 \
    bash -c "cd /tmp/ws && source /opt/ros/humble/setup.bash && colcon build [--packages-select package-name]"
```

After build, because volume is mounted, you can just push the install directory onto Pika Spark and copy into the ROS Humble Docker in it. Note that this will mess up any existing installs that use the x86 architecture. Probably a good idea to delete any existing `build` or `install` directories to avoid path issues.