In the previous post, we have the toolchain `arm-unknown-linux-gnueabi` in hand. In this tutorial, it will be used to build a u-boot and run it using QEMU.

### Get the u-boot repository
```
	git clone https://source.denx.de/u-boot/u-boot.git`
	cd u-boot
```

To make use of the toolchain, you need to add the directory to your path using the following command:
```
	PATH=~/x-tools/arm-unknown-linux-gnueabi/bin:$PATH
```

### Build u-boot
You need to inform U-Boot of the prefix for your cross compiler by setting the `make` variable
**CROSS_COMPILE** and then select the configuration file using a command `make [board]_defconfig`. Here I use QEMU to run the u-boot so I use `qemu_arm_defconfig` board configuration.

```
	/* Clean everything before making the config for the new board*/
	make clean
	make mrproper
	make distclean

	make CROSS_COMPILE=arm-unknown-linux-gnueabi- ARCH=arm qemu_arm_defconfig
```

Note: There are a lot of board_xxx_deconfig files in `<PATH>/u-boot/configs`. You can take a look to find what is the board target that you need.

Then you can make change of the configuration file by using:
```
	make CROSS_COMPILE=arm-unknown-linux-gnueabi- ARCH=arm menuconfig
```
I will keep it as default in this example. 

Then you can build the u-boot for your target board
```
	make CROSS_COMPILE=arm-unknown-linux-gnueabi- ARCH=arm
```
	

After done, it will give you some different u-boot files for different purposes:
```
- u-boot : This is U-Boot in ELF object format, suitable for use with a debugger
- u-boot.map : This is the symbol table
- u-boot.bin : This is U-Boot in raw binary format, suitable for running on
- u-boot.img : This is u-boot.bin with a U-Boot header added, suitable for
- u-boot.srec : This is U-Boot in Motorola srec format, suitable for your device uploading to a running copy of U-Boot transferring over a serial connection

```

### Run the u-boot with QEMU
```
	qemu-system-arm -machine virt -nographic -bios u-boot.bin
```
Note: The guide for running u-boot can be found at `<PATH>/u-boot/doc/board/emulation`

You should get the console output of the u-boot as below:
```
U-Boot 2022.04-rc3 (Mar 04 2022 - 22:20:37 +0700)

DRAM:  128 MiB
Core:  42 devices, 11 uclasses, devicetree: board
Flash: 64 MiB
Loading Environment from Flash... *** Warning - bad CRC, using default environment

In:    pl011@9000000
Out:   pl011@9000000
Err:   pl011@9000000
Net:   eth0: virtio-net#32
Hit any key to stop autoboot:  0 
starting USB...
No working controllers found
USB is stopped. Please issue 'usb start' first.
scanning bus for devices...

Device 0: unknown device

Device 0: unknown device
starting USB...
No working controllers found
BOOTP broadcast 1
DHCP client bound to address 10.0.2.15 (2 ms)
Using virtio-net#32 device
TFTP from server 10.0.2.2; our IP address is 10.0.2.15
Filename 'boot.scr.uimg'.
Load address: 0x40200000
Loading: *
TFTP error: 'Access violation' (2)
Not retrying...
BOOTP broadcast 1
DHCP client bound to address 10.0.2.15 (0 ms)
Using virtio-net#32 device
TFTP from server 10.0.2.2; our IP address is 10.0.2.15
Filename 'boot.scr.uimg'.
Load address: 0x40400000
Loading: *
TFTP error: 'Access violation' (2)
Not retrying...
=> 

```

You can check your cross compiler by check `version` in the u-boot. Or `help` to see all u-boot supported commands.
```
=> version
U-Boot 2022.04-rc3 (Mar 04 2022 - 22:20:37 +0700)

arm-unknown-linux-gnueabi-gcc (crosstool-NG 1.24.0.551_1e47ca1) 11.2.0
GNU ld (crosstool-NG 1.24.0.551_1e47ca1) 2.38

```

At this point, you already have the u-boot in the board target and ready to boot the kernel.
Note: This steps seem to be easy since using the QEMU, it may require more knowleges to load a u-boot to the real board.
