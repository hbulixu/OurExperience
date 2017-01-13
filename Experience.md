1. 在进行大量数据操作的时候应尽量使用事务，这可以节约极大的节约操作时间。
2. 后端在操作完数据库返回给客户端的时候要加一层过滤，因为客户端可能会因为误操作而上传脏数据，这些数据如果再由服务端返回给客户端，那么就会产生错误。比如安卓产生了脏数据上传到了服务端，那么IOS端更新下来就可能产生意想不到的错误。
3. 方法的参数过长会造成调用和使用的麻烦，这时候可以将其封装成一个类，如`- (void)getCurrentPeriodData:(void(^)(CGFloat allAcutallIncome, CGFloat allOverWorkHour,CGFloat allOverWorkIncom, NSInteger overWorkDay, NSArray *statisticCellModels))currentMonthInforBlock`这个方法可以封装成`- (void)getCurrentPeriodData:(void(^)(WBOTMonthInfo * monthInfo))currentMonthInforBlock`。
4. 根据职责单一原则，一个类不应该承载过多的任务，所以如果原始数据无法满足页面展示的需求，那么我们需要再分离出一个类来做这些工作。
5. 如果有不同的cell需要展示，那么可以给cell的数据源绑定不同类型的identifier，然后利用多态实现。
6. 指定的初始化方法可以在后面加上`NS_DESIGNATED_INITIALIZER`，如:
 
       #ifndef NS_DESIGNATED_INITIALIZER
			#if __has_attribute(objc_designated_initializer)
			#define NS_DESIGNATED_INITIALIZER      __attribute__((objc_designated_initializer))
			#else
			#define NS_DESIGNATED_INITIALIZER
			#endif
			#endif - (instancetype)initWithSessionConfiguration:(nullable NSURLSessionConfiguration *)configuration NS_DESIGNATED_INITIALIZER;
			
7. 如果某个参数在之后的版本迭代中已经不适用了，那么就在其后面添加`DEPRECATED_ATTRIBUTE`，如:`FOUNDATION_EXPORT NSString * const AFNetworkingTaskDidFinishNotification DEPRECATED_ATTRIBUTE;`。
8. 好的设计应该是:如果需求变更那么更改的代码要集中在一个类，或者某个地方，而不影响其他类，这就需要将合适的方法放到合适的类中。

####第一期CodeReview&Refactor经验汇总
1. 数据和业务相隔离，数据库的封装：使用了链式编程，代码量减少了4/5，代码更加整洁，思路更加清晰，提取出了很多方法，减少了很多重复代码，借鉴YYCache的方法实现了异步操作数据库，具体代码展示如下:


2. 引导页解决方案:抽离出引导页基类，将统一的回调方法写在基类，遵循了面向对象依赖倒换原则:**高层模块不应该依赖于底层模块，两者都应该依赖于抽象。**这样再扩展引导页的时候只要增加一个基类并且实现其回调的protocal即可。
3. 数据库原始数据到Controller之间的Adapter，这遵循了面向对象的职责单一原则:**就一个类而言应该仅有一个引起它变化的原因。**
4. 数据库同步的解决方案有两种，一种是在不同的线程中加锁，一种是另开一个单独的线程在其中操作。第一种方式实现简单，但是消耗较大，因为每次请求数据库，系统都需要另开一个线程，而开线程是很消耗系统资源的，第二种方式实现较为复杂，需要每次来一个请求都需要`performSelector:onThread:withObject:waitUntilDone:`这个方法来将其放到新建的线程，这里需要将block作为参数传递，因为withObject只能传递一个参数，所以需要给NSObject添加分类来实现，具体实现方法见**NSObject+YYAdd**。操作步骤如下:
<img src="file:/Users/a58/Desktop/MarkDown/implementOnOneThread.png" width = 100% height = 30%>
5. cell不同类型的判断其实可以放到创建Model的时候(*因为数据源其实已经确定了不同cell的类型*)给不同的Model增加不同的Identifier再利用多态可以实现。


####第二期CodeReview&Refactor问题汇总

######1. 增加Adapter之后页面展示的Model和用户操作之后要保存到数据中的数据还要需要转换，这应该也是需要一层逻辑处理。
######2. 日历页面如果改成一个UICollectionView，那么每次滑动页面都需要重新刷新这个UICollectionView,这个新建日历的数据源，进行请求数据并赋值model过程中会有明显的闪动现象，影响用户体验。
######3.在工资设定页面我没有复用cell,将所有的cell都统一作为一个属性使用懒加载进行处理，有没有更好的方法？
######4.统计页面中的补贴项，扣款项，带缴费项和设置中的数据源是相同的，但是展示逻辑不一样，怎样进行复用？
######5.设计模式是对面向对象编程中各种编程原则在具体场景中的应用，怎样根绝不同的场景去选择设计模式，怎样将每种设计模式用一句话进行总结？
######6.IGListKit(FaceBook) DataSource

1. 工厂模式（简单工厂，工厂模式，抽象工厂）:解决同类对象的创建过程。
2. 策略模式:解决同类对象相似方法的实现，无序。
3. 装饰模式:解决某个对象一系列有序的方法。
4. 代理模式:需要限制对某个对象的访问，同时需要使用该对象的方法。
5. 原型模式:简化相同对象的创建过程。
6. 模板方法模式:不同对象对某个方法的实现不同。
7. 外观模式:对很多对象进行统一的操作并提供一致的接口。
8. 建造者模式:某个对象一系列方法顺序固定，但是各个方法的实现不同。
9. 观察者模式:很多对象根据某个对象的属性改变而改变。
10. 状态模式:某个对象在不同的状态下具有不同的方法。
11. 适配器模式:某个对像的接口不适合其他对象。
12. 备忘录模式:某个对象的行为需要被记录。
13. 组合模式:多个对象之间可以任意的组合。
14. 迭代器模式:需要遍历多个相似的对象。
15. 单例模式:某个对象全局只能出现一个。
16. 桥接模式:多个类，同时具有多种功能。
17. 命令模式:封装命令，将命令的调用者和执行者隔离开。
18. 职责链模式:某个对象的某个行为在不同的条件下有不同的处理方法。
19. 中介者模式:处理多个对象之间的关系。
20. 享元模式:多个对象可以共享核心功能和部分数据。
21. 解释器模式:
22. 访问者模式:多个种类稳定的对象对于不同状态的响应方法不同。(**状态可能会改变，但是种类是稳定的，一般不会改变的**)
####AFN中的疑点:
1. AFURLRequestSerialization为什么一个类要声明一个协议，然后自己遵守并且去实现呢，而不用公用的接口呢？
2. 为什么要实现系统的NSStream类。
