## 2.Resources

本章介绍了Spring如何处理资源以及如何在Spring中使用资源。它包括以下主题：

- 介绍
- 资源接口
- 内置资源实现
- The ResourceLoader  (资源加载器接口)
- ResourceLoaderAware 接口
- 资源依赖
- 应用程序上下文和资源路径

### 2.1。Introduction   介绍

Java有应对各种URL前缀的java.net.URL标准类和标准处理程序，不幸的是他们都不满足对低级资源的所有访问。例如，没有标准化的`URL`实现可用于访问需要从类路径或相对于ServletContext获取的资源 。尽管可以为专用`URL` 前缀注册新的处理程序（类似于诸如这样的前缀的现有处理程序`http:`），但这通常相当复杂，并且`URL`接口仍然缺少某些理想的功能，例如一种检查被指向的指向资源是否存在的方法。

### 2.2。 The Resource Interface   资源接口

Spring的`Resource`接口旨在成为一种功能更强大的接口，用于抽象化对低级资源的访问。以下清单显示了`Resource`接口定义：

Java

```java
public interface Resource extends InputStreamSource {

    boolean exists();

    boolean isOpen();

    URL getURL() throws IOException;

    File getFile() throws IOException;

    Resource createRelative(String relativePath) throws IOException;

    String getFilename();

    String getDescription();
}
```

或

Kotlin

```kotlin
interface Resource : InputStreamSource {

    fun exists(): Boolean

    val isOpen: Boolean

    val url: URL

    val file: File

    @Throws(IOException::class)
    fun createRelative(relativePath: String): Resource

    val filename: String

    val description: String
}
```



如`Resource`接口的定义所示，它扩展了`InputStreamSource` 接口。以下清单显示了`InputStreamSource` 接口的定义：

Java

```java
public interface InputStreamSource {

    InputStream getInputStream() throws IOException;
}
```

 或 Kotlin：

```kotlin
interface InputStreamSource {

    val inputStream: InputStream
}
```

该`Resource`接口中一些最重要的方法是：

- `getInputStream()`：定位并打开该资源，并从该资源中返回一个`InputStream`以供读取。预计每次调用都会返回一个新的 `InputStream`。调用者有责任关闭该流。
- `exists()`：返回`boolean`指示此资源是否以实际的物理形式存在。
- `isOpen()`：返回，`boolean`指示此资源是否代表一个打开的流的句柄。如果为`true`，`InputStream`则不能多次读取，必须只读取一次，然后将其关闭以避免资源泄漏。返回`false`，所有常用资源实现（InputStreamResource除外）``。
- `getDescription()`：返回对此资源的描述，以便在使用该资源时用于错误输出。这通常是标准文件名或资源的实际URL。

其他方法允许您获取表示资源的实际url或file对象(如果底层实现兼容并支持该功能)。

`当需要Resource`时，Spring本身广泛使用抽象作为许多方法签名中的参数类型。某些Spring API中的其他方法（例如，各种`ApplicationContext`实现的构造函数）采用的 `String`形式，以无修饰或简单的形式创建适合该上下文实现的Resource，或者通过`String`路径上的特殊前缀，让调用者指定特定的必须创建和使用的`Resource`实现。

尽管`Resource`接口经常被Spring使用，但实际上，在您自己的代码中单独用作通用实用工具类来访问资源，即使您的代码不了解或不关心其他任何spring部分，也非常有用。虽然这会将您的代码耦合到Spring，但实际上仅将其耦合到这套功能更加强大的可以作为URL替代品的实用程序类中，并且可以等同于您为此目的使用的任何其他库。

|      | 该`Resource`抽象并没有改变功能。它尽可能地包装它。例如，`UrlResource`包装器包装URL，然后使用包装`URL`器完成其工作。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.3。内置资源实现

Spring包含以下`Resource`实现：

- UrlResource   url资源
- ClassPathResource   类路径资源
- FileSystemResource    文件系统资源
- ServletContextResource   服务上下文接口
- InputStreamResource   输入流资源
- ByteArrayResource     字节资源

