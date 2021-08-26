#### 代码相关

**对于语句NSString \*obj = [[NSData alloc] init]; obj在编译时和运行时分别是什么类型的对象？**
```
编译时是 NSString 的类型;运行时是 NSData 类型的对象。
```
**在一个对象的方法里面：self.name = @"object";和 _name = @"object"有什么不同吗？**
```
self.name = @"object"; 是通过点语法修改属性 name 的值。
本质上使用的是 name 属性的 setter 方法进行的赋值操作，实际上执行的代码是
[self setName:@"object"];
例如：
    @property(nonatomic, strong) NSString *name;
    // 根据 @property 关键词，系统自动生成setter方法。
    - (void)setName:(NSString *)name{
        // 根据strong关键词
        [name retain];  //内存计数+1
        [_name release];    //把之前指针指向的内容内存计数-1
        _name = name; //指向新内容
    }
而 _name = @“object”; 只是单纯的把‘_name’指针指向‘@"object"’字符串对象所在的地址， 没有调用方法。
```
**这段代码有什么问题吗？**
```
-(void)setAge:(NSString *)age { 
  self.age = age; 
}
```
```
在 age 属性的 setter 方法中，不能通过点语法给该属性赋值。
会造成 setter 方法的循环调用。最终导致程序崩溃.因为 self.age = newAge; 本质上是在调用 [self setAge:newAge]; 方法。
解决循环调用的方法是方法体修改为 _age = age;
```
**你对@interface中的成员变量和@property声明的属性的理解。**
```
@interface AA: NSObject{
    NSString *_name; //成员变量
}
@property NSString *age; //属性

如上所示：
属性拥有 setter 和 getter 方法外加 _age 成员变量。
_name 只是成员变量， 没有 setter 和 getter 方法。
```
**以下代码运行结果如何？**
```
-(void)viewDidLoad  {  
    [super viewDidLoad];  
    NSLog(@"1"); 
    dispatch_sync(dispatch_get_main_queue(), ^{  
        NSLog(@"2");  
    });  
    NSLog(@"3");  
}
结果:只会打印 1 然会就会线程卡死
因为 dispatch_get_main_queue() 得到的是一个串行队列，串行队列的特点： 一次只调度一个任务，队列中的任务一个接着一个地执行（一个任务执行完毕后，再执行下一个任务）
  同步（sync） 操作，它会阻塞当前线程并等待 Block 中的任务执行完毕，然后当前线程才会继续往下运行
  dispatch_sync 提交一个打印任务 NSLog(@”2”) 到主线程关联的串行队列中，主线程关联的串行队列现在有一个 viewDidLoad 任务，
  打印任务 NSLog(@”2”)排在 viewDidLoad 后面，队列 FIFO（先进先出）的原则，打印任务 NSLog(@”2”);
  想要得到执行必须等到 viewDidLoad 执行完毕后才能得到执行，但是 viewDidLoad 想要执行完毕必须要等打印任务 NSLog(@”2”) 执行完毕，所以就卡死在这了。
然而实际开发中很少用到,当然注意下还是有必要的.
```
**用预处理指令#define声明一个常数，用以表明一年中有多少秒**
```
#define SECONDS_PER_YEAR (60 * 60 * 24 * 365)_U_LONG
这个问题可以简单考察对于 define 的基本使用,
如果意识到这个表达式将使一个 16 位机的整型数溢出-要用到长整型符号,告诉编译器这个常数是的长整型数。
如果你在你的表达式中用到 _U_LONG（表示无符号长整型),会更好.
另外如果是较早的答案会是 UL,但是目前已经由 _U_LONG 代替
还有一个类似的题目:写一个”标准"宏MIN ，这个宏输入两个参数并返回较小的一个。
#define MIN(A,B) ((A) <= (B) ? (A) : (B))    注意写法即可

```
**下面的代码输出结果是什么？**
```
@implementation Son : Father
- (id)init {
    self = [super init];
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
// 都是 son。
self 是类的隐藏参数，指向当前调用方法的这个类的实例。而 super 是一个 Magic Keyword，
它本质是一个编译器标示符，和 self 是指向的同一个消息接受者。
上面的例子不管调用 [self class] 还是 [super class]，接受消息的对象都是当前 Son ＊xxx 这个对象。
而不同的是，super 是告诉编译器，调用 class 这个方法时，要去父类的方法，而不是本类里的。
当使用 self 调用方法时，会从当前类的方法列表中开始找，如果没有，就从父类中再找；
而当使用 super 时，则从父类的方法列表中开始找。然后调用父类的这个方法。
```
**对于 self = [super init] 的思考**
```
首先 alloc 返回一个有效的未初始化的对象实例。对于 self 是 alloc 返回的指针，同时可以在所有的方法作用域内访问。
但是对于 super，它只是一个"编译器指示符",告诉编译器在父类中搜索方法的实现。
优先调用 [super init] 是为了使超类完成它们自己的初始化工作。
那么 if (self = [super init]) 又是做啥？
这里是担心父类初始化失败，如果初始化一个对象失败，就会返回 nil，当返回 nil 的时候 self = [super init] 测试的主体就不会再继续执行。
如果不这样做，你可能会操作一个不可用的对象，它的行为是不可预测的，最终可能会导致你的程序崩溃。
```
**单例写法**
```
+ (instancetype)sharedInstance {
    static CJManage *manage = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        manage = [[CJManage alloc] init];
    });
    return manage;
}
当然这种写法 alloc，new，copy，mutableCopy 方法不可以直接调用。个人认为没必要因为这就是单例就这样创建就好.
单例的问题:单例对象一旦建立，对象指针是保存在静态区的，单例对象在堆中分配的内存空间，会在应用程序终止后才会被释放。
单例类无法继承，因此很难进行类的扩展。另外还要注意循环引用问题.
```
**下列代码输出什么**
```
void test1() {
    int a = 10;
    void (^block)(void) = ^{
        NSLog(@"a is %d", a);
    };
    a = 20;
    block(); // 10
}
void test2(){
    __block int a = 10;
    void (^block)(void) = ^{
        NSLog(@"a is %d", a);
    };
    a = 20;
    block(); // 20
}
代码已给出答案,这是因为 block 所在函数中的，捕获自动变量。但是不能修改它，不然就是“编译错误”。
但是可以改变全局变量、静态变量、全局静态变量。
首先不能修改自动变量的值是因为：block 捕获的是自动变量的 const 值，名字一样，不能修改
可以修改静态变量的值：静态变量属于类的，不是某一个变量。由于 block 内部不用调用 self 指针。所以 block 可以调用。
因此 block 不能修改自动变量的值，可以使用 __block修饰符来解决。
```
**代码打印结果**
```
int a[5] = {1,2,3,4,5};
int *ptr = (int *)(&a+1);
NSLog(@"%d,%d",*(a + 1),*(ptr - 1));// 2,5
1.*(a+1)=a[0+1]=a[1]=2
2.&a 表示对数组取地址，&a+1 表示 a[5] 后面一个地址，(ptr-1) 表示对当前数组元素地址向前移动一位，并取值，故等于 a[4]=5
```
