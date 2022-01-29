1) download sources and build linux kernel with debug symbols ...

clone linux kernel repository
```
$ git clone https://github.com/torvalds/linux.git
```

build kernel with debug symbols

```
make defconfig
echo 'CONFIG_DEBUG_INFO=y' >> .config
make -j$(grep -c ^processor /proc/cpuinfo)
```

2) mk initramfs

```
mkinitramfs -o ramdisk.img
```

3) Create image for ubuntu

```
qemu-img create -f qcow2 ubuntu.qcow 10G
```
4) download ubuntu/debian or each other and install

```
qemu-system-x86_64 \
    -hda ubuntu.qcow \
    -boot d -cdrom ./ubuntu-20.04.3-desktop-amd64.iso \
    -smp 4 \
    -m 1024 \
    -enable-kvm
```

5) run qemu and get root disk path and remember it for step 6

```
qemu-system-x86_64 -hda ubuntu.qcow -m 640 -enable-kvm
```

run this commands in vm:
```
$ df /
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda5        9736500 6632040   2590156  72% /
^^^^^^^^^
```

6) run qemu with custom kernel

```
qemu-system-x86_64 \
    -append "console=ttyS0 nokaslr root=/dev/sda5" \
    -kernel ../linux/arch/x86_64/boot/bzImage \
    -initrd ramdisk.img \
    -hda ubuntu.qcow \
    -enable-kvm \
    -nographic \
    -cpu host \
    -m 2048 \
    -smp 2
```

6) attach with gdb in another window:
```
gdb ../linux/vmlinux
(gdb) target remote :1234
(gdb) break do_sys_open
Breakpoint 1 at 0xffffffff811c4d20: file fs/open.c, line 1083.
...
```
