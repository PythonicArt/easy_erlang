服务器的工作机制
	分裂出服务进程
	循环重复处理消息
	退出

通用部分
	进程的分裂， 注册
	向进程发送消息
	进程接收消息
	进程退出

专用部分
	进程数据的初始化
	进程数据的清理
	具体的消息的处理方式

通用部分与专用部分分离的 设计模式
	分裂进程， 指定 注册名称， 回调模块位置。

	通用部分通过 回调模块位置， 调用其导出的 规定的 回调函数来使用 专用部分地功能

并发编程的问题
	竞态条件， 死锁， 临界区， 边界条件

一些需要考虑的问题
	1. client 发送请求之后通过receive进入挂起状态， 当消息来临时， 如何确定是来自哪一个server的？
	2.  client 发送请求之后通过receive进入挂起状态， 如果server奔溃， 则客户端持续挂起？
	3. 如果server接收到请求， 发送回复之后再崩溃。 由于监控的原因， 会收到对应Ref的down消息， 然而此时client可能在监控其他进程， 此条down消息
	将无法得到匹配，会持续存在邮箱中， 导致内存泄漏。如何处理？
	4. 两个进程互发消息， 都进入receive， 这种死锁如何处理？


同步消息与异步消息的区别
	实现过程？
	如何选择哪一种消息？
		ping 选择cast?

关于gen_server的一些细节
1. 新建gen_server
	两种类型 link, nolink

	Name, Type, Mod, Args, Opts, 决定了一个特殊的 gen_server

	当前进程调用 spawn, 分裂一个进程， 使用recevive 进入阻塞状态，等待新进程创建完毕消息

	新的进程初始化
		新的进程的祖先信息继承自当前进程
		执行回调模块里的初始化函数， Mod:init(Args), 获得初始状态{ok, State}
		给父进程 即 Starter 回复创建完毕消息 {ok, Pid} 或错误消息。
		自己进入循环等待 loop

	当前进程收到创建完毕消息， 获得返回结果， 返回

2. gen_server的退出
	收到退出消息 {EXIT}, 得到ExitReason
	调用回调模块 Mod:terminate


2. 关于loop循环
		code: loop(Paren, )
		就是使用receive 等待消息， 然后处理消息。

3. 消息的处理
	退出消息： 结束进程
	正常消息
		调用回调模块对应的处理函数， 获得结果。
		根据结果确定是否进行客户端同步(reply), 终止进程， 重新进入loop

4. 消息的发送
	call:
		监控目标进程 得到 mref
		发送消息后， 使用receive, 等待目标消息的回复
		当前只处理和mref相关的回复或是超时消息

		正常回复时， 使用demonitor 清理掉所有与 mref相关的消息, 因为已经不需要。
		如果出错或是超时， 自身要exit()?

	cast:
		直接使用 erlang:send 发送。不 使用receive等待。 只管发。也就是目标进程发送回复。

	其他消息：
		同cast