#### 2.3.1. `UrlResource`

`UrlResource`包装了一个`java.net.URL`，可用于访问通常可以通过URL访问的任何对象，例如文件，HTTP目标，FTP目标等。所有URL都有一个标准化的`String`表示形式，因此，使用适当的标准化前缀来表示另一种URL类型。这包括`file:`访问文件系统路径，`http:`通过HTTP协议 `ftp:`通过FTP访问资源等。

网址资源是由Java代码通过显式使用网址资源构造函数创建的，但当您调用一个采用字符串参数来表示路径的应用编程接口方法时，通常会隐式创建。对于后一种情况，JavaBeans属性编辑器最终决定创建哪种类型的资源。如果路径字符串包含众所周知的前缀(如类路径:)，它会为该前缀创建一个适当的专用资源。但是，如果它不识别前缀，它会假设该字符串是标准的网址字符串，并创建一个网址资源。

#### 2.3.2。 `ClassPathResource`

此类表示应从类路径获取的资源。它使用线程上下文类加载器，给定的类加载器或给定的类来加载资源。

此`Resource`实现支持解析，就`java.io.File`好像类路径资源驻留在文件系统中一样，而不支持类路径资源驻留在jar中并且尚未（通过servlet引擎或任何环境被扩展）到文件系统中。为了解决这个问题，各种`Resource`实现始终支持将分辨率作为`java.net.URL`。

A`ClassPathResource`是由Java代码通过显式使用`ClassPathResource` 构造函数创建的，但通常在调用带有`String`用于表示路径的参数的API方法时隐式创建 。对于后一种情况，JavaBeans会 在字符串路径上`PropertyEditor`识别特殊前缀，`classpath:`并`ClassPathResource`在这种情况下创建一个。

#### 2.3.3。 `FileSystemResource`

这是一个`Resource`执行`java.io.File`和`java.nio.file.Path`手柄。它支持分辨率为`File`和`URL`。

#### 2.3.4。 `ServletContextResource`

这是资源的`Resource`实现，该`ServletContext`资源解释相关Web应用程序根目录中的相对路径。

它始终支持流访问和URL访问，但`java.io.File`仅在扩展Web应用程序档案且资源实际位于文件系统上时才允许访问。它是在文件系统上扩展还是直接扩展，或者直接从JAR或其他类似数据库（可以想到的）访问，实际上取决于Servlet容器。

#### 2.3.5. `InputStreamResource`

一个`InputStreamResource`是`Resource`给定的实现`InputStream`。仅当没有特定的`Resource`实现适用时才应使用它。特别地，在可能`ByteArrayResource`的`Resource`情况下，首选 或任何基于文件的实现。

与其他`Resource`实现相反，这是一个已经打开的资源的描述符。因此，它`true`从返回`isOpen()`。如果您需要将资源描述符保留在某个地方，或者需要多次读取流，请不要使用它。

#### 2.3.6。 `ByteArrayResource`

这是`Resource`给定字节数组的实现。它`ByteArrayInputStream`为给定的字节数组创建一个 。

这对于从任何给定的字节数组中加载内容很有用，而不必使用一次使用`InputStreamResource`。

### 2.4。`ResourceLoader`

该`ResourceLoader`接口应由可以返回（即加载）`Resource`实例的对象实现。以下清单显示了`ResourceLoader` 接口定义：

java:

```java
public interface ResourceLoader {

    Resource getResource(String location);
}
```



Kotlin:

```kotlin
interface ResourceLoader {

    fun getResource(location: String): Resource
}
```

所有应用程序上下文均实现该`ResourceLoader`接口。因此，所有应用程序上下文都可用于获取`Resource`实例。

当您在特定的应用程序上下文调用`getResource()`，并且指定的位置路径没有特定的前缀时，您将获得`Resource`适合该特定应用程序上下文的类型。例如，假定针对`ClassPathXmlApplicationContext`实例运行以下代码段：

Java:

```kotlin
Resource template = ctx.getResource("some/resource/path/myTemplate.txt");
```

或

Kotlin:

