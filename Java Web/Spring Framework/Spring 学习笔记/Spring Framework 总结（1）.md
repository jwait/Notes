#Spring Framework总结(一)
***

#Spring的核心
Spring框架的核心包括两个内容： DI 依赖注入， 以及 AOP 面向切面编程

## IOC 容器
* 在 Spring IOC 容器读取 Bean 配置创建 Bean 实例之前, 必须对它进行实例化. 只有在容器实例化后, 才可以从 IOC 容器里获取 Bean 实例并使用.
* Spring 提供了两种类型的 IOC 容器实现.
	* BeanFactory: IOC 容器的基本实现.BeanFactory 是 Spring 框架的基础设施，面向 Spring 本身；
	* ApplicationContext: 提供了更多的高级特性. 是 BeanFactory 的子接口.plicationContext 面向使用 Spring 框架的开发者，几乎所有的应用场合都直接使用 ApplicationContext 而非底层的 BeanFactory
* 无论使用何种方式, 配置文件时相同的.
* ApplicationContext 的主要实现类：
	* ClassPathXmlApplicationContext：从 类路径下加载配置文件
	* FileSystemXmlApplicationContext: 从文件系统中加载配置文件

## Bean 的作用域
* 在 Spring 中, 可以在 <bean> 元素的 scope 属性里设置 Bean 的作用域.
* 默认情况下, Spring 只为每个在 IOC 容器里声明的 Bean 创建唯一一个实例, 整个 IOC 容器范围内都能共享该实例：所有后续的 getBean() 调用和 Bean 引用都将返回这个唯一的 Bean 实例.该作用域被称为 singleton, 它是所有 Bean 的默认作用域.

|类别|说明|
|---|----|
|singleton|在 Spring IOC 容器中仅存在一个 Bean 实例，Bean 以单例的的方式存在|
|prototype|在每次调用getBean（）时返回一个新的实例。|
|request|每次HTTP请求都会创建一个新的Bean，该作用域仅适用于WebApplicationContext 环境|
|session|同一个 HTTP Session 共享一个 Bean，不同的 HTTP Session 使用不同的 Bean。该作用域仅适用于WebApplicationContext 环境|


## Bean 的生命周期
* Spring IOC 容器可以管理 Bean 的生命周期, Spring 允许在 Bean 生命周期的特定点执行定制的任务. 
* Spring IOC 容器对 Bean 的生命周期进行管理的过程:
	1. 通过构造器或工厂方法创建 Bean 实例
	2. 为 Bean 的属性设置值和对其他 Bean 的引用
	3. 调用 Bean 的初始化方法
	4. Bean 可以使用了
	5. 当容器关闭时, 调用 Bean 的销毁方法
* 在 Bean 的声明里设置 init-method 和 destroy-method 属性, 为 Bean 指定初始化和销毁方法.

示例:

	<bean id="car" class="com.ken.spring.lifeCycle.Car"
		init-method="init"
		destroy-method="destroy">
		<property name="brand" value="Audi"></property>
	</bean>



##DI依赖注入
* 通过将普通POPJ类声明为Bean之后，该Bean将交由 Spring 的 IOC 容器进行管理。如果某个类的正常工作需要依赖其他的类来实现，那么将该类交由容器管理后，该类依赖的对象将由IOC容器自动注入，不需要传统的 new 的方式实现，从而起到松耦合的的作用。
* 依赖注入常用的两种方式：通过构造器注入、setter方法注入
* 配置Bean的两种方式：XML配置、注解配置

###XML方式配置

####声明Bean
在beans命名空间中使用`<bean>`节点来声明：
例如：

	<bean id="person" class="com.ken.spring.collections.Person"></bean>

其中 
* id 属性为该 bean 在IOC容器中唯一的标识符，声明的其他bean的 id 不得相同，若 id 没有指定，则Spring自动将类名作为bean的名字。
* class 属性为需要声明为 bean 的对象的全类名。（Spring默认采用反射的方式获得类的实例）



####依赖注入
#####属性注入

* 属性注入即通过 setter 方法注入 Bean的属性值或依赖的对象
* 属性注入使用 `<property>` 元素。

