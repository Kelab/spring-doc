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

![container magic](E:\project\labGit\spring-doc\Core\.img\container-magic.png)

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