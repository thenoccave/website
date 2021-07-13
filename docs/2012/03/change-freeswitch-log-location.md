# Change Freeswitch Log Location
By default Freeswitch logs to the log file log/freeswitch.log under the Freeswitch directory (e.g. /usr/local/freeswitch/log/freeswitch.log). The log parameters are controlled by the mod_logfile module. I wanted to move the log file to be with all the other logs in /var/log/ these are the steps:

- Create a directory to store the freeswitch logs. We create a directory so we can assign freeswitch the owner and log rotation will work
```
[root@voip ~]# mkdir /var/log/freeswitch
[root@voip ~]# chown freeswitch:freeswitch /var/log/freeswitch
```
- Create the new log file
```
[root@voip conf]# touch /var/log/freeswitch/freeswitch.log
```
- Set the permission on the log file to be owned by the user running Freeswitch. In this case freeswitch
```
[root@voip conf]# chown freeswitch:freeswitch /var/log/freeswitch/freeswitch.log
```
- Now edit the logfile.conf file to point to where you want to save it. The log file is in conf/autoload_configs/logfile.conf.xml. The line to change is
```
<param name="logfile" value="/var/log/freeswitch/freeswitch.log"/>
```
By default this line is commented out so uncomment it and change the value to the location of the new log file
- Reload mod_logfile
```
freeswitch@internal> reload mod_logfile
```

### Further Reading and References:
- [mod_logfile](http://wiki.freeswitch.org/wiki/Mod_logfile)