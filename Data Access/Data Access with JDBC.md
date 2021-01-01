## Data Access with JDBC 使用JDBC进行数据访问

下表中概述的操作序列可能最好地显示了Spring Framework JDBC抽象提供的值。该表显示了Spring负责哪些操作，哪些操作是您的责任。

**表4. Spring JDBC-谁，做什么？**

|                                |        |      |
| :----------------------------- | :----- | :--- |
| 动作                           | spring | 你   |
| 定义连接参数。                 |        | X    |
| 打开连接。                     | X      |      |
| 指定SQL语句。                  |        | X    |
| 声明参数并提供参数值           |        | X    |
| 准备并执行该语句。             | X      |      |
| 设置循环以遍历结果（如果有）。 | X      |      |
| 进行每次迭代的工作。           |        | X    |
| 处理任何异常。                 | X      |      |
| 处理交易。                     | X      |      |
| 关闭连接，语句和结果集。       | X      |      |

Spring框架负责所有可能使JDBC成为乏味的API的低级细节。

### 3.1. Choosing an Approach for JDBC Database Access 选择一种用于JDBC数据库访问的方法

您可以选择几种方法来构成JDBC数据库访问的基础。除了的三种风格外`JdbcTemplate`，一种新的`SimpleJdbcInsert`和 `SimpleJdbcCall`方法还优化了数据库元数据，并且RDBMS Object样式采用了一种更加面向对象的方法，类似于JDO Query设计。一旦开始使用这些方法之一，您仍然可以混合搭配以包含来自其他方法的功能。所有方法都需要兼容JDBC 2.0的驱动程序，某些高级功能需要JDBC 3.0驱动程序。

- `JdbcTemplate`是经典且最受欢迎的Spring JDBC方法。这种“最低级别”的方法以及所有其他方法都在后台使用了JdbcTemplate。
- `NamedParameterJdbcTemplate`封装`JdbcTemplate`以提供命名参数，而不是传统的JDBC`?`占位符。当您有多个SQL语句参数时，此方法可提供更好的文档编制和易用性。
- `SimpleJdbcInsert`并`SimpleJdbcCall`优化数据库元数据以限制必要的配置量。这种方法简化了编码，因此您只需要提供表或过程的名称，并提供与列名称匹配的参数映射即可。仅当数据库提供足够的元数据时，此方法才有效。如果数据库不提供此元数据，则必须提供参数的显式配置。
- RDBMS对象包括`MappingSqlQuery`，`SqlUpdate`和`StoredProcedure`，你需要你的数据访问层的初始化过程中创建可重用的，线程安全的对象。此方法以JDO Query为模型，其中您定义查询字符串，声明参数并编译查询。完成后，可以使用各种参数值多次调用execute方法。

### 3.2. Package Hierarchy 包层次结构

Spring框架的JDBC抽象框架由四个不同的包组成：

- `core`：该`org.springframework.jdbc.core`软件包包含`JdbcTemplate`该类及其各种回调接口，以及各种相关类。名为的子包 `org.springframework.jdbc.core.simple`包含`SimpleJdbcInsert`和 `SimpleJdbcCall`类。另一个名为`org.springframework.jdbc.core.namedparam`的子包 包含`NamedParameterJdbcTemplate` 该类和相关的支持类。请参阅[使用JDBC核心类控制基本JDBC处理和错误处理](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-core)，[JDBC批处理操作](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-advanced-jdbc)以及 [使用这些`SimpleJdbc`类简化JDBC操作](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-simple-jdbc)。
- `datasource`：该`org.springframework.jdbc.datasource`软件包包含一个实用程序类，可轻松`DataSource`访问该类， 并提供各种简单的`DataSource`实现，可用于在Java EE容器之外测试和运行未修改的JDBC代码。名为的子程序包`org.springfamework.jdbc.datasource.embedded`支持使用Java数据库引擎（例如HSQL，H2和Derby）创建嵌入式数据库。请参见 [控制数据库连接](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-connections)和[嵌入式数据库支持](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-embedded-database-support)。
- `object`：`org.springframework.jdbc.object`程序包包含将RDBMS查询，更新和存储过程表示为线程安全的可重用对象的类。请参阅将 [JDBC操作建模为Java对象](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-object)。尽管查询返回的对象自然会与数据库断开连接，但此方法由JDO建模。较高级别的JDBC抽象取决于`org.springframework.jdbc.core`程序包中的较低级别的抽象。
- `support`：该`org.springframework.jdbc.support`软件包提供了`SQLException`翻译功能和一些实用程序类。JDBC处理期间引发的异常将转换为`org.springframework.dao`包中定义的异常。这意味着使用Spring JDBC抽象层的代码不需要实现JDBC或RDBMS特定的错误处理。所有翻译的异常均未选中，这使您可以选择捕获可从中恢复的异常，同时将其他异常传播到调用方。请参阅[使用`SQLExceptionTranslator`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SQLExceptionTranslator)。

### 3.3. Using the JDBC Core Classes to Control Basic JDBC Processing and Error Handling 使用JDBC核心类控制基本JDBC处理和错误处理

本节介绍如何使用JDBC核心类来控制基本的JDBC处理，包括错误处理。它包括以下主题：

- [使用 `JdbcTemplate`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-JdbcTemplate)
- [使用 `NamedParameterJdbcTemplate`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-NamedParameterJdbcTemplate)
- [使用 `SQLExceptionTranslator`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SQLExceptionTranslator)
- [运行声明](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-statements-executing)
- [运行查询](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-statements-querying)
- [更新数据库](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-updates)
- [检索自动生成的密钥](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-auto-generated-keys)

#### 3.3.1. Using `JdbcTemplate`使用`JdbcTemplate`

`JdbcTemplate`是JDBC核心软件包中的中心类。它处理资源的创建和释放，这有助于您避免常见的错误，例如忘记关闭连接。它执行核心JDBC工作流程的基本任务（例如，语句创建和执行），而使应用程序代码提供SQL并提取结果。本`JdbcTemplate`类：

- 运行SQL查询
- 更新语句和存储过程调用
- 对`ResultSet`实例执行迭代并提取返回的参数值。
- 捕获JDBC异常，并将其转换为`org.springframework.dao`程序包中定义的通用，信息量更大的异常层次结构。（请参见[一致的异常层次结构](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#dao-exceptions)。）

当`JdbcTemplate`对代码使用时，只需实现回调接口，即可为它们提供明确定义的协定。给定该类`Connection`提供的 `JdbcTemplate`，`PreparedStatementCreator` 回调接口将创建一个准备好的语句，提供SQL和任何必要的参数。`CallableStatementCreator`创建可调用语句的接口也是如此 。该 `RowCallbackHandler`接口从的每一行提取值`ResultSet`。

您可以`JdbcTemplate`在DAO实现中通过直接实例化使用`DataSource`引用，也可以在Spring IoC容器中对其进行配置，并将其作为Bean引用提供给DAO。

![1609488638411](assets/1609488638411.png)

此类发出的所有SQL都记录在与`DEBUG`模板实例的完全限定类名称对应的类别下的级别（通常为 `JdbcTemplate`，但是如果使用`JdbcTemplate`该类的自定义子类，则可能会有所不同 ）。

以下各节提供了一些`JdbcTemplate`用法示例。这些示例不是.NET公开的所有功能的详尽列表`JdbcTemplate`。请参阅附带的[javadoc](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html)。

##### 查询（`SELECT`）

以下查询获取关系中的行数：

```java
int rowCount = this.jdbcTemplate.queryForObject("select count(*) from t_actor", Integer.class);
```

以下查询使用绑定变量：

```
int countOfActorsNamedJoe = this.jdbcTemplate.queryForObject(
        "select count(*) from t_actor where first_name = ?", Integer.class, "Joe");
```

以下查询查找`String`：

```
String lastName = this.jdbcTemplate.queryForObject(
        "select last_name from t_actor where id = ?",
        String.class, 1212L);
```

以下查询查找并填充单个域对象：

```
Actor actor = jdbcTemplate.queryForObject(
        "select first_name, last_name from t_actor where id = ?",
        (resultSet, rowNum) -> {
            Actor newActor = new Actor();
            newActor.setFirstName(resultSet.getString("first_name"));
            newActor.setLastName(resultSet.getString("last_name"));
            return newActor;
        },
        1212L);
```

以下查询查找并填充域对象列表：

```
List<Actor> actors = this.jdbcTemplate.query(
        "select first_name, last_name from t_actor",
        (resultSet, rowNum) -> {
            Actor actor = new Actor();
            actor.setFirstName(resultSet.getString("first_name"));
            actor.setLastName(resultSet.getString("last_name"));
            return actor;
        });
```

如果最后两个代码片段确实存在于同一应用程序中，则删除两个`RowMapper`lambda表达式中存在的重复项并将它们提取到单个字段中，然后可以根据需要由DAO方法引用，这是有意义的。例如，最好编写前面的代码段，如下所示：

```
private final RowMapper<Actor> actorRowMapper = (resultSet, rowNum) -> {
    Actor actor = new Actor();
    actor.setFirstName(resultSet.getString("first_name"));
    actor.setLastName(resultSet.getString("last_name"));
    return actor;
};

public List<Actor> findAllActors() {
    return this.jdbcTemplate.query( "select first_name, last_name from t_actor", actorRowMapper);
}
```

##### 更新（`INSERT`，`UPDATE`，和`DELETE`）与`JdbcTemplate`

您可以使用该`update(..)`方法执行插入，更新和删除操作。参数值通常作为变量参数提供，或者作为对象数组提供。

下面的示例插入一个新条目：

```
this.jdbcTemplate.update(
        "insert into t_actor (first_name, last_name) values (?, ?)",
        "Leonor", "Watling");
```

下面的示例更新现有条目：

```
this.jdbcTemplate.update(
        "update t_actor set last_name = ? where id = ?",
        "Banjo", 5276L);
```

下面的示例删除一个条目：

```
this.jdbcTemplate.update(
        "delete from t_actor where id = ?",
        Long.valueOf(actorId));
```

##### 其他`JdbcTemplate`操作

您可以使用该`execute(..)`方法运行任何任意SQL。因此，该方法通常用于DDL语句。带有回调接口，绑定变量数组等的变体极大地重载了该变量。以下示例创建一个表：

```
this.jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
```

下面的示例调用一个存储过程：

```
this.jdbcTemplate.update(
        "call SUPPORT.REFRESH_ACTORS_SUMMARY(?)",
        Long.valueOf(unionId));
```

[稍后](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-StoredProcedure)将[介绍](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-StoredProcedure)更复杂的存储过程支持。

##### `JdbcTemplate` 最佳实践

`JdbcTemplate`一旦配置，该类的实例是线程安全的。这很重要，因为这意味着您可以配置的单个实例，`JdbcTemplate` 然后将该共享引用安全地注入到多个DAO（或存储库）中。该`JdbcTemplate`是有状态的，因为它保持一个参考`DataSource`，但这种状态不是会话状态。

使用`JdbcTemplate`类（和关联的 [`NamedParameterJdbcTemplate`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-NamedParameterJdbcTemplate)类）时，通常的做法是`DataSource`在Spring配置文件中配置，然后将共享`DataSource`bean依赖注入到DAO类中。将`JdbcTemplate`在二传手的创建`DataSource`。这将导致类似于以下内容的DAO：

```
public class JdbcCorporateEventDao implements CorporateEventDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    // JDBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

以下示例显示了相应的XML配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="corporateEventDao" class="com.example.JdbcCorporateEventDao">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <context:property-placeholder location="jdbc.properties"/>

</beans>
```

显式配置的替代方法是使用组件扫描和注释支持进行依赖项注入。在这种情况下，您可以用注释类`@Repository` （这使它成为组件扫描的候选对象），并用注释`DataSource`setter方法`@Autowired`。以下示例显示了如何执行此操作：

```
@Repository //1
public class JdbcCorporateEventDao implements CorporateEventDao {

    private JdbcTemplate jdbcTemplate;

    @Autowired //2
    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource); //3
    }

    // JDBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

| 1    | 使用类注解`@Repository`。                    |
| ---- | -------------------------------------------- |
| 2    | 用`DataSource`的setter方法`@Autowired`。     |
| 3    | 使用创建一个新`JdbcTemplate`的`DataSource`。 |

以下示例显示了相应的XML配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- Scans within the base package of the application for @Component classes to configure as beans -->
    <context:component-scan base-package="org.springframework.docs.test" />

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <context:property-placeholder location="jdbc.properties"/>

</beans>
```

如果您使用Spring的`JdbcDaoSupport`类，并且各种JDBC支持的DAO类从中扩展，则您的子类`setDataSource(..)`将从该类继承一个方法 `JdbcDaoSupport`。您可以选择是否从此类继承。所述 `JdbcDaoSupport`类被设置为仅一个方便。

无论您选择使用（或不使用）以上哪种模板初始化样式，都不必在`JdbcTemplate`每次要运行SQL时都创建类的新实例。配置完成后，`JdbcTemplate`实例是线程安全的。如果您的应用程序访问多个数据库，则可能需要多个`JdbcTemplate`实例，这需要多个实例，随后需要`DataSources`多个不同配置的`JdbcTemplate`实例。

#### 3.3.2. Using `NamedParameterJdbcTemplate` 使用`NamedParameterJdbcTemplate`

与`NamedParameterJdbcTemplate`仅使用经典占位符（`'?'`）参数进行编程的类相反，该类增加了使用命名参数对JDBC语句进行编程的支持。本`NamedParameterJdbcTemplate`类封装了一个 `JdbcTemplate`和代表对包装`JdbcTemplate`做太多的工作。本节仅描述`NamedParameterJdbcTemplate`类中与其`JdbcTemplate`本身不同的那些区域，即使用命名参数对JDBC语句进行编程。以下示例显示如何使用`NamedParameterJdbcTemplate`：

```java
// some JDBC-backed DAO class...
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActorsByFirstName(String firstName) {

    String sql = "select count(*) from T_ACTOR where first_name = :first_name";

    SqlParameterSource namedParameters = new MapSqlParameterSource("first_name", firstName);

    return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
}
```

请注意，在分配给`sql` 变量的值和插入`namedParameters` 变量（类型为`MapSqlParameterSource`）的相应值中使用了命名参数符号。

另外，您也可以`NamedParameterJdbcTemplate`使用`Map`-based样式将命名参数及其对应的值传递给 实例。`NamedParameterJdbcOperations`由the`NamedParameterJdbcTemplate`类公开和由类实现的 其余方法遵循类似的模式，此处不再赘述。

以下示例说明了`Map`基于-样式的用法：

```
// some JDBC-backed DAO class...
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActorsByFirstName(String firstName) {

    String sql = "select count(*) from T_ACTOR where first_name = :first_name";

    Map<String, String> namedParameters = Collections.singletonMap("first_name", firstName);

    return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters,  Integer.class);
}
```

与`NamedParameterJdbcTemplate`该`SqlParameterSource`接口相关的一个不错的功能（并存在于同一Java包中）。您已经在之前的代码片段之一（`MapSqlParameterSource`该类）中看到了此接口的实现示例 。An`SqlParameterSource`是的命名参数值的来源`NamedParameterJdbcTemplate`。该`MapSqlParameterSource`班是一个简单的实现，它是围绕着一个适配器`java.util.Map`，其中的键是参数名称和值的参数值。

另一个`SqlParameterSource`实现是`BeanPropertySqlParameterSource` 类。此类包装一个任意的JavaBean（即，一个遵循[JavaBean约定](https://www.oracle.com/technetwork/java/javase/documentation/spec-136004.html)的类的实例），并将包装的JavaBean的属性用作命名参数值的源。

以下示例显示了典型的JavaBean：

```
public class Actor {

    private Long id;
    private String firstName;
    private String lastName;

    public String getFirstName() {
        return this.firstName;
    }

    public String getLastName() {
        return this.lastName;
    }

    public Long getId() {
        return this.id;
    }

    // setters omitted...

}
```

以下示例使用a`NamedParameterJdbcTemplate`返回上一示例中显示的类的成员数：

```
// some JDBC-backed DAO class...
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActors(Actor exampleActor) {

    // notice how the named parameters match the properties of the above 'Actor' class
    String sql = "select count(*) from T_ACTOR where first_name = :firstName and last_name = :lastName";

    SqlParameterSource namedParameters = new BeanPropertySqlParameterSource(exampleActor);

    return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
}
```

请记住，`NamedParameterJdbcTemplate`该类包装了经典`JdbcTemplate` 模板。如果需要访问包装的`JdbcTemplate`实例以访问仅在`JdbcTemplate`类中提供的功能，则可以使用该 `getJdbcOperations()`方法`JdbcTemplate`通过 `JdbcOperations`接口访问包装的实例。

另请参阅[`JdbcTemplate`最佳实践，](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-JdbcTemplate-idioms)以获取有关`NamedParameterJdbcTemplate`在应用程序上下文中使用类的准则 。

#### 3.3.3. Using `SQLExceptionTranslator`

`SQLExceptionTranslator`是一个由可以在`SQLExceptions`Spring自己之间转换的类实现的接口，`org.springframework.dao.DataAccessException`与数据访问策略无关。为了提高精度，实现可以是通用的（例如，使用SQLState代码用于JDBC）或专有的（例如，使用Oracle错误代码）。

`SQLErrorCodeSQLExceptionTranslator`是`SQLExceptionTranslator` 默认情况下使用的实现。此实现使用特定的供应商代码。比`SQLState`实现更精确。错误代码转换基于JavaBean类型类（称为）中保存的代码`SQLErrorCodes`。此类是由创建和填充的`SQLErrorCodesFactory`，而顾名思义，这是一个`SQLErrorCodes`基于名为的配置文件的内容 进行创建的工厂`sql-error-codes.xml`。该文件中填充有供应商代码，并基于的 `DatabaseProductName`来源`DatabaseMetaData`。使用您正在使用的实际数据库的代码。

将`SQLErrorCodeSQLExceptionTranslator`按照下列顺序应用匹配规则：

1. 子类实现的任何自定义转换。通常，将使用提供的混凝土 `SQLErrorCodeSQLExceptionTranslator`，因此该规则不适用。仅当您确实提供了子类实现时，它才适用。
2. `SQLExceptionTranslator`作为类的`customSqlExceptionTranslator`属性提供的接口的任何自定义实现`SQLErrorCodes`。
3. 搜索`CustomSQLErrorCodesTranslation`该类的实例列表（为该类的`customTranslations`属性提供 `SQLErrorCodes`）以查找匹配项。
4. 错误代码匹配被应用。
5. 使用后备翻译器。`SQLExceptionSubclassTranslator`是默认的后备翻译器。如果此翻译不可用，则下一个后备翻译器是`SQLStateSQLExceptionTranslator`。

![1609489473977](assets/1609489473977.png)

您可以扩展`SQLErrorCodeSQLExceptionTranslator`，如以下示例所示：

```java
public class CustomSQLErrorCodesTranslator extends SQLErrorCodeSQLExceptionTranslator {

    protected DataAccessException customTranslate(String task, String sql, SQLException sqlEx) {
        if (sqlEx.getErrorCode() == -12345) {
            return new DeadlockLoserDataAccessException(task, sqlEx);
        }
        return null;
    }
}
```

在前面的示例中，特定的错误代码（`-12345`）被转换，而其他错误则由默认转换程序实现转换。要使用此自定义转换器，必须将其传递给`JdbcTemplate`通过方法 `setExceptionTranslator`，并且必须`JdbcTemplate`在需要此转换器的所有数据访问处理中使用它。以下示例显示了如何使用此自定义转换器：

```java
private JdbcTemplate jdbcTemplate;

public void setDataSource(DataSource dataSource) {

    // create a JdbcTemplate and set data source
    this.jdbcTemplate = new JdbcTemplate();
    this.jdbcTemplate.setDataSource(dataSource);

    // create a custom translator and set the DataSource for the default translation lookup
    CustomSQLErrorCodesTranslator tr = new CustomSQLErrorCodesTranslator();
    tr.setDataSource(dataSource);
    this.jdbcTemplate.setExceptionTranslator(tr);

}

public void updateShippingCharge(long orderId, long pct) {
    // use the prepared JdbcTemplate for this update
    this.jdbcTemplate.update("update orders" +
        " set shipping_charge = shipping_charge * ? / 100" +
        " where id = ?", pct, orderId);
}
```

自定义转换器会传递数据源，以查找中的错误代码 `sql-error-codes.xml`。

#### 3.3.4. Running Statements 运行声明

运行SQL语句需要很少的代码。您需要一个`DataSource`和一个 `JdbcTemplate`，包括随附带的便捷方法 `JdbcTemplate`。下面的示例显示了创建一个新表的最小但功能齐全的类需要包含的内容：

```java
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;

public class ExecuteAStatement {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public void doExecute() {
        this.jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
    }
}
```

####  3.3.5. Running Queries 运行查询

一些查询方法返回单个值。要从一行中检索计数或特定值，请使用`queryForObject(..)`。后者将返回的JDBC转换`Type`为作为参数传入的Java类。如果类型转换无效， `InvalidDataAccessApiUsageException`则抛出。以下示例包含两种查询方法，一种用于`int`，一种用于`String`：

```java
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;

public class RunAQuery {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int getCount() {
        return this.jdbcTemplate.queryForObject("select count(*) from mytable", Integer.class);
    }

    public String getName() {
        return this.jdbcTemplate.queryForObject("select name from mytable", String.class);
    }
}
```

除了单个结果查询方法外，还有几种方法返回一个列表，其中包含查询返回的每一行的条目。最通用的方法是`queryForList(..)`，它使用列名作为键，返回一个`List`，其中每个元素是`Map`每个列的一个条目。如果在前面的示例中添加一种方法来检索所有行的列表，则可能如下所示：

```java
private JdbcTemplate jdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.jdbcTemplate = new JdbcTemplate(dataSource);
}

public List<Map<String, Object>> getList() {
    return this.jdbcTemplate.queryForList("select * from mytable");
}
```

返回的列表类似于以下内容：

```
[{name = Bob，id = 1}，{name = Mary，id = 2}]
```

#### 3.3.6. Updating the Database  更新数据库

下面的示例更新某个主键的列：

```java
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;

public class ExecuteAnUpdate {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public void setName(int id, String name) {
        this.jdbcTemplate.update("update mytable set name = ? where id = ?", name, id);
    }
}
```

在前面的示例中，SQL语句具有用于行参数的占位符。您可以将参数值作为varargs或作为对象数组传递。因此，您应该在原始包装器类中显式包装原始器，或者应该使用自动装箱。

#### 3.3.7. Retrieving Auto-generated Keys 检索自动生成的密钥

一种`update()`便捷方法支持检索由数据库生成的主键。此支持是JDBC 3.0标准的一部分。有关详细信息，请参见规范的第13.6章。该方法以a`PreparedStatementCreator`作为其第一个参数，这是指定所需插入语句的方式。另一个参数是a `KeyHolder`，其中包含从更新成功返回时生成的密钥。没有标准的单一方法来创建适当`PreparedStatement` 的方法（这说明了为什么方法签名就是这样）。以下示例在Oracle上有效，但在其他平台上可能不适用：

```java
final String INSERT_SQL = "insert into my_test (name) values(?)";
final String name = "Rob";

KeyHolder keyHolder = new GeneratedKeyHolder();
jdbcTemplate.update(connection -> {
    PreparedStatement ps = connection.prepareStatement(INSERT_SQL, new String[] { "id" });
    ps.setString(1, name);
    return ps;
}, keyHolder);

// keyHolder.getKey() now contains the generated key
```

###  3.4. Controlling Database Connections 控制数据库连接

本节内容包括：

- [使用 `DataSource`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-datasource)
- [使用 `DataSourceUtils`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-DataSourceUtils)
- [实施中 `SmartDataSource`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SmartDataSource)
- [延伸 `AbstractDataSource`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-AbstractDataSource)
- [使用 `SingleConnectionDataSource`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SingleConnectionDataSource)
- [使用 `DriverManagerDataSource`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-DriverManagerDataSource)
- [使用 `TransactionAwareDataSourceProxy`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-TransactionAwareDataSourceProxy)
- [使用 `DataSourceTransactionManager`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-DataSourceTransactionManager)

#### 3.4.1. Using `DataSource`

Spring通过来获得与数据库的连接`DataSource`。A`DataSource`是JDBC规范的一部分，是通用的连接工厂。它允许容器或框架从应用程序代码中隐藏连接池和事务管理问题。作为开发人员，您无需了解有关如何连接到数据库的详细信息。这是设置数据源的管理员的责任。您很可能在开发和测试代码时同时担当这两个角色，但是不必一定要知道如何配置生产数据源。

当使用Spring的JDBC层时，您可以从JNDI获取数据源，或者可以使用第三方提供的连接池实现来配置自己的数据源。传统的选择是带有豆样式`DataSource`类的Apache Commons DBCP和C3P0 。对于现代JDBC连接池，请考虑使用具有其生成器样式的API的HikariCP。

![1609489799251](assets/1609489799251.png)

下一节使用Spring的`DriverManagerDataSource`实现。`DataSource`稍后将介绍其他几种变体。

要配置一个`DriverManagerDataSource`：

1. 与`DriverManagerDataSource`通常获得JDBC连接的方式获得连接。
2. 指定JDBC驱动程序的标准类名，以便`DriverManager` 可以加载驱动程序类。
3. 提供在JDBC驱动程序之间变化的URL。（有关正确的值，请参阅驱动程序的文档。）
4. 提供用户名和密码以连接到数据库。

以下示例显示了如何`DriverManagerDataSource`在Java中配置：

```java
DriverManagerDataSource dataSource = new DriverManagerDataSource();
dataSource.setDriverClassName("org.hsqldb.jdbcDriver");
dataSource.setUrl("jdbc:hsqldb:hsql://localhost:");
dataSource.setUsername("sa");
dataSource.setPassword("");
```

以下示例显示了相应的XML配置：

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

接下来的两个示例显示了DBCP和C3P0的基本连接和配置。要了解更多有助于控制池功能的选项，请参阅相应连接池实现的产品文档。

以下示例显示了DBCP配置：

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

以下示例显示了C3P0配置：

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
    <property name="driverClass" value="${jdbc.driverClassName}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

#### 3.4.2. Using `DataSourceUtils`使用`DataSourceUtils`

该`DataSourceUtils`类是一个方便且功能强大的辅助类，提供 `static`方法来获取从JNDI和连接，如果有必要密切的联系。它支持带有的线程绑定连接`DataSourceTransactionManager`。

#### 3.4.3. Implementing `SmartDataSource`

该`SmartDataSource`接口应由可以提供与关系数据库的连接的类实现。它扩展了`DataSource`接口，以使使用它的类可以查询给定操作后是否应关闭连接。当您知道需要重用连接时，这种用法很有效。

#### 3.4.4. Extending `AbstractDataSource`

`AbstractDataSource`是`abstract`Spring`DataSource` 实现的基类。它实现了所有`DataSource`实现通用的代码。`AbstractDataSource`如果您编写自己的`DataSource` 实现，则应该扩展该类。

#### 3.4.5. Using `SingleConnectionDataSource`

`SingleConnectionDataSource`类是的一个实现`SmartDataSource` ，它包装单个接口`Connection`的是在每次使用后不关闭。这不是多线程功能。

如果有任何客户端代码要求`close`建立池连接（例如使用持久性工具时），则应将`suppressClose`属性设置为`true`。此设置返回一个环绕物理连接的封闭代理。请注意，您不能再将此对象转换为本地Oracle`Connection`或类似对象。

`SingleConnectionDataSource`主要是测试课程。例如，它结合简单的JNDI环境，可以在应用服务器外部轻松测试代码。与相比 `DriverManagerDataSource`，它始终重复使用相同的连接，避免了过多的物理连接创建。

#### 3.4.6. Using `DriverManagerDataSource`使用`DriverManagerDataSource`

该`DriverManagerDataSource`类是标准的实现`DataSource` 接口，用于配置通过bean的属性，并返回一个新的纯JDBC驱动程序 `Connection`每次。

此实现对于Java EE容器外部的测试和独立环境非常有用，可以作为`DataSource`Spring IoC容器中的bean或与简单的JNDI环境结合使用。假定池的`Connection.close()`调用将关闭连接，因此任何可`DataSource`感知的持久性代码都应起作用。但是，`commons-dbcp`即使在测试环境中，使用JavaBean风格的连接池（例如）也是如此简单，以至于总是总是最好使用这样的连接池 `DriverManagerDataSource`。

#### 3.4.7. Using `TransactionAwareDataSourceProxy` 使用`TransactionAwareDataSourceProxy`

`TransactionAwareDataSourceProxy`是目标的代理`DataSource`。代理包装该目标`DataSource`以增加对Spring管理的事务的认识。在这方面，它类似于`DataSource`Java EE服务器提供的事务性JNDI 。

![1609489990424](assets/1609489990424.png)

有关[`TransactionAwareDataSourceProxy`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/jdbc/datasource/TransactionAwareDataSourceProxy.html) 更多详细信息，请参见javadoc。

#### 3.4.8. Using `DataSourceTransactionManager` 使用`DataSourceTransactionManager`

该`DataSourceTransactionManager`类是`PlatformTransactionManager` 为单JDBC数据源的实现。它将JDBC连接从指定的数据源绑定到当前正在执行的线程，可能允许每个数据源一个线程连接。

需要应用程序代码来通过`DataSourceUtils.getConnection(DataSource)`而不是Java EE的标准 来检索JDBC连接 `DataSource.getConnection`。它抛出未经检查的`org.springframework.dao`异常，而不是checked `SQLExceptions`。所有框架类（例如`JdbcTemplate`）都隐式使用此策略。如果不与该事务管理器一起使用，则查找策略的行为与普通策略完全相同。因此，可以在任何情况下使用它。

该`DataSourceTransactionManager`级支持自定义隔离级别和超时即得到应用适当的SQL语句查询超时。为了支持后者，应用程序代码必须为每个创建的语句使用`JdbcTemplate`或调用 `DataSourceUtils.applyTransactionTimeout(..)`方法。

您可以使用此实现，而不是`JtaTransactionManager`在单资源情况下使用，因为它不需要容器支持JTA。只要您坚持要求的连接查找模式，则在两者之间切换仅是配置问题。JTA不支持自定义隔离级别。

### 3.5. JDBC Batch Operations JDBC批处理操作

如果将多个调用批处理到同一条准备好的语句，则大多数JDBC驱动程序都会提高性能。通过将更新分组，可以限制数据库的往返次数。

#### 3.5.1. Basic Batch Operations with `JdbcTemplate` 基本批处理操作`JdbcTemplate`

您可以`JdbcTemplate`通过实现特殊接口的两个方法来完成批处理`BatchPreparedStatementSetter`，并将该实现作为`batchUpdate`方法调用中的第二个参数传入。您可以使用该`getBatchSize`方法来提供当前批次的大小。您可以使用该`setValues`方法为准备好的语句的参数设置值。该方法称为您在`getBatchSize`调用中指定的次数。以下示例`t_actor`根据列表中的条目更新表，并将整个列表用作批处理：

```java
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int[] batchUpdate(final List<Actor> actors) {
        return this.jdbcTemplate.batchUpdate(
                "update t_actor set first_name = ?, last_name = ? where id = ?",
                new BatchPreparedStatementSetter() {
                    public void setValues(PreparedStatement ps, int i) throws SQLException {
                        Actor actor = actors.get(i);
                        ps.setString(1, actor.getFirstName());
                        ps.setString(2, actor.getLastName());
                        ps.setLong(3, actor.getId().longValue());
                    }
                    public int getBatchSize() {
                        return actors.size();
                    }
                });
    }

    // ... additional methods
}
```

如果您处理更新流或从文件中读取文件，则可能具有首选的批处理大小，但最后一批可能没有该数量的条目。在这种情况下，您可以使用该`InterruptibleBatchPreparedStatementSetter`接口，该接口可在输入源耗尽后中断批处理。该`isBatchExhausted`方法使您可以发出批处理结束的信号。

#### 3.5.2. Batch Operations with a List of Objects 具有对象列表的批处理操作

无论是`JdbcTemplate`与`NamedParameterJdbcTemplate`提供了提供批更新的替代方式。无需实现特殊的批处理接口，而是将调用中的所有参数值作为列表提供。框架遍历这些值并使用内部的准备好的语句设置器。API会有所不同，具体取决于您是否使用命名参数。对于命名参数，您提供的数组 `SqlParameterSource`，每个批次的成员都有一个条目。您可以使用 `SqlParameterSourceUtils.createBatch`便捷方法创建此数组，传入一个bean样式对象（带有对应于参数的getter方法），- `String`keyed`Map`实例（包含对应的参数作为值）或它们的混合的数组 。

以下示例显示了使用命名参数的批量更新：

```java
public class JdbcActorDao implements ActorDao {

    private NamedParameterTemplate namedParameterJdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
    }

    public int[] batchUpdate(List<Actor> actors) {
        return this.namedParameterJdbcTemplate.batchUpdate(
                "update t_actor set first_name = :firstName, last_name = :lastName where id = :id",
                SqlParameterSourceUtils.createBatch(actors));
    }

    // ... additional methods
}
```

对于使用经典`?`占位符的SQL语句，您传入一个包含带有更新值的对象数组的列表。该对象数组必须在SQL语句中的每个占位符处都有一个条目，并且它们的顺序必须与SQL语句中定义的顺序相同。

除了使用经典的JDBC`?`占位符外，以下示例与上述示例相同：

```java
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int[] batchUpdate(final List<Actor> actors) {
        List<Object[]> batch = new ArrayList<Object[]>();
        for (Actor actor : actors) {
            Object[] values = new Object[] {
                    actor.getFirstName(), actor.getLastName(), actor.getId()};
            batch.add(values);
        }
        return this.jdbcTemplate.batchUpdate(
                "update t_actor set first_name = ?, last_name = ? where id = ?",
                batch);
    }

    // ... additional methods
}
```

我们前面介绍的所有批处理更新方法都返回一个`int`数组，其中包含每个批处理条目的受影响行数。此计数由JDBC驱动程序报告。如果该计数不可用，则JDBC驱动程序将返回值`-2`。

![1609490164617](assets/1609490164617.png)

#### 3.5.3. Batch Operations with Multiple Batches具有多个批次的批次操作

前面的批处理更新示例处理的批处理太大，以至于您想将它们分解成几个较小的批处理。您可以通过多次调用该`batchUpdate`方法来使用前面提到的方法，但是现在有了一个更方便的方法。除了SQL语句外，此方法还使用 `Collection`包含参数的对象，每个批处理要进行的更新数量以及`ParameterizedPreparedStatementSetter`设置准备好的语句的参数值的对象。框架遍历提供的值，并将更新调用分成指定大小的批处理。

以下示例显示了使用100的批量大小的批量更新：

```java
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int[][] batchUpdate(final Collection<Actor> actors) {
        int[][] updateCounts = jdbcTemplate.batchUpdate(
                "update t_actor set first_name = ?, last_name = ? where id = ?",
                actors,
                100,
                (PreparedStatement ps, Actor actor) -> {
                    ps.setString(1, actor.getFirstName());
                    ps.setString(2, actor.getLastName());
                    ps.setLong(3, actor.getId().longValue());
                });
        return updateCounts;
    }

    // ... additional methods
}
```

此调用的批处理更新方法返回一个数组`int`数组，其中包含每个批处理的数组条目以及每个更新的受影响行数的数组。顶级数组的长度指示已执行的批处理数，第二级数组的长度指示该批处理中的更新数。每个批次中的更新数量应该是为所有批次提供的批次大小（最后一个可能更少），这取决于所提供的更新对象的总数。每个更新语句的更新计数是JDBC驱动程序报告的计数。如果该计数不可用，则JDBC驱动程序将返回值`-2`。

### 3.6. Simplifying JDBC Operations with the `SimpleJdbc` Classes使用`SimpleJdbc`类简化JDBC操作

`SimpleJdbcInsert`和`SimpleJdbcCall`类通过取可通过JDBC驱动被检索数据库的元数据的优点提供了一个简化的配置。这意味着您可以更少地进行前期配置，但是如果您希望在代码中提供所有详细信息，则可以覆盖或关闭元数据处理。

#### 3.6.1. Inserting Data by Using `SimpleJdbcInsert` 使用插入数据`SimpleJdbcInsert`

我们从查看`SimpleJdbcInsert`配置选项最少的类开始。您应该`SimpleJdbcInsert`在数据访问层的初始化方法中实例化。对于此示例，初始化方法是该 `setDataSource`方法。您不需要对类进行子`SimpleJdbcInsert`类化。而是可以创建一个新实例，并使用`withTableName`方法设置表名称。此类的配置方法遵循`fluid`返回实例的样式，该样式`SimpleJdbcInsert`使您可以链接所有配置方法。以下示例仅使用一种配置方法（我们稍后将显示多种方法的示例）：

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource).withTableName("t_actor");
    }

    public void add(Actor actor) {
        Map<String, Object> parameters = new HashMap<String, Object>(3);
        parameters.put("id", actor.getId());
        parameters.put("first_name", actor.getFirstName());
        parameters.put("last_name", actor.getLastName());
        insertActor.execute(parameters);
    }

    // ... additional methods
}
```

`execute`这里使用的方法将平原`java.util.Map`作为唯一参数。这里要注意的重要一点是，用于的键`Map`必须与数据库中定义的表的列名匹配。这是因为我们读取元数据来构造实际的insert语句。

#### 3.6.2. Retrieving Auto-generated Keys by Using `SimpleJdbcInsert`通过使用检索自动生成的密钥`SimpleJdbcInsert`

下一个示例使用与前面的示例相同的插入内容，但是它没有传递`id`，而是检索自动生成的键并将其设置在新`Actor`对象上。当创建时`SimpleJdbcInsert`，除了指定表名之外，还使用该`usingGeneratedKeyColumns`方法指定生成的键列的名称。以下清单显示了它的工作方式：

爪哇

科特林

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource)
                .withTableName("t_actor")
                .usingGeneratedKeyColumns("id");
    }

    public void add(Actor actor) {
        Map<String, Object> parameters = new HashMap<String, Object>(2);
        parameters.put("first_name", actor.getFirstName());
        parameters.put("last_name", actor.getLastName());
        Number newId = insertActor.executeAndReturnKey(parameters);
        actor.setId(newId.longValue());
    }

    // ... additional methods
}
```

