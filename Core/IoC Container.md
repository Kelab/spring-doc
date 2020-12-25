## 1.Ioc Container Ioc容器

本章介绍了Spring的Inversion of Control（IoC,控制反转）容器。

### 1.1. Introduction to the Spring IoC Container and Beans Ioc容器和Beans的介绍

本章介绍了控制反转（IoC）原理的Spring框架实现。IoC也称为依赖注入（DI）。在此过程中，对象仅通过**构造函数参数**，**工厂方法的参数**或在构造或从工厂方法返回后在对象实例上设置的属性**来定义其依赖项**（即，与它们一起使用的其他对象） 。然后，**容器在创建bean时注入那些依赖项**。此过程从根本上讲是通过使用类的直接构造或诸如服务定位器模式之类的控件来控制其依赖项的实例化或位置的bean本身的逆过程（因此称为Control的倒置）。

在`org.springframework.beans`和`org.springframework.context`包是Spring框架的IoC容器的基础。[`BeanFactory`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/beans/factory/BeanFactory.html) 接口提供了一种高级配置机制能力，能够管理任何类型的对象。 [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/context/ApplicationContext.html) 是的子接口`BeanFactory`。它增加了：

- 与Spring的AOP功能轻松集成
- 消息资源处理（用于国际化）
- 活动发布
- 应用层特定的上下文，例如`WebApplicationContext` 用于Web应用程序中的。

简而言之，`BeanFactory`提供了配置框架和基本功能，并且`ApplicationContext`增加了更多针对企业的功能。`ApplicationContext`是对一个`BeanFactory`的完整的超集，并只在本章节中描述Spring的IOC容器中使用。更多有关使用`BeanFactory`的详细信息，而不是`ApplicationContext,`移步 `BeanFactory`[链接](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanfactory)。

在Spring中，**构成应用程序的骨干**并**由Spring IoC容器管理**的**对象**称为bean。Bean是由Spring IoC容器实例化，组装和管理的对象。否则，bean仅仅是应用程序中许多对象之一。Bean及其之间的依赖关系反映在容器使用的配置元数据中。

### 1.2. Container Overview 容器概述

`org.springframework.context.ApplicationContext`接口代表Spring IoC容器，并主要负责实例化，配置和组装Bean。容器通过读取配置元数据（configuration metadata）获取有关要实例化，配置和组装哪些对象的指令。配置元数据以XML，Java注解或Java代码表示。它使您能够表达组成应用程序的对象以及这些对象之间的丰富相互依赖关系。

Spring提供了几种`ApplicationContext`的实现。在单机（stand-alone）应用程序中，通常创建[`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) 或[`FileSystemXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html)的实例 。尽管XML是定义配置元数据的传统格式，但是您可以通过提供少量XML配置来声明性地启用对这些其他元数据格式的支持，从而指示容器将Java注释或代码用作元数据格式。

