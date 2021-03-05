## spring design
1. spring最主要的特点就是IOC/DI，也就是控制反转/依赖注入，主要的目的就是实现对象之间的解耦！
2. IOC的主要思想可以简单解释为：通过引入第三方来管理对象，有助于将对象之间的依赖关系进行解耦！
3. 这个第三方就是spring，底层就是一个map容器，以类名key，以这个类的具体对象为value，spring通过类的全限定类名，以反射的方式创建对象，并将对象放入spring容器中进行管理，从而达到对类之间的依赖关系进行解耦的目的。
4. 除此之外，spring还引入了各种设计模式，工厂模式、单例模式等。
5. 控制反转主要体现在两方面：一方面是创建对象的过程被反转了；另外一方面是获取对象的过程反转了，获得对象的过程由自身管理变为IOC主动注入；所以控制反转也可以被称为依赖注入；
6. spring的另外一个特点是AOP（面向切面编程），主要作用是实现方法之间的解耦，以及代码的复用！
7. AOP的主要思想是动态代理，

## 依赖注入 
1. 构造器注入
2. setter方法注入
3. 接口注入
	- 为了将调用者和被调用者在编译期分离，我们可以通过反射创建对象，这就是接口注入的一个雏形
4. @Autowired
	- 根据类型注入：放在变量声明上面
	- 构造器注入：放在构造器上面
	- setter注入：放在setter函数上面

## AOP 
1. 切面：某些共有功能的实现，是某个类
2. 通知：切面的具体实现，是某个方法/函数
3. 切入点：定义通知应用于哪些函数中。

## spring中bean的细节
1. 创建bean的三种方式
	- 使用默认构造函数创建：使用bean标签，配置id和class属性并且没有其他属性和标签时，采用的就是默认构造函数创建bean对象，如果没有默认构造函数，则对象无法创建。
	- 使用普通工厂中的方法创建对象：使用某个类中的方法创建对象，并存入核心容器。
	- 使用静态工厂中的静态方法（使用某个类中的静态方法创建对象）
2. bean作用范围
	- 单例
## Spring常用注解
1. @Component
2. @Autowired
3. @Qualifier

## Spring注解配置
`<context:component-scan base-package="被扫描的包"></context:component-scan>`
## Spring xml 配置bean
`<bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype"></bean>`
## Spring AOP配置
1. 先配置切面
2. 再配置通知
3. 在通知中配置切入点表达式
```
    <!--
        1. 使用aop:config标签表明开始aop的配置
        2. 使用aop:aspect标签配置切面
            id属性：给切面一个唯一标识
            ref属性：指定通知类bean的id
        3. 在aop:aspect标签内部使用对应标签来配置通知的类型
                aop:before 表示前置通知
                method属性，用用户指定哪个方法是通知
                pointcut属性，用于指定切入点表达式，该表达式的含义指的是对业务中哪些方法增强
        4. 切入点表达式的写法
            关键字：execution
            表达式
                访问修饰符 返回值 包名...类名.方法名(参数列表）
                public void service.impl.AccountServiceImpl.saveAccount()
            访问修饰符可以省略
            返回值可以用*代替
            可以使用..表示当前包及其子包
            类名和方法名都可以使用*来实现通配
            参数列表：可以使用..表示有无皆可！-->
    <aop:config>
        <aop:aspect id="logAdvice" ref="logger">
            <aop:pointcut id="pt1" expression="execution(* service.impl.AccountServiceImpl.*(..))"></aop:pointcut>
<!--            <aop:before method="printLog" pointcut-ref="pt1"></aop:before>-->
<!--            -->
            <aop:around method="arountPrintLog" pointcut-ref="pt1"></aop:around>
        </aop:aspect>
        
    </aop:config>
```
## Spring 通过注解配置AOP
```
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
@Component
@Aspect
public class Logger {
    @Pointcut("execution(* service.impl.AccountServiceImpl.*(..))")
    private void pt1(){}

    //@Before("pt1()")
    public void printLog() {
        System.out.println("Logger开始记录日志--------");
    }

    /**
     * Spring框架提供了一个接口，ProceedingJoinPoint，该接口有一个方法proceed() 此方法用于明确调用切入点方法
     * 该接口可以作为环绕通知的方法参数，在程序执行时，spring框架为我们提供该接口的实现类供我们使用
     */
    @Around("pt1()")
    public Object arountPrintLog(ProceedingJoinPoint pjp) {
        Object rtValue = null;
        Object[] args = pjp.getArgs();

        try {
            System.out.println("前置通知");
            pjp.proceed(args);
            System.out.println("后置通知");
        } catch (Throwable throwable) {
            System.out.println("异常通知");
            throwable.printStackTrace();
        } finally {
            System.out.println("最终通知");
        }
        return rtValue;
    }
}
```

