# 第十一章 安装服务器端组件

首先，解压服务器端二进制文件到一个零时目录。下面假设它解压到了 ```/tmp/mod_cluster```

你的下一步取决于你的目标服务器是 JBoss AS 还是 JBossWeb/Tomcat。

## 11.1. 在 JBoss AS 6.0.0.M1 及以上中安装

在 AS 6.x 中你不需要做任何事情来安装 java端二进制文件；这是默认的，标准的，都配置了的 AS 发行版的一部分。

## 11.2. 在 JBoss AS 5.x 中安装

假设 ```$JBOSS_HOME``` 指明了你的 JBoss AS 安装的根目录，你想要在 AS 的所有配置中使用 mod_cluster：

```
cp -r /tmp/mod_cluster/mod_cluster.sar $JBOSS_HOME/server/all/deploy
```

## 11.3. 在 Tomcat 中安装

假设 ```$CATALINA_HOME``` 指明了你的 Tomcat 安装的根目录：

```
cp /tmp/mod_cluster/JBossWeb-Tomcat/lib/jboss-logging.jar $CATALINA_HOME/lib/
cp /tmp/mod_cluster/JBossWeb-Tomcat/lib/mod_cluster-container-catalina* $CATALINA_HOME/lib/
cp /tmp/mod_cluster/JBossWeb-Tomcat/lib/mod_cluster-container-spi* $CATALINA_HOME/lib/
cp /tmp/mod_cluster/JBossWeb-Tomcat/lib/mod_cluster-core* $CATALINA_HOME/lib/
```

此外，为 Tomcat6 需要：

```
cp /tmp/mod_cluster/JBossWeb-Tomcat/lib/mod_cluster-container-tomcat6* $CATALINA_HOME/lib
```

此外，为 Tomcat7 需要：

```
cp /tmp/mod_cluster/JBossWeb-Tomcat/lib/mod_cluster-container-tomcat7* $CATALINA_HOME/lib
```