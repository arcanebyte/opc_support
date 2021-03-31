---
layout: post
title: Resolving 'OVS is dead' errors in Open vSwitch agent log
categories: [Networking]
author: James Denton
---

At times, various openvswitch-related services may experience issues and no longer function as expected. This issue may manifest itself as agents reporting as 'DOWN' and/or the journal being filled with logs from `neutron-openvswitch-agent` and potentially filling the disk. The agent will restart in a loop until the issue is resolved.

Below are two possible errors related to the issue:

```
root@1131000-compute7:~# systemctl status neutron-openvswitch-agent.service
● neutron-openvswitch-agent.service - neutron-openvswitch-agent service
   Loaded: loaded (/etc/systemd/system/neutron-openvswitch-agent.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2020-10-10 05:39:02 HKT; 4 months 22 days ago
 Main PID: 1792443 (/openstack/venv)
    Tasks: 34 (limit: 12287)
   Memory: 293.4M
      CPU: 5h 21min 18.217s
   CGroup: /neutron.slice/neutron-openvswitch-agent.service
           ├─1792443 /openstack/venvs/neutron-20.1.7/bin/python3 /openstack/venvs/neutron-20.1.7/bin/neutron-openvswitch-agent --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plug
           ├─1792498 /openstack/venvs/neutron-20.1.7/bin/python3 /openstack/venvs/neutron-20.1.7/bin/privsep-helper --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2
           ├─1792540 sudo /openstack/venvs/neutron-20.1.7/bin/neutron-rootwrap-daemon /etc/neutron/rootwrap.conf
           ├─1792541 /openstack/venvs/neutron-20.1.7/bin/python3 /openstack/venvs/neutron-20.1.7/bin/neutron-rootwrap-daemon /etc/neutron/rootwrap.conf
           └─1792551 ovsdb-client monitor tcp:127.0.0.1:6640 Interface name,ofport,external_ids --format=json
 
 
Mar 03 19:42:29 1131000-compute7 neutron-openvswitch-agent[1792443]: 2021-03-03 19:42:29.076 1792443 WARNING neutron.plugins.ml2.drivers.openvswitch.agent.ovs_neutron_agent [req-473a7f5a-5dd8-4b8d-b6cd-20fc1ff9e904 - - - - -] OVS is dead. OVSNeutronAgent will keep running and checking OVS status periodically.
Mar 03 19:42:29 1131000-compute7 neutron-openvswitch-agent[1792443]: 2021-03-03 19:42:29.076 1792443 INFO neutron.plugins.ml2.drivers.openvswitch.agent.ovs_neutron_agent [req-473a7f5a-5dd8-4b8d-b6cd-20fc1ff9e904 - - - - -] Agent rpc_loop - iteration:1671866 completed. Processed ports statistics: {'regular': {'added': 0, 'updated': 0, 'removed': 0}}. Elapsed:300.003
Mar 03 19:42:29 1131000-compute7 neutron-openvswitch-agent[1792443]: 2021-03-03 19:42:29.076 1792443 INFO neutron.plugins.ml2.drivers.openvswitch.agent.ovs_neutron_agent [req-473a7f5a-5dd8-4b8d-b6cd-20fc1ff9e904 - - - - -] Agent rpc_loop - iteration:1671867 started
Mar 03 19:47:29 1131000-compute7 neutron-openvswitch-agent[1792443]: 2021-03-03 19:47:29.077 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.ofswitch [req-473a7f5a-5dd8-4b8d-b6cd-20fc1ff9e904 - - - - -] ofctl request version=0x4,msg_type=0x12,msg_len=0x38,xid=0x69a699b0,OFPFlowStatsRequest(cookie=0,cookie_mask=0,flags=0,match=OFPMatch(oxm_fields={}),out_group=4294967295,out_port=4294967295,table_id=23,type=1) timed out: eventlet.timeout.Timeout: 300 seconds
Mar 03 19:47:29 1131000-compute7 neutron-openvswitch-agent[1792443]: 2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int [req-473a7f5a-5dd8-4b8d-b6cd-20fc1ff9e904 - - - - -] Failed to communicate with the switch: RuntimeError: ofctl request version=0x4,msg_type=0x12,msg_len=0x38,xid=0x69a699b0,OFPFlowStatsRequest(cookie=0,cookie_mask=0,flags=0,match=OFPMatch(oxm_fields={}),out_group=4294967295,out_port=4294967295,table_id=23,type=1) timed out
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int Traceback (most recent call last):
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int   File "/openstack/venvs/neutron-20.1.7/lib/python3.6/site-packages/neutron/plugins/ml2/drivers/openvswitch/agent/openflow/native/ofswitch.py", line 92, in _send_msg
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int     result = ofctl_api.send_msg(self._app, msg, reply_cls, reply_multi)
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int   File "/openstack/venvs/neutron-20.1.7/lib/python3.6/site-packages/os_ken/app/ofctl/api.py", line 89, in send_msg
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int     reply_multi=reply_multi))()
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int   File "/openstack/venvs/neutron-20.1.7/lib/python3.6/site-packages/os_ken/base/app_manager.py", line 279, in send_request
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int     return req.reply_q.get()
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int   File "/openstack/venvs/neutron-20.1.7/lib/python3.6/site-packages/eventlet/queue.py", line 322, in get
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int     return waiter.wait()
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int   File "/openstack/venvs/neutron-20.1.7/lib/python3.6/site-packages/eventlet/queue.py", line 141, in wait
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int     return get_hub().switch()
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int   File "/openstack/venvs/neutron-20.1.7/lib/python3.6/site-packages/eventlet/hubs/hub.py", line 298, in switch
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int     return self.greenlet.switch()
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int eventlet.timeout.Timeout: 300 seconds
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int During handling of the above exception, another exception occurred:
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int Traceback (most recent call last):
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int   File "/openstack/venvs/neutron-20.1.7/lib/python3.6/site-packages/neutron/plugins/ml2/drivers/openvswitch/agent/openflow/native/br_int.py", line 61, in check_canary_table
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int     flows = self.dump_flows(constants.CANARY_TABLE)
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int   File "/openstack/venvs/neutron-20.1.7/lib/python3.6/site-packages/neutron/plugins/ml2/drivers/openvswitch/agent/openflow/native/ofswitch.py", line 163, in dump_flows
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int     reply_multi=True)
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int   File "/openstack/venvs/neutron-20.1.7/lib/python3.6/site-packages/neutron/plugins/ml2/drivers/openvswitch/agent/openflow/native/ofswitch.py", line 110, in _send_msg
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int     raise RuntimeError(m)
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int RuntimeError: ofctl request version=0x4,msg_type=0x12,msg_len=0x38,xid=0x69a699b0,OFPFlowStatsRequest(cookie=0,cookie_mask=0,flags=0,match=OFPMatch(oxm_fields={}),out_group=4294967295,out_port=4294967295,table_id=23,type=1) timed out
                                                                     2021-03-03 19:47:29.078 1792443 ERROR neutron.plugins.ml2.drivers.openvswitch.agent.openflow.native.br_int
Mar 03 19:47:29 1131000-compute7 neutron-openvswitch-agent[1792443]: 2021-03-03 19:47:29.078 1792443 WARNING neutron.plugins.ml2.drivers.openvswitch.agent.ovs_neutron_agent [req-473a7f5a-5dd8-4b8d-b6cd-20fc1ff9e904 - - - - -] OVS is dead. OVSNeutronAgent will keep running and checking OVS status periodically.
Mar 03 19:47:29 1131000-compute7 neutron-openvswitch-agent[1792443]: 2021-03-03 19:47:29.079 1792443 INFO neutron.plugins.ml2.drivers.openvswitch.agent.ovs_neutron_agent [req-473a7f5a-5dd8-4b8d-b6cd-20fc1ff9e904 - - - - -] Agent rpc_loop - iteration:1671867 completed. Processed ports statistics: {'regular': {'added': 0, 'updated': 0, 'removed': 0}}. Elapsed:300.003
Mar 03 19:47:29 1131000-compute7 neutron-openvswitch-agent[1792443]: 2021-03-03 19:47:29.079 1792443 INFO neutron.plugins.ml2.drivers.openvswitch.agent.ovs_neutron_agent [req-473a7f5a-5dd8-4b8d-b6cd-20fc1ff9e904 - - - - -] Agent rpc_loop - iteration:1671868 started
```

