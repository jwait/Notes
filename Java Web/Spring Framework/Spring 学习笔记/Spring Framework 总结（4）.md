#Spring Framework 总结（四）
***
## Spring 整合
####整合 Hibernate 的内容
* 由 IOC 容器来管理 Hibernate 的 Session Factory
* 让 Hibernate 使用上 Spring 的声明式事务

####整合步骤
#####第一步：配置 Hibernate
* 导入 Hibernate 所必须的jar包
* 编写好需要的实体类
* 创建 `*.cfg.xml` 文件对 Hibernate 进行配置。但是不需要配置数据源，因为数据源将由 Spring 的 IOC 容器进行管理。另外也不需要在配置文件中关联 `.hbm.xml`实体映射文件，将 IOC 容器配置 SessionFactory 实例时进行配置。仅需要配置的 Hibernate 基本属性为：方言、SQL显示和格式化，生成数据表的策略以及二级缓存等。

示例：

	<?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE hibernate-configuration PUBLIC
            "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
            "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
    <hibernate-configuration>
        <session-factory>

            <!-- 配置 hibernate 的基本属性 -->
            <!-- 1. 数据源配置到 IOC 容器当中，所以此处不需要配置数据源 -->
            <!-- 2.关联的 .hbm.xml 也在IOC容器配置SessionFactory实例时进行配置 -->
            <!-- 3.配置hibernate的基本属性：方言，SQL 显示及格式化，生成数据表的策略以及二级缓存等 -->


            <property name="hibernate.dialect">org.hibernate.dialect.MySQL5InnoDBDialect</property>
            <property name="hibernate.show_sql">true</property>
            <property name="hibernate.format_sql">true</property>
            <property name="hibernate.hbm2ddl.auto">update</property>
        </session-factory>
    </hibernate-configuration>

####第二步，配置 Spring 并进行整合
* 导入 Spring 必须的jar包
* 编写相应的类
* 创建 Spring 配置文件
* 在配置文件中对 Spring 进行配置，同时也整合 Hibernate
	* 配置数据源
	* 配置 Hibernate 的 SessionFactory 实例：通过 Spring 提供的 LocalSessionFactoryBean 配置
	* 配置 Spring 声明式事务

示例:

	<?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">

        <!-- 配置数据源 -->
        <context:property-placeholder location="classpath:db.properties"/>

        <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
            <property name="user" value="${jdbc.user}"></property>
            <property name="password" value="${jdbc.password}"></property>
            <property name="jdbcUrl" value="${jdbc.jdbcUrl}"></property>
            <property name="driverClass" value="${jdbc.driverClass}"></property>
            <property name="initialPoolSize" value="${jdbc.initialPoolSize}"></property>
            <property name="maxPoolSize" value="${jdbc.maxPoolSize}"></property>
        </bean>


        <!-- 配置 Hibernate 的 SessionFactory 实例：通过 Spring 提供的 LocalSessionFactoryBean 配置 -->
        <bean id="localSessionFactoryBean" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
            <!-- 配置数据源 -->
            <property name="dataSource" ref="dataSource"></property>
            <!-- 配置 Hibernate 配置文件的位置以及名称 -->
            <property name="configLocation" value="classpath:hibernate.cfg.xml"></property>
            <!-- 配置 Hibernate 映射文件的位置以及名称 -->
            <property name="mappingLocations" value="com/ken/spring/hibernate/entities/*.xml"></property>
        </bean>


        <!-- 配置 Spring 声明式事务 -->
        <!-- 1. 配置 Spring 事务管理器 -->
        <bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
            <property name="sessionFactory" ref="localSessionFactoryBean"></property>
        </bean>

        <!-- 2. 配置事务属性，需要一个事务管理器 -->
        <tx:advice id="txAdvice" transaction-manager="transactionManager">
            <tx:attributes>
                <tx:method name="get*" read-only="true"/>
                <tx:method name="*"/>
            </tx:attributes>
        </tx:advice>

        <!-- 3. 配置事务切点，并将切点与事务相关联 -->
        <aop:config>
            <aop:pointcut expression="execution(* com.ken.spring.hibernate.service.*.*(..))" 
                id="pointcut"/>
            <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"/>
        </aop:config>

    </beans>

---
###在 WEB 应用中使用 Spring
####1）. Spring 如何在 WEB 应用中使用
1. 需要加入额外的包
	* spring-web.jar
	* spring-webmvc.jar
2. 配置 Spring 的配置文件，与非 WEB 应用无区别
3. 如何创建 IOC 容器
	* 非 WEB 应用是直接创建的
	* 对于 WEB 应用来说，应该在 WEB 应用被服务器加载时就创建 IOC 容器：也就是在 `ServletContextListener(ServletContextEvent sce)` 方法中创建 IOC 容器
	* Spring 配置文件的名字和位置也应该是可配置的。将其配置到当前 WEB 应用的初始化参数中较为合适
4. 在 WEB 应用中其他组件如何访问 IOC 容器
	* 在 `ServletContextListener(ServletContextEvent sce)` 方法创建 IOC 容器后，可以把其放在一个 `ServletContext` （即Application 域）的属性当中
		    public void contextDestroyed(ServletContextEvent sce)  { 
                //1. 获取 Spring 配置文件
                ServletContext context = sce.getServletContext();
                String config = context.getInitParameter("configLocation");

                // 2. 创建 IOC 容器
                ApplicationContext ctx = new ClassPathXmlApplicationContext(config);

                // 3.将 IOC 容器放入到 ServletContext 的一个属性中
                context.setAttribute("ApplicationContext", ctx);

            }

######更简单的办法：
实际上，Spring 已经提供了一个 Listener 用于在 WEB 应用初始化时创建 IOC 容器

	<?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" id="WebApp_ID" version="3.1">

        <!-- needed for ContextLoaderListener -->
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </context-param>

        <!-- Bootstraps the root web application context before servlet initialization -->
        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>

    </web-app>


###Spring 整合 Struts2
整合的目标是：让 IOC 容器来管理 Struts2 的 Action.
####整合步骤
1. 正常加入 Struts2 ，包括加入 JAR 包、 `web.xml` 文件的配置
2. 在 Spring 的 IOC 容器中配置 Struts2 的 Action.需要注意的是，Spring 的 IOC 容器配置 Struts2 的 Action 时，需要配置 scope 属性，其值为 prototype
		<bean id="personAction" 
            class="com.ken.spring.struts2.action.PersonAction"
            scope="prototype">
            <property name="personService" ref="personService"></property>
        </bean>
3. 配置 Struts2 的配置文件
	* Spring 整合 Struts2时，在 Struts2 中配置的 Action 的 `class` 属性需要指向 IOC 容器中该 Bean 的 `id`，否则该 Action 将由 Struts2 创建，而不是由 Spring 的 IOC 容器创建管理
			<action name="personAction" class="personAction">
                <result>/success.jsp</result>
            </action>
4. 加入一个额外的 jar 包： `struts2-spring-plugin.jar`