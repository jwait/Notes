#Spring Framework 总结（三）
***
##Spring 中的事务管理

###事务简介
* 事务管理是企业级应用程序开发中必不可少的技术,  用来确保数据的完整性和一致性. 
* 事务就是一系列的动作, 它们被当做一个单独的工作单元. 这些动作要么全部完成, 要么全部不起作用
* 事务的四个关键属性(ACID)
	* **原子性**(atomicity): 事务是一个原子操作, 由一系列动作组成. 事务的原子性确保动作要么全部完成要么完全不起作用.
	* **一致性**(consistency): 一旦所有事务动作完成, 事务就被提交. 数据和资源就处于一种满足业务规则的一致性状态中.
	* **隔离性**(isolation): 可能有许多事务会同时处理相同的数据, 因此每个事物都应该与其他事务隔离开来, 防止数据损坏.
	* **持久性**(durability): 一旦事务完成, 无论发生什么系统错误, 它的结果都不应该受到影响. 通常情况下, 事务的结果被写到持久化存储器中.

###Spring 中的事务管理
* 作为企业级应用程序框架, Spring 在不同的事务管理 API 之上定义了一个抽象层. 而应用程序开发人员不必了解底层的事务管理 API, 就可以使用 Spring 的事务管理机制.
* Spring 既支持编程式事务管理, 也支持声明式的事务管理. 
* **编程式事务管理**: 将事务管理代码嵌入到业务方法中来控制事务的提交和回滚. 在编程式管理事务时, 必须在每个事务操作中包含额外的事务管理代码. 
* **声明式事务管理**: 大多数情况下比编程式事务管理更好用. 它将事务管理代码从业务方法中分离出来, 以声明的方式来实现事务管理. 事务管理作为一种横切关注点, 可以通过 AOP 方法模块化. Spring 通过 Spring AOP 框架支持声明式事务管理.

###Spring 中的事务管理器
* Spring 从不同的事务管理 API 中抽象了一整套的事务机制. 开发人员不必了解底层的事务 API, 就可以利用这些事务机制. 有了这些事务机制, 事务管理代码就能独立于特定的事务技术了.
* Spring 的核心事务管理抽象是`PlatformTransactionManager`它为事务管理封装了一组独立于技术的方法. 无论使用 Spring 的哪种事务管理策略(编程式或声明式), 事务管理器都是必须的.

###Spring 中的事务管理器的不同实现
* `DataSourceTransactionManager`:在应用程序中只需要处理一个数据源, 而且通过 JDBC 存取
* `JtaTransactionManager`:在 JavaEE 应用服务器上用 JTA(Java Transaction API) 进行事务管理
* `HibernateTransactionManager`:用 Hibernate 框架存取数据库
* ...

######注意：
事务管理器以普通的 Bean 形式声明在 Spring IOC 容器中

###声明式事务
####事务属性
在 Spring 中，声明式事务时通过事务属性（transaction attribute）来定义的。事务属性描述了事务策略如何应用到方法上。事务属性包含了5个方面：**传播行为、隔离级别、回滚策略、是否只读、事务超时**

#####传播行为
* 传播行为定义了客户端与被调用方法之间的事务边界，也即当事务方法被另一个事务方法调用时, 必须指定事务应该如何传播.
* 事务的传播行为可以由传播属性决定。Spring定义了7类传播行为

|传播行为|含义|
|------|----|
|PROPAGATION_MANDATORY|表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常|
|PROPAGATION_NESTTED|表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与 PROPAGATION_REQUIRED 一样|
|PROPAGATION_NEVER|表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常|
|PROPAGATION_NOT_SUPPORTED|表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用 JTATransactionManager 的话，则需要访问 TransactionManager|
|**PROPAGATION_REQUIRED**|表示当前方法必须运行在事务中，如果当前事务存在，方法将会在该事务中运行，否则，会启动一个新的事务|
|**PROPAGATION_REQUIREDS_NEW**|表示当前方法必须运行在自己的事务中，一个新的事务将会被启动，如果存在当前事务，在该方法运行期间，当前事务会被挂起。如果使用 JTATransactionManager 的话，则需要访问 TransactionManager|
|PROPAGATION_SUPPORTS|表示当前方法不需要上下文，但是如果讯在当前事务的话，那么该方法将在这个事务中运行|

####隔离级别
* 隔离级别定义了一个事务可能受其他并发事务影响的程度
* 在典型的应用程序中，多个事务并发运行，经常会操作相同的数据来完成各自的任务，这可能会导致一下的wenti
	* **脏读**
	* **不可重复读**
	* **幻读**
* 在理想的情况下，事务之间是完全隔离的，从而可以防止以上问题的发生。但是完全的隔离会导致性能的问题，因为它通常涉及锁定数据库中的记录，侵占性的锁定会阻碍并发性，要求事务互相等待完成各自的工作。
* 考虑到完全的隔离会导致性能问题，而且并不是所有的应用程序都需要完全的隔离，所以提供多种隔离级别可以在事务隔离上提供一定的灵活性。

|隔离级别|含义|
|------|----|
|ISOLATION_DEFAULT|使用后端数据库默认的隔离级别|
|ISOLATION_READ_UNCOMMITTED|允许尚未提交的数据变更，可能导致脏读、幻读或不可重复读|
|ISOLATION_READ_COMMITTED|允许读取并发事务已经提交的数据。可以阻止脏读，但是幻读或不可重复读仍然可能发生|
|ISOLATION_REPEATABLE_READ|对同一字段的多次读取结果是一致的，除非数据是被本身事务自己说修改。可以阻止脏读和不可重复读，但幻读仍然可能发生|
|ISOLATION_SERIALIZABLE|完全服从ＡＣＩＤ的隔离级别，确保阻止脏读、不可重复读以及幻读。这是非常慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的｜