```
root@1131000-compute7:~# systemctl status neutron-openvswitch-agent.service --no-pager --full
● neutron-openvswitch-agent.service - neutron-openvswitch-agent service
   Loaded: loaded (/etc/systemd/system/neutron-openvswitch-agent.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2021-03-03 22:33:03 HKT; 30s ago
 Main PID: 2779203 (/openstack/venv)
    Tasks: 64 (limit: 12287)
   Memory: 166.2M
      CPU: 2.949s
   CGroup: /neutron.slice/neutron-openvswitch-agent.service
           ├─2779203 /openstack/venvs/neutron-20.1.7/bin/python3 /openstack/venvs/neutron-20.1.7/bin/neutron-openvswitch-agent --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini --config-file /etc/neutron/plugins/ml2/openvswitch_agent.ini
           └─2779233 /openstack/venvs/neutron-20.1.7/bin/python3 /openstack/venvs/neutron-20.1.7/bin/privsep-helper --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini --config-file /etc/neutron/plugins/ml2/openvswitch_agent.ini --privsep_context neutron.privileged.default --privsep_sock_path /tmp/tmp862ih0ti/privsep.sock
 
Mar 03 22:33:06 1131000-compute7 sudo[2779225]: pam_unix(sudo:session): session closed for user root
Mar 03 22:33:06 1131000-compute7 neutron-openvswitch-agent[2779203]: 2021-03-03 22:33:06.039 2779203 INFO oslo.privsep.daemon [-] Spawned new privsep daemon via rootwrap
Mar 03 22:33:06 1131000-compute7 neutron-openvswitch-agent[2779203]: 2021-03-03 22:33:05.944 2779233 INFO oslo.privsep.daemon [-] privsep daemon starting
Mar 03 22:33:06 1131000-compute7 neutron-openvswitch-agent[2779203]: 2021-03-03 22:33:05.947 2779233 INFO oslo.privsep.daemon [-] privsep process running with uid/gid: 0/0
Mar 03 22:33:06 1131000-compute7 neutron-openvswitch-agent[2779203]: 2021-03-03 22:33:05.949 2779233 INFO oslo.privsep.daemon [-] privsep process running with capabilities (eff/prm/inh): CAP_DAC_OVERRIDE|CAP_DAC_READ_SEARCH|CAP_NET_ADMIN|CAP_SYS_ADMIN|CAP_SYS_PTRACE/CAP_DAC_OVERRIDE|CAP_DAC_READ_SEARCH|CAP_NET_ADMIN|CAP_SYS_ADMIN|CAP_SYS_PTRACE/none
Mar 03 22:33:06 1131000-compute7 neutron-openvswitch-agent[2779203]: 2021-03-03 22:33:05.949 2779233 INFO oslo.privsep.daemon [-] privsep daemon running as pid 2779233
Mar 03 22:33:16 1131000-compute7 neutron-openvswitch-agent[2779203]: 2021-03-03 22:33:16.838 2779203 WARNING neutron.plugins.ml2.drivers.openvswitch.agent.ovs_neutron_agent [-] Ovsdb command timeout!: ovsdbapp.exceptions.TimeoutException: Commands [<ovsdbapp.schema.open_vswitch.commands.AddBridgeCommand object at 0x7f1790fa1940>, <ovsdbapp.backend.ovs_idl.command.DbAddCommand object at 0x7f1790fa1978>, <ovsdbapp.backend.ovs_idl.command.DbSetCommand object at 0x7f1790fa19b0>] exceeded timeout 10 seconds
Mar 03 22:33:16 1131000-compute7 neutron-openvswitch-agent[2779203]: 2021-03-03 22:33:16.840 2779203 ERROR ovsdbapp.schema.open_vswitch.impl_idl [-] Post-commit checks failed: ovsdbapp.exceptions.TimeoutException: Commands [<ovsdbapp.schema.open_vswitch.commands.AddBridgeCommand object at 0x7f1790fa1940>, <ovsdbapp.backend.ovs_idl.command.DbAddCommand object at 0x7f1790fa1978>, <ovsdbapp.backend.ovs_idl.command.DbSetCommand object at 0x7f1790fa19b0>] exceeded timeout 10 seconds
                                                                     2021-03-03 22:33:16.840 2779203 ERROR ovsdbapp.schema.open_vswitch.impl_idl Traceback (most recent call last):
                                                                     2021-03-03 22:33:16.840 2779203 ERROR ovsdbapp.schema.open_vswitch.impl_idl   File "/openstack/venvs/neutron-20.1.7/lib/python3.6/site-packages/ovsdbapp/schema/open_vswitch/impl_idl.py", line 40, in post_commit
                                                                     2021-03-03 22:33:16.840 2779203 ERROR ovsdbapp.schema.open_vswitch.impl_idl     self.do_post_commit(txn)
                                                                     2021-03-03 22:33:16.840 2779203 ERROR ovsdbapp.schema.open_vswitch.impl_idl   File "/openstack/venvs/neutron-20.1.7/lib/python3.6/site-packages/ovsdbapp/schema/open_vswitch/impl_idl.py", line 60, in do_post_commit
                                                                     2021-03-03 22:33:16.840 2779203 ERROR ovsdbapp.schema.open_vswitch.impl_idl     timeout=self.timeout)
                                                                     2021-03-03 22:33:16.840 2779203 ERROR ovsdbapp.schema.open_vswitch.impl_idl ovsdbapp.exceptions.TimeoutException: Commands [<ovsdbapp.schema.open_vswitch.commands.AddBridgeCommand object at 0x7f1790fa1940>, <ovsdbapp.backend.ovs_idl.command.DbAddCommand object at 0x7f1790fa1978>, <ovsdbapp.backend.ovs_idl.command.DbSetCommand object at 0x7f1790fa19b0>] exceeded timeout 10 seconds
                                                                     2021-03-03 22:33:16.840 2779203 ERROR ovsdbapp.schema.open_vswitch.impl_idl
```


## Verification

To verify the issue, run the following commands:

```
# timeout 120 ovs-ofctl dump-flows br-int table=23
# timeout 120 ovs-ofctl dump-flows br-ex | head -n2
```
If these commands hang and do not return any flows after 120 seconds, you may assume the `openvswitch` service(s) are DOWN and must be restarted.

## Solution

To resolve this issue, the following must occur (in this order):

* Restart the 'openvswitch-switch' service.
* Restart the 'neutron-openvswitch-agent' service.

```
# systemctl restart openvswitch-switch
# systemctl restart neutron-openvswitch-agent
```

Re-run the verification commands above. Flows should be returned relatively quickly.