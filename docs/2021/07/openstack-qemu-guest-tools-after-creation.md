# Openstack qemu guest agent after instance creation
---
Note: To get this to work involves a hacky fix to make manual changes to the Openstack database. If you know of a way to modify system metadata on Openstack please let me know by raising an issue at [https://github.com/thenoccave/website/issues](https://github.com/thenoccave/website/issues)
---
When doing some testing on Openstack I realised I had deployed instances without setting the metadata on the image to ensure that the serial device is enabled for the qemu-guest-agent.

When there is a metadata entry for hw_qemu_guest_agent=yes on the image, when an instance is deployed from it a special serial device is deployed to allow the guest tools to talk to the host.
```
~]# virsh edit instance-00000002
...snip...
<channel type='unix'>
    <source mode='bind' path='/var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-00000002.sock'/>
    <target type='virtio' name='org.qemu.guest_agent.0'/>
    <address type='virtio-serial' controller='0' bus='0' port='1'/>
</channel>
...snip...
```
without this serial interface the guest tools will fail to start
```
● qemu-guest-agent.service - QEMU Guest Agent
   Loaded: loaded (/usr/lib/systemd/system/qemu-guest-agent.service; enabled; vendor preset: enabled)
   Active: inactive (dead)

Jul 08 00:05:21 ba09.dev.new.onsite.local systemd[1]: Dependency failed for QEMU Guest Agent.
Jul 08 00:05:21 ba09.dev.new.onsite.local systemd[1]: Job qemu-guest-agent.service/start failed with result 'dependency'.
Jul 08 00:10:40 ba09.dev.new.onsite.local systemd[1]: Dependency failed for QEMU Guest Agent.
Jul 08 00:10:40 ba09.dev.new.onsite.local systemd[1]: Job qemu-guest-agent.service/start failed with result 'dependency'.
```
The only way I could find to fix this without recreating the instance was to manually create a system metadata entry for the instance in nova.
```
MariaDB > use nova
MariaDB [nova]> insert into instance_system_metadata (created_at, instance_uuid, `key`, value, deleted) VALUES (NOW(), '<InstanceID>', 'image_hw_qemu_guest_agent', 'yes', 0);
Query OK, 1 row affected (0.035 sec)
```
** Once you stop and start the instance (A reboot doesn't do it) then the serial port should be enabled and the guest agent should start ***
```
[root@ba09 ~]# systemctl status qemu-guest-agent.service
● qemu-guest-agent.service - QEMU Guest Agent
   Loaded: loaded (/usr/lib/systemd/system/qemu-guest-agent.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2021-07-08 01:55:07 UTC; 1min 11s ago
 Main PID: 863 (qemu-ga)
    Tasks: 1
   Memory: 1.3M
   CGroup: /system.slice/qemu-guest-agent.service
           └─863 /usr/bin/qemu-ga --method=virtio-serial --path=/dev/virtio-ports/org.qemu.guest_...

Jul 08 01:55:07 ba09.dev.new.onsite.local systemd[1]: Started QEMU Guest Agent.
```