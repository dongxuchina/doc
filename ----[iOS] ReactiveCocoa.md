### RACSignal 类簇
虽然 -createSignal: 的方法签名上返回的是 RACSignal 对象的实例，但是实际上这里返回的是 RACDynamicSignal，也就是 RACSignal 的子类；同样，在 ReactiveCocoa 中也有很多其他的 RACSignal 子类。

使用类簇的方式设计的 RACSignal 在创建实例时可能会返回 RACDynamicSignal、RACEmptySignal、RACErrorSignal 和 RACReturnSignal 对象。

```
+ (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe {
    return [RACDynamicSignal createSignal:didSubscribe];
}

+ (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe {
    RACDynamicSignal *signal = [[self alloc] init];
    signal->_didSubscribe = [didSubscribe copy];
    return [signal setNameWithFormat:@"+createSignal:"];
}

+ (RACSignal *)error:(NSError *)error {
    return [RACErrorSignal error:error];
}

+ (RACSignal *)empty {
    return [RACEmptySignal empty];
}

+ (RACSignal *)return:(id)value {
    return [RACReturnSignal return:value];
}
```


### RAC 冷热信号

* 冷信号是被动的，只有当你订阅的时候，它才会发送消息。
* 热信号是主动的，即使你没有订阅事件，它仍然会时刻推送。
* 冷信号只能一对一，当有不同的订阅者，消息会从新完整发送。
* 热信号可以有多个订阅者，是一对多，信号可以与订阅者共享信息。

**冷信号**

```
RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    
    NSLog(@"send A");
    [subscriber sendNext:@"A"];
    [subscriber sendCompleted];
    return [RACDisposable disposableWithBlock:^{
        JYLog(@"Original Signal Dispose.");
    }];
}];
NSLog(@"create Subscriber S1");
[signal subscribeNext:^(id x) {
    NSLog(@"S1 recveive: %@", x);
}];
NSLog(@"create Subscriber S2");
[signal subscribeNext:^(id x) {
    NSLog(@"S2 recveive: %@", x);
}];
    
//打印结果>>
		create Subscriber S1
		send A
		S1 recveive: A
		Original Signal Dispose.
		create Subscriber S2
		send A
		S2 recveive: A
		Original Signal Dispose.
```
只有有订阅者的情况下，才发送消息，并且每定义一个订阅者，就会调用didSubscribe完成消息发送。

**热信号**

```
RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    NSLog(@"send A");
    [subscriber sendNext:@"A"];
    [subscriber sendCompleted];
    return [RACDisposable disposableWithBlock:^{
        JYLog(@"Original Signal Dispose.");
    }];
}];
//将冷信号转成热信号
RACSignal *hotSignal = [signal replay];
    
NSLog(@"create Subscriber S1");
[hotSignal subscribeNext:^(id x) {
    NSLog(@"S1: %@",x);
}];
NSLog(@"create Subscriber S2");
[hotSignal subscribeNext:^(id x) {
    NSLog(@"S2: %@",x);
}];

//打印结果>>
		send A
		Original Signal Dispose.
		create Subscriber S1
		S1: A
		create Subscriber S2
		S2: A
```
didSubscribe 只执行一次，订阅者分别接收到消息。

### RACSubject

* RACSubject: 信号提供者，自己可以充当信号，又能发送信号。
* RACReplaySubject: 信号重复提供者，RACSubject的子类。
* RACSubject 需要先订阅再发送消息，订阅者才能接收到消息，RACReplaySubject则不用考虑订阅的先后顺序。

**RACSubject**

```
RACSubject *subject = [RACSubject subject];
    
[subject subscribeNext:^(id x) {
    NSLog(@"S1: %@",x);
}];
    
[subject sendNext:@"A"];
    
//打印结果>>
		send A
		S1: A
   
```

**RACReplaySubject**

有三种类型，下面分别说下不同之处。

1. replay 

```
RACSubject *sub = [RACSubject subject];
RACSignal *reqlaylastSignal = [sub replay];
    
NSLog(@"send A");
[sub sendNext:@"A"];
[reqlaylastSignal subscribeNext:^(id x) {
    NSLog(@"S1: %@", x);
}];
    
NSLog(@"send B");
[sub sendNext:@"B"];
[reqlaylastSignal subscribeNext:^(id x) {
    NSLog(@"S2: %@", x);
}];
    
NSLog(@"send C");
[sub sendNext:@"C"];
[reqlaylastSignal subscribeNext:^(id x) {
    NSLog(@"S3: %@", x);
}];
    
//打印结果>>
	    send A
		S1: A
		send B
		S1: B
		S2: A
		S2: B
		send C
		S1: C
		S2: C
		S3: A
		S3: B
		S3: C

```
通过上面的代码可以知道，当源信号被订阅时，会立即发送给订阅者**全部历史的值**，不会重复执行源信号中的订阅代码。

