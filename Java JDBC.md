# 参考

[Java API doc](https://docs.oracle.com/javase/8/docs/api/)

[Java JDBC API document](https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/index.html)

# Java JDBC API

Java Database Connectivity（JDBC）API 为 Java 编程语言提供了一种通用的数据访问方式，开发者可以利用它实现访问任何数据源，包括关系型数据库、电子表格到文件。JDBC 还提供了一些通用的接口，利用它们可以构建一些工具或者通过继承接口来扩展它。

JDBC API 包含两个 package：

- `java.sql`；
- `javax.sql`；

如果要利用 JDBC API 实现 Java 程序和特定的数据库管理系统结合使用，就需要一个基于 JDBC 技术的驱动，这样 Java 程序才可以通过 JDBC API 操作数据库。处于多方面考虑，驱动可以用纯 Java 程序编写，也可以将 Java 程序和 Java Native Interface（JNI）混合使用。

下面看一下上面两个包的描述：

## java.sql

[java.sql](https://docs.oracle.com/javase/8/docs/api/java/sql/package-summary.html#package.description)

这个包提供了一些 API 可以让 Java 程序访问和处理存储在数据源中的数据（通常是关系型数据库）。此外这个 API 还提供了一种 framework，可以在运行时动态的查找并安装不同的驱动程序来访问不同的数据源（SPI 机制）。尽管 JDBC API 主要用于向数据库传输 sql 语句，但是它也提供了从任何表格形式的数据源读取和写入数据的功能，这种读/写能力是通过 `javax.sql.RowSet` 提供的一组接口实现了，开发者可以通过定制这些接口来实现从电子表格、文件或其他任何表格数据源中读写数据。

java.sql 主要包含以下 API：

- 通过 `DriverManager` 工具创建和 database 之间的 connection：
  - `DriverManager` 类：通过 driver 创建 connection；
  - `SQLPermission` 类：当 Java 程序中存在 SecurityManager 时，可以结合该类提供的 permission 尝试通过 DriverManager 设置 logging stream；
  - `Driver` 接口：提供了基于 JDBC 技术用于注册 driver 和创建 connection 的 API；通常只会被 DriverManager 使用；
  - `DriverPropertyInfo` 类：为 JDBC driver 的属性提供抽象，一般用户不会使用到它；
- 向 database 发送 sql 语句：
  - `Statement` 接口：用于发送 basic sql 语句；
  - `PreparedStatement`  接口：扩展了 Statement 接口，提供发送 prepared sql 语句的能力；
  - `CallableStatement` 接口：扩展了 PreparedStatement 接口，提供了执行数据库 stored procedures 的能力；
  - `Connection` 接口：connection 其实就是和数据库的一次会话，提供了执行 sql 语句、管理 connections 以及其属性的方法；
  - `Savepoint`：为 transaction 提供 savepoints；
- 检索和更新执行一次查询 sql 的结果；
  - `ResultSet` 接口；
- Java 语言提供了一些类和接口和 SQL types 一一对应：
  - `Array` 接口：对应 SQL ARRAY；
  - `Blob` 接口：对应 SQL BLOB；
  - `Clob` 接口：对应 SQL CLOB；
  - `Date` 类：对应 SQL DATE；
  - `NClob` 接口：对应 SQL NCLOB；
  - `Ref` 接口：对应 SQL REF；
  - `RowId` 接口：对应 SQL ROWID；
  - `Struct` 接口：对应 SQL STRUCT；
  - `SQLXML` 接口：对应 SQL XML；
  - `Time` 类：对应 SQL TIME；
  - `Timestamp` 类：对应 SQL TIMESTAMP；
  - `Types` 类：提供了 SQL types 的常量，相当于 type code；
- 如果自定义了一种 SQL 类型（user-defined type，UDT），Java 中也有对应的 class：
  - `SQLDate` 接口：和特定 UDT 类型对应；
  - `SQLInput` 接口：提供了从流中读取 UDT 属性的方法；
  - `SQLOutput` 接口：提供将 UDT 属性写回流的方法；
- Metadata：
  - `DatabaseMetaData` 接口：提供有关数据库的信息；
  - `ResultSetMetaData` 接口：提供关于 ResultSet 对象的列的信息；
  - `ParameterMetaData` 接口：提供有关 PreparedStatement 命令参数的信息；
- Exceptions：
  - `SQLException`：当访问数据出现问题时，大多数方法都会抛出，而由于其他原因，一些方法也会抛出；
  - `SQLWarning`：抛出表示警告；
  - `DataTruncation`：抛出表示数据可能已被截断；
  - `BatchUpdateException`：抛出，表示批处理更新中并非所有命令都成功执行；



## javax.sql

[javax.sql](https://docs.oracle.com/javase/8/docs/api/javax/sql/package-summary.html#package.description)

这个包为 Java 程序访问和处理 server side data source 提供了一些 API。javax.sql 包从 Java 1.4 发行版本开始作为 java.sql 包的补充，包含在 Java SE 和 Java EE 中。

javax.sql 包提供了以下补充：

- `DataSource` 接口作为 DriverManager 的可选方案，可以创建和 data source 的 connection；
- Connection pooling and Statement pooling，连接池和语句池；
- Distributed transactions，分布式事务；
- Rowsets，结果集；

应用程序可以直接使用 DataSource 和 RowSet API，但是 connection pooling 和 distributed transaction API 在内部中间层使用。

> 使用 DataSource 对象来建立 Connection

javax.sql 包提供了一种更好的建立和数据源之间的 connection 的方式。之前通过 DriverManager 类的机制限制仍然有效。只不过更推荐使用 DataSource 来创建 connection，因为这种方式提供了一些增强特性。

下面列出用 DataSource 的好处：

- 可以对数据源的属性进行更改，这意味着当数据源或驱动程序的某些内容发生更改时，不需要对应用程序代码进行更改；
- 连接池、语句池以及分布式事务可以通过 DataSource 对象和程序使用的某些中间层一起使用。之前通过 DriverManager 创建的 Connection 是不具备池化特性的，且不支持分布式事务；

现在数据库供应商需要提供对应的 Driver 且包含 DataSource 的实现。一个特定的 DataSource 实现代表一种物理数据库，由它创建的 Connection 连接的也是一个物理数据源。

对于一个数据源来讲，通过 Java 的 JNDI（Java Naming and Directory Interface）API 注册逻辑名称，应用程序可以通过查找已注册的逻辑名称来检索所需的 DataSource 对象，然后应用程序就可以通过这个 DataSource 对象去创建和物理数据源之间的 connection。

DataSource 对象可以实现为与中间层基础结构一起工作，这样它创建的 connections 将被池化以供重用。使用这种 DataSource 实现的应用程序可以通过池子中的 connection 对象来操作数据库。还可以实现 DataSource 对象以使用中间层基础结构，这样它产生的连接就可以用于分布式事务，而无需任何特殊编码。

> Connection Pooling and Statement Pooling

如果 DataSource 的实现支持池化技术，使用它就可以提高程序的性能，因为创建新的 Connection 对象代价是昂贵的，而复用已有的连接对象则更加高效。

同时 Connection pooling 对于应用程序而言是透明的，程序不会感知到连接池的状态，因为它们和中间层一起工作，程序不需要通过代码操作它，只需要调用 `DataSource.getConnection` 方法就可以获取池子中的连接对象，然后就可以通过 Connection 对象和数据库交互。

connection pooling 使用的类和接口如下所示：

- `ConnectionPoolDataSource` 接口；
- `PooledConnection` 接口；
- `ConnectionEvent` 类；
- `ConnectionEventListener` 接口；
- `StatementEvent` 类；
- `StatementEventListener` 接口；

连接池管理器是三层体系结构的中间层工具，它在幕后使用这些类和接口。当 `ConnectionPoolDataSource` 对象创建 `PooledConnection` 对象时，连接池管理器将会为这个 `PooledConnection` 对象注册一个 `ConnectionEventListener`。当该连接对象 closed 或者出现 error，连接池管理者（作为一个监听者）将会受到通知获取一个 `ConnectionEvent` 对象。

> Distributed Transactions

和 pooled connections 一样，通过与中间层一起工作的 DataSource 创建的 connections 可以参于到分布式事务中。它允许应用程序可以在多个服务器上执行一个事务。

分布式事务使用的接口和类有：

- `XADataSource` 接口；
- `XAConnection` 接口；

由事务管理器（transaction manager）使用这些接口，应用程序无法直接使用它们。

`XAConnection` 继承了 `PooledConnection`，因此分布式事务的 connection 也支持池化技术。中间层中的事务管理器透明地处理所有事情。应用程序代码中唯一的变化是应用程序不能做任何会干扰事务管理器处理事务的事情。具体来说，应用程序不能调用 `Connection.commit` 或 `Connection.rollback`，也不能连接设置为自动提交模式（也就是说，它不能调用 `connection . setautocommit (true)`）。

`XADataSource` 创建 `XAConnection`，`XAConnection` 又可以创建 `XAResource` 对象，事务管理器可以通过 `XAResource` 对象来管理 connection。

注：Java 的分布式事务 API 概念还不够完善，具体实现要看第三方分布式事务框架。

> Rowsets

`RowSet` 接口和其他类或接口一起使用，主要可以分为以下三类：

（1）Event Notification（事件通知）

- `RowSetListener`：
  - RowSet 对象也属于 Java Bean 机制的一部分，因为它符合 Java Bean 的定义，且参于 JavaBeans 的事件通知机制（event notification mechanism），也提供的有 `RowSetListener`，可以通过调用 `RowSet.addRowSetLinstener` 方法来绑定；
  - 当 RowSet 对象发生变化的时候（rows 中某一个发生变化、或者所有 row 都发生变化、或者移除了某个 cursor），就会通知所有注册的监听者；
- `RowSetEvent`
  - 它作为内部通知处理机制的一部分，RowSet 对象创建 RowSetEvent 实例传给它的监听者，监听者可以通过 RowSetEvent 对象来获取具体发生的事件。

（2）Metadata

- `RowSetMetadata` 此接口继承了 `ResultSetMetaData`，包含和 RowSet 对象关联的信息，应用程序可以通过该元数据对象获取 rowset 包含了多少列，每一列包含的数据；

（3）The Reader/Writer Facility

如果一个 RowSet 对象实现了 `RowSetInternal` 接口，那么它就可以调用和它关联的 `RowSetReader` 对象来操作数据，也可以调用关联的 `RowSetWriter` 对象将 rows 的变化回写给数据源。保持与数据源连接的 RowSet 不需要使用读取器和写入器，因为它可以直接对数据源进行操作。

- `RowSetInternal`

实现了该接口的 RowSet 对象可以访问它的内部的状态，且可以调用对应的 reader 和 writer；

- `RowSetReader`

A disconnected `RowSet` object that has implemented the `RowSetInternal` interface can call on its reader (the `RowSetReader` object associated with it) to populate it with data. When an application calls the `RowSet.execute` method, that method calls on the rowset's reader to do much of the work. Implementations can vary widely, but generally a reader makes a connection to the data source, reads data from the data source and populates the rowset with it, and closes the connection. A reader may also update the `RowSetMetaData` object for its rowset. The rowset's internal state is also updated, either by the reader or directly by the method `RowSet.execute`.

- `RowSetWriter`

A disconnected `RowSet` object that has implemented the `RowSetInternal` interface can call on its writer (the `RowSetWriter` object associated with it) to write changes back to the underlying data source. Implementations may vary widely, but generally, a writer will do the following:

- Make a connection to the data source
- Check to see whether there is a conflict, that is, whether a value that has been changed in the rowset has also been changed in the data source
- Write the new values to the data source if there is no conflict
- Close the connection

## JDBC 4.2 API

Java SE 的 JDBC 是有版本的，JDK 8 中 JDBC 的版本是 4.2，相关 API 在两个包中：

- `java.sql` 包中提供了JDBC 的 core API；
- `javax.sql` 包提供 JDBC 的 Optional API；

准确来说完整的 JDBC API 是在 JDK SE 7 中提出的，`javax.sql` 则是将 JDBC API 的功能从 client api 扩展到 server api，它是属于 Java EE 的重要组成部分。

JDBC 4.2 API 集成了以前版本的 JDBC API 版本：

- The JDBC 4.1 API
- The JDBC 4.0 API
- The JDBC 3.0 API
- The JDBC 2.1 core API
- The JDBC 2.0 Optional Package API
  (Note that the JDBC 2.1 core API and the JDBC 2.0 Optional Package API together are referred to as the JDBC 2.0 API.)
- The JDBC 1.2 API
- The JDBC 1.0 API

需要注意的是：有很多新特性是可选的，因此开发者选择的数据库驱动可能有一些改动，在使用特定的驱动版本的时候，最好查询驱动的文档看看它是否支持某些新特性。

# Getting Started

学习环境如下：

- 基于 Docker 创建 MySQL 容器；
- MySQL 版本为 8 +；
- Java 项目构建工具为 Maven；
- JDK 8 +；

Docker image：`mysql:8.0.32-oracle`

Maven 依赖：

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.32</version>
</dependency>

<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
```



# 使用 JDBC 处理 SQL 语句

通常来说使用 JDBC 处理 SQL 语句只需要按照如下步骤：

（1）创建 connection；

（2）创建 statement；

（3）利用 statement 执行 sql 语句；

（4）处理 ResultSet 结果集对象；

（5）关闭 connection；

工具类：

```java
package io.naivekyo.util;

import io.naivekyo.annotation.TableColumn;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;

/**
 * @author NaiveKyo
 * @version 1.0
 * @since 2023/4/2 17:32
 */
public class JDBCUtils {
    
    private static final String USERNAME = "root";
    
    private static final String PASSWORD = "123456";
    
    private static final String URL = "jdbc:mysql://192.168.154.2:3306/my_test?useSSL=false&useUnicode=true&characterEncoding=UTF-8&serverTimeZone=Asia/Shanghai";
    
    private static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USERNAME, PASSWORD);
    }

    /**
     * <p>执行插入语句</p>
     * @param sql   预处理的 sql 语句
     * @param params    要填充的数据
     * @return 受影响的行数
     */
    public static int insert(String sql, Object... params) {
        if (params == null || params.length == 0)
            throw new IllegalArgumentException("insert sql must have at least one parameter.");
        Connection connection = null;
        PreparedStatement statement = null;
        try {
            connection = getConnection();
            statement = connection.prepareStatement(sql);
            pretreatmentSQL(statement, params);
            return statement.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            releaseResource(connection, statement, null);
        }
        
        return -1;
    }

    /**
     * <p>sql 语句根据条件查询一条数据</p>
     * @param sql   sql 查询语句
     * @param clz   实体类的 Class 对象
     * @param params    填充的查询数据
     * @param <T>   数据库实体
     * @return  实体类对象
     */
    public static <T> T getOne(String sql, Class<T> clz, Object... params) {
        if (params == null || params.length == 0)
            throw new IllegalArgumentException("condition query sql must have at least one parameter.");
        Connection connection = null;
        PreparedStatement statement = null;
        ResultSet rs = null;
        T t = null;
        try {
            connection = getConnection();
            statement = connection.prepareStatement(sql);
            pretreatmentSQL(statement, params);
            rs = statement.executeQuery();
            while (rs.next()) {
                t = useReflectGetEntity(rs, clz);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            releaseResource(connection, statement, rs);
        }

        return t;
    }

    private static void pretreatmentSQL(PreparedStatement statement, Object[] params) throws SQLException {
        for (int i = 0; i < params.length; i++) {
            statement.setObject(i + 1, params[i]);
        }
    }

    private static <T> T useReflectGetEntity(ResultSet rs, Class<T> clz) throws Exception {
        Field[] fields = clz.getDeclaredFields();
        Constructor<T> constructor = clz.getConstructor();
        constructor.setAccessible(true);
        T obj = constructor.newInstance();
        
        for (Field f : fields) {
            TableColumn anno = f.getAnnotation(TableColumn.class);
            if (anno != null) {
                f.setAccessible(true);
                String columnName = anno.value();
                Object property = rs.getObject(columnName);
                f.set(obj, property);
            }
        }
        
        return obj;
    }

    /**
     * <p>执行查询 sql 获取数据库实体集合</p>
     * @param sql   sql 查询语句
     * @param clz   Java 实体类 Class 对象
     * @param params    查询参数
     * @param <T>   Java 实体类
     * @return  实体类对象集合
     */
    public static <T> List<T> getEntityList(String sql, Class<T> clz, Object... params) {
        if (params == null || params.length == 0)
            throw new IllegalArgumentException("condition query sql must have at least one parameter.");
        Connection connection = null;
        PreparedStatement statement = null;
        ResultSet rs = null;
        List<T> list = null;
        try {
            connection = getConnection();
            statement = connection.prepareStatement(sql);
            pretreatmentSQL(statement, params);
            rs = statement.executeQuery();
            list = new ArrayList<>();
            while (rs.next()) {
                list.add(useReflectGetEntity(rs, clz));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            releaseResource(connection, statement, rs);
        }
        
        return list;
    }
    
    public static int update(String sql, Object... params) {
        if (!sql.contains("where"))
            throw new IllegalArgumentException("illegal update sql statement. sql must contain 'where' clause!");
        if (params == null || params.length == 0)
            throw new IllegalArgumentException("condition update sql must have at least one parameter.");
        Connection connection = null;
        PreparedStatement statement = null;
        try {
            connection = getConnection();
            statement = connection.prepareStatement(sql);
            pretreatmentSQL(statement, params);
            return statement.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            releaseResource(connection, statement, null);
        }
        
        return -1;
    }

    private static void releaseResource(Connection connection, Statement statement, ResultSet resultSet) {
        try {
            if (connection != null)
                connection.close();
            if (statement != null)
                statement.close();
            if (resultSet != null)
                resultSet.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

字段映射注解：

```jav
/**
 * <p>数据库实体与 Java 实体类的字段名称映射</p>
 * <p>标注在对象属性上用于声明当前属性对应的数据库实体字段</p>
 * @author NaiveKyo
 * @version 1.0
 * @since 2023/4/9 21:49
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface TableColumn {
    
    String value();
    
}
```



## 建立 connection

首先你需要和要使用的 data source 建立一个 connection。通常来说一个应用要和数据源建立连接会使用下面两个类中的一个：

- `DriverManager`：这个类可以在应用程序与数据源之间建立 connection，数据源由 database URL 指定。当使用它建立 Connection 时，它会自动在类路径下寻找任何 JDBC 4.0 驱动。而对于 JDBC 4.0 之前的驱动则需要手动加载；
- `DataSource`：这个接口要比 DriverManager 类更好一些，它将底层数据的详细信息和应用程序分开，不同的 DataSource 对象拥有不同的属性集。

使用 DriverManager 时，有一些注意点：

- database connection URL 是一个字符串，DBMS JDBC driver 用它和数据库建立 connection，通常 URL 包含了特定数据库采取的协议、数据库的地址、要连接的目标数据库、连接的属性。详细的信息由对应的数据库供应商提供。

使用 DataSource 的例子后面的文章在 spring 项目中使用，DataSource 是 JDBC API 的补充，要比使用 DriverManager 更好一些，因为它支持使用连接池和分布式事务。

# 处理 SQLExceptions

当使用 JDBC 和数据源交互的时候，如果发生了错误，就会抛出 `SQLException` 该类继承自 Exception，SQLException 实例中封装了一些信息有助于判断到底是什么原因导致的错误。

- 使用 `SQLException.getMessage` 方法获取导致错误的原因的描述字符串；
- `SQLState` 字段封装了数据库状态 code，这些 code 大部分由 ISO/ANSI 和 Open Group（X/Open）组织维护，也有一些由专门的数据库供应商提供，比如 MySQL 官方文档列出了这些 [MySQL Error Numbers](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-error-sqlstates.html)；
- `vendorCode` 错误码，是一个整型数字标识了造成 SQLException 的原因，它的值和含义是和特定数据库实现相关，可能是底层数据源返回的实际错误代码。要追溯异常链，可以调用 `SQLException.getCause` 方法直到获得 null 值；
- 一个对 `chained exceptions` 的引用，如果发生了多个 error，就会形成 SQL 异常链，通过调用 `SQLException.getNextException` 获取其他异常信息。

## Retrieving Exceptions

下面的例子方法输出 SQLState、error code、error exception 以及可能存在的异常链：

```java
public static void printSQLException(SQLException ex) {

    for (Throwable e : ex) {
        if (e instanceof SQLException) {
            if (ignoreSQLException(
                ((SQLException)e).
                getSQLState()) == false) {

                e.printStackTrace(System.err);
                System.err.println("SQLState: " +
                    ((SQLException)e).getSQLState());

                System.err.println("Error Code: " +
                    ((SQLException)e).getErrorCode());

                System.err.println("Message: " + e.getMessage());

                Throwable t = ex.getCause();
                while(t != null) {
                    System.out.println("Cause: " + t);
                    t = t.getCause();
                }
            }
        }
    }
}
```

## Retrieving Warnings

`SQLWarning` 继承了 SQLException，它主要用于处理数据库的 access warnings。Warnings 并不会像异常一样终止程序的执行，它只是简单的警告用户某些事情发生了。例如，警告可能会让您知道您试图撤销的特权没有被撤销，或者一个警告可能告诉您在请求断开连接期间发生了错误。

`Connection`、`Statement`、`ResultSet` 都有一个 `getWarnings` 方法用于获取可能存在的警告信息。如果获得了 SQLWarning 对象，则可以继续调用它的 `getNextWarning` 获取可能存在的警告链。执行一条语句会自动清除前一条语句中的警告，这样它们就不会堆积起来。然而，这意味着，如果您想检索一条语句上报告的警告，则必须在执行另一条语句之前执行。

下面的例子演示如何获取 Statement 和 ResultSet 对象的完整警告信息：

```java
public static void getWarningsFromResultSet(ResultSet rs)
    throws SQLException {
    JDBCTutorialUtilities.printWarnings(rs.getWarnings());
}

public static void getWarningsFromStatement(Statement stmt)
    throws SQLException {
    JDBCTutorialUtilities.printWarnings(stmt.getWarnings());
}

public static void printWarnings(SQLWarning warning)
    throws SQLException {

    if (warning != null) {
        System.out.println("\n---Warning---\n");

    while (warning != null) {
        System.out.println("Message: " + warning.getMessage());
        System.out.println("SQLState: " + warning.getSQLState());
        System.out.print("Vendor error code: ");
        System.out.println(warning.getErrorCode());
        System.out.println("");
        warning = warning.getNextWarning();
    }
}
```

最常见的警告是 `DataTruncation` 警告，它继承了 SQLWarning，所有的 DataTruncation 对象都有一个 SQLState 01004，表明在读取或者写入数据时出现了问题。调用它的方法可以找到是哪个字段或者哪个参数的数据被截断了。无论截断是在读操作还是写操作上，应该传输多少字节，以及实际传输了多少字节。

## SQLException 的分类

您的 JDBC 驱动程序可能抛出一个 SQLException 的子类，它对应于一个公共的 SQLState 或一个与特定的 SQLState 类值不关联的公共错误状态。这使您能够编写更多可移植的错误处理代码。这些异常是以下类之一的子类:

- `SQLNonTransientException`；
- `SQLTransientException`；
- `SQLRecoverableException`

可以查阅最新版的 java.sql 包的文档说明或者应用使用的 JDBC 驱动的 api 文档。

其他 SQLException 的子类：

下面两种：

- `BatchUpdateException`：在批量更新操作过程中发生错误会抛出该异常。
- `SQLClientInfoException`：当一个 Connection的某些 client 属性没有被正确设置时会抛出这个异常。

# ResultSet

`ResultSet` 接口提供了一些方法用于检索和操作通过查询语句获得的结果。同时 ResultSet 对象也有一些不同的功能和特征，这些特征字段包括：type、concurrency 以及 cursor holdability（保持游标的能力）；

## type

ResultSet 的 type 决定了它的两方面的功能级别：是否可以操作游标，以及 ResultSet 对象如何反映底层数据源的并发更改。

ResultSet 对象的灵敏度由三种不同的 ResultSet type 之一决定：

- `TYPE_FORWARD_ONLY`：顾名思义，result set 不能滚动，游标只能向前移动，从第一行移动到最后一行。结果集中包含的行取决于底层数据库生成结果的方式。也就是说，它包含在执行查询时或在检索行时满足查询的行。
- `TYPE_SCROLL_INSENSITIVE`：这种 result set 可以滚动，游标可以从当前位置向前或向后移动，并且它还可以移动到某个绝对位置，结果集对打开时对底层数据源所做的更改不敏感。它包含在执行查询时或在检索行时满足查询的行。
- `TYPE_SCROLL_SENSITIVE`：这种 result set 可以滚动，游标可以相对于当前位置向前或向后移动，也可以移动到某个绝对位置，结果集反映对底层数据源所做的更改，而结果集保持打开状态。

默认的 ResultSet type 是 `TYPE_FORWARD_ONLY`；

注意：并不是所有的数据库和 JDBC 启动都支持这些 type，使用 `DatabaseMetaData.supportsResultSetType` 方法可以查看结果集是否支持特定的 type。（返回 true 或 false）。

## concurrency

ResultSet 对象的 concurrency 决定了它对 update functionality 的支持程度。

有两个 concurrency level：

- `CONCUR_READ_ONLY`：ResultSet 不能被相关方法更新；
- `CONCUR_UPDATEABLE`：ResultSet 对象可以被相关方法更新；

默认的 ResultSet concurrency 是 `CONCUR_READ_ONLY`；

注意：并不是所有数据库和 JDBC Driver 都支持这两个级别，可以使用 `DatabaseMetaData.supportsResultSetConcurrency` 判断是否支持

## Cursor Holdability

在当前事务中调用 `Connection.commit` 可以关闭在事务中创建的 ResultSet 对象，但是有些情况却不会像我们期望的那样。应用程序可以设置ResultSet 的 `holdability` 属性来决定是否在调用 commit 方法的时候关闭 ResultSet 的 cursor；

以下 ResultSet 常量可以提供给 Connection 的方法 createStatement、 prepareStatement 和 prepareCall：

- `HOLD_CURSORS_OVER_COMMIT`：ResultSet 的 cursor 不会被关闭，它们是 holdable 的：当调用 commit 方法提交事务后，cursor 仍然是打开的状态，如果应用程序中很多场景都是使用只读的 ResultSet 对象，可以考虑使用它；
- `CLOSE_CURSORS_AT_COMMIT`：当 commit 方法被调用时，ResultSet 的 cursor 就会关闭，这样做可以提升程序的性能；

注意：并不是所有的数据库和 JDBC Driver 实现都支持 holdable 和 non-holdable 的 cursor。

下面的例子演示 ResultSet 默认的 cursor holdability，以及是否支持前面说的两种常量：

```java
public static void cursorHoldabilitySupport(Connection conn)
    throws SQLException {

    DatabaseMetaData dbMetaData = conn.getMetaData();
    System.out.println("ResultSet.HOLD_CURSORS_OVER_COMMIT = " +
        ResultSet.HOLD_CURSORS_OVER_COMMIT);

    System.out.println("ResultSet.CLOSE_CURSORS_AT_COMMIT = " +
        ResultSet.CLOSE_CURSORS_AT_COMMIT);

    System.out.println("Default cursor holdability: " +
        dbMetaData.getResultSetHoldability());

    System.out.println("Supports HOLD_CURSORS_OVER_COMMIT? " +
        dbMetaData.supportsResultSetHoldability(
            ResultSet.HOLD_CURSORS_OVER_COMMIT));

    System.out.println("Supports CLOSE_CURSORS_AT_COMMIT? " +
        dbMetaData.supportsResultSetHoldability(
            ResultSet.CLOSE_CURSORS_AT_COMMIT));
}
```

## Cursor

正如前面所说的，可以通过一个 cursor 访问 ResultSet 中的数据，它指向 ResultSet 对象中的某一个 row。但是要注意的是当 ResultSet 刚创建的时候，cursor 是指向第一行的前一行的，所以我们要通过这样的循环方式：

```java
ResultSet rs = ...
while (rs.next()) {
   .....
}
```

当然还有其他 cursor 方法可以根据需要移动 cursor；

注意默认的 sensitivity 是 TYPE_FORWARD_ONLY，就是只能向前移动直到最后一行。





https://docs.oracle.com/javase/tutorial/jdbc/basics/retrieving.html

