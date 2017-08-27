##Forward Declaration
**[Item 2]**

把声明（`#import`、协议、属性等）尽量移到`.m`文件而不是放在头文件里。如果A类的属性或方法中用到了B类，则在`A.h`中用`@Class B`告知编译器B类的存在即可，不要`#import B.h`，**除非**A是B的子类。这样做可以：

1. 当这个头文件被引用时，避免引用者知道不必要的细节，节省其编译时间
2. 如果有两个类需要相互引用，头文件中互相`#import xxx.h`会造成交叉引用，移到`.m`文件即可避免
3. 一些私有的方法、属性和协议等可以避免在头文件中暴露

##Literal Syntax
**[Item 3]**

`NSNumber`不只可以包装纯数字：

```objc
NSNumber *boolNumber = @YES;
NSNumber *charNumber = @'a';
```

Literal Syntax只能创建不可变对象，对于可变对象，应该这样写：

```objc
NSMutableArray *mutableArray = [@[@1, @2, @3] mutableCopy];
```

##Define Constants
**[Item 4]**

不要使用`#define`，因为它不包含类型信息，而且可能不知不觉地在别处被重定义。

- 如果一个常量**仅在一个`.m`文件中使用**，应该这样写：

```objc
static const NSTimeInterval kAnimationDuration = 0.3;
```

- 如果一个常量将被多个文件共享（比如`NSNotification`的名称），应该这样写：

```objc
// EOCView.h
extern NSString *const EOCViewDidAnimateNotification;

// EOCView.m
NSString *const EOCViewDidAnimateNotification = @"EOCViewDidAnimateNotification";
```

####注意

- 常量名的命名规范：只用于一个文件时应该**以"k"为前缀**，用于多个文件时应该**以所在类的类名等为前缀**。

##Enum
**[Item 5]**

- 通用枚举：

```objc
typedef NS_ENUM(NSUInteger, EOCConnectionState) {...};
```

- 二进制位枚举：

```objc
typedef NS_OPTIONS(NSUInteger, EOCDirection) {
	EOCDirectionUp		= 1 << 0,
	EOCDirectionLeft	  = 1 << 1,
	EOCDirectionRight	 = 1 << 2
};
```

这种枚举用于描述或的关系，比如方向为左或为右时执行某一方法，应该这样判断：

```objc
EOCDirection permittedDirection = EOCDirectionLeft | EOCDirectionRight;
if (direction & permittedDirection) {
  [self doSomething];
}
```

####注意

- 在`switch`语句中判断这样的枚举类型时**不要**写`default:`。否则如果漏写了其中一种枚举的`case xxx:`，编译器不会提示。

##Property and Instance Variable
**[Item 7]**

###Property

`@property`相当于创建了加下划线的成员变量，以及`setter`和`getter`方法。`@property`后面括号里的内容是为了**给编译器自动创建`setter`和`getter`提供依据**，比如若不指定为`nonatomic`，则默认为`atomic`，系统会保证`setter`和`getter`的完整执行，不会在set到一半时get；若有`readonly`，则编译器不生成`setter`。

这个`setter`仅是对外部而言的，在类的内部仍然可以通过下划线加属性名**直接访问**对应的成员变量，从而修改其值。用点语法访问属性，**一定会调用`setter`和`getter`**。若有`readonly`，则不能用点语法来赋值。

建议**用点语法来set**，方便在`setter`方法里设断点观察状态；**用下划线来get**，这样速度快。

####注意

- **`init`方法**一定用下划线来set，因为`setter`方法可能被子类重写，比如要求一个字符串不能为空；而父类初始化时就是将字符串置空，造成`init`失败。
- 如果存在**懒加载**，即`getter`中判断实例不存在，才初始化一个实例，这时应该用点语法，保证调用`getter`，而不是用下划线，否则可能取到`nil`。

###Instance Variable

头文件里`@interface`后大括号里声明的是成员变量，可以被以下三种符号修饰：

- `@public` 对所有类都可见
- `@protected` 对该类及其子类可见
- `@private` 仅对该类可见

这些变量是没有`setter`和`getter`方法的，不能用点语法访问；在该类及其子类中可以直接用变量名来访问，在其他类中可以用实例名加`->`来访问。

##isEqual:
**[Item 8] [Item 14]**

