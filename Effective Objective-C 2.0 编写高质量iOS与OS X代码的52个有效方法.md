#####第1章 熟悉`Objective-C`
1.了解`Objective-C`语言的起源  

* `Objective-C`为`C`语言添加了面向对象特性，是其超集  
* 使用消息结构，而非函数调用（运行代码由运行环境决定而不是编译器决定）  
* 对象所占内存都分配在堆中，不能在栈中分配`oc`对象 
 
2.在类的头文件中尽量少引用其他头文件

* 将引入头文件的时机尽量延后，在头文件中尽量使用`@class`向前声明  
* `@class`向前声明能解决两个类互相引用的问题  
* 每次在头文件引入其他头文件之前，都先问问自己这样做是否有必要  

3.多用字面量语法，少用与之等价的方法

* 字面量语法能缩短源代码长度，使其更易读  
* 用字面量语法创建出来的对象都是不可变的，如果想使用可变对象，则需要使用`mutableCopy`方法复制一份  
* 应该通过取下标操作来访问数组下标或字典中的键所对应的元素（`arr[1]`、`dic[key]`）  
* 用字面量语法创建数组或字典时，若值为`nil`，则会抛出异常  

4.多用类型常量，少用`#define`预处理指令

* 不要使用预处理指令定义常量，这样定义出来的常量不含类型信息，而且有可能被别处定义的常量覆盖  
* 在实现文件中使用`static const`来定义常量时，因为此类常量只在编译单元内可见，不在全局符号表中，所以无须加前缀
* 在头文件中使用`extern`来声明全局常量，并在实现文件中定义其值，并建议添加前缀（`NSString *const XXStringConst = @"const";`）

5.用枚举表示状态、选项、状态码

* 使用枚举来表示状态，并起个好名字
* 在处理枚举类型的`switch`语句中，不要实现默认的`default`分支，这样加入新类型以后，编译器会提示未处理所有枚举

#####第2章 对象、消息、运行期
6.理解“属性”这一概念

* 应使用`nonatomic`属性，因为`atomic`会严重影响性能
* 属性其实就是对实例变量的存取封装
* `unsafe_unretained`语义与`assgin`相同，但是适用于对象类型
* `weak`属性所指向的对象被摧毁时，属性值也会被清空（`nil`）

7.在对象内部尽量直接访问实例变量

* 在对象内部读取数据时，应该直接通过实例变量来读，而写入数据时，应通过属性来写
* 在初始化方法及`dealloc`方法中，总是应该直接通过实例变量来读写数据
* 有时会用懒加载配置数据，此时需要通过属性来读取数据

8.理解“对象等同性”这一概念

* 若想检测对象的等同性，请提供`isEqual:`与`hash`方法
* 相同的对象必须具有相同的哈希码，但是两个哈希码相同的对象却未必相同
* 不要盲目的逐个检测每条属性，而是应该依照具体需求来制定检测方案
* 编写`hash`方法时，应该使用计算速度快而且哈希码碰撞几率低的算法

9.以“类族模式”隐藏实现细节

* 大部分`collection`类都是类族，例如`NSArray`
* `[arrayObject class] == [NSArray class]`永远不可能为`true`
* 子类应该继承自类族中的抽象基类
* 子类应该定义自己的数据存储方式
* 子类应该覆写超类文档中指明需要覆写的方法

10.在既有类中使用关联对象存放自定义数据

* 在设置关联对象值时，通常使用静态全局变量做键（`static void *XXkey = "XXkey";`）
* 只有在其他做法不可行时才应选用关联对象，因为使用关联对象通常会引入难以查找的`bug`

11.理解`objc_msgSend`的作用

