# PXE Boot Memtest

You will need a PXE environment setup as shown at [Centos 6 PXE Server/](https://www.thenoccave.com/2012/03/28/centos-6-pxe-server)

Get the latest copy of Memtest Precompiled Bootable Image from [http://www.memtest.org/#downiso](http://www.memtest.org/#downiso)
```
[root@management tmp]# wget http://www.memtest.org/download/4.20/memtest86+-4.20.zip
```
Unzip the binary
```
[root@management tmp]# unzip memtest86+-4.20.zip
```

Move the binary into your tftp directory **NOTE Syslinux has some weired issues with file extensions. The unziped binary will have a bin extension, when you move it move it without the extension so it is just memtest not memtest.bin. If you are seeing garbage on the screen when attempting to boot into it that is why**
```
[root@management tmp]# mv memtest.bin /tftpboot/memtest
```

Edit your pxelinux config file to add the memtest option
```
[root@management tmp]# vim /tftpboot/pxelinux.cfg/default
```
Add the following option
```
LABEL Memtest
        MENU LABEL Memtest
        root (hd0,0)
        kernel memtest
```
Note: if change the memtest location on the kernel line if you haven’t placed the binary in the root of the tftp directory
My pxelinux config file currently looks like this:
```
default menu.c32
prompt 0
timeout 300
ONTIMEOUT local
 
MENU TITLE PXE Menu
 
LABEL Centos
        MENU LABEL Centos
        kernel centos/vmlinuz
        append noapic initrd=centos/initrd.img root=/dev/ram0 init=/linuxrc ramdisk_size=100000
 
LABEL Memtest
        MENU LABEL Memtest
        root (hd0,0)
        kernel memtest
```
When you do a pxe boot you will have an option call Memtest. Select it and press enter to boot into Memtest.
