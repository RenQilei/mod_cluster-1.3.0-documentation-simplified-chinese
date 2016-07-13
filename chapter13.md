# 第十三章 从 mod_jk 迁移

Mod_cluster only support Apache httpd, there are no plan to support IIS or Iplanet.
The migration from mod_jk to mod_cluster is not very complex. Only very few worker properties can't be mapped to mod_cluster parameters.

Here is the table of worker properties and how to transfer them in the ClusterListener parameters.

| mod_jk worker property | ClusterListener parameter | Remarks |
| -- | -- | -- |
| host | - | It is read from the <Connector/> Address information |
| port | - | It is read from the <Connector/> Port information |
| type | - | It is read from the <Connector/> Protocol information |
| route | - | It is read from the <Engine/> JVMRoute information |
| domain | domain | That is not supported in this version |
| redirect | - | The nodes with loadfactor = 0 are standby nodes they will be used no other nodes are available |
| socket_timeout | nodeTimeout | Default 10 seconds |
| socket_keepalive | - | KEEP_ALIVE os is always on in mod_cluster |
| connection_pool_size | - | The max size is calculated to be AP_MPMQ_MAX_THREADS+1 (max) |
| connection_pool_minsize | smax | The defaut is max |
| connection_pool_timeout | ttl | Time to live when over smax connections. The defaut is 60 seconds |
