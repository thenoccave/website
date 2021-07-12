# Nagios SNMP checks

When setting up Nagios we wanted to use SNMP checking of Linux devices. While there are included checks to do this the decision of using SNMP was to reduce the overhead of using SSH. These are the steps to get Nagios to do SNMP monitoring as well as a Linux host to give information over SNMP.

Note: We are still learning so there may be easier ways to do this

### Install and Configure SNMP on Linux

The first step is to install net-snmp on the host

```
[root@web1 tmp]# yum install net-snmp -y
```

Next step move the automatically generated configuration and create a new configuration file

```
[root@web1 tmp]# mv /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.bak
[root@web1 tmp]# vim /etc/snmp/snmpd.conf
```

Next add the following to the snmpd.conf file

```
rocommunity public 192.168.100.0/24
```

We are using public as the snmp community name and all monitoring will come from the 192.168.100.0 network. Modify the command to suit you

Next start snmpd and set it to start on boot

```
[root@web1 tmp]# service snmpd start
[root@web1 tmp]# chkconfig snmpd on
```

The server should now respond to SNMP requests. Note: SNMP is not a secure protocol, make sure your community name is secure and you are limiting where SNMP requests can come from

### Setup Nagios to do SNMP Checks
Make sure you have Nagios plugins installed. Note you need to have net-snmp installed before you install nagios-plugins for it to install the check_snmp plugin.

This guide assumes that nagios is installed in /usr/local/nagios/ please adjust for your installation. Please check that libexec/check_snmp exists. This is the command used to do the snmp checks.

#### CPU Load

Open etc/objects/commands.cfg in your text editor
```
[root@monitoring nagios]# vim etc/objects/commands.cfg
```

Define the command for snmp monitoring

```
define command{
        command_name    check_snmp
        command_line    $USER1$/check_snmp -H $HOSTADDRESS$ $ARG1$
}
```

We are going to use this to create 3 new checks. 1 Minute Average, 5 Minute Average and 15 Minute Average. Copy the following and past it below the check_snmp command

```
define command{
        command_name    snmp_1minute_load
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.10.1.3.1 -H $HOSTADDRESS$ $ARG1$
}
 
 
define command{
        command_name    snmp_5minute_load
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.10.1.3.2 -H $HOSTADDRESS$ $ARG1$
}
 
define command{
        command_name    snmp_15minute_load
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.10.1.3.3 -H $HOSTADDRESS$ $ARG1$
}
```

The above creates 3 new service checks using check_snmp to poll the appropriate OID’s for the load averages.

Now in your configuration for checks for a host you can create the following service checks

```
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     CPU 1 Minute Average
        check_command           snmp_1minute_load!-C public
}
 
 
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     CPU 5 Minute Average
        check_command           snmp_5minute_load!-C public
}
 
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     CPU 15 Minute Average
        check_command           snmp_15minute_load!-C public
}
```

Adjust the commands as required. The -C is used to specify the community name, in our case public. Change the community to suit your installation

Restart Nagios for the changes to take effect.

```
[root@monitoring nagios]# /etc/init.d/nagios restart
```

#### RAM/Swap Usage
Open etc/objects/commands.cfg in your text editor

```
[root@monitoring nagios]# vim etc/objects/commands.cfg
```

Find the command for snmp monitoring

```
define command{
        command_name    check_snmp
        command_line    $USER1$/check_snmp -H $HOSTADDRESS$ $ARG1$
}
```

We are going to create 4 checks RAM Total/Free and Swap Total/Free. Copy the following and past it below the check_snmp command

```
define command{
        command_name    snmp_SwapSize
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.4.3.0 -H $HOSTADDRESS$ $ARG1$
}
 
 
define command{
        command_name    snmp_SwapFree
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.4.4.0 -H $HOSTADDRESS$ $ARG1$
}
 
define command{
        command_name    snmp_RamSize
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.4.5.0 -H $HOSTADDRESS$ $ARG1$
}
 
define command{
        command_name    snmp_RamFree
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.4.11.0 -H $HOSTADDRESS$ $ARG1$
}
```

Now in your configuration for checks for a host you can create the following service checks

```
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     Swap Size
        check_command           snmp_SwapSize!-C public
}

define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     Swap Free
        check_command           snmp_SwapFree!-C public
}
 
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     RAM Size
        check_command           snmp_RamSize!-C public
}
 
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     RAM Free
        check_command           snmp_RamFree!-C public
}
```

Adjust the commands as required. The -C is used to specify the community name, in our case public. Change the community to suit your installation

Restart Nagios for the changes to take effect.