```kotlin
val template = ctx.getResource("some/resource/path/myTemplate.txt")
```

针对`ClassPathXmlApplicationContext`，该代码返回`ClassPathResource`。如果对`FileSystemXmlApplicationContext`实例运行相同的方法，它将返回 `FileSystemResource`。对于`WebApplicationContext`，它将返回 `ServletContextResource`。类似地，它将为每个上下文返回适当的对象。

结果，您可以以适合特定应用程序上下文的方式加载资源。

另一方面，`ClassPathResource`无论应用程序上下文类型如何，您都可以通过指定特殊`classpath:`前缀来强制使用，如以下示例所示：

java:

```java
Resource template = ctx.getResource("classpath:some/resource/path/myTemplate.txt");
```

Kotlin:

同样，您可以`UrlResource`通过指定任何标准 `java.net.URL`前缀来强制使用a 。以下两个示例使用`file`和`http` 前缀：

java:

```java
Resource template = ctx.getResource("file:///some/resource/path/myTemplate.txt");
```

或Kotlin:

```kotlin
val template = ctx.getResource("classpath:some/resource/path/myTemplate.txt")
```





java:

```java
Resource template = ctx.getResource("https://myhost.com/resource/path/myTemplate.txt");
```

或Kotlin:

```kotlin
val template = ctx.getResource("file:///some/resource/path/myTemplate.txt")
```

下表总结了将`String`对象转换为`Resource`对象的策略：

| Prefix     | Example                          | Explanation                                                  |
| :--------- | :------------------------------- | :----------------------------------------------------------- |
| classpath: | `classpath:com/myapp/config.xml` | Loaded from the classpath.                                   |
| file:      | `file:///data/config.xml`        | Loaded as a `URL` from the filesystem. See also [`FileSystemResource` Caveats](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-filesystemresource-caveats). |
| http:      | `https://myserver/logo.png`      | Loaded as a `URL`.                                           |
| (none)     | `/data/config.xml`               | Depends on the underlying `ApplicationContext`.              |

### 2.5。ResourceLoaderAware`接口

该`ResourceLoaderAware`接口是一个特殊的回调接口，用于标识期望提供`ResourceLoader`引用的组件。以下清单显示了`ResourceLoaderAware`接口的定义：

java:

```java
public interface ResourceLoaderAware {

    void setResourceLoader(ResourceLoader resourceLoader);
}
```

或Kotlin

```kotlin
interface ResourceLoaderAware {

    fun setResourceLoader(resourceLoader: ResourceLoader)
}
```



当一个类实现`ResourceLoaderAware`并部署到应用程序上下文中（作为由Spring托管的bean）时，该类被应用程序上下文识别为`ResourceLoaderAware`。然后应用程序上下文调用`setResourceLoader(ResourceLoader)`，将自身作为参数提供（请记住，Spring中的所有应用程序上下文都实现了该`ResourceLoader`接口）。/

由于一个`ApplicationContext`就是一个 `ResourceLoader`，因此bean也可以实现 `ApplicationContextAware`接口并直接使用提供的应用程序上下文来加载资源。但是，通常，如果需要的话，最好使用专用`ResourceLoader` 接口。该代码将仅耦合到资源加载接口（可以视为实用程序接口），而不耦合到整个Spring `ApplicationContext`接口。

在应用程序组件中，您还可以依赖于的自动装配`ResourceLoader`作为实现`ResourceLoaderAware`接口的替代方法。“传统”`constructor`和`byType`自动[装配](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire)模式（如“自动[装配](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire)” 中所述）能够分别为`ResourceLoader`构造函数参数或setter方法参数提供。为了获得更大的灵活性（包括自动装配字段和多个参数方法的能力），请考虑使用基于注释的自动装配功能。在这种情况下，只要有问题的字段，构造函数或方法带有注释，它们`ResourceLoader`就会自动连接到期望`ResourceLoader`类型的字段，构造函数参数或方法参数`@Autowired`。有关更多信息，请参见[使用`@Autowired`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation)。

### 2.6。资源依赖

如果Bean本身将通过某种动态过程来确定并提供资源路径，那么Bean通过使用该`ResourceLoader` 接口来加载资源可能是有意义的。例如，考虑加载某种模板，其中所需的特定资源取决于用户的角色。如果资源是静态的，则有必要完全消除对`ResourceLoader` 接口的使用，让Bean公开所需的`Resource`属性，并期望将其注入其中。

然后注入这些属性很简单，因为所有应用程序上下文都注册并使用了特殊的JavaBeans `PropertyEditor`，它们可以将`String`路径转换为`Resource`对象。因此，如果`myBean`具有`Resource`type的template属性，则可以为该资源配置一个简单的字符串，如以下示例所示：

```xml
<bean id="myBean" class="...">
    <property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
```

请注意，资源路径没有前缀。因此，由于应用程序上下文本身将用作，因此根据上下文的确切类型，`ResourceLoader`资源本身通过a `ClassPathResource`，a`FileSystemResource`或a加载 `ServletContextResource`。

如果需要强制使用特定`Resource`类型，则可以使用前缀。以下两个示例显示了如何强制a`ClassPathResource`和a `UrlResource`（后者用于访问文件系统文件）：

```xml
<property name="template" value="classpath:some/resource/path/myTemplate.txt">
<property name="template" value="file:///some/resource/path/myTemplate.txt"/>
```

### 2.7。应用程序上下文和资源路径

本节介绍如何使用资源创建应用程序上下文，包括使用XML的快捷方式，如何使用通配符以及其他详细信息。

#### 2.7.1。构造应用程序上下文

应用程序上下文构造函数（针对特定的应用程序上下文类型）通常采用字符串或字符串数组作为资源的位置路径，例如构成上下文定义的XML文件。

当这样的位置路径没有前缀时，从该路径构建并用于加载bean定义的特定资源类型依赖于特定的应用程序上下文，并且适合于特定的应用程序上下文。例如，考虑下面的例子，它创建了`ClassPathXmlApplicationContext`：



Java：

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("conf/appContext.xml");
```

或

Kotlin：

```kotlin
val ctx = ClassPathXmlApplicationContext("conf/appContext.xml")
```

Bean定义是从类路径加载的，因为使用了a `ClassPathResource`。但是，请考虑以下示例，该示例创建一个`FileSystemXmlApplicationContext`：

Java:

```java
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("conf/appContext.xml");
```



Kotlin:

```java
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("conf/appContext.xml");
```

现在，bean定义是从文件系统位置（在这种情况下，是相对于当前工作目录）加载的。

请注意，在位置路径上使用特殊的classpath前缀或标准URL前缀会覆盖`Resource`为加载定义而创建的默认类型。考虑以下示例：

Java:

```java
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("classpath:conf/appContext.xml");
```

或  Kotlin:

```kotlin
val ctx = FileSystemXmlApplicationContext("classpath:conf/appContext.xml")
```

使用`FileSystemXmlApplicationContext`从类路径加载bean定义。但是，它仍然是 `FileSystemXmlApplicationContext`。如果随后将其用作`ResourceLoader`，则所有未前缀的路径仍将视为文件系统路径。

##### 构造`ClassPathXmlApplicationContext`实例-快捷方式

在`ClassPathXmlApplicationContext`提供了多种构造方法以便于实例。基本思想是，您只能提供一个字符串数组，其中仅包含XML文件本身的文件名（不包含前导路径信息），还提供一个`Class`。然后`ClassPathXmlApplicationContext` 从提供的类中派生路径信息。

请考虑以下目录布局：

```
com /
  foo /
    services.xml
    daos.xml
    MessengerService.class
```

下面的示例示出了如何`ClassPathXmlApplicationContext`豆组成的实例中命名的文件中定义`services.xml`和`daos.xml`（其是在类路径）可以被实例化：

Java

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext(
    new String[] {"services.xml", "daos.xml"}, MessengerService.class);
```

或 Kotlin

```kotlin
val ctx = ClassPathXmlApplicationContext(arrayOf("services.xml", "daos.xml"), MessengerService::class.java)
```

