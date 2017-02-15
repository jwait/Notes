##SpringMVC学习笔记——视图解析
###视图
* 视图的作用是渲染模型数据，将模型里的数据以某种形式呈现给客户
* 为了实现视图模型和具体实现技术的解耦，Spring 在 `org.springframework.web.servlet` 包中定义了一个高度抽象的 View 接口
* 视图对象由视图解析器负责实例化。由于视图是无状态的，所以他们不会有线程安全的问题

####常用的视图实现类

|视图类型|说明|
|-------|----|
|InternalResourceView|将 JSP 或其他资源封装成一个视图，是 InternalResourceViewResolver 默认使用的视图实现类|
|JstlView|如果 JSP 使用了JSTL 国际化标签的功能，则需要使用该视图|
|AbstractExcelView| Excel 文档的抽象类。该视图基于 POL 构造 Excel 文档|

###视图解析器
* SpringMVC 为逻辑视图名的解析提供了不同的策略，可以在 Spring WEB 上下文中配置一种或多种解析策略，并指定他们之间的先后顺序。每一种映射策略对应一个具体的视图解析器实现类。
* 视图解析器的作用比较单一：将逻辑视图解析为一个具体的视图对象。
* 所有的视图解析器都必须实现 ViewResolver 接口：

####常用的视图解析器实现类

|视图类型|说明|
|------|----|
|BeanNameViewResolver|将逻辑视图名解析为一个 Bean ，Bean 的 id 等于逻辑视图名|
|InternalResouceViewResolver|将视图名解析为一个 URL 文件，一般使用该解析器将视图名映射为一个保存在 /WEB-INF 目录下的程序文件（如： JSP）|

###视图和视图解析器工作流程
* 请求处理方法执行完成后，最终返回一个 ModelAndView 对象。对于那些返回 String，View 或 ModeMap 等类型的处理方法，Spring MVC 也会在内部将它们装配成一个ModelAndView 对象，它包含了逻辑名和模型对象的视图
* Spring MVC 借助视图解析器（ViewResolver）得到最终的视图对象（View），最终的视图可以是 JSP ，也可能是 Excel、JFreeChart 等各种表现形式的视图
* 对于最终究竟采取何种视图对象对模型数据进行渲染，处理器并不关心，处理器工作重点聚焦在生产模型数据的工作上，从而实现 MVC 的充分解耦

#####使用自定义视图解析器
* 除了使用默认的 InternalResouceViewResolver 视图解析器外，我们还可以自定义地配置一种或混用多种视图解析器。
* 配置其他的视图解析器器，只需在 Spring MVC 的配置文件中声明一个视图解析器的 Bean。
* 每个视图解析器都实现了 Ordered 接口并开放出一个 order 属性，可以通过 order 属性指定解析器的优先顺序，order 越小优先级越高。
* SpringMVC 会按视图解析器顺序的优先顺序对逻辑视图名进行解析，直到解析成功并返回视图对象，否则将抛出 ServletException 异常

示例:

	<!-- 配置视图 BeanNameViewResolver 解析器： 使用视图的名字来解析视图 -->
	<!-- 通过 order 属性来定义视图解析器的优先级，order 的值越小，优先级越高 -->
	<bean class="org.springframework.web.servlet.view.BeanNameViewResolver">
		<property name="order" value="100"></property>
	</bean>

###InternalResouceViewResolver
* 将视图名解析为一个 URL 文件，一般使用该解析器将视图名映射为一个保存在 /WEB-INF 目录下的程序文件
* 若项目中使用了 JSTL，则 SpringMVC 会自动把视图由InternalResourceView 转为 JstlView
	* 若使用 JSTL 的 fmt 标签则需要在 SpringMVC 的配置文件中配置国际化资源文件
* 若希望直接响应通过 SpringMVC 渲染的页面，不希望经过 Handler，可以使用 `<mvc:view-controller>` 标签实现
	* 在实际开发中，通常都还需配置`<annotation-driven>` 标签

###BeanNameViewResolver
* 将逻辑视图名解析为一个 Bean ，Bean 的 id 等于逻辑视图名,也即对于处理方法返回的逻辑视图名字段，将会同于匹配一个 Bean 的 id 名
* 用作视图的 Bean 需要实现 View 接口，并配置到 IOC 容器中

示例:
bean:

	package com.ken.spring.views;

    import java.util.Date;
    import java.util.Map;

    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;

    import org.springframework.stereotype.Component;
    import org.springframework.web.servlet.View;

    @Component("helloView")
    public class HelloView implements View{

        @Override
        public String getContentType() {
            // TODO Auto-generated method stub
            return "text/html";
        }

        @Override
        public void render(Map<String, ?> arg0, HttpServletRequest request, HttpServletResponse response) throws Exception {
            // TODO Auto-generated method stub
            response.getWriter().print("Hello View, time:" + new Date());
        }

    }

处理方法：

	@RequestMapping("/testView")
	public String testView(){
		System.out.println("Hello view");
		return "helloView";
	}


###关于重定向
* 一般情况下，控制器方法返回字符串类型的值会被当成逻辑视图名处理
* 如果返回的字符串中带 forward: 或 redirect: 前缀时，SpringMVC 会对他们进行特殊处理：将 forward: 和redirect: 当成指示符，其后的字符串作为 URL 来处理
	* redirect:success.jsp：会完成一个到 success.jsp 的重定向的操作
	* forward:success.jsp：会完成一个到 success.jsp 的转发操作

示例：

	@RequestMapping("testRedirect")
	public String testRedirect(){
		System.out.println("testRedirect");
		return "redirect:/index.jsp";
	}