对于对象而言，`==`所比的是两个对象的**地址**是否相同(identical)，而不是两个地址不同的对象的**内容**是否相同(equal)。后一种比较应该用`isEqual:`方法。系统类如`UIColor`、`NSArray`等都可以直接调用该方法，自定义类则不然，`isEqual:`返回的是用`==`判断的结果，因此需要重写该方法。

注意到`isEqual`方法接受的参数是`id`类型，所以重写时第一步就是判断类型是否相同：

```objc
if (![object isMemberOfClass:self.class]) return NO;
```

然后一个一个地判断其他成员变量是否相同。

另外需要重写的是`hash`方法。`NSSet`和`NSDictionary`判断两个元素是否相等时，首先比较`hash`值，若相等，再调用`isEqual:`。`hash`值是由实例的内容决定的，equal的实例`hash`值一定相同，但`hash`值相同时不一定equal。

系统类都有`hash`方法，但自定义类的`hash`方法如果不重写，返回的是地址值，即只有identical时`hash`值才相同。重写时应该对所有成员变量`hash`值做异或(`^`)：

```objc
- (NSUInteger)hash {
	NSUInteger firstNameHash = [_firstName hash];
	NSUInteger lastNameHash = [_lastName hash];
	NSUInteger ageHash = _age;
	return firstNameHash ^ lastNameHash ^ ageHash;
}
```

####注意

- 如果将可变对象加入`NSSet`，然后修改它，使之与集合中另一元素内容相同，那么对这个`NSSet`做`copy`的结果是元素会变少，因为不能向集合加入与已有元素equal的对象。必须注意`count`减少的问题，可能带来bug。（也可以利用这一特性，通过`NSArray`->`NSSet`->`NSArray`来去掉数组中重复的对象）
- 判断类型应该用`isMemberOfClass:`和`isKindOfClass:`，而不是`[object class] == [self.class]`。如果目标类利用了消息传递机制，比如`NSProxy`，`==`比的是**proxy类本身**，而`isMemberOfClass:`和`isKindOfClass:`比的是**被代理**的类，即真正接收到message的类。

##Class Cluster
**[Item 9]**

初始化一个`UIButton`时，选择不同的type，实际上系统会`switch(type)`，然后创建不同的`UIButton`的子类。向外暴露的只有`UIButton`这一个父类，而其每一个子类都重写了父类的所有方法。这就是类簇的设计模式，其他很多系统类如`NSArray`、`NSString`等都采用这一模式。

####注意

- 此时`[newButton isMemberOfClass:UIButton.class]`的结果一定是`NO`，应该用`isKindOf:`来判断。

##Message Forwarding
**[Item 12] [Item 13]**

[Item 12]介绍了消息传递机制，以及为`@dynamic`属性创建`setter`和`getter`的方法，与数据库交互的时候用得着。

[Item 13]介绍了在不创建子类的情况下为任何已有方法（如`NSString`的`lowercaseString`方法）添加附加操作（比如打印出`lowercaseString`执行前后的字符串），从而监控方法的执行。此法在debug时有用，但应该尽量少用。

##Prefixing
**[Item 15]**

所有的**自定义类名**、**C函数名**和**全局变量名**都应该加前缀，可以是APP名、公司名缩写等。前缀应由**不少于3个字母**组成（苹果保留所有2个字母的前缀），比如本书常用的`EOC`。

如果在自己创建的库中引入了第三方库，则应该给后者也加上自己的前缀(`XYZLibrary -> EOCXYZLibrary`)，因为当自己创建的库再被其他人引用的时候，其他人可能又引用了同样的第三方库（可能是不同版本的），编译时可能报`duplicated symbols`错误。

##Designated Initializer
**[Item 16]**

`init`方法可以有很多个，但应该有一个designated initializer，即只有这一个方法是会真正创建实例的，而其他`init`方法要么都调用这一方法，要么抛异常（尽量避免抛异常；可以赋默认值，然后正常实例化）。

####注意
- 如果在子类中换用另一方法作为designated initializer，记得重写父类的designated initializer，使之也调用这一方法来创建实例。

##Description Information
**[Item 17]**

为了`NSLog(@"%@", customObject);`能打印出自定义类的实例的具体信息，必须重写其`description`方法：

