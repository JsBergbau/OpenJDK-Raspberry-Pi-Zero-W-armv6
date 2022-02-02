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
Download the source code and extract it

You need a fitting crosscompiler to compile. There is currently none working so we have to build one ourselves. That is part of another article an Repo.