使用第二种方法运行插入时的主要区别在于，您没有将插入`id`到中`Map`，而是调用了该`executeAndReturnKey`方法。这将返回一个 `java.lang.Number`对象，您可以使用该对象创建域类中使用的数字类型的实例。您不能依赖所有数据库在这里返回特定的Java类。`java.lang.Number`是您可以依赖的基类。如果您有多个自动生成的列，或者生成的值是非数字的，则可以使用`KeyHolder`从`executeAndReturnKeyHolder`方法返回的。

#### 3.6.3. Specifying Columns for a `SimpleJdbcInsert`指定一个列`SimpleJdbcInsert`

您可以通过使用方法指定列名列表来限制插入的列 `usingColumns`，如以下示例所示：

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource)
                .withTableName("t_actor")
                .usingColumns("first_name", "last_name")
                .usingGeneratedKeyColumns("id");
    }

    public void add(Actor actor) {
        Map<String, Object> parameters = new HashMap<String, Object>(2);
        parameters.put("first_name", actor.getFirstName());
        parameters.put("last_name", actor.getLastName());
        Number newId = insertActor.executeAndReturnKey(parameters);
        actor.setId(newId.longValue());
    }

    // ... additional methods
}
```

插入的执行与您依靠元数据来确定要使用的列的执行相同。

#### 3.6.4. Using `SqlParameterSource` to Provide Parameter Values使用`SqlParameterSource`提供参数值

使用`Map`提供参数值可以很好地工作，但这不是最方便使用的类。Spring提供了一些`SqlParameterSource` 接口实现，您可以代替使用它们。第一个是`BeanPropertySqlParameterSource`，如果您有一个包含值的JavaBean兼容类，这是一个非常方便的类。它使用相应的getter方法提取参数值。以下示例显示如何使用`BeanPropertySqlParameterSource`：

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource)
                .withTableName("t_actor")
                .usingGeneratedKeyColumns("id");
    }

    public void add(Actor actor) {
        SqlParameterSource parameters = new BeanPropertySqlParameterSource(actor);
        Number newId = insertActor.executeAndReturnKey(parameters);
        actor.setId(newId.longValue());
    }

    // ... additional methods
}
```

另一个选项`MapSqlParameterSource`类似于，`Map`但提供了`addValue`可以链接的更方便的方法。以下示例显示了如何使用它：

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource)
                .withTableName("t_actor")
                .usingGeneratedKeyColumns("id");
    }

    public void add(Actor actor) {
        SqlParameterSource parameters = new MapSqlParameterSource()
                .addValue("first_name", actor.getFirstName())
                .addValue("last_name", actor.getLastName());
        Number newId = insertActor.executeAndReturnKey(parameters);
        actor.setId(newId.longValue());
    }

    // ... additional methods
}
```

如您所见，配置是相同的。只有执行代码才能更改为使用这些替代输入类。

#### 3.6.5. Calling a Stored Procedure with `SimpleJdbcCall`使用以下命令调用存储过程`SimpleJdbcCall`

该`SimpleJdbcCall`数据库中的类使用元数据查找的名称`in` 和`out`参数，使您不必明确声明他们。如果愿意，可以声明参数，也可以声明没有自动映射到Java类的参数（例如`ARRAY` 或`STRUCT`）。第一个示例显示了一个简单过程，该过程仅返回MySQL数据库中标量值`VARCHAR`和`DATE`格式的标量值。该示例过程读取指定的演员项并返回 `first_name`，`last_name`以及`birth_date`在形式列`out`参数。以下清单显示了第一个示例：

```sql
CREATE PROCEDURE read_actor (
    IN in_id INTEGER,
    OUT out_first_name VARCHAR(100),
    OUT out_last_name VARCHAR(100),
    OUT out_birth_date DATE)
BEGIN
    SELECT first_name, last_name, birth_date
    INTO out_first_name, out_last_name, out_birth_date
    FROM t_actor where id = in_id;
END;
```

该`in_id`参数包含`id`您要查找的参与者的。该`out` 参数返回从表中读取数据。

您可以`SimpleJdbcCall`采用类似于声明的方式进行声明`SimpleJdbcInsert`。您应该在数据访问层的初始化方法中实例化并配置该类。与`StoredProcedure`该类相比，您无需创建子类，也无需声明可以在数据库元数据中查找的参数。下面的`SimpleJdbcCall`配置示例使用前面的存储过程（除了之外，唯一的配置选项`DataSource`是存储过程的名称）：

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadActor;

    public void setDataSource(DataSource dataSource) {
        this.procReadActor = new SimpleJdbcCall(dataSource)
                .withProcedureName("read_actor");
    }

    public Actor readActor(Long id) {
        SqlParameterSource in = new MapSqlParameterSource()
                .addValue("in_id", id);
        Map out = procReadActor.execute(in);
        Actor actor = new Actor();
        actor.setId(id);
        actor.setFirstName((String) out.get("out_first_name"));
        actor.setLastName((String) out.get("out_last_name"));
        actor.setBirthDate((Date) out.get("out_birth_date"));
        return actor;
    }

    // ... additional methods
}
```

您为执行调用编写的代码涉及创建一个`SqlParameterSource` 包含IN参数的代码。您必须为输入值提供的名称与存储过程中声明的参数名称的名称匹配。大小写不必匹配，因为您使用元数据来确定在存储过程中应如何引用数据库对象。源中为存储过程指定的内容不一定是存储过程在数据库中存储的方式。一些数据库将名称转换为全部大写，而另一些数据库使用小写或指定的大小写。

该`execute`方法采用IN参数，并返回`Map`，其中包含`out` 存储过程中指定的由名称键入的任何参数。在这种情况下，他们是 `out_first_name`，`out_last_name`和`out_birth_date`。

该`execute`方法的最后一部分创建一个`Actor`实例，用于返回检索到的数据。同样，使用`out`在存储过程中声明的参数名称也很重要。同样，`out` 结果映射中存储的参数名称的大小写`out`与数据库中参数名称的大小写匹配，这在数据库之间可能会有所不同。为了使代码更具可移植性，您应该执行不区分大小写的查找或指示Spring使用`LinkedCaseInsensitiveMap`。为此，您可以创建自己的`JdbcTemplate`并将`setResultsMapCaseInsensitive` 属性设置为`true`。然后，您可以将此自定义`JdbcTemplate`实例传递到的构造函数中`SimpleJdbcCall`。以下示例显示了此配置：

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadActor;

    public void setDataSource(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.procReadActor = new SimpleJdbcCall(jdbcTemplate)
                .withProcedureName("read_actor");
    }

    // ... additional methods
}
```

通过执行此操作，可以避免在用于返回`out`参数名称的情况下发生冲突。

#### 3.6.6. Explicitly Declaring Parameters to Use for a `SimpleJdbcCall`明确声明要用于的参数`SimpleJdbcCall`

在本章的前面，我们描述了如何从元数据推导参数，但是如果需要，可以显式声明它们。您可以通过`SimpleJdbcCall`使用`declareParameters`方法创建和配置，该方法将可变数量的`SqlParameter`对象作为输入。有关如何定义的详细信息，请参见[下一部分](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-params)`SqlParameter`。

![1609491343700](assets/1609491343700.png)

您可以选择显式声明一个，一些或所有参数。在未显式声明参数的地方，仍使用参数元数据。要绕过对潜在参数的元数据查找的所有处理，并且仅使用声明的参数，可以将方法`withoutProcedureColumnMetaDataAccess`作为声明的一部分进行调用。假设您为数据库函数声明了两个或多个不同的调用签名。在这种情况下，您调用`useInParameterNames`以指定要包含在给定签名中的IN参数名称列表。

下面的示例显示一个完全声明的过程调用，并使用前面示例中的信息：

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadActor;

    public void setDataSource(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.procReadActor = new SimpleJdbcCall(jdbcTemplate)
                .withProcedureName("read_actor")
                .withoutProcedureColumnMetaDataAccess()
                .useInParameterNames("in_id")
                .declareParameters(
                        new SqlParameter("in_id", Types.NUMERIC),
                        new SqlOutParameter("out_first_name", Types.VARCHAR),
                        new SqlOutParameter("out_last_name", Types.VARCHAR),
                        new SqlOutParameter("out_birth_date", Types.DATE)
                );
    }

    // ... additional methods
}
```

两个示例的执行和最终结果相同。第二个示例明确指定所有详细信息，而不是依赖于元数据。

#### 3.6.7. How to Define `SqlParameters`如何定义`SqlParameters`

要为这些`SimpleJdbc`类以及RDBMS操作类（在将[JDBC操作建模为Java对象中发现](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-object)）定义参数，可以使用`SqlParameter`或其子类之一。为此，通常在构造函数中指定参数名称和SQL类型。通过使用`java.sql.Types`常量来指定SQL类型。在本章的前面，我们看到了类似于以下内容的声明：

```java
new SqlParameter("in_id", Types.NUMERIC),
new SqlOutParameter("out_first_name", Types.VARCHAR),
```

第一行带有`SqlParameter`一个IN参数。您可以使用IN参数`SqlQuery`及其子类（在[理解中找到`SqlQuery`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SqlQuery)）将IN参数用于存储过程调用和查询。

第二行（带有`SqlOutParameter`）声明`out`要在存储过程调用中使用的参数。还有一个`SqlInOutParameter`for`InOut`参数（为过程提供IN值并返回值的参数）。

![1609491412860](assets/1609491412860.png)

对于IN参数，除了名称和SQL类型，还可以为数字数据指定小数位，或者为自定义数据库类型指定类型名。对于`out`参数，您可以提供一个`RowMapper`处理从`REF`游标返回的行的映射。另一种选择是指定一个`SqlReturnType`，它提供了机会来定义返回值的自定义处理。

####  3.6.8. Calling a Stored Function by Using `SimpleJdbcCall`通过使用调用存储的函数`SimpleJdbcCall`

可以使用与调用存储过程几乎相同的方式来调用存储函数，除了提供函数名而不是过程名。您将该 `withFunctionName`方法用作配置的一部分，以指示您要对函数进行调用，并生成函数调用的相应字符串。专门的执行调用（`executeFunction`）用于执行函数，它以指定类型的对象的形式返回函数的返回值，这意味着您不必从结果图中检索返回值。`executeObject`对于只有一个`out` 参数的存储过程，也可以使用类似的便捷方法（名为）。下面的示例（对于MySQL）基于名为的存储函数`get_actor_name` ，该函数返回参与者的全名：

```sql
CREATE FUNCTION get_actor_name (in_id INTEGER)
RETURNS VARCHAR(200) READS SQL DATA
BEGIN
    DECLARE out_name VARCHAR(200);
    SELECT concat(first_name, ' ', last_name)
        INTO out_name
        FROM t_actor where id = in_id;
    RETURN out_name;
END;
```

要调用此函数，我们再次`SimpleJdbcCall`在初始化方法中创建一个，如以下示例所示：

```java
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;
    private SimpleJdbcCall funcGetActorName;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.funcGetActorName = new SimpleJdbcCall(jdbcTemplate)
                .withFunctionName("get_actor_name");
    }

    public String getActorName(Long id) {
        SqlParameterSource in = new MapSqlParameterSource()
                .addValue("in_id", id);
        String name = funcGetActorName.executeFunction(String.class, in);
        return name;
    }

    // ... additional methods
}
```

所`executeFunction`使用的方法返回`String`，其中包含函数调用的返回值。

#### 3.6.9. Returning a `ResultSet` or REF Cursor from a `SimpleJdbcCall``ResultSet`从a返回或REF光标`SimpleJdbcCall`

调用返回结果集的存储过程或函数有点棘手。一些数据库在JDBC结果处理期间返回结果集，而另一些数据库则需要显式注册`out`的特定类型的参数。两种方法都需要进行额外的处理才能遍历结果集并处理返回的行。使用`SimpleJdbcCall`，您可以使用`returningResultSet`方法并声明`RowMapper` 要用于特定参数的实现。如果在结果处理过程中返回了结果集，则没有定义名称，因此返回的结果必须与声明`RowMapper` 实现的顺序匹配。指定的名称仍用于将处理后的结果列表存储在从`execute`语句返回的结果图中。

下一个示例（对于MySQL）使用存储过程，该存储过程不使用IN参数，并返回`t_actor`表中的所有行：

```sql
CREATE PROCEDURE read_all_actors()
BEGIN
 SELECT a.id, a.first_name, a.last_name, a.birth_date FROM t_actor a;
END;
```

要调用此过程，可以声明`RowMapper`。因为要映射到的类遵循JavaBean规则，所以可以使用`BeanPropertyRowMapper`通过在`newInstance`方法中传入要映射的必需类而创建的。以下示例显示了如何执行此操作：

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadAllActors;

    public void setDataSource(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.procReadAllActors = new SimpleJdbcCall(jdbcTemplate)
                .withProcedureName("read_all_actors")
                .returningResultSet("actors",
                BeanPropertyRowMapper.newInstance(Actor.class));
    }

    public List getActorsList() {
        Map m = procReadAllActors.execute(new HashMap<String, Object>(0));
        return (List) m.get("actors");
    }

    // ... additional methods
}
```

该`execute`调用传入一个空值`Map`，因为此调用没有任何参数。然后从结果图中检索参与者列表，并将其返回给调用者。

### 3.7. Modeling JDBC Operations as Java Objects将JDBC操作建模为Java对象 

该`org.springframework.jdbc.object`软件包包含一些类，这些类使您可以以更加面向对象的方式访问数据库。例如，您可以执行查询并以包含业务对象的列表的形式返回结果，该业务对象的关系列数据映射到业务对象的属性。您还可以运行存储过程并运行update，delete和insert语句。

![1609491552840](assets/1609491552840.png)

#### 3.7.1. Understanding `SqlQuery`理解`SqlQuery`

`SqlQuery`是封装SQL查询的可重用，线程安全的类。子类必须实现该`newRowMapper(..)`方法以提供一个`RowMapper`实例，该实例可以通过`ResultSet`对查询执行期间创建的对象进行迭代而获得的每行创建一个对象。在`SqlQuery`类很少直接使用，因为`MappingSqlQuery`子类提供映射行Java类的更方便的实现。扩展的其他实现`SqlQuery`是 `MappingSqlQueryWithParameters`和`UpdatableSqlQuery`。

#### 3.7.2. Using `MappingSqlQuery`使用`MappingSqlQuery`

`MappingSqlQuery`是可重用的查询，其中具体的子类必须实现抽象`mapRow(..)`方法才能将提供的每一行转换`ResultSet`为指定类型的对象。下面的示例显示了一个自定义查询，该查询将数据从`t_actor`关系映射到`Actor`类的实例：

```java
public class ActorMappingQuery extends MappingSqlQuery<Actor> {

    public ActorMappingQuery(DataSource ds) {
        super(ds, "select id, first_name, last_name from t_actor where id = ?");
        declareParameter(new SqlParameter("id", Types.INTEGER));
        compile();
    }

    @Override
    protected Actor mapRow(ResultSet rs, int rowNumber) throws SQLException {
        Actor actor = new Actor();
        actor.setId(rs.getLong("id"));
        actor.setFirstName(rs.getString("first_name"));
        actor.setLastName(rs.getString("last_name"));
        return actor;
    }
}
```

该类`MappingSqlQuery`使用`Actor`类型扩展了参数化。此客户查询的构造函数将a`DataSource`作为唯一参数。在此构造函数中，您可以使用`DataSource`和应该调用SQL来检索此查询的行的超类上的构造函数。该SQL用于创建`PreparedStatement`，因此它可能包含执行期间要传递的任何参数的占位符。您必须使用`declareParameter` 传入的方法声明每个参数`SqlParameter`。的`SqlParameter`名称和定义的JDBC类型相同`java.sql.Types`。定义所有参数后，您可以调用 `compile()`方法，以便可以准备语句并在以后运行。此类在编译后是线程安全的，因此，只要在初始化DAO时创建这些实例，就可以将它们保留为实例变量并可以重用。下面的示例演示如何定义此类：

```java
private ActorMappingQuery actorMappingQuery;

@Autowired
public void setDataSource(DataSource dataSource) {
    this.actorMappingQuery = new ActorMappingQuery(dataSource);
}

