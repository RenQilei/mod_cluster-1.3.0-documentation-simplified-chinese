# 第五章 安装 httpd 模块

有效安装包可以在此获取 [http://www.jboss.org/mod_cluster/downloads.html](http://www.jboss.org/mod_cluster/downloads.html).

以防你不能在下载区域找到 mod_cluster 准备好的包，mod_cluster 可以通过源文件构建起来。你需要一个 httpd（不低于 2.2.8）发行版或（更好）一个 httpd 的源文件原始码（source tarball）和 mod_cluser 的源文件。构建介绍了如何从它的源文件构建 mod_cluster。

## 5.1. 配置

需要在 httpd（参见 [httpd.conf](chapter3.md)）中做最小化的配置。在 JBossWEB conf/server.xml (参见 [配置 JBoss AS/Web](chapter6.md))中必须添加监听器。

## 5.2. 安装和使用安装套件

安装套件是在 POSIX 平台上的 tar.gz 文件，只有像下面这样把它们解压到根目录（root）即可：

```
cd /
tar xvf mod-cluster-1.x.y-linux2-x86-ssl.tar.gz
```

httpd.conf 位于 ```/opt/jboss/httpd/httpd/conf```，快速测试只需要添加如下内容：

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

按如下方式开启 httpd:

```
/opt/jboss/httpd/sbin/apachectl start
```

注意：确保在进入生产前使用 SSL。