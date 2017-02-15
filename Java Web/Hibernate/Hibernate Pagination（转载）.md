# Hibernate Pagination

### **1. Overview**

This article is a quick **introduction to Pagination in Hibernate**. We will look at the standard HQL as well as the *ScrollableResults* API, and finally, at pagination with Hibernate Criteria.



### **2. Pagination with HQL and setFirstResult,setMaxResults API**

The simplest and most common way to do pagination in Hibernate is **using HQL**:

```java
Session session = sessionFactory.openSession();
Query query = sess.createQuery("From Foo");
query.setFirstResult(0);
query.setMaxResults(10);
List<Foo> fooList = fooList = query.list();
```

This example is using a basic *Foo* entity and is very much similar to the JPA with JQL implementation – the only difference being the query language.

If we turn on** logging for Hibernate** , we’ll see the following SQL being run:

```sql
Hibernate: 
    select
        foo0_.id as id1_1_,
        foo0_.name as name2_1_ 
    from
        Foo foo0_ limit ?
```



#### **2.1 The Total Count and the Last Page**

A pagination solution is not complete without knowing **the total number of entities**:

```java
String countQ = "Select count (f.id) from Foo f";
Query countQuery = session.createQuery(countQ);
Long countResults = (Long) countQuery.uniqueResult();
```

And lastly, from the total count and a given page size, we are able to calculate **the last page**:

```java
int pageSize = 10;
int lastPageNumber = (int) ((countResult / pageSize) + 1);
```

At this point we can look at **a more complete example for pagination**, where we are calculating the last page and the retrieving it:

```
@Test
public void givenEntitiesExist_whenRetrievingLastPage_thenCorrectSize() {
    int pageSize = 10;
    String countQ = "Select count (f.id) from Foo f";
    Query countQuery = session.createQuery(countQ);
    Long countResults = (Long) countQuery.uniqueResult();
    int lastPageNumber = (int) ((countResults / pageSize) + 1);
 
    Query selectQuery = session.createQuery("From Foo");
    selectQuery.setFirstResult((lastPageNumber - 1) * pageSize);
    selectQuery.setMaxResults(pageSize);
    List<Foo> lastPage = selectQuery.list();
 
    assertThat(lastPage, hasSize(lessThan(pageSize + 1)));
```



### **3. Pagination with Hibernate using HQL and the ScrollableResults API**

Using *ScrollableResul*ts to implement pagination has the potential to **reduce database calls**. This approach streams the result set as the program scrolls though it, therefore eliminating the need to repeat the query to fill each page:

```java
String hql = "FROM Foo f order by f.name";
Query query = session.createQuery(hql);
int pageSize = 10;
 
ScrollableResults resultScroll = query.scroll(ScrollMode.FORWARD_ONLY);
resultScroll.first();
resultScroll.scroll(0);
List<Foo> fooPage = Lists.newArrayList();
int i = 0;
while (pageSize > i++) {
    fooPage.add((Foo) resultScroll.get(0));
    if (!resultScroll.next())
        break;
}
```

This approach is not only time-efficient (only one database call), but it allows the user to get access to the **total count of the result set without an additional query**:

```java
resultScroll.last();
int totalResults = resultScroll.getRowNumber() + 1;
```

On the other hand, keep in mind that a large result may take up a decent amount of **memory**.



### **4. Pagination with Hibernate using the Criteria API**

Finally, let’s look at **a more flexible solution** – using criteria:

```java
Criteria criteria = session.createCriteria(Foo.class);
criteria.setFirstResult(0);
criteria.setMaxResults(pageSize);
List<Foo> firstPage = criteria.list();
```

Hibernate Criteria query API makes it very simple to also **get the total count** – by using a *Projection* object:

```java
Criteria criteriaCount = session.createCriteria(Foo.class);
criteriaCount.setProjection(Projections.rowCount());
Long count = (Long) criteriaCount.uniqueResult();
```

As you can see, using this API will result in minimally more more verbose code than plain HQL, but **the API is fully type safe and a lot more flexible**.



### reference

[hibernate-pagination](http://www.baeldung.com/hibernate-pagination)