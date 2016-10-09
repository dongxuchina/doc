### Block 基本用法
#### 原型及定义

![aaa](http://note.youdao.com/yws/public/resource/bb93f1a1fb98c7fb1ec45283db978a11/CD3BD1F4FE784BFFAFEC6A26260B32B6)

```
//runBlock无返回值，有一个字符串的参数
void (^runBlock)(NSString *x);
    
//jumpBlock需要字符串的返回值，并且有一个整形的参数
NSString * (^jumpBlock)(int *x);
    
```

#### 局部变量和全局变量的使用

全局变量可以被修改

```
int global = 1000;  
void method()  
{  
	void(^block)(void) = ^(void)  
   {  
   		global++;  
       NSLog(@"global:%d", global);  
   };  
   block();  
   NSLog(@"global:%d", global);   
}  
```
打印结果：

```
 global:1001    
 global:1001
```

局部变量需要 __block 修饰才可以被修改

```
 
void method()  
{  
	int local = 500; 
	void(^block)(void) = ^(void)  
   {  
   		local ++;  
       NSLog(@"local:%d", local);  
   };  
   block();  
   NSLog(@"local:%d", local);   
}  
```
打印结果：

```
 local:501
 local:501

```

### Block 类型及内存管理

#### 类型
block 属于对象，主要有三种类型：

* _NSConcreteStackBlock，存储在栈上；
* _NSConcreteGlobalBlock，存储在程序的数据区域(text段)；
* _NSConcreteMallocBlock，存储在堆上；

在 ARC 模式下使用时需要注意以下几种情况：

NSGlobalBlock：block 可以修饰成任何类型 (strong/copy/weak)，并且 block 内部没有引用任何外部变量;

```
void (^globalBlock) () = ^ () {
	NSLog(@"global block");
};
NSLog(@"%@", globalBlock);//<__NSGlobalBlock__: 0x1096e20c0>
```

NSStackBlock：block 修饰成 weak，并且 block 内部引用了外部变量；

```
int base = 100;
__weak long (^stackBlock) (int, int) = ^ long (int a, int b) {
	return base +a + b;
};
NSLog(@"%@",stackBlock);//<__NSStackBlock__: 0x7fff57c6bce0>
```

NSMallocBlock：block 修饰成 strong(copy)，并且 block 内部引用了外部变量；

```
int base = 100;
long (^stackBlock) (int, int) = ^ long (int a, int b) {
	return base +a + b;
};
NSLog(@"%@",stackBlock);//<__NSMallocBlock__: 0x7f8da961e590>
```

#### 内存管理
对象之间调用，需要注意 block 是否是 NSMallocBlock 类型；

```
@property (nonatomic, strong) void (^strongBlock)(NSString *metadata);
@property (nonatomic, copy) void (^strongBlock)(NSString *metadata);

```
全局修饰成strong／copy 都需要注意方法内部的 self 的循环引用问题；

```
//弱引用
__weak typeof(self) weakSelf = self;
^ {
    weakSelf.name
};
```
