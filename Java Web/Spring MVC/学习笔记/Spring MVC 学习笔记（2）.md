##SpringMVC学习笔记——请求映射

###使用 `@RequestMapping` 映射请求
* Spring MVC 使用 `@RequestMapping` 注解为控制器指定可以处理的哪些 URL 请求
* 在控制器的类定义以及方法定义处都可以标注 `@RequestMapping`
	* **类定义处**： 提供初步的请求映射信息。相对于 WEB 应用的根目录
	* **方法处**： 提供进一步的细分映射信息。相对于类定义处的 URL。 若类定义处未标注 `@RequestMapping` ，则方法标注出的 URL 相对于 WEB 应用的根目录
* DispatcherServlet 截获请求后，就通过控制器上的 `@RequestMapping` 提供的映射信息来确定请求所对应的处理方法。

示例：

	package com.ken.spring.handlers;

    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;

    @RequestMapping("/springMVC")
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

###映射请求参数、请求方法或请求头
* `@RequestMapping` 除了可以指定请求 URL 映射请求外，还可以指定请求方法、请求参数以及请求头映射条件。
* `@RequestMapping` 的 value、method、params及heads，分别表示请求 URL、请求方法、请求参数及请求头的映射条件，它们之间是“与”关系，联合使用多个条件可以让请求映射更加精确化。
指定请求方法（常用）实例：

        @RequestMapping(value="/testMethod",method=RequestMethod.POST)
        public String testMethod(){
            System.out.println("testMethod");
            return SUCCESS;
        }

* params 和 heads 支持简单的表达式：
	* params1：表示请求必须包含名为 params1 的请求参数
	* !params： 表示请求不能包含名为 params1 的请求参数
	* params1 != value1：表示请求包含名为 params1 的参数，但其值不能为 value1
	* {"params2","params1 != value1"}:可以使用数组形式指定多个请求参数和请求头。
请求参数和请求头示例：

            @RequestMapping(value="/testParamsAndHeads",
                    params={"username","age!=10"})
            public String testParamsAndHeads(){
                System.out.println("testParamsAndHeads");
                return SUCCESS;
            }

###Ant风格 URL
* Ant 风格资源地址支持三种匹配符
	* `？`：匹配文件名中的一个字符
	* `*`：匹配文件名中任意字符
	* `**`：匹配多层地址
* `@RequestMapping`：支持 Ant 风格的 URL
	* /user/*/createUser: 匹配 –
/user/aaa/createUser、/user/bbb/createUser 等 URL
	* /user/**/createUser: 匹配 –
/user/createUser、/user/aaa/bbb/createUser 等 URL
	* /user/createUser??: 匹配 –
/user/createUseraa、/user/createUserbb 等 URL

示例:

	@RequestMapping(value="/testAnt/*/abc")
	public String testAnt(){
		System.out.println("testAnt");
		return SUCCESS;
	}

###`@PathVariable` 映射 URL 绑定的占位符
* 带占位符的 URL 是 Spring3.0 新增的功能，该功能在 SpringMVC 向 REST 目标挺进发展过程中具有里程碑的意义
* 通过 `@PathVariable` 可以将 URL 中的占位符参数绑定到控制器处理方法的入参中：URL 中的 `{xxx}` 占位符可以通过 `@PathVariable(xxx)` 绑定到操作方法的入参中
示例：

		@RequestMapping(value="testPathVariable/{id}")
        public String testPathVariable(@PathVariable("id") Integer id){
            System.out.println("testPathVariable" + id);
            return SUCCESS;
        }


###REST 风格的 URL
####什么是 REST
* REST：即 Representational State Transfer。（资源）表现层状态转化。是目前最流行的一种互联网软件架构。它结构清晰、符合标准、易于理解、扩展方便，所以正得到越来越多网站的采用
* 资源（Resources）：网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的存在。可以用一个URI（统一资源定位符）指向它，每种资源对应一个特定的 URI 。要获取这个资源，访问它的URI就可以，因此 URI 即为每一个资源的独一无二的识别符
* 表现层（Representation）：把资源具体呈现出来的形式，叫做它的表现层 （Representation）。比如，文本可以用 txt 格式表现，也可以用 HTML 格式、XML 格式、JSON 格式表现，甚至可以采用二进制格式。
* 状态转化（State Transfer）：每发出一个请求，就代表了客户端和服务器的一次交互过程。HTTP协议，是一个无状态协议，即所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生“状态转化”（State Transfer）。而这种转化是建立在表现层之上的，所以就是 “表现层状态转化”。具体说，就是 HTTP 协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。它们分别对应四种基本操作：GET 用来获取资源，POST 用来新建资源，PUT 用来更新资源，DELETE 用来删除资源。

