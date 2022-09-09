## 一、IoC
### 什么是IoC

**容器概念、控制反转、依赖注入**
- IoC容器
	- ioc容器实际上就是个map（key，value），里面存的是各种对象（在xml里配置的bean节点、@repository、@service、@controller、@component）
	- 在项目启动的时候会读取配置文件里面的bean节点，根据全限定类名使用反射创建对象放到map里、扫描到打上上述注解的类还是通过反射创建对象放到map里。
	- 这个时候map里就有各种对象了，接下来我们在代码里需要用到里面的对象时，再通过DI注入
- 控制反转：全部对象的控制权全部上缴给“第三方”IOC容器。
- 依赖注入：控制被反转之后，获得依赖对象的过程由自身管理变为了由IOC容器主动注入。依赖注入是实现IOC的方法，就是由IOC容器在运行期间，动态地将某种依赖关系注入到对象之中。

### BeanFactory、FactoryBean 和 ApplicationContext 的区别？
BeanFactory是接口，提供了IOC容器最基本的形式，给具体的IOC容器的实现提供了规范，

FactoryBean也是接口，为IOC容器中Bean的实现提供了更加灵活的方式，FactoryBean在IOC容器的基础上给Bean的实现加上了一个简单工厂模式和装饰模式(如果想了解装饰模式参考：修饰者模式(装饰者模式，Decoration) 我们可以在getObject()方法中灵活配置。其实在Spring源码中有很多FactoryBean的实现类。

BeanFactory是个Factory，也就是IOC容器或对象工厂，FactoryBean是个Bean。在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的。但对FactoryBean而言，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似 

ApplicationContext：应用上下文，继承BeanFactory接口，它是Spring的一各更高级的容器，提供了更多的有用的功能；

> factory-bean和factory-method（@Configuration和@Bean）
```java
//实例工厂调用
public class UserFactory {

	public User getUser(){
		return new User();
	}

}
```
```java
<!-- 使用实例工厂进行创建 -->
<!-- 需要先创建factoryBean对象，再通过factoryBean对象进行调用 -->
<bean id="userFactory" class="com.imooc.entity.factory.UserFactory"/>
<bean id="user3" factory-bean="userFactory" factory-method="getUser" scope="prototype" />

```
### Spring支持的几种bean的作用域？
- singleton：默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护。该对象的生命周期是与Spring IOC容器一致的（但在第一次被注入时才会创建）。
- prototype：为每一个bean请求提供一个实例。在每次注入时都会创建一个新的对象
- request：bean被定义为在每个HTTP请求中创建一个单例对象，也就是说在单个请求中都会复用这一个单例对象。
- session：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。
- application：bean被定义为在ServletContext的生命周期中复用一个单例对象。
### @Autowired和@Resource的区别是什么？
共同点：使用这2种注解都可以实现自动装配！

区别：
- @Resource注解是javax包中的注解，它是 **`优先byName来装配`** 的，如果byName无法装配，则会 **`自动尝试byType装配`** ，在byType装配时， **`要求匹配类型的对象必须有且仅有1个，如果无法装配，则会报告错误。`**
- @Autowired注解是Spring框架中的注解，它是 **`优先byType来装配`** 的
	- 这个过程中，只会检索匹配类型的对象的数量，并不直接装配
	- 如果找到的对象的数量是0个，则直接报错
	- 如果找到的对象的数量是1个，则直接装配
	- 如果找到的对象的数量（类型）超过1个（2个或更多个），则会尝试byName来装配，如果byName装配失败，则报错。
### @Qualifier 注解有什么作用？
当需要创建多个相同类型的 Bean 并希望仅使用属性装配其中一个 Bean 时，可以使用@Qualifier注解和 @Autowired 通过指定应该装配哪个 Bean 来消除歧。

```java
@Component("bird")
public class Bird implements Fly {...}

@Component("plane")
public class Plane implements Fly {...}

@Component
public class FooService {
    @Autowired
    @Qualifier("bird")
    private Fly fly;
}
```
### @Bean和@Component有什么区别？
都是使用注解定义 Bean。两者的区别：

- @Bean 是使用 Java 代码装配 Bean，@Component 是自动装配 Bean。

- @Component 注解用在类上，表明一个类会作为组件类，并告知Spring要为这个类创建Bean，每个类对应一个 Bean。而@Bean 注解用在方法上，表示这个方法会返回一个 Bean。@Bean 需要在配置类中使用，即类上需要加上@Configuration注解。
### Bean注入容器有哪些方式？
将普通类交给Spring容器管理，通常有以下方法：

- 使用@Configuration与@Bean注解；

- 使用@Controller、@Service、@Repository、@Component 注解标注该类，然后启用@ComponentScan自动扫描；

- 使用@Import 方法。使用@Import注解把Bean导入到容器中，代码如下：

	```java
	@ComponentScan
	@Import({Demo.class})
	public class App {
	    public static void main(String[] args) throws Exception {
	        ConfigurableApplicationContext context = SpringApplication.run(App.class, args);
	        System.out.println(context.getBean(Demo.class));
	        context.close();
	    }
	}
	```

### Spring框架中的单例Bean是线程安全的么？

Spring中的Bean默认是单例模式的，框架并没有对bean进行多线程的封装处理。

如果Bean是有状态的 那就需要开发人员自己来进行线程安全的保证，最简单的办法就是改变bean的作用域 把 "singleton"改为’‘protopyte’ 这样每次请求Bean就相当于是 new Bean() 这样就可以保证线程的安全了。
- 有状态就是有数据存储功能
- 无状态就是不会保存数据 controller、service和dao层本身并不是线程安全的，只是如果只是调用里面的方法，而且多线程调用一个实例的方法，会在内存中复制变量，这是自己的线程的工作内存，是安全的。

Dao会操作数据库Connection，Connection是带有状态的，比如说数据库事务，Spring的事务管理器使用Threadlocal为不同线程维护了一套独立的connection副本，保证线程之间不会互相影响（**Spring是如何保证事务获取同一个Connection的?**）

不要在bean中声明任何有状态的实例变量或类变量，如果必须如此，那么就使用ThreadLocal把变量变为线程私有的，如果bean的实例变量或类变量需要在多个线程之间共享，那么就只能使用synchronized、lock、CAS等这些实现线程同步的方法了。
### Spring容器初始化流程？
![在这里插入图片描述](https://img-blog.csdnimg.cn/fc53c577346b45869445725ff10e9d72.png)
- 将xml或者注解等配置信息读取到内存中
- 在内存中这些配置文件会当作Resource对象
- 之后会将Resource对象解析成BeanDefination实例
- 最后将BeanDefination注册到容器中

**`DefaultListableBeanFactory ：存储容器中所有已经注册的BeanDefination的载体`**
### Spring 中的 bean 生命周期? 
Spring中Bean的生命周期主要在创建和销毁两个时期。

![在这里插入图片描述](https://img-blog.csdnimg.cn/41ab25fe8dd44447862387c65348ff28.png)
**创建过程：**
- 实例化bean对象，以及设置bean属性
- 如果通过Aware接口声明了依赖关系，则会注入Bean对容器基础设施层面的依赖，Aware接口是为了感知到自身的一些属性。
- 紧接着会调用BeanPostProcess的前置初始化方法postProcessBeforeInitialization，主要作用是在Spring完成实例化之后，初始化之前，对Spring容器实例化的Bean添加自定义的处理逻辑。有点类似于AOP。
- 如果实现了BeanFactoryPostProcessor接口的afterPropertiesSet方法，做一些属性被设定后的自定义的事情。
- 调用Bean自身定义的init方法，去做一些初始化相关的工作。
- 调用BeanPostProcess的后置初始化方法，postProcessAfterInitialization去做一些bean初始化之后的自定义工作。
- 完成以上创建之后就可以在应用里使用这个Bean了。

**销毁过程：**当Bean不再用到，便要销毁
- 若实现了DisposableBean接口，则会调用destroy方法；
- 若配置了destry-method属性，则会调用其配置的销毁方法；

大致分为五个阶段：
- 创建前准备
- 实例化
- 依赖注入
- 容器缓存
- 实例销毁

注意区分：
- 实例化：是对象创建的过程。比如使用构造方法new对象，为对象在内存中分配空间。
- 初始化：是为对象中的属性赋值的过程。
### 什么是BeanDefination？
以注解类变成Spring Bean为例，Spring会扫描指定包下面的Java类，然后将其变成beanDefinition对象，然后Spring会根据beanDefinition来创建bean，特别要记住一点， **`Spring是根据beanDefinition来创建Spring bean的`** 。
![在这里插入图片描述](https://img-blog.csdnimg.cn/b7fd2a3a067c40719637d93061b4e790.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_10,color_FFFFFF,t_70,g_se,x_16)

### 什么是Aware接口？有什么作用？
**`Aware接口是为了感知到自身的一些属性`** 。容器管理的Bean一般不需要知道 **`容器的状态和直接使用容器`** 。但是在某些情况下是需要在Bean中对IOC容器进行操作的。这时候需要在bean中设置对容器的感知。SpringIOC容器也提供了该功能，它是通过特定的Aware接口来完成的。
- 比如BeanNameAware接口，可以知道自己在容器中的名字。
- 比如Bean实现了BeanNameAware接口，调用setBeanName()方法，传入Bean的名字。
- 比如Bean实现了BeanClassLoaderAware接口，调用setBeanClassLoader()方法，传入ClassLoader对象的实例。
- 比如Bean实现了BeanFactoryAware接口，调用setBeanFactory()方法，传入BeanFactory对象的实例。

### 谈谈如何推断构造方法？TODO
### 说说依赖注入流程？TODO
### 说说Spring中的三级缓存？TODO
### Spring中出现同名bean怎么办？
- 同一个配置文件内同名的Bean，以最上面定义的为准
- 不同配置文件中存在同名Bean，后解析的配置文件会覆盖先解析的配置文件
- 同文件中@ComponentScan和@Bean出现同名Bean。同文件下@Bean的会生效，@ComponentScan扫描进来不会生效。通过@ComponentScan扫描进来的优先级是最低的，原因就是它扫描进来的Bean定义是最先被注册的
###  依赖注入的相关注解？
- @Autowired ：⾃动按类型注⼊，如果有多个匹配则按照 byName 查找，查找不到会报错。
- @Qualifier ：在⾃动按照类型注⼊的基础上再按照 Bean 的 id 注⼊，给变量注⼊时必须搭配@Autowired ，给⽅法注⼊时可单独使⽤。
- @Resource：直接按照 Bean 的 id 注⼊，只能注⼊ Bean 类型。
- @Value：⽤于注⼊基本数据类型和 String 类型。
### 谈谈BeanPostProcessor？
该接口我们也叫后置处理器，作用是在Bean对象在 **`实例化和依赖注入完毕后`** ，在显示 **`调用初始化方法的前后`** 添加我们自己的逻辑。注意是 **`Bean 实例化完毕后及依赖注入完成后触发的`** 。接口的源码如下：

```java
public interface BeanPostProcessor {
	
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
```
- postProcessBeforeInitialization：实例化、依赖注入完毕，在调用显示的初始化之前完成一些定制的初始化任务。
- postProcessAfterInitialization：实例化、依赖注入、初始化完毕时执行。

[https://blog.csdn.net/Frankltf/article/details/100522191?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164618360016780271958199%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=164618360016780271958199&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-7-100522191.pc_search_result_cache&utm_term=BeanPostProcessor%E5%92%8CAop%E7%9A%84%E5%85%B3%E7%B3%BB&spm=1018.2226.3001.4187](https://blog.csdn.net/Frankltf/article/details/100522191?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164618360016780271958199%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=164618360016780271958199&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-7-100522191.pc_search_result_cache&utm_term=BeanPostProcessor%E5%92%8CAop%E7%9A%84%E5%85%B3%E7%B3%BB&spm=1018.2226.3001.4187)

[https://blog.csdn.net/aaa_bbb_ccc_123_456/article/details/103857758](https://blog.csdn.net/aaa_bbb_ccc_123_456/article/details/103857758)

[https://blog.csdn.net/wyl6019/article/details/80136000?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164618525216781685388602%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=164618525216781685388602&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-3-80136000.pc_search_result_cache&utm_term=springAop%E8%BF%87%E7%A8%8B&spm=1018.2226.3001.4187](https://blog.csdn.net/wyl6019/article/details/80136000?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164618525216781685388602%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=164618525216781685388602&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-3-80136000.pc_search_result_cache&utm_term=springAop%E8%BF%87%E7%A8%8B&spm=1018.2226.3001.4187)

## 二、Spring AOP
### 什么是AOP？

AOP 即⾯向切⾯编程，简单地说就是将 **`代码中重复的部分抽取出来，在需要执⾏的时候使⽤动态代理技术，在不修改源码的基础上对⽅法进⾏增强。`** **AOP 中的基本单元是 Aspect(切面)。**


Spring 根据类是否实现接⼝来判断动态代理⽅式
- 如果实现接⼝会使⽤ JDK 的动态代理，**`核⼼是InvocationHandler 接⼝和 Proxy 类`** 。
- 如果没有实现接⼝会使⽤ CGLib 动态代理，CGLib 是在运⾏时动态⽣成某个类的⼦类，**`如果某个类被标记为 final，不能使⽤ CGLib`**  。

常⽤场景包括权限认证、⾃动缓存、错误处理、⽇志、调试和事务等。
### AOP相关术语？
- Aspect ：切⾯，通知和切点的结合，通知和切点共同定义了切面的全部内容。
- Joinpoint ：连接点，类里面哪些方法可以被增强，这些方法就被成为连接点。
- Pointcut ：切⼊点，实际被真正增强的方法被称为切入点。切⼊点⼀定是连接点，但连接点不⼀定是切⼊点。
- **Advice ：通知，指切⾯对于某个连接点所产⽣的动作，包括 `前置通知、后置通知、返回后通知、异常通知和环绕通知` 。**
- Proxy ：代理，Spring AOP 中有 JDK 动态代理和 CGLib 代理，⽬标对象实现了接⼝时采⽤ JDK 动态代理，反之采⽤ CGLib 代理。
- Target ：代理的⽬标对象，指⼀个或多个切⾯所通知的对象。
- Weaving：织⼊，指把增强应⽤到⽬标对象来创建代理对象的过程。
### AOP 有哪些实现方式？
实现 AOP 的技术，主要分为两大类：

- 静态代理：**`代理类在编译阶段生成`** ， **`在编译阶段将通知织入Java字节码中`** ，也称编译时增强。AspectJ使用的是静态代理。。
- 动态代理：代理类在程序运行时创建，AOP框架不会去修改字节码，而是在 **`内存中临时生成一个代理对象，在运行期间对业务方法进行增强，不会生成新类`** 。
	- **JDK 动态代理**：通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口 。 **`JDK动态代理的核心是 InvocationHandler 接口和 Proxy 类 。`**
	- **CGLIB 动态代理**： 如果目标类没有实现接口，那么 Spring AOP 会选择使用 CGLIB 来动态代理目标类 。CGLIB （ Code Generation Library ），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，  **`CGLIB 是通过继承的方式做的动态代理，因此如果某个类被标记为 final ，那么它是无法使用 CGLIB 做动态代理的`** 。

静态代理缺点：代理对象需要与目标对象实现一样的接口，并且实现接口的方法，会有冗余代码。同时，一旦接口增加方法，目标对象与代理对象都要维护。

### 详细说说CGlib动态代理？
如果目标类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。

CGLIB（Code Generation Library）可以在 **`运行时动态生成类的字节码，动态创建目标类的子类对象，在子类对象中增强目标类。`**  **`也就是通过继承实现`** 。

CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。


### Spring AOP and AspectJ AOP 有什么区别？
- Spring AOP 基于动态代理方式实现；AspectJ 基于静态代理方式实现。
- Spring AOP 仅支持 **`方法级别`** 的 PointCut；AspectJ 提供了完全的 AOP 支持，它还 **`支持属性级别`** 的 PointCut。
### Spring中AOP过程 TODO



## 三、Spring事务
### Spring事务的实现方式、实现原理以及隔离级别？重要

**实现方式：**
- 在使用Spring框架时，可以有两种使用事务的方式，一种是 **`编程式`** 的，一种是 **`申明式`** 的， **`@Transactional注解就是申明式的。`**
- 首先，事务这个概念是数据库层面的，Spring只是基于数据库中的事务进行了扩展，以及提供了一些能
让程序员更加方便操作事务的方式。
- 我们可以通过在 **`某个方法上增加@Transactional注解`** ，就可以开启事务，这个方法中所有的sql都
会在一个事务中执行，统一成功或失败。

**实现原理：**
- 在一个方法上加了@Transactional注解后，**`Spring会基于这个类生成一个代理对象，会将这个代理对象作为bean`** 。
- 在使用这个代理对象的方法时，如果这个方法上存在@Transactional注解，那么代理逻辑会先把事务的自动提交设置为false，然后再去执行原本的业务逻辑方法：
	- 如果执行业务逻辑方法没有出现异常，那么代理逻辑中就会将事务进行提交；
	- 如果执行业务逻辑方法出现了异常，那么则会将事务进行回滚。
- **`针对哪些异常回滚事务是可以配置的，可以利用@Transactional注解中的rollbackFor属性进行配置，默认情况下会对RuntimeException和Error进行回滚`** 。

**隔离级别：**
- spring事务隔离级别就是数据库的隔离级别：外加一个默认级别
	- read uncommitted（未提交读）
	- read committed（提交读、不可重复读）
	- repeatable read（可重复读）
	- serializable（可串行化）

数据库的配置隔离级别是Read Commited,而 **`Spring配置的隔离级别是Repeatable Read`** ，请问这时隔离级别是以哪一个为准？
- **`以Spring配置的为准，如果spring设置的隔离级别数据库不支持，效果取决于数据库`**
### spring事务的传播机制：多个事务方法相互调用时,事务如何在这些方法间传播？

方法A是一个事务的方法，方法A执行过程中调用了方法B，那么方法B有无事务以及方法B对事务的要求不同都会对方法A的事务具体执行造成影响，同时方法A的事务对方法B的事务执行也有影响，这种影响具体是什么就由两个方法所定义的事务传播类型所决定。
- **`REQUIRED(Spring默认的事务传播类型)`** ：如果当前没有事务，则自己新建一个事务，如果当前存在事务，则加入这个事务。
- **`SUPPORTS`** ：当前存在事务，则加入当前事务，如果当前没有事务，就以非事务方法执行。
- **`MANDATORY`** ：当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常。
- **`REQUIRES_NEW`** ：创建一个新事务，如果存在当前事务，则挂起该事务。
- **`NOT_SUPPORTED`** ：以非事务方式执行,如果当前存在事务，则挂起当前事务
- **`NEVER`** ：不使用事务，如果当前事务存在，则抛出异常
- **`NESTED`** ：如果当前事务存在，则在嵌套事务中执行，否则REQUIRED的操作一样（开启一个事务）

### NESTED 和 REQUIRES_NEW的区别？

REQUIRES_NEW是新建一个事务并且新开启的这个事务与原有事务无关。而NESTED则是当前存在事务时（我们把当前事务称之为父事务）会开启一个嵌套事务（称之为一个子事务）。
 
在NESTED情况下父事务回滚时，子事务也会回滚。而在REQUIRES_NEW情况下，原有事务回滚，不会影响新开启的事务。
### NESTED 与 REQUIRE 的区别？

REQUIRED情况下，调用方存在事务时，则被调用方和调用方使用同一事务，那么被调用方出现异常时，由于共用一个事务，所以无论调用方是否catch其异常，事务都会回滚。

而在NESTED情况下，被调用方发生异常时，调用方可以catch其异常，这样 **`只有子事务回滚，父事务不受影响`** 。
### Spring事务什么时候会失效?

spring事务的原理是AOP，进行了切面增强，那么失效的根本原因是这个AOP不起作用了！常见情况有
如下几种：
1、**`发生自调用`**，类里面使用this调用本类的方法（this通常省略），此时这个this对象不是代理类，而
是UserService对象本身！解决方法很简单，让那个this变成UserService的代理类即可！
2、**`方法不是public的`**。@Transactional 只能用于 public 的方法上，否则事务不会失效，如果要用在非 public 方法上，可以开启 AspectJ 代理模式。
3、**`数据库不支持事务`**
4、**`没有被spring管理`**
5、异常被吃掉，事务不会回滚(或者抛出的异常没有被定义，默认为RuntimeException)
## 四、Spring中用到的设计模式 TODO
- 工厂设计模式 : BeanFactory就是简单工厂模式的体现，根据传入一个唯一标识来获得 Bean 对象。
- 代理设计模式 : Spring AOP 功能的实现。
- 单例设计模式 : Spring 中的 Bean 默认都是单例的。
- 模板方法模式：定义一个操作的算法的框架，而将一些步骤延迟到子类中。使得子类可以不改变一个算法的框架即可重定义该算法的某些特定步骤。Spring 中 jdbcTemplate 、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
## 五、SpringMVC
### 什么是MVC模型？
 MVC(Model View Controller)是一种软件设计的框架模式，它采用 **`模型(Model)-视图(View)-控制器(controller)`** 的方法把业务逻辑、数据与界面显示分离。
- Model(模型)：所有的用户数据、状态以及程序逻辑，独立于视图和控制器。
- View(视图)：呈现模型，类似于Web程序中的界面，视图会从模型中拿到需要展现的状态以及数据，对于相同的数据可以有多种不同的显示形式(视图)
- Controller(控制器)：负责获取用户的输入信息，进行解析并反馈给模型，通常情况下一个视图具有一个控制器。

### SpringMVC工作流程（原理）？
![在这里插入图片描述](https://img-blog.csdnimg.cn/c2e3f40eee324c21a934cef27dd46d78.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_19,color_FFFFFF,t_70,g_se,x_16)

1. 客户端（浏览器）发送请求，直接请求到 DispatcherServlet ；
2. DispatcherServlet 根据请求信息调用 HandlerMapping  。HandlerMapping 根据请求url解析请求对应的Handler ，生成处理器执行链HandlerExecutionChain( **`包括处理器对象和处理器拦截器`** )一并返回给DispatcherServlet；
3. DispatcherServlet根据处理器Handler获取处理器适配器 **`HandlerAdapter`** 来执行HandlerAdapter处理一系列的操作，如：**`参数封装，数据格式转换，数据验证等操作`** ；
4. HandlerAdapter 会根据 Handler 来调用真正的处理器开处理请求，并处理相应的业务逻辑；
5. 处理器处理完业务后，会返回一个 ModelAndView 对象HandlerAdapter将Handler执行结果ModelAndView返回到DispatcherServlet， **`Model 是返回的数据对象， View 是个逻辑上的 View`** ；
6. DispatcherServlet将ModelAndView传给ViewReslover视图解析器，ViewReslover解析后返回具体View；
7. DispatcherServlet对View进行渲染视图（ **`即将模型数据model填充至视图中`** ）；
8. DispatcherServlet把 View 返回给请求者（浏览器）。
### 简单介绍 Spring MVC 的核心组件？
那么接下来就简单介绍一下 **DispatcherServlet 和九大组件**（按使用顺序排序的）：
- **`DispatcherServlet`** ：前端控制器。用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性,系统扩展性提高。由框架实现。
- **`MultipartResolver`** ：内容类型( Content-Type )为 multipart/* 的请求的解析器，例如解析处理文件上传的请求，便于获取参数信息以及上传的文件
- **`HandlerMapping`** ：处理器映射器。HandlerMapping负责 **`根据用户请求的url找到合适HandlerExecutionChain 处理器执行链，包含处理器（ handler ）和拦截器们（ interceptors ）`**
- **`HandlAdapter`** ：处理器适配器。通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。由框架实现。
- **`HandlerExceptionResolver`** ：**`处理器异常解析器`**，**`将处理器（ handler ）执行时发生的异常，解析( 转换 )成对应的 ModelAndView 结果`** 。
- **`ViewResolver`** ：视图解析器。ViewResolver负责将处理结果生成View视图，ViewResolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。
- **`View`** :是springmvc的封装对象，是一个接口, springmvc框架提供了很多的View视图类型，包括：jspview，pdfview,jstlView、freemarkerView、pdfView等。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。
- **`ModelAndView`** ：是springmvc的封装对象，将model和view封装在一起。
- **`Handler`** ：处理器。Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。由于Handler涉及到具体的用户业务请求，所以一般情况需要程序员根据业务需求开发Handler。
### @RestController 和 @Controller 有什么区别？
@RestController 注解，在 @Controller 基础上，增加了 @ResponseBody 注解，更加适合目前前后端分离的架构下，提供 Restful API ，返回例如 JSON 数据格式。当然，返回什么样的数据格式，根据客户端的 ACCEPT 请求头来决定。
### @RequestMapping 和 @GetMapping 注解有什么不同？
- @RequestMapping ：可注解在类和方法上； @GetMapping 仅可注册在方法上
- @RequestMapping ：可进行 GET、POST、PUT、DELETE 等请求方法； @GetMapping 是@RequestMapping 的 GET 请求方法的特例，目的是为了提高清晰度。
### @RequestMapping注解如果有两个相同的url怎么办？（重要）
**`如果存在 @RequestMapping 注解有相同的访问路径时，会在此处抛出异常 （注释很重要）`**
### @RequestParam 和 @PathVariable 两个注解的区别？
两个注解都用于方法参数，获取参数值的方式不同
- @RequestParam 注解：获取请求携带的参数；
	```java
	语法：@RequestParam(value=”参数名”,required=”true/false”,defaultValue=””)
	 
	value：参数名
	 
	required：是否包含该参数，默认为true，表示该请求路径中必须包含该参数，如果不包含就报错。
	 
	defaultValue：默认参数值，如果设置了该值，required=true将失效，自动为false,如果没有传该参数，就使用默认值
	
	/**
	 * 接收普通请求参数
	 * http://localhost:8080/hello/show16?name=linuxsir
	 * url参数中的name必须要和@RequestParam("name")一致
	 * @return
	 */
	@RequestMapping("show16")
	public ModelAndView test16(@RequestParam("name")String name){
	    ModelAndView mv = new ModelAndView();
	    mv.setViewName("hello2");
	    mv.addObject("msg", "接收普通的请求参数：" + name);
	    return mv;
	}
	```

- @PathVariable 注解从请求的 URI （请求路径）中获取。
### @RequestHeader - 获取请求头
### @CookieValue - 获取Cookie
### @RequestBody - 获取请求体【必须用@PostMapping】
### 返回 JSON 格式使用什么注解？
可以使用 **`@ResponseBody注解`** ，或者使用 **`包含 @ResponseBody 注解的 @RestController 注解`** 。
### 什么是springmvc拦截器以及如何使用它？（重要 重要 重要 重要）
Spring的处理程序映射机制包括处理程序拦截器，当你希望将特定功能应用于某些请求时，例如，检查用户主题时，这些拦截器非常有用。拦截器必须实现org.springframework.web.servlet包的 **`HandlerInterceptor接口`** 。

此接口定义了三种方法：

- preHandle：**`在执行实际处理程序之前调用`** 。
- postHandle：**`在执行完实际程序之后调用`** 。
- afterCompletion：**`在完成请求后调用`** 。

![在这里插入图片描述](https://img-blog.csdnimg.cn/92e81110e2bc411282dcfaa32b54b3e7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_20,color_FFFFFF,t_70,g_se,x_16)
配置步骤：

```java
/**
 * 1、编写一个拦截器实现HandlerInterceptor接口
 * 2、拦截器注册到容器中（实现WebMvcConfigurer的addInterceptors）
 * 3、指定拦截规则【如果是拦截所有，静态资源也会被拦截】
 */
@Configuration
public class AdminWebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")  // 所有请求都被拦截包括静态资源
                .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**",
                        "/main",  // 有问题
                        "/js/**","/aa/**"); //放行的请求
    }
}
```

### 说说 @ControllerAdvice 和 @ExceptionHandler？- 异常处理
- @ExceptionHandler：定义一个类的异常处理，**`只能用在方法上`** 。
	- 当在一个方法上注解 **`@ExceptionHandler(异常.class)`**  时，类中所有方法出现括号中的异常时会自动执行被注解的方法,提供友好视图；
	-  一个类中可以在多个方法上加上@ExceptionHandler；
	- 个ExceptionHandler 注解可以设置多个异常的处理到一个方法,但是只对本类中出现的异常进行捕获。
- @ControllerAdvice：**`定义全局的异常处理`**
	- 如果一个异常,不仅仅可能会在一个类出现,而是会在多个类中出现,比如NullPointException, 这个时候如果我们要捕获异常，**`用@ControllerAdvice 和 @ExceptionHandler 组合就捕获全局异常`** 。
```java
@ControllerAdvice
public class GlobalExceptionHandler {

    /**
     * 处理 MissingServletRequestParameterException 异常
     * @param e
     * @return
     */
    @ExceptionHandler({MissingServletRequestParameterException.class})
    public String handleMissingServletRequestParameterException(Exception e){ // 异常会被自动封装
        log.error("异常是:"+e);
        return "login";
    }
}
```
### @ResponseStatus+自定义异常
- @ResponseStatus注解有两种用法:
	- 一种是加载自定义异常类上；
	- 一种是加在目标方法中。

- 注解中有两个参数：
	- value属性设置异常的状态码 HttpCode
	- reaseon是异常的描述
- @ControllerAdvice   + @ExceptionHandle 结合可以捕获全局异常
- 再结合@ResponseStatus 使用可以进一步返回错误视图和httpcode
	```java
	@ControllerAdvice
	public class ExceptionAdvice {
	    @ExceptionHandler(NullPointerException.class)
	    @ResponseStatus(code = HttpStatus.BAD_REQUEST,reason = "失败请重试")
	    public void errro() {
	    }
	}
	```

### 什么是REST？
REST 代表着 **`抽象状态转移`** ，**`它是根据 HTTP 协议从客户端发送数据到服务端`** ，例如：服务端的一本书可以以 XML 或 JSON 格式传递到客户端。
### 什么是安全的 REST 操作?
REST 接口是通过 HTTP 方法完成操作
- 一些 HTTP 操作是安全的，如 GET 和 HEAD ，它不能在服务端修改资源
- 换句话说，PUT、POST 和 DELETE 是不安全的，因为他们能修改服务端的资源

所以，是否安全的界限，在于是否修改服务端的资源。
### REST API 是无状态的吗?
是的，REST API 应该是无状态的，因为它是基于 HTTP 的，它也是无状态的

REST API 中的请求应该包含处理它所需的所有细节。它不应该依赖于以前或下一个请求或服务器端维护的一些数据，例如会话。

REST 规范为使其无状态设置了一个约束，在设计 REST API 时，你应该记住这一点。
## 六、SpringBoot
### 为什么要用SpringBoot?
- 自动配置，使用基于类路径和应用程序上下文的智能默认值，当然也可以根据需要重写它们以满足开发人员的需求。
- 创建Spring Boot Starter 项目时，可以选择选择需要的功能，Spring Boot将为你管理依赖关系。
- SpringBoot项目可以打包成jar文件。可以使用Java-jar命令从命令行将应用程序作为独立的Java应用程序运行。
- 在开发web应用程序时，springboot会配置一个嵌入式Tomcat服务器，以便它可以作为独立的应用程序运行。（Tomcat是默认的，当然你也可以配置Jetty或Undertow）
- SpringBoot包括许多有用的非功能特性（例如安全和健康检查）。
### SpringBoot bootstrap.yml作用
bootstrap.yml（bootstrap.properties）在application.yml（application.properties）之前加载。

它通常用于“使用Spring Cloud Config Server时，应在bootstrap.yml中指定spring.application.name和spring.cloud.config.server.git.uri”以及一些加密/解密信息。

### Spring Boot中如何实现对不同环境的属性配置文件的支持？
Spring Boot支持不同环境的属性配置文件切换，通过创建application-{profile}.properties文件，其中{profile}是具体的环境标识名称：
- application-dev.properties用于开发环境
- application-test.properties用于测试环境
- application-uat.properties用于uat环境

如果要想使用application-dev.properties文件，则在application.properties文件中添加 **`spring.profiles.active=dev`** 。

如果要想使用application-test.properties文件，则在application.properties文件中添加 **`spring.profiles.active=test`** 。
### Spring Boot 的核心注解是哪个？它主要由哪几个注解组成的？
启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下3 个注解：
- **`@ComponentScan`**
	- @ComponentScan注解就是用来自动扫描被这些注解标识的类，最终生成ioc容器里的bean。
	- 可以通过设置@ComponentScan的 **`basePackages，includeFilters，excludeFilters`** 属性来动态确定自动扫描范围，类型已经不扫描的类型。
	- 默认情况下:它扫描所有类型，并且扫描范围是 **`@ComponentScan注解所在配置类包及子包的类`** 。
- **`@SpringBootConfiguration`** ：这个注解的作用与@Configuration作用相同，都是用来声明当前类是一个配置类。可以通过＠Bean注解生成IOC容器管理的bean。它实际上继承自 @Configurationg。
- **`@EnableAutoConfiguration`** ：它是springboot实现自动化配置的核心注解，通过这个注解把spring应用所需的bean注入容器中。
	- 可以打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： **`@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。`**

### Spring Boot Starter 的工作原理是什么？
Spring Boot 在启动的时候会干这几件事情：
1. Spring Boot 在启动时会去依赖的 Starter 包中寻找 **`resources/META-INF/spring.factories`** 文件，然后根据文件中配置的 Jar 包去扫描项目所依赖的 Jar 包。
2. 根据 spring.factories 配置加载 AutoConfigure 类
3. 根据 @Conditional 注解的条件，进行自动配置并将 Bean 注入 Spring Context

总结一下，其实就是 **`Spring Boot 在启动的时候，按照约定去读取 Spring Boot Starter 的配置信息，再根据配置信息对资源进行初始化，并注入到 Spring 容器中`**。这样 Spring Boot 启动完毕后，就已经准备好了一切资源，使用过程中直接注入对应 Bean 资源即可

## 七、常用注解总结
### @Configuration - 声明配置类

```csharp
/**
 * @Configuration
 *    1.告诉SpringBoot这是一个配置类 == 配置文件
 *    2.配置类里可以使用 @Bean 给容器注册组件，默认也是单实例的
 *    3.配置类本身也是一个组件
 *    4.proxyBeanMethods:代理 Bean 的方法，默认为 true
 *        Full(全模式)==》true:外部无论对配置类中的这个组件注册方法调用多少次，获
 *                           取到的都是之前注册到容器中的单实例
 *        Lite(轻量级模式)==》false:在容器中不会保持代理对象，每次调用都会产生新对象，
 *                                SpringBoot不会检查容器中是否有代理对象，会直接创建
 *                                运行速度更快
 *        应用场景：有组件依赖 ==》 true
 *                无组件依赖 ==》 false
 */
@Configuration(proxyBeanMethods=true)
public class MyConfig {}
```
### @Import - 导入组件

```csharp
 * 4、@Import({User.class, DBHelper.class})
 *      给容器中自动创建出这两个类型的组件、默认组件的名字就是 《全类名》
 */
@Import({User.class, DBHelper.class})
@Configuration(proxyBeanMethods = false) //告诉SpringBoot这是一个配置类 == 配置文件
public class MyConfig {}
```
### @Conditional - 条件装配
条件装配：满足Conditional指定的条件，则进行组件注入

```csharp
@ConditionalOnBean(name = "tom")
@Bean
public User user01(){
    return new User("zhangsan",18);
}
```
## 八、SpringCloud
### 什么是SpringCloud？
Spring Cloud 是为了解决微服务架构中服务治理而提供的 **`一系列功能`** 的开发框架，并且 Spring Cloud是完全基于 Spring Boot 而开发，Spring Cloud 利用 Spring Boot 特性整合了开源行业中优秀的组件，整体对外提供了一套在微服务架构中服务治理的解决方案。
### 了解rpc吗？了解服务与服务之间怎么通信的吗？
## 九、MyBatis
### MyBatis中 $ 和 # 的区别？
动态 sql 是 mybatis 的主要特性之一，在 mapper 中定义的参数传到 xml 中之后，在查询之前 mybatis 会对其进行动态解析。mybatis 为我们提供了两种支持动态 sql 的语法：#{} 以及 ${}。

在下面的语句中，如果 username 的值为 zhangsan，则两种方式无任何区别：
```sql
select * from user where name = #{name};
select * from user where name = ${name};
```
其解析之后的结果均为
```sql
select * from user where name = 'zhangsan';
```
但是 #{} 和 ${} 在预编译中的处理是不一样的。**`#{} 在预处理时，会把参数部分用一个占位符 ? 代替，变成如下的 sql 语句`** ：

```sql
select * from user where name = ?;
```
而 **`${} 则只是简单的字符串替换`**，在动态解析阶段，该 sql 语句会被解析成

```sql
select * from user where name = 'zhangsan';
```
　以上，**`#{} 的参数替换是发生在 DBMS 中，而 ${} 则发生在动态解析过程中`**。

那么，在使用过程中我们应该使用哪种方式呢？答案是，**`优先使用 #{}。因为 ${} 会导致 sql 注入的问题`**。看下面的例子：

```sql
select * from ${tableName} where name = #{name}
```
在这个例子中，如果表名为 **`user; delete user; --`** ，则动态解析之后 sql 如下：

```sql
select * from user; delete user; -- where name = ?;
```
**`--之后的语句被注释掉，而原本查询用户的语句变成了查询所有用户信息+删除用户表的语句`**，会对数据库造成重大损伤，极大可能导致服务器宕机。

但是表名用参数传递进来的时候，只能使用 ${} ，具体原因可以自己做个猜测，去验证。这也提醒我们在这种用法中要小心sql注入的问题。