```
[root@monitoring nagios]# /etc/init.d/nagios restart
```

#### System Uptime
Open etc/objects/commands.cfg in your text editor

```
[root@monitoring nagios]# vim etc/objects/commands.cfg
```

Find the command for snmp monitoring

```
define command{
        command_name    check_snmp
        command_line    $USER1$/check_snmp -H $HOSTADDRESS$ $ARG1$
}
```

Copy the following and past it below the check_snmp command

```
define command{
        command_name    snmp_Uptime
        command_line    $USER1$/check_snmp -o .1.3.6.1.2.1.1.3.0 -H $HOSTADDRESS$ $ARG1$
}
```

Now in your configuration for checks for a host you can create the following service check

```
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     Uptime
        check_command           snmp_Uptime!-C public
}
```

Adjust the commands as required. The -C is used to specify the community name, in our case public. Change the community to suit your installation

Restart Nagios for the changes to take effect.
```
[root@monitoring nagios]# /etc/init.d/nagios restart
```

#### Monitor Disk Usage

Monitoring disk usage over SNMP is slightly more complicated. In your snmpd.conf you need to specify the disks that you want to be able to monitor.

On the host open snmpd.conf with your text editor

```
[root@web1 tmp]# vim /etc/snmp/snmpd.conf
```

Now we need to add the disks that we want to monitor. The command to use is disk <mount point>. In our case we want to monitor the root volume (/) and a volume mounted at /backups.

```
disk /
disk /backups
```

Add all the mount points that you want to monitor

Now we need to define a checks to check the mount point, disk size, disk usage and percentage free. Because we are checking 2 partitions we need to create a check for each, you will need to create as many checks as you have partitions that you want to check.

```
[root@monitoring nagios]# vim etc/objects/commands.cfg
 
#Get the mount point of the first disk
define command{
        command_name    snmp_Disk1_Mount
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.9.1.2.1 -H $HOSTADDRESS$ $ARG1$
}
 
#Get the size of the first disk
define command{
        command_name    snmp_Disk1_Size
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.9.1.6.1 -H $HOSTADDRESS$ $ARG1$
}
 
#Get the usage of the first disk
define command{
        command_name    snmp_Disk1_Usage
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.9.1.8.1 -H $HOSTADDRESS$ $ARG1$
}
 
#Get the usage as a percentage of the first disk
define command{
        command_name    snmp_Disk1_UsedPercentage
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.9.1.9.1 -H $HOSTADDRESS$ $ARG1$
}
 
#Get the mount point of the second disk
define command{
        command_name    snmp_Disk2_Mount
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.9.1.2.2 -H $HOSTADDRESS$ $ARG1$
}
 
#Get the size of the second disk
define command{
        command_name    snmp_Disk2_Size
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.9.1.6.2 -H $HOSTADDRESS$ $ARG1$
}
 
#Get the usage of the second disk
define command{
        command_name    snmp_Disk2_Usage
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.9.1.8.2 -H $HOSTADDRESS$ $ARG1$
}
 
#Get the usage as a percentage of the second disk
define command{
        command_name    snmp_Disk2_UsedPercentage
        command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.9.1.9.2 -H $HOSTADDRESS$ $ARG1$
}
```

The thing to know is that the last number of the OID is the index starting at 1 of the disk. So our first disk the OID ends in 1 and our second disk our OID ends in 2. If you need to add more partitions just update the command name and increment the number at the end of the OID

Now in your configuration for checks for a host you can create the following service checks

```
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     Disk 1 Mountpoint
        check_command           snmp_Disk1_Mount!-C public
}
 
 
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     Disk 2 Mountpoint
        check_command           snmp_Disk2_Mount!-C public
}
 
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     Disk 1 Size
        check_command           snmp_Disk1_Size!-C public
}
 
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     Disk 2 Size
        check_command           snmp_Disk2_Size!-C public
}
 
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     Disk 1 Usage
        check_command           snmp_Disk1_Usage!-C public
}
 
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     Disk 2 Usage
        check_command           snmp_Disk2_Usage!-C public
}
 
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     Disk 1 Usage Percentage
        check_command           snmp_Disk1_UsedPercentage!-C public
}
 
define service{
        use                     generic-service
        host_name               web1.onemetric.com.au
        service_description     Disk 2 Usage Percentage
        check_command           snmp_Disk2_UsedPercentage!-C public
}
```

Adjust the commands as required. The -C is used to specify the community name, in our case public. Change the community to suit your installation

Restart Nagios for the changes to take effect.

```
[root@monitoring nagios]# /etc/init.d/nagios restart
```