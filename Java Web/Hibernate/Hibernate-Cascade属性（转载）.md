## Hibernate 中的 Cascade 属性

> Cascade（级联）属性经常在集合的映射中使用，设置 Cascade 属性可以很方便的由 Hibernate 自动为你管理关联关系中另一方对象的状态（例如：删除），而无需自己手工地编写代码进行对象状态的更新



### Cascade 属性的几种取值

* **none**：默认，关闭级联
* **save-update**：当session通过save(),update(),saveOrUpdate()方法来保存或更新对象时，级联保存所有关联的新建的临时对象，并且级联更新所有关联的游离对象。
* **delete**：级联删除，通过delete()删除当前对象时，会级联删除所有关联的对象。
* **delete-orphan**：删除所有和当前对象解除关联的对象
* **all**：级联删除, 级联更新,但解除父子关系时不会自动删除子对象
* **all-delete-orphan**：在解除父子关系时,自动删除不属于父对象的子对象, 也支持级联删除和级联保存更新



### 示例(one-to-many 关联关系)

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

###### employee.hbm.xml

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



### save-update：

当session通过save(),update(),saveOrUpdate()方法来保存或更新对象时，级联保存所有关联的新建的临时对象，并且级联更新所有关联的游离对象

##### 保存 save

###### 测试代码：

```java
@Test
	public void testCascadeSave(){
		// create department object
		Department department = new Department();
		department.setDeptName("AA");
		
		// create employee object
		Employee employee1 = new Employee();
		employee1.setEmplName("AAA");
		Employee employee2 = new Employee();
		employee2.setEmplName("AAA");
		Employee employee3 = new Employee();
		employee3.setEmplName("AAA");
		
		// define the relationship between department and employee
		employee1.setDepartment(department);
		employee2.setDepartment(department);
		employee3.setDepartment(department);
		department.getEmployees().add(employee1);
		department.getEmployees().add(employee2);
		department.getEmployees().add(employee3);
		
		// save one by one
		session.save(department);
	}
```

```xml
<hibernate-mapping>
    <class name="com.hibernate.one2many.Department" table="DEPARTMENT">
        
      ...
        <set name="employees" table="EMPLOYEE" inverse="true" lazy="true" cascade="save-update">
            <key>
                <column name="DEPT_ID" />
            </key>
            <one-to-many class="com.hibernate.one2many.Employee" />
        </set>
      ...
    </class>
</hibernate-mapping>
```

###### 产生的SQL语句：

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
    insert 
    into
        EMPLOYEE
        (EMPL_NAME, DEPT_ID) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        EMPLOYEE
        (EMPL_NAME, DEPT_ID) 
    values
        (?, ?)
```

以上的代码产生了 3 条 insert 语句，也就是说在执行 `session.save(department);`的时候，级联保存了于 department 对象关联的 3 个 employee 对象

##### 更新 update

###### 测试代码：

```java
	@Test
	public void testUpdate(){
		// fetch department from database
		Department department = (Department) session.get(Department.class, 16);
		
		// create employee object
		Employee employee = new Employee();
		employee.setEmplName("BB");
		
		// define the relationship between department and employee
		employee.setDepartment(department);
		department.getEmployees().add(employee);
		
		// update
		session.update(department);
	}
```

###### 产生的SQL语句：

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

以上的测试代码首先是从数据库中取出一个 department 对象，然后创建一个新的 employee 对象与之关联，然后执行 update 操作，可以看出`session.update(department);`执行后产生的 SQL 语句中有一条 insert 语句，说明进行了级联更新，将关联的 employee 对象保存



### delete：

级联删除，通过delete()删除当前对象时，会级联删除所有关联的对象

###### 测试代码;

```java
	@Test
	public  void testCascadeDelete(){
		Department department = (Department) session.get(Department.class, 17);
		
		session.delete(department);
	}
```

```xml
<hibernate-mapping>
    <class name="com.hibernate.one2many.Department" table="DEPARTMENT">
        
      ...
        <set name="employees" table="EMPLOYEE" inverse="true" lazy="true" cascade="save-update">
            <key>
                <column name="DEPT_ID" />
            </key>
            <one-to-many class="com.hibernate.one2many.Employee" />
        </set>
      ...
    </class>
