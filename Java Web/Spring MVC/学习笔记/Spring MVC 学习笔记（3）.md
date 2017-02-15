##SpringMVC学习笔记——处理模型数据
######Spring MVC 提供了一下几种途径输出模型数据
* `ModelAndView`：处理方法返回值类型为 ModelAndView，处理方法可以通过该对象添加模型数据
* `Map` 及 `Model`：入参为：`org.springframework.ui.Model`、`org.springframework.ui.ModelMap` 或 `java.uti.Map` ，处理方法返回时，Map 中的数据会自动添加到模型中
* `@SessionAttributes`：将模型中的某个属性暂存到 HttpSession 中，以便多个请求之间共享这个属性
* `@ModelAttribute`：方法的入参标注该注解后，入参的对象就会放到数据模型中

###ModelAndView
* 控制器处理方法的返回值如果是 `ModelAndView` 则其既包含视图信息，也包含模型数据信息
* 添加模型信息（即希望从处理方法传递到视图的数据）
	* `MoelAndView addObject(String attributeName, Object attributeValue)`
	* `ModelAndView addAllObject(Map<String, ?> modelMap)`
* 设置视图（即希望控制器处理方法之后希望转到的视图，相当于未使用 `ModelAndView` 前返回的视图名称字段）
	* `void setView(View view)`
	* `void setViewName(String viewName)`
* 添加的模型信息，自动放入到 RequestScope 域中

示例：

	@RequestMapping("/testModelAndView")
	public ModelAndView testModelAndView(){
		String viewName = SUCCESS;
		ModelAndView modelAndView = new ModelAndView(viewName);
		
		modelAndView.addObject("Time", new Date());
		
		return modelAndView;
	}

在视图中使用这些模型数据：

	time:"${requestScope.Time}"


### Map 及 Model
* Spring MVC 在内部使用了一个org.springframework.ui.Model 接口存储模型数据
* 处理方法的入参可以为 `Map` 或者是 `Model` 类型，在处理方法中将模型数据放入到以上的入参中，即可将处理方法产生的模型数据传递到视图中。
* 具体步骤：
	* Spring MVC 在调用方法前会创建一个隐含的模型对象作为模型数据的存储容器。
	* 如果方法的入参为 Map 或 Model 类型，Spring MVC 会将隐含模型的引用传递给这些入参。在方法体内，开发者可以通过这个入参对象访问到模型中的所有数据，也可以向模型中添加新的属性数据
* 模型数据同样也是放在 RequestScope （请求域）中


示例：

	@RequestMapping("/testMap")
	public String testMap(Map<String, Object> map){
		
		map.put("names", Arrays.asList("Tom","ken"));
		
		return SUCCESS;
	}

在视图中使用这些模型数据：

	names:${requestScope.names }


### `@SessionAttributes`注解
* 若希望在多个请求之间共用某个模型属性数据，则可以在控制器类上标注一个 `@SessionAttributes`, Spring MVC 将在模型中对应的属性暂存到 `HttpSession` 中。
* `@SessionAttributes` 可以设置 `value` 和 `type` 属性，使用这些属性除了可以通过**属性名**(通过`value`)指定需要放到会话中的属性外，还可以通过模型属性的**对象类型**(通过 `type`)指定哪些模型属性需要放到会话中。
例如：
	* @SessionAttributes(types=User.class) ：会将隐含模型中所有类型为 User.class 的属性添加到会话中。
	* @SessionAttributes(value={“user1”, “user2”}) ：可以使用‘{}’设置多个要放到 SessionScope 中的属性
	* @SessionAttributes(types={User.class, Dept.class})：同样地，也可以使用‘{}’设置多种需要放到 SessionScope 中的对象类型
	* @SessionAttributes(value={“user1”, “user2”}, types={Dept.class})：两种属性可以同时使用

* 需要注意的是：该注解只能放在类上面，而不能修饰方法