在大多数应用程序场景中，不需要显式的用户代码来实例化Spring IoC容器的一个或多个实例。例如，在Web应用程序场景中，应用程序文件中的简单八行（约）样板Web描述符XML`web.xml`通常就足够了（请参阅[Web应用程序的便捷ApplicationContext实例化](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-create)）。如果您使用 [Spring Tools for Eclipse](https://spring.io/tools)（Eclipse支持的开发环境），则只需单击几下鼠标或击键即可轻松创建此样板配置。

下图显示了Spring的工作原理的高级视图。您的应用程序类与配置元数据结合在一起，因此，在`ApplicationContext`创建和初始化后，您将拥有一个完全配置且可执行的系统或应用程序。

![container magic](..\Core\.img\container-magic.png)

图1. Spring IoC容器

#### 1.2.1. Configuration Metadata 配置元数据

如上图所展示,Spring IoC容器使用一种形式的配置元数据。此配置元数据表示您作为应用程序开发人员如何告诉Spring容器在应用程序中实例化、配置和组装对象。

传统上，配置元数据以简单直观的XML格式提供，这是本章大部分内容用来传达Spring IoC容器的关键概念和功能的内容。

| :1st_place_medal: | 基于XML的元数据不是配置元数据的唯一允许形式。Spring IoC容器本身与实际写入此配置元数据的格式完全脱钩。如今，许多开发人员为他们的Spring应用程序选择 [基于Java的配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java)。 |
| ----------------- | ------------------------------------------------------------ |
|                   |                                                              |

有关在Spring容器中使用其他形式的元数据的信息，请参见：

- [基于注释的配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config)：Spring 2.5引入了对基于注释的配置元数据的支持。
- [基于Java的配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java)：从Spring 3.0开始，Spring JavaConfig项目提供的许多功能成为核心Spring Framework的一部分。因此，您可以使用Java而不是XML文件来定义应用程序类外部的bean。要使用这些新功能，请参阅 [`@Configuration`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)， [`@Bean`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html)， [`@Import`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html)，和[`@DependsOn`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html)注释。

Spring配置由容器必须管理的至少一个（通常是一个以上）bean定义组成。基于XML的配置元数据将这些bean配置为`<bean/>`顶级元素内的元素`<beans/>`元素。Java配置通常在`@Configuration`类中使用带注解`@Bean`的方法。

这些bean定义对应于组成应用程序的实际对象。通常，您定义Service层对象，数据访问对象（DAO），表示对象（例如Struts`Action`实例），基础结构对象（例如Hibernate `SessionFactories`，JMS`Queues`等）。通常，不会在容器中配置细粒度的域的对象，因为DAO和业务逻辑通常负责创建和加载域对象。但是，您可以使用Spring与AspectJ的集成来配置在IoC容器的控制范围之外创建的对象。请参阅[使用AspectJ与Spring依赖注入域对象](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-atconfigurable)。

以下示例显示了基于XML的配置元数据的基本结构：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

| 1    | The `id` attribute is a string that identifies the individual bean definition. |
| ---- | ------------------------------------------------------------ |
| 2    | 该`class`属性定义Bean的类型并使用完全限定的类名。            |

该`id`属性的值是指协作对象。在此示例中未显示用于引用协作对象的XML。有关更多信息，请参见 [依赖项](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-dependencies)。

#### 1.2.2. Instantiating a Container 实例化一个容器

提供给`ApplicationContext`构造函数的一个或多个位置路径是资源字符串，可让容器从各种外部资源（例如本地文件系统，Java等）中加载配置元数据`CLASSPATH`。

Java

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

|      | 在了解了Spring的IoC容器之后，您可能想了解更多有关Spring的 `Resource`抽象（如[参考资料中所述](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources)），它提供了一种方便的机制，用于从URI语法中定义的位置读取InputStream。具体而言，`Resource`如[应用程序上下文和资源路径中](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-app-ctx)所述， 路径用于构造应用[程序上下文](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-app-ctx)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

下面的例子给出了Service层(service.xml)对象的配置文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

以下示例显示了数据访问对象`daos.xml`文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

在前面的示例中，服务层由的`PetStoreServiceImpl`类和类`JpaAccountDao`和`JpaItemDao`（基于JPA对象关系映射标准）的两个数据访问对象组成。`property name`元素引用JavaBean属性的名称，`ref`元素引用另一个bean定义的名称。`id`和`ref`元素之间的链接表示了协作对象之间的依赖关系。有关配置对象依赖项的详细信息，请参见[依赖项](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-dependencies)。

##### Composing XML-based Configuration Metadata 组合基于xml的配置元数据

通过多个XML文件定义bean可能很有用。通常，每个单独的XML配置文件都代表体系结构中的逻辑层或模块。

您可以使用应用程序上下文构造函数从所有这些XML片段中加载bean定义。如上[一节中](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-instantiation)所示，此构造函数接受多个`Resource`位置 。或者，使用一个或多个出现的`<import/>`元素从另一个文件中加载bean定义。以下示例显示了如何执行此操作：

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

在前面的例子中，外部豆定义是从三个文件加载： `services.xml`，`messageSource.xml`，和`themeSource.xml`。所有位置路径是相对于定义文件的位置导入，因此`services.xml`必须在同一个目录或类路径位置作为文件做进口，而 `messageSource.xml`和`themeSource.xml` 必须在一个 `resources` 导入文件的位置下。如您所见，首斜杠被忽略。然而，**考虑到这些路径是相对的，最好完全不使用斜线**。根据Spring模式，被导入的文件的内容，包括顶级的<beans/>元素，必须是有效的XML bean定义。

> 可以但不建议使用相对的“ ../”路径引用父目录中的文件。这样做会创建对当前应用程序外部文件的依赖关系。特别是，不建议对`classpath:`URL（例如`classpath:../services.xml`）使用此引用，在URL中，运行时解析过程会选择“最近的”类路径根，然后查看其父目录。类路径配置的更改可能导致选择其他错误的目录。
>
> 您始终可以使用完全限定的资源位置来代替相对路径：例如`file:C:/config/services.xml`或`classpath:/config/services.xml`。但是，请注意，您正在将应用程序的配置耦合到特定的绝对位置。通常最好为这样的绝对位置保留一个间接寻址，例如，通过在运行时针对JVM系统属性解析的“ $ {…}”占位符。

命名空间本身提供了导入指令功能。Spring提供的一系列XML名称空间（例如`context`和`util`名称空间）中提供了超出普通bean定义的其他配置功能。

##### Groovy Bean定义DSL

作为外部化配置元数据的另一个示例，Bean定义也可以在Spring的Groovy Bean定义DSL中表达，如Grails框架所知。通常，这种配置位于“ .groovy”文件中，其结构如以下示例所示：

```groovy
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

这种配置样式在很大程度上等同于XML bean定义，甚至支持Spring的XML配置名称空间。它还允许通过`importBeans`指令导入XML bean定义文件。

#### 1.2.3. Using the Container 使用容器

该`ApplicationContext`是一个维护bean定义以及相互依赖的注册表的高级工厂的接口。通过使用方法 `T getBean(String name, Class<T> requiredType)`，您可以检索bean的实例。

将`ApplicationContext`让你能够读取bean的定义和访问它们，如下例所示：

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

使用Groovy配置，引导看起来非常相似。它有一个不同的上下文实现类，该类可识别Groovy（但也了解XML Bean定义）。以下示例显示了Groovy配置：

```java
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```

最灵活的变体是`GenericApplicationContext`与读取器委托结合在一起的，例如，与`XmlBeanDefinitionReader`XML文件结合使用，如以下示例所示：

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

您也可以将`GroovyBeanDefinitionReader`Groovy文件用于，如以下示例所示：

```java
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

您可以`ApplicationContext`在不同的配置源中从相同的Bean中混合和匹配此类阅读器委托，以读取Bean定义。

然后，您可以`getBean`用来检索bean的实例。该`ApplicationContext` 接口还有其他几种检索bean的方法，**但是理想情况下，您的应用程序代码永远不要使用它们**。实际上，您的应用程序代码应该根本不调用该 `getBean()`方法，因此完全不依赖于Spring API。例如，**Spring与Web框架的集成为各种Web框架组件（例如控制器和JSF管理的Bean）提供了依赖项注入，使您可以通过元数据（例如自动装配注释）声明对特定Bean的依赖项**。

### 1.3. Bean Overview Bean总览

Spring IoC容器管理一个或多个bean。这些bean是使用您提供给容器的配置元数据创建的（例如，以XML`<bean/>`定义的形式 ）。

在容器本身内，这些bean定义表示为`BeanDefinition` 对象，这些对象包含（除其他信息外）以下元数据：

- 包限定的类名称：通常，定义了Bean的实际实现类。
- Bean行为配置元素，用于声明Bean在容器中的行为（作用域，生命周期回调等）。
- 该bean完成其工作所需的其他bean。这些引用也称为协作者或依赖项。
- 要在新创建的对象中设置的其他配置设置-例如，池的大小限制或在管理连接池的bean中使用的连接数。

该元数据转换为构成每个bean定义的一组属性。下表描述了这些属性：

| Property                 | Explained in…                                                |
| ------------------------ | ------------------------------------------------------------ |
| Class                    | [Instantiating Beans 初始化Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
| Name                     | [Naming Beans 命名Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanname) |
| Scope                    | [Bean Scopes Beans范围](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes) |
| Constructor arguments    | [Dependency Injection 依赖注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Properties               | [Dependency Injection 依赖注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Autowiring mode          | [Autowiring Collaborators 自动装配](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire) |
| Lazy initialization mode | [Lazy-initialized Beans 懒加载Bean](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lazy-init) |
| Initialization method    | [Initialization Callbacks 初始化回调方法](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) |
| Destruction method       | [Destruction Callbacks 销毁Bean回调方法](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean) |

除了包含有关如何创建特定bean的信息的bean定义之外，`ApplicationContext`的实现还允许注册在容器外部（由用户）创建的现有对象。这是通过通过方法访问ApplicationContext的BeanFactory`getBeanFactory()`方法来完成的，该方法返回BeanFactory`DefaultListableBeanFactory`实现。`DefaultListableBeanFactory` 通过`registerSingleton(..)`和 `registerBeanDefinition(..)`方法支持此注册。但是，典型的应用程序只能与通过常规bean定义元数据定义的bean一起使用。

> Bean元数据和手动提供的单例实例需要尽可能早地注册，以便容器在自动装配和其他自省步骤中正确地对它们进行推理。虽然在某种程度上支持覆盖现有元数据和现有的单例实例，但在运行时注册新bean(与对工厂的实时访问同时进行)不受官方支持，可能会导致并发访问异常、bean容器中的不一致状态，或者两者都有。

#### 1.3.1. Naming Beans 命名Beans

每个bean具有一个或多个标识符。这些标识符在承载Bean的容器内必须唯一。一个bean通常只有一个标识符。但是，如果需要多个，则可以将多余的别名视为别名。

在基于XML配置文件，您可以使用`id`属性，该`name`属性，或两者来指定bean标识符。该`id`属性使您可以精确指定一个ID。按照惯例，这些名称是字母数字（“ myBean”，“ someService”等），但它们也可以包含特殊字符。如果要为bean引入其他别名，还可以在`name` 属性中指定它们，并用逗号（`,`），分号（`;`）或空格分隔。作为历史记录，在Spring 3.1之前的版本中，该`id`属性被定义为一种`xsd:ID`类型，该类型限制了可能的字符。从3.1开始，它被定义为`xsd:string`类型。注意，bean`id`唯一性仍然由容器强制执行，尽管不再由XML解析器执行。

您不需要为bean提供`name`或`id`。如果不提供 `name`或`id`显式提供，容器将为该bean生成一个唯一的名称。但是，如果要通过名称引用该bean，则通过使用`ref`元素或服务定位器样式查找，必须提供一个名称。不提供名称的动机与使用[内部bean](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-inner-beans)和[自动装配合作者有关](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire)。

><h4 style="text-align:center">Bean命名约定</h1>
>
>约定是在命名bean时将标准Java约定用于实例字段名称。也就是说，bean名称以小写字母开头，并从那里用驼峰式大小写。这样的名字的例子包括`accountManager`， `accountService`，`userDao`，`loginController`，等等。
>
>一致地命名Bean使您的配置更易于阅读和理解。另外，如果您使用Spring AOP，则在将建议应用于按名称相关的一组bean时，它会很有帮助。

>通过在类路径中扫描组件，Spring为未命名组件生成bean名称，遵循前面描述的规则:本质上，采用简单的类名并将其初始字符转换为小写。但是，在有多个字符并且第一个和第二个字符都是大写的(不寻常的)特殊情况下，原始的外壳将被保留。这些规则与java.beans.Introspector.decapitalize (Spring在这里使用)定义的规则相同。

##### Aliasing a Bean outside the Bean Definition 在Bean定义之外别名Bean

在bean定义本身中，可以通过使用由`id`属性指定的最多一个名称和`name`属性中任意数量的其他名称的组合来为bean提供多个名称。这些名称可以是同一个bean的等效别名，并且在某些情况下很有用，例如，通过使用特定于该组件本身的bean名称，让应用程序中的每个组件都引用一个公共依赖项。

但是，在实际定义bean的地方指定所有别名并不总是足够的。有时需要为在别处定义的bean引入别名。这在大型系统中通常是这种情况，在大型系统中，配置在每个子系统之间分配，每个子系统都有自己的对象定义集。在基于XML的配置元数据中，您可以使用`<alias/>`元素来完成此任务。以下示例显示了如何执行此操作：

```xml
<alias name="fromName" alias="toName"/>
```

在这种情况下，`fromName`在使用该别名定义之后，也可以将名为（位于同一容器中）的bean称为`toName`。

例如，子系统A的配置元数据可以使用名称来引用数据源`subsystemA-dataSource`。子系统B的配置元数据可以使用的名称引用数据源`subsystemB-dataSource`。组成使用这两个子系统的主应用程序时，主应用程序使用名称引用数据源`myApp-dataSource`。要使所有三个名称都引用相同的对象，可以将以下别名定义添加到配置元数据中：

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

现在，每个组件和主应用程序都可以通过唯一的名称引用数据源，并保证不与任何其他定义冲突（有效地创建名称空间），但它们引用的是同一bean。

> <h4 style="text-align:center">Java配置</h1>
>
> 如果使用Java配置，则`@Bean`注释可用于提供别名。有关详细信息，请参见[使用`@Bean`注释](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-bean-annotation)。

#### 1.3.2. Instantiating Beans 实例化Beans

bean定义本质上是创建一个或多个对象的诀窍。当被请求时，容器查看已命名bean的配方，并使用该bean定义封装的配置元数据来创建(或获取)实际对象。

如果使用基于XML的配置元数据，则可以在`<bean/>`元素的`class`属性中指定要实例化的对象的类型（或类）。此 `class`属性（在内部是 实例的`Class`属性`BeanDefinition`）通常是必需的。（有关异常，请参见 [使用实例工厂方法实例化](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class-instance-factory-method)和[Bean定义继承](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-child-bean-definitions)。）可以通过以下两种方式之一使用该`Class`属性：

- 通常，在容器本身通过反射性地调用其构造函数直接创建Bean的情况下，指定要构造的Bean类，这在某种程度上等同于使用`new`运算符的Java代码。
- 要指定包含要创建对象而调用的`static`工厂方法的实际类，在不太常见的情况下，容器将在类上调用`static`工厂方法来创建Bean。从`static`工厂方法的调用返回的对象类型可以是相同的类，也可以是完全不同的另一类。

> **内部班级名称**
>
> 如果要为`static`嵌套类配置Bean定义，则必须使用嵌套类的二进制名称。
>
> 例如，如果在`com.example`包中有一个名为`SomeThing`的类，并且该 `SomeThing`类有一个名为的`static`嵌套类`OtherThing`，则`class` bean定义上的属性值将为`com.example.SomeThing$OtherThing`。
>
> 请注意，名称中使用了`$`字符，以将嵌套的类名与外部类名分开。

##### Instantiation with a Constructor 用构造函数实例化

当通过构造方法创建一个bean时，所有普通类都可以被Spring使用并兼容。也就是说，正在开发的类不需要实现任何特定的接口或以特定的方式进行编码。只需指定bean类就足够了。但是，根据您用于该特定bean的IoC的类型，您可能需要一个默认（空）构造函数。

Spring IoC容器几乎可以管理您要管理的任何类。它不仅限于管理真正的JavaBean。大多数Spring用户更喜欢实际的JavaBean，它仅具有默认（无参数）构造函数，并具有根据容器中的属性建模的适当的setter和getter。您还可以在容器中具有更多奇特的非Bean样式类。例如，如果您需要使用绝对不符合JavaBean规范的旧式连接池，则Spring也可以对其进行管理。

使用基于XML的配置元数据，您可以如下指定bean类：

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

有关在构造对象之后向构造函数提供参数（如果需要）并设置对象实例属性的机制的详细信息，请参见 [注入依赖项](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators)。

##### 用静态工厂方法实例化

当定义一个用静态工厂方法创建的bean时，使用`class`属性来指定包含`static`工厂方法的类，并使用名为`factory-method`的属性来指定工厂方法本身的名称。您应该能够调用这个方法(带有可选参数，如后面所述)并返回一个活动对象，然后将其视为通过构造函数创建的。这种bean定义的一个用途是在遗留代码中调用“静态”工厂。

以下bean定义指定通过调用工厂方法来创建bean。该定义不指定返回对象的类型（类），而仅指定包含工厂方法的类。在此示例中，该`createInstance()` 方法必须是静态方法。以下示例显示如何指定工厂方法：

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

以下示例显示了可与前面的bean定义一起使用的类：

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

有关为工厂方法提供（可选）参数并在从工厂返回对象之后设置对象实例属性的机制的详细信息，请参见[Dependencies and Configuration in Detail](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-properties-detailed)。

##### Instantiation by Using an Instance Factory Method 使用实例工厂方法实例化

类似于通过[静态工厂方法进行](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class-static-factory-method)实例化，使用实例工厂方法进行实例化会从容器中调用现有bean的非静态方法来创建新bean。要使用此机制，请将`class`属性留空，并在`factory-bean`属性中指定当前（或父容器或祖先容器）中包含要创建该对象的实例方法的Bean的名称。使用`factory-method`属性设置工厂方法本身的名称。以下示例显示了如何配置此类Bean：

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

以下示例显示了相应的类：

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以包含一个以上的工厂方法，如以下示例所示：

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

以下示例显示了相应的类：

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

这种方法表明，工厂Bean本身可以通过依赖项注入（DI）进行管理和配置。[详细信息，](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-properties-detailed)请参见[依赖性和配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-properties-detailed)。

|      | 在Spring文档中，“ factory bean”是指在Spring容器中配置并通过[实例](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class-instance-factory-method)或 [静态](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class-static-factory-method)工厂方法创建对象的bean 。相反， `FactoryBean`（注意大写）是指特定于Spring的 [`FactoryBean` ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-factorybean)实现类。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### Determining a Bean’s Runtime Type 确定Bean的运行时类型

确定特定bean的运行时类型并非易事。Bean元数据定义中的指定类只是初始类引用，可能与声明的工厂方法结合使用，或者是`FactoryBean`可能导致Bean的运行时类型不同的类，或者在实例的情况下根本不设置-级别工厂方法（通过指定`factory-bean`名称解析）。另外，AOP代理可以使用基于接口的代理包装bean实例，而目标Bean的实际类型（仅是其实现的接口）的暴露程度有限。

找出特定bean的实际运行时类型的推荐方法是`BeanFactory.getType`调用指定的bean名称。这考虑了上述所有情况，并返回了`BeanFactory.getBean`要针对相同bean名称返回的对象的类型。

###  1.4. Dependencies 依赖

典型的企业应用程序不包含单个对象（或Spring术语中的bean）。即使是最简单的应用程序，也有一些对象可以协同工作，以呈现最终用户视为一致的应用程序。下一部分将说明如何从定义多个独立的Bean定义到实现对象协作以实现目标的完全实现的应用程序。

#### 1.4.1. Dependency Injection 依赖注入

依赖注入（DI）是一个过程，通过该过程，对象在被构建或从工厂方法返回时**仅通过构造函数参数，工厂方法的参数或在构造或创建对象实例后在对象实例上设置的属性**来定义其依赖关系（即，与它们一起工作的其他对象）。然后，容器在创建bean时注入那些依赖项。从根本上讲，此过程是通过使用类的直接构造或服务定位器模式来控制bean自身依赖关系的实例化或位置的bean本身的逆过程（因此称为Control Inversion，控制反转）。

使用DI原理，代码更加简洁，当为对象提供依赖项时，去耦会更有效。该对象不查找其依赖项，并且不知道依赖项的位置或类。结果，您的类变得更易于测试，尤其是当依赖项依赖于接口或抽象基类时，它们允许在单元测试中使用存根或模拟实现。

DI存在两个主要变体：[基于构造函数的依赖注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-constructor-injection)和[基于Setter的依赖注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-setter-injection)。

##### Constructor-based Dependency Injection 基于构造方法的依赖注入

基于构造函数的DI是通过容器调用具有多个参数的构造函数来完成的，每个参数表示一个依赖项。这与调用带有特定参数的`static`工厂方法来构造Bean几乎是等效的（在Factory就决定了依赖项目，译者注），并且本次讨论也将构造函数和`static`工厂方法的参数视为类似。以下示例显示了只能通过构造函数注入进行依赖项注入的类：

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

注意，该类没有什么特别的。它是一个POJO，不依赖于特定于容器的接口，基类或注释。

###### Constructor Argument Resolution **构造函数参数解析**

构造函数参数解析匹配通过使用参数的类型进行。如果Bean定义的构造函数参数中不存在潜在的歧义，则在实例化Bean时，在Bean定义中定义构造函数参数的顺序就是将这些参数提供给适当的构造函数的顺序。考虑以下类：

```java
package x.y;
public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

假设`ThingTwo`和`ThingThree`类不是通过继承关联的，则不存在潜在的歧义。因此，以下配置可以正常工作，并且您**无需**在`<constructor-arg/>` 元素中**显式指定构造函数参数索引或类型**（即显示指定bean，译者注）。

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当引用另一个bean时，类型是已知的，并且可以发生匹配（与前面的示例一样）。当使用诸如的简单类型时 `<value>true</value>`，Spring无法确定值的类型，因此在没有帮助的情况下无法按类型进行匹配。考虑以下类：

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

**Constructor argument type matching 构造函数参数类型匹配**

在上述情况下，如果通过使用`type`属性显式指定构造函数参数的类型，则容器可以使用简单类型的类型匹配。如下例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

**Constructor argument index 构造函数参数索引**

您可以使用该`index`属性来明确指定构造函数参数的索引，如以下示例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

除了解决多个简单值的歧义性外，指定索引还可以解决构造函数具有两个相同类型参数的歧义性。

|      | 索引从0开始。 |
| ---- | ------------- |
|      |               |

**Constructor argument name 构造函数参数名称**

您还可以使用构造函数参数名称来消除歧义，如以下示例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

请记住，要立即使用该功能，必须在启用调试标志的情况下编译代码，以便Spring可以从构造函数中查找参数名称。如果您不能或不想使用debug标志编译代码，则可以使用 [@ConstructorProperties](https://download.oracle.com/javase/8/docs/api/java/beans/ConstructorProperties.html) JDK注释显式命名构造函数参数。然后，样本类必须如下所示：

```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

#####  Setter-based Dependency Injection 基于Setter的依赖注入

基于Setter的DI是通过在调用无参数构造函数或无参数`static`工厂方法以实例化您的bean之后，在您的bean上调用setter方法来完成的。

下面的示例显示只能通过使用纯setter注入来依赖注入的类。此类是常规的Java。它是一个POJO，不依赖于特定于容器的接口，基类或注释。

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

`ApplicationContext`支持基于构造函数和Setter的DI管理的Beans。在已经通过构造函数方法注入了某些依赖项之后，它还支持基于setter的DI。您可以以的形式配置依赖项，并将`BeanDefinition`其与`PropertyEditor`实例结合使用以将属性从一种格式转换为另一种格式。但是，大多数Spring用户并不直接（即以编程方式）使用这些类，而是使用XML`bean` 定义，带注释的组件（即带有`@Component`， `@Controller`等等的类）或`@Bean`基于Java的`@Configuration`类中的方法。然后将这些源在内部转换为的实例，`BeanDefinition`并用于加载整个Spring IoC容器实例。

> <h4 style="text-align:center">基于构造函数或基于setter的DI？</h4>
>
> 由于可以混合使用基于构造函数的DI和基于设定值的DI，因此将构造函数用于强制性依赖项并将setter方法或配置方法用于可选的依赖项是一个很好的经验法则。注意， 在setter方法上使用[@Required](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-required-annotation)批注可以使该属性成为必需的依赖项。但是，最好使用带有参数的程序验证的构造函数注入。
>
> Spring团队通常提倡使用构造函数注入，因为它可以让您将应用程序组件实现为不可变对象，并确保必需的依赖项不存在`null`。此外，注入构造函数的组件始终以完全初始化的状态返回到客户端（调用）代码。附带说明一下，大量的构造函数自变量是一种不好的代码味，这表明该类可能承担了太多的职责，应将其重构以更好地解决关注点分离问题。
>
> Setter注入主要应仅用于可以在类中分配合理的默认值的可选依赖项。否则，必须在代码使用依赖项的任何地方执行非空检查。setter注入的一个好处是，setter方法使该类的对象在以后可以重新配置或重新注入。因此，通过[JMX MBean进行](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jmx)管理是用于setter注入的引人注目的用例。
>
> 使用最适合特定班级的DI风格。有时，在处理您没有源代码的第三方类时，将为您做出选择。例如，如果第三方类未公开任何setter方法，则构造函数注入可能是DI的唯一可用形式。



##### Dependency Resolution Process 依赖性解析过程

容器执行bean依赖项解析，如下所示：

- 配置元数据（ configuration metadata，配置文件，译者注）创建和初始化`ApplicationContext`描述所有bean。可以通过XML，Java代码或注释指定配置元数据。
- 对于每个bean，其依赖项都以**属性，构造函数参数或static-factory方法的参数的形式**表示（如果使用它而不是普通的构造函数）。实际创建Bean时，会将这些依赖项提供给Bean。
- 每个属性或构造函数参数都是要设置的值的实际定义，或者是对容器中另一个bean的引用。
- 作为值的每个属性或构造函数参数都将从其指定的格式转换为该属性或构造函数参数的实际类型。**默认情况下，Spring能够以String类型提供值转换成所有内置类型，比如`int`， `long`，`String`，`boolean`，等等。**

在**创建容器时，Spring容器会验证每个bean的配置。但是，在实际创建Bean之前，不会设置Bean属性本身**。创建容器时，将创建具有单例作用域并设置为预先实例化（默认）的Bean。范围在[Bean范围](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes)中定义。否则，仅在请求时才创建Bean。创建和分配bean的依赖关系及其依赖关系（依此类推）时，创建bean可能会导致创建一个bean图。请注意，这些依赖项之间的分辨率不匹配可能会显示得较晚-即在第一次创建受影响的bean时。

> <h4 style="text-align:center">循环依赖</h4>
>
> 如果主要使用构造函数注入，则可能会创建无法解决的循环依赖方案。
>
> 例如：**A类通过构造函数注入需要B类的实例，而B类通过构造函数注入需要A类的实例。如果您将A类和B类的bean配置为相互注入，则Spring IoC容器会在运行时检测到此循环引用，并抛出 `BeanCurrentlyInCreationException`。**
>
> 一种可能的解决方案是编辑某些类的源代码，这些类的源代码由设置者而不是构造函数来配置。或者，避免构造函数注入，而仅使用setter注入。换句话说，尽管不建议这样做，但是您可以**使用setter注入配置循环依赖项**。
>
> (spring 只能解决setter版本的循环依赖，使用三级缓存，singletonFactories：**进入实例化阶段的单例对象工厂的cache** ，三级缓存；earlySingletonObjects ：**完成实例化但是尚未初始化的，提前暴光的单例对象的Cache** ，二级缓存；singletonObjects：**完成初始化的单例对象的cache**，一级缓存。译者注）
>
> 与典型情况（没有循环依赖关系）不同，Bean A和Bean B之间的循环依赖关系迫使其中一个Bean在完全完全初始化之前被注入另一个Bean（经典的“鸡与蛋”场景）。

通常，您可以信任Spring做正确的事。它在容器加载时检测配置问题，例如对不存在的Bean的引用和循环依赖项。在实际创建bean时，Spring设置属性并尽可能晚地解决依赖关系。这意味着，如果创建对象或其依赖项之一有问题，则正确加载了Spring的容器以后可以在您请求对象时生成异常-例如，由于缺少或无效，Bean引发异常属性。某些配置问题的这种潜在的延迟可见性是为什么`ApplicationContext`默认情况下，实现会实例化单例bean。在实际需要这些bean之前创建这些bean需要花费一些前期时间和内存，但您发现它们在`ApplicationContext`创建时（而不是稍后）就会发现配置问题。您仍然可以覆盖此默认行为，以便单例bean延迟初始化，而不是预先实例化。

如果不存在循环依赖关系，则在将一个或多个协作Bean注入从属Bean时，每个协作Bean在注入到从属Bean中之前都已完全配置。这意味着，如果bean A依赖于bean B，则Spring IoC容器会在对bean A调用setter方法之前完全配置beanB。换句话说，bean被实例化（如果不是预先实例化的singleton）。 设置其依赖项，并调用相关的生命周期方法（例如已[配置的init方法](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) 或[InitializingBean回调方法](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean)）。

##### Examples of Dependency Injection依赖注入的例子

以下示例将基于XML的配置元数据用于基于setter的DI。Spring XML配置文件的一小部分指定了一些bean定义，如下所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的`ExampleBean`类：

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的示例中，声明了setter以与XML文件中指定的属性匹配。以下示例使用基于构造函数的DI：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的`ExampleBean`类：

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

Bean定义中指定的构造函数参数用作的构造函数的参数`ExampleBean`。

现在考虑该示例的一个变体，在该变体中，不是使用构造函数，而是告诉Spring调用`static`工厂方法以返回对象的实例：

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的`ExampleBean`类：

```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

`static`工厂方法的参数由`<constructor-arg/>`元素提供，就像实际使用构造函数一样。factory方法返回的类的类型不必与包含`static`factory方法的类具有相同的类型（尽管在此示例中是）。实例（非静态）工厂方法可以以基本上相同的方式使用（除了使用`factory-bean`属性代替使用`class`属性外），因此在此不讨论这些细节。

#### 1.4.2. Dependencies and Configuration in Detail 依赖性和详细配置

如上[一节所述](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators)，您可以将bean属性和构造函数参数定义为对其他托管bean（协作者）的引用或内联定义的值。Spring的基于XML的配置元数据为此目的在其`<property/>`和`<constructor-arg/>`元素中支持子元素类型。

##### Straight Values (Primitives, Strings, and so on) 直值（原语，字符串等）

在`value`所述的属性`<property/>`元素指定属性或构造器参数的人类可读的字符串表示。Spring的 [转换服务](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#core-convert-ConversionService-API)用于将这些值从转换`String`为属性或参数的实际类型。以下示例显示了设置的各种值：

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="misterkaoli"/>
</bean>
```

以下示例使用[p-namespace](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-p-namespace)进行更简洁的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="misterkaoli"/>

</beans>
```

前面的XML更简洁。但是，除非在创建bean定义时使用支持自动属性完成的IDE（例如[IntelliJ IDEA](https://www.jetbrains.com/idea/)或[Spring Tools for Eclipse](https://spring.io/tools)），否则错字是在运行时而不是设计时发现的。强烈建议您使用此类IDE帮助。

您还可以配置`java.util.Properties`实例，如下所示：

```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

Spring容器使用JavaBeans机制将`<value/>`元素内的文本转换为 `java.util.Properties`实例`PropertyEditor`。这是一个不错的捷径，并且是Spring团队偏爱使用嵌套`<value/>`元素而不是使用`value`属性样式的几个地方之一。

###### The `idref` element `idref`元素

所述`idref`元件是一个简单的防错方法对通过`id`（一个字符串值-而不是参考）在该容器另一个bean的一个`<constructor-arg/>`或`<property/>` 元件。以下示例显示了如何使用它：

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

前面的bean定义片段（在运行时）与下面的片段完全等效：

```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种形式优于第二种形式，因为使用`idref`标记可以使容器在部署时验证所引用的命名Bean实际上是否存在。在第二个变体中，不对传递给Bean`targetName`属性的值进行验证`client`。拼写错误仅在`client`实际实例化bean时才发现（可能会导致致命的结果）。如果该`client` bean是[原型](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes)bean，则只能在部署容器后很长时间才能发现此错字和所产生的异常。

|      | 元素 的`local`属性在`idref`4.0 bean XSD中不再受支持，因为它不再提供常规`bean`引用上的值。升级到4.0模式时，将现有`idref local`引用更改`idref bean`为。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

其中一个共同的地方（至少在早期比Spring 2.0版本）`<idref/>`元素带来的值在配置[AOP拦截](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-pfb-1)在 `ProxyFactoryBean`bean定义。`<idref/>`在指定拦截器名称时使用元素可防止您拼写错误的拦截器ID。

##### References to Other Beans (Collaborators) 对其他Bean的引用（协作者）

所述`ref`元件是内部的最终元件`<constructor-arg/>`或`<property/>` 定义元素。在这里，您将bean的指定属性的值设置为对容器管理的另一个bean（协作者）的引用。引用的bean是要设置其属性的bean的依赖关系，并且在设置属性之前根据需要对其进行初始化。（如果协作者是单例bean，则它可能已经由容器初始化了。）所有引用最终都是对另一个对象的引用。作用域和验证取决于是否通过`bean`或`parent`属性指定另一个对象的ID或名称。

通过`<ref/>`标记的`bean`属性指定目标bean是最通用的形式，并且允许创建对同一容器或父容器中任何bean的引用，而不管它是否在同一XML文件中。该`bean`属性的值 可以`id`与目标Bean的属性相同，也可以与目标Bean的属性之一相同`name`。以下示例显示如何使用`ref`元素：

```xml
<ref bean="someBean"/>
```

通过`parent`属性指定目标Bean将创建对当前容器的父容器中的Bean的引用。该`parent` 属性的值可以`id`与目标Bean的属性或目标Bean的`name`属性中的值之一相同。目标Bean必须位于当前容器的父容器中。主要在具有容器层次结构并且要使用与父bean名称相同的代理将现有bean封装在父容器中时，才应使用此bean参考变量。以下清单对显示了如何使用该`parent`属性：

```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

|      | 元素 的`local`属性在`ref`4.0 bean XSD中不再受支持，因为它不再提供常规`bean`引用上的值。升级到4.0模式时，将现有`ref local`引用更改`ref bean`为。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### Inner Beans 内部Beans

元件`<property/>`或`<constructor-arg/>`元件内部的`<bean/>`限定内部Beans，如下面的示例所示：

```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean定义不需要定义的ID或名称。如果指定，则容器不使用该值作为标识符。容器还会忽略`scope`创建时的标志，因为内部Bean始终是匿名的，并且始终与外部Bean一起创建。不能独立访问内部bean，也不能将它们注入到协作bean中而不是封装在bean中。

作为一个极端的例子，可以从自定义范围接收破坏回调，例如，针对单例bean中包含的请求范围内的bean。内部bean实例的创建与其包含的bean绑定在一起，但是销毁回调使它可以参与请求范围的生命周期。这不是常见的情况。内部bean通常只共享其包含bean的作用域。

##### Collection

`<list/>`， `<set/>`， `<map/>`，和 `<props/>`元件分别设置属性 `Collection` 和Java的参数类型 `List`， `Set`， `Map`，和 `Properties`。 以下示例显示了如何使用它们： 

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

映射键或值的值或设置值也可以是以下任意一种  以下要素： 

```xml
bean | ref | idref | list | set | map | props | value | null
```

###### Collection Merging Collection合并

Spring容器还支持合并集合。 一个应用程序   开发者可以定义一个父 <list/>， <map/>， <set/>或 <props/>元素   并有孩子 <list/>， <map/>， <set/>或 <props/>元素继承和   覆盖父集合中的值。 也就是说，子集合的值是   将父项和子项集合的元素与子项的元素合并的结果   集合元素覆盖父集合中指定的值。 

关于合并的这一节讨论了父子bean机制。 不熟悉的读者  带有父级和子级bean定义的用户可能希望阅读  [相关部分， ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-child-bean-definitions)然后继续。 

下面的示例演示了集合合并： 

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

注意使用  `child`bean定义的`adminEmails`属性的`<props/>`的元素上的`merge=true`属性 。 当 `child`bean被解析并由容器实例化时，结果实例为 包含合并子项的  `adminEmails`集合与parent的 `adminEmails`集合的`adminEmails` `Properties`集合。 以下清单 显示结果： 

> ps：这段话有点拗口，可能是因为中英表达习惯不太一样，总之就是把孩子的元素加到了父代的集合中。



```
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```

孩子 `Properties`集合的值设定继承了所有属性元素  parent `<props/>`，而子级的 `support`值覆盖其中的值  父集合。 

这一合并行为同样适用于 `<list/>`， `<map/>`和 `<set/>` 集合类型。 在 的特定情况下 `<list/>`元素 ，语义  与 相关联 `List`集合类型 （即 `ordered` 值的集合）。 父级的值位于所有子级列表的值之前  价值观。 在的情况下 `Map`， `Set`和 `Properties`集合类型，没有顺序  存在。 因此，对于底层的集合类型，没有排序语义是有效的  相关的 `Map`， `Set`和 `Properties`与容器 实现类型内部使用。 

###### Limitations of Collection Merging 集合合并的局限性 

您不能合并不同的集合类型（例如 `Map`和 `List`）。 如果你  如果尝试这样做， 一个适当的 `Exception`则会抛出。 `merge`属性必须在较低的继承的子类定义中指定。 在指定 `merge`属性  父集合定义是多余的，不会导致所需的合并。 

###### Strongly-typed collection 强类型集合 

随着Java 5中泛型类型的引入，您可以使用强类型集合strongly typed collections。  也就是说，可以声明一个 `Collection`类型，使其只能包含（例如）`String`元素。 如果您使用Spring依赖注入将其强类型 `Collection`化为bean，可以利用Spring的类型转换支持，以便您的强类型元素 `Collection` 实例在添加到之前先转换为适当的类型 `Collection`。  以下Java类和bean定义显示了如何执行此操作： 

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

````xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
````

当 `accounts`的属性 `something`准备注入bean时 ，泛型有关强类型元素类型 `Map<String, Float>`的信息是可以通过反射获得。 因此，Spring的类型转换基础架构可以识别 `Float`类型的各种值元素，以及字符串值（ `9.99, 2.75`，和  `3.99`）转换为实际 `Float`类型。 

##### 空字符串值和空字符串值 

Spring将属性等的空参数视为空`Strings`。 的  以下基于XML的配置元数据片段将 `email`属性设置为空  `String`值（“”）。 

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

前面的示例等效于以下Java代码： 

```java
exampleBean.setEmail("");
```

该 `<null/>`元素处理 `null`的值。 以下清单显示了一个示例： 

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

前面的配置等效于下面的Java代码： 

```java
exampleBean.setEmail(null);
```

##### XML Shortcut with the p-namespace 具有p-命名空间的XML快捷方式 

p-namespace允许您使用 `bean`元素的属性（而不是嵌套的 `<property/>`元素）来描述协作Bean的属性值，或同时描述两者。 

Spring支持 可扩展配置格式 [带有名称空间的 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#xsd-schemas)，  它们基于XML模式定义。  `beans`的配置格式在本章的论述在XML Schema文档中定义。 但是，p命名空间未定义  在XSD文件中，仅存在于Spring的核心中。 

以下示例显示了两个XML代码段（第一个使用标准XML格式，第二种使用p-namespace）解析为相同的结果： 

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```

该示例显示了 调用的p-namespace中的属性 `email`在bean定义中 。  这告诉Spring包含一个属性声明。 如前所述，p-namespace没有架构定义，因此您可以设置属性的名称属性名称。 

下一个示例包括另外两个bean定义，它们都引用了另一个bean： 

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

此示例不仅包括使用p-namespace的属性值  而且还使用一种特殊的格式来声明属性引用。 而第一个豆  定义用于 `<property name="spouse" ref="jane"/>`从bean创建引用  `john`到bean `jane`，第二个bean定义 `p:spouse-ref="jane"`用作  属性做完全相同的事情。 在这种情况下， `spouse`是属性名称，  而该 `-ref`部分表明这不是一个直接值，而是一个  引用另一个bean。 

|      | p命名空间不如标准XML格式灵活。 例如格式  用于声明属性引用与以结尾的属性发生冲突 `Ref`，而  标准XML格式没有。 我们建议您谨慎选择方法，  与您的团队成员进行交流，以避免产生使用所有  三种方法同时进行。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### XML Shortcut with the c-namespace具有c-namespace的XML快捷方式 

与 [带有p-namespace XML Shortcut ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-p-namespace)类似 ，在Spring中引入的c-namespace  3.1，允许使用内联属性来配置构造函数参数 ，然后嵌套 `constructor-arg`元素。 

以下示例使用 `c:`名称空间执行与from相同的操作  [基于构造函数的依赖注入 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-constructor-injection)： 

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

该 `c:`命名空间使用相同的约定的 `p:`一种：（a尾随 `-ref`用于  bean引用），用于通过其名称设置构造函数参数。 同样，  即使未在XSD模式中定义，也需要在XML文件中声明它  （它存在于Spring内核中）。 

在极少数情况下，构造函数参数名称不可用（通常是  字节码是在没有调试信息的情况下编译的），您可以使用回退到  参数索引，如下所示： 

```xml
<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```

|      | 由于XML语法的原因，索引符号要求开头 `_`，  因为XML属性名称不能以数字开头（即使某些IDE允许）。  相应的索引符号也可用于 `<constructor-arg>`元素，但  通常不常用，因为在那里简单的声明顺序就足够了。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

在实践中，构造函数的解析  [机制 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-ctor-arguments-resolution)匹配 非常有效  参数，因此除非您确实需要，否则我们建议使用名称符号 （name）整个配置。 

##### Compound Property Names 复合属性名称 

设置bean属性时，可以使用复合或嵌套属性名称，只要除最终属性名称外，路径的所有组件均不是 `null`。 考虑一下  以下bean定义： 

```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

该 `something`bean有一个 `fred`属性，它有一个 `bob`属性，它有一个 `sammy` 属性，并且将最终 `sammy`属性设置为的值 `123`。 为了  要正常工作，的 `fred`财产 `something`和的 `bob`财产 `fred`不得  是 `null`被构造豆后。否则，将引发 一个`NullPointerException`。 

#### 1.4.3. Using `depends-on`  使用 `depends-on`

如果一个bean是另一个bean的依赖项，则通常意味着将一个bean设置为a  另一个的属性。 通常，您可以通过 [` <ref/>`元素 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-ref-element)基于XML的配置元数据中的 。 但是，有时bean之间的依赖不太直接。 一个示例是当需要在类中使用静态初始化程序时  触发，例如用于数据库驱动程序注册。 该 `depends-on`属性可以  使用该元素显式强制一个或多个Bean在Bean之前初始化  被初始化。 以下示例使用 `depends-on`属性表示  对单个bean的依赖： 

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表达对多个Bean的依赖关系，请提供一个Bean名称列表作为的值  该 `depends-on`属性（逗号，空格和分号是有效的  定界符）： 

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

|      | 该 `depends-on`属性既可以指定初始化时间相关性，也可以指定  仅在 的情况下 [单例 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton)豆 ，相应的  销毁时间依赖性。 定义 从属bean `depends-on`关系的  具有给定bean的对象在销毁给定bean本身之前先被销毁。  这样， `depends-on`还可以控制关机顺序。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.4.4. Lazy-initialized Beans懒加载Bean

默认情况下， `ApplicationContext`实现会急于创建和配置所有  [单例 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton)bean作为初始化处理的一部分。 通常，这种预实例化是可取的，因为配置或环境的错误错误是立即发现的，而不是花费数小时甚至几天后。 当这种行为不是所希望的时，您可以防止  通过将Bean定义标记为来预实例化Singleton Bean  延迟初始化。 延迟初始化的bean告诉IoC容器创建一个bean  实例在第一次被请求时而不是在启动时被创建。 

在XML中，这种行为是由控制 `lazy-init`上的属性 `<bean/>` 元素，如以下示例所示： 

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

当之前的配置被占用时 `ApplicationContext`， `lazy`bean  开始时并没有急切地预先实例化 `ApplicationContext`，  而 `not.lazy`Bean则早已被预先实例化。 

但是，当延迟初始化的bean是单例bean的依赖项时，  未延迟初始化，则在处 `ApplicationContext`创建延迟初始化的bean  启动，因为它必须满足单例的依赖关系。 延迟初始化的bean  被注入到未延迟初始化的其他地方的单例bean中。 

您还可以使用以下命令在容器级别控制延迟初始化  `default-lazy-init`上的属性 `<beans/>`元素 ，如以下示例所示： 

```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

#### 1.4.5. Autowiring Collaborators  自动装配协作器 

Spring容器可以自动装配协作bean之间的关系。 您可以  让Spring通过以下方式自动为您的bean解决协作者（其他bean）：  检查物品的内容 `ApplicationContext`。 自动装配具有以下特点  优点： 

- 自动装配可以大大减少指定属性或构造函数的需要。 （在这方面其他机制，例如bean模板  [本章其他地方讨论 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-child-bean-definitions)的内容也很有价值。） 
- 随着对象的发展，自动装配可以更新配置。 例如，如果您需要向类添加依赖项，该依赖项可以自动满足而无需您需要修改配置。 因此，自动接线在开发过程中可能特别有用，代码库变得更加稳定。 

使用基于XML的配置元数据时（请参阅 [Dependency Injection ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators)），您可以为  `<bean/>`元件使用bean定义指定自动装配模式 `autowire`属性。 自动装配功能具有四种模式。 您指定自动装配每个bean，因此可以选择自动装配哪个。 下表介绍了四种自动装配模式： 

| 模式          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| `no`          | （默认）无自动装配。 Bean引用必须由 定义 `ref`元素 。 改变中  对于较大的部署，建议不要使用默认设置，因为  协作者明确提供了更大的控制权和清晰度。 在某种程度上，它  记录系统的结构。 |
| `byName`      | 按属性名称自动布线。 Spring寻找与Bean具有相同名称的bean  需要自动接线的属性。 例如，如果将bean定义设置为  按名称自动装配，它包含一个 `master`属性（即，它具有一个  `setMaster(..)`方法），Spring寻找一个名为的bean定义 `master`并使用  它来设置属性。 |
| `byType`      | 如果属性类型中恰好存在一个bean，则使该属性自动连接  容器。 如果存在多个，则会引发致命异常，这表明  您可能不会 使用 `byType`对该bean 自动装配。 如果没有匹配项  Bean，什么也没发生（未设置属性）。 |
| `constructor` | 类似于 `byType`但适用于构造函数参数。 如果不完全正确  容器中构造函数参数类型的一个bean会引发致命错误。 |

使用 `byType`或 `constructor`自动装配模式，您可以连接数组和输入集合。 在这种情况下，容器中的所有自动接线候选物  提供符合预期类型的匹配项以满足相关性。 您可以自动关联强类型 `Map`实例如果预期键类型为`String`。 自动关联 `Map` 实例的值包括所有与预期类型匹配的bean实例，以及  `Map`实例的键包含相应的Bean名称。 

##### Limitations and Disadvantages of Autowiring 自动关联的局限性和缺点 

当在项目中一致使用自动装配时，自动装配效果最佳。 如果是自动装配  通常不使用，可能会使开发人员仅使用它来连接一个或多个  两个bean定义。 

考虑自动装配的局限性和缺点： 

- 显式依赖项 `property`和 `constructor-arg`设置始终被覆盖  自动接线。 您无法自动连线简单的属性，例如基元，  `Strings`和 `Classes`（以及此类简单属性的数组）。 这个限制是  设计。 
- 自动装配不如显式接线精确。 尽管如上表所示，  Spring小心避免在可能出现意料之外的歧义的情况下进行猜测  结果。 Spring管理的对象之间的关系不再  明确记录。 
- 关联信息可能不适用于可能从中生成文档的工具  一个Spring容器。 
- 容器中的多个bean定义可能与  要自动装配的setter方法或构造函数参数。 对于数组，集合或  `Map`实例，这不一定是问题。 但是，对于依赖项  期望单个值，这种歧义不会被任意解决。 如果没有唯一的bean定义可用，将引发异常。 

在后一种情况下，您有几种选择： 

- 放弃自动关联，转而使用明确的关联。 
- 通过设置其 避免自动装配bean定义 `autowire-candidate`属性  到 `false`，在如所描述的 [下一节 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire-candidate)。 
- 通过设置单个Bean定义作为主要候选者  `primary`其 属性 `<bean/>`元素的 `true`。 
- 通过基于注释的配置实施更细粒度的控件，  如 [基于注释的容器配置中所述 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config)。 

##### Excluding a Bean from Autowiring 从自动装配中排除Bean 

在每个bean的基础上，您可以从自动装配中排除一个bean。 在Spring的XML格式设置中， 设置 `<bean/>`元素的`autowire-candidate`属性 为 `false`。 容器使特定的bean定义对自动装配基础结构不可用  （包括注释样式配置，例如 [`@Autowired`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation)）。 

|      | 该 `autowire-candidate`属性旨在仅影响基于类型的自动装配。  它不会影响按名称显示的显式引用，即使指定的bean未标记为自动装配候选。 结果，如果名称匹配，则按名称自动注入Bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以基于与Bean名称的模式匹配来限制自动装配候选。 顶层 `<beans/>`元素在其 `default-autowire-candidates`属性内部接受一个或多个模式 。 例如，限制自动装配候选状态对于任何名称以`Repository`结尾的bean ，需要提供值 `*Repository`。 提供多种模式，并在以逗号分隔的列表中定义它们。 显式值  `true`或者 `false`对于bean定义的 `autowire-candidate`属性总是优先。 对于此类bean，模式匹配规则不适用。 

这些技术对您不想将其注入其他bean很有用  通过自动装配咖啡豆。 这并不意味着排除的bean本身不能由  使用自动装配。 相反，bean本身不是自动装配其他bean的候选对象。 

#### 1.4.6. Method Injection 方法注入

在大多数应用场景中，容器中的大多数bean是  [单一的（singletons） ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton)。 当单例bean需要与另一个单例bean进行协作，或者非单例bean需要与另一个非单一bean进行协作，通常通过定义一个来处理依赖关系  bean作为另一个的属性。 当bean的生命周期是  不同。 假设单例bean A需要使用非单例（原型）bean B，  也许在A的每个方法调用上。容器仅创建A的单例bean  一次，**因此只有一次机会来设置属性。容器不能在每次需要时，为Bean A提供一个新的Bean B实例**。 

解决方案是放弃某些控制反转。 你可以 [使bean A 了解容器make bean A aware of the container ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware)通过实现 `ApplicationContextAware`接口来 ， **每次bean A需要bean B实例时**，通过 [制作 `getBean("B")`到容器调用 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-client)获取（通常是新的）。 下面的例子显示了这种方法： 

```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

前面的内容是不理想的，因为业务代码知道并耦合到  Spring框架。 方法注入，Spring IoC的高级功能  容器，让您可以轻松处理此用例。 

> 您可以在以下文章中了解有关方法注入动机的更多信息  [此博客条目 ](https://spring.io/blog/2004/08/06/method-injection/)。 

##### Lookup Method Injection 查找方法注入 

查找方法注入是容器覆盖容器管理Bean的方法并在中返回另一个容器中命名Bean的查找结果的能力。 查找通常涉及原型bean，如在所描述的场景中一样  在 [上一节中 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-method-injection)。 spring框架通过使用CGLIB库中的字节码生成来实现此方法注入  动态生成覆盖该方法的子类。 

>- 为了使此动态子类起作用，Spring bean容器的类子类也不能为 `final`，并且要覆盖的方法也不能为 `final`。   
>- 对具有`abstract`方法的类进行单元测试，需要您自己对该类进行子类化，并提供该`abstract`方法的桩实现  。   
>- 组件扫描也需要具体方法，这需要具体类来检索。   
>- 另一个关键限制是查找方法不能与工厂方法一起使用，并且特别是不使用 `@Bean`配置类中的方法，因为在这种情况下，容器不负责创建实例，因此无法创建运行时生成的子类。 

对于先前代码段中的 `CommandManager`类，  Spring容器会动态覆盖 `createCommand()` 方法。 该 `CommandManager`班没有任何Spring的依赖，如重做的示例显示： 

```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

在包含要注入的方法的客户端类中（ `CommandManager`在此  情况），要注入的方法需要以下形式的签名： 

```xml
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法为 `abstract`，则动态生成的子类将实现该方法。  否则，动态生成的子类将覆盖在中定义的具体方法  原来的课。 考虑以下示例： 

```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

标识为bean的 `commandManager`调用其自己的 `createCommand()`方法  每当需要新的 实例时 `myCommand`bean 。 您必须谨慎部署  将其 `myCommand`如果实际需要的话， 作为原型。 如果是  一 [单 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton)，对同一个实例 `myCommand` 每次都返回bean。 

或者，在基于注释的组件模型中，您可以声明查找  方法通过 `@Lookup`注解，如以下示例所示： 

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

或者，更习惯地说，您可以依靠目标Bean来解决  查找方法的声明的返回类型： 

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```

请注意，您通常应使用具体的方法声明此类带注释的查找方法  存根实现，以便它们与Spring的组件兼容  扫描规则，默认情况下将忽略抽象类。 此限制不  适用于显式注册或显式导入的Bean类。 

> 访问范围不同的目标bean的另一种方法是 `ObjectFactory`/  `Provider`注射点。 请参阅 [作用域Bean作为依赖项 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection)。   
>
> 您还会发现 `ServiceLocatorFactoryBean`（在`org.springframework.beans.factory.config`包）可能是有用的。 

##### Arbitrary Method Replacement 任意方法替换 

与查找方法注入相比，方法注入的一种不太有用的形式是用另一个方法实现替换托管bean中的任意方法。 您可以安全地跳过本节的其余部分，直到您真正需要此功能为止。 

通过基于XML的配置元数据，您可以使用 `replaced-method`元素来  对于已部署的Bean，用另一个方法替换现有的方法实现。 考虑  下面的类，具有一个 `computeValue`我们要覆盖的方法： 

```java
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```

实现 `org.springframework.beans.factory.support.MethodReplacer` 接口提供了新的方法定义，如以下示例所示： 

```java
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```

用于部署原始类并指定方法重写的Bean定义将  类似于以下示例： 

```xml
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

您可以 `<arg-type/>`在 `<replaced-method/>` 元素，指示要覆盖的方法的方法签名。 签名  仅当方法重载且有多个变体时，才需要参数  存在于类中。 为了方便起见，参数的类型字符串可以是  完全限定类型名称的子字符串。 例如，以下所有匹配项  `java.lang.String`： 

```java
java.lang.String
String
Str
```

因为参数的数量通常足以区分每个可能的参数  选择，此快捷方式可以让您只键入  与参数类型匹配的最短字符串。 

### 1.5. Bean Scopes Bean范围

创建bean定义时，将创建用于创建实际实例的bean定义内容所定义的配方。 Bean定义是一个配方的想法 很重要，因为它意味着像创建类一样，您可以从一个单个配方创建许多对象的实例。

您不仅可以控制 插入到从特定bean定义创建的对象中的 要依赖的各种依赖项和配置值，还可以从特定bean定义创建的对象的范围。这种方法强大而灵活，因为您可以选择通过配置创建的对象的范围，而不必在Java类级别上烘焙（原味真的就是这个单词，好家伙，我认为是指定的意思）对象的范围。 可以将Bean定义为部署在多个范围之一中。  Spring框架支持六个范围，其中四个仅在以下情况下可用  您使用网络感知 `ApplicationContext`。 您也可以创建  [自定义范围。 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-custom)

下表描述了受支持的范围： 

| 范围                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton) | （**默认值**）将每个Spring IoC的单个bean定义范围限定为单个对象实例容器。 |
| [prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-prototype) | 将单个bean定义的作用域限定为**任意数量**的对象实例。         |
| [request](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-request) | 将单个bean定义的范围限定为**单个HTTP请求**的生命周期。 那是，  **每个HTTP请求都有自己的bean实例**，它是在单个bean的后面创建的  定义。 仅在可感知网络的Spring上下文中有效 `ApplicationContext`。 |
| [session](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-session) | 将单个bean定义的范围限定为HTTP的生命周期 `Session`。 仅在Web感知Spring的上下文 `ApplicationContext`。 |
| [application](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-application) | 将单个bean定义的作用域限定为的生命周期 `ServletContext`。 仅在  Web感知Spring的上下文 `ApplicationContext`。 |
| [websocket](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-websocket-scope) | 将单个bean定义的作用域限定为的生命周期 `WebSocket`。 仅在  Web感知Spring的上下文 `ApplicationContext`。 |

> 从Spring 3.0开始，线程作用域可用，但默认情况下未注册。 对于有关更多信息，请参见文档。  [`SimpleThreadScope`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/context/support/SimpleThreadScope.html)。 
>
> 有关如何注册此或任何其他自定义范围的说明，请参见  [使用自定义范围 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-custom-using)。  

#### 1.5.1. The Singleton Scope 单例范围

仅管理一个singleton bean的一个共享实例，并且所有对bean的请求  一个或多个与该bean定义相匹配的ID会导致该特定bean  Spring容器返回的实例。 

换句话说，当您定义bean定义并将其范围限定为  单例，Spring IoC容器仅创建由该bean定义定义的对象的一个实例。为了该命名bean的所有后续请求和引用返回缓存的对象，该单个实例存储在这样的单例bean缓存中。 下图显示了单例作用域的工作方式： 

![singleton](..\Core\.img\singleton.png)

Spring的singleton bean的概念不同于Gang of Four (GoF)模式书。 GoF单例会硬编码一个对象，这样每个对象只能创建一个特定类的一个实例ClassLoader。 最好将Spring单例的范围描述为每个容器和每个bean。 这意味着，如果您为一个特定类在一个Spring容器定义了一个bean，Spring容器创建一个且只有一个实例该bean定义所定义的类的名称。 单例范围是spring的默认scope。 要将bean定义为XML中的单例，可以定义bean，如  下面的例子： 

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

#### 1.5.2. The Prototype Scope 原型范围 

Bean部署的非单一的prototype cope将导致每次对该特定bean发出请求时导致创建新的该bean实例。 即bean被注入到另一个bean中，或者您通过 `getBean()`方法调用容器。 通常，您应该对所有状态性的Bean使用原型作用域（即表示一个动态的状态），并且对无状态bean使用单例作用域。 

下图说明了Spring原型范围： 

![prototype](..\Core\.img\prototype.png)

一个数据访问对象  （DAO）通常不配置为原型，因为典型的DAO不能容纳任何对话状态。 对我们来说，重用核心单例图更轻松。） 

以下示例将bean定义为XML原型： 

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

与其他作用域相反，Spring不能管理一个prototype bean的完整生命周期。 容器实例化，配置并以其他方式组装 原型对象并将其交给客户端，没有该prototype实例的进一步记录。 因此，无论是什么scope，尽管所有的对象都会调用生命周期中初始化回调方法，如果是原型，则配置的生命周期中，销毁的回调方法将不会被调用。 客户端代码必须清理prototype 作用域对象并释放prototype bean拥有的昂贵资源。 要得到被 Spring容器释放的由原型作用域Bean持有的资源，请尝试使用 定制 [bean后处理器 bean post-processor](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-bpp)，其中包含需要清理的bean的引用。

从某些方面来说，Spring容器对于原型范围的bean的作用是  替代Java `new`运算符。 此后的所有生命周期管理必须 由客户处理。（有关spring容器 bean的生命周期的详细信息，请参阅 [生命周期回调 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle)。） 

#### 1.5.3. Singleton Beans with Prototype-bean Dependencies 具有原型Bean依赖关系的Singleton Bean 

当您使用依赖于原型bean的单例作用域bean时，请注意  依赖关系在实例化时解决。 因此，如果您依赖注入  将原型范围的bean转换为单例范围的bean，实例化一个新的原型bean  然后将依赖项注入到Singleton bean中。 原型实例是唯一的  提供给单例范围的bean的实例。 

但是，假设您希望单例作用域的bean获取一个新的实例。  原型范围的bean在运行时反复出现。 您不能依赖注入  将原型范围内的bean放入您的singleton bean中，因为该注入仅发生  一次，当Spring容器实例化单例bean并解析时  并注入其依赖项。 如果您需要在以下位置获取原型Bean的新实例：  运行时不止一次，请参见 [方法注入 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-method-injection)

#### 1.5.4. Request, Session, Application, and WebSocket Scopes 请求，会话，应用程序和WebSocket范围 

在 `request`， `session`， `application`，和 `websocket`范围仅可用web-aware（即web app）的Spring `ApplicationContext`实现（例如  `XmlWebApplicationContext`）。 如果您将这些作用域用于常规的Spring IoC容器，  如 `ClassPathXmlApplicationContext`，一个关于一个未知的bean范围的`IllegalStateException` 被抛出。 

##### Initial Web Configuration 初始Web配置 

为了支持范围界定在 `request`， `session`， `application`，和  `websocket`级别（网络范围的Bean）的bean，一些较小的初始配置是  在定义bean之前是必需的。 （对于标准范围不需要此初始设置： `singleton`和 `prototype`。） 

如何完成此初始设置取决于您的特定Servlet环境。 

如果您实际上在Spring Web MVC中访问作用域Bean，则在以下请求中  由Spring处理 `DispatcherServlet`，无需特殊设置。  `DispatcherServlet`已经暴露了所有相关状态。 

如果您使用的是Servlet 2.5 Web容器，则在Spring的外部处理请求  `DispatcherServlet`（例如，使用JSF或Struts时），您需要注册  `org.springframework.web.context.request.RequestContextListener` `ServletRequestListener`。  对于Servlet 3.0+，可以使用 `WebApplicationInitializer` 接口。 或者，或者对于较旧的容器，将以下声明添加到  您的Web应用程序的 `web.xml`文件： 

```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

另外，如果您的监听器设置存在问题，请考虑使用Spring的  `RequestContextFilter`。 过滤器映射取决于周围的网络  应用程序配置，因此您必须适当地进行更改。 以下清单  显示了Web应用程序的过滤器部分： 

```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

`DispatcherServlet`， `RequestContextListener`和 `RequestContextFilter`所做的事情是一致的，即将HTTP请求对象绑定到 `Thread`正在服务的  该请求。 这使得请求和会话范围的bean在请求链中进一步可用  。 

##### Request scope 请求范围 

考虑以下XML配置来定义bean： 

```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

Spring容器使用来创建 的新实例。 `LoginAction`Bean  `loginAction`每个HTTP请求的bean定义。 那就是  `loginAction`bean的作用域在HTTP请求级别。 您可以更改内部  根据需要创建的实例的状态，因为其他实例  从相同的 创建的 `loginAction`bean定义 状态看不到这些变化。  它们特定于单个请求。 请求完成处理后，  范围为请求的bean被丢弃。 

使用注释驱动的组件或Java配置时， `@RequestScope`注释  可用于将组件分配给 `request`scope。 以下示例显示了  这样做： 

```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

##### Session Scope 会话范围 

考虑以下XML配置来定义bean： 

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

Spring容器使用来创建 的新实例。 `UserPreferences`Bean  `userPreferences`单个HTTP生存期的bean定义 `Session`。 其他  换句话说， `userPreferences`bean的作用域实际上是HTTP `Session`级别。 如  使用请求范围的Bean，您可以更改实例的内部状态，即  在知道其他HTTP `Session`实例也是  使用从同一 创建的实例 `userPreferences`bean定义 看不到这些  状态更改，因为它们特定于单个HTTP `Session`。 当 HTTP `Session`最终被丢弃的时候 ，该bean的作用域仅限于该特定HTTP  `Session`也被丢弃。 

使用注释驱动的组件或Java配置时，可以使用  `@SessionScope`注释以将组件分配给合并 `session`范围。 

```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

##### Application Scope 应用范围 

考虑以下XML配置来定义bean： 

```xml
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

Spring容器使用来创建 的新实例。 `AppPreferences`Bean  `appPreferences`整个Web应用程序一次定义bean。 那就是  `appPreferences`bean的作用域 `ServletContext`级别和常规存储  `ServletContext`属性。 这有点类似于Spring单例bean，但是  在两个重要方面有所不同：每个单独一个 `ServletContext`，而不是每个Spring一个  “ ApplicationContext”（在任何给定的Web应用程序中可能都有多个），  它实际上是公开的，因此作为 可见 `ServletContext`属性 。 

使用注释驱动的组件或Java配置时，可以使用  `@ApplicationScope`注解以将组件分配给合并 `application`范围。 以下示例显示了如何执行此操作： 

```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

##### Scoped Beans as Dependencies

Spring IoC容器不仅管理对象（bean）的实例化，  以及合作者（或依赖）的连线。 如果要注入（例如）将HTTP请求范围的Bean放入范围更长的另一个Bean中，您可以选择注入AOP代理代理替代作用域bean。 也就是说，您需要注入一个代理对象，该对象公开与范围对象相同的公共接口，但是可以  还可以从相关范围中检索实际目标对象（例如HTTP请求）  然后委托方法调用真实对象。 

>您也可以 `<aop:scoped-proxy/>`在范围为 `singleton`，然后通过参考进行可序列化的中间代理  因此可以在反序列化时重新获得目标单例bean。   
>
>在声明 `<aop:scoped-proxy/>`一个范围bean时 `prototype`，每种方法对共享代理的调用会导致创建新的目标实例，请求然后被转发。 
>
>同样，在生命周期安全中作用域代理不是访问范围较短的作用域中的bean的唯一方法。 您也可以声明您的注入点（即构造函数，setter参数或自动装配字段） `ObjectFactory<MyTargetBean>`，  允许 `getObject()`每次调用以按需检索当前实例  需要的时间-无需保留实例或单独存储它。
>
>作为扩展变体，您可以声明 `ObjectProvider<MyTargetBean>`，来传递其他几种访问方式，包括 `getIfAvailable`和 `getIfUnique`。
>
>称为JSR-330的变体， `Provider`并与 `Provider<MyTargetBean>` 声明和 的相应 `get()`每次检索尝试 调用。  有关 请参见 [此处 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-standard-annotations)JSR-330总体的更多详细信息， 。

以下示例中的配置只有一行，但对于  了解其背后的“why”以及“how”： 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> 
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

>定义代理的行。 

要创建这样的代理，请将子 `<aop:scoped-proxy/>`元素插入到作用域中  bean定义（请参阅 [选择要创建的代理类型 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection-proxies)和  [基于XML模式的配置 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#xsd-schemas)）。  为什么在作用域bean定义 `request`， `session`和自定义范围  水平需要 `<aop:scoped-proxy/>`元素吗？  考虑以下单例bean定义并将其与  您需要为上述范围定义的内容（请注意以下内容  `userPreferences`目前的bean定义不完整）： 

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

在前面的示例中，单例bean（ `userManager`）注入了一个引用  到HTTP `Session`范围的bean（ `userPreferences`）。 这里的重点是  `userManager`bean是一个单例：每次实例化一次  容器及其依赖项（在这种情况下，只有一个是 `userPreferences`bean）是  也只注射一次。 这意味着 `userManager`Bean仅在  完全相同的 `userPreferences`对象（即最初注入该对象的对象）。 

将寿命较短的作用域Bean注入到容器中时，这不是您想要的行为  寿命更长的作用域bean（例如，注入HTTP `Session`范围的协作  bean作为对Singleton bean的依赖）。 相反，您需要一个 `userManager` 对象，并且在HTTP的生存期内 `Session`，您需要一个 `userPreferences`对象  这是特定于HTTP的 `Session`。 因此，容器创建了一个对象  公开与 完全相同的公共接口 `UserPreferences`该类 （理想情况下，  作为 对象 `UserPreferences`实例的 ），可以获取真实的  `UserPreferences`从范围机制（HTTP请求对象， `Session`等  向前）。 容器将此代理对象注入到 `userManager`bean中，即  不知道此 `UserPreferences`引用是代理。 在此示例中，当  `UserManager`实例在注入依赖项上调用方法 `UserPreferences` 对象，实际上是在代理上调用方法。 代理然后获取真实  `UserPreferences`来自（在这种情况下）HTTP的对象， `Session`并委托  方法调用到检索到的真实 `UserPreferences`对象上。 

因此，注入时需要以下（正确和完整）的配置  `request-`和 `session-scoped`bean成为协作对象，如以下示例  显示： 

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

###### Choosing the Type of Proxy to Create 选择要创建的代理类型 

默认情况下，当Spring容器为标记为的bean创建代理时  的 `<aop:scoped-proxy/>`元件，则创建一个基于CGLIB级代理。 

|      | CGLIB代理仅拦截公共方法调用！ 不要调用非公共方法  在这样的代理上。 它们没有被委派给实际的作用域目标对象。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

另外，您可以配置Spring容器以创建标准JDK  通过指定 此类作用域bean的基于接口的代理 `false`的值，  的 `proxy-target-class`属性 `<aop:scoped-proxy/>`元素 。 使用JDK  基于接口的代理意味着您不需要其他库  应用程序类路径来影响这种代理。 但是，这也意味着  作用域Bean必须实现至少一个接口，并且所有协作者  注入范围内的bean的对象必须通过其之一引用该bean  接口。 以下示例显示了基于接口的代理： 

```xml
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.stuff.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

有关选择基于类或基于接口的代理的更多详细信息，  请参阅 [代理机制 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-proxying)。 

#### 1.5.5. Custom Scopes 自定义范围 

Bean作用域机制是可扩展的。 您可以定义自己的  范围，甚至重新定义现有范围，尽管后者被认为是不好的做法  并且您不能覆盖内置 `singleton`和 `prototype`范围。 

##### 创建自定义范围 

要将您的自定义范围集成到Spring容器中，您需要实现  `org.springframework.beans.factory.config.Scope`界面，在此进行描述  部分。 有关如何实现自己的范围的想法，请参见 `Scope` Spring框架本身以及  [`Scope`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/beans/factory/config/Scope.html)javadoc，  其中详细说明了您需要实现的方法。 

该 `Scope`接口有四种方法可以从作用域中获取对象，将其从中删除  范围，让它们被摧毁。 

会话范围的实现，例如，返回会话范围的Bean（如果有）  不存在，则在将其绑定到之后，该方法将返回该bean的新实例  会议以供将来参考）。 以下方法从  基本范围： 

```java
Object get(String name, ObjectFactory<?> objectFactory)
```

会话范围的实现，例如，从  基础会话。 应该返回该对象，但是 `null`如果  找不到具有指定名称的对象。 以下方法从中删除对象  基本范围： 

```java
Object remove(String name)
```

下面的方法注册一个回调，作用域在作用域范围内应调用该回调  销毁或作用域中的指定对象销毁时： 

```java
void registerDestructionCallback(String name, Runnable destructionCallback)
```

见 [javadoc ](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/beans/factory/config/Scope.html#registerDestructionCallback) 或Spring范围实现，以获取有关销毁回调的更多信息。 

以下方法获取基础范围的会话标识符： 

```java
String getConversationId()
```

每个范围的标识符都不相同。 对于会话范围的实施，  标识符可以是会话标识符。 

##### 使用自定义范围 

编写并测试一个或多个自定义 `Scope`实现后，您需要  Spring容器会意识到您的新作用域。 以下方法是中央  注册新方法 `Scope`在Spring容器中 ： 

```java
void registerScope(String scopeName, Scope scope);
```

该方法在 上声明 `ConfigurableBeanFactory`接口 ，可以使用  通过 的 `BeanFactory`大多数混凝土 属性 `ApplicationContext` Spring附带的实现。 

该 的第一个参数 `registerScope(..)`方法 是与  一个范围。 Spring容器本身中的此类名称示例包括 `singleton`和  `prototype`。 该 的第二个参数 `registerScope(..)`方法 是实际实例  的自定义 `Scope`您希望注册和使用 实现。 

假设您编写了自定义 `Scope`实现，然后如图所示进行注册  在下一个示例中。 

|      | 下一个示例使用 `SimpleThreadScope`Spring附带的，但不包含  默认情况下注册。 说明将与您的自定义相同 `Scope` 实现。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

然后，您可以创建符合自定义范围规则的bean定义  `Scope`， 如下： 

```xml
<bean id="..." class="..." scope="thread">
```

通过自定义 `Scope`实现，您不仅可以进行程序化注册  范围。 您也可以 `Scope`使用  `CustomScopeConfigurer`类，如以下示例所示： 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="thing2" class="x.y.Thing2" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="thing1" class="x.y.Thing1">
        <property name="thing2" ref="thing2"/>
    </bean>

</beans>
```

|      | 当您放置 `<aop:scoped-proxy/>`在 `FactoryBean`实施中时，它就是工厂  范围本身的Bean本身，而不是从返回的对象 `getObject()`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.6. Customizing the Nature of a Bean  自定义bean的性质 

pring框架提供了许多接口，您可以使用这些接口来自定义一个bean性质。 本节将它们分组如下： 

- [生命周期回调 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle)
- [`ApplicationContextAware`和 `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware)
- [其他 `Aware`介面 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aware-list)

#### 1.6.1. Lifecycle Callbacks 生命周期回调 

要与容器对bean生命周期的管理进行交互，可以实现  Spring `InitializingBean`和 `DisposableBean`接口。 容器调用  `afterPropertiesSet()`对于前者和 `destroy()`后者让bean在初始化和销毁bean时执行某些操作。 

> JSR-250 `@PostConstruct`和 `@PreDestroy`通常认为 注释是最好的  在现代Spring应用程序中接收生命周期回调的实践。 使用这些  注释意味着您的bean没有耦合到特定于Spring的接口。  有关详细信息，请参见 [使用 `@PostConstruct`和 `@PreDestroy`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations)。
>
> 如果您不想使用JSR-250批注，但仍想删除耦合，考虑 `init-method`和 `destroy-method`bean定义元数据。 

在内部，Spring框架使用 `BeanPostProcessor`实现来处理任何  可以找到并调用适当方法的回调接口。 如果需要定制  功能或其他生命周期行为Spring默认情况下不提供，您可以  实现 `BeanPostProcessor`自己。 有关更多信息，请参见  [集装箱延伸点 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension)。 

除了初始化和销毁回调外，Spring管理的对象还可能  还实现 `Lifecycle`接口，以便那些对象可以参与  由容器自身的生命周期驱动的启动和关闭过程。 

本节介绍了**生命周期回调接口。**

##### Initialization Callbacks 初始化回调 

该 `org.springframework.beans.factory.InitializingBean`接口允许一个bean  在容器设置完所有必要的属性后，执行初始化bean工作。 该 `InitializingBean`接口指定一个方法： 

```java
void afterPropertiesSet() throws Exception;
```

我们建议您不要使用该 `InitializingBean`接口，因为它不必要地将代码耦合到Spring。 另外，我们建议使用  在 [`@PostConstruct`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations)注解或指定POJO初始化方法。 对于基于XML的配置元数据，您可以使用该 `init-method`属性来指定具有空值的方法的名称  无参数签名。 通过Java配置，您可以使用以下 `initMethod`属性  `@Bean`。 请参阅 [接收生命周期回调 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-lifecycle-callbacks)。 考虑以下示例： 

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

前面的示例与下面的示例几乎具有完全相同的效果  （由两个清单组成）： 

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

但是，前面两个示例中的第一个示例并未将代码耦合到Spring。 

##### Destruction Callbacks 销毁回调 

 

实施 `org.springframework.beans.factory.DisposableBean`界面可以使  当包含它的容器被销毁时，bean将获得一个回调。 的  `DisposableBean`接口指定单个方法： 

```java
void destroy() throws Exception;
```

我们建议您不要使用 `DisposableBean`回调接口，因为它  不必要地将代码耦合到Spring。 另外，我们建议使用  在 [`@PreDestroy`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations)注释或  指定bean定义支持的通用方法。 基于XML  配置元数据，您可以使用上的 `destroy-method`属性 `<bean/>`。  通过Java配置，您可以使用的 `destroyMethod`属性 `@Bean`。 看到  [接收生命周期回调 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-lifecycle-callbacks)。 考虑以下定义： 

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

前面的定义与下面的定义几乎具有完全相同的效果： 

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

但是，前面两个定义中的第一个没有将代码耦合到Spring。 

>您可以 的 分配 `destroy-method`为 属性 `<bean>`元素 特殊的  `(inferred)`值，指示Spring自动检测公共 `close`或  `shutdown`特定bean类上的方法。 （任何实现  `java.lang.AutoCloseable`或 `java.io.Closeable`因此匹配。）您还可以设置  这个特殊 `(inferred)`的价值 `default-destroy-method`一个属性  `<beans>`将此行为应用于整套bean的元素（请参见  [默认初始化和销毁方法 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-default-init-destroy-methods)）。 请注意，这是  Java配置的默认行为。

##### Default Initialization and Destroy Methods 默认初始化和销毁方法 

当您编写不使用的初始化和销毁方法回调时，  特定于Spring的 `InitializingBean`和 `DisposableBean`回调接口  通常的名字，如写入方法 `init()`， `initialize()`， `dispose()`，等。 理想情况下，此类生命周期回调方法的名称应在项目，以便所有开发人员使用相同的方法名称并确保一致性。 

您可以将Spring容器配置为“寻找”命名的初始化和销毁在每个bean上的回调方法。这意味着您作为一个应用程序  开发人员，可以编写您的应用程序类并使用称为  `init()`，而不必为 配置一个 `init-method="init"`每个bean 属性  定义。 在创建Bean时，Spring IoC容器会调用该方法（在  按照标准的生命周期回调合同 [先前所描述 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle)）。 此功能还对以下内容强制执行一致的命名约定初始化和销毁方法回调。 

假设您的初始化回调方法已命名 `init()`并且销毁  回调方法被命名为 `destroy()`。 然后，您的class类似于  下面的例子： 

```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

然后，您可以在类似于以下内容的Bean中使用该类： 

```xml
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

存在 `default-init-method`顶级 上属性 `<beans/>`元素  属性使Spring IoC容器识别 调用的方法 `init`在bean上  类作为初始化方法的回调。 当创建并组装一个bean时，如果  bean类具有这样的方法，它在适当的时间被调用。 

您可以通过以下方式类似地（在XML中）配置destroy方法回调：  `default-destroy-method`顶级 上的属性 `<beans/>`元素 。 

现有的Bean类已经具有以方差命名的回调方法的地方  按照约定，您可以通过指定（在XML中）  通过使用 方法名称 `init-method`和 `destroy-method`属性的 `<bean/>` 本身。 

Spring容器保证调用已配置的初始化回调  在为bean提供所有依赖项之后立即执行。 因此，初始化  在原始bean引用上调用callback，这意味着AOP拦截器等  尚未应用到bean。 首先完全创建了目标bean，然后  然后应用带有其拦截器链的AOP代理（例如）。 如果目标  bean和代理是分别定义的，您的代码甚至可以与原始代码进行交互  目标Bean，绕过代理。 因此，应用  拦截器 `init`方法的 ，因为这样做会耦合  将bean定位到其代理或拦截器，并在代码时留下奇怪的语义  直接与原始目标Bean交互。 

##### Combining Lifecycle Mechanisms 组合生命周期机制 

从Spring 2.5开始，您可以使用三个选项来控制Bean生命周期行为： 

- 在 [`InitializingBean`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean)与  [`DisposableBean`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean)回调接口 
- 定制 `init()`和 `destroy()`方法 
- [`@PostConstruct`与 `@PreDestroy`  注解 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations)。 您可以结合使用这些机制来控制给定的bean。 

|      | 如果为bean配置了多种生命周期机制，并且每种机制都是  配置了不同的方法名称，则每个配置的方法都在  此注释后列出的订单。 但是，如果配置了相同的方法名称，例如，  `init()`一种初始化方法—对于这些生命周期机制中的一种以上，  该方法只运行一次，如  [前一节 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-default-init-destroy-methods)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

为同一个bean配置了多种生命周期机制，但机制不同  初始化方法如下： 

1. 用注释的方法 `@PostConstruct`
2. `afterPropertiesSet()`由 定义 `InitializingBean`回调接口 
3. 定制配置的 `init()`方法 

销毁方法的调用顺序相同： 

1. 用注释的方法 `@PreDestroy`
2. `destroy()`由 定义 `DisposableBean`回调接口 
3. 定制配置的 `destroy()`方法 

##### Startup and Shutdown Callbacks 启动和关闭回调 

该 `Lifecycle`接口定义有自己的生命周期要求的任何对象的基本方法  （例如启动和停止某些后台进程）： 

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

任何Spring管理的对象都可以实现该 `Lifecycle`接口。 然后，当  `ApplicationContext`本身接收启动和停止信号（例如，用于停止/重新启动）  **必须是容器调用该方法**），它将这些调用级联到所有 `Lifecycle`实现  在该上下文中定义。 它通过委派给 `LifecycleProcessor`显示的  在以下清单中： 

```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

请注意， `LifecycleProcessor`本身是的扩展 `Lifecycle` 接口。 它还添加了其他两种方法来对正在刷新的上下文做出反应  并关闭。 

>请注意，常规 普通 `org.springframework.context.Lifecycle`接口是 接口  明确的启动和停止通知的合同，并不意味着在上下文中自动启动  刷新时间。 为了对特定Bean的自动启动进行精细控制（包括启动阶段），  考虑 实施 `org.springframework.context.SmartLifecycle`改为 。 
>
>另外，请注意，不能保证会在销毁之前发出停止通知。  在常规关闭时，所有 `Lifecycle`Bean都首先收到停止通知，然后才一般销毁回调将被传播。 但是，在上下文的生命周期或停止刷新尝试时，仅调用destroy方法。 

启动和关闭调用的顺序可能很重要。 如果“依赖”  任何两个对象之间都存在关系，从属方在其  依赖关系，并且在依赖之前停止。 但是，有时，直接依赖关系是未知的。 您可能只知道某种类型的对象应该开始  在其他类型的对象之前。 在这些情况下， `SmartLifecycle`接口定义  另一个选项，即 的 `getPhase()`在其超级接口上定义 方法，  `Phased`。 以下清单显示了 的定义 `Phased`接口 ： 

```java
public interface Phased {

    int getPhase();
}
```

以下清单显示了 的定义 `SmartLifecycle`接口 ： 

```java
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

启动时，相位最低的对象首先启动。 停止时，  遵循相反的顺序。 因此，实现 `SmartLifecycle`和  其 `getPhase()`方法返回 `Integer.MIN_VALUE`将是第一个开始的方法  最后一站。 在频谱的另一端，相位值为  `Integer.MAX_VALUE`将指示该对象应最后启动并停止  首先（可能是因为它取决于正在运行的其他进程）。 在考虑  相位值，知道任何“正常”的默认相位也很重要  `Lifecycle`未实现的对象 `SmartLifecycle`是 `0`。 因此，任何  负相位值指示对象应在这些标准之前开始  组件（然后停下来）。 对于任何正相位值，反之亦然。 

定义的stop方法 `SmartLifecycle`接受回调。 任何  实现必须 调用该回调的 `run()`在该实现的之后 方法  关闭过程完成。 这样可以在必要时启用异步关机，因为  的默认实现 `LifecycleProcessor`接口 ，  `DefaultLifecycleProcessor`，等待对象组的超时值  在每个阶段中调用该回调。 默认的每阶段超时为30秒。  您可以通过定义一个名为Bean来覆盖默认的生命周期处理器实例。  `lifecycleProcessor`在上下文中。 如果您只想修改超时，  定义以下内容就足够了： 

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

如前所述，该 `LifecycleProcessor`接口为  以及上下文的刷新和关闭。 后者驱动关机  好像 过程 `stop()`已经被显式调用的 ，但是当上下文是  关闭。 另一方面，“刷新”回调启用了  `SmartLifecycle`豆子。 刷新上下文时（所有对象  实例化和初始化），则调用该回调。 那时，  默认生命周期处理器检查每个返回的布尔值  `SmartLifecycle`对象的 `isAutoStartup()`方法。 如果为 `true`，则该对象为  从这一点开始，而不是等待上下文的显式调用  自己的 `start()`方法（与上下文刷新不同，上下文启动不会发生  自动用于标准上下文实现）。 该 `phase`值与任何  如前所述，“依赖”关系确定启动顺序。 

> **Liftcycle不能被自动执行，想被自动执行需要使用SmartLifecycle**

##### Shutting Down the Spring IoC Container Gracefully in Non-Web Applications 在非Web应用程序中正常关闭Spring IoC容器 

>本部分仅适用于非Web应用程序。 基于Web的Spring  `ApplicationContext`实现中已经有代码可以正常关闭  相关Web应用程序关闭时，使用Spring IoC容器

如果您在非Web应用程序环境中使用Spring的IoC容器（对于  例如，在富客户端桌面环境中），向  JVM。 这样做可以确保正常关机，并在您的计算机上调用相关的destroy方法  单例bean，以便释放所有资源。 您仍然必须配置  并正确实现这些destroy回调。 

要注册关闭挂钩，请调用以下 `registerShutdownHook()`方法：  在 上声明 `ConfigurableApplicationContext`接口 ，如以下示例所示： 

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

#### 1.6.2. `ApplicationContextAware` and `BeanNameAware`

当 `ApplicationContext`创建创建实现的对象实例时，  `org.springframework.context.ApplicationContextAware`接口，提供对`ApplicationContext`此引用的实例  。 以下清单显示了 `ApplicationContextAware` 接口的定义： 

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

因此，bean可以以编程方式操纵创建它们的 `ApplicationContext`，  通过 `ApplicationContext`接口或通过将引用转换为已知的此接口的子类（例如 `ConfigurableApplicationContext`，暴露了附加功能）。 一种用途是通过编程方式检索其他bean。有时，此功能很有用。 但是，通常应避免使用它，因为  它将代码耦合到Spring，并且不遵循控制反转样式，  将协作者作为属性提供给bean的地方。 其他方法  `ApplicationContext`提供对文件资源的访问权限，发布应用程序事件，  并访问一个 `MessageSource`。 这些附加功能在  [的其他功能 `ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction)。 

Autowiring是获得参考的另一种选择  `ApplicationContext`。 在 *传统*  `constructor`和 `byType`自动装配模式  （如 [自动装配协作器中所述 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire)）可以提供类型的依赖关系  `ApplicationContext`对于构造函数参数或setter方法参数，  分别。 为了获得更大的灵活性，包括能够自动连接字段和  多参数方法，请使用基于注解的自动装配功能。 如果你这样做  将 `ApplicationContext`自动连接到字段，构造函数参数或方法中  期望 参数， `ApplicationContext`类型的 如果是字段，构造函数或  有问题的方法带有 `@Autowired`注释。 有关更多信息，请参见  [使用 `@Autowired`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation)。 

当 `ApplicationContext`创建创建实现 `org.springframework.beans.factory.BeanNameAware`接口的类时， ，该类提供了  对在其关联对象定义中定义的名称的引用。 以下清单  显示了BeanNameAware接口的定义： 

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

在填充正常的bean属性之后但在  初始化回调，例如 `InitializingBean`， `afterPropertiesSet`或自定义  初始化方法。 

#### 1.6.3. Other `Aware` Interfaces

除了 `ApplicationContextAware`和 `BeanNameAware`（前面 讨论 [已经 过 ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware)），  Spring提供了各种各样的 `Aware`回调接口，这些接口使bean可以指示容器  他们需要一定的基础设施依赖性。 通常，名称表示  依赖类型。 下表总结了最重要的 `Aware`接口： 

Table 4. Aware interfaces

| Name                             | Injected Dependency                                          | Explained in…                                                |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `ApplicationContextAware`        | 说明`ApplicationContext`.                                    | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| `ApplicationEventPublisherAware` | 封闭的 `ApplicationContext`的事件发布者.                     | [Additional Capabilities of the `ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction) |
| `BeanClassLoaderAware`           | 类加载器，用于加载Bean类。                                   | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
| `BeanFactoryAware`               | 说明 `BeanFactory`.                                          | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| `BeanNameAware`                  | 声明bean的名称。                                             | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| `LoadTimeWeaverAware`            | 定义的编织器，用于在加载时处理类定义。                       | [Load-time Weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-aj-ltw) |
| `MessageSourceAware`             | 解决消息的已配置策略（支持参数化和  国际化）。               | [Additional Capabilities of the `ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction) |
| `NotificationPublisherAware`     | Spring JMX通知发布者。                                       | [Notifications](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jmx-notifications) |
| `ResourceLoaderAware`            | 配置的加载程序，用于对资源的低级访问。                       | [Resources](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources) |
| `ServletConfigAware`             | 当前 `ServletConfig`容器在其中运行。仅在可感知网络的Spring中有效  `ApplicationContext`。 | [Spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc) |
| `ServletContextAware`            | 当前 `ServletContext`容器在其中运行。仅在可感知网络的Spring中有效  `ApplicationContext`。 | [Spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc) |

### 1.7. Bean Definition Inheritance Bean定义继承 

 