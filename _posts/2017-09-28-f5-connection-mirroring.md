---
layout: post
title: Enabling F5 Connection Mirroring
categories: [Networking,Deployment]
author: James Denton
---

## Problem 

F5 BIG-IP devices in RPC OpenStack architectures are primarily used to load balance connections to the OpenStack API services. This is known within the group as **Control Plane Load Balancing**.

When connection mirroring is *not* enabled, a load balancer failover event may cause new connections to build up between the newly-active F5 and the Galera server(s) while old connections remain in limbo. This may cause the max connection limit on the Galera server to be reached, which in turn will cause outages for multiple OpenStack services and a lack of stability with regard to RabbitMQ and other services.

## Solution

To provide high availability during a load balancer failover event, connection mirroring should be enabled on the following virtual servers:

* Galera/MySQL

On the `ACTIVE` device, find the virtual server name:

```
# tmsh list ltm virtual /RPC/* one-line | grep -i 'galera'
 
ltm virtual /RPC/RPC_VS_GALERA { destination /RPC/10.156.8.10:3306 ip-protocol tcp mask 255.255.255.255 partition RPC pool /RPC/RPC_POOL_GALERA profiles { fastL4 { } } source 0.0.0.0/0 source-address-translation { pool /RPC/RPC_SNATPOOL type snat } vs-index 68 }
```

Enable connection mirroring on the virtual server and also modify the pool as follows:

* Log in to the Configuration utility.
* Navigate to **Device Management > Devices**.
* In the **Device List**, select the name of the local device indicated with (Self).
* In the **Device Connectivity** section, select **Mirroring**.
* Select the appropriate IP addresses from the **Primary Local Mirroring Address** and **Secondary Local Mirroring Address** list.
* Click **Update**.

Mirror enabling can be done via CLI, but you rather need to collect details first.

```
modify /cm device <device_name> mirror-ip <IP address> mirror-secondary-ip <IP address>

modify ltm virtual /RPC/RPC_VS_GALERA mirror enabled
modify ltm pool /RPC/RPC_POOL_GALERA service-down-action reselect
modify ltm pool /RPC/RPC_POOL_GALERA slow-ramp-time 0
```

On each device, verify connections are being mirrored properly:

```
[james.denton@699175-F5-02-prod:Active:In Sync] ~ # tmsh show /sys connection type mirror
Sys::Connections
10.156.11.15:38996   10.156.8.10:3306  10.156.8.15:47670  10.156.11.63:3306  tcp  20   (tmm: 0)  none
10.156.10.172:49812  10.156.8.10:3306  10.156.8.15:49812  10.156.11.63:3306  tcp  257  (tmm: 0)  none
10.156.10.243:53600  10.156.8.10:3306  10.156.8.15:41009  10.156.11.63:3306  tcp  17   (tmm: 0)  none
10.156.9.43:45829    10.156.8.10:3306  10.156.8.15:47676  10.156.11.63:3306  tcp  11   (tmm: 0)  both-sides
10.156.11.139:52933  10.156.8.10:3306  10.156.8.15:52933  10.156.11.63:3306  tcp  0    (tmm: 0)  both-sides
10.156.11.244:39654  10.156.8.10:3306  10.156.8.15:39654  10.156.11.63:3306  tcp  33   (tmm: 0)  none
10.156.11.234:33986  10.156.8.10:3306  10.156.8.15:33986  10.156.11.63:3306  tcp  214  (tmm: 0)  none
10.156.10.235:38535  10.156.8.10:3306  10.156.8.15:38535  10.156.11.63:3306  tcp  73   (tmm: 0)  none
10.156.9.73:54114    10.156.8.10:3306  10.156.8.15:54114  10.156.11.63:3306  tcp  135  (tmm: 0)  none
10.156.10.221:60285  10.156.8.10:3306  10.156.8.15:60285  10.156.11.63:3306  tcp  15   (tmm: 0)  client-side
10.156.11.167:39145  10.156.8.10:3306  10.156.8.15:47522  10.156.11.63:3306  tcp  66   (tmm: 0)  none
10.156.10.172:50855  10.156.8.10:3306  10.156.8.15:60330  10.156.11.63:3306  tcp  90   (tmm: 0)  none
10.156.10.221:33880  10.156.8.10:3306  10.156.8.15:33880  10.156.11.63:3306  tcp  44   (tmm: 0)  none
10.156.9.43:46309    10.156.8.10:3306  10.156.8.15:2684   10.156.11.63:3306  tcp  30   (tmm: 0)  none
10.156.9.43:49447    10.156.8.10:3306  10.156.8.15:49447  10.156.11.63:3306  tcp  48   (tmm: 0)  none
10.156.11.15:34400   10.156.8.10:3306  10.156.8.15:34400  10.156.11.63:3306  tcp  50   (tmm: 0)  none
10.156.9.43:46226    10.156.8.10:3306  10.156.8.15:46226  10.156.11.63:3306  tcp  59   (tmm: 0)  none
10.156.11.188:47426  10.156.8.10:3306  10.156.8.15:47426  10.156.11.63:3306  tcp  10   (tmm: 0)  both-sides
...
10.156.10.221:38799  10.156.8.10:3306  10.156.8.15:28653  10.156.11.63:3306  tcp  49   (tmm: 1)  none
10.156.11.188:47518  10.156.8.10:3306  10.156.8.15:47518  10.156.11.63:3306  tcp  65   (tmm: 1)  none
10.156.11.30:34991   10.156.8.10:3306  10.156.8.15:22257  10.156.11.63:3306  tcp  36   (tmm: 1)  none
10.156.9.43:58473    10.156.8.10:3306  10.156.8.15:58473  10.156.11.63:3306  tcp  25   (tmm: 1)  none
10.156.9.43:59599    10.156.8.10:3306  10.156.8.15:59599  10.156.11.63:3306  tcp  25   (tmm: 1)  none
10.156.10.243:58353  10.156.8.10:3306  10.156.8.15:58353  10.156.11.63:3306  tcp  18   (tmm: 1)  none
10.156.10.221:32920  10.156.8.10:3306  10.156.8.15:32920  10.156.11.63:3306  tcp  3    (tmm: 1)  both-sides
10.156.10.243:56581  10.156.8.10:3306  10.156.8.15:56581  10.156.11.63:3306  tcp  0    (tmm: 1)  both-sides
Total records returned: 6455
[james.denton@699175-F5-02-prod:Active:In Sync] ~ #
```

