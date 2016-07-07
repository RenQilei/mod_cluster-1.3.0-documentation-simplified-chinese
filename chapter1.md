# 第一章 概览

mod_cluster 是一个基于 HTTP 的负载均衡器。像 mod_jk 和 mod_proxy，mod_cluster使用一个通信通道来转发来自 httpd 的请求到一组应用服务器的一个节点上。不像 mod_jk 和 mod_proxy 的是，mod_cluster 在应用服务器节点和 httpd 间创建了一个额外的连接。应用服务器节点使用这个连接来传输服务器端的负载均衡因素和生命周期事件到 httpd，通过一组自定义的HTTP方法，亲切地称之为Mod-Cluster Management Protocol (MCMP)。这个额外的反馈通道允许 mod_cluster 实现一定的别的负载均衡解决方案中无法找寻的智能和粒度。

在 httpd 中，mod_cluster通过基于一组带有支持 mod_proxy 功能的 httpd 模块来实现。大量的逻辑来自于 mod_proxy，如：mod_proxy_ajp 提供了 mod_cluster 所需的全部 AJP 逻辑。

## 1.1. 平台

JBoss 准备的[二级制包](http://www.jboss.org/mod_cluster/downloads.html) [http://www.jboss.org/mod_cluster/downloads.html]已经包含了 httpd 和 mod_cluster，所以你可以在以下平台上很快的尝试 mod_cluster：

* Linux x86, x64, ia64
* Solaris x86, SPARC
* Windows x86, x64, ia64
* HP-UX PA-RISC, ia64

## 1.2. 优势

mod_cluster 相较于其它的基于httpd的负载均衡器拥有以下优势：

**httpd 工作服务器的动态配置**

传统的基于 httpd 的负载均衡器需要工作服务器针对代理有明确有效的配置。在 mod_cluster，一堆的代理配置则摆放在了应用服务器上。一组决定哪个应用服务器将会通信的代理要么通过静态的列表，要么通过使用基于广告机制的动态发现。应用服务器依赖于代理的生命周期时间（如服务器开启/关闭）允许它们自己高效的自我配置。值得注意的是，服务器优雅的关闭将不会导致像传统的基于 httpd 的负载局衡器那样，通过代理来响应故障迁移。

**服务器端负载均衡因素计算**

相较于传统的基于 httpd 的负载均衡器，mod_cluster 使用了通过应用服务器提供的负载均衡因素计算，而不是在代理中计算这些。

结果是，mod_cluster 提供了比代理更为健全的和精确的一组负载均衡度量衡。详情可参见 Server-Side Load Metrics 章节。

**粒度良好的 web 应用生命周期控制**

传统的基于 httpd 的负载均衡器不能很好的处理 未部署的web 应用。从代理的角度来看，从为了一个不存在的资源发出请求，来请求一个未部署的 web 应用是不易察觉的，并会导致404错误。在 mod_cluster 中，每个服务器向代理转发任何 web 应用上下文根的生命周期时间（如 web-app 部署/未部署）来通知它开启/关闭对一个给定上下文根的路由请求。

**AJP是一个可选项**

不像 mod_jk，mod_cluster 不需要引入 AJP。httpd 连接到应用服务器节点可以使用 HTTP，HTTPS 或者 AJP。

基本的定义在 [wiki](http://www.jboss.org/community/docs/DOC-11431) [http://www.jboss.org/community/docs/DOC-11431]中有所描述。

## 1.3. 环境要求

* httpd-2.2.8+
* JBoss AS 5.0.0+ 或者 JBossWeb 2.1.1+


> **注意**
> 
> httpd-2.2.8+ 集成，所以如果你使用了集成版，你就不需要下载 Apache httpd。

## 1.4. 限制

mod_cluster 使用共享的内存来保证节点描述的同步，共享的内存创建于 httpd 开启时，并且每个物件的结构都是固定的。下列所属便不可以通过配置指令更改。

* 别名（Alias）长度上限为 40 个字符 (Host: hostname header, Alias in ```<Host/>```).
* 上下文根（context）长度上限为 40 个字符 (for example myapp.war deploys in /myapp /myapp is the context).
* 均衡器 （balancer）名字长度上限为 40 个字符 (balancer property in mbean).
* JVMRoute 字符串长度上限为 80 个字符 (JVMRoute in ```<Engine/>```).
* 负载均衡组（load balancing group）名字长度上限为 20 个字符 (domain property in mbean).
* 一个节点（node）的 hostname 的长度上限为 64 个字符 (address in the ```<Connector/>```).
* 一个节点的端口（port） 的长度上限为 7 个字符 (8009 is 4 characters, port in the ```<Connector/>```).
* 一个节点的 scheme 的长度上限为 6 个字符 (possible values are http, https, ajp, liked with the protocol of ```<Connector/>```).
* cookie 的名字的长度上限为 30 个字符 (the header cookie name for sessionid default value: ```JSESSIONID``` from ```org.apache.catalina.Globals.SESSION_COOKIE_NAME```).
* 路径（path）名字的长度上限为 30 个字符 (the parameter name for the sessionid default value: ```jsessionid``` from ```org.apache.catalina.Globals.SESSION_PARAMETER_NAME```).
* sessionid 的长度上限为 120 个字符 (something like ```BE81FAA969BF64C8EC2B6600457EAAAA.node01```).

## 1.5. 下载

可以在 [这里](http://www.jboss.org/mod_cluster/downloads/ latest) [http://www.jboss.org/mod_cluster/downloads/latest] 下载最新的 mod_cluster 发行版。

这个版本由系列组件组成：

* 通用平台的 httpd 二进制文件
* JBoss AS/JBossWeb/Tomcat 二进制发布包

与此同时，你可以通过使用 Git 仓库的源文件来构建：

[https://github.com/modcluster/mod_cluster/tagsrelease/](https://github.com/modcluster/mod_cluster/tagsrelease/) [https://github.com/modcluster/mod_cluster/tags]

* 构建 httpd modules [缺少锚点链接]
* 构建 server-side components [缺少锚点链接]

## 1.6. 配置

如果你想要跳过具体细节，只是启用一个小型有效的 mod_cluster 安装，见 快速上手指南 [缺少链接].

* 配置 httpd [缺少锚点链接]
* 配置 JBoss AS [缺少锚点链接]
* 配置 JBoss Web 或 Tomcat [缺少锚点链接]

## 1.7. 迁移

从 mod_jk 和 mod_proxy 迁移基本上是没有什么问题的。总的来说，绝大多数之前在 httpd.conf 中才能找到的配置项现在都在应用服务器节点上定义了。

* 从 mod_jk [缺少锚点链接] 迁移
* 从 mod_proxy [缺少锚点链接] 迁移

## 1.8. SSL 支持

所有 httpd 和应用服务器节点间的请求连接和节点与 httpd 间的反馈通道都可以是安全的。前者通过 mod_proxy_https 模块和 JBoss Web 相应的支持 ssl 的 HTTP 连接器实现。后者则需要在 JBoss AS/Web 中引入 mod_ssl 模块和明确的配置。

## 1.9. 负载均衡示例应用

提供给 JBoss AS/JBoss Web/Tomcat 的 mod_cluster 二进制发行包包含了一个示例应用来帮助展现在负载均衡下不同的服务器端场景是如何影响客户端的路由请求的。示例应用存放在 mod_cluster 发行包的 demo 文件夹下。

> **高强度加密警告**
> 
> mod_cluster 包含了 mod_ssl，因此请注意下列警告（转载自 [OpenSSL](http://www.openssl.org/) [http://www.openssl.org/]）：
> 
> 请牢记强加密软件的 导出/导入 和/或 使用，在一些地方，提供加密钩子或仅是讨论关于加密软件的技术细节是非法的。所以，当你在导入这个包，再分发它或者只是通过邮件提出技术建议或者仅是对作者或其他人提供源补丁，强烈建议你密切关注你所在的国家适用于你的关于 导出/导入 和/或 使用的法律。OpenSSL的作者不对你在这里做出的任何违法行为负责。所以请小心，这是你的责任。