public Customer getCustomer(Long id) {
    return actorMappingQuery.findObject(id);
}
```

前面示例中的方法检索以`id`传入的客户作为唯一参数的客户。由于我们只希望返回一个对象，因此我们以参数as为参数来调用`findObject`便捷方法`id`。相反，如果有一个查询返回一个对象列表并采用其他参数，则将使用一种`execute` 采用以varargs形式传递参数值数组的方法。以下示例显示了这种方法：

```java
public List<Actor> searchForActors(int age, String namePattern) {
    List<Actor> actors = actorSearchMappingQuery.execute(age, namePattern);
    return actors;
}
```

####  3.7.3. Using `SqlUpdate`使用`SqlUpdate`

本`SqlUpdate`类封装了一个SQL更新。与查询一样，更新对象是可重用的，并且与所有`RdbmsOperation`类一样，更新可以具有参数并在SQL中定义。此类提供了许多`update(..)`类似于 `execute(..)`查询对象方法的方法。该`SQLUpdate`班是具体的。可以将其子类化-例如，添加自定义更新方法。但是，不必类的子`SqlUpdate` 类，因为可以通过设置SQL和声明参数来轻松地对其进行参数化。以下示例创建一个名为的自定义更新方法`execute`：

```java
import java.sql.Types;
import javax.sql.DataSource;
import org.springframework.jdbc.core.SqlParameter;
import org.springframework.jdbc.object.SqlUpdate;

public class UpdateCreditRating extends SqlUpdate {

    public UpdateCreditRating(DataSource ds) {
        setDataSource(ds);
        setSql("update customer set credit_rating = ? where id = ?");
        declareParameter(new SqlParameter("creditRating", Types.NUMERIC));
        declareParameter(new SqlParameter("id", Types.NUMERIC));
        compile();
    }

    /**
     * @param id for the Customer to be updated
     * @param rating the new value for credit rating
     * @return number of rows updated
     */
    public int execute(int id, int rating) {
        return update(rating, id);
    }
}
```

#### 3.7.4. Using `StoredProcedure`使用`StoredProcedure`

的`StoredProcedure`类是RDBMS存储过程的对象的抽象类的超类。此类为`abstract`，其各种`execute(..)`方法都`protected`可以访问，除了通过提供更严格类型的子类以外，其他方法 无法使用。

继承的`sql`属性是RDBMS中存储过程的名称。

要为`StoredProcedure`该类定义一个参数，可以使用一个`SqlParameter`或一个子类。您必须在构造函数中指定参数名称和SQL类型，如以下代码片段所示：

```java
new SqlParameter("in_id", Types.NUMERIC),
new SqlOutParameter("out_first_name", Types.VARCHAR),
```

SQL类型是使用`java.sql.Types`常量指定的。

第一行（带有`SqlParameter`）声明一个IN参数。您可以将IN参数用于存储过程调用以及使用`SqlQuery`和及其子类（在[理解`SqlQuery`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SqlQuery)中[了解](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SqlQuery)）进行查询。

第二行（带有`SqlOutParameter`）声明`out`要在存储过程调用中使用的参数。还有一个`SqlInOutParameter`for`InOut`参数（为`in`过程提供值并返回值的参数）。

对于`in`参数，除了名称和SQL类型外，还可以为数字数据指定小数位，或者为自定义数据库类型指定类型名。对于`out`参数，您可以提供一个`RowMapper`处理从`REF`游标返回的行的映射。另一种选择是指定一个`SqlReturnType`，它允许您定义返回值的自定义处理。

下一个简单DAO的示例使用a`StoredProcedure`来调用`sysdate()`任何Oracle数据库附带的函数（）。要使用存储过程功能，您必须创建一个extends类`StoredProcedure`。在此示例中，`StoredProcedure`该类是内部类。但是，如果您需要重用 `StoredProcedure`，则可以将其声明为顶级类。本示例没有输入参数，但是使用`SqlOutParameter`该类将输出参数声明为日期类型 。该`execute()`方法运行过程并从结果中提取返回的日期`Map`。`Map`通过使用参数名称作为键，结果为每个声明的输出参数（在本例中为一个）都有一个条目。以下清单显示了我们的自定义StoredProcedure类：

```java
import java.sql.Types;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import javax.sql.DataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.SqlOutParameter;
import org.springframework.jdbc.object.StoredProcedure;

public class StoredProcedureDao {

    private GetSysdateProcedure getSysdate;

    @Autowired
    public void init(DataSource dataSource) {
        this.getSysdate = new GetSysdateProcedure(dataSource);
    }

    public Date getSysdate() {
        return getSysdate.execute();
    }

    private class GetSysdateProcedure extends StoredProcedure {

        private static final String SQL = "sysdate";

        public GetSysdateProcedure(DataSource dataSource) {
            setDataSource(dataSource);
            setFunction(true);
            setSql(SQL);
            declareParameter(new SqlOutParameter("date", Types.DATE));
            compile();
        }

        public Date execute() {
            // the 'sysdate' sproc has no input parameters, so an empty Map is supplied...
            Map<String, Object> results = execute(new HashMap<String, Object>());
            Date sysdate = (Date) results.get("date");
            return sysdate;
        }
    }

}
```

以下示例的a`StoredProcedure`具有两个输出参数（在本例中为Oracle REF游标）：

```java
import java.util.HashMap;
import java.util.Map;
import javax.sql.DataSource;
import oracle.jdbc.OracleTypes;
import org.springframework.jdbc.core.SqlOutParameter;
import org.springframework.jdbc.object.StoredProcedure;

public class TitlesAndGenresStoredProcedure extends StoredProcedure {

    private static final String SPROC_NAME = "AllTitlesAndGenres";

    public TitlesAndGenresStoredProcedure(DataSource dataSource) {
        super(dataSource, SPROC_NAME);
        declareParameter(new SqlOutParameter("titles", OracleTypes.CURSOR, new TitleMapper()));
        declareParameter(new SqlOutParameter("genres", OracleTypes.CURSOR, new GenreMapper()));
        compile();
    }

    public Map<String, Object> execute() {
        // again, this sproc has no input parameters, so an empty Map is supplied
        return super.execute(new HashMap<String, Object>());
    }
}
```

请注意`declareParameter(..)`，在`TitlesAndGenresStoredProcedure`构造函数中使用的方法的重载变体如何传递给`RowMapper` 实现实例。这是重用现有功能的非常方便且强大的方法。接下来的两个示例提供了两种`RowMapper`实现的代码。

所述`TitleMapper`类映射一个`ResultSet`到一个`Title`用于在提供的各行的域对象`ResultSet`，如下所示：

```java
import java.sql.ResultSet;
import java.sql.SQLException;
import com.foo.domain.Title;
import org.springframework.jdbc.core.RowMapper;

public final class TitleMapper implements RowMapper<Title> {

    public Title mapRow(ResultSet rs, int rowNum) throws SQLException {
        Title title = new Title();
        title.setId(rs.getLong("id"));
        title.setName(rs.getString("name"));
        return title;
    }
}
```

所述`GenreMapper`类映射一个`ResultSet`到一个`Genre`用于在提供的各行的域对象`ResultSet`，如下所示：

```java
import java.sql.ResultSet;
import java.sql.SQLException;
import com.foo.domain.Genre;
import org.springframework.jdbc.core.RowMapper;

public final class GenreMapper implements RowMapper<Genre> {

    public Genre mapRow(ResultSet rs, int rowNum) throws SQLException {
        return new Genre(rs.getString("name"));
    }
}
```

要将参数传递给RDBMS中定义中具有一个或多个输入参数的存储过程，可以编写一个强类型`execute(..)`方法，该方法将委派给`execute(Map)`超类中的未类型方法，如以下示例所示：

```java
import java.sql.Types;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import javax.sql.DataSource;
import oracle.jdbc.OracleTypes;
import org.springframework.jdbc.core.SqlOutParameter;
import org.springframework.jdbc.core.SqlParameter;
import org.springframework.jdbc.object.StoredProcedure;

public class TitlesAfterDateStoredProcedure extends StoredProcedure {

    private static final String SPROC_NAME = "TitlesAfterDate";
    private static final String CUTOFF_DATE_PARAM = "cutoffDate";

    public TitlesAfterDateStoredProcedure(DataSource dataSource) {
        super(dataSource, SPROC_NAME);
        declareParameter(new SqlParameter(CUTOFF_DATE_PARAM, Types.DATE);
        declareParameter(new SqlOutParameter("titles", OracleTypes.CURSOR, new TitleMapper()));
        compile();
    }

    public Map<String, Object> execute(Date cutoffDate) {
        Map<String, Object> inputs = new HashMap<String, Object>();
        inputs.put(CUTOFF_DATE_PARAM, cutoffDate);
        return super.execute(inputs);
    }
}
```

### 3.8. Common Problems with Parameter and Data Value Handling 参数和数据值处理的常见问题

Spring Framework的JDBC支持提供的不同方法中存在参数和数据值的常见问题。本节介绍如何解决它们。

#### 3.8.1. Providing SQL Type Information for Parameters

通常，Spring根据传入的参数类型确定参数的SQL类型。可以在设置参数值时显式提供要使用的SQL类型。有时需要正确设置`NULL`值。

您可以通过几种方式提供SQL类型信息：

- 许多更新和查询方法`JdbcTemplate`采用`int`数组形式的附加参数。该数组用于通过使用`java.sql.Types`类中的常量值来指示相应参数的SQL类型。为每个参数提供一个条目。
- 您可以使用`SqlParameterValue`该类包装需要此附加信息的参数值。为此，请为每个值创建一个新实例，然后在构造函数中传入SQL类型和参数值。您还可以为数字值提供可选的比例参数。
- 对于使用命名参数的方法，可以使用`SqlParameterSource`类 `BeanPropertySqlParameterSource`或`MapSqlParameterSource`。它们都具有用于为任何命名参数值注册SQL类型的方法。

1. 1. 1. [ Complex Types for Stored Procedure Calls](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-complex-types)
   2. 3.9. Embedded Database Support
      1. 
      2. 
      3. 
      4. 
         1. 
         2. 
         3. 
      5. 
      6. 
      7. 
   3. 3.10. Initializing a DataSource
      1. 
         1. 
2. \4. Object Relational Mapping (ORM) Data Access
   1. 
   2. 
      1. 
      2. 
   3. 
      1. 
      2. 
      3. 
      4. 
      5. 
      6. 
      7. 
   4. 
      1. 
         1. 
         2. 
         3. 
         4. 
         5. 
      2. 
      3. 
      4. 
      5. 
      6. 
3. \5. Marshalling XML by Using Object-XML Mappers
   1. 
      1. 
      2. 
      3. 
   2. 
      1. 
      2. 
      3. 
   3. 
   4. 
   5. 
      1. 
         1. 
   6. 
      1. 
         1. 
   7. 
      1. 
4. \6. Appendix
   1. 
      1. 
      2. 

This part of the reference documentation is concerned with data access and the interaction between the data access layer and the business or service layer.

Spring’s comprehensive transaction management support is covered in some detail, followed by thorough coverage of the various data access frameworks and technologies with which the Spring Framework integrates.

## 1. Transaction Management

Comprehensive transaction support is among the most compelling reasons to use the Spring Framework. The Spring Framework provides a consistent abstraction for transaction management that delivers the following benefits:

- A consistent programming model across different transaction APIs, such as Java Transaction API (JTA), JDBC, Hibernate, and the Java Persistence API (JPA).
- Support for [declarative transaction management](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-declarative).
- A simpler API for [programmatic](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-programmatic) transaction management than complex transaction APIs, such as JTA.
- Excellent integration with Spring’s data access abstractions.

The following sections describe the Spring Framework’s transaction features and technologies:

- [Advantages of the Spring Framework’s transaction support model](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-motivation) describes why you would use the Spring Framework’s transaction abstraction instead of EJB Container-Managed Transactions (CMT) or choosing to drive local transactions through a proprietary API, such as Hibernate.
- [Understanding the Spring Framework transaction abstraction](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-strategies) outlines the core classes and describes how to configure and obtain `DataSource` instances from a variety of sources.
- [Synchronizing resources with transactions](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#tx-resource-synchronization) describes how the application code ensures that resources are created, reused, and cleaned up properly.
- [Declarative transaction management](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-declarative) describes support for declarative transaction management.
- [Programmatic transaction management](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-programmatic) covers support for programmatic (that is, explicitly coded) transaction management.
- [Transaction bound event](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-event) describes how you could use application events within a transaction.

The chapter also includes discussions of best practices, [application server integration](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-application-server-integration), and [solutions to common problems](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-solutions-to-common-problems).

### 1.1. Advantages of the Spring Framework’s Transaction Support Model

Traditionally, Java EE developers have had two choices for transaction management: global or local transactions, both of which have profound limitations. Global and local transaction management is reviewed in the next two sections, followed by a discussion of how the Spring Framework’s transaction management support addresses the limitations of the global and local transaction models.

#### 1.1.1. Global Transactions

Global transactions let you work with multiple transactional resources, typically relational databases and message queues. The application server manages global transactions through the JTA, which is a cumbersome API (partly due to its exception model). Furthermore, a JTA `UserTransaction` normally needs to be sourced from JNDI, meaning that you also need to use JNDI in order to use JTA. The use of global transactions limits any potential reuse of application code, as JTA is normally only available in an application server environment.

Previously, the preferred way to use global transactions was through EJB CMT (Container Managed Transaction). CMT is a form of declarative transaction management (as distinguished from programmatic transaction management). EJB CMT removes the need for transaction-related JNDI lookups, although the use of EJB itself necessitates the use of JNDI. It removes most but not all of the need to write Java code to control transactions. The significant downside is that CMT is tied to JTA and an application server environment. Also, it is only available if one chooses to implement business logic in EJBs (or at least behind a transactional EJB facade). The negatives of EJB in general are so great that this is not an attractive proposition, especially in the face of compelling alternatives for declarative transaction management.

#### 1.1.2. Local Transactions

Local transactions are resource-specific, such as a transaction associated with a JDBC connection. Local transactions may be easier to use but have a significant disadvantage: They cannot work across multiple transactional resources. For example, code that manages transactions by using a JDBC connection cannot run within a global JTA transaction. Because the application server is not involved in transaction management, it cannot help ensure correctness across multiple resources. (It is worth noting that most applications use a single transaction resource.) Another downside is that local transactions are invasive to the programming model.

#### 1.1.3. Spring Framework’s Consistent Programming Model

Spring resolves the disadvantages of global and local transactions. It lets application developers use a consistent programming model in any environment. You write your code once, and it can benefit from different transaction management strategies in different environments. The Spring Framework provides both declarative and programmatic transaction management. Most users prefer declarative transaction management, which we recommend in most cases.

With programmatic transaction management, developers work with the Spring Framework transaction abstraction, which can run over any underlying transaction infrastructure. With the preferred declarative model, developers typically write little or no code related to transaction management and, hence, do not depend on the Spring Framework transaction API or any other transaction API.

Do you need an application server for transaction management?

The Spring Framework’s transaction management support changes traditional rules as to when an enterprise Java application requires an application server.

In particular, you do not need an application server purely for declarative transactions through EJBs. In fact, even if your application server has powerful JTA capabilities, you may decide that the Spring Framework’s declarative transactions offer more power and a more productive programming model than EJB CMT.

Typically, you need an application server’s JTA capability only if your application needs to handle transactions across multiple resources, which is not a requirement for many applications. Many high-end applications use a single, highly scalable database (such as Oracle RAC) instead. Stand-alone transaction managers (such as [Atomikos Transactions](https://www.atomikos.com/) and [JOTM](http://jotm.objectweb.org/)) are other options. Of course, you may need other application server capabilities, such as Java Message Service (JMS) and Java EE Connector Architecture (JCA).

The Spring Framework gives you the choice of when to scale your application to a fully loaded application server. Gone are the days when the only alternative to using EJB CMT or JTA was to write code with local transactions (such as those on JDBC connections) and face a hefty rework if you need that code to run within global, container-managed transactions. With the Spring Framework, only some of the bean definitions in your configuration file need to change (rather than your code).

### 1.2. Understanding the Spring Framework Transaction Abstraction

The key to the Spring transaction abstraction is the notion of a transaction strategy. A transaction strategy is defined by the `org.springframework.transaction.PlatformTransactionManager` interface, which the following listing shows:

Java

Kotlin

```java
public interface PlatformTransactionManager {

    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

This is primarily a service provider interface (SPI), although you can use it [programmatically](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-programmatic-ptm) from your application code. Because `PlatformTransactionManager` is an interface, it can be easily mocked or stubbed as necessary. It is not tied to a lookup strategy, such as JNDI. `PlatformTransactionManager` implementations are defined like any other object (or bean) in the Spring Framework IoC container. This benefit alone makes Spring Framework transactions a worthwhile abstraction, even when you work with JTA. You can test transactional code much more easily than if it used JTA directly.

Again, in keeping with Spring’s philosophy, the `TransactionException` that can be thrown by any of the `PlatformTransactionManager` interface’s methods is unchecked (that is, it extends the `java.lang.RuntimeException` class). Transaction infrastructure failures are almost invariably fatal. In rare cases where application code can actually recover from a transaction failure, the application developer can still choose to catch and handle `TransactionException`. The salient point is that developers are not *forced* to do so.

The `getTransaction(..)` method returns a `TransactionStatus` object, depending on a `TransactionDefinition` parameter. The returned `TransactionStatus` might represent a new transaction or can represent an existing transaction, if a matching transaction exists in the current call stack. The implication in this latter case is that, as with Java EE transaction contexts, a `TransactionStatus` is associated with a thread of execution.

The `TransactionDefinition` interface specifies:

- Propagation: Typically, all code executed within a transaction scope runs in that transaction. However, you can specify the behavior if a transactional method is executed when a transaction context already exists. For example, code can continue running in the existing transaction (the common case), or the existing transaction can be suspended and a new transaction created. Spring offers all of the transaction propagation options familiar from EJB CMT. To read about the semantics of transaction propagation in Spring, see [Transaction Propagation](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#tx-propagation).
- Isolation: The degree to which this transaction is isolated from the work of other transactions. For example, can this transaction see uncommitted writes from other transactions?
- Timeout: How long this transaction runs before timing out and being automatically rolled back by the underlying transaction infrastructure.
- Read-only status: You can use a read-only transaction when your code reads but does not modify data. Read-only transactions can be a useful optimization in some cases, such as when you use Hibernate.

These settings reflect standard transactional concepts. If necessary, refer to resources that discuss transaction isolation levels and other core transaction concepts. Understanding these concepts is essential to using the Spring Framework or any transaction management solution.

The `TransactionStatus` interface provides a simple way for transactional code to control transaction execution and query transaction status. The concepts should be familiar, as they are common to all transaction APIs. The following listing shows the `TransactionStatus` interface:

Java

Kotlin

```java
public interface TransactionStatus extends SavepointManager {

    boolean isNewTransaction();

    boolean hasSavepoint();

    void setRollbackOnly();

    boolean isRollbackOnly();

    void flush();

    boolean isCompleted();
}
```

Regardless of whether you opt for declarative or programmatic transaction management in Spring, defining the correct `PlatformTransactionManager` implementation is absolutely essential. You typically define this implementation through dependency injection.

`PlatformTransactionManager` implementations normally require knowledge of the environment in which they work: JDBC, JTA, Hibernate, and so on. The following examples show how you can define a local `PlatformTransactionManager` implementation (in this case, with plain JDBC.)

You can define a JDBC `DataSource` by creating a bean similar to the following:

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
</bean>
```

The related `PlatformTransactionManager` bean definition then has a reference to the `DataSource` definition. It should resemble the following example:

```xml
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

If you use JTA in a Java EE container, then you use a container `DataSource`, obtained through JNDI, in conjunction with Spring’s `JtaTransactionManager`. The following example shows what the JTA and JNDI lookup version would look like:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/jee
        https://www.springframework.org/schema/jee/spring-jee.xsd">

    <jee:jndi-lookup id="dataSource" jndi-name="jdbc/jpetstore"/>

    <bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager" />

    <!-- other <bean/> definitions here -->

</beans>
```

The `JtaTransactionManager` does not need to know about the `DataSource` (or any other specific resources) because it uses the container’s global transaction management infrastructure.

|      | The preceding definition of the `dataSource` bean uses the `<jndi-lookup/>` tag from the `jee` namespace. For more information see [The JEE Schema](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/integration.html#xsd-schemas-jee). |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

You can also use easily Hibernate local transactions, as shown in the following examples. In this case, you need to define a Hibernate `LocalSessionFactoryBean`, which your application code can use to obtain Hibernate `Session` instances.

The `DataSource` bean definition is similar to the local JDBC example shown previously and, thus, is not shown in the following example.

|      | If the `DataSource` (used by any non-JTA transaction manager) is looked up through JNDI and managed by a Java EE container, it should be non-transactional, because the Spring Framework (rather than the Java EE container) manages the transactions. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The `txManager` bean in this case is of the `HibernateTransactionManager` type. In the same way as the `DataSourceTransactionManager` needs a reference to the `DataSource`, the `HibernateTransactionManager` needs a reference to the `SessionFactory`. The following example declares `sessionFactory` and `txManager` beans:

```xml
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="mappingResources">
        <list>
            <value>org/springframework/samples/petclinic/hibernate/petclinic.hbm.xml</value>
        </list>
    </property>
    <property name="hibernateProperties">
        <value>
            hibernate.dialect=${hibernate.dialect}
        </value>
    </property>
</bean>

<bean id="txManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory"/>
</bean>
```

If you use Hibernate and Java EE container-managed JTA transactions, you should use the same `JtaTransactionManager` as in the previous JTA example for JDBC, as the following example shows:

```xml
<bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
```

|      | If you use JTA, your transaction manager definition should look the same, regardless of what data access technology you use, be it JDBC, Hibernate JPA, or any other supported technology. This is due to the fact that JTA transactions are global transactions, which can enlist any transactional resource. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

In all these cases, application code does not need to change. You can change how transactions are managed merely by changing configuration, even if that change means moving from local to global transactions or vice versa.

### 1.3. Synchronizing Resources with Transactions

How to create different transaction managers and how they are linked to related resources that need to be synchronized to transactions (for example `DataSourceTransactionManager` to a JDBC `DataSource`, `HibernateTransactionManager` to a Hibernate `SessionFactory`, and so forth) should now be clear. This section describes how the application code (directly or indirectly, by using a persistence API such as JDBC, Hibernate, or JPA) ensures that these resources are created, reused, and cleaned up properly. The section also discusses how transaction synchronization is (optionally) triggered through the relevant `PlatformTransactionManager`.

#### 1.3.1. High-level Synchronization Approach

The preferred approach is to use Spring’s highest-level template based persistence integration APIs or to use native ORM APIs with transaction-aware factory beans or proxies for managing the native resource factories. These transaction-aware solutions internally handle resource creation and reuse, cleanup, optional transaction synchronization of the resources, and exception mapping. Thus, user data access code does not have to address these tasks but can focus purely on non-boilerplate persistence logic. Generally, you use the native ORM API or take a template approach for JDBC access by using the `JdbcTemplate`. These solutions are detailed in subsequent chapters of this reference documentation.

#### 1.3.2. Low-level Synchronization Approach

Classes such as `DataSourceUtils` (for JDBC), `EntityManagerFactoryUtils` (for JPA), `SessionFactoryUtils` (for Hibernate), and so on exist at a lower level. When you want the application code to deal directly with the resource types of the native persistence APIs, you use these classes to ensure that proper Spring Framework-managed instances are obtained, transactions are (optionally) synchronized, and exceptions that occur in the process are properly mapped to a consistent API.

For example, in the case of JDBC, instead of the traditional JDBC approach of calling the `getConnection()` method on the `DataSource`, you can instead use Spring’s `org.springframework.jdbc.datasource.DataSourceUtils` class, as follows:

```java
Connection conn = DataSourceUtils.getConnection(dataSource);
```

If an existing transaction already has a connection synchronized (linked) to it, that instance is returned. Otherwise, the method call triggers the creation of a new connection, which is (optionally) synchronized to any existing transaction and made available for subsequent reuse in that same transaction. As mentioned earlier, any `SQLException` is wrapped in a Spring Framework `CannotGetJdbcConnectionException`, one of the Spring Framework’s hierarchy of unchecked `DataAccessException` types. This approach gives you more information than can be obtained easily from the `SQLException` and ensures portability across databases and even across different persistence technologies.

This approach also works without Spring transaction management (transaction synchronization is optional), so you can use it whether or not you use Spring for transaction management.

Of course, once you have used Spring’s JDBC support, JPA support, or Hibernate support, you generally prefer not to use `DataSourceUtils` or the other helper classes, because you are much happier working through the Spring abstraction than directly with the relevant APIs. For example, if you use the Spring `JdbcTemplate` or `jdbc.object` package to simplify your use of JDBC, correct connection retrieval occurs behind the scenes and you need not write any special code.

#### 1.3.3. `TransactionAwareDataSourceProxy`

At the very lowest level exists the `TransactionAwareDataSourceProxy` class. This is a proxy for a target `DataSource`, which wraps the target `DataSource` to add awareness of Spring-managed transactions. In this respect, it is similar to a transactional JNDI `DataSource`, as provided by a Java EE server.

You should almost never need or want to use this class, except when existing code must be called and passed a standard JDBC `DataSource` interface implementation. In that case, it is possible that this code is usable but is participating in Spring-managed transactions. You can write your new code by using the higher-level abstractions mentioned earlier.

### 1.4. Declarative transaction management

|      | Most Spring Framework users choose declarative transaction management. This option has the least impact on application code and, hence, is most consistent with the ideals of a non-invasive lightweight container. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The Spring Framework’s declarative transaction management is made possible with Spring aspect-oriented programming (AOP). However, as the transactional aspects code comes with the Spring Framework distribution and may be used in a boilerplate fashion, AOP concepts do not generally have to be understood to make effective use of this code.

The Spring Framework’s declarative transaction management is similar to EJB CMT, in that you can specify transaction behavior (or lack of it) down to the individual method level. You can make a `setRollbackOnly()` call within a transaction context, if necessary. The differences between the two types of transaction management are:

- Unlike EJB CMT, which is tied to JTA, the Spring Framework’s declarative transaction management works in any environment. It can work with JTA transactions or local transactions by using JDBC, JPA, or Hibernate by adjusting the configuration files.
- You can apply the Spring Framework declarative transaction management to any class, not merely special classes such as EJBs.
- The Spring Framework offers declarative [rollback rules](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-declarative-rolling-back), a feature with no EJB equivalent. Both programmatic and declarative support for rollback rules is provided.
- The Spring Framework lets you customize transactional behavior by using AOP. For example, you can insert custom behavior in the case of transaction rollback. You can also add arbitrary advice, along with transactional advice. With EJB CMT, you cannot influence the container’s transaction management, except with `setRollbackOnly()`.
- The Spring Framework does not support propagation of transaction contexts across remote calls, as high-end application servers do. If you need this feature, we recommend that you use EJB. However, consider carefully before using such a feature, because, normally, one does not want transactions to span remote calls.

The concept of rollback rules is important. They let you specify which exceptions (and throwables) should cause automatic rollback. You can specify this declaratively, in configuration, not in Java code. So, although you can still call `setRollbackOnly()` on the `TransactionStatus` object to roll back the current transaction back, most often you can specify a rule that `MyApplicationException` must always result in rollback. The significant advantage to this option is that business objects do not depend on the transaction infrastructure. For example, they typically do not need to import Spring transaction APIs or other Spring APIs.

Although EJB container default behavior automatically rolls back the transaction on a system exception (usually a runtime exception), EJB CMT does not roll back the transaction automatically on an application exception (that is, a checked exception other than `java.rmi.RemoteException`). While the Spring default behavior for declarative transaction management follows EJB convention (roll back is automatic only on unchecked exceptions), it is often useful to customize this behavior.

#### 1.4.1. Understanding the Spring Framework’s Declarative Transaction Implementation

It is not sufficient merely to tell you to annotate your classes with the `@Transactional` annotation, add `@EnableTransactionManagement` to your configuration, and expect you to understand how it all works. To provide a deeper understanding, this section explains the inner workings of the Spring Framework’s declarative transaction infrastructure in the event of transaction-related issues.

The most important concepts to grasp with regard to the Spring Framework’s declarative transaction support are that this support is enabled [via AOP proxies](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop-understanding-aop-proxies) and that the transactional advice is driven by metadata (currently XML- or annotation-based). The combination of AOP with transactional metadata yields an AOP proxy that uses a `TransactionInterceptor` in conjunction with an appropriate `PlatformTransactionManager` implementation to drive transactions around method invocations.

|      | Spring AOP is covered in [the AOP section](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop). |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The following images shows a Conceptual view of calling a method on a transactional proxy:

![tx](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/images/tx.png)

#### 1.4.2. Example of Declarative Transaction Implementation

Consider the following interface and its attendant implementation. This example uses `Foo` and `Bar` classes as placeholders so that you can concentrate on the transaction usage without focusing on a particular domain model. For the purposes of this example, the fact that the `DefaultFooService` class throws `UnsupportedOperationException` instances in the body of each implemented method is good. That behavior lets you see transactions be created and then rolled back in response to the `UnsupportedOperationException` instance. The following listing shows the `FooService` interface:

Java

Kotlin

```java
// the service interface that we want to make transactional

package x.y.service;

public interface FooService {

    Foo getFoo(String fooName);

    Foo getFoo(String fooName, String barName);

    void insertFoo(Foo foo);

    void updateFoo(Foo foo);

}
```

The following example shows an implementation of the preceding interface:

Java

Kotlin

```java
package x.y.service;

public class DefaultFooService implements FooService {

    @Override
    public Foo getFoo(String fooName) {
        // ...
    }

    @Override
    public Foo getFoo(String fooName, String barName) {
        // ...
    }

    @Override
    public void insertFoo(Foo foo) {
        // ...
    }

    @Override
    public void updateFoo(Foo foo) {
        // ...
    }
}
```

Assume that the first two methods of the `FooService` interface, `getFoo(String)` and `getFoo(String, String)`, must execute in the context of a transaction with read-only semantics, and that the other methods, `insertFoo(Foo)` and `updateFoo(Foo)`, must execute in the context of a transaction with read-write semantics. The following configuration is explained in detail in the next few paragraphs:

```xml
<!-- from the file 'context.xml' -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- this is the service object that we want to make transactional -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>

    <!-- the transactional advice (what 'happens'; see the <aop:advisor/> bean below) -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <!-- the transactional semantics... -->
        <tx:attributes>
            <!-- all methods starting with 'get' are read-only -->
            <tx:method name="get*" read-only="true"/>
            <!-- other methods use the default transaction settings (see below) -->
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <!-- ensure that the above transactional advice runs for any execution
        of an operation defined by the FooService interface -->
    <aop:config>
        <aop:pointcut id="fooServiceOperation" expression="execution(* x.y.service.FooService.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceOperation"/>
    </aop:config>

    <!-- don't forget the DataSource -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
        <property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
        <property name="username" value="scott"/>
        <property name="password" value="tiger"/>
    </bean>

    <!-- similarly, don't forget the PlatformTransactionManager -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- other <bean/> definitions here -->

</beans>
```

Examine the preceding configuration. It assumes that you want to make a service object, the `fooService` bean, transactional. The transaction semantics to apply are encapsulated in the `<tx:advice/>` definition. The `<tx:advice/>` definition reads as “all methods, on starting with `get`, are to execute in the context of a read-only transaction, and all other methods are to execute with the default transaction semantics”. The `transaction-manager` attribute of the `<tx:advice/>` tag is set to the name of the `PlatformTransactionManager` bean that is going to drive the transactions (in this case, the `txManager` bean).

|      | You can omit the `transaction-manager` attribute in the transactional advice (`<tx:advice/>`) if the bean name of the `PlatformTransactionManager` that you want to wire in has the name `transactionManager`. If the `PlatformTransactionManager` bean that you want to wire in has any other name, you must use the `transaction-manager` attribute explicitly, as in the preceding example. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The `<aop:config/>` definition ensures that the transactional advice defined by the `txAdvice` bean executes at the appropriate points in the program. First, you define a pointcut that matches the execution of any operation defined in the `FooService` interface (`fooServiceOperation`). Then you associate the pointcut with the `txAdvice` by using an advisor. The result indicates that, at the execution of a `fooServiceOperation`, the advice defined by `txAdvice` is run.

The expression defined within the `<aop:pointcut/>` element is an AspectJ pointcut expression. See [the AOP section](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop) for more details on pointcut expressions in Spring.

A common requirement is to make an entire service layer transactional. The best way to do this is to change the pointcut expression to match any operation in your service layer. The following example shows how to do so:

```xml
<aop:config>
    <aop:pointcut id="fooServiceMethods" expression="execution(* x.y.service.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceMethods"/>
</aop:config>
```

|      | In the preceding example, it is assumed that all your service interfaces are defined in the `x.y.service` package. See [the AOP section](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop) for more details. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Now that we have analyzed the configuration, you may be asking yourself, “What does all this configuration actually do?”

The configuration shown earlier is used to create a transactional proxy around the object that is created from the `fooService` bean definition. The proxy is configured with the transactional advice so that, when an appropriate method is invoked on the proxy, a transaction is started, suspended, marked as read-only, and so on, depending on the transaction configuration associated with that method. Consider the following program that test drives the configuration shown earlier:

Java

Kotlin

```java
public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("context.xml", Boot.class);
        FooService fooService = (FooService) ctx.getBean("fooService");
        fooService.insertFoo (new Foo());
    }
}
```

The output from running the preceding program should resemble the following (the Log4J output and the stack trace from the UnsupportedOperationException thrown by the insertFoo(..) method of the DefaultFooService class have been truncated for clarity):

```xml
<!-- the Spring container is starting up... -->
[AspectJInvocationContextExposingAdvisorAutoProxyCreator] - Creating implicit proxy for bean 'fooService' with 0 common interceptors and 1 specific interceptors

<!-- the DefaultFooService is actually proxied -->
[JdkDynamicAopProxy] - Creating JDK dynamic proxy for [x.y.service.DefaultFooService]

<!-- ... the insertFoo(..) method is now being invoked on the proxy -->
[TransactionInterceptor] - Getting transaction for x.y.service.FooService.insertFoo

<!-- the transactional advice kicks in here... -->
[DataSourceTransactionManager] - Creating new transaction with name [x.y.service.FooService.insertFoo]
[DataSourceTransactionManager] - Acquired Connection [org.apache.commons.dbcp.PoolableConnection@a53de4] for JDBC transaction

<!-- the insertFoo(..) method from DefaultFooService throws an exception... -->
[RuleBasedTransactionAttribute] - Applying rules to determine whether transaction should rollback on java.lang.UnsupportedOperationException
[TransactionInterceptor] - Invoking rollback for transaction on x.y.service.FooService.insertFoo due to throwable [java.lang.UnsupportedOperationException]

<!-- and the transaction is rolled back (by default, RuntimeException instances cause rollback) -->
[DataSourceTransactionManager] - Rolling back JDBC transaction on Connection [org.apache.commons.dbcp.PoolableConnection@a53de4]
[DataSourceTransactionManager] - Releasing JDBC Connection after transaction
[DataSourceUtils] - Returning JDBC Connection to DataSource

Exception in thread "main" java.lang.UnsupportedOperationException at x.y.service.DefaultFooService.insertFoo(DefaultFooService.java:14)
<!-- AOP infrastructure stack trace elements removed for clarity -->
at $Proxy0.insertFoo(Unknown Source)
at Boot.main(Boot.java:11)
```

#### 1.4.3. Rolling Back a Declarative Transaction

The previous section outlined the basics of how to specify transactional settings for classes, typically service layer classes, declaratively in your application. This section describes how you can control the rollback of transactions in a simple, declarative fashion.

The recommended way to indicate to the Spring Framework’s transaction infrastructure that a transaction’s work is to be rolled back is to throw an `Exception` from code that is currently executing in the context of a transaction. The Spring Framework’s transaction infrastructure code catches any unhandled `Exception` as it bubbles up the call stack and makes a determination whether to mark the transaction for rollback.

In its default configuration, the Spring Framework’s transaction infrastructure code marks a transaction for rollback only in the case of runtime, unchecked exceptions. That is, when the thrown exception is an instance or subclass of `RuntimeException`. ( `Error` instances also, by default, result in a rollback). Checked exceptions that are thrown from a transactional method do not result in rollback in the default configuration.

You can configure exactly which `Exception` types mark a transaction for rollback, including checked exceptions. The following XML snippet demonstrates how you configure rollback for a checked, application-specific `Exception` type:

```xml
<tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
    <tx:method name="get*" read-only="true" rollback-for="NoProductInStockException"/>
    <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
```

If you do not want a transaction rolled back when an exception is thrown, you can also specify 'no rollback rules'. The following example tells the Spring Framework’s transaction infrastructure to commit the attendant transaction even in the face of an unhandled `InstrumentNotFoundException`:

```xml
<tx:advice id="txAdvice">
    <tx:attributes>
    <tx:method name="updateStock" no-rollback-for="InstrumentNotFoundException"/>
    <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
```

When the Spring Framework’s transaction infrastructure catches an exception and it consults the configured rollback rules to determine whether to mark the transaction for rollback, the strongest matching rule wins. So, in the case of the following configuration, any exception other than an `InstrumentNotFoundException` results in a rollback of the attendant transaction:

```xml
<tx:advice id="txAdvice">
    <tx:attributes>
    <tx:method name="*" rollback-for="Throwable" no-rollback-for="InstrumentNotFoundException"/>
    </tx:attributes>
</tx:advice>
```

You can also indicate a required rollback programmatically. Although simple, this process is quite invasive and tightly couples your code to the Spring Framework’s transaction infrastructure. The following example shows how to programmatically indicate a required rollback:

Java

Kotlin

```java
public void resolvePosition() {
    try {
        // some business logic...
    } catch (NoProductInStockException ex) {
        // trigger rollback programmatically
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}
```

You are strongly encouraged to use the declarative approach to rollback, if at all possible. Programmatic rollback is available should you absolutely need it, but its usage flies in the face of achieving a clean POJO-based architecture.

#### 1.4.4. Configuring Different Transactional Semantics for Different Beans

Consider the scenario where you have a number of service layer objects, and you want to apply a totally different transactional configuration to each of them. You can do so by defining distinct `<aop:advisor/>` elements with differing `pointcut` and `advice-ref` attribute values.

As a point of comparison, first assume that all of your service layer classes are defined in a root `x.y.service` package. To make all beans that are instances of classes defined in that package (or in subpackages) and that have names ending in `Service` have the default transactional configuration, you could write the following:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <aop:config>

        <aop:pointcut id="serviceOperation"
                expression="execution(* x.y.service..*Service.*(..))"/>

        <aop:advisor pointcut-ref="serviceOperation" advice-ref="txAdvice"/>

    </aop:config>

    <!-- these two beans will be transactional... -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>
    <bean id="barService" class="x.y.service.extras.SimpleBarService"/>

    <!-- ... and these two beans won't -->
    <bean id="anotherService" class="org.xyz.SomeService"/> <!-- (not in the right package) -->
    <bean id="barManager" class="x.y.service.SimpleBarManager"/> <!-- (doesn't end in 'Service') -->

    <tx:advice id="txAdvice">
        <tx:attributes>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <!-- other transaction infrastructure beans such as a PlatformTransactionManager omitted... -->

</beans>
```

The following example shows how to configure two distinct beans with totally different transactional settings:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <aop:config>

        <aop:pointcut id="defaultServiceOperation"
                expression="execution(* x.y.service.*Service.*(..))"/>

        <aop:pointcut id="noTxServiceOperation"
                expression="execution(* x.y.service.ddl.DefaultDdlManager.*(..))"/>

        <aop:advisor pointcut-ref="defaultServiceOperation" advice-ref="defaultTxAdvice"/>

        <aop:advisor pointcut-ref="noTxServiceOperation" advice-ref="noTxAdvice"/>

    </aop:config>

    <!-- this bean will be transactional (see the 'defaultServiceOperation' pointcut) -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>

    <!-- this bean will also be transactional, but with totally different transactional settings -->
    <bean id="anotherFooService" class="x.y.service.ddl.DefaultDdlManager"/>

    <tx:advice id="defaultTxAdvice">
        <tx:attributes>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <tx:advice id="noTxAdvice">
        <tx:attributes>
            <tx:method name="*" propagation="NEVER"/>
        </tx:attributes>
    </tx:advice>

    <!-- other transaction infrastructure beans such as a PlatformTransactionManager omitted... -->

</beans>
```

#### 1.4.5. <tx:advice/> Settings

This section summarizes the various transactional settings that you can specify by using the `<tx:advice/>` tag. The default `<tx:advice/>` settings are:

- The [propagation setting](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#tx-propagation) is `REQUIRED.`
- The isolation level is `DEFAULT.`
- The transaction is read-write.
- The transaction timeout defaults to the default timeout of the underlying transaction system or none if timeouts are not supported.
- Any `RuntimeException` triggers rollback, and any checked `Exception` does not.

You can change these default settings. The following table summarizes the various attributes of the `<tx:method/>` tags that are nested within `<tx:advice/>` and `<tx:attributes/>` tags:

| Attribute         | Required? | Default    | Description                                                  |
| :---------------- | :-------- | :--------- | :----------------------------------------------------------- |
| `name`            | Yes       |            | Method names with which the transaction attributes are to be associated. The wildcard (*) character can be used to associate the same transaction attribute settings with a number of methods (for example, `get*`, `handle*`, `on*Event`, and so forth). |
| `propagation`     | No        | `REQUIRED` | Transaction propagation behavior.                            |
| `isolation`       | No        | `DEFAULT`  | Transaction isolation level. Only applicable to propagation settings of `REQUIRED` or `REQUIRES_NEW`. |
| `timeout`         | No        | -1         | Transaction timeout (seconds). Only applicable to propagation `REQUIRED` or `REQUIRES_NEW`. |
| `read-only`       | No        | false      | Read-write versus read-only transaction. Applies only to `REQUIRED` or `REQUIRES_NEW`. |
| `rollback-for`    | No        |            | Comma-delimited list of `Exception` instances that trigger rollback. For example, `com.foo.MyBusinessException,ServletException`. |
| `no-rollback-for` | No        |            | Comma-delimited list of `Exception` instances that do not trigger rollback. For example, `com.foo.MyBusinessException,ServletException`. |

#### 1.4.6. Using `@Transactional`

In addition to the XML-based declarative approach to transaction configuration, you can use an annotation-based approach. Declaring transaction semantics directly in the Java source code puts the declarations much closer to the affected code. There is not much danger of undue coupling, because code that is meant to be used transactionally is almost always deployed that way anyway.

|      | The standard `javax.transaction.Transactional` annotation is also supported as a drop-in replacement to Spring’s own annotation. Please refer to JTA 1.2 documentation for more details. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The ease-of-use afforded by the use of the `@Transactional` annotation is best illustrated with an example, which is explained in the text that follows. Consider the following class definition:

Java

Kotlin

```java
// the service class that we want to make transactional
@Transactional
public class DefaultFooService implements FooService {

    Foo getFoo(String fooName) {
        // ...
    }

    Foo getFoo(String fooName, String barName) {
        // ...
    }

    void insertFoo(Foo foo) {
        // ...
    }

    void updateFoo(Foo foo) {
        // ...
    }
}
```

Used at the class level as above, the annotation indicates a default for all methods of the declaring class (as well as its subclasses). Alternatively, each method can get annotated individually. Note that a class-level annotation does not apply to ancestor classes up the class hierarchy; in such a scenario, methods need to be locally redeclared in order to participate in a subclass-level annotation.

When a POJO class such as the one above is defined as a bean in a Spring context, you can make the bean instance transactional through an `@EnableTransactionManagement` annotation in a `@Configuration` class. See the [javadoc](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/transaction/annotation/EnableTransactionManagement.html) for full details.

In XML configuration, the `<tx:annotation-driven/>` tag provides similar convenience:

```xml
<!-- from the file 'context.xml' -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- this is the service object that we want to make transactional -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>

    <!-- enable the configuration of transactional behavior based on annotations -->
    <tx:annotation-driven transaction-manager="txManager"/><!-- a PlatformTransactionManager is still required --> 

    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- (this dependency is defined somewhere else) -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- other <bean/> definitions here -->

</beans>
```

|      | The line that makes the bean instance transactional. |
| ---- | ---------------------------------------------------- |
|      |                                                      |

|      | You can omit the `transaction-manager` attribute in the `<tx:annotation-driven/>` tag if the bean name of the `PlatformTransactionManager` that you want to wire in has the name, `transactionManager`. If the `PlatformTransactionManager` bean that you want to dependency-inject has any other name, you have to use the `transaction-manager` attribute, as in the preceding example. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Method visibility and `@Transactional`

When you use proxies, you should apply the `@Transactional` annotation only to methods with public visibility. If you do annotate protected, private or package-visible methods with the `@Transactional` annotation, no error is raised, but the annotated method does not exhibit the configured transactional settings. If you need to annotate non-public methods, consider using AspectJ (described later).

You can apply the `@Transactional` annotation to an interface definition, a method on an interface, a class definition, or a public method on a class. However, the mere presence of the `@Transactional` annotation is not enough to activate the transactional behavior. The `@Transactional` annotation is merely metadata that can be consumed by some runtime infrastructure that is `@Transactional`-aware and that can use the metadata to configure the appropriate beans with transactional behavior. In the preceding example, the `<tx:annotation-driven/>` element switches on the transactional behavior.

|      | The Spring team recommends that you annotate only concrete classes (and methods of concrete classes) with the `@Transactional` annotation, as opposed to annotating interfaces. You certainly can place the `@Transactional` annotation on an interface (or an interface method), but this works only as you would expect it to if you use interface-based proxies. The fact that Java annotations are not inherited from interfaces means that, if you use class-based proxies (`proxy-target-class="true"`) or the weaving-based aspect (`mode="aspectj"`), the transaction settings are not recognized by the proxying and weaving infrastructure, and the object is not wrapped in a transactional proxy. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | In proxy mode (which is the default), only external method calls coming in through the proxy are intercepted. This means that self-invocation (in effect, a method within the target object calling another method of the target object) does not lead to an actual transaction at runtime even if the invoked method is marked with `@Transactional`. Also, the proxy must be fully initialized to provide the expected behavior, so you should not rely on this feature in your initialization code (that is, `@PostConstruct`). |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Consider using of AspectJ mode (see the `mode` attribute in the following table) if you expect self-invocations to be wrapped with transactions as well. In this case, there no proxy in the first place. Instead, the target class is woven (that is, its byte code is modified) to turn `@Transactional` into runtime behavior on any kind of method.

| XML Attribute         | Annotation Attribute                                         | Default                     | Description                                                  |
| :-------------------- | :----------------------------------------------------------- | :-------------------------- | :----------------------------------------------------------- |
| `transaction-manager` | N/A (see [`TransactionManagementConfigurer`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/transaction/annotation/TransactionManagementConfigurer.html) javadoc) | `transactionManager`        | Name of the transaction manager to use. Required only if the name of the transaction manager is not `transactionManager`, as in the preceding example. |
| `mode`                | `mode`                                                       | `proxy`                     | The default mode (`proxy`) processes annotated beans to be proxied by using Spring’s AOP framework (following proxy semantics, as discussed earlier, applying to method calls coming in through the proxy only). The alternative mode (`aspectj`) instead weaves the affected classes with Spring’s AspectJ transaction aspect, modifying the target class byte code to apply to any kind of method call. AspectJ weaving requires `spring-aspects.jar` in the classpath as well as having load-time weaving (or compile-time weaving) enabled. (See [Spring configuration](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop-aj-ltw-spring) for details on how to set up load-time weaving.) |
| `proxy-target-class`  | `proxyTargetClass`                                           | `false`                     | Applies to `proxy` mode only. Controls what type of transactional proxies are created for classes annotated with the `@Transactional` annotation. If the `proxy-target-class` attribute is set to `true`, class-based proxies are created. If `proxy-target-class` is `false` or if the attribute is omitted, then standard JDK interface-based proxies are created. (See [Proxying Mechanisms](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop-proxying) for a detailed examination of the different proxy types.) |
| `order`               | `order`                                                      | `Ordered.LOWEST_PRECEDENCE` | Defines the order of the transaction advice that is applied to beans annotated with `@Transactional`. (For more information about the rules related to ordering of AOP advice, see [Advice Ordering](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop-ataspectj-advice-ordering).) No specified ordering means that the AOP subsystem determines the order of the advice. |

|      | The default advice mode for processing `@Transactional` annotations is `proxy`, which allows for interception of calls through the proxy only. Local calls within the same class cannot get intercepted that way. For a more advanced mode of interception, consider switching to `aspectj` mode in combination with compile-time or load-time weaving. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | The `proxy-target-class` attribute controls what type of transactional proxies are created for classes annotated with the `@Transactional` annotation. If `proxy-target-class` is set to `true`, class-based proxies are created. If `proxy-target-class` is `false` or if the attribute is omitted, standard JDK interface-based proxies are created. (See [core.html](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop-proxying) for a discussion of the different proxy types.) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | `@EnableTransactionManagement` and `<tx:annotation-driven/>` looks for `@Transactional` only on beans in the same application context in which they are defined. This means that, if you put annotation-driven configuration in a `WebApplicationContext` for a `DispatcherServlet`, it checks for `@Transactional` beans only in your controllers and not your services. See [MVC](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc-servlet) for more information. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The most derived location takes precedence when evaluating the transactional settings for a method. In the case of the following example, the `DefaultFooService` class is annotated at the class level with the settings for a read-only transaction, but the `@Transactional` annotation on the `updateFoo(Foo)` method in the same class takes precedence over the transactional settings defined at the class level.

Java

Kotlin

```java
@Transactional(readOnly = true)
public class DefaultFooService implements FooService {

    public Foo getFoo(String fooName) {
        // ...
    }

    // these settings have precedence for this method
    @Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW)
    public void updateFoo(Foo foo) {
        // ...
    }
}
```

##### `@Transactional` Settings

The `@Transactional` annotation is metadata that specifies that an interface, class, or method must have transactional semantics (for example, “start a brand new read-only transaction when this method is invoked, suspending any existing transaction”). The default `@Transactional` settings are as follows:

- The propagation setting is `PROPAGATION_REQUIRED.`
- The isolation level is `ISOLATION_DEFAULT.`
- The transaction is read-write.
- The transaction timeout defaults to the default timeout of the underlying transaction system, or to none if timeouts are not supported.
- Any `RuntimeException` triggers rollback, and any checked `Exception` does not.

You can change these default settings. The following table summarizes the various properties of the `@Transactional` annotation:

| Property                                                     | Type                                                         | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| [value](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#tx-multiple-tx-mgrs-with-attransactional) | `String`                                                     | Optional qualifier that specifies the transaction manager to be used. |
| [propagation](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#tx-propagation) | `enum`: `Propagation`                                        | Optional propagation setting.                                |
| `isolation`                                                  | `enum`: `Isolation`                                          | Optional isolation level. Applies only to propagation values of `REQUIRED` or `REQUIRES_NEW`. |
| `timeout`                                                    | `int` (in seconds of granularity)                            | Optional transaction timeout. Applies only to propagation values of `REQUIRED` or `REQUIRES_NEW`. |
| `readOnly`                                                   | `boolean`                                                    | Read-write versus read-only transaction. Only applicable to values of `REQUIRED` or `REQUIRES_NEW`. |
| `rollbackFor`                                                | Array of `Class` objects, which must be derived from `Throwable.` | Optional array of exception classes that must cause rollback. |
| `rollbackForClassName`                                       | Array of class names. The classes must be derived from `Throwable.` | Optional array of names of exception classes that must cause rollback. |
| `noRollbackFor`                                              | Array of `Class` objects, which must be derived from `Throwable.` | Optional array of exception classes that must not cause rollback. |
| `noRollbackForClassName`                                     | Array of `String` class names, which must be derived from `Throwable.` | Optional array of names of exception classes that must not cause rollback. |

Currently, you cannot have explicit control over the name of a transaction, where 'name' means the transaction name that appears in a transaction monitor, if applicable (for example, WebLogic’s transaction monitor), and in logging output. For declarative transactions, the transaction name is always the fully-qualified class name + `.` + the method name of the transactionally advised class. For example, if the `handlePayment(..)` method of the `BusinessService` class started a transaction, the name of the transaction would be: `com.example.BusinessService.handlePayment`.

##### Multiple Transaction Managers with `@Transactional`

Most Spring applications need only a single transaction manager, but there may be situations where you want multiple independent transaction managers in a single application. You can use the `value` attribute of the `@Transactional` annotation to optionally specify the identity of the `PlatformTransactionManager` to be used. This can either be the bean name or the qualifier value of the transaction manager bean. For example, using the qualifier notation, you can combine the following Java code with the following transaction manager bean declarations in the application context:

Java

Kotlin

```java
public class TransactionalService {

    @Transactional("order")
    public void setSomething(String name) { ... }

    @Transactional("account")
    public void doSomething() { ... }
}
```

The following listing shows the bean declarations:

```xml
<tx:annotation-driven/>

    <bean id="transactionManager1" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        ...
        <qualifier value="order"/>
    </bean>

    <bean id="transactionManager2" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        ...
        <qualifier value="account"/>
    </bean>
```

In this case, the two methods on `TransactionalService` run under separate transaction managers, differentiated by the `order` and `account` qualifiers. The default `<tx:annotation-driven>` target bean name, `transactionManager`, is still used if no specifically qualified `PlatformTransactionManager` bean is found.

##### Custom Shortcut Annotations

If you find you repeatedly use the same attributes with `@Transactional` on many different methods, [Spring’s meta-annotation support](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-meta-annotations) lets you define custom shortcut annotations for your specific use cases. For example, consider the following annotation definitions:

Java

Kotlin

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional("order")
public @interface OrderTx {
}

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional("account")
public @interface AccountTx {
}
```

The preceding annotations lets us write the example from the previous section as follows:

Java

Kotlin

```java
public class TransactionalService {

    @OrderTx
    public void setSomething(String name) {
        // ...
    }

    @AccountTx
    public void doSomething() {
        // ...
    }
}
```

In the preceding example, we used the syntax to define the transaction manager qualifier, but we could also have included propagation behavior, rollback rules, timeouts, and other features.

#### 1.4.7. Transaction Propagation

This section describes some semantics of transaction propagation in Spring. Note that this section is not an introduction to transaction propagation proper. Rather, it details some of the semantics regarding transaction propagation in Spring.

In Spring-managed transactions, be aware of the difference between physical and logical transactions, and how the propagation setting applies to this difference.

##### Understanding `PROPAGATION_REQUIRED`

![tx prop required](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/images/tx_prop_required.png)

`PROPAGATION_REQUIRED` enforces a physical transaction, either locally for the current scope if no transaction exists yet or participating in an existing 'outer' transaction defined for a larger scope. This is a fine default in common call stack arrangements within the same thread (for example, a service facade that delegates to several repository methods where all the underlying resources have to participate in the service-level transaction).

|      | By default, a participating transaction joins the characteristics of the outer scope, silently ignoring the local isolation level, timeout value, or read-only flag (if any). Consider switching the `validateExistingTransactions` flag to `true` on your transaction manager if you want isolation level declarations to be rejected when participating in an existing transaction with a different isolation level. This non-lenient mode also rejects read-only mismatches (that is, an inner read-write transaction that tries to participate in a read-only outer scope). |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

When the propagation setting is `PROPAGATION_REQUIRED`, a logical transaction scope is created for each method upon which the setting is applied. Each such logical transaction scope can determine rollback-only status individually, with an outer transaction scope being logically independent from the inner transaction scope. In the case of standard `PROPAGATION_REQUIRED` behavior, all these scopes are mapped to the same physical transaction. So a rollback-only marker set in the inner transaction scope does affect the outer transaction’s chance to actually commit.

However, in the case where an inner transaction scope sets the rollback-only marker, the outer transaction has not decided on the rollback itself, so the rollback (silently triggered by the inner transaction scope) is unexpected. A corresponding `UnexpectedRollbackException` is thrown at that point. This is expected behavior so that the caller of a transaction can never be misled to assume that a commit was performed when it really was not. So, if an inner transaction (of which the outer caller is not aware) silently marks a transaction as rollback-only, the outer caller still calls commit. The outer caller needs to receive an `UnexpectedRollbackException` to indicate clearly that a rollback was performed instead.

##### Understanding `PROPAGATION_REQUIRES_NEW`

![tx prop requires new](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/images/tx_prop_requires_new.png)

`PROPAGATION_REQUIRES_NEW`, in contrast to `PROPAGATION_REQUIRED`, always uses an independent physical transaction for each affected transaction scope, never participating in an existing transaction for an outer scope. In such an arrangement, the underlying resource transactions are different and, hence, can commit or roll back independently, with an outer transaction not affected by an inner transaction’s rollback status and with an inner transaction’s locks released immediately after its completion. Such an independent inner transaction can also declare its own isolation level, timeout, and read-only settings and not inherit an outer transaction’s characteristics.

##### Understanding `PROPAGATION_NESTED`

`PROPAGATION_NESTED` uses a single physical transaction with multiple savepoints that it can roll back to. Such partial rollbacks let an inner transaction scope trigger a rollback for its scope, with the outer transaction being able to continue the physical transaction despite some operations having been rolled back. This setting is typically mapped onto JDBC savepoints, so it works only with JDBC resource transactions. See Spring’s [`DataSourceTransactionManager`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/jdbc/datasource/DataSourceTransactionManager.html).

#### 1.4.8. Advising Transactional Operations

Suppose you want to execute both transactional operations and some basic profiling advice. How do you effect this in the context of `<tx:annotation-driven/>`?

When you invoke the `updateFoo(Foo)` method, you want to see the following actions:

- The configured profiling aspect starts.
- The transactional advice executes.
- The method on the advised object executes.
- The transaction commits.
- The profiling aspect reports the exact duration of the whole transactional method invocation.

|      | This chapter is not concerned with explaining AOP in any great detail (except as it applies to transactions). See [AOP](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop) for detailed coverage of the AOP configuration and AOP in general. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The following code shows the simple profiling aspect discussed earlier:

Java

Kotlin

```java
package x.y;

import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.util.StopWatch;
import org.springframework.core.Ordered;

public class SimpleProfiler implements Ordered {

    private int order;

    // allows us to control the ordering of advice
    public int getOrder() {
        return this.order;
    }

    public void setOrder(int order) {
        this.order = order;
    }

    // this method is the around advice
    public Object profile(ProceedingJoinPoint call) throws Throwable {
        Object returnValue;
        StopWatch clock = new StopWatch(getClass().getName());
        try {
            clock.start(call.toShortString());
            returnValue = call.proceed();
        } finally {
            clock.stop();
            System.out.println(clock.prettyPrint());
        }
        return returnValue;
    }
}
```

The ordering of advice is controlled through the `Ordered` interface. For full details on advice ordering, see [Advice ordering](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop-ataspectj-advice-ordering).

The following configuration creates a `fooService` bean that has profiling and transactional aspects applied to it in the desired order:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="fooService" class="x.y.service.DefaultFooService"/>

    <!-- this is the aspect -->
    <bean id="profiler" class="x.y.SimpleProfiler">
        <!-- execute before the transactional advice (hence the lower order number) -->
        <property name="order" value="1"/>
    </bean>

    <tx:annotation-driven transaction-manager="txManager" order="200"/>

    <aop:config>
            <!-- this advice will execute around the transactional advice -->
            <aop:aspect id="profilingAspect" ref="profiler">
                <aop:pointcut id="serviceMethodWithReturnValue"
                        expression="execution(!void x.y..*Service.*(..))"/>
                <aop:around method="profile" pointcut-ref="serviceMethodWithReturnValue"/>
            </aop:aspect>
    </aop:config>

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
        <property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
        <property name="username" value="scott"/>
        <property name="password" value="tiger"/>
    </bean>

    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

</beans>
```

You can configure any number of additional aspects in similar fashion.

The following example creates the same setup as the previous two examples but uses the purely XML declarative approach:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="fooService" class="x.y.service.DefaultFooService"/>

    <!-- the profiling advice -->
    <bean id="profiler" class="x.y.SimpleProfiler">
        <!-- execute before the transactional advice (hence the lower order number) -->
        <property name="order" value="1"/>
    </bean>

    <aop:config>
        <aop:pointcut id="entryPointMethod" expression="execution(* x.y..*Service.*(..))"/>
        <!-- will execute after the profiling advice (c.f. the order attribute) -->

        <aop:advisor advice-ref="txAdvice" pointcut-ref="entryPointMethod" order="2"/>
        <!-- order value is higher than the profiling aspect -->

        <aop:aspect id="profilingAspect" ref="profiler">
            <aop:pointcut id="serviceMethodWithReturnValue"
                    expression="execution(!void x.y..*Service.*(..))"/>
            <aop:around method="profile" pointcut-ref="serviceMethodWithReturnValue"/>
        </aop:aspect>

    </aop:config>

    <tx:advice id="txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <!-- other <bean/> definitions such as a DataSource and a PlatformTransactionManager here -->

</beans>
```

The result of the preceding configuration is a `fooService` bean that has profiling and transactional aspects applied to it in that order. If you want the profiling advice to execute after the transactional advice on the way in and before the transactional advice on the way out, you can swap the value of the profiling aspect bean’s `order` property so that it is higher than the transactional advice’s order value.

You can configure additional aspects in similar fashion.

#### 1.4.9. Using `@Transactional` with AspectJ

You can also use the Spring Framework’s `@Transactional` support outside of a Spring container by means of an AspectJ aspect. To do so, first annotate your classes (and optionally your classes' methods) with the `@Transactional` annotation, and then link (weave) your application with the `org.springframework.transaction.aspectj.AnnotationTransactionAspect` defined in the `spring-aspects.jar` file. You must also configure The aspect with a transaction manager. You can use the Spring Framework’s IoC container to take care of dependency-injecting the aspect. The simplest way to configure the transaction management aspect is to use the `<tx:annotation-driven/>` element and specify the `mode` attribute to `aspectj` as described in [Using `@Transactional`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-declarative-annotations). Because we focus here on applications that run outside of a Spring container, we show you how to do it programmatically.

|      | Prior to continuing, you may want to read [Using `@Transactional`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-declarative-annotations) and [AOP](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop) respectively. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The following example shows how to create a transaction manager and configure the `AnnotationTransactionAspect` to use it:

Java

Kotlin

```java
// construct an appropriate transaction manager
DataSourceTransactionManager txManager = new DataSourceTransactionManager(getDataSource());

// configure the AnnotationTransactionAspect to use it; this must be done before executing any transactional methods
AnnotationTransactionAspect.aspectOf().setTransactionManager(txManager);
```

|      | When you use this aspect, you must annotate the implementation class (or the methods within that class or both), not the interface (if any) that the class implements. AspectJ follows Java’s rule that annotations on interfaces are not inherited. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The `@Transactional` annotation on a class specifies the default transaction semantics for the execution of any public method in the class.

The `@Transactional` annotation on a method within the class overrides the default transaction semantics given by the class annotation (if present). You can annotate any method, regardless of visibility.

To weave your applications with the `AnnotationTransactionAspect`, you must either build your application with AspectJ (see the [AspectJ Development Guide](https://www.eclipse.org/aspectj/doc/released/devguide/index.html)) or use load-time weaving. See [Load-time weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop-aj-ltw) for a discussion of load-time weaving with AspectJ.

### 1.5. Programmatic Transaction Management

The Spring Framework provides two means of programmatic transaction management, by using:

- The `TransactionTemplate`.
- A `PlatformTransactionManager` implementation directly.

The Spring team generally recommends the `TransactionTemplate` for programmatic transaction management. The second approach is similar to using the JTA `UserTransaction` API, although exception handling is less cumbersome.

#### 1.5.1. Using the `TransactionTemplate`

The `TransactionTemplate` adopts the same approach as other Spring templates, such as the `JdbcTemplate`. It uses a callback approach (to free application code from having to do the boilerplate acquisition and release transactional resources) and results in code that is intention driven, in that your code focuses solely on what you want to do.

|      | As the examples that follow show, using the `TransactionTemplate` absolutely couples you to Spring’s transaction infrastructure and APIs. Whether or not programmatic transaction management is suitable for your development needs is a decision that you have to make yourself. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Application code that must execute in a transactional context and that explicitly uses the `TransactionTemplate` resembles the next example. You, as an application developer, can write a `TransactionCallback` implementation (typically expressed as an anonymous inner class) that contains the code that you need to execute in the context of a transaction. You can then pass an instance of your custom `TransactionCallback` to the `execute(..)` method exposed on the `TransactionTemplate`. The following example shows how to do so:

Java

Kotlin

```java
public class SimpleService implements Service {

    // single TransactionTemplate shared amongst all methods in this instance
    private final TransactionTemplate transactionTemplate;

    // use constructor-injection to supply the PlatformTransactionManager
    public SimpleService(PlatformTransactionManager transactionManager) {
        this.transactionTemplate = new TransactionTemplate(transactionManager);
    }

    public Object someServiceMethod() {
        return transactionTemplate.execute(new TransactionCallback() {
            // the code in this method executes in a transactional context
            public Object doInTransaction(TransactionStatus status) {
                updateOperation1();
                return resultOfUpdateOperation2();
            }
        });
    }
}
```

If there is no return value, you can use the convenient `TransactionCallbackWithoutResult` class with an anonymous class, as follows:

Java

Kotlin

```java
transactionTemplate.execute(new TransactionCallbackWithoutResult() {
    protected void doInTransactionWithoutResult(TransactionStatus status) {
        updateOperation1();
        updateOperation2();
    }
});
```

Code within the callback can roll the transaction back by calling the `setRollbackOnly()` method on the supplied `TransactionStatus` object, as follows:

Java

Kotlin

```java
transactionTemplate.execute(new TransactionCallbackWithoutResult() {

    protected void doInTransactionWithoutResult(TransactionStatus status) {
        try {
            updateOperation1();
            updateOperation2();
        } catch (SomeBusinessException ex) {
            status.setRollbackOnly();
        }
    }
});
```

##### Specifying Transaction Settings

You can specify transaction settings (such as the propagation mode, the isolation level, the timeout, and so forth) on the `TransactionTemplate` either programmatically or in configuration. By default, `TransactionTemplate` instances have the [default transactional settings](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-declarative-txadvice-settings). The following example shows the programmatic customization of the transactional settings for a specific `TransactionTemplate:`

Java

Kotlin

```java
public class SimpleService implements Service {

    private final TransactionTemplate transactionTemplate;

    public SimpleService(PlatformTransactionManager transactionManager) {
        this.transactionTemplate = new TransactionTemplate(transactionManager);

        // the transaction settings can be set here explicitly if so desired
        this.transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_UNCOMMITTED);
        this.transactionTemplate.setTimeout(30); // 30 seconds
        // and so forth...
    }
}
```

The following example defines a `TransactionTemplate` with some custom transactional settings by using Spring XML configuration:

```xml
<bean id="sharedTransactionTemplate"
        class="org.springframework.transaction.support.TransactionTemplate">
    <property name="isolationLevelName" value="ISOLATION_READ_UNCOMMITTED"/>
    <property name="timeout" value="30"/>
</bean>
```

You can then inject the `sharedTransactionTemplate` into as many services as are required.

Finally, instances of the `TransactionTemplate` class are thread-safe, in that instances do not maintain any conversational state. `TransactionTemplate` instances do, however, maintain configuration state. So, while a number of classes may share a single instance of a `TransactionTemplate`, if a class needs to use a `TransactionTemplate` with different settings (for example, a different isolation level), you need to create two distinct `TransactionTemplate` instances.

#### 1.5.2. Using the `PlatformTransactionManager`

You can also use the `org.springframework.transaction.PlatformTransactionManager` directly to manage your transaction. To do so, pass the implementation of the `PlatformTransactionManager` you use to your bean through a bean reference. Then, by using the `TransactionDefinition` and `TransactionStatus` objects, you can initiate transactions, roll back, and commit. The following example shows how to do so:

Java

Kotlin

```java
DefaultTransactionDefinition def = new DefaultTransactionDefinition();
// explicitly setting the transaction name is something that can be done only programmatically
def.setName("SomeTxName");
def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

TransactionStatus status = txManager.getTransaction(def);
try {
    // execute your business logic here
}
catch (MyException ex) {
    txManager.rollback(status);
    throw ex;
}
txManager.commit(status);
```

### 1.6. Choosing Between Programmatic and Declarative Transaction Management

Programmatic transaction management is usually a good idea only if you have a small number of transactional operations. For example, if you have a web application that requires transactions only for certain update operations, you may not want to set up transactional proxies by using Spring or any other technology. In this case, using the `TransactionTemplate` may be a good approach. Being able to set the transaction name explicitly is also something that can be done only by using the programmatic approach to transaction management.

On the other hand, if your application has numerous transactional operations, declarative transaction management is usually worthwhile. It keeps transaction management out of business logic and is not difficult to configure. When using the Spring Framework, rather than EJB CMT, the configuration cost of declarative transaction management is greatly reduced.

### 1.7. Transaction-bound Events

As of Spring 4.2, the listener of an event can be bound to a phase of the transaction. The typical example is to handle the event when the transaction has completed successfully. Doing so lets events be used with more flexibility when the outcome of the current transaction actually matters to the listener.

You can register a regular event listener by using the `@EventListener` annotation. If you need to bind it to the transaction, use `@TransactionalEventListener`. When you do so, the listener is bound to the commit phase of the transaction by default.

The next example shows this concept. Assume that a component publishes an order-created event and that we want to define a listener that should only handle that event once the transaction in which it has been published has committed successfully. The following example sets up such an event listener:

Java

Kotlin

```java
@Component
public class MyComponent {

    @TransactionalEventListener
    public void handleOrderCreatedEvent(CreationEvent<Order> creationEvent) {
        // ...
    }
}
```

The `@TransactionalEventListener` annotation exposes a `phase` attribute that lets you customize the phase of the transaction to which the listener should be bound. The valid phases are `BEFORE_COMMIT`, `AFTER_COMMIT` (default), `AFTER_ROLLBACK`, and `AFTER_COMPLETION` that aggregates the transaction completion (be it a commit or a rollback).

If no transaction is running, the listener is not invoked at all, since we cannot honor the required semantics. You can, however, override that behavior by setting the `fallbackExecution` attribute of the annotation to `true`.

### 1.8. Application server-specific integration

Spring’s transaction abstraction is generally application server-agnostic. Additionally, Spring’s `JtaTransactionManager` class (which can optionally perform a JNDI lookup for the JTA `UserTransaction` and `TransactionManager` objects) autodetects the location for the latter object, which varies by application server. Having access to the JTA `TransactionManager` allows for enhanced transaction semantics — in particular, supporting transaction suspension. See the [`JtaTransactionManager`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/transaction/jta/JtaTransactionManager.html) javadoc for details.

Spring’s `JtaTransactionManager` is the standard choice to run on Java EE application servers and is known to work on all common servers. Advanced functionality, such as transaction suspension, works on many servers as well (including GlassFish, JBoss and Geronimo) without any special configuration required. However, for fully supported transaction suspension and further advanced integration, Spring includes special adapters for WebLogic Server and WebSphere. These adapters are discussed in the following sections.

For standard scenarios, including WebLogic Server and WebSphere, consider using the convenient `<tx:jta-transaction-manager/>` configuration element. When configured, this element automatically detects the underlying server and chooses the best transaction manager available for the platform. This means that you need not explicitly configure server-specific adapter classes (as discussed in the following sections). Rather, they are chosen automatically, with the standard `JtaTransactionManager` as the default fallback.

#### 1.8.1. IBM WebSphere

On WebSphere 6.1.0.9 and above, the recommended Spring JTA transaction manager to use is `WebSphereUowTransactionManager`. This special adapter uses IBM’s `UOWManager` API, which is available in WebSphere Application Server 6.1.0.9 and later. With this adapter, Spring-driven transaction suspension (suspend and resume as initiated by `PROPAGATION_REQUIRES_NEW`) is officially supported by IBM.

#### 1.8.2. Oracle WebLogic Server

On WebLogic Server 9.0 or above, you would typically use the `WebLogicJtaTransactionManager` instead of the stock `JtaTransactionManager` class. This special WebLogic-specific subclass of the normal `JtaTransactionManager` supports the full power of Spring’s transaction definitions in a WebLogic-managed transaction environment, beyond standard JTA semantics. Features include transaction names, per-transaction isolation levels, and proper resuming of transactions in all cases.

### 1.9. Solutions to Common Problems

This section describes solutions to some common problems.

#### 1.9.1. Using the Wrong Transaction Manager for a Specific `DataSource`

Use the correct `PlatformTransactionManager` implementation based on your choice of transactional technologies and requirements. Used properly, the Spring Framework merely provides a straightforward and portable abstraction. If you use global transactions, you must use the `org.springframework.transaction.jta.JtaTransactionManager` class (or an [application server-specific subclass](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#transaction-application-server-integration) of it) for all your transactional operations. Otherwise, the transaction infrastructure tries to perform local transactions on such resources as container `DataSource` instances. Such local transactions do not make sense, and a good application server treats them as errors.

### 1.10. Further Resources

For more information about the Spring Framework’s transaction support, see:

- [Distributed transactions in Spring, with and without XA](https://www.javaworld.com/javaworld/jw-01-2009/jw-01-spring-transactions.html) is a JavaWorld presentation in which Spring’s David Syer guides you through seven patterns for distributed transactions in Spring applications, three of them with XA and four without.
- [*Java Transaction Design Strategies*](https://www.infoq.com/minibooks/JTDS) is a book available from [InfoQ](https://www.infoq.com/) that provides a well-paced introduction to transactions in Java. It also includes side-by-side examples of how to configure and use transactions with both the Spring Framework and EJB3.

## 2. DAO Support

The Data Access Object (DAO) support in Spring is aimed at making it easy to work with data access technologies (such as JDBC, Hibernate, or JPA) in a consistent way. This lets you switch between the aforementioned persistence technologies fairly easily, and it also lets you code without worrying about catching exceptions that are specific to each technology.

### 2.1. Consistent Exception Hierarchy

Spring provides a convenient translation from technology-specific exceptions, such as `SQLException` to its own exception class hierarchy, which has `DataAccessException` as the root exception. These exceptions wrap the original exception so that there is never any risk that you might lose any information about what might have gone wrong.

In addition to JDBC exceptions, Spring can also wrap JPA- and Hibernate-specific exceptions, converting them to a set of focused runtime exceptions. This lets you handle most non-recoverable persistence exceptions in only the appropriate layers, without having annoying boilerplate catch-and-throw blocks and exception declarations in your DAOs. (You can still trap and handle exceptions anywhere you need to though.) As mentioned above, JDBC exceptions (including database-specific dialects) are also converted to the same hierarchy, meaning that you can perform some operations with JDBC within a consistent programming model.

The preceding discussion holds true for the various template classes in Spring’s support for various ORM frameworks. If you use the interceptor-based classes, the application must care about handling `HibernateExceptions` and `PersistenceExceptions` itself, preferably by delegating to the `convertHibernateAccessException(..)` or `convertJpaAccessException()` methods, respectively, of `SessionFactoryUtils`. These methods convert the exceptions to exceptions that are compatible with the exceptions in the `org.springframework.dao` exception hierarchy. As `PersistenceExceptions` are unchecked, they can get thrown, too (sacrificing generic DAO abstraction in terms of exceptions, though).

The following image shows the exception hierarchy that Spring provides. (Note that the class hierarchy detailed in the image shows only a subset of the entire `DataAccessException` hierarchy.)

![DataAccessException](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/images/DataAccessException.png)

### 2.2. Annotations Used to Configure DAO or Repository Classes

The best way to guarantee that your Data Access Objects (DAOs) or repositories provide exception translation is to use the `@Repository` annotation. This annotation also lets the component scanning support find and configure your DAOs and repositories without having to provide XML configuration entries for them. The following example shows how to use the `@Repository` annotation:

Java

Kotlin

```java
@Repository 
public class SomeMovieFinder implements MovieFinder {
    // ...
}
```

|      | The `@Repository` annotation. |
| ---- | ----------------------------- |
|      |                               |

Any DAO or repository implementation needs access to a persistence resource, depending on the persistence technology used. For example, a JDBC-based repository needs access to a JDBC `DataSource`, and a JPA-based repository needs access to an `EntityManager`. The easiest way to accomplish this is to have this resource dependency injected by using one of the `@Autowired`, `@Inject`, `@Resource` or `@PersistenceContext` annotations. The following example works for a JPA repository:

Java

Kotlin

```java
@Repository
public class JpaMovieFinder implements MovieFinder {

    @PersistenceContext
    private EntityManager entityManager;

    // ...
}
```

If you use the classic Hibernate APIs, you can inject `SessionFactory`, as the following example shows:

Java

Kotlin

```java
@Repository
public class HibernateMovieFinder implements MovieFinder {

    private SessionFactory sessionFactory;

    @Autowired
    public void setSessionFactory(SessionFactory sessionFactory) {
        this.sessionFactory = sessionFactory;
    }

    // ...
}
```

The last example we show here is for typical JDBC support. You could have the `DataSource` injected into an initialization method or a constructor, where you would create a `JdbcTemplate` and other data access support classes (such as `SimpleJdbcCall` and others) by using this `DataSource`. The following example autowires a `DataSource`:

Java

Kotlin

```java
@Repository
public class JdbcMovieFinder implements MovieFinder {

    private JdbcTemplate jdbcTemplate;

    @Autowired
    public void init(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    // ...
}
```

|      | See the specific coverage of each persistence technology for details on how to configure the application context to take advantage of these annotations. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 3. Data Access with JDBC

The value provided by the Spring Framework JDBC abstraction is perhaps best shown by the sequence of actions outlined in the following table below. The table shows which actions Spring takes care of and which actions are your responsibility.

| Action                                                   | Spring | You  |
| :------------------------------------------------------- | :----- | :--- |
| Define connection parameters.                            |        | X    |
| Open the connection.                                     | X      |      |
| Specify the SQL statement.                               |        | X    |
| Declare parameters and provide parameter values          |        | X    |
| Prepare and execute the statement.                       | X      |      |
| Set up the loop to iterate through the results (if any). | X      |      |
| Do the work for each iteration.                          |        | X    |
| Process any exception.                                   | X      |      |
| Handle transactions.                                     | X      |      |
| Close the connection, the statement, and the resultset.  | X      |      |

The Spring Framework takes care of all the low-level details that can make JDBC such a tedious API.

### 3.1. Choosing an Approach for JDBC Database Access

You can choose among several approaches to form the basis for your JDBC database access. In addition to three flavors of `JdbcTemplate`, a new `SimpleJdbcInsert` and `SimpleJdbcCall` approach optimizes database metadata, and the RDBMS Object style takes a more object-oriented approach similar to that of JDO Query design. Once you start using one of these approaches, you can still mix and match to include a feature from a different approach. All approaches require a JDBC 2.0-compliant driver, and some advanced features require a JDBC 3.0 driver.

- `JdbcTemplate` is the classic and most popular Spring JDBC approach. This “lowest-level” approach and all others use a JdbcTemplate under the covers.
- `NamedParameterJdbcTemplate` wraps a `JdbcTemplate` to provide named parameters instead of the traditional JDBC `?` placeholders. This approach provides better documentation and ease of use when you have multiple parameters for an SQL statement.
- `SimpleJdbcInsert` and `SimpleJdbcCall` optimize database metadata to limit the amount of necessary configuration. This approach simplifies coding so that you need to provide only the name of the table or procedure and provide a map of parameters matching the column names. This works only if the database provides adequate metadata. If the database does not provide this metadata, you have to provide explicit configuration of the parameters.
- RDBMS objects, including `MappingSqlQuery`, `SqlUpdate` and `StoredProcedure`, require you to create reusable and thread-safe objects during initialization of your data-access layer. This approach is modeled after JDO Query, wherein you define your query string, declare parameters, and compile the query. Once you do that, execute methods can be called multiple times with various parameter values.

### 3.2. Package Hierarchy

The Spring Framework’s JDBC abstraction framework consists of four different packages:

- `core`: The `org.springframework.jdbc.core` package contains the `JdbcTemplate` class and its various callback interfaces, plus a variety of related classes. A subpackage named `org.springframework.jdbc.core.simple` contains the `SimpleJdbcInsert` and `SimpleJdbcCall` classes. Another subpackage named `org.springframework.jdbc.core.namedparam` contains the `NamedParameterJdbcTemplate` class and the related support classes. See [Using the JDBC Core Classes to Control Basic JDBC Processing and Error Handling](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-core), [JDBC Batch Operations](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-advanced-jdbc), and [Simplifying JDBC Operations with the `SimpleJdbc` Classes](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-simple-jdbc).
- `datasource`: The `org.springframework.jdbc.datasource` package contains a utility class for easy `DataSource` access and various simple `DataSource` implementations that you can use for testing and running unmodified JDBC code outside of a Java EE container. A subpackage named `org.springfamework.jdbc.datasource.embedded` provides support for creating embedded databases by using Java database engines, such as HSQL, H2, and Derby. See [Controlling Database Connections](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-connections) and [Embedded Database Support](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-embedded-database-support).
- `object`: The `org.springframework.jdbc.object` package contains classes that represent RDBMS queries, updates, and stored procedures as thread-safe, reusable objects. See [Modeling JDBC Operations as Java Objects](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-object). This approach is modeled by JDO, although objects returned by queries are naturally disconnected from the database. This higher-level of JDBC abstraction depends on the lower-level abstraction in the `org.springframework.jdbc.core` package.
- `support`: The `org.springframework.jdbc.support` package provides `SQLException` translation functionality and some utility classes. Exceptions thrown during JDBC processing are translated to exceptions defined in the `org.springframework.dao` package. This means that code using the Spring JDBC abstraction layer does not need to implement JDBC or RDBMS-specific error handling. All translated exceptions are unchecked, which gives you the option of catching the exceptions from which you can recover while letting other exceptions be propagated to the caller. See [Using `SQLExceptionTranslator`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SQLExceptionTranslator).

### 3.3. Using the JDBC Core Classes to Control Basic JDBC Processing and Error Handling

This section covers how to use the JDBC core classes to control basic JDBC processing, including error handling. It includes the following topics:

- [Using `JdbcTemplate`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-JdbcTemplate)
- [Using `NamedParameterJdbcTemplate`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-NamedParameterJdbcTemplate)
- [Using `SQLExceptionTranslator`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SQLExceptionTranslator)
- [Running Statements](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-statements-executing)
- [Running Queries](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-statements-querying)
- [Updating the Database](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-updates)
- [Retrieving Auto-generated Keys](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-auto-generated-keys)

#### 3.3.1. Using `JdbcTemplate`

`JdbcTemplate` is the central class in the JDBC core package. It handles the creation and release of resources, which helps you avoid common errors, such as forgetting to close the connection. It performs the basic tasks of the core JDBC workflow (such as statement creation and execution), leaving application code to provide SQL and extract results. The `JdbcTemplate` class:

- Runs SQL queries
- Updates statements and stored procedure calls
- Performs iteration over `ResultSet` instances and extraction of returned parameter values.
- Catches JDBC exceptions and translates them to the generic, more informative, exception hierarchy defined in the `org.springframework.dao` package. (See [Consistent Exception Hierarchy](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#dao-exceptions).)

When you use the `JdbcTemplate` for your code, you need only to implement callback interfaces, giving them a clearly defined contract. Given a `Connection` provided by the `JdbcTemplate` class, the `PreparedStatementCreator` callback interface creates a prepared statement, providing SQL and any necessary parameters. The same is true for the `CallableStatementCreator` interface, which creates callable statements. The `RowCallbackHandler` interface extracts values from each row of a `ResultSet`.

You can use `JdbcTemplate` within a DAO implementation through direct instantiation with a `DataSource` reference, or you can configure it in a Spring IoC container and give it to DAOs as a bean reference.

|      | The `DataSource` should always be configured as a bean in the Spring IoC container. In the first case the bean is given to the service directly; in the second case it is given to the prepared template. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

All SQL issued by this class is logged at the `DEBUG` level under the category corresponding to the fully qualified class name of the template instance (typically `JdbcTemplate`, but it may be different if you use a custom subclass of the `JdbcTemplate` class).

The following sections provide some examples of `JdbcTemplate` usage. These examples are not an exhaustive list of all of the functionality exposed by the `JdbcTemplate`. See the attendant [javadoc](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html) for that.

##### Querying (`SELECT`)

The following query gets the number of rows in a relation:

Java

Kotlin

```java
int rowCount = this.jdbcTemplate.queryForObject("select count(*) from t_actor", Integer.class);
```

The following query uses a bind variable:

Java

Kotlin

```java
int countOfActorsNamedJoe = this.jdbcTemplate.queryForObject(
        "select count(*) from t_actor where first_name = ?", Integer.class, "Joe");
```

The following query looks for a `String`:

Java

Kotlin

```java
String lastName = this.jdbcTemplate.queryForObject(
        "select last_name from t_actor where id = ?",
        String.class, 1212L);
```

The following query finds and populates a single domain object:

Java

Kotlin

```java
Actor actor = jdbcTemplate.queryForObject(
        "select first_name, last_name from t_actor where id = ?",
        (resultSet, rowNum) -> {
            Actor newActor = new Actor();
            newActor.setFirstName(resultSet.getString("first_name"));
            newActor.setLastName(resultSet.getString("last_name"));
            return newActor;
        },
        1212L);
```

The following query finds and populates a list of domain objects:

Java

Kotlin

```java
List<Actor> actors = this.jdbcTemplate.query(
        "select first_name, last_name from t_actor",
        (resultSet, rowNum) -> {
            Actor actor = new Actor();
            actor.setFirstName(resultSet.getString("first_name"));
            actor.setLastName(resultSet.getString("last_name"));
            return actor;
        });
```

If the last two snippets of code actually existed in the same application, it would make sense to remove the duplication present in the two `RowMapper` lambda expressions and extract them out into a single field that could then be referenced by DAO methods as needed. For example, it may be better to write the preceding code snippet as follows:

Java

Kotlin

```java
private final RowMapper<Actor> actorRowMapper = (resultSet, rowNum) -> {
    Actor actor = new Actor();
    actor.setFirstName(resultSet.getString("first_name"));
    actor.setLastName(resultSet.getString("last_name"));
    return actor;
};

public List<Actor> findAllActors() {
    return this.jdbcTemplate.query( "select first_name, last_name from t_actor", actorRowMapper);
}
```

##### Updating (`INSERT`, `UPDATE`, and `DELETE`) with `JdbcTemplate`

You can use the `update(..)` method to perform insert, update, and delete operations. Parameter values are usually provided as variable arguments or, alternatively, as an object array.

The following example inserts a new entry:

Java

Kotlin

```java
this.jdbcTemplate.update(
        "insert into t_actor (first_name, last_name) values (?, ?)",
        "Leonor", "Watling");
```

The following example updates an existing entry:

Java

Kotlin

```java
this.jdbcTemplate.update(
        "update t_actor set last_name = ? where id = ?",
        "Banjo", 5276L);
```

The following example deletes an entry:

Java

Kotlin

```java
this.jdbcTemplate.update(
        "delete from t_actor where id = ?",
        Long.valueOf(actorId));
```

##### Other `JdbcTemplate` Operations

You can use the `execute(..)` method to run any arbitrary SQL. Consequently, the method is often used for DDL statements. It is heavily overloaded with variants that take callback interfaces, binding variable arrays, and so on. The following example creates a table:

Java

Kotlin

```java
this.jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
```

The following example invokes a stored procedure:

Java

Kotlin

```java
this.jdbcTemplate.update(
        "call SUPPORT.REFRESH_ACTORS_SUMMARY(?)",
        Long.valueOf(unionId));
```

More sophisticated stored procedure support is [covered later](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-StoredProcedure).

##### `JdbcTemplate` Best Practices

Instances of the `JdbcTemplate` class are thread-safe, once configured. This is important because it means that you can configure a single instance of a `JdbcTemplate` and then safely inject this shared reference into multiple DAOs (or repositories). The `JdbcTemplate` is stateful, in that it maintains a reference to a `DataSource`, but this state is not conversational state.

A common practice when using the `JdbcTemplate` class (and the associated [`NamedParameterJdbcTemplate`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-NamedParameterJdbcTemplate) class) is to configure a `DataSource` in your Spring configuration file and then dependency-inject that shared `DataSource` bean into your DAO classes. The `JdbcTemplate` is created in the setter for the `DataSource`. This leads to DAOs that resemble the following:

Java

Kotlin

```java
public class JdbcCorporateEventDao implements CorporateEventDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    // JDBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

The following example shows the corresponding XML configuration:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="corporateEventDao" class="com.example.JdbcCorporateEventDao">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <context:property-placeholder location="jdbc.properties"/>

</beans>
```

An alternative to explicit configuration is to use component-scanning and annotation support for dependency injection. In this case, you can annotate the class with `@Repository` (which makes it a candidate for component-scanning) and annotate the `DataSource` setter method with `@Autowired`. The following example shows how to do so:

Java

Kotlin

```java
@Repository 
public class JdbcCorporateEventDao implements CorporateEventDao {

    private JdbcTemplate jdbcTemplate;

    @Autowired 
    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource); 
    }

    // JDBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

|      | Annotate the class with `@Repository`.                     |
| ---- | ---------------------------------------------------------- |
|      | Annotate the `DataSource` setter method with `@Autowired`. |
|      | Create a new `JdbcTemplate` with the `DataSource`.         |

The following example shows the corresponding XML configuration:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- Scans within the base package of the application for @Component classes to configure as beans -->
    <context:component-scan base-package="org.springframework.docs.test" />

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <context:property-placeholder location="jdbc.properties"/>

</beans>
```

If you use Spring’s `JdbcDaoSupport` class and your various JDBC-backed DAO classes extend from it, your sub-class inherits a `setDataSource(..)` method from the `JdbcDaoSupport` class. You can choose whether to inherit from this class. The `JdbcDaoSupport` class is provided as a convenience only.

Regardless of which of the above template initialization styles you choose to use (or not), it is seldom necessary to create a new instance of a `JdbcTemplate` class each time you want to run SQL. Once configured, a `JdbcTemplate` instance is thread-safe. If your application accesses multiple databases, you may want multiple `JdbcTemplate` instances, which requires multiple `DataSources` and, subsequently, multiple differently configured `JdbcTemplate` instances.

#### 3.3.2. Using `NamedParameterJdbcTemplate`

The `NamedParameterJdbcTemplate` class adds support for programming JDBC statements by using named parameters, as opposed to programming JDBC statements using only classic placeholder ( `'?'`) arguments. The `NamedParameterJdbcTemplate` class wraps a `JdbcTemplate` and delegates to the wrapped `JdbcTemplate` to do much of its work. This section describes only those areas of the `NamedParameterJdbcTemplate` class that differ from the `JdbcTemplate` itself — namely, programming JDBC statements by using named parameters. The following example shows how to use `NamedParameterJdbcTemplate`:

Java

Kotlin

```java
// some JDBC-backed DAO class...
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActorsByFirstName(String firstName) {

    String sql = "select count(*) from T_ACTOR where first_name = :first_name";

    SqlParameterSource namedParameters = new MapSqlParameterSource("first_name", firstName);

    return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
}
```

Notice the use of the named parameter notation in the value assigned to the `sql` variable and the corresponding value that is plugged into the `namedParameters` variable (of type `MapSqlParameterSource`).

Alternatively, you can pass along named parameters and their corresponding values to a `NamedParameterJdbcTemplate` instance by using the `Map`-based style.The remaining methods exposed by the `NamedParameterJdbcOperations` and implemented by the `NamedParameterJdbcTemplate` class follow a similar pattern and are not covered here.

The following example shows the use of the `Map`-based style:

Java

Kotlin

```java
// some JDBC-backed DAO class...
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActorsByFirstName(String firstName) {

    String sql = "select count(*) from T_ACTOR where first_name = :first_name";

    Map<String, String> namedParameters = Collections.singletonMap("first_name", firstName);

    return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters,  Integer.class);
}
```

One nice feature related to the `NamedParameterJdbcTemplate` (and existing in the same Java package) is the `SqlParameterSource` interface. You have already seen an example of an implementation of this interface in one of the previous code snippets (the `MapSqlParameterSource` class). An `SqlParameterSource` is a source of named parameter values to a `NamedParameterJdbcTemplate`. The `MapSqlParameterSource` class is a simple implementation that is an adapter around a `java.util.Map`, where the keys are the parameter names and the values are the parameter values.

Another `SqlParameterSource` implementation is the `BeanPropertySqlParameterSource` class. This class wraps an arbitrary JavaBean (that is, an instance of a class that adheres to [the JavaBean conventions](https://www.oracle.com/technetwork/java/javase/documentation/spec-136004.html)) and uses the properties of the wrapped JavaBean as the source of named parameter values.

The following example shows a typical JavaBean:

Java

Kotlin

```java
public class Actor {

    private Long id;
    private String firstName;
    private String lastName;

    public String getFirstName() {
        return this.firstName;
    }

    public String getLastName() {
        return this.lastName;
    }

    public Long getId() {
        return this.id;
    }

    // setters omitted...

}
```

The following example uses a `NamedParameterJdbcTemplate` to return the count of the members of the class shown in the preceding example:

Java

Kotlin

```java
// some JDBC-backed DAO class...
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActors(Actor exampleActor) {

    // notice how the named parameters match the properties of the above 'Actor' class
    String sql = "select count(*) from T_ACTOR where first_name = :firstName and last_name = :lastName";

    SqlParameterSource namedParameters = new BeanPropertySqlParameterSource(exampleActor);

    return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
}
```

Remember that the `NamedParameterJdbcTemplate` class wraps a classic `JdbcTemplate` template. If you need access to the wrapped `JdbcTemplate` instance to access functionality that is present only in the `JdbcTemplate` class, you can use the `getJdbcOperations()` method to access the wrapped `JdbcTemplate` through the `JdbcOperations` interface.

See also [`JdbcTemplate` Best Practices](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-JdbcTemplate-idioms) for guidelines on using the `NamedParameterJdbcTemplate` class in the context of an application.

#### 3.3.3. Using `SQLExceptionTranslator`

`SQLExceptionTranslator` is an interface to be implemented by classes that can translate between `SQLExceptions` and Spring’s own `org.springframework.dao.DataAccessException`, which is agnostic in regard to data access strategy. Implementations can be generic (for example, using SQLState codes for JDBC) or proprietary (for example, using Oracle error codes) for greater precision.

`SQLErrorCodeSQLExceptionTranslator` is the implementation of `SQLExceptionTranslator` that is used by default. This implementation uses specific vendor codes. It is more precise than the `SQLState` implementation. The error code translations are based on codes held in a JavaBean type class called `SQLErrorCodes`. This class is created and populated by an `SQLErrorCodesFactory`, which (as the name suggests) is a factory for creating `SQLErrorCodes` based on the contents of a configuration file named `sql-error-codes.xml`. This file is populated with vendor codes and based on the `DatabaseProductName` taken from `DatabaseMetaData`. The codes for the actual database you are using are used.

The `SQLErrorCodeSQLExceptionTranslator` applies matching rules in the following sequence:

1. Any custom translation implemented by a subclass. Normally, the provided concrete `SQLErrorCodeSQLExceptionTranslator` is used, so this rule does not apply. It applies only if you have actually provided a subclass implementation.
2. Any custom implementation of the `SQLExceptionTranslator` interface that is provided as the `customSqlExceptionTranslator` property of the `SQLErrorCodes` class.
3. The list of instances of the `CustomSQLErrorCodesTranslation` class (provided for the `customTranslations` property of the `SQLErrorCodes` class) are searched for a match.
4. Error code matching is applied.
5. Use the fallback translator. `SQLExceptionSubclassTranslator` is the default fallback translator. If this translation is not available, the next fallback translator is the `SQLStateSQLExceptionTranslator`.

|      | The `SQLErrorCodesFactory` is used by default to define `Error` codes and custom exception translations. They are looked up in a file named `sql-error-codes.xml` from the classpath, and the matching `SQLErrorCodes` instance is located based on the database name from the database metadata of the database in use. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

You can extend `SQLErrorCodeSQLExceptionTranslator`, as the following example shows:

Java

Kotlin

```java
public class CustomSQLErrorCodesTranslator extends SQLErrorCodeSQLExceptionTranslator {

    protected DataAccessException customTranslate(String task, String sql, SQLException sqlEx) {
        if (sqlEx.getErrorCode() == -12345) {
            return new DeadlockLoserDataAccessException(task, sqlEx);
        }
        return null;
    }
}
```

In the preceding example, the specific error code (`-12345`) is translated, while other errors are left to be translated by the default translator implementation. To use this custom translator, you must pass it to the `JdbcTemplate` through the method `setExceptionTranslator`, and you must use this `JdbcTemplate` for all of the data access processing where this translator is needed. The following example shows how you can use this custom translator:

Java

Kotlin

```java
private JdbcTemplate jdbcTemplate;

public void setDataSource(DataSource dataSource) {

    // create a JdbcTemplate and set data source
    this.jdbcTemplate = new JdbcTemplate();
    this.jdbcTemplate.setDataSource(dataSource);

    // create a custom translator and set the DataSource for the default translation lookup
    CustomSQLErrorCodesTranslator tr = new CustomSQLErrorCodesTranslator();
    tr.setDataSource(dataSource);
    this.jdbcTemplate.setExceptionTranslator(tr);

}

public void updateShippingCharge(long orderId, long pct) {
    // use the prepared JdbcTemplate for this update
    this.jdbcTemplate.update("update orders" +
        " set shipping_charge = shipping_charge * ? / 100" +
        " where id = ?", pct, orderId);
}
```

The custom translator is passed a data source in order to look up the error codes in `sql-error-codes.xml`.

#### 3.3.4. Running Statements

Running an SQL statement requires very little code. You need a `DataSource` and a `JdbcTemplate`, including the convenience methods that are provided with the `JdbcTemplate`. The following example shows what you need to include for a minimal but fully functional class that creates a new table:

Java

Kotlin

```java
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;

public class ExecuteAStatement {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public void doExecute() {
        this.jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
    }
}
```

#### 3.3.5. Running Queries

Some query methods return a single value. To retrieve a count or a specific value from one row, use `queryForObject(..)`. The latter converts the returned JDBC `Type` to the Java class that is passed in as an argument. If the type conversion is invalid, an `InvalidDataAccessApiUsageException` is thrown. The following example contains two query methods, one for an `int` and one that queries for a `String`:

Java

Kotlin

```java
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;

public class RunAQuery {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int getCount() {
        return this.jdbcTemplate.queryForObject("select count(*) from mytable", Integer.class);
    }

    public String getName() {
        return this.jdbcTemplate.queryForObject("select name from mytable", String.class);
    }
}
```

In addition to the single result query methods, several methods return a list with an entry for each row that the query returned. The most generic method is `queryForList(..)`, which returns a `List` where each element is a `Map` containing one entry for each column, using the column name as the key. If you add a method to the preceding example to retrieve a list of all the rows, it might be as follows:

Java

Kotlin

```java
private JdbcTemplate jdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.jdbcTemplate = new JdbcTemplate(dataSource);
}

public List<Map<String, Object>> getList() {
    return this.jdbcTemplate.queryForList("select * from mytable");
}
```

The returned list would resemble the following:

```
[{name=Bob, id=1}, {name=Mary, id=2}]
```

#### 3.3.6. Updating the Database

The following example updates a column for a certain primary key:

Java

Kotlin

```java
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;

public class ExecuteAnUpdate {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public void setName(int id, String name) {
        this.jdbcTemplate.update("update mytable set name = ? where id = ?", name, id);
    }
}
```

In the preceding example, an SQL statement has placeholders for row parameters. You can pass the parameter values in as varargs or ,alternatively, as an array of objects. Thus, you should explicitly wrap primitives in the primitive wrapper classes, or you should use auto-boxing.

#### 3.3.7. Retrieving Auto-generated Keys

An `update()` convenience method supports the retrieval of primary keys generated by the database. This support is part of the JDBC 3.0 standard. See Chapter 13.6 of the specification for details. The method takes a `PreparedStatementCreator` as its first argument, and this is the way the required insert statement is specified. The other argument is a `KeyHolder`, which contains the generated key on successful return from the update. There is no standard single way to create an appropriate `PreparedStatement` (which explains why the method signature is the way it is). The following example works on Oracle but may not work on other platforms:

Java

Kotlin

```java
final String INSERT_SQL = "insert into my_test (name) values(?)";
final String name = "Rob";

KeyHolder keyHolder = new GeneratedKeyHolder();
jdbcTemplate.update(connection -> {
    PreparedStatement ps = connection.prepareStatement(INSERT_SQL, new String[] { "id" });
    ps.setString(1, name);
    return ps;
}, keyHolder);

// keyHolder.getKey() now contains the generated key
```

### 3.4. Controlling Database Connections

This section covers:

- [Using `DataSource`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-datasource)
- [Using `DataSourceUtils`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-DataSourceUtils)
- [Implementing `SmartDataSource`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SmartDataSource)
- [Extending `AbstractDataSource`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-AbstractDataSource)
- [Using `SingleConnectionDataSource`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SingleConnectionDataSource)
- [Using `DriverManagerDataSource`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-DriverManagerDataSource)
- [Using `TransactionAwareDataSourceProxy`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-TransactionAwareDataSourceProxy)
- [Using `DataSourceTransactionManager`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-DataSourceTransactionManager)

#### 3.4.1. Using `DataSource`

Spring obtains a connection to the database through a `DataSource`. A `DataSource` is part of the JDBC specification and is a generalized connection factory. It lets a container or a framework hide connection pooling and transaction management issues from the application code. As a developer, you need not know details about how to connect to the database. That is the responsibility of the administrator who sets up the datasource. You most likely fill both roles as you develop and test code, but you do not necessarily have to know how the production data source is configured.

When you use Spring’s JDBC layer, you can obtain a data source from JNDI, or you can configure your own with a connection pool implementation provided by a third party. Traditional choices are Apache Commons DBCP and C3P0 with bean-style `DataSource` classes; for a modern JDBC connection pool, consider HikariCP with its builder-style API instead.

|      | You should use the `DriverManagerDataSource` and `SimpleDriverDataSource` classes (as included in the Spring distribution) only for testing purposes! Those variants do not provide pooling and perform poorly when multiple requests for a connection are made. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The following section uses Spring’s `DriverManagerDataSource` implementation. Several other `DataSource` variants are covered later.

To configure a `DriverManagerDataSource`:

1. Obtain a connection with `DriverManagerDataSource` as you typically obtain a JDBC connection.
2. Specify the fully qualified classname of the JDBC driver so that the `DriverManager` can load the driver class.
3. Provide a URL that varies between JDBC drivers. (See the documentation for your driver for the correct value.)
4. Provide a username and a password to connect to the database.

The following example shows how to configure a `DriverManagerDataSource` in Java:

Java

Kotlin

```java
DriverManagerDataSource dataSource = new DriverManagerDataSource();
dataSource.setDriverClassName("org.hsqldb.jdbcDriver");
dataSource.setUrl("jdbc:hsqldb:hsql://localhost:");
dataSource.setUsername("sa");
dataSource.setPassword("");
```

The following example shows the corresponding XML configuration:

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

The next two examples show the basic connectivity and configuration for DBCP and C3P0. To learn about more options that help control the pooling features, see the product documentation for the respective connection pooling implementations.

The following example shows DBCP configuration:

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

The following example shows C3P0 configuration:

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
    <property name="driverClass" value="${jdbc.driverClassName}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

#### 3.4.2. Using `DataSourceUtils`

The `DataSourceUtils` class is a convenient and powerful helper class that provides `static` methods to obtain connections from JNDI and close connections if necessary. It supports thread-bound connections with, for example, `DataSourceTransactionManager`.

#### 3.4.3. Implementing `SmartDataSource`

The `SmartDataSource` interface should be implemented by classes that can provide a connection to a relational database. It extends the `DataSource` interface to let classes that use it query whether the connection should be closed after a given operation. This usage is efficient when you know that you need to reuse a connection.

#### 3.4.4. Extending `AbstractDataSource`

`AbstractDataSource` is an `abstract` base class for Spring’s `DataSource` implementations. It implements code that is common to all `DataSource` implementations. You should extend the `AbstractDataSource` class if you write your own `DataSource` implementation.

#### 3.4.5. Using `SingleConnectionDataSource`

The `SingleConnectionDataSource` class is an implementation of the `SmartDataSource` interface that wraps a single `Connection` that is not closed after each use. This is not multi-threading capable.

If any client code calls `close` on the assumption of a pooled connection (as when using persistence tools), you should set the `suppressClose` property to `true`. This setting returns a close-suppressing proxy that wraps the physical connection. Note that you can no longer cast this to a native Oracle `Connection` or a similar object.

`SingleConnectionDataSource` is primarily a test class. For example, it enables easy testing of code outside an application server, in conjunction with a simple JNDI environment. In contrast to `DriverManagerDataSource`, it reuses the same connection all the time, avoiding excessive creation of physical connections.

#### 3.4.6. Using `DriverManagerDataSource`

The `DriverManagerDataSource` class is an implementation of the standard `DataSource` interface that configures a plain JDBC driver through bean properties and returns a new `Connection` every time.

This implementation is useful for test and stand-alone environments outside of a Java EE container, either as a `DataSource` bean in a Spring IoC container or in conjunction with a simple JNDI environment. Pool-assuming `Connection.close()` calls close the connection, so any `DataSource`-aware persistence code should work. However, using JavaBean-style connection pools (such as `commons-dbcp`) is so easy, even in a test environment, that it is almost always preferable to use such a connection pool over `DriverManagerDataSource`.

#### 3.4.7. Using `TransactionAwareDataSourceProxy`

`TransactionAwareDataSourceProxy` is a proxy for a target `DataSource`. The proxy wraps that target `DataSource` to add awareness of Spring-managed transactions. In this respect, it is similar to a transactional JNDI `DataSource`, as provided by a Java EE server.

|      | It is rarely desirable to use this class, except when already existing code must be called and passed a standard JDBC `DataSource` interface implementation. In this case, you can still have this code be usable and, at the same time, have this code participating in Spring managed transactions. It is generally preferable to write your own new code by using the higher level abstractions for resource management, such as `JdbcTemplate` or `DataSourceUtils`. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

See the [`TransactionAwareDataSourceProxy`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/jdbc/datasource/TransactionAwareDataSourceProxy.html) javadoc for more details.

#### 3.4.8. Using `DataSourceTransactionManager`

The `DataSourceTransactionManager` class is a `PlatformTransactionManager` implementation for single JDBC datasources. It binds a JDBC connection from the specified data source to the currently executing thread, potentially allowing for one thread connection per data source.

Application code is required to retrieve the JDBC connection through `DataSourceUtils.getConnection(DataSource)` instead of Java EE’s standard `DataSource.getConnection`. It throws unchecked `org.springframework.dao` exceptions instead of checked `SQLExceptions`. All framework classes (such as `JdbcTemplate`) use this strategy implicitly. If not used with this transaction manager, the lookup strategy behaves exactly like the common one. Thus, it can be used in any case.

The `DataSourceTransactionManager` class supports custom isolation levels and timeouts that get applied as appropriate JDBC statement query timeouts. To support the latter, application code must either use `JdbcTemplate` or call the `DataSourceUtils.applyTransactionTimeout(..)` method for each created statement.

You can use this implementation instead of `JtaTransactionManager` in the single-resource case, as it does not require the container to support JTA. Switching between both is just a matter of configuration, provided you stick to the required connection lookup pattern. JTA does not support custom isolation levels.

### 3.5. JDBC Batch Operations

Most JDBC drivers provide improved performance if you batch multiple calls to the same prepared statement. By grouping updates into batches, you limit the number of round trips to the database.

#### 3.5.1. Basic Batch Operations with `JdbcTemplate`

You accomplish `JdbcTemplate` batch processing by implementing two methods of a special interface, `BatchPreparedStatementSetter`, and passing that implementation in as the second parameter in your `batchUpdate` method call. You can use the `getBatchSize` method to provide the size of the current batch. You can use the `setValues` method to set the values for the parameters of the prepared statement. This method is called the number of times that you specified in the `getBatchSize` call. The following example updates the `t_actor` table based on entries in a list, and the entire list is used as the batch:

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int[] batchUpdate(final List<Actor> actors) {
        return this.jdbcTemplate.batchUpdate(
                "update t_actor set first_name = ?, last_name = ? where id = ?",
                new BatchPreparedStatementSetter() {
                    public void setValues(PreparedStatement ps, int i) throws SQLException {
                        Actor actor = actors.get(i);
                        ps.setString(1, actor.getFirstName());
                        ps.setString(2, actor.getLastName());
                        ps.setLong(3, actor.getId().longValue());
                    }
                    public int getBatchSize() {
                        return actors.size();
                    }
                });
    }

    // ... additional methods
}
```

If you process a stream of updates or reading from a file, you might have a preferred batch size, but the last batch might not have that number of entries. In this case, you can use the `InterruptibleBatchPreparedStatementSetter` interface, which lets you interrupt a batch once the input source is exhausted. The `isBatchExhausted` method lets you signal the end of the batch.

#### 3.5.2. Batch Operations with a List of Objects

Both the `JdbcTemplate` and the `NamedParameterJdbcTemplate` provides an alternate way of providing the batch update. Instead of implementing a special batch interface, you provide all parameter values in the call as a list. The framework loops over these values and uses an internal prepared statement setter. The API varies, depending on whether you use named parameters. For the named parameters, you provide an array of `SqlParameterSource`, one entry for each member of the batch. You can use the `SqlParameterSourceUtils.createBatch` convenience methods to create this array, passing in an array of bean-style objects (with getter methods corresponding to parameters), `String`-keyed `Map` instances (containing the corresponding parameters as values), or a mix of both.

The following example shows a batch update using named parameters:

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private NamedParameterTemplate namedParameterJdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
    }

    public int[] batchUpdate(List<Actor> actors) {
        return this.namedParameterJdbcTemplate.batchUpdate(
                "update t_actor set first_name = :firstName, last_name = :lastName where id = :id",
                SqlParameterSourceUtils.createBatch(actors));
    }

    // ... additional methods
}
```

For an SQL statement that uses the classic `?` placeholders, you pass in a list containing an object array with the update values. This object array must have one entry for each placeholder in the SQL statement, and they must be in the same order as they are defined in the SQL statement.

The following example is the same as the preceding example, except that it uses classic JDBC `?` placeholders:

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int[] batchUpdate(final List<Actor> actors) {
        List<Object[]> batch = new ArrayList<Object[]>();
        for (Actor actor : actors) {
            Object[] values = new Object[] {
                    actor.getFirstName(), actor.getLastName(), actor.getId()};
            batch.add(values);
        }
        return this.jdbcTemplate.batchUpdate(
                "update t_actor set first_name = ?, last_name = ? where id = ?",
                batch);
    }

    // ... additional methods
}
```

All of the batch update methods that we described earlier return an `int` array containing the number of affected rows for each batch entry. This count is reported by the JDBC driver. If the count is not available, the JDBC driver returns a value of `-2`.

|      | In such a scenario, with automatic setting of values on an underlying `PreparedStatement`, the corresponding JDBC type for each value needs to be derived from the given Java type. While this usually works well, there is a potential for issues (for example, with Map-contained `null` values). Spring, by default, calls `ParameterMetaData.getParameterType` in such a case, which can be expensive with your JDBC driver. You should use a recent driver version and consider setting the `spring.jdbc.getParameterType.ignore` property to `true` (as a JVM system property or in a `spring.properties` file in the root of your classpath) if you encounter a performance issue — for example, as reported on Oracle 12c (SPR-16139).Alternatively, you might consider specifying the corresponding JDBC types explicitly, either through a 'BatchPreparedStatementSetter' (as shown earlier), through an explicit type array given to a 'List<Object[]>' based call, through 'registerSqlType' calls on a custom 'MapSqlParameterSource' instance, or through a 'BeanPropertySqlParameterSource' that derives the SQL type from the Java-declared property type even for a null value. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 3.5.3. Batch Operations with Multiple Batches

The preceding example of a batch update deals with batches that are so large that you want to break them up into several smaller batches. You can do this with the methods mentioned earlier by making multiple calls to the `batchUpdate` method, but there is now a more convenient method. This method takes, in addition to the SQL statement, a `Collection` of objects that contain the parameters, the number of updates to make for each batch, and a `ParameterizedPreparedStatementSetter` to set the values for the parameters of the prepared statement. The framework loops over the provided values and breaks the update calls into batches of the size specified.

The following example shows a batch update that uses a batch size of 100:

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int[][] batchUpdate(final Collection<Actor> actors) {
        int[][] updateCounts = jdbcTemplate.batchUpdate(
                "update t_actor set first_name = ?, last_name = ? where id = ?",
                actors,
                100,
                (PreparedStatement ps, Actor actor) -> {
                    ps.setString(1, actor.getFirstName());
                    ps.setString(2, actor.getLastName());
                    ps.setLong(3, actor.getId().longValue());
                });
        return updateCounts;
    }

    // ... additional methods
}
```

The batch update methods for this call returns an array of `int` arrays that contain an array entry for each batch with an array of the number of affected rows for each update. The top level array’s length indicates the number of batches executed and the second level array’s length indicates the number of updates in that batch. The number of updates in each batch should be the batch size provided for all batches (except that the last one that might be less), depending on the total number of update objects provided. The update count for each update statement is the one reported by the JDBC driver. If the count is not available, the JDBC driver returns a value of `-2`.

### 3.6. Simplifying JDBC Operations with the `SimpleJdbc` Classes

The `SimpleJdbcInsert` and `SimpleJdbcCall` classes provide a simplified configuration by taking advantage of database metadata that can be retrieved through the JDBC driver. This means that you have less to configure up front, although you can override or turn off the metadata processing if you prefer to provide all the details in your code.

#### 3.6.1. Inserting Data by Using `SimpleJdbcInsert`

We start by looking at the `SimpleJdbcInsert` class with the minimal amount of configuration options. You should instantiate the `SimpleJdbcInsert` in the data access layer’s initialization method. For this example, the initializing method is the `setDataSource` method. You do not need to subclass the `SimpleJdbcInsert` class. Instead, you can create a new instance and set the table name by using the `withTableName` method. Configuration methods for this class follow the `fluid` style that returns the instance of the `SimpleJdbcInsert`, which lets you chain all configuration methods. The following example uses only one configuration method (we show examples of multiple methods later):

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource).withTableName("t_actor");
    }

    public void add(Actor actor) {
        Map<String, Object> parameters = new HashMap<String, Object>(3);
        parameters.put("id", actor.getId());
        parameters.put("first_name", actor.getFirstName());
        parameters.put("last_name", actor.getLastName());
        insertActor.execute(parameters);
    }

    // ... additional methods
}
```

The `execute` method used here takes a plain `java.util.Map` as its only parameter. The important thing to note here is that the keys used for the `Map` must match the column names of the table, as defined in the database. This is because we read the metadata to construct the actual insert statement.

#### 3.6.2. Retrieving Auto-generated Keys by Using `SimpleJdbcInsert`

The next example uses the same insert as the preceding example, but, instead of passing in the `id`, it retrieves the auto-generated key and sets it on the new `Actor` object. When it creates the `SimpleJdbcInsert`, in addition to specifying the table name, it specifies the name of the generated key column with the `usingGeneratedKeyColumns` method. The following listing shows how it works:

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource)
                .withTableName("t_actor")
                .usingGeneratedKeyColumns("id");
    }

    public void add(Actor actor) {
        Map<String, Object> parameters = new HashMap<String, Object>(2);
        parameters.put("first_name", actor.getFirstName());
        parameters.put("last_name", actor.getLastName());
        Number newId = insertActor.executeAndReturnKey(parameters);
        actor.setId(newId.longValue());
    }

    // ... additional methods
}
```

The main difference when you run the insert by using this second approach is that you do not add the `id` to the `Map`, and you call the `executeAndReturnKey` method. This returns a `java.lang.Number` object with which you can create an instance of the numerical type that is used in your domain class. You cannot rely on all databases to return a specific Java class here. `java.lang.Number` is the base class that you can rely on. If you have multiple auto-generated columns or the generated values are non-numeric, you can use a `KeyHolder` that is returned from the `executeAndReturnKeyHolder` method.

#### 3.6.3. Specifying Columns for a `SimpleJdbcInsert`

You can limit the columns for an insert by specifying a list of column names with the `usingColumns` method, as the following example shows:

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource)
                .withTableName("t_actor")
                .usingColumns("first_name", "last_name")
                .usingGeneratedKeyColumns("id");
    }

    public void add(Actor actor) {
        Map<String, Object> parameters = new HashMap<String, Object>(2);
        parameters.put("first_name", actor.getFirstName());
        parameters.put("last_name", actor.getLastName());
        Number newId = insertActor.executeAndReturnKey(parameters);
        actor.setId(newId.longValue());
    }

    // ... additional methods
}
```

The execution of the insert is the same as if you had relied on the metadata to determine which columns to use.

#### 3.6.4. Using `SqlParameterSource` to Provide Parameter Values

Using a `Map` to provide parameter values works fine, but it is not the most convenient class to use. Spring provides a couple of implementations of the `SqlParameterSource` interface that you can use instead. The first one is `BeanPropertySqlParameterSource`, which is a very convenient class if you have a JavaBean-compliant class that contains your values. It uses the corresponding getter method to extract the parameter values. The following example shows how to use `BeanPropertySqlParameterSource`:

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource)
                .withTableName("t_actor")
                .usingGeneratedKeyColumns("id");
    }

    public void add(Actor actor) {
        SqlParameterSource parameters = new BeanPropertySqlParameterSource(actor);
        Number newId = insertActor.executeAndReturnKey(parameters);
        actor.setId(newId.longValue());
    }

    // ... additional methods
}
```

Another option is the `MapSqlParameterSource` that resembles a `Map` but provides a more convenient `addValue` method that can be chained. The following example shows how to use it:

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource)
                .withTableName("t_actor")
                .usingGeneratedKeyColumns("id");
    }

    public void add(Actor actor) {
        SqlParameterSource parameters = new MapSqlParameterSource()
                .addValue("first_name", actor.getFirstName())
                .addValue("last_name", actor.getLastName());
        Number newId = insertActor.executeAndReturnKey(parameters);
        actor.setId(newId.longValue());
    }

    // ... additional methods
}
```

As you can see, the configuration is the same. Only the executing code has to change to use these alternative input classes.

#### 3.6.5. Calling a Stored Procedure with `SimpleJdbcCall`

The `SimpleJdbcCall` class uses metadata in the database to look up names of `in` and `out` parameters so that you do not have to explicitly declare them. You can declare parameters if you prefer to do that or if you have parameters (such as `ARRAY` or `STRUCT`) that do not have an automatic mapping to a Java class. The first example shows a simple procedure that returns only scalar values in `VARCHAR` and `DATE` format from a MySQL database. The example procedure reads a specified actor entry and returns `first_name`, `last_name`, and `birth_date` columns in the form of `out` parameters. The following listing shows the first example:

```sql
CREATE PROCEDURE read_actor (
    IN in_id INTEGER,
    OUT out_first_name VARCHAR(100),
    OUT out_last_name VARCHAR(100),
    OUT out_birth_date DATE)
BEGIN
    SELECT first_name, last_name, birth_date
    INTO out_first_name, out_last_name, out_birth_date
    FROM t_actor where id = in_id;
END;
```

The `in_id` parameter contains the `id` of the actor that you are looking up. The `out` parameters return the data read from the table.

You can declare `SimpleJdbcCall` in a manner similar to declaring `SimpleJdbcInsert`. You should instantiate and configure the class in the initialization method of your data-access layer. Compared to the `StoredProcedure` class, you need not create a subclass and you need not to declare parameters that can be looked up in the database metadata. The following example of a `SimpleJdbcCall` configuration uses the preceding stored procedure (the only configuration option, in addition to the `DataSource`, is the name of the stored procedure):

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadActor;

    public void setDataSource(DataSource dataSource) {
        this.procReadActor = new SimpleJdbcCall(dataSource)
                .withProcedureName("read_actor");
    }

    public Actor readActor(Long id) {
        SqlParameterSource in = new MapSqlParameterSource()
                .addValue("in_id", id);
        Map out = procReadActor.execute(in);
        Actor actor = new Actor();
        actor.setId(id);
        actor.setFirstName((String) out.get("out_first_name"));
        actor.setLastName((String) out.get("out_last_name"));
        actor.setBirthDate((Date) out.get("out_birth_date"));
        return actor;
    }

    // ... additional methods
}
```

The code you write for the execution of the call involves creating an `SqlParameterSource` containing the IN parameter. You must match the name provided for the input value with that of the parameter name declared in the stored procedure. The case does not have to match because you use metadata to determine how database objects should be referred to in a stored procedure. What is specified in the source for the stored procedure is not necessarily the way it is stored in the database. Some databases transform names to all upper case, while others use lower case or use the case as specified.

The `execute` method takes the IN parameters and returns a `Map` that contains any `out` parameters keyed by the name, as specified in the stored procedure. In this case, they are `out_first_name`, `out_last_name`, and `out_birth_date`.

The last part of the `execute` method creates an `Actor` instance to use to return the data retrieved. Again, it is important to use the names of the `out` parameters as they are declared in the stored procedure. Also, the case in the names of the `out` parameters stored in the results map matches that of the `out` parameter names in the database, which could vary between databases. To make your code more portable, you should do a case-insensitive lookup or instruct Spring to use a `LinkedCaseInsensitiveMap`. To do the latter, you can create your own `JdbcTemplate` and set the `setResultsMapCaseInsensitive` property to `true`. Then you can pass this customized `JdbcTemplate` instance into the constructor of your `SimpleJdbcCall`. The following example shows this configuration:

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadActor;

