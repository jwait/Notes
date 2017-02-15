##Spring MVC 学习笔记——数据处理
####数据绑定流程
1. Spring MVC 主框架将 `ServletRequest` 对象及目标方法的入参实例传递给 `WebDataBinderFactory` 实例，以创建 `DataBinder` 实例对象
2. `DataBinder` 调用装配在 Spring MVC 上下文中的 `ConversionService` 组件进行数据**类型转换**、**数据格式化**工作。将 `Servlet` 中的请求信息填充到入参对象中
3. 调用 `Validator` 组件对已经绑定了请求消息的入参对象进行数据合法性校验，并最终生成数据绑定结果 `BindingData` 对象
4. Spring MVC 抽取 `BindingResult` 中的入参对象和校验错误对象，将它们赋给处理方法的响应入参

####数据类型转换
* Spring MVC 上下文中内建了很多转换器，可完成大多数 Java 类型的转换工作
* 内建的数据转换器是 `ConversionService`
* 提供的内建数据转换有：
	* java.lang.Boolean -> java.lang.String
	* java.lang.String -> java.lang.Boolean
	* java.lang.Character -> java.lang.Number
	* java.lang.Number -> java.lang.Character
	* java.lang.Number -> java.lang.String
	* java.lang.String -> java.lang.Number
	* java.lang.Character -> java.lang.String
	* java.lang.String -> java.lang.Character
	* java.lang.String -> java.lang.Enum
	* java.lang.Enum -> java.lang.String
	* java.lang.String -> java.util.Locale
	* java.util.Locale -> java.lang.String
	* java.lang.String -> java.util.UUID
	* java.util.UUID -> java.lang.String
	* java.util.Properties -> java.lang.String
	* java.lang.String -> java.util.Properties
	* java.lang.Number -> java.lang.Number

#####自定义类型转换器
* `ConversionService` 是 Spring 类型转换体系的核心接口
* 可以利用 `ConversionServiceFactoryBean` 在 Spring 的 IOC 容器中定义一个 `ConversionService`. Spring 将自动识别出IOC 容器中的 `ConversionService`，并在 Bean 属性配置及Spring MVC 处理方法入参绑定等场合使用它进行数据的转换
* 可通过 `ConversionServiceFactoryBean` 的 `converters` 属性注册自定义的类型转换器

######Spring 支持的转换器
* Spring 定义了 3 种类型的转换器接口，实现任意一个转换器接口都可以作为自定义转换器注册到 `ConversionServiceFactroyBean` 中：
	 * `Converter<S,T>`：将 S 类型对象转为 T 类型对象
	 * `ConverterFactory`：将相同系列多个 “同质” Converter 封装在一起。如果希望将一种类型的对象转换为另一种类型及其子类的对象（例如将 String 转换为 Number 及 Number 子类（Integer、Long、Double 等）对象）可使用该转换器工厂类
	 * `GenericConverter`会根据源类对象及目标类对象所在的宿主类中的上下文信息进行类型转换

######自定义类型转换器示例:
（目标是将`lastName-email-gender-department.id `格式的字符串转换成 `Employee` 对象）
* 控制器：

    @Controller
    public class SpringMVCTest {

        @Autowired
        private EmployeeDao employeeDao;

        @RequestMapping("/testConversionServiceConverter")
        public String testConverter(@RequestParam("employee") Employee employee){
            System.out.println("save :" + employee);
            employeeDao.save(employee);
            return "redirect:/emps";
        }
    }

* 编写自定义的转换器：
（使用的是`Converter<S,T>`类型转换器，需要实现`org.springframework.core.convert.converter.Converter<String, Employee>` 接口）

        @Component
        public class EmployeeConverter implements Converter<String, Employee> {

            @Override
            public Employee convert(String source) {
                if(source != null){
                    String[] vals = source.split("-");

                    if(vals != null && vals.length == 4){
                        String lastName = vals[0];
                        String email = vals[1];
                        Integer gender =Integer.parseInt(vals[2]);
                        Department department = new Department();
                        department.setId(Integer.parseInt(vals[3]));

                        Employee employee = new Employee(null, lastName, email, gender, department);

                        return employee;
                    }
                }
                return null;
            }

        }

* 注册自定义的类型转换器：
需要在 SpringMVC 的配置文件中配置一个 `ConversionServiceFactoryBean`，并通过其 `converts`属性将一个或多个自定义的类型转换器注册，最后需要在 `<mvc:annotation-driven` 标签中将自定义的 `ConversionService` 注册到 SpringMVC 的上下文中。
配置了自定义的类型转换器后，原来 SpringMVC 中内建的类型转换器仍然可以使用

        <mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>

        <!-- 配置 conversionService -->
        <bean id="conversionService"
            class="org.springframework.context.support.ConversionServiceFactoryBean">
            <property name="converters">
                <set>
                    <ref bean="employeeConverter"/>
                </set>
            </property>
        </bean>

####数据格式化
* 对属性对象的输入/输出进行格式化，从其本质上讲依然属于 “类型转换” 的范畴
* Spring 在格式化模块中定义了一个实现 `ConversionService` 接口的 `FormattingConversionService` 实现类，该实现类扩展了 `GenericConversionService`，因此它既具有类型转换的功能，又具有格式化的功能
* `FormattingConversionService` 拥有一个 `FormattingConversionServiceFactroyBean` 工厂类，后者用于在 Spring 上下文中构造前者


