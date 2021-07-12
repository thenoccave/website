# Linksys SPA932 XML Configuration
Below is an example configuration of how to configure a Linksys SPA 932 sidecar using XML.

Key 1 is configured with BLF, Speed Dial and Call Pickup. Key 2 is an external number with just Speed Dial

```
<flat-profile>
<Subscribe_Expires ua=”na”>1800</Subscribe_Expires>
<Subscribe_Retry_Interval ua=”na”>30</Subscribe_Retry_Interval>
<Unit_1_Enable ua=”na”>Yes</Unit_1_Enable>
<Subscribe_Delay ua=”na”>1</Subscribe_Delay>
<Server_Type ua=”na”>Asterisk</Server_Type>
<Unit_1_Key_1 ua=”na”>fnc=blf+sd+cp;sub=<extNo>@<proxy>;nme=<name></Unit_1_Key_1>
<Unit_1_Key_2 ua=”na”>fnc=sd;sub=<phone>@<proxy>;nme=<name></Unit_1_Key_2>
</flat-profile>
```

Include the configuration in your existing SPA phones configuration using a Profile Rule:
<Profile_Rule_B>Location of sidecar XML file</Profile_Rule_B>