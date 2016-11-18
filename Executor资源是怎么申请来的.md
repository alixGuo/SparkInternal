## Executor资源是怎么申请来的

-----------------
- Yarn/Client向RM注册申请启动AM。
- AM启动后向RM注册。AM在向RM注册时，使用的是YarnRMClient.register()，注册完成后返回的是一个YarnAllocator的对象，返回对象后直接YarnAllocator.allocateResources()，进行资源分配。分配的执行器总资源量就等于**spark.executors.instance**或者指定的**--num-executors**的数目。
- 走读**allocateResources()**

1、首先函数代码块加锁，因为这个方法中读取的变量值可能会被其他同时运行的线程方法改变。
	
2、方法主要功能是分配RM给予的containers，也就是在启动的containers里面lanunch executors。
	
3、方法开始会跟RM同步一次资源需求，即执行`updateResourceRequests()`，这个方法获取到pending的container申请，比如说AM申请了container，但是RM现在没有这么多资源，只给了3个，那还有2个就是pending的。获取到这个参数以后，会计算资源申请数目的差额，即用户指定的executors数目减去pending的containers申请数目再减去正在运行的executors数目（这里要注意一下，executor和container是一一对应的，一个container只能运行一个executor）。如果这个差额大于0，会进行差额的申请，如果小于0，说明现在pending的资源申请加上正在运行的资源已经大于用户申请的总资源量了，所以这个时候需要取消额外申请，取消申请的资源数目取的是差额和pending申请资源的最小值。
	
在任务刚启动的时候，pending的申请数目和正在运行的executor数目都是0，所以第一次会申请用户提交任务时指定的或者默认的executors数目。
	
请求的资源会以`amClient.addContainerRequest(request)`的方式添加到amClient中。
	
4、接着`amClient.allocate(progressIndicator)`向RM提交资源，RM会给一个返回`allocateResponse`，通过这个返回我们可以知道RM通知NM启动的containers（`allocatedContainers`），也可以知道剩余的可用资源。下一步就是通过` handleAllocatedContainers(allocatedContainers)`处理RM给予的containers。对于spark来说，就是把executor启动在container里面。

5、**`handleAllocatedContainers`**将真正用到的containers挑选出来，执行`runAllocatedContainers`，方法中为executorId赋值，更新hostToContainer列表，最关键的是`ExecutorRunnable`，它继承了Runable接口，使用`launcherPool.execute(executorRunnable)`启动executor线程。
	
		
