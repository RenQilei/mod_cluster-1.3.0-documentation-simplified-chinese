# 第二章 快速入手指南

下面是在单 httpd 服务器和单后端（应用）服务器环境下最小化配置使 mod_cluster 可工作的步骤， 其中应用服务器可以是 JBoss AS, JBoss Web，Undertow 或 Tomcat。这些步骤可以按需重复运用于你包含多台 httpd 服务器或后端（应用）服务器的集群中。

这里演示步骤的目的并非直接展示如何搭建一个安装 mod_cluster 的生产环境；如此处并没未涉及如何使用 [SSL 来实现安全访问](chapter12.md) httpd 端的 mod_manager 组件。需要参见 [均衡器端](chapter3.md)和[工作者端](chapter6.md)的配置文档来获取全部的配置选项集。

## 2.1. 下载 mod_cluster 组件

下载最新的 httpd 和 [java 发行包](http://www.jboss.org/mod_cluster/ downloads/latest.html)。如果没有适合你的操作系统或系统架构的预集成好的 httpd 安装包，你可以[从源文件自行构建二进制文件](chapter4.md)。

## 2.2. 安装 httpd 二进制文件

### 2.2.1. 安装完整的 httpd

httpd 端的安装包压缩成了包含完整 httpd 安装文件。因为它们已经包含了 Apache httpd 安装文件，所以你不需要再下载了。只要把他们解压在 ```root``` 中即可，如：

```
cd /
unzip mod_cluster-1.3.1.Final-linux2-x64-ssl.zip
```

这样你就会在你的```/opt/jboss```目录下获取完整的 httpd 安装文件，且包含了 所有mod_cluster 模块。

### 2.2.2. 仅安装模块

如果你有一个已使用且正在运行的 httpd，那么你只需下载一个与你所用平台对应的名为 ```mod_cluster httpd dynamic libraries``` 的动态库，提取模块并把它们复制到你的 httpd 的模块目录中。

```
cd /tmp
tar xvf mod_cluster-1.x.y.Final-linux2-x86-so.tar.gz
```

然后你需要把下面的文件复制到你的模块目录中：

* mod_slotmem.so
* mod_manager.so
* mod_proxy_cluster.so
* mod_advertise.so

> **httpd 版本不匹配：** ```[warn] httpd version mismatch detected```
> 
> 请注意上述模块不能简单的加载到任意的 httpd 安装文件中。这些模块是基于特定的 httpd 版本构建的，因此它们不能被使用于老版本中。如， 基于 httpd 2.4.x 构建的 mod_cluster 模块并不能加载到 httpd 2.2.x 中。总是检查版本说明中提到的构建中使用的版本情况：[mod_cluster 发行](https://github.com/Karm/mod_cluster/releases)。

### 2.2.3. 在任意目录下安装

在 httpd 发行包中有一个 ```opt/jboss/httpd/sbin/installhome.sh``` 脚本，允许对安装包进行重新配置，以让它可以运行在用户的 home 目录下。实现方法只需解压安装包到你的 ```home``` 目录下并运行脚本。一旦完成，httpd 将在 8000 端口下运行，并允许 MCMP 在 localhost:6666 下进行通信，且在相同的主机和端口上提供```/mod_cluster_manager```。

### 2.2.4. 在 Windows 下安装

解压与你的架构对应的安装包。更换到你解压安装包的子文件夹 ````httpd-*``` 的 ```bin``` 目录下，运行 shell 脚本 ```installconf.bat```。

```
cd httpd-2.4\bin
installconf.bat
```

你可以直接运行 httpd，如下：

```
httpd.exe
```

或将 Apache httpd 安装为服务：

```
httpd.exe -k install
```

并通过 ```net start``` 或 使用```httpd.exe``` 开启该服务：

```
net start myApache
```

或

```
httpd.exe -k start
```

Note that in the windows bundles have a flat directory structure, so you have ```httpd-2.2/modules/``` instead of ```opt/jboss/httpd/lib/httpd/modules```.

注意 windows 的安装包为扁平的目录结构，所以你的目录会是 ```httpd-2.2/modules/``` 而不是 ```opt/jboss/httpd/lib/httpd/modules```。

## 2.3. 配置 httpd

```httpd.conf``` 通过 Quick Start 值进行预配置。你应该在你的配置中采用默认值。下面是示例的配置。如果你如上述解压了下载的安装包到 ```root``` 目录下，并使用了你解压的 httpd 安装文件， ```httpd.conf``` 的位置为```/opt/jboss/httpd/httpd/conf```。

```
LoadModule proxy_module /opt/jboss/httpd/lib/httpd/modules/mod_proxy.so
LoadModule proxy_ajp_module /opt/jboss/httpd/lib/httpd/modules/mod_proxy_ajp.so
LoadModule cluster_slotmem_module /opt/jboss/httpd/lib/httpd/modules/mod_cluster_slotmem.so
LoadModule manager_module /opt/jboss/httpd/lib/httpd/modules/mod_manager.so
LoadModule proxy_cluster_module /opt/jboss/httpd/lib/httpd/modules/mod_proxy_cluster.so
LoadModule advertise_module /opt/jboss/httpd/lib/httpd/modules/mod_advertise.so

<IfModule manager_module>
  Listen 127.0.0.1:6666
  ManagerBalancerName mycluster
  <VirtualHost 127.0.0.1:6666>
    <Location />
     Require ip 127.0.0
    </Location>

    KeepAliveTimeout 300
    MaxKeepAliveRequests 0
    #ServerAdvertise on http://@IP@:6666
    AdvertiseFrequency 5
    #AdvertiseSecurityKey secret
    #AdvertiseGroup @ADVIP@:23364
    EnableMCPMReceive

    <Location /mod_cluster_manager>
       SetHandler mod_cluster-manager
       Require ip 127.0.0
    </Location>

  </VirtualHost>
</IfModule>
```

如果你正在使用你自己安装的 httpd，你可以在你的安装的文件夹里的 conf 目录下找到 httpd.conf。添加到 httpd.conf 中的内容和上面的有一些不一样（几个 .so 文件的不同路径）：

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
LoadModule cluster_slotmem_module modules/mod_slotmem.so
LoadModule manager_module modules/mod_manager.so
LoadModule proxy_cluster_module modules/mod_proxy_cluster.so
LoadModule advertise_module modules/mod_advertise.so

... ...
```

> **mod_cluster 1.2.x vs mod_cluster 1.3.x:**
> 
> 注意从 mod_cluster 1.3.x 开始，slotmem 模块被定义为 ```LoadModule cluster_slotmem_module modules/mod_cluster_slotmem.so``` 而不再是过去的 ```LoadModule slotmem_module modules/mod_slotmem.so```。

## 2.4. 安装工作者端二进制文件

First, extract the java libraries distribution to a temporary directory. One can grab the mod_cluster-java-libs-*-bin.zip from mod_cluster release. The following text assumes it is extracted to /tmp/mod-cluster.

Your next step depends on whether your target server is JBoss AS 5.x, JBoss AS 7.x, JBoss Web/Tomcat 6, 7, 8 or Wildfly (Undertow).

首先，解压 java 二进制发行文件到一个临时文件夹中。可以从 [mod_cluster 发行](https://github.com/Karm/mod_cluster/releases)处获得 ```mod_cluster-java-libs-*-bin.zip```。下面假设你把它解压到了 ```/tmp/mod-cluster``` 目录。

你的下一步操作将取决于你的目标服务器是 JBoss AS, JBoss Web/Tomcat 还是 Wildfly (Undertow)。

### 2.4.1. 在 JBoss AS 6.x 中安装

在 AS 6.x 中，你不需要做任何操作来安装 java 端的二进制文件；它是 AS 发行版中默认、标准并配置完成的一部分。

### 2.4.2. 在 JBoss AS 5.x 中安装

假设 ```$JBOSS_HOME``` 是你的 JBoss AS 安装文件的根目录，你想在 AS 的所有配置中使用 mod_cluster：

```
cp -r /tmp/mod-cluster/mod-cluster.sar $JBOSS_HOME/server/all/deploy
```

### 2.4.3. 在 Tomcat 中安装
假设 ```$CATALINA_HOME``` 是你的 Tomcat 安装包的根目录：

```
cp -r /tmp/mod-cluster/JBossWeb-Tomcat/lib/* $CATALINA_HOME/lib/
```

Note that you should remove in the ```$CATALINA_HOME/lib/``` directory the mod_cluster-containertomcat6*
file in Tomcat7 and the mod_cluster-container-tomcat7* in Tomcat 6.

注意在 Tomcat7 中的话你应该移除 ```$CATALINA_HOME/lib/``` 目录里的 ```mod_cluster-containertomcat6*``` 文件，而在 Tomcat 6 中的话你应该移除相应的 ```mod_cluster-container-tomcat7*``` 文件。

### 2.4.4. ~~在 JBoss AS 4.2.x 或 4.3.x 中安装~~（1.3.x 弃用）

假设 ```$JBOSS_HOME``` 是你的 JBoss AS 安装包的根目录，你想在 AS 的所有配置中使用 mod_cluster：

```
cp -r /tmp/mod-cluster/JBossWeb-Tomcat/lib/* $JBOSS_HOME/server/all/deploy/jbossweb.
deployer/
```

### 2.4.5. 在 Wildfly 中安装

> 文档仍在完善中……

## 2.5. （应用）服务器端配置

### 2.5.1. 在 JBoss AS 5.x+ 上配置 mod_cluster

不需要任何后续安装配置！

### 2.5.2. 在 standalone JBoss Web 或 Tomcat 上配置 mod_cluster

编辑 ```$CATALINA_HOME/conf/server.xml``` 文件，添加下面的内容到其它 ```<Listener/>``` 元素的后面：

```
<Listener className="org.jboss.modcluster.container.catalina.standalone.ModClusterListener"
advertise="true"/>
```

### 2.5.3. ~~在 JBoss AS 4.2.x 和 4.3.x 上集成 mod_cluster~~（1.3.x 弃用）

Edit the ```$JBOSS_HOME/server/all/deploy/jboss-web.deployer/server.xml``` file, adding the following next to the other ```<Listener/>``` elements:

```
<Listener className="org.jboss.modcluster.container.catalina.standalone.ModClusterListener"
advertise="true"/>
```

## 2.6. 启用 httpd

根据如下操作来启用 httpd：

```
/opt/jboss/httpd/sbin/apachectl start
```

## 2.7. 启用后端（应用）服务

### 2.7.1. 启用 JBoss AS

```
cd $JBOSS_HOME/bin
./run.sh -c all
```

### 2.7.2. 启用 JBossWeb 或 Tomcat

```
cd $CATALINA_HOME
./startup.sh
```

## 2.8. 架设更多的后端（应用）服务器

为你的集群中的每台服务器重复后端（应用）服务器安装和配置步骤即可。

## 2.9. 使用负载均衡演示应用进行测试

尝试[负载均衡示例应用](chapter15.md)是一个很好的办法来了解 mod_cluster 的能力。