```objc
- (NSString *)description {
	return [NSString stringWithFormat:@"%@ %@", 
		    _firstName,
		    _lastName];
}
```

另外为了断点时`po customObject`也能打印出具体信息，还应重写`debugDescription`方法：

```objc
- (NSString *)debugDescription {
	return [NSString stringWithFormat:@"<%@: %p, \"%@ %@\">", 
		    self, 
		    [self class], 
		    _firstName, 
		    _lastName];
}
```

具体信息的部分用字典来写就更漂亮一些：

```objc
- (NSString *)debugDescription {
	return [NSString stringWithFormat:@"<%@: %p, %@>", 
		    self, 
		    [self class], 
		    @{@"firstName":_firstName,
		      @"lastName":_lastName}
		    ];
}
```

##Immutable
**[Item 18]**

为了避免数据遭到不必要的修改：

1. 不应该被外部直接修改的属性，应该声明`(readonly)`（可以在`Extension`中重新声明为`(readwrite)`，使它对外只读，对内可读写）。
2. 不要把`NSMutable(Array/Dictionary/Set...)`暴露出来，应该只留一个`setter`给外部使用，以免它们被其他类修改时，类自身难以察觉。
3. 如果数据不是特别多，`copy`的代价不是特别大，留给其他类的`getter`应尽量用`copy`方法，返回一个不可变的对象。

##Naming
**[Item 19] [Item 20]**

1. 如果方法返回一个新生成的值，应该以该值的类型开头，例如`stringWithFormat:`。直接返回某个属性的方法除外，直接用属性名即可。
2. 方法名里应该用整个单词(string)，不要用缩写(str)。
3. `BOOL`属性加前缀`is`。返回`BOOL`值的方法（但不是直接返回某个`BOOL`型的属性）应该以`is`或`has`为前缀。
4. 用前缀`p_`来标示私有方法。不要只用下划线，因为苹果的保留这种方式，应避免无意中覆盖系统方法的情况。

##Error Handling
**[Item 21] [Item 32]**

与JAVA、C++等语言不同，OC不会频繁地用到异常(`NSException`)。虽然也有`@try`、`@catch`和`@finally`这些处理异常的语句，但在纯OC文件中，异常一般只用于致命的、必须中断APP生命周期的情况，其他非致命情况下都用`NSError`来告知错误信息；而前述那些语句一般用在Objective-C++文件里，或者是调用第三方库、无法修改源代码时使用。

可以给被调用的方法传入一个`NSError`参数：

```objc
- (BOOL)doSomething:(NSError **)error {
	......
	if (/* 出错 */) {
		*error = [NSError errorWithDomain:domain
									 code:code
							     userInfo:userInfo];
		return NO;
	}
}
```

然后做如下调用：

```objc
NSError *error = nil;
BOOL ret = [self doSomething:&error];
if (ret) {
	/* 查看error */
	......
}
```

用额外的`BOOL`量来判断是否出错，而不是`if(error)`，是因为调用者可能不关心具体错误是什么，所以这样调用：`[self doSomething:nil]`。

####注意
- 传参时传的是\*error的地址，所有有两个\*。如果传参时只有一个\*，那么传的就是指向error的指针的**拷贝**，在方法里面对它赋值，只是让这个拷贝指向了新的对象，而外面的指向error的指针以及error的内容均未改变。
- ARC不会给`NSException`加上释放资源的代码，因为一般情况下出现异常都会引发APP终止，所有资源都被销毁。如果要像JAVA、C++等那样在`@finally`里面`release`资源，需要加上`-fobjc-arc-exceptions`的flag。

##Copying
**[Item 22]**

要使自定义类具有`copy`方法，**应该重写的是`copyWithZone`方法**：

```objc
- (id)copyWithZone:(NSZone *)zone {
	return [[[self class] allocWithZone:zone] init...];
}
```

然后调用`copy`方法即可实现复制。如果该类有可变的属性，需要使用`mutableCopy`，那么还需要重写`mutableCopyWithZone`方法。

####注意
- `copyWithZone`方法里应该用designated initializer来初始化实例。
- **所有类**的`copy`或`mutableCopy`都不保证是深拷贝还是浅拷贝，只能根据文档判断。自定义时应该尽量用浅拷贝，减少系统开销。