例如，类的写法如下（为需要注入依赖的属性写一个 setter 方法）

	public class car{
    	privare String brand;
    	private double price;
        
        public void setBrand(String brand){
        	this.brand = brand;
        }
        
        publivc void setPrice(double price){
        	this.price = price;
        }
    }

在 XML 中的 bean 元素中配置如下：

	<bean id="car2" class="com.ken.spring.collections.Car">
    	<property name="brand" value="Ford">
        <property name="price" value="300000">
    </bean>

其中：
name属性指定 Bean 的属性名称，value 属性或 <value> 子节点指定属性值

#####构造器注入

* 通过构造方法注入 Bean 属性或依赖的对象，它保证了 Bean 实例在实例化之后就可以使用
* 构造器注入在 `<constructor-arg>` 元素中声明属性

例如，类的写法如下（使用带参数的构造方法）

	public class Car {

		private String brand;
		private double price;
		private int maxspeed;
	
	
		public Car(String brand, double price) {
			super();
			this.brand = brand;
			this.price = price;
		}

		public Car(String brand, int maxspeed) {
			super();
			this.brand = brand;
			this.maxspeed = maxspeed;
		}

		@Override
		public String toString() {
			return "Car [brand=" + brand + ", price=" + price + ", maxspeed=" + 				maxspeed + "]";
		}

		public void setPrice(double price) {
			this.price = price;
		}
	}

在XML文件中的配置如下：

	<bean id="car1" class="com.ken.spring.collections.Car">
		<constructor-arg value="Ford"></constructor-arg>
		<constructor-arg value="250000" type="double"></constructor-arg>
		<constructor-arg value="180" type="int"></constructor-arg>
	</bean>

* `<bean>`的子节点`<constructor-arg>`中指定属性的顺序即为对应构造器中参数的顺序，当同时有多个构造器匹配时，可以使用 `type`属性按类型匹配入参，或者使用`index`属性来按索引匹配。

######注意：

* 使用`<value>`元素或者`value`属性注入的为字面值。
* 基本数据类型及其封装类、String等类型都可以采取字面值注入的方式。
* 若字面值中包含特殊字符，，可以使用`<![CDATA[]]>`把字面值包裹起来。

#####注入对象（即引用其他Bean）

* 在 Bean 的配置文件中，可以通过`<ref>`元素或者`ref`属性为Bean的属性或者构造器参数指定对Bean的引用。
	例如：（person对象中引用了car对象）
        <bean id="person4" class="com.ken.spring.collections.Person">
            <property name="name" value="Jack"></property>
            <property name="age" value="17"></property>
            <property name="cars" ref="cars"></property>
        </bean>
* 也可以在属性（`<property>元素`）或构造器（`<constructor-arg>`元素）里包含Bean的声明，不需要设置任何id或name。这样的Bean称为内部Bean，并且这样的Bean不能够被其他的Bean所引用。
例如：
		<bean id="person" class="com.ken.spring.beans.Person">
			<property name="name" value="Ken"></property>
			<property name="age" value="21"></property>
			<property name="car">
				<bean class="com.ken.spring.beans.Car">
					<property name="price" value="30000"></property>
				</bean>
			</property>
		</bean>

######注意：

* 可以使用专用的`<null/>`元素标签来为Bean的字符串或其他对象类型的属性注入null值。
* 和Struts、Hibernate等框架一样，Spring支持级联属性设置。
* 为了简化XML文件的配置，可以通过使用P命名空间来代替`<property>`元素标签来配置属性。
	例如·：
		<bean id="address" class="com.ken.spring.autowiring.Address"
		p:city="GuangZhou" p:street="No.1"></bean>



####自动装配Bean属性

