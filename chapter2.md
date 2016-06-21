# 第二章 快速入手指南

下面是为 mod_cluster 在单一的 httpd 服务器和单一的后端（back end）服务器上配置最小化能工作的安装的步骤， 可以是 JBoss AS, JBoss Web 或 Tomcat。这些步骤可以按需重复运用于你的集群中的多台 httpd 服务器和后端服务器中。

这里演示的步骤不是旨在展示如何搭建一个安装 mod_cluster 的生产环境；如并没有包含如何使用 SSL 来实现安全访问 [缺少锚点链接] httpd 端的 mod_manager 组件。需要参见 httpd 端 [缺少锚点链接]和 java 端 [缺少锚点链接]的配置文档来获取全部的配置选项组。

## 2.1. 下载 mod_cluster 组件

下载最新的 httpd 和 [java 发行包](http://www.jboss.org/mod_cluster/ downloads/latest.html) [http://www.jboss.org/mod_cluster/
downloads/latest.html]。如果没有适合你的操作系统或系统架构的预集成好的 httpd 安装包，你可以从源文件 [缺少锚点链接]自行构建二进制文件。

## 2.2. Install the httpd binary

### 2.2.1. Install the whole httpd

The httpd-side bundles are gzipped tars and include a full httpd install. As they contain already an Apache httpd install you don't need to download Apache httpd. Just extract them in root, e.g.

```
cd /
tar xvf mod-cluster-1.0.0-linux2-x86-ssl.tar.gz
```

That will give you a full httpd install in your ```/opt/jboss``` directory.

### 2.2.2. Install only the modules

If you already have a working httpd install that you would prefer to use, you'll need to download the bundle named mod_cluster httpd dynamic libraries corresponding to you plaform, extract the modules and copy them directory to your httpd install's module directory.

```
cd /tmp
tar xvf mod_cluster-1.x.y.Final-linux2-x86-so.tar.gz
```

And then you have to copy the files below to you module directory:

* mod_slotmem.so
* mod_manager.so
* mod_proxy_cluster.so
* mod_advertise.so

### 2.2.3. Install in home directory

Since 1.1.0.CR2 a script opt/jboss/httpd/sbin/installhome.sh allows reconfiguration of the bundle installation so that it can run in user's home directory. To do that just extract the bundle in your home directory and run the script. Once that done, httpd will run on port 8000 and will accept MCMP messages on localhost:6666 and offer /mod_cluster_manager on the same host and port.

### 2.2.4. Install in Windows

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

## 2.3. Configure httpd

Since 1.1.0.CR2 httpd.conf is preconfigured with the Quick Start values. You should adapt the default values to your configuration with older mod_cluster we will have to add the following to httpd.conf. If you extracted the download bundle to root as shown above and are using that extract as your httpd install, httpd.conf is located in ```/opt/jboss/httpd/httpd/conf```.

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

If you are using your own install of httpd, httpd.conf is found in your install's conf directory. The content to add to httpd.conf is slightly different from the above (different path to the various .so files):

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


