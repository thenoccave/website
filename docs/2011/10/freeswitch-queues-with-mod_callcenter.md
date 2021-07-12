# Freeswitch Queues with Mod_callcenter

Configuring Call Queues with mod_callcenter in freeswitch isn’t difficult but can be a tad confusing. For this I am setting up queues support, sales and reception. The users 101, 102 and 103 will be members of those queues.

First you need to define your queues in conf/autoload_configs/callcenter.conf.xml
```
<configuration name="callcenter.conf" description="CallCenter">
    <settings>
    </settings>
    <queues>
        <queue name="onemetricsupport@default">
            <param name="strategy" value="top-down"/>
            <param name="moh-sound" value="$${hold_music}"/>
            <param name="time-base-score" value="system"/>
            <param name="max-wait-time" value="0"/>
            <param name="max-wait-time-with-no-agent" value="0"/>
            <param name="max-wait-time-with-no-agent-time-reached" value="20"/>
            <param name="tier-rules-apply" value="false"/>
            <param name="tier-rule-wait-second" value="300"/>
            <param name="tier-rule-wait-multiply-level" value="true"/>
            <param name="tier-rule-no-agent-no-wait" value="false"/>
            <param name="discard-abandoned-after" value="60"/>
            <param name="abandoned-resume-allowed" value="false"/>
        </queue>
            <queue name="onemetricreception@default">
            <param name="strategy" value="top-down"/>
            <param name="moh-sound" value="$${hold_music}"/>
            <param name="time-base-score" value="system"/>
            <param name="max-wait-time" value="0"/>
            <param name="max-wait-time-with-no-agent" value="0"/>
            <param name="max-wait-time-with-no-agent-time-reached" value="20"/>
            <param name="tier-rules-apply" value="false"/>
            <param name="tier-rule-wait-second" value="300"/>
            <param name="tier-rule-wait-multiply-level" value="true"/>
            <param name="tier-rule-no-agent-no-wait" value="false"/>
            <param name="discard-abandoned-after" value="60"/>
            <param name="abandoned-resume-allowed" value="false"/>
        </queue>
        <queue name="onemetricaccounts@default">
            <param name="strategy" value="top-down"/>
            <param name="moh-sound" value="$${hold_music}"/>
            <param name="time-base-score" value="system"/>
            <param name="max-wait-time" value="0"/>
            <param name="max-wait-time-with-no-agent" value="0"/>
            <param name="max-wait-time-with-no-agent-time-reached" value="20"/>
            <param name="tier-rules-apply" value="false"/>
            <param name="tier-rule-wait-second" value="300"/>
            <param name="tier-rule-wait-multiply-level" value="true"/>
            <param name="tier-rule-no-agent-no-wait" value="false"/>
            <param name="discard-abandoned-after" value="60"/>
            <param name="abandoned-resume-allowed" value="false"/>
        </queue>
    </queues>
</configuration>
```

As you can see we are using the default context and our queues are called onemetricsupport, onemetricreception and onemetricaccounts. The key configuration items are:

```
name - The name of the queue
strategy - This is the method that the queue determines the next agent to offer the queue. A list of strategies and how they operate can be found at http://wiki.freeswitch.org/wiki/Mod_callcenter#Distribution_Strategy
moh-sound - The music on hold to play to callers in the queue. In this case we are are using the hold_music variable defined in conf/vars.xml
```

