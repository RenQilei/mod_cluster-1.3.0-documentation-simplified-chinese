# 第十五章 均衡演示应用

## 15.1. 概述

mod_cluster 发行版包括了一个演示应用来帮助展示服务器端场景使用负载均衡器对客户请求路由的影响有什么不同。这个演示应用位于 mod_cluster 发行版的 ```demo``` 目录中。

演示应用包含了两个组件：

1. 第一个组件是一个需要被部署进 JBossWeb/Tomcat/JBoss AS 的 war 文件。这个 war 文件包含了若干 servlet。
2. 第二个组件是一个允许用户登录线程池以重复向负载均衡器发送请求的 GUI 应用。请求最终都会被路由到演示 war 文件的主 servlet。应用会跟踪哪些服务器正在处理请求并用图表来展示这些信息。这个应用还可以发送不同的请求到演示 war 文件的负载生成（load generation）servlet，以使用户看到不同的负载情况如何影响请求的均衡。

注意无论如何演示应用并没有真正的依赖于 mod_cluster。它的依赖性仅在 JBossWeb/Tomcat（The demo's "Datasource Use" load generation scenario requires the use of JBoss Application Server.）上。因此，演示可以被用来展示在各负载均衡器做出路由决策上不同服务器端场景的效果。

还要注意这个演示应用不是用来作为负载测试工具使用的；例如：用来展示集群配置能够处理的最大负载。像这样使用它是一个很好的机会来向你展示客户端（client）可以生成的最大负载，而不是你的集群可以处理的最大负载。

## 15.2. Basic Usage

To run the demo application:

1. Unpack the mod_cluster distribution on your filesystem. Here we assume it has been unzipped to ~/mod_cluster or C:\mod_cluster.
2. Install mod_cluster into your httpd server as described at Installation of the httpd part
3. Install mod_cluster into your JBossAS/JBossWeb/Tomcat servers as described at Installation on the Java side
4. In AS7 you have to set org.apache.tomcat.util.ENABLE_MODELER to true, Something like:
```
<system-properties>
  <property name="org.apache.tomcat.util.ENABLE_MODELER" value="true"/>
</system-properties>
```
5. Start httpd and your JBossAS/JBossWeb/Tomcat servers
6. Deploy the load-demo.war found in the distribution's demo/server folder to your JBossAS/JBossWeb/Tomcat servers.
7. Start the demo application:
  * On *nix:
  ```
  cd ~/mod_cluster/demo/client
  ./run-demo.sh
  ```
  * On Windows:
  ```
  C:\>cd mod_cluster\demo\client
  C:\mod_cluster\demo\client>run-demo
  ```
