# gitSpring事务管理

# Transaction Management

全面的事务支持是使用Spring Framework的最令人信服的原因

Spring框架为事务管理提供了一致的抽象，具有以下优点：

- 跨不同事务API（例如Java事务API（JTA），JDBC，Hibernate和Java Persistence API（JPA））的一致编程模型。

- 支持声明式事务管理。

- 与诸如JTA之类的复杂事务API相比，用于程序化事务管理的API更简单。

- 与Spring的数据访问抽象的出色集成。

以下各节描述了Spring Framework的事务特性和技术：

- [Advantages of the Spring Framework’s transaction support model](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-motivation) `Spring Framework事务支持模型的优点`描述了为什么您将使用Spring Framework的事务抽象而不是EJB容器管理的事务（CMT）或选择通过专有API驱动本地事务的原因，例如Hibernate。
- [Understanding the Spring Framework transaction abstraction](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-strategies) `了解Spring Framework事务抽象`概述了核心类，并描述了如何从各种来源配置和获取`DataSource`实例。
- [Synchronizing resources with transactions](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#tx-resource-synchronization) `将资源与事务同步`描述了应用程序代码如何确保正确创建，重用和清理资源。
- [Declarative transaction management](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative) `声明式事务管理`描述了对声明式事务管理的支持。
- [Programmatic transaction management](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-programmatic) `程序化事务管理`涵盖对程序化（即，显式编码）事务管理的支持。
- [Transaction bound event](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-event)`事务绑定事件`描述了如何在事务中使用应用程序事件。

本章还讨论了最佳实践，应用程序服务器集成 [application server integration](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-application-server-integration),以及常见问题的解决方案[solutions to common problems](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-solutions-to-common-problems)。

## 1.Spring框架的事务支持模型的优点

传统上，Java EE开发人员在事务管理中有两种选择：全局或本地事务，两者都有深刻的局限性。下两节将回顾全局和本地事务管理，然后讨论Spring框架的事务管理的支持如何解决全局和本地事务模型的局限性。

### 1.1. Global Transactions 全局事务

全局事务使您可以使用多个事务资源，通常是关系数据库和消息队列。 应用服务器通过JTA管理全局事务，该JTA是一个繁琐的API（部分是由于其异常模型）。 此外，JTA `UserTransaction`通常需要从JNDI派生，这意味着您还需要使用JNDI才能使用JTA。 全局事务的使用限制了应用程序代码的任何潜在重用，因为JTA通常仅在应用程序服务器环境中可用。

以前，使用全局事务的首选方法是通过EJB CMT（容器管理的事务）。 CMT是声明式事务管理的一种形式（与程序性事务管理不同）。 尽管使用EJB本身需要使用JNDI，但是EJB CMT消除了与事务相关的JNDI查找的需要。 它消除了编写Java代码来控制事务的大部分（但不是全部）需求。 重大缺点是CMT与JTA和应用程序服务器环境相关联。此外，它是唯一可用的，如果一个人选择使用EJB实现业务逻辑（或至少一个事务EJB门面后面）。 通常，EJB的负面影响是如此之大，以至于这不是一个有吸引力的主张，尤其是面对声明式事务管理的强制选择时。

### 1.2. Local Transactions 本地事务

本地事务是特定于资源的，例如与JDBC连接关联的事务。 本地事务可能更易于使用，但有一个明显的缺点：它们不能跨多个事务资源工作。 例如，使用JDBC连接管理事务的代码不能在全局JTA事务中运行。 由于应用程序服务器不参与事务管理，因此它无法帮助确保多个资源之间的正确性。 （值得注意的是，大多数应用程序使用单个事务资源。）另一个缺点是本地事务侵入了编程模型。

### 1.3. Spring Framework’s Consistent Programming Model

Spring框架的一致编程模型

Spring解决了全局和本地事务的弊端。 它使应用程序开发人员可以在任何环境中使用一致的编程模型。 您只需编写一次代码，它就可以从不同环境中的不同事务管理策略中受益。 Spring框架提供了声明式和程序化事务管理。 大多数用户喜欢声明式事务管理，在大多数情况下我们建议这样做。

通过程序化事务管理，开发人员可以使用Spring Framework事务抽象，该抽象可以在任何基础事务基础架构上运行。 使用首选的声明性模型，开发人员通常只编写很少或没有编写与事务管理相关的代码，因此，它们不依赖于Spring Framework事务API或任何其他事务API。

>您需要用于事务管理的应用程序服务器吗？ 
>
>Spring Framework的事务管理支持更改了有关企业Java应用程序何时需要应用程序服务器的传统规则。
>
>特别是，您不需要纯粹用于通过EJB进行声明式事务的应用程序服务器。实际上，即使您的应用程序服务器具有强大的JTA功能，您也可能会认为，与EJB CMT相比，Spring Framework的声明式事务提供更多的功能和更高效的编程模型。
>
>通常，仅当您的应用程序需要处理跨多个资源的事务时才需要应用程序服务器的JTA功能，而这并不是许多应用程序所必需的。许多高端应用程序使用单个高度可扩展的数据库（例如Oracle RAC）来代替。独立事务管理器（例如Atomikos Transactions和JOTM）是其他选择。当然，您可能需要其他应用程序服务器功能，例如Java消息服务（JMS）和Java EE连接器体系结构（JCA）。
>
>Spring Framework使您可以选择何时将应用程序扩展到完全加载的应用程序服务器。不再使用EJB CMT或JTA的唯一选择是使用本地事务（例如JDBC连接上的事务）编写代码，并且如果您需要使代码在全局的，容器管理的事务中运行，则面临大量的返工。使用Spring Framework，仅需要更改配置文件中的某些Bean定义（而不是代码）。

## 2. Understanding the Spring Framework Transaction Abstraction 了解Spring Framework事务抽象

Spring事务抽象的关键是事务策略的概念。交易策略由`TransactionManager`定义，特别是 `org.springframework.transaction.PlatformTransactionManager`接口用于命令式交易管理的以及`org.springframework.transaction.ReactiveTransactionManager`接口用于响应式交易管理。以下清单显示了`PlatformTransactionManager`API的定义 ：
Java

```java
public interface PlatformTransactionManager extends TransactionManager {

    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

尽管您可以从应用程序代码中以[编程](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-programmatic-ptm)方式使用它，但它主要是一个服务提供商接口（SPI） 。因为 `PlatformTransactionManager`是接口，所以可以根据需要轻松地对其进行mocked（模拟，类似代理）或stubbed（根除） 。它与诸如JNDI之类的查找策略无关。 `PlatformTransactionManager`实现的定义与Spring Framework IoC容器中的任何其他对象（或bean）一样。这一优点使得即使使用JTA，Spring框架事务也成为有价值的抽象。与直接使用JTA相比，您可以更轻松地测试事务代码。

同样，与Spring的哲学一致，任何可以被`PlatformTransactionManager`接口的方法抛出的`TransactionException`异常都是未检测的unchecked（即，它扩展了`java.lang.RuntimeException`该类）。事务基础架构故障几乎总是致命的。在极少数情况下，应用程序代码实际上可以从事务失败中恢复，应用程序开发人员仍然可以选择catch and handle `TransactionException`。突出的一点是，开发人员没有 ***被迫***这样做。

`getTransaction(..)`方法根据一个`TransactionDefinition`参数返回一个`TransactionStatus`对象 。如果当前调用堆栈中存在匹配的事务，则返回的`TransactionStatus`可能表示新事务，也可能表示现有事务。后一种情况的含义是，与Java EE事务上下文一样，`TransactionStatus`与执行线程相关联。

从Spring Framework 5.2开始，Spring还为使用响应式式类型或Kotlin协程（reactive types or Kotlin Coroutines）的响应式式应用程序(reactive applications)提供了事务管理抽象。以下清单显示了由定义的事务策略 `org.springframework.transaction.ReactiveTransactionManager`：

````java
public interface ReactiveTransactionManager extends TransactionManager {

    Mono<ReactiveTransaction> getReactiveTransaction(TransactionDefinition definition) throws TransactionException;

    Mono<Void> commit(ReactiveTransaction status) throws TransactionException;

    Mono<Void> rollback(ReactiveTransaction status) throws TransactionException;
}
````

响应式事务管理器主要是服务提供者接口（SPI），尽管您可以从应用程序代码中以[编程](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-programmatic-rtm)方式使用它。因为`ReactiveTransactionManager`是接口，所以可以根据需要轻松地对其进行mocked（模拟，类似代理）或stubbed（根除）

该`TransactionDefinition`接口指定：

- Propagation传播：通常，事务范围内的所有代码都在该事务中运行。但是，如果在已存在事务上下文的情况下运行事务方法，则可以指定行为。例如，代码可以在现有事务中继续运行（常见情况），或者可以暂停现有事务并创建新事务。Spring提供了EJB CMT熟悉的所有事务传播选项。要了解有关Spring中事务传播的语义的信息，请参阅[事务传播](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#tx-propagation)。
- Isolation隔离度：此事务与其他事务的工作隔离的程度。例如，该交易能否看到其他交易的未提交写入？
- Timeout超时：超时之前该事务运行了多长时间，并被基础事务基础结构自动回滚。
- Read-only status只读状态：当代码读取但不修改数据时，可以使用只读事务。在某些情况下，例如使用Hibernate时，只读事务可能是有用的优化。

这些设置反映了标准的事务概念。如有必要，请参考讨论事务隔离级别和其他核心事务概念的资源。了解这些概念对于使用Spring Framework或任何事务管理解决方案至关重要。

`TransactionStatus`接口为事务代码提供了一种控制事务执行和查询事务状态的简单方法。这些概念需要熟悉，因为它们对于所有事务API都是通用的。以下清单显示了该 `TransactionStatus`接口：

````java
public interface TransactionStatus extends TransactionExecution, SavepointManager, Flushable {

    @Override
    boolean isNewTransaction();

    boolean hasSavepoint();

    @Override
    void setRollbackOnly();

    @Override
    boolean isRollbackOnly();

    void flush();

    @Override
    boolean isCompleted();
}
````



无论您在Spring中选择声明式还是程序化事务管理，定义正确的`TransactionManager`实现都是绝对必要的。通常，您可以通过依赖注入来定义此实现。

`TransactionManager`实现通常需要了解其工作环境：JDBC，JTA，Hibernate等。以下示例显示了如何定义本地`PlatformTransactionManager`实现（在这种情况下，使用纯JDBC）。

您可以`DataSource`通过创建类似于以下内容的bean来定义JDBC ：

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
</bean>
```

然后，相关的`PlatformTransactionManager`bean定义将引用该 `DataSource`定义。它应类似于以下示例：

```xml
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

如果您在Java EE容器中使用JTA，那么您将使用`DataSource`通过JNDI获得的容器以及Spring的容器`JtaTransactionManager`。以下示例显示了JTA和JNDI查找版本的外观：

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

在`JtaTransactionManager`并不需要了解`DataSource`（或任何其他特定资源），因为它使用容器的全局事务管理。

|      | `dataSource`Bean 的先前定义使用名称空间中的`<jndi-lookup/>`标记`jee`。有关更多信息，请参见 [JEE Schema](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#xsd-schemas-jee)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果您使用JTA，则无论使用哪种数据访问技术（无论是JDBC，Hibernate JPA或任何其他受支持的技术），事务管理器定义都应该看起来相同。这是由于JTA事务是全局事务，它可以征用任何事务资源。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

在所有Spring事务设置中，无需更改应用程序代码。您可以仅通过更改配置来更改事务的管理方式，即使更改意味着从本地事务转移到全局事务，反之亦然。

