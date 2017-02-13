
#### Xcode 升级后插件失效
[Xcode升级后插件失效的原理与修复办法](http://joeshang.github.io/2015/04/10/fix-xcode-upgrade-plugin-invalid/)

[解决Xcode8插件失效的方法](https://github.com/LFL2018/XcodePluginUpgrade-LFL)

#### iOS 定位坐标在百度地图上偏移过大
[iOS 火星坐标相关整理及解决方案汇总](http://blog.csdn.net/jiajiayouba/article/details/25140967)


#### macOS 10.12+ 脚本打包出现权限错误
```
SecKey API returned: -67671, (null)/opt/iOSPack/work/anmi/app250732_10004/build/Applications/Discuz_phpwind.app: unknown error -1=ffffffffffffffff
Command /usr/bin/codesign failed with exit code 1
```

[[RESOLVED] CODESIGN ERROR OVER SSH ON MACOS 10.12+](https://sinofool.net/blog/archives/322)

[security / codesign in Sierra: Keychain ignores access control settings and UI-prompts for permission](http://stackoverflow.com/questions/39868578/security-codesign-in-sierra-keychain-ignores-access-control-settings-and-ui-p)


#### textViewDidBeginEditing:(UITextView *)textView 光标位置
此方法每次调用时 textView 返回的是上一次光标的位置，并不是当前位置的，应该是调用周期比较靠前，此时新光标还没有定位；解决办法如下：

```
//延迟一点时间调用 textViewDidChange，此时光标的位置是准确的；
- (void)textViewDidBeginEditing:(UITextView *)textView {
    [self performSelector:@selector(textViewDidChange:) withObject:textView afterDelay:0.1f];
}

- (void)textViewDidChange:(UITextView *)textView {
    textView.selectedTextRange//通过 textViewDidChange 获取光标位置
}


```

