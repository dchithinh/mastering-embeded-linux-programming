Instead of archiving the rootfs as a .cpio format and used -initrd option to boot. There is another way to put rootfs into the target device by mounting it via NFS.

For this to work, your kernel has to be configured with CONFIG_ROOT_NFS.

## Install nfs server on your host and run it
Note: The below steps are applied for ubuntu 20.04
```
    sudo apt install nfs-kernel-server
```
The NFS server needs to be told which directories are being exported to the network which is controlled by /etc/exports. Add the following line to /etc/exports file:
```
    sudo vi /etc/exports
    <PATH>/rootfs *(rw,sync,no_subtree_check,no_root_squash)
```
Then, restart the nfs server
```
sudo /etc/init.d/nfs-kernel-server restart
```
#
## Use the below script for testing with QEMU
The jobs of this scripts can be described as follow:
- Create a virutal network interface on the host running NFS server.
- Add command to create a network interface on target board (QEMU board) and mount the rootfs from NFS server. (net nic -net tap,ifname=tap0,script=no)

The command for QEMU boot is almost the same with the previous except remove the -initrd option and rdinit=/bin/sh

**Note**: You should change the `<PATH>` to your rootfs, zImage and dtb corresponding to yours. Also, can change the `IP` if it is conflicted with your IP address in your PC.
```
#!/bin/bash

KERNEL=${HOME}/learn/linux/arch/arm/boot/zImage
DTB=${HOME}/learn/linux/arch/arm/boot/dts/vexpress-v2p-ca9.dtb
ROOTDIR=${HOME}/rootfs
HOST_IP=192.168.1.10
TARGET_IP=192.168.1.20
NET_NUMBER=192.168.1.0
NET_MASK=255.255.255.0

if [ ! -f ${KERNEL} ]; then
	echo "${KERNEL} does not exist"
	exit 1
fi
if [ ! -f ${DTB} ]; then
	echo "${DTB} does not exist"
	exit 1
fi

sudo tunctl -u $(whoami) -t tap0
sudo ifconfig tap0 ${HOST_IP}
sudo route add -net ${NET_NUMBER} netmask ${NET_MASK} dev tap0
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"

QEMU_AUDIO_DRV=none qemu-system-arm -m 256M -nographic \
    -M vexpress-a9 -kernel ${KERNEL} \
    -dtb ${DTB} \
    -append "console=ttyAMA0,115200 \
        root=/dev/nfs rw nfsroot=${HOST_IP}:${ROOTDIR},v3 ip=${TARGET_IP}" \
	-net nic -net tap,ifname=tap0,script=no
```

It should boot up as before, but now using the staging directory directly via the NFS export. Any files that you create in that directory will be immediately visible to the target device and any files created in the device will be visible to the development PC.
#
