# OpenJDK RaspberryPi Zero W armv6hf
In this repository you find OpenJDK builds for Raspberry PI Zero W / armv6hf architecture and also a tutorial how to build your own OpenJDK release in case you need that.

Raspberry PI Zero and Zero W use BCM2835 SoC. Also original Raspberry PI uses this SoC. CPU architecture is 32 Bit ARMv6hf which means hardfloat support. 

When trying to install Java from the official repository latest version is Java 11 and when trying to execute you get an error

```
sudo apt install default-jre

pi@raspberrypi:~ $ java
Error occurred during initialization of VM
Server VM is only supported on ARMv7+ VFP
```

## Azul JDK

At first place have a look at https://www.azul.com/downloads/?package=jdk

They provide a working JDK 11 for Raspberry PI Zero W.

Make sure to select ARM 32-bit HF and then have a look at the Architecture column. There must be "v6" present otherwise the package won't run on the Raspberry PI Zero W.

Current (February 2022) there is no JDK17 for armv6.

## Build the JDK on your own

You can build your JDK on your own. Go to releases on the official repository https://github.com/openjdk/jdk/tags and select your desired version. For this repo I've chosen https://github.com/openjdk/jdk/releases/tag/jdk-17-ga 
Download the source code and extract it.

We use Debian to build the JDK. This tutorial is based on https://github.com/openjdk/jdk17/blob/master/doc/building.md#cross-compiling-with-debian-sysroots

### Crosscompiler for Raspberry PI Zero W

You need a fitting crosscompiler to compile. There is currently none working so we have to build one ourselves. That is part of another article an Repo. You find the build instructions as well a ready to go download there https://github.com/JsBergbau/GCC-cross-compiler-for-Raspberry-PI-Zero-W

The path of the cross compiler matters. It has to be put into `/opt/cross-pi-gcc/`

### System requirements for building OpenJDK for armv6

You need at least about 7 GB free diskspace. With packing files and stuff like that I recommend at least 10 GB free space, better 15 GB. A fast SSD is very helpful. A VirtualBox Debian is quite useful for compiling. 

Memory, the more the better. I think it should be at least 4 GB, I've configured the VM with 15 GB, because there is enough RAM in my machine.

There are some prequisites required. Please install them via:
`sudo apt install qemu-user-static debootstrap autoconf unzip zip libz-dev openjdk-17-jdkjava`

### Building OpenJDK on Debian x64 bullseye

`cd` to your home directory (or any other you like). In the following we use the home directory for the builds.

```
sudo qemu-debootstrap \
  --arch=armhf \
  --verbose \
  --include=fakeroot,symlinks,build-essential,libx11-dev,libxext-dev,libxrender-dev,libxrandr-dev,libxtst-dev,libxt-dev,libcups2-dev,libfontconfig1-dev,libasound2-dev,libfreetype6-dev,libpng-dev,libffi-dev \
  --resolve-deps \
  buster \
  ~/sysroot-armhf \
  http://httpredir.debian.org/debian/
```
We use buster here as sysroot and not bullseye. This JDK build will also run on Debian 11 / Bullseye. The other way round could be problematic.

Targetsystem where OpenJDK will be running:
```
uname -a
Linux raspberrypi 5.10.92+ #1514 Mon Jan 17 17:35:21 GMT 2022 armv6l GNU/Linux`
lsb_release -a
No LSB modules are available.
Distributor ID: Raspbian
Description:    Raspbian GNU/Linux 11 (bullseye)
Release:        11
Codename:       bullseye
```

Now lets make the crosscompiler 
`export PATH=/opt/cross-pi-gcc/bin:/opt/cross-pi-gcc/libexec/gcc/arm-linux-gnueabihf/10.1.0:$PATH`

```
cd ~
wget https://github.com/openjdk/jdk/archive/refs/tags/jdk-17-ga.tar.gz
tar xfv jdk-17-ga.tar.gz
cd jdk-jdk-17-ga

sh ./configure --openjdk-target=arm-linux-gnueabihf --with-sysroot=/home/pi/sysroot-armhf --with-abi-profile=armv6-vfp-hflt --with-num-cores=12 --with-memory-size=14000 --with-native-debug-symbols=zipped --with-jvm-variants=client
make images
```

The last step takes a while. I think about 7 to 15 minutes (didn't measure the time) on a Ryzen 5 Pro 5650GE.

Lets explain the options
* `openjdk-target=arm-linux-gnueabihf` for Raspberry PI Zero W which is armv6hf
* `--with-sysroot=/home/pi/sysroot-armhf` There are the header files for the target system
* `--with-abi-profile=armv6-vfp-hflt` for Raspberry PI Zero W which is armv6hf
* `-with-num-cores=12` This is just to speed up build process. The more cores, the more memory you need 
* `--with-memory-size=14000` maximum memory size that should be used for the build process
* `--with-native-debug-symbols=zipped` in case you need Debug symbols. I've removed them in the download file.
* `--with-jvm-variants=client` That is very important. Without this you will get ```Error occurred during initialization of VM
Server VM is only supported on ARMv7+ VFP```

Your JDK is now in `~/jdk-jdk-17-ga/build/linux-arm-client-release/images/jdk`
If you don't need debug information use 
* `find . -iname '*.diz' -delete` to delete all debug information files
* `find . -exec strip --strip-debug {} +` to remove debugging information in executable files, if there are. Prints a lot of errors. If someone has a better version please open a pullrequest or issue.

You can now pack your distribution `tar cfz jdk.tar.gz jdk`. With this method executable bits are preserved.

Distribute VM to your target system and have fun. 

I used this VM to run https://github.com/AsamK/signal-cli on a Raspberry PI Zero W and it works. You have to install rust on Raspberry PI and compile the native signal-client library on the Raspberry PI Zero W. This took about 193 minutes, about 3,25 hours. But after that it works. For more details of how to build, see https://github.com/AsamK/signal-cli/wiki/Provide-native-lib-for-libsignal







