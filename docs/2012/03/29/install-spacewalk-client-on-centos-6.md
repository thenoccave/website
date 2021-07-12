# Install Spacewalk Client on Centos 6

The following instructions are for configuring a Centos 6 box to talk to a Spacewalk server. Instructions on how to setup a Spacewalk server can be found at [https://www.thenoccave.com/2012/03/29/install-spacewalk-server-on-centos-6/](https://www.thenoccave.com/2012/03/29/install-spacewalk-server-on-centos-6/)

Install the Spacewalk Client repo

```
[root@spacewalkclient ~]# rpm -Uvh http://spacewalk.redhat.com/yum/1.7/RHEL/6/i386/spacewalk-client-repo-1.7-5.el6.noarch.rpm
```

Clean all the yum repos

```
[root@spacewalkclient ~]# yum clean all
```

Install the required packages

```
[root@spacewalkclient ~]# yum install rhn-client-tools rhn-check rhn-setup rhnsd m2crypto yum-rhn-plugin
```

Now you need to generate a activation key. The steps to do this are:

- Log into the Spacewalk web interface
- From the top navigation select Systems
- In the left navigation select Activation Keys
- In the top right hand corner click New Key
- Enter a name for this activation key and click Create Activation Key
- The activation key will then populate Note: This the text box doesn’t contain the complete key, the text before the textbox is part of the key. To see the complete key select Activation Keys from the left nav and it is shown in the list of keys.
- On the client run the configuration tool. Replace the IP and activation key with those from your setup. Once done your computer will be visible in the list of systems in spacewalk.

```
root@spacewalkclient ~]# rhnreg_ks --serverUrl=http://192.168.100.105/XMLRPC --activationkey=1-414ace93e6c3e94c85aec54a653e761d
```
