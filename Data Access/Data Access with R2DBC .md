## 4. Data Access with R2DBC 使用R2DBC进行数据访问

R2DBC（“反应式关系数据库连接”）是一个社区驱动的规范工作，旨在使用反应式模式标准化对SQL数据库的访问。

### 4.1. Package Hierarchy 包层次结构

Spring框架的R2DBC抽象框架由两个不同的包组成：

- core:org.springframework.r2dbc.core包含DatabaseClient类和各种相关类。请参阅使用R2DBC核心类控制基本R2DBC处理和错误处理。

- connection:org.springframework.r2dbc.connection包含一个用于轻松访问ConnectionFactory的实用程序类和各种简单的ConnectionFactory实现类，您可以使用这些实现来测试和运行未修改的R2DBC。请参见控制数据库连接。

### 4.2. Using the R2DBC Core Classes to Control Basic R2DBC Processing and Error Handling 使用R2DBC核心类控制基本的R2DBC处理和错误处理

本节介绍如何使用R2DBC核心类来控制基本的R2DBC处理，包括错误处理。它包括以下主题：

- 使用DatabaseClient

- 执行语句

- 查询（选择）

- 使用DatabaseClient更新（插入、更新和删除）

- 语句筛选器

- 检索自动生成的密钥

#### 4.2.1. Using DatabaseClient  使用DatabaseClient

DatabaseClient是R2DBC核心包中的中心类。它处理资源的创建和释放，这有助于避免常见错误，例如忘记关闭连接。它执行核心R2DBC工作流的基本任务（例如语句创建和执行），让应用程序代码提供SQL和提取结果。DatabaseClient类：

- 运行SQL查询

- 更新语句和存储过程调用

- 对结果实例执行迭代

- 捕获R2DBC异常并将其转换为org.springframework.dao包裹。（请参见一致的异常层次结构。）

客户机有一个功能性的、流畅的API，使用反应式类型进行声明性合成。

当您将DatabaseClient用于代码时，只需实现java.util.function函数接口，给他们一个明确定义的协议。给定DatabaseClient类提供的连接，函数回调将创建一个发布服务器。对于提取行结果的映射函数也是如此。

可以通过直接实例化ConnectionFactory引用在DAO类中使用DatabaseClient，也可以在springioc容器中配置DatabaseClient，并将其作为bean引用提供给DAOs。

创建DatabaseClient对象的最简单方法是通过静态工厂方法，如下所示：

```java
DatabaseClient client = DatabaseClient.create(connectionFactory);
```

```
ConnectionFactory应该始终配置为Spring IoC容器中的bean。
```

前面的方法使用默认设置创建DatabaseClient。

您还可以从数据库客户端.builder(). 您可以通过调用以下方法自定义客户端：

….bindMarkers(…): 提供特定的BindMarkersFactory以配置命名参数到数据库绑定标记转换。

….executeFunction(…): 设置ExecuteFunction语句对象的运行方式。

….namedParameters(false): 禁用命名参数扩展。默认启用。

```
方言由BindMarkersFactoryResolver从ConnectionFactory解析，通常通过检查ConnectionFactoryMetadata。
您可以通过注册一个实现org.springframework.r2dbc.core.binding.BindMarkersFactoryResolver$BindMarkerFactoryProvider通过META-INF/spring.factories。BindMarkersFactoryResolver使用Spring的SpringFactoriesLoader从类路径发现绑定标记提供程序实现。
```

当前支持的数据库包括：

- H2

- MariaDB

- Microsoft SQL Server

- MySQL

- Postgres

这个类发出的所有SQL都记录在与客户端实例的完全限定类名（通常是DefaultDatabaseClient）相对应的类别下的调试级别。此外，每次执行在反应序列中注册一个检查点，以帮助调试。

下面几节提供了一些DatabaseClient用法的示例。这些示例并不是DatabaseClient公开的所有功能的详尽列表。请参阅相关的javadoc。

##### Executing Statements 执行语句

DatabaseClient提供运行语句的基本功能。以下示例显示了创建新表的最少但功能齐全的代码需要包含的内容：

```
Mono<Void> completion = client.sql("CREATE TABLE person (id VARCHAR(255) PRIMARY KEY, name VARCHAR(255), age INTEGER);")
        .then();
```

DatabaseClient是为方便、流畅的使用而设计的。它在执行规范的每个阶段公开中间方法、延续方法和终端方法。上面的示例使用then（）返回一个完成发布服务器，该发布服务器在查询（或查询，如果SQL查询包含多个语句）完成时立即完成。

