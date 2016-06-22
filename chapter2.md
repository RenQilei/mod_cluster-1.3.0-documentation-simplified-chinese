# 第二章 快速入手指南

下面是为 mod_cluster 在单一的 httpd 服务器和单一的后端（back end）服务器上配置最小化能工作的安装的步骤， 可以是 JBoss AS, JBoss Web 或 Tomcat。这些步骤可以按需重复运用于你的集群中的多台 httpd 服务器和后端服务器中。

这里演示的步骤不是旨在展示如何搭建一个安装 mod_cluster 的生产环境；如并没有包含如何使用 SSL 来实现安全访问 [缺少锚点链接] httpd 端的 mod_manager 组件。需要参见 httpd 端 [缺少锚点链接]和 java 端 [缺少锚点链接]的配置文档来获取全部的配置选项组。

## 2.1. 下载 mod_cluster 组件

下载最新的 httpd 和 [java 发行包](http://www.jboss.org/mod_cluster/ downloads/latest.html) [http://www.jboss.org/mod_cluster/
downloads/latest.html]。如果没有适合你的操作系统或系统架构的预集成好的 httpd 安装包，你可以从源文件 [缺少锚点链接]自行构建二进制文件。

## 2.2. 安装 httpd 二进制文件

### 2.2.1. 安装完整的 httpd

httpd 端的安装套件压缩成了包含完整 httpd 安装功能的 tar 文件。因为它们已经包含了 Apache httpd 安装文件，所以你不需要再下载 Apache httpd 了。只要在 ```root``` 中提取它们，如：

```
cd /
tar xvf mod-cluster-1.0.0-linux2-x86-ssl.tar.gz
```

这样你就会在你的```/opt/jboss```目录下获取完整的 httpd 安装文件。

### 2.2.2. 仅安装模块

如果你已经有了你跟愿意继续使用的运行着的 httpd，你将需要下载一个与你的平台对应的名为 mod_cluster 的 httpd 动态库，提取模块并把它们复制到你的 httpd 的模块目录中。

```
cd /tmp
tar xvf mod_cluster-1.x.y.Final-linux2-x86-so.tar.gz
```

然后你需要把下面的文件复制到你的模块路径中：

* mod_slotmem.so
* mod_manager.so
* mod_proxy_cluster.so
* mod_advertise.so

### 2.2.3. 在 home 目录下安装

从 1.1.0.CR2 开始脚本```opt/jboss/httpd/sbin/installhome.sh```允许对安装套件进行重新配置，以让它可以运行在用户的 home 目录下。要想这要的话你只需要提取安装包到你的 home 目录下并运行脚本。一旦完成，httpd 将在8000端口下运行，并允许 MCMP 通信在 localhost:6666 下进行，且在相同的主机和端口上提供```/mod_cluster_manager```。

### 2.2.4. 在 Windows 下安装

Unzip the bundle corresponding to your architecture.

Change to the bin directory of the subfolder httpd-2.2 where you unzip the bundle and run the installconf.bat shell script.

```
cd httpd-2.2\bin
installconf.bat
```

You may run httpd directly by using:

```
httpd.exe
```

or install Apache httpd as a service:

```
httpd.exe -k install
```

and start the service via net start or using httpd.exe:

```
net start Apache2.2
```

```
httpd.exe -k start
```

Note that in the windows bundles have a flat directory structure, so you have ```httpd-2.2/modules/``` instead of ```opt/jboss/httpd/lib/httpd/modules```.

## 2.3. 配置 httpd

从1.1.0.CR2版本开始，httpd.conf 通过 Quick Start 值进行预配置。你应该在你的更旧的 mod_cluster 中配置中采纳默认值，我们将在 http.d 中添加如下配置。如果你提取了下载的安装包到上述的 root 目录下，并使用了你提取出中的 httpd 安装文件， httpd.conf 的位置为```/opt/jboss/httpd/httpd/conf```。

```
LoadModule proxy_module /opt/jboss/httpd/lib/httpd/modules/mod_proxy.so
LoadModule proxy_ajp_module /opt/jboss/httpd/lib/httpd/modules/mod_proxy_ajp.so
LoadModule slotmem_module /opt/jboss/httpd/lib/httpd/modules/mod_slotmem.so
LoadModule manager_module /opt/jboss/httpd/lib/httpd/modules/mod_manager.so
LoadModule proxy_cluster_module /opt/jboss/httpd/lib/httpd/modules/mod_proxy_cluster.so
LoadModule advertise_module /opt/jboss/httpd/lib/httpd/modules/mod_advertise.so

Listen 10.33.144.3:6666
<VirtualHost 10.33.144.3:6666>

  <Directory />
    Order deny,allow
    Deny from all
    Allow from 10.33.144.
  </Directory>

  KeepAliveTimeout 60
  MaxKeepAliveRequests 0
  ManagerBalancerName mycluster
  AdvertiseFrequency 5
</VirtualHost>
```

