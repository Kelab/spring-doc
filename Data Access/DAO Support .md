## 2. DAO Support DAO支持

Spring中的数据访问对象（DAO）支持旨在使数据访问技术（如JDBC、Hibernate或JPA）能够以一致的方式轻松地工作。这使您可以相当容易地在上述持久性技术之间切换，而且还可以让您在编写代码时不必担心捕获每种技术特有的异常。

### 2.1.Consistent Exception Hierarchy 一致的异常层次结构

Spring提供了从特定于技术的异常（例如SQLException）到它自己的异常类层次结构（以DataAccessException作为根异常）的方便转换。这些异常将原始异常包装起来，这样就不会有任何可能丢失有关可能出错的信息的风险。

除了JDBC异常之外，Spring还可以包装特定于JPA和Hibernate的异常，将它们转换为一组集中的运行时异常。这样，您就可以在适当的层中处理大多数不可恢复的持久性异常，而不必在dao中使用烦人的catch-and-throw块和异常声明。（您仍然可以在任何需要的地方捕获和处理异常。）如上所述，JDBC异常（包括特定于数据库的方言）也被转换为相同的层次结构，这意味着您可以在一致的编程模型中使用JDBC执行一些操作。

前面的讨论适用于Spring支持各种ORM框架的各种模板类。如果使用基于拦截器的类，则应用程序必须关心如何处理HibernateException和PersistenceException本身，最好分别委托给SessionFactoryUtils的convertHibernateAccessException（..）或convertJpaAccessException（）方法。这些方法将异常转换为与中的异常兼容的异常org.springframework.dao异常层次结构。当PersistenceExceptions未被检查时，它们也会被抛出（不过，在异常方面牺牲了泛型DAO抽象）。

下图显示了Spring提供的异常层次结构。（请注意，图中详述的类层次结构仅显示整个DataAccessException层次结构的一个子集。）

![DataAccessException](https://docs.spring.io/spring-framework/docs/current/reference/html/images/DataAccessException.png)

### 2.2.Annotations Used to Configure DAO or Repository Classes 用于配置DAO或存储库类的注释

确保数据访问对象（DAOs）或数据库库提供异常转换的最佳方法是使用@Repository注释。此注释还允许组件扫描支持查找和配置DAOs和数据库，而不必为它们提供XML配置条目。以下示例演示如何使用@Repository注释：

```java
@Repository ①
public class SomeMovieFinder implements MovieFinder {
    // ...
}
```

① @Repository注释。

任何DAO或数据库库实现都需要访问持久性资源，具体取决于所使用的持久性技术。例如，基于JDBC的数据库库需要访问JDBC数据源，基于JPA的数据库库需要访问EntityManager。实现这一点最简单的方法是使用@Autowired、@Inject、@resource或@PersistenceContext注释之一注入这个资源依赖项。以下示例适用于JPA数据库：

```java
@Repository
public class JpaMovieFinder implements MovieFinder {

    @PersistenceContext
    private EntityManager entityManager;

    // ...
}
```

如果使用经典的Hibernate API，则可以注入SessionFactory，如下例所示：

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

我们在这里展示的最后一个例子是典型的JDBC支持。您可以将数据源注入到初始化方法或构造函数中，在该方法或构造函数中，您将使用此数据源创建JdbcTemplate和其他数据访问支持类（如SimpleJdbcCall和其他类）。以下示例自动连接数据源：

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

```
有关如何配置应用程序上下文以利用这些注释的详细信息，请参阅每种持久性技术的具体介绍。
```

