In this folder you find releases of OpenJDK for Raspberry PI Zero W / armv6hf.

A file name like `jdk-17-gcc-10.1_glibc-2.28_binutils-2.31_Buster.tar` means jdk-17-ga as JDK version, compiled with gcc 10.1, glibc 2.28, binutils-2.31 and Buster as sysroot.
This means it runs on Debian Buster and above.

With `ldd --version` you get the glibc version of your system. It is important that your system has a glibc version equal or above the JDK version.
I think binutils version is not really important, but just in case, I've added that information. You get the binutils version with `ld -v`. 

The GCC version should only be informational.
