##Spring MVC 学习笔记——文件上传
* Spring MVC 为文件上传提供了直接的支持，这种支持是通 过即插即用的 MultipartResolver 实现的。Spring 用 `Jakarta Commons FileUpload` 技术实现了一个 `MultipartResolver` 实现类：`CommonsMultipartResovler`
* Spring MVC 上下文中默认没有装配 `MultipartResovler`，因此默认情况下不能处理文件的上传工作，如果想使用 Spring 的文件上传功能，需现在上下文中配置 `MultipartResolver`

####配置 MultipartResolver
* defaultEncoding: 必须和用户 JSP 的 pageEncoding 属性一致，以便正确解析表单的内容
* 为了让 `CommonsMultipartResovler` 正确工作，必须先将` Jakarta Commons FileUpload` 及 `Jakarta Commons io`的类包添加到类路径下

配置示例：

	<!-- 配置 MultipartResolver -->
	<bean id="multipartResolver" 
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<property name="defaultEncoding" value="UTF-8"></property>
		<property name="maxUploadSize" value="1024000"></property>
	</bean>

文件上传处方法示例：

	@RequestMapping("/testFileUpLoad")
	public String testFileUpLoad(@RequestParam("desc") String desc,
			@RequestParam("file") MultipartFile file) throws IOException{
		System.out.println("desc: "+ desc);
		System.out.println("OriginalFilename" + file.getOriginalFilename());
		System.out.println("inputStream: " + file.getInputStream());
		return "success";
	}

文件上传表单示例：

	<form action="testFileUpLoad" method="post" enctype="multipart/form-data">
		File:<input type="file" name="file">
		Desc:<input type="text" name="desc">
		<input type="submit" value="submit">
	</form>