
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201230192816453.png)
# 1 Spring核心组件
一句话概括Spring：**Spring是一个轻量级、非入侵式的控制反转(IoC)和面向切面(AOP)的框架**。
| Spring Framework版本	  | JDK版本 |
|--|--|
| 1.x | 1.3：引入了动态代理机制，AOP 底层就是动态代理，所以 Spring 必须是 JDK 1.3 |
| 2.x |   [1.4](https://blog.csdn.net/qq_39739850/article/details/97798617)：正常升级
| 3.x |  5： 引入注解，Spring 3 最低版本是 Java 5 ,从此以后不叫1.x 直接叫x
| 4.x |  6： **Spring 4 是有划时代意义的版本**，开始支持 Spring Boot 1.X
| 5.x |  8：lambda 表达式等功能
PS ：目前Java 开发的标配是 Spring Framwork 5 + Spring Boot 2 + JDK 8
### 1.1 Spring 简介
现如今的Java开发又简称为Spring开发，Spring是Java目前第一大框架，它让现有的技术更容易使用，促进良好的编程习惯，大大简化应用程序的开发。因为你想啊，如果我们想实现某个功能，代码量一般都是固定的，要么全自己写，要么用已有的优秀框架，而Spring不仅已经给我们提供了各种优秀组件，还提供了良好的代码组织逻辑跟业务开发流程规范框架，它的主要优点如下：
1. IOC跟DI的支持
> Spring就是一个大工厂容器，可以将所有对象创建和依赖关系维护，交给Spring管理，Spring工厂是用于生成Bean，并且管理Bean的生命周期，实现**高内聚低耦合**的设计理念。

2. AOP编程的支持
> Spring提供**面向切面编程**，可以方便的实现对程序进行权限拦截、运行监控等功能。

3. 声明式事务的支持
> 只需要通过配置就可以完成对事务的管理，而无需手动编程，以前重复的一些JDBC操作，统统不需我们再写了。

4. 方便程序的测试
> Spring对Junit4提供支持，可以通过**注解**方便的测试Spring程序。

5. 粘合剂功能
> 方便集成各种优秀框架，Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架（如：Struts、Hibernate、MyBatis、Quartz等）的直接支持。

6. 降低 JavaEE API的使用难度
> Spring 对 JavaEE 开发中非常难用的一些API（JDBC、JavaMail、远程调用等）都提供了封装，这些API的提供使得应用难度大大降低。

### 1.2 Spring组件
Spring框架是分模块存在，除了最核心的`Spring Core Container`(即Spring容器)是必要模块之外，其他模块都是`可选`，大约有20多个模块。
![在这里插入图片描述](https://img-blog.csdnimg.cn/202004062040229.png)

[Spring框架](https://sowhat.blog.csdn.net/article/details/105340337) 有很多特性，这些特性由7个定义良好的模块构成。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200406215608153.png)
> 1. **Spring Core**：Spring核心，它是框架最基础的部分，提供IOC和依赖注入DI特性。
>2. **Spring Context**：Spring上下文容器，它是 BeanFactory 功能加强的一个子接口。
>3. **Spring Web**：它提供Web应用开发的支持。
>4. **Spring MVC**：它针对Web应用中MVC思想的实现。
>5. **Spring DAO**：提供对JDBC抽象层，简化了JDBC编码，同时，编码更具有健壮性。
>6. **Spring ORM**：它支持用于流行的ORM框架的整合，比如：Spring + Hibernate、Spring + iBatis、Spring + JDO的整合等。
>7.  **Spring AOP**：即面向切面编程，它提供了与AOP联盟兼容的编程实现。

# 2  IOC 跟 AOP
提到Spring永远离不开的两个话题就是 [IOC跟AOP](https://sowhat.blog.csdn.net/article/details/104491509)，这应该是Spring的两大核心知识点了，初学者不要被IOC、AOP、Aspect、Pointcut、Advisor这些术语吓着了，这些术语都是无聊的人为了发论文硬造的。
### 2.1 IOC
Java是个面向对象的编程语言，一般一个应用程序是由一组对象通过相互协作开发出的业务逻辑组成的，那么如何管理这些对象呢？抽象工厂、工厂方法设计模式可以帮我们创建对象，生成器模式帮我们处理对象间的依赖关系，可是这些又需要我们创建另一些工厂类、生成器类，我们又要而外管理这些类，增加了我们的负担。如果程序在对象需要的时候，就能自动管理对象的声明周期，不用我们自己再去管理Bean的声明周期了，这样不就实现解耦了么。

Spring提出了一种思想：就是由`Spring`来负责控制对象的生命周期和对象间的关系。所有的类都会在Spring容器中登记，告诉`Spring`是什么，需要什么，然后Spring会在系统运行到适当的时候，把你要的东西`主动`给你，同时也把你交给`其他`需要你的东西。所有的类的创建、销毁都由` Spring`来控制，也就是说控制对象生存周期的不再是引用它的对象，而是Spring。对于某个具体的对象而言，**以前是它控制其他对象，现在是所有对象都被spring控制**，所以这叫控制反转(Inversion of Controller)，也可以叫依赖注入 DI(Dependency Injection)。

知道大致思想后其实可以如果尝试[自己实现IOC](https://mp.weixin.qq.com/s/YFxL6juq-9WXO7xFFAfXOw)的话就会发现核心就是 **反射** + **XML解析/注解解析** 。
> 1. 读取 XML 获取 bean 相关信息，类信息、属性值信息。
> 2. 通过`反射机制`获取到目标类的构造函数，调用构造函数，再给对象赋值。

如果想自己跟下源码你会发现[IOC的源码](https://mp.weixin.qq.com/s/e1-tRXLCfAGRiFXkhTs8vQ)入口是`refresh()`，里面包含13个提供不同功能的函数，具体流程较复杂，公众号回复 **IOC** 获得原图。
### 2.2 Context 
**IOC**容器只是提供一个管理对象的空间而已，如何向容器中放入我们需要容器代为管理的对象呢？这就涉及到Spring的应用上下文**Context**。

**应用上下文Context** :
> 基于 Core 和 Beans，提供了大量的扩展，包括国际化操作（基于 JDK ）、资源加载（基于 JDK properties）、数据校验（Spring 自己封装的数据校验机制）、数据绑定（Spring 特有，HTTP 请求中的参数直接映射称 POJO）、类型转换，ApplicationContext 接口是 Context 的核心，可以理解为Bean的**上下文或背景信息**。

可以简单的理解 [应用上下文](https://sowhat.blog.csdn.net/article/details/60140785) 是Spring容器的一种抽象化表述，而我们常见的 **ApplicationContext** 本质上就是一个维护Bean定义以及对象之间协作关系的高级接口。Spring 框架本身就提供了很多个容器的实现，大概分为两种类型：
1. 一种是不常用的**BeanFactory**，这是最简单的容器，只能提供基本的**DI**功能。
2. 另外一种就是继承了**BeanFactory**后派生而来的应用上下文，其抽象接口也就是我们上面提到的的**ApplicationContext**，它能提供更多企业级的服务，例如解析配置文本信息等等，这也是应用上下文实例对象最常见的应用场景。有了上下文对象，我们就能向容器注册需要Spring管理的对象了。对于上下文抽象接口，Spring也为我们提供了多种类型的容器实现，供我们在不同的应用场景选择。
>1. **AnnotationConfigApplicationContext**：从一个或多个基于java的配置类中加载上下文定义，适用于java注解的方式。
>2. **ClassPathXmlApplicationContext**：从类路径下的一个或多个xml配置文件中加载上下文定义，适用于xml配置的方式。
>3. **FileSystemXmlApplicationContext**：从文件系统下的一个或多个xml配置文件中加载上下文定义，也就是说系统盘符中加载xml配置文件。
>4. **AnnotationConfigWebApplicationContext**：专门为web应用准备的，适用于注解方式。
>5. **XmlWebApplicationContext**：从web应用下的一个或多个xml配置文件加载上下文定义，适用于xml配置方式。

工作中通过XML配置或注解 将需要管理的Bean跟Bean之间的协作关系配置好，然后利用应用上下文对象Context加载进Spring容器，容器就能为你的程序提供你想要的对象管理服务了。比如追踪下 **ClassPathXmlApplicationContext** 的底层源码：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122909272469.png)
可以看到一个XML文件的解析就可以上延8层，可见Spring容器为了实现IOC进行了全面性的考虑。
### 2.3 AOP
如果想编码实现计算器功能，我们的目标是实现加减乘除的运算，可是如何在每种运算前后进行打印日志跟数字合规的校验呢。
> 1. 把`日志记录`和`数据校验`可重用的功能模块分离出来，然后在程序的执行的合适的地方动态地植入这些代码并执行。这样就简化了代码的书写。
> 2. 业务逻辑代码中没有参和通用逻辑的代码，业务模块更简洁，只包含核心业务代码。实现了业务逻辑和通用逻辑的代码分离，便于维护和升级，降低了业务逻辑和通用逻辑的耦合性。
 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228154117468.png)
**思路**：代码最终是要加载到内存中实现new出对象，那么如果我们把可重用的功能提取出来，然后将这些通用功能在内存中通过入的方式实现构造出一个新的目标对象不就OK了么！

Spring AOP(Aspect Oriented Programming) 恰恰提供从另一个角度来考虑程序结构以完善面向对象编程，如果说依赖注入的目的是让相互协作的组件保持一种较为松散的耦合状态的话，AOP则是将遍布应用各处的功能分离出来形成可重用的组件。在编译期间、装载期间或运行期间实现在不修改源代码的情况下给程序动态添加功能的一种技术。从而实现对业务逻辑的隔离，提高代码的模块化能力。

AOP 的核心其实就是**动态代理**，如果是实现了接口的话就会使用 JDK 动态代理，否则使用 CGLIB 代理，主要应用于处理一些具有横切性质的系统级服务，如日志收集、事务管理、安全检查、缓存、对象池管理等。

Spring主要提供了 Aspect 切面、JoinPoint 连接点、PointCut 切入点、Advice 增强等实现方式，[AOP](https://blog.csdn.net/ethan_199402/article/details/109491789)一般有**5种**环绕方式：
>1. 前置通知 (@Before) 
>1. 返回通知 (@AfterReturning) 
>1. 异常通知 (@AfterThrowing) 
>1. 后置通知 (@After)
>1. 环绕通知 (@Around) :（优先级最高）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210104095628790.png)
PS :多个切面的情况下，可以通过@Order指定先后顺序，数字越小，优先级越高
# 3  JDK 动态代理和 CGLIB 代理区别
JDK动态代理与CGLib动态代理均是实现Spring AOP的基础，它们的实现方式有所不同。
### 3.1 JDK动态代理
特点
>1. **Interface**：对于JDK动态代理，业务类需要一个Interface。
>2. **Proxy**：Proxy类是动态产生的，这个类在调用 Proxy.newProxyInstance() 方法之后，产生一个Proxy类的实例。实际上，这个Proxy类也是存在的，不仅仅是类的实例，这个Proxy类可以保存在硬盘上。
>3. **Method**：对于业务委托类的每个方法，现在Proxy类里面都不用静态显示出来。
>4. **InvocationHandler**：这个类在业务委托类执行时，会先调用invoke方法。invoke方法在执行想要的代理操作，可以实现对业务方法的再包装。

总结：
>1. JDK动态代理类实现了**InvocationHandler**接口，重写的**invoke**方法。
> 2. JDK动态代理的基础是反射机制（method.invoke(对象，参数)）Proxy.newProxyInstance()

### 3.2 CGLib动态代理
**特点**：
>1. 使用字节码处理框架**ASM**，其原理是通过**字节码**技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。
> 2. **CGLib**创建的动态代理对象性能比JDK创建的动态代理对象的性能高不少，但是CGLib在创建代理对象时所花费的时间却比JDK多得多，所以对于单例的对象，因为无需频繁创建对象，用CGLib合适，反之，使用JDK方式要更为合适一些。同时，由于CGLib由于是采用动态创建子类的方法，对于final方法，无法进行代理。

**注意**：
>JDK的[动态代理](https://mp.weixin.qq.com/s/7YYcSkdhJMrvD9We9dsMNA)`只可以`为接口去完成操作，而 Cglib 它既可以为没有实现接口的类去做代理，也可以为实现接口的类去做代理。
### 3.3 代码实现部分
**公共代码** ：
```java
//接口类
public interface FoodService {
    public void makeNoodle();
    public void makeChicken();
}
```
```java
//实现接口
public class FoodServiceImpl implements FoodService {
    @Override
    public void makeNoodle() {
        System.out.println("make noodle");
    }

    @Override
    public void makeChicken() {
        System.out.println("make Chicken");
    }
}
```
**jdk动态代理代码**：
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class JDKProxyFactory implements InvocationHandler
{
	private Object target;

	public JDKProxyFactory(Object target)
	{
		super();
		this.target = target;
	}

	// 创建代理对象
	public Object createProxy()
	{
		// 1.得到目标对象的类加载器
		ClassLoader classLoader = target.getClass().getClassLoader();
		// 2.得到目标对象的实现接口
		Class<?>[] interfaces = target.getClass().getInterfaces();
		// 3.第三个参数需要一个实现invocationHandler接口的对象
		Object newProxyInstance = Proxy.newProxyInstance(classLoader, interfaces, this);
		return newProxyInstance;
	}

	// 第一个参数:代理对象.一般不使用;第二个参数:需要增强的方法;第三个参数:方法中的参数
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable
	{
		System.out.println("这是增强方法前......");
		Object invoke = method.invoke(target, args);
		System.out.println("这是增强方法后......");
		return invoke;
	}

	public static void main(String[] args)
	{
		// 1.创建对象
		FoodServiceImpl foodService = new FoodServiceImpl();
		// 2.创建代理对象
		JDKProxyFactory proxy = new JDKProxyFactory(foodService);
		// 3.调用代理对象的增强方法,得到增强后的对象
		FoodService createProxy = (FoodService) proxy.createProxy();
		createProxy.makeChicken();
	}
}
```
**Cglib动态代理代码**：
```java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

public class CglibProxyFactory implements MethodInterceptor
{
    //得到目标对象
    private Object target;
    //使用构造方法传递目标对象
    public CglibProxyFactory(Object target) {
        super();
        this.target = target;
    }
    //创建代理对象
    public Object createProxy(){
        //1.创建Enhancer
        Enhancer enhancer = new Enhancer();
        //2.传递目标对象的class
        enhancer.setSuperclass(target.getClass());
        //3.设置回调操作
        enhancer.setCallback(this);
        return enhancer.create();
    }

    //参数一:代理对象;参数二:需要增强的方法;参数三:需要增强方法的参数;参数四:需要增强的方法的代理
	@Override
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("这是增强方法前......");
        Object invoke = methodProxy.invoke(target, args);
        System.out.println("这是增强方法后......");
        return invoke;
    }

    public static void main(String[] args) {
        // 1.创建对象
        FoodServiceImpl foodService = new FoodServiceImpl();
        // 2.创建代理对象
        CglibProxyFactory proxy = new CglibProxyFactory(foodService);
        // 3.调用代理对象的增强方法,得到增强后的对象
        FoodService createProxy = (FoodService) proxy.createProxy();
        createProxy.makeChicken();
    }
}
```
# 4. Spring AOP 和 AspectJ AOP区别
### 4.1 Spring AOP
Spring AOP 属于`运行时增强`，主要具有如下特点：
1. 基于动态代理来实现，默认如果使用接口的，用JDK提供的动态代理实现，如果是方法则使用CGLIB实现
2. Spring AOP 需要依赖 IOC 容器来管理，并且只能作用于Spring容器，使用纯Java代码实现
3. 在性能上，由于Spring AOP是基于**动态代理**来实现的，在容器启动时需要生成代理实例，在方法调用上也会增加栈的深度，使得Spring AOP的性能不如AspectJ的那么好。
4. Spring AOP致力于解决企业级开发中最普遍的AOP(方法织入)。
### 4.2 AspectJ
[AspectJ](https://www.cnblogs.com/chaoesha/p/13037368.html) 是一个易用的功能强大的AOP框架，属于`编译时增强`，  可以单独使用，也可以整合到其它框架中，是 AOP 编程的完全解决方案。AspectJ需要用到单独的编译器ajc。

AspectJ属于**静态织入**，通过修改代码来实现，在实际运行之前就完成了织入，所以说它生成的类是没有额外运行时开销的，一般有如下几个织入的时机：
1. 编译期织入（Compile-time weaving）： 如类 A 使用 AspectJ 添加了一个属性，类 B 引用了它，这个场景就需要编译期的时候就进行织入，否则没法编译类 B。
2. 编译后织入（Post-compile weaving）： 也就是已经生成了 .class 文件，或已经打成 jar 包了，这种情况我们需要增强处理的话，就要用到编译后织入。
3. 类加载后织入（Load-time weaving）： 指的是在加载类的时候进行织入，要实现这个时期的织入，有几种常见的方法
### 4.3 对比
| Spring AOP | AspectJ |
|--|--|
| 在纯Java中实现 |用Java编程语言扩展实现  |
| 编译器javac | 一般需要ajc |
|  只可运行时织入| 支持编译时、编译后、加载时织入 |
| 仅支持方法级编织 | 可编织字段、方法、构造函数、静态初始值等 |
| 只可在spring管理的Bean上实现 | 可在所有域对象实现 |
|仅支持方法执行切入点  | 支持所有切入点 |
|  比AspectJ 慢很多| 速度比AOP快很多 |
| 易学习使用 | 比AOP更复杂 |
| 代理由目标对象创建，切面应用在代理上 | 执行程序前，各方面直接织入代码中 |

# 5.  BeanFactory 和 FactoryBean 
### 5.1 BeanFactory
1. **BeanFactory** 以 Factory 结尾，表示它是一个工厂类(接口)，BeanFacotry 是 Spring 中比较原始的Factory。
2. **BeanFactory** 无法支持 Spring 的许多插件，如AOP功能、Web应用等。**ApplicationContext** 接口由BeanFactory接口派生而来，提供了国际化访问、事件传播等多个功能。
3. **BeanFactory** 是 IOC 容器的核心，负责生产和管理 Bean 对象。
### 5.2 FactoryBean
1. **FactoryBean** 以 Bean 结尾，表示它是一个Bean。
2. **FactoryBean** 是工厂类接口，用户可以通过实现该接口定制实例化 Bean 的逻辑。FactoryBean 接口对于 Spring 框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现。
3. 当在IOC容器中的Bean实现了 [FactoryBean](https://www.cnblogs.com/aspirant/p/9082858.html) 后，通过getBean(String BeanName)获取到的 Bean 对象并不是 FactoryBean 的实现类对象，而是这个实现类中的 getObject() 方法返回的对象。要想获取FactoryBean的实现类，就要 getBean(String &BeanName)，在BeanName之前加上 **&**。
# 6. Spring生命周期
Spring IOC 初始化跟销毁 Bean 的过程大致分为Bean定义、Bean初始化、Bean的生存期 跟 Bean的销毁4个部分。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228203152476.png)

如果仅仅是实例化跟依赖注入当然简单，问题是如果我们要完成**自定义**的要求，Spring提供了一系列接口跟配置来完成 Bean 的初始化过程，看下整个IOC容器[初始化Bean的流程](https://www.cnblogs.com/javazhiyin/p/10905294.html)。

一般情况下我们自定义Bean的初始化跟销毁方法下面三种：
1. 通过 XML 或者 @Bean配置
> 通过`xml`或者`@Bean(initMethod="init", destroyMethod="destory")`来实现。
2. 使用 [JSR250](https://wiki.jikexueyuan.com/project/spring/annotation-based-configuration/spring-jsr250-annotation.html) 规则定义的(java规范)两个注解来实现
>  1. `@PostConstruct`: 在Bean创建完成，且属于赋值完成后进行初始化，属于JDK规范的注解。
  > 2. `@PreDestroy`: 在bean将被移除之前进行通知，在容器销毁之前进行清理工作。
> 3. `提示`： JSR是由JDK提供的一组规范。
3. 通过继承实现类方法
> 1. 实现`InitializingBean`接口的`afterPropertiesSet()`方法，当`beanFactory`创建好对象，且把bean所有`属性设置好`之后会调这个方法，相当于初始化方法。
> 2. 实现`DisposableBean`的`destory()`方法，当bean销毁时会把单实例bean进行销毁
>3. 对于`单`实例的bean，可以正常调用初始化和销毁方法。 对于`多`实例的bean，容器只负责调用时候初始化，但不会管理bean， 容器关闭时不会调用销毁方法。
    
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200408190739445.png)
# 7. Spring中的设计模式
 Spring 框架中广泛使用了不同类型的[设计模式](https://mp.weixin.qq.com/s/leULkF969IAWuJW9KvfGEg)，下面我们来看看到底有哪些设计模式?
1. **工厂设计模式** : Spring 使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象。
2. **代理设计模式** : Spring AOP 功能的实现。
3. **单例设计模式** : Spring 中的 Bean 默认都是单例的。
3. **模板方法模式** : Spring 中 jdbcTemplate、hibernateTemplate 等以 **Template** 结尾的对数据库操作的类，它们就使用到了模板模式。
4. **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
5. **观察者模式**: Spring 事件驱动模型就是观察者模式很经典的一个应用。
6. **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller。

# 8. Spring循环依赖
### 8.1 简说循环依赖
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201229133643512.png)

[Spring循环依赖](https://mp.weixin.qq.com/s/Y4xCFTphtc_NectWB005ew)：说白了就是一个或多个对象实例之间存在直接或间接的依赖关系，这种依赖关系构成了构成一个**环形调用**。发生循环依赖的两个**前提条件**是：
>1. 出现循环依赖的Bean必须要是单例(**singleton**)，如果依赖**prototype**则完全不会有此需求。
> 2. 依赖注入的方式不能全是构造器注入的方式，只能解决setter方法的循环依赖，这是错误的。

假设AB之间相互依赖，通过尝试不同的注入方式注入后可的如下结论：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201229121617531.png)
PS：第四种可以而第五种不可以的原因是 Spring在创建Bean时默认会根据自然排序进行创建，所以A会先于B进行创建。
### 8.2 循环依赖通俗说
Spring通过**三级缓存**解决了循环依赖。
1. 一级缓存 : Map<String,Object> **singletonObjects**，单例池，用于保存实例化、注入、初始化完成的bean实例
2. 二级缓存 : Map<String,Object> **earlySingletonObjects**，早期曝光对象，用于保存实例化完成的bean实例
3. 三级缓存 : Map<String,ObjectFactory<?>> **singletonFactories**，早期曝光对象工厂，用于保存bean创建工厂，以便于后面扩展有机会创建代理对象。

当A、B两个类发生循环引用时，在A完成实例化后，就使用实例化后的对象去创建一个对象工厂，并添加到三级缓存中，如果A被AOP代理，那么通过这个工厂获取到的就是A代理后的对象，如果A没有被AOP代理，那么这个工厂获取到的就是A实例化的对象。当A进行属性注入时，会去创建B，同时B又依赖了A，所以创建B的同时又会去调用getBean(a)来获取需要的依赖，此时的getBean(a)会从缓存中获取：
>1. 第一步，先获取到三级缓存中的工厂。
>2. 第二步，调用对象工工厂的getObject方法来获取到对应的对象，得到这个对象后将其注入到B中。紧接着B会走完它的生命周期流程，包括初始化、后置处理器等。

当B创建完后，会将B再注入到A中，此时A再完成它的整个生命周期。至此[循环依赖](https://mp.weixin.qq.com/s/VpCt49_Li35caK5IaQTuNg)结束！
### 8.2 三级缓存意义何在？
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201229155849448.png)

先跟踪下源码(如上图)，跟踪过程中注意区别下**有AOP的依赖**跟**没有AOP的依赖**两种情况，跟踪后你会发现三级缓存的功能是只有真正发生循环依赖的时候，才去提前生成代理对象，否则只会创建一个工厂并将其放入到三级缓存中，但是不会去通过这个工厂去真正创建对象。至于提速这一说法，还是拉闸吧。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201229154258383.png)
如上图所示，如果使用二级缓存解决循环依赖，意味着所有Bean在实例化后就要完成AOP代理，这样违背了Spring设计的原则，Spring在设计之初就是通过AnnotationAwareAspectJAutoProxyCreator这个后置处理器来在Bean生命周期的最后一步来完成AOP代理，而不是在实例化后就立马进行AOP代理。
# 9. Spring事务
Spring 事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。Spring只提供统一事务管理接口，具体实现都是由各数据库自己实现，数据库事务的提交和回滚是通过binlog或者undolog实现的，具体流程在[MySQL](https://mp.weixin.qq.com/s/O_NHjv_YVUi4lSqXnhx5Mg)中讲过了。Spring会在事务开始时，根据当前环境中设置的隔离级别，调整数据库隔离级别，由此保持一致。
### 9.1  Spring事务的种类
Spring 支持`编程式事务`管理和`声明式`事务管理两种方式：
1. 编程式事务
> 编程式事务管理使用TransactionTemplate。
2. 声明式事务
> 1. 声明式事务管理建立在AOP之上的。其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前启动一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。
>2. 优点是不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明或通过@Transactional注解的方式，便可以将事务规则应用到业务逻辑中，减少业务代码的污染。唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。
### 9.2 Spring的事务传播机制
spring事务的传播机制说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为。事务传播机制实际上是使用简单的ThreadLocal实现的，所以，如果调用的方法是在新线程调用的，事务传播实际上是会失效的。
1. **propagation_requierd**：如果当前没有事务，就新建一个事务，如果已存在一个事务中，加入到这个事务中，这是最常见的选择，也是默认模式，它适合于绝大多数情况。
2. **propagation_supports**：支持当前事务，如果没有当前事务，就以非事务方法执行。
3. **propagation_mandatory**：使用当前事务，如果没有当前事务，就抛出异常。
4. **propagation_required_new**：新建事务，如果当前存在事务，把当前事务挂起。
5. **propagation_not_supported**：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
6. **propagation_never**：以非事务方式执行操作，如果当前事务存在则抛出异常。
7. **propagation_nested**：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与propagation_required类似的操作
### 9.2 Spring的事务隔离级别
TransactionDefinition接口中定义了五个表示隔离级别的常量，这里其实关键还是要看[MySQL](https://mp.weixin.qq.com/s/O_NHjv_YVUi4lSqXnhx5Mg)的隔离级别：
1. TransactionDefinition.ISOLATION_DEFAULT：使用后端数据库默认的隔离界别，MySQL默认可重复读，Oracle默认读已提交。
2. TransactionDefinition.ISOLATION_READ_UNCOMMITTED：读未提交
3. TransactionDefinition.ISOLATION_READ_COMMITTED：读已提交
4. TransactionDefinition.ISOLATION_REPEATABLE_READ：可重复读
5. TransactionDefinition.ISOLATION_SERIALIZABLE：串行化。
# 10. Spring MVC
### 10.1 什么是 MVC ？
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201229194850493.png)
MVC模式中M是指业务模型，V是指用户界面，C则是控制器，使用MVC的目的是将 M 和 V 的实现代码分离，开发中一般将应用程序分为 Controller、Model、View 三层，Controller 接收客户端请求，调用 Model 生成业务数据，传递给 View，View最终展示前端结果。

Spring MVC 就是对上述这套流程的封装，屏蔽了很多底层代码，开放出接口，让开发者可以更加轻松、便捷地完成基于 MVC 模式的 Web 开发。
### 10.2 Spring 跟 Spring MVC关系
最开始只有Spring，只提供IOC跟AOP核心功能，后来出了乱七八糟的比如MVC，Security，Boot等。原来的Spring就变成了现在的Spring Core，MVC指的是Web的MVC框架。
>1. **Spring MVC** 就是一个MVC框架，其实大范围上来说属于Spring，Spring MVC是一个类似于Struts的MVC模式的WEB开发框架，Spring MVC是基于 Spring 功能之上添加的 Web 框架，Spring 跟 SpringMVC可以理解为父子容器的关系，想用 Spring MVC 必须先依赖Spring。 
>2. **Spring MVC** 是控制层，用来接收前台传值，调用service层和持久层，返回数据再通过 Spring MVC把数据返回前台

### 10.3 Spring MVC 的核心组件
> 1.  **DispatcherServlet**：前置控制器，是整个流程控制的**核心**，控制其他组件的执行，进行统一调度，降低组件之间的耦合性，相当于总指挥。
> 2.  **Handler**：处理器，完成具体的业务逻辑，相当于 Servlet 或 Action。
> 3.  **HandlerMapping**：DispatcherServlet 接收到请求之后，通过 HandlerMapping 将不同的请求映射到不同的 Handler。
> 4.  **HandlerInterceptor**：处理器拦截器，是一个接口，如果需要完成一些拦截处理，可以实现该接口。
> 5.  **HandlerExecutionChain**：处理器执行链，包括两部分内容：Handler 和 HandlerInterceptor（系统会有一个默认的 HandlerInterceptor，如果需要额外设置拦截，可以添加拦截器）。
> 6.  **HandlerAdapter**：处理器适配器，Handler 执行业务方法之前，需要进行一系列的操作，包括表单数据的验证、数据类型的转换、将表单数据封装到 JavaBean 等，这些操作都是由 HandlerApater 来完成，开发者只需将注意力集中业务逻辑的处理上，DispatcherServlet 通过 HandlerAdapter 执行不同的 Handler。
> 7.  **ModelAndView**：装载了模型数据和视图信息，作为 Handler 的处理结果，返回给 DispatcherServlet。
> 8. **ViewResolver**：视图解析器，DispatcheServlet 通过它将逻辑视图解析为物理视图，最终将渲染结果响应给客户端。

### 10.4 Spring MVC 的工作流程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813192129755.png)
>1. DispatcherServlet 表示前置控制器，是整个SpringMVC的控制中心。用户发出请求，接收请求并拦截请求。
>2. HandlerMapping 为处理器映射。DispatcherServlet调用 HandlerMapping，HandlerMapping根据请求url查找Handler。
>3. HandlerExecution 表示具体的Handler,其主要作用是根据url查找控制器，如上url被查找控制器为：hello。
>4. HandlerExecution 将解析后的信息传递给 DispatcherServlet，如解析控制器映射等。
>5. HandlerAdapter 表示处理器适配器，其按照特定的规则去执行Handler。
>6. Handler 让具体的 Controller 执行。
>7. Controller 将具体的执行信息返回给 HandlerAdapter,如ModelAndView。
>8. HandlerAdapte r将视图逻辑名或模型传递给 DispatcherServlet。
>9. DispatcherServlet 调用视图解析器(ViewResolver)来解析 HandlerAdapter 传递的逻辑视图名。
>10. 视图解析器将解析的逻辑视图名传给 DispatcherServlet。
>11. DispatcherServlet 根据视图解析器解析的视图结果，调用具体的视图。
>12. 最终视图呈现给用户。

**Spring MVC** 虽然整体流程复杂，但是实际开发中很简单，大部分的组件不需要开发者创建跟管理，只需要通过配置文件的方式完成配置即可，真正需要开发者进行处理的只有 **Handler** 、**View** 、**Modle**。

但是随着前后端分离跟微服务的发展，一包走天下的开发模式其实用的不是很多了，大部分情况下是 **SpringBoot** + **Vue**。
# 11. Spring Boot
### 11.1 Spring Boot简介
Spring Boot 基于 Spring 开发，Spirng Boot 本身并不提供 Spring 框架的核心特性以及扩展功能，只是用于快速、敏捷地开发新一代基于 Spring 框架的应用程序。它并不是用来替代 Spring 的解决方案，而是和 Spring 框架紧密结合用于提升 Spring 开发者体验的工具。Spring Boot 以`约定大于配置`核心思想开展工作，相比Spring具有如下优势：
> 1. Spring Boot 可以建立独立的Spring应用程序。
> 2. Spring Boot 内嵌了如Tomcat，Jetty和Undertow这样的容器，也就是说可以直接跑起来，用不着再做部署工作了。
> 3. Spring Boot 无需再像Spring那样搞一堆繁琐的xml文件的配置。
> 4. Spring Boot 可以自动配置(核心)Spring。SpringBoot将原有的XML配置改为Java配置，将bean注入改为使用注解注入的方式(@Autowire)，并将多个xml、properties配置浓缩在一个appliaction.yml配置文件中。
> 5. Spring Boot 提供了一些现有的功能，如量度工具，表单数据验证以及一些外部配置这样的一些第三方功能。
> 6. Spring Boot 整合常用依赖（开发库，例如spring-webmvc、jackson-json、validation-api和tomcat等），提供的POM可以简化Maven的配置。当我们引入核心依赖时，SpringBoot会自引入其他依赖。

### 11.2 SpringBoot 注意点
1. SpringBoot 抽离
> 将所有的功能场景都抽取出来，做成一个个的starter，spring-boot-starter-xxx 就是spring-boot的场景启动器。只需要在项目中引入这些starter即可，所有相关的依赖都会导入进来 。
2. 自动配置原理
> 1. SpringBoot在启动的时候从类路径下的 META-INF/spring.factories 中获取 EnableAutoConfiguration 指定的值
>2. 我们看我们需要的功能有没有在SpringBoot默认写好的自动配置类 xxxxAutoConfigurartion 当中。
>3. 我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件存在在其中，我们就不需要再手动配置了）。
>4. 给容器中自动配置类添加组件的时候，会从 xxxxProperties 类中获取某些属性。我们只需要在配置文件中指定这些属性的值即可。
3. 各种组件的整合
> 比如整合MyBatis、Redis、Swagger、Security、Shrio、Druid等，百度[教程](https://blog.csdn.net/forezp/category_9268735.html)即可。


### 11.3  Springboot启动原理的底层
SpringApplication这个类主要做了以下四件事情：
>1. 推断应用的类型是普通的项目还是Web项目
>2. 查找并加载所有可用初始化器 ， 设置到initializers属性中
>3. 找出所有的应用程序监听器，设置到listeners属性中
>4. 推断并设置main方法的定义类，找到运行的主类

[SpringBoot](https://blog.csdn.net/weixin_43570367/article/details/104960677)启动大致流程如下(源网侵删)：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201230110046871.png)

### 11.3 架构演进
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122921003671.png)
1. 单体应用
> 传统项目把所有的业务功能在一个项目中，这种单体架构结构逻辑比较简单，对于小型项目来说很实用。但随着业务量的增多，逻辑越来越复杂，项目会逐渐变得非常庞大，逻辑也会变得混乱，给维护和开发造成比较大的困难。
2. 垂直架构
>把原来比较大的单体项目根据`业务逻辑`拆分成多个小的单体项目，比如把物流系统、客户关系系统从原来的电子商城系统中抽离出来，构建成两个小的项目。这种架构虽然解决了原来单体项目过大的问题，但也带来了数据冗余、耦合性大的问题。
3. SOA架构
>**面向服务架构**(Service Oriented Architecture)，它在垂直架构的基础上，把原来项目中的**公共组件**抽离出来做成形成服务，为各个系统提供服务。服务层即抽取出的公共组件。系统层的多个小型系统通过ESB企业服务总线（它是项目与服务之间通信的桥梁）以及Webservice调用服务，完成各自的业务逻辑。但是SOA架构抽取的粒度比较粗糙，系统与服务之间的**耦合性很大**，系统与服务界限不够清晰，给开发和维护造成一定的困难。
4. 微服务架构
>微服务架构对服务的**抽取粒度更小**，把系统中的服务层完全隔离出来。**遵循单一原则**，系统层和服务层的界限清晰，各个系统通过服务网关调用所需微服务。微服务之间通过**RESTful**等轻量级协议传输，相比ESB更轻量。但这种架构的开发成本比较高（因为新增了容错、分布式事务等要求），对团队的要求比较大，所以不适合小项目、小团队。

5. 框架演变
> 从一个复杂应用场景衍生一种规范框架，用户只需要进行各种配置而不需要自己去实现它，这时候强大的配置功能成了优点。发展到一定程度之后，人们根据实际生产应用情况，选取其中实用功能和设计精华，重构出一些轻量级的框架。之后为了提高开发效率，嫌弃原先的各类配置过于麻烦，于是开始提倡**约定大于配置**的方案，进而衍生出一些一站式的解决方案。
# 12. Spring Cloud 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200926120420882.png)
微服务的定义 ：
> 1. 2014 年 **Martin Fowler** 提出的一种新的架构形式。微服务架构是一种**架构模式**，提倡将单一应用程序划分成一组小的服务，服务之间相互协调，互相配合，为用户提供最终价值。每个服务运行在其独立的进程中，服务与服务之间采用轻量级的通信机制(如HTTP或Dubbo)互相协作，每个服务都围绕着具体的业务进行构建，并且能够被独立的部署到生产环境中，另外，应尽量避免统一的，集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具(如Maven)对其进行构建。
> 2. 微服务化的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合，每一个微服务提供单个业务功能的服务，一个服务做一件事情，从技术角度看就是一种小而独立的处理过程，类似进程的概念，能够自行单独启动或销毁，拥有自己独立的数据库。


微服务时代**核心**问题(问题根本：`网络不可靠`)：
>1. 服务很多，客户端怎么访问，如何提供对外网关?
>2. 这么多服务，服务之间如何通信? HTTP还是RPC?
>3. 这么多服务，如何治理? 服务的注册跟发现。
>4. 服务挂了怎么办？ 熔断机制。

主流微服务框架：
> 1. Spring Cloud Netflix
> 2. Spring Cloud Alibaba
> 3.  Spring +  Dubbo  +  ZooKeeper 

关于 [SpringCloud Netflix](https://mp.weixin.qq.com/s/A4yRiLBM9JTcPV8nulln2A) 前面详细写过，在此不再重复。
# 13.  常用注解
感觉Spring这块没啥特别好写的不知道为啥，可能跟自己用的少也有关吧，最后来几个简单注解收尾，一般有Spring核心注解、SpringBoot注解、SpringCloud注解、任务执行跟调度注解、测试注解、JPA注解、SpringMVC跟REST注解等等，这里只罗列下几个核心注解(全部注解公众号回复`注解`)：
>1. **@Component** : 可以配置CommandLineRunner使用，当一个组件不好归属到下面类的时候会用该注解标注，**@Controller** 、**@Service**、 **@Repository** 属于 Component的细化。
> 5. **@Autowired** ： 自动导入依赖的Bean，默认byType，完成属性，方法的组装，可以对类成员变量，方法，构造函数进行标注，加上(required=false)时找不到也不报错
> 6. **@Import** : 跟@Bean类似，更灵活的导入其他配置类。**ImportSelector**、**ImportBeanDefinitionRegistrar**等
> 7. **@Bean** : 等价xml中配置的bean, 用在方法上哦，来生产出一个bean，然后交给Spring管理
> 8. **@Value** : 可用在字段，构造器参数跟方法参数，指定一个默认值，支持 #{} 跟 ${} 两个方式。
一般将SpringbBoot中的application.properties 配置的属性值赋值给变量。
> 9. **@Configuration** : 等价于Spring的XML配置文件，使用Java代码可以检查类型安全。如果有些时候必须用到xml配置文件，可通过@Configuration 类作为项目的配置主类，使用@ImportResource注解加载xml 文件
> 10. **@Qualifier** : 该注解通常跟@Autowired一起使用，当想对注入的过程做更多的控制，@Qualifier可帮助配置，比如两个以上相同类型的Bean时 Spring无法抉择，用到此注解

# 14. 参考
> Spring MVC 常见面试题：https://blog.csdn.net/a745233700/article/details/80963758
> Spring 常见面试题：https://thinkwon.blog.csdn.net/article/details/104397516
---

