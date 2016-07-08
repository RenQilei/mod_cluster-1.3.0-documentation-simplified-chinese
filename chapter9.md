# 第九章 服务器端配置属性

The tables below enumerate the configuration properties available to an application server node. The location for these properties depends on how mod_cluster is configured.

底下的表格列举了一台应用服务器（application server）节点上可选的配置属性。这些属性的配置的位置取决于 [mod_cluster 是被如何配置的](chapter6.md)。

## 9.1. Proxy Discovery Configuration

The list of proxies from which an application expects to receive AJP connections is either defined statically, via the addresses defined in the proxyList configuration property; or discovered dynamically via the advertise mechanism. Using a special mod_advertise module, proxies can advertise their existence by periodically broadcasting a multicast message containing its address/port. This functionality is enabled via the advertise configuration property. If configured to listen, a server can learn of the proxy's existence, then notify that proxy of its own existence, and update
its configuration accordingly. This frees both the proxy and the server from having to define static, environment-specific configuration values.

**Session draining strategy**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| -- | -- | -- | -- | -- |
| sessionDrainingStrategy | session-draining-strategy | DEFAULT | Worker | Worker |

Indicates the session draining strategy used during undeployment of a web application. There are three possible values:

* ```DEFAULT```: Drain sessions before web application undeploy only if the web application is non-distributable.
* ```ALWAYS```: Always drain sessions before web application undeploy, even for distributable web applications.
* ```NEVER```: Do not drain sessions before web application undeploy, even for non-distributable web application.

**Proxies**

| Tomcat attribute | AS7 attribute | Wildfly attribute |Default | Location | Scope |
| -- | -- | -- | -- | -- | -- |
| proxyList | proxy-list | proxies | None | Worker | Worker |

* Tomcat/AS7: Defines a comma delimited list of httpd proxies with which this node will initially communicate. Value should be of the form: address1:port1,address2:port2. Using the default configuration, this property can by manipulated via the jboss.mod_cluster.proxyList system property.
* Wildfly: In Wildfly, the ```proxy-list``` attribute of the modcluster subsystem element is deprecated. Instead, one uses an output socket binding. The following example leverages ```jboss-cli.sh```, e.g. :
  * Add a socket binding: ```/socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=my-proxies:add(host=10.10.10.11,port=3333)```
  * Add the socket binding to the modcluster subsystem: ```/subsystem=modcluster/mod-cluster-config=configuration:write-attribute(name=proxies, value="my-proxies")```

**Excluded contexts**

| Tomcat attribute | AS7/Wildfly attribute | Wildfly Default | Tomcat/AS7 Default | Location | Scope |
| -- | -- | -- | -- | -- | -- |
| excludedContexts | excluded-contexts | None | ROOT, admin-console, invoker, bossws, jmx-console, juddi, web-console | Worker | Worker |

List of contexts to exclude from httpd registration, of the form: *host1:context1,host2:context2,host3:context3* If no host is indicated, it is assumed to be the default host of the server (e.g. localhost). “ROOT” indicates the root context. Using the default configuration, this property can by manipulated via the jboss.mod_cluster.excludedContexts system property.

**Auto Enable Contexts**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| -- | -- | -- | -- | -- |
| autoEnableContexts | auto-enable-contexts | true | Worker | Worker |

If false the contexts are registered disabled in httpd, they need to be enabled via the enable() mbean method, jboss-cli command or via mod_cluster_manager web console on Apache HTTP Server.

**Stop context timeout**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| stopContextTimeout | stop-context-timeout | 10 s | Worker | Worker |

The amount of time in seconds for which to wait for a clean shutdown of a context (completion of pending requests for a distributable context; or destruction/expiration of active sessions for a non-distributable context).

**Stop context timeout unit**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| stopContextTimeoutUnit | None | TimeUnit.SECONDS | Worker | Worker |

Tomcat allows for configuring an arbitrary TimeUnit for Stop context timeout




