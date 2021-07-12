# Configuring DynDNS on a Cisco Router
The following instructions detail how to configure a Cisco router to update DynDNS. I am going to assume you know why you need to do this and that you can work your way around the Cisco CLI


- Setup your domain name in your DynDNS control panel
- You need to be able to resolve the address members.dyndns.org to update your details, so you need to check your router has DNS setup. You can check this by pinging a host, if it can’t resolve then follow the commands below to add DNS servers otherwise skip this step

```
rt1(config)#ip domain lookup
rt1(config)#ip name-server 8.8.8.8
rt1(config)#ip name-server 4.4.4.4
 
rt1#ping google.com.au
 
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 74.125.237.151, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/18/28 ms
```

Next create a DDNS update method. I have named mine DynDNS however you can name yours anything

```
rt1(config)#ip ddns update method DynDNS
```

Cisco routers can either use DNS to do an update or HTTP. In this case we are going to use HTTP

```
rt1(DDNS-update-method)#http
```

Next you need to specify the URL to update DynDNS when an IP changes. The URL is

```
http://username:password@members.dyndns.org/nic/update?hostname=<h>&myip=<a>
```

You need to specify the URL to Add the IP and Remove the IP

```
rt1(DDNS-HTTP)#add http://username:password@members.dyndns.org/nic/update?hostname=&lt;h&gt;&amp;myip=&lt;a&gt;
rt1(DDNS-HTTP)#remove http://username:password@members.dyndns.org/nic/update?hostname=&lt;h&gt;&amp;myip=&lt;a&gt;
```

Replace the username and password with your DynDNS user account details. Notice the < h > and < a > they are meant to be there they are not place holders for you.

**Also Note the question mark in the URL. Cisco uses the question mark in the CLI so to enter the question mark in as a command you need to press Control-V then enter the question mark. Copying and pasting won’t get you around the issue, if you copy and past you will have to scroll back and enter it manually**

Next specify how often your router will update DynDNS. How long you set this to depends on your situation however I recommend setting it to a low value such as 1 minute while you are setting it up and debugging it. The digits represent Days, Hours, Minutes, Seconds

```
rt1(DDNS-HTTP)#exit
rt1(DDNS-update-method)#interval maximum 0 0 1 0
```

Now you need to go into the interface configuration for the interface which has the IP address that will be changing. Here you specify the name of the update method you created earlier and the domain name. In this case we are using an ADSL connection so we are using a Dialer interface.

```
rt1(config)#int Dialer 1
rt1(config-if)#ip ddns update DynDNS
rt1(config-if)#ip ddns update hostname onemetric.dyndns.org
```

To debug the DDNS enter the following commands

```
rt1#term mon
rt1#debug ip ddns update
```

You will see debugging information every time an update runs (hence why it is a good idea to set a low interval while debugging) A successful update looks like the following

```
Apr 24 11:18:59.133: DYNDNSUPD: Adding DNS mapping for onemetric.dyndns.org <=> 1.1.1.1
Apr 24 11:18:59.133: HTTPDNS: Update add called for onemetric.dyndns.org <=> 1.1.1.1
Apr 24 11:18:59.133: HTTPDNSUPD: Session ID = 0x9
Apr 24 11:18:59.133: HTTPDNSUPD: URL = 'http://username:password@members.dyndns.org/nic/update?hostname=onemetric.dyndns.org&myip=1.1.1.1'
Apr 24 11:18:59.133: HTTPDNSUPD: Sending request
Apr 24 11:18:59.692: HTTPDNSUPD: Response for update onemetric.dyndns.org <=> 1.1.1.1
Apr 24 11:18:59.696: HTTPDNSUPD: DATA START
good 1.1.1.1
Apr 24 11:18:59.696: HTTPDNSUPD: DATA END, Status is Response data recieved, successfully
Apr 24 11:18:59.696: HTTPDNSUPD: Call returned SUCCESS for update onemetric.dyndns.org <=> 1.1.1.1
Apr 24 11:18:59.696: DYNDNSUPD: Another update completed (outstanding=0, total=0)
Apr 24 11:18:59.696: HTTPDNSUPD: Clearing all session 9 info
```