For the rest of the options view [http://wiki.freeswitch.org/wiki/Mod_callcenter#Queue_options](http://wiki.freeswitch.org/wiki/Mod_callcenter#Queue_options)


Next the agent configuration must be defined:
```
<agents>
    <agent name="101" type="callback" contact="[call_timeout=60]user/101" status="Logged Out" max-no-answer="3" wrap-up-time="10" reject-delay-time="30" busy-delay-time="60" no-answer-delay-time="10" />
    <agent name="102" type="callback" contact="[call_timeout=60]user/102" status="Logged Out" max-no-answer="3" wrap-up-time="10" reject-delay-time="30" busy-delay-time="60" no-answer-delay-time="10" />
    <agent name="103" type="callback" contact="[call_timeout=60]user/103" status="Logged Out" max-no-answer="3" wrap-up-time="10" reject-delay-time="30" busy-delay-time="60" no-answer-delay-time="10"/>
</agents>
```
The key configuration items for the agents are:
```
name - The name of the agent. This can be anything however to make it easy to do agent login/logout buttons on the SPA phones we keep it the same as the extension
contact - The dialstring to reach the agent. Note the call_timeout value defines how long you will offer the agent the call each time. In this case the agent has 60 seconds to answer the phone.
status - This is the status that the agent will be created as being in. I recommend you add your agents as logged out and then let them make themselves available to the queue
max-no-answer - The agent will be tried this many times before the agent is considered on break.
wrap-up-time - The agent will be given this long before being offered a new call. In this case once an agent is done with a call they will have 10 seconds before the next call is offered to them
reject-delay-time - If an agent rejects a call manually then this is the time to wait before a call is offered to them.
```
More options can be found at [wiki.freeswitch.org/wiki/Mod_callcenter#Agent_options](wiki.freeswitch.org/wiki/Mod_callcenter#Agent_options)

Next we need to configure which agents belong to which queues
```
<tiers>
    <tier agent="101" queue="onemetricsupport@default" level="1" position="1"/>
    <tier agent="101" queue="onemetricaccounts@default" level="1" position="2"/>
    <tier agent="101" queue="onemetricreception@default" level="1" position="1"/>
 
    <tier agent="102" queue="onemetricsupport@default" level="1" position="3"/>
    <tier agent="102" queue="onemetricaccounts@default" level="1" position="3"/>
    <tier agent="102" queue="onemetricreception@default" level="1" position="3"/>
 
    <tier agent="103" queue="onemetricsupport@default" level="1" position="2"/>
    <tier agent="103" queue="onemetricaccounts@default" level="1" position="1"/>
    <tier agent="103" queue="onemetricreception@default" level="1" position="2"/>
</tiers>
```
The key configuration items are:
```
agent - The name of the agent
queue - The name of the queue the agent should be in
position - We are using a top down strategy so calls should preference the agent with the lowest position and work its way up from there.
```
The finial configuration looks like this:
```
<configuration name="callcenter.conf" description="CallCenter">
    <settings>
    </settings>
    <queues>
        <queue name="onemetricsupport@default">
            <param name="strategy" value="top-down"/>
            <param name="moh-sound" value="$${hold_music}"/>
            <param name="time-base-score" value="system"/>
            <param name="max-wait-time" value="0"/>
            <param name="max-wait-time-with-no-agent" value="0"/>
            <param name="max-wait-time-with-no-agent-time-reached" value="20"/>
            <param name="tier-rules-apply" value="false"/>
            <param name="tier-rule-wait-second" value="300"/>
            <param name="tier-rule-wait-multiply-level" value="true"/>
            <param name="tier-rule-no-agent-no-wait" value="false"/>
            <param name="discard-abandoned-after" value="60"/>
            <param name="abandoned-resume-allowed" value="false"/>
        </queue>
                <queue name="onemetricreception@default">
            <param name="strategy" value="top-down"/>
            <param name="moh-sound" value="$${hold_music}"/>
            <param name="time-base-score" value="system"/>
            <param name="max-wait-time" value="0"/>
            <param name="max-wait-time-with-no-agent" value="0"/>
            <param name="max-wait-time-with-no-agent-time-reached" value="20"/>
            <param name="tier-rules-apply" value="false"/>
            <param name="tier-rule-wait-second" value="300"/>
            <param name="tier-rule-wait-multiply-level" value="true"/>
            <param name="tier-rule-no-agent-no-wait" value="false"/>
            <param name="discard-abandoned-after" value="60"/>
            <param name="abandoned-resume-allowed" value="false"/>
        </queue>
        <queue name="onemetricaccounts@default">
            <param name="strategy" value="top-down"/>
            <param name="moh-sound" value="$${hold_music}"/>
            <param name="time-base-score" value="system"/>
            <param name="max-wait-time" value="0"/>
            <param name="max-wait-time-with-no-agent" value="0"/>
            <param name="max-wait-time-with-no-agent-time-reached" value="20"/>
            <param name="tier-rules-apply" value="false"/>
            <param name="tier-rule-wait-second" value="300"/>
            <param name="tier-rule-wait-multiply-level" value="true"/>
            <param name="tier-rule-no-agent-no-wait" value="false"/>
            <param name="discard-abandoned-after" value="60"/>
            <param name="abandoned-resume-allowed" value="false"/>
        </queue>
    </queues>
    <agents>
        <agent name="101" type="callback" contact="[call_timeout=60]user/101" status="Logged Out" max-no-answer="3" wrap-up-time="10" reject-delay-time="30" busy-delay-time="60" no-answer-delay-time="10" />
        <agent name="102" type="callback" contact="[call_timeout=60]user/102" status="Logged Out" max-no-answer="3" wrap-up-time="10" reject-delay-time="30" busy-delay-time="60" no-answer-delay-time="10" />
        <agent name="103" type="callback" contact="[call_timeout=60]user/103" status="Logged Out" max-no-answer="3" wrap-up-time="10" reject-delay-time="30" busy-delay-time="60" no-answer-delay-time="10"/>
    </agents>
    <tiers>
        <tier agent="101" queue="onemetricsupport@default" level="1" position="1"/>
        <tier agent="101" queue="onemetricaccounts@default" level="1" position="2"/>
        <tier agent="101" queue="onemetricreception@default" level="1" position="1"/>
        <tier agent="102" queue="onemetricsupport@default" level="1" position="3"/>
        <tier agent="102" queue="onemetricaccounts@default" level="1" position="3"/>
        <tier agent="102" queue="onemetricreception@default" level="1" position="3"/>
        <tier agent="103" queue="onemetricsupport@default" level="1" position="2"/>
        <tier agent="103" queue="onemetricaccounts@default" level="1" position="1"/>
        <tier agent="103" queue="onemetricreception@default" level="1" position="2"/>
    </tiers>
</configuration>
```
Next we need to get calls into the queues. We have an ivr that distributes the calls to extensions that call the mod_callcenter app. In our conf/ivr_menus/Level1OneMetric.xml we have the following:
```
<include>
        <menu name="Level1OneMetric"
                greet-long="say:Welcome to One Metric. Press 1 for support, 2 for accounts, 3 for reception or 4 to join a confrence call"
                greet-short="say:Welcome to One Metric. Press 1 for support, 2 for accounts, 3 for reception or 4 to join a confrence call"
                invalid-sound="ivr/ivr-that_was_an_invalid_entry.wav"
                exit-sound="voicemail/vm-goodbye.wav"
                confirm-macro=""
                confirm-key=""
                tts-engine="flite"
                tts-voice="slt"
                confirm-attempts="3"
                timeout="3000"
                inter-digit-timeout="2000"
                max-failures="3"
                max-timeouts="3"
                digit-len="4">
 
                <entry action="menu-exec-app" digits="1" param="transfer 450 XML default"/>    <!-- Support -->
                <entry action="menu-exec-app" digits="2" param="transfer 451 XML default"/>    <!-- Accounts -->
                <entry action="menu-exec-app" digits="3" param="transfer 452 XML default"/>    <!-- Reception -->
 
        </menu>
</include>
```
As you can see calls are set to the extensions 450, 451 and 452 based on the selected option. In our conf/dialplan/default.xml we have those extensions setup.
```
<!-- One Metric Support -->
<extension>
        <ondition field="destination_number" expression="^(450)$">
                <action application="set" data="caller_id_name=One Metric Support" />
                <action application="set" data="call_timeout=60" />
                <action application="set" data="originate_timeout=60" />
                <action application="callcenter" data="onemetricsupport@default"/>
        </condition>
</extension>

<!-- One Metric Accounts -->
<extension>
        <ondition field="destination_number" expression="^(451)$">
                <action application="set" data="caller_id_name=One Metric Accounts" />
                <action application="callcenter" data="onemetricaccounts@default"/>
        </condition>
</extension>

<!-- One Metric Reception -->
<extension>
        <ondition field="destination_number" expression="^(452)$">
                <action application="set" data="caller_id_name=One Metric Reception" />
                <action application="callcenter" data="onemetricreception@default"/>
        </condition>
</extension>
```
We change the caller_id_name variable. This will show up on the phone so the agent knows what queue the incoming call is coming from.

Next we need a way for the agents to log into the queues. We setup the extensions 300 to log the agent in and 301 to log the agent out.
```
<!-- Log Out of the Call Queues -->
<extension name="agent_login">
        <ondition field="destination_number" expression="^300$">
                <action application="set" data="res=${callcenter_config(agent set status ${caller_id_number} 'Available')}" />
                <action application="answer" data=""/>
                <action application="sleep" data="500"/>
                <action application="playback" data="ivr/ivr-you_are_now_logged_in.wav"/>
                <action application="hangup" data="NORMAL_CLEARING"/>
        </condition>
</extension>
 
<!-- Log Into the Call Queues -->
<extension name="agent_logoff">
        <ondition field="destination_number" expression="^301$">
                <action application="set" data="res=${callcenter_config(agent set status ${caller_id_number} 'Logged Out')}" />
                <action application="answer" data=""/>
                <action application="sleep" data="500"/>
                <action application="playback" data="ivr/ivr-you_are_now_logged_out.wav"/>
                <action application="hangup" data=""/>
        </condition>
</extension>
```
Now we can configure our phones to have line buttons to log in and out of the call queues by adding the following the extended function
```
Login - fnc=blf+sd+cp;sub=300@$PROXY;ext=300@$PROXY
Logout - fnc=blf+sd+cp;sub=301@$PROXY;ext=301@$PROXY
```
Once you have made changes you will have to run reload mod_callcenter from the cli
```
freeswitch@internal> reload mod_callcenter
```

### Commands to know

Display the call queues:
```
freeswitch@internal> callcenter_config queue list
```
Display calls in a queue:
```
freeswitch@internal> callcenter_config queue list members onemetricsupport@default
```
Display all the agents:
```
freeswitch@internal> callcenter_config agent list
```
Display all the agents in the queue
```
freeswitch@internal> callcenter_config tier list agents
```

### Further Reading and References:
[wiki.freeswitch.org/wiki/Main_Page](wiki.freeswitch.org/wiki/Main_Page)
[wiki.freeswitch.org/wiki/Mod_callcenter](wiki.freeswitch.org/wiki/Mod_callcenter)
