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

获取bean
1、获取BeanDefinition信息
2、调用factoryBean的createBean()方法，createBeanInstance根据情况可能用jdk代理、cglib代理依赖关系  list map array
populateBean方法 注入，做类型转换

 总结：
 Spring ioc
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

 二、Spring DI：
 1.读取beanDefinition获取其依赖关系
 2、实例化(代理对象)
 3、注入：设值
    createBeanInstance() 创建实例  放入到ioc容器中
    populated() 对bean属性的依赖注入进行处理

   如果使用了延时 依赖注入这个动作发生在调用getBean方法的时候
