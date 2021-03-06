# NSNotification、delegate和KVO优缺点
## delegate
* 优势
    1. 有非常严格的语法，所有触发代理的方法在代理中有清晰的定义
    2. 如果代理中的一个必要方法没有实现会出现编译警告
    3. 整个触发和调用的控制流程可跟踪并且可识别，通信过程清晰明了
* 缺点
    1. 需要定义很多代码：协议定义；遵循代理的delegate属性；在遵循delegate对象实现中实现delegate方法定义
    2. 在释放代理对象时，需要小心的将delegate改为nil。一旦设定失败，那么调用释放对象的方法将会出现内存crash

## NSNotification
* 优势
    1. 不需要编写多少代码，实现比较简单
    2. 对于一个发出的通知，多个对象能够做出反应，即一对多的方式实现简单
    3. 发出通知的时候可以传递context对象,context对象携带了关于发送通知的定义的信息
* 缺点
    1. 在编译期不会检查通知是否能够被观察者正确的处理
    2. 在释放注册的对象时，需要在通知中心取消注册
    3. 在调试的时候应用的工作以及控制过程难跟踪
    4. 通知发出后，不能从观察者获取任何的反馈信息

## KVO 键值监听
* 优势
    1. 能够提供一种简单的方法实现两个对象之间的同步
    2. 能够对非我们创建的对象，即内部对象的状态改变做出响应，而且不需要改变内部的实现
    3. 能够提供观察的属性的最新值以及先前值
    4. 用key paths来观察属性，因此也可以观察嵌套对象
    5. 完成了对观察者的抽象，因为不需要额外的代码来允许观察值能够被观察
* 缺点
    1. 我们观察的属性必须使用strings来定义。因此在编译器不会出现警告以及检查
    2. 对属性重构将导致我们的观察代码不在可用