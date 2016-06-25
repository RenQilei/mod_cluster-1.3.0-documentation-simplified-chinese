# 第三章 httpd 配置

## 3.1. Apache httpd 配置

你需要像下例所示加载 mod_cluster 需要的模块：

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
LoadModule slotmem_module modules/mod_slotmem.so
LoadModule manager_module modules/mod_manager.so
LoadModule proxy_cluster_module modules/mod_proxy_cluster.so
LoadModule advertise_module modules/mod_advertise.so
```

mod_proxy 和 mod_proxy_ajp 都是标准的 httpd 模块。 mod_slotmem 则是 a shared slotmem memory provider。mod_manager 是一个从 JBoss AS/JBossWeb/Tomcat 读取信息并更新共享内存信息的模块。mod_proxy_cluster 是一个为 mod_cluster 包含均衡器的模块。 mod_advertise 是一个额外的允许 httpd 通过多路广播包广告正在监听的 mod_cluster 的 IP 和端口的模块。这个多模块架构允许用户根据需要方便地更换模块。比如他们正在使用 http 而不是 ajp，只要

```
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
```

更换为：

```
LoadModule proxy_http_module modules/mod_proxy_http.so
```

## 3.2. mod_proxy 配置

像 ProxyIOBufferSiz 这样的 mod_cluster 指令可以被使用于配置 mod_cluster 中。不需要使用 ProxyPass 指令因为 mod_cluster 对 URL 应该被转发到哪台 JBoss Web 做了自动的配置。

## 3.3. mod_slotmem 配置

实际的版本不需要任何配置指令。

## 3.4. mod_proxy_cluster

### 3.4.1. CreateBalancers

CreateBalancers: 定义在 httpd 的 VirtualHosts 中均衡器是如何被创建的，这允许通过像这样的指令来表达：

```
ProxyPass / balancer://mycluster1/
```

0: 在所有 httpd 中定义的 VirtualHosts 中创建。

1: 不创建均衡器（需要最少一个 ProxyPass/ProxyPassMatch 来定义均衡器的名字）。

2: 只在主服务器中创建。

默认: 2

*注意:* 当使用1时不要忘记在 ProxyPass 指令中配置均衡器，因为默认是空stickysession 和 nofailover=Off 且通过 MCMP 配置信息收到的值是被忽略的。

### 3.4.2. UseAlias

UseAlias: 检查和 ServerName（参见 [主机名别名](http://labs.jboss.com/file-access/default/members/jbossweb/freezone/docs/latest/config/host.html) [http://labs.jboss.com/file-access/default/members/jbossweb/freezone/docs/latest/config/host.html]） 相应的别名。

Off: 不检查 (忽略别名)

On: 检查别名

Default: 关闭，忽略从节点来的别名信息。

*版本比 1.3.1.Final 更老的只支持相应的 0 和 1 两个值。*

### 3.4.3. LBstatusRecalTime

LBstatusRecalTime: 负载均衡逻辑来重新计算节点的状态的时间间隔，按秒（seconds）计算。

Default: 5 秒

实际重新计算节点状态的公式是：

```
status = lbstatus + (elected - oldelected) * 1000)/lbfactor;
```

lbfactor 是通过 STATUS 信息为节点接收到的。lbstatus 是重新计算每个 LBstatusRecalTime 秒数使用的公式：

```
lbstatus = (elected - oldelected) * 1000)/lbfactor;
```

elected is the number of time the worker was elected.oldelected is elected last time the lbstatus was recalculated.The node with the lowest status is selected. Nodes with lbfactor # 0 are skipped by the both calculation logic.

### 3.4.4. WaitForRemove

WaitForRemove: Time in seconds before a removed node is forgotten by httpd

Default: 10 seconds

### 3.4.5. ProxyPassMatch/ProxyPass

ProxyPassMatch/ProxyPass: ProxyPassMatch and ProxyPass are mod_proxy directives that when using ! (instead the back-end url) prevent to reverse-proxy in the path. This could be used allow httpd to serve static information like images.

```
ProxyPassMatch ^(/.*\.gif)$ !
```

The above for example will allow httpd to server directly the .gif files.

### 3.4.6. EnableOptions

Use OPTIONS method to periodically check the active connection. Fulfils the same role as the CPING/CPONG used by AJP but for HTTP/HTTPS connections. The endpoint needs to implement at least HTTP/1.1.

On (or no value): Use OPTIONS (default)

Off: Don't use OPTIONS


