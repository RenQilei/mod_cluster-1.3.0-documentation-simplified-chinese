# 第四章 创建 httpd 模块

## 4.1. Build a patched httpd from it sources

To build httpd-2.2.x from its sources see ASF httpd doc [http://httpd.apache.org/docs/2.2/
install.html]

If needed, patch the httpd-2.2.x sources with (The patch prevents long waiting time when the node IP can't be resolved that should not happen so you can skip the patch part if you don't want to rebuild httpd). mod_proxy_ajp.patch [https://github.com/modcluster/mod_cluster/blob/master/
native/mod_proxy_cluster/mod_proxy_ajp.patch]

```
(cd modules/proxy
  patch -p0 < $location/mod_proxy_ajp.patch
)
```

Configure httpd with something like:

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

## 4.2. Build the 4 modules of mod_cluster

You need an httpd installation with mod_proxy (--enable-proxy) and ajp protocol (--enable-proxyajp) enabled and with dso enabled (--enable-so)

Download the mod_cluster sources:

```
git clone git://github.com/modcluster/mod_cluster.git
```

Build the mod_cluster modules components, for each subdirectory advertise, mod_manager, mod_proxy_cluster and mod_slotmem do something like:

```
sh buildconf
./configure --with-apxs=apxs_file_location
make
cp *.so apache_installation_directory/modules
```

Where apache_installation_directory is the location of an installed version of httpd-2-2.x.

NOTE: You can ignore the libtool message on most platform:

```
libtool: install: warning: remember to run `libtool --finish apache_installation_directory/modules'
```

Once that is done use [Apache httpd configuration](chapter3.md##3.1) to configure mod_cluster.

## 4.3. Build the mod_proxy module

It is only needed for httpd-2.2.x where x < 11. Process like the other mod_cluster modules.