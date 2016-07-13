# 第一章 概览

  mod_cluster 是一个基于 HTTP 的负载均衡器。与 mod_jk 和 mod_proxy 一样，mod_cluster也是使用通信信道将来自 httpd 的请求转发到一组应用服务器的某个节点上。而与 mod_jk 和 mod_proxy 不同的是，mod_cluster 在应用服务器节点和 httpd 间创建（leverage）了一个额外的连接。应用服务器节点基于该连接、通过一组自定义的 HTTP 方法，向 httpd 传输（应用）服务器端的负载均衡因子（factor）和生命周期事件等信息，该连接被亲切地称之为Mod-Cluster Management Protocol (mod_cluster 管理协议，MCMP)。这个额外的回传通道（feedback channel）使 mod_cluster 一定程度上拥有了其它负载均衡解决方案所无法提供的智能和粒度。

对于已经启用了 mod_proxy 的 httpd 来说，mod_cluster 只是一组 httpd 的模块（module）。它大量的逻辑源其实自于 mod_proxy，如：mod_proxy_ajp 提供了 mod_cluster 所需的全部 AJP 逻辑。

## 1.1. 平台

JBoss 提供的[二级制包](http://www.jboss.org/mod_cluster/downloads.html)已经包含了 httpd 和 mod_cluster，所以你可以在下列的平台上很快地尝试 mod_cluster：

* Linux x86, x64, ia64
* Solaris x86, SPARC
* Windows x86, x64, ia64
* HP-UX PA-RISC, ia64

## 1.2. 优势

mod_cluster 相较于其它基于 httpd 的负载均衡器拥有以下优势：

* **httpd 工作者（httpd workers，本文即所有应用服务器节点）的动态配置。**传统的基于 httpd 的负载均衡器需要明确地配置工作者到有效的代理服务器（proxy）。而在 mod_cluster 中，一堆代理的配置则摆放在了应用服务器上。与哪个应用服务器通信的一组代理要么通过静态列表来确定、要么通过使用广告机制（advertise mechanism）的动态发现来确定。应用服务器依赖于到代理服务器的生命周期事件（如服务器开启/关停）来允许它们高效地自动配置它们自己。值得注意的是，服务器和谐地关停将不会导致像传统的基于 httpd 的负载局衡器那样通过代理服务器配置而导致的响应失败。

* **（应用）服务器端负载均衡因子计算。**相较于传统的基于 httpd 的负载均衡器，mod_cluster 使用的负载均衡因子是通过应用服务器计算并提供的，而不是代理自己计算了这些。因此，mod_cluster 提供了比在代理服务器上配置来的更为健全的和精确的一组负载度量衡（load metric）。（更多可参见[服务器端负载度量衡](chapter10.md)）

* **粒度良好的 web 应用生命周期控制。**特别地，传统的基于 httpd 的负载均衡器并不能很好地处理取消部署 web 应用。从代理服务器的角度来看，向一个取消部署的 web 服务器发请求很难与向一个不存在的资源发请求相区分开，这还会导致 404 错误。在 mod_cluster 中，每个（应用）服务器都会向代理服务器转发 web 应用上下文根（context）的生命周期事件（如 web 应用的部署/取消部署）来通知它开启/停止到该（应用）服务器的给定上下文根的路由请求。

* **AJP是一个可选项。**与 mod_jk 不同的是，mod_cluster 不必须使用 AJP。httpd 连接到应用服务器节点可以使用 HTTP，HTTPS 或者 AJP。

基本的定义在 [wiki](http://www.jboss.org/community/docs/DOC-11431) 中都有所描述。

## 1.3. 环境要求

### 1.3.1. 均衡器端

* Apache HTTP Server 2.2.15+ (legacy 2.2.8+)

### 1.3.2. 工作者端

mod_cluster 已经为下面提到的所有容器提供了 java 模块：

* Tomcat 6
* Tomcat 7
* Tomcat 8
* JBoss AS7+
* Wildfly

## 1.4. 限制

mod_cluster 使用共享的内存来保证节点描述的同步，共享的内存创建于 httpd 启动时，并且每个物件的结构都是固定的。下列内容便不可以通过配置指令更改。

* **别名（Alias）**长度上限为 40 个字符 (主机（Host）: 主机名头（hostname header），```<Host/>``` 中的别名).
* **上下文根（context）**长度上限为 40 个字符 (例如：在 ```/myapp``` 里部署 ```myapp.war```，则 ```/myapp``` 即为上下文根).
* **均衡器 （balancer）**名字长度上限为 40 个字符 (```mbean``` 中的均衡器属性（balancer property）).
* **JVMRoute 字符串**长度上限为 80 个字符 (```<Engine/>``` 中的 ```JVMRoute```).
* **负载均衡组（load balancing group）**名字长度上限为 20 个字符 (```mbean``` 中的域属性（domain property）).
* **节点主机名（hostname）**长度上限为 64 个字符 (```<Connector/>``` 中的地址).
* **节点端口（port）**长度上限为 7 个字符 (8009 是 4 个字符，```<Connector/>```中的端口).
* **节点模式（scheme）**长度上限为 6 个字符 (可选值为 ```http```、```https```、```ajp```，这与 ```<Connector/>``` 的协议相关).
* **cookie 名字**长度上限为 30 个字符 (头部（header）```sessionid``` 的cookie 名字的默认值： 来自```org.apache.catalina.Globals.SESSION_COOKIE_NAME```的 ```JSESSIONID```).
* **路径（path）名字**长度上限为 30 个字符 (```sessionid```的参数名称（parameter name）的默认值：来自 ```org.apache.catalina.Globals.SESSION_PARAMETER_NAME``` 的 ```jsessionid```).
* **sessionid** 长度上限为 120 个字符 (类似于 ```BE81FAA969BF64C8EC2B6600457EAAAA.node01```).

## 1.5. 下载

下载最新的 mod_cluster 发行版，[官网](http://www.jboss.org/mod_cluster/downloads/latest)，[github](https://github.com/Karm/mod_cluster/releases)。

这个版本由下列组件组成：

* 通用平台的 httpd 二进制文件
* JBoss AS/JBossWeb/Tomcat 二进制发布包

与此同时，你可以通过使用 Git 仓库的源文件来构建。

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