####使用HiddenHttpMethodFilter
* 浏览器 form 表单只支持 GET 与 POST 请求，而DELETE、PUT 等 method 并不支持。所以可以使用这个过滤器可以将 POST 请求 转换成 DELETE 与 PUT 请求

####以CRUD 为例：
######基本操作：
* 新增：/order POST
* 修改：/order/1 PUT
* 获取：/order/1 GET
* 删除：/order/1 DELETE

######如何发送 PUT 请求 和 Delete 请求
1. 需要配置HiddenHttpMethodFilter

        <filter>
            <filter-name>HiddenHttpMethodFilter</filter-name>
            <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
        </filter>
        <filter-mapping>
            <filter-name>HiddenHttpMethodFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>

2. 需要发送 POST 请求
3. 需要在发送 POST 请求是携带一个 `name=“_method”` 的隐藏域，值为 PUT 或 DELETE

		<form action="springMVC/testRest/1" method="post">
            <input type="hidden" name="_method" value="DELETE">
            <input type="submit" value="Rest_delete">
        </form>
处理方法：

		@RequestMapping(value="/testRest/{id}",method=RequestMethod.DELETE)
        public String testRestDelete(@PathVariable("id") Integer id){
            System.out.println("Rest Delete:" + id);
            return SUCCESS;
        }

###使用 `@RequestParam` 映射请求参数
* 在处理方法入参处，使用 `@RequestParam` 可以将 URL 中的请求参数传递给请求方法
* value属性：请求参数的参数名
* require：该参数是否必须。默认值为 true
* defaultValue：请求参数的默认值。

示例:
处理方法

	@RequestMapping(value="/testRequestParam")
	public String testRequestParam(@RequestParam(value="username") String username,
			@RequestParam(value="age",required=false) Integer age){
		System.out.println("testRequestParam" + " username:" + username + ",age:" + age);
		return SUCCESS;
	}

当请求 URL 为：`http://localhost:8080/SpringMVC-1/springMVC/testRequestParam?username=ken&age=21` 时，处理方法将获得名为 username 和 age 的两个请求参数，因此输出为：`testRequestParam username:ken,age:21`
当请求 URL 为：`http://localhost:8080/SpringMVC-1/springMVC/testRequestParam?username=ken`，时，URL 只含有一个请求参数，而名为 age 请求参数不存在，因为 `require` 属性设置为 false，故输出为：`testRequestParam username:ken,age:null`

###使用 `@RequestHeader` 映射请求头
* 请求头包含了若干个属性，服务器可据此获知客户端的信息，通过 @RequestHeader 即可将请求头中的属性值绑定到处理方法的入参中
* `@RequestHeader` 注解的用法与 `@RequestParam` 注解用法类似

示例：

	@RequestMapping(value="/testRequestHeader")
	public String testRequestHeader(@RequestHeader(value="Accept-Language") String al){
		System.out.println("testRequestHeader:" + al);
		return SUCCESS;
	}

###使用 `@CookieValue` 映射请求中的 Cookie 值
* 与 `@RequestHeader` 和 `@RequestParam` 的作用类似，可以将处理方法的一个入参与某个 Cookie 绑定
* `@CookieValue` 注解可以设置的属性有： `defaultValue`、`value`、`require`

示例：

	@RequestMapping("/CookieValue")
	public String CookieValue(@CookieValue(value="JSESSIONID") String sessionId){
		System.out.println("CookieValue :" + sessionId);
		return SUCCESS;
	}

###使用 POPJ 对象绑定请求参数值
* 使用 `@RequestParam` 注解可以绑定请求参数值。但是当请求参数比较多的时候，而且这些参数都是某个 POPJ 对象的属性时，若使用 `@RequestParam` 需要逐个请求参数进行绑定，还需要将它们组装成 POPJ 对象，比较麻烦。
* Spring MVC 可以按请求参数名和 POPJ 属性名进行自动匹配，自动为该对象填充属性值。而且还支持级联属性。

示例：
处理方法

	@RequestMapping("/testPopj")
	public String testPopj(User user){
		System.out.println(user);
		return SUCCESS;
	}
POPJ 对象（User）

	public class User {

        private String username;
        private String password;
        private String email;
        private String age;
        private Address address;
        
        ...
	}

###使用 Servlet API 作为入参数
* 当开发时需要使用到 Servlet API 时，可以将其作为入参传入处理方法
* 处理方法接受的 Servlet API 类型
	* HttpServletRequest
	* HttpServletResponse
	* HttpSession
	* java,security,Principal
	* Locale
	* InputStream
	* OutputStream
	* Reader
	* Writer

