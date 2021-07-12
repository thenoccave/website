# Linksys SPA5xx Wifi Provisioning
Below is an example configuration to get WIFI working on a Linksys SPA5xx phone that supports wireless using auto provisioning.

We are including this in an existing configuration using a profile rule
```
<Profile_Rule_C>Location of .xml</Profile_Rule_C>
```
**NOTE: Wireless will appear configured but won’t work with the ethernet cable plugged in. I assume it is to prevent loops. I lost a lot of hair trying to work out why it wouldn’t connect**
```
<flat-profile>
<profileName_1_ group=”WirelessProfileEntry”>Descriptive Name<profileName_1_>
<ssid_1_ group=”WirelessProfileEntry”>SSID</ssid_1_>
<securityMode_1_ group=”WirelessProfileEntry”>WPA2-PSK</securityMode_1_>
<cipherType_1_ group=”WirelessProfileEntry”>TKIP</cipherType_1_>
<pskKey_1_ group=”WirelessProfileEntry”>Wifi Pass</pskKey_1_>
<SPA525-wifi-on group=”System/Wi-Fi_Settings”>Yes</SPA525-wifi-on>
</flat-profile>
```
If you aren’t using WPA2 then you can used WPA-PSK for the WirelessProfileEntry.