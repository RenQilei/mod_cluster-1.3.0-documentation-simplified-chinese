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
| - | workerTimeout | 	Max time to wait for a free worker default 1 second |
| retries | maxAttempts | Max retries before returning an error Default: 3 |
| recovery_options | - | mod_cluster behave like mod_jk with value 7 |
| fail_on_status | - | Not supported |
| max_packet_size | iobuffersize/receivebuffersize | Not supported in this version. Use ProxyIOBufferSize |
| max_reply_timeouts | - | Not supported |
| recovert_time | - | The ClusterListener will tell (via a STATUS message) mod_cluster that the node is up again |
| activation | - | mod_cluster receives this information via ENABLE/DISABLE/STOP messages |
| distance | - | mod_cluster handles this via the loadfactor logic |
| mount | - | The context “mounted” automaticly via the ENABLE-APP messages. ProxyPass could be used too |
| secret | - | Not supported |
| connect_timeout | - | Not supported. Use ProxyTimeout or server TimeOut (Default 300 seconds) |
| prepost_timeout | ping | Default 10 seconds |
| reply_timeout | - | Not supported. Use ProxyTimeout or server TimeOut? directive (Default 300 seconds) |
