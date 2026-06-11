# Environment Setup
```bash
git clone https://github.com/pika-spark/pika-spark-docker && cd pika-spark-docker
docker build -t yocto-nxp .
```
- **Builds Tools from Source:** Uses a 2-stage process to compile **Skopeo** (container management) and **ostreeuploader** (`fiopush`/`fiocheck`) using specific Go versions (1.18 and 1.19).
- **Prepares Yocto/BitBake:** Installs a suite of dependencies required to compile entire operating systems from scratch.
- **Configures User Access:** Creates a dedicated `builder` user and uses `gosu` to safely switch from root to that user, satisfying strict build tool requirements.
- **Integrates Cloud & Edge Tools:** Bakes in the Docker CLI, Docker Compose, and AWS CLI to handle container pre-loading and firmware deployment. The Docker CLI inside the container is a prerequisite for 'oe_builtin App preload'. According to Claude, `bitbake` uses this during the build process to parse and canonicalise the Compose YAML, then tools like `skopeo` ad `fiopush/fioecheck` handle the actual image pulling and embedding

Prepare folder structure and clone repo. 

```bash
mkdir yocto
cd yocto
YOCTO_DIR=$(pwd)
```
# Building Image

Clone required repositories. This is NXP's curated snapshot of Yocto with their own layers bundled too, i.e. `meta-imx`. `repo sync` pulls in the recipes that will be used to build.
```bash
cd $YOCTO_DIR
repo init -u https://github.com/nxp-imx/imx-manifest.git -m imx-6.6.52-2.2.0.xml -b imx-linux-scarthgap
repo sync -j1 --fail-fast
```

BSP: Board Support package. Collection of software that makes generic Linux actually run on target hardware. Includes stuff like device tree, kernel configs, etc.
> [! NOTE]
> freescale was the designer of the chips before being acquired by NXP

1st time running Docker image. NXP provides a shell script to set up build environment. Calls `setup-environment` internally, but aso sets up other NXP specific stuff beforehand like configuring MACHINE and DISTRO.
First command runs the docker image containing build toolchain built in previous step. Mounts YOCTO_DIR, names it `yocto-nxp-scarthgap` and starts bash in it. Terminal input will now be brought into the docker container
Second command runs the `imx-setup-release.sh` script, setting build directory to `bld-xwayland`
```bash
docker run -it -u $UID -v ${YOCTO_DIR}:/workdir -w /workdir --name yocto-nxp-scarthgap yocto-nxp bash
MACHINE=portenta-x8 DISTRO=fsl-imx-xwayland EULA=yes source ./imx-setup-release.sh -b bld-xwayland
```

> [! NOTE]
> Curly brakcets should be used for YOCTO_DIR, otherwise it will try to execute it

> [!Note]
> Poky is the reference embedded distribution and build system for the Yocto Poject. All the `meta-` layers build on top of this.

Cloning additional layers on top of Poky and NXP's layers
```bash
cd sources
# meta-arduino
git clone --depth 1 --branch scarthgap https://github.com/pika-spark/meta-arduino.git
# meta-pika-spark
git clone --depth 1 --branch scarthgap https://github.com/pika-spark/meta-pika-spark.git
# meta-ros
git clone --depth 1 --branch scarthgap https://github.com/ros/meta-ros

cd ../bld-xwayland  # the build directory
# Add the actual layers into bblayers.conf
echo 'BBLAYERS += "${BSPDIR}/sources/meta-arduino/meta-arduino-common"' >> conf/bblayers.conf
echo 'BBLAYERS += "${BSPDIR}/sources/meta-arduino/meta-arduino-nxp"' >> conf/bblayers.conf

echo 'BBLAYERS += "${BSPDIR}/sources/meta-pika-spark"' >> conf/bblayers.conf

echo 'BBLAYERS += "${BSPDIR}/sources/meta-ros/meta-ros-common"' >> conf/bblayers.conf
echo 'BBLAYERS += "${BSPDIR}/sources/meta-ros/meta-ros2"' >> conf/bblayers.conf
echo 'BBLAYERS += "${BSPDIR}/sources/meta-ros/meta-ros2-jazzy"' >> conf/bblayers.conf
```
> [!Note]
> echo BBLAYER is same as `bitbake add-layer`

Use this to check the layers
```bash
bitbake-layers show-layers
```

Build the image. Will take *forever*
```bash
bitbake <image e.g. pika-spark-base-image>  # Defined in a bb file
```


> [!Warning]
> Will need a minimum of 140GB in free disk space, even for the most basic image
> After building, you may delete `/tmp` in the build directory to recover disk space. Contains build artefacts, logs, the actual final kernel image, source code etc.

The `sstate-cache` contains the archived versions of the built binaries.

~~`vim` failed to build. Maybe an X11 [issue](https://community.nxp.com/t5/i-MX-Processors/NXP-Yocto-5-4-3-vim-build-error/m-p/1095029). Simply add:
```bash
PACKAGECONFIG_remove = "x11"
```
to `sources/poky/meta/recipes-support/vim/vim_8.1.1518.bb` or whatever version it is. ~~

Actually it's just that vim can't have GUI in it. In `local.conf`, add:
```bash
PACKAGECONFIG:remove:pn-vim = "gtkgui"
```
Then, do a 'make clean':
```bash
bitbake vim -c cleansstate
bitbake vim
```
>[!Tip]
> Use `bitbake -e vim | grep PACKAGECONFIG` to check if gui still there. 

Interestingly, X11 is still there after running the `grep` command:
```
PACKAGECONFIG=" acl x11 nls "
```
# Flashing
- Follow this [guide](https://docs.lxrobotics.com/en/products/pika-spark/tutorials/flash-yocto-image) to flash image: essentially, install `uuu`, then:
```bash
uuu -b emmc_all tmp/deploy/images/portenta-x8/pika-spark-base-image-portenta-x8.rootfs.wic.zst
```

- May need to set up `udev` rules if first time so that you can recognise the new device.
# Patching with `PREEMPT_RT`
- https://community.nxp.com/t5/i-MX-Processors-Knowledge-Base/i-MX-9-How-to-Use-the-Preempt-RT-Kernel-in-the-Standard-Yocto/ta-p/1956566?profile.language=en
- Too difficult, gave up lol
# Entering Docker Build Container
```bash
docker start yocto-nxp-scarthgap
docker exec -it yocto-nxp-scarthgap bash
source setup-environment bld-xwayland
```

# Current Image
- Using the `pika-spark-ros-jazzy-image` that Alex has built
- Need to load manually compiled device tree blob sent in Telegram into `/boot/devicetree` to activate `spidev` needed for IMU driver
- 
