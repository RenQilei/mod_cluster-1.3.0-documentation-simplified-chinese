# 第七章 AS7 modcluster subsystem 配置

## 7.1. JBoss AS7 中的 ModCluster subsystem

mod_cluster 集成（integration）通过 modcluster subsystem 完成。AS7 中仅 1.1.x 及以后的版本支持。

## 7.2. ModCluster subsystem 最小配置

The minimal configuration is having the modcluster schemaLocation in the schemaLocation list:

最小配置包括了 modcluster ```schemaLocation``` 列表中的 ```schemaLocation```。

```
urn:jboss:domain:modcluster:1.0 jboss-mod-cluster.xsd
```

以及```扩展（extension）```列表中的 ```扩展模块（extension module）``` :

```
<extension module="org.jboss.as.modcluster"/>
```

以及 ```subsystem``` 申明如:

```
<subsystem xmlns="urn:jboss:domain:modcluster:1.0"/>
```

像这样配置后 modcluster 将会在 ```224.0.1.105:23364``` 上使用负载因子为 1 的 ```simple-load-provider``` 来监听广告。

## 7.3. ModCluster Subsystem configuration

### 7.3.1. mod-cluster-config Attributes

属性（attributs）对应的标签（properties） [缺少锚链接]

**代理发现配置（Proxy Discovery Configuration）**

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

**代理配置（Proxy Configuration）**

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

**SSL 配置**
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

### 7.3.5. 自定义负载度量衡（custom-load-metric）配置

自定义负载度量衡是为用户提供定义收集信息的“类”。它们和负载度量衡一样，除了类型（type）被类（class）替换了：

自定义

| 属性(Attribute) | 标签(Property) | 默认(default) |
| -- | -- | -- |
| class | Name of your class | *Mandatory* |

### 7.3.6. 和 jboss 管理控制台（admin console）相关的负载度量衡配置

负载度量衡有4个指令来增加/移除度量衡。

#### 7.3.6.1. add-metric

允许添加一个 ```load-metric``` 到 ```dynamic-load-provider```

例如:

```
./:add-metric(type=cpu, weight=2, capacity=1)
```

#### 7.3.6.2. remove-metric

允许从 ```dynamic-load-provider``` 移除一个 ```load-metric```

例如:

```
./:remove-metric(type=cpu)
```

#### 7.3.6.3. add-custom-metric

允许添加一个 ```load-custom-metric``` 到 ```dynamic-load-provider```

例如:

```
./:add-custom-metric(class=myclass, weight=2, capacity=1, property=[("pool" => "mypool"),("var" => "myvariable")])
```

#### 7.3.6.4. remove-custom-metric

允许从 ```dynamic-load-provider``` 移除 ```load-custom-metric```

例如:

```
./:remove-custom-metric(class=myclass)
```