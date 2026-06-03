# Docker
- This method is good for testing if code runs correctly on target architecture, i.e. arm64
- You'd need to compress image and `adb push` it then `docker load` on target machine
- Work with Docker with simulated arm64 via QEMU

Install emulator via Docker:
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