2. replayLast

```
RACSubject *sub = [RACSubject subject];
RACSignal *reqlaylastSignal = [sub replayLast];
    
NSLog(@"send A");
[sub sendNext:@"A"];
[reqlaylastSignal subscribeNext:^(id x) {
    NSLog(@"S1: %@", x);
}];
    
NSLog(@"send B");
[sub sendNext:@"B"];
[reqlaylastSignal subscribeNext:^(id x) {
    NSLog(@"S2: %@", x);
}];
    
NSLog(@"send C");
[sub sendNext:@"C"];
[reqlaylastSignal subscribeNext:^(id x) {
    NSLog(@"S3: %@", x);
}];

//打印结果>>
		send A
		S1: A
		send B
		S1: B
		S2: B
		send C
		S1: C
		S2: C
		S3: C
```
通过上面的代码可以知道，当源信号被订阅时，会立即发送给订阅者**最新的值**，不会重复执行源信号中的订阅代码。


3. replayLazily

```
RACSubject *sub = [RACSubject subject];
RACSignal *reqlaylastSignal = [sub replayLazily];
    
NSLog(@"send A");
[sub sendNext:@"A"];
[reqlaylastSignal subscribeNext:^(id x) {
    NSLog(@"S1: %@", x);
}];
    
NSLog(@"send B");
[sub sendNext:@"B"];
[reqlaylastSignal subscribeNext:^(id x) {
    NSLog(@"S2: %@", x);
}];
    
NSLog(@"send C");
[sub sendNext:@"C"];
[reqlaylastSignal subscribeNext:^(id x) {
    NSLog(@"S3: %@", x);
}];
    
//打印结果>>
		send A
		send B
		S1: B
		S2: B
		send C
		S1: C
		S2: C
		S3: C
```
通过上面的代码可以知道，当源信号被订阅时，会立即发送给订阅者**全部历史的值**，不会重复执行源信号中的订阅代码，与replay不同的是只有第一次订阅才会激活信号，在激活信号前，发消息都是无效的。是一种懒加载的方式。

### RACMulticastConnection

```
RACSignal+Operation.h中
- (RACMulticastConnection *)publish;

- (RACMulticastConnection *)multicast:(RACSubject *)subject;

- (RACSignal *)replay;

- (RACSignal *)replayLast;

- (RACSignal *)replayLazily;
```

**publish & multicast**

```
RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    
    NSLog(@"send A");
    [subscriber sendNext:@"A"];
    [subscriber sendCompleted];
    return [RACDisposable disposableWithBlock:^{
        NSLog(@"Original Signal Dispose.");
    }];
}];
    
//    RACMulticastConnection *connection = [signal publish]; //默认RACSubject
RACMulticastConnection *connection = [signal multicast:[RACReplaySubject subject]];//RACReplaySubject
    
[connection connect]; //如果使用的是RACReplaySubject，connect可以在创建订阅者之前执行
    
[connection.signal subscribeNext:^(id x) {
    NSLog(@"S1: %@", x);
}];
[connection.signal subscribeNext:^(id x) {
    NSLog(@"S2: %@", x);
}];

//打印结果>>
		send A
		Original Signal Dispose.
		S1: A
		S2: A
```

**replay & replayLast & replayLazily**

```
RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    
    NSLog(@"send A");
    [subscriber sendNext:@"A"];
    [subscriber sendCompleted];
    return [RACDisposable disposableWithBlock:^{
        NSLog(@"Original Signal Dispose.");
    }];
}];
    
RACSignal *hotSignal = [signal replay];
//    RACSignal *hotSignal = [signal replayLast];
//    RACSignal *hotSignal = [signal replayLazily];
    
[hotSignal subscribeNext:^(id x) {
    NSLog(@"S1: %@", x);
}];
[hotSignal subscribeNext:^(id x) {
    NSLog(@"S2: %@", x);
}];

//打印结果>>
		send A
		Original Signal Dispose.
		S1: A
		S2: A

```

### RAC信号流

RAC 提供了许多 RACSignal Operation 方便我们使用 ，其中 [RACSignal bind:] 操作是信号变换的核心。并衍生出 flattenMap、 map、flatten 等。

**#bind**

bind函数会返回一个新的信号N。整体思路是对原信号O进行订阅，每当信号O产生一个值就将其转变成一个中间信号M，并马上订阅M, 之后将信号M的输出作为新信号N的输出。
    