####回滚策略
* 默认情况下，事务只有在遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚
* 但是可以声明事务在遇到特定的异常时像遇到运行期异常那样回滚。同时也可以声明事务遇到特定的异常不回滚，即使这些异常时运行期异常。

####只读
* 如果一个事务只对后端数据库进行读操作，而不进行写操作，那么将事务设置为只读可以让数据库引擎对这个事务进行优化
* 因为只读优化是在事务启动的时候有数据库实施的，只有对那些具备启动一个新的事务的传播行为（PROPAGATION_REQUIRED、PROPAGATION_REQUIREDS_NEW、PROPAGATION_NESTTED）方法来说，声明为只读才有意义

####事务超时
* 因为事务可能涉及对后端数据库的锁定，所以长时间的事务会不必要地占有是数据库的资源
* 声明事务超时属性，可以在特定的秒数后自动回滚，而不是等待其结束
* 因为超时时钟会在事务开始时启动，所以只有对那些具备可能启动一个新的事务的传播行为（PROPAGATION_REQUIRED、PROPAGATION_REQUIREDS_NEW、PROPAGATION_NESTTED）的方法来说，声明事务超时才有意义


###在 XML 中定义事务
事务管理是一种横切关注点

####第一步：声明事务管理器
Spring并不直接管理事务，而是提供了多种事务管理器，它们将事务管理的职责委托给 JTA 或其他持久化机制所提供的平台事务实现。因而在声明事务管理器时需要根据使用的不同持久化平台来选择不同的事务管理器
######以 JDBC 事务为例（其他事务管理器的声明类似）：

	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>

####第二步:声明事务通知
* 往`<beans>`根元素添加 tx 命名空间，添加后可以使用`<tx-advice>`元素声明事务通知
声明事务通知示例：

		<tx:advice id="txAdvice" transaction-manager="transactionManager">
			<tx:attributes>
				<tx:method name="purchase" propagation="REQUIRES_NEW"/>
				<tx:method name="*"/>
			</tx:attributes>
		</tx:advice>

* 对于 `<tx-advice>` 来说，事务属性定义在 `<tx-attribute>` 元素中，该元素包含了一个或多个的 `<tx-method>` 元素。`<tx-method>`元素为某个（或某些）`name`属性（属性值为匹配表达式）指定的方法定义事务参数
* `<tx-method>`中也有多个属性来帮助定义方法的事务策略
	|隔离级别|含义|
    |------|----|
    |isolation|指定事务的隔离级别|
    |propagation|定义事务的传播规则|
    |read-only|指定事务为只读|
    |rollback-for，no-rollback-for|rollback-for 指定事务对于哪些检查型异常应当回滚而不提交；no-rollback-for 指定事务对于哪些异常应当继续运行而不回滚|
    |timeout|对于长时间运行的事务定义超时时间|

####第三步：声明事务通知需要通知的方法（即需要事务管理的方法）
* `<tx-advice>` 只是定义了 AOP 通知，用于把事务边界通知给方法，但仍然不是完整的事务性切面。因为还没有声明这些事务通知需要应用到哪些方法上
* 因而需要在 XML 的`<aop：config>`标签下使用`<aop：advisor>`标签定义一个通知器

		<aop:config>
            <aop:pointcut expression="execution(* com.ken.spring.tx.xml.service.*.*(..))" 
                id="txPointCut"/>
            <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut"/>
        </aop:config>

* pointcut属性使用 AspectJ表达式来表明通知器所使用的方法，而`<aop：advisor>`标签则将事务通知和切点关联起来


###使用注解定义事务
####第一步：声明事务管理器
与 XML 声明事务的方式一样，使用注解仍然需要声明一个事务管理器（声明方式一样）。

	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>

####第二步：启动注解驱动
* 使用tx命名空间下的 `<tx：annotation-driven>`标签来启动注解驱动
* `<tx：annotation-driven>`元素告诉 Spring 检查上下文中所有的 Bean 并查找使用 `@Transactional` 注解的 Bean ，而不管这个注解是用在类级别上还是方法级别上。对于每一个使用 `@Transactional` 注解的 Bean ，`<tx：annotation-driven>` 会自动给它添加事务通知
* 可以通过 `transactionmanager` 属性来指定特定的事务管理器，默认值为 transactionManager

示例:

	<tx:annotation-driven transaction-manager="transactionManager"/>

####第三步：为需要事务管理的位置添加注解
* 使用 `@Transactional` 进行注解
* `@Transactional` 注解可以使用在类级别上，或者是方法级别上
* 若需要设置事务属性，可以在 `@Transactional` 中进行设置

示例：

	public class BookShopServiceImpl implements BookShopService {

        private BookShopDao bookShopDao;

        public void setBookShopDao(BookShopDao bookShopDao) {
            this.bookShopDao = bookShopDao;
        }

        @Transactional(propagation=Propagation.REQUIRES_NEW)
        @Override
        public void purchase(String username, String isbn) {

            int price = bookShopDao.findBookPriceByIsbn(isbn);

            bookShopDao.updateBookStock(isbn);

            bookShopDao.updateUserAccount(username, price);
        }

	}
