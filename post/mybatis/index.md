## 自定义mybatis
### 配置文件
配置文件中主要包括连接数据库的基本信息和Mapper配置文件，基于xml和基于注解的唯一不同点在于得到Mappers对象的方式不同，但是无论xml还是注解都会先得到Mappers对象
### mappers
- 对应接口的全限定类名+方法名为key
- 以sql语句+返回参数类型（实体类）作为value
### 基本流程
以配置文件为基础，依次得到SqlSessionFactoryBuilder、SqlSessionFactory、SqlSession，SqlSession再根据传入的接口的全限定类名以动态代理的方式得到对应接口的代理类。代理类中有mappers和connection，根据connection和mappers就可以连接数据库并执行对应的sql语句，然后将返回结果封装好后返回。
## mybatis环境搭建


## mybatis表格之间的关系
### 一对一
1. 使用 typeMap 定义映射关系
2. 在typeMap中使用association定义另外一个表的内容
3. 例如在account中包含user的typeMap定义如下
```
<resultMap id="accountUserMap" type="account">
	<id property="id" column="aid"></id>
	<result property="uid" column="uid"></result>
	<result property="money" column="money"></result>
	<association property="user" column="uid" javaType="user">
		<id property="id" column="id"></id>
		<result column="username" property="username"></result>
		<result column="sex" property="sex"></result>
		<result column="birthday" property="birthday"></result>
		<result column="address" property="address"></result>
	</association>
</resultMap>
```
### 一对多
1. 使用 typeMap 定义映射关系
2. 在 typeMap 中，使用collection关键字定义一对多映射关系
3. 例如在user中包含多个account，定义如下
```
<resultMap id="userAccountMap" type="user">
	<id property="id" column="id"></id>
	<result property="username" column="username"></result>
	<result property="address" column="address"></result>
	<result property="sex" column="sex"></result>
	<result property="birthday" column="birthday"></result>
	<collection property="accounts" ofType="account">
		<id property="id" column="aid"></id>
		<result property="uid" column="uid"></result>
		<result property="money" column="money"></result>
	</collection>
</resultMap>
```
4. join, left join, right join 
	- join: 左右两边都不为null
	- left join: 左边的数据都选出来，右边的数据可以为null
	- right join: 右边的数据都选出来，左边的数据可以为null
### 多对多

## 延迟加载
1. 延迟加载：在真正使用时才发起查询，不用的时候不查询
2. 立即加载：不管用不用，立即发起查询。
3. 使用 association 时使用延迟加载
	- 开启mybatis对延迟加载的支持
	```
	<settings>
        <!--开启mybatis支持延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>
	```
	- 更改select语句以及resultMap
	```
	<resultMap id="accountUserMap" type="account">
        <id property="id" column="id"></id>
        <result property="uid" column="uid"></result>
        <result property="money" column="money"></result>
        <!-- 使用select属性指定的内容，查询用户的唯一标识
         column属性指定的内容，是用户根据id查询时所需要的参数-->
        <association property="user" column="uid" javaType="user" select="cn.dxp.dao.IUserDao.findById"></association>
    </resultMap>

    <select id="findAll" resultMap="accountUserMap">
        select * from account;
    </select>
	```
3. 使用 collection时延迟加载
	- 更改查询语句
	```
	<select id="findAll" resultMap="userAccountMap">
        select * from user;
    </select>
	```
	- 更改resultMap
	```
	<resultMap id="userAccountMap" type="user">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <result property="address" column="address"></result>
        <result property="sex" column="sex"></result>
        <result property="birthday" column="birthday"></result>
        <collection property="accounts" column="id" ofType="account" select="cn.dxp.dao.IAccountDao.findAccountByUid">
        </collection>
    </resultMap>
	```
	- 为Account添加findAccountByUid方法
	
## mybatis缓存
1. 减少和数据库的交互次数，提高执行效率
2. 适用于缓存
	- 经常查询
	- 不经常改变
	- 而且数据的正确与否对结果影响不大
3. 不适用缓存
	- 经常改变
	- 数据的正确与否对结果影响很大
4. 一级缓存
	- 指的是mybatis中SqlSession对象的缓存
	- 执行查询之后，查询结果会同时存入SqlSession为我们提供的一块区域中，该区域构造是一个map。
	- 当SqlSession消失时，mybatis一级缓存也消失了！ 
	- SqlSession.clearCache() 可以清空一级缓存
	- 当SqlSession 调用修改、添加、删除、commit、close等方法时，就会清空一级缓存。
5. 二级缓存
	- 指的是Mybatis中SqlSessionFactory对象的缓存，由同一个SqlSessionFactory对象创建的SqlSession共享这个缓存。
	- 但是二级缓存中存放的不是对象，而是数据！
	- 二级缓存的使用步骤
		- 让mybatis框架支持二级缓存 （cacheEnable，默认true，无需配置）
		- 让当前的映射文件支持二级缓存（添加<cache/>标签即可）
		- 让当前的操作支持二级缓存（useCache=true）
		
## mybatis注解开发
### 注解开发环境搭建
### 常用注解
1. SQL语句映射
	- @Insert
	- @Select
	- @SelectKey
	- @Update
	- @Delete
2. 结果集映射
	- @Result
	- @Results
	- @ResultMap
3. 关系映射
	- @one
	- @many

## mybatis细节

### mybatis开启别名
```
<typeAliases>
	<!-- 配置扫描的包，该包内的类就可以使用别名，而无需使用全限定类名了 -->
    <package name="cn.dxp.entities"/>
</typeAliases>
```
### mybatis事务
```
config = Resources.getResourceAsStream("sqlconfig.xml");
SqlSessionFactoryBuilder builder=new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(config);
// true：关闭事务，自动便会提交
// false：开启事务，需要用 session.commit() 手动提交事务
session = factory.openSession(true);
accountDao = session.getMapper(IAccountDao.class);
```
### mybatis 实例类
必须可序列化