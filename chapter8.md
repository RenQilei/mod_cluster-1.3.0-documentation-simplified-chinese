# 第八章 构建服务器端组件

## 8.1. 要求

从源文件构建 mod_cluster 的服务器端的组件需要以下工具：

* JDK 5.0+
* Maven 2.0+

## 8.2. 构建

构建的步骤:

1. 下载 mod_cluster 源文件

```
git clone git://github.com/modcluster/mod_cluster.git
```

  2. 使用 maven "dist" profile 来构建:

```
cd mod_cluster
mvn -P dist package
```

> **Note**
>
> 一些单元测试需要 UDP 端口 23365。确保你的本地防火墙允许这个端口。

## 8.3. 构建的人工痕迹（Artifacts）

构建会在目标文件夹中产生下面的这些输出：

**mod-cluster.sar**

解压格式为 sar 的文件并复制到你的 JBoss AS 安装包的部署文件夹中。

**JBossWeb-Tomcat/lib directory**

Jar 文件复制到你的 JBossWeb 或 Tomcat 安装包的函数库文件夹中，来支持 mod_cluster 的使用。

**demo directory**
负载均衡实例 [缺少锚链接] 应用。

**mod-cluster-XXX.tar.gz**
完整发布版的 tarball，包括上述元素。