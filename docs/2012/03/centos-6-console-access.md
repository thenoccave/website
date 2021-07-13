# Centos 6 Console Access
I recently had to deploy a new headless Centos 6 box, because is headless I set it up for access of RS323. Centos 6 makes this ridiculously easy. These are the steps:

- Open /boot/grub/grub.conf in your favorite text editor
- Append to the end of the kernel line console=ttyS2,115200n8
    - Note we are using serial port ttyS2 so you will need to change that to match your setup
    - We are outputting at 115200bps no parity. You can change that to match your application
- My kernel line ends up being:
```
kernel /boot/vmlinuz-2.6.32-131.0.15.el6.x86_64 ro root=UUID=7d12ae61-62230-4440-91ce-f08804f625f2 rd_NO_LUKS rd_NO_LVM rd_NO_MD rd_NO_DM LANG=en_US.UTF-- 8 SYSFONT=latarcyrheb-sun16 KEYBOARDTYPE=pc KEYTABLE=us crashkernel=auto console=tty0 console=ttyS2,115200n8
```
- Now grub just needs to be configured to output to serial. Open /boot/grub/grub.conf in a text editor
Before the OS declarations add
```
serial --unit=2 --speed=115200 --word=8 --parity=no --stop=1
terminal serial
```
- The unit represents the serial port number starting a 0 for com1
- Set the speed and parity as required