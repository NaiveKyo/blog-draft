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



## JDBC 4.2 API

Java SE 的 JDBC 是分版本的，JDK 8 中 JDBC 的版本是 4.2，相关 API 在两个包中：

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