* 消息由接收者、选择子及参数构成
* 此方法需要在接收者所属的类中搜寻其“方法列表”（`list of method`），如果能找到与选择子名称相同的方法，就跳至其实现代码。若是找不到，就继续沿着继承体系向上查找，等找到对应的方法之后再跳转。如果最后还是找不到对应的方法，那就执行“消息转发”（`message forwarding`）操作
* `objc_msgSend`会把查找方法的结果缓存在“快速映射表”（`fast map`）里，每个类都有这么一块缓存，以便下次快速执行对应的方法
* `objc_msgSend_stret`用来返回结构体
* `objc_msgSend_fpret`用来返回浮点数
* `objc_msgSendSuper`用来给超类发消息
* 每个类里都有一张表格，选择子的名称为键，值为对应方法实现函数的指针

12.理解消息转发机制

* 若对象无法响应某个选择子，则进入消息转发流程
* 通过运行期的动态方法解析功能，我们需要在用到某个方法时再将其加入类中（`resolveInstanceMethod:`）
* 对象可以把其无法解读的某些选择子转交给其他对象来处理（`forwardingTargetForSelector:`）
* 经过上述两步后，如果还是没办法处理选择子，就启动完整的消息转发机制（`forwardingInvocation:`）

13.用“方法调配技术”调试“黑盒方法”

* 在运行期，可以向类中新增或替换选择子所对应的方法实现
* 使用另一份实现来替换原有的方法实现，这道工序叫做“方法调配”，开发者常用此技术向原有实现中添加新功能（`method_exchangeImplementations`）
* 一般来说，只有调试程序的时候才需要在运行期修改方法实现，这种做法不宜滥用

14.理解“类对象”的用意

* “在运行期检视对象类型”这一操作也叫做“类型信息查询”（`introspection`，内省）
* 类对象所属的类型（也就是`isa`指针所指向的类型）是另外一个类，叫做“元类”（`metaclass`），用来描述类对象本身所具有的元数据。类方法就定义于此处，因为这些方法可以理解成类对象的实例方法。每个类仅有一个类对象，而每个类对象仅有一个与之相关的“元类”
* 尽量使用类型信息查询方法来确定对象类型，而不要直接比较类对象，因为某些对象可能实现了消息转发功能

#####第3章 接口与 `API` 设计 
15.用前缀避免命名空间冲突

* 在代码中所有的类、方法和分类中，使用适当的前缀，避免冲突
* 若我们自己所开发的程序中用到了第三方库，则应为其中的名称加上前缀
* 前缀最好是三个字母，因为`Apple`宣称其保留所有“两字母前缀”的权利

16.提供“全能初始化方法”

* 可为对象提供必要信息以便其能完成工作的初始化方法叫做“全能初始化方法”
* 在类中提供一个全能初始化方法，并于文档声明，其他初始化方法均应调用此方法
* 若全能初始化方法与超类不同，则需覆写超类中的对应方法
* 如果超类的初始化方法不适用于之类，那么应该覆写这个超类方法，并在其中抛出异常

17.实现`description`方法

* 实现`description`方法返回一个有意义的字符串，用以描述该实例
* 在新实现的`description`方法中，也应该像默认的实现那样，打印出类的名字和指针地址

18.尽量只用不可变对象

* 尽量创建不可变的对象
* 应该尽量把对外公布出来的属性设为只读，而且在确有必要时才将属性对外公布
* 若某属性仅可于对象内部修改，则在“`class-continuation`分类”中将其由`readonly`属性扩展为`readwrite`属性
* 不要把可变的`collection`作为属性公开，而应提供相关方法，以此修改对象中的可变`collection`

19.使用清晰而协调的命名方式

* 起名时遵从标准的`Objective-C`命名规范，这样创建出来的接口更容易为开发者所理解
* 方法名要言简意赅，从左至右读起来像个日常用语中的句子才好
* 方法名里不要使用缩略后的类型名称
* 给方法起名时的第一要务就是确保其风格与你自己的代码或所要集成的框架相符
* 常用规则总结：
 * 如果方法的返回值是新创建的，那么方法名的首个词应是返回值的类型，除非前面还有修饰语，例如`localizedString`（存取方法不遵循此命名方式）
 * 应该把表示参数类型的名词放在参数前面
 * 如果方法要在当前对象上执行操作，那么就应该包含动词；若执行操作时还需要参数，则应该在动词后面加上一个或多个名词
 * `Boolean`属性应加`is`前缀，如果某方法返回非属性的`Boolean`值，那么应该根据其功能，选用`has`或`is`当前缀
 * 将`get`这个前缀留给那些借由“输出参数”来保存返回值的方法

