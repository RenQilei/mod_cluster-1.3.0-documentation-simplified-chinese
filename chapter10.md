
# 第十章 服务器端负载度量衡

A major feature of mod\_cluster is the ability to use server-side load metrics to determine how best to balance requests.

The `DynamicLoadBalanceFactorProvider` bean computes the load balance factor of a node
from a defined set of load metrics.

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

Load metrics can be configured with an associated weight and capacity.

The weight \(default is 1\) indicates the significance of a metric with respect to the other metrics. For example, a metric of weight 2 will have twice the impact on the overall load factor than a metric of weight 1.

The capacity of a metric serves 2 functions:

* To normalize the load values from each metric. In some load metrics, capacity is already reflected in the load values. The capacity of a metric should be configured such that 0 &lt;= \(load \/ capacity\) &gt;= 1.
* To favor some nodes over others. By setting the metric capacities to different values on each node, proxies will effectively favor nodes with higher capacities, since they will return smaller load values. This adds an interesting level of granularity to node weighting. Consider a cluster of two nodes, one with more memory, and a second with a faster CPU; and two metrics, one memory-based and the other CPU-based. In the memory-based metric, the first node would be given a higher load capacity than the second node. In a CPU-based metric, the second node would be given a higher load capacity than the first node.

Each load metric contributes a value to the overall load factor of a node. The load factors from each metric are aggregated according to their weights.

In general, the load factor contribution of given metric is: \(load \/ capacity\) \* weight \/ total weight.

The DynamicLoadBalanceFactorProvider applies a time decay function to the loads returned by each metric. The aggregate load, with respect to previous load values, can be expressed by the following formula:


$$
L = (L_0 + {L_1 \over D} + {L_2 \over D^2} + {L_3 \over D^3} + … + {L_H \over D^H}) * (1 + D + D^2 + D^3 + … D^H)
$$


… or more concisely as:


$$
L = (\sum_i=0^H {L_i \over D^i}) * (∑^H^~i=0~ D^i^)
$$


… where D = decayFactor, and H = history.

Setting history = 0 effectively disables the time decay function and only the current load for each metric will be considered in the load balance factor computation.

The mod\_cluster load balancer expects the load factor to be an integer between 0 and 100, where 0 indicates max load and 100 indicates zero load. Therefore, the final load factor sent to the load balancer


$$
L~Final~ = 100 - (L * 100)
$$


While you are free to write your own load metrics, the following LoadMetrics are available out of the box:

