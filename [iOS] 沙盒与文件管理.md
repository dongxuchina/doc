### 沙盒机制
iOS APP 可以在自己的沙盒里读写文件，但是，不可以访问其他 APP 的沙盒。每一个APP都是一个信息孤岛，相互是不可以进行通信的，唯独可以通过 URL Scheme。沙盒里面的文件可以是照片、声音文件、文本、属性列表等。

![](http://note.youdao.com/yws/public/resource/e78cdcb5242a224fa0e80a30e772a3b3/WEBRESOURCE43143b94569bc9cc84bcd8ddc3fd235f)

1. **AppName.app**: 包含APP及其所有资源。不可写。不被 iTunes 备份。
2. **Documents**：用于存储用户数据，iTunes 备份和恢复的时候会包括此目录，所以，苹果建议将程序中建立的或在程序中浏览到的文件数据保存在该目录下。
3. **Library**：包含两个子目录：Caches 和 Preferences。Caches 用来存放用户需要换成的文件。Preferences是APP的偏好设置，可以通过NSUserDefaults来读取和设置。
4. **tmp**： 用于存放临时文件，这个可以放一些当APP退出后不再需要的文件。

### 获取沙盒路径

#### 获取沙盒根目录
获取沙盒根目录，直接调用 NSHomeDirectory()：

```
//获取沙盒根目录
NSString *directory = NSHomeDirectory();
NSLog(@"directory:%@", directory);
```
控制台输出：

```
2015-07-22 00:40:16.185 iOSStrongDemo[1605:555658] directory:/var/mobile/Containers/Data/Application/F9418815-51A9-4A0A-A76C-6FD37C400928
```

#### 获取 Documents 路径

```
//获取Documents路径
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
NSString *path = [paths objectAtIndex:0];
NSLog(@"path:%@", path);
```
控制台输出：

```
2015-07-22 00:41:41.397 iOSStrongDemo[1613:556159] path:/var/mobile/Containers/Data/Application/A62B886B-A8F0-4215-B59D-1F505C3997BD/Documents
```
获取Documents文件夹目录,第一个参数是说明获取Doucments文件夹目录，第二个参数说明是在当前应用沙盒中获取。

#### 获取 Library 路径

```
//获取Library路径
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES);
NSString *path = [paths objectAtIndex:0];
NSLog(@"path：%@", path);
```

控制台输出：

```
2015-07-22 00:43:15.803 iOSStrongDemo[1619:556638] /var/mobile/Containers/Data/Application/17300507-4643-4DE7-BC68-E13DB19C8D98/Library
```
#### 获取 Caches 路径

```
//获取Caches路径
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
NSString *path = [paths objectAtIndex:0];
NSLog(@"path：%@", path);
```
控制台输出：

```
2015-07-22 00:44:31.383 iOSStrongDemo[1626:557083] path：/var/mobile/Containers/Data/Application/1E945B52-E29D-4041-A489-1AA1B11BB960/Library/Caches
```

#### 获取 tmp 路径

```
NSString *tmp = NSTemporaryDirectory();
NSLog(@"tmp：%@", tmp);
```
控制台输出：

```
2015-07-22 00:46:07.846 iOSStrongDemo[1632:557537] tmp：/private/var/mobile/Containers/Data/Application/4BE02307-1CC5-47E8-BEA8-CEBB7ED5A402/tmp/
```

### NSFileManager 文件操作
NSFileManager是一个单列类，也是一个文件管理器。可以通过NSFileManager创建文件夹、创建文件、写文件、读文件内容等等基本功能。

#### 创建文件夹

```
-(void)createDirectory{
    NSString *documentsPath =[self getDocumentsPath];
    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSString *iOSDirectory = [documentsPath stringByAppendingPathComponent:@"iOS"];
    BOOL isSuccess = [fileManager createDirectoryAtPath:iOSDirectory withIntermediateDirectories:YES attributes:nil error:nil];
    if (isSuccess) {
        NSLog(@"success");
    } else {
        NSLog(@"fail");
    }
}
```
#### 创建文件

```
-(void)createFile{
    NSString *documentsPath =[self getDocumentsPath];
    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSString *iOSPath = [documentsPath stringByAppendingPathComponent:@"iOS.txt"];
    BOOL isSuccess = [fileManager createFileAtPath:iOSPath contents:nil attributes:nil];
    if (isSuccess) {
        NSLog(@"success");
    } else {
        NSLog(@"fail");
    }
}
```
#### 写文件

```
-(void)writeFile{
    NSString *documentsPath =[self getDocumentsPath];
    NSString *iOSPath = [documentsPath stringByAppendingPathComponent:@"iOS.txt"];
    NSString *content = @"我要写数据啦";
    BOOL isSuccess = [content writeToFile:iOSPath atomically:YES encoding:NSUTF8StringEncoding error:nil];
    if (isSuccess) {
        NSLog(@"write success");
    } else {
        NSLog(@"write fail");
    }
}
```
#### 读取文件内容

```
-(void)readFileContent{
    NSString *documentsPath =[self getDocumentsPath];
    NSString *iOSPath = [documentsPath stringByAppendingPathComponent:@"iOS.txt"];
    NSString *content = [NSString stringWithContentsOfFile:iOSPath encoding:NSUTF8StringEncoding error:nil];
    NSLog(@"read success: %@",content);
}
```

#### 判断文件是否存在

```
- (BOOL)isSxistAtPath:(NSString *)filePath{
    NSFileManager *fileManager = [NSFileManager defaultManager];
    BOOL isExist = [fileManager fileExistsAtPath:filePath];
    return isExist;
}
```
#### 计算文件大小

```
- (unsigned long long)fileSizeAtPath:(NSString *)filePath{
    NSFileManager *fileManager = [NSFileManager defaultManager];
    BOOL isExist = [fileManager fileExistsAtPath:filePath];
    if (isExist){
        unsigned long long fileSize = [[fileManager attributesOfItemAtPath:filePath error:nil] fileSize];
        return fileSize;
    } else {
        NSLog(@"file is not exist");
        return 0;
    }
}
```
#### 计算整个文件夹中所有文件大小

```
- (unsigned long long)folderSizeAtPath:(NSString*)folderPath{
    NSFileManager *fileManager = [NSFileManager defaultManager];
    BOOL isExist = [fileManager fileExistsAtPath:folderPath];
    if (isExist){
        NSEnumerator *childFileEnumerator = [[fileManager subpathsAtPath:folderPath] objectEnumerator];
        unsigned long long folderSize = 0;
        NSString *fileName = @"";
        while ((fileName = [childFileEnumerator nextObject]) != nil){
            NSString* fileAbsolutePath = [folderPath stringByAppendingPathComponent:fileName];
            folderSize += [self fileSizeAtPath:fileAbsolutePath];
        }
        return folderSize / (1024.0 * 1024.0);
    } else {
        NSLog(@"file is not exist");
        return 0;
    }
}
```
#### 删除文件

```
-(void)deleteFile{
    NSString *documentsPath =[self getDocumentsPath];
    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSString *iOSPath = [documentsPath stringByAppendingPathComponent:@"iOS.txt"];
    BOOL isSuccess = [fileManager removeItemAtPath:iOSPath error:nil];
    if (isSuccess) {
        NSLog(@"delete success");
    }else{
        NSLog(@"delete fail");
    }
}
```
#### 移动文件

```
- (void)moveFileName
{
    NSString *documentsPath =[self getDocumentsPath];
    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSString *filePath = [documentsPath stringByAppendingPathComponent:@"iOS.txt"];
    NSString *moveToPath = [documentsPath stringByAppendingPathComponent:@"iOS.txt"];
    BOOL isSuccess = [fileManager moveItemAtPath:filePath toPath:moveToPath error:nil];
    if (isSuccess) {
        NSLog(@"rename success");
    }else{
        NSLog(@"rename fail");
    }
}
```

#### 重命名

```
- (void)renameFileName
{
    //通过移动该文件对文件重命名
    NSString *documentsPath =[self getDocumentsPath];
    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSString *filePath = [documentsPath stringByAppendingPathComponent:@"iOS.txt"];
    NSString *moveToPath = [documentsPath stringByAppendingPathComponent:@"rename.txt"];
    BOOL isSuccess = [fileManager moveItemAtPath:filePath toPath:moveToPath error:nil];
    if (isSuccess) {
        NSLog(@"rename success");
    }else{
        NSLog(@"rename fail");
    }
}
```