```
execute(…)接受SQL查询字符串或Supplier<String>将实际查询创建推迟到执行。
```

##### Querying (`SELECT`) 查询（选择）

SQL查询可以通过行对象或受影响的行数返回值。DatabaseClient可以返回更新的行数或行本身，具体取决于发出的查询。

以下查询从表中获取id和name列：

```java
Mono<Map<String, Object>> first = client.sql("SELECT id, name FROM person")
        .fetch().first();
```

以下查询使用绑定变量：

```
Mono<Map<String, Object>> first = client.sql("SELECT id, name FROM person WHERE first_name = :fn")
        .bind("fn", "Joe")
        .fetch().first();
```

您可能已经注意到在上面的示例中使用了fetch（）。fetch（）是一个延续运算符，它允许您指定要使用的数据量。

调用first（）返回结果中的第一行并丢弃其余行。可以使用以下运算符使用数据：

- first()  返回整个结果的第一行。对于不可为null的返回值，它的Kotlin协程变量命名为awaitSingle（），如果该值是可选的，则命名为awaitSingleOrNull（）。

- one() 只返回一个结果，如果结果包含更多行，则返回失败。使用Kotlin协程，只对一个值使用awaitOne（），如果值可能为null，则使用awaitOneOrNull（）。

- all() 返回结果的所有行。使用Kotlin协程时，请使用flow（）。

- rowsUpdated() 返回受影响的行数（插入/更新/删除计数）。它的Kotlin协程变体名为awaitRowSupdate（）。

在不指定进一步的映射细节的情况下，查询将表格式结果作为映射返回，其键是映射到其列值的不区分大小写的列名。

您可以通过为每一行提供一个函数<Row，T>来控制结果映射，这样它就可以返回任意值（单数值、集合和映射以及对象）。

以下示例提取id列并发出其值：

```
Flux<String> names = client.sql("SELECT name FROM person")
        .map(row -> row.get("id", String.class))
        .all();
```

```
                        空值呢？
关系数据库结果可以包含空值。反应流规范禁止零值的发射。这个要求要求在提取器函数中进行正确的空处理。虽然可以从行中获取空值，但不能发出空值。必须将任何空值包装在对象中（例如，对于单数值是可选的），以确保提取函数不会直接返回空值。
```

##### Updating (INSERT, UPDATE, and DELETE) with DatabaseClient 使用DatabaseClient更新（插入、更新和删除）

修改语句的唯一区别在于，这些语句通常不返回表格数据，因此您可以使用rowsupdate（）来使用结果。

以下示例显示了一个UPDATE语句，该语句返回已更新的行数：

```java
Mono<Integer> affectedRows = client.sql("UPDATE person SET first_name = :fn")
        .bind("fn", "Joe")
        .fetch().rowsUpdated();
```

##### Binding Values to Queries 将值绑定到查询

典型的应用程序需要参数化的SQL语句来根据某些输入选择或更新行。这些语句通常是由WHERE子句或接受输入参数的INSERT和UPDATE语句约束的SELECT语句。如果参数没有正确转义，参数化语句将承担SQL注入的风险。DatabaseClient利用R2DBC的bindapi来消除查询参数的SQL注入风险。可以使用execute（…）运算符提供参数化SQL语句，并将参数绑定到实际语句。然后R2DBC驱动程序通过使用准备好的语句和参数替换来运行该语句。

参数绑定支持两种绑定策略：

- 按索引，使用从零开始的参数索引。

- 按名称，使用占位符名称。

以下示例显示查询的参数绑定：

```java
db.sql("INSERT INTO person (id, name, age) VALUES(:id, :name, :age)")
    .bind("id", "joe")
    .bind("name", "Joe")
    .bind("age", 34);
```

```
                    R2DBC本机绑定标记
R2DBC使用依赖于实际数据库供应商的数据库本机绑定标记。例如，Postgres使用索引标记，如$1、$2、$n。另一个示例是SQL Server，它使用前缀为@的命名绑定标记。
这与JDBC不同，JDBC需要？作为绑定标记。在JDBC中，实际的驱动程序是转义的？作为语句执行的一部分，将标记绑定到数据库本机标记。
springframework的R2DBC支持允许您使用本机绑定标记或带有：name语法的命名绑定标记。
命名参数支持利用BindMarkersFactory实例在查询执行时将命名参数扩展为本机绑定标记，这为您提供了跨不同数据库供应商的某种程度的查询可移植性。
```

