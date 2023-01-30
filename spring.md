# Spring

### 动态代理基本原理和实现

```
JDK动态代理
代理类实现了InvocationHandler接口，与静态代理不同它持有的目标对象类型是Object，因此代理类能够代理任意的目标对象，给目标对象添加事务控制的逻辑。因此动态代理真正实现了将代码中横向切面逻辑的剥离，实现了代码复用。


JDK动态代理的缺点：
（一）通过反射类Proxy和InvocationHandler回调接口实现JDK动态代理，要求目标类必须实现一个接口，对于没有实现接口的类，无法通过这种方式实现动态代理。
（二）动态代理会为接口中的声明的所有方法添加上相同的代理逻辑，不够灵活




CGLIB动态代理
CGLIB(Code Generation Library)是一个基于ASM的字节码生成库，它允许我们在运行时对字节码进行修改和动态生成。CGLIB支持接口、继承方式实现代理。
原理：动态生成一个要代理类的子类，子类重写要代理的类的所有不是final的方法。在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势将横切逻辑织入（weave）目标对象


两种动态代理方式的比较
JDK动态代理不需要任何外部依赖，但是只能基于接口进行代理；CGLIB通过继承的方式进行代理，无论目标对象有没有实现接口都可以代理，但是无法代理final对象与final方法。（final类型不能有子类，final方法不能被重载）
动态代理是 AOP(Aspect Orient Programming)编程思想，理解动态代理原理，对学习AOP框架至关重要
```



### 事务传播特性，举一个具体的场景

```
当执行某个操作，前10次成功，第11次失败。a 全部回滚；
b 前10次提交，第11次抛异常。ab场景分别如何设置spring事务？


（1）使用默认的事物传播行为就行。
（2）采用PROPAGATION_REQUIRES_NEW


1.PROPAGATION_REQUIRED：如果存在一个事务，则支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。
2.PROPAGATION_SUPPORTS：如果存在一个事务，支持当前事务，如果当前没有事务，就以非事务方式执行。
3.PROPAGATION_MANDATORY：如果存在一个事务，支持当前事务，如果当前没有事务，就抛出异常。PROPAGATION_REQUIRES_NEW：新建事务，如果当前存在事务，把当前事务挂起。
4.PROPAGATION_REQUIRES_NEW：新建事务，如果当前存在事务，把当前事务挂起。
5.PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
6.PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
7.PROPAGATION_NESTED：支持当前事务，新增 Savepoint 点，与当前事务同步提交或回滚。
```



### 事务注解失效

```
1、非 public 修饰的方法上
2、propagation 设置错误（PROPAGATION_SUPPORTS 存在事物，加入，不存在，非事物执行 PROPAGATION_NOT_SUPPORTED 存在事物，挂起，以非事物执行 PROPAGATION_NEVER 存在事物 抛出异常）
3、rollbackFor 设置错误
4、同一个类中方法调用
5、异常被 catch
6、数据库引擎不支持事务（myisam）
```



### 如何调整切面的执行顺序

```
1.通过实现org.springframework.core.Ordered接口
2.通过注解@order
3.通过xml文件
<aop:config expose-proxy="true">
	<aop:aspect ref="aopBean" order="0">  
		<aop:pointcut id="testPointcut"  expression="@annotation(xxx.xxx.xxx.annotation.xxx)"/>  
   		<aop:around pointcut-ref="testPointcut" method="doAround" />  
        </aop:aspect>  
```



### Spring ioc 启动初始化过程

```
Spring 的启动流程主要是定位 -> 加载 -> 注册 -> 实例化
定位 - 获取配置文件路径
加载 - 把配置文件读取成 BeanDefinition
注册 - 存储 BeanDefinition
实例化 - 根据 BeanDefinition 创建实例


关键入口方法 refresh() 
```



### bean的三种注入方式和常见的数据注入类型

```
1. 构造器/方法/属性
```



### bean的生命周期

```
Spring启动，查找并加载需要被Spring管理的bean，进行Bean的实例化


Bean实例化后对将Bean的引入和值注入到Bean的属性中


如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法


如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入


如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将bean所在应用上下文引用传入进来。


如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法。


如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用


如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法。


此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。


如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。

```



### 怎么解决的循环依赖的

```
实例化A的时候，先将A创建（早期对象）放入一个池子中。这个时候虽然属性没有赋值，但是容器已经能认识这个是A对象，只是属性全是null而已。在populateBean方法中对属性赋值的时候，发现A依赖了B，那么就先去创建B了，又走一遍bean的创建过程（创建B）。同样也会把B的早期对象放入缓存中。当B又走到populateBean方法的时候，发现依赖了A，好吧，我们又去创建A呗，但是这个时候去创建A，发现我们在缓存能找到A（早期对象）了。就可以把B的A属性赋值了，这个时候B就初始化完成了。现在回到A调用的populateBean方法中。返回的就是B对象了，对A的B属性进行赋值就可以了
```



### springmvc 拦截器与filter的区别

```
1. filter是servlet规范，被web server管理，interceptor是Spring定义和管理的。
2. interceptor是aop的一种，基于反射，
3. web 请求会先被filter处理，后被interceptor处理
```



### springmvc Controller 是不是单例

```
默认单例
```



### springmvc请求处理流程

```
1、浏览器发起请求，DispatcherServlet接受request，然后从HandlerMapping查找到对应的controller
2、controller处理request请求，并返回ModelAndView对象
3、视图解析器解析ModelAndView对象并返回对应的视图给客户端
```



