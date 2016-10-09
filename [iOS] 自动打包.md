### xcodebuild 简介
* 可以通过man xcodebuild 或 xcodebuild -h 查看如何使用命令

	请参阅 [xcodebuild](https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/xcodebuild.1.html)

* 设置 xcodebuild 环境变量
	`export PATH="$PATH:/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild"`

	`xcodebuild -showsdks` 查看sdk版本；
	
	`xcodebuild -list` 查看 project 中的 targets 、configurations 、workspace、 schemes；
	
	`xcodebuild -version` 查看 xcode 版本；

	`xcodebuild -showBuildSettings` 查看 build setting 的信息；
	
* -project
	
	如果目录中存在多个 projects，可以使用 -project  来指定需要 build 的项目；
	
* -target

	在不指定 build 的 target 的时候，默认情况下会 build project 下的第一个 target；
	
* -workspace -scheme
	
	当 build workspace 时，需要同时指定 -workspace 和 -scheme 参数，scheme 参数控制了哪些 targets 会被 build 以及以怎样的方式 build;

### 使用xcodebuild和xcrun打包签名

* 基本用法

	打开终端，进入项目根目录下，运行命令：`xcodebuild -project Discuz_phpwind.xcodeproj -target Discuz_phpwind -configuration Release` 如果看到 `** BUILD SUCCEEDED **` 说明编译成功；

	编译成功后，根目录下创建出一个 build 目录，该目录下有 Release-iphoneos 和 xxxx.build 文件，根据我们 build -configuration 配置的参数不同，Release-iphoneos 的文件名会不同；

	在 Release-iphoneos 文件夹下，有我们需要的xxxx.app文件，但是要安装到真机上，我们需要使用xcrun 命令导出为ipa文件 `xcrun -sdk iphoneos -v PackageApplication ./build/Release-iphoneos/Discuz_phpwind.app -o ~/Desktop/Discuz_phpwind.ipa`

	查看帮助信息 `xcrun -sdk iphoneos -v PackageApplication -help`;

* workspace举例
	
	将打包证书导入到钥匙串中 `security import "/Users/dongxu/Downloads/mobcent.p12" -k login.keychain -P "1" -T /usr/bin/codesign` 也可以专为打包创建keychain
	
	```
	//创建钥匙链；
	security create-keychain -p pass packer.keychain
	//搜索钥匙链切换；
	security list-keychains -s packer.keychain
	//解锁钥匙链；
	security unlock-keychain -p pass packer.keychain
	```
	
	执行描述文件 xxxxx.mobileprovision，进入 `~/Library/MobileDevice/Provisioning Profiles` 可以查看描述文件的UUID；
	
	为了防止编译出现冲突，需要执行 `xcodebuild clean`
	
	可以指定bundleId和profile等信息进行编译：`xcodebuild PRODUCT_BUNDLE_IDENTIFIER=com.xxxxx PROVISIONING_PROFILE=23fd5136-b4fc-41b6-a817-2c461d5b91a5 -workspace  "xxxx.xcworkspace" -scheme  "xxxx" -derivedDataPath build -sdk iphoneos archive DSTROOT=build`
	
	编译成功后再通过xcrun命令导出ipa文件。
	
###打包过程脚本化

请参阅：[IOS工程自动打包并发布脚本实现](https://my.oschina.net/u/727843/blog/391946)
	
	
	

