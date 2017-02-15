#Spring Framework 总结（二）
***
##Spring 对 JDBC 的支持

###JdbcTemplate （JDBC模板）简介
* 为了使 JDBC 更加易于使用, Spring 在 JDBC API 上定义了一个抽象层, 以此建立一个 JDBC 存取框架.
* 作为 Spring JDBC 框架的核心, JDBC 模板的设计目的是为不同类型的 JDBC 操作提供模板方法. 每个模板方法都能控制整个过程, 并允许覆盖过程中的特定任务. 通过这种方式, 可以在尽可能保留灵活性的情况下, 将数据库存取的工作量降到最低.

###简化 JDBC 模板查询
* 每次使用都创建一个 JdbcTemplate 的新实例, 这种做法效率很低下.
* JdbcTemplate 类被设计成为线程安全的, 所以可以再 IOC 容器中声明它的单个实例, 并将这个实例注入到所有的 DAO 实例中.
* JdbcTemplate 也利用了 Java 1.5 的特定(自动装箱, 泛型, 可变长度等)来简化开发
* Spring JDBC 框架还提供了一个 JdbcDaoSupport 类来简化 DAO 实现. 该类声明了 jdbcTemplate 属性, 它可以从 IOC 容器中注入, 或者自动从数据源中创建.


###使用 JDBC 模板
####配置数据源（DataSource）
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="user" value="${user}"></property>
		<property name="password" value="${password}"></property>
		<property name="jdbcUrl" value="${jdbcUrl}"></property>
		<property name="driverClass" value="${driverClass}"></property>
		<property name="initialPoolSize" value="${initialPoolSize}"></property>
		<property name="maxPoolSize" value="${maxPoolSize}"></property>
	</bean>

####配置 JdbcTemplate 模板 Bean
	<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
在配置 JdbcTemplate 时需要使用 `<property>`元素 引用或配置数据源

####将 JdbcTemplate 注入到 DAO 实现中
	<bean id="bookShopDao" class="com.ken.spring.tx.xml.BookShopDaoImpl">
		<property name="jdbcTemplate" ref="jdbcTemplate"></property>
	</bean>

###使用扩展 JdbcDaoSupport 支持类
Spring提供了3个内置的基类，分别是：`JdbcDaoSupport`,`SimpleJdbcDaoSupport`,`NamedParameterJdbcDaoSupport`,每一个个分别对应不同的Spring模板。使用这些Dao支持类可进一步简化配置。

####定义一个Dao类继承 JDBC DAO 支持类
示例：

	public class PersonDAO excents JdbcDaoSupport{
    
    }
####配置Dao类
#####方法一:
直接将一个 JdbcTemplate Bean装配到它的 jdbcTemplate 属性当中。

	<bean id="person" class="com.ken.spring.jdbc.PersonDao>
    	<property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>

#####方法二:
直接将数据源装配到 dataSource 属性，相比于方法一更加简化

	<bean id="person" class="com.ken.spring.jdbc.PersonDao>
    	<property name="dataSource" ref="dataSource"/>
    </bean>

###使用 JdbcTemplate 更新数据库
* 用 sql 语句和参数更新数据库：
		update
        public int update(String sql,Object...args)throws DataAccessExecption
* 批量更新数据库
		batchUpdate
        public int[] batchUpdate(String sql, List<Object[]> batchArgs)

###使用 JdbcTemplate 查询数据库
* 查询单行
		queryForObject
        public <T> T queryForObject(String sql,
        							ParamaterizedRowMapper<T> rm,
                                    Object... args)
                                throws DataAccessException
* 查询多行
		query
        public <T> List<T> query(String sql,
        					ParamaterizedRowMapper<T> rm,
                            Object... args)
                     throws DataAccessException
* 单值查询
		queryzForObject
        public <T> T queryzForObject(String sql,
        				        	class<T> requiredType,
                                    Object... args)
                             throws DataAccessException