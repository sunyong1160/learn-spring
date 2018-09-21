学习源码：加注释
ClassPathXmlApplicationContext与FileSystemXmlApplicationContext源码结构图
ClassPathXmlApplicationContext，本身就是一个工厂，这个类是我们自己new 出来的，就意味着我们有可能在多线程环境中使用


获取bean
1、从abstractBeanFactory中的doGetBean方法中获取
2、AbstractAutowireCapableBeanFactory创建bean实例对象 creatBean方法


spring中生成的bean一般是代理模式生成的，默认cglib，拥有这个代理类的控制权

SimpleInstantiationStrategy

BeanDefinitionValueResolver解析属性值 数据拷贝：原型模式

ioc容器存储bean对象是用concurrentHashMap存储的，FactoryBeanRegisterSupport
真正的ioc容器 factoryBeanObjectCache  map

 总结：
 Spring ioc  ClassPathXmlApplicationContext和FileSystemXmlApplicationContext入口 顶层接口BeanFactory
 private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
 1.定位配置文件
 2.载入 读取配置文件
 3.注册 把加载好的配置文件解释成beandefinition
 ----------------
 一、ioc 容器初始化的基本步骤：
 1、初始化的入口在容器实现中的refresh()调用来完成.
 2、对bena定义载入IOC容器使用的方法是loadBeanDefinition().其中的大致过程：通过ResourceLoader来完成资源文件位置的定位
    DefaultResourceLoader是默认的实现，同时上下文本身就给出了ResourceLoader的实现，可以从类路径，文件系统，URL等方式
    来定位资源位置。如果是XmlBeanFactory作为IOC容器，那么需要为它指定bean定义的资源，也就是说bean定义为文件时通过抽象成
    Resource来被IOC容器处理的，容器通过BeanDefinitionReader来完成定义信息的解析和Bean信息的注册，往往使用的是XMLBeanDefinitionReader
    来解析bean的xml定义文件-实际的处理过程是委托给BeanDefinitionParserDelegate来完成的，从而得到bean的定义信息，这些信息
    在Spring中使用BeanDefinition对象来表示这个名字可以让我们想到loadBeanDefinition，RegisterBeanDefinition 这些相关方法
    他们都是为处理BeanDefinition服务的，容器解析得到BeanDefinitionIoC以后需要把它在IOC容器中注册，这由IOC实现，BeanDefinitionRegister接口来实现，
    注册过程就是IOC容器内部维护一个HashMap来保存得到的BeanDefinition的过程。这个HashMap是IOC容器持有bean信息的场所，以后对bean的
    操作都是围绕这个HashMap来实现的
 3、然后我们就可以通过BeanFactory和ApplicationContext来享受到SpringIOC的服务了，在使用IOC容器的时候，我们注意到除了少量粘合代码，
    绝大多数以正确IOC风格编写的应用程序代码完全不用关心到如何达到工厂，因为容器将把这些对象与容器管理的其他对象挂钩在一起，以及代码实
    际需要访问工厂的地方。Spring本身提供了声明式载入web应用程序用法的应用程序上下文，并将其存储在ServletContext中的框架实现。

  在使用Spring IOC容器的时候我们还需要区别两个概念：
    BeanFactory和FactoryBean，其中BeanFactory指的是IOC容器的编程抽象，比如ApplicationContext，XmlBeanFactory等，这些都是IOC
    的具体表现，需要使用什么样的容器由客户决定，但Spring为我们提供了丰富的选择。FactoryBean只是一个可以在IOC容器中被管理的一个bean，
    是对各种处理过程和资源使用的抽象，FactoryBean在需要时产生另一个对象，而不会返回FactoryBean本身，我们可以把它看成是一个抽象工厂，
    对它的调用返回的是工厂生产的产品。所有的FactoryBean都实现特殊的org.springframework.beans.factory.FactoryBean接口，当使用容器中FactoryBean
    的时候，该容器不会返回FactoryBean本身，而是返回其生成的对象。Spring包括了大部分的通用资源和服务访问抽象的FactoryBean的实现，其中baokuo
    对JNDI查询的处理，对代理对象的处理，对事务性代理的处理，对RMI代理的处理等，这些我们都可以看成是具体的工厂，看成是Spring为我们建立好的
    工厂。也就是说Spring通过使用抽象工厂模式为我们准备了一系列工厂来生产一些特定的对象，免除我们手工重复的工作，我们要使用时只需要在IOC
    容器里配置好就能很方便的使用了。

 IOC容器的依赖注入：
   1、依赖注入发生的时间
      当Spring IOC容器完成了Bean定义资源的定位、载入和解析注册以后，IOC容器中已经管理类Bean定义的相关数据，但是此时IOC容
      器还没有对所管理的Bean进行依赖注入，依赖注入在以下两种情况下发生：
      (1).用户第一次通过getBean()方法向IOC容器索要Bean时，IOC容器触发依赖注入。
      (2).当用户在Bean定义资源中为<bean></bean> 元素配置了lazy-init属性，即让容器在解析注册Bean定义时进行预实例化，触发依赖注入。

      BeanFactory接口定义Spring IOC容器的基本功能规范，是Spring IOC容器所应遵守的最底层和最基本的编程规范。BeanFactory接口中
      定义了几个getBean()方法，就是用户向IOC容器索取管理的Bean的方法，我们通过分析其子类的具体实现，理解Spring IOC容器在用户索取Bean时
      如何完成依赖注入。

 BeanFactory和FactoryBean区别：
    BeanFactory：bean工厂，是一个工厂（factory）我们spring ioc容器的最顶层接口就是这个BeanFactory，他的作用是管理Bean，
    即实例化、定位、配置应用程序中的对象及建立这些对象之间的依赖。
    FactoryBean：工厂bean，是一个bean，作用是产生其他bean实例，通常情况下，这种bean没有什么特别的要求，仅需要提供一个工
    厂方法，该方法用来返回bean实例。通常情况，bean无须自己实现工厂模式，spring容器担任工厂角色，但少数情况下，容器中
    的bean本身就是工厂，其作用是产生其它bean实例。




 ioc判断，如果被代理的对象实现了一个接口，那么默认用jdk代理
         如果被代理的对象，没有实现一个接口，那么就是cglib代理

