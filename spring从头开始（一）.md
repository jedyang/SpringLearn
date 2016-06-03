### 目的：简化java企业级开发

### 途径：
- 轻量级，使用pojo减少开发中的侵入性
- 通过DI和面向接口编程实现松耦合
- 通过切面和管理实现声明式编程
- 通过切面和模板代码来消除样本文件

spring尽量不干扰你正常的业务代码，你不需要继承或者实现spring的接口。基本感受不到spring的存在，当然会有spring的注解，但还是一个POJO。

一个POJO是spring的一个component，然后通过spring的DI，对POJO进行组装，释放POJO的威力，实现松耦合。

### DI是如何工作的
通常，最挫的代码是直接维护自己的实现。这样业务一旦复杂起来，产生的问题及时代码难以测试，难以复用，难以修改。
另一种是，不要直接自己new一个实现，而是基于接口，通过构造器，或者更常用的setter方法注入实现。好处是基于接口，松耦合。易于测试，直接mock即可。
现在DI交给spring来做。spring有两种常用方式织入依赖。
通过xml文件 vs 通过注解。
还有一种，通过java类表示xml配置。用的人不多。

配置好依赖后，applicationContext开始载入配置文件，根据依赖配置，生产bean。

### aspect
所谓切面，即是指独立于核心业务之外的通用服务，如日志，安全，事务管理等。
如果不使用面向切面，你可能会面临两个问题，一是这种通用服务会在你应用的各个模块中都有使用，一旦变更，需要修改各个模块中的代码。即使是面向接口编程，对通用服务的调用还是会散落在各个模块。二是这种通用服务的代码会扰乱你正常的业务代码。
spring的aop模块化这些通用服务，在业务模块中可以声明式的使用。使各个模块更内聚，专注于自己的核心业务。保持POJO的纯粹性。
在配置文件中配置aop的方法示例：

```
<bean id="minstrel" class="com.springinaction.knights.Minstrel">
  <constructor-arg value="#{T(System).out}" />
</bean> Declare Minstrel bean
  <aop:config>
  <aop:aspect ref="minstrel">
    <aop:pointcut id="embark"
        expression="execution(* *.embarkOnQuest(..))"/>
Define pointcut
        <aop:before pointcut-ref="embark"
          method="singBeforeQuest"/>
      <aop:after pointcut-ref="embark"
          method="singAfterQuest"/>
    </aop:aspect>
  </aop:config>
  
```

### Spring容器
spring容器是bean的生存环境。控制着bean的生命周期。
spring容器是整个spring框架的核心，它使用DI去管理bean，组合成整个应用。
spring容器可以被分为两类。
1. bean factory。实现*org.springframework.beans.factory.BeanFactory*接口。提供了对DI的支持。是最基础的spring容器。
2. application context。实现*org.springframework.context.ApplicationContext*接口。在bean factory容器概念基础上提供spring框架级的服务，如读取properties文件，发布应用事件。
bean factory这类容器太低级，很难满足大多数应用，我们用的基本都是application context这类。

有几种常用的application context，

- AnnotationConfigApplicationContext—Loads a Spring application context from one or more Java-based configuration classes

- AnnotationConfigWebApplicationContext—Loads a Spring web application context from one or more Java-based configuration classes

- ClassPathXmlApplicationContext—Loads a context definition from one or more XML files located in the classpath, treating context-definition files as class- path resources

- FileSystemXmlApplicationContext—Loads a context definition from one or more XML files in the filesystem

- XmlWebApplicationContext—Loads context definitions from one or more XML files contained in a web application

### spring容器中bean的生命周期

![盗图一张](http://img.blog.csdn.net/20160507224033117)
1. 初始化bean。只是开辟一块内存空间。
2. 为bean的属性赋值
3. **如果**这个bean实现了*BeanNameAware*接口，那么要执行他的setBeanName()方法。
4. **如果**这个bean实现了*BeanFactoryAware*接口，那么去执行他的setBeanFactory()方法。
5. **如果**这个bean实现了*ApplicationContextAware*接口，那么去执行他的setApplicationContext()方法。
6. **如果**这个bean实现了*BeanPostProcessor*接口，那么去执行setProcessBeforeInitialization()方法。
7. **如果**这个bean实现了*InitializingBean*接口，那么去执行afterPropertiesSet()方法。
8. 执行init方法，如果有。
9. **如果**这个bean实现了*BeanPostProcessor*接口，那么去执行setProcessAfterInitialization()方法。
10. 现在bean可用了。直到application context销毁。
11. **如果**这个bean实现了*DisposableBean*接口，那么销毁时回去执行他的destory()方法。同样的，如果这个bean定义了自己的销毁方法，也会得到执行。

### Spring概览
![spring模块](http://img.blog.csdn.net/20160507230422061)
spring核心：

1. spring的核心是三个：core，context，bean。要在这三个中再找一个更核心的那就是bean。因为整个spring的设计都是围绕bean来进行的，spring可以称为是BOP（面向bean编程）。  
为什么bean如此重要？回想一下最初为什么要用spring？是为了将我们从java的依赖管理中解脱出来，spring做到了这一点，怎么做呢？将java的object封装在一个称谓bean的数据结构内，然后围绕这个bean构建他的生命周期，生存环境。其他的框架一般也是这样的设计理念。  

2. 组件间的交互  
演出一台舞台剧，要有各种演员，要用剧本，还要有道具。bean是演员，context就可以比喻成剧本，配置了各个bean之间的关系。context就是bean关系的集合，这个关系集合就是IOC容器。  
core提供了这种发现，建立，维护bean关系的各种工具，其实core叫util更好理解。  
所以他们三个合在一起就是为了解决那个问题：维护object间的依赖关系。  

3. 当 Spring 成功解析你定义的一个 <bean/> 节点后，在 Spring 的内部他就被转化成 BeanDefinition 对象。以后所有的操作都是对这个对象完成的。  
4. Context 组件  
context组件的核心功能是给 Spring 提供一个运行时的环境，用以保存各个对象的状态。  
总体来说 ApplicationContext 必须要完成以下几件事：  
	* 标识一个应用环境  
	* 利用 BeanFactory 创建 Bean 对象
	* 保存对象关系表
	* 能够捕获各种事件  
Context 作为 Spring 的 Ioc 容器，基本上整合了 Spring 的大部分功能，或者说是大部分功能的基础。

5. Core组件  
关键词：资源resource。加载资源


