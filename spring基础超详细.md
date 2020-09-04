# spring

**1、Spring是什么?**

Spring是一个轻量级的IoC和AOP容器框架。是为Java应用程序提供基础性服务的一套框架，目的是用于简化企业应用程序的开发，它使得开发者只需要关心业务需求。常见的配置方式有三种：基于XML的配置、基于注解的配置、基于Java的配置。

主要由以下几个模块组成：

Spring Core：核心类库，提供IOC服务；

Spring Context：提供框架式的Bean访问方式，以及企业级功能（JNDI、定时任务等）；

Spring AOP：AOP服务；

Spring DAO：对JDBC的抽象，简化了数据访问异常的处理；

Spring ORM：对现有的ORM框架的支持；

Spring Web：提供了基本的面向Web的综合特性，例如多方文件上传；

Spring MVC：提供面向Web应用的Model-View-Controller实现。

**2、Spring 的优点？**

（1）spring属于低侵入式设计，代码的污染极低；

（2）spring的DI机制将对象之间的依赖关系交由框架处理，减低组件的耦合性；

（3）Spring提供了AOP技术，支持将一些通用任务，如安全、事务、日志、权限等进行集中式管理，从而提供更好的复用。

（4）spring对于主流的应用框架提供了集成支持。

 

**3、Spring的AOP理解：**

OOP面向对象，允许开发者定义纵向的关系，但并适用于定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用。

AOP，一般称为面向切面，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理。

AOP实现的关键在于 代理模式，AOP代理主要分为静态代理和动态代理。静态代理的代表为AspectJ；动态代理则以Spring AOP为代表。

（1）AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在**编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象。

（2）Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

Spring AOP中的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理：

​    ①JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler 通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，Proxy利用 InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。

​    ②如果代理类没有实现 InvocationHandler 接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

（3）静态代理与动态代理区别在于生成AOP代理对象的时机不同，相对来说AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。

>  InvocationHandler 的 invoke(Object proxy,Method method,Object[] args)：proxy是最终生成的代理实例; method 是被代理目标实例的某个具体方法; args 是被代理目标实例某个方法的具体入参, 在方法反射调用时使用。

 

**4、Spring的IoC理解：**

（1）IOC就是控制反转，是指创建对象的控制权的转移，以前创建对象的主动权和时机是由自己把控的，而现在这种权力转移到Spring容器中，并由容器根据配置文件去创建实例和管理各个实例之间的依赖关系，对象与对象之间松散耦合，也利于功能的复用。DI依赖注入，和控制反转是同一个概念的不同角度的描述，即 应用程序在运行时依赖IoC容器来动态注入对象需要的外部资源。

（2）最直观的表达就是，IOC让对象的创建不用去new了，可以由spring自动生产，使用java的反射机制，根据配置文件在运行时动态的去创建对象以及管理对象，并调用对象的方法的。

（3）Spring的IOC有三种注入方式 ：构造器注入、setter方法注入、根据注解注入。

> IoC让相互协作的组件保持松散的耦合，而AOP编程允许你把遍布于应用各层的功能分离出来形成可重用的功能组件。

 

**5、BeanFactory和ApplicationContext有什么区别？**

​    BeanFactory和ApplicationContext是Spring的两大核心接口，都可以当做Spring的容器。其中ApplicationContext是BeanFactory的子接口。

（1）BeanFactory：是Spring里面最底层的接口，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。ApplicationContext接口作为BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能：

①继承MessageSource，因此支持国际化。

②统一的资源文件访问方式。

③提供在监听器中注册bean的事件。

④同时加载多个配置文件。

⑤载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层。

（2）①BeanFactroy采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化。这样，我们就不能发现一些存在的Spring的配置问题。如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常。

​    ②ApplicationContext，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误，这样有利于检查所依赖属性是否注入。 ApplicationContext启动后预载入所有的单实例Bean，通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

​    ③相对于基本的BeanFactory，ApplicationContext 唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。

（3）BeanFactory通常以编程的方式被创建，ApplicationContext还能以声明的方式创建，如使用ContextLoader。

（4）BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册。

 

**6、请解释Spring Bean的生命周期？**

 首先说一下Servlet的生命周期：实例化，初始init，接收请求service，销毁destroy；

 Spring上下文中的Bean生命周期也类似，如下：

（1）实例化Bean：

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。对于ApplicationContext容器，当容器启动结束后，通过获取BeanDefinition对象中的信息，实例化所有的bean。

（2）设置对象属性（依赖注入）：

实例化后的对象被封装在BeanWrapper对象中，紧接着，Spring根据BeanDefinition中的信息 以及 通过BeanWrapper提供的设置属性的接口完成依赖注入。

（3）处理Aware接口：

接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给Bean：

①如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String beanId)方法，此处传递的就是Spring配置文件中Bean的id值；

②如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()方法，传递的是Spring工厂自身。

③如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文；

（4）BeanPostProcessor：

如果想对Bean进行一些自定义的处理，那么可以让Bean实现了BeanPostProcessor接口，那将会调用postProcessBeforeInitialization(Object obj, String s)方法。

（5）InitializingBean 与 init-method：

如果Bean在Spring配置文件中配置了 init-method 属性，则会自动调用其配置的初始化方法。

（6）如果这个Bean实现了BeanPostProcessor接口，将会调用postProcessAfterInitialization(Object obj, String s)方法；由于这个方法是在Bean初始化结束时调用的，所以可以被应用于内存或缓存技术；

> 以上几个步骤完成后，Bean就已经被正确创建了，之后就可以使用这个Bean了。

（7）DisposableBean：

当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的destroy()方法；

（8）destroy-method：

最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

 

**7、 解释Spring支持的几种bean的作用域。**

Spring容器中的bean可以分为5个范围：

（1）singleton：默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护。

（2）prototype：为每一个bean请求提供一个实例。

（3）request：为每一个网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。

（4）session：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。

（5）global-session：全局作用域，global-session和Portlet应用相关。当你的应用部署在Portlet容器中工作时，它包含很多portlet。如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中。全局作用域与Servlet中的session作用域效果相同。

 

