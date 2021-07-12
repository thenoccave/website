# Centos 6 Postfix Configure SPF Checking
The following instructions walk through getting Postfix to perform checking of SPF records on incoming mail. If a domain is properly configured it will have an SPF record and by configuring checking of SPF records Postfix won’t accept mail for the domain except from authorised servers.

I assume you already have postfix up and running how you need it

Install EPEL:
```
[root@mx1 ~]# rpm -Uvh http://mirror.as24220.net/pub/epel/6/i386/epel-release-6-8.noarch.rpm
```
Install the required Perl modules
```
[root@fs1 ~]# yum install perl-Mail-SPF -y
[root@mx1 ~]# yum install perl-Sys-Hostname-Long -y
```
Change to the tmp directory and download the latest version of policyd-spf-perl from [https://launchpad.net/postfix-policyd-spf-perl/](https://launchpad.net/postfix-policyd-spf-perl/)

```
[root@mx1 ~]# cd /tmp/
[root@mx1 tmp]# wget https://launchpad.net/postfix-policyd-spf-perl/trunk/release2.010/+download/postfix-policyd-spf-perl-2.010.tar.gz
```
Extract the downloaded tar
```
[root@mx1 tmp]# tar -zxvf postfix-policyd-spf-perl-2.010.tar.gz
postfix-policyd-spf-perl-2.010/
postfix-policyd-spf-perl-2.010/CHANGES
postfix-policyd-spf-perl-2.010/INSTALL
postfix-policyd-spf-perl-2.010/postfix-policyd-spf-perl
postfix-policyd-spf-perl-2.010/README
postfix-policyd-spf-perl-2.010/LICENSE
```

Create a new directory in /usr/lib/ and copy the postfix-policyd-spf-perl file across

```
[root@mx1 tmp]# mkdir /usr/lib/postfix
[root@mx1 tmp]# cp postfix-policyd-spf-perl-2.010/postfix-policyd-spf-perl /usr/lib/postfix/
```

Now test the module works ok. By running the following commmand you should be taken to a blank prompt, if not an error will be displayed that you will need to deal with

```
[root@fs1 tmp]# perl /usr/lib/postfix/postfix-policyd-spf-perl
```

Open the Postfix main.cf file in your text editor

```
[root@mx1 ~]# vim /etc/postfix/main.cf
```

You need to modify your smtpd_recipient_restrictions to include "check_policy_service unix:private/policy" as well as add the policy_time_limit option

```
smtpd_recipient_restrictions = permit_mynetworks,reject_unauth_destination,check_policy_service unix:private/policy
policy_time_limit = 3600s
```

Open the Postfix master.cf file in your text editor

```
[root@mx1 ~]# vim /etc/postfix/master.cf
```
Add the following line
```
policy    unix  -       n       n       -       0       spawn user=nobody argv=/usr/bin/perl /usr/lib/postfix/postfix-policyd-spf-perl
```

Restart Postfix for the changes to take effect

```
[root@mx1 tmp]# /etc/init.d/postfix restart
Shutting down postfix:                                     [  OK  ]
Starting postfix:                                          [  OK  ]
```

Now if you try send email to the server from an IP not listed in the SPF record you will be greated with

```
550 5.7.1 <Email Address>: Recipient address rejected:
```

If you have any issues look in maillog (/var/log/maillog.log)