Spring IOC 容器可以自动装配Bean。需要做的仅仅是在`<bean>` 的autowire属性中指定自动装配的模式。
Spring提供了4中不同的自动装配策略：
* `byName`：把与 Bean 的属性具有相同名字（或者ID）的其他 Bean 自动装配到 Bean 的对应属性当中。如	果没有跟属性的名字相匹配的 Bean ，则该属性不进行装配。
* `byType`：把鱼 Bean 的属性具有相同类型的其他 Bean 自动装配到 Bean 的对应属性当中。如果没有与属	性的类型相匹配的 Bean ，则该属性不被装配。
* `constructor`：把鱼 Bean 的构造器入参具有相同类型的其他 Bean 自动装配到 Bean 的构造器的对应入	参。
* `autodetect`：首先尝试使用 autodetect 进行自动装配。如果失败，再尝试使用byType进行自动装配。

自动装配的缺点：
* 在 Bean 配置文件里设置 autowire 属性进行自动装配将会装配 Bean 的所有属性. 然而, 若只希望装配个	别属性时, autowire 属性就不够灵活了.
* autowire 属性要么根据类型自动装配, 要么根据名称自动装配, 不能两者兼而有之.

######注意：
* 可以混合使用自动装配和显式装配，可以显示地装配来覆盖自动装配。




###注解的配置方式
使用注解配置方式可以最小化 Spring XML 的配置，但要使用注解的配置方式仍然需要使用 XML 文件配置少量代码。

####启用自动检测 Bean
>当在 Spring 配置中增加 `<context：annotation-config>`时，Spring 将允许我们使用注解来指导 Bean的装配。可以消除 Spring 配置中的`<property>`和`<constructor-arg>` 元素，但是仍然需要显式地使用`<bean>`元素来定义 Bean

因而可以使用`<context:annotation-scan>`元素来配置 Spring 自动检测和定义 Bean。

例如:

	<context:component-scan base-package="com.ken.spring.beans.anntation"></context:component-scan>

其中：`base-package`属性指定了`<context:annotation-scan>`元素所要扫描的包及其所有子包，若需要扫描多个包，则可以在属性的之中用空格或`,`分隔。


####自动检测标注 Bean

默认情况下`<context:annotation-scan>`查找使用构造型（stereotype）注解标注的类，这些特殊的注解如下：
* `@Component`： 通用的构造型注解，标识该类为 Spring 组件。
* `@Controller`： 标识将该类定义为 Spring MVC controller。
* `@Repository`： 标识将该类定义为数据仓库。
* `@Service`： 标识将该类定义为服务。
* 使用`@Component`标注的任意自定义注解。

######注意：
对于扫描到的组件, Spring 有默认的命名策略: 使用非限定类名, 第一个字母小写. 也可以在注解中通过 value 属性值标识组件的名称


####过滤组件扫描
通过为`<context:component-scan> `配置`<context:include-filter> `和`<context:exclude-filter> `子元素，我们可以随意调整扫描行为。

* `<context:component-scan> `： 表示要包含的目标类。
* `<context:exclude-filter> `： 表示要排除在外的目标类。


###使用注解装配：
####使用`@Autowired`
* `@Autowired `注解自动装配具有兼容类型的单个 Bean属性
* 构造器, 普通字段(即使是非 public), 一切具有参数的方法都可以应用`@Authwired` 注解
* 默认情况下, 所有使用 `@Authwired` 注解的属性都需要被设置. 当 Spring 找不到匹配的 Bean 装配属性时, 会抛出异常, 若某一属性允许不被设置, 可以设置 `@Authwired` 注解的 `required` 属性为 false
* 默认情况下, 当 IOC 容器里存在多个类型兼容的 Bean 时, 通过类型的自动装配将无法工作. 此时可以在 `@Qualifier` 注解里提供 Bean 的名称. Spring 允许对方法的入参标注 `@Qualifiter` 已指定注入 Bean 的名称
*  `@Authwired` 注解也可以应用在数组类型的属性上, 此时 Spring 将会把所有匹配的 Bean 进行自动装配.
*  `@Authwired` 注解也可以应用在集合属性上, 此时 Spring 读取该集合的类型信息, 然后自动装配所有与之兼容的 Bean.
*  `@Authwired` 注解用在 java.util.Map 上时, 若该 Map 的键值为 String, 那么 Spring 将自动装配与之 Map 值类型兼容的 Bean, 此时 Bean 的名称作为键值.

####使用 `@Resource` 或 `@Inject` 自动装配 Bean
* 这两个注解和 `@Autowired` 注解的功用类似
* `@Resource` 注解要求提供一个 Bean 名称的属性，若该属性为空，则自动采用标注处的变量或方法名作为 Bean 的名称
* `@Inject` 和 `@Autowired` 注解一样也是按类型匹配注入的 Bean， 但没有 reqired 属性



##AOP（Aspect-Oriented Programing，面向切面编程）
###AOP简介
通常在软件中，有些行为对于大多数应用都是通用的。例如日志、安全和事物管理，这些功能的确很重要，但是却与应用对象本身的业务领域关联不大。软件开发中，这些分布在应用多处的功能被称为横切关注点。通常这些横切关注点在概念上是与应用的业务逻辑相分离的（但是往往直接嵌入到应用的业务逻辑当中）。将这些横切关注带点与业务逻辑相分离是面向切面编程所要解决的。

* AOP的主要编程对象是切面（Aspect），而切面模块化横切关注点。可以实现横切关注点与它们所影响的对象之间的解耦。
* 在应用 AOP 编程时, 仍然需要定义公共功能, 但可以明确的定义这个功能在哪里, 以什么方式应用, 并且不必修改受影响的类. 这样一来横切关注点就被模块化到特殊的对象(切面)里.
* AOP的好处：
	* 每个事务逻辑位于一个位置，代码不分离，便于维护和升级
	* 业务模块更加整洁，只包含核心业务代码






###AOP术语
* 切面（Aspect）： 横切关注点（跨越应用程序多个模块的功能）被模块化的特殊对象。通知和切点共同定义了关于切面的所有内容——它是什么，在何时和何处完成其功能。
* 通知（Advice）： 切面必须完成的工作。除了描述描述切面要完成的工作，通知还解决了核何时执行这个工作的问题。
* 连接点（Joinpoint）： 连接点是应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至是修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新功能。
* 切点（Poincut）： 切点是使用明确的类和方法名来指定，或者是利用正则表达式定义匹配的类和方法名来指定的一个或多个连接点。这些连接点通知所需要织入的地方。如果通知定义了切面的“什么”和“何时”，那么切点定义了“何处”。
* 引入（Introduction）： 引入允许我们向现有的类添加新方法和属性。
* 织入（Weaving）： 织入是件切面应用到目标对象来创建代理对象的过程。


###通知的类型
Spring切面支持五种类型的通知
* Before——在方法被调用之前调用通知
* After——在方法完成之后调用通知，无论方法执行是否成功
* After-returning——在方法成功执行之后调用通知
* After-throwing——在方法抛出异常后调用通知
* Around——通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为


###切点表达式
利用方法签名编写 AspectJ 切入点表达式
最典型的切入点表达式时根据方法的签名来匹配各种方法:
* `execution * com.atguigu.spring.ArithmeticCalculator.*(..)`: 匹配 `ArithmeticCalculator` 中声明的所有方法,第一个 * 代表任意修饰符及任意返回值. 第二个 * 代表任意方法. .. 匹配任意数量的参数. 若目标类与接口与该切面在同一个包中, 可以省略包名.
* `execution public * ArithmeticCalculator.*(..)`: 匹配 `ArithmeticCalculator` 接口的所有公有方法.
* `execution public double ArithmeticCalculator.*(..)`: 匹配 `ArithmeticCalculator` 中返回 `double` 类型数值的方法
* `execution public double ArithmeticCalculator.*(double, ..)`: 匹配第一个参数为 `double` 类型的方法, .. 匹配任意数量任意类型的参数
* `execution public double ArithmeticCalculator.*(double, double)`: 匹配参数类型为 `double, double` 类型的方法.

###使用注解声明切面

####在Spring中启用AspectJ注解支持
1. 在 `classpath` 下包含 AspectJ 类库: aopalliance.jar、aspectj.weaver.jar 和 spring-aspects.jar
2. 将 aop Schema 添加到` <beans> `根元素中.
3. 在 Bean 配置文件中定义一个空的 XML 元素 `<aop:aspectj-autoproxy>`.

