# Data Access

## 1. Transaction Management事务管理

全面的事务支持是使用**spring**框架的最令人信服的理由之一。Spring 框架为事务管理提供了一致的抽象，提供以下好处：

* 跨不同事务 API（如 Java 事务 API （JTA）、JDBC、Hibernate 和 Java 持久性 API （JPA））的一致编程模型。
* 支持[声明性事务管理](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative)。
* 与复杂的事务API一样，用于编程事务管理的 API 更简单。
* 与 Spring 的数据访问抽象的出色集成。

以下各节介绍 Spring 框架的事务特性和技术：

- [Spring Framework 的事务支持模型的优点](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-motivation)描述了为什么使用 Spring Framework 的事务抽象而不是 EJB 容器管理事务 （CMT） 或选择通过专有 API（如 Hibernate）驱动本地事务。
- [了解 Spring Framework 事务抽象](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-strategies)概述了核心类，并介绍了如何配置和从各种源获取实例。`DataSource`
- [将资源与事务](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#tx-resource-synchronization)同步描述应用程序代码如何确保资源被正确创建、重用和清理。
- [声明性事务管理](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative)描述对声明性事务管理的支持。
- [编程事务](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-programmatic)管理包括对编程（即显式编码）事务管理的支持。
- [事务绑定](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-event)事件描述如何在事务中使用应用程序事件。

本章还包括最佳实践、应用程序[服务器集成](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-application-server-integration)和[常见问题解决方案的讨论](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-solutions-to-common-problems)。

### 1.1. Advantages of the Spring Framework’s Transaction Support Modelspring框架事务支持模型的优势

传统上，Java EE 开发人员对事务管理有两种选择：全局事务或本地事务，这两种事务都有深刻的局限性。接下来的两节将审查全局和本地事务管理，然后讨论 Spring Framework 的事务管理支持如何处理全局和本地事务模型的局限性。

#### 1.1.1.Global Transactions全局事务

全局事务允许您使用多个事务资源，通常是关系数据库和消息队列。应用程序服务器通过 JTA 管理全局事务，这是一个繁琐的 API（部分原因是其异常模型）。此外，JTA 通常需要从 JNDI 获得，这意味着您还需要使用 JNDI 才能使用 JTA。使用全局事务会限制应用程序代码的任何潜在重用，因为 JTA 通常仅在应用程序服务器环境中可用。`UserTransaction`

以前，使用全局事务的首选方式是通过 EJB CMT（容器托管事务）。CMT 是一种声明性事务管理形式（与编程事务管理不同）。EJB CMT 消除了与事务相关的 JNDI 查找需求，尽管使用 EJB 本身需要使用 JNDI。它消除了编写 Java 代码来控制事务所需的大部分（但不是所有的）显著缺点是 CMT 与 JTA 和应用程序服务器环境绑定。此外，只有当选择在 EJB 中实现业务逻辑（或至少在事务性 EJB 外观后面）时，它才可用。EJB 的底片一般是很大的，这不是一个有吸引力的命题，特别是在面对令人信服的替代声明易管理。

#### 1.1.2. Local Transactions本地事务

本地事务特定于资源，例如与 JDBC 连接关联的事务。本地事务可能更易于使用，但有一个显著缺点：它们无法跨多个事务资源工作。例如，使用 JDBC 连接管理事务的代码不能在全局 JTA 事务中运行。由于应用程序服务器不参与事务管理，因此无法确保跨多个资源的正确性。（值得注意的是，大多数应用程序使用单个事务资源。另一个缺点是本地事务侵入了编程模型。

#### 1.1.3.  Spring Framework’s Consistent Programming Model Spring框架的一致编程模型

Spring 解决了全局和本地事务的缺点。它允许应用程序开发人员在任何环境中使用一致的编程模型。编写代码一次，它可以从不同环境中的不同事务管理策略中受益。Spring 框架提供声明性事务和编程事务管理。大多数用户更喜欢声明性事务管理，在大多数情况下，我们建议进行声明性事务管理。

使用编程事务管理，开发人员使用 Spring Framework 事务抽象，它可以运行任何基础事务基础结构。使用首选声明性模型时，开发人员通常编写很少或没有与事务管理相关的代码，因此不依赖于 Spring Framework 事务 API 或任何其他事务 API。

<h4 align = "center">是否需要应用程序服务器进行事务管理？</h4>

> Spring Framework 的事务管理支持改变了企业 Java 应用程序何时需要应用程序服务器的传统规则。
>
> 特别是，您不需要应用程序服务器，纯粹通过 EJB 进行声明性事务。事实上，即使您的应用程序服务器具有强大的 JTA 功能，您也可能认为 Spring Framework 的申报事务比 EJB CMT 提供更大的功能和高效的编程模型。
>
> 通常，只有当应用程序需要处理跨多个资源的事务时，才需要应用程序服务器的 JTA 功能，这对于许多应用程序来说不是要求。许多高端应用程序使用单个高度可扩展的数据库（如 Oracle RAC）。独立事务管理器（如[原子事务和](https://www.atomikos.com/) [JOTM）](http://jotm.objectweb.org/)是其他选项。当然，您可能需要其他应用程序服务器功能，如 Java 消息服务 （JMS） 和 Java EE 连接器体系结构 （JCA）。
>
> Spring 框架允许您选择何时将应用程序扩展到满载的应用程序服务器。使用 EJB CMT 或 JTA 的唯一替代方案就是使用本地事务（如 JDBC 连接上的代码）编写代码，如果需要该代码在全球容器托管事务中运行，则面临大量返工。使用 Spring 框架时，只需更改配置文件中的部分 bean 定义（而不是代码）。

### 1.2.  Understanding the Spring Framework Transaction Abstraction了解spring框架事务抽象

Spring事务抽象的关键是事务策略的概念。 事务策略由`TransactionManager`定义，特别是用于命令式事务管理的`org.springframework.transaction.PlatformTransactionManager`接口和用于反应式事务管理的`org.springframework.transaction.ReactiveTransactionManager`接口。 以下清单显示了`PlatformTransactionManager` API的定义：

```java
public interface PlatformTransactionManager extends TransactionManager {

    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```



这主要是服务提供商接口 （SPI），尽管可以从应用程序[代码以编程](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-programmatic-ptm)方式使用它。因为`PlatformTransactionManager`是一个接口，它可以很容易地模拟或存根作为必要的。它与查找策略（如 JNDI） `PlatformTransactionManager`实现的定义与 Spring Framework IoC 容器中任何其他对象（或 bean）一样。即使使用 JTA，仅此一项优势也使 Spring Framework 事务成为有价值的抽象。与直接使用 JTA 时，您可以更轻松地测试事务代码。



同样，根据 Spring 的理念，`TransactionException`接口的任何方法都可以抛出的 不选中（也就是说，它扩展了`java.lang.RuntimeException`类）。事务基础结构故障几乎总是致命的。在极少数情况下，应用程序代码实际上可以从事务故障中恢复，应用程序开发人员仍然可以选择捕获和处理`TransactionException` 。突出的一点是，开发人员*不会被迫*这样做。



该`getTransaction(..)`方法`TransactionStatus`根据`TransactionDefinition`参数返回一个对象 。`TransactionStatus`如果当前调用堆栈中存在匹配的事务，则返回的结果可能表示新事务，也可能表示现有事务。后一种情况的含义是，与Java EE事务上下文一样，a`TransactionStatus`与执行线程相关联。



从Spring Framework 5.2开始，Spring还为使用反应式类型或Kotlin协程的反应式应用程序提供了事务管理抽象。以下清单显示了由定义的事务策略 `org.springframework.transaction.ReactiveTransactionManager`：

```
public interface ReactiveTransactionManager extends TransactionManager {

    Mono<ReactiveTransaction> getReactiveTransaction(TransactionDefinition definition) throws TransactionException;

    Mono<Void> commit(ReactiveTransaction status) throws TransactionException;

    Mono<Void> rollback(ReactiveTransaction status) throws TransactionException;
}
```



响应式事务管理器主要是服务提供商接口（SPI），尽管您可以从应用程序代码中以[编程](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-programmatic-rtm)方式使用它。因为`ReactiveTransactionManager`是接口，所以可以根据需要轻松地对其进行模拟或存根。

该`TransactionDefinition`接口指定：

- 传播：通常，事务范围内的所有代码都在该事务中运行。但是，如果在已存在事务上下文的情况下运行事务方法，则可以指定行为。例如，代码可以在现有事务中继续运行（常见情况），或者可以暂停现有事务并创建新事务。Spring提供了EJB CMT熟悉的所有事务传播选项。要了解有关Spring中事务传播的语义的信息，请参阅[事务传播](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#tx-propagation)。
- 隔离度：此事务与其他事务的工作隔离的程度。例如，此事务能否看到其他事务未提交的写入？
- 超时：超时之前该事务运行了多长时间，并被基础事务基础结构自动回滚。
- 只读状态：当代码读取但不修改数据时，可以使用只读事务。在某些情况下，例如使用Hibernate时，只读事务可能是有用的优化。

这些设置反映了标准的事务概念。如有必要，请参考讨论事务隔离级别和其他核心事务概念的资源。了解这些概念对于使用Spring Framework或任何事务管理解决方案至关重要。



该`TransactionStatus`接口为事务代码提供了一种控制事务执行和查询事务状态的简单方法。这些概念应该很熟悉，因为它们对于所有事务API都是通用的。以下清单显示了该 `TransactionStatus`接口：

```
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
```

无论您在Spring中选择声明式还是程序化事务管理，定义正确的`TransactionManager`实现都是绝对必要的。通常，您可以通过依赖注入来定义此实现。

`TransactionManager`实现通常需要了解其工作环境：JDBC，JTA，Hibernate等。以下示例显示了如何定义本地`PlatformTransactionManager`实现（在这种情况下，使用纯JDBC）。

您可以`DataSource`通过创建类似于以下内容的bean来定义JDBC ：

```
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
</bean>
```

然后，相关的`PlatformTransactionManager`bean定义将引用该 `DataSource`定义。它应类似于以下示例：

```
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

如果您在Java EE容器中使用JTA，则可以使用`DataSource`通过JNDI与Spring的容器一起获得的容器`JtaTransactionManager`。以下示例显示了JTA和JNDI查找版本的外观：

```
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

> `dataSource`Bean 的先前定义使用名称空间中的`<jndi-lookup/>`标记`jee`。有关更多信息，请参见 [JEE Schema](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#xsd-schemas-jee)。
>
>
> 如果使用JTA，则无论使用哪种数据访问技术（无论是JDBC，Hibernate JPA或任何其他受支持的技术），事务管理器定义都应该看起来相同。这是由于JTA事务是全局事务，它可以征用任何事务资源。

在所有Spring事务设置中，无需更改应用程序代码。您可以仅通过更改配置来更改事务的管理方式，即使更改意味着从本地事务转移到全局事务，反之亦然。

参考文档的这一部分涉及数据访问以及数据访问层与业务或服务层之间的交互。

详细介绍了Spring全面的事务管理支持，然后全面介绍了Spring框架所集成的各种数据访问框架和技术。



#### 1.2.1. Hibernat事务设置

您还可以轻松使用Hibernate本地事务，如以下示例所示。在这种情况下，您需要定义一个Hibernate `LocalSessionFactoryBean`，您的应用程序代码可使用该Hibernate获取Hibernate`Session`实例。



`DataSource`bean定义类似于前面所示的本地JDBC示例，并且因此，在下面的示例中未示出。

> 如果`DataSource`（由任何非JTA事务管理器使用）通过JNDI查找并由Java EE容器管理，则它应该是非事务性的，因为Spring框架（而不是Java EE容器）管理事务。

`txManager`在这种情况下，bean是`HibernateTransactionManager`类型。在作为相同的方式`DataSourceTransactionManager`需要在一个参考`DataSource`时， `HibernateTransactionManager`需要将一个参考`SessionFactory`。以下示例声明`sessionFactory`和`txManager`bean：

```
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

如果使用Hibernate和Java EE容器管理的JTA事务，则应使用与`JtaTransactionManager`前面的JDBC的JTA示例相同的方法，如以下示例所示。另外，建议使Hibernate通过其事务协调器以及可能的连接释放模式配置来了解JTA：

```
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
            hibernate.transaction.coordinator_class=jta
            hibernate.connection.handling_mode=DELAYED_ACQUISITION_AND_RELEASE_AFTER_STATEMENT
        </value>
    </property>
</bean>

<bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
```

或者，您也可以将传递给`JtaTransactionManager`您`LocalSessionFactoryBean` 以执行相同的默认设置：

```
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
    <property name="jtaTransactionManager" ref="txManager"/>
</bean>

<bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
```

### 1.3.Synchronizing Resources with Transactions 将资源与事务同步

现在应该清楚如何创建不同的事务管理器，以及如何将它们链接到需要与事务同步的相关资源（例如，`DataSourceTransactionManager` 到JDBC `DataSource`，`HibernateTransactionManager`Hibernate`SessionFactory`等）。本节描述应用程序代码如何（通过使用诸如JDBC，Hibernate或JPA之类的持久性API直接或间接）确保正确创建，重用和清理这些资源。本节还讨论了如何通过相关的触发（可选）事务同步`TransactionManager`。

#### 1.3.1. High-level Synchronization Approach 高级同步方法

首选方法是使用Spring基于最高级别模板的持久性集成API或将本机ORM API与具有事务感知功能的工厂bean或代理一起使用，以管理本机资源工厂。这些支持事务的解决方案在内部处理资源的创建和重用，清理，资源的可选事务同步以及异常映射。因此，用户数据访问代码不必解决这些任务，而可以完全专注于非样板持久性逻辑。通常，您使用本机ORM API或通过使用模板方法来进行JDBC访问`JdbcTemplate`。这些解决方案将在本参考文档的后续部分中详细介绍。

#### 1.3.2. Low-level Synchronization Approach低级同步方法

诸如`DataSourceUtils`（对于JDBC），`EntityManagerFactoryUtils`（对于JPA）， `SessionFactoryUtils`（对于Hibernate）之类的类处于较低级别。当您希望应用程序代码直接处理本机持久性API的资源类型时，可以使用这些类来确保获得正确的Spring Framework管理的实例，（可选）同步事务以及处理过程中发生的异常。正确映射到一致的API。

例如，在JDBC的情况下，可以使用Spring的类，而不是使用 传统的JDBC方法在上调用`getConnection()`方法，如下所示：`DataSource``org.springframework.jdbc.datasource.DataSourceUtils`

```
Connection conn = DataSourceUtils.getConnection(dataSource);
```

如果现有事务已具有与其同步（链接）的连接，则返回该实例。否则，方法调用将触发创建新连接，该连接（可选）同步到任何现有事务，并可供该同一事务中的后续重用使用。如前所述，任何内容 `SQLException`都包装在Spring Framework中`CannotGetJdbcConnectionException`，Spring Framework是未经检查的`DataAccessException`类型的层次结构之一。这种方法为您提供的信息多于从轻松获得的信息，`SQLException`并确保了跨数据库甚至跨不同持久性技术的可移植性。

这种方法在没有Spring事务管理的情况下也可以使用（事务同步是可选的），因此无论是否使用Spring进行事务管理，都可以使用它。

当然，一旦使用了Spring的JDBC支持，JPA支持或Hibernate支持，您通常就不愿使用`DataSourceUtils`或其他帮助程序类，因为与直接使用相关的API相比，通过Spring抽象进行工作要快乐得多。例如，如果您使用Spring`JdbcTemplate`或 `jdbc.object`程序包来简化JDBC的使用，则正确的连接检索将在后台进行，并且您无需编写任何特殊代码。



#### 1.3.3.`TransactionAwareDataSourceProxy`

`TransactionAwareDataSourceProxy`该类的最低级别。这是target的代理`DataSource`，它包装了目标`DataSource`以增加对Spring管理的事务的了解。在这方面，它类似于`DataSource`Java EE服务器提供的事务性JNDI 。

您几乎永远不需要或不想使用此类，除非必须调用现有代码并传递标准的JDBC`DataSource`接口实现。在这种情况下，该代码可能可用，但参与了Spring管理的事务。您可以使用前面提到的高级抽象来编写新代码。

### 1.4. Declarative transaction management声明式事务管理

**大多数Spring Framework用户选择声明式事务管理。此选项对应用程序代码的影响最小，因此与无创轻量级容器的理念最一致。**



Spring面向切面的编程（AOP）使Spring框架的声明式事务管理成为可能。但是，由于事务方面的代码随Spring Framework发行版一起提供并且可以以样板方式使用，因此通常不必理解AOP概念即可有效地使用此代码。

Spring Framework的声明式事务管理与EJB CMT相似，因为您可以指定事务行为（或缺少事务行为），直至单个方法级别。`setRollbackOnly()`如有必要，您可以在事务上下文中进行呼叫。两种类型的事务管理之间的区别是：

- 与绑定到JTA的EJB CMT不同，Spring框架的声明式事务管理可在任何环境中工作。它可以通过使用JDBC，JPA或Hibernate通过调整配置文件来处理JTA事务或本地事务。
- 您可以将Spring Framework声明式事务管理应用于任何类，而不仅限于诸如EJB之类的特殊类。
- Spring框架提供了声明性 [回滚规则](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative-rolling-back)，这是没有EJB等效项的功能。提供了对回滚规则的编程和声明性支持。
- Spring Framework允许您使用AOP自定义事务行为。例如，在事务回滚的情况下，您可以插入自定义行为。您还可以添加任意建议以及事务建议。使用EJB CMT，您不能影响容器的事务管理，除非使用 `setRollbackOnly()`。
- Spring框架不像高端应用程序服务器那样支持跨远程调用传播事务上下文。如果需要此功能，建议您使用EJB。但是，在使用这种功能之前，请仔细考虑，因为通常情况下，您不希望事务跨越远程调用。

回滚规则的概念很重要。他们让您指定哪些异常（和可抛出对象）应引起自动回滚。您可以在配置中而不是在Java代码中以声明方式指定。所以，尽管你仍然可以调用`setRollbackOnly()`上的`TransactionStatus`对象回滚当前事务回，经常可以指定一个规则，`MyApplicationException`必须始终导致回滚。此选项的主要优点是业务对象不依赖于事务基础结构。例如，他们通常不需要导入Spring事务API或其他Spring API。

尽管EJB容器的默认行为会在系统异常（通常是运行时异常）时自动回滚事务，但是EJB CMT不会在应用程序异常（即以外的已检查异常`java.rmi.RemoteException`）下自动回滚事务。尽管Spring声明式事务管理的默认行为遵循EJB约定（仅针对未检查的异常会自动回滚），但自定义此行为通常很有用。

#### 1.4.1. Understanding the Spring Framework’s Declarative Transaction Implementation 了解Spring框架的声明式事务实现

仅告诉您使用注释对类进行`@Transactional`注释，添加`@EnableTransactionManagement`到配置中并希望您了解其全部工作原理是不够的 。为了提供更深入的理解，本节介绍了在与事务相关的问题的上下文中，Spring框架的声明式事务基础结构的内部工作方式。

关于Spring框架的声明式事务支持，要把握的最重要的概念是，该支持是[通过AOP代理](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-understanding-aop-proxies)启用的 ，并且事务建议由元数据（当前基于XML或基于注释）驱动。AOP与事务元数据的组合产生了一个AOP代理，该代理`TransactionInterceptor`结合使用a和适当的`TransactionManager`实现来驱动方法调用周围的事务。



**Spring AOP在[AOP部分中介绍](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)。**



Spring Frameworks`TransactionInterceptor`为命令式和反应式编程模型提供事务管理。拦截器通过检查方法返回类型来检测所需的事务管理风格。返回反应式类型（例如`Publisher`或Kotlin `Flow`（或其子类型））的方法符合反应式事务管理的条件。所有其他返回类型包括`void`使用代码路径进行命令式事务管理。

事务管理风格影响需要哪个事务管理器。命令式事务需要使用`PlatformTransactionManager`，而反应式事务则使用 `ReactiveTransactionManager`实现。

#### 1.4.2. Example of Declarative Transaction Implementation 声明式事务实现示例

考虑以下接口及其附带的实现。本示例使用 `Foo`和`Bar`类作为占位符，以便您可以专注于事务使用而不必关注特定的域模型。出于本示例的目的，`DefaultFooService`类`UnsupportedOperationException` 在每个已实现方法的主体中引发实例的事实是很好的。该行为使您可以查看正在创建的事务，然后回滚以响应 `UnsupportedOperationException`实例。以下清单显示了该`FooService` 接口：

```
// the service interface that we want to make transactional

package x.y.service;

public interface FooService {

    Foo getFoo(String fooName);

    Foo getFoo(String fooName, String barName);

    void insertFoo(Foo foo);

    void updateFoo(Foo foo);

}
```

以下示例显示了上述接口的实现：

```
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

假设`FooService`接口的前两个方法`getFoo(String)`和 `getFoo(String, String)`必须在具有只读语义的事务的上下文中运行，而其他方法`insertFoo(Foo)`和`updateFoo(Foo)`必须在具有读写语义的事务的上下文中运行。以下几节将详细说明以下配置：

```
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

    <!-- similarly, don't forget the TransactionManager -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- other <bean/> definitions here -->

</beans>
```

检查前面的配置。它假定您要使服务对象（`fooService`bean）具有事务性。要应用的事务语义封装在`<tx:advice/>`定义中。该`<tx:advice/>`定义为“所有`get`以只读开头的方法都将在只读事务的上下文中运行，而所有其他方法都将以默认的事务语义运行”。标签的 `transaction-manager`属性`<tx:advice/>`设置`TransactionManager`为将要驱动事务的bean的名称 （在本例中为 `txManager`bean）。

==可以省略`transaction-manager`的事务通知（属性`<tx:advice/>`如果的bean的名字）`TransactionManager`，你想丝具有名称`transactionManager`。如果`TransactionManager`要连接的bean有其他名称，则必须`transaction-manager` 显式使用该属性，如上例所示。==

该`<aop:config/>`定义可确保`txAdvice`Bean定义的事务建议 在程序中的适当位置运行。首先，您定义一个切入点，该切入点与`FooService`接口（`fooServiceOperation`）中定义的任何操作的执行相匹配。然后`txAdvice`，使用顾问将切入点与关联。结果表明，在执行时`fooServiceOperation`，`txAdvice`运行定义的建议。

`<aop:pointcut/>`元素内定义的表达式是AspectJ切入点表达式。有关Spring中切入点表达式的更多详细信息，请参见[AOP部分](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)。

一个普遍的要求是使整个服务层具有事务性。最好的方法是更改切入点表达式以匹配服务层中的任何操作。以下示例显示了如何执行此操作：

```
<aop:config>
    <aop:pointcut id="fooServiceMethods" expression="execution(* x.y.service.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceMethods"/>
</aop:config>
```

现在，我们已经分析了配置，您可能会问自己：“所有这些配置实际上是做什么的？”

前面显示的配置用于围绕从`fooService`Bean定义创建的对象创建事务代理。代理配置有事务建议，以便在代理上调用适当的方法时，根据与该方法相关联的事务配置，事务将被启动，挂起，标记为只读等。考虑下面的程序，该程序测试驱动前面显示的配置：

```
public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("context.xml", Boot.class);
        FooService fooService = (FooService) ctx.getBean("fooService");
        fooService.insertFoo (new Foo());
    }
}
```

运行前面程序的输出应类似于以下内容（为清楚起见，Log4J输出和该类`insertFoo(..)`方法`UnsupportedOperationException`抛出 `DefaultFooService`的堆栈跟踪已被截断）：

```
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

要使用反应式事务管理，代码必须使用反应式类型。

==Spring Framework使用`ReactiveAdapterRegistry`来确定方法返回类型是否为反应型。==

以下代码显示了先前使用的的修改版本`FooService`，但是这次代码使用了反应类型：

```
// the reactive service interface that we want to make transactional

package x.y.service;

public interface FooService {

    Flux<Foo> getFoo(String fooName);

    Publisher<Foo> getFoo(String fooName, String barName);

    Mono<Void> insertFoo(Foo foo);

    Mono<Void> updateFoo(Foo foo);

}
```

以下示例显示了上述接口的实现：

```
package x.y.service;

public class DefaultFooService implements FooService {

    @Override
    public Flux<Foo> getFoo(String fooName) {
        // ...
    }

    @Override
    public Publisher<Foo> getFoo(String fooName, String barName) {
        // ...
    }

    @Override
    public Mono<Void> insertFoo(Foo foo) {
        // ...
    }

    @Override
    public Mono<Void> updateFoo(Foo foo) {
        // ...
    }
}
```



强制性和响应性事务管理对事务边界和事务属性定义共享相同的语义。强制事务和被动事务之间的主要区别在于后者的延期性质。`TransactionInterceptor` 用事务运算符修饰返回的反应式类型，以开始和清理事务。因此，调用事务响应式方法将实际的事务管理推迟到激活响应式类型的处理的预订类型。

响应式事务管理的另一方面涉及数据转义，这是编程模型的自然结果。

成功终止方法后，从事务方法返回命令性事务的方法返回值，以使部分计算的结果不会逃脱方法的关闭。

响应式事务方法返回一个反应式包装类型，该包装类型表示一个计算序列以及一个开始和完成计算的承诺。

`Publisher`当事务正在进行但不一定完成时，A可以发出数据。因此，依赖于成功完成整个事务的方法需要确保完成并缓冲调用代码中的结果。

#### 1.4.3. Rolling Back a Declarative Transaction 回滚声明式事务

上一节概述了如何在应用程序中声明性地指定类（通常是服务层类）的事务性设置的基础。本节介绍如何以简单的声明方式控制事务的回滚。

向Spring框架的事务基础结构指示要回滚事务的推荐方法是抛出一个`Exception`在事务上下文中当前正在执行的from代码。Spring框架的事务基础结构代码`Exception`会在气泡堆积到调用堆栈时捕获任何未处理的内容，并确定是否将事务标记为回滚。

在其默认配置中，Spring Framework的事务基础结构代码仅在运行时未经检查的异常情况下将事务标记为回滚。也就是说，当抛出的异常是的实例或子类时`RuntimeException`。（ `Error`默认情况下，实例也会导致回滚）。从事务方法引发的检查异常不会导致默认配置中的回滚。

您可以准确配置哪些`Exception`类型将事务标记为回滚，包括已检查的异常。以下XML代码段演示了如何为选中的，特定于应用程序的`Exception`类型配置回滚：

```
<tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
    <tx:method name="get*" read-only="true" rollback-for="NoProductInStockException"/>
    <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
```

如果您不希望在引发异常时回滚事务，则还可以指定“无回滚规则”。下面的示例告诉Spring框架的事务基础结构即使在未处理的情况下也要提交伴随的事务`InstrumentNotFoundException`：

```
<tx:advice id="txAdvice">
    <tx:attributes>
    <tx:method name="updateStock" no-rollback-for="InstrumentNotFoundException"/>
    <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
```

当Spring Framework的事务基础结构捕获到异常并查询配置的回滚规则以确定是否将事务标记为回滚时，最强的匹配规则获胜。因此，在以下配置的情况下，除`InstrumentNotFoundException`结果以外的任何异常都会导致附带事务的回滚：

```
<tx:advice id="txAdvice">
    <tx:attributes>
    <tx:method name="*" rollback-for="Throwable" no-rollback-for="InstrumentNotFoundException"/>
    </tx:attributes>
</tx:advice>
```

您还可以通过编程方式指示所需的回滚。尽管很简单，但是此过程具有很大的侵入性，并将您的代码紧密耦合到Spring Framework的事务基础结构。以下示例显示如何以编程方式指示所需的回滚：

```
public void resolvePosition() {
    try {
        // some business logic...
    } catch (NoProductInStockException ex) {
        // trigger rollback programmatically
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}
```

强烈建议您尽可能使用声明性方法进行回滚。如果您绝对需要它，则可以使用程序化回滚，但是面对一个干净的基于POJO的体系结构，它的用法就不那么理想了。

#### 1.4.4.  Configuring Different Transactional Semantics for Different Beans 为不同的Bean配置不同的事务语义

考虑以下场景：您有许多服务层对象，并且您希望对每个对象应用完全不同的事务配置。您可以通过定义不同的做`<aop:advisor/>`用不同的元素`pointcut`和 `advice-ref`属性值。

作为比较点，首先假定所有服务层类都在根`x.y.service`包中定义。要使所有在该包（或子包）中定义的类实例的bean的名称以`Service`默认事务配置为结尾，可以编写以下代码：

```
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

    <!-- other transaction infrastructure beans such as a TransactionManager omitted... -->

</beans>
```

以下示例显示如何使用完全不同的事务设置配置两个不同的Bean：

```
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

    <!-- other transaction infrastructure beans such as a TransactionManager omitted... -->

</beans>
```

#### 1.4.5.<tx:advice/> Settings <tx：advice />设置

本节总结了可以使用`<tx:advice/>`标记指定的各种事务设置。默认`<tx:advice/>`设置为：

- [传播环境](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#tx-propagation)是`REQUIRED.`
- 隔离级别为 `DEFAULT.`
- 事务是读写的。
- 事务超时默认为基础事务系统的默认超时，如果不支持超时，则默认为无。
- 任何`RuntimeException`触发器都会回滚，而任何检查`Exception`都不会。

您可以更改这些默认设置。下表总结了`<tx:method/>`嵌套在`<tx:advice/>`和`<tx:attributes/>`标签中的标签的各种属性：

| 属性              | 需要？ | 默认       | 描述                                                         |
| :---------------- | :----- | :--------- | :----------------------------------------------------------- |
| `name`            | 是     |            | 与事务属性关联的方法名称。通配符（*）字符可以被用于相同的事务属性的设置有许多方法相关联（例如，`get*`，`handle*`，`on*Event`，等等）。 |
| `propagation`     | 没有   | `REQUIRED` | 事务传播行为。                                               |
| `isolation`       | 没有   | `DEFAULT`  | 事务隔离级别。仅适用于`REQUIRED`或的传播设置`REQUIRES_NEW`。 |
| `timeout`         | 没有   | -1         | 事务超时（秒）。仅适用于传播`REQUIRED`或传播`REQUIRES_NEW`。 |
| `read-only`       | 没有   | 假         | 读写与只读事务。仅适用于`REQUIRED`或`REQUIRES_NEW`。         |
| `rollback-for`    | 没有   |            | 逗号分隔的`Exception`触发回滚的实例列表。例如， `com.foo.MyBusinessException,ServletException`。 |
| `no-rollback-for` | 没有   |            | `Exception`不触发回滚的实例的逗号分隔列表。例如， `com.foo.MyBusinessException,ServletException`。 |

#### 1.4.6. Using `@Transactional` 使用`@Transactional`

除了基于XML的声明式方法进行事务配置外，还可以使用基于注释的方法。直接在Java源代码中声明事务语义会使声明更加接近受影响的代码。不存在过度耦合的危险，因为原本打算以事务方式使用的代码几乎总是以这种方式部署。

==`javax.transaction.Transactional`还支持 使用标准注释来替代Spring自己的注释。请参阅JTA 1.2文档以获取更多详细信息。==

通过`@Transactional`示例可以最好地说明使用批注提供的易用性，下面将对此进行说明。考虑以下类定义：

```
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

注释在上面的类级别使用，注释指示声明类（及其子类）的所有方法的默认值。另外，每种方法都可以单独注释。请注意，类级别的注释不适用于类层次结构中的祖先类。在这种情况下，需要在本地重新声明方法，以参与子类级别的注释。

当一个POJO类（例如上面的一个）在Spring上下文中定义为bean时，您可以通过类中的`@EnableTransactionManagement` 注释使bean实例具有事务性`@Configuration`。有关 完整详细信息，请参见 [javadoc](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/transaction/annotation/EnableTransactionManagement.html)。

在XML配置中，`<tx:annotation-driven/>`标记提供了类似的便利：

```
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
    <tx:annotation-driven transaction-manager="txManager"/><!-- a TransactionManager is still required --> 

    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- (this dependency is defined somewhere else) -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- other <bean/> definitions here -->

</beans>
```

**使bean实例具有事务性的行。**

> 可以省略`transaction-manager`的属性`<tx:annotation-driven/>` 标签，如果的bean的名字`TransactionManager`要在具有名称线， `transactionManager`。如果`TransactionManager`要依赖注入的Bean有其他名称，则必须使用该`transaction-manager`属性，如上例所示。

相对于命令式编程安排，响应式事务方法使用响应式返回类型，如下清单所示：

```
// the reactive service class that we want to make transactional
@Transactional
public class DefaultFooService implements FooService {

    Publisher<Foo> getFoo(String fooName) {
        // ...
    }

    Mono<Foo> getFoo(String fooName, String barName) {
        // ...
    }

    Mono<Void> insertFoo(Foo foo) {
        // ...
    }

    Mono<Void> updateFoo(Foo foo) {
        // ...
    }
}
```

请注意，对于返回的`Publisher`响应流取消信号有一些特殊考虑。有关更多详细信息，请参见“使用TransactionOperator”下的“[取消信号”](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#tx-prog-operator-cancel)部分。

**方法可见性和 `@Transactional`**

使用代理时，应仅将`@Transactional`注释应用于具有公共可见性的方法。如果使用注释对受保护的，私有的或程序包可见的方法进行`@Transactional`注释，则不会引发任何错误，但是带注释的方法不会显示已配置的事务设置。如果需要注释非公共方法，请考虑使用AspectJ（稍后描述）。



您可以将`@Transactional`注释应用于接口定义，接口上的方法，类定义或类上的公共方法。但是，仅存在`@Transactional`注释不足以激活事务行为。该`@Transactional`注解仅仅是元数据可以通过一些运行时基础架构即消耗`@Transactional`知晓的，并且可以使用元数据与事务行为的配置适当的豆。在前面的示例中，`<tx:annotation-driven/>`元素打开事务行为。

> Spring团队建议您仅使用注释对具体类（以及具体类的方法）进行`@Transactional`注释，而不是对接口进行注释。您当然可以将`@Transactional`注释放置在接口（或接口方法）上，但这仅在您使用基于接口的代理时才可以预期地起作用。Java批注不是从接口继承的事实意味着，如果您使用基于类的代理（`proxy-target-class="true"`）或基于织入的方面（`mode="aspectj"`），则代理和织入基础结构无法识别事务设置，并且不会包装对象在事务代理中。

> 在代理模式（默认设置）下，仅拦截通过代理传入的外部方法调用。这意味着自调用（实际上是目标对象中的方法调用目标对象的另一种方法）即使在调用的方法上标有，也不会在运行时导致实际事务`@Transactional`。另外，必须完全初始化代理以提供预期的行为，因此您不应在初始化代码（即`@PostConstruct`）中依赖此功能。

`mode`如果希望自调用也与事务一起包装，请考虑使用AspectJ模式（请参见下表中的属性）。在这种情况下，首先没有代理。而是织入目标类（即，修改其字节码）以将其`@Transactional`转换为任何方法的运行时行为。

| XML属性               | 注释属性                                                     | 默认                        | 描述                                                         |
| :-------------------- | :----------------------------------------------------------- | :-------------------------- | :----------------------------------------------------------- |
| `transaction-manager` | 不适用（请参见[`TransactionManagementConfigurer`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/transaction/annotation/TransactionManagementConfigurer.html)javadoc） | `transactionManager`        | 要使用的事务管理器的名称。`transactionManager`如上例所示，仅当事务管理器的名称不是时才需要。 |
| `mode`                | `mode`                                                       | `proxy`                     | 默认模式（`proxy`）使用Spring的AOP框架处理要注释的bean（遵循代理语义，如前所述，仅适用于通过代理传入的方法调用）。`aspectj`相反，替代模式（）使用Spring的AspectJ事务方面织入受影响的类，修改目标类字节代码以应用于任何类型的方法调用。AspectJ织入需要`spring-aspects.jar`在类路径中进行，并且需要 启用加载时织入（或编译时织入）。（有关 如何设置加载时织入的详细信息，请参见[Spring配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-aj-ltw-spring)。） |
| `proxy-target-class`  | `proxyTargetClass`                                           | `false`                     | `proxy`仅适用于模式。控制为带有`@Transactional`注释的类创建哪种类型的事务代理。如果该 `proxy-target-class`属性设置为`true`，则会创建基于类的代理。如果`proxy-target-class`为is`false`或如果省略该属性，那么将创建基于标准JDK接口的代理。（有关 不同代理类型的详细检查，请参见[代理机制](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-proxying)。） |
| `order`               | `order`                                                      | `Ordered.LOWEST_PRECEDENCE` | 定义应用于带注释的Bean的事务通知的顺序 `@Transactional`。（有关与AOP通知的排序有关的规则的更多信息，请参阅“[建议排序”](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-ataspectj-advice-ordering)。）没有指定的排序意味着AOP子系统确定建议的顺序。 |

==处理`@Transactional`注释的默认建议模式是`proxy`，它仅允许通过代理拦截呼叫。同一类内的本地调用无法以这种方式被拦截。对于更高级的侦听模式，请考虑`aspectj`结合编译时或加载时织入切换到模式。==

==该`proxy-target-class`属性控制为带有`@Transactional`注释的类创建哪种类型的事务代理。如果 `proxy-target-class`设置为`true`，则会创建基于类的代理。如果 `proxy-target-class`为is`false`或如果省略该属性，则创建基于标准JDK接口的代理。（有关不同代理类型的讨论，请参见[core.html](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-proxying)。）==

==`@EnableTransactionManagement`并且仅在定义它们的相同应用程序上下文中`<tx:annotation-driven/>`查找 `@Transactional`Bean。这意味着，如果将注释驱动的配置放在的`WebApplicationContext` 中`DispatcherServlet`，则`@Transactional`仅在控制器中检查bean，而不在服务中检查bean。有关更多信息，请参见[MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet)。==

在评估方法的事务设置时，最派生的位置优先。在以下示例的情况下，`DefaultFooService`使用只读事务的设置在类级别对类进行注释，但同一类中方法的 `@Transactional`注释`updateFoo(Foo)`优先于在类级别定义的事务设置。

```
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

##### `@Transactional` 设定值

`@Transactional`注解是元数据，指定的接口，类或方法必须有事务语义（例如，“调用该方法时启动一个全新的只读事务，悬浮剂任何现有的事务”）。默认`@Transactional`设置如下：

- 传播设置为 `PROPAGATION_REQUIRED.`
- 隔离级别为 `ISOLATION_DEFAULT.`
- 事务是读写的。
- 事务超时默认为基础事务系统的默认超时，如果不支持超时，则默认为无。
- 任何`RuntimeException`触发器都会回滚，而任何检查`Exception`都不会。

您可以更改这些默认设置。下表总结了`@Transactional`注释的各种属性：

| 属性                                                         | 类型                                         | 描述                                                         |
| :----------------------------------------------------------- | :------------------------------------------- | :----------------------------------------------------------- |
| [value](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#tx-multiple-tx-mgrs-with-attransactional) | `String`                                     | 可选的限定词，指定要使用的事务管理器。                       |
| [propagation](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#tx-propagation) | `enum`： `Propagation`                       | 可选的传播设置。                                             |
| `isolation`                                                  | `enum`： `Isolation`                         | 可选的隔离级别。仅适用于`REQUIRED`或的传播值`REQUIRES_NEW`。 |
| `timeout`                                                    | `int` （以秒为单位）                         | 可选的事务超时。仅适用于`REQUIRED`或的传播值`REQUIRES_NEW`。 |
| `readOnly`                                                   | `boolean`                                    | 读写与只读事务。仅适用于`REQUIRED`或的值`REQUIRES_NEW`。     |
| `rollbackFor`                                                | `Class`对象数组，必须从中派生`Throwable.`    | 必须引起回滚的异常类的可选数组。                             |
| `rollbackForClassName`                                       | 类名数组。这些类必须源自`Throwable.`         | 必须引起回滚的异常类名称的可选数组。                         |
| `noRollbackFor`                                              | `Class`对象数组，必须从中派生`Throwable.`    | 不能导致回滚的异常类的可选数组。                             |
| `noRollbackForClassName`                                     | `String`类名数组，必须从中派生`Throwable.`   | 不能引起回滚的异常类名称的可选数组。                         |
| `label`                                                      | `String`标签数组，用于向事务添加表达性描述。 | 标签可以由事务管理器评估，以将特定于实现的行为与实际事务相关联。 |

当前，您无法对事务名称进行显式控制，其中“名称”是指出现在事务监视器（如果适用）（例如，WebLogic的事务监视器）和日志输出中的事务名称。对于声明式事务，事务名称始终是完全合格的类名称+`.` 事务建议类的方法名称。例如，如果类的 `handlePayment(..)`方法`BusinessService`启动了事务，则事务的名称为：`com.example.BusinessService.handlePayment`。

##### 多个事务管理 `@Transactional`

大多数Spring应用程序仅需要一个事务管理器，但是在某些情况下，您可能需要在一个应用程序中使用多个独立的事务管理器。您可以使用注释的`value`或`transactionManager`属性 `@Transactional`来选择性地指定`TransactionManager`要使用的身份 。这可以是bean名称，也可以是事务管理器bean的限定符值。例如，使用限定符表示法，可以在应用程序上下文中将以下Java代码与以下事务管理器bean声明进行组合：

```
public class TransactionalService {

    @Transactional("order")
    public void setSomething(String name) { ... }

    @Transactional("account")
    public void doSomething() { ... }

    @Transactional("reactive-account")
    public Mono<Void> doSomethingReactive() { ... }
}
```

以下清单显示了bean声明：

```
<tx:annotation-driven/>

    <bean id="transactionManager1" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        ...
        <qualifier value="order"/>
    </bean>

    <bean id="transactionManager2" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        ...
        <qualifier value="account"/>
    </bean>

    <bean id="transactionManager3" class="org.springframework.data.r2dbc.connectionfactory.R2dbcTransactionManager">
        ...
        <qualifier value="reactive-account"/>
    </bean>
```

在这种情况下，在各个方法`TransactionalService`下单独的事务管理器运行，通过差异化`order`，`account`和`reactive-account` 预选赛。如果未找到特别限定的bean ，则仍将使用默认的`<tx:annotation-driven>`目标bean名称。`transactionManager``TransactionManager`

##### 自定义组成的注释

如果发现重复使用相同的属性`@Transactional`用于许多不同的方法，则[Spring的元注释支持](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-meta-annotations)使您可以为特定用例定义自定义组成的注释。例如，考虑以下注释定义：

```
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional(transactionManager = "order", label = "causal-consistency")
public @interface OrderTx {
}

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional(transactionManager = "account", label = "retryable")
public @interface AccountTx {
}
```

前面的注释使我们可以按照上一节的内容编写示例，如下所示：

```
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

在前面的示例中，我们使用了语法来定义事务管理器限定符和事务性标签，但是我们还可以包括传播行为，回滚规则，超时和其他功能。

#### 1.4.7. Transaction Propagation 事务传播

本节描述了Spring中事务传播的一些语义。请注意，本节不是对事务传播的适当介绍。相反，它详细介绍了有关Spring中事务传播的一些语义。

在Spring管理的事务中，请注意物理事务和逻辑事务之间的差异，以及传播设置如何应用于此差异。

##### 理解 `PROPAGATION_REQUIRED`

![需要TX道具](https://docs.spring.io/spring-framework/docs/current/reference/html/images/tx_prop_required.png)

`PROPAGATION_REQUIRED`强制执行物理事务，如果尚不存在当前事务，则在本地为当前范围执行，或参与为较大范围定义的现有“外部”事务。这是同一线程（例如，委派给几种存储库方法的服务立面，所有基础资源都必须参与服务级事务的服务立面）的优良默认设置。

==默认情况下，参与的事务将加入外部作用域的特征，而忽略本地隔离级别，超时值或只读标志（如果有）。如果希望在参与具有不同隔离级别的现有事务时拒绝隔离级别声明，请考虑将`validateExistingTransactions`标志切换到`true`事务管理器上。这种非宽容模式还拒绝只读不匹配（即，内部的读写事务试图参与只读的外部作用域）。==

当传播设置为时`PROPAGATION_REQUIRED`，将为应用该设置的每种方法创建一个逻辑事务作用域。每个这样的逻辑事务作用域可以单独确定仅回滚状态，而外部事务作用域在逻辑上独立于内部事务作用域。在标准`PROPAGATION_REQUIRED`行为的情况下，所有这些范围都映射到同一物理事务。因此，内部事务范围中设置的仅回滚标记确实会影响外部事务实际提交的机会。

但是，在内部事务范围设置仅回滚标记的情况下，外部事务尚未决定回滚本身，因此回滚（由内部事务范围默默触发）是意外的。此时`UnexpectedRollbackException`将抛出一个对应项 。这是预期的行为，因此永远不会误导事务的调用方以假定在确实未执行提交的情况下执行该提交。因此，如果内部事务（外部调用者不知道）将事务无提示地标记为仅回滚，则外部调用者仍会调用commit。外部调用者需要接收，`UnexpectedRollbackException`以清楚地指示已执行回滚。

##### 理解 `PROPAGATION_REQUIRES_NEW`

![发射道具需要新的](https://docs.spring.io/spring-framework/docs/current/reference/html/images/tx_prop_requires_new.png)

`PROPAGATION_REQUIRES_NEW`与相比`PROPAGATION_REQUIRED`，始终为每个受影响的事务范围使用独立的物理事务，而从不参与外部范围的现有事务。在这种安排中，基础资源事务是不同的，因此可以独立地提交或回滚，而外部事务不受内部事务的回滚状态的影响，并且内部事务的锁在完成后立即释放。这样一个独立的内部事务也可以声明其自己的隔离级别，超时和只读设置，而不继承外部事务的特征。

##### 理解 `PROPAGATION_NESTED`

`PROPAGATION_NESTED`使用具有多个可还原到的保存点的单个物理事务。这种部分回滚使内部事务范围触发其范围的回滚，尽管某些操作已回滚，但外部事务仍能够继续物理事务。此设置通常映射到JDBC保存点，因此仅适用于JDBC资源事务。参见Spring的[`DataSourceTransactionManager`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/jdbc/datasource/DataSourceTransactionManager.html)。

#### 1.4.8. Advising Transactional Operations事务事务咨询

假设您要同时运行事务性操作和一些基本的配置建议。您如何在的背景下实现这一目标`<tx:annotation-driven/>`？

调用该`updateFoo(Foo)`方法时，您希望看到以下操作：

- 配置的外观方面开始。
- 事务建议运行。
- 建议对象上的方法运行。
- 事务提交。
- 分析方面报告整个事务方法调用的确切持续时间。

==本章不关心任何详细的AOP解释（除非它适用于事务）。有关[AOP](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)配置和AOP的详细介绍，请参见[AOP](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)。==

以下代码显示了前面讨论的简单配置方面：

```
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

建议的顺序是通过`Ordered`界面控制的。有关建议订购的完整详细信息，请参阅 [建议订购](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-ataspectj-advice-ordering)。

以下配置创建了一个`fooService`以所需顺序应用了概要分析和事务方面的bean：

```
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
        <!-- run before the transactional advice (hence the lower order number) -->
        <property name="order" value="1"/>
    </bean>

    <tx:annotation-driven transaction-manager="txManager" order="200"/>

    <aop:config>
            <!-- this advice runs around the transactional advice -->
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

您可以以类似方式配置任意数量的其他方面。

以下示例创建与前两个示例相同的设置，但是使用纯XML声明性方法：

```
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
        <!-- run before the transactional advice (hence the lower order number) -->
        <property name="order" value="1"/>
    </bean>

    <aop:config>
        <aop:pointcut id="entryPointMethod" expression="execution(* x.y..*Service.*(..))"/>
        <!-- runs after the profiling advice (c.f. the order attribute) -->

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

    <!-- other <bean/> definitions such as a DataSource and a TransactionManager here -->

</beans>
```

先前配置的结果是一个`fooService`按顺序应用了概要分析和事务方面的bean。如果要使概要分析建议在进来的事务建议之后和出局的事务建议之前运行，则可以交换概要分析方面bean的`order`属性的值，以使其高于事务建议的订单值。

您可以以类似方式配置其他方面。

#### 1.4.9. Using `@Transactional` with AspectJ`@Transactional`与AspectJ一起使用

您还可以`@Transactional`通过AspectJ方面在Spring容器之外使用Spring Framework的支持。为此，首先使用注释为您的类（以及可选的类的方法）添加`@Transactional`注释，然后将您的应用程序与文件中的`org.springframework.transaction.aspectj.AnnotationTransactionAspect`定义 链接（织入） `spring-aspects.jar`。您还必须使用事务管理器配置方面。您可以使用Spring Framework的IoC容器来进行依赖注入方面。配置事务管理方面的最简单方法是使用`<tx:annotation-driven/>`元素并将`mode` 属性指定为`aspectj`，如[使用中所述`@Transactional`](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative-annotations)。因为这里我们专注于在Spring容器之外运行的应用程序，所以我们向您展示了如何以编程方式进行操作。

==在继续之前，您可能需要分别阅读《[使用》`@Transactional`](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative-annotations)和《 [AOP》](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)。==

以下示例显示了如何创建事务管理器并对其进行配置 `AnnotationTransactionAspect`以使用它：

```
// construct an appropriate transaction manager
DataSourceTransactionManager txManager = new DataSourceTransactionManager(getDataSource());

// configure the AnnotationTransactionAspect to use it; this must be done before executing any transactional methods
AnnotationTransactionAspect.aspectOf().setTransactionManager(txManager);
```

==使用此方面时，必须注释实现类（或该类中的方法或两者），而不是注释该类所实现的接口（如果有）。AspectJ遵循Java的规则，即不继承接口上的注释。==

`@Transactional`类上的注释指定了用于执行该类中任何公共方法的默认事务语义。

`@Transactional`类内方法的注释将覆盖类注释（如果存在）给出的默认事务语义。您可以注释任何方法，而不管可见性如何。

要使用织入您的应用程序`AnnotationTransactionAspect`，您必须使用AspectJ构建您的应用程序（请参阅 [AspectJ开发指南](https://www.eclipse.org/aspectj/doc/released/devguide/index.html)）或使用加载时织入。有关[使用AspectJ进行加载时织入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-aj-ltw)的讨论，请参见[Spring框架](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-aj-ltw)中的使用AspectJ进行加载时织入。

### 1.5.Programmatic Transaction Management 程序化事务管理

Spring框架通过使用以下两种方式提供程序化事务管理的方法：

- 的`TransactionTemplate`或`TransactionalOperator`。
- 一个`TransactionManager`直接的实现。

Spring团队通常建议`TransactionTemplate`在命令性流程中对程序化事务管理以及`TransactionalOperator`响应性代码进行推荐。第二种方法类似于使用JTA `UserTransaction`API，尽管异常处理不那么麻烦。

#### 1.5.1. Using the `TransactionTemplate` 使用`TransactionTemplate`

将`TransactionTemplate`采用相同的方法和其他Spring模板，如`JdbcTemplate`。它使用回调方法（使应用程序代码不必进行样板获取和释放事务性资源），并生成意向驱动的代码，因为您的代码仅专注于要执行的操作。

==如以下示例所示，使用`TransactionTemplate`绝对链接可将您与Spring的事务基础结构和API结合使用。程序化事务管理是否适合您的开发需求是您必须自己做的决定。==

必须在事务性上下文中运行并且显式使用的应用程序代码 `TransactionTemplate`类似于下一个示例。作为应用程序开发人员，您可以编写一个`TransactionCallback`实现（通常表示为匿名内部类），其中包含需要在事务上下文中运行的代码。然后，您可以将自定义实例传递`TransactionCallback`给上`execute(..)`公开的 方法`TransactionTemplate`。以下示例显示了如何执行此操作：

```
public class SimpleService implements Service {

    // single TransactionTemplate shared amongst all methods in this instance
    private final TransactionTemplate transactionTemplate;

    // use constructor-injection to supply the PlatformTransactionManager
    public SimpleService(PlatformTransactionManager transactionManager) {
        this.transactionTemplate = new TransactionTemplate(transactionManager);
    }

    public Object someServiceMethod() {
        return transactionTemplate.execute(new TransactionCallback() {
            // the code in this method runs in a transactional context
            public Object doInTransaction(TransactionStatus status) {
                updateOperation1();
                return resultOfUpdateOperation2();
            }
        });
    }
}
```

如果没有返回值，则可以将便捷`TransactionCallbackWithoutResult`类与匿名类一起使用，如下所示：

```
transactionTemplate.execute(new TransactionCallbackWithoutResult() {
    protected void doInTransactionWithoutResult(TransactionStatus status) {
        updateOperation1();
        updateOperation2();
    }
});
```

回滚中的代码可以通过调用`setRollbackOnly()`提供的`TransactionStatus`对象上的方法来回滚事务 ，如下所示：

```
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

##### 指定事务设置

您可以`TransactionTemplate`通过编程方式或配置方式指定事务设置（例如传播模式，隔离级别，超时等）。默认情况下，`TransactionTemplate`实例具有 [默认的事务设置](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative-txadvice-settings)。以下示例显示了针对特定事务设置的编程自定义`TransactionTemplate:`

```
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

以下示例`TransactionTemplate`通过使用Spring XML配置定义了具有一些自定义事务设置的：

```
<bean id="sharedTransactionTemplate"
        class="org.springframework.transaction.support.TransactionTemplate">
    <property name="isolationLevelName" value="ISOLATION_READ_UNCOMMITTED"/>
    <property name="timeout" value="30"/>
</bean>
```

然后，您可以`sharedTransactionTemplate` 根据需要将其注入到尽可能多的服务中。

最后，`TransactionTemplate`该类的实例是线程安全的，因为该实例不维护任何对话状态。`TransactionTemplate`但是，实例确实保持配置状态。因此，尽管许多类可以共享一个的单个实例`TransactionTemplate`，但是如果一个类需要使用`TransactionTemplate`具有不同设置（例如，不同的隔离级别）的a，则需要创建两个不同的`TransactionTemplate`实例。

#### 1.5.2. Using the `PlatformTransactionManager` 使用`TransactionOperator`

在`TransactionOperator`如下操作者设计是类似于其它反应性运算符。它使用回调方法（使应用程序代码不必进行样板获取和释放事务性资源），并生成意向驱动的代码，因为您的代码仅专注于要执行的操作。

==如以下示例所示，使用`TransactionOperator`绝对链接可将您与Spring的事务基础结构和API结合使用。程序化事务管理是否适合您的开发需求是您必须自己做的决定。==

必须在事务上下文中运行并且显式使用以下`TransactionOperator`类似的应用程序代码：

```
public class SimpleService implements Service {

    // single TransactionOperator shared amongst all methods in this instance
    private final TransactionalOperator transactionalOperator;

    // use constructor-injection to supply the ReactiveTransactionManager
    public SimpleService(ReactiveTransactionManager transactionManager) {
        this.transactionOperator = TransactionalOperator.create(transactionManager);
    }

    public Mono<Object> someServiceMethod() {

        // the code in this method runs in a transactional context

        Mono<Object> update = updateOperation1();

        return update.then(resultOfUpdateOperation2).as(transactionalOperator::transactional);
    }
}
```

`TransactionalOperator` 可以以两种方式使用：

- 使用Project Reactor类型（`mono.as(transactionalOperator::transactional)`）的操作员样式
- 其他情况的回调样式（`transactionalOperator.execute(TransactionCallback<T>)`）

回调中的代码可以通过调用`setRollbackOnly()` 提供的`ReactiveTransaction`对象上的方法来回滚事务，如下所示：

```
transactionalOperator.execute(new TransactionCallback<>() {

    public Mono<Object> doInTransaction(ReactiveTransaction status) {
        return updateOperation1().then(updateOperation2)
                    .doOnError(SomeBusinessException.class, e -> status.setRollbackOnly());
        }
    }
});
```

##### 取消信号

在反应式流中，a`Subscriber`可以取消`Subscription`并停止其 `Publisher`。运营商在工程反应堆，以及在其它的库，例如`next()`， `take(long)`，`timeout(Duration)`，和其他可以发出取消。无法知道取消的原因，无论是由于错误还是仅仅是出于进一步消费的兴趣。由于版本5.3的取消信号导致回滚。因此，重要的是要考虑事务下游使用的运营商 `Publisher`。特别是在a`Flux`或其他多值的情况下，`Publisher`必须使用完整的输出以使事务完成。

##### 指定事务设置

您可以为指定事务设置（例如传播模式，隔离级别，超时等）`TransactionalOperator`。默认情况下， `TransactionalOperator`实例具有 [默认的事务设置](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative-txadvice-settings)。以下示例显示了针对特定事务设置的自定义 `TransactionalOperator:`

```
public class SimpleService implements Service {

    private final TransactionalOperator transactionalOperator;

    public SimpleService(ReactiveTransactionManager transactionManager) {
        DefaultTransactionDefinition definition = new DefaultTransactionDefinition();

        // the transaction settings can be set here explicitly if so desired
        definition.setIsolationLevel(TransactionDefinition.ISOLATION_READ_UNCOMMITTED);
        definition.setTimeout(30); // 30 seconds
        // and so forth...

        this.transactionalOperator = TransactionalOperator.create(transactionManager, definition);
    }
}
```

### 1.6. Choosing Between Programmatic and Declarative Transaction Management 在编程事务管理和声明性事务管理之间进行选择

编程事务管理通常是一个好主意，只有当您有少量的事务操作时。例如，如果您有一个网络应用程序只需要某些更新操作的事务，那么您可能不希望使用Spring或任何其他技术来设置事务代理。在这种情况下，使用TransactionTemplate可能是一个好方法。能够显式地设置事务名称也是只能通过使用事务管理的编程方法来实现的。

另一方面，如果您的应用程序有许多事务操作，声明性事务管理通常是值得的。它将事务管理排除在业务逻辑之外，并且不难配置。当使用Spring框架而不是EJB-CMT时，声明性事务管理的配置成本大大降低。

### 1.7. Transaction-bound Events 事务绑定事件

从Spring4.2开始，事件的监听器可以绑定到事务的某个阶段。典型的例子是在事务成功完成时处理事件。当当前事务的结果对监听器非常重要时，这样做可以更灵活地使用事件。

可以使用@EventListener注释注册常规事件监听器。如果需要将其绑定到事务，请使用@TransactionalEventListener。当您这样做时，默认情况下监听器绑定到事务的提交阶段。

下一个例子展示了这个概念。假设一个组件发布一个订单生成事件，并且我们希望定义一个监听器，该监听器只应在成功提交已发布的事务后处理该事件。以下示例设置这样的事件监听器：

```java
@Component
public class MyComponent {

    @TransactionalEventListener
    public void handleOrderCreatedEvent(CreationEvent<Order> creationEvent) {
        // ...
    }
}
```

@TransactionaleEventListener注释公开了一个阶段属性，该属性允许您自定义监听器应绑定到的事务的阶段。有效的阶段是在“提交”之前、“提交”之后（默认）、在“回滚”之后，以及聚合事务完成（无论是提交还是回滚）的“完成”之后。

如果没有事务正在运行，则根本不会调用侦听器，因为我们无法遵守所需的语义。但是，可以通过将注释的fallbackExecution属性设置为true来覆盖该行为。

```
@TransactionalEventListener仅适用于由PlatformTransactionManager管理的线程绑定事务。由ReactiveTransactionManager管理的反应性事务使用Reactor上下文而不是线程本地属性，因此从事件监听器的角度来看，它不能参与任何兼容的活动事务。
```

### 1.8. Application server-specific integration 特定于应用程序服务器的集成

Spring的事务抽象通常与应用服务器无关。此外，Spring的Jtaransactionmanager类（可以选择执行JTA usertransaction和TransactionManager对象的JNDI查找）自动检测后一个对象的位置，后者因应用服务器而异。访问JTA TransactionManager可以增强事务语义，特别是支持事务暂停。有关详细信息，请参见JtaTransactionManager java文档。

Spring的JtaTransactionManager是运行在 Java EE 默认服务器上的标准选择，众所周知，它可以在所有常用服务器上运行。高级功能，如事务挂起，也可以在许多服务器上工作（包括GlassFish、JBoss和Geronimo），而不需要任何特殊的配置。然而，对于完全支持的事务暂停和进一步的高级集成，Spring包括用于WebLogic Server和WebSphere的特殊适配器。下面几节将讨论这些适配器。

对于标准场景，包括WebLogic Server和WebSphere，考虑使用方便的<tx:jta transaction manager/>配置元素。配置后，此元素将自动检测底层服务器，并选择平台可用的最佳事务管理器。这意味着您不需要显式地配置特定于服务器的适配器类（如下面几节中所讨论的）。相反，它们是自动选择的，标准JtaTransactionManager作为默认的后备。

#### 1.8.1. IBM WebSphere

在WebSphere6.1.0.9及更高版本上，建议使用的SpringJTA事务管理器是WebSphereRowTransactionManager。这个特殊的适配器使用IBM的UOWManager API，它在WebSphere Application Server 6.1.0.9和更高版本中提供。有了这个适配器，IBM正式支持Spring驱动的事务暂停（由PROPAGATION_REQUIRES_NEW启动的暂停和恢复）。

#### 1.8.2. Oracle WebLogic Server

在WebLogicServer9.0或更高版本上，您通常会使用WebLogicJtaTransactionManager，而不是常用的JtaTransactionManager类。普通JtaTransactionManager的这个特殊的特定于WebLogic的子类支持在WebLogic管理的事务环境中Spring的事务定义的全部功能，而不仅仅是标准的JTA语义。特性包括事务名称、每个事务的隔离级别以及在所有情况下正确地恢复事务。

### 1.9. Solutions to Common Problems 常见问题的解决方案

本节介绍一些常见问题的解决方案。

#### 1.9.1. Using the Wrong Transaction Manager for a Specific DataSource 对特定数据源使用错误的事务管理器

根据您选择的事务技术和需求，使用正确的PlatformTransactionManager实现。如果使用得当，Spring框架仅仅提供了一个简单和可移植的抽象类。如果使用全局事务，则必须使用org.springframework.transaction.jta.JtaTransactionManager（或其特定于应用程序服务器的子类）用于所有事务操作。否则，事务基础结构将尝试在容器数据源实例等资源上执行本地事务。这样的本地事务没有意义，一个好的应用服务器会将它们视为错误。

### 1.10. Further Resources 更多资源

有关Spring框架事务支持的更多信息，请参阅：

- [Distributed transactions in Spring, with and without XA](https://www.javaworld.com/javaworld/jw-01-2009/jw-01-spring-transactions.html)是一个JavaWorld演示，Spring的David Syer指导您了解Spring应用程序中分布式事务的七种模式，其中三种使用XA，四种不使用XA。
- [*Java Transaction Design Strategies*](https://www.infoq.com/minibooks/JTDS)是InfoQ提供的一本书，它提供了一个关于Java事务的有节奏的介绍。它还包括了如何配置和使用Spring框架和EJB3事务的并行示例。