20.为私有方法名加前缀

* 给私有方法的名称加上前缀，这样很容易与公用方法区分开，便于修改
* 不要单用一个下划线做私有方法的前缀，因为这是预留给`Apple`用的

21.理解`Objective-C`错误模型

* 只有发生了可使整个应用程序崩溃的严重错误时，才应使用异常
* 在错误不那么严重的情况下，可以指派“委托方法（`delegate method`）”来处理错误，也可以把错误信息放在`NSError`对象里，经由“输出参数”返回给调用者

22.理解`NSCopying`协议

* 若想令自己所写的对象具有拷贝功能，则需实现`NSCopying`协议
* 如果自定义的对象分为可变版本与不可变版本，那么就要同时实现`NSCopying`与`NSMutableCopying`协议
* 深拷贝的意思就是：在拷贝对象自身时，将其底层数据也一并复制过去。`Foundation`框架中的所有`collection`类在默认情况下都执行浅拷贝，也就是说，只拷贝容器对象本身，而不复制其中数据（1.未必容器内的对象可以复制 2.调用者未必想深拷贝）
* 复制对象时需决定采用浅拷贝还是深拷贝，一般情况下应该尽量执行浅拷贝
* 如果你所写的对象需要深拷贝，那么可考虑新增一个专门执行深拷贝的方法

#####第4章 协议与分类 
23.通过委托与数据源协议进行对象间通信

* 委托模式为对象提供了一套接口，使其可由此将相关事件告知其他对象
* 将委托对象应该支持的接口定义成协议，在协议中把可能需要处理的事件定义成方法
* 当对象需要从另外一个对象中获取数据时，可以使用委托模式。这种情况下，改模式亦称“数据源协议”（`data source protocal`）

24.将类的实现代码分散到便于管理的数个分类之中

* 使用分类机制把类的实现代码划分成易于管理的小块
* 将应该视为“私有”的方法归入名叫`Private`的分类中，以隐藏实现细节

25.总是为第三方类的分类名称加前缀

* 向第三方类中添加分类时，应该给其名称加上你专用的前缀
* 向第三方类中添加分类时，应该给其中的方法名加上你专用的前缀

26.勿在分类中声明属性

* 把封装数据所用的全部属性都定义在主接口里
* 在“`class-continuation`分类”之外的其他分类中，可以定义存取方法，但是尽量不要定义属性

27.使用“`class-continuation`分类”隐藏实现细节

* 通过“`class-continuation`分类”向类中新增实例变量
* 如果某属性在主接口中声明为只读，而类的内部又要用设置方法修改此属性，那么就在“`class-continuation`分类”中将其扩展为可读写
* 把私有方法的原型声明在“`class-continuation`分类”里面
* 若想使类所遵循的协议不为人所知，则可于“`class-continuation`分类”中声明

28.通过协议提供匿名对象

* 协议可在某种程度上提供匿名类型。具体的对象类型可以淡化成遵从某协议的`id`类型，协议里规定了对象所应实现的方法
* 使用匿名对象来隐藏类型名称（或类名）
* 如果具体类型不重要，重要的是对象能够响应（定义在协议里的）特定方法，那么可以用匿名对象来表示

#####第5章 内存管理 
29.理解引用计数

* 引用计数机制通过可以递增递减的计数器来管理内存
* 对象创建好以后，其保留计数至少为`1`
* 若对象的保留计数为正，则对象继续存活；当保留计数降为`0`时，对象就被销毁了
* 在对象生命周期中，其余对象通过引用来保留或者释放对象，保留或者释放操作分别会递增或递减保留计数

30.以`ARC`简化引用计数

* `ARC`管理对象生命期的基本办法就是在合适的地方插入“保留”及“释放”操作
* `ARC`只负责管理`Objective-C`对象的内存，`CoreFoundation`对象不归`ARC`管理，开发者必须适时调用`CFRetain / CFRelease`

