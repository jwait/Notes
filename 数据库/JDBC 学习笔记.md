# JDBC 学习笔记

参考资料：[The Java Tutorials - JDBC Basics](https://docs.oracle.com/javase/tutorial/jdbc/basics/processingsqlstatements.html)



## 什么是JDBC

JDBC的全称是 Java DataBase Connection，也就是数据库连接，我们可以用它来操作关系型数据库。JDBC接口及相关类在java.sql包和javax.sql包里。我们可以用它来连接数据库，执行SQL查询，存储过程，并处理返回的结果。

JDBC接口让Java程序和JDBC驱动实现了松耦合，使得切换不同的数据库变得更加简单。



## Getting Started

通常，使用JDBC执行SQL语句都会按照以下的步骤进行：

1. 建立连接Connection，得到一个Connection对象

2. 创建语句Statement。该statement对象由Connection对象创建，代表SQL语句。当执行了Statement对象后将会得到一个代表数据库结果集的对象ResultSet

   JDBC中有三中Statement

   * Statement：用于不带参数的简单的SQL语句
   * PreparedStatement：用于可能带有输入参数的预编译SQL语句
   * CallableStatement：用于执行可能带有输入输出参数的存储过程

   ```
   stmt = con.createStatement();
   ```

3. 执行查询。需要执行SQL语句进行查询，仅需要调用Statement对象的execute方法，Statement对象提供以下种execute方法：

   * execute：若查询返回的第一个对象是ResultSet对象则返回true；此方法适用于查询可能辉返回一个或多个ResultSet的情况；若需要获取返回的ResultSet对象只需要重复调用Statement对象的getResultSet()方法
   * executeQuery：只返回一个ResultSet对象
   * executeUpdate：返回SQL语句执行后，数据库受影响的行的数目，例如执行Insert、Update、Delete语句时。

   ```
   ResultSet rs = stmt.executeQuery(query);
   ```

4. 处理ResultSet对象。通过游标可以访问ResultSet对象中的数据，需要注意的是这个游标并不是数据库游标。游标就是一个指向ResultSet对象中某一行数据的指针，初始时，游标指向第一行结果前

   ```
   try {
       stmt = con.createStatement();
       ResultSet rs = stmt.executeQuery(query);
       while (rs.next()) {
           String coffeeName = rs.getString("COF_NAME");
           int supplierID = rs.getInt("SUP_ID");
           float price = rs.getFloat("PRICE");
           int sales = rs.getInt("SALES");
           int total = rs.getInt("TOTAL");
           System.out.println(coffeeName + "\t" + supplierID +
                              "\t" + price + "\t" + sales +
                              "\t" + total);
       }
   }
   // ...
   ```

5. 关闭Connection。当结束使用Statement对象时，马上调用Statement对象的`close()`释放资源。当你关闭Connection对象时，关联的ResultSet对象也会被关闭。关闭Connection关闭的处理可以有两种不同的方式：

   * 使用finish块手动关闭，这样即便出现了异常也可以正常将Connection关闭。

     如：

     ```
     } finally {
         if (stmt != null) { stmt.close(); }
     }
     ```

   * 使用try-with-resource块有系统自动关闭。

     如：

     ```
     try (Statement stmt = con.createStatement()) {
         // ...
     }
     ```





## 建立 Connection 对象

建立Connection时使用的数据源可以是DBMS、文件系统或者其他数据源对应的JDBC。典型的，JDBC应用可以使用以下两种类之一连接到数据源：

* `DriverManager`：This fully implemented class connects an application to a data source, which is specified by a database URL. When this class first attempts to establish a connection, it automatically loads any JDBC 4.0 drivers found within the class path. Note that your application must manually load any JDBC drivers prior to version 4.0.
* `DataSource`：This interface is preferred over `DriverManager` because it allows details about the underlying data source to be transparent to your application. A `DataSource` object's properties are set so that it represents a particular data source.

### 使用 DriverManager 类

通过调用DriverManager类的`getConnection()`方法可以连接到你的DBMS当中，而这个连接用 Connection 对象代表。

示例：

```
public Connection getConnection() throws SQLException {

    Connection conn = null;
    Properties connectionProps = new Properties();
    connectionProps.put("user", this.userName);
    connectionProps.put("password", this.password);

    if (this.dbms.equals("mysql")) {
        conn = DriverManager.getConnection(
                   "jdbc:" + this.dbms + "://" +
                   this.serverName +
                   ":" + this.portNumber + "/",
                   connectionProps);
    } else if (this.dbms.equals("derby")) {
        conn = DriverManager.getConnection(
                   "jdbc:" + this.dbms + ":" +
                   this.dbName +
                   ";create=true",
                   connectionProps);
    }
    System.out.println("Connected to database");
    return conn;
}
```



并且在调用`getConnection()`方法建立连接时需要指定一个数据库URL，这个URL用于你的DBMS JDBC驱动去连接到数据库，这个URL包含了如：从什么地方搜索数据库，连接到那一个数据库，并且配置参数等信息。具体的URL语法格式取决于使用的DBMS。

下面是 MySQL Connector/J 的URL语法格式：

```
jdbc:mysql://[host][,failoverhost...]
    [:port]/[database]
    [?propertyName1][=propertyValue1]
    [&propertyName2][=propertyValue2]...
```

* host:port 表示装载数据库的主机名以及端口号。若没有指定，则默认为127.0.0.1:3306
* database 表示需要连接的数据库名
* failover 表示备份数据库的主机名
* propertyName = propertyValue 表述数据库的配置参数



最后需要注意的是关于`Class.forName()`方法的使用：在JDBC 4.0 之前，获取Connection需要调用`forName`方法初始化JDBC，此方法的参数为Driver的名称，如 MySQL Connector/J 就是 com.mysql.jdbc.Driver。但是在 4.0 版本之后，系统将会在 class path 路径下系统加载



### 使用DataSource类

使用DataSource的有优点是：

* 因为获取 Connection 连接的驱动名和 JDBC 配置到到了DataSource中，应用开发人员不再需要在应用程序硬编码  JDBC 驱动以及 JDBC URL ，只需要从DataSource 中调用 getConnection 方法即可获得 Connection，并且当 DataSource 需要发生变化时，只需要系统管理员对配置进行修改，而无需担心对应用程序造成影响
* 同时使用 DataSource 可以支持 Connection 连接池管理以及分布式事务



## 处理 SQL 异常

当 JDBC 发生内部错误时，其将会抛出一个SQLException的实例。这一个SQLException实例包含了以下信息可以帮助你确定异常的原因：

* description ：对错误的描述，通过调用方法`SQLException.getMessage`获得
* SQLState code：通过调用方法`SQLException.getSQLState`获得
* error code：通过调用方法`SQLException.getErrorCode`获得
* cause：一个SQLException异常可能由于一个或多个Throwable对象的抛出所导致，为了获得错误链，可以递归调用`SQLException.getCause`方法获得


#### 常见的 JDBC 异常

* `java.sql.SQLException`：JDBC异常的基类
* `java.sql.BatchUpdateException`：当批处理操作执行失败的时候可能会抛出这个异常。这取决于具体的JDBC驱动的实现，它也可能直接抛出基类异常java.sql.SQLException
* `java.sql.SQLWarning`：SQL操作出现的警告信息
* `java.sql.DataTruncation`：字段值由于某些非正常原因被截断了（不是因为超过对应字段类型的长度限制）


## 使用 ResultSet对象

### ResultSet接口

ResultSet接口提供了方法用于获取和操纵查询的结果。并且ResultSet对象可以拥有不同的功能和特性，这些特性是：type、concurrency、cursor holdability



#### ResultSet Types

ResultSet对象的类型决定其在两个方面上的功能：游标可以被如何操纵，当底层数据发生变化时如何反映到ResultSet对象中

|          type           |               description                |
| :---------------------: | :--------------------------------------: |
|    TYPE_FORWARD_ONLY    |            结果集不允许滚动，游标只能想前移动             |
| TYPE_SCROLL_INSENSITIVE | 结果集可滚动，游标允许前以及向后移动，但是ResultSet不能感知底层数据的变化 |
|  TYPE_SCROLL_SENSITIVE  |    结果集可滚动，游标可向前和向后移动，并且结果集可以感知底层数据的变化    |

默认的ResultSet 类型是：TYPE_FORWARD_ONLY

#### ResultSet Concurrent

ResultSet的并发性决定了其支持怎样的更新功能

| concurrency levels |     description     |
| :----------------: | :-----------------: |
|  CONCUR_READ_ONLY  |  ResultSet只读，不能被更新  |
|  CONCUR_UPDATABLE  | 允许通过ResultSet接口进行更新 |

默认的ResultSet并发性是：CONCUR_READ_ONLY

#### Cursor Holdability

|    cursor holdability    |               description                |
| :----------------------: | :--------------------------------------: |
| HOLD_CURSORS_OVER_COMMIT | ResultSet cursors are not closed; they are holdable: they are held open when the method commit is called. Holdable cursors might be ideal if your application uses mostly read-only ResultSet objects |
| CLOSE_CURSORS_AT_COMMIT  | ResultSet objects (cursors) are closed when the commit method is called. Closing cursors when this method is called can result in better performance |



#### 设置Statement的类型

为了是Statement执行后返回的ResultSet具有指定的特性，如可滚动、可更新。可以通过在创建Statement的时候指定其ResultSet Types与ResultSet Concurrent类型

示例：

```
 stmt = con.createStatement(ResultSet.TYPE_SCROLL_SENSITIVE,
                   ResultSet.CONCUR_UPDATABLE);
```

### 获取行中的某一列的值

ResultSet接口定义了一系列的getter方法用于获取当前行中某一列的值，如某一列为boolean则调用getBoolean方法，某一列为long类型时，则调用getLong方法。调用getter方法时需要指定需要获取列的列索引值或者是列名、列别名。使用列索引更有效，列索引从1开始编排。获取列的值时从左至右读取，并且每一列只读一次

示例：

```
public static void alternateViewTable(Connection con)
    throws SQLException {

    Statement stmt = null;
    String query =
        "select COF_NAME, SUP_ID, PRICE, " +
        "SALES, TOTAL from COFFEES";

    try {
        stmt = con.createStatement();
        ResultSet rs = stmt.executeQuery(query);
        while (rs.next()) {
            String coffeeName = rs.getString(1);
            int supplierID = rs.getInt(2);
            float price = rs.getFloat(3);
            int sales = rs.getInt(4);
            int total = rs.getInt(5);
            System.out.println(coffeeName + "\t" + supplierID +
                               "\t" + price + "\t" + sales +
                               "\t" + total);
        }
    } catch (SQLException e ) {
        JDBCTutorialUtilities.printSQLException(e);
    } finally {
        if (stmt != null) { stmt.close(); }
    }
}
```



### 使用游标cursor

通过游标可以访问到ResultSet对象中的数据，游标指向ResultSet对象中的一行。当ResultSet对象第一次被创建的时候，游标指向ResultSet对象第一行的前面。另外通过以下提供的一些方法可以移动游标从而允许访问到ResultSet中的不同数据：

* `next`：把游标向后移动一行。如果移动后游标指向的是新的一行则返回true，否则若移动后指向的是ResultSet对象中的最后一行之后则返回false
* `Previous`：把游标向前移动一行。如果移动后游标指向的是新的一行则返回true，否则若移动后游标指向的是ResultSet对象中第一行的前面则返回false
* `last`：将游标移动到ResultSet对象的最后一行。若移动后游标指向的是行则返回true，若ResultSet对象不包含任何行，则返回false
* `beforeFirst`：把游标放到ResultSet对象的最前面，即游标指向第一行的前面。若ResultSet对象不包含任何行，则此操作无作用
* `afterLast`：吧有表放到ResultSet对象的最后面，即游标指向最后一行的后面。若ResultSet对象不包含任何行，则此操作无作用
* `relative(int rows)`：将游标移动到相对于当前行若干行的位置
* `absolute(int row)`：把游标移动到row参数指定的行上



### 对ResultSet对象中的行进行更新

有时候我们希望对返回的ResultSet对象中数据进行更新，但是需要注意的是默认的ResultSet对象是只读的，并且游标仅能够向前移动。为了能够对ResultSet对象中的数据进行更新，ResultSet对象必须至少是可更新的。在设置好ResultSet对象的类型后，我们可以按照以下步骤更新数据：

1. 移动游标使其指向我们需要操作的行
2. 调用row的updatXXX方法（不同的类型调用不同方法，如：updateFloat、updateBoolean）对列中的值进行更新
3. 最后调用row的`updateRow`方法将修改更新到数据库中

示例：

```
public void modifyPrices(float percentage) throws SQLException {

    Statement stmt = null;
    try {
        stmt = con.createStatement();
        stmt = con.createStatement(ResultSet.TYPE_SCROLL_SENSITIVE,
                   ResultSet.CONCUR_UPDATABLE);
        ResultSet uprs = stmt.executeQuery(
            "SELECT * FROM " + dbName + ".COFFEES");

        while (uprs.next()) {
            float f = uprs.getFloat("PRICE");
            uprs.updateFloat( "PRICE", f * percentage);
            uprs.updateRow();
        }

    } catch (SQLException e ) {
        JDBCTutorialUtilities.printSQLException(e);
    } finally {
        if (stmt != null) { stmt.close(); }
    }
}
```



### 使用Statement进行批量更新

JDBC中的Statement不仅仅只能执行一条SQL语句，还可以一次性执行一系列的SQL语句。实际上，Statement、PreparedStatem、CallableStatem对象均支持关联多条SQL语句，这些SQL语句可以是插入、删除、更新等DML语句，也可以是创建表等DDL语句。但是需要注意的是，能够批量执行的语句不允许是可以返回ResultSet对象的语句

使用Statem进行批量更新可以按照以下步骤：

1. 禁用Connection的auto-commit模式

   ```
   this.con.setAutoCommit(false);
   ```

2. 创建Statement对象并调用`addBatch()`方法往其中添加多条需要执行的SQL语句。另外Statement创建后初始为空；若需要清空Statement中已经添加的SQL语句，可以调用`clearBatch`方法

3. 调用`executeBatch()`方法执行批量更新，即执行Statement中的所有SQL语句。执行批量更新后，将返回一个int数组，分别是各个语句执行后数据库中受影响的行数。同样的，当有异常发生时，将返回一个数组，分别是各个SQL语句可能抛出的异常

示例：

```
public void batchUpdate() throws SQLException {

    Statement stmt = null;
    try {
        this.con.setAutoCommit(false);
        stmt = this.con.createStatement();

        stmt.addBatch(
            "INSERT INTO COFFEES " +
            "VALUES('Amaretto', 49, 9.99, 0, 0)");

        stmt.addBatch(
            "INSERT INTO COFFEES " +
            "VALUES('Hazelnut', 49, 9.99, 0, 0)");

        stmt.addBatch(
            "INSERT INTO COFFEES " +
            "VALUES('Amaretto_decaf', 49, " +
            "10.99, 0, 0)");

        stmt.addBatch(
            "INSERT INTO COFFEES " +
            "VALUES('Hazelnut_decaf', 49, " +
            "10.99, 0, 0)");

        int [] updateCounts = stmt.executeBatch();
        this.con.commit();

    } catch(BatchUpdateException b) {
        JDBCTutorialUtilities.printBatchUpdateException(b);
    } catch(SQLException ex) {
        JDBCTutorialUtilities.printSQLException(ex);
    } finally {
        if (stmt != null) { stmt.close(); }
        this.con.setAutoCommit(true);
    }
}
```



### 往ResultSet对象插入新行

在对结果的操作中除了对ResultSet对象中的数据进行更新外，同时也需要往其中插入一条新的数据。同样的由于默认的ResultSet对象是不支持滚动和只读的，在往ResultSet对象中插入新行的时候要求ResultSet对象必须是可更新的（CONCUR_UPDATABLE）并且是可滚动的

可以按照以下步骤往ResultSet对象中插入新行

1. 获得一个可更新以及可滚动的对象
2. 调用ResultSet对象的`moveToInsertRow`方法将游标指向到新插入行。实际上这一个row是将新行插入到ResultSet对象前的一个缓存
3. 调用UpdateXXX方法设置该新行各个列的上的值
4. 调用ResultSet对象的`insertRow()`方法将新插入行的内容插入到ResultSet对象中，并且更新到数据库中

需要注意的是：

* 并不是所有的JDBC均支持通过ResultSet接口插入新的行
* 在调用`insertRow()`方法插入新的行后，应该将游标移动到除 insert row以外的其他行，否则以后应用试图移动游标的时候辉发生不可预料的错误

示例：

```java
public void insertRow(String coffeeName, int supplierID,
                      float price, int sales, int total)
    throws SQLException {

    Statement stmt = null;
    try {
        stmt = con.createStatement(
            ResultSet.TYPE_SCROLL_SENSITIVE
            ResultSet.CONCUR_UPDATABLE);

        ResultSet uprs = stmt.executeQuery(
            "SELECT * FROM " + dbName +
            ".COFFEES");

        uprs.moveToInsertRow();
        uprs.updateString("COF_NAME", coffeeName);
        uprs.updateInt("SUP_ID", supplierID);
        uprs.updateFloat("PRICE", price);
        uprs.updateInt("SALES", sales);
        uprs.updateInt("TOTAL", total);

        uprs.insertRow();
        uprs.beforeFirst();
    } catch (SQLException e ) {
        JDBCTutorialUtilities.printSQLException(e);
    } finally {
        if (stmt != null) { stmt.close(); }
    }
}
```





## 使用preparedStatement

有时候我们需要执行多条类似的SQL语句，但是这些SQL语句只有部分参数不一样，例如：查询一条指定ID的记录。如果使用之前的Statement，那么将需要为每一次查询都编写一条SQL并且创建对应的Statement，这样显得非常的麻烦。

#### PreparedStatement的使用：

因此，JDBC提供了 preparedStatement 使得实现上面提到的要求更加的简单、便捷，并且通过预编译和缓存机制可以提升执行的效率。使用方法也很简单：

1. 创建preparedStatement对象，使用的SQL语句类似，但是在需要参数化的地方使用`?`代替。

   如：

   ```
   String updateString =
       "update " + dbName + ".COFFEES " +
       "set SALES = ? where COF_NAME = ?";
   updateSales = con.prepareStatement(updateString);
   ```

2. 在需要使用preparedStatement前，绑定参数的值。这里调用Statement的setXXX方法对参数进行绑定，在绑定的时候需要指定绑定的是哪一个参数，指定的方式传入参数的索引值。需要注意的是，设置的参数将会一直保持直至对其设置了新的值，或者是调用`clearParameters`方法进行重置

   如：

   ```
   updateSales.setInt(1, e.getValue().intValue());
   updateSales.setString(2, e.getKey());
   ```



示例：

```
public void updateCoffeeSales(HashMap<String, Integer> salesForWeek)
    throws SQLException {

    PreparedStatement updateSales = null;
    PreparedStatement updateTotal = null;

    String updateString =
        "update " + dbName + ".COFFEES " +
        "set SALES = ? where COF_NAME = ?";

    String updateStatement =
        "update " + dbName + ".COFFEES " +
        "set TOTAL = TOTAL + ? " +
        "where COF_NAME = ?";

    try {
        con.setAutoCommit(false);
        updateSales = con.prepareStatement(updateString);
        updateTotal = con.prepareStatement(updateStatement);

        for (Map.Entry<String, Integer> e : salesForWeek.entrySet()) {
            updateSales.setInt(1, e.getValue().intValue());
            updateSales.setString(2, e.getKey());
            updateSales.executeUpdate();
            updateTotal.setInt(1, e.getValue().intValue());
            updateTotal.setString(2, e.getKey());
            updateTotal.executeUpdate();
            con.commit();
        }
    } catch (SQLException e ) {
        JDBCTutorialUtilities.printSQLException(e);
        if (con != null) {
            try {
                System.err.print("Transaction is being rolled back");
                con.rollback();
            } catch(SQLException excep) {
                JDBCTutorialUtilities.printSQLException(excep);
            }
        }
    } finally {
        if (updateSales != null) {
            updateSales.close();
        }
        if (updateTotal != null) {
            updateTotal.close();
        }
        con.setAutoCommit(true);
    }
}
```



#### PreparedStatement的优点：

相比于Statement，其具有以下优点：

- PreparedStatement有助于防止SQL注入，因为它会自动对特殊字符转义。
- PreparedStatement可以用来进行动态查询。
- PreparedStatement执行更快。尤其当你重用它或者使用它的批量查询接口执行多条语句时。
- 使用PreparedStatement的setter方法更容易写出面向对象的代码，而Statement的话，我们得拼接字符串来生成查询语句。如果参数太多了，字符串拼接看起来会非常丑陋并且容易出错。



## 使用事务Transactions

有时候我们并不希望执行了一系列SQL命令后，一部分成功执行了而一部分却失败，我们希望这些SQL命令要么全部执行，要么全部不执行。为了实现这个要求，就可以使用JDBC的事务，只有定义在事务内部的所有SQL命令都成功执行后我们才将事务提交，也即确认对数据库的修改，否则可以选择将事务回滚，恢复到SQL命令执行前的状态。

* 默认情况下，当一个Connection对象被创建后，默认开启auto-commit模式，这意味着每一个SQL Statement将被视为一个事务，并且该事务在SQL Statement 完成后自动提交
* 可以通过connection对象的`setAutoCommit(boolean)`方法设置事务是否自动提交
* 当需要实现自己的事务，即有多条SQL语句需要执行时，需要禁用auto-commit模式，改为手动提交模式

### 使用Transaction

1. 调用connection对象的`setAutoCommit(boolean)`方法禁用auto-commit模式
2. 执行业务中必须全部执行的SQL语句
3. 调用connection的`commit()`方法手动提交事务

示例：

```
public void updateCoffeeSales(HashMap<String, Integer> salesForWeek)
    throws SQLException {

    PreparedStatement updateSales = null;
    PreparedStatement updateTotal = null;

    String updateString =
        "update " + dbName + ".COFFEES " +
        "set SALES = ? where COF_NAME = ?";

    String updateStatement =
        "update " + dbName + ".COFFEES " +
        "set TOTAL = TOTAL + ? " +
        "where COF_NAME = ?";

    try {
        con.setAutoCommit(false);
        updateSales = con.prepareStatement(updateString);
        updateTotal = con.prepareStatement(updateStatement);

        for (Map.Entry<String, Integer> e : salesForWeek.entrySet()) {
            updateSales.setInt(1, e.getValue().intValue());
            updateSales.setString(2, e.getKey());
            updateSales.executeUpdate();
            updateTotal.setInt(1, e.getValue().intValue());
            updateTotal.setString(2, e.getKey());
            updateTotal.executeUpdate();
            con.commit();
        }
    } catch (SQLException e ) {
        JDBCTutorialUtilities.printSQLException(e);
        if (con != null) {
            try {
                System.err.print("Transaction is being rolled back");
                con.rollback();
            } catch(SQLException excep) {
                JDBCTutorialUtilities.printSQLException(excep);
            }
        }
    } finally {
        if (updateSales != null) {
            updateSales.close();
        }
        if (updateTotal != null) {
            updateTotal.close();
        }
        con.setAutoCommit(true);
    }
}
```



### 事务隔离级别（ISOlate Level）

在事务执行过程中可能会出现：脏读、不可重复读以及幻读等现象，导致数据出现错误。为了避免在事务内部出现这样的冲突，DBMS会对读的数据进行加锁，禁止其他事务进行访问。采用不同的封锁策略可以从不同程度上避免以上的异常的发生，采用什么的封锁策略取决于事务的隔离级别

| Isolation Level                | Transactions  | Dirty Reads      | Non-Repeatable Reads | Phantom Reads    |
| ------------------------------ | ------------- | ---------------- | -------------------- | ---------------- |
| `TRANSACTION_NONE`             | Not supported | *Not applicable* | *Not applicable*     | *Not applicable* |
| `TRANSACTION_READ_COMMITTED`   | Supported     | Prevented        | Allowed              | Allowed          |
| `TRANSACTION_READ_UNCOMMITTED` | Supported     | Allowed          | Allowed              | Allowed          |
| `TRANSACTION_REPEATABLE_READ`  | Supported     | Prevented        | Prevented            | Allowed          |
| `TRANSACTION_SERIALIZABLE`     | Supported     | Prevented        | Prevented            | Prevented        |



通常，你并不需要对事务的隔离级别进行任何操作。你可以使用DBMS提供的默认事务隔离级别，默认的隔离级别取决与具体的DBMS。同时JDBC允许你获取当前的事务隔离级别，只需调用connection对象的`getTransactionIsolation`方法，或者也可以调用connection对象的`setTransactionIsolation`方法对事务隔离级别进行设置



### 设置和回滚到 SafePoint

若你希望将当前事务回滚到执行过程中的某一个状态，而不是整个回滚。那么就可以使用SafePoint，通过在事务执行过程中设置safePoint，那么我们就可以将事务回滚到safePoint被设置时的状态

* 在事务内通过调用Connection对象的`setSavePoint()`方法设置safePoint

  如：

  ```
  Savepoint save1 = con.setSavepoint();
  ```

* 若需要回滚，则可以调用conection对象的`rollBack(SafePoint)`方法将事务回滚到某一个状态，该方法需要提供一个SafePoint对象作为参数

  如：

  ```
  con.rollback(save1);
  ```



### 释放SafePoint

* 手动调用Connection对象的`releaseSafePoint`方法可以将指定的safePoint对象从当前事务中移出并释放
* 当事务提交后，在事务内部创建的所有SafePoint将被释放
* 当指定某一个SafePoint对事务进行回滚后，那么在该SafePoint后创建的其他SafePoint将被移出释放



### 什么时候进行回滚

调用`rollback`方法时可以结束当前事务并且将值还原到原来的状态。因而，当你需要执行一条或多条SQL语句并发生异常的时候，调用`rollback`方法时唯一的解决方法，因为调用`rollback`方法可以结束回滚并结束事务，然后重新开始。但是如果仅仅只是捕捉了异常，那么将无法得知语句成功执行，什么语句没有成功执行



## 使用RowSet

* RowSet也是用于存储数据的查询结果

* RowSet接口以及其实现比使用ResultSet对象操作结果集更加的容易和灵活

* RowSet接口派生自ResultSet接口，故其拥有ResultSet接口的全部功能，同时在此基础上添加了两个新的功能，分别是：

  * 其可以作为Java Bean 组件的功能
  * RowSet对象可滚动以及可更新

* RowSet接口拥有以下的子接口

  * JdbcRowSet
  * CachedRowSet
  * WebRowSet
  * JoinRowSet
  * FilteredRowSet

* RowSet对象的分类。根据RowSet对象是否需要长期连接数据库可将其分为两类：connected与disconnected

  * connected：该类型的RowSet对象在其整个生命周期内保持与数据源的连接
  * disconnected：该类型的对象仅仅在读入或者是写出数据的时候才去连接数据库

  在RowSet的所有子接口中，只有JdbcRowSet术语connected，其余均属于disconnected



### JdbcRowSet

* JdbcRowSet接口属于connected类型，故其与ResultSet一样在其生命周期内保持与数据源的连接，但是两者之间的最大区别在与JdbcRowSet拥有Properties和listener notification机制，其可作为Java Bean组件

* JdbcRowSet接口的主要用途是使得ResultSet可滚动以及可更新

* 创建JdbcRowSet对象的几种方式

  * 通过调用以ResultSet作为参数的构造器

    如：

    ```
    stmt = con.createStatement(
               ResultSet.TYPE_SCROLL_SENSITIVE,
               ResultSet.CONCUR_UPDATABLE);
    rs = stmt.executeQuery("select * from COFFEES");
    jdbcRs = new JdbcRowSetImpl(rs);
    ```

  * 通过调用以connection为参数的构造器

    如：

    ```
    jdbcRs = new JdbcRowSetImpl(con);
    jdbcRs.setCommand("select * from COFFEES");
    jdbcRs.execute();
    ```

  * 通过调用默认的构造器，并设置相应的参数

    如：

    ```
    public void createJdbcRowSet(String username, String password) {

        jdbcRs = new JdbcRowSetImpl();
        jdbcRs.setCommand("select * from COFFEES");
        jdbcRs.setUrl("jdbc:myDriver:myAttribute");
        jdbcRs.setUsername(username);
        jdbcRs.setPassword(password);
        jdbcRs.execute();
        // ...
    }
    ```

    使用默认构造器后JdbcRowSet默认的参数如下

    - `type`: `ResultSet.TYPE_SCROLL_INSENSITIVE` (has a scrollable cursor)
    - `concurrency`: `ResultSet.CONCUR_UPDATABLE` (can be updated)
    - `escapeProcessing`: `true` (the driver will do escape processing; when escape processing is enabled, the driver will scan for any escape syntax and translate it into code that the particular database understands)
    - `maxRows`: `0` (no limit on the number of rows)
    - `maxFieldSize`: `0` (no limit on the number of bytes for a column value; applies only to columns that store `BINARY`, `VARBINARY`, `LONGVARBINARY`, `CHAR`, `VARCHAR`, and `LONGVARCHAR` values)
    - `queryTimeout`: `0` (has no time limit for how long it takes to execute a query)
    - `showDeleted`: `false` (deleted rows are not visible)
    - `transactionIsolation`: `Connection.TRANSACTION_READ_COMMITTED` (reads only data that has been committed)
    - `typeMap`: `null` (the type map associated with a `Connection` object used by this `RowSet` object is `null`)

    使用默认构造器创建JdbcRowSet后填充数据

    ```
    jdbcRs.setUrl("jdbc:myDriver:myAttribute");
    jdbcRs.setUsername(username);
    jdbcRs.setPassword(password);

    jdbcRs.setCommand("select * from COFFEES");
    jdbcRs.execute();
    ```

  * 通过RowSetFactory实例创建

    如：

    ```
    public void createJdbcRowSetWithRowSetFactory(
        String username, String password)
        throws SQLException {

        RowSetFactory myRowSetFactory = null;
        JdbcRowSet jdbcRs = null;
        ResultSet rs = null;
        Statement stmt = null;

        try {
            myRowSetFactory = RowSetProvider.newFactory();
            jdbcRs = myRowSetFactory.createJdbcRowSet();

            jdbcRs.setUrl("jdbc:myDriver:myAttribute");
            jdbcRs.setUsername(username);
            jdbcRs.setPassword(password);

            jdbcRs.setCommand("select * from COFFEES");
            jdbcRs.execute();

            // ...
        }
    }
    ```

* 对JdbcRowSet中的数据进行操纵的方法与具有scrollable和updatable属性的ResultSet一样

  * update

    ```
    jdbcRs.absolute(3);
    jdbcRs.updateFloat("PRICE", 10.99f);
    jdbcRs.updateRow();
    ```

  * insert

    ```
    jdbcRs.moveToInsertRow();
    jdbcRs.updateString("COF_NAME", "HouseBlend");
    jdbcRs.updateInt("SUP_ID", 49);
    jdbcRs.updateFloat("PRICE", 7.99f);
    jdbcRs.updateInt("SALES", 0);
    jdbcRs.updateInt("TOTAL", 0);
    jdbcRs.insertRow();

    ```

  * delete

    ```
    jdbcRs.last();
    jdbcRs.deleteRow();
    ```



### CachedRowSet

* CachedRowSet接口是disconnected类型，其并不一直保持与数据源的连接，仅仅在读取数据和写出数据的时候连接数据源

* CachedRowSet接口是WebRowSet、JoinRowSet、FilteredRowSet接口的父接口

* 创建CachedRowSet对象的几种方式：

  * 使用默认构造器

    ```
    CachedRowSet crs = new CachedRowSetImpl();
    ```

    与使用默认构造器创建JdbcRowSet一样，CachedRowSet的各个参数将被赋上默认值，可以调用对应的setter方法进行更改

    使用默认构造器创建的CachedRowSet对象并不包含任何数据，使用前需要先对其填充数据，如下：

    ```
    public void setConnectionProperties(
        String username, String password) {
        crs.setUsername(username);
        crs.setPassword(password);
        crs.setUrl("jdbc:mySubprotocol:mySubname");
        // ...
        crs.setCommand("select * from MERCH_INVENTORY");
        jdbcRs.execute();
    ```

  * 使用RowSetFactory实例进行创建。与JdbcRowSet的创建一样

* CachedRowSet对其数据的操纵方式与JdbcRowSet的一样

  * update

    ```
    while (crs.next()) {
        System.out.println(
            "Found item " + crs.getInt("ITEM_ID") +
            ": " + crs.getString("ITEM_NAME"));
        if (crs.getInt("ITEM_ID") == 1235) {
            int currentQuantity = crs.getInt("QUAN") + 1;
            System.out.println("Updating quantity to " +
              currentQuantity);
            crs.updateInt("QUAN", currentQuantity + 1);
            crs.updateRow();
            // Synchronizing the row
            // back to the DB
            crs.acceptChanges(con);
        }
    ```

  * insert

    ```
    crs.moveToInsertRow();
    crs.updateInt("ITEM_ID", newItemId);
    crs.updateString("ITEM_NAME", "TableCloth");
    crs.updateInt("SUP_ID", 927);
    crs.updateInt("QUAN", 14);
    Calendar timeStamp;
    timeStamp = new GregorianCalendar();
    timeStamp.set(2006, 4, 1);
    crs.updateTimestamp(
        "DATE_VAL",
        new Timestamp(timeStamp.getTimeInMillis()));
    crs.insertRow();
    crs.moveToCurrentRow();
    ```

  * delete

    ```
    while (crs.next()) {
        if (crs.getInt("ITEM_ID") == 12345) {
            crs.deleteRow();
            break;
        }
    }
    ```

* 更新数据源。CachedRowSet对象进行变化与JdbcRowSet对象的进行变化有很大区别，因为JdbcRowSet是connected类型的，故调用了updateRow、insertRow、`deleteRow` 等方法后数据的变化将直接写回到数据库；但是CachedRowSet是disconnected的，执行执行上述几个方法造成的变化并不会影响到数据源中的数据，必须调用`acceptChanges()`方法才能将变化保存到数据源

* CachedRowSet对象包含了一个SyncProvider对象，SyncProvider提供了一个RowSetReader和一个RowSetWriter对象用于读取和下回数据

  * 当调用execute方法时，将通过RowSetReader对象从Connection中读取数据库并填充RowSet

  * 当RowSet中的数据进行更新时，会将任务委托给RowSetWriter完成。但是由于CachedRowSet属于disconnected类型，故其在写回数据的时候可能发生明发冲突

    * 默认的SyncProvider实现是RIOptimisticProvider，其采用了乐观并发策略，即并不设置锁，直接将数据写回到数据库，若没有检测到任何操作则完成，否则不把数据写回
    * 当RIOptimisticProvider调用acceptChanges时发生冲突，将抛出一个SyncProviderException，通过这个异常可以获得一个syncResolver对象。syncResolver对象是一个RowSet对象，其仅包含导致冲突的值，其余列均为空，可以通过其得知哪里发生了冲突，并选择如何处理

    ```
    try {
        crs.acceptChanges();
    } catch (SyncProviderException spe) {
        SyncResolver resolver = spe.getSyncResolver();
    }
    ```



### JoinRowSet

* JoinRowSet允许你在RowSet之间进行SQL的JOIN操作，优点是此操作无需连接到数据库，可以减少连接到数库的开销

* JoinRowSet适用于disConnected类型的RowSet，而connected类型的RowSet由于一直保持与数据源的连接，故没必要使用

* 创建JoinRowSet的几种方式：

  * 使用默认构造器。其使用与CachedRowSet等的使用类似

    ```
    JoinRowSet jrs = new JoinRowSetImpl();
    ```

  * 使用RowSetFactory实例创建

* 往JoinRowSet中添加RowSet对象

  只要可以作为JOIN操作的一部分，任何的RowSet均可以添加到JoinRowSet中。在想JoinRowSet中添加RowSet是需要指定一个列，JOIN操作就是基于这些指定的列进行，这些列称为match column。指定这些列的方式可以是通过列好或者是列名，如：

  ```
  jrs.addRowSet(coffees, 2);

  jrs.addRowSet(coffees, "SUP_ID");
  ```



### FilteredRowSet

* FilteredRowSet允许你对RowSet中的Row进行筛选。同样的，有与FilteredRowSet属于disconnected类型，不需要连接到数据库，可减少连接的开销

* FilteredRowSet是CacheedRowSet、JoinRowSet以及WebRowSet的子接口，拥有以上接口的所有功能

* 创建FilteredRowSet对象的几种方式：

  * 使用默认构造器。使用与其他RowSet类似（如设置参数和填充数据）
  * 通过RowSetFactory实例创建

* FilterRowSet对象功能是用于对数据进行筛选，那么需要设置筛选条件。在这里通过创建和设置predicate对象实现。predicate对象用于定义筛选条件，筛选的条件包括：筛选的列，最大值，最小值，定义的predicate必须实现predicate接口。

  如：

  ```
  StateFilter myStateFilter = new StateFilter(10000, 10999, 1);
  ```

  * 创建了predicate后，调用`setFilter`方法可以将其应用到RowSet中对数据进行筛选
  * 当需要在RowSet上应用多个Filter时，可以创建多个predicate，并多次调用setFilter方法将其应用

* FilterRowSet中对数据的操作与RowSet类似，但是有一点不一样是，数据的改变不允许违背当前设置的Filter条件



### WebRowSet

* WebRowSet对象除恶拥有CachedRowSet的所有功能外，其能够实现与XML之间的相互转换
* WebRoeSet可用于不同Web Service 之间发送数据库数据





## JDBC 的数据类型

* **CLOB**：Character Large OBjects，字符大对象，它是由单字节字符组成的字符串数据，有自己专门的代码页。这种数据类型适用于存储超长的文本信息，那些可能会超出标准的VARCHAR数据类型长度限制（上限是32KB）的文本
* **BLOB**：Binary Larget OBject，它是二进制大对象，由二进制数据组成，没有专门的代码页。它能用于存储超过VARBINARY限制（32KB）的二进制数据。这种数据类型适合存储图片，声音，图形，或者其它业务程序特定的数据