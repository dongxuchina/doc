### 准备

逆向工程中用到的一些工具。

#### Mac 端
* pp助手 : 越狱手机。
* Theos : 越狱开发工具包。
* class-dump : 用于取得应用的头文件。
* insert_dylib : 导入动态库。
* Hopper Disassembler : Mac端上的反汇编工具。
* FileZilla : FTP客户端。

#### iOS 端
* Cycript : 用作运行时运行特定的方法。
* LLDB与debugserver : 动态调试工具。
* dumpdecrypted : 砸壳工具。
* OpenSSH : 提供ssh接入的工具。


### 开始

#### 在 iOS 设备中对可执行文件砸壳
从 App Store 中下载回来的app都是经过加密的.所以直接使用 class dump 的话并没有用.所以需要用dumpdecrypted对可执行文件砸壳。

* 通过 FileZilla 将 dumpdecrypted.dylib 文件拷贝到WeChat根目录下。
* mac 端通过 ssh 连接到 iOS 端，通过下面命令进行砸壳操作,会生成 WeChat.decrypted。
	`DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib WeChat.app/WeChat`

#### class dump 二进制文件
将 WeChat.decrypted 拷贝到mac端，并提取头文件
`class-dump --arch arm64 WeChat.decrypted -H -o WeChat-dump`

#### 定位 tweak 入口
查看微信进程 `ps aux | grep WeChat` 

使用 Cycript,可以在 app 运行的环境下调用函数。`cycript -p 1278`

`UIApp.keyWindow.recursiveDescription().toString()`

通过nextResponder可以找到入口页面。

主页面：MMTabBarController

微信：NewMainFrameViewController(MMUINavigationController)

通讯录：ContactsViewController(MMUINavigationController)

聊天页面：BaseMsgContentViewController

#### 编写和安装 tweak

通过 logify.pl 导出 tweak 文件
$THEOS/bin/logify.pl WeChat-dump/BaseMsgContentViewController.h >> Tweak.xm

创建 hook 项目 `$THEOS/bin/nic.pl`

```
Project Name (required): jyhook
Package Name [com.yourcompany.jyhook]: com.xxxx.jyhook
Author/Maintainer Name [xxx]: xxx
[iphone/tweak] MobileSubstrate Bundle filter [com.apple.springboard]: com.tencent.xin
[iphone/tweak] List of applications to terminate upon installation (space-separated, '-' for none) [SpringBoard]: WeChat
Instantiating iphone/tweak in jyhook/...
Done.
```

编辑 Makefile

```
THEOS_DEVICE_IP = localhost
THEOS_DEVICE_PORT = 2222
TARGET = iphone:latest:10.0

include $(THEOS)/makefiles/common.mk

TWEAK_NAME = jyhook
jyhook_FILES = Tweak.xm
jyhook_FRAMEWORKS = UIKit
jyhook_CFLAGS = -fobjc-arc

include $(THEOS_MAKE_PATH)/tweak.mk

after-install::
        install.exec "killall -9 WeChat"
```

将生成的 Tweak.xm 覆盖到项目里。

编译项目：`make package install` 

编译过程中会出现一些无法编译成功的错误,缺少类，缺少代理，只需要引入就可以解决。

```
@class CMessageWrap;

@protocol MessageWrapImgDelegate <NSObject>

@optional
- (void)onModMsgByBitSet:(NSString *)arg1 MsgWrap:(CMessageWrap *)arg2 BitSet:(unsigned int)arg3;
- (CMessageWrap *)onGetMsg:(NSString *)arg1 LocalID:(unsigned int)arg2 Wrap:(CMessageWrap *)arg3;
- (void)onGetBigImageErrorWithWrap:(CMessageWrap *)arg1;
- (void)onGetBigImageResultWithWrap:(CMessageWrap *)arg1 image:(UIImage *)arg2 imageData:(NSData *)arg3 isSaveImgOK:(_Bool)arg4;
- (void)onUploadImageRequestWithWrap:(CMessageWrap *)arg1;
@end

```
安装成功在 iOS 端 下/Library/MobileSubstrate/DynamicLibraries/

#### 重新签名ipa

越狱手机，需要安装 Apple File Conduit "2" (http://apt.25pp.com) 和 AppSync Unified (http://cydia.angelxwind.net)

```
codesign -f -s "iPhone Distribution: xxxxxxx" jyhook.dylib
scp -P 2222 jyhook.dylib root@localhost:/var/tmp
insert_dylib /var/tmp/jyhook.dylib WeChat.app/WeChat
//将 WeChat_patched 替换 WeChat

cp embedded.mobileprovision WeChat.app
rm -rf WeChat.app/_CodeSignature

security cms -D -i "WeChat.app/$APPLICATION/embedded.mobileprovision" > t_entitlements_full.plist
/usr/libexec/PlistBuddy -x -c 'Print:Entitlements' t_entitlements_full.plist > t_entitlements.plist

cp t_entitlements.plist WeChat.app/entitlements.plist

codesign -f -s "iPhone Distribution: xxxxxxxxx" -i com.fans-love --entitlements WeChat.app/entitlements.plist -vv WeChat.app

xcrun -sdk iphoneos PackageApplication -v WeChat.app  -o ~/WeChat.ipa

```
查看签名信息：`codesign --display --verbose=4 WeChat.app`


#### 使用 lldb 和 debugserver

[使用说明](http://www.cnblogs.com/ludashi/p/5730338.html)

iOS 端:
`/usr/bin/debugserver *:1234 -a "WeChat"`

mac 端:

```
wget http://cgit.sukimashita.com/usbmuxd.git/snapshot/usbmuxd-1.0.8.tar.bz2
tar xjfv usbmuxd-1.0.8.tar.bz2
cd usbmuxd-1.0.8/python-client/
python tcprelay.py -t 1234:1234
```
这样所有试图链接到localhost:1234的连接都会通过USB被重定向到你的iOS设备的1234端口

```
lldb
process connect connect://localhost:1234
```

查看基地址: `image list -o -f`

```
hopper: 0000000100000000
偏移: 64000
lldb: 0000000100034000
```

通过 Hopper Disassembler 找到地址并计算 lldb 地址.

`br s -a 0x101e905fc`


### 相关资料

[移动App入侵与逆向破解技术－iOS篇](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653577384&idx=1&sn=b44a9c9651bf09c5bea7e0337031c53c&scene=0&key=459b7f9000fef90d400a075b3512cdbaaf245aac51a9527baae881ae14683122d562d0701f9f745bd0e2bea348e8d33bb0e1d824bfef8238dd39ff87527b754f10f61a94b0ef44e1e6262d373d197a7d&ascene=0&uin=NTQ4ODA1NQ%3D%3D&devicetype=iMac+MacBookPro11%2C4+OSX+OSX+10.11.6+build(15G31)&version=12000510&nettype=WIFI&fontScale=100&pass_ticket=vmze77IpUMciMNU%2Bt0Ib%2FYV6swhFvdnyTz%2B6mI%2B%2BqzE%3D)

[iOS反编译-hook微信之艾特所有人](http://www.jianshu.com/p/50e6b8c24430?nomobile=yes)

[一步一步实现iOS微信自动抢红包(非越狱)](http://www.90159.com/2016/03/28/step-by-step-WeChat/)

[iOS逆向工程之Hopper+LLDB调试第三方App](http://www.cnblogs.com/ludashi/p/5730338.html)












