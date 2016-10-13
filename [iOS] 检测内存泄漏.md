### 内存泄漏的检测

#### 静态检测
使用 XCode 分析功能，Product->Analyze

* 可能存在的内存泄漏监测（Memory）

	 `Potential leak of an object`
* 无效数据监测（Dead store）

	 `Unused`、`Never read....`
* 逻辑错误监测（Logic error）

#### 动态检测
使用 XCode Instruments 功能，Product->Profile

* Separate by Thread：每个线程被单独考虑。这能让你知道哪一个线程占用 CPU 最多。
Invert Call Tree：选中该选项后，调用栈会自上至下显示。这通常是你需要的，因为你想知道 CPU 花费时间的那个最深的方法。
* Hide System Libraries：选中该选项后，只有你自己 app 中出现的符号会被显示出来。通常选中该选项是有用的，因为你只关心CPU在你自己的代码中的哪一部分花费时间，你没法对系统库使用 CPU 做多少改变。
* Flatten Recursion：该选项将每一个调用栈中的递归函数（调用它们自身的函数）视作单一入口，而不是多入口。
* Top Functions：选上这一选项让 Instruments 将花费在一个函数中的总时间视作在该函数中直接花费的时间加上调用的其他函数花费的时间。所以如果函数 A 调用了函数 B，那么函数A花费的总时间被记为 A 花费的时间加上 B 花费的时间。这一选项非常有用，因为它能让你在每次进入调用栈时找到花费最长的时间，瞄准你最耗时的方法。

##### 1、Timer Profilter
![](http://note.youdao.com/yws/public/resource/bb93f1a1fb98c7fb1ec45283db978a11/6243AA3A085A40D995B6F2533744676E)
查看秒数最大的记录，超过100ms都需要特殊注意；双击可以进入相关代码中，分析后解决问题；

##### 2、Allocations
* Persistent Bytes：净分配字节数，当前已经分配内存但是仍然没有被释放的字节的总  数。
* \#Persistent：净分配数，当前已经分配内存但仍然没有被释放的对象或内存块的数量。
* \#Transient：临时分配数，当前已经分配内存但仍然没有被释放的对象或内存块的数量。
* Total Bytes：总分配字节数，所有已经分配内存,而且包括已经被释放了的字节的总数。
* \#Total：总分配数，所有当前已经分配内存,包括已经被释放了的对象或内存块的总数。

![](http://note.youdao.com/yws/public/resource/bb93f1a1fb98c7fb1ec45283db978a11/E2518414D07B46E39A802E91506C61F8)

主要观察 Persistent Bytes 的增长情况。

![](http://note.youdao.com/yws/public/resource/bb93f1a1fb98c7fb1ec45283db978a11/7E13862E3C904038A99BC80BA54B725E)

还可以通过 Mark Generation 来分析增量情况。

##### 3、Leaks

![](http://note.youdao.com/yws/public/resource/bb93f1a1fb98c7fb1ec45283db978a11/446BBD07609242B3A410A915B9E9C980)

找到内存泄漏的记录，双击可以进入相关代码中，分析后解决问题；

### 内存泄漏的常见问题
#### CF 类型内存
注意以 `create`,`copy` 作为关键字的函数都是需要释放内存的，注意配对使用。比如：`CGColorCreate<-->CGColorRelease`
#### MRC 内存使用
也需要注意配对使用，需要说明的是，如果代码中有部分文件是 MRC 的，在已有文件中加代码的时候注意一下，不能都按照 ARC 的方式处理；
#### ARC 内存使用
不必再显示的调用 `retain`，`release`，但是还是要注意循环引用的问题。
问题经常出现在 `NSTimer `、`NSNotification`、`block` 等。

### 参考
[Find Memory Leaks](https://developer.apple.com/library/prerelease/content/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/FindingLeakedMemory.html#//apple_ref/doc/uid/TP40004652-CH81-SW1)

[iOS 究极体测试工具 内存泄漏](http://www.jianshu.com/p/cdae09aa4f8d?utm_campaign=hugo&utm_medium=reader_share&utm_content=note&utm_source=qq)