查询预处理器将命名集合参数展开为一系列绑定标记，以消除基于参数数量动态创建查询的需要。嵌套对象数组被展开以允许使用（例如）选择列表。

考虑以下查询：

```sql
SELECT id, name, state FROM table WHERE (name, age) IN (('John', 35), ('Ann', 50))
```

前面的查询可以参数化并按如下方式运行：

```java
List<Object[]> tuples = new ArrayList<>();
tuples.add(new Object[] {"John", 35});
tuples.add(new Object[] {"Ann",  50});

client.sql("SELECT id, name, state FROM table WHERE (name, age) IN (:tuples)")
    .bind("tuples", tuples);
```

```
选择列表的使用取决于供应商。
```

以下示例显示了使用IN谓词的更简单的变量：

```java
client.sql("SELECT id, name, state FROM table WHERE age IN (:ages)")
    .bind("ages", Arrays.asList(35, 50));
```

```
R2DBC本身不支持类集合的值。尽管如此，在上面的例子中扩展一个给定的列表对于Spring的R2DBC支持中的命名参数是有效的，例如在上面所示的in子句中使用。但是，插入或更新数组类型的列（例如在Postgres中）需要底层R2DBC驱动程序支持的数组类型：通常是Java数组，例如String[]来更新text[]列。不要将Collection<String>等作为数组参数传递。
```

##### Statement Filters 语句筛选器

有时，您需要在实际语句运行之前对其进行微调。通过DatabaseClient注册语句过滤器（StatementFilterFunction），在语句执行过程中拦截和修改语句，如下例所示：

```java
client.sql("INSERT INTO table (name, state) VALUES(:name, :state)")
    .filter((s, next) -> next.execute(s.returnGeneratedValues("id")))
    .bind("name", …)
    .bind("state", …);
```

DatabaseClient还公开了简化的filter(…)重载接受Function<Statement, Statement>:

```java
client.sql("INSERT INTO table (name, state) VALUES(:name, :state)")
    .filter(statement -> s.returnGeneratedValues("id"));

client.sql("SELECT id, name, state FROM table")
    .filter(statement -> s.fetchSize(25));
```

StatementFilterFunction实现允许过滤语句和过滤结果对象。

##### `DatabaseClient` Best Practices 数据库客户端最佳实践

DatabaseClient类的实例在配置后是线程安全的。这一点很重要，因为这意味着您可以配置DatabaseClient的单个实例，然后将这个共享引用安全地注入到DAOs（或存储库）中。DatabaseClient是有状态的，因为它维护对ConnectionFactory的引用，但这种状态不是会话状态。

使用DatabaseClient类时的一个常见做法是在Spring配置文件中配置ConnectionFactory，然后依赖关系将共享的ConnectionFactory bean注入DAO类。DatabaseClient是在ConnectionFactory的setter中创建的。这将导致类似于以下内容的DAOs：

```java
public class R2dbcCorporateEventDao implements CorporateEventDao {

    private DatabaseClient databaseClient;

    public void setConnectionFactory(ConnectionFactory connectionFactory) {
        this.databaseClient = DatabaseClient.create(connectionFactory);
    }

    // R2DBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

显式配置的另一种选择是使用组件扫描和注释支持进行依赖注入。在这种情况下，可以用@Component注释类（这使得它成为组件扫描的候选对象），并用@Autowired为ConnectionFactory setter方法添加注释。下面的示例演示如何执行此操作：

```
@Component ①
public class R2dbcCorporateEventDao implements CorporateEventDao {

    private DatabaseClient databaseClient;

    @Autowired ②
    public void setConnectionFactory(ConnectionFactory connectionFactory) {
        this.databaseClient = DatabaseClient.create(connectionFactory);③ 
    }

    // R2DBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

①用@Component注释类。

②Annotate the `ConnectionFactory` setter method with `@Autowired`.

③Create a new `DatabaseClient` with the `ConnectionFactory`.

无论您选择使用（或不使用）上述哪种模板初始化样式，每次运行SQL时都很少需要创建DatabaseClient类的新实例。一旦配置好，DatabaseClient实例就是线程安全的。如果您的应用程序访问多个数据库，您可能需要多个DatabaseClient实例，这需要多个ConnectionFactory，然后需要多个不同配置的DatabaseClient实例。