当 Spring IOC 容器侦测到 Bean 配置文件中的 <aop:aspectj-autoproxy> 元素时, 会自动为与 AspectJ 切面匹配的 Bean 创建代理.

####声明切面
要在 Spring 中声明 AspectJ 切面, 只需要在 IOC 容器中将切面声明为 Bean 实例. 当在 Spring IOC 容器中初始化 AspectJ 切面之后, Spring IOC 容器就会为那些与 AspectJ 切面相匹配的 Bean 创建代理.
* 在 AspectJ 注解中, 切面只是一个带有 `@Aspect` 注解的 Java 类.
* 通知是标注有某种注解的简单的 Java 方法.
* AspectJ 支持 5 种类型的通知注解:
	* `@Before`: 前置通知, 在方法执行之前执行
	* `@After`: 后置通知, 在方法执行之后执行
	* `@AfterRunning`: 返回通知, 在方法返回结果之后执行
	* `@AfterThrowing`: 异常通知, 在方法抛出异常之后
	* `@Around`: 环绕通知, 围绕着方法执行

示例：

    @Component
    @Aspect
    public class LoggingAspect {

        @Pointcut("execution( * com.ken.spring.aop.impl.ArithmeicCalculatorImpl.*(..))")
        public void declareJoinPointExpression(){
        }

        @Before("declareJoinPointExpression()")
        public void beforeMethod(JoinPoint joinpoint){
            String methodName = joinpoint.getSignature().getName();
            List<Object> args = Arrays.asList(joinpoint.getArgs());
            System.out.println("The Method " + methodName + " begins with " + args);
        }

        @After("declareJoinPointExpression()")
        public void afterMethod(JoinPoint joinPoint){
            String methodName = joinPoint.getSignature().getName();
            System.out.println("The method " + methodName + " end");
        }

        @AfterReturning(value="declareJoinPointExpression()",
                returning="result")
        public void afterReturning(JoinPoint joinPoint, int result){
            String methodName = joinPoint.getSignature().getName();
            System.out.println("The method " + methodName + " end with " + result);
        }

        @AfterThrowing(value="declareJoinPointExpression()",
                throwing="e")
        public void afterThrowing(JoinPoint joinPoint, Exception e){
            String methodName = joinPoint.getSignature().getName();
            System.out.println("The method " + methodName + " occurs exception: " + e);
        }
    }


####前置通知
* 使用`@bBefore`注解
* 前置通知:在方法执行之前执行的通知
* 前置通知使用 @Before 注解, 并将切入点表达式的值作为注解值(用于匹配需要执行该通知的方法).

######访问连接点细节：
若通知方法需要访问当前连接点的细节，可以在通知方法中声明一个类型为 `JoinPoint` 的参数，然后就可以访问连接点的细节，如方法名称参数名。

示例：

	@Before("execution( * com.ken.spring.aop.impl.ArithmeicCalculatorImpl.*(..))")
	public void beforeMethod(JoinPoint joinpoint){
		String methodName = joinpoint.getSignature().getName();
		List<Object> args = Arrays.asList(joinpoint.getArgs());
		System.out.println("The Method " + methodName + " begins with " + args);
	}
表示`com.ken.spring.aop.impl.ArithmeicCalculatorImpl`包下，任意类型的任意方法都将执行此通知。

####后置通知
* 使用`@After`注解
* 后置通知是在连接点完成之后执行的, 即连接点返回结果或者抛出异常的时候, 下面的后置通知记录了方法的终止.
* 同样也需要将切入点表达式作为注解值

示例：

	@After("execution( * com.ken.spring.aop.impl.ArithmeicCalculatorImpl.*(..))")
	public void afterMethod(JoinPoint joinPoint){
		String methodName = joinPoint.getSignature().getName();
		System.out.println("The method " + methodName + " end");
	}

####返回通知
* 使用`@After-returning`注解
* 无论连接点是正常返回还是抛出异常, 后置通知都会执行.
* 在返回通知中若需要访问连接点的返回值，需要将将 `returning` 属性添加到 `@AfterReturning` 注解中,然后必须在通知方法的签名中添加一个同名参数. 在运行时, Spring AOP 会通过这个参数传递返回值.

