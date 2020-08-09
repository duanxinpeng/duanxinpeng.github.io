## 设计到的技术
1. mysql
	- 表设计
	- sql技巧
	- 事务和行级锁
2. mybatis
	- dao层的设计与开发
	- mybatis开发
	- mybatis和spring整合
3. spring mvc
	- resful接口设计和使用
	- 框架运作流程
	- controller开发技巧
4. spring
	- spring ioc 整合service
	- 声明式事务
5. 前端
	- 交互设计
	- Bootstrap
	- jquery
6. maven
7. 高并发优化

## 秒杀逻辑
1. 秒杀列表
2. 商品详情页：是否可以秒杀
3. cookie操作

## Idea 快捷键
1. 规整化代码：<kbd>ctrl</kbd>+<kbd>alt</kbd>+<kbd>L</kbd>
2. 插入get、set方法：<kbd>alt</kbd>+<kbd>insert</kbd>
3. 快速创建测试类：<kbd>ctrl</kbd>+<kbd>shift</kbd>+<kbd>T</kbd>

## Java高并发秒杀API之业务分析与DAO层
### 开发流程
1. 创建项目，配置各种依赖
2. 设计数据库，以及 sql 表
3. 针对每一个 sql 表格，编写对对应的实体类，用于封装返回结果
4. 针对每一个 sql 表格，编写对应的 dao 接口，用于实现对表格的增删改查
5. 编写 mybatis-config.xml 全局配置文件
	- 开启驼峰命名转换
	- 使用 jdbc 的 getGenerateKeys 获取数据库自增主键。
6. 编写对应于每个 dao 接口的 mapper。
	- 对应于哪个接口的哪个方法
	- 参数类型
	- 返回值类型（封装）
	- sql 语句
7. 编写 spring-dao.xml 配置文件，整合 spring 和 mybatis
	- 配置数据库连接池 dataSource（driver、url、username、password等）
	- 配置 sqlSessionFactory（dataSource, mybatis-config.xml, entity位置, mapper位置）
	- 配置扫描 dao 接口，利用 sqlSessionFactory 动态实现 dao 接口，注入到spring容器中。
8. 测试
	- @RunWith(SpringJUnit4ClassRunner.class)：在启动junit时，启动SpringIOC容器
	- @ContextConfiguration({"classpath:spring/spring-dao.xml"})：，告诉junit，spring的配置文件在哪里

### dao 层实现逻辑
1. SeckillDao.reduceNumber(seckillId, killTime): 减库存操作，判断库存容量，判断时间是否满足。
2. SeckillDao.queryById(seckillId)：通过 id 查询商品详情
3. SeckillDao.queryAll(offset, limit): 查询offset之后的limit个商品详情
3. SuccessKilledDao.insertSuccessKilled(seckillId,userPhone)：插入成功秒杀信息，以id+userphone为主键，忽略重复插入
4. SuccessKilledDao.queryByIdWithSeckill(seckillId, userPhone)：根据id和userphone查询成功秒杀信息。
	
## Java高并发秒杀API之业务分析与Service层
### 开发流程
1. 编写 service 层接口及其实现。
2. 编写 dto， 用于封装 service 层各种操作的返回值。
3. 编写各种 exception 类
4. 编写秒杀状态枚举类
5. 编写 spring-service.xml 配置文件
	- <context:component-scan base-package="org.seckill.service"/>： 扫描service包下所有使用注解的类型（service类），并初始化这个类。
	- 配置事务管理器
	- 配置基于注解的声明式事务
6. 利用注解实现 SeckillServiceImpl 类的依赖注入
7. 利用注解配置声明式事务
8. 测试	
	- 和 dao 层的测试过程类似，但是需要将 spring-service.xml 的位置也告诉junit。
### service层实现逻辑
1. SeckillService.getSeckillList()：返回从0开始的4个商品的详情。
2. SeckillService.getById(seckillId)：返回 seckillId 所对应的商品的详情。
3. SeckillService.exportSeckillUrl(seckillId)：
	- 根据 seckillId 查询并得到商品的详情
	- 判断秒杀是否已经开始或者已经结束
	- 根据 seckillId 加盐之后得到一个 md5 码。
	- 将是否暴露链接、md5码、seckillId、秒杀时间等内容封装到 dto 下的 Exposer 类中返回。
4. SeckillService.executeSeckill(seckillId,userPhone,md5)
	- 判断 md5 码是否合法
	- 根据 seckillId 和 当前时间执行减库存操作
	- 根据减库存操作判断是否成功，判断是否需要将秒杀成功信息插入数据库。
	- 将seckillId、秒杀状态、秒杀是否成功等信息封装到 dto 下的 SeckillExecutor 类中并将其返回。
	- 处理各种异常：重复秒杀异常、秒杀关闭异常等

### 声明式事务
1. 在SeckillService.executeSeckill操作中，因为有两个以上的数据库操作，所以需要配置事务，此处采用声明式事务。
2. 如果不采用声明式事务，就很麻烦了，需要先得到数据库连接，并且用threadlocal保存该连接，通过该数据库连接进行事务操作。
### 枚举类
1. 它既是一种 class 类型，又比class类型多了些特殊约束，但是这些约束的存在也造就了枚举类型的简洁性、安全性和便捷性。
2.
 