31.在`dealloc`方法中只释放引用并解除监听

* 在`dealloc`方法里，应该做的事情就是释放指向其他对象的引用，并取消原来订阅的`KVO`或`NSNotificationCenter`等通知
* 如果对象持有文件描述符等系统资源，那么应该专门编写一个方法来释放此种资源。这样的类要和其使用者约定：用完资源后必须调用`close`方法
* 执行异步任务的方法不应在`dealloc`中调用；只能在正常状态下调用的那些方法也不应该在`dealloc`中调用，因为此时对象已处于正在回收的状态了

32.编写“异常安全代码”时留意内存管理问题

* 捕获异常时，一定要注意将`try`块内所创立的对象清理干净
* 在默认情况下，`ARC`不生成安全处理异常所需的清理代码。开启编译器标志后，可生成这种代码，不过会导致应用程序变大，而且会降低运行效率

33.以弱引用避免保留环

* 将某些引用设为`weak`，可避免出现“保留环”
* 用`unsafe_unretained`修饰的属性特质，其语义同`assign`等价，而后者通常用于基本类型，前者通常用于对象类型
* 用`weak`修饰的对象被销毁时，属性值为自动设为`nil`

34.以“自动释放池块”降低内存峰值

* 自动释放池排布在栈中，对象收到`autorelease`消息后，系统将其放入最顶端的池中
* 合理运用自动释放池，可降低应用程序的内存峰值
* `@autoreleasepool`这种新式写法能创建出更为轻便的自动释放池

35.用“僵尸对象”调试内存管理问题
  
* 系统在回收对象时，可以不将其真的回收，而是把它转化为僵尸对象（`NSZombieEnabled`选项） 
* 系统会修改对象的`isa`指针，令其指向特殊的僵尸类，从而使该对象变为僵尸对象。僵尸类能够响应所有的选择子，响应方式为：打印一条包含信息内容及其接收者的消息，然后终止应用程序

36.不要使用`retainCount`    

* `retainCount`只反应了某一时刻的值，并没有考虑到自动释放操作  
* `retainCount`永远不会返回`0`，系统会优化操作，当`retainCount`为`1`并接受到`release`消息时，就直接释放对象，而不会再对`retainCount`执行`-1`操作  

#####第6章 `Block` 与 `GCD` 
37.理解块的概念

* 块本身可视为对象，且有引用计数
* 在块的内存布局中，最重要的就是`invoke`变量，这是指向块实现代码的指针
* 给块发送`copy`消息可以将块从栈复制到堆上。一旦复制到堆上，块就成了带引用计数的对象了，后续的复制操作只会增加块的引用计数

38.为常见的块类型创建`typedef`

* 以`typedef`重新定义块类型，可令块变量用起来更简单
* 定义新类型时应遵从现有的命名习惯，勿使其名称与别的类型相冲突
* 不妨为同一个块签名定义多个类型别名。如果要重构的代码使用了块类型的某个别名，那么只需修改相应`typedef`中的块签名即可，无须改动其他`typedef`

39.用`handler`块降低代码分散程度

* 建议使用同一个块来处理成功或者失败的情况
* 设计`API`时如果用到了`handler`块，那么可以增加一个参数，使调用者可通过此参数来决定应该把块安排在哪个队列上执行

40.用块引用其所属对象时不要出现保留环

* 如果块所捕获的对象直接或间接的保留了块本身，那就需要当心保留环问题
* 一定要找个适当的时机解除保留环，而不能把责任推给`API`的调用者

41.多用派发队列，少用同步锁

* 将同步与异步派发结合起来，可以实现与普通加锁机制一样的同步行为，而这么做却不会阻塞执行异步派发的线程
* 使用同步队列及栅栏块，可以令同步行为更加高效

42.多用`GCD`，少用`performSelector`系列方法

* `performSelector`系列方法容易引起内存管理方面的问题
* `performSelector`系列方法所能处理的选择子太过局限，选择子的返回类型及发送给方法的参数个数都受到限制
* 如果想把任务放在另一个线程上执行，最好用`GCD`来实现

