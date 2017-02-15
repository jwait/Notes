##Spring MVC 学习笔记——自定义拦截器
* Spring MVC 也可以使用拦截器对请求进行拦截处理，用户可以自定义拦截器来实现特定的功能

###编写自定义拦截器
######第一步：编写一个自定义的拦截器
* 自定义的拦截器必须实现 `HandlerInterceptor` 接口，并根据需要实现接口下的三个方法：`preHandle()` `postHandle()` `afterCompletion()`

示例：

    package com.ken.springmvc.interceptors;

    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;

    import org.springframework.web.servlet.HandlerInterceptor;
    import org.springframework.web.servlet.ModelAndView;

    public class FirstInterceptor implements HandlerInterceptor{

        @Override
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object object, Exception exception)
                throws Exception {
            System.out.println("[FirstInterceptor] : afterCompletion");
        }

        @Override
        public void postHandle(HttpServletRequest request, HttpServletResponse response, Object object, ModelAndView modelAndView)
                throws Exception {
            System.out.println("[FirstInterceptor] : postHandle");
        }

        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object object) throws Exception {
            System.out.println("[FirstInterceptor] : preHandle");
            return true;
        }

    }

######第二步：配置自定义的拦截器
* 要使用自定义的拦截器需要在 SpringMVC 配置文件的 `<mvc:interceptors>` 标签下进行配置

示例：

	<mvc:interceptors>
		<!-- 配置自定义拦截器 -->
		<bean class="com.ken.springmvc.interceptors.FirstInterceptor"></bean>
	</mvc:interceptors>


###`HandlerInterceptor` 接口
* 接口下提供了三个方法：`preHandle()` `postHandle()` `afterCompletion()`，可以实现处理请求的不同时期进行相关的处理

|方法|描述|
|-|-|
|`preHandle()`|这个方法在业务处理器处理请求之前被调用，在该方法中对用户的请求 request 进行处理。如果需要该拦截器对请求进行拦截处理后还要调用其它的拦截器，或者要调用业务处理器去进行处理，则该方法要返回 true ；如果不需要调用其它组件取处理请求，则返回 false
|`postHandle()`|这个方法在业务处理器处理请求后，在 DispatcherServlet 向客户端返回响应前被调用，在该方法对用户请求iurequest 进行处理
|`afterCompletion()`|这个方法在 DispatcherServlet 完全处理请求后被调用，可以在方法中进行一些资源清理的操作

###拦截器方法执行流程
![拦截器方法执行流程](http://al0n4k.com/imgs/springmvc_interceptor_runningProcess.png)

拦截器的 preHandler 方法 --> 目标方法 --> 拦截器的 postHandler 方法 --> 渲染视图 --> 拦截器的 afterCompletion 方法

###拦截器的配置
* 直接使用 `<mvc:interceptors>` 标签下的 `<bean>` 配置的拦截器将拦截所用的请求
	示例：

        <mvc:interceptors>
            <!-- 配置自定义拦截器 -->
            <bean class="com.ken.springmvc.interceptors.FirstInterceptor"></bean>
        </mvc:interceptors>

* 如果需要控制拦截器去拦截指定的资源，或者不拦截指定的资源，则可以将拦截器配置到 `<mvc:interceptors>` 下的 `<mvc:interceptor>` 标签下。使用 `<mvc:mapping>`或`<mvc:exclude-mapping>`

	示例：
    拦截指定的资源
    
        <mvc:interceptors>
            <!-- 配置自定义拦截器 -->
            <bean class="com.ken.springmvc.interceptors.FirstInterceptor"></bean>

            <mvc:interceptor>
                <mvc:mapping path="/emps"/>
                <bean class="com.ken.springmvc.interceptors.SecondInterceptor"></bean>
            </mvc:interceptor>
        </mvc:interceptors>
    
    不拦截指定的资源
    
    	<mvc:interceptors>
            <!-- 配置自定义拦截器 -->
            <bean class="com.ken.springmvc.interceptors.FirstInterceptor"></bean>

            <mvc:interceptor>
                <mvc:mapping path="/*"/>
                <mvc:exclude-mapping path="/emps"/>
                <bean class="com.ken.springmvc.interceptors.SecondInterceptor"></bean>
            </mvc:interceptor>
        </mvc:interceptors>

###多个拦截方法的执行顺序
![多个拦截方法的执行顺序](http://image.jeepshoe.org/upload/0/9c/09cdbaa7ba9438fc97be3ccf408e8365_thumb.png)

* SpringMVC 中的拦截器执行顺序与 Struts2 拦截器处理的拦截顺序一样，都是根据拦截器栈中，请求消息和响应消息依次经过各个拦截器的顺序执行。
* 也即，在执行 `preHandle()` 方法时按照拦截器栈正序执行，执行 `postHandle()` 方法时按照拦截器栈的反序执行，而执行 `afterCompletion()` 方法时也是按照拦截器栈的反序执行