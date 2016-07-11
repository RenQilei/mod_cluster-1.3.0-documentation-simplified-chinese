# 第九章 服务器端配置属性

下面的表格列举了一台应用服务器（application server）节点上可选的配置属性。这些属性的配置的位置取决于 [mod_cluster 是被如何配置的](chapter6.md)。

## 9.1. 代理发现配置（Proxy Discovery Configuration）

应用希望接收到 AJP 连接的代理列表是通过定义在```proxyList```配置属性里的地址静态的定义，或者通过广告机制动态的发现。使用一个特殊的 mod_advertise 模块，代理能够通过周期性的广播包含自己地址/端口（address/port）的组播消息（multicast message）来广告它们的存在。这个功能通过```advertise```配置属性来启用。如果配置了监听，服务器就可以获知代理的存在，然后告知代理自己的存在，并相应地更新自己的配置。这将代理和服务器从必须配置静态的（static）、环境指定的（environment-specific）值中解放出来。

> 翻译注：
> 
> * **Location**: 位置
> * **Scope**: 范围
> * **Worker**: 应用服务器或 web 服务器，指部署 Tomcat/AS 7/Wildfly 等的服务器

**会话流失策略（Session draining strategy）**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| -- | -- | -- | -- | -- |
| sessionDrainingStrategy | session-draining-strategy | DEFAULT | Worker | Worker |

表明会话流失策略在 web 应用取消部署期间的使用。有三个可能的值：

* ```DEFAULT```: 仅在 web 服务器为不可分配（non-distributable）时，在 web 应用取消部署前流失会话。
* ```ALWAYS```: 在 web 应用取消部署前总是流失会话，甚至是可分配的 web 服务器。
* ```NEVER```: 在 web 应用取消部署前不流失会话，甚至是可不分配的 web 服务器。

**代理（Proxies）**

| Tomcat attribute | AS7 attribute | Wildfly attribute |Default | Location | Scope |
| -- | -- | -- | -- | -- | -- |
| proxyList | proxy-list | proxies | None | Worker | Worker |

* Tomcat/AS7: 定义一个节点将初始化通信的 httpd 代理的列表，以逗号分隔。值应该是以下格式: ```address1:port1,address2:port2```。使用默认配置，这些属性（property）能通过 jboss.mod_cluster.proxyList 系统属性（system property）修改。
* Wildfly: 在 Wildfly 中，modcluster subsystem 元素的属性（attribute）```proxy-list```已经被弃用了。为此，它使用了输出套接字绑定（output socket binding）。下面的例子利用了 ```jboss-cli.sh```，如：
  * 添加一个套接字绑定: ```/socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=my-proxies:add(host=10.10.10.11,port=3333)```
  * 添加一个套接字绑定到 modcluster subsystem: ```/subsystem=modcluster/mod-cluster-config=configuration:write-attribute(name=proxies, value="my-proxies")```

**剔除上下文根（Excluded contexts）**

| Tomcat attribute | AS7/Wildfly attribute | Wildfly Default | Tomcat/AS7 Default | Location | Scope |
| -- | -- | -- | -- | -- | -- |
| excludedContexts | excluded-contexts | None | ROOT, admin-console, invoker, bossws, jmx-console, juddi, web-console | Worker | Worker |

用于从 httpd 注册中剔除的上下文根列表，格式为: *host1:context1,host2:context2,host3:context3* 如果没有指定的主机（host），则它会使用服务器的默认主机（如 localhost）。“ROOT” 表明了 root 上下文根。使用默认的配置，这个属性（property）可以通过 ```jboss.mod_cluster.excludedContexts``` 系统属性（system prperty）修改。

**自动启用上下文根（Auto Enable Contexts）**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| -- | -- | -- | -- | -- |
| autoEnableContexts | auto-enable-contexts | true | Worker | Worker |

如果在 httpd 中禁用了上下文根，它们需要通过 ```enable() mbean``` 方法、```jboss-cli```命令，或者通过 在 Apache HTTP 服务器上的 ```mod_cluster_manager``` web 控制台。

**停止上下文根超时（Stop context timeout）**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| -- | -- | -- | -- | -- |
| stopContextTimeout | stop-context-timeout | 10 s | Worker | Worker |

等待上下文根的纯净关闭的时间量，以秒为单位（为可分配的上下文根的等待请求；或者为不可分配的上下文根的活跃会话的毁灭/过期）。

