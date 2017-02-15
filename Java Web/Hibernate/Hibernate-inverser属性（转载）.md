##Hibernate 中的 inverse 属性

> inverse 属性是 Hibernate 双向的关系映射中常用的属性
> 通常使用在 one-to-many 和 many-to-many 关联关系中
> inverse 属性的取值有： true 以及 false ，默认设置为 false

###inverse 属性的作用
* inverse 属性的设置决定了：关联关系的两端中由哪一端负责维护双方的关系。
* 当 inverse 属性为 true 是表示由该一端维护关联关系，反之， inverse 属性为 false 是表示该一端不负责维护双方关系

###示例(one-to-many 关联关系)
下面是一个简单的一对多关联关系，department 与 employee 的关系，一个 department 中包含零到多个 employee

###### department.java

```java
package com.hibernate.one2many;

import java.util.HashSet;

import java.util.Set;

public class Department {

    private Integer id;

    private String deptName;

    private Set<Employee> employees = new HashSet<>();

   ....

}

```

###### employee.java

```java
package com.hibernate.one2many;

public class Employee {

    private Integer id;

    private String emplName;

    private Department department;

	....

}

```

###### department.hbm.xml

```xml
<hibernate-mapping>
    <class name="com.hibernate.one2many.Department" table="DEPARTMENT">
		....
        
        <set name="employees" table="EMPLOYEE" inverse="false" lazy="true">
            <key>
                <column name="DEPT_ID" />
            </key>
            <one-to-many class="com.hibernate.one2many.Employee" />
        </set>
        
		....
    </class>
</hibernate-mapping>
```

######employee.hbm.xml

```xml
<hibernate-mapping>
    <class name="com.hibernate.one2many.Employee" table="EMPLOYEE">
		....
        
        <many-to-one name="department" class="com.hibernate.one2many.Department" fetch="join">
            <column name="DEPT_ID" />
        </many-to-one>
        ....

    </class>
</hibernate-mapping>
```


##### inverse 属性的取值
######1. inverse="true"
若 set 节点中，inverse="true"，那么意味着 employee 就是这个关联关系的所有者，而 department 将不会更新这个关系

```xml
<hibernate-mapping>
    <class name="com.hibernate.one2many.Department" table="DEPARTMENT">
		....
        
        <set name="employees" table="EMPLOYEE" inverse="true" lazy="true">
```

######2. inverse="false"
若 set 节点中，inverse="false"（默认），那么意味着 department 就是这个关联关系的所有者，而 employee 将不会更新这个关系

```xml
<hibernate-mapping>
    <class name="com.hibernate.one2many.Department" table="DEPARTMENT">
		....
        
        <set name="employees" table="EMPLOYEE" inverse="true" lazy="true">
```

##### example1：inverse="false"时操作
department 负责维护双方的关联关系

######测试代码1(插入操作)：

```java
@Test
public void testNonInverse(){
	// employee object is the owner of relationship
	Department department = new Department();
	department.setDeptName("CC");
	
	Employee employee = new Employee();
	employee.setEmplName("CC");
	
	employee.setDepartment(department);
	department.getEmployees().add(employee);
	
	session.save(department);
	session.save(employee);
}
```

######产生的SQL语句:

```sql
Hibernate: 
    insert 
    into
        DEPARTMENT
        (DEPT_NAME) 
    values
        (?)
Hibernate: 
    insert 
    into
        EMPLOYEE
        (EMPL_NAME, DEPT_ID) 
    values
        (?, ?)
Hibernate: 
    update
        EMPLOYEE 
    set
        DEPT_ID=? 
    where
        ID=?
```

一共产生3条SQL语句，其中两条插入语句，1条更新语句

>需要注意的是：update 语句是用于更新 department 与 employee 的关联关系。这是因为 department 与 employee 的 one-to-many 关联关系是基于外键的，也就是说把 employee 数据表中的 DEPT_ID 字段作为外键，存放了对应 department 的索引(关联关系中一的一端)。此时，department 作为主控方，在插入了两条记录后需要由 department 通过 update 语句设置外键来建立关联。

######测试代码2(更新操作)：

