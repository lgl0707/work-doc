###### Glusterfs源码-server     进程名glusterfsd

------

GlusterFS版本6.6

疑惑 ：结构体时如赋值的  ret = glusterd_restore();//获取 peer， volume， bricks 等信息，保存到结构体中。

mgmt初始化过程中对结构体进行赋值

glusterfsd.c中的main函数为入口：



1、 ret = glusterfs_globals_init(ctx);

//该函数主要初始化全局变量

2、ret = glusterfs_ctx_defaults_init(ctx);

//初始化 ctx 众多参数

3、ret = parse_cmdline(argc, argv, ctx);

//解析命令行参数

4、ret = logging_init(ctx, argv[0]);

//log 初始化，打开 log 文件

5、ret = create_fuse_mount(ctx);

//damon和server 由于没有 mount point，所以会直接返回，我们将在 client 中详细分析。

6、ret = daemonize(ctx);

//根据是否 debug 模式等参数，来决定是否启动新进程重新定向输入输入，并关闭此 shell。

7、mem_pools_init();

//内存池的初始化

8、ret = glusterfs_volumes_init(ctx);

//创建 brick 监听端口，同 damon 建立连接获取相关 brick 配置信息。

​	8.1 if (cmd_args->sock_file) { ret = glusterfs_listener_init (ctx);...}

​	//创建 brick 监听端口， 注册事件处理函数，监听客户端发送过来的信息

​		8.1.1ret = rpcsvc_transport_unix_options_build (&options,cmd_args->sock_file);

​		//设置 rpc-server 的参数信息 options，包含 sock_file,等信息。

​		8.1.2 rpc = rpcsvc_init (ctx, options);

​		//创建个 prc-server，并初始化，类似第二章的 8.3.3.2.2.

​		8.1.3 ret = rpcsvc_register_notify (rpc, glusterfs_rpcsvc_notify, THIS);

​		//注册 rpc-server 的通知处理函数，参考上章的相关信息

​		8.1.4 ret = rpcsvc_create_listeners (rpc, options, "glusterfsd");

​		//创建 listeners，参考上章的相关信息

​		8.1.5 ret = rpcsvc_program_register (rpc, &glusterfs_mop_prog);

​		//注册事件处理，参考上章相关信息

​	8.2 if (cmd_args->volfile_server) { ret = glusterfs_mgmt_init (ctx);...}

​	//同 damon 建立连接，获取 brick 配置信息。

​		8.2.1 ret = rpc_transport_inet_options_build (&options, host, port);

​		//设置字典 options 的 host 和 port 等信息。即连接本 host 的 deamon 的监听端口

​		8.2.2 rpc = rpc_clnt_new (options, THIS->ctx, THIS->name);

​		//创建一个 RPC

​		8.2.3 ret = rpcclnt_cbk_program_register (rpc, &mgmt_cbk_prog);

​		//注册 rpc 的回调处理函数结构 mgmt_cbk_prog

​		8.2.4 ret = rpc_clnt_start (rpc);

​		//调用 rpc 的 conn 启动 trans

9、ret = gf_event_dispatch(ctx->event_pool);

//调用创建 socket 时候注册的事件处理函数处理事件。