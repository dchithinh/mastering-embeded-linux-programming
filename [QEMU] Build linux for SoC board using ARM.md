In this tutorial, I will show you how to build a kernel with our toolchain.
**Important note: Please follow the linux branch as the guide to ensure successfully of kernel as well as the root filesystem which will be guided in the next post.**

### Get the linux source code
``` 
git clone https://github.com/torvalds/linux.git
```
In the configuration of cross compiler, we use the linux version of v4.10.17 so we should checkout to that linux branch to avoid errors.
```
cd linux
git checkout v4.10.17
```
### Make the configuration file for a target board
There are a set of known working configuration files in `<PATH>/arch/arm/configs`, each containing suitable configuration values for a single SoC or a group of SoCs. You can select one with `make configuration_file_name`.

For example, to configure Linux to run on a wide range of SoCs using the armv7-a architecture, you would type:
```
	make ARCH=arm CROSS_COMPILE=arm-unknown-linux-gnueabi- multi_v7_defconfig 
```

`multi_v7_defconfig` is a generic kernel that runs on various different boards. For a more specialized application, for example when using a vendor-supplied kernel, the default configuration file is part of the board support package; you will need to find out which one to use before you can build the kernel.

Again, you may want to change the default configuration in `.config` file by using the `menuconfig` or edit `.config` file.
```
	make ARCH=arm CROSS_COMPILE=arm-unknown-linux-gnueabi- menuconfig
```
I will keep it as default.

### Build linux with zImage output
```
	make ARCH=arm CROSS_COMPILE=arm-unknown-linux-gnueabi- zImage
```
Check the kernel version: `make kernelversion`

### Build dtbs
```
	make ARCH=arm CROSS_COMPILE=arm-unknown-linux-gnueabi- dtbs
```

After that, you already have zImage and dtb output in `<PATH>/arch/arm/boot/` directory.

### Run the zImage with `vexpress-a9` board as the following commands:
```
	QEMU_AUDIO_DRV=none qemu-system-arm -m 256M -nographic -M vexpress-a9 -kernel arch/arm/boot/zImage -dtb arch/arm/boot/dts/vexpress-v2p-ca9.dtb -append "console=ttyAMA0"
```

The console would end up with the  `Kernel panic` error because it can't find the `rootfs` which is the next jobs we have to build.
```
...
[    2.272285] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
[    2.272707] CPU: 0 PID: 1 Comm: swapper/0 Not tainted 4.10.17 #1
[    2.272888] Hardware name: ARM-Versatile Express
[    2.274024] [<c030fed4>] (unwind_backtrace) from [<c030b4a8>] (show_stack+0x10/0x14)
[    2.274337] [<c030b4a8>] (show_stack) from [<c05aff5c>] (dump_stack+0x84/0x98)
[    2.274559] [<c05aff5c>] (dump_stack) from [<c03d4214>] (panic+0xd8/0x264)
[    2.274861] [<c03d4214>] (panic) from [<c1201218>] (mount_block_root+0x1b8/0x250)
[    2.275114] [<c1201218>] (mount_block_root) from [<c1201624>] (prepare_namespace+0x180/0x1c4)
[    2.275351] [<c1201624>] (prepare_namespace) from [<c1200e04>] (kernel_init_freeable+0x1c8/0x1d8)
[    2.276585] [<c1200e04>] (kernel_init_freeable) from [<c0c3cd18>] (kernel_init+0x8/0x10c)
[    2.276756] [<c0c3cd18>] (kernel_init) from [<c0307978>] (ret_from_fork+0x14/0x3c)
[    2.277493] ---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)

...
```

