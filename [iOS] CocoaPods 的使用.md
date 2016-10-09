### 为什么要使用管理工具
* 不需要手动导入三方库到项目中; 
* 不需手动设置配置文件,例如库中依赖的框架,动态库; 
* 不需手动设置编译参数;
* 更新项目中的库时,比较麻烦; 
* 当使用cocoapods管理工具时,就可以弥补以上不足,比较方便

### CocoaPods 的安装
CocoaPods是ruby写的,如果当前ruby版本不支持,需要升级ruby版本,在终端输入 

`sudo gem update --system`

将ruby源换成国内淘宝的,终端输入

`gem sources --remove http://rubygems.org/`
`gem sources -a https://ruby.taobao.org/`

查看是否更换成功

`gem sources -l`

安装cocoapods

`sudo gem install cocoa pods`

将`https://github.com/CocoaPods/Specs`上的`podspec`的索引文件下载到本地`~/.cocoapods`目录下

`pod setup`

进入到.cocoapods目录下查看下载的进度

`du -sh * `

### CocoaPods 的使用

安装好cocoapods后,在终端进入到工程目录,创建Podfile文件

`pod init`

编辑Podfile文件，添加需要的第三方库 `vi Podfile`

```
# Uncomment this line to define a global platform for your project
# platform :ios, '9.0'

target 'app-test' do
  # Uncomment this line if you're using Swift or would like to use dynamic frameworks
  # use_frameworks!

  pod 'ASIHTTPRequest', '~> 1.8.2'
  
  # Pods for app-test

  target 'app-testTests' do
    inherit! :search_paths
    # Pods for testing
  end

  target 'app-testUITests' do
    inherit! :search_paths
    # Pods for testing
  end

end
```
下载并安装Podfile所引入的库`pod install --verbose --no-repo-update`

```
  Preparing

Analyzing dependencies

Inspecting targets to integrate
  Using `ARCHS` setting to build architectures of target
  `Pods-app-test`: (``)
  Using `ARCHS` setting to build architectures of target
  `Pods-app-testTests`: (``)
  Using `ARCHS` setting to build architectures of target
  `Pods-app-testUITests`: (``)

Resolving dependencies of `Podfile`

Comparing resolved specification to the sandbox manifest

Downloading dependencies
  - Running pre install hooks

...

Sending stats
  Pod installation complete! There are 0 dependencies from the
  Podfile and 0 total pods installed.

[!] The Podfile does not contain any dependencies.
```
执行完毕，工程目录下会增加几个相关文件

```
dongxudeMacBook-Pro:app-test dongxu$ ls
Podfile			app-test.xcodeproj
Podfile.lock		app-test.xcworkspace
Pods			app-testTests
app-test		app-testUITests
dongxudeMacBook-Pro:app-test dongxu$
```

使用 CocoaPods 生成的 .xcworkspace 文件来打开工程，而不是以前的 .xcodeproj 文件。

每次更改了 Podfile 文件，你需要重新执行一次pod update命令

### 查找第三方库
```
-> AFNetworking (3.1.0)
   A delightful iOS and OS X networking framework.
   pod 'AFNetworking', '~> 3.1.0'
   - Homepage: https://github.com/AFNetworking/AFNetworking
   - Source:   https://github.com/AFNetworking/AFNetworking.git
   - Versions: 3.1.0, 3.0.4, 3.0.3, 3.0.2, 3.0.1, 3.0.0,
   3.0.0-beta.3, 3.0.0-beta.2, 3.0.0-beta.1, 2.6.3, 2.6.2, 2.6.1,
   2.6.0, 2.5.4, 2.5.3, 2.5.2, 2.5.1, 2.5.0, 2.4.1, 2.4.0, 2.3.1,
   2.3.0, 2.2.4, 2.2.3, 2.2.2, 2.2.1, 2.2.0, 2.1.0, 2.0.3, 2.0.2,
   2.0.1, 2.0.0, 2.0.0-RC3, 2.0.0-RC2, 2.0.0-RC1, 1.3.4, 1.3.3,
   1.3.2, 1.3.1, 1.3.0, 1.2.1, 1.2.0, 1.1.0, 1.0.1, 1.0, 1.0RC3,
   1.0RC2, 1.0RC1, 0.10.1, 0.10.0, 0.9.2, 0.9.1, 0.9.0, 0.7.0,
   0.5.1 [master repo]
   - Subspecs:
     - AFNetworking/Serialization (3.1.0)
     - AFNetworking/Security (3.1.0)
     - AFNetworking/Reachability (3.1.0)
     - AFNetworking/NSURLSession (3.1.0)
     - AFNetworking/UIKit (3.1.0)

-> AFNetworking+AutoRetry (0.0.5)
   Auto Retries for AFNetworking requests
   pod 'AFNetworking+AutoRetry', '~> 0.0.5'
   - Homepage: https://github.com/shaioz/AFNetworking-AutoRetry
   - Source:   https://github.com/shaioz/AFNetworking-AutoRetry.git
   - Versions: 0.0.5, 0.0.4, 0.0.3, 0.0.2, 0.0.1 [master repo]
...
```
### 关于 Podfile
* 当你执行`pod install`之后，除了 Podfile 外，CocoaPods 还会生成一个名为`Podfile.lock`的文件，`Podfile.lock` 应该加入到版本控制里面，不应该把这个文件加入到`.gitignore`中。因为`Podfile.lock`会锁定当前各依赖库的版本，之后如果多次执行`pod install` 不会更改版本，要`pod update`才会改`Podfile.lock`了。这样多人协作的时候，可以防止第三方库升级时造成大家各自的第三方库版本不一致。

	```
	PODS:
	  - module-live (1.0.0.beta)
	  - module-support (1.0.0)
	
	DEPENDENCIES:
	  - module-live (~> 1.0.0)
	  - module-support (~> 1.0.0)
	
	SPEC CHECKSUMS:
	  module-live: 331365117ab518405a91570d0fb1df2f231818f7
	  module-support: 1b007a3f9e2310b33a46043ed8c9679102132ae9
	
	PODFILE CHECKSUM: ba1a5a32f150afb3913039f9a10adf696fff9744
	
	COCOAPODS: 1.0.1
	```
