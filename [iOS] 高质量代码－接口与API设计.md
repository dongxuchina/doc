#### 使用前缀避免命名冲突
* Apple 保留使用 two-letter prefix 的权利，所以前缀应该使用三个大写字母比较好，个人建议前两个字母可以是项目或是公司相关的首字母，第三个字母可以是模块的首字母。
* 类名及方法名都应加前缀，分类及分类中的方法也需加前缀。
* 纯c的方法及全局变量需要加前缀，不要使用简单的方法名。如：share() >> DZTDetailShare()
* 如果自己开发的程序库引入了第三方库，应该其中加入前缀。

#### 全能初始化方法
* 在类中提供一个全能初始化方法，其它初始化方法均调用此方法。
* 若全能初始化方法与父类不同，则需覆盖父类中的对应方法。
* 如果父类的初始化方法不适用子类，那么应该覆写这个父类方法，并在其中抛出异常。

	父类：

	```
	@implementation DZTRectangle
	
	//全能初始化方法
	-(id)initWithWidth:(float)width andHeight:(float)height{
	    if (self = [super init]) {
	        self.width = width;
	        self.height = height;
	    }
	    return self;
	}
	
	//使用默认值来覆写init
	-(instancetype)init{
	    return [self initWithWidth:10.0 andHeight:10.0];
	}
	
	//如果不建议使用init方法，可以抛出异常
	- (instancetype)init{
	    @throw [NSException exceptionWithName:NSInternalInconsistencyException 
	    		reason:@"Must use initWithWidth:andHeight: instead." userInfo:nil];
	}
	
	@end

	```

	子类：
	
	```
	@implementation DZTSquare
	
	//正方形等边，所以需要自己的初始化方法
	-(id)initWithDimension:(float)dimension{
	    return [super initWithWidth:dimension andHeight:dimension];
	}
	
	//在不覆写的情况下，子类是可以使用这个初始化方法的，有不等边的风险
	-(id)initWithWidth:(float)width andHeight:(float)height{
	    //用最大边做为正方形的编程
	    float dimesion = MAX(width, height);
	    return [self initWithDimension:dimesion];
	}
	
	//如果子类不想使用父类的方法，也可以抛出异常
	- (id)initWithWidth:(float)width andHeight:(float)height{
	      @throw [NSException exceptionWithName:NSInternalInconsistencyException 
	      		reason:@"Must use initWithDimension: instead." userInfo:nil];
	}
	
	@end
	```

#### 实现 description 方法

* 实现 description 方法返回一个有意义的字符串，用以描述该实例。
* 若干项在调试时打印对象的详细的描述信息，则应事先 debugDescription 方法。

```
//打印对象会调用 description 方法
- (NSString *)description{
	//使用字典来封装数据，可以是打印更简洁
    NSMutableDictionary *dic = [NSMutableDictionary dictionary];
    [dic setValue:[NSNumber numberWithFloat:_width] forKey:@"width"];
    [dic setValue:[NSNumber numberWithFloat:_height] forKey:@"height"];
    return [NSString stringWithFormat:@"%@",dic];
}

//LLDB 调试代码, 打印对象会调用 debugDescription 方法，可以把详细的信息写在这里
- (NSString *)debugDescription{
    return [NSString stringWithFormat:@"<%@: %p, %@>", [self class], self, 
    	[self description]];
}
```

#### 尽量使用不可变对象

* 尽量定义不可变的对象给外界使用（readonly）。
* 若某属性需要对象内部修改，则需要对 readonly 属性扩展为 readwrite 属性。
* 不要把可变的 collection 做为属性公开，可以 copy 一份不可变的供外界使用，外界想修改 collection 需要通过内部的公开方法来进行修改。

```
//  DZTRectangle.h

#import <Foundation/Foundation.h>

@interface DZTRectangle : NSObject

@property (nonatomic, assign, readonly) float width;
@property (nonatomic, assign) float height;

-(id)initWithWidth:(float)width andHeight:(float)height;

@end



//  DZTRectangle.m

#import "DZTRectangle.h"

@interface DZTRectangle()

//扩展为读写，内部可以进行修改
@property (nonatomic, assign, readwrite) float width;

@end

@implementation DZTRectangle

-(id)initWithWidth:(float)width andHeight:(float)height{
    if (self = [super init]) {
        self.width = width;
        self.height = height;
    }
    return self;
}

@end

```

虽然对外界设定了不可变，但是外界还是可以通过其它的手段来改变内部的值（例如 KVC）,但是这样做是绕开了 API,是不合规范的。


```
//声明文件定义只读属性
@property (nonatomic, strong, readonly) NSSet *friends;


//实现文件定义可变属性
NSMutableSet *_internalFriends;

//让外界使用的只读属性，是可变属性的 copy
- (NSSet *)friends{
    return [_internalFriends copy];
}

//外部可以通过方法来修改可变的对象，不要直接对可变对象进行底层操作，这样可以保持数据的一致性
- (void) addFriends:(NSObject *) friend{
    [_internalFriends addObject:friend];
}


```

#### 理解 OC 错误模型

* 只有发生了可能使整个应用程序崩溃时，才使用异常。
* 在错误不严重的情况下，可以指派 delegate 来处理错误，也可把错误信息放在 NSError 对象里，经由输出参数返回给调用者。

```
id someResource = /*.....*/;
if (/* check for error*/) {
     @throw [NSException exceptionWithName:NSInternalInconsistencyException 
     reason:@"There was an error" userInfo:nil];
}
//如果抛出异常，那么本作用域末尾的对象就无法自动释放了
[someResource doSomething];
[someResource release];
```
在抛出异常之前做释放，虽然能解决问题，但是当项目执行路径更为复杂的话，这样写的代码会很乱。我们应该只有在极罕见的情况下抛出异常，无需考虑恢复直接退出就可以了。比如定义抽象类对没有实现的方法抛出异常。

我们可以通过 NSError 来处理错误，第一种常见的用法是通过 delegate 来传递错误

`-(void)connection:(NSURLConnection *)connection withError:(NSError *)error;`

另一种用法是经由方法的返回参数给调用者

```

NSError *error = nil;
BOOL ret = [objet doSometing:&error];
if(error){
	//There was an error
}

--------------------------------------

-(BOOL)doSomething:(NSError **)error{
	if(/* there was an error */){
		//调用者经常会传入nil，必须判判断避免程序崩溃
		if(error){
			*error = [NSError errorWithDomain:domain code:code 
			userInfo:userInfo];
		}
		return NO;
	}else{
		return YES;
	}
}

```




