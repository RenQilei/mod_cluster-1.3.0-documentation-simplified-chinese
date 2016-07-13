# 第十四章 从 mod_proxy 迁移

As mod_cluster is a sophisticated balancer migration from mod_proxy to mod_cluster is strait forward. mod_cluster replaces a reverse proxy with loadbalancing. A reversed proxy is configured like:

```
ProxyRequests Off
<Proxy *>
Order deny,allow
Allow from all
</Proxy>
ProxyPass /foo http://foo.example.com/bar
ProxyPassReverse /foo http://foo.example.com/bar
```

All the general proxy parameters could be used in mod_cluster they work like in mod_proxy, only the balancers and the workers definitions are slightly different.

## 14.1. Workers

| Mod_proxy Parameter | ClusterListener parameter | Remarks |
| -- | -- | -- |
| min | - | Not supported in this version |
| max | - | mod_cluster uses mod_proxy default value |
| smax | smax | Same as mod_proxy |
| ttl | ttl | Same as mod_proxy |
| acquire | workerTimeout | Same as mod_proxy acquire but in seconds |
| disablereuse | - | mod_cluster will disable the node in case of error and the ClusterListener will for the reuse via the STATUS message |
| flushPackets | flushPackets | Same as mod_proxy |
| flushwait | flushwait | Same as mod_proxy |
| keepalive | - | Always on: OS KEEP_ALIVE is always used. Use connectionTimeout in the <Connector> if needed |
| lbset | - | Not supported |
| ping | ping | Same as mod_proxy Default value 10 seconds |
| lbfactor | - | The load factor is received by mod_cluster from a calculated value in the ClusterListener |
| redirect | - | Not supported lbfactor sent to 0 makes a standby node |
| retry | - | ClusterListener will test when the node is back online |
| route | JVMRoute | In fact JBossWEB via the JVMRoute in the Engine will add it |
| status | - | mod_cluster has a finer status handling: by context via the ENABLE/STOP/DISABLE/REMOVE application messages. hot-standby is done by lbfactor = 0 and Error by lbfactor = 1 both values are sent in STATUS message by the ClusterListener |
| timeout | nodeTimeout | Default wait for ever (http://httpd.apache.org/docs/2.2/mod/mod_proxy.html is wrong there) |
| ttl | ttl | Default 60 seconds |

## 14.2. Balancers

| Mod_proxy Parameter | ClusterListener parameter | Remarks |
| -- | -- | -- |
| lbmethod | - | There is only one load balancing method in mod_cluster “cluster_byrequests” |
| maxattempts | maxAttempts | Default 1 |
| nofailover | stickySessionForce | Same as in mod_proxy |
| stickysession | StickySessionCookie/StickySessionPath | The 2 parameters in the ClusterListener are combined in one that behaves like in mod_proxy |
| timeout | workerTimeout | Default 1 seconds |