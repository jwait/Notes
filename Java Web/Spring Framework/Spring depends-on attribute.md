# Spring depends-on attribute

In spring application XML, if we need to initialize any bean before any other bean, depends-on do this job. While creating bean we need to define depends-on attribute in bean. 
In the below example I have tried to show the working of *depends-on*. I have taken three bean A,B and C. 
I have implemented as 

B depends on C

A depends on B. 

First A will be initialized, and then B and then C. 
In case we want to achieve the same using annotation, we need to use `@DependsOn`. Find the [link.](http://www.concretepage.com/spring/example_dependson_spring) 
Now find the example for XML based `depends-on` here.



### XML file using depends-on

**app-conf.xml**

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="testA" class="com.concretepage.A"  depends-on="testB"/>
    <bean id="testB" class="com.concretepage.B" depends-on="testC"/>
    <bean id="testC" class="com.concretepage.C" />
</beans> 
```

### Create Beans

**A.java**

```
package com.concretepage;
public class A {
	public A(){
		System.out.println("Bean A is initialized");
	}
} 
```

**B.java**

```
package com.concretepage;
public class B {
	public B(){
		System.out.println("Bean B is initialized");
	}
} 
```

**C.java**

```
package com.concretepage;
public class C {
	public C(){
		System.out.println("Bean C is Initialized.");
	}
} 
```

### Run Application

**SpringTest.java**

```
package com.concretepage;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class SpringTest {
	public static void main(String[] args) {
		 ApplicationContext context = new ClassPathXmlApplicationContext("app-conf.xml");
	}
} 
```

Find the output.

```
Bean C is Initialized.
Bean B is initialized
Bean A is initialized 
```