##Protocol and Delegate
**[Item 23]**

1. 一个对象向多个对象传消息 -> `NSNotification`
2. 多个对象向一个对象传消息 -> `Protocol and Delegate`

第二种的典型情况包括一个页面持有多个下载器，页面遵循一个`protocol`，作为下载器的`delegate`；下载器的下载进度有更新时，把下载进度传给页面，让页面以进度为参数执行某些协议中规定了的方法。

协议规定了一些方法，一些是`@require`的，即声明了遵循此协议的代理类必须实现的方法；另一些是`@optional`的，代理类可以选择性实现。对于后者，调用协议方法前必须用`respondToSelector:`来检查代理类是否实现了这个方法。比如前面的例子中，页面可能不关心下载的中间进度，只关心下载完成的信息，那么下载器在进度更新但下载未完成时，就不必也不能调用协议方法。

对于**频繁调用的可选的协议方法**（如下载器更新进度时调用的方法），为了避免多次检查`respondToSelector:`，可以只在初始化时检查一次，把结果存在全局的结构体里面，此后直接向结构体查询。

####注意
- `delegate`作为属性，必须用`weak`修饰。否则如前面下载器的例子，页面持有下载器，下载器又通过`delegate`持有页面，就造成循环引用，应该让后者为`weak`：

```objc
@property (nonatomic, weak) id <EOCNetworkFetcherDelegate> delegate;
```

##Category and Extension (Class-Continuation Category)
**[Item 24] [Item 25] [Item 27]**

###Category

`Category`可以给已有的类（包括系统类）添加方法，但不能添加属性。比如给`CustomClass`添加一个名为`SayHello`的`category`，Xcode会创建名为`CustomClass+SayHello`的`.h`和`.m`文件，然后可以添加方法，比如`sayHello`。在其他类中`#import "CustomClass+SayHello.h"`，则初始化的`CustomClass`的实例就可以执行`sayHello`方法。如果`CustomClass`的子类也`#import "CustomClass+SayHello.h"`，则`sayHello`方法也被子类继承。

每个类本身实质上都是单例，在某一个`category`里面定义了一个方法，就是把方法加到类自身的单例里面，并没有创建新的子类。所以如果既有`#import "CustomClass+SayHello.h"`，又有`#import "CustomClass+SayBye.h"`，则初始化的实例仍属于`CustomClass`，但既能执行`sayHello`，又能执行`sayBye`。实际上在任何地方，即使不引用`category`的头文件，也能利用消息传递机制直接调用`category`新加的方法，引用头文件只是让编译器知道有这个方法。

`Category`新加入的方法在APP整个生命周期中都有效。如果多个`category`都有同名的方法，它们会相互覆盖，最终执行的是系统最后`load`进来的方法。如果第三方库被他人引用（尤其是第三方库给系统类新加了`category`时），用户新创建`category`，且方法名与第三方库重复，就说不清最后执行的是哪个方法。所以应该给`category`及其方法都加上前缀（前者加`EOC_`，后者加`eoc_`）。

###Extension (Class-Continuation Category)

`Extension`可以声明一些私有的成员变量和方法：

```objc

@interface CustomClass () <PrivateProtocol> {
	id m_variable;
}

@property (nonatomic, readwrite) id object;

- (void)p_method;

@end
```

在类名后加一对小括号就是`extension`，这段代码一般直接写在`.m`文件里。

1. 大括号里可以声明**私有变量**，对其他类（包括子类）都是不可见的（也可以把私有变量写在`@implemention`后的大括号里，效果是一样的，只是习惯问题）。
2. 可以用`@property`把已经在头文件里声明为`readonly`的属性声明为`readwrite`，这样这个属性对外只读，对本类**可读可写**。
3. 可以声明**私有方法**，应该以`p_`为前缀。
4. 如果不想把遵循的**协议**暴露在头文件里，可以写在`extension`里面。

##Anonymous Object Using Protocol
**[Item 28]**

隐藏类名可以通过`id <Protocol> object;`来实现。

有些情况下类实现的方法（遵循的协议）比类名更重要，比如某个方法需要返回某个操作数据库的类，但操作不同数据库(`MySQL`,`PostgreSQL`,...)的类来自不同的库，继承自不同父类，因此方法返回类型无法设为它们共同的父类，设为`id`然后再判断类型也很麻烦。

