# Install Spacewalk Server on Centos 6

Spacewalk is Linux system management system. The following steps detail how to install it on Centos 6. Unfortunatly your database options are Postgres or Oracle so in this case we are using Postgres:

First install the spacewalk repo

```
[root@spacewalk tmp]# rpm -Uvh http://spacewalk.redhat.com/yum/latest/RHEL/6/x86_64/spacewalk-repo-1.7-5.el6.noarch.rpm
```

Next install the yum priorities plugin

```
[root@management tmp]# yum install yum-plugin-priorities
```

Next install the rest of the required repo’s

```
[root@spacewalk tmp]# rpm -Uvh http://mirrors.dotsrc.org/jpackage/5.0/generic/free/RPMS/jpackage-release-5-4.jpp5.noarch.rpm
[root@spacewalk tmp]# rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-5.noarch.rpm
[root@spacewalk tmp]# rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-5.noarch.rpm
[root@spacewalk ~]# rpm -ihv http://spacewalk.redhat.com/yum/1.7-client/RHEL/5/x86_64/spacewalk-client-repo-1.7-5.el5.noarch.rpm
```

Next you need to make a change to the jpackage repo

```
root@management tmp]# vim /etc/yum.repos.d/jpackage.repo
```

Look for the following section at the bottom

```
[jpackage-distro]
name=JPackage (free) for distro $releasever
mirrorlist=http://www.jpackage.org/mirrorlist.php?dist=redhat-el-$releasever&type=free&release=5.0
failovermethod=priority
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-jpackage
enabled=1
priority=10
```

You need to change the mirrorlist line so it looks like the following. The difference is the $releasever has been changed to 5.0

```
[jpackage-distro]
name=JPackage (free) for distro $releasever
mirrorlist=http://www.jpackage.org/mirrorlist.php?dist=redhat-el-5.0&type=free&release=5.0
failovermethod=priority
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-jpackage
enabled=1
priority=10
```

Run yum clean all

```
[root@management tmp]# yum clean all
```

Next install Postgres, set it to run on boot and run the initial configuration

```
[root@spacewalk ~]# yum install postgresql postgresql-contrib postgresql-devel postgresql-server
[root@spacewalk ~]# chkconfig postgresql on
[root@spacewalk ~]# service postgresql initdb
[root@spacewalk ~]# service postgresql start
```

Create the database and user for Spacewalk

```
[root@spacewalk ~]# su - postgres -c 'PGPASSWORD=spacepw; createdb spacedb ; createlang plpgsql spacedb ; yes $PGPASSWORD | createuser -P -sDR spaceuser'
```

Now you need to configure the user to use an md5 password

```
[root@spacewalk ~]# vim /var/lib/pgsql/data/pg_hba.conf
```

At the bottom of the config file you will see to following lines

```
# "local" is for Unix domain socket connections only
local   all         all                               ident
# IPv4 local connections:
host    all         all         127.0.0.1/32          ident
# IPv6 local connections:
host    all         all         ::1/128               ident
```

Before those lines add the following:

```
local spacedb spaceuser md5
host  spacedb spaceuser 127.0.0.1/8 md5
host  spacedb spaceuser ::1/128 md5
local spacedb postgres  ident
```

The end of the config file ends up looking like this:

```
local spacedb spaceuser md5
host  spacedb spaceuser 127.0.0.1/8 md5
host  spacedb spaceuser ::1/128 md5
local spacedb postgres  ident
 
# "local" is for Unix domain socket connections only
local   all         all                               ident
# IPv4 local connections:
host    all         all         127.0.0.1/32          ident
# IPv6 local connections:
host    all         all         ::1/128               ident
```

Reload Postgres for the changes to take effect

```
[root@spacewalk ~]# service postgresql reload
```

Now install Spacewalk. It will take some time on a minimal install it had to install over 400 dependencies.

```
[root@spacewalk ~]# yum install spacewalk-postgresql
```

Once it is installed run the Spacewalk setup

```
[root@spacewalk ~]# spacewalk-setup --disconnected
```

You will first be prompted for the database information. Leave the location blank, the database is spacedb, the username is spaceuser and the password is spacepw

```
** Database: Setting up database connection for PostgreSQL backend.
Hostname (leave empty for local)?
Database? spacedb
Username? spaceuser
Password?
```

The rest you can use the defaults. The final step will be generating a SSL certificate, just enter your details as required
Disable the firewall. Once you have everything up and running renable it. You will have to allow ports 443 and 5222 inbound

```
[root@spacewalk ~]# system-config-firewall-tui
```

Access the web interface by going to https://IPADDRESS you will be prompted to create a user. Once done you are ready to start connecting client machines.

You can now start connecting clients to your server. Instructions on how to setup Centos 6 can be found at [https://www.thenoccave.com/2012/03/29/install-spacewalk-client-on-centos-6/](https://www.thenoccave.com/2012/03/29/install-spacewalk-client-on-centos-6/)