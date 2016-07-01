# 第六章 服务器端（worker-side）配置

## 6.1. JBoss AS

mod_cluster is supported in AS7 via the modcluster subsystem See AS7 [缺少锚链接].

In other AS version mod_cluster's configuration resides within the following file:```$JBOSS_HOME/server/$PROFILE/deploy/mod_cluster.sar/META-INF/mod_cluster-jbossbeans.xml```file.

The entry point for mod_cluster's server-side configuration is the ModClusterListener bean, which delegates web container (i.e. JBoss Web) specific events to a container agnostic event handler.

In general, the ModClusterListener bean defines:

1. A ContainerEventHandler in which to handle events from the web container.
2. A reference to the JBoss mbean server.

e.g.

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

In non-clustered mode, each JBoss AS node communicates with the load balancer directly, and do not communicate with each other. Non-clustered mode is configured via the ```ModClusterService``` bean.

In general, the ```ModClusterService``` bean defines:

1. An object containing mod_cluster's configuration properties [缺少锚链接].
2. An object responsible for calculating the load balance factor for this node. This is described in detail in the chapter entitled Server-Side Load Metrics [缺少锚链接].

e.g.

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

### 6.1.2. Configuration Properties

The ModClusterConfig bean enumerates the configuration properties used by mod_cluster. For the complete list of configuration properties and their default values, see the chapter entitled Server-side Configuration Properties.

e.g.

```
<bean name="ModClusterConfig" class="org.jboss.modcluster.config.ModClusterConfig" mode="On Demand">
  <!-- Specify configuration properties here -->
</bean>
```

### 6.1.3. Connectors

Like mod_jk and mod_proxy_balancer, mod_cluster requires a connector in your server.xml to which to forward web requests. Unlike mod_jk and mod_proxy_balancer, mod_cluster is not confined to AJP, but can use HTTP as well. While AJP is generally faster, an HTTP connector can optionally be secured via SSL. If multiple possible connectors are defined in your server.xml, mod_cluster uses the following algorithm to choose between them:

1. If an native (APR) AJP connector is available, use it.
2. If an AJP connector is available, use it.
3. Otherwise, choose the HTTP connector with the highest max threads.

