# Freeswitch, Linksys SPAs and Anonymous Calls

We have several customers (ourselves included) using Linksys SPA VOIP Phones. The only major issue with these phones is how they deal with Anonymous calls. When an anonymous call comes through even if a Caller Name is set they won’t show it and instead just display Anonymous Call. We often use the Caller Name to display what queue the call is coming from or in one case a Virtual Receptionist setup what customer the callee has called.

On the newer firmwares of the SPA5xx there is an option to disable this. In the web interface in the Admin Advanced mode select the SIP tab. Change the “Display Anonymous From Header” from No to Yes.

However we have a lot of SPA9xx deployed and a lot of SPA5xx not running the latest firmware so we created the following fix. Create the following extension at the top of your dialplan. It works by detecting a caller id number of anonymous and replacing it with Unknown. The SPAs will then display the correct Caller Name.

```
<extension name="anonymous-fix" continue="true">
       <condition field="caller_id_number" expression="^anonymous$">
              <action application="set" data="effective_caller_id_number=Unknown"/>
       </condition>
</extension>
```