**8、Spring框架中的单例Beans是线程安全的么？**

​    Spring框架并没有对单例bean进行任何多线程的封装处理。关于单例bean的线程安全和并发问题需要开发者自行去搞定。但实际上，大部分的Spring bean并没有可变的状态(比如Serview类和DAO类)，所以在某种程度上说Spring的单例bean是线程安全的。如果你的bean有多种状态的话（比如 View Model 对象），就需要自行保证线程安全。最浅显的解决办法就是将多态bean的作用域由“singleton”变更为“prototype”。

**9、Spring如何处理线程并发问题？**

在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域，因为Spring对一些Bean中非线程安全状态采用ThreadLocal进行处理，解决线程安全问题。

ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。同步机制采用了“时间换空间”的方式，仅提供一份变量，不同的线程在访问前需要获取锁，没获得锁的线程则需要排队。而ThreadLocal采用了“空间换时间”的方式。

ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal。

 

**10-1、Spring基于xml注入bean的几种方式：**

（1）Set方法注入；

（2）构造器注入：①通过index设置参数的位置；②通过type设置参数类型；

（3）静态工厂注入；

（4）实例工厂；

详细内容可以阅读：https://www.iteye.com/blog/blessht-1162131

**10-2、Spring的自动装配：**

在spring中，对象无需自己查找或创建与其关联的其他对象，由容器负责把需要相互协作的对象引用赋予各个对象，使用autowire来配置自动装载模式。

在Spring框架xml配置中共有5种自动装配：

（1）no：默认的方式是不进行自动装配的，通过手工设置ref属性来进行装配bean。

（2）byName：通过bean的名称进行自动装配，如果一个bean的 property 与另一bean 的name 相同，就进行自动装配。 

（3）byType：通过参数的数据类型进行自动装配。

（4）constructor：利用构造函数进行装配，并且构造函数的参数通过byType进行装配。

（5）autodetect：自动探测，如果有构造方法，通过 construct的方式自动装配，否则使用 byType的方式自动装配。

基于注解的方式：

使用@Autowired注解来自动装配指定的bean。在使用@Autowired注解之前需要在Spring配置文件进行配置，<context:annotation-config />。在启动spring IoC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，就会在IoC容器自动查找需要的bean，并装配给该对象的属性。在使用@Autowired时，首先在容器中查询对应类型的bean：

如果查询结果刚好为一个，就将该bean装配给@Autowired指定的数据；

如果查询的结果不止一个，那么@Autowired会根据名称来查找；

如果上述查找的结果为空，那么会抛出异常。解决方法时，使用required=false。

> @Autowired可用于：构造函数、成员变量、Setter方法

注：@Autowired和@Resource之间的区别

(1) @Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）。

(2) @Resource默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入。

 

**11、Spring 框架中都用到了哪些设计模式？**

（1）工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例；

（2）单例模式：Bean默认为单例模式。

（3）代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；

（4）模板方法：用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。

（5）观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中listener的实现--ApplicationListener。

 

**12、Spring事务的实现方式和实现原理：**

Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的。

**（1）Spring事务的种类：**

spring支持编程式事务管理和声明式事务管理两种方式：

①编程式事务管理使用TransactionTemplate。

②声明式事务管理建立在AOP之上的。其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

> 声明式事务最大的优点就是不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明或通过@Transactional注解的方式，便可以将事务规则应用到业务逻辑中。

声明式事务管理要优于编程式事务管理，这正是spring倡导的非侵入式的开发方式，使业务代码不受污染，只要加上注解就可以获得完全的事务支持。唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。

**（2）spring的事务传播行为：**

spring事务的传播行为说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为。

① PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。

② PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。‘

③ PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。

④ PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。

⑤ PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

⑥ PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。

⑦ PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

**（3）Spring中的隔离级别：**

① ISOLATION_DEFAULT：这是个 PlatfromTransactionManager 默认的隔离级别，使用数据库默认的事务隔离级别。

② ISOLATION_READ_UNCOMMITTED：读未提交，允许另外一个事务可以看到这个事务未提交的数据。

③ ISOLATION_READ_COMMITTED：读已提交，保证一个事务修改的数据提交后才能被另一事务读取，而且能看到该事务对已有记录的更新。

④ ISOLATION_REPEATABLE_READ：可重复读，保证一个事务修改的数据提交后才能被另一事务读取，但是不能看到该事务对已有记录的更新。

⑤ ISOLATION_SERIALIZABLE：一个事务在执行的过程中完全看不到其他事务对数据库所做的更新。

 

**13、Spring框架中有哪些不同类型的事件？**

Spring 提供了以下5种标准的事件：

（1）上下文更新事件（ContextRefreshedEvent）：在调用ConfigurableApplicationContext 接口中的refresh()方法时被触发。

（2）上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。

（3）上下文停止事件（ContextStoppedEvent）：当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。

（4）上下文关闭事件（ContextClosedEvent）：当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。

（5）请求处理事件（RequestHandledEvent）：在Web应用中，当一个http请求（request）结束触发该事件。

如果一个bean实现了ApplicationListener接口，当一个ApplicationEvent 被发布以后，bean会自动被通知。

 

**14、解释一下Spring AOP里面的几个名词：**

（1）切面（Aspect）：被抽取的公共模块，可能会横切多个对象。 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @AspectJ 注解来实现。

（2）连接点（Join point）：指方法，在Spring AOP中，一个连接点 总是 代表一个方法的执行。 

（3）通知（Advice）：在切面的某个特定的连接点（Join point）上执行的动作。通知有各种类型，其中包括“around”、“before”和“after”等通知。许多AOP框架，包括Spring，都是以拦截器做通知模型， 并维护一个以连接点为中心的拦截器链。

（4）切入点（Pointcut）：切入点是指 我们要对哪些Join point进行拦截的定义。通过切入点表达式，指定拦截的方法，比如指定拦截add*、search*。

