---
layout: post
title: Resolving 'MySQL has gone away' errors in OpenStack deployments
categories: [Networking]
author: James Denton
---

In some log files, errors stating 'MySQL has gone away' will be observed.

```
2019-09-19 12:23:35.506 17812 ERROR oslo_db.sqlalchemy.engines [req-70b5c157-0990-4485-b4fe-e6b42a518188 - - - - -] Database connection was found disconnected; reconn
ecting: DBConnectionError: (pymysql.err.OperationalError) (2006, "MySQL server has gone away (error(104, 'Connection reset by peer'))") [SQL: u'SELECT 1'] (Background
 on this error at: http://sqlalche.me/e/e3q8)
```

In this particular environment, the issue is due to a mismatch between the F5 load balancer's idle timeout and SQL Alchemy's connection pooling timeout.

The F5 will close the connection before SQL Alchemy, causing the latter to believe the connection is still open and not opening a new one. This can result in service availability issues.

## OpenStack Solution

The F5's default idle timeout is 300 seconds, while SQLAlchemy has been 3600 seconds. You can lower the timeout for SQLAlchemy to something lower than 300, such as 240 seconds.

Reference: 190919-09724 

## F5 Solution

A new FastL4 profile can be created with an idle timeout matching that of OpenStack and can be applied to the Galera virtual servers.

```
ltm profile fastl4 PROF-FASTL4-LONGTIMEOUT {
 app-service none
 defaults-from /Common/fastL4
 idle-timeout 3600
}
```

Here is what Virtual Server looks like after the change:

```
ltm virtual RPC_VS_GALERA {
 destination x.x.x.x:3306
 ip-protocol tcp
 mask 255.255.255.255
 mirror enabled
 partition RPC
 pool RPC_POOL_GALERA
 profiles {
 PROF-FASTL4-LONGTIMEOUT { }
 }
 source 0.0.0.0/0
 source-address-translation {
 pool RPC_SNATPOOL
 type snat
 }
}

ltm virtual RPC_VS_GALERA_READ {
 destination x,x.x.x:3307
 ip-protocol tcp
 mask 255.255.255.255
 mirror enabled
 partition RPC
 pool RPC_POOL_GALERA_READ
 profiles {
 PROF-FASTL4-LONGTIMEOUT { }
 }
 source 0.0.0.0/0
 source-address-translation {
 pool RPC_SNATPOOL
 type snat
 }
}
```

## Reference

##### Idle Timeout

The BIG-IP system maintains connection information including the idle time for ongoing sessions. The Idle Timeout setting in the TCP profile specifies the length of time that a connection is idle before the connection is eligible for deletion. If no traffic flow is detected within the idle session timeout, the BIG-IP system can delete the session. *The default is 300 seconds*.

##### Keep Alive Interval

The Keep Alive Interval setting in the TCP profile is used to adjust the frequency at which the BIG-IP system sends TCP Keep-Alive packets to a remote host for connection validation. If the system does not receive a response for three consecutive TCP Keep-Alive packets, the connection reaches TCP Keep-Alive timeout and the system removes the connection. The TCP Keep-Alives, not to be confused with the HTTP Keep-Alive header, is a defined, optional TCP implementation in Request for Comments (RFC) 1122. *The default is 1800 seconds*.

While you would use the Idle Timeout setting to maintain connections and resources on the BIG-IP system, you would use the Keep Alive Interval setting when the server application has TCP Keep-Alives set for maintaining long-lived TCP connections. Fine-tune the Keep Alive Interval so the application design is aligned with the BIG-IP system in response to the nature of the full-proxy implementation.

The two settings are calculated independently in the BIG-IP system and you should consider them as separate entities when you adjust the timeout or interval value. The default Keep Alive Interval (1800 seconds) is greater than the default Idle Timeout (300 seconds) on a TCP profile.

[https://support.f5.com/csp/article/K13004262](https://support.f5.com/csp/article/K13004262)
