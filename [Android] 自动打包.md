### Ant 编译
[Ant自动编译打包&发布 android项目](http://www.cnblogs.com/yaozhongxiao/p/3523061.html)

[Android SDK开发包国内下载地址](http://www.cnblogs.com/bjzhanghao/archive/2012/11/14/android-platform-sdk-download-mirror.html)


### Gradle 编译
[Gradle 完整指南（Android）](https://gold.xitu.io/entry/57c7a00e0a2b58006b1a1358/promote?utm_source=baidu&utm_medium=keyword&utm_content=android_gradle&utm_campaign=q3_search)

[Gradle 下载地址](http://services.gradle.org/distributions/)


### 编译环境常见的问题总结

在 Linux 上搭建编译环境时可能会遇到以下几个问题

#### 问题1

```
finished with non-zero exit value n
```
解决办法参考：[android studio gradle 编译异常小结](http://www.jacpy.com/2016/04/22/android-studio-error-collection.html)


#### 问题2
```
com.android.ide.common.process.ProcessException: 
org.gradle.process.internal.ExecException: 
Process 'command 'C:\Users\Vishnu Ruhela\AppData\Local\Android\sdk\build-tools
\23.0.2\aapt.exe'' finished with non- zero exit value 1
```
Gradle 编译时出现上面错误，并且没有任何提示，

此时我们使用 Ant 脚本编译可以发现错误，如下：

```
[aapt] /opt/android_build/android-sdk-linux-simple/build-tools/23.0.2/aapt: /lib/libc.so.6: version `GLIBC_2.11' not found (required by /opt/android_build/android-sdk-linux-simple/build-tools/23.0.2/aapt)
[aapt] /opt/android_build/android-sdk-linux-simple/build-tools/23.0.2/aapt: /lib/libz.so.1: no version information available (required by /opt/android_build/android-sdk-linux-simple/build-tools/23.0.2/aapt)
```

执行`strings /lib64/libc.so.6 |grep GLIBC_` 查看 libc 目前支持的版本 list，如下：

```
[root@node1 values]# strings /lib64/libc.so.6 |grep GLIBC_
GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_PRIVATE
```
说明现有 Linux 版本不支持 GLIBC_2.11,那么可以尝试做更新,如下：

```
[root@node1 values]# yum update /lib64/libc.so.6
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.yun-idc.com
 * extras: mirrors.yun-idc.com
 * updates: mirrors.yun-idc.com
Setting up Update Process
No Packages marked for Update
[root@node1 values]#
```
说明没有可更新的版本，那么不要去尝试手动下载安装最新版本，必须升级系统才可解决这个问题。


#### 问题3
```
Execute failed: java.io.IOException: Cannot run program "sdk-linux/build-tools/22.0.0/aapt": error=2
```
此错误的主要原因是在 64 位 Linux 下打包成APK时缺少 x86 下 C++ 语言库。

解决办法：

[Cannot run program "sdk-linux/build-tools/22.0.0/aapt": error=2](http://blog.csdn.net/catoop/article/details/47157631)

[Centos 64位安装aapt、jdk、tomcat的详细教程](http://www.jb51.net/article/96942.htm)


#### 问题4
```
ERROR: ld.so: object '/lib/libnano.so.4' from /etc/ld.so.preload cannot be preloaded: ignored.]
```
此错误为 64 位系统没有这个文件所导致。

解决办法：# echo "" > /etc/ld.so.preload