```
    RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@"A"];
        [subscriber sendCompleted];
        return [RACDisposable disposableWithBlock:^{
            JYLog(@"Original Signal Dispose.");
        }];
    }];
    
    RACSignal *bindSignal = [signal bind:^RACStreamBindBlock{
        return ^(NSString *value, BOOL *stop) {
            return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
                JYLog(@"Original Signal value = %@",value);
                NSString *bindValue = [NSString stringWithFormat:@"%@ & %@",value, @"B"];
                JYLog(@"Binding Signal value = %@",bindValue);
                [subscriber sendNext:bindValue];
                [subscriber sendCompleted];
                return [RACDisposable disposableWithBlock:^{
                    JYLog(@"Binding Signal Dispose.");
                }];
            }];
        };
    }];
    
    [bindSignal subscribeNext:^(id  _Nullable x) {
        JYLog(@"bindSignal: %@", x);
    }];
```
执行结果

```
dongxu
Original Signal Dispose.
Original Signal value = A
Binding Signal value = A & B
bindSignal: A & B
Binding Signal Dispose.
Original Signal Dispose
```
通过结果我们可以清楚bind的原理及释放的顺序。


**#flattenMap**

在RAC的使用中，flattenMap这个操作较为常见。事实上flattenMap是对bind的包装，为bind提供bindBlock。因此flattenMap与bind操作实质上是一样的(管线图可直接参考bind)，都是将原信号传出的值map成中间信号，同时马上去订阅这个中间信号，之后将中间信号的输出作为新信号的输出。不过flattenMap在bindBlock基础上加入了一些安全检查 (1)，因此推荐还是更多的使用flattenMap而非bind。

**#map**

map操作可将原信号输出的数据通过自定义的方法转换成所需的数据， 同时将变化后的数据作为新信号的输出。它实际调用了flattenMap, 只不过中间信号是直接将mapBlock处理的值返回 (1)。代码与管线图如下。此外，我们常用的filter内部也是使用了flattenMap。与map相同，它也是将filter后的结果使用中间信号进行包装并对其进行订阅，之后将中间信号的输出作为新信号的输出，以此来达到输出filter结果的目的。

**#flatten**

```
RACSubject *letters = [RACSubject subject];
RACSubject *numbers = [RACSubject subject];
RACSignal *signalOfSignals = [RACSignal createSignal:^ RACDisposable * (id<RACSubscriber> subscriber) {
    [subscriber sendNext:letters];
    [subscriber sendNext:numbers];
    [subscriber sendCompleted];
    return nil;
}];
    
RACSignal *flattened = [signalOfSignals flatten];
    
// Outputs: A 1 B C 2
[signalOfSignals subscribeNext:^(NSString *x) {
    NSLog(@"%@", x);
}];
    
[letters sendNext:@"A"];
[numbers sendNext:@"1"];
[letters sendNext:@"B"];
[letters sendNext:@"C"];
[numbers sendNext:@"2"];
```

**#merge**

merge就是把多个信号(signalA,signalB)合并成一个信号mergeSignal，只要其中任何一个信号发送数据，合并的信号mergeSignal都能接收到事件。

```
RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@"A"];
    return nil;
}];
RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@"B"];
    return nil;
}];
RACSignal *mergeSignal = [signalA merge:signalB];
[mergeSignal subscribeNext:^(id x) {
    NSLog(@"S: %@",x);
}];
```

**#concat**

concat是有序的组合，第一个信号完成之后才发送第二个信号。只有当第一个信号完成之后才能收到第二个信号的值，因为第二个信号是在第一个信号completed的闭包里面订阅的，所以第一个信号不结束，第二个信号也不会被订阅。两个信号concat在一起之后，新的信号的结束信号在第二个信号结束的时候才结束。

```
RACSignal *signal = [RACSignal createSignal:
                     ^RACDisposable *(id<RACSubscriber> subscriber)
                     {
                         [subscriber sendNext:@"A"];
                         [subscriber sendCompleted];
                         return [RACDisposable disposableWithBlock:^{
                             NSLog(@"signal dispose");
                         }];
                     }];
    
    
RACSignal *signals = [RACSignal createSignal:
                      ^RACDisposable *(id<RACSubscriber> subscriber)
                      {
                          [subscriber sendNext:@"B"];
                          [subscriber sendNext:@"C"];
                          [subscriber sendNext:@"D"];
                          [subscriber sendCompleted];
                          return [RACDisposable disposableWithBlock:^{
                              NSLog(@"signal dispose");
                          }];
                      }];
    
RACSignal *concatSignal = [signal concat:signals];
    
[concatSignal subscribeNext:^(id x) {
    NSLog(@"S: %@", x);
}];

//打印结果>>
		S: A
		S: B
		S: C
		S: D
		signal dispose
		signal dispose
```

**#zipWith**

两个信号每次发送一个，就先存储在数组中，只要有“配对”的另一个信号，就一起打包成元组RACTuple发送出去。

