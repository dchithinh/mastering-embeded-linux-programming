We use a crosstool-NG script to build a toolchain.
You will need a working native toolchain and build tools on your
host PC. To work with crosstool-NG on an Ubuntu host, you will need to install the
packages using the following command:

`sudo apt-get install automake bison chrpath flex g++ git gperf gawk libexpat1-dev libncurses5-dev libsdl1.2-dev libtool python2.7-dev texinfo`

### Get the crosstool-NG
`git clone https://github.com/crosstool-ng/crosstool-ng.git`

### Create the front-end menu system for crosstool-NG
```
cd crosstool-ng
./bootstrap
$ ./configure --enable-local
$ make
```
You will have the `ct-ng` execute file.

### Check which toolchain is supported 
`./ct-ng list-samples`
You will get something as follows. Find your needed toolchain and build it.
```
Status  Sample name
[L...]   aarch64-ol7u9-linux-gnu
[L...]   aarch64-rpi3-linux-gnu
[L...]   aarch64-rpi4-linux-gnu
[L..X]   aarch64-unknown-linux-android
[L...]   aarch64-unknown-linux-gnu
[L...]   aarch64-unknown-linux-uclibc
[L...]   alphaev56-unknown-linux-gnu
[L...]   alphaev67-unknown-linux-gnu
[L...]   arc-arc700-linux-uclibc
[L...]   arc-archs-linux-gnu
[L...]   arc-multilib-elf32
[L...]   arc-multilib-linux-gnu
```

In this tutorial, I will build a toolchain for ARM 32bit for QEMU. It is `arm-unknown-linux-gnueabi`

You can take a look the details this toolchain by command:
`./ct-ng show-arm-unknown-linux-gnueabi`

### Build toolchain for the target
#### Generate the configuratio for `arm-unknown-linux-gnueabi`
`./ct-ng arm-unknown-linux-gnueabi`

After this step, there is a **.config** file in the same directory which is configured for `arm-unknown-linux-gnueabi`

At this point, you can review the configuration and make changes using the configuration menu command `menuconfig` or edit `.config` file:
```
  ./ct-ng menuconfig
  or
  vi .config
```
In this, I will edit `.config` file.
- Use linux v4.10.17
- Disable all debug option in crosstool

by changing the below CONFIG option:
```
CT_LINUX_V_4_10=y
...
CT_LINUX_VERSION="4.10.17"
...
# CT_DEBUG_DUMA is not set
# CT_DEBUG_GDB is not set
# CT_DEBUG_LTRACE is not set
# CT_DEBUG_STRACE is not set
```



Then save and start building

### Build
`./ct-ng build`

You will see some logs as below:
```
...
[INFO ]  Installing final gcc compiler: done in 1272.69s (at 50:42)
[INFO ]  =================================================================
[INFO ]  Finalizing the toolchain's directory
[INFO ]    Stripping all toolchain executables
[EXTRA]    Installing the populate helper
[EXTRA]    Installing a cross-ldd helper
[EXTRA]    Creating toolchain aliases
[EXTRA]    Collect license information from: /home/dcthinh/learn/crosstool-ng/.build/arm-unknown-linux-gnueabi/src
[EXTRA]    Put the license information to: /home/dcthinh/x-tools/arm-unknown-linux-gnueabi/share/licenses
[INFO ]  Finalizing the toolchain's directory: done in 8.68s (at 50:51)
[INFO ]  Build completed at 20220304.210625
[INFO ]  (elapsed: 50:49.87)
[INFO ]  Finishing installation (may take a few seconds)...
...
```

The build will take about half an hour, after which you will find your toolchain is present in ~/x-tools/
For example:
```
ls -l x-tools/
...
drwxrwxr-x 8 dcthinh dcthinh 4096 Thg 3   1 21:29 aarch64-rpi3-linux-gnu
drwxrwxr-x 8 dcthinh dcthinh 4096 Thg 3   1 18:34 aarch64-unknown-linux-gnu
dr-xr-xr-x 8 dcthinh dcthinh 4096 Thg 3   4 21:06 arm-unknown-linux-gnueabi
```

`arm-unknown-linux-gnueabi` is the cross compiler for our board target.

At this point, we already have a cross compiler which can be used for compile u-boot, linux, dtb for the target board. I will share it in the next posts.

Note: The post is not intended for teching anything, it is a share what I've learnt. Please point out if you see something is incorrect.