* `FormattingConversionServiceFactroyBean` 内部已经注册了 :
	* `NumberFormatAnnotationFormatterFactroy`：支持对数字类型的属性使用 `@NumberFormat` 注解
	* `JodaDateTimeFormatAnnotationFormatterFactroy`：支持对日期类型的属性使用 `@DateTimeFormat` 注解
* 装配了 `FormattingConversionServiceFactroyBean` 后，就可 •以在 Spring MVC 入参绑定及模型数据输出时使用注解驱动了
* `<mvc:annotation-driven/>` 默认创建的ConversionService 实例即为 `FormattingConversionServiceFactroyBean`
* 若需要同时使用数据格式化和自定义的数据转换器，则注册数据转换器是使用`org.springframework.format.support.FormattingConversionServiceFactoryBean`
	示例：
    
    	<bean id="conversionService"
		class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
		<property name="converters">
			<set>
				<ref bean="employeeConverter"/>
			</set>
		</property>
	</bean>

#####日期格式化
* `@DateTimeFormat` 注解可对 `java.util.Date`、`java.util.Calendar`、`java.long.Long` 时间类型进行标注：
	* `pattern` 属性：类型为字符串。指定解析/格式化字段数据的模式，如：”yyyy-MM-dd hh:mm:ss”
	* `iso` 属性：类型为 `DateTimeFormat.ISO`。指定解析/格式化字段数据 –的ISO模式，包括四种：`ISO.NONE`（不使用） 默认、`ISO.DATE(yyyy-MM-dd)` 、`ISO.TIME(hh:mm:ss.SSSZ)`、 `ISO.DATE_TIME(yyyy-MM-dd hh:mm:ss.SSSZ)`
	* `style` 属性：字符串类型。通过样式指定日期时间的格式，由两位字符组成，第一位表示日期的格式，第二位表示时间的格式：S：短日期/时间格式、M：中日期/时间格式、L：长日期/时间格式、F：完整日期/时间格式、-：忽略日期或时间格式

#####数值格式化
* `@NumberFormat` 可对类似数字类型的属性进行标注，它拥有两个互斥的属性:
	* `style`：类型为 NumberFormat.Style。用于指定样式类型，包括三种：Style.NUMBER（正常数字类型）、Style.CURRENCY（货币类型）、 Style.PERCENT（百分数类型）
	* `pattern`：类型为 String，自定义样式， 如patter="#,###"；

###数据校验
####JSR 303
* JSR 303 是 Java 为 Bean 数据合法性校验提供的标准框架，它已经包含在 JavaEE 6.0 中 .
* JSR 303 通过在 Bean 属性上标注类似于` @NotNull`、`@Max` 等标准的注解指定校验规则，并通过标准的验证接口对 Bean 进行验证

|注解|功能说明|
|----|----|
|`@Null`|被注释的元素必须为 null|
|`@NotNUll`|被注释的元素必须不为 null|
|`@AssertTrue`|被注释的元素必须为 true|
|`@AssertFasle`|被注释的元素必须为 false|
|`@Min(value)`|被注释的元素必须是一个数字，其值必须大于或等于指定的最小值|
|`@Max(value)`|被注释的元素必须是一个数字，其值必须小于或等于指定的最大值|
|`@DecimalMin(value)`|被注释的元素必须是一个数字，其值必须大于或等于指定的最小值|
|`@DecimalMax(value)`|被注释的元素必须是一个数字，其值必须小于或等于指定的最大值|
|`@Size(max,min)`|被注释的元素的大小必须在指定的范围内|
|`@Digits(integer,fraction)`被注释的元素必须是一个数字，其值必须在可接受的范围内|
|`@Past`|被注释的元素必须是一个过去的日期|
|`@Futrue`|被注释的元素必须一个将来的日期|
|`@Pattern(value)`|被注释的元素必须符合指定的正则表达式|

####Hibernate Validator 扩展注解
* `Hibernate Validator` 是 JSR 303 的一个参考实现，除支持所有标准的校验注解外，它还支持以下的扩展注解

|注解|功能说明|
|-|-|
|`@email`|被注解的元素必须是电子邮箱|
|`@lenth`|被注释的字符串大小必须在指定的范围内|
|`@NotEmpty`|被注释的字符串必须非空|
|`@Range`|被注释的元素必须在合适的范围内|

####使用数据校验
1. Spring 本身并没有提供 JSR303 的实现，所以必须将JSR303 的实现者的 jar 包放到类路径下。
	即 hibernate validator 验证框架 jar 包
2. 在 Spring MVC 的配置文件中添加 `<mvc:annotation-driven` ，因为`<mvc:annotation-driven`会默认装配好一个`LocalValidatorFactoryBean`。
3. 需要在 Bean 的属性上添加对应的注解
4. 在目标方法 bean 类型的前面添加 `@valid` 注解

######注意：需校验的 Bean 对象和其绑定结果对象或错误对象是成对出现的，它们之间不允许声明其他入参

