##SpringMVC学习笔记——HelloWorld
###步骤：
####第一步：加入 jar 包
* commons-logging-1.2.jar
* spring-aop-4.3.0.RELEASE.jar
* spring-beans-4.3.0.RELEASE.jar
* spring-context-4.3.0.RELEASE.jar
* spring-core-4.3.0.RELEASE.jar
* spring-expression-4.3.0.RELEASE.jar
* spring-web-4.3.0.RELEASE.jar
* spring-webmvc-4.3.0.RELEASE.jar

####第二步：在 web.xml 文件中配置 DispatherServlet
* DispatcherServlet 默认加载 `/WEB-INF/<servletName-servlet>.xml` 的配置文件，启动 WEB 层的 Spring 容器。
* 但是，也可以通过 contextConfigLocation 初始化参数自定义配置文件的位置和名称

        <?xml version="1.0" encoding="UTF-8"?>
        <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://xmlns.jcp.org/xml/ns/javaee"
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
        id="WebApp_ID" version="3.1">

        <!-- 配置 DispatcherServlet -->
        <!-- The front controller of this Spring Web application, responsible for handling all application requests -->
        <servlet>
            <servlet-name>springDispatcherServlet</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

            <!-- 配置 DispatcherServlet 的初始化参数：配置 springMVC 配置文件的位置和名称 -->
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>classpath:springmvc.xml</param-value>
            </init-param>

            <load-on-startup>1</load-on-startup>
        </servlet>

        <!-- Map all requests to the DispatcherServlet for handling -->
        <servlet-mapping>
            <servlet-name>springDispatcherServlet</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>

        </web-app>

####第三步：加入 SpringMVC 配置文件
* 在 `web.xml` 配置文件中，默认或自定义的位置中加入指定名称的的 SpringMVC 配置文件
* 包括：
	* 配置自动扫描的包
		
            <context:component-scan base-package="com.ken.spring.handlers"></context:component-scan>
        
	* 配置视图解析器。视图解析器会将配置的前缀（prefix）和后缀（suffix）与控制器返回的字段拼接，即 prefix + returnVal +suffix 的到实际的物理视图，然后做转发操作
		
            <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            	<property name="prefix" value="/WEB-INF/views/"></property>
            	<property name="suffix" value=".jsp"></property>
       	 </bean>

####第四步：编写处理请求处理器
* 使用 `@Controller` 注解标记为控制器
* 使用 `@requestMapping` 注解来映射请求的 URL

        @Controller
    	public class HelloWorld {

            /**
             * 1.使用 @RequestMapping 注解来映射请求的URL
             * 2.返回值会通过视图解析器解析为实际的物理视图，对于 InternalResourceViewResolver 视图解析器会做
             * 以下操作： 通过 prefix + returnVal +suffix 这样的方式得到实际的物理视图 ，然后做转发操作
             * @return
             */
            @RequestMapping("/helloWorld")
            public String hello(){
                System.out.println("Hello World");
                return "success";
            }

   	 }

####第五步:编写视图
