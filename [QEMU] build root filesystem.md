You are stuck at "Kernel panic" in the previous note. Since kernel can't load the filesystem. We will get through this error in this tutorial.

### Staging directory
You should begin by creating a staging directory on your host computer where you can assemble the files that will eventually be transferred to the target. In the following examples, I have used `~/rootfs` . You need to create a skeleton directory structure in that, for example:
```
    mkdir ~/rootfs
    cd ~/rootfs
    mkdir bin dev etc home lib proc sbin sys tmp usr var
    mkdir usr/bin usr/lib usr/sbin
    mkdir var/log
```
Then use `tree -d` to show th results
```
    .
    ├── bin
    ├── dev
    ├── etc
    ├── home
    ├── lib
    ├── proc
    ├── sbin
    ├── sys
    ├── tmp
    ├── usr
    │   ├── bin
    │   ├── lib
    │   └── sbin
    └── var
        └── log
```
The first thing to build is busy box
### Get busybox source code
```
    git clone git://busybox.net/busybox.git
    cd busybox
```
### Build busybox
Clean everything first.
```
    make distclean
    make ARCH=arm CROSS_COMPILE=arm-unknown-linux-gnueabi- defconfig
    ...
```

Change the install directory which will be installed in your staging directory.
```
    vi .config
    ...
    /* Note: CONFIG_PREFIX=~/rootfs doesn't work*/
    CONFIG_PREFIX=${HOME}/rootfs 
    
```

Then build
```
    make ARCH=arm CROSS_COMPILE=arm-unknown-linux-gnueabi-
```
Copy the binary to the directory configured in CONFIG_PREFIX and
create all the symbolic links to it.
```
    make ARCH=arm CROSS_COMPILE=arm-unknown-linux-gnueabi- install
```

Check your staging directory to see if there are executable files of busybox in it. For example:
```
ls -l bin/
...
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 arch -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 ash -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 base32 -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 base64 -> busybox
-rwxr-xr-x 1 dcthinh dcthinh 1100904 Thg 3   5 16:17 busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 cat -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 chattr -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 chgrp -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 chmod -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 chown -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 conspy -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 cp -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 cpio -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 cttyhack -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 date -> busybox
lrwxrwxrwx 1 dcthinh dcthinh       7 Thg 3   5 16:17 dd -> busybox
...
```
The busybox need libraries to work properly. So you should copy the need libraries from sysroot directory in your toolchain to the your stagaing directory.

```
    cd ~/rootfs
    cp -a ${HOME}/x-tools/arm-unknown-linux-gnueabi/arm-unknown-linux-gnueabi/sysroot/lib/* lib

    ls -l lib/
    ...
    -r-xr-xr-x 1 dcthinh dcthinh  1233544 Thg 3   4 20:45 ld-linux.so.3
    -r-xr-xr-x 1 dcthinh dcthinh    11968 Thg 3   4 20:45 libanl.so.1
    -r--r--r-- 1 dcthinh dcthinh   545928 Thg 3   4 21:06 libatomic.a
    ...

```
### Create device node
You need just two nodes to boot with BusyBox: console and null.
- The console only needs to be accessible to root, the owner of the device node, so the access permissions are 600.
- The null device should be readable and writable by everyone, so the mode is 666.

You can use the -m option to `mknod` to set the mode when creating the node. You need to be root to create a device node:
```
    Note: If you uses docker to build, you need to enter root to make these 2 nodes by `docker exec -u root -it <container name> `
    cd ~/rootfs
    sudo mknod -m 666 dev/null c 1 3
    sudo mknod -m 600 dev/console c 5 1

    ls -l dev
    crw------- 1 root root 5, 1 Thg 3   5 16:40 console
    crw-rw-rw- 1 root root 1, 3 Thg 3   5 16:39 null
```
### The `proc` and `sysfs` filesystems
They both represent kernel data as files and provide another way to interact with device drivers and other kernel code.

`proc` and `sysfs` should be mounted on the directories `/proc` and `/sys`:
```
    mount -t proc proc /proc
    mount -t sysfs sysfs /sys
```

