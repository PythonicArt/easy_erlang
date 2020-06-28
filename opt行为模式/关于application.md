# application
将一系列资源打包成半独立单位的行为模式
可以被配置, 可以作为一个整体启动和停止

每一个节点有一个 application controller, 用于管理各个独立的application

normal application / library application

# 目录结构
root
    ebin  
        包含 beam文件 配置文件
    src
        源代码
    priv
        非erlang文件， 图像，驱动程序， 脚本等
    include
        头文件

# 启动过程
```erlang
    application:load(Name).
    需要Name在文件搜索路径内， 当前目录默认加入搜索路径,
    erl -pa PATH 可以添加路径到搜索路径中

    application:start(Name).

    加载application, 启动顶级监督者，

    加载所有打包在application中的模块， 启动application master进程
    通过调用 回调模块里的  mode:start() 启动一个进程, 返回的进程pid作为 顶级监督者
    由其负责启动子进程, 形成监督树
```

# 停止过程
```erlang
    application:stop(Name).
    停止，在监督树包括所有下级worker被终止后, mod:stop() 被调用
```

# 资源文件
application的描述， 位于ebin文件夹， 和.beam文件在一起
节点启动通过指定pa目录， 节点可以此加载和启动application
资源文件定义以下内容
    名称
    版本号
    包含的模块名称
        在application加载时, 自动从 ebin中加载。 如果没有指明, 则在调用时加载
    依赖的application
        在启动当前application前, 其依赖的application必须启动.
        可以手动逐一启动， 也可以使用一下命令批量启动
        ```erlang
            application:ensure_all_started(Name)
        ```
    使用的环境变量
    回调模块
        指明回调模块和启动参数

# 环境变量
每个application有独立的环境变量设置，可以根据配置而调整
1. 资源文件配置
2. 统一配置
```erlang
    erl -config Config
    [
        {Application1, []},
        {Application2, []},
        ...
    ]

```
3.
erl -applicationName key value

获取环境变量
```erlang
    application:get_env(AppName, Key).
    application:get_env(Key). %% 当前进程位于的App内的环境变量
    application:get_application(). %% 查询当前进程位于哪一个application
```


# 分布式application
## 故障转移
在集群中运行application， 某一个节点提供服务， 当该节点出现故障， 则由其他节点提供服务， 这叫故障转移
不同节点有不同的优先级， 优先级高的节点出现故障， 由下一级的节点提供服务

## 接管
当出现优先级高的节点, 则application迁移到该节点, 由该节点提供服务

# 分阶段启动
在资源文件中配置 start_phase选项, 会在启动时调用 mod:start_phase(), 多个启动阶段会顺序地执行。

# sasl
报告类型
查看报告
	rb:list()
	rb:show()
