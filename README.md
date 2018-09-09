学习源码：加注释
ClassPathXmlApplicationContext与FileSystemXmlApplicationContext源码结构图


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

 Spring DI
 1.读取beanDefinition获取其依赖关系
 2、实例化(代理对象)
 3、注入：设值
    createBeanInstance 创建实例  放入到ioc容器中
    populated 对bean属性的依赖注入进行处理

   如果使用了延时 依赖注入这个动作发生在调用getBean方法的时候

   BeanFactory和FactoryBean
   BeanFactory：bean工厂，是一个工厂（factory）我们spring ioc容器的最顶层接口就是这个BeanFactory，他的作用是管理Bean，
    即实例化、定位、配置应用程序中的对象及建立这些对象之间的依赖
   FactoryBean：工厂bean，是一个bean，作用是产生其他bean实例，通常情况下，这种bean没有什么特别的要求，仅需要提供一个工
    厂方法，该方法用来返回bean实例。通常情况，bean无须自己实现工厂模式，spring容器担任工厂角色，但少数情况下，容器中
    的bean本身就是工厂，其作用是产生其它bean实例


   ioc判断，如果被代理的对象实现了一个接口，那么默认用jdk代理
        如果被代理的对象，没有实现一个接口，那么就是cglib代理

   ClassPathXmlApplicationContext，本身就是一个工厂，这个类是我们自己new 出来的，就意味着我们有可能在多线程环境中使用


