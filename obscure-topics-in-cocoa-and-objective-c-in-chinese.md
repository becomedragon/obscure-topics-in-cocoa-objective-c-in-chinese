NSHIPSTER

obscure topics in cocoa & objective-c

《objevtiv-c和cocoa中的逼格点》

Object－C

#＃paragma
＃pragma声明是OC语言中的标注技巧。虽然编程语言最初是被设计成跨编译器的，但是Xcode中使用＃pragma声明是违背于上述原则的。

在现代的编程语境中，＃pragma的作用是居于注释和源代码之间的。作为一个预处理指令，＃pragma是在编译时运行的，但是和其他的宏定义不同的是，＃pragma不会改变runtime的行为。＃pragma声明尝尝被用在两个地方：代码的整理和控制警告输出。

###代码整理
代码整理就像是个人卫生，你的代码风格能反映出你和你的工作风格。缺少良好的代码风格会给人感觉很不专业，这也将增大项目维护和协同开发的困难性。

一个良好的习惯应该从#pragma mark开始：

@implementation ViewController - (id)init {
...
  }
  ＃pragma mark - UIViewController - (void)viewDidLoad {
...
  }
  ＃pragma mark - IBAction
- (IBAction)cancel:(id)sender {
 ...
  }
  ＃pragma mark - UITableViewDataSource
- (NSInteger)tableView:(UITableView *)tableView 
numberOfRowsInSection:(NSInteger)section 
 {
 }
＃pragma mark - UITableViewDelegate
- (void)tableView:(UITableView *)tableView 
didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
 ...
  }
  在@implementation中使用#pragma mark能够将代码按照逻进行划分。该声明不但能让代码本身更加易于阅读，还能在Xcode的导航中表现出来。
  按照方法的实现类别可以将多个同属一类的方法看成是一个组。例如一个NSInputStream的子类可以看做是同属NSInputStream组的。
像是IBAction outlets，或者target/action,notification,KVO这几类的代码都可以归为各自的组别，将他们划分为不同的组。
如果一个类要实现某一个@protocol，则可以将该@protocol中的方法划分为一组，用#pragma mark进行标识。
###警告控制
比糟糕的代码风格更让人讨厌的是代码编译后所产生的各种警告，特别是第三方提供的代码。如果你看到某一个第三方代码编译之后产生了200+个警告，我想你一定会疯掉。
但是在某些情况下，没有办法避免警告的出现。使用早期版本的API和代码中可能会出现的循环引用都是让编译器提示警告，这两个情况也是非常常见的。在这时你是非常希望编译器不要将这些警告提示出来，#pragma就能起到屏蔽这些警告的作用：
   
    #pragma clang diagnostic push    #pragma clang diagnostic ignored "-Wunused-variable"    OSStatus status = SecItemExport(...);    NSCAssert(status == errSecSuccess, @"%d", status);    #pragma clang diagnostic pop
上面的代码是一些常用的用来屏蔽不可避免发生警告的例子。在release模式下，assertion代码会被编译器忽略，这时status变量就会产生unused variable警告，此时我们加上了#pragma clang diagnostic ignored "-Wunused-variable"这句声明就会让编译器屏蔽该警告。

使用#pragma clang diagnostic push/pop声明，是告诉编译器警告控制所作用的区域。

记住千万不要将编译器合理的警告给屏蔽掉，这样只会带来更大的麻烦，因为这会让你无法发觉潜在的隐患。


#nil/Nil/NULL/NSNull
理解“空”这个概念就是像是理解一个哲学问题那样，但这个“空”不是抽象的概念而是一个实实在在的东西。我们是由宇宙中的一些“东西”组合而成，这些“东西”看似合理却又存在很多不确定性。就像是一个合理的物理现象一样，计算机面临一个棘手的问题，如何用一个“东西”去表示“无”这个概念。

在Objevtive-C语言中，存在很多不同的形式用来表示“空”。

C语言中用0来表示数值的“空”，用NULL来标识指针的“空”。

OC在C的基础上增加了nil来表示“空”。nil表示一个对象为空，从语义上与NULL是有区别的，但是他们之间又是相等的。

在OC的底层框架中定义了一个NSNull的类，该类是一个单例类，通过+null方法返回该类的实例。NSNull与不同于nil和NULL，它是一个实实在在的对象，而不是为0的数值。

在Foundation/NSObjcRuntime.h文件中还定义了一个Nil用来表示一个类的指针为空。该标识没有nil那么的常见，但是Nil比nil表示一种更深层次的空。

###关于nil
一个新建的NSObject在生命周期的开始阶段其内容都为0，就是说所有的指针指向的都是nil。

也许最让人注意到的是，nil是可以处理消息调用的，[nil message]不会引起crash，因为nil可以处理该message。

这在其他语言中已经会引起crash，例如在c++中，像null发送消息就一定会引起crash，但是在OC中nil调用一个方法会返回0.这种特性将有助于简化代码的编写，在让一个对象调用方法之前不需要在对该对象做判断空的处理：


     // For example, this expression...      if (name != nil && [name isEqualToString:@"Steve"])   { ... }     !     // ...can be simplified to:      if ([name isEqualToString:@"steve"]) { ... }
在了解了nil这个特性之后，在给编码代码带来便利性的同时也要注意这种特性也会让bug更加难以的查出。

###NSNull：用来表示“空”的类
NSNull被用在很多的底层框架和其他的系统框架中，例如在NSArray和NSDictionary这种结合类中使用NSNull可以避开不能使用nil的限制。NSNull可以有效的将NULL或者nil封装成一个对象，这样nil就能被当做对象处理存储到集合类中。
     NSMutableDictionary *mutableDictionary = [NSMutableDictionary dictionary];    !     mutableDictionary[@"someKey"] = [NSNull null];   // Sets value of NSNull singleton for `someKey` !    NSLog(@"Keys: %@", [mutableDictionary allKeys]);   // @[@"someKey"]
下面将总结四种在OC中用来表示“空”的标识：

* NULL，  数值为： (void*)0，     在c语言中表示指针为空。
* nil ，  数值为： (id)0，        表示OC中的某一个对象指向空。
* Nil ，  数值为： (Class）0，    表示OC中的某一个类为空。
* NSNull，数值为： [NSNull null]  是OC中一个表示null的单例类。

#BOOL/bool/Boolean/NSCFBoolean
