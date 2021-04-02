Glusterfs源码-RPC模块

一、RPC服务端源码

main()

​	ret = glusterfs_volumes_init(ctx);

​		ret = glusterfs_listener_init(ctx);

​				ret = rpcsvc_create_listeners(rpc, options, "glusterfsd");

​					ret = rpcsvc_create_listener(svc, options, transport_name);

​						ret = rpc_transport_listen(trans);//监听

​						ret = rpc_transport_register_notify(trans, rpcsvc_notify, svc);//注册函数

二、RPC客户端源码

main()

​	    global_rpc = cli_rpc_init(&state);//rpc的初始化等操作

​				rpc = rpc_clnt_new(options, this, this->name, 16);//加载socket.so库

​				ret = rpc_clnt_register_notify(rpc, cli_rpc_notify, this);//注册cli_rpc_notify函数在socket.c的connect的socket_event_poll_out中的rpc_transport_notify调用

​				ret = rpc_clnt_start(rpc);//connect连接

​		ret = cli_cmds_register(&state);//发送请求

​								ret = cli_to_glusterd(&req, frame, gf_cli_create_volume_cbk,

​																		(xdrproc_t)xdr_gf_cli_req, dict,

​																		GLUSTER_CLI_CREATE_VOLUME, this, cli_rpc_prog, NULL);     //socket.c的submit_request

​		rpc_clnt_new()

socket.c //socket相关操作

![image-20210301175146389](/home/lgl/.config/Typora/typora-user-images/image-20210301175146389.png)

三、RPC通信过程

![img](https://pic002.cnblogs.com/images/2012/162856/2012051602183195.png)