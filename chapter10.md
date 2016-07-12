
# 第十章 服务器端负载度量衡

mod_cluster 一个主要的特性就是能够使用服务器端负载度量衡的能力来决定如何实现对请求最优的均衡。

`DynamicLoadBalanceFactorProvider` bean 就是通过一组确定的负载度量衡来计算节点的负载均衡因素的。

```
<bean name="DynamicLoadBalanceFactorProvider" class="org.jboss.modcluster.load.impl.DynamicLoadBalanceFactorProvider" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=LoadBalanceFactorProvider",exposedInterface=org.jboss.modcluster.load.impl.DynamicLoadBalanceFactorProviderMBean.class)</annotation>
  <constructor>
    <parameter>
      <set elementClass="org.jboss.modcluster.load.metric.LoadMetric">
        <inject bean="BusyConnectorsLoadMetric"/>
        <inject bean="HeapMemoryUsageLoadMetric"/>
      </set>
    </parameter>
  </constructor>
  <property name="history">9</property>
  <property name="decayFactor">2</property>
</bean>
```

负载度量衡可以关联 ```权重（weight）``` 和 ```承载力（capacity）``` 来配置。

权重（默认为1）表明了这个度量衡相对于其它的重要性。例如，权重为2的度量衡对整体负载因素的影响将是权重为1的度量衡的两倍。

度量衡的承载力则提供2个功能：

* 标准化各度量衡的负载值。一些负载度量衡中，承载力已经反映在了负载值上。度量衡的承载力应该如此配置：$$0 \leq {load \over capacity} \leq 1$$。
* 考虑一些优于其它节点的节点。通过给每个节点上度量衡的承载力设置不同的值，代理将会优先考虑拥有更高承载力的节点，因为他们将会返回更小的负载值。这给节点权重增添了有意思的粒度级别。考虑一个有两个节点的集群，一个有更多的内存（memory），一个有更快的CPU；此时有两个度量衡，一个基于内存（memory-based），一个基于CPU（CPU-based）。基于内存的度量衡方面，第一个节点将会提供一个比第二个节点更高的负载承载力。而基于CPU的度量衡方面，第二个节点则会提供一个比第一个节点更高的节点承载力。

每个负载度量衡给整体节点的负载因素带来了一个值。每个度量衡的负载因素根据它们的权重进行累计。

总的来说，给定度量衡的负载因素贡献为：$${{load \over capacity} * weight} \over total weight$$。

```DynamicLoadBalanceFactorProvider``` 有适用于每个度量衡返回的负载的时间衰减功能。与过去负载值相关的总负荷可以通过下面的公式来表达：

$$
L = (L_0 + {L_1 \over D} + {L_2 \over D^2} + {L_3 \over D^3} + … + {L_H \over D^H}) * (1 + D + D^2 + D^3 + … D^H)
$$

或更简洁：

$$
L = (\sum_{i=0}^H {L_i \over D^i}) * (\sum_{i=0}^H {D^i})
$$

其中 $$D = decayFactor, H = history$$.

设置 ```history = 0``` 则有效地禁用了时间衰减功能，在计算负载均衡因素时只有每个度量衡的当前负载会被考虑。

mod_cluster 负载均衡器希望负载因素是一个0到100间的整数，0表示最大负载而100表示零负载。因此，发送到负载均衡器最终的负载因素为：

$$
L_{Final} = 100 - (L * 100)
$$

当然你可以自由的写你自己的负载度量衡，下面是随时可用的 ```LoadMetrics```：

## 10.1. Web 容器度量衡（Web Container metrics）

### 10.1.1. 活跃会话负载度量衡（Active Sessions Load Metric）

* 需要一个明确的承载力（capacity）
* 使用 ```SessionLoadMetricSource{.code}``` 来查询会话管理器（session managers）
* 与 mod_jk 中的 ```method=S``` 类似

例如，在 JBoss AS 5 中：

```
<bean name="ActiveSessionsLoadMetric" class="org.jboss.modcluster.load.metric.impl.ActiveSessionsLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=ActiveSessionsLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter><inject bean="SessionLoadMetricSource"/></parameter>
  </constructor>
  <property name="capacity">1000</property>
</bean>
<bean name="SessionLoadMetricSource" class="org.jboss.modcluster.load.metric.impl.SessionLoadMetricSource" mode="On Demand">
  <constructor>
    <parameter class="javax.management.MBeanServer">
      <inject bean="JMXKernel" property="mbeanServer"/>
    </parameter>
  </constructor>
</bean>
```

### 10.1.2. 忙连接器负载度量衡（Busy Connectors Load Metric）

* 返回线程池中是忙服务请求（busy servicing requests）的连接器线程（connector thread）的比例
* 使用 ```ThreadPoolLoadMetricSource``` 来查询连接线程
* 与 mod_jk 中的 ```method=B``` 类似
* ```BusyConnectorsLoadMetric.java```

例如，在 JBoss AS 5 中：

```
<bean name="BusyConnectorsLoadMetric" class="org.jboss.modcluster.load.metric.impl.BusyConnectorsLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=BusyConnectorsLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter><inject bean="ThreadPoolLoadMetricSource"/></parameter>
  </constructor>
</bean>
<bean name="ThreadPoolLoadMetricSource" class="org.jboss.modcluster.load.metric.impl.ThreadPoolLoadMetricSource" mode="On Demand">
  <constructor>
    <parameter class="javax.management.MBeanServer">
      <inject bean="JMXKernel" property="mbeanServer"/>
    </parameter>
  </constructor>
</bean>
```

