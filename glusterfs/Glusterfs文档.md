

[TOC]



## Glusterfs文档

#### 1、Glusterfs介绍

#### 2、Glusterfs目录结构

#### 3、Glusterfs配置文件及日志文件

​		test-volume2  --卷名    

​		/opt/brick1 --brick路径 

​		服务端：服务进程的graph文件   /var/lib/glusterd/vols/test-volume2/test-volume2.192.168.4.121.opt-brick1.vol

​						  服务端日志文件：/var/log/glusterfs/bricks/opt-brick1.log

​															  /var/log/glusterfs/glusterd.log

​		客户端：客户端挂载的graph文件  /var/lib/glusterd/vols/test-volume2/trusted-test-volume2.tcp-fuse.vol

​						  客户端挂载日志：挂载时指定  sudo mount.glusterfs 192.168.4.121:test-volume2 /glusterfs/data/ -o log-level=info,log-file=**/home/lgl/glusterfs.log**

#### 4、常用命令

#### 5、xlator

#### 6、源码分析

1. 快照
2. 文件操作
3. dht模块
4. mem-pool
5. rpc