############################################################

 二、Spring DI： AbstractBeanFactory和AbstractAutowireCapableBeanFactory
 private final Map<String, Object> factoryBeanObjectCache = new ConcurrentHashMap<>(16);
 1.读取beanDefinition获取其依赖关系
 2、实例化(代理对象)
 3、注入：设值
    createBeanInstance() 创建实例  放入到ioc容器中
    populateBEan() 对bean属性的依赖注入进行处理

 获取bean
 1、获取BeanDefinition信息
 2、调用factoryBean的createBean()方法，createBeanInstance根据情况可能用jdk代理、cglib代理依赖关系  list map array
 populateBean方法 注入，做类型转换

   如果使用了延时 依赖注入这个动作发生在调用getBean方法的时候


   ======================================


   AOP
   proxyFactory
   因为IOC容器中存放的就是FactoryBean，AOP工厂生产出来的Bean是存放在IOC容器中去的。

   IOC也是Factory入手的，找到一个getBean()的方法
   AOP的ProxyFactory中的getProxy，这个方法就是用来获取一个代理以后的Bean，跟IOC体系里面的getBean()异曲同工之处。
   AOP是依赖于IOC容器的，先等我们IOC容器启动以后，进行二次操作，
   AOP不要执行上面的那些创建对象的操作了，只要能够拿到IOC容器的引用，直接从IOC容器中取出需要被二次操作的所有的对象。
   IOC里已经是代理类了，那么AOP中，是对代理类的二次深操作，每个动作都要被管控。IOC里除了是原型的，都是被代理的类

   主流程可以简述为：获取可以应用到此方法上的通知链(Interceptor Chain)，如果有，则应用通知，并执行JoinPoint；
                    如果没有，则直接反射执行joinpoint。
   首先，通知链是通过Advised.getInterceptorAndDynamicInterceptionAdvice()这个方法来获取的。

   invocationHandler这个是JDK提供的，做动态代理必须实现的接口
   由切点进入切面，切点实际就是Method，通知的操作，
   切点，转换为MethodInterceptor，保存到一个容器里面，这个容器一定是一个链表结构，一定是有顺序的，它知道它的上一个是谁，下一个是谁
    MethodInterceptor的容器是List

    AOP流程：
    1、加载配置信息，解析成AopConfig
    2、交给AopProxyFactory，调用一个createAopProxy的方法

    DefaultAdvisorChainFactory

    JdkDynamicAopProxy调用AdvisedSupport的getInterceptorsAndDynamicInterceptionAdvice方法得到方法拦截器，并保存到一个容器

    递归执行拦截器方法proceed()方法


   ======================================
    SpringMVC
    初始化：onRefresh();调用doService()方法
    HandlerMapping和HandlerAdapters都是通过注解扫描得来的保存在一个工厂里面


