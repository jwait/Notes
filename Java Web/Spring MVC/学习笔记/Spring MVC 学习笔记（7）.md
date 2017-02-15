##Spring MVC 学习笔记——使用JSON
###处理JSON
######第一步：加入 JAR 包
例如，若使用的是 Jackson，则导入以下的 jar bao
* jackson-annotations-2.7.0.jar
* jackson-core-2.7.5.jar
* jackson-databind-2.7.0.jar

######第二步：编写目标方法，使其返回 JSON 对应的对象或集合
######第三步：在方法上添加 `@ResponseBody` 注解
示例:

	@ResponseBody
	@RequestMapping("/testJson")
	public Collection<Employee> testJson(){
		return employeeDao.getAll();
	}


###HttpMessageConverter<T>
* HttpMessageConverter<T> 是 Spring3.0 新添加的一个接口，负责将请求信息转换为一个对象（类型为 T），将对象（类型为 T）输出为响应信息
* HttpMessageConverter<T>接口定义的方法：
	* `Boolean canRead(Class<?> clazz,MediaType mediaType)`: 指定转换器可以读取的对象类型，即转换器是否可将请求信息转换为 clazz 类型的对象，同时指定支持 MIME 类型(text/html,applaiction/json等)
	* `Boolean canWrite(Class<?> clazz,MediaType mediaType)`:指定转换器是否可将 clazz 类型的对象写到响应流中，响应流支持的媒体类型在MediaType 中定义。
	* `LIst<MediaType> getSupportMediaTypes()`：该转换器支持的媒体类
型。
	* `T read(Class<? extends T> clazz,HttpInputMessage inputMessage)`：将请求信息流转换为 T 类型的对象。
	* `void write(T t,MediaType contnetType,HttpOutputMessgae outputMessage)`:将T类型的对象写到响应流中，同时指定相应的媒体类型为 contentType。
* `DispatcherServlet` 默认装配 `RequestMappingHandlerAdapter` ，而`RequestMappingHandlerAdapter` 默认装配如下 `HttpMessageConverter`：
	* ByteArrayHttpMessageConverter
	* StringHttpMessageConverter
	* ResourceHttpMessageConverter
	* SourceHttpMessageConverter
	* AllEncompassingFormHttpMessageConverter
	* Jaxb2RootElementHttpMessageConverter


####工作流程
![image](http://static.oschina.net/uploads/space/2013/1026/091627_zgNV_118997.png)


####使用 `HttpMessageConverter<T>`
* 使用 `HttpMessageConverter<T>` 将请求信息转化并绑定到处理方法的入参中或将响应结果转为对应类型的响应信息，Spring 提供了两种途径：
	 * 使用 `@RequestBody` / `@ResponseBody` 对处理方法进行标注
	 * 使用 `HttpEntity<T>` / `ResponseEntity<T>` 作为处理方法的入参或返回值
* 当控制器处理方法使用到 `@RequestBody`/`@ResponseBody` 或 `HttpEntity<T>` / `ResponseEntity<T>`时, Spring 首先根据请求头或响应头的 Accept 属性选择匹配的`HttpMessageConverter`, 进而根据参数类型或泛型类型的过滤得到匹配的 `HttpMessageConverter`, 若找不到可用的 `HttpMessageConverter` 将报错
* `@RequestBody` 和 `@ResponseBody` 不需要成对出现