## Java高并发秒杀API之业务分析与web层
### 开发流程
1. 按照 restful 规范设计 URL
	- GET /seckill/list 获取秒杀列表
	- GET /seckill/{id}/detail 获取商品{id}详情页
	- GET /seckill/timie/now 获取服务器系统时间
	- POST /seckill/{id}/exposer  暴露秒杀地址
	- POST /seckill/{id}/{md5}/execution  执行秒杀
2. 配置 SpringMVC 框架
	- webapp/WEB-INF/web.xml 
		- 配置 DispatcherServlet 
			- 配置 SpringMVC 需要加载的配置文件 spring-dao.xml, spring-service.xml, spring-web.xml
		- 配置 servlet-mapping，以映射某些请求到 DispatcherServlet 上
	- spring-web.xml
		- 开启 SpringMVC 注解模式
		- 静态资源配置
		- 配置 ViewResolver，jsp文件的位置、前缀、后缀。
		- 扫描 web 相关的 bean（SeckillController）
3. 实现 Restful 接口（用 Controller 实现 URL 对应的功能）
4. 利用 BootStrap 框架开发前端页面
5. 用 JavaScript 实现前端页面交互逻辑
	
### SpringMCV 框架流程
1. 接收到一个 http 请求之后， DispatcherServlet 根据 HandlerMapping 来选择调用适当的 Controller.
2. Controller 接收请求，根据请求的 URL 及其 GET/POST 类型，来调用对应的 service 方法。
3. Controller 执行完 service 层的逻辑之后，向 DispatcherServlet 返回 ModelAndView， 这里的 mode 就是 service 层的返回数据，view 就是用来装载这些数据的静态页面。
4. DispatcherServlet 从 ViewResolver 获取帮助，为请求检取定义试图（将 model 封装到 view 中？）
5. DispatcherServlet 将 model 传给 view 并将 view 返回给浏览器。

### Controller实现细节
1. SeckillControler.list(Model)
	- /seckill/list `GET`
	- 调用 service 层的 getSeckillList()方法，将返回结果添加到 model中
	- 返回 String "list"
2. SeckillControler.detail(seckillId,model)
	- /seckill/{seckillId}/detail `GET`
	- 调用 service 层的getById() 方法, 将返回结果加入到 model 中
	- 返回 String "detail" /"redirect:/seckill/list"/"forward:/seckill/list"。
3. SeckillController.time()
	- /seckill/time/now `GET`
	- @ResponseBody 
	- 生成系统当前时间，并将结果封装到 SeckillResult 中返回。（以 json 的数据格式封装到 response 的 body 区域？）
4. SeckillControler.exposer(seckillId)
	- /seckill/{seckillId}/exposer `POST`
	- @ResponseBody
	- 调用 service 层的 exportSeckillUrl 方法，得到返回值 Exposer，并将其封装到 SeckillResult 中返回。
5. SeckillControler.execute(seckillId,md5,phone)
	- /seckill/{seckillId}/{md5}/execution
	- phone 从 cookie 中获取！
	- 调用 service 层的 executeSeckill 方法，得到 SeckillExecutor 返回值，并将其封装到 SeckillResult 中返回。
### BootStrap 
1. list
	- 可从 mode 中直接获取 "list"。${list}
2. detail 
	- 
### restful 规范
1. 每个 URL 都代表一种资源
2. 客户端和服务器之间，传递这种资源的某种表现层
3. 客户端通过四个 Http 动词，对服务器的资源进行操作，实现“表现层状态转化”
	- GET 查询操作，获取资源
	- POST 添加/修改操作，新建/更新资源
	- PUT 修改操作，更新资源
	- DELETE 删除操作，删除资源
## Java高并发秒杀API之高并发优化
### redis 优化秒杀接口暴露
1. 配置依赖
	- redis 客户端依赖 
		- jedis
	- protostuff 序列化依赖
		- protostuff-core
		- protostuff-runtime
2. 在 dao 层添加 RedisDao 类，用于对 redis 的操作
	- putSeckill
	- getSeckill
3. 通过配置将 RedisDao 注入 Spring 容器
4. 修改 service 层逻辑
	- 先通过 redis get 
	- 如果为 null， 再通过  db get，并将结果 put 到 redis 中。
### 利用 mysql 的存储过程优化秒杀操作
1. 在 MySQL 中创建一个执行
### 利用 protostuff 序列化以及反序列化
1. protostuff 进行序列化要比 java 自带的序列化效率更高
2. 序列化 
	```
	private RuntimeSchema<SecKill> schema = RuntimeSchema.createFrom(SecKill.class);
	byte[] bytes = ProtostuffIOUtil.toByteArray(seckill, schema,
                        LinkedBuffer.allocate(LinkedBuffer.DEFAULT_BUFFER_SIZE));
	```
3. 反序列化
	```
	private RuntimeSchema<SecKill> schema = RuntimeSchema.createFrom(SecKill.class);
	SecKill secKill = schema.newMessage();
	ProtostuffIOUtil.mergeFrom(bytes, secKill, schema);
	```
## 常用注解
1. @PathVariable：将 URL 中占位符参数绑定到处理器的方法形参中。
2. @RequestMapping：用来处理请求地址映射。
3. @Param("") ：mybatis提供的注解，作用是用来传递参数
4. @ResponseBody：将方法的返回值以特定的格式写入到 response 的 body 区域，进而将数据返回给客户端。如果不写此注解，方法则将返回值封装为 ModelAndView 对象。
5. @Cookie : 利用 jquery-cookie 插件从 cookie 中获取数据。