设计模式                                    应用场景                                                            一句话归纳

代理模式                    1.两个参与角色：执行者（代理人），被代理人                                     办事要求人，所以找代理
(proxy)                     2.对于被代理人来说，这件事情是一点要做的，但是我自己又不想去做，
                               或者没有时间去做，找代理。
                            3.代理人必须需要获取被代理人的个人资料（持有被代理人的引用）

工厂模式                    1.对于调用者来说，隐藏了复杂的逻辑处理过程，调用者只关心执行结果                只需对结果负责，不要三无产品
factory                     2.对于工厂来说要对结果负责，保证生产出符合规范的产品

单例模式                    1.保证从系统启动到系统停止，全过程只会产生一个实例                              保证独一无二
singleton                   2.当我们在应用中遇到功能性冲突的时候，需要使用单例模式

委派模式                    1.两个参与角色：委托人和被委托人                                               干活是你的(普通员工)
delegate                    2.委托人和被委托人在权力上是平等的(即实现同一接口)                              功劳是我的(项目经理)
                            3.委托人持有被委托人的引用
                            4.不关心过程，只关心结果

策略模式                    1.最终执行结果是固定的                                                         我行我素，达到目的即可
strategy                    2.执行过程和执行逻辑不一样

原型模式                    1.首先有一个原型                                                               拔一根猴毛，吹出千万个
prototype                   2.数据内容相同，但对象实例不同(完全两个个体)

模板模式                    1.执行流程是固定，但中间有些步骤有细微差别(运行时才确定)                         流程标准化，原料自给加
template                    2.可实现批量生产


SpringMvc的优化：
1、controller如果能保持单例，尽量使用单例，这样可以减少创建对象和回收对象的开销，也就是说，如果controller的类变量和实例变量可以以方法形参声明的
    尽量以方法的形参声明，不要以类变量和实例变量声明，这样可以避免线程安全问题
2、处理request的方法中的形参务必加上@RequestParam注解，这样可以避免springmvc使用asm框架读取class文件获取方法参数名的过程，即便springmvc对读取出
    的方法参数名进行缓存，如果不要读取class文件当然是更加好.
3、阅读源码的过程中，发现springmvc并没有对处理url的方法进行缓存，也就是说每次都要根据请求url去匹配controller中的方法url，如果url和method的关系缓存起来，
    会不会带来性能上的提升呢？有点恶心的是，负责解析url和method对应关系的ServletHandlerMethodResolver是一个private的内部类，不能直接继承该类增强代码，
    必须要改代码后重新编译。当然，如果缓存起来，必须要考虑缓存的线程安全问题


spring事务总结：
1、什么是事务:
    一个整体的执行逻辑单元，只有两个结果，要么全成功，要么全失败
2、事务的特性:
    原子性、隔离性、持久性、一致性
3、事务的基本原理:
    从数据库角度来说:就是提供了一种后悔机制(代码写错了，可以SVN、Git)
    使用临时表实现后悔,执行增、删、改之前，先将满足条件的数据查询出来放入到临时表中
    将数据操作现在临时表中完成，完成过程中如果没有出现任何问题，就将数据同步(剪切)实际的数据表中，并返回影响行数
    将数据操作在临时表中完成，完成过程中一旦出现错误，那就将临时表中满足条件的数据清掉，并返回错误码

    两个都是增删改操作，就会出现死锁(死锁超时，就需要人工解决)
    杀进程(写SQL语句杀死进程)

    如果要想对一个数据表的数据进行清空(千千万万别用delete from，这种情况，一定就是锁表)
    加了where条件，就是行锁

4、Spring的事务管理
    AOP配置，配置哪些方法需要加事务
    声明式事务配置，事务的传播属性、隔离级别、回滚条件

    传播属性: 1.default 2.required 3.request_new 4.supports 5.nested
    隔离级别: 1.default 2.read_uncommited 3.read_commited 4.repeatable_read 5.serializable

5、源码 TransactionDefinition  PlatformTransactionManager  TransactionStatus
    通过解析配置文件得到TransactionDefinition，实际上就是AOP中的methodInteceptor(方法代理)
    就可以在满足条件的方法调用之前和调用之后加一些东西

    platformTransactionManager中的方法
    getTransaction 调用了TransactionSynchronizationManager类的getResource()
    从ThreadLocal里面取值，Map<Key:DataSource,Value:ConnectionHolder(相当于获取一个连接对象Connection)>

    conn.setAutoCommit

    Commit conn.commit();