```java
@Test
    public void testUpdateDepartment(){
        Department department = (Department) session.get(Department.class, 8);

        // add a new employee
        Employee employee = new Employee();
        employee.setEmplName("123");
        employee.setDepartment(department);
        department.getEmployees().add(employee);

        // modify
        department.setDeptName("BBB");

        // update the modification
        session.save(employee);
        session.update(department);
    }
```

######产生的SQL语句:

```sql
Hibernate: 
    select
        department0_.ID as ID1_0_0_,
        department0_.DEPT_NAME as DEPT_NAM2_0_0_ 
    from
        DEPARTMENT department0_ 
    where
        department0_.ID=?
Hibernate: 
    select
        employees0_.DEPT_ID as DEPT_ID3_0_0_,
        employees0_.ID as ID1_1_0_,
        employees0_.ID as ID1_1_1_,
        employees0_.EMPL_NAME as EMPL_NAM2_1_1_,
        employees0_.DEPT_ID as DEPT_ID3_1_1_ 
    from
        EMPLOYEE employees0_ 
    where
        employees0_.DEPT_ID=?
Hibernate: 
    insert 
    into
        EMPLOYEE
        (EMPL_NAME, DEPT_ID) 
    values
        (?, ?)
Hibernate: 
    update
        EMPLOYEE 
    set
        DEPT_ID=? 
    where
        ID=?
```

以上更新操作将产生2条语句，其中一条插入语句，1条更新语句用于更新关联关系

##### example2：inverse="true"时的操作
employee负责维护双方的关联关系
######测试代码1(插入操作):

```java
@Test
public void testInverse(){
	// department object is the owner of relationship
	Department department = new Department();
	department.setDeptName("BB");
	
	Employee employee = new Employee();
	employee.setEmplName("BB");
	
	employee.setDepartment(department);
	department.getEmployees().add(employee);
	
	session.save(department);
	session.save(employee);
}
```

######产生的SQL语句;

```sql
Hibernate: 
    insert 
    into
        DEPARTMENT
        (DEPT_NAME) 
    values
        (?)
Hibernate: 
    insert 
    into
        EMPLOYEE
        (EMPL_NAME, DEPT_ID) 
    values
        (?, ?)
```

以上插入操作共产生2条插入语句

######测试代码2(更新操作)：

```java
@Test
public void testUpdateDepartment(){
	Department department = (Department) session.get(Department.class, 8);
	
	// add a new employee
	Employee employee = new Employee();
	employee.setEmplName("123");
	employee.setDepartment(department);
	department.getEmployees().add(employee);
	
	// modify
	department.setDeptName("BBB");
	
	// update the modification
	session.save(employee);
	session.update(department);
}
```

######产生的SQL语句:

```sql
Hibernate: 
    select
        department0_.ID as ID1_0_0_,
        department0_.DEPT_NAME as DEPT_NAM2_0_0_ 
    from
        DEPARTMENT department0_ 
    where
        department0_.ID=?
Hibernate: 
    select
        employees0_.DEPT_ID as DEPT_ID3_0_0_,
        employees0_.ID as ID1_1_0_,
        employees0_.ID as ID1_1_1_,
        employees0_.EMPL_NAME as EMPL_NAM2_1_1_,
        employees0_.DEPT_ID as DEPT_ID3_1_1_ 
    from
        EMPLOYEE employees0_ 
    where
        employees0_.DEPT_ID=?
Hibernate: 
    insert 
    into
        EMPLOYEE
        (EMPL_NAME, DEPT_ID) 
    values
        (?, ?)
```

以上更新操作产生1条插入语句

###总结
* 正确地设置 inverse 属性可以优化 hibernate 代码，帮助避免产生不必要的更新语句,例如 inverse="true" 时的插入与更新操作不产生 update 语句。
* 最后需要记住的是，inverse="true"意味者该一端为关联关系的拥有者，负责更新关系。而一般inverse="true" 设置在多的一端

######参考网址
[https://www.mkyong.com/hibernate/inverse-true-example-and-explanation/](http://https://www.mkyong.com/hibernate/inverse-true-example-and-explanation/)
[http://blog.csdn.net/johnnywww/article/details/8575662](http://http://blog.csdn.net/johnnywww/article/details/8575662)