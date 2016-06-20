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

[https://github.com/modcluster/mod_cluster/tagsrelease/](https://github.com/modcluster/mod_cluster/tagsrelease/) [https://github.com/modcluster/
mod_cluster/tags]

* 构建 httpd modules [缺少锚点链接]
* 构建 server-side components [缺少锚点链接]

## 1.6. 配置

如果你想要跳过具体细节，只是启用一个小型有效的 mod_cluster 安装，见 快速上手指导 [缺少链接].

* 配置 httpd [缺少锚点链接]
* 配置 JBoss AS [缺少锚点链接]
* 配置 JBoss Web 或 Tomcat [缺少锚点链接]

## 1.7. 迁移

Migrating from mod_jk or mod_proxy is fairly straight forward. In general, much of the configuration previously found in httpd.conf is now defined in the application server nodes.

* Migrating from mod_jk
* Migrating from mod_proxy

## 1.8. SSL 支持

Both the request connections between httpd and the application server nodes, and the feedback channel between the nodes and httpd can be secured. The former is achieved via the
mod_proxy_https module and a corresponding ssl-enabled HTTP connector in JBoss Web. The
latter requires the mod_ssl module and explicit configuration in JBoss AS/Web.

## 1.9. 负载均衡示例应用

The mod_cluster binary distribution for JBoss AS/JBossWeb/Tomcat includes a demo application that helps demonstrate how different server-side scenarios affect the routing of client requests by the load balancer. The demo application is located in the mod_cluster distribution's demo directory.

> **Strong cryptography warning**
> 
> mod_cluster contains mod_ssl, therefore the following warning (copied from OpenSSL [http://www.openssl.org/]) applies:
> PLEASE REMEMBER THAT EXPORT/IMPORT AND/OR USE OF STRONG CRYPTOGRAPHY SOFTWARE, PROVIDING CRYPTOGRAPHY HOOKS OR EVEN JUST COMMUNICATING TECHNICAL DETAILS ABOUT CRYPTOGRAPHY SOFTWARE IS ILLEGAL IN SOME PARTS OF THE WORLD. SO, WHEN YOU IMPORT THIS PACKAGE TO YOUR COUNTRY, RE-DISTRIBUTE IT FROM THERE OR EVEN JUST EMAIL TECHNICAL SUGGESTIONS OR EVEN SOURCE PATCHES TO THE AUTHOR OR OTHER PEOPLE YOU ARE STRONGLY ADVISED TO PAY CLOSE ATTENTION TO ANY EXPORT/IMPORT AND/OR USE LAWS WHICH APPLY TO YOU. THE AUTHORS OF OPENSSL ARE NOT LIABLE FOR ANY VIOLATIONS YOU MAKE HERE. SO BE CAREFUL, IT IS YOUR RESPONSIBILITY.