* 在编辑Podfile文件时,如果不指定版本默认为指向最新版;
* 引入本地库,pod 'AFNetworking', :path => '~/Documents/AFNetworking'
* [Podfile详情点击可查看](https://guides.cocoapods.org/using/the-podfile.html)

### 创建 podspec 文件

我们可以为自己的开源项目创建`podspec`文件，首先通过如下命令初始化一个`podspec`文件：

```
pod spec create your_pod_spec_name
```
该命令执行之后，CocoaPods 会生成一个名为`your_pod_spec_name.podspec`的文件，然后我们修改其中的相关内容即可。[如何编写一个 CocoaPods 的 spec 文件](http://ishalou.com/blog/2012/10/16/how-to-create-a-cocoapods-spec-file/)

### 使用私有库
需要维护两个仓库，分别是代码仓库和spec仓库。

* 代码仓库：通过git来管理和发布我们的代码；
* spec仓库：管理所有模块的spec文件；

存放spec仓库的文件规范

```
├── Specs  
    └── [SPEC_NAME]  
        └── [VERSION]  
            └── [SPEC_NAME].podspec
```
例如

```
├── Specs  
    └── module-live  
        └── 1.0.1  
            └── module-live.podspec
    └── module-support  
        └── 1.0.1  
            └── module-support.podspec
```

需要将spec仓库的地址，导入到Podfile文件中才能使用

```
source 'https://github.com/dongxuchina/Specs.git'  
source 'https://github.com/CocoaPods/Specs.git'

target 'app-test' do

  pod 'module-live', '~> 1.0.1'
  pod 'module-support', '~> 1.0.1'

end
```
### 不更新 podspec

CocoaPods 在执行pod install和pod update时，会默认先更新一次podspec索引。使用--no-repo-update参数可以禁止其做索引更新操作

```
pod install --no-repo-update
pod update --no-repo-update
```
### 强制更新相同版本的库文件
podspec 文件在版本不升级的情况下想要更新成最新的代码，需要清理下CocoaPods的缓存文件，位置在`~/Library/Caches/CocoaPods`,删除掉对应库的所有版本，重新`pod update`

```
├── Pods
	└── Release  
    	└── module-live  
        	└── 1.0.1-5ecef  
            	└── module-live
    		└── 1.0.1-e94a1  
            	└── module-live
```

###问题汇总
* 使用 2FA authentication code 导致无法安装私有库的解决办法

	```
	dongxudeMacBook-Pro:ios-app-discuz dongxu$ pod install
	Cloning spec repo `yunpro-dongxu-specs` from `https://git.yunpro.cn/dongxu/Specs.git`
	Username for 'https://git.yunpro.cn': dongxu
	Password for 'https://dongxu@git.yunpro.cn':
	[!] Unable to add a source with url `https://git.yunpro.cn/dongxu/Specs.git` named `yunpro-dongxu-specs`.
	You can try adding it manually in `~/.cocoapods/repos` or via `pod repo add`.
	
	```
	按照提示我们使用`pod repo add`
	
	```
	dongxudeMacBook-Pro:ios-app-discuz dongxu$ pod repo add yunpro-dongxu-specs https://git.yunpro.cn/dongxu/Specs.git
	Cloning spec repo `yunpro-dongxu-specs` from `https://git.yunpro.cn/dongxu/Specs.git`
	Username for 'https://git.yunpro.cn': dongxu
	Password for 'https://dongxu@git.yunpro.cn':
	[!] /usr/bin/git clone https://git.yunpro.cn/dongxu/Specs.git yunpro-dongxu-specs
	
	Cloning into 'yunpro-dongxu-specs'...
	remote: HTTP Basic: Access denied
	remote: You have 2FA enabled, please use a personal access token for Git over HTTP.
	remote: You can generate one at https://git.yunpro.cn/profile/personal_access_tokens
	fatal: Authentication failed for 'https://git.yunpro.cn/dongxu/Specs.git/'
	```
	
	我们发现是 2FA 的问题 `Password for 'https://dongxu@git.yunpro.cn':` 我们使用 personal access token 作为密码就可以验证通过了；





参考文章：

[用CocoaPods做iOS程序的依赖管理](http://blog.devtang.com/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)

[Podfile.lock背后的那点事](http://blog.startry.com/2015/10/28/Somthing-about-Podfile-lock/)

[The Podfile](https://guides.cocoapods.org/using/the-podfile.html)