#### 创建队列
系统提供了两个队列，一个是 MainDispatchQueue，一个是 GlobalDispatchQueue。

* 前者会将任务插入主线程的RunLoop当中去执行，所以显然是个串行队列，我们可以使用它来更新UI。
 
 ```
 dispatch_queue_t queue = dispatch_get_main_queue()
 ```
* 后者则是一个全局的并行队列，有高、默认、低和后台4个优先级。

 ```
dispatch queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRORITY_DEFAULT, 0)
 ```

#### 往队列中添加一个任务

```
dispatch_async(queue, block)
```
block是一个任务。

#### 任务只执行一次

这种线程安全的方式可以使用在单例构造中

```
 + (id)shareInstance {
　      static dispatch_once_t onceToken;
　      dispatch_once(&onceToken, ^{
　          _shareInstance = [[self alloc] init];
　　    });
```

#### 执行异步任务

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_async(queue, ^{
	//异步任务代码段
	dispatch_async(dispatch_get_main_queue(), ^{
 	//回到主ui线程中，可以更新ui
 	});
});

```

#### 任务组

可以让多个任务并行执行，全部执行结束统一处理后续操作

```
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, queue, ^{ NSLog(@"1"); });
dispatch_group_async(group, queue, ^{ NSLog(@"2"); });
dispatch_group_notify(group, dispatch_get_main_queue(), ^{ NSLog(@"done"); });
```

#### 延迟执行

```
dispatch_after(dispatch_time(DISPATCH_TIME_NOW,(int64_t)(10*NSEC_PER_SEC), dispatch_get_main_queue(),^{
 //需要延迟执行的任务
});
```

#### 同步和异步调用

dispatch_async 异步调用、dispatch_sync 同步调用

```
dispatch_async(queue, ^{
	NSLog(@"1");
});
NSLog(@"2”);
```

异步调用，输出结果可能是12或是21，同是在RunLoop中的线程，不清楚谁先执行；

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
　　 dispatch_sync(queue, ^{
	NSLog(@"1");
});
NSLog(@"2”);
```
同步调用，将任务放在子线程中执行，等待子线程执行完毕，在继续煮线程的操作，执行结果12；

```
dispatch_queue_t queue = dispatch_get_main_queue();
　　 dispatch_sync(queue, ^{
	NSLog(@"1");
});
```
在主线程中，不能使用主线程队列的同步调用，这样就会发生死锁，互相等待完成；

* 主线程通过dispatch_sync把block交给主队列后，会等待block里的任务结束再往下走自身的任务；
* 而队列是先进先出的，block里的任务也在等待主队列当中排在它之前的任务都执行完了再走自己。

#### 串行并行队列及读写安全
略

#### 暂停和恢复

使用 dispatch_suspend(queue) 可以暂停队列中任务的执行，使用 dispatch_result(queue) 可以继续执行被暂停的队列。