    public void setDataSource(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.procReadActor = new SimpleJdbcCall(jdbcTemplate)
                .withProcedureName("read_actor");
    }

    // ... additional methods
}
```

By taking this action, you avoid conflicts in the case used for the names of your returned `out` parameters.

#### 3.6.6. Explicitly Declaring Parameters to Use for a `SimpleJdbcCall`

Earlier in this chapter, we described how parameters are deduced from metadata, but you can declare them explicitly if you wish. You can do so by creating and configuring `SimpleJdbcCall` with the `declareParameters` method, which takes a variable number of `SqlParameter` objects as input. See the [next section](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-params) for details on how to define an `SqlParameter`.

|      | Explicit declarations are necessary if the database you use is not a Spring-supported database. Currently, Spring supports metadata lookup of stored procedure calls for the following databases: Apache Derby, DB2, MySQL, Microsoft SQL Server, Oracle, and Sybase. We also support metadata lookup of stored functions for MySQL, Microsoft SQL Server, and Oracle. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

You can opt to explicitly declare one, some, or all of the parameters. The parameter metadata is still used where you do not explicitly declare parameters. To bypass all processing of metadata lookups for potential parameters and use only the declared parameters, you can call the method `withoutProcedureColumnMetaDataAccess` as part of the declaration. Suppose that you have two or more different call signatures declared for a database function. In this case, you call `useInParameterNames` to specify the list of IN parameter names to include for a given signature.

The following example shows a fully declared procedure call and uses the information from the preceding example:

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadActor;

    public void setDataSource(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.procReadActor = new SimpleJdbcCall(jdbcTemplate)
                .withProcedureName("read_actor")
                .withoutProcedureColumnMetaDataAccess()
                .useInParameterNames("in_id")
                .declareParameters(
                        new SqlParameter("in_id", Types.NUMERIC),
                        new SqlOutParameter("out_first_name", Types.VARCHAR),
                        new SqlOutParameter("out_last_name", Types.VARCHAR),
                        new SqlOutParameter("out_birth_date", Types.DATE)
                );
    }

    // ... additional methods
}
```

The execution and end results of the two examples are the same. The second example specifies all details explicitly rather than relying on metadata.

#### 3.6.7. How to Define `SqlParameters`

To define a parameter for the `SimpleJdbc` classes and also for the RDBMS operations classes (covered in [Modeling JDBC Operations as Java Objects](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-object)) you can use `SqlParameter` or one of its subclasses. To do so, you typically specify the parameter name and SQL type in the constructor. The SQL type is specified by using the `java.sql.Types` constants. Earlier in this chapter, we saw declarations similar to the following:

Java

Kotlin

```java
new SqlParameter("in_id", Types.NUMERIC),
new SqlOutParameter("out_first_name", Types.VARCHAR),
```

The first line with the `SqlParameter` declares an IN parameter. You can use IN parameters for both stored procedure calls and for queries by using the `SqlQuery` and its subclasses (covered in [Understanding `SqlQuery`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SqlQuery)).

The second line (with the `SqlOutParameter`) declares an `out` parameter to be used in a stored procedure call. There is also an `SqlInOutParameter` for `InOut` parameters (parameters that provide an IN value to the procedure and that also return a value).

|      | Only parameters declared as `SqlParameter` and `SqlInOutParameter` are used to provide input values. This is different from the `StoredProcedure` class, which (for backwards compatibility reasons) lets input values be provided for parameters declared as `SqlOutParameter`. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

For IN parameters, in addition to the name and the SQL type, you can specify a scale for numeric data or a type name for custom database types. For `out` parameters, you can provide a `RowMapper` to handle mapping of rows returned from a `REF` cursor. Another option is to specify an `SqlReturnType` that provides an opportunity to define customized handling of the return values.

#### 3.6.8. Calling a Stored Function by Using `SimpleJdbcCall`

You can call a stored function in almost the same way as you call a stored procedure, except that you provide a function name rather than a procedure name. You use the `withFunctionName` method as part of the configuration to indicate that you want to make a call to a function, and the corresponding string for a function call is generated. A specialized execute call (`executeFunction`) is used to execute the function, and it returns the function return value as an object of a specified type, which means you do not have to retrieve the return value from the results map. A similar convenience method (named `executeObject`) is also available for stored procedures that have only one `out` parameter. The following example (for MySQL) is based on a stored function named `get_actor_name` that returns an actor’s full name:

```sql
CREATE FUNCTION get_actor_name (in_id INTEGER)
RETURNS VARCHAR(200) READS SQL DATA
BEGIN
    DECLARE out_name VARCHAR(200);
    SELECT concat(first_name, ' ', last_name)
        INTO out_name
        FROM t_actor where id = in_id;
    RETURN out_name;
END;
```

To call this function, we again create a `SimpleJdbcCall` in the initialization method, as the following example shows:

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;
    private SimpleJdbcCall funcGetActorName;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.funcGetActorName = new SimpleJdbcCall(jdbcTemplate)
                .withFunctionName("get_actor_name");
    }

    public String getActorName(Long id) {
        SqlParameterSource in = new MapSqlParameterSource()
                .addValue("in_id", id);
        String name = funcGetActorName.executeFunction(String.class, in);
        return name;
    }

    // ... additional methods
}
```

The `executeFunction` method used returns a `String` that contains the return value from the function call.

#### 3.6.9. Returning a `ResultSet` or REF Cursor from a `SimpleJdbcCall`

Calling a stored procedure or function that returns a result set is a bit tricky. Some databases return result sets during the JDBC results processing, while others require an explicitly registered `out` parameter of a specific type. Both approaches need additional processing to loop over the result set and process the returned rows. With the `SimpleJdbcCall`, you can use the `returningResultSet` method and declare a `RowMapper` implementation to be used for a specific parameter. If the result set is returned during the results processing, there are no names defined, so the returned results must match the order in which you declare the `RowMapper` implementations. The name specified is still used to store the processed list of results in the results map that is returned from the `execute` statement.

The next example (for MySQL) uses a stored procedure that takes no IN parameters and returns all rows from the `t_actor` table:

```sql
CREATE PROCEDURE read_all_actors()
BEGIN
 SELECT a.id, a.first_name, a.last_name, a.birth_date FROM t_actor a;
END;
```

To call this procedure, you can declare the `RowMapper`. Because the class to which you want to map follows the JavaBean rules, you can use a `BeanPropertyRowMapper` that is created by passing in the required class to map to in the `newInstance` method. The following example shows how to do so:

Java

Kotlin

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadAllActors;

    public void setDataSource(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.procReadAllActors = new SimpleJdbcCall(jdbcTemplate)
                .withProcedureName("read_all_actors")
                .returningResultSet("actors",
                BeanPropertyRowMapper.newInstance(Actor.class));
    }

    public List getActorsList() {
        Map m = procReadAllActors.execute(new HashMap<String, Object>(0));
        return (List) m.get("actors");
    }

    // ... additional methods
}
```

The `execute` call passes in an empty `Map`, because this call does not take any parameters. The list of actors is then retrieved from the results map and returned to the caller.

### 3.7. Modeling JDBC Operations as Java Objects

The `org.springframework.jdbc.object` package contains classes that let you access the database in a more object-oriented manner. As an example, you can execute queries and get the results back as a list that contains business objects with the relational column data mapped to the properties of the business object. You can also run stored procedures and run update, delete, and insert statements.

|      | Many Spring developers believe that the various RDBMS operation classes described below (with the exception of the [`StoredProcedure`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-StoredProcedure) class) can often be replaced with straight `JdbcTemplate` calls. Often, it is simpler to write a DAO method that calls a method on a `JdbcTemplate` directly (as opposed to encapsulating a query as a full-blown class).However, if you are getting measurable value from using the RDBMS operation classes, you should continue to use these classes. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 3.7.1. Understanding `SqlQuery`

`SqlQuery` is a reusable, thread-safe class that encapsulates an SQL query. Subclasses must implement the `newRowMapper(..)` method to provide a `RowMapper` instance that can create one object per row obtained from iterating over the `ResultSet` that is created during the execution of the query. The `SqlQuery` class is rarely used directly, because the `MappingSqlQuery` subclass provides a much more convenient implementation for mapping rows to Java classes. Other implementations that extend `SqlQuery` are `MappingSqlQueryWithParameters` and `UpdatableSqlQuery`.

#### 3.7.2. Using `MappingSqlQuery`

`MappingSqlQuery` is a reusable query in which concrete subclasses must implement the abstract `mapRow(..)` method to convert each row of the supplied `ResultSet` into an object of the type specified. The following example shows a custom query that maps the data from the `t_actor` relation to an instance of the `Actor` class:

Java

Kotlin

```java
public class ActorMappingQuery extends MappingSqlQuery<Actor> {

    public ActorMappingQuery(DataSource ds) {
        super(ds, "select id, first_name, last_name from t_actor where id = ?");
        declareParameter(new SqlParameter("id", Types.INTEGER));
        compile();
    }

    @Override
    protected Actor mapRow(ResultSet rs, int rowNumber) throws SQLException {
        Actor actor = new Actor();
        actor.setId(rs.getLong("id"));
        actor.setFirstName(rs.getString("first_name"));
        actor.setLastName(rs.getString("last_name"));
        return actor;
    }
}
```

The class extends `MappingSqlQuery` parameterized with the `Actor` type. The constructor for this customer query takes a `DataSource` as the only parameter. In this constructor, you can call the constructor on the superclass with the `DataSource` and the SQL that should be executed to retrieve the rows for this query. This SQL is used to create a `PreparedStatement`, so it may contain placeholders for any parameters to be passed in during execution. You must declare each parameter by using the `declareParameter` method passing in an `SqlParameter`. The `SqlParameter` takes a name, and the JDBC type as defined in `java.sql.Types`. After you define all parameters, you can call the `compile()` method so that the statement can be prepared and later run. This class is thread-safe after it is compiled, so, as long as these instances are created when the DAO is initialized, they can be kept as instance variables and be reused. The following example shows how to define such a class:

Java

Kotlin

```java
private ActorMappingQuery actorMappingQuery;

@Autowired
public void setDataSource(DataSource dataSource) {
    this.actorMappingQuery = new ActorMappingQuery(dataSource);
}

public Customer getCustomer(Long id) {
    return actorMappingQuery.findObject(id);
}
```

The method in the preceding example retrieves the customer with the `id` that is passed in as the only parameter. Since we want only one object to be returned, we call the `findObject` convenience method with the `id` as the parameter. If we had instead a query that returned a list of objects and took additional parameters, we would use one of the `execute` methods that takes an array of parameter values passed in as varargs. The following example shows such a method:

Java

Kotlin

```java
public List<Actor> searchForActors(int age, String namePattern) {
    List<Actor> actors = actorSearchMappingQuery.execute(age, namePattern);
    return actors;
}
```

#### 3.7.3. Using `SqlUpdate`

The `SqlUpdate` class encapsulates an SQL update. As with a query, an update object is reusable, and, as with all `RdbmsOperation` classes, an update can have parameters and is defined in SQL. This class provides a number of `update(..)` methods analogous to the `execute(..)` methods of query objects. The `SQLUpdate` class is concrete. It can be subclassed — for example, to add a custom update method. However, you do not have to subclass the `SqlUpdate` class, since it can easily be parameterized by setting SQL and declaring parameters. The following example creates a custom update method named `execute`:

Java

Kotlin

```java
import java.sql.Types;
import javax.sql.DataSource;
import org.springframework.jdbc.core.SqlParameter;
import org.springframework.jdbc.object.SqlUpdate;

public class UpdateCreditRating extends SqlUpdate {

    public UpdateCreditRating(DataSource ds) {
        setDataSource(ds);
        setSql("update customer set credit_rating = ? where id = ?");
        declareParameter(new SqlParameter("creditRating", Types.NUMERIC));
        declareParameter(new SqlParameter("id", Types.NUMERIC));
        compile();
    }

    /**
     * @param id for the Customer to be updated
     * @param rating the new value for credit rating
     * @return number of rows updated
     */
    public int execute(int id, int rating) {
        return update(rating, id);
    }
}
```

#### 3.7.4. Using `StoredProcedure`

The `StoredProcedure` class is a superclass for object abstractions of RDBMS stored procedures. This class is `abstract`, and its various `execute(..)` methods have `protected` access, preventing use other than through a subclass that offers tighter typing.

The inherited `sql` property is the name of the stored procedure in the RDBMS.

To define a parameter for the `StoredProcedure` class, you can use an `SqlParameter` or one of its subclasses. You must specify the parameter name and SQL type in the constructor, as the following code snippet shows:

Java

Kotlin

```java
new SqlParameter("in_id", Types.NUMERIC),
new SqlOutParameter("out_first_name", Types.VARCHAR),
```

The SQL type is specified using the `java.sql.Types` constants.

The first line (with the `SqlParameter`) declares an IN parameter. You can use IN parameters both for stored procedure calls and for queries using the `SqlQuery` and its subclasses (covered in [Understanding `SqlQuery`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-SqlQuery)).

The second line (with the `SqlOutParameter`) declares an `out` parameter to be used in the stored procedure call. There is also an `SqlInOutParameter` for `InOut` parameters (parameters that provide an `in` value to the procedure and that also return a value).

For `in` parameters, in addition to the name and the SQL type, you can specify a scale for numeric data or a type name for custom database types. For `out` parameters, you can provide a `RowMapper` to handle mapping of rows returned from a `REF` cursor. Another option is to specify an `SqlReturnType` that lets you define customized handling of the return values.

The next example of a simple DAO uses a `StoredProcedure` to call a function (`sysdate()`), which comes with any Oracle database. To use the stored procedure functionality, you have to create a class that extends `StoredProcedure`. In this example, the `StoredProcedure` class is an inner class. However, if you need to reuse the `StoredProcedure`, you can declare it as a top-level class. This example has no input parameters, but an output parameter is declared as a date type by using the `SqlOutParameter` class. The `execute()` method runs the procedure and extracts the returned date from the results `Map`. The results `Map` has an entry for each declared output parameter (in this case, only one) by using the parameter name as the key. The following listing shows our custom StoredProcedure class:

Java

Kotlin

```java
import java.sql.Types;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import javax.sql.DataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.SqlOutParameter;
import org.springframework.jdbc.object.StoredProcedure;

public class StoredProcedureDao {

    private GetSysdateProcedure getSysdate;

    @Autowired
    public void init(DataSource dataSource) {
        this.getSysdate = new GetSysdateProcedure(dataSource);
    }

    public Date getSysdate() {
        return getSysdate.execute();
    }

    private class GetSysdateProcedure extends StoredProcedure {

        private static final String SQL = "sysdate";

        public GetSysdateProcedure(DataSource dataSource) {
            setDataSource(dataSource);
            setFunction(true);
            setSql(SQL);
            declareParameter(new SqlOutParameter("date", Types.DATE));
            compile();
        }

        public Date execute() {
            // the 'sysdate' sproc has no input parameters, so an empty Map is supplied...
            Map<String, Object> results = execute(new HashMap<String, Object>());
            Date sysdate = (Date) results.get("date");
            return sysdate;
        }
    }

}
```

The following example of a `StoredProcedure` has two output parameters (in this case, Oracle REF cursors):

Java

Kotlin

```java
import java.util.HashMap;
import java.util.Map;
import javax.sql.DataSource;
import oracle.jdbc.OracleTypes;
import org.springframework.jdbc.core.SqlOutParameter;
import org.springframework.jdbc.object.StoredProcedure;

public class TitlesAndGenresStoredProcedure extends StoredProcedure {

    private static final String SPROC_NAME = "AllTitlesAndGenres";

    public TitlesAndGenresStoredProcedure(DataSource dataSource) {
        super(dataSource, SPROC_NAME);
        declareParameter(new SqlOutParameter("titles", OracleTypes.CURSOR, new TitleMapper()));
        declareParameter(new SqlOutParameter("genres", OracleTypes.CURSOR, new GenreMapper()));
        compile();
    }

    public Map<String, Object> execute() {
        // again, this sproc has no input parameters, so an empty Map is supplied
        return super.execute(new HashMap<String, Object>());
    }
}
```

Notice how the overloaded variants of the `declareParameter(..)` method that have been used in the `TitlesAndGenresStoredProcedure` constructor are passed `RowMapper` implementation instances. This is a very convenient and powerful way to reuse existing functionality. The next two examples provide code for the two `RowMapper` implementations.

The `TitleMapper` class maps a `ResultSet` to a `Title` domain object for each row in the supplied `ResultSet`, as follows:

Java

Kotlin

```java
import java.sql.ResultSet;
import java.sql.SQLException;
import com.foo.domain.Title;
import org.springframework.jdbc.core.RowMapper;

public final class TitleMapper implements RowMapper<Title> {

    public Title mapRow(ResultSet rs, int rowNum) throws SQLException {
        Title title = new Title();
        title.setId(rs.getLong("id"));
        title.setName(rs.getString("name"));
        return title;
    }
}
```

The `GenreMapper` class maps a `ResultSet` to a `Genre` domain object for each row in the supplied `ResultSet`, as follows:

Java

Kotlin

```java
import java.sql.ResultSet;
import java.sql.SQLException;
import com.foo.domain.Genre;
import org.springframework.jdbc.core.RowMapper;

public final class GenreMapper implements RowMapper<Genre> {

    public Genre mapRow(ResultSet rs, int rowNum) throws SQLException {
        return new Genre(rs.getString("name"));
    }
}
```

To pass parameters to a stored procedure that has one or more input parameters in its definition in the RDBMS, you can code a strongly typed `execute(..)` method that would delegate to the untyped `execute(Map)` method in the superclass, as the following example shows:

Java

Kotlin

```java
import java.sql.Types;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import javax.sql.DataSource;
import oracle.jdbc.OracleTypes;
import org.springframework.jdbc.core.SqlOutParameter;
import org.springframework.jdbc.core.SqlParameter;
import org.springframework.jdbc.object.StoredProcedure;

public class TitlesAfterDateStoredProcedure extends StoredProcedure {

    private static final String SPROC_NAME = "TitlesAfterDate";
    private static final String CUTOFF_DATE_PARAM = "cutoffDate";

    public TitlesAfterDateStoredProcedure(DataSource dataSource) {
        super(dataSource, SPROC_NAME);
        declareParameter(new SqlParameter(CUTOFF_DATE_PARAM, Types.DATE);
        declareParameter(new SqlOutParameter("titles", OracleTypes.CURSOR, new TitleMapper()));
        compile();
    }

    public Map<String, Object> execute(Date cutoffDate) {
        Map<String, Object> inputs = new HashMap<String, Object>();
        inputs.put(CUTOFF_DATE_PARAM, cutoffDate);
        return super.execute(inputs);
    }
}
```

### 3.8. Common Problems with Parameter and Data Value Handling

Common problems with parameters and data values exist in the different approaches provided by Spring Framework’s JDBC support. This section covers how to address them.

#### 3.8.1. Providing SQL Type Information for Parameters

Usually, Spring determines the SQL type of the parameters based on the type of parameter passed in. It is possible to explicitly provide the SQL type to be used when setting parameter values. This is sometimes necessary to correctly set `NULL` values.

You can provide SQL type information in several ways:

- Many update and query methods of the `JdbcTemplate` take an additional parameter in the form of an `int` array. This array is used to indicate the SQL type of the corresponding parameter by using constant values from the `java.sql.Types` class. Provide one entry for each parameter.
- You can use the `SqlParameterValue` class to wrap the parameter value that needs this additional information. To do so, create a new instance for each value and pass in the SQL type and the parameter value in the constructor. You can also provide an optional scale parameter for numeric values.
- For methods that work with named parameters, you can use the `SqlParameterSource` classes, `BeanPropertySqlParameterSource` or `MapSqlParameterSource`. They both have methods for registering the SQL type for any of the named parameter values.

#### 3.8.2. Handling BLOB and CLOB objects处理BLOB和CLOB对象

您可以在数据库中存储图像，其他二进制数据和大块文本。这些大对象被称为用于二进制数据的BLOB（二进制大对象）和用于字符数据的CLOB（字符大对象）。在Spring中，您可以`JdbcTemplate`直接使用RDBMS Objects和`SimpleJdbc`类提供的高级抽象，也可以直接使用这些对象。所有这些方法都使用`LobHandler`接口的实现来对LOB（大对象）数据进行实际管理。 通过该方法`LobHandler`提供对`LobCreator`类的访问，该类`getLobCreator`用于创建要插入的新LOB对象。

`LobCreator`并`LobHandler`为LOB输入和输出提供以下支持：

- BLOB
  - `byte[]`：`getBlobAsBytes`和`setBlobAsBytes`
  - `InputStream`：`getBlobAsBinaryStream`和`setBlobAsBinaryStream`
- CLOB
  - `String`：`getClobAsString`和`setClobAsString`
  - `InputStream`：`getClobAsAsciiStream`和`setClobAsAsciiStream`
  - `Reader`：`getClobAsCharacterStream`和`setClobAsCharacterStream`

下一个示例显示了如何创建和插入BLOB。稍后，我们展示如何从数据库中读取它。

本示例使用`JdbcTemplate`和的实现 `AbstractLobCreatingPreparedStatementCallback`。它实现了一种方法 `setValues`。此方法提供了一个`LobCreator`我们用来设置SQL插入语句中LOB列的值的方法。

对于此示例，我们假设存在一个变量，该变量`lobHandler`已设置为的实例`DefaultLobHandler`。通常，您可以通过依赖注入来设置此值。

以下示例显示如何创建和插入BLOB：

爪哇

科特林

```java
final File blobIn = new File("spring2004.jpg");
final InputStream blobIs = new FileInputStream(blobIn);
final File clobIn = new File("large.txt");
final InputStream clobIs = new FileInputStream(clobIn);
final InputStreamReader clobReader = new InputStreamReader(clobIs);

jdbcTemplate.execute(
    "INSERT INTO lob_table (id, a_clob, a_blob) VALUES (?, ?, ?)",
    new AbstractLobCreatingPreparedStatementCallback(lobHandler) {  //1
        protected void setValues(PreparedStatement ps, LobCreator lobCreator) throws SQLException {
            ps.setLong(1, 1L);
            lobCreator.setClobAsCharacterStream(ps, 2, clobReader, (int)clobIn.length());  //2
            lobCreator.setBlobAsBinaryStream(ps, 3, blobIs, (int)blobIn.length());  //3
        }
    }
);

blobIs.close();
clobReader.close();
```

| 1    | 传递`lobHandler`that（在此示例中）为plain `DefaultLobHandler`。 |
| ---- | ------------------------------------------------------------ |
| 2    | 使用该方法`setClobAsCharacterStream`来传递CLOB的内容。       |
| 3    | 使用该方法`setBlobAsBinaryStream`来传递BLOB的内容。          |

![1609491857206](assets/1609491857206.png)

现在是时候从数据库中读取LOB数据了。再次，您使用`JdbcTemplate` 具有相同实例变量的`lobHandler`以及对的引用`DefaultLobHandler`。以下示例显示了如何执行此操作：

爪哇

科特林

```java
List<Map<String, Object>> l = jdbcTemplate.query("select id, a_clob, a_blob from lob_table",
    new RowMapper<Map<String, Object>>() {
        public Map<String, Object> mapRow(ResultSet rs, int i) throws SQLException {
            Map<String, Object> results = new HashMap<String, Object>();
            String clobText = lobHandler.getClobAsString(rs, "a_clob");  //1
            results.put("CLOB", clobText);
            byte[] blobBytes = lobHandler.getBlobAsBytes(rs, "a_blob");  //2
            results.put("BLOB", blobBytes);
            return results;
        }
    });
```

| 1    | 使用该方法`getClobAsString`检索CLOB的内容。 |
| ---- | ------------------------------------------- |
| 2    | 使用该方法`getBlobAsBytes`检索BLOB的内容。  |

#### 3.8.3. Passing in Lists of Values for IN Clause传入IN子句的值列表

SQL标准允许基于包含变量值列表的表达式选择行。一个典型的例子是`select * from T_ACTOR where id in (1, 2, 3)`。JDBC标准不直接为准备好的语句支持此变量列表。您不能声明可变数量的占位符。您需要准备好所需数量的占位符的多种变体，或者一旦知道需要多少个占位符，就需要动态生成SQL字符串。`NamedParameterJdbcTemplate`和中提供的命名参数支持`JdbcTemplate`采用后一种方法。您可以将值作为`java.util.List`原始对象传递。该列表用于插入所需的占位符，并在语句执行期间传递值。

![1609491959520](assets/1609491959520.png)

除了值列表中的原始值之外，您还可以创建一个`java.util.List` 对象数组。此列表可以支持为`in` 子句定义的多个表达式，例如`select * from T_ACTOR where (id, last_name) in ((1, 'Johnson'), (2, 'Harrop'\))`。当然，这要求您的数据库支持此语法。

#### 3.8.4. Handling Complex Types for Stored Procedure Calls处理存储过程调用的复杂类型

调用存储过程时，有时可以使用特定于数据库的复杂类型。为了适应这些类型，`SqlReturnType`当它们从存储过程调用返回`SqlTypeValue`时以及当它们作为参数传递到存储过程时，Spring提供了一个处理它们的方法。

该`SqlReturnType`接口具有`getTypeValue`必须实现的单个方法（名为）。此接口用作的声明的一部分`SqlOutParameter`。以下示例显示了返回`STRUCT`用户声明类型的Oracle对象的值`ITEM_TYPE`：

```java
public class TestItemStoredProcedure extends StoredProcedure {

    public TestItemStoredProcedure(DataSource dataSource) {
        // ...
        declareParameter(new SqlOutParameter("item", OracleTypes.STRUCT, "ITEM_TYPE",
            (CallableStatement cs, int colIndx, int sqlType, String typeName) -> {
                STRUCT struct = (STRUCT) cs.getObject(colIndx);
                Object[] attr = struct.getAttributes();
                TestItem item = new TestItem();
                item.setId(((Number) attr[0]).longValue());
                item.setDescription((String) attr[1]);
                item.setExpirationDate((java.util.Date) attr[2]);
                return item;
            }));
        // ...
    }
```

您可以`SqlTypeValue`用来将Java对象（例如`TestItem`）的值传递给存储过程。该`SqlTypeValue`接口具有`createTypeValue`必须实现的单个方法（名为 ）。活动连接被传入，您可以使用它来创建特定于数据库的对象，例如`StructDescriptor`实例或`ArrayDescriptor`实例。以下示例创建一个`StructDescriptor`实例：

```java
final TestItem testItem = new TestItem(123L, "A test item",
        new SimpleDateFormat("yyyy-M-d").parse("2010-12-31"));

SqlTypeValue value = new AbstractSqlTypeValue() {
    protected Object createTypeValue(Connection conn, int sqlType, String typeName) throws SQLException {
        StructDescriptor itemDescriptor = new StructDescriptor(typeName, conn);
        Struct item = new STRUCT(itemDescriptor, conn,
        new Object[] {
            testItem.getId(),
            testItem.getDescription(),
            new java.sql.Date(testItem.getExpirationDate().getTime())
        });
        return item;
    }
};
```

现在，您可以将其添加`SqlTypeValue`到，`Map`其中包含用于`execute`存储过程调用的输入参数 。

的另一个用途`SqlTypeValue`是将值数组传递给Oracle存储过程。`ARRAY`在这种情况下，Oracle具有自己的内部类，您可以使用`SqlTypeValue`来创建Oracle的实例，`ARRAY`并使用Java中的值填充该实例`ARRAY`，如以下示例所示：



```java
final Long[] ids = new Long[] {1L, 2L};

SqlTypeValue value = new AbstractSqlTypeValue() {
    protected Object createTypeValue(Connection conn, int sqlType, String typeName) throws SQLException {
        ArrayDescriptor arrayDescriptor = new ArrayDescriptor(typeName, conn);
        ARRAY idArray = new ARRAY(arrayDescriptor, conn, ids);
        return idArray;
    }
};
```

### 3.9. Embedded Database Support嵌入式数据库支持

该`org.springframework.jdbc.datasource.embedded`软件包提供对嵌入式Java数据库引擎的支持。本地提供对[HSQL](http://www.hsqldb.org/)， [H2](https://www.h2database.com/)和[Derby的](https://db.apache.org/derby)支持。您还可以使用可扩展的API来插入新的嵌入式数据库类型和 `DataSource`实现。

#### 3.9.1. Why Use an Embedded Database?为什么要使用嵌入式数据库？

嵌入式数据库由于其轻量级的特性，因此在项目的开发阶段可能会很有用。好处包括易于配置，启动时间短，可测试性以及在开发过程中快速演化SQL的能力。

#### 3.9.2. Creating an Embedded Database by Using Spring XML使用Spring XML创建嵌入式数据库

如果要在Spring中将嵌入式数据库实例作为Bean公开 `ApplicationContext`，则可以`embedded-database`在`spring-jdbc`名称空间中使用标记：

```xml
<jdbc:embedded-database id="dataSource" generate-name="true">
    <jdbc:script location="classpath:schema.sql"/>
    <jdbc:script location="classpath:test-data.sql"/>
</jdbc:embedded-database>
```

前面的配置创建了一个嵌入式HSQL数据库，该数据库中填充了来自类路径根目录中`schema.sql`和`test-data.sql`资源的SQL 。另外，作为最佳实践，将为嵌入式数据库分配一个唯一生成的名称。嵌入式数据库作为类型的bean提供给Spring容器 `javax.sql.DataSource`，然后可以根据需要将其注入到数据访问对象中。

#### 3.9.3. Creating an Embedded Database Programmatically以编程方式创建嵌入式数据库

`EmbeddedDatabaseBuilder`类提供一个流畅API以编程构造的嵌入式数据库。当您需要在独立环境或独立集成测试中创建嵌入式数据库时，可以使用此方法，如以下示例所示：

```java
EmbeddedDatabase db = new EmbeddedDatabaseBuilder()
        .generateUniqueName(true)
        .setType(H2)
        .setScriptEncoding("UTF-8")
        .ignoreFailedDrops(true)
        .addScript("schema.sql")
        .addScripts("user_data.sql", "country_data.sql")
        .build();

// perform actions against the db (EmbeddedDatabase extends javax.sql.DataSource)

db.shutdown()
```

有关 所有支持的选项的更多详细信息，请参见[javadoc`EmbeddedDatabaseBuilder`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/jdbc/datasource/embedded/EmbeddedDatabaseBuilder.html)。

您还可以`EmbeddedDatabaseBuilder`通过Java配置使用来创建嵌入式数据库，如以下示例所示：

```java
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
                .generateUniqueName(true)
                .setType(H2)
                .setScriptEncoding("UTF-8")
                .ignoreFailedDrops(true)
                .addScript("schema.sql")
                .addScripts("user_data.sql", "country_data.sql")
                .build();
    }
}
```

#### 3.9.4. Selecting the Embedded Database Type选择嵌入式数据库类型

本节介绍如何选择Spring支持的三个嵌入式数据库之一。它包括以下主题：

- [使用HSQL](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-embedded-database-using-HSQL)
- [使用H2](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-embedded-database-using-H2)
- [使用德比](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-embedded-database-using-Derby)

##### 使用HSQL

Spring支持HSQL 1.8.0及更高版本。如果未明确指定类型，则HSQL是默认的嵌入式数据库。要明确指定HSQL，请将标记的`type`属性 设置`embedded-database`为`HSQL`。如果您使用构建器API，请使用调用 `setType(EmbeddedDatabaseType)`方法`EmbeddedDatabaseType.HSQL`。

##### 使用H2

Spring支持H2数据库。要启用H2，请将标签的`type`属性 设置`embedded-database`为`H2`。如果您使用构建器API，请使用调用 `setType(EmbeddedDatabaseType)`方法`EmbeddedDatabaseType.H2`。

##### 使用derby

Spring支持Apache Derby 10.5及更高版本。要启用Derby，请将标签的`type` 属性设置`embedded-database`为`DERBY`。如果您使用构建器API，请使用调用`setType(EmbeddedDatabaseType)`方法`EmbeddedDatabaseType.DERBY`。

#### 3.9.5. Testing Data Access Logic with an Embedded Database使用嵌入式数据库测试数据访问逻辑

嵌入式数据库提供了一种轻量级的方法来测试数据访问代码。下一个示例是使用嵌入式数据库的数据访问集成测试模板。当嵌入式数据库不需要在测试类之间重用时，使用这种模板可以一次性使用。但是，如果您希望创建在测试套件中共享的嵌入式数据库，请考虑使用[Spring TestContext Framework](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#testcontext-framework)并将嵌入式数据库配置为Spring中的Bean，`ApplicationContext`如[使用Spring XML](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-embedded-database-xml)[创建](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-embedded-database-java)[嵌入式数据库](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-embedded-database-xml)和[创建](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-embedded-database-java)[嵌入式数据库中](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-embedded-database-xml)所述。[以编程方式数据库](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-embedded-database-java)。以下清单显示了测试模板：



```java
public class DataAccessIntegrationTestTemplate {

    private EmbeddedDatabase db;

    @BeforeEach
    public void setUp() {
        // creates an HSQL in-memory database populated from default scripts
        // classpath:schema.sql and classpath:data.sql
        db = new EmbeddedDatabaseBuilder()
                .generateUniqueName(true)
                .addDefaultScripts()
                .build();
    }

    @Test
    public void testDataAccess() {
        JdbcTemplate template = new JdbcTemplate(db);
        template.query( /* ... */ );
    }

    @AfterEach
    public void tearDown() {
        db.shutdown();
    }

}
```

#### 3.9.6. Generating Unique Names for Embedded Databases为嵌入式数据库生成唯一名称

如果开发团队的测试套件无意中尝试重新创建同一数据库的其他实例，则开发团队经常会遇到错误。如果XML配置文件或`@Configuration`类负责创建嵌入式数据库，然后在同一测试套件（即，同一JVM进程）内的多个测试场景中重用相应的配置，例如集成，则这很容易发生。对嵌入式数据库进行测试，这些嵌入式数据库的 `ApplicationContext`配置仅在激活哪些Bean定义配置文件方面有所不同。

此类错误的根本原因是，Spring `EmbeddedDatabaseFactory`（由`<jdbc:embedded-database>`XML命名空间元素和`EmbeddedDatabaseBuilder`for Java配置在 内部使用）将嵌入式数据库的名称设置为， `testdb`除非另行指定。对于`<jdbc:embedded-database>`，通常为嵌入式数据库分配一个与Bean相同的名称`id`（通常是`dataSource`）。因此，随后创建嵌入式数据库的尝试不会产生新的数据库。取而代之的是，相同的JDBC连接URL被重用，并且尝试创建新的嵌入式数据库实际上指向的是从相同配置创建的现有嵌入式数据库。

为了解决这个常见问题，Spring Framework 4.2提供了对生成嵌入式数据库的唯一名称的支持。要启用生成名称的使用，请使用以下选项之一。

- `EmbeddedDatabaseFactory.setGenerateUniqueDatabaseName()`

- `EmbeddedDatabaseBuilder.generateUniqueName()`

- `<jdbc:embedded-database generate-name="true" … >`

  

#### 3.9.7. Extending the Embedded Database Support扩展嵌入式数据库支持

您可以通过两种方式扩展Spring JDBC嵌入式数据库的支持：

- 实施`EmbeddedDatabaseConfigurer`以支持新的嵌入式数据库类型。
- 实现`DataSourceFactory`以支持新的`DataSource`实现，例如用于管理嵌入式数据库连接的连接池。

我们鼓励您在[GitHub Issues](https://github.com/spring-projects/spring-framework/issues)上为Spring社区贡献扩展 。

###  3.10. 初始化一个`DataSource`扩展嵌入式数据库支持

该`org.springframework.jdbc.datasource.init`软件包提供了对现有的初始化的支持`DataSource`。嵌入式数据库支持为创建和初始化`DataSource`应用程序提供了一个选项。但是，有时您可能需要初始化在某处的服务器上运行的实例。

#### 3.10.1. Initializing a Database by Using Spring XML使用Spring XML初始化数据库

如果要初始化数据库，并且可以提供对`DataSource` bean的引用，则可以`initialize-database`在`spring-jdbc`名称空间中使用标记：

```xml
<jdbc:initialize-database data-source="dataSource">
    <jdbc:script location="classpath:com/foo/sql/db-schema.sql"/>
    <jdbc:script location="classpath:com/foo/sql/db-test-data.sql"/>
</jdbc:initialize-database>
```

前面的示例对数据库运行两个指定的脚本。第一个脚本创建模式，第二个脚本用测试数据集填充表。脚本位置也可以是带有通配符的模式，这些通配符具有用于Spring中资源的常用Ant样式（例如 `classpath*:/com/foo/**/sql/*-data.sql`）。如果使用模式，则脚本以其URL或文件名的词法顺序运行。

数据库初始化程序的默认行为是无条件运行提供的脚本。这可能并不总是您想要的—例如，如果您对已经有测试数据的数据库运行脚本。通过遵循先创建表然后插入数据的通用模式（如前所示），可以减少意外删除数据的可能性。如果表已经存在，则第一步失败。

但是，为了更好地控制现有数据的创建和删除，XML名称空间提供了一些其他选项。第一个是用于打开和关闭初始化的标志。您可以根据环境进行设置（例如，从系统属性或环境Bean中获取布尔值）。以下示例从系统属性获取值：

```xml
<jdbc:initialize-database data-source="dataSource"
    enabled="#{systemProperties.INITIALIZE_DATABASE}"> //1
    <jdbc:script location="..."/>
</jdbc:initialize-database>
```

| 1    | `enabled`从名为的系统属性中获取的值`INITIALIZE_DATABASE`。 |
| ---- | ---------------------------------------------------------- |
|      |                                                            |

控制现有数据发生情况的第二种选择是更容忍故障。为此，您可以控制初始化程序忽略脚本执行的SQL中某些错误的能力，如以下示例所示：

```xml
<jdbc:initialize-database data-source="dataSource" ignore-failures="DROPS">
    <jdbc:script location="..."/>
</jdbc:initialize-database>
```

在前面的示例中，我们说我们期望有时脚本是针对空数据库运行的，并且脚本中有一些`DROP`语句因此会失败。因此失败的SQL`DROP`语句将被忽略，但其他失败将导致异常。如果您的SQL方言不支持`DROP … IF EXISTS`（或类似），但是您想要在重新创建它之前无条件地删除所有测试数据，这将很有用。在这种情况下，第一个脚本通常是一组`DROP`语句，然后是一组`CREATE`语句。

该`ignore-failures`选项可以设置为`NONE`（默认值），`DROPS`（忽略失败的丢弃）或`ALL`（忽略所有失败）。

`;`如果`;`脚本中根本没有该字符，则每个语句都应以或换行。您可以全局控制该脚本，也可以按脚本控制，如以下示例所示：

```xml
<jdbc:initialize-database data-source="dataSource" separator="@@"> //1
    <jdbc:script location="classpath:com/myapp/sql/db-schema.sql" separator=";"/> //2
    <jdbc:script location="classpath:com/myapp/sql/db-test-data-1.sql"/>
    <jdbc:script location="classpath:com/myapp/sql/db-test-data-2.sql"/>
</jdbc:initialize-database>
```

| 1    | 将分隔符脚本设置为`@@`。         |
| ---- | -------------------------------- |
| 2    | 设置分隔符`db-schema.sql`来`;`。 |

在这个例子中，这两个`test-data`脚本使用`@@`的语句分隔符，只有`db-schema.sql`用途`;`。此配置指定默认分隔符为，`@@`并覆盖`db-schema`脚本的默认分隔符。

如果您需要比从XML名称空间获得更多控制权，则可以`DataSourceInitializer`直接使用 Direct并将其定义为应用程序中的组件。

##### 初始化依赖于数据库的其他组件

大量的应用程序（那些在Spring上下文启动之后才使用数据库的应用程序）可以使用数据库初始化程序，而不会带来更多麻烦。如果您的应用程序不是其中之一，则可能需要阅读本节的其余部分。

数据库初始化程序取决于`DataSource`实例，并运行其初始化回调中提供的脚本（类似于`init-method`XML bean定义中的，`@PostConstruct`组件中的`afterPropertiesSet()` 方法或实现的组件中的方法`InitializingBean`）。如果其他bean依赖于相同的数据源并在初始化回调中使用该数据源，则可能存在问题，因为数据尚未初始化。一个常见的例子是一个高速缓存，它会在应用程序启动时急于初始化并从数据库加载数据。

要解决此问题，您有两个选择：将高速缓存初始化策略更改为下一个阶段，或者确保首先初始化数据库初始化程序。

如果应用程序在您的控制之下，则更改缓存初始化策略可能很容易，否则就不那么容易。有关如何实现这一点的一些建议包括：

- 使缓存在首次使用时延迟初始化，从而缩短了应用程序的启动时间。
- 让您的缓存或单独的组件初始化缓存实现 `Lifecycle`或`SmartLifecycle`。当应用程序上下文启动时，您可以`SmartLifecycle`通过设置其`autoStartup`标志来自动启动a ，并且可以`Lifecycle`通过调用`ConfigurableApplicationContext.start()` 封闭的上下文来手动启动a 。
- 使用Spring`ApplicationEvent`或类似的自定义观察器机制来触发缓存初始化。`ContextRefreshedEvent`通常在准备好使用时（在所有bean都初始化之后）由上下文发布，因此通常是一个有用的钩子（`SmartLifecycle`默认情况下，这是这样工作的）。

确保首先初始化数据库初始化程序也很容易。关于如何实现这一点的一些建议包括：

- 依靠Spring的默认行为`BeanFactory`，即按注册顺序初始化bean。您可以通过采用`<import/>`XML配置中一组对应用程序模块进行排序的元素的通用做法，并确保首先列出数据库和数据库初始化，来轻松地进行安排。
- 将`DataSource`和使用它的业务组件分开，并通过将它们放在单独的`ApplicationContext`实例中来控制它们的启动顺序（例如，父上下文包含`DataSource`，子上下文包含这些业务组件）。这种结构在Spring Web应用程序中很常见，但可以更广泛地应用。