### 10.1.3 接收流量负载度量衡（Receive Traffic Load Metric）

* 返回一个流入请求 POST 的流量，以 KB/sec 为单位（应用需要读 POST 数据）
* 需要一个明确的承载力（capacity）
* 使用 ```RequestProcessorLoadMetricSource``` 来查询请求处理器（request processors）
* 与 mod_jk 中的 ```method=T``` 类似

例如，在 JBoss AS 5 中：

```
<bean name="ReceiveTrafficLoadMetric" class="org.jboss.modcluster.load.metric.impl.ReceiveTrafficLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=ReceiveTrafficLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter class="org.jboss.modcluster.load.metric.impl.RequestProcessorLoadMetricSource">
      <inject bean="RequestProcessorLoadMetricSource"/>
    </parameter>
  </constructor>
  <property name="capacity">1024</property>
</bean>
<bean name="RequestProcessorLoadMetricSource" class="org.jboss.modcluster.load.metric.impl.RequestProcessorLoadMetricSource" mode="On Demand">
  <constructor>
    <parameter class="javax.management.MBeanServer">
      <inject bean="JMXKernel" property="mbeanServer"/>
    </parameter>
  </constructor>
</bean>
```

### 10.1.4. 发送流量负载度量衡（Send Traffic Load Metric）

* 返回流出请求的流量，以 KB/sec 为单位
* 需要一个明确的承载力（capacity）
* 使用 ```RequestProcessorLoadMetricSource{.code}``` 来查询请求处理器（request processors）
* 与 mod_jk 中的 ```method=T``` 类似

例如，在 JBoss AS 5 中：

```
<bean name="SendTrafficLoadMetric" class="org.jboss.modcluster.load.metric.impl.SendTrafficLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=SendTrafficLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter class="org.jboss.modcluster.load.metric.impl.RequestProcessorLoadMetricSource">
      <inject bean="RequestProcessorLoadMetricSource"/>
    </parameter>
  </constructor>
  <property name="capacity">512</property>
</bean>
```

### 10.1.5. 请求数负载度量衡（Request Count Load Metric）

* 返回每秒请求（requests/sec）的数量
* 需要一个明确的承载力（capacity）
* 使用 ```RequestProcessorLoadMetricSource``` 来查询请求处理器（request processors）
* 与 mod_jk 中的 ```method=R``` 类似 

例如，在 JBoss AS 5 中：

```
<bean name="RequestCountLoadMetric" class="org.jboss.modcluster.load.metric.impl.RequestCountLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=RequestCountLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter class="org.jboss.modcluster.load.metric.impl.RequestProcessorLoadMetricSource">
      <inject bean="RequestProcessorLoadMetricSource"/>
    </parameter>
  </constructor>
  <property name="capacity">1000</property>
</bean>
```

## 10.2. 系统/JVM 度量衡（System/JVM metrics）

### 10.2.1. 平均系统负载度量衡（Average System Load Metric）

* 返回 CPU 负载
* 需要 Java 1.6+
* 使用 ```OperatingSystemLoadMetricSource``` 整体地读属性（attributes）
* Windows 上不可用
* ```AverageSystemLoadMetric.java```

例如，在 JBoss AS 5 中：

```
<bean name="AverageSystemLoadMetric" class="org.jboss.modcluster.load.metric.impl.AverageSystemLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=AverageSystemLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter><inject bean="OperatingSystemLoadMetricSource"/></parameter>
  </constructor>
</bean>
<bean name="OperatingSystemLoadMetricSource" class="org.jboss.modcluster.load.metric.impl.OperatingSystemLoadMetricSource" mode="On Demand">
</bean>
```

### 10.2.2. 堆内存使用量负载度量衡（Heap Memory Usage Load Metric）

* 返回堆内存使用量作为最大堆尺寸（max heap size）的比例

例如，在 JBoss AS 5 中：

```
<bean name="HeapMemoryUsageLoadMetric" class="org.jboss.modcluster.load.metric.impl.HeapMemoryUsageLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=HeapMemoryUsageLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
</bean>
```

## 10.3. 其它度量衡

### 10.3.1. 连接池使用量负载度量衡（Connection Pool Usage Load Metric）

* 返回连接池（connection pool）中正在使用的连接（connection）的比例
* 使用 ```ConnectionPoolLoadMetricSource``` 来查询 JCA 连接池（connection pools）

例如，在 JBoss AS 5 中：

```
<bean name="ConnectionPoolUsageMetric" class="org.jboss.modcluster.load.metric.impl.ConnectionPoolUsageLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=ConnectionPoolUsageLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter><inject bean="ConnectionPoolLoadMetricSource"/></parameter>
  </constructor>
</bean>
<bean name="ConnectionPoolLoadMetricSource" class="org.jboss.modcluster.load.metric.impl.ConnectionPoolLoadMetricSource" mode="On Demand">
  <constructor>
    <parameter class="javax.management.MBeanServer">
      <inject bean="JMXKernel" property="mbeanServer"/>
    </parameter>
  </constructor>
</bean>
```