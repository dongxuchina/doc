
KVC（Key-value coding）键值编码，可以允许开发者通过Key名直接访问对象的属性，或者给对象的属性赋值。而不需要调用明确的存取方法。这样就可以在运行时动态在访问和修改对象的属性。而不是在编译时确定。很多高级的iOS开发技巧都是基于KVC实现的。

**KVC的定义**

KVC的定义都是对NSObject的扩展来实现的，所以对于所有继承了NSObject在类型都能使用KVC，下面是KVC最为重要的四个方法

```
//直接通过Key来取值
- (nullable id)valueForKey:(NSString *)key; 

//通过Key来设值
- (void)setValue:(nullable id)value forKey:(NSString *)key; 

//通过KeyPath来取值
- (nullable id)valueForKeyPath:(NSString *)keyPath; 

//通过KeyPath来设值
- (void)setValue:(nullable id)value forKeyPath:(NSString *)keyPath; 
```
NSKeyValueCoding类别中还有其他的一些方法

```
//默认返回YES，表示如果没有找到Set方法的话，会按照_key，_iskey，key，iskey的顺序搜索成员，设置成NO就不这样搜索
+ (BOOL)accessInstanceVariablesDirectly;

//KVC提供属性值确认的API，它可以用来检查set的值是否正确、为不正确的值做一个替换值或者拒绝设置新值并返回错误原因。
- (BOOL)validateValue:(inout id __nullable * __nonnull)ioValue forKey:(NSString *)inKey error:(out NSError **)outError;

//这是集合操作的API，里面还有一系列这样的API，如果属性是一个NSMutableArray，那么可以用这个方法来返回
- (NSMutableArray *)mutableArrayValueForKey:(NSString *)key;

//如果Key不存在，且没有KVC无法搜索到任何和Key有关的字段或者属性，则会调用这个方法，默认是抛出异常
- (nullable id)valueForUndefinedKey:(NSString *)key;

//和上一个方法一样，只不过是设值。
- (void)setValue:(nullable id)value forUndefinedKey:(NSString *)key;

//如果你在SetValue方法时面给Value传nil，则会调用这个方法
- (void)setNilValueForKey:(NSString *)key;

//输入一组key,返回该组key对应的Value，再转成字典返回，用于将Model转到字典。
- (NSDictionary *)dictionaryWithValuesForKeys:(NSArray *)keys;
```

**KVC 是如何寻找 key 的**

当调用 setValue:forKey 代码时，搜索方式如下：

* 程序优先调用 set<Key>: 属性值方法，如果成员变量用 @property 修饰，自动会生成setter方法，所以这种情况下会直接搜索到。
* 如果没有找到 set<Key>: 方法，KVC 机制会检查 + (BOOL)accessInstanceVariablesDirectly 方法有没有返回 YES（默认返回 YES ），如果返回 NO，会执行 setValue：forUNdefinedKey：方法，如果返回 YES，则会按 _<key>，_is<Key>，<key>，is<key> 的顺序搜索成员名，如果都不存在，系统将会执行该对象的 setValue：forUNdefinedKey：方法，默认会抛出异常。

当调用 valueForKey: 代码时，搜索方式不同于 setValue:forKey，其搜索方式如下：

* 首先按 get<Key>、<key>、is<Key> 的顺序查找 getter 方法，找到直接调用。如果是 bool、int 等值类型，会做 NSNumber 的转换。
* 如果 getter 没有找到，则查找 countOf<Key>、objectIn<Key>AtIndex:、<Key>AtIndexes 格式的方法。如果 countOf<Key> 和另外两个方法中的一个被找到，那么就会返回一个可以响应 NSArray 所有方法的代理集合 (collection proxy object)。调用这个代理集合的方法，或者说给这个代理集合发送 NSArray 的方法，就会以 countOf<Key>、objectIn<Key>AtIndex:、<Key>AtIndexes 这几个方法组合的形式调用。还有一个可选的 get<Key>:range: 方法。
* 如果上面没有找到，那么会查找 countOf<Key>、enumeratorOf<Key>、memberOf<Key>: 格式的方法。如果这三个方法都找到，那么就返回一个可以响应 NSSet 所有方法的代理集合，以发送给这个代理集合消息方法，就会以 countOf<Key>、enumeratorOf<Key>、memberOf<Key>: 组合的形式调用。
* 如果还没有找到，并且类方法 accessInstanceVariablesDirectly 返回 YES，那么按 _<key>，_is<Key>，<key>，is<key> 的顺序直接搜索成员名。
* 还没有找到的话，调用 valueForUndefinedKey: 。