这种情况下可以令这些操作类都遵循统一的协议，都实现操作数据库需要的几种方法(`connect`,`disconnect`,`performQuery:`,...)，然后方法返回的类型就设为`id <EOCDatabaseConnection>`，这样就不必判断具体类型，直接使用协议方法即可。

##Dealloc Method
**[Item 31]**

`dealloc`方法应该处理：

1. 由`malloc`分配的空间应该`free`
2. `CoreFoundation`对象应该`CFRelease`
3. `NSNotificationCenter`移除`observer`
4. `KVO`移除`observer`

应该避免在`dealloc`方法里调用其他方法，因为后者用到的对象可能已经被销毁。不是所有的清理工作都应该由`dealloc`承担，比如以下情况：

1. 一些比较expensive的资源，如`fileDescriptor`、`sockets`和`malloc`分配的内存，不应该多次销毁和重新创建，应该写`open`和`close`方法，多次使用。
2. APP被突然中止时`dealloc`不会被调用，系统会直接销毁所有资源。需要在这时完成的工作，比如保存信息，应该写在`AppDelegate`的`applicationWillTerminate:`里。

##Weak
**[Item 33]**

`unsafe_unretained`与`weak`的区别是，当被引用的对象被释放时，前者仍然指向那块内存，而`weak`指针在ARC的作用下变成`nil`。

只有**持有**一个对象才用强指针。例如，在头文件中声明UI控件都用`weak`，是因为`StoryBoard`才是控件的真正持有者，其他类只是弱引用。

##Autoreleasepool
**[Item 34]**

对于一段反复执行的代码，必须一个`for`循环，如果每次它都会创建临时对象，那么临时对象就会一直堆积；应该在代码段的首尾加上一个`@autoreleasepool`，使得每次执行后都释放临时对象：

```objc
for (/* 循环条件 */) {
	@autoreleasepool {
		......
	}
}
```

##Zombie Objects
**[Item 35]**

已经被释放的对象，只要它所在的内存区域还没有被重写，它就仍然可能接受消息并响应；即使内存被重写，新的对象也可能接收到发给旧对象的消息，作出不可预料的响应。为了记录运行过程中是否向已经被释放的对象发送了消息，可以在debug时`Enable Zombie Objects`。此时旧对象被释放后，这块内存不会被重写，如果尝试向它发送消息，会引起异常`message sent to deallocated instance...`。

##Block
**[Item 37] [Item 38] [Item 39] [Item 40] [Item 52]**

###存储位置

如果一个`block`没有用到任何外部变量，它是global的，和其他静态变量存在一起。如果用到了外部变量，它将被创建在栈上；如果对`block`用了`copy`方法，或者在ARC环境下它被赋值给一个强指针，它就会被复制到堆上。

###typedef

```objc
int (^EOCSomeBlock)(BOOL flag, int value) = ^(BOOL flag, int value) {...};
```

这样的写法会降低可读性，名称`EOCSomeBlock`被放到了括号里面。应该这样写：

```objc
typedef int (^EOCSomeBlock)(BOOL flag, int value);
EOCSomeBlock block = ^(BOOL flag, int value) {...};
```
如果有两个`block`的参数和返回值都一样，但完成的功能不同，也应该写成两个`typefdef`，分别用不同的名字，提高可读性。记得命名时要加上必要的前缀。

###Handler

有时可以用`block`来代替`delegate`的设计模式，这样不需要把处理回调的代码写在单独的代理方法里面，而是就近写在调用处，增强可读性。也可以让`block`回传`NSError`来判断执行情况：

```objc
typedef void (^EOCNetworkFetcherCompletionHandler)(NSData *data, NSError *error);
```

另一个好处是`block`很容易通过`GCD`等安排在指定线程上执行。

###Break Retain Cycle

用`block`很容易造成循环引用。如果`block`里引用了当前类的成员变量，它就会保持对`self`的强引用；把它作为参数传给另一个实例(比如`NetworkFetcher`)的方法，则这个实例持有`block`（`_completionHandler`），同时它又被`self`持有（`_networkFetcher`），造成循环引用。