### Transfering the root filesystem to the target board
Having created a skeleton root filesystem in your staging directory, the next task is to transfer it to the target.
#### Standalone ramdisk
The following sequence of instructions creates the archive, compresses it and adds a U-Boot header ready for loading to the target:
```
    cd ~/rootfs
    find . | cpio -H newc -ov --owner root:root > ../initramfs.cpio
    cd ..
    gzip initramfs.cpio
    mkimage -A arm -O linux -T ramdisk -d initramfs.cpio.gz uRamdisk
```

### Boot the device
```
    cd linux

    QEMU_AUDIO_DRV=none qemu-system-arm -m 256M -nographic \
    -M vexpress-a9 -kernel arch/arm/boot/zImage \
    -dtb arch/arm/boot/dts/vexpress-v2p-ca9.dtb \
    -append "console=ttyAMA0 rdinit=/bin/sh" \
    -initrd /home/dcthinh/initramfs.cpio.gz

```

If all goes well, you will get a root shell prompt on the console.
```
[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 4.10.17 (dcthinh@dcthinh) (gcc version 11.2.0 (crosstool-NG 1.24.0.551_1e47ca1) ) #1 SMP Sun Mar 6 11:03:23 +07 2022
[    0.000000] CPU: ARMv7 Processor [410fc090] revision 0 (ARMv7), cr=10c5387d
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT nonaliasing instruction cache
[    0.000000] OF: fdt:Machine model: V2P-CA9
[    0.000000] efi: Getting EFI parameters from FDT:
[    0.000000] efi: UEFI not found.
[    0.000000] cma: Reserved 64 MiB at 0x6c000000
[    0.000000] Memory policy: Data cache writeback
[    0.000000] CPU: All CPU(s) started in SVC mode.
[    0.000000] percpu: Embedded 14 pages/cpu @cbd94000 s26188 r8192 d22964 u57344
[    0.000000] Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 65024
[    0.000000] Kernel command line: console=ttyAMA0 rdinit=/bin/sh
[    0.000000] PID hash table entries: 1024 (order: 0, 4096 bytes)
[    0.000000] Dentry cache hash table entries: 32768 (order: 5, 131072 bytes)
[    0.000000] Inode-cache hash table entries: 16384 (order: 4, 65536 bytes)
[    0.000000] Memory: 154452K/262144K available (10240K kernel code, 1140K rwdata, 3928K rodata, 2048K init, 534K bss, 42156K reserved, 65536K cma-reserved, 0K highmem)
[    0.000000] Virtual kernel memory layout:
[    0.000000]     vector  : 0xffff0000 - 0xffff1000   (   4 kB)
[    0.000000]     fixmap  : 0xffc00000 - 0xfff00000   (3072 kB)
[    0.000000]     vmalloc : 0xd0800000 - 0xff800000   ( 752 MB)
[    0.000000]     lowmem  : 0xc0000000 - 0xd0000000   ( 256 MB)
[    0.000000]     pkmap   : 0xbfe00000 - 0xc0000000   (   2 MB)
[    0.000000]     modules : 0xbf000000 - 0xbfe00000   (  14 MB)
[    0.000000]       .text : 0xc0208000 - 0xc0d00000   (11232 kB)
[    0.000000]       .init : 0xc1200000 - 0xc1400000   (2048 kB)
[    0.000000]       .data : 0xc1400000 - 0xc151d000   (1140 kB)
[    0.000000]        .bss : 0xc151e000 - 0xc15a3990   ( 535 kB)
...
...
[    3.452270] Key type dns_resolver registered
[    3.452588] ThumbEE CPU extension supported.
[    3.452740] Registering SWP/SWPB emulation handler
[    3.461470] rtc-pl031 10017000.rtc: setting system clock to 2022-03-06 06:02:28 UTC (1646546548)
[    3.465434] uart-pl011 10009000.uart: no DMA platform data
[    3.496453] Freeing unused kernel memory: 2048K
/ # 
/ # 
/ # 
/ # busybox
BusyBox v1.36.0.git (2022-03-06 11:07:42 +07) multi-call binary.
BusyBox is copyrighted by many authors between 1998-2015.
Licensed under GPLv2. See source distribution for detailed
copyright notices.

```
