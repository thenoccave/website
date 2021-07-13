# Install Centos 6 using PXE
You will need a PXE environment setup as shown at [Centos 6 PXE Server/](https://www.thenoccave.com/2012/03/28/centos-6-pxe-server/)

We will be doing the install over http

- Install httpd
```
[root@management tmp]# yum install httpd
```
- Start Apache and set it to start on boot
```
[root@management tmp]# /etc/init.d/httpd start
[root@management tmp]# chkconfig httpd on
```
- For this I am just using a default install of Apache. The install files will be available from http://serverip/centos/6/ We need to create the directories to put the install files
```
[root@management tmp]# mkdir /var/www/html/centos
[root@management tmp]# mkdir /var/www/html/centos/6/
```
- Now mount the Centos ISO so you can get the install files off it
```
[root@management tmp]# mkdir /mnt/centos
[root@management tmp]# mount -o loop /tmp/CentOS-6.0-x86_64-bin-DVD1.iso /mnt/centos/
```
- Now copy the install files and then umount the iso
```
[root@management tmp]# cp -r /mnt/centos/* /var/www/html/centos/6/
[root@management tmp]# umount /mnt/centos/
```
- Now create a directory in your tftp directory then copy vmlinuz and initrd.img to it
```
[root@management tmp]# mkdir /tftpboot/centos
[root@management tmp]# cp /var/www/html/centos/6/isolinux/vmlinuz /tftpboot/centos/
[root@management tmp]# cp /var/www/html/centos/6/isolinux/initrd.img /tftpboot/centos/
```
- Now edit the pxelinux.cfg menu to add Centos to it.
```
[root@management tmp]# vim /tftpboot/pxelinux.cfg/default
```
- Set the contents of the file to below. Update the kernel location and initrd location to reflect where you put your vmlinuz and initrd.img
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
```
- Now start a server that you want to install on. Boot from PXE, you will be prompted by a black screen with a blue box in the middle with Centos in it. Hit enter to start the install process
- The start of the install process is similar to a normal install. Select your country language and keyboard.
- When you get to the Installation Method screen select URL
- If you need to make changes to your network do it on the Network Setup screen. In most cases you can leave this as it is
- Enter the url to your install files on the next screen. In this case URL setup enter http://192.168.100.10/centos/6/
- Setup will then copy some files and start the install as if it was running from a CD/DVD, continue the rest of the install as normal.
