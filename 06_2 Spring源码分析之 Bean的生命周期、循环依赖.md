## 1 Spring Bean的生命周期
### 1.1 前置知识
> 什么是Spring Bean，它和Java 对象的区别

spring bean是受spring容器管理的对象，可能经过了完整的spring bean生命周期（为什么是可能？难道还有bean是没有经过bean生命周期的？答案是有的，具体我们后面文章分析），最终存在spring容器当中；一个bean一定是个对象。

任何符合java语法规则实例化出来的对象，但是一个对象并不一定是spring bean。

Spring会根据配置文件或者配置类来生成描述Bean的BeanDefinition，常用属性：
- @Scope：作用域
	- singleton
	- prototype
	- request
	- session
	- globalsession
- @lazy：懒加载，决定Bean是否延迟加载
- @Primiary：设置为true的Bean会是优先实现的类
- factory-bean和factory-method（@Configuration和@Bean）



### 1.2 Spring Bean生命周期简述
![在这里插入图片描述](https://img-blog.csdnimg.cn/75d2df208f334db186edaedb84b31411.png)

> 文字描述

1. 实例化一个ApplicationContext的对象；
2. 调用bean工厂后置处理器完成扫描；
3. 循环解析扫描出来的类信息；
4. 实例化一个BeanDefinition对象来存储解析出来的信息；
5. 把实例化好的beanDefinition对象put到beanDefinitionMap当中缓存起来，以便后面实例化bean；
6. 再次调用bean工厂后置处理器；
7. 当然spring还会干很多事情，比如国际化，比如注册BeanPostProcessor等等，如果我们只关心如何实例化一个bean的话那么这一步就是spring调用finishBeanFactoryInitialization方法来实例化单例的bean，实例化之前spring要做验证，需要遍历所有扫描出来的类，依次判断这个bean是否Lazy，是否prototype，是否abstract等等；
8. 如果验证完成，spring在实例化一个bean之前需要推断构造方法，因为spring实例化对象是通过构造方法反射，故而需要知道用哪个构造方法；
9. 推断完构造方法之后spring调用构造方法反射实例化一个对象；注意我这里说的是**对象、对象、对象**；这个时候对象已经实例化出来了，但是并不是一个完整的bean，最简单的体现是这个时候实例化出来的**对象属性是没有注入的**，所以不是一个完整的bean；
10. spring处理合并后的beanDefinition(合并？是spring当中非常重要的一块内容，后面的文章我会分析)；
11. 判断是否支持循环依赖，如果支持则提前把一个工厂存入singletonFactories——map；
12. 判断是否需要完成属性注入
13. 如果需要完成属性注入，则开始注入属性
14. 判断bean的类型回调Aware接口
15. 调用生命周期回调方法
16. 如果需要代理则完成代理
17. put到单例池 ==> bean完成 ==>存在spring容器当中


> <font size=4 color=blue>**现在我们假设TestService1 和 TestService2 相互依赖，这个例子会用于下面的循环依赖讲解**</font>

```java
@Service
public class TestService1 {

    @Autowired
    private TestService2 testService2;

    @Async
    public void test1() {
    }
}

@Service
public class TestService2 {

    @Autowired
    private TestService1 testService1;

    public void test2() {
    }
}
```
### 1.3 源码分析 之 doGetBean
Spring容器创建的逻辑在AbstractApplicationContext的refresh()方法中，该方法里面调用了一个非常重要的方法 - AbstractBeanFactory的 doGetBean() 的方法。这个方法非常重要，不仅仅针对循环依赖，甚至整个spring bean生命周期中这个方法也有着举足轻重的地位。
- 简化版doGetBean()方法如下：

	```java
	protected <T> T doGetBean(final String name, 
						@Nullable final Class<T> requiredType,
	      				@Nullable final Object[] args, 
	      				boolean typeCheckOnly)
	      				throws BeansException {
	    //读者可以简单的认为就是对beanName做一个校验特殊字符串的功能
	    //我会在下次更新博客的时候重点讨论这个方法
	    //transformedBeanName(name)这里的name就是bean的名字
	   final String beanName = transformedBeanName(name);
	   
	   //定义了一个对象，用来存将来返回出来的bean
	   Object bean;
	
		//deGetBean-1
	   Object sharedInstance = getSingleton(beanName);
	   
		//deGetBean-2
		if (sharedInstance != null && args == null) {
	      bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
	   }else{
	   		// deGetBean-3
	   		if (isPrototypeCurrentlyInCreation(beanName)) {
	         	throw new BeanCurrentlyInCreationException(beanName);
	      }else{
	      	//doGetBean-4
	      	if (mbd.isSingleton()) {
	            sharedInstance = getSingleton(beanName, () -> {
	               try {
	                  return createBean(beanName, mbd, args);
	               }
	               catch (BeansException ex) {
	                  destroySingleton(beanName);
	                  throw ex;
	               }
	            });
	            
	      }
	   }
	 }
	
	```



#### 1.3.1 doGetBean - 1 之 getSingleton(beanName)
总结一下 <font color=red>**Object sharedInstance = getSingleton(beanName);**</font>目前看来主要是用于在spring初始化bean的时候判断bean是否在容器当中；以及供程序员直接get某个bean。

注意这里只是从目前来看；因为getSingleton(beanName);这个方法代码比较多； <font color=red>**它里面的逻辑是实现循环依赖最主要的代码**</font>， 文章下面我会回过头再来讲这个方法的全部意义；

根据上面的分析，这个时候我的TestService1  Bean肯定没有被创建，所以这里返回**sharedInstance == null**。

#### 1.3.2 doGetBean - 2
由于sharedInstance == null，因此不进入该分支
```java
//deGetBean-2
if (sharedInstance != null && args == null) {
  bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
}
```
> <font size=4 color=blue>**由于 sharedInstance == null 故而不会进入这个if分支，那么什么时候不等于null呢？**</font>

有两种情况：
- 情况一：在spring初始化完成后程序员调用getBean(“testService1 ”)的时候得到的sharedInstance 就不等于null
- 情况二：循环依赖的时候第二次获取对象的时候这里也不等于空。比如 X 依赖 Y，Y依赖X这种情况：
	- Spring做初始化第一次执行到这里的时候 X 肯定等于null，然后接着往下执行；
	- 当执行到属性注入Y的时候，Y也会执行到这里，那么Y也是null，因为Y也没初始化，Y也会接着往下执行；
	- 当Y执行到属性注入的时候获取容器中获取X，也就是第二次执行获取X，这个时候X则不为空，这就是循环依赖，我们接着往下分析。

#### 1.3.3 doGetBean - 3

```java
else{
	// deGetBean-3
	if (isPrototypeCurrentlyInCreation(beanName)) {
	   	throw new BeanCurrentlyInCreationException(beanName);
	}
}
```
不进if分支，则进入这个else分支 - 判断当前初始化的TestService1  Bean 是不是正在创建原型Bean集合当中？

一般情况下这里返回false，也就是不会进入if分支抛异常；为什么呢说一般情况下呢？

- 首先这里是判断当前的类是不是正在创建的原型集合当中，即里面只会存原型；一般情况下我们的类不是原型，而是单例的，大家都知道**spring默认是单例**；所以返回false；
- 再就是即使这个bean是原型也很少会在这里就存在**正在创建的原型集合**当中。因为不管单例还是原型，bean在创建的过程中会add到这个集合当中（**在后面完成**），但是创建完成之后就会从这个集合remove掉，故而下一次实例化同一个原型bean（原型可以实例化无数次）的时候当代码执行到这里也不可能存在集合当中了；
- 除非循环依赖会在bean还没有在这个集合remove之前再次判断一次，才有可能会存在，故而我前面说了一般情况下这里都返回false；那么单例情况我们已经说了一定返回false，原型情况只有循环依赖才会成立，但是只要是正常人就不会对原型对象做循环依赖的；即使你用原型做了循环依赖这里也出抛异常（因为if成立，进入分支 throw exception）。<font color=red>**再一次说明原型不支持循环依赖。**</font>
#### 1.3.4 doGetBean - 4 之 - getSingleton()重载
```java
else{
	//doGetBean-4
	if (mbd.isSingleton()) {
      sharedInstance = getSingleton(beanName, () -> {
         try {
            return createBean(beanName, mbd, args);
         }
         catch (BeansException ex) {
            destroySingleton(beanName);
            throw ex;
         }
      });
      
}
```

<font color=red>**if (mbd.isSingleton())**</font> 比较简单，判断当前bean是否单例；本文环境下是成立的。

这里又调用了一次getSingleton，在doGetBean-1中也调用了一次getSingleton，这是方法 <font size=4 color=blue>**重载**</font> 。<font color=red>**本次调用getSingleton()方法就会把我们bean创建出来**</font>，换言之整个bean如何被初始化的都是在这个方法里面；至此本文当中笔者例举出来的doGetBean方法的核心代码看起来解析完成了；

### 1.4 源码分析 之 doGetBean4 - getSingleton()
上文doGetBean()方法第二次调用的getSingleton()方法是<font color=red>**DefaultSingletonBeanRegistry**</font>类中的 **getSingleton()** 方法。

删减后的代码如下

```java
public Object getSingleton(String beanName, ObjectFactory<?> 
singletonFactory) {
	//getSingleton2 -1
	Object singletonObject = this.singletonObjects.get(beanName);
			//getSingleton2 -2
			if (singletonObject == null) {
				//getSingleton2 -3
				if (this.singletonsCurrentlyInDestruction) {
					throw new Exception(beanName,
							"excepition");
				}
				//getSingleton2 -4
				beforeSingletonCreation(beanName);
				//getSingleton2 -5
				singletonObject = singletonFactory.getObject();	
			}
			return singletonObject;
		}
}		
```
#### 1.4.1 getSingleton() - 1

```java
Object singletonObject = this.singletonObjects.get(beanName);
```
第二次getSingleton上来便调用了this.singletonObjects.get(beanName)，直接从单例池当中获取这个对象，由于这里是创建故而**一定返回null**；

singletonObjects是一个map集合，即所谓的单例池；用大白话说spring所有的单例bean实例化好都存放在这个map当中，这也是很多读者以前认为的spring容器，但是笔者想说这种理解是错误的，因为spring容器的概念比较抽象，而单例池只是spring容器的一个组件而已；但是你如果一定要找一个平衡的说法，只能说这个map——singletonObjects仅仅是狭义上的容器；比如你的原型bean便不在这个map当中，所以是狭义的spring容器；下图为这个map在spring源码当中的定义：

```java
//一级缓存：单例对象缓存池，beanName->Bean，其中存储的就是实例化，属性赋值成功之后的单例对象
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);、
```

#### 1.4.2 getSingleton() - 2
上面解释了，在spring 初始化bean的时候这里肯定为空，故而成立，会进入这个if
```java
if (singletonObject == null) {
```
#### 1.4.3 getSingleton() - 3

```java
if (this.singletonsCurrentlyInDestruction) {
	throw new Exception(beanName,"excepition");
}

```
判断当前实例化的bean是否正在销毁的集合里面；spring不管销毁还是创建一个bean的过程都比较繁琐，都会先把他们放到一个集合当中标识正在创建或者销毁。

如果一个bean正在创建，但是有正在销毁那么则会出异常；为什么会有这种情况？其实也很简单，多线程可能会吧。

#### 1.4.4 getSingleton() - 4 - 重要
```java
beforeSingletonCreation(beanName);
```
当经过各种判断spring觉得可以着手来创建bean的时候首先便是调用 <font color=red>**beforeSingletonCreation(beanName);**</font> <font color=orange>**判断当前正在实例化的bean是否在正在创建的集合当中，说白了就是判断当前是否正在被创建**</font>；

因为spring不管创建原型bean还是单例bean，当他需要正式创建bean的时候他会记录一下这个bean正在创建(add到一个set集合当中)；故而当他正式创建之前他要去看看这个bean有没有正在被创建（是否存在集合当中）; 

为什么spring要去判断是否存在这个集合呢？**其实这个集合主要是为了循环依赖服务的**，我们来详细看它是怎么运作的。

> 源码分析

```java
protected void beforeSingletonCreation(String beanName) {
	if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
		throw new BeanCurrentlyInCreationException(beanName);
	}
}
```
- <font color=red>**this.inCreationCheckExclusions.contains(beanName);**</font> 这里是判断当前需要创建的bean是否在Exclusions集合，被排除的bean，程序员可以提供一些bean不被spring初始化（哪怕被扫描到了，也不初始化），那么这些提供的bean便会存在这个集合当中；一般情况下我们不会提供，而且与循环依赖无关，这里不做深入分析；

- <font color=red>**this.singletonsCurrentlyInCreation.add(beanName)**</font>，如果当前bean不在排除的集合当中那么则这个bean添加到singletonsCurrentlyInCreation（这里只是把bean名字添加到集合），这里就是把 **testService1** 加入到这个集合中。

关于singletonsCurrentlyInCreation的定义参考下图：
```java
// 三级缓存是用来解决循环依赖，而这个缓存就是用来检测是否存在循环依赖的
private final Set<String> singletonsCurrentlyInCreation =
		Collections.newSetFromMap(new ConcurrentHashMap<>(16));
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/807741ddb7e64ee09d3f83391a35fd74.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_13,color_FFFFFF,t_70,g_se,x_16)
#### 1.4.5 getSingleton() - 5

```java
singletonObject = singletonFactory.getObject();	
```
这个方法就是doGetBean-4中的lambda表达式

```java
ObjectFactory<?>  singletonFactory = new ObjectFactory(){
	public Object getObject(){
		//其实这是个抽象类，不能实例化
		//createBean是子类实现的，这里就不关心了
		//你就理解这不是一个抽象类吧
		AbstractBeanFactory abf = new AbstractBeanFactory();
		Object bean = abf.createBean(beanName, mbd, args);
		return bean;
	};
};
//传入 beanName 和singletonFactory 对象
sharedInstance = getSingleton(beanName,singletonFactory);

这样看是不是明白多了呢？
sharedInstance = getSingleton(beanName, () -> {
	try {
	   return createBean(beanName, mbd, args);
	}
	catch (BeansException ex) {
	   destroySingleton(beanName);
	   throw ex;
	}
}
```
<font color=red>**singletonFactory.getObject();**</font>中调用的其实是<font color=red>**AbstractAutowireCapableBeanFactory.createBean(beanName, mbd, args)；**</font>，该方法最终把创建好Bean并返回出来，至此第二次getSingleton方法结束。

<font size=4 color=green>**接下来分析createBean的源码，引出循环依赖循并探讨其原理**</font>
## 2 Spring Bean创建过程中的循环依赖
在上文getSingleton()-5中调用的是<font color=red>**AbstractAutowireCapableBeanFactory.createBean()**</font>方法，该方法中中调用了doCreateBean() 方法来创建Bean；

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
		throws BeanCreationException {
		
	BeanWrapper instanceWrapper = null;
	if (mbd.isSingleton()) {
		instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
	}
	if (instanceWrapper == null) {
		// doCreateBean()-1
		instanceWrapper = createBeanInstance(beanName, mbd, args);
	}

	// doCreateBean()-2
	//这里判断的逻辑主要有三个：
	//1.是否为单例
	//2.是否允许循环引用
	//3.是否是在创建中的bean
	boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
			isSingletonCurrentlyInCreation(beanName));
	if (earlySingletonExposure) {
		// doCreateBean()-3
		//这里是一个匿名内部类，为了防止循环引用，尽早持有对象的引用
		addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
	}
	
	Object exposedObject = bean;
	try {
		// doCreateBean()-4 属性填充
		populateBean(beanName, mbd, instanceWrapper);

		// 初始化bean
		exposedObject = initializeBean(beanName, exposedObject, mbd);
	} 
	return exposedObject;
}
```
### 2.1 源码分析之doCreateBean()
#### 2.1.1 doCreateBean() - 1 - 创建Bean实例（Bean生命周期开始）

```java
// doCreateBean()-1
// 创建bean的时候，这里创建bean的实例有三种方法
// 1.工厂方法创建
// 2.构造方法的方式注入
// 3.无参构造方法注入
instanceWrapper = createBeanInstance(beanName, mbd, args);
```

同类方法AbstractAutowireCapableBeanFactory的createBeanInstance()方法 顾名思义就是创建一个实例，注意这里仅仅是创建一个实例对象，还不能称为bean；<font color=red>**此时bean的生命周期才刚刚开始**</font>。

<font color=green>**createBeanInstance()方法会做两件事**</font>：<font color=red>**推断构造方法**</font> 和 <font color=red>**利用构造方法反射来实例化对象**</font>。

至此 TestService1 对象已经实例化出来，代码往下执行到合并BeanDefinition，但是其实合并BeanDefinition和本文讨论的循环依赖无关，先不做讨论；

#### 2.1.2 doCreateBean() - 2 - 判断是否支持循环依赖

```java
//向容器中缓存单例模式的Bean对象，以防循环引用
//判断是否是早期引用的bean，如果是，则允许其提前暴露引用
//这里判断的逻辑主要有三个：
//1.是否为单例
//2.是否允许循环引用
//3.是否是在创建中的bean
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
		isSingletonCurrentlyInCreation(beanName));
```
这段代码其实比较简单，就是给earlySingletonExposure这个布尔类型的变量赋值；这个变量的意义是 -  <font color=red>**是否支持（开启了）循环依赖**</font>：
- 如果返回true则spring会做一些特殊的操作来完成循环依赖；
- 如果返回false，则不会有特殊操作。

那么这个布尔变量的赋值逻辑是怎样的呢？上面代码可知三个条件做&&运算，同时成立才会返回true：
- <font color=red>**bd.isSingleton()**</font>：判断当前实例化的bean是否为单例；再一次说明原型是不支持循环依赖的；因为如果是原型这里就会返回false，由于是&&运算，整个结果都为false；
- <font color=red>**this.allowCircularReferences**</font>：表示是否支持循环引用，整个全局变量spring 默认为true；当然spring提供了api供程序员修改，在没有修改的情况下这里也返回true；
- <font color=red>**isSingletonCurrentlyInCreation(beanName)**</font>：判断当前正在创建的bean是否在正在创建bean的集合当中；
	- 在1.4.4小节中分析过了，singletonsCurrentlyInCreation这个集合现在里面存在且只有一个x；故而也会返回true；


其实这三种情况**需要关心的只有第二种**；因为第一种是否单例一般都是成立的，因为如果是原型的循环依赖前面代码已经报错了；压根不会执行到这里；第三种情况也一般是成立，因为这个集合是spring操作的，没有提供api给程序员去操作；而正常流程下代码执行到这里，当前正在创建的bean是一定在那个集合里面的；换句话说这三个条件1和3基本恒成立；<font color=red>**唯有第二种情况可能会不成立，因为程序员可以通过api来修改第二个条件的结果**</font>。

<font color=red>**总结：spring的循环依赖，不支持原型，不支持构造方法注入的bean；默认情况下单例bean是支持循环依赖的，但是也支持关闭，关闭的原理就是设置allowCircularReferences=false；spring提供了api来设置这个值**</font>。

至此我们知道boolean earlySingletonExposure=true，那么代码接着往下执行判断这个变量，if成立，进入分支。

```java
if (earlySingletonExposure) {
}
```
#### 2.1.3 doCreateBean() - 3 - 加入三级缓存

```java
if (earlySingletonExposure) {
	// doCreateBean()-3
	addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
```
这里是往<font color=red>**DefaultSingletonBeanRegistry.addSingletonFactory(beanName,singletonFactory);**</font>方法中传入了一个lambda表达式；

<font color=red>**addSingletonFactory(beanName,singletonFactory);**</font>顾名思义 <font color=orange>**添加一个单例工厂**</font> 。注意：<font color=orange>**这里是add一个工厂对象 singletonFactory，而不是add一个bean**</font>。

那么这个Bean工厂对象到底add到哪里去了，查看源码

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
	Assert.notNull(singletonFactory, "Singleton factory must not be null");
	synchronized (this.singletonObjects) {
		if (!this.singletonObjects.containsKey(beanName)) {
			// 往三级缓存里添加
			this.singletonFactories.put(beanName, singletonFactory);
			// 清除此Bean在二级缓存里的缓存信息
			this.earlySingletonObjects.remove(beanName);
			// 这里为了记录注册单例的顺序
			this.registeredSingletons.add(beanName);
		}
	}
}
```
> <font size=4 color=blue>**Spring处理循环依赖时涉及到的三个map**</font>

- **singletonObjects**：一级缓存，用于保存实例化、注入、初始化完成的bean实例；
- **earlySingletonObjects**：二级缓存，用于保存实例化完成的bean实例；
- **singletonFactories**：三级缓存，用于保存bean创建工厂，以便于后面扩展有机会创建代理对象。


通过代码可以得知singletonFactory主要被add到三级缓存中，至于为什么要add到这个map？就是为例解决循环依赖，提前暴露这个工厂。
> <font size=4 color=blue>**此时Spring中处理循环依赖的三个map中的情况如下：**</font>

- 一级缓存：可能存在很多bean，比如spring各种内置bean，比如你项目里面其他的已经创建好的bean，但是在TestService1的创建过程中，一级缓存中绝对是没有TestService1 Bean的，也没用 TestService2；
- 二级缓存：姑且认为里面什么都没有吧;
- 三级缓存：里面现在仅仅存在一个工厂对象，对应的key为TestService1的beanName，并且这个Bean工厂对象的getObect()方法能返回现在的这个时候的TestService1（半成品的TestService1 bean）

put完成之后，代码接着往下执行。
#### 2.1.4 doCreateBean()-4 - 属性填充（循环引用正式开始）
```java
// doCreateBean()-4
populateBean(beanName, mbd, instanceWrapper);
```
该方法主要就是完成属性注入，也就是常说的自动注入；由于TestService1 有一个属性依赖TestService2，那么在运行完这行代码那么则会注入TestService2 ，而TestService2 又引用了TestService1 ，但是在没有执行populateBean()之前只实例化了TestService1 ，TestService2 并没实例化，那么TestService2 也不能注入了；接下来看看执行完这行代码之后的情况
> <font size=4 color=blue>**populateBean() 到底干了什么事？**</font>

1. TestService1 填充 TestService2  首先肯定需要获取 TestService2，在populateBean()中会调用getBean("testService2")方法来获取TestService2，而getBean()的本质上是<font color=red>**调用1.3.1小节中的用getSingleton()方法**</font>；

2. 第一次getSingleton()会从单例池获取一下TestService2  ，如果TestService2  没有存在单例池则开始创建y；创建y的流程和创建TestService1  一模一样，都会走Bean的生命周期；比如把TestService2  添加到正在创建的Bean的集合当中，推断构造方法，实例化TestService2  ，提前暴露工厂对象（二级缓存里面现在有两个工厂了，分别是TestService1  和TestService2  ）等等。。。。重复TestService1 的步骤；

3、直到TestService2 的生命周期走到填充TestService1的时候，会<font color=red>**再次调用1.3.1小节中的getSingletion()方法获取TestService1**</font>。<font size=4 color=blue>**那么这次能否获取到TestService1呢？**</font>

在回答这个问题之前我们先把该画的图贴出来，首先那个正在被创建bean的集合已经不在是只有一个TestService1了，TestService2也在其中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/26c25a02bab5439fbef7fd072aa9dd2c.png)
<font size=4 color=green>**这里我们回过头再来分析一下1.3.1小节中的getSingletion()方法。**</font>


### 2.2 源码分析 之 getSingleton()（解决循环引用）
下面来开始对第一次getSIngleton源码做深入分析

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
	//从单例池当（一级缓存）中直接拿，也就是文章里面'目前'的解释
	//这也是为什么getBean("xx")能获取一个初始化好bean的根本代码
	Object singletonObject = this.singletonObjects.get(beanName);
	//如果这个时候是x注入y，创建y，y注入x，获取x的时候那么x不在容器
	//第一个singletonObject == null成立
	//第二个条件判断是否存在正在创建bean的集合当中，前面我们分析过，成立
	//进入if分支
	if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
		synchronized (this.singletonObjects) {
			//先从三级缓存那x？为什么先从三级缓存拿？下文解释
			singletonObject = this.earlySingletonObjects.get(beanName);
			//讲道理是拿不到的，因为这三个map现在只有二级缓存中存了一个工厂对象
			//回顾一下文章上面的流程讲工厂对象那里，把他存到了二级缓存
			//所以三级缓存拿到的singletonObject==null  第一个条件成立
			//第二个条件allowEarlyReference=true，这个前文有解释
			//就是spring循环依赖的开关，默认为true 进入if分支
			if (singletonObject == null && allowEarlyReference) {
				//从二级缓存中获取一个 singletonFactory，回顾前文，能获取到
				//由于这里的beanName=x，故而获取出来的工厂对象，能产生一个x半成品bean
				ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
				//由于获取到了，进入if分支
				if (singletonFactory != null) {
					//调用工厂对象的getObject()方法，产生一个x的半成品bean
					//怎么产生的？下文解释，比较复杂
					singletonObject = singletonFactory.getObject();
					//拿到了半成品的xbean之后，把他放到三级缓存；为什么？下文解释
					this.earlySingletonObjects.put(beanName, singletonObject);
					//然后从二级缓存清除掉x的工厂对象；？为什么，下文解释
					this.singletonFactories.remove(beanName);
				}
			}
		}
	}
	return singletonObject;
}
```
步骤说明：

1、首先spring从单例池当中获取TestService1 ，前面说过获取不到，然后判断是否在正在创建Bean的集合当中；

2、前面分析过这个集合现在存在TestService1 和 TestService2 ，所以 if 成立进入分支；进入分支Spring直接从二级缓存中获取TestService1 ，根据前面的分析二级缓存当中现在什么都没有，故而返回null；

3、进入下一个if分支，从三级缓存中获取一个ObjectFactory工厂对象；根据前面分析，三级缓存中存在TestService1 ，故而可以获取到；

4、跟着调用singletonFactory.getObject();拿到一个半成品的TestService1 Bean对象；然后把这个TestService1 对象放到二级缓存，同时把三级缓存中TestService1  ObjectFactory清除（**此时三级缓存中只存在一个TestService2  ObjectFactory了，而二级缓存中多了一个TestService1**）；

### 2.3 第三级缓存中为什么要添加ObjectFactory对象，直接保存实例对象不行吗？
不行，因为假如你想对添加到三级缓存中的实例对象进行增强，直接用实例对象是行不通的。

在AbstractAutowireCapableBeanFactory类doCreateBean方法的这段代码中：
![在这里插入图片描述](https://img-blog.csdnimg.cn/92b5e94d9a794118bed9d5920052e105.jpeg)

它定义了一个匿名内部类，通过getEarlyBeanReference方法获取代理对象，其实底层是通过AbstractAutoProxyCreator类的getEarlyBeanReference生成代理对象。
### 2.4 为什么要用第二级缓存呢？
关于为什么要用二级缓存，以及为什么在用Bean工厂创建完Bean后要把创建的Bean放入二级缓存，并删除三级缓存中的Bean工厂。
```java
@Service
public class TestService1 {

    @Autowired
    private TestService2 testService2;
    @Autowired
    private TestService3 testService3;

    public void test1() {
    }
}

@Service
public class TestService2 {

    @Autowired
    private TestService1 testService1;

    public void test2() {
    }
}

@Service
public class TestService3 {

    @Autowired
    private TestService1 testService1;

    public void test3() {
    }
}
```
TestService1依赖于TestService2和TestService3，而TestService2依赖于TestService1，同时TestService3也依赖于TestService1。按照上图的流程可以把TestService1注入到TestService2，并且TestService1的实例是从第三级缓存中获取的。

假设没有第二级缓存，TestService1注入到TestService3又需要从第三级缓存中获取实例，而第三级缓存里保存的并非真正的实例对象，而是ObjectFactory对象。说白了，两次从三级缓存中获取都是ObjectFactory对象，而通过它创建的实例对象每次可能都不一样的。

为了解决这个问题，spring引入的第二级缓存。上面图1其实TestService1对象的实例已经被添加到第二级缓存中了，而在TestService1注入到TestService3时，只用从第二级缓存中获取该对象即可。


### 2.5 循环依赖的场景
参考：[https://www.zhihu.com/question/445446018/answer/2355907930?utm_source=wechat_session&utm_medium=social&utm_oi=1108279169968074752&utm_content=group3_Answer&utm_campaign=shareopn](https://www.zhihu.com/question/445446018/answer/2355907930?utm_source=wechat_session&utm_medium=social&utm_oi=1108279169968074752&utm_content=group3_Answer&utm_campaign=shareopn)
![在这里插入图片描述](https://img-blog.csdnimg.cn/cf4b5bec9d204e8a917bb1b7cdfc89cb.jpeg)
### 2.6 循环依赖流程总结
![在这里插入图片描述](https://img-blog.csdnimg.cn/1949b27b8d534241970781f41a6e2bb7.jpeg)


Spring容器创建的逻辑在 **AbstractApplicationContext** 的 **refresh()** 方法中，该方法里面调用了AbstractBeanFactory的 **`doGetBean()`** 的方法。**这个方法非常重要，不仅仅针对循环依赖，甚至整个spring bean生命周期中这个方法也有着举足轻重的地位。**

总结一下三级缓存解决流程，假设x、y相互依赖：

```java
================= x 创建流程 ======================
AbstractBeanFactory.doGetBean(): 
1.首先调用getSingleton(beanName);尝试从一、二级缓存中获取Bean实例，
  x还未创建，因此获取不到位null；
2.判断当前的类是否在正在创建的原型集合当中，如果在直接抛异常
  BeanCurrentlyInCreationException(beanName);
3.调用getSingleton(beanName,singletonFactory);方法，这个方法是AbstractBeanFactory的父类 
  DefaultSingletonBeanRegistry中的方法，传入的singleFactory是AbstractBeanFactory。
  1、首先从this.singletonObjects.get(beanName)，尝试直接从单例池当中获取这个对象，由于x正在创建，
     所以返回null。
  2、判断当前实例化的bean是否正在销毁的集合里面（多线程时会出现这种情况）
  3、调用beforeSingletonCreation(beanName);方法判断
  	 1)判断当前需要创建的bean是否在Exclusions集合；
  	 2)判断当前正在实例化的bean是否存在正在创建的集合当中（为循环依赖服务）======================
  4、调用singletonFactory.getObject();调用的是传入的AbstractBeanFactory中的getObject()方法，
     而getObject()方法中调用的是AbstractBeanFactory的子类AbstractAutowireCapableBeanFactory.createBean()方法
     1)调用createBeanInstance(beanName, mbd, args);方法对Bean进行实例化
       【1】推断构造方法
       【2】利用构造方法反射来实例化对象
     2)判断是否支持循环依赖，判断依据有三个
       【1】是否为单例
       【2】是否允许循环引用（可以手动设置为false）、
       【3】是否是在创建中的bean
     3)调用addSingletonFactory(beanName,singletonFactory);方法向二级缓存中添加一个bean工厂
     ============ 循环依赖开始 ==============
     ============ 循环依赖开始 ==============
     4)调用populateBean(beanName, mbd, instanceWrapper);方法对x进行属性填充
       【1】x 填充 y 首先肯定需要获取y，调用getBean(y)，而getBean()方法其实调用的就是doGetBean()方法
       【2】此时进入doGetBean()的第一次调用的getSingleton()方法，获取不到y，则进入y的创建流程
==================== x需要填充属性需要y，开始y的创建 ======================
1.y的创建流程和x一样，知道填充属性时，需要获取x，调用getBean(x)，而getBean()方法其实调用的就是doGetBean()方法
2.首先调用getSingleton(beanName);尝试获取 x Bean
  1)首先尝试从单例池（一级缓存）当中获取x，由于x还未完成创建，因此获取不到；
  2)接着尝试从三级缓存中获取x对象，也获取不到；
  3)紧接着从二级缓存中获取一个ObjectFactory工厂对象；根据前面分析，二级缓存中存在x，故而可以获取到；
  4)跟着调用singletonFactory.getObject();拿到一个半成品的x bean对象；然后把这个x对象放到三级缓存，同时把二级 
    缓存中x清除（**此时二级缓存中只存在一个y了，而三级缓存中多了一个x**）；
3.这是y填充完属性x，继续走完它的生命周期。
===================== y创建完成，继续回到x的创建 =========================
x继续填充属性，getBean(y)获取到y，继续走完它的生命周期

```
