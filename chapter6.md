# 第六章 服务器端（worker-side）配置

## 6.1. JBoss AS

参见 AS7 [缺少锚链接]，mod_cluster 在 AS7 中已通过 modcluser 子系统（subsystem）获得支持。

在其他 AS 版本中 mod_cluster 的配置在下面的文件中：```$JBOSS_HOME/server/$PROFILE/deploy/mod_cluster.sar/META-INF/mod_cluster-jbossbeans.xml```

mod_cluster 服务器端配置的入口点是 ```ModClusterListener bean```，来指派 web 容器（如 JBoss Web）的特定事件到一个与事件处理无关的容器。

总的来说，```ModClusterListener bean```是这样定义的：

1. 一个从 web 容器处理事件的 ContainerEventHandler。
2. 一个 JBoss mbean 服务器的索引。

例如：

```
<bean name="ModClusterListener" class="org.jboss.modcluster.container.jbossweb.JBossWebEventHandlerAdapter">
  <constructor>
    <parameter class="org.jboss.modcluster.container.ContainerEventHandler">
      <inject bean="ModClusterService"/>
    </parameter>
    <parameter class="javax.management.MBeanServer">
      <inject bean="JMXKernel" property="mbeanServer"/>
    </parameter>
  </constructor>
</bean>
```

### 6.1.1. Non-clustered mode

非 cluster 模式（non-clustered mode）中，每个 JBoss AS 节点会直接与负载均衡器通信，而不会相互间通信。非 cluster 模式通过 ```ModClusterService bean``` 来配置。

总的来说, ```ModClusterService bean``` 定义如下:

1. 一个包含 mod_cluster 配置属性 [缺少锚链接]的对象。
2. 一个负责为节点计算负载均衡因素的对象。细节将在 服务器端负载度量衡 [缺少锚链接]这个一章节中描述。

例如：

```
<bean name="ModClusterService" class="org.jboss.modcluster.ModClusterService" mode="OnDemand">
annotation>
  <constructor>
    <parameter class="org.jboss.modcluster.config.ModClusterConfig">
      <inject bean="ModClusterConfig"/>
    </parameter>
    <parameter class="org.jboss.modcluster.load.LoadBalanceFactorProvider">
      <inject bean="DynamicLoadBalanceFactorProvider"/>
    </parameter>
  </constructor>
</bean>
```

### 6.1.2. 配置属性

```ModClusterConfig bean``` 列举了 mod_cluster 使用的配置属性。配置属性的完整列表和它们的默认值参见服务器端配置属性一章。

例如：

```
<bean name="ModClusterConfig" class="org.jboss.modcluster.config.ModClusterConfig" mode="On Demand">
  <!-- Specify configuration properties here -->
</bean>
```

### 6.1.3. 连接器（Connectors）

像 mod_jk 和 mod_proxy_balancer 一样，mod_cluster 在你的```sever.xml```中需要一个连接器来转发 web 请求。不同于 mod_jk 和 mod_cluster_balancer 的是，mod_cluster 并不限制于 AJP，而是也可以使用 HTTP。尽管 AJP 整体上可以更快，但一个 HTTP 的连接器则可以选择通过 SSL 来加密。如果你在你的```server.xml```里定义了多个可能的连接器，mod_cluster 则使用如下算法来选择它们：

1. 如有一个本地（APR）AJP 连接器可用，则使用。
2. 如 AJP 连接器可用，则使用。
3. 否则，选择 HTTP 连接器，并使用最大量线程数。