43.掌握`GCD`及操作队列的使用时机

* `GCD`是纯`C`的`API`，而操作队列则是`Objective-C`的对象
* 使用`NSOperation`及`NSOperationQueue`的好处如下：
 * 取消某个操作
 * 指定操作间的依赖关系
 * 通过键值观测机制监控`NSOperation`对象的属性
 * 指定操作的优先级
 * 重用`NSOperation`对象

44.使用`Dispatch Group`机制，根据系统资源状况来执行任务

 * 一系列任务可归入一个`dispatch group`之中，开发者可以在这组任务执行完毕后获得通知

45.使用`dispatch_once`来执行只需运行一次的线程安全代码

* 使用`dispatch_once`来实现`sharedInstance`的效率几乎是用`@synchronized`实现的两倍

46.不要使用`dispatch_get_current_queue`

* `dispatch_get_current_queue`函数的行为常常与开发者预期的不同，该函数已经废弃，只应做调试之用
* 由于派发队列是按层级来组织的，所以无法单用某个队列对象来描述“当前队列”这一概念

#####第7章 系统框架 
47.熟悉系统框架

* `Foundation`和`CoreFoundation`这两个框架提供了构建应用程序所需的许多核心功能
* 无缝桥接功能可以把`CoreFoundation`中的`C`语言数据结构平滑转换为`Foundation`中的`Objective-C`对象

48.多用块枚举，少用`for`循环

* 遍历`collection`有四种方式：
 * `for`循环
 * `NSEnumerator`遍历法
 * 快速遍历法
 * 块枚举（效率最高）
* 块枚举本身就能通过`GCD`来并发执行遍历操作，无须另外编写代码，而其他遍历方式则无法轻易实现这一点
* 若提前知道待遍历的`collection`含有何种对象，则应修改块签名，指出对象的具体类型

49.对自定义其内存管理语义的`collection`使用无缝桥接

* `__bridge`只做类型转换，但是不修改对象（内存）管理权
* `__bridge_retained`（也可以使用`CFBridgingRetain`）将`Objective-C`的对象转换为`Core Foundation`的对象，同时将对象（内存）的管理权交给我们，后续需要使用`CFRelease`或者相关方法来释放对象
* `__bridge_transfer`（也可以使用`CFBridgingRelease`）将`Core Foundation`的对象转换为`Objective-C`的对象，同时将对象（内存）的管理权交给`ARC`

50.构建缓存时选用`NSCache`，而非`NSDictionary`

* 实现缓存时应选用`NSCache`而非`NSDictionary`对象，因为`NSCache`可以提供优雅的自动删减功能，而且是线程安全的。此外，`NSCache`并不会拷贝键
* 将`NSPurgeableData`与`NSCache`搭配使用，可以实现自动清除数据的功能（当`NSPurgeableData`对象所占内存被系统丢弃时，该对象自身也会从缓存中移除）

51.精简`initialize`与`load`的实现代码

* 在加载阶段，如果类实现了`load`方法，那么系统就会调用。分类里也可以定义此方法，但类的`load`方法要比分类中的先调用
* `load`方法不参与覆写机制
* 应用程序会阻塞并等待所有类的`load`方法执行完才会继续
* 首次使用某个类之前，系统会向其发送`initialize`消息。由于此方法遵从普通的覆写规则，所以通常应该在里面判断当前要初始化的是哪个类
* `initialize`与`load`方法都应该实现的精简一些，这有助于保持应用程序的响应能力，可能减少“依赖环”的几率
* 无法在编译期设定的全局变量，可以放在`initialize`方法里初始化

52.别忘了`NSTimer`会保留其目标对象

* `NSTimer`会保留其目标对象，直到计时器本身失效为止，调用`invalidate`方法可令计时器失效。另外，一次性的计时器在触发完任务以后也会失效
* 反复执行任务的计时器很容易引入保留环，如果这种计时器的目标对象又保留了计时器本身，则肯定会导致保留环