示例：

	@AfterReturning(value="execution( * com.ken.spring.aop.impl.ArithmeicCalculatorImpl.*(..))",returning="result")
	public void afterReturning(JoinPoint joinPoint, int result){
		String methodName = joinPoint.getSignature().getName();
		System.out.println("The method " + methodName + " end with " + result);
	}
示例中注解中声明了需要访问返回值，名称为`result`，因为在方法的入参中带有 `result` 的同名参数。

####异常通知
* 使用 `@After-throwing` 通知
* 只在连接点抛出异常时才执行异常通知
* 若需要访问连接点抛出的异常，将 throwing 属性添加到 `@AfterThrowing` 注解中，同样的在方法的入参中要带有一个同名的参数
* 如果只对某种特殊的异常类型感兴趣, 可以在方法中将参数声明为其他异常的参数类型. 然后通知就只在抛出这个类型及其子类的异常时才被执行.

示例：

	@AfterThrowing(value="com.ken.spring.aop.impl.ArithmeicCalculatorImpl.*(..))",throwing="e")
	public void afterThrowing(JoinPoint joinPoint, Exception e){
		String methodName = joinPoint.getSignature().getName();
		System.out.println("The method " + methodName + " occurs exception: " + e);
	}

####环绕通知
* 使用 `Around` 注解
* 环绕通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为
* 环绕通知是所有通知类型中功能最为强大的, 能够全面地控制连接点. 甚至可以控制是否执行连接点.
* 对于环绕通知来说, 连接点的参数类型必须是 `ProceedingJoinPoint` . 它是 JoinPoint 的子接口, 允许控制何时执行, 是否执行连接点
* 在环绕通知中需要明确调用 `ProceedingJoinPoint` 的 `proceed()` 方法来执行被代理的方法. 如果忘记这样做就会导致通知被执行了, 但目标方法没有被执行.

示例：

	@Around("com.ken.spring.aop.impl.ArithmeicCalculatorImpl.*(..))")
	public Object around(ProceedingJoinPoint joinPoint){
		Object result = null;
		String methodName = joinPoint.getSignature().getName();
		List<Object> args = Arrays.asList(joinPoint.getArgs());
		
		try {
			System.out.println("The Method " + methodName + " begins with " + args);
			result = joinPoint.proceed();
			System.out.println("The method " + methodName + " end with " + result);
		} catch (Throwable e) {
			System.out.println("The method " + methodName + " occurs exception: " + e);
		}
		
		System.out.println("The method" + methodName + " end");
		
		return result;
	}

######注意：
环绕通知的方法需要返回目标方法执行之后的结果, 即调用 `joinPoint.proceed()` 的返回值, 否则会出现空指针异常

####指定切面的优先级
* 在同一个连接点上应用不止一个切面时, 除非明确指定, 否则它们的优先级是不确定的.
* 切面的优先级可以通过实现 Ordered 接口或利用 @Order 注解指定.
* 实现 Ordered 接口, getOrder() 方法的返回值越小, 优先级越高.
* 若使用 @Order 注解, 序号出现在注解中

示例：

	@Aspect
    @Order(0)
    public class CalculatorValidationAspect{
    
    }
    
    @Aspect
    @Order(1)
    public class CalculatorLoggingAspect{
    
    }

####使用 `@PointCut` 注解
在声明通知的过程中，如果某一些切点重复出现，即相同的切点表达式在多个通知方法的注解声明中多次出现，则可以使用 `@PointCut` 注解来简化配置。要注意的是 `@PointCut` 需要注解到一个方法上，这个方法的名称作为在前面的其他通知进行注解时的切点表达式名称，这个方法不需要实现。
示例：

	@Pointcut("execution( * com.ken.spring.aop.impl.ArithmeicCalculatorImpl.*(..))")
	public void declareJoinPointExpression(){
	}
