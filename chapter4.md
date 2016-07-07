# 第四章 构建 httpd 模块

## 4.1. 从它的源文件构建一个打补丁的 httpd

从它的源文件来构建一个 httpd-2.2.x，请参见 [ASF httpd 文档](http://httpd.apache.org/docs/2.2/install.html) [http://httpd.apache.org/docs/2.2/install.html]

如果需要，可以和 [mod_proxy_ajp.patch](https://github.com/modcluster/mod_cluster/blob/master/ native/mod_proxy_cluster/mod_proxy_ajp.patch) [https://github.com/modcluster/mod_cluster/blob/master/
native/mod_proxy_cluster/mod_proxy_ajp.patch] 来给 httpd-2.2.x 源文件打补丁（不定防止当节点 IP 不能被决定这样不该发生的事造成长时间等待时，如果你不想要重新构建 httpd，你可以跳过补丁这部分）。

```
(cd modules/proxy
  patch -p0 < $location/mod_proxy_ajp.patch
)
```

像下面这样配置 httpd：

```
./configure --prefix=apache_installation_directory \
  --with-mpm=worker \
  --enable-mods-shared=most \
  --enable-maintainer-mode \
  --with-expat=builtin \
  --enable-ssl \
  --enable-proxy \
  --enable-proxy-http \
  --enable-proxy-ajp \
  --disable-proxy-balancer
```

Rebuild (make) and reinstall (make install) after that.

## 4.2. 构建 mod_cluster 的 4 个模块

你需要安装 httpd，包括启用的 mod_proxy (--enable-proxy) 和 ajp 协议 (--enable-proxyajp)，和启用的 dso (--enable-so)。

下载 mod_cluster 源文件：

```
git clone git://github.com/modcluster/mod_cluster.git
```

构建 mod_cluster 模块组件，为每个子目录 advertise, mod_manager, mod_proxy_cluster 和 mod_slotmem 做如下处理：

```
sh buildconf
./configure --with-apxs=apxs_file_location
make
cp *.so apache_installation_directory/modules
```

apache_installation_directory 是 httpd-2.2.x 的已安装的版本的路径。

注意：你可以在绝大多数平台上忽略 libtool 信息：

```
libtool: install: warning: remember to run `libtool --finish apache_installation_directory/modules'
```

一旦完成，使用 [Apache httpd 配置](chapter3.md) 中所述来配置 mod_cluster。

## 4.3. 构建 mod_proxy 模块

它仅在 httpd-2.2.x 且 x<11 时需要。处理类似于其他 mod_cluster 模块。