一种解决方法是**解除`self`对另一个实例的持有**，即在`block`里面最后令方法所在实例为`nil`（`_networkFetcher = nil;`），这就要求每次传进来的`block`都加这一句。如果是我们自己编写了`NetworkFetcher`类，作为第三方库被别人引用，很难保证用户一定写了这一句。

另一个办法是**解除另一个实例对`block`的持有**，即在另一个实例中用完`block`以后，把它释放掉：

```objc
_completionHandler(_downloadedData);
self.completionBlock = nil;
```

这样就在`NetworkFetcher`内部解决了循环引用的问题，比较安全。

还可以用`__weak`指针来**解除`block`对`self`的持有**，即：

```objc
__weak EOCClass *weakSelf = self;
EOCNetworkFetcherCompletionHandler completionHandler = ^(NSData *data, NSError *error) {
	EOCClass *strongSelf = weakSelf;
	[strongSelf doSomething];
	......
}
```

`block`里面先转化为强指针，避免执行的过程中`self`被释放掉。执行完后遇到大括号，`strongSelf`就被销毁，再也不存在对`self`的强引用。

##GCD

**[Item 41]** 安排同步／异步操作

**[Item 42]** 代替`performSelector`系列

**[Item 43]** 有些情况下更适合用`NSOperation`和`NSOperationQueue`，因为相较于`GCD`，它们可以取消执行、设置依赖、`KVO`观察完成进度、设置单个操作（而不是整个队列）的优先级、可重用。不过它们是`OC`类，没有`GCD`(C API)那么轻便。

**[Item 44]** 安排分组任务

**[Item 45]** 对于只执行一次，而且要保证线程安全的代码，应该用`dispatch_once`，比如单例的`sharedInstance`方法，原来的写法是：

```objc
static EOCClass *sharedInstance = nil;
@synchronized(self) {
	if (!sharedInstance) {
		sharedInstance = [[self alloc] init...];
	}
}
```

可以改成：

```objc
static EOCClass *sharedInstance = nil;
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
	sharedInstance = [[self alloc] init...];
});
```

后一种实现比使用`@synchronized`快得多。

##Enumeration
**[Item 48]**

**快速枚举**即`for...in`，可以这样实现反向枚举：

```objc
for (id object in [anArray reverseObjectEnumerator]) {...}
```

**`block`枚举**即`enumerate...UsingBlock:`方法，它的`block`有一个参数`BOOL *stop`，用`*stop = YES`即可停止枚举。这种枚举的好处在于：

- 枚举时可有选项，用`enumerate...WithOptions:usingBlock:`方法，第一个参数是`NS_OPTIONS`类型，可选枚举时**是否并发**、**是否反向**
- 枚举`NSArray`的时候能知道**下标**
- 枚举`NSDictionary`可以**同时得到`key`和`value`**，而快速枚举只能枚举`key`或`value`，再用`key`去取`value`或相反

`block`里面`key`和`value`默认的类型是`id`，这是可以直接改的，比如改成：

```objc
[aDictionary enumerateKeysAndObjectsUsingBlock:
	^(NSString *key, CustomClass *obj, BOOL *stop) {...}];
```

##CF Objects
**[Item 49]**

`CoreFoundation`是C的API，其类以`CF`为前缀；`Foundation`是OC的库，其类以`NS`为前缀。

`NS` -> `CF`：

- `__bridge`是不移交所有权的转换，ARC仍然负责其引用计数
- `__bridge_retained`则是移交控制权，ARC不再负责，需要手动调用`CFRelease()`

`CF` -> `NS`：

- `__bridge`仍是不移交所有权的转换，仍需要手动调用`CFRelease()`
- `__bridge_transfer`则是把引用计数交给ARC

很多`NS`类都有对应的`CF`类，但两者有不同的特点。例如`NSDictionary`的内存管理策略是`key`为`copy`，`value`为`retain`，策略不能更改；而`CFDictionary`的内存管理都是可以自定义的。

##NSCache and NSPurgeableData
**[Item 50]**

###NSCache

对于比较expensive的资源，比如从网络下载的或者从硬盘读取的图片，应该在内存允许的情况下存进字典，下一次使用时直接从内存读取，而不是重新下载或从硬盘读取。但内存紧张时，如`NSDictionary`之类不会自动决定要释放哪些资源，为避免手动管理的麻烦，应该用`NSCache`。