（5）引入（Introduction）：（也被称为内部类型声明（inter-type declaration））。声明额外的方法或者某个类型的字段。Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用一个引入来使bean实现 IsModified 接口，以便简化缓存机制。

（6）目标对象（Target Object）： 被一个或者多个切面（aspect）所通知（advise）的对象。也有人把它叫做 被通知（adviced） 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象。

（7）织入（Weaving）：指把增强应用到目标对象来创建新的代理对象的过程。Spring是在运行时完成织入。

切入点（pointcut）和连接点（join point）匹配的概念是AOP的关键，这使得AOP不同于其它仅仅提供拦截功能的旧技术。 切入点使得定位通知（advice）可独立于OO层次。 例如，一个提供声明式事务管理的around通知可以被应用到一组横跨多个对象中的方法上（例如服务层的所有业务操作）。

![img](https://img-blog.csdn.net/20180708154818891)

**15、Spring通知有哪些类型？**

https://blog.csdn.net/qq_32331073/article/details/80596084

（1）前置通知（Before advice）：在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）。

（2）返回后通知（After returning advice）：在某连接点（join point）正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。 

（3）抛出异常后通知（After throwing advice）：在方法抛出异常退出时执行的通知。 

（4）后通知（After (finally) advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。 

（5）环绕通知（Around Advice）：包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型。 环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。 环绕通知是最常用的一种通知类型。大部分基于拦截的AOP框架，例如Nanning和JBoss4，都只提供环绕通知。 

> 同一个aspect，不同advice的执行顺序：

①没有异常情况下的执行顺序：

around before advice
before advice
target method 执行
around after advice
after advice
afterReturning

②有异常情况下的执行顺序：

around before advice
before advice
target method 执行
around after advice
after advice
afterThrowing:异常发生
java.lang.RuntimeException: 异常发生

# Spring中的循环依赖

## 什么是循环依赖？

很简单，就是A对象依赖了B对象，B对象依赖了A对象。



比如：

```
// A依赖了B
class A{
    public B b;
}

// B依赖了A
class B{
    public A a;
}
```

那么循环依赖是个问题吗？



如果不考虑Spring，循环依赖并不是问题，因为对象之间相互依赖是很正常的事情。



比如

```
A a = new A();
B b = new B();

a.b = b;
b.a = a;
```



这样，A,B就依赖上了。



但是，在Spring中循环依赖就是一个问题了，为什么？

因为，在Spring中，一个对象并不是简单new出来了，而是会经过一系列的Bean的生命周期，就是因为Bean的生命周期所以才会出现循环依赖问题。当然，在Spring中，出现循环依赖的场景很多，有的场景Spring自动帮我们解决了，而有的场景则需要程序员来解决，下文详细来说。



要明白Spring中的循环依赖，得先明白Spring中Bean的生命周期。



## Bean的生命周期

这里不会对Bean的生命周期进行详细的描述，只描述一下大概的过程。



Bean的生命周期指的就是：在Spring中，Bean是如何生成的？



被Spring管理的对象叫做Bean。Bean的生成步骤如下：

1. Spring扫描class得到BeanDefinition
2. 根据得到的BeanDefinition去生成bean
3. 首先根据class推断构造方法
4. 根据推断出来的构造方法，反射，得到一个对象（暂时叫做原始对象）
5. 填充原始对象中的属性（依赖注入）
6. 如果原始对象中的某个方法被AOP了，那么则需要根据原始对象生成一个代理对象
7. 把最终生成的代理对象放入单例池（源码中叫做singletonObjects）中，下次getBean时就直接从单例池拿即可



可以看到，对于Spring中的Bean的生成过程，步骤还是很多的，并且不仅仅只有上面的7步，还有很多很多，比如Aware回调、初始化等等，这里不详细讨论。



可以发现，在Spring中，构造一个Bean，包括了new这个步骤（第4步构造方法反射）。



得到一个原始对象后，Spring需要给对象中的属性进行依赖注入，那么这个注入过程是怎样的？



比如上文说的A类，A类中存在一个B类的b属性，所以，当A类生成了一个原始对象之后，就会去给b属性去赋值，此时就会根据b属性的类型和属性名去BeanFactory中去获取B类所对应的单例bean。如果此时BeanFactory中存在B对应的Bean，那么直接拿来赋值给b属性；如果此时BeanFactory中不存在B对应的Bean，则需要生成一个B对应的Bean，然后赋值给b属性。



问题就出现在第二种情况，如果此时B类在BeanFactory中还没有生成对应的Bean，那么就需要去生成，就会经过B的Bean的生命周期。



那么在创建B类的Bean的过程中，如果B类中存在一个A类的a属性，那么在创建B的Bean的过程中就需要A类对应的Bean，但是，触发B类Bean的创建的条件是A类Bean在创建过程中的依赖注入，所以这里就出现了循环依赖：



ABean创建-->依赖了B属性-->触发BBean创建--->B依赖了A属性--->需要ABean（但ABean还在创建过程中）



从而导致ABean创建不出来，BBean也创建不出来。



这是循环依赖的场景，但是上文说了，在Spring中，通过某些机制帮开发者解决了部分循环依赖的问题，这个机制就是**三级缓存**。



## 三级缓存



三级缓存是通用的叫法。

一级缓存为：**singletonObjects**

二级缓存为：**earlySingletonObjects**

三级缓存为**：** **singletonFactories**

**先稍微解释一下这三个缓存的作用，后面详细分析：**

- **singletonObjects**中缓存的是已经经历了完整生命周期的bean对象。
- **earlySingletonObjects**比singletonObjects多了一个early，表示缓存的是早期的bean对象。早期是什么意思？表示Bean的生命周期还没走完就把这个Bean放入了earlySingletonObjects。
- **singletonFactories**中缓存的是ObjectFactory，表示对象工厂，用来创建某个对象的。

## 解决循环依赖思路分析

先来分析为什么缓存能解决循环依赖。



上文分析得到，之所以产生循环依赖的问题，主要是：



A创建时--->需要B---->B去创建--->需要A，从而产生了循环



![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1592471211638-86636131-146d-46c3-8775-421ef3322cc3.png)



那么如何打破这个循环，加个中间人（缓存）

![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1592471597769-3e23cc26-2b1d-4742-8c74-cea46327ada7.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5ZGo55Gc%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



A的Bean在创建过程中，在进行依赖注入之前，先把A的原始Bean放入缓存（提早暴露，只要放到缓存了，其他Bean需要时就可以从缓存中拿了），放入缓存后，再进行依赖注入，此时A的Bean依赖了B的Bean，如果B的Bean不存在，则需要创建B的Bean，而创建B的Bean的过程和A一样，也是先创建一个B的原始对象，然后把B的原始对象提早暴露出来放入缓存中，然后在对B的原始对象进行依赖注入A，此时能从缓存中拿到A的原始对象（虽然是A的原始对象，还不是最终的Bean），B的原始对象依赖注入完了之后，B的生命周期结束，那么A的生命周期也能结束。



因为整个过程中，都只有一个A原始对象，所以对于B而言，就算在属性注入时，注入的是A原始对象，也没有关系，因为A原始对象在后续的生命周期中在堆中没有发生变化。



从上面这个分析过程中可以得出，只需要一个缓存就能解决循环依赖了，那么为什么Spring中还需要**singletonFactories**呢？



这是难点，基于上面的场景想一个问题：如果A的原始对象注入给B的属性之后，A的原始对象进行了AOP产生了一个代理对象，此时就会出现，对于A而言，它的Bean对象其实应该是AOP之后的代理对象，而B的a属性对应的并不是AOP之后的代理对象，这就产生了冲突。



**B依赖的A和最终的A不是同一个对象**。



那么如何解决这个问题？这个问题可以说没有办法解决。



因为在一个Bean的生命周期最后，Spring提供了BeanPostProcessor可以去对Bean进行加工，这个加工不仅仅只是能修改Bean的属性值，也可以替换掉当前Bean。



举个例子：

```
@Component
public class User {
}
```



```
@Component
public class LubanBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {

        // 注意这里，生成了一个新的User对象
        if (beanName.equals("user")) {
            System.out.println(bean);
            User user = new User();
            return user;
        }

        return bean;
    }
}
```

**
**

```
public class Test {
    public static void main(String[] args) {

        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(AppConfig.class);
        
        User user = context.getBean("user", User.class);
        System.out.println(user);

    }
}
```



运行main方法，得到的打印如下：

```
com.luban.service.User@5e025e70
com.luban.service.User@1b0375b3
```



所以在BeanPostProcessor中可以完全替换掉某个beanName对应的bean对象。



而BeanPostProcessor的执行在Bean的生命周期中是处于属性注入之后的，循环依赖是发生在属性注入过程中的，所以很有可能导致，**注入给B对象的A对象和经历过完整生命周期之后的A对象，不是一个对象**。这就是有问题的。



**所以在这种情况下的循环依赖，Spring是解决不了的，因为在属性注入时，Spring也不知道A对象后续会经过哪些** **BeanPostProcessor以及会对A对象做什么处理**。



## Spring到底解决了哪种情况下的循环依赖



虽然上面的情况可能发生，但是肯定发生得很少，我们通常在开发过程中，不会这样去做，但是，某个beanName对应的最终对象和原始对象不是一个对象却会经常出现，这就是AOP。



AOP就是通过一个BeanPostProcessor来实现的，这个BeanPostProcessor就是AnnotationAwareAspectJAutoProxyCreator，它的父类是AbstractAutoProxyCreator，而在Spring中AOP利用的要么是JDK动态代理，要么CGLib的动态代理，所以如果给一个类中的某个方法设置了切面，那么这个类最终就需要生成一个代理对象。



一般过程就是：A类--->生成一个普通对象-->属性注入-->基于切面生成一个代理对象-->把代理对象放入singletonObjects单例池中。



而AOP可以说是Spring中除开IOC的另外一大功能，而循环依赖又是属于IOC范畴的，所以这两大功能想要并存，Spring需要特殊处理。



如何处理的，就是利用了第三级缓存**singletonFactories**。



首先，singletonFactories中存的是某个beanName对应的ObjectFactory，在bean的生命周期中，生成完原始对象之后，就会构造一个ObjectFactory存入singletonFactories中。这个ObjectFactory是一个函数式接口，所以支持Lambda表达式：**() -> getEarlyBeanReference(****beanName****,** **mbd****,** **bean****)**



上面的Lambda表达式就是一个ObjectFactory，执行该Lambda表达式就会去执行getEarlyBeanReference方法，而该方法如下：

```
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
            }
        }
    }
    return exposedObject;
}
```

该方法会去执行SmartInstantiationAwareBeanPostProcessor中的getEarlyBeanReference方法，而这个接口下的实现类中只有两个类实现了这个方法，一个是AbstractAutoProxyCreator，一个是InstantiationAwareBeanPostProcessorAdapter，它的实现如下：



```
// InstantiationAwareBeanPostProcessorAdapter
@Override
public Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {
    return bean;
}
```



```
// AbstractAutoProxyCreator
@Override
public Object getEarlyBeanReference(Object bean, String beanName) {
    Object cacheKey = getCacheKey(bean.getClass(), beanName);
    this.earlyProxyReferences.put(cacheKey, bean);
    return wrapIfNecessary(bean, beanName, cacheKey);
}
```



所以很明显，在整个Spring中，默认就只有AbstractAutoProxyCreator真正意义上实现了getEarlyBeanReference方法，而该类就是用来进行AOP的。上文提到的AnnotationAwareAspectJAutoProxyCreator的父类就是AbstractAutoProxyCreator。



那么getEarlyBeanReference方法到底在干什么？

首先得到一个cachekey，cachekey就是beanName。

然后把beanName和bean（这是原始对象）存入earlyProxyReferences中

调用wrapIfNecessary进行AOP，得到一个代理对象。



那么，什么时候会调用getEarlyBeanReference方法呢？回到循环依赖的场景中



![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1592539097062-7912a20c-f209-47bd-bdc0-d6d4485ab395.png)



**左边文字**：

这个ObjectFactory就是上文说的labmda表达式，中间有getEarlyBeanReference方法，注意存入singletonFactories时并不会执行lambda表达式，也就是不会执行getEarlyBeanReference方法



**右边文字**：

从singletonFactories根据beanName得到一个ObjectFactory，然后执行ObjectFactory，也就是执行getEarlyBeanReference方法，此时会得到一个A原始对象经过AOP之后的代理对象，然后把该代理对象放入earlySingletonObjects中，注意此时并没有把代理对象放入singletonObjects中，那什么时候放入到singletonObjects中呢？



我们这个时候得来理解一下earlySingletonObjects的作用，此时，我们只得到了A原始对象的代理对象，这个对象还不完整，因为A原始对象还没有进行属性填充，所以此时不能直接把A的代理对象放入singletonObjects中，所以只能把代理对象放入earlySingletonObjects，假设现在有其他对象依赖了A，那么则可以从earlySingletonObjects中得到A原始对象的代理对象了，并且是A的同一个代理对象。



当B创建完了之后，A继续进行生命周期，而A在完成属性注入后，会按照它本身的逻辑去进行AOP，而此时我们知道A原始对象已经经历过了AOP，所以对于A本身而言，不会再去进行AOP了，那么怎么判断一个对象是否经历过了AOP呢？会利用上文提到的earlyProxyReferences，在AbstractAutoProxyCreator的postProcessAfterInitialization方法中，会去判断当前beanName是否在earlyProxyReferences，如果在则表示已经提前进行过AOP了，无需再次进行AOP。



对于A而言，进行了AOP的判断后，以及BeanPostProcessor的执行之后，就需要把A对应的对象放入singletonObjects中了，但是我们知道，应该是要A的代理对象放入singletonObjects中，所以此时需要从earlySingletonObjects中得到代理对象，然后入singletonObjects中。



**整个循环依赖解决完毕。**



## 总结

至此，总结一下三级缓存：

1. **singletonObjects**：缓存某个beanName对应的经过了完整生命周期的bean
2. **earlySingletonObjects**：缓存提前拿原始对象进行了AOP之后得到的代理对象，原始对象还没有进行属性注入和后续的BeanPostProcessor等生命周期
3. **singletonFactories**：缓存的是一个ObjectFactory，主要用来去生成原始对象进行了AOP之后得到的代理对象，在每个Bean的生成过程中，都会提前暴露一个工厂，这个**工厂可能用到，也可能用不到**，如果没有出现循环依赖依赖本bean，那么这个工厂无用，本bean按照自己的生命周期执行，执行完后直接把本bean放入singletonObjects中即可，如果出现了循环依赖依赖了本bean，则另外那个bean执行ObjectFactory提交得到一个AOP之后的代理对象(如果有AOP的话，如果无需AOP，则直接得到一个原始对象)。
4. 其实还要一个缓存，就是**earlyProxyReferences**，它用来记录某个原始对象是否进行过AOP了。

# Spring依赖注入原理分析(未完成)

**Spring中有几种依赖注入的方式？**

这是一个面试高频题，但是我在面程序员的时候，听过很多种答案。那么标准答案是什么？我们先不说，一步步来分析。



## 什么是依赖注入

首先，我们得知道什么是依赖注入？就是填充属性。



一个对象通常都会有属性，比如：



```
public class OrderService {
    private UserService userService;
    
    public UserService getUserService() {
        return userService;
    }
}
```



OrderService中有一个属性UserService， UserService就是OrderService的依赖。



那么Spring的依赖注入，就是Spring框架去进行属性的填充。那么我们就要站在Spring的角度去思考：**如果你是Spring的开发者，如果实现对一个对象的属性进行填充?**

**
**

在进行属性填充之前，我们得先知道：**在一个对象中，哪些属性可以进行填充？**



肯定是需要业务开发者去告诉程序员的，比如在属性上加一个特定的注解，比如Spring中的@Autowired。当Spring遇到该属性时，发现该属性存在这个注解，Spring就会对当前这个属性进行填充。那么怎么填充呢？



填充其实就是对属性进行赋值，那么Spring能怎么对这个属性进行赋值呢？赋的什么值呢？



假设赋的值是xx。我们先考虑如果把这个xx赋值给OrderService对象。



可能我们立马能想到的就是：

```
orderService.userService = xx;
```



这个思路没错，但是不能满足所有情况，因为orderService这个对象是在Spring中实例化的，userService这个属性的权限修饰符是private，所以在Spring中不能直接进行赋值，但是可以通过反射，比如：



```
// 随便new一个表示xx，Spring中寻找要注入的值是一个比较复杂的过程
UserService xx = new UserService();

Class c= Class.forName("com.luban.service.OrderService");
Object cInstance = c.newInstance();
Field[] fields = c.getDeclaredFields();
fields[0].setAccessible(true);  // fields[0]表示的就是userServce属性
fields[0].set(cInstance, xx);
System.out.println(((OrderService)cInstance).getUserService());
```



这样，通过反射就能对属性进行赋值了。那么怎么寻找到准确的应该赋值给该属性的值呢？



上文中的xx肯定也是一个对象，也就是说是Spring中的一个bean，那么Spring该如何根据当前属性去找到对应的bean呢？只有两种方式:



1. 根据属性的名字
2. 根据属性的类型



根据属性的名字去Spring容器中去找bean，要么找不到，要么就能找到一个bean，因为Spring中的beanName是唯一的。



格局属性的类型去Spring容器中去找bean，要么找不到，要么可能找到一个或多个bena，因为Spring容器中的bean实际就是一个对象，而一个类型是可以有多个对象的，在Spring容器中也是如此。



那么Spring针对这两种方式会如何选择呢？二选一，还是二合一。



答案很明显，肯定是二合一，一种方式找不到就利用另外一个方式去找。那么两种方式中会优先利用哪种方式去找？



答案很明显，肯定是先利用属性的名字，因为利用名字去找更精确。



### byName

根据属性名，去



### byType

# Spring中一些概念的总结

## 对象和Bean的区别？



个人观点：所谓的bean也是一个java对象，只不过这个对象是通过spring定义的，而一开始就是通过<bean>标签定义的，所以叫做bean。



普通对象和Bean对象还有其他区别，因为Bean对象是由Spring生成的，Spring在生成Bean对象的过程中，会历经很多其他步骤，比如属性注入，aop，new实例，调用初始化方法。



## 如何理解BeanDefinition？

顾名思义，BeanDefinition是用来描述一个Bean的，Spring会根据BeanDefinition来生成一个Bean。



## BeanFactory和FactoryBean的区别



### BeanFactory

BeanFactory是Spring IOC容器的顶级接口，其实现类有XMLBeanFactory，DefaultListableBeanFactory以及AnnotationConfigApplicationContext等。BeanFactory为Spring管理Bean提供了一套通用的规范。接口中提供的一些方法如下：

```
boolean containsBean(String beanName)

Object getBean(String)

Object getBean(String, Class)

Class getType(String name)

boolean isSingleton(String)

String[] getAliases(String name)
```



可以通过BeanFactory获得Bean。



### FactoryBean

FactoryBean首先也是一个Bean，但不是简单的Bean，而是一个能生产对象的工厂Bean，可以通过定义FactoryBean中的getObject()方法来创建生成过程比较复杂的Bean。



## 如何理解BeanFactoryPostProcessor？

BeanFactoryPostProcessor也叫做BeanFactory后置处理器。这里包括两个概念，一个是BeanFactory，一个是后置处理器。



BeanFactory表示Bean工厂，可以基于BeanDefinition来生成Bean对象，所以在BeanFactory中存在所有的BeanDefinition。

后置处理器可以理解为：当某物品生产好了以后，可以进一步通过后置处理器来对此物品进行处理。



所以BeanFactoryPostProcessor可以理解为，可以得到BeanFactory对象并对它进行处理，比如修改它其中的某个BeanDefinition，或者直接向BeanFactory中添加某个对象作为bean。



## 如何理解BeanDefinitionRegistryPostProcessor？

BeanDefinitionRegistryPostProcessor是一个接口，继承了BeanFactoryPostProcessor，所以它也是一个BeanFactory后置处理器，所以它可以操作BeanFactory。



它特殊的地方在于，它拥有的功能比BeanFactoryPostProcessor多，比如BeanFactoryPostProcessor是不能向BeanFactory中添加BeanDefinition的（只能添加Bean对象），但是BeanDefinitionRegistryPostProcessor是可以向BeanFactory中添加BeanDefinition的。



## 如何理解@Import与ImportBeanDefinitionRegistrar？

### Import注解

@Import首先是一个注解，在Spring中是用来向Spring容器中导入Bean的。换个角度理解，就是我们一般都是通过在某个类上加上@Component注解来标志一个bean的，但如果我们希望以一种更灵活的方式去定义bean的话，就可以利用@Import注解。



@Import注解所指定的类，在Spring启动过程中会对指定的类进行判断，判断当前类是不是实现了比较特殊的接口，比如ImportBeanDefinitionRegistrar，如果存在特殊的接口就执行特殊的逻辑，如果没有则生成该类对应的BeanDefinition并放入BeanFactory中。



### ImportBeanDefinitionRegistrar

通过Import注解可以注册bean，虽然它也支持同时注册多个bean，但是不方便，特别是当我们想通过实现一些复杂逻辑来注册bean的话，仅仅通过Import注解是不方便的，这时就可以使用ImportBeanDefinitionRegistrar这个接口来动态的注册bean了，我这里说的注册bean指的是：通过生成BeanDefinition，并且把BeanDefinition放入BeanFactory中。



## 如何理解BeanDefinitionRegistry和BeanFactory?



BeanFactory表示Bean工厂，可以利用BeanFactory来生成bean。

BeanDefinitionRegistry表示BeanDefinition的注册表，可以用来添加或移除BeanDefinition。

# Spring整合Mybatis原理

在介绍Spring整合Mybatis原理之前，我们得先来稍微介绍Mybatis的工作原理。



## Mybatis的基本工作原理

在Mybatis中，我们可以使用一个接口去定义要执行sql，简化代码如下：



定义一个接口，@Select表示要执行查询sql语句。

```
public interface UserMapper {
  @Select("select * from user where id = #{id}")
  User selectById(Integer id);
}
```



以下为执行sql代码：

```
InputStream inputStream = Resources.getResourceAsStream("mybatis.xml");
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
SqlSession sqlSession = sqlSessionFactory.openSession();

// 以下使我们需要关注的重点
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
Integer id = 1;
User user = mapper.selectById(id);
```



Mybatis的目的是：**使得程序员能够以****调用方法****的方式****执行某个指定的sql，将执行sql的底层逻辑进行了封装。**

**
**

这里重点思考以下mapper这个对象，当调用SqlSession的getMapper方法时，会对传入的接口生成一个**代理对象**，而程序要真正用到的就是这个代理对象，在调用代理对象的方法时，Mybatis会取出该方法所对应的sql语句，然后利用JDBC去执行sql语句，最终得到结果。



## 分析需要解决的问题

Spring和Mybatis时，我们重点要关注的就是这个代理对象。因为整合的目的就是：**把某个Mapper的****代理对象****作为一个****bean****放入Spring容器中，使得能够像使用一个普通bean一样去使用这个代理对象，比如能被@Autowire自动注入。**

**
**

比如当Spring和Mybatis整合之后，我们就可以使用如下的代码来使用Mybatis中的代理对象了：

```
@Component
public class UserService {
    @Autowired
    private UserMapper userMapper;

    public User getUserById(Integer id) {
        return userMapper.selectById(id);
    }
}
```

**
**

UserService中的userMapper属性就会被自动注入为Mybatis中的代理对象。如果你基于一个已经完成整合的项目去调试即可发现，userMapper的类型为：org.apache.ibatis.binding.MapperProxy@41a0aa7d。证明确实是Mybatis中的代理对象。



好，那么现在我们要解决的问题的就是：**如何****能够把Mybatis的代理对象作为一个bean放入Spring容器中？**



要解决这个，我们需要对Spring的bean生成过程有一个了解。



## Spring中Bean的产生过程

Spring启动过程中，大致会经过如下步骤去生成bean

1. 扫描指定的包路径下的class文件
2. 根据class信息生成对应的BeanDefinition
3. 在此处，程序员可以利用某些机制去修改BeanDefinition
4. 根据BeanDefinition生成bean实例
5. 把生成的bean实例放入Spring容器中



假设有一个A类，假设有如下代码：



一个A类：

```
@Component
public class A {
}
```



一个B类，不存在@Component注解

```
public class B {
}
```



执行如下代码：

```
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
System.out.println(context.getBean("a"));
```



输出结果为：com.luban.util.A@6acdbdf5



A类对应的bean对象类型仍然为A类。但是这个结论是不确定的，我们可以利用BeanFactory后置处理器来修改BeanDefinition，我们添加一个BeanFactory后置处理器：



```
@Component
public class LubanBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        BeanDefinition beanDefinition = beanFactory.getBeanDefinition("a");
        beanDefinition.setBeanClassName(B.class.getName());
    }
}
```



这样就会导致，原本的A类对应的BeanDefiniton被修改了，被修改成了B类，那么后续正常生成的bean对象的类型就是B类。此时，调用如下代码会报错：



```
context.getBean(A.class);
```



但是调用如下代码不会报错，尽管B类上没有@Component注解：

```
context.getBean(B.class);
```



并且，下面代码返回的结果是：com.luban.util.B@4b1c1ea0

```
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
System.out.println(context.getBean("a"));
```





之所以讲这个问题，是想说明一个问题：**在Spring中，bean对象跟class没有直接关系，跟BeanDefinition才有直接关系。**



那么回到我们要解决的问题：**如何****能够把Mybatis的代理对象作为一个bean放入Spring容器中？**

**
**

在Spring中，**如果你想生成一个bean，那么得先生成一个BeanDefinition**，就像你想new一个对象实例，得先有一个class。

**
**

## 解决问题

继续回到我们的问题，我们现在想自己生成一个bean，那么得先生成一个BeanDefinition，只要有了BeanDefinition，通过在BeanDefinition中设置**bean对象的类型**，然后把BeanDefinition添加给Spring，Spring就会根据BeanDefinition自动帮我们生成一个类型对应的bean对象。



所以，现在我们要解决两个问题：

1. **Mybatis的代理对象的类型是什么？因为我们要设置给BeanDefinition**
2. **我们怎么把BeanDefinition添加给Spring容器？**

**
**

注意：上文中我们使用的BeanFactory后置处理器，他只能修改BeanDefinition，并不能新增一个BeanDefinition。我们应该使用Import技术来添加一个BeanDefinition。后文再详细介绍如果使用Import技术来添加一个BeanDefinition，可以先看一下伪代码实现思路。



假设：我们有一个UserMapper接口，他的代理对象的类型为UserMapperProxy。

那么我们的思路就是这样的，伪代码如下：



```
BeanDefinitoin bd = new BeanDefinitoin();
bd.setBeanClassName(UserMapperProxy.class.getName());
SpringContainer.addBd(bd);
```



但是，这里有一个严重的问题，就是上文中的UserMapperProxy是我们假设的，他表示一个代理类的类型，然而Mybatis中的代理对象是利用的JDK的动态代理技术实现的，也就是代理对象的代理类是动态生成的，我们根本无法确定代理对象的代理类到底是什么。



所以回到我们的问题：**Mybatis的代理对象的类型是什么？**

**
**

本来可以有两个答案：

1. 代理对象对应的代理类
2. 代理对象对应的接口

**
**

那么答案1就相当于没有了，因为是代理类是动态生成的，那么我们来看答案2：**代理对象对应的接口**



如果我们采用答案2，那么我们的思路就是：

```
BeanDefinition bd = new BeanDefinitoin();
// 注意这里，设置的是UserMapper
bd.setBeanClassName(UserMapper.class.getName());
SpringContainer.addBd(bd);
```



但是，实际上给BeanDefinition对应的类型设置为一个接口是**行不通**的，因为Spring没有办法根据这个BeanDefinition去new出对应类型的实例，接口是没法直接new出实例的。



那么现在问题来了，我要解决的问题：**Mybatis的代理对象的类型是什么？**

两个答案都被我们否定了，所以这个问题是无解的，所以我们不能再沿着这个思路去思考了，只能回到最开始的问题：**如何****能够把Mybatis的代理对象作为一个bean放入Spring容器中？**



总结上面的推理：**我们想通过设置BeanDefinition的class类型，然后由Spring自动的帮助我们去生成对应的bean，但是这条路是行不通的。**



## 终极解决方案

那么我们还有没有其他办法，可以去生成bean呢？并且**生成bean的逻辑不能由Spring来帮我们做**了，得由我们自己来做。



### FactoryBean

有，那就是Spring中的FactoryBean。我们可以利用FactoryBean去自定义我们要生成的bean对象，比如：

```
@Component
public class LubanFactoryBean implements FactoryBean {
    @Override
    public Object getObject() throws Exception {
        Object proxyInstance = Proxy.newProxyInstance(LubanFactoryBean.class.getClassLoader(), new Class[]{UserMapper.class}, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                if (Object.class.equals(method.getDeclaringClass())) {
                    return method.invoke(this, args);
                } else {
                    // 执行代理逻辑
                    return null;
                }
            }
        });

        return proxyInstance;
    }

    @Override
    public Class<?> getObjectType() {
        return UserMapper.class;
    }
}
```

**
**

我们定义了一个LubanFactoryBean，它实现了FactoryBean，getObject方法就是用来自定义生成bean对象逻辑的。



执行如下代码：

```
public class Test {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        System.out.println("lubanFactoryBean: " + context.getBean("lubanFactoryBean"));
        System.out.println("&lubanFactoryBean: " + context.getBean("&lubanFactoryBean"));
        System.out.println("lubanFactoryBean-class: " + context.getBean("lubanFactoryBean").getClass());
    }
}
```



将打印：

lubanFactoryBean: com.luban.util.LubanFactoryBean$1@4d41cee

&lubanFactoryBean: com.luban.util.LubanFactoryBean@3712b94

lubanFactoryBean-class: class com.sun.proxy.$Proxy20



从结果我们可以看到，从Spring容器中拿名字为"lubanFactoryBean"的bean对象，就是我们所自定义的jdk动态代理所生成的代理对象。



所以，我们可以通过FactoryBean来向Spring容器中添加一个自定义的bean对象。上文中所定义的LubanFactoryBean对应的就是UserMapper，表示我们定义了一个LubanFactoryBean，相当于把UserMapper对应的代理对象作为一个bean放入到了容器中。



但是作为程序员，我们不可能每定义了一个Mapper，还得去定义一个LubanFactoryBean，这是很麻烦的事情，我们改造一下LubanFactoryBean，让他变得更通用，比如：



```
@Component
public class LubanFactoryBean implements FactoryBean {

    // 注意这里
    private Class mapperInterface;
    public LubanFactoryBean(Class mapperInterface) {
        this.mapperInterface = mapperInterface;
    }

    @Override
    public Object getObject() throws Exception {
        Object proxyInstance = Proxy.newProxyInstance(LubanFactoryBean.class.getClassLoader(), new Class[]{mapperInterface}, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

                if (Object.class.equals(method.getDeclaringClass())) {
                    return method.invoke(this, args);
                } else {
                    // 执行代理逻辑
                    return null;
                }
            }
        });

        return proxyInstance;
    }

    @Override
    public Class<?> getObjectType() {
        return mapperInterface;
    }
}
```



改造LubanFactoryBean之后，LubanFactoryBean变得灵活了，可以在构造LubanFactoryBean时，通过构造传入不同的Mapper接口。



实际上LubanFactoryBean也是一个Bean，我们也可以通过生成一个BeanDefinition来生成一个LubanFactoryBean，并给构造方法的参数设置不同的值，比如伪代码如下：

```
BeanDefinition bd = new BeanDefinitoin();
// 注意一：设置的是LubanFactoryBean
bd.setBeanClassName(LubanFactoryBean.class.getName());
// 注意二：表示当前BeanDefinition在生成bean对象时，会通过调用LubanFactoryBean的构造方法来生成，并传入UserMapper
bd.getConstructorArgumentValues().addGenericArgumentValue(UserMapper.class.getName())
SpringContainer.addBd(bd);
```



特别说一下注意二，表示表示当前BeanDefinition在生成bean对象时，会通过调用LubanFactoryBean的构造方法来生成，并传入UserMapper的Class对象。那么在生成LubanFactoryBean时就会生成一个UserMapper接口对应的代理对象作为bean了。



到此为止，其实就完成了我们要解决的问题：**把Mybatis中的代理对象作为一个bean放入Spring容器中**。只是我们这里是用简单的JDK代理对象模拟的Mybatis中的代理对象，如果有时间，我们完全可以调用Mybatis中提供的方法区生成一个代理对象。这里就不花时间去介绍了。



### Import

到这里，我们还有一个事情没有做，就是怎么真正的定义一个BeanDefinition，并把它**添加**到Spring中，上文说到我们要利用Import技术，比如可以这么实现：



定义如下类：

```
public class LubanImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition();
        AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();
        beanDefinition.setBeanClass(LubanFactoryBean.class);
        beanDefinition.getConstructorArgumentValues().addGenericArgumentValue(UserMapper.class);
        // 添加beanDefinition
        registry.registerBeanDefinition("luban"+UserMapper.class.getSimpleName(), beanDefinition);
    }
}
```



并且在AppConfig上添加@Import注解：

```
@Import(LubanImportBeanDefinitionRegistrar.class)
public class AppConfig {
```



这样在启动Spring时就会新增一个BeanDefinition，该BeanDefinition会生成一个LubanFactoryBean对象，并且在生成LubanFactoryBean对象时会传入UserMapper.class对象，通过LubanFactoryBean内部的逻辑，相当于会自动生产一个UserMapper接口的代理对象作为一个bean。



## 总结

总结一下，通过我们的分析，我们要整合Spring和Mybatis，需要我们做的事情如下：

1. 定义一个LubanFactoryBean
2. 定义一个LubanImportBeanDefinitionRegistrar
3. 在AppConfig上添加一个注解@Import(LubanImportBeanDefinitionRegistrar.class)



## 优化

这样就可以基本完成整合的需求了，当然还有两个点是可以优化的



第一，单独再定义一个@LubanScan的注解，如下：

```
@Retention(RetentionPolicy.RUNTIME)
@Import(LubanImportBeanDefinitionRegistrar.class)
public @interface LubanScan {
}
```

这样在AppConfig上直接使用@LubanScan即可



第二，在LubanImportBeanDefinitionRegistrar中，我们可以去扫描Mapper，在LubanImportBeanDefinitionRegistrar我们可以通过AnnotationMetadata获取到对应的@LubanScan注解，所以我们可以在@LubanScan上设置一个value，用来指定待扫描的包路径。然后在LubanImportBeanDefinitionRegistrar中获取所设置的包路径，然后扫描该路径下的所有Mapper，生成BeanDefinition，放入Spring容器中。



所以，到此为止，Spring整合Mybatis的核心原理就结束了，再次总结一下：

1. 定义一个LubanFactoryBean，用来将Mybatis的代理对象生成一个bean对象
2. 定义一个LubanImportBeanDefinitionRegistrar，用来生成不同Mapper对象的LubanFactoryBean
3. 定义一个@LubanScan，用来在启动Spring时执行LubanImportBeanDefinitionRegistrar的逻辑，并指定包路径



以上这个三个要素分别对象org.mybatis.spring中的：

1. MapperFactoryBean
2. MapperScannerRegistrar
3. @MapperScan