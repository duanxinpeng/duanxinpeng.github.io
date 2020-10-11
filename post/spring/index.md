
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
	- 三级缓存解决bean的本质其实就是利用了bean处在实例化但还未初始化的这种中间状态
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