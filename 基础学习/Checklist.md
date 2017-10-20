## Checklist

>### 性能调优
1. [iOS 性能监控方案 Wedjat 之基础性能篇](https://github.com/aozhimin/iOS-Monitor-Platform/blob/master/iOS-Monitor-Platform_Basic.md)
2. [iOS 性能监控方案 Wedjat 之网络篇](https://github.com/aozhimin/iOS-Monitor-Platform/blob/master/iOS-Monitor-Platform_Network.md)

>### Algorithm

1. [leecode 题解][4]

>### 项目经验

1. 音视频基础和总结
2. OpenGL
    
    * [obj 文件解析](http://blog.csdn.net/szchtx/article/details/8628265)
    
    * [几何变换](http://www.cnblogs.com/graphics/archive/2012/08/08/2609005.html)
    
>### Blog 

1. [iOSInterviewQuestions][1]
2. [OpenGL 学习][2]
3. [YUV格式][3]
4. [iOS开发下的函数响应式编程][5]

>### Interview iOS

1. 上来让介绍各种属性，其实挺基础的，讲到strong和Retain就傻逼了，两个都是强引用，然后我说strong更偏向自己生成的强引用，其实它们的区别只是ARC和非ARC情况下的强引用，然后面试官不依不饶盯住这个点说Retain就不是自己生成的了？纠结了有一会，此处已扣分，然后其他属性就没说了(真他妈该先介绍其他属性的)

2. copy属性的理解，我说会利用原对象是返回一个不可变的引用，然后他写了个语句，生成一个数组赋给一个copy属性的变量，让我说这个过程怎么发生，我说利用这个数组去生成一个不可变的数组然后赋给变量，当时真没考虑到不可变的数组是直接引用过去，不会生成新的实例，没想到他考的是这个点，然后这个地方也是扣分了

3. 问了个具体怎么实现MVVM的，我胡扯一通

4. NSOperation和GCD的区别，答的还行

5. 三个回调方式的区别

6. 图片缓存，说用的SDWebImage，说了下SDWebImage的策略

7. 说下对RunTime的理解

8. 当修改属性的setter方法，还能对int类型的属性用kvo进行监听么

9. 对HTTP的理解，只说了GET，POST

10. 强引用循环怎么解决

11. autorelease释放时机

12. [※]@property中有哪些属性关键字？

13. [※]weak属性需要在dealloc中置nil么？

14. [※※]@synthesize和@dynamic分别有什么作用？

15. [※※※]ARC下，不显式指定任何属性关键字时，默认的关键字都有哪些？

16. [※※※]用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？

17. [※※※]@synthesize合成实例变量的规则是什么？假如property名为foo，存在一个名为_foo的实例变量，那么还会自动合成新变量么？

18. [※※※※※]在有了自动合成属性实例变量之后，@synthesize还有哪些使用场景？

19. [※※]objc中向一个nil对象发送消息将会发生什么？

20. [※※※]objc中向一个对象发送消息[obj foo]和objc_msgSend()函数之间有什么关系？

21. [※※※]什么时候会报unrecognized selector的异常？

22. [※※※※]一个objc对象如何进行内存布局？（考虑有父类的情况）

23. [※※※※]一个objc对象的isa的指针指向什么？有什么作用？

24. [※※※※]下面的代码输出什么？

```
@implementation Son : Father
- (id)init
{
    self = [super init];
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```

25. [※※※※]runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）
26. [※※※※]使用runtime Associate方法关联的对象，需要在主对象dealloc的时候释放么？
27. [※※※※※]objc中的类方法和实例方法有什么本质区别和联系？

28. [※※※※※]_objc_msgForward函数是做什么的，直接调用它将会发生什么？

29. [※※※※※]runtime如何实现weak变量的自动置nil？

30. [※※※※※]能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

31. [※※※]runloop和线程有什么关系？

32. [※※※]runloop的mode作用是什么？

33. [※※※※]以+ scheduledTimerWithTimeInterval...的方式触发的timer，在滑动页面上的列表时，timer会暂定回调，为什么？如何解决？

34. [※※※※※]猜想runloop内部是如何实现的？

35. [※]objc使用什么机制管理对象内存？

36. [※※※※]ARC通过什么方式帮助开发者管理内存？

37. [※※※※]不手动指定autoreleasepool的前提下，一个autorealese对象在什么时刻释放？（比如在一个vc的viewDidLoad中创建）

38. [※※※※]BAD_ACCESS在什么情况下出现？

39. [※※※※※]苹果是如何实现autoreleasepool的？

40. [※※]使用block时什么情况会发生引用循环，如何解决？

41. [※※]在block内如何修改block外部变量？

42. [※※※]使用系统的某些block api（如UIView的block版本写动画时），是否也考虑引用循环问题？

43. [※※]GCD的队列（dispatch_queue_t）分哪两种类型？

44. [※※※※]如何用GCD同步若干个异步调用？（如根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图）

45. [※※※※]dispatch_barrier_async的作用是什么？

46. [※※※※※]苹果为什么要废弃dispatch_get_current_queue？

47. [※※※※※]以下代码运行结果如何？

```
- (void)viewDidLoad
{
    [super viewDidLoad];
    NSLog(@"1");
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"2");
    });
    NSLog(@"3");
}
```

48. [※※]addObserver:forKeyPath:options:context:各个参数的作用分别是什么，observer中需要实现哪个方法才能获得KVO回调？
49. [※※※]如何手动触发一个value的KVO
50. [※※※]若一个类有实例变量NSString *_foo，调用setValue:forKey:时，可以以foo还是_foo作为key？

51. [※※※※]KVC的keyPath中的集合运算符如何使用？

52. [※※※※]KVC和KVO的keyPath一定是属性么？

53. [※※※※※]如何关闭默认的KVO的默认实现，并进入自定义的KVO实现？

54. [※※※※※]apple用什么方式实现对一个对象的KVO？

55. [※※]IBOutlet连出来的视图属性为什么可以被设置成weak?

56. [※※※※※]IB中User Defined Runtime Attributes如何使用？

57. [※※※]如何调试BAD_ACCESS错误

58. [※※※]lldb（gdb）常用的调试命令？

[1]: https://github.com/ChenYilong/iOSInterviewQuestions
[ 2 ]: https://learnopengl-cn.github.io/01%20Getting%20started/06%20Textures/
[3]: http://blog.csdn.net/beyond_cn/article/details/12998247
[4]: https://siddontang.gitbooks.io/leetcode-solution/content/
[5]: http://williamzang.com/blog/2016/06/27/ios-kai-fa-xia-de-han-shu-xiang-ying-shi-bian-cheng/
