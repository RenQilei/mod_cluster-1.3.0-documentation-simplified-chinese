# 第五章 安装 httpd 模块

有效安装包可以在此获取 [http://www.jboss.org/mod_cluster/downloads.html](http://www.jboss.org/mod_cluster/downloads.html).

以防你不能在下载区域找到 mod_cluster 准备好的包，mod_cluster 可以通过源文件构建起来。你需要一个 httpd（不低于 2.2.8）发行版或（更好）一个 httpd 的源文件原始码（source tarball）和 mod_cluser 的源文件。构建介绍了如何从它的源文件构建 mod_cluster。

## 5.1. 配置

A minimal configuration is needed in httpd (See httpd.conf). A listener must be a added in JBossWEB conf/server.xml (See Configuring JBoss AS/Web).
需要在 httpd（参见 [httpd.conf](chapter3.md)）中做最小化的配置。在 JBossWEB conf/server.xml (参见 Configuring JBoss AS/Web)中必须添加监听器。

## 5.2. Installing and using the bundles

The bundles are tar.gz on POSIX platforms just extract them in root something like:

```
cd /
tar xvf mod-cluster-1.x.y-linux2-x86-ssl.tar.gz
```

The httpd.conf is located in /opt/jboss/httpd/httpd/conf to quick test just add something like:

```
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

To start httpd do the following:

```
/opt/jboss/httpd/sbin/apachectl start
```

NOTE: Make sure to use SSL before going in production.