```
[james.denton@699174-F5-01-prod:Standby:In Sync] ~ # tmsh show /sys connection type mirror
Sys::Connections
10.156.11.188:40190  10.156.8.10:3306  10.156.8.15:40190  10.156.11.63:3306  tcp  0  (tmm: 1)  none
10.156.11.4:44222    10.156.8.10:3306  10.156.8.15:44222  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.11.234:59828  10.156.8.10:3306  10.156.8.15:59828  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.10.243:57384  10.156.8.10:3306  10.156.8.15:57384  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.10.243:58187  10.156.8.10:3306  10.156.8.15:58187  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.11.188:48155  10.156.8.10:3306  10.156.8.15:48155  10.156.11.63:3306  tcp  0  (tmm: 1)  none
10.156.9.43:48967    10.156.8.10:3306  10.156.8.15:48967  10.156.11.63:3306  tcp  0  (tmm: 1)  none
10.156.11.15:46075   10.156.8.10:3306  10.156.8.15:46075  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.10.186:38279  10.156.8.10:3306  10.156.8.15:38279  10.156.11.63:3306  tcp  0  (tmm: 1)  none
10.156.9.88:47285    10.156.8.10:3306  10.156.8.15:47285  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.11.15:44866   10.156.8.10:3306  10.156.8.15:44866  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.11.15:43980   10.156.8.10:3306  10.156.8.15:43980  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.10.186:49548  10.156.8.10:3306  10.156.8.15:49548  10.156.11.63:3306  tcp  0  (tmm: 1)  none
10.156.10.186:37633  10.156.8.10:3306  10.156.8.15:37633  10.156.11.63:3306  tcp  0  (tmm: 1)  none
10.156.11.56:47489   10.156.8.10:3306  10.156.8.15:47489  10.156.11.63:3306  tcp  0  (tmm: 1)  none
10.156.9.43:60956    10.156.8.10:3306  10.156.8.15:60956  10.156.11.63:3306  tcp  0  (tmm: 1)  none
10.156.10.221:35163  10.156.8.10:3306  10.156.8.15:33649  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.9.43:52825    10.156.8.10:3306  10.156.8.15:59461  10.156.11.63:3306  tcp  1  (tmm: 1)  none
...
10.156.11.15:40288   10.156.8.10:3306  10.156.8.15:20498  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.10.221:38799  10.156.8.10:3306  10.156.8.15:28653  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.11.188:47518  10.156.8.10:3306  10.156.8.15:47518  10.156.11.63:3306  tcp  0  (tmm: 1)  none
10.156.11.30:34991   10.156.8.10:3306  10.156.8.15:22257  10.156.11.63:3306  tcp  0  (tmm: 1)  none
10.156.9.43:58473    10.156.8.10:3306  10.156.8.15:58473  10.156.11.63:3306  tcp  0  (tmm: 1)  none
10.156.9.43:59599    10.156.8.10:3306  10.156.8.15:59599  10.156.11.63:3306  tcp  0  (tmm: 1)  none
10.156.10.243:58353  10.156.8.10:3306  10.156.8.15:58353  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.10.221:32920  10.156.8.10:3306  10.156.8.15:32920  10.156.11.63:3306  tcp  1  (tmm: 1)  none
10.156.10.243:56581  10.156.8.10:3306  10.156.8.15:56581  10.156.11.63:3306  tcp  1  (tmm: 1)  none
Total records returned: 6456
[james.denton@699174-F5-01-prod:Standby:In Sync] ~ #
```

Connection counts between the two devices should be similar, and it is not expected that they be exactly the same. It may take a few minutes for connections to be mirrored between devices when enabled for the first time.
