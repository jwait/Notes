##Spring MVC 学习笔记——异常处理
###SpringMVC 中的异常处理
* Spring MVC 通过 HandlerExceptionResolver 处理程序的异常，包括 Handler 映射、数据绑定以及目标方法执行时发生的异常。
* 使用 `<mvc:annotation-driven/>` 配置后，`DispatcherServlet` 默认装配的 `HandlerExceptionResolver`有:
	* ExceptionHandlerExceptionResolver
	* ResponseStatusExceptionResolver
	* DefaultHandlerExceptionResolver

###ExceptionHandlerExceptionResolver
* 处理 Handler 中使用 `@ExceptionHandler` 注解定义的方法，该`@ExceptionHandler` 注解定义的方法用于处理 Handler 中处理方法抛出的异常
* 在 `@ExceptionHandler` 方法的入参中可以加入 Exception 类型的参数，该参数即为对应的异常对象
* `@ExceptionHandler` 方法的入参中不能传入 Map。若希望把异常信息传到页面上，需要使用 ModelAndView 对象作为返回值。
* `@ExceptionHandler` 标记的方法有优先级的问题：：例如发生的是 NullPointerException，但是声明的异常有 RuntimeException 和 Exception，此候会根据异常的最近继承关系找到继承深度最浅的那个 `@ExceptionHandler` 注解方法，即标记了 RuntimeException 的方法
* `@ExceptionAdvice` ：如果在当前的 Handler 中找不到 `@ExceptionHandler` 方法来处理当前方法出现的异常，则将去 `@ExceptionAdvice` 标记的类中查找 `@ExceptionHandler` 标记的方法来处理异常

###ResponseStatusExceptionResolver
* 当处理方法出现异常后，在异常及异常父类中找到 `@ResponseStatus` 注解，然后使用这个注解的属性进行处理。
* 定义一个 `@ResponseStatus` 注解修饰的异常类

		@ResponseStatus(HttpStatus.UNAUTHORIZED)
		public class UnauthorizedException extends RunTimeException{
        
        }

若在处理器方法中抛出了上述异常：
若ExceptionHandlerExceptionResolver 不解析述异常。由于触发的异常 UnauthorizedException 带有@ResponseStatus 注解。因此会被 ResponseStatusExceptionResolver 解析到。最后响应HttpStatus.UNAUTHORIZED 代码给客户端。HttpStatus.UNAUTHORIZED 代表响应码401，无权限。
关于其他的响应码请参考 HttpStatus 枚举类型源码。

###DefaultHandlerExceptionResolver
* 对一些特殊的异常进行处理

###SimpleMappingExceptionResolver
* 如果希望对所有异常进行统一处理，可以使用 SimpleMappingExceptionResolver，它将异常类名映射为视图名，即发生异常时使用对应的视图报告异常

	示例：
    
    	<!-- 配置使用 SimpleMappingExceptionResolver 来映射异常 -->
        <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
            <property name="exceptionAttribute" value="ex"></property>
            <property name="exceptionMappings">
                <props>
                    <prop key="java.lang.ArrayIndexOutOfBoundsException">error</prop>
                </props>
            </property>
        </bean>	