如果你正在使用你自己安装的 httpd，你可以在你的安装的文件夹里的 conf 目录下找到 httpd.conf。添加到 httpd.conf 中的内容和上面的有一些不一样（几个 .so 文件的不同路径）：

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
LoadModule slotmem_module modules/mod_slotmem.so
LoadModule manager_module modules/mod_manager.so
LoadModule proxy_cluster_module modules/mod_proxy_cluster.so
LoadModule advertise_module modules/mod_advertise.so

Listen 10.33.144.3:6666
<VirtualHost 10.33.144.3:6666>
  <Directory />
    Order deny,allow
    Deny from all
    Allow from 10.33.144.
  </Directory>
  
  KeepAliveTimeout 60
  MaxKeepAliveRequests 0
  
  ManagerBalancerName mycluster
  AdvertiseFrequency 5
</VirtualHost>
```

## 2.4. 安装服务器端二进制文件

首先，提取 java 端二进制文件到一个临时文件夹中。下面假设你把它提取到了```/tmp/mod-cluster```路径。

你的下一步取决于你的目标服务器是 JBoss AS 还是 JBoss Web/Tomcat。

### 2.4.1. 在 JBoss AS 6.x 中安装

在 AS 6.x 中，你不需要做任何操作来安装 java 端的二进制文件；它是 AS 发行版中默认、标准并配置完成的一部分。

### 2.4.2. 在 JBoss AS 5.x 中安装

Assuming $JBOSS_HOME indicates the root of your JBoss AS install and that you want to use mod_cluster in the AS's all config:

```
cp -r /tmp/mod-cluster/mod-cluster.sar $JBOSS_HOME/server/all/deploy
```

### 2.4.3. 在 Tomcat 中安装
Assuming $CATALINA_HOME indicates the root of your Tomcat install:

```
cp -r /tmp/mod-cluster/JBossWeb-Tomcat/lib/* $CATALINA_HOME/lib/
```

Note that you should remove in the ```$CATALINA_HOME/lib/``` directory the mod_cluster-containertomcat6*
file in Tomcat7 and the mod_cluster-container-tomcat7* in Tomcat 6.

### 2.4.4. 在 JBoss AS 4.2.x 或 4.3.x 中安装

Assuming $JBOSS_HOME indicates the root of your JBoss AS install and that you want to use mod_cluster in the AS's all config:

```
cp -r /tmp/mod-cluster/JBossWeb-Tomcat/lib/* $JBOSS_HOME/server/all/deploy/jbossweb.
deployer/
```

## 2.5. 服务器端配置

### 2.5.1. 在 JBoss AS 5.x+ 上配置 mod_cluster

不需要任何后续安装配置！

### 2.5.2. 在 单独（standalone）的 JBoss Web or Tomcat 上配置 mod_cluster

Edit the $CATALINA_HOME/conf/server.xml file, adding the following next to the other ```<Listener/>``` elements:

```
<Listener className="org.jboss.modcluster.container.catalina.standalone.ModClusterListener"
advertise="true"/>
```

### 2.5.3. 在 JBoss AS 4.2.x and 4.3.x 集成 mod_cluster

Edit the ```$JBOSS_HOME/server/all/deploy/jboss-web.deployer/server.xml``` file, adding the following next to the other ```<Listener/>``` elements:

```
<Listener className="org.jboss.modcluster.container.catalina.standalone.ModClusterListener"
advertise="true"/>
```

## 2.6. 启动 httpd

根据如下操作来开启 httpd：

```
/opt/jboss/httpd/sbin/apachectl start
```

## 2.7. 启动后端服务

### 2.7.1. 启动 JBoss AS

```
cd $JBOSS_HOME/bin
./run.sh -c all
```

### 2.7.2. 启动 JBossWeb 或 Tomcat

```
cd $CATALINA_HOME
./startup.sh
```

## 2.8. 架设更多的后端服务器

为你的集群中的每台服务器重复后端服务器安装和配置步骤。

## 2.9. 试验负载均衡示例应用

尝试负载均衡示例应用 [缺少锚点链接]是一个很好的办法来了解 mod_cluster 的能力.