## Spring bean的生命周期
1. 声明周期大概包括以下三个阶段，分别对应于以下三个函数
	- 实例化：createBeanInstance()
	- 填充属性：populateBean()
	- 初始化：initializeBean()
2. 创建一个bean的过程如下所示
	- 获取bean定义
	- 实例化一个bean
	- 执行生命周期函数（前，注册注入选项？）
	- 创建依赖项（注册注入选项？）
	- 填充bean（@Autowired注入）
	- 初始化bean
	- 执行生命周期函数（后）
	
## Spring 如何解决循环依赖
1. Spring的循环依赖包括以下三种
	- 构造函数注入产生的循环依赖：无法解决
	- set方法注入产生的循环依赖：无法解决
	- prototype属性的循环依赖：无法解决 
2. 三级缓存
	- 实例化bean之后将其放入缓存中
	- 然后对bean进行属性注入时，如果有依赖其他的bean，就会首先从缓存中获取该bean，如果没有再去实例化bean
	- 三级缓存解决bean的本质其实就是利用了`bean处在实例化但还未初始化的这种中间状态`
3. 只用一级缓存
	- 此时缓存中既有完整的已经注入属性的对象，也有不完整的没有注入属性的对象，如果有其他线程来获取对象，就有可能获取到不完整的对象。
4. 只用二级缓存
	- 如果有aop的话，代理对象是在初始化完成之后的postProcessAfterInitialization()中进行替换的
	- 只用二级缓存的话，在aop情况下，会导致注入到其他bean的不是最终的代理对象，而是原始对象
	
	
## SpringAOP 代理对象的创建
1. 代理对象是在bean实例化完成之后再去创建的
2. 具体是在BeanPostProcessor的postProcessorAfterInitialization()方法中进行
3. 首先找到所有带有@Aspect注解的类，并获取其中没有@Pointcut注解的方法，循环创建切面（切面需要增强和切入点两个要素）根据通知的类型，创建对应的Advance类。切面创建好之后则需要循环判断哪些切面能对当前的bean实例的方法进行增强并排序，最后通过ProxyFactory创建代理对象。
## 如何控制多个aop的执行顺序？
1. 使用@Order(1)注解，order值越小优先级越高
2. @Order注解其实是决定了bean的执行顺序！
## AOP链式调用
1. 

## spring 事务

### 自定义事务管理器
自定义事务管理器的主要思路：
1. 因为事务管理应该在service层，所以可能调用不同的数据库操作，比如两次更新，为了保证多次调用数据库操作使用的是同一个数据库连接，应该用ThreadLocal将数据库连接绑定到当前线程。
2. 事务管理器主要包括四个部分：开启事务、提交事务、回滚事务、释放连接，可以直接将这四个部分提取出来，用动态代理或者spring的aop来进行复用！
3. 上述四个部分中，释放连接可以直接提交事务、回滚事务后面，所以开启事务是多余的；开启事务可以在绑定连接到线程时直接设置成false，所以开启事务也多余；而且提交事务和回滚事务也是固定的！
4. 其实spring已经为我们提供了事务管理器，也就是 PlatformTransactionManager 接口下的一系列实现；

### spring事务
1. spring管理方式
	- 编程式事务
	- 声明式事务
3. 声明式事务管理
	- 基于注解 
	- 基于 xml 配置文件方式
### 声明式事务 
1. PlatformTransactionManager接口 
	- DataSourceTransactionManager实现类（jdbc）
	- HibernateTransactionManager实现类（hibernate）
2. TransactionDefinition（定义了事务的各种信息）
	- 事务名称
	- 事务隔离级别
	- 事务传播行为
		- REQUIRED
		- SUPPORT
		- REQUIRED_NEW
	- 事务的超时时间 
	- 是否只读
3. TransactionStatus
	- 刷新事务
	- 事务是否完成
	- 事务是否回滚
4. TransactionSynchronizationManager
	- Spring将 Dao、Service 类中影响线程安全的所有 “ 状态 ” 都统一抽取到该类中，并用 ThreadLocal 进行封装，这样一来， Dao （基于模板类或资源获取工具类创建的 Dao ）和 Service （采用 Spring 事务管理机制）就变成线程安全的对象啦 O(∩_∩)O~
	- DataSourceUtils：为了保证拿到的数据库连接是线程安全的，需要用此工具类得到数据库连接，而此工具类就是从TransactionSynchronizationManager中获取数据库连接的！
	- ThreadLocal<Map<Object, Object>> resources：将一些资源用tl和线程绑定到一块！
1. 在xml中配置事务管理器
2. 在xml中开启事务注解
3. 把@Transactional添加到类或者方法上面