在下例中，`@Before` 注解中使用了 `declareJoinPointExpression()` 名称的切点表达式

    @Before("declareJoinPointExpression()")
	public void beforeMethod(JoinPoint joinpoint){
		String methodName = joinpoint.getSignature().getName();
		List<Object> args = Arrays.asList(joinpoint.getArgs());
		System.out.println("The Method " + methodName + " begins with " + args);
	}

######注意：
若需要使用多条切点表达式，可以通过操作符 &&, ||, ! 结合起来. 


###基于 XML 声明切面
* 当使用 XML 声明切面时, 需要在 `<beans>` 根元素中导入 aop Schema
* 在 Bean 配置文件中, 所有的 Spring AOP 配置都必须定义在 `<aop:config>` 元素内部. 

Spring中的 AOP 配置元素

|AOP配置元素|描述|
|----------|----|
|`<aop：advisor>`|定义 AOP 通知器|
|`<aop:after>`|定义 AOP 后置通知（不管被通知方法是否执行成功）|
|`<aop:after-returing>`|定义 AOP after-returning 通知|
|`<aop:after-throwing>`|定义 AOP 通知|
|`<aop:around>`|定义AOP 环绕通知|
|`<aop:aspect>`|定义切面|
|`<aop:aspectj-autoproxy>`|启用 `@AspectJ`注解驱动的切面|
|`<aop:before>`|定义 AOP 前置通知|
|`<aop:config>`|顶层的 AOP 配置元素。大多数的 `<aop：*>`必须包含在 `<aop:config>` 元素内|
|`<aop:declare-parents>`|为被通知的对象引入额外的接口，并透明地实现|
|`<aop:pointcut>`|定义切点|

示例：

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd">

        <bean id="arithmeticCalculator" 
            class="com.ken.spring.aop.impl.xml.ArithmeicCalculatorImpl"></bean>

        <bean id="loggingAspect" 
            class="com.ken.spring.aop.impl.xml.LoggingAspect"></bean>

        <aop:config>
            <aop:pointcut expression="execution(* com.ken.spring.aop.impl.xml.ArithmeicCalculatorImpl.*(..))" 
                id="pointCut"/>

            <aop:aspect ref="loggingAspect">
                <aop:before method="beforeMethod" pointcut-ref="pointCut"/>
                <aop:after method="afterMethod" pointcut-ref="pointCut"/>
                <aop:after-returning method="afterReturning" pointcut-ref="pointCut" returning="result"/>
                <aop:after-throwing method="afterThrowing" pointcut-ref="pointCut" throwing="e"/>
            </aop:aspect>
        </aop:config>
    </beans>


####声明切面
* 对于每个切面而言, 都要创建一个 `<aop:aspect>` 元素来为具体的切面实现引用后端 Bean 实例.
* 切面 Bean 必须有一个标示符, 供 `<aop:aspect>` 元素引用

示例：

	 <aop:aspect ref="loggingAspect">

####声明切点
* 切入点使用 <aop:pointcut> 元素声明
* 切入点必须定义在 <aop:aspect> 元素下, 或者直接定义在 <aop:config> 元素下.
	* 定义在 <aop:aspect> 元素下: 只对当前切面有效
	* 定义在 <aop:config> 元素下: 对所有切面都有效
* 基于 XML 的 AOP 配置不允许在切入点表达式中用名称引用其他切入点.

示例:

	<aop:pointcut expression="execution(* com.ken.spring.aop.impl.xml.ArithmeicCalculatorImpl.*(..))" id="pointCut"/>


####声明通知
* 在 aop Schema 中, 每种通知类型都对应一个特定的 XML 元素. 
* 通知元素需要使用 `<pointcut-ref>` 来引用切入点, 或用 `<pointcut>` 直接嵌入切入点表达式.  `method` 属性指定切面类中通知方法的名称.
示例：

		<aop:aspect ref="loggingAspect">
			<aop:before method="beforeMethod" pointcut-ref="pointCut"/>
			<aop:after method="afterMethod" pointcut-ref="pointCut"/>
			<aop:after-returning method="afterReturning" pointcut-ref="pointCut" returning="result"/>
			<aop:after-throwing method="afterThrowing" pointcut-ref="pointCut" throwing="e"/>
		</aop:aspect>


