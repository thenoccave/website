# Centos 6 PXE Server

The following instructions detail how to setup a Centos 6 server to act as a PXE server allowing for installs over the network. In this case this server acts as a dedicated management server for our server environment so it acts as a DHCP server for the management network. If you are running a separate DHCP server then when it comes to configuring DHCP do it on your DHCP server.

- There was some weird conflict with the version of DHCP on the Centos 6 version that I installed so I had to upgrade it. You may not have to but it won’t hurt if you do anyway
```
[root@management ~]# yum update dhclient
```
- Next you need to install the DHCP server, tftp server and syslinux
```
[root@management ~]# yum install dhcp tftp-server syslinux
```
- Disable SELinux if you haven’t already
```
[root@management ~]# vim /etc/sysconfig/selinux
```
- Set the line SELINUX=enforcing to
```
SELINUX=disabled
```
- Disable SELinux immediately without a reboot
```
[root@management ~]# setenforce 0
```
- Create a directory to store your tftp files. I chose /tftpboot
```
[root@management ~]# mkdir /tftpboot
```
- Now copy the required syslinux files into your tftpboot directory. If you choose a directory other then /tftpboot then you will have to modify the copy destination to the directory you chose
```
[root@management ~]# cp /usr/share/syslinux/pxelinux.0 /tftpboot/
[root@management ~]# cp /usr/share/syslinux/menu.c32 /tftpboot/
[root@management ~]# cp /usr/share/syslinux/memdisk /tftpboot/
[root@management ~]# cp /usr/share/syslinux/mboot.c32 /tftpboot/
[root@management ~]# cp /usr/share/syslinux/chain.c32 /tftpboot/
```
- Create a directory to store your pxe config
```
[root@management ~]# mkdir /tftpboot/pxelinux.cfg
```
- Now put the required config in pxelinux.cfg
```
[root@management ~]# vim /tftpboot/pxelinux.cfg/default
```
- Add the following to the default file
```
default menu.c32
prompt 0
timeout 300
ONTIMEOUT local
 
MENU TITLE PXE Menu
```
- You now need to setup configure xinit for tftp
```
[root@management ~]# vim /etc/xinetd.d/tftp
```
- You need to update the following two lines
```
server_args = -s /tftpboot
disable = no
```
Set the server_args to your directory and set disable to no. My final config is
```
service tftp
{
        socket_type             = dgram
        protocol                = udp
        wait                    = yes
        user                    = root
        server                  = /usr/sbin/in.tftpd
        server_args             = -s /tftpboot
        disable                 = no
        per_source              = 11
        cps                     = 100 2
        flags                   = IPv4
}
```
- Restart xinit for your changes to take effect
```
[root@management ~]# /etc/init.d/xinetd restart
```
- Now setup the dhcp server
```
[root@management ~]# vim /etc/dhcp/dhcpd.conf
```
My DHCP config is below. If you already have DHCP configured you only need to copy the config above the subnet configuration. Change the IP of next-server to the IP of the PXE server
```
allow booting;
allow bootp;
option option-128 code 128 = string;
option option-129 code 129 = text;
next-server 192.168.100.10;
filename &quot;/pxelinux.0&quot;;
 
subnet 192.168.100.0 netmask 255.255.255.0{
        range 192.168.100.100 192.168.100.200;
        option routers                  192.168.100.1;  # default gateway
        option subnet-mask              255.255.255.0;
        option broadcast-address        192.168.100.255;
        option domain-name-servers      192.168.100.10;
        option domain-name              &quot;management.onemetric.local&quot;;
}
```
- Start dhcp and set it to start on boot
```
[root@management ~]# /etc/init.d/dhcpd start
[root@management ~]# chkconfig dhcpd on
```