```
RACSignal *signal = [RACSignal createSignal:
                     ^RACDisposable *(id<RACSubscriber> subscriber)
                     {
                         [subscriber sendNext:@"A"];
                         [subscriber sendNext:@"B"];
                         [subscriber sendNext:@"C"];
                         [subscriber sendCompleted];
                         return [RACDisposable disposableWithBlock:^{
                             NSLog(@"signal1 dispose");
                         }];
                     }];
    
    
RACSignal *signals = [RACSignal createSignal:
                      ^RACDisposable *(id<RACSubscriber> subscriber)
                      {
                          [subscriber sendNext:@"1"];
                          [subscriber sendNext:@"2"];
                          [subscriber sendNext:@"3"];
                          [subscriber sendNext:@"4"];
                          [subscriber sendCompleted];
                          return [RACDisposable disposableWithBlock:^{
                              NSLog(@"signal2 dispose");
                          }];
                      }];
    
RACSignal *concatSignal = [signal zipWith:signals];
    
[concatSignal subscribeNext:^(id x) {
    NSLog(@"S: %@", x);
}];
    
//打印结果>>
		signal1 dispose
		S: <RACTuple: 0x60800001f0a0> (
		    A,
		    1
		)
		S: <RACTuple: 0x60000001f030> (
		    B,
		    2
		)
		S: <RACTuple: 0x6100002077f0> (
		    C,
		    3
		)
		signal2 dispose  
```

**#materialize / dematerialize**

materialize: 把信号包装成RACEvent类型。
dematerialize: materialize的逆向操作。它会把包装成RACEvent信号重新还原为正常的值信号。

**#delay**

订阅者延迟接收消息

**#interval:onScheduler:**

interval:onScheduler: 定时器

interval:onScheduler:withLeeway: 延迟执行定时器


**#filter / ignore / ignoreValues**

filter: 返回YES的才会通过。

ignore: 忽略给定的值。

ignoreValues: 忽略所有值，只关心Signal结束，也就是只取Comletion和Error两个消息，中间所有值都丢弃

**#distinctUntilChanged**

一个相当常用的Filter（但它不是- filter:的衍生方法），它将这一次的值与上一次做比较，相同值被忽略掉。

```
RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber)
                      {
                          NSLog(@"send A");
                          [subscriber sendNext:@"A"];
                          [subscriber sendNext:@"A"];
                          [subscriber sendNext:@"C"];
                          [subscriber sendCompleted];
                          return [RACDisposable disposableWithBlock:^{
                          }];
                      }];
    
RACSignal *signalB = [signalA distinctUntilChanged];
    
[signalB subscribeNext:^(id x) {
    NSLog(@"S: %@", x);
}];

//打印结果>>
		send A
		S: A
		S: C
```
**#take / takeLast**
take: 从开始取N次的next值。
takeLast: 

```
[[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@"1"];
    [subscriber sendNext:@"2"];
    [subscriber sendNext:@"3"];
    [subscriber sendCompleted];
    return nil;
}] take:2] subscribeNext:^(id x) {
    NSLog(@"only 1 and 2 will be print: %@", x);
}];

```






**#takeUntil / takeUntilBlock**

```
[[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@"1"];
    [subscriber sendNext:@"2"];
    [subscriber sendNext:@"3"];
    [subscriber sendNext:@"4"];
    [subscriber sendNext:@"5"];
    [subscriber sendCompleted];
    return nil;
}] takeUntilBlock:^BOOL(id x) {
    return [x isEqualToString:@"3"];
}] subscribeNext:^(id x) {
    NSLog(@"S: %@", x);
}];

//打印结果>>
		S: 1
		S: 2
```

**#skip / skipUntilBlock / skipWhileBlock**

skip: 从开始跳过N次的next值。
skipUntilBlock: 和 takeUntilBlock: 同理，一直跳，直到block为YES。
skipWhileBlock: 和 takeWhileBlock: 同理，一直跳，直到block为NO。


### 副作用

### 多线程



### 内存释放


### 场景



参考

[驱动的世界：ReactiveCocoa](http://www.jianshu.com/p/4e14ee63a12b)

[ReactiveCocoa核心方法bind(绑定)](http://blog.csdn.net/u013232867/article/details/51437998)

[美团](http://tech.meituan.com/tag/ReactiveCocoa)

[ReactiveCocoa](http://www.jianshu.com/p/4a8b38b102e8)

[ReactiveCocoa 中 RACSignal 所有变换操作底层实现分析](https://halfrost.com/reactivecocoa_racsignal_operations1/)

[应用](http://www.tuicool.com/articles/e2Q7beN)
[应用2](http://www.cocoachina.com/ios/20150817/13071.html)
[应用3](http://blog.csdn.net/xdrt81y/article/details/30624469)