**KVC 中使用 KeyPath**

一个类的成员变量有可能是其他的自定义类，你可以先用 KVC 获取出来再该属性，然后再次用 KVC 来获取这个自定义类的属性，但这样是比较繁琐的，对此，KVC 提供了一个解决方案，那就是键路径 KeyPath

```
//通过 KeyPath 取值
- (nullable id)valueForKeyPath:(NSString *)keyPath; 

//通过 KeyPath 设置值             
- (void)setValue:(nullable id)value forKeyPath:(NSString *)keyPath;  

```


```
@interface Address : NSObject

@end

@interface Address()
@property (nonatomic,copy)NSString* country;
@end

@implementation Address

@end

---------------------------------

@interface People : NSObject

@end

@interface People()
@property (nonatomic,copy) NSString* name;
@property (nonatomic,strong) Address* address;
@property (nonatomic,assign) NSInteger age;
@end

@implementation People

@end

---------------------------------

int main(int argc, const char * argv[]) {
    @autoreleasepool {
    	 Address *address = [Address new];
        address.country = @"Beijing";
        
        People *people1 = [People new];
        people1.address = address;
        
        NSString *country1 = people1.address.country;
        NSString *country2 = [people1 valueForKeyPath:@"address.country"];
        NSLog(@"country1:%@   country2:%@",country1,country2);
        
        [people1 setValue:@"Shanghai" forKeyPath:@"address.country"];
        country1 = people1.address.country;
        country2 = [people1 valueForKeyPath:@"address.country"];
        NSLog(@"country1:%@   country2:%@",country1,country2);
    }
    return 0;
}

```

输出结果

```
2016-04-17 15:55:22.487 KVCDemo[1190:82636] country1:Beijing  country2:Beijing
2016-04-17 15:55:22.489 KVCDemo[1190:82636] country1:Shanghai  country2:Shanghai
```

如果你不小心错误的使用了 key 而非 KeyPath 的话，KVC会直接查找 address.country 这个属性，很明显，这个属性并不存在，所以会再调用 UndefinedKey 相关方法。而 KVC 对于 KeyPath 是搜索机制第一步就是分离 key，用小数点.来分割key，然后再像普通 key 一样按照先前介绍的顺序搜索下去。

**KVC 异常处理**

KVC中最常见的异常就是不小心使用了错误的Key，或者在设值中不小心传递了nil的值，KVC中有专门的方法来处理这些异常。

```
-(id)valueForUndefinedKey:(NSString *)key{
    NSLog(@"出现异常，该key不存在%@",key);
    return nil;
}

- (void)setValue:(id)value forUndefinedKey:(NSString *)key{
    NSLog(@"出现异常，该key不存在%@",key);
}
```


**KVC 与容器**

略

**KVC 与字典**

当对NSDictionary对象使用KVC时，valueForKey:的表现行为和objectForKey:一样。所以使用valueForKeyPath:用来访问多层嵌套的字典是比较方便的。

KVC里面还有两个关于NSDictionary的方法

```
- (NSDictionary *)dictionaryWithValuesForKeys:(NSArray *)keys;
- (void)setValuesForKeysWithDictionary:(NSDictionary *)keyedValues;
```
dictionaryWithValuesForKeys:是指输入一组key，返回这组key对应的属性，再组成一个字典。
setValuesForKeysWithDictionary是用来修改Model中对应key的属性。

**KVC 正确性验证**

KVC 提供了属性值,用来验证 key 对应的 Value 是否可用的方法

```
- (BOOL) validateValue:<#(inout id  _Nullable __autoreleasing * _Nonnull)#> forKey:<#(nonnull NSString *)#> error:<#(out NSError * _Nullable __autoreleasing * _Nullable)#>
```

**KVC 的使用**

* 动态地取值和设值
* 访问和修改私有变量
* Model和字典转换
* 操作集合


参考：
[详解KVC](http://www.jianshu.com/p/45cbd324ea65)