示例:

	@SessionAttributes(value={"user"},types={String.class})
    @RequestMapping("/springMVC")
    @Controller
    public class HelloWorld {

        private static final String SUCCESS = "success";

        @RequestMapping("/testSessionAttribute")
        public String testSessionAttribute(Map<String , Object> map){
            User user = new User("user", "123", "11@qq.com", "21");
            map.put("user", user);
            map.put("language", "Java");

            return SUCCESS;

        }
	...
    }

在视图中使用这些模型数据：

	requestScope user:${requestScope.user }
	<br>
	sessionScope user:${sessionScope.user }
	<br>
	requestScope language:${requestScope.language }
	<br>
	sessionScope language:${sessionScope.language }
	<br>

### `@ModelAttribute` 注解
* 在方法定义上使用 @ModelAttribute 注解：Spring MVC 在调用目标处理方法前，会先逐个调用在方法级上标注了 `@ModelAttribute` 的方法。
* `@ModelAttribute` 注解也可以来修饰目标方法 POJO 类型的入参，其 value 值有以下作用：
	* Spring MVC 使用 value 属性值在 implicitModel 中查找对应的对象，若存在则会则会直接传入到目标方法的入参中。
	* Spring MVC 会以 value 属性值为 key，POJO 类型的对象为 value ，放入到请求域（requestScope）中


示例:

	@RequestMapping("/springMVC")
    @Controller
    public class HelloWorld {

        private static final String SUCCESS = "success";

        @ModelAttribute
        public void getUser(@RequestParam(value="id",required=false) Integer id,
                Map<String, Object> map){
            if(id != null){
                User user = new User(1, "ken", "123456", "11@qq.com", "12");
                System.out.println("从数据库中获取一个对象：" + user);
                map.put("user", user);
            }
        }

        @RequestMapping("/testModelAttribute")
        public String testModelAttribute(@ModelAttribute("user") User user){
            System.out.println(user);

            return SUCCESS;
        }
        ...
	}

以上代码中：
1、执行 `@ModelAttribute` 注解标注的方法：从数据库中取出对象，把对象放到 Map 中，键为: user
2、Spring MVC 从 Map 中取出 User 对象，并把表单的请求参数赋值给该 User 对象的对应属性
3、Spring MVC 把上述对象传入目标方法的参数

注意： 在`@ModelAttribute` 标注的方法中，放入到 Map 时的键值需要和目标方法入参类型的第一个字母小写的字符串一致


#####`@ModelAttribute`源码分析流程：
1. 调用 @ModelAttribute 注解修饰的方法。实际上是把 @ModelAttribute 修饰方法中 Map 的数据放到 implicitModel 中。
2. 解析请求处理器的目标参数，实际上该参数来自于 WebDataBinder 对象的 target 属性
	1.创建 WebDataBinder 对象：
		1)确定 ObjectName 属性：若传入的 attrName 属性为 ""，则 ObjectName 为类名的第一个字母。
        注意：attrName。若目标方法的 POJO 属性使用了 @ModelAttribute 来修饰，则 attrName 值即为 @ModelAttribute 的 value 值
        2）确定 target 属性：
        >在implicitModel 中查找 atrName 对应的属性值。若存在，则OK
        >若不存在：则验证当前 Handler 是否使用了 @sessionAttribute 进行修饰，若使用了，则尝试从 session 中获得 attrName 所对应的属性值。若 session 中没有对应的属性值，则抛出异常。
        > 若 Handler 没有使用 @sessionAttribute 进行修饰，或 @sessionAttribute 中没有使用 value 值指定的 key 和 attrName 相匹配，则通过反射创建了 POJO 对象
	2.SprinMVC 将表单请求参数赋值给 WebDataBinder 的 target 属性
	3.SpringMVC 把 WebDataBinder 的 attrrName 和 target 给到 implicitModel。进而传到 request 域对象中
	4.把 WebDataBinder 的 target 作为参数传递给目标方法的参数。