**停止上下文根超时单元（Stop context timeout unit）**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| stopContextTimeoutUnit | None | TimeUnit.SECONDS | Worker | Worker |

Tomcat 允许配置任意的 TimeUnit 来停止上下文根超时。

**代理 URL（Proxy URL）**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| -- | -- | -- | -- | -- |
| proxyURL | proxy-url | / | Worker | Balancer |

如果定义了，这个值将被视作为 MCMP 命令的 URL。

**套接字超时（Socket timeout）**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| -- | -- | -- | -- | -- |
| socketTimeout | socket-timeout | 20 s | Worker | Worker |

在超时并标记代理为错误状态前，等待多久从 httpd 代理到 MCMP 命令的回复。

| Tomcat/AS7/Wildfly attribute | Default | Location | Scope |
| -- | -- | -- | -- |
| advertise	| true, if proxyList is undefined, false otherwise | Worker | Worker |

如果启用，httpd 代理将会通过接收组播宣告信息被自动发现。在复杂环境（concert）或配有静态代理的地方均可使用。

**广告套接字组（Advertise socket group）**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| -- | -- | -- | -- | -- |
| advertiseGroupAddress | advertise-socket | 224.0.1.105 | Worker | Worker |
| advertisePort | in advertise-socket | 23364 | Worker | Worker |

UDP 组播 ```address:port``` 来监听 httpd 代理的组播广告。需要小心你的均衡器/应用服务器发送/接收的实际接口。参见 Apache HTTP 服务器行为的 MODCLUSTER-487 和 Tomcat 的 caveat 的 MODCLUSTER-495。

**广告安全秘钥（Advertise security key）**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| -- | -- | -- | -- | -- |
| advertiseSecurityKey | advertise-security-key | None | Worker | Balancer |

如果特别指定，httpd 代理广告校验码（用这个值作为盐值）将需要被在服务器端验证。这个选项不确保你的安装文件的安全，它不替换本身的 SSL 配置。它只是确保仅确定的应用服务器可以与确定的均衡器通信。要小心 MODCLUSTER-446。

**广告线程工厂（Advertise thread factory）**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| -- | -- | -- | -- | -- |
| advertiseThreadFactory | None | Executors.defaultThreadFactory() | Worker | Worker |

线程工厂用来创建后台广告监听器。

**JVMRoute 工厂（JVMRoute factory）**

| Tomcat attribute | AS7/Wildfly attribute | Default | Location | Scope |
| -- | -- | -- | -- | -- |
| jvmRouteFactory | None | new SystemPropertyJvmRouteFactory(new UUIDJvmRouteFactory(), “jboss.mod_cluster.jvmRoute”) | Worker | Worker |

如果在 Tomcat 的 server.xml 中没有指定，则为确定节点的 jvm 路由定义策略。默认工厂首先查看 ```jboss.mod_cluster.jvmRoute``` 系统属性（property）。如果这个系统属性（property）没有被定义，则 jvm 被分配一个 UUID。带有 Undertow web subsystem 的 Wildfly 使用 Undertow 的实例ID（instance-id）或 ```jboss.mod_cluster.jvmRoute``` 系统属性（property）或 UUID。

## 9.2. 代理配置（Proxy Configuration）

当代理通过广告机制被检测到时或当修复错误代理配置重设期间，下面的配置值会在服务器启动时发送给代理。

