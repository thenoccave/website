# Neutron linux bridge agent failing with SELinux
Recently I have been working on setting up an OpenStack cluster as a proof of concept. Of the many issues that I ran into one was the neutron-linuxbridge-agent failing to work with SELinux. The daemon was running but the logs were full of
```
2021-07-08 02:38:06.565 415641 INFO neutron.common.config [-] /usr/bin/neutron-linuxbridge-agent version 18.0.0
2021-07-08 02:38:06.565 415641 DEBUG neutron.common.config [-] command line: /usr/bin/neutron-linuxbridge-agent --config-file /usr/share/neutron/neutron-dist.conf --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/linuxbridge_agent.ini --config-dir /etc/neutron/conf.d/common --config-dir /etc/neutron/conf.d/neutron-linuxbridge-agent --log-file /var/log/neutron/linuxbridge-agent.log setup_logging /usr/lib/python3.6/site-packages/neutron/common/config.py:112
2021-07-08 02:38:06.567 415641 INFO neutron.plugins.ml2.drivers.linuxbridge.agent.linuxbridge_neutron_agent [-] Interface mappings: {'bond0': 'bond0'}
2021-07-08 02:38:06.567 415641 INFO neutron.plugins.ml2.drivers.linuxbridge.agent.linuxbridge_neutron_agent [-] Bridge mappings: {}
2021-07-08 02:38:06.568 415641 INFO oslo.privsep.daemon [-] Running privsep helper: ['sudo', 'neutron-rootwrap', '/etc/neutron/rootwrap.conf', 'privsep-helper', '--config-file', '/usr/share/neutron/neutron-dist.conf', '--config-file', '/etc/neutron/neutron.conf', '--config-file', '/etc/neutron/plugins/ml2/linuxbridge_agent.ini', '--config-dir', '/etc/neutron/conf.d/neutron-linuxbridge-agent', '--privsep_context', 'neutron.privileged.default', '--privsep_sock_path', '/tmp/tmpshoxdpd4/privsep.sock']

2021-07-08 02:38:07.748 415641 CRITICAL oslo.privsep.daemon [-] privsep helper command exited non-zero (1)
2021-07-08 02:38:07.749 415641 CRITICAL neutron [-] Unhandled error: oslo_privsep.daemon.FailedToDropPrivileges: privsep helper command exited non-zero (1)
2021-07-08 02:38:07.749 415641 ERROR neutron Traceback (most recent call last):
2021-07-08 02:38:07.749 415641 ERROR neutron   File "/usr/bin/neutron-linuxbridge-agent", line 10, in <module>
2021-07-08 02:38:07.749 415641 ERROR neutron     sys.exit(main())
2021-07-08 02:38:07.749 415641 ERROR neutron   File "/usr/lib/python3.6/site-packages/neutron/cmd/eventlet/plugins/linuxbridge_neutron_agent.py", line 28, in main
2021-07-08 02:38:07.749 415641 ERROR neutron     agent_main.main()
2021-07-08 02:38:07.749 415641 ERROR neutron   File "/usr/lib/python3.6/site-packages/neutron/plugins/ml2/drivers/linuxbridge/agent/linuxbridge_neutron_agent.py", line 1035, in main
2021-07-08 02:38:07.749 415641 ERROR neutron     manager = LinuxBridgeManager(bridge_mappings, interface_mappings)
2021-07-08 02:38:07.749 415641 ERROR neutron   File "/usr/lib/python3.6/site-packages/neutron/plugins/ml2/drivers/linuxbridge/agent/linuxbridge_neutron_agent.py", line 80, in __init__
2021-07-08 02:38:07.749 415641 ERROR neutron     self.validate_interface_mappings()
2021-07-08 02:38:07.749 415641 ERROR neutron   File "/usr/lib/python3.6/site-packages/neutron/plugins/ml2/drivers/linuxbridge/agent/linuxbridge_neutron_agent.py", line 95, in validate_interface_mappings
2021-07-08 02:38:07.749 415641 ERROR neutron     if not ip_lib.device_exists(interface):
2021-07-08 02:38:07.749 415641 ERROR neutron   File "/usr/lib/python3.6/site-packages/neutron/agent/linux/ip_lib.py", line 746, in device_exists
2021-07-08 02:38:07.749 415641 ERROR neutron     return IPDevice(device_name, namespace=namespace).exists()
2021-07-08 02:38:07.749 415641 ERROR neutron   File "/usr/lib/python3.6/site-packages/neutron/agent/linux/ip_lib.py", line 328, in exists
2021-07-08 02:38:07.749 415641 ERROR neutron     return privileged.interface_exists(self.name, self.namespace)
2021-07-08 02:38:07.749 415641 ERROR neutron   File "/usr/lib/python3.6/site-packages/oslo_privsep/priv_context.py", line 246, in _wrap
2021-07-08 02:38:07.749 415641 ERROR neutron     self.start()
2021-07-08 02:38:07.749 415641 ERROR neutron   File "/usr/lib/python3.6/site-packages/oslo_privsep/priv_context.py", line 258, in start
2021-07-08 02:38:07.749 415641 ERROR neutron     channel = daemon.RootwrapClientChannel(context=self)
2021-07-08 02:38:07.749 415641 ERROR neutron   File "/usr/lib/python3.6/site-packages/oslo_privsep/daemon.py", line 367, in __init__
2021-07-08 02:38:07.749 415641 ERROR neutron     raise FailedToDropPrivileges(msg)
2021-07-08 02:38:07.749 415641 ERROR neutron oslo_privsep.daemon.FailedToDropPrivileges: privsep helper command exited non-zero (1)
2021-07-08 02:38:07.749 415641 ERROR neutron
```
The fix is a simple SELinux boolean change and daemon restart
```
~]# setsebool os_neutron_dac_override on
~]# systemctl restart neutron-linuxbridge-agent.service
```