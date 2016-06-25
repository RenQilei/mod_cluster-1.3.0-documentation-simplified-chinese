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

elected 是 worker（译者注：节点应用服务器）被选中的时间。oldelected 是 lbstatus 被选中的最后一次被重新计算的时间。最低状态的节点会被选中。带有 lbfactor 为 #0 的节点会被计算逻辑忽略。

### 3.4.4. WaitForRemove

WaitForRemove: 在节点被移除前被 httpd 忽略的时间，按秒计算。

Default: 10 seconds

### 3.4.5. ProxyPassMatch/ProxyPass

ProxyPassMatch/ProxyPass: ProxyPassMatch 和 ProxyPass 是mod_cluster 指令， 当使用 ! (取代后台url)防止路径中的 reverse-proxy。这可能被用来允许 httpd 来处理（server）像图片这类的静态信息。

```
ProxyPassMatch ^(/.*\.gif)$ !
```

上面的例子将允许 httpd 直接处理（server） .gif 文件。

### 3.4.6. EnableOptions

使用 OPTIONS 方法来周期性的检查有效的连接。满足相同的角色作为 CPING/CPONG 被 AJP 使用，但为了 HTTP/HTTPS 连接。（Fulfils the same role as the CPING/CPONG used by AJP but for HTTP/HTTPS connections.）端点需要至少由 HTTP/1.1 来实现。

On (or no value): 使用 OPTIONS (默认)

Off: 不使用 OPTIONS

## 3.5. mod_manager

The Context of a mod_manger directive is VirtualHost except mentioned otherwise. ```server config``` means that it must be outside a VirtualHost configuration. If not an error message will be displayed and httpd won't start.

  mod_manager 指令的使用环境（Context）是虚拟主机（VirtualHost）除非你特意申明。```server config```表明它必须在 VirtualHost外配置。不然的话就会有错误信息显示，httpd也将不能启动。

### 3.5.1. EnableMCPMReceive

EnableMCPMReceive - Allow the VirtualHost to receive MCPM. Allow the VirtualHost to receive the MCPM from the nodes. You need one EnableMCPMReceive in your httpd configuration to allow mod_cluster to work, put it in the VirtualHost where you configure advertise.

### 3.5.2. MemManagerFile

MemManagerFile: That is the base name for the names mod_manager will use to store configuration, generate keys for shared memory or lock files. That must be an absolute path name; the directories will created if needed. It is highly recommended that those files are placed on a local drive and not an NFS share. (Context: server config)

Default: $server_root/logs/

### 3.5.3. Maxcontext

Maxcontext: That is the number max of contexts supported by mod_cluster. (Context: server config)

Default: 100

### 3.5.4. Maxnode

Maxnode: That is the number max nodes supported by mod_cluster. (Context: server config)

Default: 20

### 3.5.5. Maxhost
Maxhost: That is the number max host (Aliases) supported by mod_cluster. That is also the max number of balancers. (Context: server config)

Default: 20

### 3.5.6. Maxsessionid

Maxsessionid: That is the number of active sessionid we store to give number of active sessions in the mod_cluster-manager handler. A session is unactive when mod_cluster doesn't receive any information from the session in 5 minutes. (Context: server config)

Default: 0 (the logic is not activated).

### 3.5.7. MaxMCMPMaxMessSize

MaxMCMPMaxMessSize: Maximum size of MCMP messages. from other Max directives.

Default: calculated from other Max directives. Min: 1024

### 3.5.8. ManagerBalancerName

ManagerBalancerName: That is the name of balancer to use when the JBoss AS/JBossWeb/Tomcat doesn't provide a balancer name.

Default: mycluster

### 3.5.9. PersistSlots

PersistSlots: Tell mod_slotmem to persist the nodes, Alias and Context in files. (Context: server config)

Default: Off

### 3.5.10. CheckNonce

CheckNonce: Switch check of nonce when using mod_cluster-manager handler on | off Since 1.1.0.CR1

Default: on Nonce checked

### 3.5.11. AllowDisplay

AllowDisplay: Switch additional display on mod_cluster-manager main page on | off Since 1.1.0.GA

Default: off Only version displayed

### 3.5.12. AllowCmd

AllowCmd: Allow commands using mod_cluster-manager URL on | off Since 1.1.0.GA

Default: on Commmands allowed

### 3.5.13. ReduceDisplay

ReduceDisplay - Reduce the information the main mod_cluster-manager page to allow more nodes in the page. on | off

Default: off Full information displayed

### 3.5.14. SetHandler mod_cluster-manager

SetHandler mod_cluster-manager: That is the handler to display the node mod_cluster sees from the cluster. It displays the information about the nodes like INFO and additionaly counts the number of active sessions.

```
<Location /mod_cluster_manager>
SetHandler mod_cluster-manager
Order deny,allow
Deny from all
Allow from 127.0.0.1
</Location>
```

When accessing the location you define in httpd.conf you get something like:




