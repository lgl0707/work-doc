Glusterfs源码-daemon                进程名glusterd

一、glusterfsd.c中的main函数为入口：

1、 ret = glusterfs_globals_init(ctx);

//该函数主要初始化全局变量

2、ret = glusterfs_ctx_defaults_init(ctx);

//初始化 ctx 众多参数

3、ret = parse_cmdline(argc, argv, ctx);

//解析命令行参数

4、ret = logging_init(ctx, argv[0]);

//log 初始化，打开 log 文件

​	4.1 if (ENABLE_DEBUG_MODE == cmd_args->debug_mode)

​	//debug 模式则重新定向 log 的等级和输出。

​	4.2 process_mode = gf_get_process_mode (argv[0]);

​	//根据可执行程序的名称来判定该应用程序的工作模式。 并设置相关卷文件路径该：

​	cmd_args->volfile = gf_strdup (DEFAULT_CLIENT_VOLFILE)。该 damon 模式为GF_GLUSTERD_PROCESS，还有 GF_SERVER_PROCESS（server），                                           	GF_CLIENT_PROCESS（client）

5、ret = create_fuse_mount(ctx);

//damon和server 由于没有 mount point，所以会直接返回，我们将在 client 中详细分析。

6、ret = daemonize(ctx);

//根据是否 debug 模式等参数，来决定是否启动新进程重新定向输入输入，并关闭此 shell。

7、mem_pools_init();

//内存池的初始化

8、ret = glusterfs_volumes_init(ctx);

//该函数负责创建监听端口，与监听端口建立连接

​	8.1 if (cmd_args->sock_file) //建立 brick 监听端口           if (cmd_args->volfile_server)//同 damon 监听端口建立连接

​	//damon 命令行不含该参数，所以不会执行两处 if 语句。

​	8.2 fp = get_volfp (ctx);

​	//打开卷配置文件，获取文件操作句柄，此卷文件为：/usr/local/etc/glusterfs/glusterd.vol

​	8.3 ret = glusterfs_process_volfp (ctx, fp);

​	//对卷文件进行解析，并执行该卷信息所对应的 translator 的操作。

​		8.3.1 graph = glusterfs_graph_construct (fp);

​		//利语法分析器 yyparse（）（google search）函数用解析卷配置文件构建一个graph 结构体。 在解析过程中调用 xlator_set_type 已经将各个 translator 对		应的动态库打开，并获取了相关函数和结构体的指针。 参考： 8.3.2.1.2 和libglusterfs/src/graph.y 文件。

​		8.3.2 ret = glusterfs_graph_prepare (graph, ctx);

​		//对构建的 graph 结构预处理一下，其内部处理情况为：

​			8.3.2.1 ret = glusterfs_graph_settop (graph, ctx);

​			//设置 graph 的 top translator，默认卷配置文件中的最后一个 translator

​			8.3.2.1 ret = glusterfs_graph_readonly (graph, ctx);

​			ret = glusterfs_graph_acl (graph, ctx);

​			ret = glusterfs_graph_mac_compat (graph, ctx);

​			//根据 cmd_args 参数信息调用 glusterfs_graph_insert（）函数来决定是否额外添加一个 translator，并设置为 raph 的 top。

​		8.3.3 ret = glusterfs_graph_activate (graph, ctx);

​		//初始化 graph 中的各个 translator，创建 socket，建立事件监听函数。

​			8.3.3.1 ret = glusterfs_graph_init (graph);

​			//参考 8.3.1，自上而下调用 graph 中各个 translator 的 init （）函数初始化。此 damon只会调用 mgmt 的 init 函数，详细请参考GlusterFS的mgmt模块                                                                			（xlators/mgmt/glusterd/src/glusterd.c）。在初始化过程中建立/etc/glusterd/下的 vols， peers 等文件夹，创建监听端口，设置事件处理函数，恢复上			次 deamon 退出的状态灯操作

​		8.3.3.2 ret = glusterfs_graph_parent_up (graph);

​		//调用卷配置文件中的各个 translator 的 notify 函数，由于 ctx->master 为空，所以不会执行 ret = xlator_notify (ctx->master,GF_EVENT_GRAPH_NEW, 			graph);调用 mgmt 的 notify 函数，发送GF_EVENT_PARENT_UP命令。该 mgmt 调用 default_notify (this, event, data);没有做什么具体操作，可以忽略。

9、ret = gf_event_dispatch(ctx->event_pool);

//调用创建 socket 时候注册的事件处理函数处理事件。





二、mgmt的init（xlators/mgmt/glusterd/src/glusterd.c）

1、 dir_data = dict_get (this->options, "working-directory");

//获取工作主目录

ret = gf_cmd_log_init (cmd_log_filename);

//设置 CLI 命令 log 文件

ret = mkdir (voldir, 0777);

//创建 vols， peers 等工作目录

2、ret = glusterd_rpcsvc_options_build (this->options);

//设置 PRC server 的选项配置信息 options

​	2.1 rpc = rpcsvc_init (this->ctx, this->options);

​	//创建一个 rpc server 结构体，并初始化一些参数信息。利用函数 ret = rpcsvc_program_register (svc, &gluster_dump_prog);添加gluster_dump_prog 到 		svc->programs 链表上。

3、ret = rpcsvc_register_notify (rpc, glusterd_rpcsvc_notify, this);

//主次 rpc server 一个 notify 处理函数。将该处理函数添加到svc->notify 列表上。

4、 ret = rpcsvc_create_listeners (rpc, this->options, this->name);

​	//利用 rpc 类型调用 ret = rpcsvc_create_listener (svc,options, transport_name);创建 listener， 创建过程如下：

​	4.1 trans = rpcsvc_transport_create (svc, options, name);

​	//创建一个 rpc_transport_t 结构体，该结构体动态载入 soket库以及其函数指针，创建一个 socket，其创建过程如下：

​		4.1.1 trans = rpc_transport_load (svc->ctx, options, name);

​		//动态载入相应类型的 RPC 库并调用库的 init 初始化。

​		4.1.2 ret = rpc_transport_listen (trans);

​		//调用 sokcet 库的 listen 建立监听端口， 并调用ctx->event_pool 的 event_register_epoll 函数将 sokcet 句柄利用 epoll_ctl 监听。 详细见参考 soket 的 listen 函数的源码和event_register_epoll 函数。

​		4.1.3 ret = rpc_transport_register_notify (trans,rpcsvc_notify, svc); rpcsvc_notify

​		//注册 trans 的 notify 处理函数为

​	4.2 listener = rpcsvc_listener_alloc (svc, trans);

​	//创建 listener，并将该 rpc server 和 trans 赋值给 listener。将该 listener 添加到 prc server 的 svc->listeners 链表上。

5、ret = glusterd_program_register (this, rpc,&glusterd1_mop_prog);

ret = glusterd_program_register (this, rpc, &gd_svc_cli_prog);

ret = glusterd_program_register (this, rpc, &gd_svc_mgmt_prog);

ret = glusterd_program_register (this, rpc, &gluster_pmap_prog);

ret = glusterd_program_register (this, rpc, &gluster_handshake_prog);

//注册 5 个事件处理结构体到 rpc->programs 列表上

6、ret = configure_syncdaemon (conf);

//a.定义 RUN_GSYNCD_CMD(prf)，该函数用来 damon 启动执行命令

//b.配置 geosync 信息。

7、ret = glusterd_restore ();

//从/etc/glusterd/目录获取获取以前操作的 peer， volume， bricks 等信息，保存到结构体中。

8、glusterd_restart_bricks (conf);

//根据上次 damon 运行状态针对每个 brick 启动一个 brick 服务“glusterd_brick_start (volinfo, brickinfo);” 该函数会调用“ret = glusterd_volume_start_glusterfs (volinfo, brickinfo);”启动卷服务。该函数前面大部分工作在于设置命令行参数，最后调用“ret = gf_system (cmd_str);”执行 ret = execvp (argv[0], argv)来创建新的进程启动服务。最后并确定是否启动 nfs 服务：“glusterd_check_generate_start_nfs ();”

9、ret = glusterd_restart_gsyncds (conf);

//启动远程同步功能