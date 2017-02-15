# C3P0 Configuration

参考资料：[JDBC3 Connection and Statement Pooling Documentation](http://www.mchange.com/projects/c3p0/) 



## Basic Pool Configuration

c3p0 Connection pools are very easy to configure via the following basic parameters:

- [acquireIncrement](http://www.mchange.com/projects/c3p0/#acquireIncrement)
- [initialPoolSize](http://www.mchange.com/projects/c3p0/#initialPoolSize)
- [maxPoolSize](http://www.mchange.com/projects/c3p0/#maxPoolSize)
- [maxIdleTime](http://www.mchange.com/projects/c3p0/#maxIdleTime)
- [minPoolSize](http://www.mchange.com/projects/c3p0/#minPoolSize)

initialPoolSize, minPoolSize, maxPoolSize define the number of Connections that will be pooled. Please ensure that minPoolSize <= maxPoolSize. Unreasonable values of initialPoolSize will be ignored, and minPoolSize will be used instead.

Within the range between minPoolSize and maxPoolSize, the number of Connections in a pool varies according to usage patterns. The number of Connections increases whenever a Connection is requested by a user, no Connections are available, and the pool has not yet reached maxPoolSize in the number of Connections managed.

Since Connection acquisition is very slow, it is almost always useful to increase the number of Connections eagerly, in batches, rather than forcing each client to wait for a new Connection to provoke a single acquisition when the load is increasing. acquireIncrement determines how many Connections a c3p0 pool will attempt to acquire when the pool has run out of Connections. (Regardless of acquireIncrement, the pool will never allow maxPoolSize to be exceeded.)

The number of Connections in a pool decreases whenever a pool tests a Connection and finds it to be broken (see [Configuring Connection Testing](http://www.mchange.com/projects/c3p0/#configuring_connection_testing) below), or when a Connection is expired by the pool after sitting idle for a period of time, or for being too old (See [Managing Pool Size and Connection Age](http://www.mchange.com/projects/c3p0/#managing_pool_size).)



## Managing Pool Size and Connection Age

Different applications have different needs with regard to trade-offs between performance, footprint, and reliability. C3P0 offers a wide variety of options for controlling how quickly pools that have grown large under load revert to minPoolSize, and whether "old" Connections in the pool should be proactively replaced to maintain their reliablity.

- [maxConnectionAge](http://www.mchange.com/projects/c3p0/#maxConnectionAge)
- [maxIdleTime](http://www.mchange.com/projects/c3p0/#maxIdleTime)
- [maxIdleTimeExcessConnections](http://www.mchange.com/projects/c3p0/#maxIdleTimeExcessConnections)

By default, pools will never expire Connections. If you wish Connections to be expired over time in order to maintain "freshness", set maxIdleTime and/or maxConnectionAge. maxIdleTimedefines how many seconds a Connection should be permitted to go unused before being culled from the pool. maxConnectionAge forces the pool to cull any Connections that were acquired from the database more than the set number of seconds in the past.

maxIdleTimeExcessConnections is about minimizing the number of Connections held by c3p0 pools when the pool is not under load. By default, c3p0 pools grow under load, but only shrink if Connections fail a Connection test or are expired away via the parameters described above. Some users want their pools to quickly release unnecessary Connections after a spike in usage that forces a large pool size. You can achieve this by setting maxIdleTimeExcessConnections to a value much shorter than maxIdleTime, forcing Connections beyond your set minimum size to be released if they sit idle for more than a short period of time.

Some general advice about all of these timeout parameters: Slow down! The point of Connection pooling is to bear the cost of acquiring a Connection only once, and then to reuse the Connection many, many times. Most databases support Connections that remain open for hours at a time. There's no need to churn through all your Connections every few seconds or minutes. Setting maxConnectionAge or maxIdleTime to 1800 (30 minutes) is quite aggressive. For most databases, several hours may be more appropriate. You can ensure the reliability of your Connections by testing them, rather than by tossing them. (see [Configuring Connection Testing](http://www.mchange.com/projects/c3p0/#configuring_connection_testing).) The only one of these parameters that should generally be set to a few minutes or less is maxIdleTimeExcessConnections.



## Configuring Connection Test

c3p0 can be configured to test the Connections that it pools in a variety of ways, to minimize the likelihood that your application will see broken or "stale" Connections. Pooled Connections can go bad for a variety of reasons -- some JDBC drivers intentionally "time-out" long-lasting database Connections; back-end databases or networks sometimes go down "stranding" pooled Connections; and Connections can simply become corrupted over time and use due to resource leaks, driver bugs, or other causes.

c3p0 provides users a great deal of flexibility in testing Connections, via the following configuration parameters:

- [automaticTestTable](http://www.mchange.com/projects/c3p0/#automaticTestTable)
- [connectionTesterClassName](http://www.mchange.com/projects/c3p0/#connectionTesterClassName)
- [idleConnectionTestPeriod](http://www.mchange.com/projects/c3p0/#idleConnectionTestPeriod)
- [preferredTestQuery](http://www.mchange.com/projects/c3p0/#preferredTestQuery)
- [testConnectionOnCheckin](http://www.mchange.com/projects/c3p0/#testConnectionOnCheckin)
- [testConnectionOnCheckout](http://www.mchange.com/projects/c3p0/#testConnectionOnCheckout)

idleConnectionTestPeriod, testConnectionOnCheckout, and testConnectionOnCheckin control when Connections will be tested. automaticTestTable, connectionTesterClassName, and preferredTestQuery control how they will be tested.

When configuring Connection testing, first try to minimize the cost of each test. If you are using a JDBC driver that you are certain supports the new(ish) jdbc4 API — and if you are using c3p0-0.9.5 or higher! — let your driver handle this for you. jdbc4 Connections include a method called isValid() that should be implemented as a fast, reliable Connection test. By default, c3p0 will use that method if it is present.

However, if your driver does not support this new-ish API, c3p0's default behavior is to test Connections by calling the getTables() method on a Connection's associated DatabaseMetaData object. This has the advantage of being very robust and working with any database, regardless of the database schema. However, a call to DatabaseMetaData.getTables() is often much slower than a simple database query, and using this test may significantly impair your pool's performance.

The simplest way to speed up Connection testing under a JDBC 3 driver (or a pre-0.9.5 version of c3p0) is to define a test query with the preferredTestQuery parameter. Be careful, however. Setting preferredTestQuery will lead to errors as Connection tests fail if the query target table does not exist in your database *prior to initialization of your DataSource*. Depending on your database and JDBC driver, a table-independent query like SELECT 1 may (or may not) be sufficient to verify the Connection. If a table-independent query is not sufficient, instead of preferredTestQuery, you can set the parameter automaticTestTable. Using the name you provide, c3p0 will create an empty table, and make a simple query against it to test the database.

The most reliable time to test Connections is on check-out. But this is also the most costly choice from a client-performance perspective. Most applications should work quite reliably using a combination of idleConnectionTestPeriod and testConnectionOnCheckin. Both the idle test and the check-in test are performed asynchronously, which can lead to better performance, both perceived and actual.

For some applications, high performance is more important than the risk of an occasional database exception. In its default configuration, c3p0 does no Connection testing at all. Setting a fairly long idleConnectionTestPeriod, and not testing on checkout and check-in at all is an excellent, high-performance approach.



## Configuring Statement Pooling

c3p0 implements transparent PreparedStatement pooling as defined by the JDBC spec. Under some circumstances, statement pooling can dramatically improve application performance. Under other circumstances, the overhead of statement pooling can slightly harm performance. Whether and how much statement pooling will help depends on how much parsing, planning, and optimizing of queries your databases does when the statements are prepared. Databases (and JDBC drivers) vary widely in this respect. It's a good idea to benchmark your application with and without statement pooling to see if and how much it helps.

You configure statement pooling in c3p0 via the following configuration parameters:

- [maxStatements](http://www.mchange.com/projects/c3p0/#maxStatements)
- [maxStatementsPerConnection](http://www.mchange.com/projects/c3p0/#maxStatementsPerConnection)
- [statementCacheNumDeferredCloseThreads](http://www.mchange.com/projects/c3p0/#statementCacheNumDeferredCloseThreads)

maxStatements is JDBC's standard parameter for controlling statement pooling. maxStatements defines the total number PreparedStatements a DataSource will cache. The pool will destroy the least-recently-used PreparedStatement when it hits this limit. This sounds simple, but it's actually a strange approach, because cached statements conceptually belong to individual Connections; they are not global resources. To figure out a size for maxStatements that does not "churn" cached statements, you need to consider the number of *frequently used* PreparedStatements in your application,	and multiply that by the number of Connections you expect in the pool (maxPoolSize in a busy application).

maxStatementsPerConnection is a non-standard configuration parameter that makes a bit more sense conceptually. It defines how many statements each pooled Connection is allowed to own. You can set this to a bit more than the number of PreparedStatements your application *frequently* uses, to avoid churning.

If either of these parameters are greater than zero, statement pooling will be enabled. If both parameters are greater than zero, both limits will be enforced. If only one is greater than zero, statement pooling will be enabled, but only one limit will be enforced.

If statementCacheNumDeferredCloseThreads is greater than zero, the Statement pool will defer physically close()ing cached Statements until its parent Connection is not in use by any client or internally (in e.g. a test) by the pool itself. For some JDBC drivers (especially Oracle), attempts to close a Statement freeze if the parent Connection is in use. This parameter defaults to 0. Set it to a positive value if you observe "APPARENT DEADLOCKS" realted to Connection close tasks. Almost always, that value should be one: if you need more than one Thread dedicated solely to Statement destruction, you probably should set maxStatements and/or maxStatementsPerConnection to higher values so you don't churn through cached Statements so quickly.



## Configuring Recovery From Datebase Outages

c3p0 DataSources are designed (and configured by default) to recover from temporary database outages, such as those which occur during a database restart or brief loss of network connectivity. You can affect how c3p0 handles errors in acquiring Connections via the following configurable properties:

- [acquireRetryAttempts](http://www.mchange.com/projects/c3p0/#acquireRetryAttempts)
- [acquireRetryDelay](http://www.mchange.com/projects/c3p0/#acquireRetryDelay)
- [breakAfterAcquireFailure](http://www.mchange.com/projects/c3p0/#breakAfterAcquireFailure)

When a c3p0 DataSource attempts and fails to acquire a Connection, it will retry up to acquireRetryAttempts times, with a delay of acquireRetryDelay between each attempt. If all attempts fail, any clients waiting for Connections from the DataSource will see an Exception, indicating that a Connection could not be acquired. Note that clients do not see any Exception until a full round of attempts fail, which may be some time after the initial Connection attempt. If acquireRetryAttempts is set to 0, c3p0 will attempt to acquire new Connections indefinitely, and calls to getConnection() may block indefinitely waiting for a successful acquisition.

Once a full round of acquisition attempts fails, there are two possible policies. By default, the c3p0 DataSource will remain active, and will try again to acquire Connections in response to future requests for Connections. If you set breakAfterAcquireFailure to true, the DataSource will consider itself broken after a failed round of Connection attempts, and future client requests will fail immediately.

Note that if a database restart occurs, a pool may contain previously acquired but now stale Connections. By default, these stale Connections will only be detected and purged lazily, when an application attempts to use them, and sees an Exception. Setting maxIdleTime or maxConnectionAge can help speed up the replacement of broken Connections. (See [Managing ConnectionAge](http://www.mchange.com/projects/c3p0/#managing_pool_size).) If you wish to avoid application Exceptions entirely, you must adopt a connection testing strategy that is likely to detect stale Connections prior to their delivery to clients. (See "[Configuring Connection Testing](http://www.mchange.com/projects/c3p0/#configuring_connection_testing)".) Even with active Connection testing (testConnectionOnCheckout set to true, or testConnectionOnCheckin and a short idleConnectionTestPeriod), your application may see occasional Exceptions on database restart, for example if the restart occurs after a Connection to the database has already been checked out.



## Managing Connection Lifecycles with Connection Customizer

Application frequently wish to set up Connections in some standard, reusable way immediately after Connection acquisitions. Examples of this include setting-up character encodings, or date and time related behavior, using vendor-specific APIs or non-standard SQL statement executions. Occasionally it is useful to override the default values of standard Connection properties such as transactionIsolation, holdability, or readOnly. c3p0 provides a "hook" interface that you can implement, which gives you the opportunity to modify or track Connections just after they are checked out from the database, immediately just prior to being handed to clients on checkout, just prior to being returned to the pool on check-in, and just prior to final destruction by the pool. The Connections handed to ConnectionCustomizers are raw, physical Connections, with all vendor-specific API accessible





## Configuring Unresolvded Transaction Handling 

Connections checked into a pool cannot have any unresolved transactional work associated with them. If users have set autoCommit to false on a Connection, and c3p0 cannot guarantee that there is no pending transactional work, c3p0 must either rollback() or commit() on check-in (when a user calls close()). The JDBC spec is (unforgivably) silent on the question of whether unresolved work should be committed or rolled back on Connection close. By default, c3p0 rolls back unresolved transactional work when a user calls close().

You can adjust this behavior via the following configuration properties:

- [autoCommitOnClose](http://www.mchange.com/projects/c3p0/#autoCommitOnClose)
- [forceIgnoreUnresolvedTransactions](http://www.mchange.com/projects/c3p0/#forceIgnoreUnresolvedTransactions)

If you wish c3p0 to allow unresolved transactional work to commit on checkin, set autoCommitOnClose to true. If you wish c3p0 to leave transaction management to you, and neither commit nor rollback (nor modify the state of Connection autoCommit), you may set forceIgnoreUnresolvedTransactions to true. Setting forceIgnoreUnresolvedTransactions is strongly discouraged, because if clients are not careful to commit or rollback themselves prior to close(), or do not set Connection autoCommit consistently, bizarre unreproduceable behavior and database lockups can occur.



## Configuring to Debug and Workaround Broken Client Applications

Sometimes client applications are sloppy about close()ing all Connections they check out. Eventually, the pool grows to maxPoolSize, and then runs out of Connections, because of these bad clients.

The right way to address this problem is to fix the client application. c3p0 can help you debug, by letting you know where Connections are checked out that occasionally don't get checked in. In rare and unfortunate situations, development of the client application is closed, and even though it is buggy, you cannot fix it. c3p0 can help you work around the broken application, preventing it from exhausting the pool.

The following parameters can help you debug or workaround broken client applications.

- [debugUnreturnedConnectionStackTraces](http://www.mchange.com/projects/c3p0/#debugUnreturnedConnectionStackTraces)
- [unreturnedConnectionTimeout](http://www.mchange.com/projects/c3p0/#unreturnedConnectionTimeout)

unreturnedConnectionTimeout defines a limit (in seconds) to how long a Connection may remain checked out. If set to a nozero value, unreturned, checked-out Connections that exceed this limit will be summarily destroyed, and then replaced in the pool. Obviously, you must take care to set this parameter to a value large enough that all intended operations on checked out Connections have time to complete. You can use this parameter to merely workaround unreliable client apps that fail to close() Connections.

Much better than working-around is fixing. If, *in addition to setting* unreturnedConnectionTimeout, you set debugUnreturnedConnectionStackTraces to true, then a stack trace will be captured each time a Connection is checked-out. Whenever an unreturned Connection times out, that stack trace will be printed, revealing where a Connection was checked out that was not checked in promptly. debugUnreturnedConnectionStackTraces is intended to be used only for debugging, as capturing a stack trace can slow down Connection check-out.



## Configuring To Avoid Memory Leak On Hot Redeploy Of Client 

c3p0 spawns a variety of Threads ([helper threads](http://www.mchange.com/projects/c3p0/#numHelperThreads), java.util.Timer threads), and does so lazily in response to the first client request experienced by a PooledDataSource. By default, the Threads spawned by c3p0 inherit a java.security.AccessControlContext and a contextClassLoader property from this first-calling Thread. If that Thread came from a client that may need to be hot-undeployed, references to these objects may prevent the undeployed application, often partitioned into a ClassLoader, from being garbage collected. (See for example[this description of Tomcat memory leaks on redeployment](https://wiki.apache.org/tomcat/MemoryLeakProtection).)

c3p0 provides two configuration parameters that can help with this:

- [contextClassLoaderSource](http://www.mchange.com/projects/c3p0/#contextClassLoaderSource)
- [privilegeSpawnedThreads](http://www.mchange.com/projects/c3p0/#privilegeSpawnedThreads)

contextClassLoaderSource should be set to one of caller, library, or none. The default (which yields the default behavior described above) is caller. Set this to library to use c3p0's ClassLoader, so that no reference is maintained to a client that may need to be redeployed.

privilegeSpawnedThreads is a boolean, false by default. Set this to true so that c3p0's Threads use the the c3p0 library's AccessControlContext, rather than an AccessControlContext that may be associated with the client application and prevent its garbage collection.





## All Configuration

* **acquireIncrement**：默认值3. 定义当连接池中连接耗尽的时候，C3P0一次可以申请创建多少连接
* **acquireRetryAttempt：**默认值30。定义C3P0在放弃从数据库获取连接前尝试获取连接的时间长度。当值为0或小于0时，C3P0会一直尝试获取数据库连接
* **acquireRetryDelay：**默认值1000毫秒，定义C3P0在每一次尝试获取连接间的等待时间
* **autoCommitOnClose：**默认值false，JDBC规范中并没有对此有要求。C3P0默认的策略是回滚所有未提交、挂起的事务。若将autoCommitOnClose属性设置未true，那么C3P0将提交所有未提交和挂起的事务
* **automaticTestTable：**默认值null，若设置了automaticTestTable，那么C3P0将创建一个空的数据表，以设置的值为表名，并且对这个表进行查询以测试连接是否可用。需要注意的是preferredTestQuery设置将被忽略，并且不应该使用这个由C3P0创建的数据表
* **breakAfterAcquireFailure：**默认值false。若设置为true，当多次尝试从数据库中获取连接失败后，DataSource连接池将声明其已损坏并永久关闭。若设置为false，会导致所有等待从连接池获取连接的线程收到异常，但是连接池仍然可用，并且在收到getConnection请求后仍会尝试获取连接
* **checkoutTimeout：**默认值0. 规定客户端在等待连接checked-in或者连接池耗尽是等待连接时的超时时间长度，单位时毫秒。值为0意味者将一直等待。发生超时后将获取失败并抛出SQLException异常
* **connectionCustomizerClassName：**默认值null。[The fully qualified class-name of an implememtation of the ](undefined)[ConnectionCustomizer](http://www.mchange.com/projects/c3p0/apidocs/com/mchange/v2/c3p0/ConnectionCustomizer.html) interface, which users can implement to set up Connections when they are acquired from the database, or on check-out, and potentially to clean things up on check-in and Connection destruction. If standard Connection properties (holdability, readOnly, or transactionIsolation) are set in the ConnectionCustomizer's onAcquire() method, these will override the Connection default values.
* **connectionTesterClassName：**默认值com.mchange.v2.c3p0.impl.DefaultConnectionTester。[The fully qualified class-name of an implememtation of the ](undefined)[ConnectionTester](http://www.mchange.com/projects/c3p0/apidocs/com/mchange/v2/c3p0/ConnectionTester.html) interface, or [QueryConnectionTester](http://www.mchange.com/projects/c3p0/apidocs/com/mchange/v2/c3p0/QueryConnectionTester.html) if you would like instances to have access to a user-configured preferredTestQuery. This can be used to customize how c3p0 DataSources test Connections, but with the introduction of automaticTestTable and preferredTestQuery configuration parameters, "rolling your own" should be overkill for most users.
* **contexClassLoaderSource：**默认值caller。[Must be one of caller, library, or none. Determines how the contextClassLoader (see java.lang.Thread) of c3p0-spawned Threads is determined. If caller, c3p0-spawned Threads (](undefined)[helper threads](http://www.mchange.com/projects/c3p0/#numHelperThreads), java.util.Timer threads) inherit their contextClassLoader from the client Thread that provokes initialization of the pool. If library, the contextClassLoader will be the class that loaded c3p0 classes. If none, no contextClassLoader will be set (the property will be null), which in practice means the system ClassLoader will be used. The default setting of caller is sometimes a problem when client applications will be hot redeployed by an app-server. When c3p0's Threads hold a reference to a contextClassLoader from the first client that hits them, it may be impossible to garbage collect a ClassLoader associated with that client when it is undeployed in a running VM. Setting this to library can resolve these issues. 
* **dataSourceName：**默认值：当创建实例的时候给定的名称，否则是连接池的身份令牌。Every c3p0 pooled data source is given a dataSourceName, which serves two purposes. It helps users find DataSources via [C3P0Registry](http://www.mchange.com/projects/c3p0/#using_c3p0_registry_box), and it is included in the name of JMX mBeans in order to help track and distinguish between multiple c3p0 DataSources even across application or JVM restarts. dataSourceName defaults to the pool's configuration name, if a [named config](http://www.mchange.com/projects/c3p0/#named_configurations) was used, or else to an "identity token" (an opaque, guaranteed unique String associated with every c3p0 DataSource). You may update this property to any name you find convenient. dataSourceName is *not* guaranteed to be unique — for example, multiple DataSource created from the same named configuration will share the same dataSourceName. But if you are going to make use of dataSourceName, you will probably want to ensure that all pooled DataSources within your JVM do have unique names.
* **debugUnreturnedConnectionStackTraces：**默认值false。若设置为true，并且unreturnedConnectionTimeOut设置了一个正值，那么连接池将捕捉所有的check-out连接的跟踪堆栈，并且在check-out连接超时后将其打印。这是用于debug应用程序时候出现内存泄露，应该在对程序debug的时候才设置这个参数
* **driverClass：**默认值null。希望用于提供连接的JDBC驱动的全限定类名。
* **extensions：**默认值为空的java.util.Map。包含所有为这个DataSource定义的用户定义篇配置扩展
* **factoryClassLocation：**默认值null。DataSources that will be bound by JNDI and use that API's Referenceable interface to store themselves may specify a URL from which the class capable of dereferencing a them may be loaded. If (as is usually the case) the c3p0 libraries will be locally available to the JNDI service, leave this set as null.
* **forceIgnoreUnreturnedTransactions：**默认值fals。忽略连接归还时仍未处理的事务，强烈不推荐使用，设置为true可能会导致不易察觉的奇怪的Bug
* **forceSynchronousCheckins：**默认值为false。Setting this to true forces Connections to be checked-in synchronously, which under some circumstances may improve performance. Ordinarily Connections are checked-in asynchronously so that clients avoid any overhead of testing or custom check-in logic. However, asynchronous check-in contributes to thread pool congestion, and very busy pools might find clients delayed waiting for check-ins to complete. Expanding numHelperThreads can help manage Thread pool congestion, but memory footprint and switching costs put limits on practical thread pool size. To reduce thread pool load, you can set forceSynchronousCheckins to true. Synchronous check-ins are likely to improve overall performance whentestConnectionOnCheckin is set to false and no slow work is performed in a ConnectionCustomizer's onCheckIn(...) method. If Connections are tested or other slow work is performed on check-in, then this setting will cause clients to experience the overhead of that work on Connection.close(), which you must trade-off against any improvements in pool performance.
* **foreceUseNameDriverClass：**默认值false。[Setting the parameter ](undefined)[driverClass](http://www.mchange.com/projects/c3p0/#driverClass) causes that class to preload and register with java.sql.DriverManager. However, it does not on its own ensure that the driver used will be an instance of [driverClass](http://www.mchange.com/projects/c3p0/#driverClass), as DriverManager may (in unusual cases) know of other driver classes which can handle the specified [jdbcUrl](http://www.mchange.com/projects/c3p0/#jdbcUrl). Setting this parameter to true causes c3p0 to ignore DriverManager and simply instantiate [driverClass](http://www.mchange.com/projects/c3p0/#driverClass) directly.
* **idleConnectionTestPeriod：**默认值0. If this is a number greater than 0, c3p0 will test all idle, pooled but unchecked-out connections, every this number of seconds
* **initialPoolSize**：默认值3.Number of Connections a pool will try to acquire upon startup. Should be between minPoolSize and maxPoolSize.
* **jdbcUrl**：默认值null。The JDBC URL of the database from which Connections can and should be acquired. Should resolve via java.sql.DriverManager to an appropriate JDBC Driver (which you can ensure will be loaded and available by setting[driverClass), or if you wish to specify which driver to use directly (and avoid DriverManager resolution), you may specify [driverClass](http://www.mchange.com/projects/c3p0/#driverClass) in combination with [forceUseNamedDriverClass](http://www.mchange.com/projects/c3p0/#forceUseNamedDriverClass). Unless you are supplying your own unpooled DataSource, a [must always be provided and appropriate for the JDBC driver, however it is resolved.
* **maxAdministrativeTaskTime**：Default: 0Seconds before c3p0's thread pool will try to interrupt an apparently hung task. Rarely useful. Many of c3p0's functions are not performed by client threads, but asynchronously by an internal thread pool. c3p0's asynchrony enhances client performance directly, and minimizes the length of time that critical locks are held by ensuring that slow jdbc operations are performed in non-lock-holding threads. If, however, some of these tasks "hang", that is they neither succeed nor fail with an Exception for a prolonged period of time, c3p0's thread pool can become exhausted and administrative tasks backed up. If the tasks are simply slow, the best way to resolve the problem is to increase the number of threads, vianumHelperThreads. But if tasks sometimes hang indefinitely, you can use this parameter to force a call to the task thread's interrupt() method if a task exceeds a set time limit. [c3p0 will eventually recover from hung tasks anyway by signalling an "APPARENT DEADLOCK" (you'll see it as a warning in the logs), replacing the thread pool task threads, and interrupt()ing the original threads. But letting the pool go into APPARENT DEADLOCK and then recover means that for some periods, c3p0's performance will be impaired. So if you're seeing these messages, increasing [numHelperThreads](http://www.mchange.com/projects/c3p0/#numHelperThreads) and setting maxAdministrativeTaskTime might help. maxAdministrativeTaskTime should be large enough that any resonable attempt to acquire a Connection from the database, to test a Connection, or to destroy a Connection, would be expected to succeed or fail within the time set. Zero (the default) means tasks are never interrupted, which is the best and safest policy under most circumstances. If tasks are just slow, allocate more threads. If tasks are hanging forever, try to figure out why, and maybe setting maxAdministrativeTaskTime can help in the meantime.Does Not Support Per-User Overrides.
* **maxConnectionAge**：Default: 0,Seconds, effectively a time to live. A Connection older than maxConnectionAge will be destroyed and purged from the pool. This differs from maxIdleTime in that it refers to absolute age. Even a Connection which has not been much idle will be purged from the pool if it exceeds maxConnectionAge. Zero means no maximum absolute age is enforced
* **maxIdleTime**:Default: 0,Seconds a Connection can remain pooled but unused before being discarded. Zero means idle connections never expire.
* **maxIdleTimeExcessConnections**:Default: 0,Number of seconds that Connections in excess of minPoolSize should be permitted to remain idle in the pool before being culled. Intended for applications that wish to aggressively minimize the number of open Connections, shrinking the pool back towards minPoolSize if, following a spike, the load level diminishes and Connections acquired are no longer needed. If maxIdleTime is set, maxIdleTimeExcessConnections should be smaller if the parameter is to have any effect. Zero means no enforcement, excess Connections are not idled out.
* **maxPoolSize**:Default: 15,Maximum number of Connections a pool will maintain at any given time. 
* **maxStatements**:Default: 0The size of c3p0's global PreparedStatement cache. If both maxStatements and maxStatementsPerConnection are zero, statement caching will not be enabled. If maxStatements is zero but maxStatementsPerConnection is a non-zero value, statement caching will be enabled, but no global limit will be enforced, only the per-connection maximum. maxStatements controls the total number of Statements cached, for all Connections. If set, it should be a fairly large number, as each pooled Connection requires its own, distinct flock of cached statements. As a guide, consider how many distinct PreparedStatements are used *frequently* in your application, and multiply that number by maxPoolSize to arrive at an appropriate value. Though maxStatements is the JDBC standard parameter for controlling statement caching, users may find c3p0's alternative maxStatementsPerConnection more intuitive to use.
* **maxStatementsPerConnection**:Default: 0,The number of PreparedStatements c3p0 will cache for a single pooled Connection. If both maxStatements and maxStatementsPerConnection are zero, statement caching will not be enabled. If maxStatementsPerConnection is zero but maxStatements is a non-zero value, statement caching will be enabled, and a global limit enforced, but otherwise no limit will be set on the number of cached statements for a single Connection. If set, maxStatementsPerConnection should be set to about the number distinct PreparedStatements that are used *frequently*in your application, plus two or three extra so infrequently statements don't force the more common cached statements to be culled. Though maxStatements is the JDBC standard parameter for controlling statement caching, users may find maxStatementsPerConnection more intuitive to use.
* **minPoolSize**:Default: 3,Minimum number of Connections a pool will maintain at any given time.
* **numHelperThreads**:Default: 3,c3p0 is very asynchronous. Slow JDBC operations are generally performed by helper threads that don't hold contended locks. Spreading these operations over multiple threads can significantly improve performance by allowing multiple operations to be performed simultaneously.Does Not Support Per-User Overrides.
* **overrideDefaultUser**:Default: null,Forces the username that should by PooledDataSources when a user calls the default getConnection() method. This is primarily useful when applications are pooling Connections from a non-c3p0 unpooled DataSource. Applications that use ComboPooledDataSource, or that wrap any c3p0-implemented unpooled DataSource can use the simple userproperty.Does Not Support Per-User Overrides.
* **overrideDefaultPassword**:Default: null,Forces the password that should by PooledDataSources when a user calls the default getConnection() method. This is primarily useful when applications are pooling Connections from a non-c3p0 unpooled DataSource. Applications that use ComboPooledDataSource, or that wrap any c3p0-implemented unpooled DataSource can use the simple password property.Does Not Support Per-User Overrides.
* **password**:Default: null,For applications using ComboPooledDataSource or any c3p0-implemented unpooled DataSources — DriverManagerDataSource or the DataSource returned by DataSources.unpooledDataSource( ... ) — defines the password that will be used for the DataSource's default getConnection() method. Does Not Support Per-User Overrides.
* **preferredTestQuery**:Default: nullDefines the query that will be executed for all connection tests, if the default ConnectionTester (or some other implementation of  QueryConnectionTester, or better yet [FullQueryConnectionTester](http://www.mchange.com/projects/c3p0/apidocs/com/mchange/v2/c3p0/FullQueryConnectionTester.html)) is being used. Defining a preferredTestQuery that will execute quickly in your database may dramatically speed up Connection tests. (If no preferredTestQuery is set, the default ConnectionTester executes a getTables() call on the Connection's DatabaseMetaData. Depending on your database, this may execute more slowly than a "normal" database query.) NOTE: The table against which your preferredTestQuery will be run must exist in the database schema prior to your initialization of your DataSource. If your application defines its own schema, try automaticTestTable instead.
* **privilegeSpawnedThreads**:Default: false.If true, c3p0-spawned Threads will have the java.security.AccessControlContext associated with c3p0 library classes. By default, c3p0-spawned Threads ([helper threads](http://www.mchange.com/projects/c3p0/#numHelperThreads), java.util.Timer threads) inherit their AccessControlContext from the client Thread that provokes initialization of the pool. This can sometimes be a problem, especially in application servers that support hot redeployment of client apps. If c3p0's Threads hold a reference to an AccessControlContext from the first client that hits them, it may be impossible to garbage collect a ClassLoader associated with that client when it is undeployed in a running VM. Also, it is possible client Threads might lack sufficient permission to perform operations that c3p0 requires. Setting this to true can resolve these issues. Does Not Support Per-User Overrides.
* **propertyCycle**:Default: 0,Maximum time in seconds before user configuration constraints are enforced. Determines how frequently maxConnectionAge, maxIdleTime, maxIdleTimeExcessConnections,unreturnedConnectionTimeout are enforced. c3p0 periodically checks the age of Connections to see whether they've timed out. This parameter determines the period. Zero means automatic: A suitable period will be determined by c3p0. You can call getEffectivePropertyCycle...() methods on a c3p0 PooledDataSource to find the period automatically chosen.]
* **statementCacheNumDeferredCloseThreads**:Default: 0,If set to a value greater than 0, the statement cache will track when Connections are in use, and only destroy Statements when their parent Connections are not otherwise in use. Although closing of a Statement while the parent Connection is in use is formally within spec, some databases and/or JDBC drivers, most notably Oracle, do not handle the case well and freeze, leading to deadlocks. Setting this parameter to a positive value should eliminate the issue. This parameter should only be set if you observe that attempts by c3p0 to close() cached statements freeze (usually you'll see APPARENT DEADLOCKS in your logs). If set, this parameter should almost always be set to 1. Basically, if you need more than one Thread dedicated solely to destroying cached Statements, you should set maxStatements and/or maxStatementsPerConnection so that you don't churn through Statements so quickly. Does Not Support Per-User Overrides.
* **testConnectionOnCheckin**:Default: false If true, an operation will be performed asynchronously at every connection checkin to verify that the connection is valid. Use in combination with idleConnectionTestPeriod for quite reliable, always asynchronous Connection testing. Also, setting an automaticTestTable or preferredTestQuery will usually speed up all connection tests. 
* **testConnectionOnCheckout**:Default: false If true, an operation will be performed at every connection checkout to verify that the connection is valid. **Be sure to set an efficient** preferredTestQuery **or** automaticTestTable **if you set this to** true. **Performing the (expensive) default Connection test on every client checkout will harm client performance.** Testing Connections in checkout is the simplest and most reliable form of Connection testing, but for better performance, consider verifying connections periodically using idleConnectionTestPeriod. 
* **unreturnedConnectionTimeout**:Default: 0.Seconds. If set, if an application checks out but then fails to check-in [i.e. close()\] a Connection within the specified period of time, the pool will unceremoniously destroy() the Connection. This permits applications with occasional Connection leaks to survive, rather than eventually exhausting the Connection pool. And that's a shame. Zero means no timeout, applications are expected to close() their own Connections. Obviously, if a non-zero value is set, it should be to a value longer than any Connection should reasonably be checked-out. Otherwise, the pool will occasionally kill Connections in active use, which is bad. **This is basically a bad idea, but it's a commonly requested feature. Fix your $%!@% applications so they don't leak Connections! Use this temporarily in combination with debugUnreturnedConnectionStackTraces to figure out where Connections are being checked-out that don't make it back into the pool!** 
* **user**:Default: nullFor applications using ComboPooledDataSource or any c3p0-implemented unpooled DataSources — DriverManagerDataSource or the DataSource returned by DataSources.unpooledDataSource() — defines the username that will be used for the DataSource's default getConnection() method. Does Not Support Per-Use
* **usesTraditionalReflectiveProxies**:Default: false.Deprecated. c3p0 originally used reflective dynamic proxies for implementations of Connections and other JDBC interfaces. As of c3p0-0.8.5, non-reflective, code-generated implementations are used instead. As this was a major change, and the old codebase had been extensively used and tested, this parameter was added to allow users to revert of they had problems. The new, non-reflexive implementation is faster, and has now been widely deployed and tested, so it is unlikely that this parameter will be useful. Both the old reflective and newer non-reflective codebases are being maintained, but support for the older codebase may (or may not) be dropped in the future.

