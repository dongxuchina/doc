## 获取 Crash 文件
我们可以通过应用程序crash的时候上传crash信息（包含第三方统计），还可以通过用户反馈拿到crash文件。
## Crash 文件的内容
```
Last Exception Backtrace:
0   CoreFoundation                	0x18177d900 __exceptionPreprocess + 124
1   libobjc.A.dylib               	0x180debf80 objc_exception_throw + 56
2   CoreFoundation                	0x18178461c -[NSObject(NSObject) doesNotRecognizeSelector:] + 212
3   CoreFoundation                	0x1817815b8 ___forwarding___ + 872
4   CoreFoundation                	0x18168568c _CF_forwarding_prep_0 + 92
5   libobjc.A.dylib               	0x180dfafc8 objc_setProperty_nonatomic_copy + 48
6   appName                	        0x1007beca8 0x1000dc000 + 7220392
7   appName                	        0x100795cd0 0x1000dc000 + 7052496
...
18  libsystem_pthread.dylib       	0x1813e5470 _pthread_wqthread + 1092
19  libsystem_pthread.dylib       	0x1813e5020 start_wqthread + 4

Binary Images:
0x1000dc000 - 0x100a23fff Discuz_phpwind arm64  <63122614085f3684963db072fa3739f2> /var/mobile/Containers/Bundle/Application/67C617A8-B86A-406B-B3EB-8FBD01FBD5C9/Discuz_phpwind.app/Discuz_phpwind
0x120004000 - 0x120033fff dyld arm64  <9e98992ceed735e2ac4784cb28efe7c1> /usr/lib/dyld
0x180d6c000 - 0x180d6dfff libSystem.B.dylib arm64  <c4cd04b37e5f34698856a9384aefff40> /usr/lib/libSystem.B.dylib
0x180d70000 - 0x180dc3fff libc++.1.dylib arm64  <d430d0ad16893b76bbc52468f65d5906> /usr/lib/libc++.1.dylib
0x180dc4000 - 0x180de3fff libc++abi.dylib arm64  <1c0a8ef87e8c37b2a577dc1a44e2b16e> /usr/lib/libc++abi.dylib
0x180de4000 - 0x181150fff libobjc.A.dylib arm64  <da8e482b3e7d3c40a798a0c86a3d6890> /usr/lib/libobjc.A.dylib
0x181154000 - 0x181158fff libcache.dylib arm64  <242f50f854a1301fa6f76b4531101238> /usr/lib/system/libcache.dylib
...
0x181210000 - 0x181211fff libremovefile.dylib arm64  <2fb2b791a3453c019640b22cee6a0c00> /usr/lib/system/libremovefile.dylib

```
我们可以看到错误的位置
```
CoreFoundation                	0x18178461c -[NSObject(NSObject) doesNotRecognizeSelector:] + 212
```
但是程序出现的crash:
```
6   appName                	        0x1007beca8 0x1000dc000 + 7220392
7   appName                	        0x100795cd0 0x1000dc000 + 7052496
```
没有指出具体问题的位置。

##如何符号化 crash 文件（Symbolicating crash logs）
###方法1 使用命令工具 symbolicatecrash
在处理之前，请依然将“.app“, “.dSYM”和 ".crash"文件放到同一个目录下。现在打开终端(Terminal)然后输入如下的命令：
>export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer

然后输入命令：
>/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/PrivateFrameworks/DTDeviceKitBase.framework/Versions/A/Resources/symbolicatecrash appName.crash appName.app > appName.log

现在，符号化的 crash log就保存在 appName.log 中了。

###方法2 使用命令工具 atos
首先要保证 ipa 和 crash 文件的 UUID 一致，再通过ipa对应的DSYMB文件进行符号化。
####怎样获取crash文件的 UUID
```
Binary Images:
0x1000dc000 - 0x100a23fff Discuz_phpwind arm64  <ddd126556cf73be98385f48eaf80e937>
```
从 crash 文件中我们可以找到 Binary Images 这一行，`ddd126556cf73be98385f48eaf80e937`为UUID,同时我们还可以知道是 arm64 的机型；

还可以通过第三方的错误统计中获取 UUID;

或者通过命令获取：
> grep "appName armv" *crash

或
> grep --after-context=2 "Binary Images:" *crash

####怎样获取 ipa 文件的 UUID
将ipa改成zip解压，找到.app文件

可以使用命令：`xcrun dwarfdump -–uuid <AppName.app/ExecutableName>`

如：
>xcrun dwarfdump --uuid appName.app/appName

结果：
>UUID: 0C3BE413-16DC-36B9-ACE2-943CAA01E739 (armv7) appName.app/appName
UUID: DDD12655-6CF7-3BE9-8385-F48EAF80E937 (arm64) appName.app/appName

这个 app 有2个 UUID，表明它是一个 fat binnary。

它能利用最新硬件的特性，又能兼容老版本的设备。

我们发现 crash 文件和 ipa 文件的 UUID 相同
>DDD12655-6CF7-3BE9-8385-F48EAF80E937

####使用 atos 命令来符号化代码地址
可以使用命令：

`atos [-o AppName.app/AppName] [-l loadAddress] [-arch architecture]`

```
6   appName                	        0x1007beca8 0x1000dc000 + 7220392
7   appName                	        0x100795cd0 0x1000dc000 + 7052496
```
前面出现的 0x1000dc000 为 loadAddress

命令如下：
>xcrun atos -o appName.app.dSYM/Contents/Resources/DWARF/appName -l 0x1000dc000 -arch arm64

然后找到 0x1000dc000 前面的地址输入
>0x1007beca8

回车
> -［MarkupParser attrStringFromMarkup:］ (in appName) (MarkupParser.m:64)

这样就找到了 crash 的具体位置了。

参考：

[命令行工具解析Crash文件,dSYM文件进行符号化](http://www.jianshu.com/p/0b6f5148dab8)

[分析iOS Crash文件：符号化iOS Crash文件的3种方法](http://www.cocoachina.com/industry/20140514/8418.html)
