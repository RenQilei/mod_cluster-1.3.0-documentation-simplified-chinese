# 第七章 AS7 modcluster subsystem 配置

## 7.1. JBoss AS7 中的 ModCluster subsystem

mod_cluster 集成通过 modcluster subsystem 完成。AS7 中仅 1.1.x 及以后的版本支持。

## 7.2. ModCluster subsystem 最小配置

The minimal configuration is having the modcluster schemaLocation in the schemaLocation list:

```
urn:jboss:domain:modcluster:1.0 jboss-mod-cluster.xsd
```

and the extension module in the extensions list:

```
<extension module="org.jboss.as.modcluster"/>
```

and subsystem declaration like:

```
<subsystem xmlns="urn:jboss:domain:modcluster:1.0"/>
```

With that configuration modcluster will listen for advertise on ```224.0.1.105:23364``` use the ```simple-load-provider``` with a load factor of 1.

## 7.3. ModCluster Subsystem configuration

### 7.3.1. mod-cluster-config Attributes

The attributs correspond to the properties [缺少锚链接]

代理发现配置（Proxy Discovery Configuration）

| 属性(Attribute) | 标签(Property) | 默认(default) |
| -- | -- | -- |
| proxy-list | proxyList | *None* |
| proxy-url | proxyURL | *None* |
| advertise | advertise | *true* |
| advertise-security-key | advertiseSecurityKey | *None* |
| excluded-contexts | excludedContexts | *None* |
| auto-enable-contexts | autoEnableContexts | *true* |
| stop-context-timeout | stopContextTimeout | *10 seconds (in seconds)* |
| socket-timeout | nodeTimeout | *20 seconds (in milli seconds)* |

代理配置（Proxy Configuration）

| 属性(Attribute) | 标签(Property) | 默认(default) |
| -- | -- | -- |
| sticky-session | stickySession | *true* |
| sticky-session-remove | stickySessionRemove | *false* |
| sticky-session-force | stickySessionForce | *true* |
| node-timeout | workerTimeout | *-1* |
| max-attempts | maxAttempts | *1* |
| flush-packets | flushPackets | *false* |
| flush-wait | flushWait | *-1* |
| ping | ping | *10* |
| smax | smax | *-1 it uses default value* |
| ttl | ttl | *-1 it uses default value* |
| domain | loadBalancingGroup | *None* |
| load-balancing-group | loadBalancingGroup | *None* |

SSL 配置
SSL 配置部分也需要被添加在这里。

### 7.3.2. simple-load-provider 属性

简单负载提供者（simple load provider）总是发送相同的负载因素。默认是 1。如下：

```
<subsystem xmlns="urn:jboss:domain:modcluster:1.0">
  <mod-cluster-config>
    <simple-load-provider factor="1"/>
  </mod-cluster-config>
</subsystem>
```

| 属性(Attribute) | 标签(Property) | 默认(default) |
| -- | -- | -- |
| factor | LoadBalancerFactor | *1* |

### 7.3.3. 动态负载提供者（dynamic-load-provider）属性

动态负载提供者允许拥有负载度量衡（load-metric）和自定义负载度量衡（custom-load-metric）。如：

```
<subsystem xmlns="urn:jboss:domain:modcluster:1.0">
  <mod-cluster-config advertise-socket="mod_cluster">
    <dynamic-load-provider history="10" decay="2">
      <load-metric type="cpu" weight="2" capacity="1"/>
      <load-metric type="sessions" weight="1" capacity="512"/>
      <custom-load-metric class="mypackage.myclass" weight="1" capacity="512">
        <property name="myproperty" value="myvalue" />
        <property name="otherproperty" value="othervalue" />
      </custom-load-metric>
    </dynamic-load-provider>
  </mod-cluster-config>
</subsystem>
```

| 属性(Attribute) | 标签(Property) | 默认(default) |
| -- | -- | -- |
| history | history | *512* |
| decay | decayFactor | *512* |

### 7.3.4. 负载度量衡（load-metric）配置

The load-metric are the "classes" collecting information to allow computation of the load factor sent to httpd

负载度量衡是收集信息并将负载因子的计算结果发送到 httpd 的“类（classes）”。

| 属性(Attribute) | 标签(Property) | 默认(default) |
| -- | -- | -- |
| type | A Server-Side Load Metrics | *Mandatory* |
| weight | weight | *9* |
| capacity | capacity | *512* |

#### 7.3.4.1. 支持的负载度量衡类型

| 类型(type) | 相应服务器端负载度量衡(Corresponding Server-Side Load Metric) |
| -- | -- |
| cpu | *AverageSystem [缺少锚链接]* LoadMetric |
| mem | *SystemMemoryUsage [缺少锚链接]* LoadMetric |
| heap | *HeapMemoryUsage [缺少锚链接]* LoadMetric |
| sessions | *ActiveSessions [缺少锚链接]* LoadMetric |
| requests | *RequestCount [缺少锚链接]* LoadMetric |
| send-traffic | *SendTraffic [缺少锚链接]* LoadMetric |
| receive-traffic | *ReceiveTraffic [缺少锚链接]* LoadMetric |
| busyness | *BusyConnectors [缺少锚链接]* LoadMetric |
| connection-pool | *ConnectionPoolUsage [缺少锚链接]* LoadMetric |

### 7.3.5. custom-load-metric Configuration

The custom-load-metric are for user defined "classes" collecting information. They are like the load-metric except type is replaced by class:

| 属性(Attribute) | 标签(Property) | 默认(default) |
| -- | -- | -- |
| class | Name of your class | *Mandatory* |

### 7.3.6. load-metric Configuration with the jboss admin console

The load-metric have 4 commands to add / remove metrics

#### 7.3.6.1. add-metric

Allows to add a ```load-metric``` to the ```dynamic-load-provider```

For example:

```
./:add-metric(type=cpu, weight=2, capacity=1)
```

#### 7.3.6.2. remove-metric

Allows to remove a ```load-metric``` from the ```dynamic-load-provider```

For example:

```
./:remove-metric(type=cpu)
```

#### 7.3.6.3. add-custom-metric

Allows to add a ```load-custom-metric``` to the ```dynamic-load-provider```

For example:

```
./:add-custom-metric(class=myclass, weight=2, capacity=1, property=[("pool" => "mypool"),("var" => "myvariable")])
```

#### 7.3.6.4. remove-custom-metric

Allows to remove a ```load-custom-metric``` from the ```dynamic-load-provider```

For example:

```
./:remove-custom-metric(class=myclass)
```







