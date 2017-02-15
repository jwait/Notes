##Spring MVC 学习笔记——处理静态资源
####为什么需要处理静态资源
* 优雅的 REST 风格的资源URL 不希望带 .html 或 .do 等后缀
* 若将 DispatcherServlet 请求映射配置为 /，则 Spring MVC 将捕获WEB 容器的所有请求，包括静态资源的请求， SpringMVC 会将他们当成一个普通请求处理，因找不到对应处理器将导致错误。

####处理的方法
* 可以在 SpringMVC 的配置文件中配置 `<mvc:default-servlet- handler/>` 的方式解决静态资源的问题：

####`<mvc:default-servlet-handler/>` 标签
* `<mvc:default-servlet-handler/>` 将在 SpringMVC 上下文中定义个`DefaultServletHttpRequestHandler`，它会对进入 `DispatcherServlet` 的请求进行筛查，如果发现是没有经过映射的请求，就将该请求交由 WEB 应用服务器默认的 Servlet 处理，如果不是静态资源的请求，才由 `DispatcherServlet` 继续处理
* 一般 WEB 应用服务器默认的 Servlet 的名称都是 default。若所使用的 WEB 服务器的默认 Servlet 名称不是 default，则需要通过 `default-servlet-name` 属性显式指定