有关[`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) 各种构造函数的详细信息，请参见javadoc。



##### 蚂蚁风格的模式

路径位置可以包含Ant样式的模式，如以下示例所示：

```
/WEB-INF/*-context.xml 
com / mycompany / ** / applicationContext.xml
文件：C：/ some / path / *-context.xml
类路径：com / mycompany / ** / applicationContext.xml
```

当路径位置包含Ant样式的模式时，解析程序将遵循更复杂的过程来尝试解析通配符。它为到达最后一个非通配符段的路径生成一个`Resource`，并从中获取一个URL。如果此URL不是`jar:`URL或特定`zip:`于容器的变体（例如WebLogic，`wsjar`WebSphere等），`java.io.File`则从中获取a并用于遍历文件系统来解析通配符。对于jar URL，解析器可以从中获取a`java.net.JarURLConnection`或手动解析jar URL，然后遍历jar文件的内容以解析通配符。

##### 对可移植性的影响

如果指定的路径已经是一个文件URL（要么是隐式的，因为基址`ResourceLoader`是一个文件系统，要么 是显式的），因此可以确保通配符以完全可移植的方式工作。

如果指定的路径是类路径位置，则解析器必须通过`Classloader.getResource()`调用来获取最后一个非通配符路径段URL 。由于这只是路径的一个节点（而不是末尾的文件），因此实际上（在`ClassLoader`javadoc中）未定义 到底返回了哪种URL。实际上，它总是`java.io.File`代表目录（类路径资源解析为文件系统位置）或某种jar URL（类路径资源解析为jar位置）。尽管如此，此操作仍存在可移植性问题。

如果为最后一个非通配符段获取了jar URL，则解析程序必须能够从中获取a`java.net.JarURLConnection`或手动解析jar URL，以便能够遍历jar的内容并解析通配符。这在大多数环境中确实有效，但在其他环境中则无效，因此我们强烈建议您在依赖特定环境之前，对来自jars的资源的通配符解析进行彻底测试。

##### 该`classpath*:`前缀

构造基于XML的应用程序上下文时，位置字符串可以使用特殊`classpath*:`前缀，如以下示例所示：

java:

```java
ApplicationContext ctx =
    new ClassPathXmlApplicationContext("classpath*:conf/appContext.xml");
```







这个特殊的前缀指定必须获取与给定名称匹配的所有类路径资源（内部，这本质上是通过对的调用发生 `ClassLoader.getResources(…)`），然后合并以形成最终的应用程序上下文定义。

|      | 通配符类路径依赖于`getResources()`基础类加载器的方法。由于当今大多数应用程序服务器提供其自己的类加载器实现，因此行为可能有所不同，尤其是在处理jar文件时。检查是否`classpath*`可行的一个简单测试是使用类加载器从classpath：的jar中加载文件 `getClass().getClassLoader().getResources("<someFileInsideTheJar>")`。尝试对具有相同名称但位于两个不同位置的文件进行此测试。如果返回了不合适的结果，请检查应用程序服务器文档中可能影响类加载器行为的设置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以在其余的位置路径（例如，）中将`classpath*:`前缀与`PathMatcher`模式结合使用`classpath*:META-INF/*-beans.xml`。在这种情况下，解析策略非常简单：`ClassLoader.getResources()`在最后一个非通配符路径段上使用一个调用，以获取类加载器层次结构中的所有匹配资源，然后在每个资源之外，使用与`PathMatcher`前面所述相同的解析策略来进行处理。通配符子路径。

##### 与通配符相关的其他注释

请注意`classpath*:`，当与Ant样式的模式结合使用时，除非模式文件实际驻留在文件系统中，否则在模式启动之前，它至少必须与至少一个根目录可靠地配合使用。这意味着诸如之类的模式 `classpath*:*.xml`可能不会从jar文件的根目录检索文件，而只会从扩展目录的根目录检索文件。

Spring检索类路径条目的能力源自JDK的 `ClassLoader.getResources()`方法，该方法仅返回文件系统位置的空字符串（指示可能要搜索的根目录）。Spring还会评估 `URLClassLoader`运行时配置和`java.class.path`jar文件中的清单，但这不能保证会导致可移植行为。

|      | 扫描类路径包需要在类路径中存在相应的目录条目。使用Ant构建JAR时，请勿激活JAR任务的仅文件开关。另外，在某些环境中，基于安全策略，可能不会暴露类路径目录，例如，在JDK 1.7.0_45及更高版本上的独立应用程序（要求在清单中设置“受信任的库”。请参阅 [https：/ /stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources](https://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources)）。在JDK 9的模块路径（Jigsaw）上，Spring的类路径扫描通常可以按预期进行。强烈建议在此处将资源放入专用目录，以避免在搜索jar文件根目录级别时出现上述可移植性问题。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`classpath:`如果要搜索的根包在多个类路径位置可用，则不能保证带有资源的蚂蚁样式模式找到匹配的资源。考虑以下资源位置示例：

```
com / mycompany / package1 / service-context.xml
```

现在考虑某人可能用来尝试找到该文件的Ant样式的路径：

```
classpath：com / mycompany / ** / service-context.xml
```

这样的资源可能只在一个位置，但是当使用诸如前面示例的路径尝试解析它时，解析器将根据返回的（第一个）URL工作 `getResource("com/mycompany");`。如果此基本包节点存在于多个类加载器位置，则实际的最终资源可能不存在。因此，在这种情况下，您应该首选使用`classpath*:`相同的Ant样式模式，该模式搜索包含根包的所有类路径位置。

#### 2.7.3。`FileSystemResource`注意事项

未附加到文件系统应用程序上下文的文件系统资源(即，当文件系统应用程序上下文不是实际的资源加载器时)会按照您的预期处理绝对路径和相对路径。相对路径相对于当前工作目录，而绝对路径相对于文件系统的根目录。

但是，出于向后兼容（历史）的原因，当`FileSystemApplicationContext`是`ResourceLoader`时，情况会发生变化 。文件系统应用程序上下文强制所有附加的文件系统资源实例将所有位置路径视为相对路径，无论它们是否以斜杠开头。实际上，这意味着以下示例是等效的:

Java：

```java
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("conf/context.xml");
```

或 Kotlin：

```kotlin
val ctx = FileSystemXmlApplicationContext("conf/context.xml")
```



java:

```java
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("/conf/context.xml");
```

或 Kotlin:

```kotlin
val ctx = FileSystemXmlApplicationContext("/conf/context.xml")
```



以下示例也是等效的（即使它们有所不同也有意义，因为一种情况是相对的，另一种情况是绝对的）：

java:

```java
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("some/resource/path/myTemplate.txt");
```

或 Kotlin

```kotlin
val ctx: FileSystemXmlApplicationContext = ...
ctx.getResource("some/resource/path/myTemplate.txt")
```

java:

```java
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("/some/resource/path/myTemplate.txt");
```

或 Kotlin：

```kotlin
val ctx: FileSystemXmlApplicationContext = ...
ctx.getResource("/some/resource/path/myTemplate.txt")
```

实际上，如果需要真正的绝对文件系统路径，则应避免将绝对路径与`FileSystemResource`或一起`FileSystemXmlApplicationContext`使用，而应`UrlResource`通过使用`file:`URL前缀来强制使用a 。以下示例显示了如何执行此操作：

java:

```java
// actual context type doesn't matter, the Resource will always be UrlResource
ctx.getResource("file:///some/resource/path/myTemplate.txt");
```

或 Kotlin:

```kotlin
// actual context type doesn't matter, the Resource will always be UrlResource
ctx.getResource("file:///some/resource/path/myTemplate.txt")
```



java:

```java
// force this FileSystemXmlApplicationContext to load its definition via a UrlResource
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("file:///conf/context.xml");
```

或 Kotlin：

```kotlin
// force this FileSystemXmlApplicationContext to load its definition via a UrlResource
val ctx = FileSystemXmlApplicationContext("file:///conf/context.xml")
```