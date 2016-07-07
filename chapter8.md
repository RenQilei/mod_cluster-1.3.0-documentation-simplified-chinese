# 第八章 构建服务器端组件

## 8.1. Requirements

Building mod_cluster's server-side components from source requires the following tools:

* JDK 5.0+
* Maven 2.0+

## 8.2. Building

Steps to build:

1. Download the mod_cluster sources

```
git clone git://github.com/modcluster/mod_cluster.git
```

2. Use maven "dist" profile to build:

```
cd mod_cluster
mvn -P dist package
```

```
**Note**
Some unit tests require UDP port 23365. Make sure your local firewall allows the port.
```