`NSCache`和`NSDictionary`很像，都有一对一对的`key`和`value`。但`NSCache`可以用不可`copy`的对象作为`key`，而且是线程安全的，可以在不加锁的状态下给它发数条读取命令。

`NSCache`用两个参数来辅助决定是否释放资源、释放哪些资源：一是`key`-`value`对的总数，而是所有`value`的`cost`的总和的上限。这两个参数都是手动设置的，前一个比较简单，也就是设定字典的`count`值的上限；第二个所谓的`cost`是需要自己定义的，建议直接用`NSData`的数据大小，不要用复杂的方法来计算`cost`，增加系统开销。代码可能是这样的：

```objc
_cache.countLimit = 100;
_cache.totalCostLimit = 5 * 1024 * 1024;
```

即最多装100个对象，`cost`总量的上限，也就是数据大小的上限值是5MB。

####注意
- 这些上限值仅作参考，并不是超过了总量就一定会释放、没超总量就一定不释放，全凭系统决定，不要手动干预这个过程。

###NSPurgeableData

`NSPurgeableData`是可以配合`NSCache`使用的一个类，可以用`NSData`来创建。`NSCache`有一个`BOOL`型的属性`evictsObjectsWithDiscardedContent`，设为`YES`时，`NSPurgeableData`可以被系统直接移出`NSCache`；设为`NO`时不会被自动移除，但其数据可能被系统释放掉，使用之前须调用`isContentDiscarded`检查数据还在不在。

使用`NSPurgeableData`之前（除了刚刚初始化的时候），应该调用`beginContenAcess`方法，告知系统即将开始使用资源，不要在使用期间释放资源；用完以后要执行`endContentAcess`方法，告诉系统现在开始可以自行决定是否释放。

##Load and Intialize Methods
**[Item 51]**

`NSObject`有一个叫`load`的类方法，当一个类或是`category`被**加入runtime**时被调用一次。此时处于同一个`library`里面的其他类都不保证已被`load`，所以不要调用他们。这个方法里**只应该做简单的打印**，不要做太多操作，尤其是会阻塞主线程的。

另外有一个叫`initialize`的类方法，当一个类**第一次被使用时**调用一次（未被使用的类的`initialize`方法永远不被调用）。此时执行该类的其他方法或者调用其他类都是安全的，但仍应该避免做太多操作。除了打印以外，可以**初始化一些编译时不能确定的量**，如：

```objc
static NSMutableArray *anArray;

@implemention EOCClass

+ (void)load {
	NSLog("EOCClass loaded");
}

+ (void)initialize {
	if (self == [EOCClass class]) {
		anArray = [NSMutableArray new];
	}
}
```

####注意
- `load`方法不会被继承或重写，所以不需要判断`class`；`initialize`则参与继承和重写。因为静态变量只有一个，被父类和子类共享，所以要判断`class`，子类无需再初始化一遍。

##NSTimer
**[Item 52]**

用**`NSTimer`**也很容易造成循环引用，因为它创建时有一句`target:self`。一种解决方法是给`NSTimer`加一个`category`，于其内做初始化，`target:self`就相当于`target:[NSTimer class]`，虽然循环引用仍然存在，但`NSTimer`类自身是个单例，自己循环引用自己是没问题的。

此时可以把原来要执行的方法装进`block`传到`category`，放在`NSTimer`的`userInfo`里面；新建一个方法，接收`NSTimer`作为参数，方法的内容就是从其`userInfo`里取出`block`并执行。将这一方法作为初始化`NSTimer`时用的`@selector`。`category`的代码如下：

```objc
+ (NSTimer *)eoc_scheduledTimerWithTimeInterval:(NSTimeInterval)interval
						    			  block:(void(^)())block
									    repeats:(BOOL)repeats {
	return [self scheduledTimerWithTimeInterval:interval
									     target:self
									   selector:@selector(eoc_blockInvoke:)
									   userInfo:[block copy]
									    repeats:repeats];
}

+ (void) eoc_blockInvoke:(NSTimer *)timer {
	void (^block)() = timer.userInfo;
	if (block) {
		block();
	}
}
```

因为用到了`block`，也要注意它的循环引用问题。