| Attribute | AS7 Attribute | Default | Scope | Description |
| -- | -- | -- | -- | -- |
| stickySession | sticky-session | true | Balancer | 如果可能，用以指明后续给定会话（session）的请求是否应该被路由到相同节点。 |
| stickySessionRemove | sticky-session-remove | false | Balancer | Indicates whether the httpd proxy should remove session stickiness in the event that the balancer is unable to route a request to the node to which it is stuck. This property is ignored if stickySession is false. |
| stickySessionForce | sticky-session-force | false | Balancer | Indicates whether the httpd proxy should return an error in the event that the balancer is unable to route a request to the node to which it is stuck. This property is ignored if stickySession is false. |
| workerTimeout | worker-timeout | -1 | Balancer | Number of seconds to wait for a worker to become available to handle a request. When no workers of a balancer are usable, mod_cluster will retry after a while (workerTimeout/100). That is timeout in the balancer mod_proxy documentation. A value of -1 indicates that the httpd will not wait for a worker to be available and will return an error if none is available. |
| maxAttempts | max-attempts | 1 | Balancer | Number of times an httpd proxy will attempt to send a given request to a worker before giving up. The minimum value is 1, meaning try only once. (Note that mod_proxy default is also 1: no retry). |
| flushPackets | flush-packets | false | Node | Enables/disables packet flushing |
| flushWait | flush-wait | -1 | Node | Time to wait before flushing packets in milliseconds. A value of -1 means wait forever. |
| ping | ping | 10 | Node | Time (in seconds) in which to wait for a pong answer to a ping |
| smax | smax | Determined by httpd configuration | Node | Soft maximum idle connection count (that is the smax in worker mod_proxy documentation). The maximum value depends on the httpd thread configuration (ThreadsPerChild or 1). |
| ttl | ttl | 60 | Node | Time to live (in seconds) for idle connections above smax |
| nodeTimeout | node-timeout | -1 | Node | Timeout (in seconds) for proxy connections to a node. That is the time mod_cluster will wait for the back-end response before returning error. That corresponds to timeout in the worker mod_proxy documentation. A value of -1 indicates no timeout. Note that mod_cluster always uses a cping/cpong before forwarding a request and the connectiontimeout value used by mod_cluster is the ping value. |
| balancer | balancer | mycluster | Node | The balancer name |
| loadBalancingGroup | domain load-balancing-group | None | Node | If specified, load will be balanced among jvmRoutes withing the same load balancing group. A loadBalancingGroup is conceptually equivalent to a mod_jk domain directive. This is primarily used in conjunction with partitioned session replication (e.g. buddy replication). |

> **Note:**
> 
> When nodeTimeout is not defined the ProxyTimeout directive Proxy is used. If ProxyTimeout is not defined the server timeout (Timeout) is used (default 300 seconds). nodeTimeout, ProxyTimeout or Timeout is set at the socket level.

## 9.3. SSL Configuration

The communication channel between application servers and httpd proxies uses HTTP by default. This channel can be secured using HTTPS by setting the ssl property to true.

> **Note:**
> 
> This HTTP/HTTPS channel should not be confused with the processing of HTTP/HTTPS client requests.

| Attribute | AS7 Attribute | Default | Description |
| -- | -- | -- | -- |
| ssl | None | false | Should connection to proxy use a secure socket layer |
| sslCiphers | cipher-suite | *The default JSSE cipher suites* | Overrides the cipher suites used to initialize an SSL socket ignoring any unsupported ciphers |
| sslProtocol | protocol | TLS (ALL in AS7) | Overrides the default SSL socket protocol. |
| sslCertificateEncodingAlgorithm | None | The default JSSE key manager algorithm | The algorithm of the key manager factory |
| sslKeyStore | certificate-key-file | System.getProperty(“user.home”) + “/.keystore” | The location of the key store containing client certificates |
| sslKeyStorePassword | password | changeit | The password granting access to the key store (and trust store in AS7) |
| sslKeyStoreType | None | JKS | The type of key store |
| sslKeyStoreProvider | None | The default JSSE security provider | The key store provider |
| sslTrustAlgorithm | None | The default JSSE trust manager algorithm | The algorithm of the trust manager factory |
| sslKeyAlias | key-alias | | The alias of the key holding the client certificates in the key store |
| sslCrlFile | ca-revocation-url | | Certificate revocation list |
| sslTrustMaxCertLength | None | 5 | The maximum length of a certificate held in the trust store |
| sslTrustStore | None | javax.net.ssl.trustStorePassword | The location of the file containing the trust store |
| sslTrustStorePassword | None | javax.net.ssl.trustStore | The password granting access to the trust store. |
| sslTrustStoreType | None | javax.net.ssl.trustStoreType | The trust store type |
| sslTrustStoreProvider | None | javax.net.ssl.trustStoreProvider | The trust store provider |

## 9.4. JBoss Web 和 Tomcat 的负载均衡

mod_cluster 在配置 JBoss Web standalone 和 Tomcat 时使用的额外配置属性（property）。

| Attribute | Default | Description |
| -- | -- | -- |
| loadMetricClass | org.jboss.modcluster.load.metric.impl.BusyConnectorsLoadMetric | Class name of an object implementing org.jboss.load.metric.LoadMetric |
| loadMetricCapacity | 1 | The capacity of the load metric defined via the loadMetricClass property |
| loadHistory | 9 | The number of historic load values to consider in the load balance factor computation. |
| loadDecayFactor | 2 | The factor by which a historic load values should degrade in significance. |


