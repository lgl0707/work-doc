GlusterFS架构

一、GlusterFS工作时的四大进程

1. cli即命令行执行工具

   该应用负责把对 GlusterFS 的操作请求发送到glusterd 上去执行。

   例：gluster vol create test-volume2 192.168.4.121:/glusterfs/data/ force   //创建卷

2. demon进程。

   该 damon 负责接收 gluster 发送过来的操作请求，并执行相关的操作，例如调用 glusterfsd 启动 brick 服务。

   ps aux | grep gluster    可看见进程/usr/local/sbin/glusterd -p /var/run/glusterd.pid

3. server进程

   由 glusterd 启动。根据卷配置信息执行由 glusterfs 发送过来的请求。 通过 rpc 连接 glusterd 获取该服务的卷信息。

   ps aux | grep gluster    可看见该进程

   /usr/local/sbin/glusterfsd -s 192.168.4.121 --volfile-id test-volume2.192.168.4.121.glusterfs-data -p /var/run/gluster/vols/test-volume2/192.168.4.121-glusterfs-data.pid -S /var/run/gluster/7e7fc7156a78fe2a.socket --brick-name /glusterfs/data -l /var/log/glusterfs/bricks/glusterfs-data.log --xlator-option *-posix.glusterd-uuid=20564e3d-4745-416e-93e7-ef6d6b9ab516 --process-name brick --brick-port 49153 --xlator-option test-volume2-server.listen-port=49153

4. 客户端挂载进程

   根据卷配置信息将 fuse 过来的操作请求逐层专递到最底层的 protocol/client translater 上，该 translater 通过 RPC 与 glusterfsd 连接，将请求发送到 glusterfsd 服务端继续执行。 通过 rpc 连接 glusterd 获取该服务的卷信息。

   ps aux | grep gluster    可看见该进程

   /usr/local/sbin/glusterfs --process-name fuse --volfile-server=192.168.4.121 --volfile-id=test-volume2 /home/lgl/test/

二、工作原理

[GlusterFS 结构体系分析]: https://blog.csdn.net/wangyuling1234567890/article/details/24601417?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.control&amp;dist_request_id=93d66fdf-5fec-4ef3-b195-3eee659feca9&amp;depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.control



[复制卷的恢复机制]: https://www.cnblogs.com/kevingrace/p/8764902.html



[GlusterFS分布式存储]: https://www.cnblogs.com/kevingrace/p/8709544.html