</hibernate-mapping>
```

###### 产生的SQL语句：

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
    delete 
    from
        EMPLOYEE 
    where
        ID=?
Hibernate: 
    delete 
    from
        EMPLOYEE 
    where
        ID=?
Hibernate: 
    delete 
    from
        EMPLOYEE 
    where
        ID=?
Hibernate: 
    delete 
    from
        EMPLOYEE 
    where
        ID=?
Hibernate: 
    delete 
    from
        DEPARTMENT 
    where
        ID=?
```

以上测试代码，仅仅显式地对 department 进行 delete 操作，在执行了`session.delete(department);`后，进行了级联删除，生成了 5 条 delete 语句，删除了 1 个 department 对象，4 个 employee 对象。



### delete-orphan：

删除所有和当前对象解除关联的对象

###### 测试代码：

```java
	@Test
	public void testDeleteOrphan(){
		// fetch department from database
		Department department = (Department) session.get(Department.class, 16);
		
		// fetch employee from database
		Employee employee1 = (Employee) session.get(Employee.class, 23);
		Employee employee2 = (Employee) session.get(Employee.class, 24);
		
		// remove employee from the set of department
		department.getEmployees().remove(employee1);
		department.getEmployees().remove(employee2);
		
		// update
		session.update(department);
	}
```

```xml
<hibernate-mapping>
    <class name="com.hibernate.one2many.Department" table="DEPARTMENT">
        
      ...
        <set name="employees" table="EMPLOYEE" inverse="true" lazy="true" cascade="delete-orphan">
            <key>
                <column name="DEPT_ID" />
            </key>
            <one-to-many class="com.hibernate.one2many.Employee" />
        </set>
      ...
    </class>
</hibernate-mapping>
```

###### 产生的SQL语句：

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
        employee0_.ID as ID1_1_0_,
        employee0_.EMPL_NAME as EMPL_NAM2_1_0_,
        employee0_.DEPT_ID as DEPT_ID3_1_0_,
        department1_.ID as ID1_0_1_,
        department1_.DEPT_NAME as DEPT_NAM2_0_1_ 
    from
        EMPLOYEE employee0_ 
    left outer join
        DEPARTMENT department1_ 
            on employee0_.DEPT_ID=department1_.ID 
    where
        employee0_.ID=?
Hibernate: 
    select
        employee0_.ID as ID1_1_0_,
        employee0_.EMPL_NAME as EMPL_NAM2_1_0_,
        employee0_.DEPT_ID as DEPT_ID3_1_0_,
        department1_.ID as ID1_0_1_,
        department1_.DEPT_NAME as DEPT_NAM2_0_1_ 
    from
        EMPLOYEE employee0_ 
    left outer join
        DEPARTMENT department1_ 
            on employee0_.DEPT_ID=department1_.ID 
    where
        employee0_.ID=?
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
    delete 
    from
        EMPLOYEE 
    where
        ID=?
Hibernate: 
    delete 
    from
        EMPLOYEE 
    where
        ID=?
```

以上测试代码中 department 对象再于 2 个 employee 对象解除关联关系后，执行 update 操作。可以看出执行`session.update(department);`后共产生 2 条 delete 语句，对解除关联关系的游离对象进行删除。



### Cascade 与 Inverse 的区别

虽然 cascade与 inverse 属性看上去相似，但实际上是不一样的

* inverse 属性用于定义关联关系中由哪一方维护双方之间的关联关系，也就是说有哪一方来 insert 或者 update 外键
* cascade 属性用于定义当对一方对象执行操作（如save、update、delete等）后，与之存在关联关系的另一方对象是否也应该执行相应的操作（如 cascade save、cascade update、cascade delete 等）



### 总结

​	Cascade 属性可以非常方便地对另一边关联的对象的状态进行管理。但是，如果不正确地使用会产生一些比必要的级联操作级联更新)而降低了运行效率，或者是删除(级联删除)了一些不希望被删除的数据



###### 参考网站

[Hibernate – Cascade example (save, update, delete and delete-orphan)](https://www.mkyong.com/hibernate/hibernate-cascade-example-save-update-delete-and-delete-orphan/)

[Different between cascade and inverse](http://www.mkyong.com/hibernate/different-between-cascade-and-inverse/)