# Intergration

## 1. REST端点(REST Endpoints)

#### 1.1. `RestTemplate`

在`RestTemplate`通过HTTP客户端库提供了一个更高水平的API。它使在一行中轻松调用REST端点变得容易。它公开了以下几组重载方法：

| 方法组            | 描述                                                         |
| :---------------- | :----------------------------------------------------------- |
| `getForObject`    | 通过GET检索表示形式。                                        |
| `getForEntity`    | `ResponseEntity`使用GET检索（即状态，标题和正文）。          |
| `headForHeaders`  | 通过使用HEAD检索资源的所有标头。                             |
| `postForLocation` | 通过使用POST创建新资源，并`Location`从响应中返回标头。       |
| `postForObject`   | 通过使用POST创建新资源，并从响应中返回表示形式。             |
| `postForEntity`   | 通过使用POST创建新资源，并从响应中返回表示形式。             |
| `put`             | 通过使用PUT创建或更新资源。                                  |
| `patchForObject`  | 通过使用PATCH更新资源，并从响应中返回表示形式。请注意，JDK`HttpURLConnection`不支持`PATCH`，但是Apache HttpComponents和其他支持。 |
| `delete`          | 使用DELETE删除指定URI处的资源。                              |
| `optionsForAllow` | 通过使用ALLOW检索资源的允许的HTTP方法。                      |
| `exchange`        | 前述方法的通用性强（且意见少的）版本，在需要时提供了额外的灵活性。它接受`RequestEntity`（包括HTTP方法，URL，标头和正文作为输入）并返回`ResponseEntity`。这些方法允许使用`ParameterizedTypeReference`而不是`Class`使用泛型来指定响应类型。 |
| `execute`         | 执行请求的最通用方法，完全控制通过回调接口进行的请求准备和响应提取。 |

#### 1.1.1. 初始化(Initialization)

默认构造函数用于`java.net.HttpURLConnection`执行请求。您可以使用的实现切换到其他HTTP库`ClientHttpRequestFactory`。内置支持以下内容：

- Apache HttpComponents
- 净额
- OkHttp

例如，要切换到Apache HttpComponents，可以使用以下命令：

```java
RestTemplate template = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
```

每个都`ClientHttpRequestFactory`公开特定于基础HTTP客户端库的配置选项-例如，用于凭证，连接池和其他详细信息。

```java
RestTemplate template = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
```

##### 统一资源标识符(URIs)

许多`RestTemplate`方法都接受URI模板和URI模板变量（作为`String`变量参数或）`Map<String,String>`。

下面的示例使用一个`String`变量参数：

```java
String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/bookings/{booking}", String.class, "42", "21");
```

以下示例使用`Map<String, String>`：

```java
Map<String, String> vars = Collections.singletonMap("hotel", "42");

String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/rooms/{hotel}", String.class, vars);
```

请注意，URI模板是自动编码的，如以下示例所示：

```JAVA
restTemplate.getForObject("https://example.com/hotel list", String.class);

// Results in request to "https://example.com/hotel%20list"
```

您可以使用的`uriTemplateHandler`属性来自`RestTemplate`定义URI的编码方式。或者，您可以准备一个`java.net.URI`并将其传递到`RestTemplate`接受的方法之一`URI`。

有关使用和编码URI的更多详细信息，请参阅[URI链接](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-uri-building)。

##### 标头(Headers)

您可以使用这些`exchange()`方法指定请求标头，如以下示例所示：

```java
String uriTemplate = "https://example.com/hotels/{hotel}";
URI uri = UriComponentsBuilder.fromUriString(uriTemplate).build(42);

RequestEntity<Void> requestEntity = RequestEntity.get(uri)
        .header(("MyRequestHeader", "MyValue")
        .build();

ResponseEntity<String> response = template.exchange(requestEntity, String.class);

String responseHeader = response.getHeaders().getFirst("MyResponseHeader");
String body = response.getBody();
```

您可以通过许多`RestTemplate`return的方法变体 获得响应头`ResponseEntity`。

#### 1.1.2. 主干(Body)

`RestTemplate`在的帮助下，传入和传出方法的对象与原始内容进行转换`HttpMessageConverter`。

在POST上，输入对象被序列化到请求主体，如以下示例所示：

```java
URI location = template.postForLocation("https://example.com/people", person);
```

您无需显式设置请求的Content-Type标头。在大多数情况下，您可以根据源`Object`类型找到兼容的消息转换器，并且所选的消息转换器会相应地设置内容类型。如有必要，可以使用这些 `exchange`方法显式提供`Content-Type`请求标头，进而影响选择哪个消息转换器。

在GET上，响应的主体反序列化为output `Object`，如以下示例所示：

```java
Person person = restTemplate.getForObject("https://example.com/people/{id}", Person.class, 42);
```

`Accept`不需要显式设置请求的标头。在大多数情况下，可以根据预期的响应类型找到兼容的消息转换器，这有助于填充`Accept`标头。如有必要，可以使用这些`exchange` 方法`Accept`显式提供标头。

默认情况下，根据有助于确定存在哪些可选转换库的类路径检查，`RestTemplate`注册所有内置 [消息转换器](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-message-conversion)。您还可以将消息转换器设置为显式使用。

#### 1.1.3. 信息转换(Message Conversion)

该`spring-web`模块包含了`HttpMessageConverter`用于阅读和写作，通过HTTP请求和响应的身体合同`InputStream`和`OutputStream`。 `HttpMessageConverter`在客户端（例如在中`RestTemplate`）和服务器端（例如在Spring MVC REST控制器中）使用实例。

框架中提供了主要媒体（MIME）类型的具体实现，默认情况下，已`RestTemplate`在客户端和`RequestMethodHandlerAdapter`服务器端注册了具体实现 （请参阅 [配置消息转换器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-message-converters)）。

`HttpMessageConverter`以下各节介绍了的实现。对于所有转换器，都使用默认媒体类型，但是您可以通过设置`supportedMediaTypes`bean属性来覆盖它 。下表描述了每种实现：

| 消息转换器                               | 描述                                                         |
| :--------------------------------------- | :----------------------------------------------------------- |
| `StringHttpMessageConverter`             | `HttpMessageConverter`可以`String`从HTTP请求和响应读取和写入实例的实现。默认情况下，此转换器支持所有文字媒体类型（`text/*`），并用写`Content-Type`的`text/plain`。 |
| `FormHttpMessageConverter`               | `HttpMessageConverter`可以从HTTP请求和响应中读取和写入表单数据的实现。默认情况下，此转换器读取和写入 `application/x-www-form-urlencoded`媒体类型。从中读取表格数据并将其写入 `MultiValueMap<String, String>`。转换器还可以写入（但不能读取）从读取的多部分数据`MultiValueMap<String, Object>`。默认情况下，`multipart/form-data`受支持。从Spring Framework 5.2开始，可以支持其他多部分子类型来编写表单数据。有关`FormHttpMessageConverter`更多详细信息，请查阅javadoc 。 |
| `ByteArrayHttpMessageConverter`          | `HttpMessageConverter`可以从HTTP请求和响应读取和写入字节数组的实现。默认情况下，该转换器支持所有媒体类型（`*/*`），并用写`Content-Type`的`application/octet-stream`。您可以通过设置`supportedMediaTypes`属性和overriding来覆盖它`getContentType(byte[])`。 |
| `MarshallingHttpMessageConverter`        | 一种`HttpMessageConverter`实现，可以通过使用Spring的XML`Marshaller`和`Unmarshaller`来自`org.springframework.oxm`包的抽象来读写XML 。该转换器需要`Marshaller`和`Unmarshaller`才能使用。您可以通过构造函数或bean属性注入它们。默认情况下，此转换器支持 `text/xml`和`application/xml`。 |
| `MappingJackson2HttpMessageConverter`    | 一个`HttpMessageConverter`可以读取和使用杰克逊的写JSON实现 `ObjectMapper`。您可以根据需要使用Jackson提供的注释来自定义JSON映射。当需要进一步控制时（对于需要为特定类型提供自定义JSON序列化器/反序列化器的情况），可以`ObjectMapper` 通过`ObjectMapper`属性注入自定义。默认情况下，此转换器支持`application/json`。 |
| `MappingJackson2XmlHttpMessageConverter` | 一个`HttpMessageConverter`可以通过使用读写XML实施 [杰克逊XML](https://github.com/FasterXML/jackson-dataformat-xml)扩展的 `XmlMapper`。您可以根据需要使用JAXB或Jackson提供的注释来自定义XML映射。当需要进一步控制时（对于需要为特定类型提供自定义XML序列化器/反序列化器的情况），可以`XmlMapper` 通过`ObjectMapper`属性注入自定义。默认情况下，此转换器支持`application/xml`。 |
| `SourceHttpMessageConverter`             | `HttpMessageConverter`可以`javax.xml.transform.Source`从HTTP请求和响应读取和写入的实现 。只有`DOMSource`， `SAXSource`和`StreamSource`支持。默认情况下，此转换器支持 `text/xml`和`application/xml`。 |
| `BufferedImageHttpMessageConverter`      | `HttpMessageConverter`可以`java.awt.image.BufferedImage`从HTTP请求和响应读取和写入的实现 。该转换器读取和写入Java I / O API支持的媒体类型。 |

#### 1.1.4. JSON 视图(Jackson JSON Views)

您可以指定[Jackson JSON视图](https://www.baeldung.com/jackson-json-view-annotation) 以仅序列化对象属性的一部分，如以下示例所示：

```java
MappingJacksonValue value = new MappingJacksonValue(new User("eric", "7!jd#h23"));
value.setSerializationView(User.WithoutPasswordView.class);

RequestEntity<MappingJacksonValue> requestEntity =
    RequestEntity.post(new URI("https://example.com/user")).body(value);

ResponseEntity<String> response = template.exchange(requestEntity, String.class);
```



##### 多部分(Multipart)

要发送多部分数据，您需要提供`MultiValueMap<String, Object>`其值可以是`Object`部分内容，`Resource`文件部分或`HttpEntity`带有标题的部分内容的。例如：

```java
MultiValueMap<String, Object> parts = new LinkedMultiValueMap<>();

parts.add("fieldPart", "fieldValue");
parts.add("filePart", new FileSystemResource("...logo.png"));
parts.add("jsonPart", new Person("Jason"));

HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_XML);
parts.add("xmlPart", new HttpEntity<>(myBean, headers));
```

在大多数情况下，您不必`Content-Type`为每个零件指定。内容类型是根据`HttpMessageConverter`选择的要自动序列化的内容确定的，或者在`Resource`基于文件扩展名的情况下确定的。如果必要的话，你可以明确地提供`MediaType`与`HttpEntity`包装。

一旦`MultiValueMap`准备好了，你可以把它传递给`RestTemplate`，如下显示：

```java
MultiValueMap<String, Object> parts = ...;
template.postForObject("https://example.com/upload", parts, Void.class);
```



如果`MultiValueMap`包含至少一个非`String`值时，`Content-Type`被设定为`multipart/form-data`通过`FormHttpMessageConverter`。如果`MultiValueMap`有 `String`值`Content-Type`默认为`application/x-www-form-urlencoded`。如有必要，`Content-Type`也可以明确设置。

### 1.2. 使用`AsyncRestTemplate`（不推荐使用）

在`AsyncRestTemplate`已被弃用。对于您可能考虑使用的所有用例，请 `AsyncRestTemplate`改用[WebClient](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client)。

## 2.远程处理和Web服务(Remoting and Web Services)

Spring提供了使用各种技术进行远程处理的支持。远程支持简化了通过Java接口和对象作为输入和输出实现的启用远程服务的开发。当前，Spring支持以下远程技术：

- [Java Web服务](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-web-services)：Spring通过JAX-WS提供对Web服务的远程支持。
- [AMQP](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-amqp)：单独的Spring AMQP项目支持通过AMQP作为基础协议进行远程处理。

|      | 从Spring Framework 5.3开始，出于安全原因和更广泛的行业支持，现已不支持多种远程技术。支持的基础结构将从Spring Framework的下一个主要版本中删除。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

现在不推荐使用以下远程处理技术，并且不会被取代：

- [远程方法调用（RMI）](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-rmi)：通过使用`RmiProxyFactoryBean`和 `RmiServiceExporter`，Spring支持传统的RMI（具有`java.rmi.Remote` 接口和`java.rmi.RemoteException`）和通过RMI调用程序（具有任何Java接口）的透明远程支持。
- [Spring HTTP Invoker（不建议使用）](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-httpinvoker)：Spring提供了一种特殊的远程处理策略，该策略允许通过HTTP进行Java序列化，从而支持任何Java接口（就像RMI调用程序一样）。对应的支持类是`HttpInvokerProxyFactoryBean` 和`HttpInvokerServiceExporter`。
- [Hessian](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-caucho-protocols-hessian)：通过使用Spring`HessianProxyFactoryBean`和 `HessianServiceExporter`，您可以通过Caucho提供的基于HTTP的轻量级二进制协议透明地公开您的服务。
- [JMS（不建议使用）](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-jms)：通过JMS作为基础协议进行远程处理，通过模块中的`JmsInvokerServiceExporter`和`JmsInvokerProxyFactoryBean`类 来支持 `spring-jms`。

在讨论Spring的远程功能时，我们使用以下域模型和相应的服务：

```java
public class Account implements Serializable{

    private String name;

    public String getName(){
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

```java
public interface AccountService {

    public void insertAccount(Account account);

    public List<Account> getAccounts(String name);
}
```

```java
// the implementation doing nothing at the moment
public class AccountServiceImpl implements AccountService {

    public void insertAccount(Account acc) {
        // do something...
    }

    public List<Account> getAccounts(String name) {
        // do something...
    }
}
```

本节首先使用RMI将服务公开给远程客户端，然后再谈谈使用RMI的缺点。然后继续以使用Hessian作为协议的示例。

### 2.1. AMQP

Spring AMQP项目支持通过AMQP作为基础协议进行远程处理。有关更多详细信息，请访问 Spring AMQP参考中的[Spring Remoting](https://docs.spring.io/spring-amqp/docs/current/reference/html/#remoting)部分。

|      | 远程接口未实现自动检测对于远程接口，不会自动检测已实现接口的主要原因是为了避免向远程呼叫者打开太多的门。目标对象可能实现内部回调接口，例如`InitializingBean`或`DisposableBean` 不希望向调用者公开的接口。在本地情况下，提供具有目标所实现的所有接口的代理通常无关紧要。但是，在导出远程服务时，应公开特定的服务接口，以及用于远程使用的特定操作。除了内部回调接口之外，目标还可以实现多个业务接口，其中只有一个用于远程公开。由于这些原因，我们需要指定这样的服务接口。这是配置便利性与内部方法意外暴露的风险之间的折衷方案。始终指定服务接口并不会花费太多精力，这使您可以安全地控制特定方法的使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.2. 选择技术时的注意事项

这里介绍的每种技术都有其缺点。选择一种技术时，应仔细考虑您的需求，公开的服务以及通过网络发送的对象。

使用RMI时，除非通过隧道传送RMI流量，否则无法通过HTTP协议访问对象。RMI是一个重量级协议，因为它支持全对象序列化，当您使用需要通过网络进行序列化的复杂数据模型时，RMI非常重要。但是，RMI-JRMP与Java客户端绑定在一起。它是Java到Java的远程解决方案。

如果您需要基于HTTP的远程处理而且还依赖Java序列化，那么Spring的HTTP调用程序是一个不错的选择。它与RMI调用程序共享基本的基础结构，但使用HTTP作为传输。请注意，HTTP调用程序不仅限于Java到Java远程处理，还不仅限于客户端和服务器端的Spring。（后者也适用于非RMI接口的Spring RMI调用程序。）

在异类环境中运行时，Hessian可能会提供重要的价值，因为它们明确允许使用非Java客户端。但是，非Java支持仍然有限。已知的问题包括Hibernate对象的序列化以及延迟初始化的集合。如果您有这样的数据模型，请考虑使用RMI或HTTP调用程序而不是Hessian。

JMS对于提供服务集群以及让JMS代理负责负载平衡，发现和自动故障转移很有用。默认情况下，Java序列化用于JMS远程处理，但是JMS提供程序可以使用其他机制进行线路格式化，例如XStream，以使服务器可以用其他技术实现。

最后但并非最不重要的一点是，EJB具有优于RMI的优势，因为它支持标准的基于角色的身份验证和授权以及远程事务传播。尽管核心Spring并没有提供RMI调用程序或HTTP调用程序来支持安全上下文传播，但也有可能。Spring仅提供用于插入第三方或自定义解决方案的挂钩。

###  2.3. Java Web服务

Spring提供了对标准Java Web服务API的全面支持：

- 使用JAX-WS公开Web服务
- 使用JAX-WS访问Web服务

除了在Spring Core中对JAX-WS的库存支持之外，Spring产品组合还具有[Spring Web Services](http://www.springframework.org/spring-ws)，它是合同优先，文档驱动的Web服务的解决方案-强烈建议构建现代的，面向未来的Web服务。

#### 2.3.1. 使用JAX-WS公开基于Servlet的Web服务

Spring提供了JAX-WS servlet的端点实现提供了一个方便的基类： `SpringBeanAutowiringSupport`。为了公开我们的内容`AccountService`，我们扩展了Spring的 `SpringBeanAutowiringSupport`类并在此处实现我们的业务逻辑，通常将调用委派给业务层。我们使用Spring的`@Autowired` 注释来表达对Spring管理的bean的依赖。以下示例显示了扩展的类`SpringBeanAutowiringSupport`：

```java
/**
 * JAX-WS compliant AccountService implementation that simply delegates
 * to the AccountService implementation in the root web application context.
 *
 * This wrapper class is necessary because JAX-WS requires working with dedicated
 * endpoint classes. If an existing service needs to be exported, a wrapper that
 * extends SpringBeanAutowiringSupport for simple Spring bean autowiring (through
 * the @Autowired annotation) is the simplest JAX-WS compliant way.
 *
 * This is the class registered with the server-side JAX-WS implementation.
 * In the case of a Java EE server, this would simply be defined as a servlet
 * in web.xml, with the server detecting that this is a JAX-WS endpoint and reacting
 * accordingly. The servlet name usually needs to match the specified WS service name.
 *
 * The web service engine manages the lifecycle of instances of this class.
 * Spring bean references will just be wired in here.
 */
import org.springframework.web.context.support.SpringBeanAutowiringSupport;

@WebService(serviceName="AccountService")
public class AccountServiceEndpoint extends SpringBeanAutowiringSupport {

    @Autowired
    private AccountService biz;

    @WebMethod
    public void insertAccount(Account acc) {
        biz.insertAccount(acc);
    }

    @WebMethod
    public Account[] getAccounts(String name) {
        return biz.getAccounts(name);
    }
}
```

我们`AccountServiceEndpoint`需要在与Spring上下文相同的Web应用程序中运行，以允许访问Spring的设施。在Java EE环境中，默认情况下就是这种情况，使用标准合约进行JAX-WS servlet端点部署。有关详细信息，请参见各种Java EE Web服务教程。

####  2.3.2. 使用JAX-WS导出独立的Web服务

Oracle JDK随附的内置JAX-WS提供程序通过使用JDK中也包含的内置HTTP服务器来支持Web服务公开。Spring `SimpleJaxWsServiceExporter`会`@WebService`在Spring应用程序上下文中检测所有带注释的bean，并通过默认的JAX-WS服务器（JDK HTTP服务器）导出它们。

在这种情况下，终结点实例被定义和管理为Spring bean本身。它们已在JAX-WS引擎中注册，但是它们的生命周期取决于Spring应用程序上下文。这意味着您可以将Spring功能（例如显式依赖项注入）应用于端点实例。注释驱动的注入也`@Autowired`同样有效。以下示例显示了如何定义这些bean：

```xml
<bean class="org.springframework.remoting.jaxws.SimpleJaxWsServiceExporter">
    <property name="baseAddress" value="http://localhost:8080/"/>
</bean>

<bean id="accountServiceEndpoint" class="example.AccountServiceEndpoint">
    ...
</bean>

...
```

该`AccountServiceEndpoint`可以，但并没有从Spring的推导`SpringBeanAutowiringSupport`，因为在这个例子中，端点是完全春季管理的bean。这意味着端点实现可以如下所示（不声明任何超类，并且`@Autowired`仍采用Spring的配置注释）：

```java
@WebService(serviceName="AccountService")
public class AccountServiceEndpoint {

    @Autowired
    private AccountService biz;

    @WebMethod
    public void insertAccount(Account acc) {
        biz.insertAccount(acc);
    }

    @WebMethod
    public List<Account> getAccounts(String name) {
        return biz.getAccounts(name);
    }
}
```

####  2.3.3. 使用JAX-WS RI的Spring支持导出Web服务

作为GlassFish项目的一部分开发的Oracle JAX-WS RI，将Spring支持作为其JAX-WS Commons项目的一部分。这允许将JAX-WS端点定义为Spring管理的Bean，类似于上[一节中](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-web-services-jaxws-export-standalone)讨论的独立模式，  但这次是在Servlet环境中。

|      | 这在Java EE环境中不可移植。它主要适用于将JAX-WS RI嵌入为Web应用程序一部分的非EE环境，例如Tomcat。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

与导出基于servlet的终结点的标准样式的不同之处在于，终结点实例本身的生命周期由Spring管理，并且在中仅定义了一个JAX-WS servlet `web.xml`。使用标准的Java EE样式（如前所述），每个服务端点都有一个servlet定义，每个端点通常都委派给Spring Bean（`@Autowired`如前所述，通过使用）。

有关 设置和使用样式的详细信息，请参见https://jax-ws-commons.java.net/spring/。

#### 2.3.4.  使用JAX-WS访问Web服务

Spring提供了两个工厂bean来创建JAX-WS Web服务代理，分别是 `LocalJaxWsServiceFactoryBean`和`JaxWsPortProxyFactoryBean`。前者只能返回一个JAX-WS服务类供我们使用。后者是完整版本，可以返回实现我们的业务服务接口的代理。在以下示例中，我们`JaxWsPortProxyFactoryBean`用于`AccountService`（再次）为端点创建代理 ：

```xml
<bean id="accountWebService" class="org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean">
    <property name="serviceInterface" value="example.AccountService"/> 
    <property name="wsdlDocumentUrl" value="http://localhost:8888/AccountServiceEndpoint?WSDL"/>
    <property name="namespaceUri" value="https://example/"/>
    <property name="serviceName" value="AccountService"/>
    <property name="portName" value="AccountServiceEndpointPort"/>
</bean>
```

`serviceInterface`客户使用的我们的业务接口在哪里？

`wsdlDocumentUrl`是WSDL文件的URL。Spring在启动时需要此来创建JAX-WS服务。`namespaceUri`对应`targetNamespace`于.wsdl文件中的。`serviceName`对应于.wsdl文件中的服务名称。`portName` 对应于.wsdl文件中的端口名称。

访问Web服务很容易，因为我们有一个供其使用的bean工厂，它将它公开为一个名为的接口`AccountService`。以下示例显示了如何在Spring中进行连接：

```xml
<bean id="client" class="example.AccountClientImpl">
    ...
    <property name="service" ref="accountWebService"/>
</bean>
```

```java
public class AccountClientImpl {

    private AccountService service;

    public void setService(AccountService service) {
        this.service = service;
    }

    public void foo() {
        service.insertAccount(...);
    }
}
```

上面的内容略有简化，因为JAX-WS要求端点接口和实现类使用`@WebService`，`@SOAPBinding`等等注释进行注释。这意味着您不能（轻松）使用纯Java接口和实现类作为JAX-WS端点工件。您需要首先对它们进行注释。查看JAX-WS文档以获取有关这些需求的详细信息。

### 	2.4. RMI（不建议使用）

从Spring Framework 5.3开始，RMI支持已被弃用，不会被取代。

通过使用Spring对RMI的支持，您可以通过RMI基础结构透明地公开服务。进行了此设置之后，除了不存在对安全上下文传播或远程事务传播的标准支持这一事实之外，您基本上具有与远程EJB相似的配置。当您使用RMI调用程序时，Spring确实为此类附加调用上下文提供了挂钩，因此，例如，您可以插入安全框架或自定义安全凭证。

#### 2.4.1 通过使用导出服务`RmiServiceExporter`

使用`RmiServiceExporter`，我们可以将AccountService对象的接口公开为RMI对象。可以使用`RmiProxyFactoryBean`或通过传统RMI服务通过普通RMI访问该接口。在`RmiServiceExporter`明确支持通过RMI调用任何非RMI业务。

我们首先必须在Spring容器中设置服务。以下示例显示了如何执行此操作：

```java
<bean id="accountService" class="example.AccountServiceImpl">
    <!-- any additional properties, maybe a DAO? -->
</bean>
```

接下来，我们必须使用公开我们的服务`RmiServiceExporter`。以下示例显示了如何执行此操作：

```xml
<bean class="org.springframework.remoting.rmi.RmiServiceExporter">
    <!-- does not necessarily have to be the same name as the bean to be exported -->
    <property name="serviceName" value="AccountService"/>
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
    <!-- defaults to 1099 -->
    <property name="registryPort" value="1199"/>
</bean>
```

在前面的示例中，我们覆盖了RMI注册表的端口。通常，您的应用服务器还维护一个RMI注册表，因此最好不要干涉该注册表。此外，服务名称用于绑定服务。因此，在前面的示例中，服务绑定在`'rmi://HOST:1199/AccountService'`。我们稍后将使用此URL来链接客户端的服务。

该`servicePort`属性已被省略（默认为0）。这意味着将使用匿名端口与服务进行通信。

#### 2.4.2. 在客户端链接服务

我们的客户是一个使用`AccountService`来管理帐户的简单对象，如以下示例所示：

```java
public class SimpleObject {

    private AccountService accountService;

    public void setAccountService(AccountService accountService) {
        this.accountService = accountService;
    }

    // additional methods using the accountService
}
```

为了在客户端上链接服务，我们创建了一个单独的Spring容器，其中包含以下简单对象和服务链接配置位：

```xml
<bean class="example.SimpleObject">
    <property name="accountService" ref="accountService"/>
</bean>

<bean id="accountService" class="org.springframework.remoting.rmi.RmiProxyFactoryBean">
    <property name="serviceUrl" value="rmi://HOST:1199/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

这就是我们要在客户端上支持远程帐户服务所需要做的一切。Spring透明地创建一个调用程序，并通过远程启用帐户服务 `RmiServiceExporter`。在客户端，我们使用进行链接`RmiProxyFactoryBean`。

### 2.5. 使用Hessian通过HTTP远程调用服务（不建议使用）

从Spring Framework 5.3开始，Hessian支持已被弃用，不会被取代。

Hessian提供了一个基于HTTP的二进制远程协议。它由Caucho开发，您可以在[https://www.caucho.com/上](https://www.caucho.com/)找到有关Hessian本身的更多信息

#### 2.5.1. Hessian

essian通过HTTP进行通信，并通过使用自定义servlet进行通信。通过使用Spring的 `DispatcherServlet`原理（请参阅[webmvc.html](https://docs.spring.io/spring-framework/docs/current/reference/html/webmvc.html#mvc-servlet)），我们可以连接这样的servlet以公开您的服务。首先，我们必须在应用程序中创建一个新的servlet，如以下摘录所示:

```xml
<servlet>
    <servlet-name>remoting</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>remoting</servlet-name>
    <url-pattern>/remoting/*</url-pattern>
</servlet-mapping>
```

如果您熟悉Spring的`DispatcherServlet`原理，那么您可能知道现在必须在目录中创建一个名为`remoting-servlet.xml`（在Servlet名称之后）的Spring容器配置资源 `WEB-INF`。下一节将使用应用程序上下文。

或者，考虑使用Spring的simpler `HttpRequestHandlerServlet`。这样做可以使您将远程导出器定义嵌入到根应用程序上下文中（默认情况下，在中`WEB-INF/applicationContext.xml`），并使用指向特定导出器bean的单个servlet定义。在这种情况下，每个servlet名称都必须与其目标导出器的bean名称相匹配。

#### 2.5.2. 使用公开您的bean

在名为的新创建的应用程序上下文中`remoting-servlet.xml`，我们创建一个 `HessianServiceExporter`来导出我们的服务，如以下示例所示：

```xml
<bean id="accountService" class="example.AccountServiceImpl">
    <!-- any additional properties, maybe a DAO? -->
</bean>

<bean name="/AccountService" class="org.springframework.remoting.caucho.HessianServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

现在，我们准备在客户端链接服务。没有指定显式处理程序映射（将请求URL映射到服务），因此我们使用`BeanNameUrlHandlerMapping` used。因此，将在包含`DispatcherServlet`实例的映射（如先前定义）中 通过其bean名称指示的URL导出服务`https://HOST:8080/remoting/AccountService`。

或者，您可以`HessianServiceExporter`在根应用程序上下文中创建一个（例如，在中`WEB-INF/applicationContext.xml`），如以下示例所示：

```xml
<bean name="accountExporter" class="org.springframework.remoting.caucho.HessianServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

在后一种情况下，您应该在中为该导出器定义一个相应的servlet `web.xml`，并得到相同的最终结果：导出器被映射到位于的请求路径 `/remoting/AccountService`。注意，servlet名称需要与目标导出器的bean名称匹配。以下示例显示了如何执行此操作：

```xml
<servlet>
    <servlet-name>accountExporter</servlet-name>
    <servlet-class>org.springframework.web.context.support.HttpRequestHandlerServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>accountExporter</servlet-name>
    <url-pattern>/remoting/AccountService</url-pattern>
</servlet-mapping>
```

#### 2.5.3. 在客户端上链接服务

通过使用`HessianProxyFactoryBean`，我们可以在客户端链接服务。与RMI示例相同的原理适用。我们创建一个单独的bean工厂或应用程序上下文，并`SimpleObject`通过使用`AccountService`来管理帐户来提及以下bean ，如以下示例所示：

```xml
<bean class="example.SimpleObject">
    <property name="accountService" ref="accountService"/>
</bean>

<bean id="accountService" class="org.springframework.remoting.caucho.HessianProxyFactoryBean">
    <property name="serviceUrl" value="https://remotehost:8080/remoting/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

#### 2.5.4. 将HTTP基本身份验证应用于通过Hessian公开的服务

Hessian的优点之一是我们可以轻松地应用HTTP基本身份验证，因为这两种协议都是基于HTTP的。`web.xml`例如，可以通过使用安全功能来应用常规的HTTP服务器安全性机制。通常，您无需在此处使用每个用户的安全凭证。相反，您可以使用在该`HessianProxyFactoryBean`级别定义的共享凭据（类似于JDBC `DataSource`），如以下示例所示：

```xml
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping">
    <property name="interceptors" ref="authorizationInterceptor"/>
</bean>

<bean id="authorizationInterceptor"
        class="org.springframework.web.servlet.handler.UserRoleAuthorizationInterceptor">
    <property name="authorizedRoles" value="administrator,operator"/>
</bean>
```

在前面的示例中，我们明确提到`BeanNameUrlHandlerMapping`并设置了一个拦截器，以仅让管理员和操作员调用此应用程序上下文中提到的bean。

### 2.6. Spring HTTP Invoker（不建议使用）

从Spring Framework 5.3开始，不推荐使用HTTP Invoker支持，并且不会替换。

与Hessian相反，Spring HTTP调用程序都是轻量级协议，它们使用自己的苗条序列化机制，并使用标准Java序列化机制通过HTTP公开服务。如果您的参数和返回类型是无法通过使用Hessian使用的序列化机制进行序列化的复杂类型，则这将具有巨大的优势（有关选择远程处理技术的更多注意事项，请参阅下一节）。

在幕后，Spring使用JDK或Apache提供的标准功能`HttpComponents`来执行HTTP调用。如果您需要更高级且更易于使用的功能，请使用后者。有关 更多信息，请参见 [hc.apache.org/httpcomponents-client-ga/](https://hc.apache.org/httpcomponents-client-ga/)。

注意由于不安全的Java反序列化而导致的漏洞：操纵输入流可能会在反序列化步骤中导致服务器上不必要的代码执行。因此，请勿将HTTP调用方终结点暴露给不受信任的客户端。而是仅在您自己的服务之间公开它们。通常，强烈建议您改用其他任何消息格式（例如JSON）。

如果您担心由Java序列化引起的安全漏洞，请考虑在核心JVM级别上使用通用序列化筛选器机制，该机制最初是为JDK 9开发的，但同时又移植到了JDK 8、7和6。参见 https://blogs.oracle.com/java-platform-group/entry/incoming_filter_serialization_data_a 和https://openjdk.java.net/jeps/290。

#### 2.6.1. 公开服务对象

为服务对象设置HTTP调用程序基础结构与使用Hessian进行设置的方法非常相似。正如Hessian支持提供的那样 `HessianServiceExporter`，Spring的HttpInvoker支持提供了 `org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter`。

为了`AccountService`在Spring Web MVC中公开（前面提到的） `DispatcherServlet`，需要在调度程序的应用程序上下文中进行以下配置，如以下示例所示：

```xml
<bean name="/AccountService" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

此类导出器定义通过`DispatcherServlet`实例的标准映射功能公开，如[有关Hessian的部分所述](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-caucho-protocols)。

或者，您可以`HttpInvokerServiceExporter`在根应用程序上下文中创建一个（例如，在中`'WEB-INF/applicationContext.xml'`），如以下示例所示：

```xml
<bean name="accountExporter" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

另外，您可以在中为该导出器定义一个相应的servlet，该`web.xml`servlet名称与目标导出器的bean名称匹配，如以下示例所示：

```xml
<servlet>
    <servlet-name>accountExporter</servlet-name>
    <servlet-class>org.springframework.web.context.support.HttpRequestHandlerServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>accountExporter</servlet-name>
    <url-pattern>/remoting/AccountService</url-pattern>
</servlet-mapping>
```

#### 2.6.2. 在客户端链接服务

同样，从客户端链接服务非常类似于使用Hessian时的方式。通过使用代理，Spring可以将对HTTP POST请求的调用转换为指向导出服务的URL。以下示例显示如何配置此安排：

```xml
<bean id="httpInvokerProxy" class="org.springframework.remoting.httpinvoker.HttpInvokerProxyFactoryBean">
    <property name="serviceUrl" value="https://remotehost:8080/remoting/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

如前所述，您可以选择要使用的HTTP客户端。默认情况下，会 `HttpInvokerProxy`使用JDK的HTTP功能，但是您也可以`HttpComponents`通过设置`httpInvokerRequestExecutor`属性来使用Apache 客户端。以下示例显示了如何执行此操作：

```xml
<property name="httpInvokerRequestExecutor">
    <bean class="org.springframework.remoting.httpinvoker.HttpComponentsHttpInvokerRequestExecutor"/>
</property>
```

### 2.7. JMS（已弃用）

从Spring Framework 5.3开始，已弃用JMS远程支持，并且将不会替换。

您还可以通过使用JMS作为基础通信协议来透明地公开服务。Spring框架中的JMS远程支持非常基本。它在`same thread`和上以相同的非事务发送和接收`Session`。结果，吞吐量取决于实现方式。请注意，这些单线程和非事务性约束仅适用于Spring的JMS远程支持。有关Spring对基于JMS的消息传递的丰富支持的信息，请参见[JMS（Java消息服务）](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms)。

服务器和客户端均使用以下接口：

```java
package com.foo;

public interface CheckingAccountService {

    public void cancelAccount(Long accountId);
}
```

在服务器端使用上述接口的以下简单实现：

```java
package com.foo;

public class SimpleCheckingAccountService implements CheckingAccountService {

    public void cancelAccount(Long accountId) {
        System.out.println("Cancelling account [" + accountId + "]");
    }
}
```

以下配置文件包含在客户端和服务器上共享的JMS基础结构Bean：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="connectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://ep-t43:61616"/>
    </bean>

    <bean id="queue" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg value="mmm"/>
    </bean>

</beans>
```

#### 2.4.1. 服务器端配置

在服务器上，您需要公开使用的服务对象 `JmsInvokerServiceExporter`，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="checkingAccountService"
            class="org.springframework.jms.remoting.JmsInvokerServiceExporter">
        <property name="serviceInterface" value="com.foo.CheckingAccountService"/>
        <property name="service">
            <bean class="com.foo.SimpleCheckingAccountService"/>
        </property>
    </bean>

    <bean class="org.springframework.jms.listener.SimpleMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="destination" ref="queue"/>
        <property name="concurrentConsumers" value="3"/>
        <property name="messageListener" ref="checkingAccountService"/>
    </bean>

</beans>
```

```java
package com.foo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Server {

    public static void main(String[] args) throws Exception {
        new ClassPathXmlApplicationContext("com/foo/server.xml", "com/foo/jms.xml");
    }
}
```

#### 2.7.2. 客户端配置

客户端只需要创建一个客户端代理即可实现约定的接口（`CheckingAccountService`）。

下面的示例定义了可以注入到其他客户端对象中的bean（代理负责通过JMS将调用转发到服务器端对象）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="checkingAccountService"
            class="org.springframework.jms.remoting.JmsInvokerProxyFactoryBean">
        <property name="serviceInterface" value="com.foo.CheckingAccountService"/>
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="queue" ref="queue"/>
    </bean>

</beans>
```

```java
package com.foo;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Client {

    public static void main(String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("com/foo/client.xml", "com/foo/jms.xml");
        CheckingAccountService service = (CheckingAccountService) ctx.getBean("checkingAccountService");
        service.cancelAccount(new Long(10));
    }
}
```

## 3. 企业JavaBeans（EJB）集成

本节介绍如何访问EJB

### 3.1.1. 概念

要在本地或远程无状态会话Bean上调用方法，客户机代码通常必须执行JNDI查找以获取（本地或远程）EJB Home对象，然后`create`对该对象使用方法调用以获取实际的（本地或远程）EJB对象。EJB对象。然后在EJB上调用一种或多种方法。

为了避免重复的低级代码，许多EJB应用程序都使用服务定位器和业务委托模式。这些比在整个客户端代码中喷射JNDI查找要好，但是它们的常规实现有很多缺点：

- 通常，使用EJB的代码取决于Service Locator或Business Delegate单例，因此很难进行测试。
- 在不使用业务委托的情况下使用服务定位器模式的情况下，应用程序代码仍然最终必须`create()`在EJB home上调用该方法并处理产生的异常。因此，它仍然与EJB API和EJB编程模型的复杂性联系在一起。
- 实现业务委托模式通常会导致大量的代码重复，我们必须编写许多在EJB上调用相同方法的方法。

Spring的方法是允许创建和使用代理对象（通常在Spring容器内配置），这些代理对象充当无代码的业务委托。除非您在此类代码中实际添加了实际价值，否则您无需在手动编码的业务委托中编写另一个Service Locator，另一个JNDI查找或重复方法。

#### 3.1.2. 访问本地SLSB

假设我们有一个Web控制器，需要使用本地EJB。我们遵循最佳实践，并使用EJB业务方法接口模式，以便EJB的本地接口扩展了非EJB特定的业务方法接口。我们称这种业务方法为接口`MyComponent`。以下示例显示了这样的接口：

```java
public interface MyComponent {
    ...
}
```

使用业务方法接口模式的主要原因之一是确保本地接口中的方法签名与bean实现类之间的同步是自动的。另一个原因是，如果有必要的话，以后可以使我们更轻松地切换到服务的POJO（普通旧Java对象）实现。我们还需要实现本地home接口，并提供一个实现类`SessionBean`和`MyComponent`业务方法接口的实现类。现在，将Web层控制器连接到EJB实现所需的唯一Java编码是`MyComponent` 在控制器上公开类型的setter方法。这会将引用另存为控制器中的实例变量。以下示例显示了如何执行此操作：

```java
private MyComponent myComponent;

public void setMyComponent(MyComponent myComponent) {
    this.myComponent = myComponent;
}
```

随后，我们可以在控制器中的任何业务方法中使用此实例变量。现在，假设我们从Spring容器中获取了控制器对象，我们可以（在相同上下文中）配置一个`LocalStatelessSessionProxyFactoryBean`实例，即EJB代理对象。我们配置代理，并`myComponent`使用以下配置条目设置控制器的 属性：

```xml
<bean id="myComponent"
        class="org.springframework.ejb.access.LocalStatelessSessionProxyFactoryBean">
    <property name="jndiName" value="ejb/myBean"/>
    <property name="businessInterface" value="com.mycom.MyComponent"/>
</bean>

<bean id="myController" class="com.mycom.myController">
    <property name="myComponent" ref="myComponent"/>
</bean>
```

尽管没有强迫您使用AOP概念来欣赏结果，但仍要依靠Spring AOP框架在幕后进行大量工作。该 `myComponent`bean定义创建的EJB，它实现了业务方法接口的代理。EJB本地宿主在启动时被缓存，因此只有一个JNDI查找。每次调用EJB时，代理都会`classname`在本地EJB上调用方法，并在EJB上调用相应的业务方法。

该`myController`bean定义设置`myComponent`控制器类到EJB代理财产。

或者（最好在许多此类代理定义的情况下），考虑`<jee:local-slsb>`在Spring的“ jee”命名空间中使用配置元素。以示例显示了如何执行此操作：

```xml
<jee:local-slsb id="myComponent" jndi-name="ejb/myBean"
        business-interface="com.mycom.MyComponent"/>

<bean id="myController" class="com.mycom.myController">
    <property name="myComponent" ref="myComponent"/>
</bean>
```

这种EJB访问机制极大地简化了应用程序代码。Web层代码（或其他EJB客户端代码）与EJB的使用无关。要用POJO或模拟对象或其他测试存根替换该EJB引用，我们可以更改`myComponent`Bean定义而无需更改一行Java代码。此外，作为应用程序的一部分，我们不必编写任何一行JNDI查找或其他EJB管道代码。

实际应用中的基准和经验表明，这种方法的性能开销（涉及目标EJB的反射调用）是最小的，并且在常规使用中是无法检测到的。请记住，无论如何我们都不希望对EJB进行细粒度的调用，因为与应用程序服务器中的EJB基础结构相关联的成本很高。

关于JNDI查找有一个警告。在bean容器中，此类通常最好用作单例（没有理由使其成为原型）。但是，如果该bean容器预先实例化了单例（与各种XML `ApplicationContext`变体一样），那么如果在EJB容器加载目标EJB之前加载了bean容器，则可能会出现问题。这是因为JNDI查找是在`init()`此类的方法中执行的，然后进行了缓存，但是EJB尚未绑定到目标位置。解决方案是不预先实例化该工厂对象，而是让它在首次使用时创建。在XML容器中，您可以使用`lazy-init`属性来控制它。

尽管大多数Spring用户都不感兴趣，但是那些使用EJB进行编程AOP的人可能要看一下`LocalSlsbInvokerInterceptor`。

#### 3.1.3.  访问远程SLSB

访问远程EJB与访问本地EJB本质上相同，只是使用了 `SimpleRemoteStatelessSessionProxyFactoryBean`or`<jee:remote-slsb>`配置元素。当然，无论是否使用Spring，远程调用语义都适用：调用另一台计算机上另一台VM中的对象上的方法时，有时在使用情况和故障处理方面必须区别对待。

与非Spring方法相比，Spring的EJB客户端支持增加了另一优势。通常，在本地或远程调用EJB之间轻松地来回切换EJB客户端代码是有问题的。这是因为远程接口方法必须声明它们throw `RemoteException`，而客户端代码必须对此进行处理，而本地接口方法则不需要。通常需要修改为需要移至远程EJB的本地EJB编写的客户端代码，以添加对远程异常的处理，为需要移至本地EJB的远程EJB编写的客户端代码可以保持不变，但可以执行以下操作：许多不必要的远程异常处理，或进行修改以删除该代码。使用Spring远程EJB代理，您可以声明不抛出任何异常`RemoteException`在您的业务方法接口和实现EJB代码中，拥有一个完全相同的远程接口（除了它确实抛出`RemoteException`），并且依靠代理来动态地将这两个接口视为相同。也就是说，客户端代码不必处理已检查的`RemoteException`类。`RemoteException`在EJB调用期间抛出的所有实数都将重新抛出为未经检查的`RemoteAccessException`类，该类是的子类`RuntimeException`。然后，您可以在本地EJB或远程EJB（甚至纯Java对象）实现之间随意切换目标服务，而无需了解或关心客户端代码。当然，这是可选的：没有什么可以阻止您`RemoteException`在业务界面中进行声明。

#### 3.1.4. 访问EJB 2.x SLSB与EJB 3 SLSB

通过Spring访问EJB 2.x会话Bean和EJB 3会话Bean在很大程度上是透明的。Spring的EJB访问器（包括`<jee:local-slsb>`和 `<jee:remote-slsb>`设施）在运行时透明地适应实际组件。它们会处理Home接口（如果找到）（EJB 2.x样式），或者在没有Home接口可用时（EJB 3样式）执行直接组件调用。

注意：对于EJB 3会话Bean，您也可以有效地使用`JndiObjectFactoryBean`/ `<jee:jndi-lookup>`，因为公开了完全可用的组件引用以用于在那里的普通JNDI查找。定义显式`<jee:local-slsb>`或`<jee:remote-slsb>` 查找可提供一致且更显式的EJB访问配置。

## 4. JMS（Java消息服务）

Spring提供了一个JMS集成框架，该框架简化了JMS API的使用，就像Spring对JDBC API的集成一样。

JMS可以大致分为两个功能区域，即消息的产生和使用。所述`JmsTemplate`类用于消息生成和同步消息接收操作。对于类似于Java EE的消息驱动bean样式的异步接收，Spring提供了许多消息侦听器容器，可用于创建消息驱动POJO（MDP）。Spring还提供了一种声明式方法来创建消息侦听器。

该`org.springframework.jms.core`软件包提供了使用JMS的核心功能。它包含JMS模板类，该类通过处理资源的创建和释放来简化JMS的使用，就像`JdbcTemplate`JDBC一样。Spring模板类共有的设计原则是提供帮助器方法来执行常用操作，并且对于更复杂的用法，将处理任务的实质委托给用户实现的回调接口。JMS模板遵循相同的设计。这些类提供了各种方便的方法，用于发送消息，同步使用消息以及向用户公开JMS会话和消息生成器。

该`org.springframework.jms.support`软件包提供`JMSException`翻译功能。转换将检查的`JMSException`层次结构转换为未检查的异常的镜像层次结构。如果`javax.jms.JMSException`存在checked的任何提供程序特定的子类，则将此异常包装在unchecked中`UncategorizedJmsException`。

该`org.springframework.jms.support.converter`软件包提供了一个`MessageConverter` 抽象，可以在Java对象和JMS消息之间进行转换。

该`org.springframework.jms.support.destination`软件包提供了用于管理JMS目的地的各种策略，例如为JNDI中存储的目的地提供服务定位器。

该`org.springframework.jms.annotation`软件包通过提供了必要的基础结构来支持注释驱动的侦听器端点`@JmsListener`。

该`org.springframework.jms.config`软件包提供了`jms`名称空间的解析器实现 以及java config支持，以配置侦听器容器和创建侦听器端点。

最后，该`org.springframework.jms.connection`软件包提供了`ConnectionFactory`适用于独立应用程序的实现。它还包含Spring `PlatformTransactionManager`for JMS的实现（巧妙地命名为 `JmsTransactionManager`）。这允许将JMS作为事务资源无缝集成到Spring的事务管理机制中。

|      | 从Spring Framework 5开始，Spring的JMS软件包完全支持JMS 2.0，并要求在运行时提供JMS 2.0 API。我们建议使用JMS 2.0兼容的提供程序。如果您碰巧在系统中使用了较旧的消息代理，则可以尝试为现有代理生成升级到JMS 2.0兼容驱动程序。或者，您也可以尝试在基于JMS 1.1的驱动程序上运行，只需将JMS 2.0 API jar放在类路径上，而仅对驱动程序使用兼容JMS 1.1的API。Spring的JMS支持默认情况下遵守JMS 1.1约定，因此通过相应的配置，它确实支持这种情况。但是，请仅在过渡方案中考虑这一点。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.1

本节描述如何使用Spring的JMS组件。

#### 4.1.1. 使用`JmsTemplate`

`JmsTemplate`类是在JMS芯封装的核心类。由于它在发送或同步接收消息时处理资源的创建和释放，因此它简化了JMS的使用。

```
JmsTemplate`仅使用需求的代码即可实现为回调接口提供清晰定义的高级协定。当中的调用代码提供`MessageCreator`时，回调接口会创建一条消息。为了允许更复杂地使用JMS API，提供JMS会话并公开和 对。`Session``JmsTemplate``SessionCallback``ProducerCallback``Session``MessageProducer
```

JMS API公开了两种类型的发送方法，一种采用交付模式，优先级和生存时间作为服务质量（QOS）参数，另一种不采用QOS参数并使用默认值。由于`JmsTemplate`有许多发送方法，因此设置QOS参数已作为bean属性公开，以避免重复发送方法。同样，使用`setReceiveTimeout`属性设置同步接收调用的超时值。

一些JMS提供程序允许通过配置来管理默认QOS值`ConnectionFactory`。这样做的结果是，对`MessageProducer`实例`send`方法（`send(Destination destination, Message message)`）的调用 使用与JMS规范中指定的QOS默认值不同的QOS默认值。为了提供QOS值的一致管理，`JmsTemplate`必须的，因此，专门启用通过设置布尔值属性使用自己的QOS值 `isExplicitQosEnabled`到`true`。

为了方便起见，`JmsTemplate`还公开了一个基本的请求-答复操作，该操作允许发送消息并等待作为该操作一部分创建的临时队列的答复。

|      | `JmsTemplate`一旦配置，该类的 实例是线程安全的。这很重要，因为这意味着您可以配置的单个实例，`JmsTemplate` 然后将该共享引用安全地注入多个协作者中。需要明确的`JmsTemplate`是，状态是有状态的，因为它保持对的引用 `ConnectionFactory`，但是此状态不是会话状态。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

从Spring Framework 4.1开始，它`JmsMessagingTemplate`是基于`JmsTemplate` 消息传递抽象构建的，并且提供了与消息传递抽象的集成，即 `org.springframework.messaging.Message`。这使您可以创建以通用方式发送的消息。

#### 4.1.2. 连接数

该`JmsTemplate`要求的一个参考`ConnectionFactory`。它`ConnectionFactory` 是JMS规范的一部分，并且是使用JMS的切入点。客户端应用程序将其用作工厂以创建与JMS提供程序的连接，并封装各种配置参数，其中许多是特定于供应商的，例如SSL配置选项。

当在EJB中使用JMS时，供应商提供JMS接口的实现，以便它们可以参与声明式事务管理并执行连接和会话的池化。为了使用此实现，Java EE容器通常要求您将JMS连接工厂声明为`resource-ref`EJB或Servlet部署描述符中的内部。为了确保在`JmsTemplate`EJB内部使用这些功能 ，客户端应用程序应确保其引用的托管实现`ConnectionFactory`。

##### 缓存消息资源

标准API涉及创建许多中间对象。要发送消息，请执行以下“ API”遍历：

```
ConnectionFactory->连接->会话-> MessageProducer->发送
```

在`ConnectionFactory`和`Send`操作之间，创建并销毁了三个中间对象。为了优化资源使用并提高性能，Spring提供了两种实现`ConnectionFactory`。

#####  使用 `SingleConnectionFactory`

Spring提供了`ConnectionFactory`接口的实现，该接口 在所有调用`SingleConnectionFactory`中返回相同的内容`Connection`， `createConnection()`并忽略对的调用`close()`。这对于测试和独立环境非常有用，因此同一连接可用于`JmsTemplate`可能跨越任意数量事务的多个 调用。`SingleConnectionFactory` 引用了`ConnectionFactory`通常来自JNDI的标准。

#####  使用 `CachingConnectionFactory`

该`CachingConnectionFactory`扩展的功能`SingleConnectionFactory` ，并增加了缓存`Session`，`MessageProducer`和`MessageConsumer`实例。初始缓存大小设置为`1`。您可以使用该`sessionCacheSize`属性来增加缓存的会话数。请注意，由于根据会话的确认模式缓存会话，因此实际缓存的会话数大于该数量，因此当`sessionCacheSize`设置为one时，最多可以有四个缓存的会话实例（每个确认模式一个）。`MessageProducer`和`MessageConsumer`实例在其自己的会话中缓存，并且在缓存时还要考虑生产者和使用者的独特属性。MessageProducers根据其目的地进行缓存。基于由目标，选择器，noLocal传递标志和持久订阅名称（如果创建持久使用者）组成的键来缓存MessageConsumers。

####  4.1.3. 目的地管理

作为`ConnectionFactory`实例，目标是可以在JNDI中存储和检索的JMS管理的对象。在配置Spring应用程序上下文时，可以使用JNDI`JndiObjectFactoryBean`工厂类或`<jee:jndi-lookup>`对对象对JMS目标的引用执行依赖项注入。但是，如果应用程序中有大量目标，或者JMS提供程序具有独特的高级目标管理功能，则此策略通常很麻烦。这种高级目标管理的示例包括动态目标的创建或对目标的分层名称空间的支持。将`JmsTemplate`目标名称的解析委托给实现`DestinationResolver`接口的JMS目标对象 。`DynamicDestinationResolver`是由...使用的默认实现`JmsTemplate`并适应解析的动态目标。`JndiDestinationResolver`还提供A 充当JNDI中包含的目的地的服务定位器，并且可以选择回退到中包含的行为 `DynamicDestinationResolver`。

通常，仅在运行时才知道JMS应用程序中使用的目的地，因此，在部署应用程序时无法通过管理方式创建。这通常是因为在交互的系统组件之间存在共享的应用程序逻辑，这些组件根据已知的命名约定在运行时创建目标。即使创建动态目标不属于JMS规范的一部分，但大多数供应商都提供了此功能。动态目标是使用用户定义的名称创建的，该名称将它们与临时目标区分开来，并且通常未在JNDI中注册。提供商之间使用的用于创建动态目标的API有所不同，因为与目标关联的属性是特定于供应商的。然而，`TopicSession` `createTopic(String topicName)`或`QueueSession` `createQueue(String queueName)`使用默认目标属性创建新目标的方法。`DynamicDestinationResolver`然后，根据供应商的实现，还可以创建一个物理目标，而不是仅解决一个目标。

布尔值属性`pubSubDomain`用于使用`JmsTemplate`正在使用的JMS域的知识来配置。默认情况下，此属性的值为false，表示`Queues`将使用点对点域。（由所使用`JmsTemplate`）此属性通过`DestinationResolver`接口的实现确定动态目标解析的行为。

您还可以`JmsTemplate`通过属性将设置为具有默认目的地`defaultDestination`。默认目标是带有不引用特定目标的发送和接收操作。

#### 4.1.4. 消息侦听器容器

在EJB世界中，JMS消息最常见的用途之一就是驱动消息驱动的bean（MDB）。Spring提供了一种解决方案，以不将用户绑定到EJB容器的方式创建消息驱动的POJO（MDP）。（有关Spring对MDP支持的详细介绍，请参见[异步接收：消息驱动的POJO](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-receiving-async)。）从Spring Framework 4.1开始，可以使用以下方法对端点方法进行注释`@JmsListener` -有关更多详细信息，请参见[注释驱动的侦听器端点](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-annotated)。

消息侦听器容器用于从JMS消息队列接收消息并驱动`MessageListener`注入到其中的消息。侦听器容器负责消息接收的所有线程，并分派到侦听器中进行处理。消息侦听器容器是MDP与消息传递提供程序之间的中介，并负责注册接收消息，参与事务，资源获取和释放，异常转换等。这使您可以编写与接收消息（并可能响应消息）相关的（可能很复杂的）业务逻辑，并将样板JMS基础结构问题委托给框架。

Spring附带了两个标准的JMS消息侦听器容器，每个容器都有其专门的功能集。

- [`SimpleMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-mdp-simple)

- [`DefaultMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-mdp-default)

  

##### 使用 `SimpleMessageListenerContainer`

此消息侦听器容器是两种标准样式中的简单容器。它在启动时创建固定数量的JMS会话和使用者，使用标准JMS`MessageConsumer.setMessageListener()`方法注册侦听器，然后将其留给JMS提供者以执行侦听器回调。此变体不允许动态适应运行时需求或参与外部管理的事务。在兼容性方面，它非常接近独立JMS规范的精神，但通常与Java EE的JMS限制不兼容。

|      | 尽管`SimpleMessageListenerContainer`不允许参与外部管理的事务，但它确实支持本机JMS事务。要启用此功能，可以将`sessionTransacted`标志切换为，`true`或者在XML名称空间中将`acknowledge`属性设置 为`transacted`。从您的侦听器抛出的异常会导致回滚，并重新传递消息。或者，考虑使用 `CLIENT_ACKNOWLEDGE`模式，该模式在出现异常的情况下也提供重新交付但不使用事务处理的`Session`实例，因此`Session`在事务协议中不包括任何其他 操作（例如发送响应消息）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 默认`AUTO_ACKNOWLEDGE`模式不能提供适当的可靠性保证。当侦听器执行失败时（由于提供者会在侦听器调用后自动确认每条消息，并且没有异常要传播到提供者），或者在侦听器容器关闭时（您可以通过设置`acceptMessagesWhileStopping`标志进行配置），消息可能会丢失。确保出于可靠性需求（例如，为了可靠的队列处理和持久的主题订阅）使用事务处理的会话。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 使用 `DefaultMessageListenerContainer`

大多数情况下使用此消息侦听器容器。与之相反 `SimpleMessageListenerContainer`，此容器变量允许动态适应运行时需求，并且能够参与外部管理的事务。配置为时，每个接收到的消息都会在XA事务中注册 `JtaTransactionManager`。结果，处理可以利用XA事务语义。该侦听器容器在JMS提供程序的低要求，高级功能（例如参与外部管理的事务）以及与Java EE环境的兼容性之间取得了良好的平衡。

您可以自定义容器的缓存级别。请注意，当未启用缓存时，将为每个消息接收创建一个新的连接和一个新的会话。将此内容与具有高负载的非持久订阅结合使用可能会导致消息丢失。在这种情况下，请确保使用适当的缓存级别。

当代理崩溃时，此容器还具有可恢复的功能。默认情况下，一个简单的`BackOff`实现每五秒钟重试一次。您可以`BackOff`为更细粒度的恢复选项指定自定义实现。有关`ExponentialBackOff`示例，请参见api-spring-framework / util / backoff / ExponentialBackOff.html [ ]。

|      | 与其兄弟（[`SimpleMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-mdp-simple)） 一样，它`DefaultMessageListenerContainer`支持本机JMS事务，并允许自定义确认模式。如果对您的方案可行，则强烈建议在外部管理的事务上使用此方法，也就是说，如果JVM死亡，您可以偶尔接收重复的消息。业务逻辑中的自定义重复消息检测步骤可以解决这种情况-例如，以业务实体存在检查或协议表检查的形式。任何这样的安排是显著比其他更有效的：用一个XA事务包裹整个处理（通过在配置 `DefaultMessageListenerContainer`有`JtaTransactionManager`）以覆盖所述JMS消息的接收，以及业务逻辑在您的消息侦听器的执行（包括数据库操作等）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 默认`AUTO_ACKNOWLEDGE`模式不能提供适当的可靠性保证。当侦听器执行失败时（由于提供者会在侦听器调用后自动确认每条消息，并且没有异常要传播到提供者），或者在侦听器容器关闭时（您可以通过设置`acceptMessagesWhileStopping`标志进行配置），消息可能会丢失。确保出于可靠性需求（例如，为了可靠的队列处理和持久的主题订阅）使用事务处理的会话。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

####  4.1.5. 交易管理

Spring提供了一个`JmsTransactionManager`管理单个JMS事务的 `ConnectionFactory`。这使JMS应用程序可以利用Spring的托管事务功能，如 [“数据访问”一章的“事务管理”部分所述](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction)。在`JmsTransactionManager`执行本地资源交易，结合从指定的JMS连接/会话对`ConnectionFactory`给线程。 `JmsTemplate`自动检测此类交易资源并相应地对其进行操作。

在Java EE环境中，`ConnectionFactory`池连接和实例实例，因此这些资源可有效地跨事务重用。在独立环境中，使用Spring的`SingleConnectionFactory`结果在共享的JMS中`Connection`，每个事务都有自己独立的`Session`。或者，考虑使用提供程序专用的池适配器，例如ActiveMQ的`PooledConnectionFactory` 类。

您还可以`JmsTemplate`与`JtaTransactionManager`和具有XA功能的JMS一起使用 `ConnectionFactory`以执行分布式事务。请注意，这需要使用JTA事务管理器以及正确的XA配置的ConnectionFactory。（检查您的Java EE服务器或JMS提供程序的文档。）

跨托管和非托管的事务环境下重用代码可以使用JMS API来创建一个混乱的时候`Session`从一个`Connection`。这是因为JMS API只有一种工厂方法可以创建`Session`，并且它需要事务和确认模式的值。在托管环境中，设置这些值是环境的事务基础结构的责任，因此，供应商对JMS Connection的包装将忽略这些值。当您使用`JmsTemplate` 在非托管环境中，您可以通过使用属性来指定这些值`sessionTransacted`和`sessionAcknowledgeMode`。当使用 `PlatformTransactionManager`with时`JmsTemplate`，模板总是被赋予事务性JMS `Session`。

### 4.2. 发送信息

将`JmsTemplate`包含许多方便的方法来发送消息。发送方法通过使用`javax.jms.Destination`对象指定目的地，其他方法通过`String`在JNDI查找中使用指定目的地。`send`不使用目标参数的方法使用默认目标。

以下示例使用`MessageCreator`回调从提供的`Session`对象创建文本消息：

```java
import javax.jms.ConnectionFactory;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Queue;
import javax.jms.Session;

import org.springframework.jms.core.MessageCreator;
import org.springframework.jms.core.JmsTemplate;

public class JmsQueueSender {

    private JmsTemplate jmsTemplate;
    private Queue queue;

    public void setConnectionFactory(ConnectionFactory cf) {
        this.jmsTemplate = new JmsTemplate(cf);
    }

    public void setQueue(Queue queue) {
        this.queue = queue;
    }

    public void simpleSend() {
        this.jmsTemplate.send(this.queue, new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                return session.createTextMessage("hello queue world");
            }
        });
    }
}
```

在前面的示例中，`JmsTemplate`通过将引用传递给来构造 `ConnectionFactory`。作为替代方案，提供了一个零参数的构造函数，该构造函数 `connectionFactory`可用于以JavaBean样式（使用`BeanFactory`Java或纯Java代码）构造实例。另外，考虑从Spring的`JmsGatewaySupport`便捷基类派生，该基类为JMS配置提供了预构建的bean属性。

该`send(String destinationName, MessageCreator creator)`方法使您可以使用目标的字符串名称发送消息。如果这些名称已在JNDI中注册，则应将`destinationResolver`模板的属性设置为的实例 `JndiDestinationResolver`。

如果您创建`JmsTemplate`并指定了默认目标，则会向该目标 `send(MessageCreator c)`发送一条消息。

1. 1. 
      4.3。接收讯息
      1. 
      2. 
      3. 
      4. 
      5. 
   2. [4.4。支持JCA消息端点](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-jca-message-endpoint-manager)
   3. 4.5。注释驱动的侦听器端点
      1. 
      2. 
      3. 
      4. 
   4. [4.6。JMS命名空间支持](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-namespace)
2. \5. JMX
   1. 
      1. 
      2. 
      3. 
      4. 
      5. 
   2. 
      1. 
      2. 
      3. 
      4. 
      5. 
      6. 
   3. 
      1. 
      2. 
      3. 
   4. 
      1. 
      2. 
      3. 
   5. 
   6. 
      1. 
      2. 
   7. 
3. 6.电子邮件
   1. 
      1. 
      2. 
   2. 
      1. 
         1. 
         2. 
      2. 
4. 7.任务执行和计划
   1. 
      1. 
      2. 
   2. 
      1. 
      2. 
      3. 
   3. 
      1. 
      2. 
      3. 
      4. 
      5. 
   4. 
      1. 
      2. 
      3. 
   5. 
      1. 
      2. 
      3. 
5. 8.缓存抽象
   1. 
   2. 
      1. 
         1. 
         2. 
         3. 
         4. 
         5. 
         6. 
         7. 
      2. 
      3. 
      4. 
      5. 
      6. 
      7. 
   3. 
      1. 
      2. 
   4. 
   5. 
      1. 
      2. 
      3. 
      4. 
      5. 
      6. 
   6. 
   7. 
6. 9.附录
   1. 
      1. 
         1. 
         2. 
         3. 
         4. 
         5. 
         6. 
         7. 
      2. 
      3. 
      4. 

参考文档的这一部分涵盖了Spring Framework与多种技术的集成。

## 1. REST端点

The Spring Framework provides two choices for making calls to REST endpoints:

- [`RestTemplate`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-resttemplate): The original Spring REST client with a synchronous, template method API.
- [WebClient](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client): a non-blocking, reactive alternative that supports both synchronous and asynchronous as well as streaming scenarios.

|      | As of 5.0 the `RestTemplate` is in maintenance mode, with only minor requests for changes and bugs to be accepted going forward. Please, consider using the [WebClient](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client) which offers a more modern API and supports sync, async, and streaming scenarios. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.1. `RestTemplate`

The `RestTemplate` provides a higher level API over HTTP client libraries. It makes it easy to invoke REST endpoints in a single line. It exposes the following groups of overloaded methods:

| Method group      | Description                                                  |
| :---------------- | :----------------------------------------------------------- |
| `getForObject`    | Retrieves a representation via GET.                          |
| `getForEntity`    | Retrieves a `ResponseEntity` (that is, status, headers, and body) by using GET. |
| `headForHeaders`  | Retrieves all headers for a resource by using HEAD.          |
| `postForLocation` | Creates a new resource by using POST and returns the `Location` header from the response. |
| `postForObject`   | Creates a new resource by using POST and returns the representation from the response. |
| `postForEntity`   | Creates a new resource by using POST and returns the representation from the response. |
| `put`             | Creates or updates a resource by using PUT.                  |
| `patchForObject`  | Updates a resource by using PATCH and returns the representation from the response. Note that the JDK `HttpURLConnection` does not support the `PATCH`, but Apache HttpComponents and others do. |
| `delete`          | Deletes the resources at the specified URI by using DELETE.  |
| `optionsForAllow` | Retrieves allowed HTTP methods for a resource by using ALLOW. |
| `exchange`        | More generalized (and less opinionated) version of the preceding methods that provides extra flexibility when needed. It accepts a `RequestEntity` (including HTTP method, URL, headers, and body as input) and returns a `ResponseEntity`.These methods allow the use of `ParameterizedTypeReference` instead of `Class` to specify a response type with generics. |
| `execute`         | The most generalized way to perform a request, with full control over request preparation and response extraction through callback interfaces. |

#### 1.1.1. Initialization

The default constructor uses `java.net.HttpURLConnection` to perform requests. You can switch to a different HTTP library with an implementation of `ClientHttpRequestFactory`. There is built-in support for the following:

- Apache HttpComponents
- Netty
- OkHttp

For example, to switch to Apache HttpComponents, you can use the following:

```java
RestTemplate template = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
```

Each `ClientHttpRequestFactory` exposes configuration options specific to the underlying HTTP client library — for example, for credentials, connection pooling, and other details.

|      | Note that the `java.net` implementation for HTTP requests can raise an exception when accessing the status of a response that represents an error (such as 401). If this is an issue, switch to another HTTP client library. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### URIs

Many of the `RestTemplate` methods accept a URI template and URI template variables, either as a `String` variable argument, or as `Map<String,String>`.

The following example uses a `String` variable argument:

```java
String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/bookings/{booking}", String.class, "42", "21");
```

The following example uses a `Map<String, String>`:

```java
Map<String, String> vars = Collections.singletonMap("hotel", "42");

String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/rooms/{hotel}", String.class, vars);
```

Keep in mind URI templates are automatically encoded, as the following example shows:

```java
restTemplate.getForObject("https://example.com/hotel list", String.class);

// Results in request to "https://example.com/hotel%20list"
```

You can use the `uriTemplateHandler` property of `RestTemplate` to customize how URIs are encoded. Alternatively, you can prepare a `java.net.URI` and pass it into one of the `RestTemplate` methods that accepts a `URI`.

For more details on working with and encoding URIs, see [URI Links](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-uri-building).

##### Headers

You can use the `exchange()` methods to specify request headers, as the following example shows:

```java
String uriTemplate = "https://example.com/hotels/{hotel}";
URI uri = UriComponentsBuilder.fromUriString(uriTemplate).build(42);

RequestEntity<Void> requestEntity = RequestEntity.get(uri)
        .header(("MyRequestHeader", "MyValue")
        .build();

ResponseEntity<String> response = template.exchange(requestEntity, String.class);

String responseHeader = response.getHeaders().getFirst("MyResponseHeader");
String body = response.getBody();
```

You can obtain response headers through many `RestTemplate` method variants that return `ResponseEntity`.

#### 1.1.2. Body

Objects passed into and returned from `RestTemplate` methods are converted to and from raw content with the help of an `HttpMessageConverter`.

On a POST, an input object is serialized to the request body, as the following example shows:

```
URI location = template.postForLocation("https://example.com/people", person);
```

You need not explicitly set the Content-Type header of the request. In most cases, you can find a compatible message converter based on the source `Object` type, and the chosen message converter sets the content type accordingly. If necessary, you can use the `exchange` methods to explicitly provide the `Content-Type` request header, and that, in turn, influences what message converter is selected.

On a GET, the body of the response is deserialized to an output `Object`, as the following example shows:

```
Person person = restTemplate.getForObject("https://example.com/people/{id}", Person.class, 42);
```

The `Accept` header of the request does not need to be explicitly set. In most cases, a compatible message converter can be found based on the expected response type, which then helps to populate the `Accept` header. If necessary, you can use the `exchange` methods to provide the `Accept` header explicitly.

By default, `RestTemplate` registers all built-in [message converters](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-message-conversion), depending on classpath checks that help to determine what optional conversion libraries are present. You can also set the message converters to use explicitly.

#### 1.1.3. Message Conversion

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs)

The `spring-web` module contains the `HttpMessageConverter` contract for reading and writing the body of HTTP requests and responses through `InputStream` and `OutputStream`. `HttpMessageConverter` instances are used on the client side (for example, in the `RestTemplate`) and on the server side (for example, in Spring MVC REST controllers).

Concrete implementations for the main media (MIME) types are provided in the framework and are, by default, registered with the `RestTemplate` on the client side and with `RequestMethodHandlerAdapter` on the server side (see [Configuring Message Converters](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-message-converters)).

The implementations of `HttpMessageConverter` are described in the following sections. For all converters, a default media type is used, but you can override it by setting the `supportedMediaTypes` bean property. The following table describes each implementation:

| MessageConverter                         | Description                                                  |
| :--------------------------------------- | :----------------------------------------------------------- |
| `StringHttpMessageConverter`             | An `HttpMessageConverter` implementation that can read and write `String` instances from the HTTP request and response. By default, this converter supports all text media types (`text/*`) and writes with a `Content-Type` of `text/plain`. |
| `FormHttpMessageConverter`               | An `HttpMessageConverter` implementation that can read and write form data from the HTTP request and response. By default, this converter reads and writes the `application/x-www-form-urlencoded` media type. Form data is read from and written into a `MultiValueMap<String, String>`. The converter can also write (but not read) multipart data read from a `MultiValueMap<String, Object>`. By default, `multipart/form-data` is supported. As of Spring Framework 5.2, additional multipart subtypes can be supported for writing form data. Consult the javadoc for `FormHttpMessageConverter` for further details. |
| `ByteArrayHttpMessageConverter`          | An `HttpMessageConverter` implementation that can read and write byte arrays from the HTTP request and response. By default, this converter supports all media types (`*/*`) and writes with a `Content-Type` of `application/octet-stream`. You can override this by setting the `supportedMediaTypes` property and overriding `getContentType(byte[])`. |
| `MarshallingHttpMessageConverter`        | An `HttpMessageConverter` implementation that can read and write XML by using Spring’s `Marshaller` and `Unmarshaller` abstractions from the `org.springframework.oxm` package. This converter requires a `Marshaller` and `Unmarshaller` before it can be used. You can inject these through constructor or bean properties. By default, this converter supports `text/xml` and `application/xml`. |
| `MappingJackson2HttpMessageConverter`    | An `HttpMessageConverter` implementation that can read and write JSON by using Jackson’s `ObjectMapper`. You can customize JSON mapping as needed through the use of Jackson’s provided annotations. When you need further control (for cases where custom JSON serializers/deserializers need to be provided for specific types), you can inject a custom `ObjectMapper` through the `ObjectMapper` property. By default, this converter supports `application/json`. |
| `MappingJackson2XmlHttpMessageConverter` | An `HttpMessageConverter` implementation that can read and write XML by using [Jackson XML](https://github.com/FasterXML/jackson-dataformat-xml) extension’s `XmlMapper`. You can customize XML mapping as needed through the use of JAXB or Jackson’s provided annotations. When you need further control (for cases where custom XML serializers/deserializers need to be provided for specific types), you can inject a custom `XmlMapper` through the `ObjectMapper` property. By default, this converter supports `application/xml`. |
| `SourceHttpMessageConverter`             | An `HttpMessageConverter` implementation that can read and write `javax.xml.transform.Source` from the HTTP request and response. Only `DOMSource`, `SAXSource`, and `StreamSource` are supported. By default, this converter supports `text/xml` and `application/xml`. |
| `BufferedImageHttpMessageConverter`      | An `HttpMessageConverter` implementation that can read and write `java.awt.image.BufferedImage` from the HTTP request and response. This converter reads and writes the media type supported by the Java I/O API. |

#### 1.1.4. Jackson JSON Views

You can specify a [Jackson JSON View](https://www.baeldung.com/jackson-json-view-annotation) to serialize only a subset of the object properties, as the following example shows:

```java
MappingJacksonValue value = new MappingJacksonValue(new User("eric", "7!jd#h23"));
value.setSerializationView(User.WithoutPasswordView.class);

RequestEntity<MappingJacksonValue> requestEntity =
    RequestEntity.post(new URI("https://example.com/user")).body(value);

ResponseEntity<String> response = template.exchange(requestEntity, String.class);
```

##### Multipart

To send multipart data, you need to provide a `MultiValueMap<String, Object>` whose values may be an `Object` for part content, a `Resource` for a file part, or an `HttpEntity` for part content with headers. For example:

```java
MultiValueMap<String, Object> parts = new LinkedMultiValueMap<>();

parts.add("fieldPart", "fieldValue");
parts.add("filePart", new FileSystemResource("...logo.png"));
parts.add("jsonPart", new Person("Jason"));

HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_XML);
parts.add("xmlPart", new HttpEntity<>(myBean, headers));
```

In most cases, you do not have to specify the `Content-Type` for each part. The content type is determined automatically based on the `HttpMessageConverter` chosen to serialize it or, in the case of a `Resource` based on the file extension. If necessary, you can explicitly provide the `MediaType` with an `HttpEntity` wrapper.

Once the `MultiValueMap` is ready, you can pass it to the `RestTemplate`, as show below:

```java
MultiValueMap<String, Object> parts = ...;
template.postForObject("https://example.com/upload", parts, Void.class);
```

If the `MultiValueMap` contains at least one non-`String` value, the `Content-Type` is set to `multipart/form-data` by the `FormHttpMessageConverter`. If the `MultiValueMap` has `String` values the `Content-Type` is defaulted to `application/x-www-form-urlencoded`. If necessary the `Content-Type` may also be set explicitly.

### 1.2. Using `AsyncRestTemplate` (Deprecated)

The `AsyncRestTemplate` is deprecated. For all use cases where you might consider using `AsyncRestTemplate`, use the [WebClient](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client) instead.

## 2. Remoting and Web Services

Spring provides support for remoting with various technologies. The remoting support eases the development of remote-enabled services, implemented via Java interfaces and objects as input and output. Currently, Spring supports the following remoting technologies:

- [Java Web Services](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-web-services): Spring provides remoting support for web services through JAX-WS.
- [AMQP](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-amqp): Remoting via AMQP as the underlying protocol is supported by the separate Spring AMQP project.

|      | As of Spring Framework 5.3, support for several remoting technologies is now deprecated for security reasons and broader industry support. Supporting infrastructure will be removed from Spring Framework for its next major release. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The following remoting technologies are now deprecated and will not be replaced:

- [Remote Method Invocation (RMI)](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-rmi): Through the use of `RmiProxyFactoryBean` and `RmiServiceExporter`, Spring supports both traditional RMI (with `java.rmi.Remote` interfaces and `java.rmi.RemoteException`) and transparent remoting through RMI invokers (with any Java interface).
- [Spring HTTP Invoker (Deprecated)](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-httpinvoker): Spring provides a special remoting strategy that allows for Java serialization though HTTP, supporting any Java interface (as the RMI invoker does). The corresponding support classes are `HttpInvokerProxyFactoryBean` and `HttpInvokerServiceExporter`.
- [Hessian](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-caucho-protocols-hessian): By using Spring’s `HessianProxyFactoryBean` and the `HessianServiceExporter`, you can transparently expose your services through the lightweight binary HTTP-based protocol provided by Caucho.
- [JMS (Deprecated)](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-jms): Remoting via JMS as the underlying protocol is supported through the `JmsInvokerServiceExporter` and `JmsInvokerProxyFactoryBean` classes in the `spring-jms` module.

While discussing the remoting capabilities of Spring, we use the following domain model and corresponding services:

```java
public class Account implements Serializable{

    private String name;

    public String getName(){
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
public interface AccountService {

    public void insertAccount(Account account);

    public List<Account> getAccounts(String name);
}
// the implementation doing nothing at the moment
public class AccountServiceImpl implements AccountService {

    public void insertAccount(Account acc) {
        // do something...
    }

    public List<Account> getAccounts(String name) {
        // do something...
    }
}
```

This section starts by exposing the service to a remote client by using RMI and talk a bit about the drawbacks of using RMI. It then continues with an example that uses Hessian as the protocol.

### 2.1. AMQP

Remoting via AMQP as the underlying protocol is supported in the Spring AMQP project. For further details please visit the [Spring Remoting](https://docs.spring.io/spring-amqp/docs/current/reference/html/#remoting) section of the Spring AMQP reference.

|      | Auto-detection is not implemented for remote interfacesThe main reason why auto-detection of implemented interfaces does not occur for remote interfaces is to avoid opening too many doors to remote callers. The target object might implement internal callback interfaces, such as `InitializingBean` or `DisposableBean` which one would not want to expose to callers.Offering a proxy with all interfaces implemented by the target usually does not matter in the local case. However, when you export a remote service, you should expose a specific service interface, with specific operations intended for remote usage. Besides internal callback interfaces, the target might implement multiple business interfaces, with only one of them intended for remote exposure. For these reasons, we require such a service interface to be specified.This is a trade-off between configuration convenience and the risk of accidental exposure of internal methods. Always specifying a service interface is not too much effort and puts you on the safe side regarding controlled exposure of specific methods. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.2. Considerations when Choosing a Technology

Each and every technology presented here has its drawbacks. When choosing a technology, you should carefully consider your needs, the services you expose, and the objects you send over the wire.

When using RMI, you cannot access the objects through the HTTP protocol, unless you tunnel the RMI traffic. RMI is a fairly heavy-weight protocol, in that it supports full-object serialization, which is important when you use a complex data model that needs serialization over the wire. However, RMI-JRMP is tied to Java clients. It is a Java-to-Java remoting solution.

Spring’s HTTP invoker is a good choice if you need HTTP-based remoting but also rely on Java serialization. It shares the basic infrastructure with RMI invokers but uses HTTP as transport. Note that HTTP invokers are not limited only to Java-to-Java remoting but also to Spring on both the client and the server side. (The latter also applies to Spring’s RMI invoker for non-RMI interfaces.)

Hessian might provide significant value when operating in a heterogeneous environment, because they explicitly allow for non-Java clients. However, non-Java support is still limited. Known issues include the serialization of Hibernate objects in combination with lazily-initialized collections. If you have such a data model, consider using RMI or HTTP invokers instead of Hessian.

JMS can be useful for providing clusters of services and letting the JMS broker take care of load balancing, discovery, and auto-failover. By default, Java serialization is used for JMS remoting, but the JMS provider could use a different mechanism for the wire formatting, such as XStream to let servers be implemented in other technologies.

Last but not least, EJB has an advantage over RMI, in that it supports standard role-based authentication and authorization and remote transaction propagation. It is possible to get RMI invokers or HTTP invokers to support security context propagation as well, although this is not provided by core Spring. Spring offers only appropriate hooks for plugging in third-party or custom solutions.

### 2.3. Java Web Services

Spring provides full support for the standard Java web services APIs:

- Exposing web services using JAX-WS
- Accessing web services using JAX-WS

In addition to stock support for JAX-WS in Spring Core, the Spring portfolio also features [Spring Web Services](http://www.springframework.org/spring-ws), which is a solution for contract-first, document-driven web services — highly recommended for building modern, future-proof web services.

#### 2.3.1. Exposing Servlet-based Web Services by Using JAX-WS

Spring provides a convenient base class for JAX-WS servlet endpoint implementations: `SpringBeanAutowiringSupport`. To expose our `AccountService`, we extend Spring’s `SpringBeanAutowiringSupport` class and implement our business logic here, usually delegating the call to the business layer. We use Spring’s `@Autowired` annotation to express such dependencies on Spring-managed beans. The following example shows our class that extends `SpringBeanAutowiringSupport`:

```java
/**
 * JAX-WS compliant AccountService implementation that simply delegates
 * to the AccountService implementation in the root web application context.
 *
 * This wrapper class is necessary because JAX-WS requires working with dedicated
 * endpoint classes. If an existing service needs to be exported, a wrapper that
 * extends SpringBeanAutowiringSupport for simple Spring bean autowiring (through
 * the @Autowired annotation) is the simplest JAX-WS compliant way.
 *
 * This is the class registered with the server-side JAX-WS implementation.
 * In the case of a Java EE server, this would simply be defined as a servlet
 * in web.xml, with the server detecting that this is a JAX-WS endpoint and reacting
 * accordingly. The servlet name usually needs to match the specified WS service name.
 *
 * The web service engine manages the lifecycle of instances of this class.
 * Spring bean references will just be wired in here.
 */
import org.springframework.web.context.support.SpringBeanAutowiringSupport;

@WebService(serviceName="AccountService")
public class AccountServiceEndpoint extends SpringBeanAutowiringSupport {

    @Autowired
    private AccountService biz;

    @WebMethod
    public void insertAccount(Account acc) {
        biz.insertAccount(acc);
    }

    @WebMethod
    public Account[] getAccounts(String name) {
        return biz.getAccounts(name);
    }
}
```

Our `AccountServiceEndpoint` needs to run in the same web application as the Spring context to allow for access to Spring’s facilities. This is the case by default in Java EE environments, using the standard contract for JAX-WS servlet endpoint deployment. See the various Java EE web service tutorials for details.

#### 2.3.2. Exporting Standalone Web Services by Using JAX-WS

The built-in JAX-WS provider that comes with Oracle’s JDK supports exposure of web services by using the built-in HTTP server that is also included in the JDK. Spring’s `SimpleJaxWsServiceExporter` detects all `@WebService`-annotated beans in the Spring application context and exports them through the default JAX-WS server (the JDK HTTP server).

In this scenario, the endpoint instances are defined and managed as Spring beans themselves. They are registered with the JAX-WS engine, but their lifecycle is up to the Spring application context. This means that you can apply Spring functionality (such as explicit dependency injection) to the endpoint instances. Annotation-driven injection through `@Autowired` works as well. The following example shows how to define these beans:

```xml
<bean class="org.springframework.remoting.jaxws.SimpleJaxWsServiceExporter">
    <property name="baseAddress" value="http://localhost:8080/"/>
</bean>

<bean id="accountServiceEndpoint" class="example.AccountServiceEndpoint">
    ...
</bean>

...
```

The `AccountServiceEndpoint` can but does not have to derive from Spring’s `SpringBeanAutowiringSupport`, since the endpoint in this example is a fully Spring-managed bean. This means that the endpoint implementation can be as follows (without any superclass declared — and Spring’s `@Autowired` configuration annotation is still honored):

```java
@WebService(serviceName="AccountService")
public class AccountServiceEndpoint {

    @Autowired
    private AccountService biz;

    @WebMethod
    public void insertAccount(Account acc) {
        biz.insertAccount(acc);
    }

    @WebMethod
    public List<Account> getAccounts(String name) {
        return biz.getAccounts(name);
    }
}
```

#### 2.3.3. Exporting Web Services by Using JAX-WS RI’s Spring Support

Oracle’s JAX-WS RI, developed as part of the GlassFish project, ships Spring support as part of its JAX-WS Commons project. This allows for defining JAX-WS endpoints as Spring-managed beans, similar to the standalone mode discussed in the [previous section](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-web-services-jaxws-export-standalone) — but this time in a Servlet environment.

|      | This is not portable in a Java EE environment. It is mainly intended for non-EE environments, such as Tomcat, that embed the JAX-WS RI as part of the web application. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The differences from the standard style of exporting servlet-based endpoints are that the lifecycle of the endpoint instances themselves are managed by Spring and that there is only one JAX-WS servlet defined in `web.xml`. With the standard Java EE style (as shown earlier), you have one servlet definition per service endpoint, with each endpoint typically delegating to Spring beans (through the use of `@Autowired`, as shown earlier).

See https://jax-ws-commons.java.net/spring/ for details on setup and usage style.

#### 2.3.4. Accessing Web Services by Using JAX-WS

Spring provides two factory beans to create JAX-WS web service proxies, namely `LocalJaxWsServiceFactoryBean` and `JaxWsPortProxyFactoryBean`. The former can return only a JAX-WS service class for us to work with. The latter is the full-fledged version that can return a proxy that implements our business service interface. In the following example, we use `JaxWsPortProxyFactoryBean` to create a proxy for the `AccountService` endpoint (again):

```xml
<bean id="accountWebService" class="org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean">
    <property name="serviceInterface" value="example.AccountService"/> 
    <property name="wsdlDocumentUrl" value="http://localhost:8888/AccountServiceEndpoint?WSDL"/>
    <property name="namespaceUri" value="https://example/"/>
    <property name="serviceName" value="AccountService"/>
    <property name="portName" value="AccountServiceEndpointPort"/>
</bean>
```

|      | Where `serviceInterface` is our business interface that the clients use. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`wsdlDocumentUrl` is the URL for the WSDL file. Spring needs this at startup time to create the JAX-WS Service. `namespaceUri` corresponds to the `targetNamespace` in the .wsdl file. `serviceName` corresponds to the service name in the .wsdl file. `portName` corresponds to the port name in the .wsdl file.

Accessing the web service is easy, as we have a bean factory for it that exposes it as an interface called `AccountService`. The following example shows how we can wire this up in Spring:

```xml
<bean id="client" class="example.AccountClientImpl">
    ...
    <property name="service" ref="accountWebService"/>
</bean>
```

From the client code, we can access the web service as if it were a normal class, as the following example shows:

```java
public class AccountClientImpl {

    private AccountService service;

    public void setService(AccountService service) {
        this.service = service;
    }

    public void foo() {
        service.insertAccount(...);
    }
}
```

|      | The above is slightly simplified in that JAX-WS requires endpoint interfaces and implementation classes to be annotated with `@WebService`, `@SOAPBinding` etc annotations. This means that you cannot (easily) use plain Java interfaces and implementation classes as JAX-WS endpoint artifacts; you need to annotate them accordingly first. Check the JAX-WS documentation for details on those requirements. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.4. RMI (Deprecated)

|      | As of Spring Framework 5.3, RMI support is deprecated and will not be replaced. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

By using Spring’s support for RMI, you can transparently expose your services through the RMI infrastructure. After having this set up, you basically have a configuration similar to remote EJBs, except for the fact that there is no standard support for security context propagation or remote transaction propagation. Spring does provide hooks for such additional invocation context when you use the RMI invoker, so you can, for example, plug in security frameworks or custom security credentials.

#### 2.4.1. Exporting the Service by Using `RmiServiceExporter`

Using the `RmiServiceExporter`, we can expose the interface of our AccountService object as RMI object. The interface can be accessed by using `RmiProxyFactoryBean`, or via plain RMI in case of a traditional RMI service. The `RmiServiceExporter` explicitly supports the exposing of any non-RMI services via RMI invokers.

We first have to set up our service in the Spring container. The following example shows how to do so:

```xml
<bean id="accountService" class="example.AccountServiceImpl">
    <!-- any additional properties, maybe a DAO? -->
</bean>
```

Next, we have to expose our service by using `RmiServiceExporter`. The following example shows how to do so:

```xml
<bean class="org.springframework.remoting.rmi.RmiServiceExporter">
    <!-- does not necessarily have to be the same name as the bean to be exported -->
    <property name="serviceName" value="AccountService"/>
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
    <!-- defaults to 1099 -->
    <property name="registryPort" value="1199"/>
</bean>
```

In the preceding example, we override the port for the RMI registry. Often, your application server also maintains an RMI registry, and it is wise to not interfere with that one. Furthermore, the service name is used to bind the service. So, in the preceding example, the service is bound at `'rmi://HOST:1199/AccountService'`. We use this URL later on to link in the service at the client side.

|      | The `servicePort` property has been omitted (it defaults to 0). This means that an anonymous port is used to communicate with the service. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.4.2. Linking in the Service at the Client

Our client is a simple object that uses the `AccountService` to manage accounts, as the following example shows:

```java
public class SimpleObject {

    private AccountService accountService;

    public void setAccountService(AccountService accountService) {
        this.accountService = accountService;
    }

    // additional methods using the accountService
}
```

To link in the service on the client, we create a separate Spring container, to contain the following simple object and the service linking configuration bits:

```xml
<bean class="example.SimpleObject">
    <property name="accountService" ref="accountService"/>
</bean>

<bean id="accountService" class="org.springframework.remoting.rmi.RmiProxyFactoryBean">
    <property name="serviceUrl" value="rmi://HOST:1199/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

That is all we need to do to support the remote account service on the client. Spring transparently creates an invoker and remotely enables the account service through the `RmiServiceExporter`. At the client, we link it in by using the `RmiProxyFactoryBean`.

### 2.5. Using Hessian to Remotely Call Services through HTTP (Deprecated)

|      | As of Spring Framework 5.3, Hessian support is deprecated and will not be replaced. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Hessian offers a binary HTTP-based remoting protocol. It is developed by Caucho, and you can find more information about Hessian itself at https://www.caucho.com/.

#### 2.5.1. Hessian

Hessian communicates through HTTP and does so by using a custom servlet. By using Spring’s `DispatcherServlet` principles (see [webmvc.html](https://docs.spring.io/spring-framework/docs/current/reference/html/webmvc.html#mvc-servlet)), we can wire up such a servlet to expose your services. First, we have to create a new servlet in our application, as shown in the following excerpt from `web.xml`:

```xml
<servlet>
    <servlet-name>remoting</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>remoting</servlet-name>
    <url-pattern>/remoting/*</url-pattern>
</servlet-mapping>
```

If you are familiar with Spring’s `DispatcherServlet` principles, you probably know that now you have to create a Spring container configuration resource named `remoting-servlet.xml` (after the name of your servlet) in the `WEB-INF` directory. The application context is used in the next section.

或者，考虑使用Spring的simpler `HttpRequestHandlerServlet`。这样做可以使您将远程导出器定义嵌入到根应用程序上下文中（默认情况下，在中`WEB-INF/applicationContext.xml`），并使用指向特定导出器bean的单个servlet定义。在这种情况下，每个servlet名称都必须与其目标导出器的bean名称相匹配。

#### 2.5.2。使用公开您的豆子`HessianServiceExporter`

在名为的新创建的应用程序上下文中`remoting-servlet.xml`，我们创建一个 `HessianServiceExporter`来导出我们的服务，如以下示例所示：

```xml
<bean id="accountService" class="example.AccountServiceImpl">
    <!-- any additional properties, maybe a DAO? -->
</bean>

<bean name="/AccountService" class="org.springframework.remoting.caucho.HessianServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

现在，我们准备在客户端链接服务。没有指定显式处理程序映射（将请求URL映射到服务），因此我们使用`BeanNameUrlHandlerMapping` used。因此，将在包含`DispatcherServlet`实例的映射（如先前定义）中 通过其bean名称指示的URL导出服务`https://HOST:8080/remoting/AccountService`。

或者，您可以`HessianServiceExporter`在根应用程序上下文中创建一个（例如，在中`WEB-INF/applicationContext.xml`），如以下示例所示：

```xml
<bean name="accountExporter" class="org.springframework.remoting.caucho.HessianServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

在后一种情况下，您应该在中为该导出器定义一个相应的servlet `web.xml`，并得到相同的最终结果：导出器被映射到位于的请求路径 `/remoting/AccountService`。注意，servlet名称需要与目标导出器的bean名称匹配。以下示例显示了如何执行此操作：

```xml
<servlet>
    <servlet-name>accountExporter</servlet-name>
    <servlet-class>org.springframework.web.context.support.HttpRequestHandlerServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>accountExporter</servlet-name>
    <url-pattern>/remoting/AccountService</url-pattern>
</servlet-mapping>
```

#### 2.5.3。在客户端上链接服务

通过使用`HessianProxyFactoryBean`，我们可以在客户端链接服务。与RMI示例相同的原理适用。我们创建一个单独的bean工厂或应用程序上下文，并`SimpleObject`通过使用`AccountService`来管理帐户来提及以下bean ，如以下示例所示：

```xml
<bean class="example.SimpleObject">
    <property name="accountService" ref="accountService"/>
</bean>

<bean id="accountService" class="org.springframework.remoting.caucho.HessianProxyFactoryBean">
    <property name="serviceUrl" value="https://remotehost:8080/remoting/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

#### 2.5.4。将HTTP基本身份验证应用于通过Hessian公开的服务

Hessian的优点之一是我们可以轻松地应用HTTP基本身份验证，因为这两种协议都是基于HTTP的。`web.xml`例如，可以通过使用安全功能来应用常规的HTTP服务器安全性机制。通常，您无需在此处使用每个用户的安全凭证。相反，您可以使用在该`HessianProxyFactoryBean`级别定义的共享凭据（类似于JDBC `DataSource`），如以下示例所示：

```xml
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping">
    <property name="interceptors" ref="authorizationInterceptor"/>
</bean>

<bean id="authorizationInterceptor"
        class="org.springframework.web.servlet.handler.UserRoleAuthorizationInterceptor">
    <property name="authorizedRoles" value="administrator,operator"/>
</bean>
```

在前面的示例中，我们明确提到`BeanNameUrlHandlerMapping`并设置了一个拦截器，以仅让管理员和操作员调用此应用程序上下文中提到的bean。

|      | 前面的示例未显示灵活的安全基础结构。有关安全性的更多选项，请参阅https://projects.spring.io/spring-security/上的Spring Security项目。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.6。Spring HTTP Invoker（不建议使用）

|      | 从Spring Framework 5.3开始，不推荐使用HTTP Invoker支持，并且不会替换。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

与Hessian相反，Spring HTTP调用程序都是轻量级协议，它们使用自己的苗条序列化机制，并使用标准Java序列化机制通过HTTP公开服务。如果您的参数和返回类型是无法通过使用Hessian使用的序列化机制进行序列化的复杂类型，则这将具有巨大的优势（有关选择远程处理技术的更多注意事项，请参阅下一节）。

在幕后，Spring使用JDK或Apache提供的标准功能`HttpComponents`来执行HTTP调用。如果您需要更高级且更易于使用的功能，请使用后者。有关 更多信息，请参见 [hc.apache.org/httpcomponents-client-ga/](https://hc.apache.org/httpcomponents-client-ga/)。

|      | 注意由于不安全的Java反序列化而导致的漏洞：操纵输入流可能会在反序列化步骤中导致服务器上不必要的代码执行。因此，请勿将HTTP调用方终结点暴露给不受信任的客户端。而是仅在您自己的服务之间公开它们。通常，强烈建议您改用其他任何消息格式（例如JSON）。如果您担心由Java序列化引起的安全漏洞，请考虑在核心JVM级别上使用通用序列化筛选器机制，该机制最初是为JDK 9开发的，但同时又移植到了JDK 8、7和6。参见 https://blogs.oracle.com/java-platform-group/entry/incoming_filter_serialization_data_a 和https://openjdk.java.net/jeps/290。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.6.1。公开服务对象

为服务对象设置HTTP调用程序基础结构与使用Hessian进行设置的方法非常相似。正如Hessian支持提供的那样 `HessianServiceExporter`，Spring的HttpInvoker支持提供了 `org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter`。

为了`AccountService`在Spring Web MVC中公开（前面提到的） `DispatcherServlet`，需要在调度程序的应用程序上下文中进行以下配置，如以下示例所示：

```xml
<bean name="/AccountService" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

此类导出器定义通过`DispatcherServlet`实例的标准映射功能公开，如[有关Hessian的部分所述](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-caucho-protocols)。

或者，您可以`HttpInvokerServiceExporter`在根应用程序上下文中创建一个（例如，在中`'WEB-INF/applicationContext.xml'`），如以下示例所示：

```xml
<bean name="accountExporter" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

另外，您可以在中为该导出器定义一个相应的servlet，该`web.xml`servlet名称与目标导出器的bean名称匹配，如以下示例所示：

```xml
<servlet>
    <servlet-name>accountExporter</servlet-name>
    <servlet-class>org.springframework.web.context.support.HttpRequestHandlerServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>accountExporter</servlet-name>
    <url-pattern>/remoting/AccountService</url-pattern>
</servlet-mapping>
```

#### 2.6.2。在客户端链接服务

同样，从客户端链接服务非常类似于使用Hessian时的方式。通过使用代理，Spring可以将对HTTP POST请求的调用转换为指向导出服务的URL。以下示例显示如何配置此安排：

```xml
<bean id="httpInvokerProxy" class="org.springframework.remoting.httpinvoker.HttpInvokerProxyFactoryBean">
    <property name="serviceUrl" value="https://remotehost:8080/remoting/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

如前所述，您可以选择要使用的HTTP客户端。默认情况下，会 `HttpInvokerProxy`使用JDK的HTTP功能，但是您也可以`HttpComponents`通过设置`httpInvokerRequestExecutor`属性来使用Apache 客户端。以下示例显示了如何执行此操作：

```xml
<property name="httpInvokerRequestExecutor">
    <bean class="org.springframework.remoting.httpinvoker.HttpComponentsHttpInvokerRequestExecutor"/>
</property>
```

### 2.7。JMS（已弃用）

|      | 从Spring Framework 5.3开始，已弃用JMS远程支持，并且将不会替换。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以通过使用JMS作为基础通信协议来透明地公开服务。Spring框架中的JMS远程支持非常基本。它在`same thread`和上以相同的非事务发送和接收`Session`。结果，吞吐量取决于实现方式。请注意，这些单线程和非事务性约束仅适用于Spring的JMS远程支持。有关Spring对基于JMS的消息传递的丰富支持的信息，请参见[JMS（Java消息服务）](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms)。

服务器和客户端均使用以下接口：

```java
package com.foo;

public interface CheckingAccountService {

    public void cancelAccount(Long accountId);
}
```

在服务器端使用上述接口的以下简单实现：

```java
package com.foo;

public class SimpleCheckingAccountService implements CheckingAccountService {

    public void cancelAccount(Long accountId) {
        System.out.println("Cancelling account [" + accountId + "]");
    }
}
```

以下配置文件包含在客户端和服务器上共享的JMS基础结构Bean：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="connectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://ep-t43:61616"/>
    </bean>

    <bean id="queue" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg value="mmm"/>
    </bean>

</beans>
```

#### 2.7.1。服务器端配置

在服务器上，您需要公开使用的服务对象 `JmsInvokerServiceExporter`，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="checkingAccountService"
            class="org.springframework.jms.remoting.JmsInvokerServiceExporter">
        <property name="serviceInterface" value="com.foo.CheckingAccountService"/>
        <property name="service">
            <bean class="com.foo.SimpleCheckingAccountService"/>
        </property>
    </bean>

    <bean class="org.springframework.jms.listener.SimpleMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="destination" ref="queue"/>
        <property name="concurrentConsumers" value="3"/>
        <property name="messageListener" ref="checkingAccountService"/>
    </bean>

</beans>
package com.foo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Server {

    public static void main(String[] args) throws Exception {
        new ClassPathXmlApplicationContext("com/foo/server.xml", "com/foo/jms.xml");
    }
}
```

#### 2.7.2。客户端配置

客户端只需要创建一个客户端代理即可实现约定的接口（`CheckingAccountService`）。

下面的示例定义了可以注入到其他客户端对象中的bean（代理负责通过JMS将调用转发到服务器端对象）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="checkingAccountService"
            class="org.springframework.jms.remoting.JmsInvokerProxyFactoryBean">
        <property name="serviceInterface" value="com.foo.CheckingAccountService"/>
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="queue" ref="queue"/>
    </bean>

</beans>
package com.foo;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Client {

    public static void main(String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("com/foo/client.xml", "com/foo/jms.xml");
        CheckingAccountService service = (CheckingAccountService) ctx.getBean("checkingAccountService");
        service.cancelAccount(new Long(10));
    }
}
```

## 3.企业JavaBeans（EJB）集成

作为轻量级容器，Spring通常被认为是EJB的替代品。我们确实相信，对于许多（即使不是大多数）应用程序和用例，Spring作为容器，结合其在事务，ORM和JDBC访问方面的丰富支持功能，比通过EJB实现等效功能是更好的选择。容器和EJB。

但是，请务必注意，使用Spring不会阻止您使用EJB。实际上，Spring使访问EJB以及在其中实现EJB和功能变得更加容易。另外，使用Spring访问EJB提供的服务可以使这些服务的实现稍后在本地EJB，远程EJB或POJO（普通旧Java对象）变体之间透明切换，而不必更改客户端代码。

在本章中，我们将研究Spring如何帮助您访问和实现EJB。当访问无状态会话Bean（SLSB）时，Spring提供了特殊的价值，因此我们从讨论这个主题开始。

### 3.1。访问EJB

本节介绍如何访问EJB。

#### 3.1.1。概念

要在本地或远程无状态会话Bean上调用方法，客户机代码通常必须执行JNDI查找以获取（本地或远程）EJB Home对象，然后`create`对该对象使用方法调用以获取实际的（本地或远程）EJB对象。EJB对象。然后在EJB上调用一种或多种方法。

为了避免重复的低级代码，许多EJB应用程序都使用服务定位器和业务委托模式。这些比在整个客户端代码中喷射JNDI查找要好，但是它们的常规实现有很多缺点：

- 通常，使用EJB的代码取决于Service Locator或Business Delegate单例，因此很难进行测试。
- 在不使用业务委托的情况下使用服务定位器模式的情况下，应用程序代码仍然最终必须`create()`在EJB home上调用该方法并处理产生的异常。因此，它仍然与EJB API和EJB编程模型的复杂性联系在一起。
- 实现业务委托模式通常会导致大量的代码重复，我们必须编写许多在EJB上调用相同方法的方法。

Spring的方法是允许创建和使用代理对象（通常在Spring容器内配置），这些代理对象充当无代码的业务委托。除非您在此类代码中实际添加了实际价值，否则您无需在手动编码的业务委托中编写另一个Service Locator，另一个JNDI查找或重复方法。

#### 3.1.2。访问本地SLSB

假设我们有一个Web控制器，需要使用本地EJB。我们遵循最佳实践，并使用EJB业务方法接口模式，以便EJB的本地接口扩展了非EJB特定的业务方法接口。我们称这种业务方法为接口`MyComponent`。以下示例显示了这样的接口：

```java
public interface MyComponent {
    ...
}
```

使用业务方法接口模式的主要原因之一是确保本地接口中的方法签名与bean实现类之间的同步是自动的。另一个原因是，如果有必要的话，以后可以使我们更轻松地切换到服务的POJO（普通旧Java对象）实现。我们还需要实现本地home接口，并提供一个实现类`SessionBean`和`MyComponent`业务方法接口的实现类。现在，将Web层控制器连接到EJB实现所需的唯一Java编码是`MyComponent` 在控制器上公开类型的setter方法。这会将引用另存为控制器中的实例变量。以下示例显示了如何执行此操作：

```java
private MyComponent myComponent;

public void setMyComponent(MyComponent myComponent) {
    this.myComponent = myComponent;
}
```

随后，我们可以在控制器中的任何业务方法中使用此实例变量。现在，假设我们从Spring容器中获取了控制器对象，我们可以（在相同上下文中）配置一个`LocalStatelessSessionProxyFactoryBean`实例，即EJB代理对象。我们配置代理，并`myComponent`使用以下配置条目设置控制器的 属性：

```xml
<bean id="myComponent"
        class="org.springframework.ejb.access.LocalStatelessSessionProxyFactoryBean">
    <property name="jndiName" value="ejb/myBean"/>
    <property name="businessInterface" value="com.mycom.MyComponent"/>
</bean>

<bean id="myController" class="com.mycom.myController">
    <property name="myComponent" ref="myComponent"/>
</bean>
```

尽管没有强迫您使用AOP概念来欣赏结果，但仍要依靠Spring AOP框架在幕后进行大量工作。该 `myComponent`bean定义创建的EJB，它实现了业务方法接口的代理。EJB本地宿主在启动时被缓存，因此只有一个JNDI查找。每次调用EJB时，代理都会`classname`在本地EJB上调用方法，并在EJB上调用相应的业务方法。

该`myController`bean定义设置`myComponent`控制器类到EJB代理财产。

或者（最好在许多此类代理定义的情况下），考虑`<jee:local-slsb>`在Spring的“ jee”命名空间中使用配置元素。以下示例显示了如何执行此操作：

```xml
<jee:local-slsb id="myComponent" jndi-name="ejb/myBean"
        business-interface="com.mycom.MyComponent"/>

<bean id="myController" class="com.mycom.myController">
    <property name="myComponent" ref="myComponent"/>
</bean>
```

这种EJB访问机制极大地简化了应用程序代码。Web层代码（或其他EJB客户端代码）与EJB的使用无关。要用POJO或模拟对象或其他测试存根替换该EJB引用，我们可以更改`myComponent`Bean定义而无需更改一行Java代码。此外，作为应用程序的一部分，我们不必编写任何一行JNDI查找或其他EJB管道代码。

实际应用中的基准和经验表明，这种方法的性能开销（涉及目标EJB的反射调用）是最小的，并且在常规使用中是无法检测到的。请记住，无论如何我们都不希望对EJB进行细粒度的调用，因为与应用程序服务器中的EJB基础结构相关联的成本很高。

关于JNDI查找有一个警告。在bean容器中，此类通常最好用作单例（没有理由使其成为原型）。但是，如果该bean容器预先实例化了单例（与各种XML `ApplicationContext`变体一样），那么如果在EJB容器加载目标EJB之前加载了bean容器，则可能会出现问题。这是因为JNDI查找是在`init()`此类的方法中执行的，然后进行了缓存，但是EJB尚未绑定到目标位置。解决方案是不预先实例化该工厂对象，而是让它在首次使用时创建。在XML容器中，您可以使用`lazy-init`属性来控制它。

尽管大多数Spring用户都不感兴趣，但是那些使用EJB进行编程AOP的人可能要看一下`LocalSlsbInvokerInterceptor`。

#### 3.1.3。访问远程SLSB

访问远程EJB与访问本地EJB本质上相同，只是使用了 `SimpleRemoteStatelessSessionProxyFactoryBean`or`<jee:remote-slsb>`配置元素。当然，无论是否使用Spring，远程调用语义都适用：调用另一台计算机上另一台VM中的对象上的方法时，有时在使用情况和故障处理方面必须区别对待。

与非Spring方法相比，Spring的EJB客户端支持增加了另一优势。通常，在本地或远程调用EJB之间轻松地来回切换EJB客户端代码是有问题的。这是因为远程接口方法必须声明它们throw `RemoteException`，而客户端代码必须对此进行处理，而本地接口方法则不需要。通常需要修改为需要移至远程EJB的本地EJB编写的客户端代码，以添加对远程异常的处理，为需要移至本地EJB的远程EJB编写的客户端代码可以保持不变，但可以执行以下操作：许多不必要的远程异常处理，或进行修改以删除该代码。使用Spring远程EJB代理，您可以声明不抛出任何异常`RemoteException`在您的业务方法接口和实现EJB代码中，拥有一个完全相同的远程接口（除了它确实抛出`RemoteException`），并且依靠代理来动态地将这两个接口视为相同。也就是说，客户端代码不必处理已检查的`RemoteException`类。`RemoteException`在EJB调用期间抛出的所有实数都将重新抛出为未经检查的`RemoteAccessException`类，该类是的子类`RuntimeException`。然后，您可以在本地EJB或远程EJB（甚至纯Java对象）实现之间随意切换目标服务，而无需了解或关心客户端代码。当然，这是可选的：没有什么可以阻止您`RemoteException`在业务界面中进行声明。

#### 3.1.4。访问EJB 2.x SLSB与EJB 3 SLSB

通过Spring访问EJB 2.x会话Bean和EJB 3会话Bean在很大程度上是透明的。Spring的EJB访问器（包括`<jee:local-slsb>`和 `<jee:remote-slsb>`设施）在运行时透明地适应实际组件。它们会处理Home接口（如果找到）（EJB 2.x样式），或者在没有Home接口可用时（EJB 3样式）执行直接组件调用。

注意：对于EJB 3会话Bean，您也可以有效地使用`JndiObjectFactoryBean`/ `<jee:jndi-lookup>`，因为公开了完全可用的组件引用以用于在那里的普通JNDI查找。定义显式`<jee:local-slsb>`或`<jee:remote-slsb>` 查找可提供一致且更显式的EJB访问配置。

## 4. JMS（Java消息服务）

Spring提供了一个JMS集成框架，该框架简化了JMS API的使用，就像Spring对JDBC API的集成一样。

JMS可以大致分为两个功能区域，即消息的产生和使用。所述`JmsTemplate`类用于消息生成和同步消息接收操作。对于类似于Java EE的消息驱动bean样式的异步接收，Spring提供了许多消息侦听器容器，可用于创建消息驱动POJO（MDP）。Spring还提供了一种声明式方法来创建消息侦听器。

该`org.springframework.jms.core`软件包提供了使用JMS的核心功能。它包含JMS模板类，该类通过处理资源的创建和释放来简化JMS的使用，就像`JdbcTemplate`JDBC一样。Spring模板类共有的设计原则是提供帮助器方法来执行常用操作，并且对于更复杂的用法，将处理任务的实质委托给用户实现的回调接口。JMS模板遵循相同的设计。这些类提供了各种方便的方法，用于发送消息，同步使用消息以及向用户公开JMS会话和消息生成器。

该`org.springframework.jms.support`软件包提供`JMSException`翻译功能。转换将检查的`JMSException`层次结构转换为未检查的异常的镜像层次结构。如果`javax.jms.JMSException`存在checked的任何提供程序特定的子类，则将此异常包装在unchecked中`UncategorizedJmsException`。

该`org.springframework.jms.support.converter`软件包提供了一个`MessageConverter` 抽象，可以在Java对象和JMS消息之间进行转换。

该`org.springframework.jms.support.destination`软件包提供了用于管理JMS目的地的各种策略，例如为JNDI中存储的目的地提供服务定位器。

该`org.springframework.jms.annotation`软件包通过提供了必要的基础结构来支持注释驱动的侦听器端点`@JmsListener`。

该`org.springframework.jms.config`软件包提供了`jms`名称空间的解析器实现 以及java config支持，以配置侦听器容器和创建侦听器端点。

最后，该`org.springframework.jms.connection`软件包提供了`ConnectionFactory`适用于独立应用程序的实现。它还包含Spring `PlatformTransactionManager`for JMS的实现（巧妙地命名为 `JmsTransactionManager`）。这允许将JMS作为事务资源无缝集成到Spring的事务管理机制中。

|      | 从Spring Framework 5开始，Spring的JMS软件包完全支持JMS 2.0，并要求在运行时提供JMS 2.0 API。我们建议使用JMS 2.0兼容的提供程序。如果您碰巧在系统中使用了较旧的消息代理，则可以尝试为现有代理生成升级到JMS 2.0兼容驱动程序。或者，您也可以尝试在基于JMS 1.1的驱动程序上运行，只需将JMS 2.0 API jar放在类路径上，而仅对驱动程序使用兼容JMS 1.1的API。Spring的JMS支持默认情况下遵守JMS 1.1约定，因此通过相应的配置，它确实支持这种情况。但是，请仅在过渡方案中考虑这一点。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.1。使用Spring JMS

本节描述如何使用Spring的JMS组件。

#### 4.1.1。使用`JmsTemplate`

的`JmsTemplate`类是在JMS芯封装的核心类。由于它在发送或同步接收消息时处理资源的创建和释放，因此它简化了JMS的使用。

```
JmsTemplate`仅使用需求的代码即可实现为回调接口提供清晰定义的高级协定。当中的调用代码提供`MessageCreator`时，回调接口会创建一条消息。为了允许更复杂地使用JMS API，提供JMS会话并公开和 对。`Session``JmsTemplate``SessionCallback``ProducerCallback``Session``MessageProducer
```

JMS API公开了两种类型的发送方法，一种采用交付模式，优先级和生存时间作为服务质量（QOS）参数，另一种不采用QOS参数并使用默认值。由于`JmsTemplate`有许多发送方法，因此设置QOS参数已作为bean属性公开，以避免重复发送方法。同样，使用`setReceiveTimeout`属性设置同步接收调用的超时值。

一些JMS提供程序允许通过配置来管理默认QOS值`ConnectionFactory`。这样做的结果是，对`MessageProducer`实例`send`方法（`send(Destination destination, Message message)`）的调用 使用与JMS规范中指定的QOS默认值不同的QOS默认值。为了提供QOS值的一致管理，`JmsTemplate`必须的，因此，专门启用通过设置布尔值属性使用自己的QOS值 `isExplicitQosEnabled`到`true`。

为了方便起见，`JmsTemplate`还公开了一个基本的请求-答复操作，该操作允许发送消息并等待作为该操作一部分创建的临时队列的答复。

|      | `JmsTemplate`一旦配置，该类的 实例是线程安全的。这很重要，因为这意味着您可以配置的单个实例，`JmsTemplate` 然后将该共享引用安全地注入多个协作者中。需要明确的`JmsTemplate`是，状态是有状态的，因为它保持对的引用 `ConnectionFactory`，但是此状态不是会话状态。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

从Spring Framework 4.1开始，它`JmsMessagingTemplate`是基于`JmsTemplate` 消息传递抽象构建的，并且提供了与消息传递抽象的集成，即 `org.springframework.messaging.Message`。这使您可以创建以通用方式发送的消息。

#### 4.1.2。连接数

该`JmsTemplate`要求的一个参考`ConnectionFactory`。它`ConnectionFactory` 是JMS规范的一部分，并且是使用JMS的切入点。客户端应用程序将其用作工厂以创建与JMS提供程序的连接，并封装各种配置参数，其中许多是特定于供应商的，例如SSL配置选项。

当在EJB中使用JMS时，供应商提供JMS接口的实现，以便它们可以参与声明式事务管理并执行连接和会话的池化。为了使用此实现，Java EE容器通常要求您将JMS连接工厂声明为`resource-ref`EJB或Servlet部署描述符中的内部。为了确保在`JmsTemplate`EJB内部使用这些功能 ，客户端应用程序应确保其引用的托管实现`ConnectionFactory`。

##### 缓存消息资源

标准API涉及创建许多中间对象。要发送消息，请执行以下“ API”遍历：

```
ConnectionFactory->连接->会话-> MessageProducer->发送
```

在`ConnectionFactory`和`Send`操作之间，创建并销毁了三个中间对象。为了优化资源使用并提高性能，Spring提供了两种实现`ConnectionFactory`。

##### 使用 `SingleConnectionFactory`

Spring提供了`ConnectionFactory`接口的实现，该接口 在所有调用`SingleConnectionFactory`中返回相同的内容`Connection`， `createConnection()`并忽略对的调用`close()`。这对于测试和独立环境非常有用，因此同一连接可用于`JmsTemplate`可能跨越任意数量事务的多个 调用。`SingleConnectionFactory` 引用了`ConnectionFactory`通常来自JNDI的标准。

##### 使用 `CachingConnectionFactory`

该`CachingConnectionFactory`扩展的功能`SingleConnectionFactory` ，并增加了缓存`Session`，`MessageProducer`和`MessageConsumer`实例。初始缓存大小设置为`1`。您可以使用该`sessionCacheSize`属性来增加缓存的会话数。请注意，由于根据会话的确认模式缓存会话，因此实际缓存的会话数大于该数量，因此当`sessionCacheSize`设置为one时，最多可以有四个缓存的会话实例（每个确认模式一个）。`MessageProducer`和`MessageConsumer`实例在其自己的会话中缓存，并且在缓存时还要考虑生产者和使用者的独特属性。MessageProducers根据其目的地进行缓存。基于由目标，选择器，noLocal传递标志和持久订阅名称（如果创建持久使用者）组成的键来缓存MessageConsumers。

#### 4.1.3。目的地管理

作为`ConnectionFactory`实例，目标是可以在JNDI中存储和检索的JMS管理的对象。在配置Spring应用程序上下文时，可以使用JNDI`JndiObjectFactoryBean`工厂类或`<jee:jndi-lookup>`对对象对JMS目标的引用执行依赖项注入。但是，如果应用程序中有大量目标，或者JMS提供程序具有独特的高级目标管理功能，则此策略通常很麻烦。这种高级目标管理的示例包括动态目标的创建或对目标的分层名称空间的支持。将`JmsTemplate`目标名称的解析委托给实现`DestinationResolver`接口的JMS目标对象 。`DynamicDestinationResolver`是由...使用的默认实现`JmsTemplate`并适应解析的动态目标。`JndiDestinationResolver`还提供A 充当JNDI中包含的目的地的服务定位器，并且可以选择回退到中包含的行为 `DynamicDestinationResolver`。

通常，仅在运行时才知道JMS应用程序中使用的目的地，因此，在部署应用程序时无法通过管理方式创建。这通常是因为在交互的系统组件之间存在共享的应用程序逻辑，这些组件根据已知的命名约定在运行时创建目标。即使创建动态目标不属于JMS规范的一部分，但大多数供应商都提供了此功能。动态目标是使用用户定义的名称创建的，该名称将它们与临时目标区分开来，并且通常未在JNDI中注册。提供商之间使用的用于创建动态目标的API有所不同，因为与目标关联的属性是特定于供应商的。然而，`TopicSession` `createTopic(String topicName)`或`QueueSession` `createQueue(String queueName)`使用默认目标属性创建新目标的方法。`DynamicDestinationResolver`然后，根据供应商的实现，还可以创建一个物理目标，而不是仅解决一个目标。

布尔值属性`pubSubDomain`用于使用`JmsTemplate`正在使用的JMS域的知识来配置。默认情况下，此属性的值为false，表示`Queues`将使用点对点域。（由所使用`JmsTemplate`）此属性通过`DestinationResolver`接口的实现确定动态目标解析的行为。

您还可以`JmsTemplate`通过属性将设置为具有默认目的地`defaultDestination`。默认目标是带有不引用特定目标的发送和接收操作。

#### 4.1.4。消息侦听器容器

在EJB世界中，JMS消息最常见的用途之一就是驱动消息驱动的bean（MDB）。Spring提供了一种解决方案，以不将用户绑定到EJB容器的方式创建消息驱动的POJO（MDP）。（有关Spring对MDP支持的详细介绍，请参见[异步接收：消息驱动的POJO](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-receiving-async)。）从Spring Framework 4.1开始，可以使用以下方法对端点方法进行注释`@JmsListener` -有关更多详细信息，请参见[注释驱动的侦听器端点](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-annotated)。

消息侦听器容器用于从JMS消息队列接收消息并驱动`MessageListener`注入到其中的消息。侦听器容器负责消息接收的所有线程，并分派到侦听器中进行处理。消息侦听器容器是MDP与消息传递提供程序之间的中介，并负责注册接收消息，参与事务，资源获取和释放，异常转换等。这使您可以编写与接收消息（并可能响应消息）相关的（可能很复杂的）业务逻辑，并将样板JMS基础结构问题委托给框架。

Spring附带了两个标准的JMS消息侦听器容器，每个容器都有其专门的功能集。

- [`SimpleMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-mdp-simple)
- [`DefaultMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-mdp-default)

##### 使用 `SimpleMessageListenerContainer`

此消息侦听器容器是两种标准样式中的简单容器。它在启动时创建固定数量的JMS会话和使用者，使用标准JMS`MessageConsumer.setMessageListener()`方法注册侦听器，然后将其留给JMS提供者以执行侦听器回调。此变体不允许动态适应运行时需求或参与外部管理的事务。在兼容性方面，它非常接近独立JMS规范的精神，但通常与Java EE的JMS限制不兼容。

|      | 尽管`SimpleMessageListenerContainer`不允许参与外部管理的事务，但它确实支持本机JMS事务。要启用此功能，可以将`sessionTransacted`标志切换为，`true`或者在XML名称空间中将`acknowledge`属性设置 为`transacted`。从您的侦听器抛出的异常会导致回滚，并重新传递消息。或者，考虑使用 `CLIENT_ACKNOWLEDGE`模式，该模式在出现异常的情况下也提供重新交付但不使用事务处理的`Session`实例，因此`Session`在事务协议中不包括任何其他 操作（例如发送响应消息）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 默认`AUTO_ACKNOWLEDGE`模式不能提供适当的可靠性保证。当侦听器执行失败时（由于提供者会在侦听器调用后自动确认每条消息，并且没有异常要传播到提供者），或者在侦听器容器关闭时（您可以通过设置`acceptMessagesWhileStopping`标志进行配置），消息可能会丢失。确保出于可靠性需求（例如，为了可靠的队列处理和持久的主题订阅）使用事务处理的会话。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 使用 `DefaultMessageListenerContainer`

大多数情况下使用此消息侦听器容器。与之相反 `SimpleMessageListenerContainer`，此容器变量允许动态适应运行时需求，并且能够参与外部管理的事务。配置为时，每个接收到的消息都会在XA事务中注册 `JtaTransactionManager`。结果，处理可以利用XA事务语义。该侦听器容器在JMS提供程序的低要求，高级功能（例如参与外部管理的事务）以及与Java EE环境的兼容性之间取得了良好的平衡。

您可以自定义容器的缓存级别。请注意，当未启用缓存时，将为每个消息接收创建一个新的连接和一个新的会话。将此内容与具有高负载的非持久订阅结合使用可能会导致消息丢失。在这种情况下，请确保使用适当的缓存级别。

当代理崩溃时，此容器还具有可恢复的功能。默认情况下，一个简单的`BackOff`实现每五秒钟重试一次。您可以`BackOff`为更细粒度的恢复选项指定自定义实现。有关`ExponentialBackOff`示例，请参见api-spring-framework / util / backoff / ExponentialBackOff.html [ ]。

|      | 与其兄弟（[`SimpleMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-mdp-simple)） 一样，它`DefaultMessageListenerContainer`支持本机JMS事务，并允许自定义确认模式。如果对您的方案可行，则强烈建议在外部管理的事务上使用此方法，也就是说，如果JVM死亡，您可以偶尔接收重复的消息。业务逻辑中的自定义重复消息检测步骤可以解决这种情况-例如，以业务实体存在检查或协议表检查的形式。任何这样的安排是显著比其他更有效的：用一个XA事务包裹整个处理（通过在配置 `DefaultMessageListenerContainer`有`JtaTransactionManager`）以覆盖所述JMS消息的接收，以及业务逻辑在您的消息侦听器的执行（包括数据库操作等）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 默认`AUTO_ACKNOWLEDGE`模式不能提供适当的可靠性保证。当侦听器执行失败时（由于提供者会在侦听器调用后自动确认每条消息，并且没有异常要传播到提供者），或者在侦听器容器关闭时（您可以通过设置`acceptMessagesWhileStopping`标志进行配置），消息可能会丢失。确保出于可靠性需求（例如，为了可靠的队列处理和持久的主题订阅）使用事务处理的会话。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 4.1.5。交易管理

Spring提供了一个`JmsTransactionManager`管理单个JMS事务的 `ConnectionFactory`。这使JMS应用程序可以利用Spring的托管事务功能，如 [“数据访问”一章的“事务管理”部分所述](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction)。在`JmsTransactionManager`执行本地资源交易，结合从指定的JMS连接/会话对`ConnectionFactory`给线程。 `JmsTemplate`自动检测此类交易资源并相应地对其进行操作。

在Java EE环境中，`ConnectionFactory`池连接和实例实例，因此这些资源可有效地跨事务重用。在独立环境中，使用Spring的`SingleConnectionFactory`结果在共享的JMS中`Connection`，每个事务都有自己独立的`Session`。或者，考虑使用提供程序专用的池适配器，例如ActiveMQ的`PooledConnectionFactory` 类。

您还可以`JmsTemplate`与`JtaTransactionManager`和具有XA功能的JMS一起使用 `ConnectionFactory`以执行分布式事务。请注意，这需要使用JTA事务管理器以及正确的XA配置的ConnectionFactory。（检查您的Java EE服务器或JMS提供程序的文档。）

跨托管和非托管的事务环境下重用代码可以使用JMS API来创建一个混乱的时候`Session`从一个`Connection`。这是因为JMS API只有一种工厂方法可以创建`Session`，并且它需要事务和确认模式的值。在托管环境中，设置这些值是环境的事务基础结构的责任，因此，供应商对JMS Connection的包装将忽略这些值。当您使用`JmsTemplate` 在非托管环境中，您可以通过使用属性来指定这些值`sessionTransacted`和`sessionAcknowledgeMode`。当使用 `PlatformTransactionManager`with时`JmsTemplate`，模板总是被赋予事务性JMS `Session`。

### 4.2。发送信息

将`JmsTemplate`包含许多方便的方法来发送消息。发送方法通过使用`javax.jms.Destination`对象指定目的地，其他方法通过`String`在JNDI查找中使用指定目的地。`send`不使用目标参数的方法使用默认目标。

以下示例使用`MessageCreator`回调从提供的`Session`对象创建文本消息：

```java
import javax.jms.ConnectionFactory;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Queue;
import javax.jms.Session;

import org.springframework.jms.core.MessageCreator;
import org.springframework.jms.core.JmsTemplate;

public class JmsQueueSender {

    private JmsTemplate jmsTemplate;
    private Queue queue;

    public void setConnectionFactory(ConnectionFactory cf) {
        this.jmsTemplate = new JmsTemplate(cf);
    }

    public void setQueue(Queue queue) {
        this.queue = queue;
    }

    public void simpleSend() {
        this.jmsTemplate.send(this.queue, new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                return session.createTextMessage("hello queue world");
            }
        });
    }
}
```

在前面的示例中，`JmsTemplate`通过将引用传递给来构造 `ConnectionFactory`。作为替代方案，提供了一个零参数的构造函数，该构造函数 `connectionFactory`可用于以JavaBean样式（使用`BeanFactory`Java或纯Java代码）构造实例。另外，考虑从Spring的`JmsGatewaySupport`便捷基类派生，该基类为JMS配置提供了预构建的bean属性。

该`send(String destinationName, MessageCreator creator)`方法使您可以使用目标的字符串名称发送消息。如果这些名称已在JNDI中注册，则应将`destinationResolver`模板的属性设置为的实例 `JndiDestinationResolver`。

如果您创建`JmsTemplate`并指定了默认目标，则会向该目标 `send(MessageCreator c)`发送一条消息。

#### 4.2.1。使用消息转换器

1. 1. 
      4.3。接收讯息
      1. 
      2. 
      3. 
      4. 
      5. 
   2. [4.4。支持JCA消息端点](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-jca-message-endpoint-manager)
   3. 4.5。注释驱动的侦听器端点
      1. 
      2. 
      3. 
      4. 
   4. [4.6。JMS命名空间支持](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-namespace)
2. \5. JMX
   1. 
      1. 
      2. 
      3. 
      4. 
      5. 
   2. 
      1. 
      2. 
      3. 
      4. 
      5. 
      6. 
   3. 
      1. 
      2. 
      3. 
   4. 
      1. 
      2. 
      3. 
   5. 
   6. 
      1. 
      2. 
   7. 
3. 6.电子邮件
   1. 
      1. 
      2. 
   2. 
      1. 
         1. 
         2. 
      2. 
4. 7.任务执行和计划
   1. 
      1. 
      2. 
   2. 
      1. 
      2. 
      3. 
   3. 
      1. 
      2. 
      3. 
      4. 
      5. 
   4. 
      1. 
      2. 
      3. 
   5. 
      1. 
      2. 
      3. 
5. 8.缓存抽象
   1. 
   2. 
      1. 
         1. 
         2. 
         3. 
         4. 
         5. 
         6. 
         7. 
      2. 
      3. 
      4. 
      5. 
      6. 
      7. 
   3. 
      1. 
      2. 
   4. 
   5. 
      1. 
      2. 
      3. 
      4. 
      5. 
      6. 
   6. 
   7. 
6. 9.附录
   1. 
      1. 
         1. 
         2. 
         3. 
         4. 
         5. 
         6. 
         7. 
      2. 
      3. 
      4. 

参考文档的这一部分涵盖了Spring Framework与多种技术的集成。

## 1. REST端点

The Spring Framework provides two choices for making calls to REST endpoints:

- [`RestTemplate`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-resttemplate): The original Spring REST client with a synchronous, template method API.
- [WebClient](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client): a non-blocking, reactive alternative that supports both synchronous and asynchronous as well as streaming scenarios.

|      | As of 5.0 the `RestTemplate` is in maintenance mode, with only minor requests for changes and bugs to be accepted going forward. Please, consider using the [WebClient](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client) which offers a more modern API and supports sync, async, and streaming scenarios. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.1. `RestTemplate`

The `RestTemplate` provides a higher level API over HTTP client libraries. It makes it easy to invoke REST endpoints in a single line. It exposes the following groups of overloaded methods:

| Method group      | Description                                                  |
| :---------------- | :----------------------------------------------------------- |
| `getForObject`    | Retrieves a representation via GET.                          |
| `getForEntity`    | Retrieves a `ResponseEntity` (that is, status, headers, and body) by using GET. |
| `headForHeaders`  | Retrieves all headers for a resource by using HEAD.          |
| `postForLocation` | Creates a new resource by using POST and returns the `Location` header from the response. |
| `postForObject`   | Creates a new resource by using POST and returns the representation from the response. |
| `postForEntity`   | Creates a new resource by using POST and returns the representation from the response. |
| `put`             | Creates or updates a resource by using PUT.                  |
| `patchForObject`  | Updates a resource by using PATCH and returns the representation from the response. Note that the JDK `HttpURLConnection` does not support the `PATCH`, but Apache HttpComponents and others do. |
| `delete`          | Deletes the resources at the specified URI by using DELETE.  |
| `optionsForAllow` | Retrieves allowed HTTP methods for a resource by using ALLOW. |
| `exchange`        | More generalized (and less opinionated) version of the preceding methods that provides extra flexibility when needed. It accepts a `RequestEntity` (including HTTP method, URL, headers, and body as input) and returns a `ResponseEntity`.These methods allow the use of `ParameterizedTypeReference` instead of `Class` to specify a response type with generics. |
| `execute`         | The most generalized way to perform a request, with full control over request preparation and response extraction through callback interfaces. |

#### 1.1.1. Initialization

The default constructor uses `java.net.HttpURLConnection` to perform requests. You can switch to a different HTTP library with an implementation of `ClientHttpRequestFactory`. There is built-in support for the following:

- Apache HttpComponents
- Netty
- OkHttp

For example, to switch to Apache HttpComponents, you can use the following:

```java
RestTemplate template = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
```

Each `ClientHttpRequestFactory` exposes configuration options specific to the underlying HTTP client library — for example, for credentials, connection pooling, and other details.

|      | Note that the `java.net` implementation for HTTP requests can raise an exception when accessing the status of a response that represents an error (such as 401). If this is an issue, switch to another HTTP client library. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### URIs

Many of the `RestTemplate` methods accept a URI template and URI template variables, either as a `String` variable argument, or as `Map<String,String>`.

The following example uses a `String` variable argument:

```java
String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/bookings/{booking}", String.class, "42", "21");
```

The following example uses a `Map<String, String>`:

```java
Map<String, String> vars = Collections.singletonMap("hotel", "42");

String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/rooms/{hotel}", String.class, vars);
```

Keep in mind URI templates are automatically encoded, as the following example shows:

```java
restTemplate.getForObject("https://example.com/hotel list", String.class);

// Results in request to "https://example.com/hotel%20list"
```

You can use the `uriTemplateHandler` property of `RestTemplate` to customize how URIs are encoded. Alternatively, you can prepare a `java.net.URI` and pass it into one of the `RestTemplate` methods that accepts a `URI`.

For more details on working with and encoding URIs, see [URI Links](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-uri-building).

##### Headers

You can use the `exchange()` methods to specify request headers, as the following example shows:

```java
String uriTemplate = "https://example.com/hotels/{hotel}";
URI uri = UriComponentsBuilder.fromUriString(uriTemplate).build(42);

RequestEntity<Void> requestEntity = RequestEntity.get(uri)
        .header(("MyRequestHeader", "MyValue")
        .build();

ResponseEntity<String> response = template.exchange(requestEntity, String.class);

String responseHeader = response.getHeaders().getFirst("MyResponseHeader");
String body = response.getBody();
```

You can obtain response headers through many `RestTemplate` method variants that return `ResponseEntity`.

#### 1.1.2. Body

Objects passed into and returned from `RestTemplate` methods are converted to and from raw content with the help of an `HttpMessageConverter`.

On a POST, an input object is serialized to the request body, as the following example shows:

```
URI location = template.postForLocation("https://example.com/people", person);
```

You need not explicitly set the Content-Type header of the request. In most cases, you can find a compatible message converter based on the source `Object` type, and the chosen message converter sets the content type accordingly. If necessary, you can use the `exchange` methods to explicitly provide the `Content-Type` request header, and that, in turn, influences what message converter is selected.

On a GET, the body of the response is deserialized to an output `Object`, as the following example shows:

```
Person person = restTemplate.getForObject("https://example.com/people/{id}", Person.class, 42);
```

The `Accept` header of the request does not need to be explicitly set. In most cases, a compatible message converter can be found based on the expected response type, which then helps to populate the `Accept` header. If necessary, you can use the `exchange` methods to provide the `Accept` header explicitly.

By default, `RestTemplate` registers all built-in [message converters](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-message-conversion), depending on classpath checks that help to determine what optional conversion libraries are present. You can also set the message converters to use explicitly.

#### 1.1.3. Message Conversion

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs)

The `spring-web` module contains the `HttpMessageConverter` contract for reading and writing the body of HTTP requests and responses through `InputStream` and `OutputStream`. `HttpMessageConverter` instances are used on the client side (for example, in the `RestTemplate`) and on the server side (for example, in Spring MVC REST controllers).

Concrete implementations for the main media (MIME) types are provided in the framework and are, by default, registered with the `RestTemplate` on the client side and with `RequestMethodHandlerAdapter` on the server side (see [Configuring Message Converters](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-message-converters)).

The implementations of `HttpMessageConverter` are described in the following sections. For all converters, a default media type is used, but you can override it by setting the `supportedMediaTypes` bean property. The following table describes each implementation:

| MessageConverter                         | Description                                                  |
| :--------------------------------------- | :----------------------------------------------------------- |
| `StringHttpMessageConverter`             | An `HttpMessageConverter` implementation that can read and write `String` instances from the HTTP request and response. By default, this converter supports all text media types (`text/*`) and writes with a `Content-Type` of `text/plain`. |
| `FormHttpMessageConverter`               | An `HttpMessageConverter` implementation that can read and write form data from the HTTP request and response. By default, this converter reads and writes the `application/x-www-form-urlencoded` media type. Form data is read from and written into a `MultiValueMap<String, String>`. The converter can also write (but not read) multipart data read from a `MultiValueMap<String, Object>`. By default, `multipart/form-data` is supported. As of Spring Framework 5.2, additional multipart subtypes can be supported for writing form data. Consult the javadoc for `FormHttpMessageConverter` for further details. |
| `ByteArrayHttpMessageConverter`          | An `HttpMessageConverter` implementation that can read and write byte arrays from the HTTP request and response. By default, this converter supports all media types (`*/*`) and writes with a `Content-Type` of `application/octet-stream`. You can override this by setting the `supportedMediaTypes` property and overriding `getContentType(byte[])`. |
| `MarshallingHttpMessageConverter`        | An `HttpMessageConverter` implementation that can read and write XML by using Spring’s `Marshaller` and `Unmarshaller` abstractions from the `org.springframework.oxm` package. This converter requires a `Marshaller` and `Unmarshaller` before it can be used. You can inject these through constructor or bean properties. By default, this converter supports `text/xml` and `application/xml`. |
| `MappingJackson2HttpMessageConverter`    | An `HttpMessageConverter` implementation that can read and write JSON by using Jackson’s `ObjectMapper`. You can customize JSON mapping as needed through the use of Jackson’s provided annotations. When you need further control (for cases where custom JSON serializers/deserializers need to be provided for specific types), you can inject a custom `ObjectMapper` through the `ObjectMapper` property. By default, this converter supports `application/json`. |
| `MappingJackson2XmlHttpMessageConverter` | An `HttpMessageConverter` implementation that can read and write XML by using [Jackson XML](https://github.com/FasterXML/jackson-dataformat-xml) extension’s `XmlMapper`. You can customize XML mapping as needed through the use of JAXB or Jackson’s provided annotations. When you need further control (for cases where custom XML serializers/deserializers need to be provided for specific types), you can inject a custom `XmlMapper` through the `ObjectMapper` property. By default, this converter supports `application/xml`. |
| `SourceHttpMessageConverter`             | An `HttpMessageConverter` implementation that can read and write `javax.xml.transform.Source` from the HTTP request and response. Only `DOMSource`, `SAXSource`, and `StreamSource` are supported. By default, this converter supports `text/xml` and `application/xml`. |
| `BufferedImageHttpMessageConverter`      | An `HttpMessageConverter` implementation that can read and write `java.awt.image.BufferedImage` from the HTTP request and response. This converter reads and writes the media type supported by the Java I/O API. |

#### 1.1.4. Jackson JSON Views

You can specify a [Jackson JSON View](https://www.baeldung.com/jackson-json-view-annotation) to serialize only a subset of the object properties, as the following example shows:

```java
MappingJacksonValue value = new MappingJacksonValue(new User("eric", "7!jd#h23"));
value.setSerializationView(User.WithoutPasswordView.class);

RequestEntity<MappingJacksonValue> requestEntity =
    RequestEntity.post(new URI("https://example.com/user")).body(value);

ResponseEntity<String> response = template.exchange(requestEntity, String.class);
```

##### Multipart

To send multipart data, you need to provide a `MultiValueMap<String, Object>` whose values may be an `Object` for part content, a `Resource` for a file part, or an `HttpEntity` for part content with headers. For example:

```java
MultiValueMap<String, Object> parts = new LinkedMultiValueMap<>();

parts.add("fieldPart", "fieldValue");
parts.add("filePart", new FileSystemResource("...logo.png"));
parts.add("jsonPart", new Person("Jason"));

HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_XML);
parts.add("xmlPart", new HttpEntity<>(myBean, headers));
```

In most cases, you do not have to specify the `Content-Type` for each part. The content type is determined automatically based on the `HttpMessageConverter` chosen to serialize it or, in the case of a `Resource` based on the file extension. If necessary, you can explicitly provide the `MediaType` with an `HttpEntity` wrapper.

Once the `MultiValueMap` is ready, you can pass it to the `RestTemplate`, as show below:

```java
MultiValueMap<String, Object> parts = ...;
template.postForObject("https://example.com/upload", parts, Void.class);
```

If the `MultiValueMap` contains at least one non-`String` value, the `Content-Type` is set to `multipart/form-data` by the `FormHttpMessageConverter`. If the `MultiValueMap` has `String` values the `Content-Type` is defaulted to `application/x-www-form-urlencoded`. If necessary the `Content-Type` may also be set explicitly.

### 1.2. Using `AsyncRestTemplate` (Deprecated)

The `AsyncRestTemplate` is deprecated. For all use cases where you might consider using `AsyncRestTemplate`, use the [WebClient](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client) instead.

## 2. Remoting and Web Services

Spring provides support for remoting with various technologies. The remoting support eases the development of remote-enabled services, implemented via Java interfaces and objects as input and output. Currently, Spring supports the following remoting technologies:

- [Java Web Services](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-web-services): Spring provides remoting support for web services through JAX-WS.
- [AMQP](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-amqp): Remoting via AMQP as the underlying protocol is supported by the separate Spring AMQP project.

|      | As of Spring Framework 5.3, support for several remoting technologies is now deprecated for security reasons and broader industry support. Supporting infrastructure will be removed from Spring Framework for its next major release. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The following remoting technologies are now deprecated and will not be replaced:

- [Remote Method Invocation (RMI)](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-rmi): Through the use of `RmiProxyFactoryBean` and `RmiServiceExporter`, Spring supports both traditional RMI (with `java.rmi.Remote` interfaces and `java.rmi.RemoteException`) and transparent remoting through RMI invokers (with any Java interface).
- [Spring HTTP Invoker (Deprecated)](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-httpinvoker): Spring provides a special remoting strategy that allows for Java serialization though HTTP, supporting any Java interface (as the RMI invoker does). The corresponding support classes are `HttpInvokerProxyFactoryBean` and `HttpInvokerServiceExporter`.
- [Hessian](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-caucho-protocols-hessian): By using Spring’s `HessianProxyFactoryBean` and the `HessianServiceExporter`, you can transparently expose your services through the lightweight binary HTTP-based protocol provided by Caucho.
- [JMS (Deprecated)](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-jms): Remoting via JMS as the underlying protocol is supported through the `JmsInvokerServiceExporter` and `JmsInvokerProxyFactoryBean` classes in the `spring-jms` module.

While discussing the remoting capabilities of Spring, we use the following domain model and corresponding services:

```java
public class Account implements Serializable{

    private String name;

    public String getName(){
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
public interface AccountService {

    public void insertAccount(Account account);

    public List<Account> getAccounts(String name);
}
// the implementation doing nothing at the moment
public class AccountServiceImpl implements AccountService {

    public void insertAccount(Account acc) {
        // do something...
    }

    public List<Account> getAccounts(String name) {
        // do something...
    }
}
```

This section starts by exposing the service to a remote client by using RMI and talk a bit about the drawbacks of using RMI. It then continues with an example that uses Hessian as the protocol.

### 2.1. AMQP

Remoting via AMQP as the underlying protocol is supported in the Spring AMQP project. For further details please visit the [Spring Remoting](https://docs.spring.io/spring-amqp/docs/current/reference/html/#remoting) section of the Spring AMQP reference.

|      | Auto-detection is not implemented for remote interfacesThe main reason why auto-detection of implemented interfaces does not occur for remote interfaces is to avoid opening too many doors to remote callers. The target object might implement internal callback interfaces, such as `InitializingBean` or `DisposableBean` which one would not want to expose to callers.Offering a proxy with all interfaces implemented by the target usually does not matter in the local case. However, when you export a remote service, you should expose a specific service interface, with specific operations intended for remote usage. Besides internal callback interfaces, the target might implement multiple business interfaces, with only one of them intended for remote exposure. For these reasons, we require such a service interface to be specified.This is a trade-off between configuration convenience and the risk of accidental exposure of internal methods. Always specifying a service interface is not too much effort and puts you on the safe side regarding controlled exposure of specific methods. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.2. Considerations when Choosing a Technology

Each and every technology presented here has its drawbacks. When choosing a technology, you should carefully consider your needs, the services you expose, and the objects you send over the wire.

When using RMI, you cannot access the objects through the HTTP protocol, unless you tunnel the RMI traffic. RMI is a fairly heavy-weight protocol, in that it supports full-object serialization, which is important when you use a complex data model that needs serialization over the wire. However, RMI-JRMP is tied to Java clients. It is a Java-to-Java remoting solution.

Spring’s HTTP invoker is a good choice if you need HTTP-based remoting but also rely on Java serialization. It shares the basic infrastructure with RMI invokers but uses HTTP as transport. Note that HTTP invokers are not limited only to Java-to-Java remoting but also to Spring on both the client and the server side. (The latter also applies to Spring’s RMI invoker for non-RMI interfaces.)

Hessian might provide significant value when operating in a heterogeneous environment, because they explicitly allow for non-Java clients. However, non-Java support is still limited. Known issues include the serialization of Hibernate objects in combination with lazily-initialized collections. If you have such a data model, consider using RMI or HTTP invokers instead of Hessian.

JMS can be useful for providing clusters of services and letting the JMS broker take care of load balancing, discovery, and auto-failover. By default, Java serialization is used for JMS remoting, but the JMS provider could use a different mechanism for the wire formatting, such as XStream to let servers be implemented in other technologies.

Last but not least, EJB has an advantage over RMI, in that it supports standard role-based authentication and authorization and remote transaction propagation. It is possible to get RMI invokers or HTTP invokers to support security context propagation as well, although this is not provided by core Spring. Spring offers only appropriate hooks for plugging in third-party or custom solutions.

### 2.3. Java Web Services

Spring provides full support for the standard Java web services APIs:

- Exposing web services using JAX-WS
- Accessing web services using JAX-WS

In addition to stock support for JAX-WS in Spring Core, the Spring portfolio also features [Spring Web Services](http://www.springframework.org/spring-ws), which is a solution for contract-first, document-driven web services — highly recommended for building modern, future-proof web services.

#### 2.3.1. Exposing Servlet-based Web Services by Using JAX-WS

Spring provides a convenient base class for JAX-WS servlet endpoint implementations: `SpringBeanAutowiringSupport`. To expose our `AccountService`, we extend Spring’s `SpringBeanAutowiringSupport` class and implement our business logic here, usually delegating the call to the business layer. We use Spring’s `@Autowired` annotation to express such dependencies on Spring-managed beans. The following example shows our class that extends `SpringBeanAutowiringSupport`:

```java
/**
 * JAX-WS compliant AccountService implementation that simply delegates
 * to the AccountService implementation in the root web application context.
 *
 * This wrapper class is necessary because JAX-WS requires working with dedicated
 * endpoint classes. If an existing service needs to be exported, a wrapper that
 * extends SpringBeanAutowiringSupport for simple Spring bean autowiring (through
 * the @Autowired annotation) is the simplest JAX-WS compliant way.
 *
 * This is the class registered with the server-side JAX-WS implementation.
 * In the case of a Java EE server, this would simply be defined as a servlet
 * in web.xml, with the server detecting that this is a JAX-WS endpoint and reacting
 * accordingly. The servlet name usually needs to match the specified WS service name.
 *
 * The web service engine manages the lifecycle of instances of this class.
 * Spring bean references will just be wired in here.
 */
import org.springframework.web.context.support.SpringBeanAutowiringSupport;

@WebService(serviceName="AccountService")
public class AccountServiceEndpoint extends SpringBeanAutowiringSupport {

    @Autowired
    private AccountService biz;

    @WebMethod
    public void insertAccount(Account acc) {
        biz.insertAccount(acc);
    }

    @WebMethod
    public Account[] getAccounts(String name) {
        return biz.getAccounts(name);
    }
}
```

Our `AccountServiceEndpoint` needs to run in the same web application as the Spring context to allow for access to Spring’s facilities. This is the case by default in Java EE environments, using the standard contract for JAX-WS servlet endpoint deployment. See the various Java EE web service tutorials for details.

#### 2.3.2. Exporting Standalone Web Services by Using JAX-WS

The built-in JAX-WS provider that comes with Oracle’s JDK supports exposure of web services by using the built-in HTTP server that is also included in the JDK. Spring’s `SimpleJaxWsServiceExporter` detects all `@WebService`-annotated beans in the Spring application context and exports them through the default JAX-WS server (the JDK HTTP server).

In this scenario, the endpoint instances are defined and managed as Spring beans themselves. They are registered with the JAX-WS engine, but their lifecycle is up to the Spring application context. This means that you can apply Spring functionality (such as explicit dependency injection) to the endpoint instances. Annotation-driven injection through `@Autowired` works as well. The following example shows how to define these beans:

```xml
<bean class="org.springframework.remoting.jaxws.SimpleJaxWsServiceExporter">
    <property name="baseAddress" value="http://localhost:8080/"/>
</bean>

<bean id="accountServiceEndpoint" class="example.AccountServiceEndpoint">
    ...
</bean>

...
```

The `AccountServiceEndpoint` can but does not have to derive from Spring’s `SpringBeanAutowiringSupport`, since the endpoint in this example is a fully Spring-managed bean. This means that the endpoint implementation can be as follows (without any superclass declared — and Spring’s `@Autowired` configuration annotation is still honored):

```java
@WebService(serviceName="AccountService")
public class AccountServiceEndpoint {

    @Autowired
    private AccountService biz;

    @WebMethod
    public void insertAccount(Account acc) {
        biz.insertAccount(acc);
    }

    @WebMethod
    public List<Account> getAccounts(String name) {
        return biz.getAccounts(name);
    }
}
```

#### 2.3.3. Exporting Web Services by Using JAX-WS RI’s Spring Support

Oracle’s JAX-WS RI, developed as part of the GlassFish project, ships Spring support as part of its JAX-WS Commons project. This allows for defining JAX-WS endpoints as Spring-managed beans, similar to the standalone mode discussed in the [previous section](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-web-services-jaxws-export-standalone) — but this time in a Servlet environment.

|      | This is not portable in a Java EE environment. It is mainly intended for non-EE environments, such as Tomcat, that embed the JAX-WS RI as part of the web application. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The differences from the standard style of exporting servlet-based endpoints are that the lifecycle of the endpoint instances themselves are managed by Spring and that there is only one JAX-WS servlet defined in `web.xml`. With the standard Java EE style (as shown earlier), you have one servlet definition per service endpoint, with each endpoint typically delegating to Spring beans (through the use of `@Autowired`, as shown earlier).

See https://jax-ws-commons.java.net/spring/ for details on setup and usage style.

#### 2.3.4. Accessing Web Services by Using JAX-WS

Spring provides two factory beans to create JAX-WS web service proxies, namely `LocalJaxWsServiceFactoryBean` and `JaxWsPortProxyFactoryBean`. The former can return only a JAX-WS service class for us to work with. The latter is the full-fledged version that can return a proxy that implements our business service interface. In the following example, we use `JaxWsPortProxyFactoryBean` to create a proxy for the `AccountService` endpoint (again):

```xml
<bean id="accountWebService" class="org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean">
    <property name="serviceInterface" value="example.AccountService"/> 
    <property name="wsdlDocumentUrl" value="http://localhost:8888/AccountServiceEndpoint?WSDL"/>
    <property name="namespaceUri" value="https://example/"/>
    <property name="serviceName" value="AccountService"/>
    <property name="portName" value="AccountServiceEndpointPort"/>
</bean>
```

|      | Where `serviceInterface` is our business interface that the clients use. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`wsdlDocumentUrl` is the URL for the WSDL file. Spring needs this at startup time to create the JAX-WS Service. `namespaceUri` corresponds to the `targetNamespace` in the .wsdl file. `serviceName` corresponds to the service name in the .wsdl file. `portName` corresponds to the port name in the .wsdl file.

Accessing the web service is easy, as we have a bean factory for it that exposes it as an interface called `AccountService`. The following example shows how we can wire this up in Spring:

```xml
<bean id="client" class="example.AccountClientImpl">
    ...
    <property name="service" ref="accountWebService"/>
</bean>
```

From the client code, we can access the web service as if it were a normal class, as the following example shows:

```java
public class AccountClientImpl {

    private AccountService service;

    public void setService(AccountService service) {
        this.service = service;
    }

    public void foo() {
        service.insertAccount(...);
    }
}
```

|      | The above is slightly simplified in that JAX-WS requires endpoint interfaces and implementation classes to be annotated with `@WebService`, `@SOAPBinding` etc annotations. This means that you cannot (easily) use plain Java interfaces and implementation classes as JAX-WS endpoint artifacts; you need to annotate them accordingly first. Check the JAX-WS documentation for details on those requirements. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.4. RMI (Deprecated)

|      | As of Spring Framework 5.3, RMI support is deprecated and will not be replaced. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

By using Spring’s support for RMI, you can transparently expose your services through the RMI infrastructure. After having this set up, you basically have a configuration similar to remote EJBs, except for the fact that there is no standard support for security context propagation or remote transaction propagation. Spring does provide hooks for such additional invocation context when you use the RMI invoker, so you can, for example, plug in security frameworks or custom security credentials.

#### 2.4.1. Exporting the Service by Using `RmiServiceExporter`

Using the `RmiServiceExporter`, we can expose the interface of our AccountService object as RMI object. The interface can be accessed by using `RmiProxyFactoryBean`, or via plain RMI in case of a traditional RMI service. The `RmiServiceExporter` explicitly supports the exposing of any non-RMI services via RMI invokers.

We first have to set up our service in the Spring container. The following example shows how to do so:

```xml
<bean id="accountService" class="example.AccountServiceImpl">
    <!-- any additional properties, maybe a DAO? -->
</bean>
```

Next, we have to expose our service by using `RmiServiceExporter`. The following example shows how to do so:

```xml
<bean class="org.springframework.remoting.rmi.RmiServiceExporter">
    <!-- does not necessarily have to be the same name as the bean to be exported -->
    <property name="serviceName" value="AccountService"/>
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
    <!-- defaults to 1099 -->
    <property name="registryPort" value="1199"/>
</bean>
```

In the preceding example, we override the port for the RMI registry. Often, your application server also maintains an RMI registry, and it is wise to not interfere with that one. Furthermore, the service name is used to bind the service. So, in the preceding example, the service is bound at `'rmi://HOST:1199/AccountService'`. We use this URL later on to link in the service at the client side.

|      | The `servicePort` property has been omitted (it defaults to 0). This means that an anonymous port is used to communicate with the service. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.4.2. Linking in the Service at the Client

Our client is a simple object that uses the `AccountService` to manage accounts, as the following example shows:

```java
public class SimpleObject {

    private AccountService accountService;

    public void setAccountService(AccountService accountService) {
        this.accountService = accountService;
    }

    // additional methods using the accountService
}
```

To link in the service on the client, we create a separate Spring container, to contain the following simple object and the service linking configuration bits:

```xml
<bean class="example.SimpleObject">
    <property name="accountService" ref="accountService"/>
</bean>

<bean id="accountService" class="org.springframework.remoting.rmi.RmiProxyFactoryBean">
    <property name="serviceUrl" value="rmi://HOST:1199/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

That is all we need to do to support the remote account service on the client. Spring transparently creates an invoker and remotely enables the account service through the `RmiServiceExporter`. At the client, we link it in by using the `RmiProxyFactoryBean`.

### 2.5. Using Hessian to Remotely Call Services through HTTP (Deprecated)

|      | As of Spring Framework 5.3, Hessian support is deprecated and will not be replaced. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Hessian offers a binary HTTP-based remoting protocol. It is developed by Caucho, and you can find more information about Hessian itself at https://www.caucho.com/.

#### 2.5.1. Hessian

Hessian communicates through HTTP and does so by using a custom servlet. By using Spring’s `DispatcherServlet` principles (see [webmvc.html](https://docs.spring.io/spring-framework/docs/current/reference/html/webmvc.html#mvc-servlet)), we can wire up such a servlet to expose your services. First, we have to create a new servlet in our application, as shown in the following excerpt from `web.xml`:

```xml
<servlet>
    <servlet-name>remoting</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>remoting</servlet-name>
    <url-pattern>/remoting/*</url-pattern>
</servlet-mapping>
```

If you are familiar with Spring’s `DispatcherServlet` principles, you probably know that now you have to create a Spring container configuration resource named `remoting-servlet.xml` (after the name of your servlet) in the `WEB-INF` directory. The application context is used in the next section.

或者，考虑使用Spring的simpler `HttpRequestHandlerServlet`。这样做可以使您将远程导出器定义嵌入到根应用程序上下文中（默认情况下，在中`WEB-INF/applicationContext.xml`），并使用指向特定导出器bean的单个servlet定义。在这种情况下，每个servlet名称都必须与其目标导出器的bean名称相匹配。

#### 2.5.2。使用公开您的豆子`HessianServiceExporter`

在名为的新创建的应用程序上下文中`remoting-servlet.xml`，我们创建一个 `HessianServiceExporter`来导出我们的服务，如以下示例所示：

```xml
<bean id="accountService" class="example.AccountServiceImpl">
    <!-- any additional properties, maybe a DAO? -->
</bean>

<bean name="/AccountService" class="org.springframework.remoting.caucho.HessianServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

现在，我们准备在客户端链接服务。没有指定显式处理程序映射（将请求URL映射到服务），因此我们使用`BeanNameUrlHandlerMapping` used。因此，将在包含`DispatcherServlet`实例的映射（如先前定义）中 通过其bean名称指示的URL导出服务`https://HOST:8080/remoting/AccountService`。

或者，您可以`HessianServiceExporter`在根应用程序上下文中创建一个（例如，在中`WEB-INF/applicationContext.xml`），如以下示例所示：

```xml
<bean name="accountExporter" class="org.springframework.remoting.caucho.HessianServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

在后一种情况下，您应该在中为该导出器定义一个相应的servlet `web.xml`，并得到相同的最终结果：导出器被映射到位于的请求路径 `/remoting/AccountService`。注意，servlet名称需要与目标导出器的bean名称匹配。以下示例显示了如何执行此操作：

```xml
<servlet>
    <servlet-name>accountExporter</servlet-name>
    <servlet-class>org.springframework.web.context.support.HttpRequestHandlerServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>accountExporter</servlet-name>
    <url-pattern>/remoting/AccountService</url-pattern>
</servlet-mapping>
```

#### 2.5.3。在客户端上链接服务

通过使用`HessianProxyFactoryBean`，我们可以在客户端链接服务。与RMI示例相同的原理适用。我们创建一个单独的bean工厂或应用程序上下文，并`SimpleObject`通过使用`AccountService`来管理帐户来提及以下bean ，如以下示例所示：

```xml
<bean class="example.SimpleObject">
    <property name="accountService" ref="accountService"/>
</bean>

<bean id="accountService" class="org.springframework.remoting.caucho.HessianProxyFactoryBean">
    <property name="serviceUrl" value="https://remotehost:8080/remoting/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

#### 2.5.4。将HTTP基本身份验证应用于通过Hessian公开的服务

Hessian的优点之一是我们可以轻松地应用HTTP基本身份验证，因为这两种协议都是基于HTTP的。`web.xml`例如，可以通过使用安全功能来应用常规的HTTP服务器安全性机制。通常，您无需在此处使用每个用户的安全凭证。相反，您可以使用在该`HessianProxyFactoryBean`级别定义的共享凭据（类似于JDBC `DataSource`），如以下示例所示：

```xml
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping">
    <property name="interceptors" ref="authorizationInterceptor"/>
</bean>

<bean id="authorizationInterceptor"
        class="org.springframework.web.servlet.handler.UserRoleAuthorizationInterceptor">
    <property name="authorizedRoles" value="administrator,operator"/>
</bean>
```

在前面的示例中，我们明确提到`BeanNameUrlHandlerMapping`并设置了一个拦截器，以仅让管理员和操作员调用此应用程序上下文中提到的bean。

|      | 前面的示例未显示灵活的安全基础结构。有关安全性的更多选项，请参阅https://projects.spring.io/spring-security/上的Spring Security项目。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.6。Spring HTTP Invoker（不建议使用）

|      | 从Spring Framework 5.3开始，不推荐使用HTTP Invoker支持，并且不会替换。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

与Hessian相反，Spring HTTP调用程序都是轻量级协议，它们使用自己的苗条序列化机制，并使用标准Java序列化机制通过HTTP公开服务。如果您的参数和返回类型是无法通过使用Hessian使用的序列化机制进行序列化的复杂类型，则这将具有巨大的优势（有关选择远程处理技术的更多注意事项，请参阅下一节）。

在幕后，Spring使用JDK或Apache提供的标准功能`HttpComponents`来执行HTTP调用。如果您需要更高级且更易于使用的功能，请使用后者。有关 更多信息，请参见 [hc.apache.org/httpcomponents-client-ga/](https://hc.apache.org/httpcomponents-client-ga/)。

|      | 注意由于不安全的Java反序列化而导致的漏洞：操纵输入流可能会在反序列化步骤中导致服务器上不必要的代码执行。因此，请勿将HTTP调用方终结点暴露给不受信任的客户端。而是仅在您自己的服务之间公开它们。通常，强烈建议您改用其他任何消息格式（例如JSON）。如果您担心由Java序列化引起的安全漏洞，请考虑在核心JVM级别上使用通用序列化筛选器机制，该机制最初是为JDK 9开发的，但同时又移植到了JDK 8、7和6。参见 https://blogs.oracle.com/java-platform-group/entry/incoming_filter_serialization_data_a 和https://openjdk.java.net/jeps/290。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.6.1。公开服务对象

为服务对象设置HTTP调用程序基础结构与使用Hessian进行设置的方法非常相似。正如Hessian支持提供的那样 `HessianServiceExporter`，Spring的HttpInvoker支持提供了 `org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter`。

为了`AccountService`在Spring Web MVC中公开（前面提到的） `DispatcherServlet`，需要在调度程序的应用程序上下文中进行以下配置，如以下示例所示：

```xml
<bean name="/AccountService" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

此类导出器定义通过`DispatcherServlet`实例的标准映射功能公开，如[有关Hessian的部分所述](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-caucho-protocols)。

或者，您可以`HttpInvokerServiceExporter`在根应用程序上下文中创建一个（例如，在中`'WEB-INF/applicationContext.xml'`），如以下示例所示：

```xml
<bean name="accountExporter" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

另外，您可以在中为该导出器定义一个相应的servlet，该`web.xml`servlet名称与目标导出器的bean名称匹配，如以下示例所示：

```xml
<servlet>
    <servlet-name>accountExporter</servlet-name>
    <servlet-class>org.springframework.web.context.support.HttpRequestHandlerServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>accountExporter</servlet-name>
    <url-pattern>/remoting/AccountService</url-pattern>
</servlet-mapping>
```

#### 2.6.2。在客户端链接服务

同样，从客户端链接服务非常类似于使用Hessian时的方式。通过使用代理，Spring可以将对HTTP POST请求的调用转换为指向导出服务的URL。以下示例显示如何配置此安排：

```xml
<bean id="httpInvokerProxy" class="org.springframework.remoting.httpinvoker.HttpInvokerProxyFactoryBean">
    <property name="serviceUrl" value="https://remotehost:8080/remoting/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

如前所述，您可以选择要使用的HTTP客户端。默认情况下，会 `HttpInvokerProxy`使用JDK的HTTP功能，但是您也可以`HttpComponents`通过设置`httpInvokerRequestExecutor`属性来使用Apache 客户端。以下示例显示了如何执行此操作：

```xml
<property name="httpInvokerRequestExecutor">
    <bean class="org.springframework.remoting.httpinvoker.HttpComponentsHttpInvokerRequestExecutor"/>
</property>
```

### 2.7。JMS（已弃用）

|      | 从Spring Framework 5.3开始，已弃用JMS远程支持，并且将不会替换。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以通过使用JMS作为基础通信协议来透明地公开服务。Spring框架中的JMS远程支持非常基本。它在`same thread`和上以相同的非事务发送和接收`Session`。结果，吞吐量取决于实现方式。请注意，这些单线程和非事务性约束仅适用于Spring的JMS远程支持。有关Spring对基于JMS的消息传递的丰富支持的信息，请参见[JMS（Java消息服务）](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms)。

服务器和客户端均使用以下接口：

```java
package com.foo;

public interface CheckingAccountService {

    public void cancelAccount(Long accountId);
}
```

在服务器端使用上述接口的以下简单实现：

```java
package com.foo;

public class SimpleCheckingAccountService implements CheckingAccountService {

    public void cancelAccount(Long accountId) {
        System.out.println("Cancelling account [" + accountId + "]");
    }
}
```

以下配置文件包含在客户端和服务器上共享的JMS基础结构Bean：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="connectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://ep-t43:61616"/>
    </bean>

    <bean id="queue" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg value="mmm"/>
    </bean>

</beans>
```

#### 2.7.1。服务器端配置

在服务器上，您需要公开使用的服务对象 `JmsInvokerServiceExporter`，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="checkingAccountService"
            class="org.springframework.jms.remoting.JmsInvokerServiceExporter">
        <property name="serviceInterface" value="com.foo.CheckingAccountService"/>
        <property name="service">
            <bean class="com.foo.SimpleCheckingAccountService"/>
        </property>
    </bean>

    <bean class="org.springframework.jms.listener.SimpleMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="destination" ref="queue"/>
        <property name="concurrentConsumers" value="3"/>
        <property name="messageListener" ref="checkingAccountService"/>
    </bean>

</beans>
package com.foo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Server {

    public static void main(String[] args) throws Exception {
        new ClassPathXmlApplicationContext("com/foo/server.xml", "com/foo/jms.xml");
    }
}
```

#### 2.7.2。客户端配置

客户端只需要创建一个客户端代理即可实现约定的接口（`CheckingAccountService`）。

下面的示例定义了可以注入到其他客户端对象中的bean（代理负责通过JMS将调用转发到服务器端对象）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="checkingAccountService"
            class="org.springframework.jms.remoting.JmsInvokerProxyFactoryBean">
        <property name="serviceInterface" value="com.foo.CheckingAccountService"/>
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="queue" ref="queue"/>
    </bean>

</beans>
package com.foo;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Client {

    public static void main(String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("com/foo/client.xml", "com/foo/jms.xml");
        CheckingAccountService service = (CheckingAccountService) ctx.getBean("checkingAccountService");
        service.cancelAccount(new Long(10));
    }
}
```

## 3.企业JavaBeans（EJB）集成

作为轻量级容器，Spring通常被认为是EJB的替代品。我们确实相信，对于许多（即使不是大多数）应用程序和用例，Spring作为容器，结合其在事务，ORM和JDBC访问方面的丰富支持功能，比通过EJB实现等效功能是更好的选择。容器和EJB。

但是，请务必注意，使用Spring不会阻止您使用EJB。实际上，Spring使访问EJB以及在其中实现EJB和功能变得更加容易。另外，使用Spring访问EJB提供的服务可以使这些服务的实现稍后在本地EJB，远程EJB或POJO（普通旧Java对象）变体之间透明切换，而不必更改客户端代码。

在本章中，我们将研究Spring如何帮助您访问和实现EJB。当访问无状态会话Bean（SLSB）时，Spring提供了特殊的价值，因此我们从讨论这个主题开始。

### 3.1。访问EJB

本节介绍如何访问EJB。

#### 3.1.1。概念

要在本地或远程无状态会话Bean上调用方法，客户机代码通常必须执行JNDI查找以获取（本地或远程）EJB Home对象，然后`create`对该对象使用方法调用以获取实际的（本地或远程）EJB对象。EJB对象。然后在EJB上调用一种或多种方法。

为了避免重复的低级代码，许多EJB应用程序都使用服务定位器和业务委托模式。这些比在整个客户端代码中喷射JNDI查找要好，但是它们的常规实现有很多缺点：

- 通常，使用EJB的代码取决于Service Locator或Business Delegate单例，因此很难进行测试。
- 在不使用业务委托的情况下使用服务定位器模式的情况下，应用程序代码仍然最终必须`create()`在EJB home上调用该方法并处理产生的异常。因此，它仍然与EJB API和EJB编程模型的复杂性联系在一起。
- 实现业务委托模式通常会导致大量的代码重复，我们必须编写许多在EJB上调用相同方法的方法。

Spring的方法是允许创建和使用代理对象（通常在Spring容器内配置），这些代理对象充当无代码的业务委托。除非您在此类代码中实际添加了实际价值，否则您无需在手动编码的业务委托中编写另一个Service Locator，另一个JNDI查找或重复方法。

#### 3.1.2。访问本地SLSB

假设我们有一个Web控制器，需要使用本地EJB。我们遵循最佳实践，并使用EJB业务方法接口模式，以便EJB的本地接口扩展了非EJB特定的业务方法接口。我们称这种业务方法为接口`MyComponent`。以下示例显示了这样的接口：

```java
public interface MyComponent {
    ...
}
```

使用业务方法接口模式的主要原因之一是确保本地接口中的方法签名与bean实现类之间的同步是自动的。另一个原因是，如果有必要的话，以后可以使我们更轻松地切换到服务的POJO（普通旧Java对象）实现。我们还需要实现本地home接口，并提供一个实现类`SessionBean`和`MyComponent`业务方法接口的实现类。现在，将Web层控制器连接到EJB实现所需的唯一Java编码是`MyComponent` 在控制器上公开类型的setter方法。这会将引用另存为控制器中的实例变量。以下示例显示了如何执行此操作：

```java
private MyComponent myComponent;

public void setMyComponent(MyComponent myComponent) {
    this.myComponent = myComponent;
}
```

随后，我们可以在控制器中的任何业务方法中使用此实例变量。现在，假设我们从Spring容器中获取了控制器对象，我们可以（在相同上下文中）配置一个`LocalStatelessSessionProxyFactoryBean`实例，即EJB代理对象。我们配置代理，并`myComponent`使用以下配置条目设置控制器的 属性：

```xml
<bean id="myComponent"
        class="org.springframework.ejb.access.LocalStatelessSessionProxyFactoryBean">
    <property name="jndiName" value="ejb/myBean"/>
    <property name="businessInterface" value="com.mycom.MyComponent"/>
</bean>

<bean id="myController" class="com.mycom.myController">
    <property name="myComponent" ref="myComponent"/>
</bean>
```

尽管没有强迫您使用AOP概念来欣赏结果，但仍要依靠Spring AOP框架在幕后进行大量工作。该 `myComponent`bean定义创建的EJB，它实现了业务方法接口的代理。EJB本地宿主在启动时被缓存，因此只有一个JNDI查找。每次调用EJB时，代理都会`classname`在本地EJB上调用方法，并在EJB上调用相应的业务方法。

该`myController`bean定义设置`myComponent`控制器类到EJB代理财产。

或者（最好在许多此类代理定义的情况下），考虑`<jee:local-slsb>`在Spring的“ jee”命名空间中使用配置元素。以下示例显示了如何执行此操作：

```xml
<jee:local-slsb id="myComponent" jndi-name="ejb/myBean"
        business-interface="com.mycom.MyComponent"/>

<bean id="myController" class="com.mycom.myController">
    <property name="myComponent" ref="myComponent"/>
</bean>
```

这种EJB访问机制极大地简化了应用程序代码。Web层代码（或其他EJB客户端代码）与EJB的使用无关。要用POJO或模拟对象或其他测试存根替换该EJB引用，我们可以更改`myComponent`Bean定义而无需更改一行Java代码。此外，作为应用程序的一部分，我们不必编写任何一行JNDI查找或其他EJB管道代码。

实际应用中的基准和经验表明，这种方法的性能开销（涉及目标EJB的反射调用）是最小的，并且在常规使用中是无法检测到的。请记住，无论如何我们都不希望对EJB进行细粒度的调用，因为与应用程序服务器中的EJB基础结构相关联的成本很高。

关于JNDI查找有一个警告。在bean容器中，此类通常最好用作单例（没有理由使其成为原型）。但是，如果该bean容器预先实例化了单例（与各种XML `ApplicationContext`变体一样），那么如果在EJB容器加载目标EJB之前加载了bean容器，则可能会出现问题。这是因为JNDI查找是在`init()`此类的方法中执行的，然后进行了缓存，但是EJB尚未绑定到目标位置。解决方案是不预先实例化该工厂对象，而是让它在首次使用时创建。在XML容器中，您可以使用`lazy-init`属性来控制它。

尽管大多数Spring用户都不感兴趣，但是那些使用EJB进行编程AOP的人可能要看一下`LocalSlsbInvokerInterceptor`。

#### 3.1.3。访问远程SLSB

访问远程EJB与访问本地EJB本质上相同，只是使用了 `SimpleRemoteStatelessSessionProxyFactoryBean`or`<jee:remote-slsb>`配置元素。当然，无论是否使用Spring，远程调用语义都适用：调用另一台计算机上另一台VM中的对象上的方法时，有时在使用情况和故障处理方面必须区别对待。

与非Spring方法相比，Spring的EJB客户端支持增加了另一优势。通常，在本地或远程调用EJB之间轻松地来回切换EJB客户端代码是有问题的。这是因为远程接口方法必须声明它们throw `RemoteException`，而客户端代码必须对此进行处理，而本地接口方法则不需要。通常需要修改为需要移至远程EJB的本地EJB编写的客户端代码，以添加对远程异常的处理，为需要移至本地EJB的远程EJB编写的客户端代码可以保持不变，但可以执行以下操作：许多不必要的远程异常处理，或进行修改以删除该代码。使用Spring远程EJB代理，您可以声明不抛出任何异常`RemoteException`在您的业务方法接口和实现EJB代码中，拥有一个完全相同的远程接口（除了它确实抛出`RemoteException`），并且依靠代理来动态地将这两个接口视为相同。也就是说，客户端代码不必处理已检查的`RemoteException`类。`RemoteException`在EJB调用期间抛出的所有实数都将重新抛出为未经检查的`RemoteAccessException`类，该类是的子类`RuntimeException`。然后，您可以在本地EJB或远程EJB（甚至纯Java对象）实现之间随意切换目标服务，而无需了解或关心客户端代码。当然，这是可选的：没有什么可以阻止您`RemoteException`在业务界面中进行声明。

#### 3.1.4。访问EJB 2.x SLSB与EJB 3 SLSB

通过Spring访问EJB 2.x会话Bean和EJB 3会话Bean在很大程度上是透明的。Spring的EJB访问器（包括`<jee:local-slsb>`和 `<jee:remote-slsb>`设施）在运行时透明地适应实际组件。它们会处理Home接口（如果找到）（EJB 2.x样式），或者在没有Home接口可用时（EJB 3样式）执行直接组件调用。

注意：对于EJB 3会话Bean，您也可以有效地使用`JndiObjectFactoryBean`/ `<jee:jndi-lookup>`，因为公开了完全可用的组件引用以用于在那里的普通JNDI查找。定义显式`<jee:local-slsb>`或`<jee:remote-slsb>` 查找可提供一致且更显式的EJB访问配置。

## 4. JMS（Java消息服务）

Spring提供了一个JMS集成框架，该框架简化了JMS API的使用，就像Spring对JDBC API的集成一样。

JMS可以大致分为两个功能区域，即消息的产生和使用。所述`JmsTemplate`类用于消息生成和同步消息接收操作。对于类似于Java EE的消息驱动bean样式的异步接收，Spring提供了许多消息侦听器容器，可用于创建消息驱动POJO（MDP）。Spring还提供了一种声明式方法来创建消息侦听器。

该`org.springframework.jms.core`软件包提供了使用JMS的核心功能。它包含JMS模板类，该类通过处理资源的创建和释放来简化JMS的使用，就像`JdbcTemplate`JDBC一样。Spring模板类共有的设计原则是提供帮助器方法来执行常用操作，并且对于更复杂的用法，将处理任务的实质委托给用户实现的回调接口。JMS模板遵循相同的设计。这些类提供了各种方便的方法，用于发送消息，同步使用消息以及向用户公开JMS会话和消息生成器。

该`org.springframework.jms.support`软件包提供`JMSException`翻译功能。转换将检查的`JMSException`层次结构转换为未检查的异常的镜像层次结构。如果`javax.jms.JMSException`存在checked的任何提供程序特定的子类，则将此异常包装在unchecked中`UncategorizedJmsException`。

该`org.springframework.jms.support.converter`软件包提供了一个`MessageConverter` 抽象，可以在Java对象和JMS消息之间进行转换。

该`org.springframework.jms.support.destination`软件包提供了用于管理JMS目的地的各种策略，例如为JNDI中存储的目的地提供服务定位器。

该`org.springframework.jms.annotation`软件包通过提供了必要的基础结构来支持注释驱动的侦听器端点`@JmsListener`。

该`org.springframework.jms.config`软件包提供了`jms`名称空间的解析器实现 以及java config支持，以配置侦听器容器和创建侦听器端点。

最后，该`org.springframework.jms.connection`软件包提供了`ConnectionFactory`适用于独立应用程序的实现。它还包含Spring `PlatformTransactionManager`for JMS的实现（巧妙地命名为 `JmsTransactionManager`）。这允许将JMS作为事务资源无缝集成到Spring的事务管理机制中。

|      | 从Spring Framework 5开始，Spring的JMS软件包完全支持JMS 2.0，并要求在运行时提供JMS 2.0 API。我们建议使用JMS 2.0兼容的提供程序。如果您碰巧在系统中使用了较旧的消息代理，则可以尝试为现有代理生成升级到JMS 2.0兼容驱动程序。或者，您也可以尝试在基于JMS 1.1的驱动程序上运行，只需将JMS 2.0 API jar放在类路径上，而仅对驱动程序使用兼容JMS 1.1的API。Spring的JMS支持默认情况下遵守JMS 1.1约定，因此通过相应的配置，它确实支持这种情况。但是，请仅在过渡方案中考虑这一点。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.1。使用Spring JMS

本节描述如何使用Spring的JMS组件。

#### 4.1.1。使用`JmsTemplate`

的`JmsTemplate`类是在JMS芯封装的核心类。由于它在发送或同步接收消息时处理资源的创建和释放，因此它简化了JMS的使用。

```
JmsTemplate`仅使用需求的代码即可实现为回调接口提供清晰定义的高级协定。当中的调用代码提供`MessageCreator`时，回调接口会创建一条消息。为了允许更复杂地使用JMS API，提供JMS会话并公开和 对。`Session``JmsTemplate``SessionCallback``ProducerCallback``Session``MessageProducer
```

JMS API公开了两种类型的发送方法，一种采用交付模式，优先级和生存时间作为服务质量（QOS）参数，另一种不采用QOS参数并使用默认值。由于`JmsTemplate`有许多发送方法，因此设置QOS参数已作为bean属性公开，以避免重复发送方法。同样，使用`setReceiveTimeout`属性设置同步接收调用的超时值。

一些JMS提供程序允许通过配置来管理默认QOS值`ConnectionFactory`。这样做的结果是，对`MessageProducer`实例`send`方法（`send(Destination destination, Message message)`）的调用 使用与JMS规范中指定的QOS默认值不同的QOS默认值。为了提供QOS值的一致管理，`JmsTemplate`必须的，因此，专门启用通过设置布尔值属性使用自己的QOS值 `isExplicitQosEnabled`到`true`。

为了方便起见，`JmsTemplate`还公开了一个基本的请求-答复操作，该操作允许发送消息并等待作为该操作一部分创建的临时队列的答复。

|      | `JmsTemplate`一旦配置，该类的 实例是线程安全的。这很重要，因为这意味着您可以配置的单个实例，`JmsTemplate` 然后将该共享引用安全地注入多个协作者中。需要明确的`JmsTemplate`是，状态是有状态的，因为它保持对的引用 `ConnectionFactory`，但是此状态不是会话状态。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

从Spring Framework 4.1开始，它`JmsMessagingTemplate`是基于`JmsTemplate` 消息传递抽象构建的，并且提供了与消息传递抽象的集成，即 `org.springframework.messaging.Message`。这使您可以创建以通用方式发送的消息。

#### 4.1.2。连接数

该`JmsTemplate`要求的一个参考`ConnectionFactory`。它`ConnectionFactory` 是JMS规范的一部分，并且是使用JMS的切入点。客户端应用程序将其用作工厂以创建与JMS提供程序的连接，并封装各种配置参数，其中许多是特定于供应商的，例如SSL配置选项。

当在EJB中使用JMS时，供应商提供JMS接口的实现，以便它们可以参与声明式事务管理并执行连接和会话的池化。为了使用此实现，Java EE容器通常要求您将JMS连接工厂声明为`resource-ref`EJB或Servlet部署描述符中的内部。为了确保在`JmsTemplate`EJB内部使用这些功能 ，客户端应用程序应确保其引用的托管实现`ConnectionFactory`。

##### 缓存消息资源

标准API涉及创建许多中间对象。要发送消息，请执行以下“ API”遍历：

```
ConnectionFactory->连接->会话-> MessageProducer->发送
```

在`ConnectionFactory`和`Send`操作之间，创建并销毁了三个中间对象。为了优化资源使用并提高性能，Spring提供了两种实现`ConnectionFactory`。

##### 使用 `SingleConnectionFactory`

Spring提供了`ConnectionFactory`接口的实现，该接口 在所有调用`SingleConnectionFactory`中返回相同的内容`Connection`， `createConnection()`并忽略对的调用`close()`。这对于测试和独立环境非常有用，因此同一连接可用于`JmsTemplate`可能跨越任意数量事务的多个 调用。`SingleConnectionFactory` 引用了`ConnectionFactory`通常来自JNDI的标准。

##### 使用 `CachingConnectionFactory`

该`CachingConnectionFactory`扩展的功能`SingleConnectionFactory` ，并增加了缓存`Session`，`MessageProducer`和`MessageConsumer`实例。初始缓存大小设置为`1`。您可以使用该`sessionCacheSize`属性来增加缓存的会话数。请注意，由于根据会话的确认模式缓存会话，因此实际缓存的会话数大于该数量，因此当`sessionCacheSize`设置为one时，最多可以有四个缓存的会话实例（每个确认模式一个）。`MessageProducer`和`MessageConsumer`实例在其自己的会话中缓存，并且在缓存时还要考虑生产者和使用者的独特属性。MessageProducers根据其目的地进行缓存。基于由目标，选择器，noLocal传递标志和持久订阅名称（如果创建持久使用者）组成的键来缓存MessageConsumers。

#### 4.1.3。目的地管理

作为`ConnectionFactory`实例，目标是可以在JNDI中存储和检索的JMS管理的对象。在配置Spring应用程序上下文时，可以使用JNDI`JndiObjectFactoryBean`工厂类或`<jee:jndi-lookup>`对对象对JMS目标的引用执行依赖项注入。但是，如果应用程序中有大量目标，或者JMS提供程序具有独特的高级目标管理功能，则此策略通常很麻烦。这种高级目标管理的示例包括动态目标的创建或对目标的分层名称空间的支持。将`JmsTemplate`目标名称的解析委托给实现`DestinationResolver`接口的JMS目标对象 。`DynamicDestinationResolver`是由...使用的默认实现`JmsTemplate`并适应解析的动态目标。`JndiDestinationResolver`还提供A 充当JNDI中包含的目的地的服务定位器，并且可以选择回退到中包含的行为 `DynamicDestinationResolver`。

通常，仅在运行时才知道JMS应用程序中使用的目的地，因此，在部署应用程序时无法通过管理方式创建。这通常是因为在交互的系统组件之间存在共享的应用程序逻辑，这些组件根据已知的命名约定在运行时创建目标。即使创建动态目标不属于JMS规范的一部分，但大多数供应商都提供了此功能。动态目标是使用用户定义的名称创建的，该名称将它们与临时目标区分开来，并且通常未在JNDI中注册。提供商之间使用的用于创建动态目标的API有所不同，因为与目标关联的属性是特定于供应商的。然而，`TopicSession` `createTopic(String topicName)`或`QueueSession` `createQueue(String queueName)`使用默认目标属性创建新目标的方法。`DynamicDestinationResolver`然后，根据供应商的实现，还可以创建一个物理目标，而不是仅解决一个目标。

布尔值属性`pubSubDomain`用于使用`JmsTemplate`正在使用的JMS域的知识来配置。默认情况下，此属性的值为false，表示`Queues`将使用点对点域。（由所使用`JmsTemplate`）此属性通过`DestinationResolver`接口的实现确定动态目标解析的行为。

您还可以`JmsTemplate`通过属性将设置为具有默认目的地`defaultDestination`。默认目标是带有不引用特定目标的发送和接收操作。

#### 4.1.4。消息侦听器容器

在EJB世界中，JMS消息最常见的用途之一就是驱动消息驱动的bean（MDB）。Spring提供了一种解决方案，以不将用户绑定到EJB容器的方式创建消息驱动的POJO（MDP）。（有关Spring对MDP支持的详细介绍，请参见[异步接收：消息驱动的POJO](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-receiving-async)。）从Spring Framework 4.1开始，可以使用以下方法对端点方法进行注释`@JmsListener` -有关更多详细信息，请参见[注释驱动的侦听器端点](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-annotated)。

消息侦听器容器用于从JMS消息队列接收消息并驱动`MessageListener`注入到其中的消息。侦听器容器负责消息接收的所有线程，并分派到侦听器中进行处理。消息侦听器容器是MDP与消息传递提供程序之间的中介，并负责注册接收消息，参与事务，资源获取和释放，异常转换等。这使您可以编写与接收消息（并可能响应消息）相关的（可能很复杂的）业务逻辑，并将样板JMS基础结构问题委托给框架。

Spring附带了两个标准的JMS消息侦听器容器，每个容器都有其专门的功能集。

- [`SimpleMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-mdp-simple)
- [`DefaultMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-mdp-default)

##### 使用 `SimpleMessageListenerContainer`

此消息侦听器容器是两种标准样式中的简单容器。它在启动时创建固定数量的JMS会话和使用者，使用标准JMS`MessageConsumer.setMessageListener()`方法注册侦听器，然后将其留给JMS提供者以执行侦听器回调。此变体不允许动态适应运行时需求或参与外部管理的事务。在兼容性方面，它非常接近独立JMS规范的精神，但通常与Java EE的JMS限制不兼容。

|      | 尽管`SimpleMessageListenerContainer`不允许参与外部管理的事务，但它确实支持本机JMS事务。要启用此功能，可以将`sessionTransacted`标志切换为，`true`或者在XML名称空间中将`acknowledge`属性设置 为`transacted`。从您的侦听器抛出的异常会导致回滚，并重新传递消息。或者，考虑使用 `CLIENT_ACKNOWLEDGE`模式，该模式在出现异常的情况下也提供重新交付但不使用事务处理的`Session`实例，因此`Session`在事务协议中不包括任何其他 操作（例如发送响应消息）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 默认`AUTO_ACKNOWLEDGE`模式不能提供适当的可靠性保证。当侦听器执行失败时（由于提供者会在侦听器调用后自动确认每条消息，并且没有异常要传播到提供者），或者在侦听器容器关闭时（您可以通过设置`acceptMessagesWhileStopping`标志进行配置），消息可能会丢失。确保出于可靠性需求（例如，为了可靠的队列处理和持久的主题订阅）使用事务处理的会话。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 使用 `DefaultMessageListenerContainer`

大多数情况下使用此消息侦听器容器。与之相反 `SimpleMessageListenerContainer`，此容器变量允许动态适应运行时需求，并且能够参与外部管理的事务。配置为时，每个接收到的消息都会在XA事务中注册 `JtaTransactionManager`。结果，处理可以利用XA事务语义。该侦听器容器在JMS提供程序的低要求，高级功能（例如参与外部管理的事务）以及与Java EE环境的兼容性之间取得了良好的平衡。

您可以自定义容器的缓存级别。请注意，当未启用缓存时，将为每个消息接收创建一个新的连接和一个新的会话。将此内容与具有高负载的非持久订阅结合使用可能会导致消息丢失。在这种情况下，请确保使用适当的缓存级别。

当代理崩溃时，此容器还具有可恢复的功能。默认情况下，一个简单的`BackOff`实现每五秒钟重试一次。您可以`BackOff`为更细粒度的恢复选项指定自定义实现。有关`ExponentialBackOff`示例，请参见api-spring-framework / util / backoff / ExponentialBackOff.html [ ]。

|      | 与其兄弟（[`SimpleMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-mdp-simple)） 一样，它`DefaultMessageListenerContainer`支持本机JMS事务，并允许自定义确认模式。如果对您的方案可行，则强烈建议在外部管理的事务上使用此方法，也就是说，如果JVM死亡，您可以偶尔接收重复的消息。业务逻辑中的自定义重复消息检测步骤可以解决这种情况-例如，以业务实体存在检查或协议表检查的形式。任何这样的安排是显著比其他更有效的：用一个XA事务包裹整个处理（通过在配置 `DefaultMessageListenerContainer`有`JtaTransactionManager`）以覆盖所述JMS消息的接收，以及业务逻辑在您的消息侦听器的执行（包括数据库操作等）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 默认`AUTO_ACKNOWLEDGE`模式不能提供适当的可靠性保证。当侦听器执行失败时（由于提供者会在侦听器调用后自动确认每条消息，并且没有异常要传播到提供者），或者在侦听器容器关闭时（您可以通过设置`acceptMessagesWhileStopping`标志进行配置），消息可能会丢失。确保出于可靠性需求（例如，为了可靠的队列处理和持久的主题订阅）使用事务处理的会话。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 4.1.5。交易管理

Spring提供了一个`JmsTransactionManager`管理单个JMS事务的 `ConnectionFactory`。这使JMS应用程序可以利用Spring的托管事务功能，如 [“数据访问”一章的“事务管理”部分所述](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction)。在`JmsTransactionManager`执行本地资源交易，结合从指定的JMS连接/会话对`ConnectionFactory`给线程。 `JmsTemplate`自动检测此类交易资源并相应地对其进行操作。

在Java EE环境中，`ConnectionFactory`池连接和实例实例，因此这些资源可有效地跨事务重用。在独立环境中，使用Spring的`SingleConnectionFactory`结果在共享的JMS中`Connection`，每个事务都有自己独立的`Session`。或者，考虑使用提供程序专用的池适配器，例如ActiveMQ的`PooledConnectionFactory` 类。

您还可以`JmsTemplate`与`JtaTransactionManager`和具有XA功能的JMS一起使用 `ConnectionFactory`以执行分布式事务。请注意，这需要使用JTA事务管理器以及正确的XA配置的ConnectionFactory。（检查您的Java EE服务器或JMS提供程序的文档。）

跨托管和非托管的事务环境下重用代码可以使用JMS API来创建一个混乱的时候`Session`从一个`Connection`。这是因为JMS API只有一种工厂方法可以创建`Session`，并且它需要事务和确认模式的值。在托管环境中，设置这些值是环境的事务基础结构的责任，因此，供应商对JMS Connection的包装将忽略这些值。当您使用`JmsTemplate` 在非托管环境中，您可以通过使用属性来指定这些值`sessionTransacted`和`sessionAcknowledgeMode`。当使用 `PlatformTransactionManager`with时`JmsTemplate`，模板总是被赋予事务性JMS `Session`。

### 4.2. 发送信息

将`JmsTemplate`包含许多方便的方法来发送消息。发送方法通过使用`javax.jms.Destination`对象指定目的地，其他方法通过`String`在JNDI查找中使用指定目的地。`send`不使用目标参数的方法使用默认目标。

以下示例使用`MessageCreator`回调从提供的`Session`对象创建文本消息：

```java
import javax.jms.ConnectionFactory;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Queue;
import javax.jms.Session;

import org.springframework.jms.core.MessageCreator;
import org.springframework.jms.core.JmsTemplate;

public class JmsQueueSender {

    private JmsTemplate jmsTemplate;
    private Queue queue;

    public void setConnectionFactory(ConnectionFactory cf) {
        this.jmsTemplate = new JmsTemplate(cf);
    }

    public void setQueue(Queue queue) {
        this.queue = queue;
    }

    public void simpleSend() {
        this.jmsTemplate.send(this.queue, new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                return session.createTextMessage("hello queue world");
            }
        });
    }
}
```

在前面的示例中，`JmsTemplate`通过将引用传递给来构造 `ConnectionFactory`。作为替代方案，提供了一个零参数的构造函数，该构造函数 `connectionFactory`可用于以JavaBean样式（使用`BeanFactory`Java或纯Java代码）构造实例。另外，考虑从Spring的`JmsGatewaySupport`便捷基类派生，该基类为JMS配置提供了预构建的bean属性。

该`send(String destinationName, MessageCreator creator)`方法使您可以使用目标的字符串名称发送消息。如果这些名称已在JNDI中注册，则应将`destinationResolver`模板的属性设置为的实例 `JndiDestinationResolver`。

如果您创建`JmsTemplate`并指定了默认目标，则会向该目标 `send(MessageCreator c)`发送一条消息。

#### 4.2.1 用消息转换器

为了方便域模型对象的发送，的`JmsTemplate`发送方法有多种，它们将Java对象作为消息数据内容的参数。重载的方法`convertAndSend()`和中的`receiveAndConvert()`方法 `JmsTemplate`将转换过程委托给`MessageConverter` 接口的实例。该接口定义了一个简单的协定，可以在Java对象和JMS消息之间进行转换。默认实现（`SimpleMessageConverter`）支持在`String`和`TextMessage`，`byte[]`和`BytesMesssage`，和`java.util.Map` 和之间进行转换`MapMessage`。通过使用转换器，您和您的应用程序代码可以专注于通过JMS发送或接收的业务对象，而不必担心如何将其表示为JMS消息。

沙盒当前包含一个`MapMessageConverter`，使用反射在JavaBean和一个之间进行转换`MapMessage`。您可能自己实现的其他流行实现选择是使用现有XML编组程序包（例如JAXB或XStream）创建`TextMessage`表示对象的的转换器。

为了适应消息属性，标头和正文的设置，这些设置通常不能封装在转换器类中，因此，`MessagePostProcessor`接口可以在消息转换后但发送之前访问消息。以下示例显示将a`java.util.Map`转换为消息后如何修改消息头和属性 ：

```java
public void sendWithConversion() {
    Map map = new HashMap();
    map.put("Name", "Mark");
    map.put("Age", new Integer(47));
    jmsTemplate.convertAndSend("testQueue", map, new MessagePostProcessor() {
        public Message postProcessMessage(Message message) throws JMSException {
            message.setIntProperty("AccountID", 1234);
            message.setJMSCorrelationID("123-00001");
            return message;
        }
    });
}
```

这将导致以下形式的消息：

```java
apMessage={
    Header={
        ... standard headers ...
        CorrelationID={123-00001}
    }
    Properties={
        AccountID={Integer:1234}
    }
    Fields={
        Name={String:Mark}
        Age={Integer:47}
    }
}
```

#### 4.2.2. 使用`SessionCallback`和`ProducerCallback`

尽管发送操作涵盖了许多常见的使用场景，但是您有时可能希望在JMS`Session`或上执行多个操作`MessageProducer`。的 `SessionCallback`和`ProducerCallback`暴露的JMS`Session`和`Session`/ `MessageProducer`分别对，。这些`execute()`方法将`JmsTemplate`运行这些回调方法。

### 4.3. 接收讯息

这描述了如何在Spring中使用JMS接收消息。

#### 4.3.1. 同步接收

虽然JMS通常与异步处理相关联，但是您可以同步使用消息。重载的`receive(..)`方法提供了此功能。在同步接收期间，调用线程将阻塞，直到消息可用为止。这可能是危险的操作，因为调用线程可能会无限期地被阻塞。该`receiveTimeout`属性指定接收者在放弃等待消息之前应该等待多长时间。

#### 4.3.2. 异步接收：消息驱动的POJO

消息驱动POJO（MDP）以类似于EJB世界中的消息驱动Bean（MDB）的方式充当JMS消息的接收者。MDP上的一个限制（但请参见 [使用`MessageListenerAdapter`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-receiving-async-message-listener-adapter)）是它必须实现`javax.jms.MessageListener`接口。注意，如果您的POJO在多个线程上接收消息，那么确保您的实现是线程安全的，这一点很重要。

以下示例显示了MDP的简单实现：

```java
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

public class ExampleListener implements MessageListener {

    public void onMessage(Message message) {
        if (message instanceof TextMessage) {
            try {
                System.out.println(((TextMessage) message).getText());
            }
            catch (JMSException ex) {
                throw new RuntimeException(ex);
            }
        }
        else {
            throw new IllegalArgumentException("Message must be of type TextMessage");
        }
    }
}
```



 一旦实现了`MessageListener`，就可以创建一个消息侦听器容器了。

以下示例显示了如何定义和配置Spring附带的消息侦听器容器（在本例中为`DefaultMessageListenerContainer`）：

```xml
<!-- this is the Message Driven POJO (MDP) -->
<bean id="messageListener" class="jmsexample.ExampleListener"/>

<!-- and this is the message listener container -->
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
</bean>
```

有关每个实现支持的功能的完整说明，请参见各种消息侦听器容器的Spring javadoc（所有这些都实现了 [MessageListenerContainer](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/jms/listener/MessageListenerContainer.html)）。

#### 4.3.3. 使用`SessionAwareMessageListener`介面

该`SessionAwareMessageListener`接口是特定于Spring的接口，该接口提供与JMS`MessageListener`接口相似的协定，但还允许消息处理方法访问`Session`从其`Message`接收JMS的JMS 。以下清单显示了`SessionAwareMessageListener`接口的定义：

```java
package org.springframework.jms.listener;

public interface SessionAwareMessageListener {

    void onMessage(Message message, Session session) throws JMSException;
}
```

`MessageListener`如果希望MDP能够响应任何接收到的消息（通过使用 方法中`Session`提供的信息），则可以选择让MDP实现此接口（优先于标准JMS接口`onMessage(Message, Session)`）。Spring附带的所有消息侦听器容器实现都支持实现`MessageListener`或 `SessionAwareMessageListener`接口的MDP 。`SessionAwareMessageListener`需要注意的是，实现该类的类 随后将它们通过接口绑定到Spring。是否使用它的选择完全取决于您是应用程序开发人员还是架构师。

请注意， 接口的`onMessage(..)`方法`SessionAwareMessageListener`throw `JMSException`。与标准JMS`MessageListener` 接口相反，使用该`SessionAwareMessageListener`接口时，客户端代码负责处理所有引发的异常。

#### 4.3.4. 使用`MessageListenerAdapter`

本`MessageListenerAdapter`类是Spring的异步支持消息的最后一个组件。简而言之，它使您几乎可以将任何类公开为MDP（尽管存在一些约束）。

考虑以下接口定义：

```java
public interface MessageDelegate {

    void handleMessage(String message);

    void handleMessage(Map message);

    void handleMessage(byte[] message);

    void handleMessage(Serializable message);
}
```

请注意，尽管接口既不扩展接口`MessageListener`也不 扩展`SessionAwareMessageListener`接口，但仍可以通过使用`MessageListenerAdapter`该类将其用作MDP 。还请注意，如何根据各种消息处理方法`Message`可以接收和处理的各种类型的内容对其进行强类型化。

现在考虑`MessageDelegate`接口的以下实现：

```java
public class DefaultMessageDelegate implements MessageDelegate {
    // implementation elided for clarity...
}
```

特别要注意，`MessageDelegate`接口（ `DefaultMessageDelegate`该类）的先前实现是如何完全没有JMS依赖性的。这确实是一个POJO，我们可以通过以下配置将其变成MDP：

```xml
<!-- this is the Message Driven POJO (MDP) -->
<bean id="messageListener" class="org.springframework.jms.listener.adapter.MessageListenerAdapter">
    <constructor-arg>
        <bean class="jmsexample.DefaultMessageDelegate"/>
    </constructor-arg>
</bean>

<!-- and this is the message listener container... -->
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
</bean>
```

下一个示例显示另一个MDP，它只能处理接收到的JMS `TextMessage`消息。请注意，实际上是如何调用消息处理方法的`receive`（消息处理方法 的名称`MessageListenerAdapter` 默认为`handleMessage`），但是它是可配置的（如本节后面所述）。还要注意如何`receive(..)`强类型化该方法以仅接收和响应JMS `TextMessage`消息。以下清单显示了`TextMessageDelegate`接口的定义：

```java
public interface TextMessageDelegate {

    void receive(TextMessage message);
}
```

以下清单显示了实现该`TextMessageDelegate`接口的类

```java
public class DefaultTextMessageDelegate implements TextMessageDelegate {
    // implementation elided for clarity...
}
```

话务员的配置`MessageListenerAdapter`如下：

```xml
<bean id="messageListener" class="org.springframework.jms.listener.adapter.MessageListenerAdapter">
    <constructor-arg>
        <bean class="jmsexample.DefaultTextMessageDelegate"/>
    </constructor-arg>
    <property name="defaultListenerMethod" value="receive"/>
    <!-- we don't want automatic message context extraction -->
    <property name="messageConverter">
        <null/>
    </property>
</bean>
```

需要注意的是，如果`messageListener`接收到一个JMS`Message`以外的类型的`TextMessage`，一个`IllegalStateException`被抛出（随后吞咽）。`MessageListenerAdapter`该类的另一个功能是，`Message`如果处理程序方法返回非空值，则能够自动发送回响应。考虑以下接口和类：

```java
public interface ResponsiveTextMessageDelegate {

    // notice the return type...
    String receive(TextMessage message);
}
```



```java
public class DefaultResponsiveTextMessageDelegate implements ResponsiveTextMessageDelegate {
// implementation elided for clarity...
}
```

如果将`DefaultResponsiveTextMessageDelegate`s与a结合使用 `MessageListenerAdapter`，则从`'receive(..)'`方法执行返回的任何非null值（在默认配置下）都将转换为 `TextMessage`。`TextMessage`然后将结果发送到原始`Destination`JMS`Reply-To`属性中定义的（如果存在）`Message`或（如果已配置）默认`Destination`设置的JMS属性`MessageListenerAdapter`。如果未`Destination`找到，`InvalidDestinationException`则会引发an （请注意，该异常不会被吞没，并且会在调用堆栈中传播）。

#### 4.3.5. 处理事务中的消息

在事务中调用消息侦听器仅需要重新配置侦听器容器。

您可以通过`sessionTransacted`侦听器容器定义上的标志激活本地资源事务。然后，每个消息侦听器调用都在活动的JMS事务内运行，并且在侦听器执行失败的情况下回滚消息接收。（通过`SessionAwareMessageListener`）发送响应消息是同一本地事务的一部分，但是任何其他资源操作（例如数据库访问）都是独立运行的。这通常需要在侦听器实现中进行重复消息检测，以解决数据库处理已提交但消息处理未能提交的情况。

考虑以下bean定义：

```xml
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
    <property name="sessionTransacted" value="true"/>
</bean>
```

要参与外部管理的事务，您需要配置一个事务管理器并使用支持外部管理的事务的侦听器容器（通常为`DefaultMessageListenerContainer`）。

要为XA事务参与配置消息侦听器容器，您需要配置一个`JtaTransactionManager`（默认情况下，它委托给Java EE服务器的事务子系统）。请注意，底层的JMS`ConnectionFactory`需要具有XA功能，并已向您的JTA事务协调器正确注册。（检查Java EE服务器的JNDI资源配置。）这使消息接收以及（例如）数据库访问成为同一事务的一部分（具有统一的提交语义，但以XA事务日志开销为代价）。

以下bean定义创建一个事务管理器：

```xml
<bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
```

然后，我们需要将其添加到我们之前的容器配置中。容器负责其余的工作。以下示例显示了如何执行此操作：

```xml
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
    <property name="transactionManager" ref="transactionManager"/> 
</bean>
```

### 4.4. 支持JCA消息端点

从2.5版开始，Spring还提供了对基于JCA的 `MessageListener`容器的支持。在`JmsMessageEndpointManager`尝试自动确定`ActivationSpec`从提供的类名 `ResourceAdapter`类名。因此，通常可以提供Spring的generic `JmsActivationSpecConfig`，如以下示例所示：

```java
<bean class="org.springframework.jms.listener.endpoint.JmsMessageEndpointManager">
    <property name="resourceAdapter" ref="resourceAdapter"/>
    <property name="activationSpecConfig">
        <bean class="org.springframework.jms.listener.endpoint.JmsActivationSpecConfig">
            <property name="destinationName" value="myQueue"/>
        </bean>
    </property>
    <property name="messageListener" ref="myMessageListener"/>
</bean>
```

或者，您可以`JmsMessageEndpointManager`使用给定的 `ActivationSpec`对象设置一个。该`ActivationSpec`对象也可能来自JNDI查找（使用`<jee:jndi-lookup>`）。以下示例显示了如何执行此操作：

```xml
<bean class="org.springframework.jms.listener.endpoint.JmsMessageEndpointManager">
    <property name="resourceAdapter" ref="resourceAdapter"/>
    <property name="activationSpec">
        <bean class="org.apache.activemq.ra.ActiveMQActivationSpec">
            <property name="destination" value="myQueue"/>
            <property name="destinationType" value="javax.jms.Queue"/>
        </bean>
    </property>
    <property name="messageListener" ref="myMessageListener"/>
</bean>
```

使用Spring的`ResourceAdapterFactoryBean`，您可以在`ResourceAdapter` 本地配置目标，如以下示例所示：

```xml
<bean id="resourceAdapter" class="org.springframework.jca.support.ResourceAdapterFactoryBean">
    <property name="resourceAdapter">
        <bean class="org.apache.activemq.ra.ActiveMQResourceAdapter">
            <property name="serverUrl" value="tcp://localhost:61616"/>
        </bean>
    </property>
    <property name="workManager">
        <bean class="org.springframework.jca.work.SimpleTaskWorkManager"/>
    </property>
</bean>
```

指定的对象`WorkManager`也可以指向特定于环境的线程池-通常通过`SimpleTaskWorkManager`实例的`asyncTaskExecutor`属性。`ResourceAdapter`如果碰巧使用多个适配器，请考虑为所有实例定义一个共享线程池。

在某些环境（例如WebLogic 9或更高版本）中，您可以改为`ResourceAdapter`使用JNDI获取整个对象`<jee:jndi-lookup>`。然后，基于Spring的消息侦听器可以与服务器托管的服务器交互，该服务器`ResourceAdapter`也使用服务器的内置服务器`WorkManager`。

有关，和 的更多信息[`JmsMessageEndpointManager`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/jms/listener/endpoint/JmsMessageEndpointManager.html)， 请参见javadoc 。[`JmsActivationSpecConfig`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/jms/listener/endpoint/JmsActivationSpecConfig.html)[`ResourceAdapterFactoryBean`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/jca/support/ResourceAdapterFactoryBean.html)

Spring还提供了一个不受JMS约束的通用JCA消息端点管理器 `org.springframework.jca.endpoint.GenericMessageEndpointManager`。该组件允许使用任何消息侦听器类型（例如JMS `MessageListener`）和任何提供程序特定的`ActivationSpec`对象。请参阅JCA提供程序的文档以了解连接器的实际功能，并请参阅 [`GenericMessageEndpointManager`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/jca/endpoint/GenericMessageEndpointManager.html) javadoc以获取特定于Spring的配置详细信息。

基于JCA的消息端点管理与EJB 2.1消息驱动Bean非常相似。它使用相同的基础资源提供者合同。与EJB 2.1 MDB一样，您也可以在Spring上下文中使用JCA提供程序支持的任何消息侦听器接口。尽管如此，Spring为JMS提供了显式的“便利”支持，因为JMS是JCA端点管理协定中最常用的端点API。

### 4.5 注释驱动的侦听器端点

异步接收消息的最简单方法是使用带注释的侦听器端点基础结构。简而言之，它使您可以将托管Bean的方法公开为JMS侦听器端点。以下示例显示了如何使用它：

```java
@Component
public class MyService {

    @JmsListener(destination = "myDestination")
    public void processOrder(String data) { ... }
}
```

前面示例的思想是，只要消息可用，就会 相应地调用`javax.jms.Destination` `myDestination`该`processOrder`方法（在这种情况下，使用JMS消息的内容，类似于[`MessageListenerAdapter`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-receiving-async-message-listener-adapter) 提供的内容）。

带注释的终结点基础结构通过使用，为每种带注释的方法在幕后创建一个消息侦听器容器`JmsListenerContainerFactory`。这样的容器不会针对应用程序上下文进行注册，但是可以通过使用`JmsListenerEndpointRegistry`Bean方便地定位用于管理目的。

`@JmsListener`是Java 8上的可重复注释，因此您可以通过`@JmsListener` 向其添加其他声明来将多个JMS目标与同一方法相关联。

#### 4.5.1. Annotation-driven Listener Endpoints

要启用对`@JmsListener`注释的支持，可以将其添加`@EnableJms`到一个`@Configuration`类中，如以下示例所示：

```java
@Configuration
@EnableJms
public class AppConfig {

    @Bean
    public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
        DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory());
        factory.setDestinationResolver(destinationResolver());
        factory.setSessionTransacted(true);
        factory.setConcurrency("3-10");
        return factory;
    }
}
```

默认情况下，基础结构会寻找一个名为Bean`jmsListenerContainerFactory` 的工厂供其用来创建消息侦听器容器的源。在这种情况下（并忽略JMS基础结构设置），您可以`processOrder` 使用三个线程的核心轮询大小和十个线程的最大池大小来调用该方法。

您可以自定义用于每个注释的侦听器容器工厂，也可以通过实现`JmsListenerConfigurer`接口来配置显式默认值。仅当至少一个端点在没有特定容器工厂的情况下注册时，才需要使用默认值。有关[`JmsListenerConfigurer`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/jms/annotation/JmsListenerConfigurer.html) 详细信息和示例，请参见实现的类的javadoc 。

如果您更喜欢[XML配置](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-namespace)，那么可以使用`<jms:annotation-driven>` 元素，如以下示例所示：

```xml
<jms:annotation-driven/>

<bean id="jmsListenerContainerFactory"
        class="org.springframework.jms.config.DefaultJmsListenerContainerFactory">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destinationResolver" ref="destinationResolver"/>
    <property name="sessionTransacted" value="true"/>
    <property name="concurrency" value="3-10"/>
</bean>
```

#### 4.5.2. 程序化端点注册

`JmsListenerEndpoint`提供JMS端点的模型，并负责为该模型配置容器。除了`JmsListener`注释所检测的端点之外，基础结构还允许您以编程方式配置端点。以下示例显示了如何执行此操作：

```java
@Configuration
@EnableJms
public class AppConfig implements JmsListenerConfigurer {

    @Override
    public void configureJmsListeners(JmsListenerEndpointRegistrar registrar) {
        SimpleJmsListenerEndpoint endpoint = new SimpleJmsListenerEndpoint();
        endpoint.setId("myJmsEndpoint");
        endpoint.setDestination("anotherQueue");
        endpoint.setMessageListener(message -> {
            // processing
        });
        registrar.registerEndpoint(endpoint);
    }
}
```

在前面的示例中，我们使用`SimpleJmsListenerEndpoint`了提供实际 `MessageListener`调用的。但是，您也可以构建自己的端点变体来描述自定义调用机制。

请注意，您可以`@JmsListener`完全跳过使用，而可以通过编程方式仅注册端点`JmsListenerConfigurer`。

 4.5.3. 带注释的端点方法签名

```java
@Component
public class MyService {

    @JmsListener(destination = "myDestination")
    public void processOrder(Order order, @Header("order_type") String orderType) {
        ...
    }
}
```

您可以在JMS侦听器端点中注入的主要元素如下：

- 原始`javax.jms.Message`或其任何子类（前提是它与传入的消息类型匹配）。
- 该`javax.jms.Session`可选访问本地JMS API（例如，用于发送一个自定义的回复）。
- 的`org.springframework.messaging.Message`，它表示传入的JMS消息。请注意，此消息同时包含自定义标头和标准标头（由定义`JmsHeaders`）。
- `@Header`-带注释的方法参数，以提取特定的标头值，包括标准的JMS标头。
- 一个带`@Headers`注释的参数，也必须可分配给该参数，以`java.util.Map`访问所有标头。
- 不是支持的类型（`Message`或 `Session`）之一的非注释元素被视为有效负载。您可以通过使用注释参数来使其明确`@Payload`。您还可以通过添加额外的来启用验证 `@Valid`。

注入Spring的`Message`抽象功能特别有用，它可以受益于存储在特定于传输的消息中的所有信息，而无需依赖于特定于传输的API。以下示例显示了如何执行此操作：

```java
@JmsListener(destination = "myDestination")
public void processOrder(Message<Order> order) { ... }
```

方法参数的处理由提供`DefaultMessageHandlerMethodFactory`，您可以进一步自定义以支持其他方法参数。您也可以在那里自定义转换和验证支持。

例如，如果要`Order`在处理之前确保我们的有效，则可以使用注释有效负载`@Valid`并配置必要的验证器，如以下示例所示：

```java
@Configuration
@EnableJms
public class AppConfig implements JmsListenerConfigurer {

    @Override
    public void configureJmsListeners(JmsListenerEndpointRegistrar registrar) {
        registrar.setMessageHandlerMethodFactory(myJmsHandlerMethodFactory());
    }

    @Bean
    public DefaultMessageHandlerMethodFactory myHandlerMethodFactory() {
        DefaultMessageHandlerMethodFactory factory = new DefaultMessageHandlerMethodFactory();
        factory.setValidator(myValidator());
        return factory;
    }
}
```

#### 4.5.4. 反应管理

现有的支持[`MessageListenerAdapter`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-receiving-async-message-listener-adapter) 已经使您的方法具有非`void`返回类型。在这种情况下，调用的结果将封装在中`javax.jms.Message`，发送`JMSReplyTo`到原始消息的标头中指定的目标位置或侦听器上配置的默认目标位置。现在，您可以使用`@SendTo`消息传递抽象的注释来设置默认目标。

假设我们的`processOrder`方法现在应该返回`OrderStatus`，我们可以将其编写为自动发送响应，如以下示例所示：

```java
@JmsListener(destination = "myDestination")
@SendTo("status")
public OrderStatus processOrder(Order order) {
    // order processing
    return status;
}
```

如果需要以与传输无关的方式设置其他标头，则可以`Message`使用类似于以下方法的方法返回a ：

```java
@JmsListener(destination = "myDestination")
@SendTo("status")
public Message<OrderStatus> processOrder(Order order) {
    // order processing
    return MessageBuilder
            .withPayload(status)
            .setHeader("code", 1234)
            .build();
}
```
如果需要在运行时计算响应目标，则可以将响应封装在一个JmsResponse实例中，该实例还提供要在运行时使用的目标。我们可以如下重写前一个示例：
```java
@JmsListener(destination = "myDestination")
public JmsResponse<Message<OrderStatus>> processOrder(Order order) {
    // order processing
    Message<OrderStatus> response = MessageBuilder
            .withPayload(status)
            .setHeader("code", 1234)
            .build();
    return JmsResponse.forQueue(response, "status");
}
```
最后，如果需要为响应指定一些QoS值，例如优先级或生存时间，则可以进行相应的配置JmsListenerContainerFactory，如以下示例所示：
```java
@Configuration
@EnableJms
public class AppConfig {

    @Bean
    public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
        DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory());
        QosSettings replyQosSettings = new QosSettings();
        replyQosSettings.setPriority(2);
        replyQosSettings.setTimeToLive(10000);
        factory.setReplyQosSettings(replyQosSettings);
        return factory;
    }
}
```

### 4.6. JMS命名空间支持

Spring提供了一个XML名称空间来简化JMS配置。要使用JMS名称空间元素，您需要引用JMS模式，如以下示例所示：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jms="http://www.springframework.org/schema/jms" 
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/jms https://www.springframework.org/schema/jms/spring-jms.xsd">

    <!-- bean definitions here -->

</beans>
```
这个命名空间由三个顶级元素：<annotation-driven/>，<listener-container/> 和<jca-listener-container/>。<annotation-driven/>启用注释驱动的侦听器终结点的使用。<listener-container/>并<jca-listener-container/> 定义共享的侦听器容器配置，并且可以包含<listener/>子元素。以下示例显示了两个侦听器的基本配置：
```xml
<jms:listener-container>

    <jms:listener destination="queue.orders" ref="orderService" method="placeOrder"/>

    <jms:listener destination="queue.confirmations" ref="confirmationLogger" method="log"/>

</jms:listener-container>
```
前面的示例等效于创建两个不同的侦听器容器bean定义和两个不同的MessageListenerAdapterbean定义，如Using中MessageListenerAdapter所示。除了前面示例中显示的属性之外，该listener元素还可以包含几个可选属性。下表描述了所有可用属性：

表3. JMS <listener>元素的属性
属性	描述
id

托管侦听器容器的Bean名称。如果未指定，将自动生成Bean名称。

destination （需要）

此侦听器的目标名称，通过DestinationResolver 策略解析。

ref （需要）

处理程序对象的bean名称。

method

要调用的处理程序方法的名称。如果ref属性指向MessageListener 或Spring SessionAwareMessageListener，则可以忽略此属性。

response-destination

向其发送响应消息的默认响应目标的名称。在请求消息不包含JMSReplyTo字段的情况下适用。此目标的类型由listener-container的 response-destination-type属性确定。请注意，这仅适用于具有返回值的侦听器方法，为此，每个结果对象都将转换为响应消息。

subscription

持久订阅的名称（如果有）。

selector

此侦听器的可选消息选择器。

concurrency

要启动此侦听器的并发会话或使用者的数量。此值可以是表示最大数的简单数字（例如5），也可以是表示下限和上限的范围（例如3-5）。请注意，指定的最小值仅是一个提示，在运行时可能会被忽略。默认值为容器提供的值。

该<listener-container/>元素还接受几个可选属性。这允许自定义各种策略（例如taskExecutor和 destinationResolver）以及基本的JMS设置和资源引用。通过使用这些属性，您可以定义高度自定义的侦听器容器，同时仍然受益于命名空间的便利性。

您可以JmsListenerContainerFactory通过指定id通过factory-id属性公开的Bean来自动将这些设置显示为，如以下示例所示：
```xml
<jms:listener-container connection-factory="myConnectionFactory"
        task-executor="myTaskExecutor"
        destination-resolver="myDestinationResolver"
        transaction-manager="myTransactionManager"
        concurrency="10">

    <jms:listener destination="queue.orders" ref="orderService" method="placeOrder"/>

    <jms:listener destination="queue.confirmations" ref="confirmationLogger" method="log"/>

</jms:listener-container>
```
下表描述了所有可用属性。有关AbstractMessageListenerContainer 各个属性的更多详细信息，请参见和其具体子类的类级javadoc 。Javadoc还讨论了事务选择和消息重新交付方案。

表4. JMS <listener-container>元素的属性
属性	描述
container-type

此侦听器容器的类型。可用选项为default，simple， default102或者simple102（默认选项default）。

container-class

自定义侦听器容器实现类，作为完全限定的类名称。根据属性，默认值为Spring的标准DefaultMessageListenerContainer或 。SimpleMessageListenerContainercontainer-type

factory-id

将此元素定义的设置公开为JmsListenerContainerFactory 带有指定的，id以便它们可以与其他端点重用。

connection-factory

对JMS ConnectionFactoryBean的引用（默认Bean名称为 connectionFactory）。

task-executor

TaskExecutor对JMS侦听器调用程序的Spring的引用。

destination-resolver

DestinationResolver对解决JMSDestination实例的策略的引用。

message-converter

MessageConverter对将JMS消息转换为侦听器方法参数的策略的参考。默认值为SimpleMessageConverter。

error-handler

引用一种ErrorHandler策略，用于处理在执行期间可能发生的任何未捕获的异常MessageListener。

destination-type

这个监听器的JMS目的地类型：queue，topic，durableTopic，sharedTopic，或sharedDurableTopic。这潜在地使pubSubDomain，subscriptionDurable 与subscriptionShared所述容器的性质。默认值为queue（禁用这三个属性）。

response-destination-type

响应的JMS目标类型：queue或topic。默认值为该destination-type属性的值 。

client-id

此侦听器容器的JMS客户端ID。使用持久订阅时必须指定它。

cache

对JMS资源的高速缓存级别：none，connection，session，consumer，或 auto。默认情况下（auto），consumer除非指定了外部事务管理器，否则缓存级别是有效的-在这种情况下，有效的默认值将是none（假设Java EE风格的事务管理，其中给定的ConnectionFactory是可识别XA的池）。

acknowledge

本机JMS确认模式：auto，client，dups-ok，或transacted。值transacted激活本地交易Session。或者，您可以指定transaction-manager属性，如表中稍后所述。默认值为auto。

transaction-manager

对外部PlatformTransactionManager（通常是基于XA的事务协调器，例如Spring的JtaTransactionManager）的引用。如果未指定，则使用本机确认（请参阅acknowledge属性）。

concurrency

每个侦听器启动的并发会话或使用者的数量。它可以是表示最大数的简单数字（例如5），也可以是表示下限和上限的范围（例如3-5）。请注意，指定的最小值只是一个提示，在运行时可能会被忽略。默认值为1。如果1主题侦听器或队列顺序很重要，则应将并发限制在一定范围内。考虑将其提高到一般队列。

prefetch

加载到单个会话中的最大消息数。请注意，增加此数字可能会导致并发消费者饥饿。

receive-timeout

用于接听电话的超时时间（以毫秒为单位）。默认值为1000（一秒钟）。-1表示没有超时。

back-off

指定BackOff用于计算恢复尝试间隔的实例。如果BackOffExecution实现返回BackOffExecution#STOP，则侦听器容器不会进一步尝试恢复。recovery-interval 设置此属性后，将忽略该值。缺省值为a FixedBackOff，间隔为5000毫秒（即5秒）。

recovery-interval

指定两次恢复尝试之间的时间间隔（以毫秒为单位）。它提供了一种方便的方法来创建FixedBackOff具有指定间隔的。有关更多恢复选项，请考虑指定一个BackOff实例。缺省值为5000毫秒（即5秒）。

phase

该容器应在其中启动和停止的生命周期阶段。值越低，此容器启动越早，而其停止越晚。默认值为 Integer.MAX_VALUE，表示容器尽可能晚地启动，并尽快停止。

使用jms模式支持配置基于JCA的侦听器容器非常相似，如以下示例所示：
```xml
<jms:jca-listener-container resource-adapter="myResourceAdapter"
        destination-resolver="myDestinationResolver"
        transaction-manager="myTransactionManager"
        concurrency="10">

    <jms:listener destination="queue.orders" ref="myMessageListener"/>

</jms:jca-listener-container>
```
下表描述了JCA变体的可用配置选项：

表5. JMS <jca-listener-container />元素的属性
属性	描述
factory-id

将此元素定义的设置公开为JmsListenerContainerFactory 带有指定的，id以便它们可以与其他端点重用。

resource-adapter

对JCA ResourceAdapterbean的引用（默认bean名称为 resourceAdapter）。

activation-spec-factory

对的引用JmsActivationSpecFactory。默认设置是自动检测JMS提供程序及其ActivationSpec类（请参阅参考资料DefaultJmsActivationSpecFactory）。

destination-resolver

DestinationResolver解决JMS的策略的参考Destinations。

message-converter

MessageConverter对将JMS消息转换为侦听器方法参数的策略的参考。默认值为SimpleMessageConverter。

destination-type

这个监听器的JMS目的地类型：queue，topic，durableTopic，sharedTopic。或sharedDurableTopic。这潜在地使pubSubDomain，subscriptionDurable和subscriptionShared容器的性质。默认值为queue（禁用这三个属性）。

response-destination-type

响应的JMS目标类型：queue或topic。默认值为该destination-type属性的值 。

client-id

此侦听器容器的JMS客户端ID。使用持久订阅时需要指定它。

acknowledge

本机JMS确认模式：auto，client，dups-ok，或transacted。值transacted激活本地交易Session。或者，您可以指定transaction-manager稍后描述的属性。默认值为auto。

transaction-manager

对SpringJtaTransactionManager或a的 引用，javax.transaction.TransactionManager用于为每个传入消息启动XA事务。如果未指定，则使用本机确认（请参阅 acknowledge属性）。

concurrency

每个侦听器启动的并发会话或使用者的数量。它可以是表示最大值的简单数字（例如5），也可以是表示上下限的范围（例如3-5）。请注意，指定的最小值只是一个提示，通常在运行时使用JCA侦听器容器时将被忽略。预设值为1。

prefetch

加载到单个会话中的最大消息数。请注意，增加此数字可能会导致并发消费者饥饿。
## 5. JMX
Spring中的JMX（Java管理扩展）支持提供了一些功能，使您可以轻松，透明地将Spring应用程序集成到JMX基础结构中。
具体来说，Spring的JMX支持提供了四个核心功能：

将任何Spring bean自动注册为JMX MBean。

用于控制bean的管理界面的灵活机制。

通过远程JSR-160连接器以声明方式公开MBean。

本地和远程MBean资源的简单代理。

这些功能旨在在不将应用程序组件耦合到Spring或JMX接口和类的情况下起作用。实际上，在大多数情况下，您的应用程序类无需了解Spring或JMX即可利用Spring JMX功能。
### 5.1. 将您的Bean导出到JMX
Spring的JMX框架的核心类是MBeanExporter。此类负责获取您的Spring bean并向JMX注册它们MBeanServer。例如，考虑以下类：
```java
package org.springframework.jmx;

public class JmxTestBean implements IJmxTestBean {

    private String name;
    private int age;
    private boolean isSuperman;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public int add(int x, int y) {
        return x + y;
    }

    public void dontExposeMe() {
        throw new RuntimeException();
    }
}
```
要将此Bean的属性和方法公开为MBean的属性和操作，可以MBeanExporter在配置文件中配置该类的实例并传入Bean，如以下示例所示：
```xml
<beans>
    <!-- this bean must not be lazily initialized if the exporting is to happen -->
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter" lazy-init="false">
        <property name="beans">
            <map>
                <entry key="bean:name=testBean1" value-ref="testBean"/>
            </map>
        </property>
    </bean>
    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>
</beans>
```
前面的配置片段中相关的bean定义是exporter bean。该beans属性MBeanExporter确切地告诉您必须将哪些bean导出到JMX MBeanServer。在默认配置中，中的每个条目的键beans Map都用作ObjectName相应条目值所引用的Bean。您可以更改此行为，如控制ObjectNameBean的实例中所述。

使用此配置，该testBeanBean在下方显示为MBean ObjectName bean:name=testBean1。默认情况下，publicbean的所有属性都公开为属性，所有public方法（从Object类继承的方法除外 ）都公开为操作。

MBeanExporter是一个Lifecyclebean（请参阅启动和关闭回调）。默认情况下，MBean在应用程序生命周期中尽可能晚地导出。您可以phase通过设置autoStartup标志来配置导出发生的位置或禁用自动注册。
#### 5.1.1. 创建一个MBeanServer
上一节中显示的配置假定该应用程序正在一个（并且只有一个）MBeanServer 已经运行的环境中运行。在这种情况下，Spring尝试找到正在运行的MBeanServer服务器，并将您的bean注册到该服务器（如果有）。当您的应用程序在具有自己的容器（例如Tomcat或IBM WebSphere）中运行时，此行为很有用MBeanServer。

但是，这种方法在独立环境中或在未提供的容器中运行时没有用MBeanServer。为了解决这个问题，您可以MBeanServer通过将org.springframework.jmx.support.MBeanServerFactoryBean类的实例添加到配置中来声明性地创建 实例 。您还可以MBeanServer通过将MBeanExporter实例server属性的MBeanServer值设置为所返回的值 来确保使用特定对象 MBeanServerFactoryBean，如以下示例所示：
```xml
<beans>

    <bean id="mbeanServer" class="org.springframework.jmx.support.MBeanServerFactoryBean"/>

    <!--
    this bean needs to be eagerly pre-instantiated in order for the exporting to occur;
    this means that it must not be marked as lazily initialized
    -->
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
            <map>
                <entry key="bean:name=testBean1" value-ref="testBean"/>
            </map>
        </property>
        <property name="server" ref="mbeanServer"/>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

</beans>
```
在前面的示例中，的实例由MBeanServer创建，MBeanServerFactoryBean并MBeanExporter通过server属性提供给。提供您自己的 MBeanServer实例时，MBeanExporter不会尝试查找正在运行 MBeanServer的MBeanServer实例并使用提供的实例。为了使其正常工作，您必须在类路径上具有JMX实现。
#### 5.1.2. 重用现有的MBeanServer
如果未指定服务器，则MBeanExporter尝试自动检测正在运行 MBeanServer。在大多数仅使用一个MBeanServer实例的环境中，这是可行的。但是，当存在多个实例时，导出器可能选择了错误的服务器。在这种情况下，应使用MBeanServer agentId指示要使用的实例，如以下示例所示：
```xml
<beans>
    <bean id="mbeanServer" class="org.springframework.jmx.support.MBeanServerFactoryBean">
        <!-- indicate to first look for a server -->
        <property name="locateExistingServerIfPossible" value="true"/>
        <!-- search for the MBeanServer instance with the given agentId -->
        <property name="agentId" value="MBeanServer_instance_agentId>"/>
    </bean>
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="server" ref="mbeanServer"/>
        ...
    </bean>
</beans>
```
对于平台或现有的MBeanServer具有agentId通过查找方法检索到的动态（或未知）动态的 情况，应使用 factory-method，如以下示例所示：
```xml
<beans>
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="server">
            <!-- Custom MBeanServerLocator -->
            <bean class="platform.package.MBeanServerLocator" factory-method="locateMBeanServer"/>
        </property>
    </bean>

    <!-- other beans here -->

</beans>
```
#### 5.1.3. 延迟初始化的MBean
如果使用MBeanExporter还配置了延迟初始化的来配置Bean ，MBeanExporter则不会破坏该协定，并避免实例化该Bean。相反，它向注册了一个代理，MBeanServer并推迟从容器中获取Bean，直到对该代理进行第一次调用为止。
#### 5.1.4. 自动注册MBean
通过导出的MBeanExporter，已经是有效MBean的所有Bean都将按原样注册，MBeanServer而无需Spring的进一步干预。MBeanExporter通过将autodetect 属性设置为true，可以使MBean被自动检测，如以下示例所示：
```xml
<bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
    <property name="autodetect" value="true"/>
</bean>

<bean name="spring:mbean=true" class="org.springframework.jmx.export.TestDynamicMBean"/>
```
在前面的示例中，被调用的beanspring:mbean=true已经是有效的JMX MBean，并由Spring自动注册。默认情况下，自动检测到JMX注册的bean的bean名称用作ObjectName。您可以覆盖此行为，如控制ObjectNameBean的实例中所述。
#### 5.1.5 控制注册行为
考虑这样一个春天的情景MBeanExporter尝试注册一个MBean 与MBeanServer使用ObjectName bean:name=testBean1。如果MBean 实例已经在该实例下注册ObjectName，则默认行为是失败（并抛出InstanceAlreadyExistsException）。

您可以精确控制将MBean进行注册时发生的情况MBeanServer。Spring的JMX支持允许三种不同的注册行为来控制注册行为，当注册过程发现anMBean已在相同的之下注册时ObjectName。下表总结了这些注册行为：

表6.注册行为
注册行为	说明
FAIL_ON_EXISTING

这是默认的注册行为。如果一个MBean实例已经在相同的注册ObjectName，将MBean正在注册不注册，而InstanceAlreadyExistsException被抛出。现有 MBean不受影响。

IGNORE_EXISTING

如果MBean已经在同一实例下注册了实例ObjectName，那么 MBean正在注册的实例不会被注册。现有MBean不受影响，并且不会Exception抛出任何异常。这在多个应用程序要共享一个共享MBean中的一个共享的设置中很有用MBeanServer。

REPLACE_EXISTING

如果某个MBean实例已经在同一实例下注册ObjectName，那么MBean先前已注册的现有实例将被取消注册，并且新 实例将MBean在其位置注册（新MBean实例将有效替换先前的实例）。

上表中的值被定义为RegistrationPolicy该类上的枚举。如果要更改默认注册行为，则需要将定义registrationPolicy上的属性值设置 MBeanExporter为这些值之一。

下面的示例显示如何从默认注册行为更改为REPLACE_EXISTING行为：
```xml
<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
            <map>
                <entry key="bean:name=testBean1" value-ref="testBean"/>
            </map>
        </property>
        <property name="registrationPolicy" value="REPLACE_EXISTING"/>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

</beans>
```
### 5.2. 控制Bean的管理接口
在上一节的示例中，您几乎没有控制bean的管理接口。public 每个导出的bean的所有属性和方法分别作为JMX属性和操作公开。为了对出口的Bean的哪些属性和方法实际上作为JMX属性和操作公开而进行更细粒度的控制，Spring JMX提供了一种全面且可扩展的机制来控制Bean的管理接口。
#### 5.2.1. 使用MBeanInfoAssembler介面
在后台，MBeanExporter代表org.springframework.jmx.export.assembler.MBeanInfoAssembler接口的实现，该 接口负责定义每个公开的bean的管理接口。默认实现 org.springframework.jmx.export.assembler.SimpleReflectiveMBeanInfoAssembler定义了一个管理接口，该接口公开了所有公共属性和方法（如前面各节中的示例所示）。Spring提供了MBeanInfoAssembler接口的两个附加实现，使您可以使用源级元数据或任何任意接口来控制生成的管理接口
#### 5.2.2. 使用源级元数据：Java注释
通过使用MetadataMBeanInfoAssembler，您可以使用源级元数据为bean定义管理界面。org.springframework.jmx.export.metadata.JmxAttributeSource接口封装了元数据的读取。Spring JMX提供了使用Java注释的默认实现，即 org.springframework.jmx.export.annotation.AnnotationJmxAttributeSource。您必须MetadataMBeanInfoAssembler使用JmxAttributeSource接口的实现实例配置，才能使其正常运行（没有默认值）。

要标记要导出到JMX的bean，应使用注释对bean类进行 ManagedResource注释。您必须使用ManagedOperation注释将要公开的每个方法标记为一个操作，并使用注释将要公开的每个属性标记为ManagedAttribute。标记属性时，可以省略getter或setter的注释，以分别创建只写或只读属性。

带ManagedResource注释的Bean必须是公共的，公开操作或属性的方法也必须是公共的。
以下示例显示了JmxTestBean我们在创建MBeanServer中使用的类的带注释版本：
```java
package org.springframework.jmx;

import org.springframework.jmx.export.annotation.ManagedResource;
import org.springframework.jmx.export.annotation.ManagedOperation;
import org.springframework.jmx.export.annotation.ManagedAttribute;

@ManagedResource(
        objectName="bean:name=testBean4",
        description="My Managed Bean",
        log=true,
        logFile="jmx.log",
        currencyTimeLimit=15,
        persistPolicy="OnUpdate",
        persistPeriod=200,
        persistLocation="foo",
        persistName="bar")
public class AnnotationTestBean implements IJmxTestBean {

    private String name;
    private int age;

    @ManagedAttribute(description="The Age Attribute", currencyTimeLimit=15)
    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @ManagedAttribute(description="The Name Attribute",
            currencyTimeLimit=20,
            defaultValue="bar",
            persistPolicy="OnUpdate")
    public void setName(String name) {
        this.name = name;
    }

    @ManagedAttribute(defaultValue="foo", persistPeriod=300)
    public String getName() {
        return name;
    }

    @ManagedOperation(description="Add two numbers")
    @ManagedOperationParameters({
        @ManagedOperationParameter(name = "x", description = "The first number"),
        @ManagedOperationParameter(name = "y", description = "The second number")})
    public int add(int x, int y) {
        return x + y;
    }

    public void dontExposeMe() {
        throw new RuntimeException();
    }

}
```
在前面的示例中，您可以看到JmxTestBean该类标记有 ManagedResource注释，并且该ManagedResource注释配置了一组属性。这些属性可用于配置MBean生成的MBean的各个方面，MBeanExporter稍后将在Source-level Metadata Types中进行详细说明。

无论是age和name属性注释与ManagedAttribute 注释，但是，在的情况下age财产，只标记了getter。这导致这两个属性都作为属性包含在管理界面中，但是该age属性是只读的。

最后，该add(int, int)方法带有ManagedOperation属性标记，而dontExposeMe()方法则没有。使用时，这将导致管理界面仅包含一个操作（add(int, int)）MetadataMBeanInfoAssembler。

以下配置显示了如何配置MBeanExporter来使用 MetadataMBeanInfoAssembler：
```xml
<beans>
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="assembler" ref="assembler"/>
        <property name="namingStrategy" ref="namingStrategy"/>
        <property name="autodetect" value="true"/>
    </bean>

    <bean id="jmxAttributeSource"
            class="org.springframework.jmx.export.annotation.AnnotationJmxAttributeSource"/>

    <!-- will create management interface using annotation metadata -->
    <bean id="assembler"
            class="org.springframework.jmx.export.assembler.MetadataMBeanInfoAssembler">
        <property name="attributeSource" ref="jmxAttributeSource"/>
    </bean>

    <!-- will pick up the ObjectName from the annotation -->
    <bean id="namingStrategy"
            class="org.springframework.jmx.export.naming.MetadataNamingStrategy">
        <property name="attributeSource" ref="jmxAttributeSource"/>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.AnnotationTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>
</beans>
```
在前面的示例中，MetadataMBeanInfoAssembler已经为Bean配置了AnnotationJmxAttributeSource该类的实例，并通过MBeanExporter 汇编程序属性将其传递给。这是利用Spring公开的MBean的元数据驱动的管理接口所需要的全部。
#### 5.2.3. 源级元数据类型
下表描述了可在Spring JMX中使用的源级别元数据类型：

表7.源级元数据类型
目的	注解	注释类型
将a的所有实例标记Class为JMX托管资源。

@ManagedResource

类

将方法标记为JMX操作。

@ManagedOperation

方法

将一个getter或setter标记为JMX属性的一半。

@ManagedAttribute

方法（仅getter和setter）

定义操作参数的描述。

@ManagedOperationParameter 和 @ManagedOperationParameters

方法

下表描述了可在这些源级元数据类型上使用的配置参数：

表8.源级元数据参数
参数	描述	适用于
ObjectName

用于MetadataNamingStrategy确定ObjectName托管资源的。

ManagedResource

description

设置资源，属性或操作的友好描述。

ManagedResource，ManagedAttribute，ManagedOperation，或者ManagedOperationParameter

currencyTimeLimit

设置currencyTimeLimit描述符字段的值。

ManagedResource 要么 ManagedAttribute

defaultValue

设置defaultValue描述符字段的值。

ManagedAttribute

log

设置log描述符字段的值。

ManagedResource

logFile

设置logFile描述符字段的值。

ManagedResource

persistPolicy

设置persistPolicy描述符字段的值。

ManagedResource

persistPeriod

设置persistPeriod描述符字段的值。

ManagedResource

persistLocation

设置persistLocation描述符字段的值。

ManagedResource

persistName

设置persistName描述符字段的值。

ManagedResource

name

设置操作参数的显示名称。

ManagedOperationParameter

index

设置操作参数的索引。

ManagedOperationParameter
#### 5.2.4. 用AutodetectCapableMBeanInfoAssembler介面
为了进一步简化配置，Spring包含了该 AutodetectCapableMBeanInfoAssembler接口，该MBeanInfoAssembler 接口扩展了该接口以添加对自动检测MBean资源的支持。如果您MBeanExporter使用实例配置 AutodetectCapableMBeanInfoAssembler，则可以对包含的Bean进行“投票”以暴露给JMX。

该AutodetectCapableMBeanInfo接口的唯一实现是MetadataMBeanInfoAssembler，该投票将包括该ManagedResource属性标记的任何bean 。在这种情况下，默认方法是将Bean名称用作ObjectName，这将导致类似于以下内容的配置：
```xml
<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <!-- notice how no 'beans' are explicitly configured here -->
        <property name="autodetect" value="true"/>
        <property name="assembler" ref="assembler"/>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

    <bean id="assembler" class="org.springframework.jmx.export.assembler.MetadataMBeanInfoAssembler">
        <property name="attributeSource">
            <bean class="org.springframework.jmx.export.annotation.AnnotationJmxAttributeSource"/>
        </property>
    </bean>

</beans>
```
请注意，在上述配置中，没有将任何bean传递给MBeanExporter。但是，由于JmxTestBean仍使用ManagedResource 属性标记并MetadataMBeanInfoAssembler检测到该属性并对其进行表决，因此该属性仍处于注册状态。这种方法的唯一问题是JmxTestBeannow的名称具有商业意义。您可以通过更改“控制Bean实例”中ObjectName 定义的默认创建行为来解决此问题。ObjectName
#### 5.2.5. 使用Java接口定义管理接口
除了之外MetadataMBeanInfoAssembler，Spring还包括 InterfaceBasedMBeanInfoAssembler，您可以使用来约束基于接口集合中定义的方法集公开的方法和属性。

尽管公开MBean的标准机制是使用接口和简单的命名方案，InterfaceBasedMBeanInfoAssembler但是通过消除对命名约定的需要，允许您使用多个接口以及消除对实现MBean接口的bean的需求，扩展了此功能。

考虑以下接口，该接口用于为JmxTestBean我们前面显示的类定义管理接口 ：
```java
public interface IJmxTestBean {

    public int add(int x, int y);

    public long myOperation();

    public int getAge();

    public void setAge(int age);

    public void setName(String name);

    public String getName();

}
```
该接口定义在JMX MBean上作为操作和属性公开的方法和属性。以下代码显示了如何配置Spring JMX以使用此接口作为管理接口的定义：
```xml
<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
            <map>
                <entry key="bean:name=testBean5" value-ref="testBean"/>
            </map>
        </property>
        <property name="assembler">
            <bean class="org.springframework.jmx.export.assembler.InterfaceBasedMBeanInfoAssembler">
                <property name="managedInterfaces">
                    <value>org.springframework.jmx.IJmxTestBean</value>
                </property>
            </bean>
        </property>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

</beans>
```
在前面的示例中，将InterfaceBasedMBeanInfoAssembler其配置为IJmxTestBean在构造任何bean的管理接口时使用该 接口。重要的是要理解，InterfaceBasedMBeanInfoAssembler 不需要由处理的bean来实现用于生成JMX管理接口的接口。

在上述情况下，该IJmxTestBean接口用于为所有bean构造所有管理接口。在许多情况下，这不是理想的行为，您可能想对不同的bean使用不同的接口。在这种情况下，您可以通过该 属性传递 InterfaceBasedMBeanInfoAssembler一个Properties实例interfaceMappings，其中每个条目的键是Bean名称，每个条目的值是一个逗号分隔的接口名称列表，用于该Bean。

如果没有通过managedInterfaces或 interfaceMappings属性指定管理接口，则InterfaceBasedMBeanInfoAssemblerref会反映在bean上，并使用该bean实现的所有接口来创建管理接口。
#### 5.2.6. 使用MethodNameBasedMBeanInfoAssembler
MBeanInfoAssembler接口
5.2.2。使用源级元数据：Java注释
5.2.3。源级元数据类型
5.2.4。使用AutodetectCapableMBeanInfoAssembler接口
5.2.5。使用Java接口定义管理接口
5.2.6。使用MethodNameBasedMBeanInfoAssembler
5.3。控制您的Bean的ObjectName实例
5.4。使用JSR-160连接器
5.5。通过代理访问MBean
5.6。通知事项
5.7。更多资源
6.电子邮件
7.任务执行和计划
8.缓存抽象
9.附录
参考文档的这一部分涵盖了Spring Framework与多种技术的集成。

1. REST端点
Spring框架提供了两种选择来调用REST端点：

RestTemplate：具有同步模板方法API的原始Spring REST客户端。

WebClient：一种非阻塞的，反应性的替代方案，它支持同步和异步以及流方案。

从5.0版本开始，RestTemplate它处于维护模式，以后只有很少的更改和错误请求被接受。请考虑使用提供更现代API并支持同步，异步和流方案的 WebClient。
1.1。 RestTemplate
在RestTemplate通过HTTP客户端库提供了一个更高水平的API。它使在一行中轻松调用REST端点变得容易。它公开了以下几组重载方法：

表1. RestTemplate方法
方法组	描述
getForObject

通过GET检索表示形式。

getForEntity

ResponseEntity使用GET检索（即状态，标题和正文）。

headForHeaders

通过使用HEAD检索资源的所有标头。

postForLocation

通过使用POST创建新资源，并Location从响应中返回标头。

postForObject

通过使用POST创建新资源，并从响应中返回表示形式。

postForEntity

通过使用POST创建新资源，并从响应中返回表示形式。

put

通过使用PUT创建或更新资源。

patchForObject

通过使用PATCH更新资源，并从响应中返回表示形式。请注意，JDKHttpURLConnection不支持PATCH，但是Apache HttpComponents和其他支持。

delete

使用DELETE删除指定URI处的资源。

optionsForAllow

通过使用ALLOW检索资源的允许的HTTP方法。

exchange

前述方法的通用性强（且意见少的）版本，在需要时提供了额外的灵活性。它接受RequestEntity（包括HTTP方法，URL，标头和正文作为输入）并返回ResponseEntity。

这些方法允许使用ParameterizedTypeReference而不是Class使用泛型来指定响应类型。

execute

执行请求的最通用方法，完全控制通过回调接口进行的请求准备和响应提取。

1.1.1。初始化
默认构造函数用于java.net.HttpURLConnection执行请求。您可以使用的实现切换到其他HTTP库ClientHttpRequestFactory。内置支持以下内容：

Apache HttpComponents

净额

OkHttp

例如，要切换到Apache HttpComponents，可以使用以下命令：

RestTemplate template = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
Each ClientHttpRequestFactory exposes configuration options specific to the underlying HTTP client library — for example, for credentials, connection pooling, and other details.

Note that the java.net implementation for HTTP requests can raise an exception when accessing the status of a response that represents an error (such as 401). If this is an issue, switch to another HTTP client library.
URIs
Many of the RestTemplate methods accept a URI template and URI template variables, either as a String variable argument, or as Map<String,String>.

The following example uses a String variable argument:

String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/bookings/{booking}", String.class, "42", "21");
The following example uses a Map<String, String>:

Map<String, String> vars = Collections.singletonMap("hotel", "42");

String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/rooms/{hotel}", String.class, vars);
Keep in mind URI templates are automatically encoded, as the following example shows:

restTemplate.getForObject("https://example.com/hotel list", String.class);

// Results in request to "https://example.com/hotel%20list"
You can use the uriTemplateHandler property of RestTemplate to customize how URIs are encoded. Alternatively, you can prepare a java.net.URI and pass it into one of the RestTemplate methods that accepts a URI.

For more details on working with and encoding URIs, see URI Links.

Headers
You can use the exchange() methods to specify request headers, as the following example shows:

String uriTemplate = "https://example.com/hotels/{hotel}";
URI uri = UriComponentsBuilder.fromUriString(uriTemplate).build(42);

RequestEntity<Void> requestEntity = RequestEntity.get(uri)
        .header(("MyRequestHeader", "MyValue")
        .build();

ResponseEntity<String> response = template.exchange(requestEntity, String.class);

String responseHeader = response.getHeaders().getFirst("MyResponseHeader");
String body = response.getBody();
You can obtain response headers through many RestTemplate method variants that return ResponseEntity.

1.1.2. Body
Objects passed into and returned from RestTemplate methods are converted to and from raw content with the help of an HttpMessageConverter.

On a POST, an input object is serialized to the request body, as the following example shows:

URI location = template.postForLocation("https://example.com/people", person);
You need not explicitly set the Content-Type header of the request. In most cases, you can find a compatible message converter based on the source Object type, and the chosen message converter sets the content type accordingly. If necessary, you can use the exchange methods to explicitly provide the Content-Type request header, and that, in turn, influences what message converter is selected.

On a GET, the body of the response is deserialized to an output Object, as the following example shows:

Person person = restTemplate.getForObject("https://example.com/people/{id}", Person.class, 42);
The Accept header of the request does not need to be explicitly set. In most cases, a compatible message converter can be found based on the expected response type, which then helps to populate the Accept header. If necessary, you can use the exchange methods to provide the Accept header explicitly.

By default, RestTemplate registers all built-in message converters, depending on classpath checks that help to determine what optional conversion libraries are present. You can also set the message converters to use explicitly.

1.1.3. Message Conversion
WebFlux

The spring-web module contains the HttpMessageConverter contract for reading and writing the body of HTTP requests and responses through InputStream and OutputStream. HttpMessageConverter instances are used on the client side (for example, in the RestTemplate) and on the server side (for example, in Spring MVC REST controllers).

Concrete implementations for the main media (MIME) types are provided in the framework and are, by default, registered with the RestTemplate on the client side and with RequestMethodHandlerAdapter on the server side (see Configuring Message Converters).

The implementations of HttpMessageConverter are described in the following sections. For all converters, a default media type is used, but you can override it by setting the supportedMediaTypes bean property. The following table describes each implementation:

Table 2. HttpMessageConverter Implementations
MessageConverter	Description
StringHttpMessageConverter

An HttpMessageConverter implementation that can read and write String instances from the HTTP request and response. By default, this converter supports all text media types (text/*) and writes with a Content-Type of text/plain.

FormHttpMessageConverter

An HttpMessageConverter implementation that can read and write form data from the HTTP request and response. By default, this converter reads and writes the application/x-www-form-urlencoded media type. Form data is read from and written into a MultiValueMap<String, String>. The converter can also write (but not read) multipart data read from a MultiValueMap<String, Object>. By default, multipart/form-data is supported. As of Spring Framework 5.2, additional multipart subtypes can be supported for writing form data. Consult the javadoc for FormHttpMessageConverter for further details.

ByteArrayHttpMessageConverter

An HttpMessageConverter implementation that can read and write byte arrays from the HTTP request and response. By default, this converter supports all media types (*/*) and writes with a Content-Type of application/octet-stream. You can override this by setting the supportedMediaTypes property and overriding getContentType(byte[]).

MarshallingHttpMessageConverter

An HttpMessageConverter implementation that can read and write XML by using Spring’s Marshaller and Unmarshaller abstractions from the org.springframework.oxm package. This converter requires a Marshaller and Unmarshaller before it can be used. You can inject these through constructor or bean properties. By default, this converter supports text/xml and application/xml.

MappingJackson2HttpMessageConverter

An HttpMessageConverter implementation that can read and write JSON by using Jackson’s ObjectMapper. You can customize JSON mapping as needed through the use of Jackson’s provided annotations. When you need further control (for cases where custom JSON serializers/deserializers need to be provided for specific types), you can inject a custom ObjectMapper through the ObjectMapper property. By default, this converter supports application/json.

MappingJackson2XmlHttpMessageConverter

An HttpMessageConverter implementation that can read and write XML by using Jackson XML extension’s XmlMapper. You can customize XML mapping as needed through the use of JAXB or Jackson’s provided annotations. When you need further control (for cases where custom XML serializers/deserializers need to be provided for specific types), you can inject a custom XmlMapper through the ObjectMapper property. By default, this converter supports application/xml.

SourceHttpMessageConverter

An HttpMessageConverter implementation that can read and write javax.xml.transform.Source from the HTTP request and response. Only DOMSource, SAXSource, and StreamSource are supported. By default, this converter supports text/xml and application/xml.

BufferedImageHttpMessageConverter

An HttpMessageConverter implementation that can read and write java.awt.image.BufferedImage from the HTTP request and response. This converter reads and writes the media type supported by the Java I/O API.

1.1.4. Jackson JSON Views
You can specify a Jackson JSON View to serialize only a subset of the object properties, as the following example shows:

MappingJacksonValue value = new MappingJacksonValue(new User("eric", "7!jd#h23"));
value.setSerializationView(User.WithoutPasswordView.class);

RequestEntity<MappingJacksonValue> requestEntity =
    RequestEntity.post(new URI("https://example.com/user")).body(value);

ResponseEntity<String> response = template.exchange(requestEntity, String.class);
Multipart
To send multipart data, you need to provide a MultiValueMap<String, Object> whose values may be an Object for part content, a Resource for a file part, or an HttpEntity for part content with headers. For example:

MultiValueMap<String, Object> parts = new LinkedMultiValueMap<>();

parts.add("fieldPart", "fieldValue");
parts.add("filePart", new FileSystemResource("...logo.png"));
parts.add("jsonPart", new Person("Jason"));

HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_XML);
parts.add("xmlPart", new HttpEntity<>(myBean, headers));
In most cases, you do not have to specify the Content-Type for each part. The content type is determined automatically based on the HttpMessageConverter chosen to serialize it or, in the case of a Resource based on the file extension. If necessary, you can explicitly provide the MediaType with an HttpEntity wrapper.

Once the MultiValueMap is ready, you can pass it to the RestTemplate, as show below:

MultiValueMap<String, Object> parts = ...;
template.postForObject("https://example.com/upload", parts, Void.class);
If the MultiValueMap contains at least one non-String value, the Content-Type is set to multipart/form-data by the FormHttpMessageConverter. If the MultiValueMap has String values the Content-Type is defaulted to application/x-www-form-urlencoded. If necessary the Content-Type may also be set explicitly.

1.2. Using AsyncRestTemplate (Deprecated)
The AsyncRestTemplate is deprecated. For all use cases where you might consider using AsyncRestTemplate, use the WebClient instead.

2. Remoting and Web Services
Spring provides support for remoting with various technologies. The remoting support eases the development of remote-enabled services, implemented via Java interfaces and objects as input and output. Currently, Spring supports the following remoting technologies:

Java Web Services: Spring provides remoting support for web services through JAX-WS.

AMQP: Remoting via AMQP as the underlying protocol is supported by the separate Spring AMQP project.

As of Spring Framework 5.3, support for several remoting technologies is now deprecated for security reasons and broader industry support. Supporting infrastructure will be removed from Spring Framework for its next major release.
The following remoting technologies are now deprecated and will not be replaced:

Remote Method Invocation (RMI): Through the use of RmiProxyFactoryBean and RmiServiceExporter, Spring supports both traditional RMI (with java.rmi.Remote interfaces and java.rmi.RemoteException) and transparent remoting through RMI invokers (with any Java interface).

Spring HTTP Invoker (Deprecated): Spring provides a special remoting strategy that allows for Java serialization though HTTP, supporting any Java interface (as the RMI invoker does). The corresponding support classes are HttpInvokerProxyFactoryBean and HttpInvokerServiceExporter.

Hessian: By using Spring’s HessianProxyFactoryBean and the HessianServiceExporter, you can transparently expose your services through the lightweight binary HTTP-based protocol provided by Caucho.

JMS (Deprecated): Remoting via JMS as the underlying protocol is supported through the JmsInvokerServiceExporter and JmsInvokerProxyFactoryBean classes in the spring-jms module.

While discussing the remoting capabilities of Spring, we use the following domain model and corresponding services:

public class Account implements Serializable{

    private String name;
    
    public String getName(){
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}
public interface AccountService {

    public void insertAccount(Account account);
    
    public List<Account> getAccounts(String name);
}
// the implementation doing nothing at the moment
public class AccountServiceImpl implements AccountService {

    public void insertAccount(Account acc) {
        // do something...
    }
    
    public List<Account> getAccounts(String name) {
        // do something...
    }
}
This section starts by exposing the service to a remote client by using RMI and talk a bit about the drawbacks of using RMI. It then continues with an example that uses Hessian as the protocol.

2.1. AMQP
Remoting via AMQP as the underlying protocol is supported in the Spring AMQP project. For further details please visit the Spring Remoting section of the Spring AMQP reference.

Auto-detection is not implemented for remote interfaces

The main reason why auto-detection of implemented interfaces does not occur for remote interfaces is to avoid opening too many doors to remote callers. The target object might implement internal callback interfaces, such as InitializingBean or DisposableBean which one would not want to expose to callers.

Offering a proxy with all interfaces implemented by the target usually does not matter in the local case. However, when you export a remote service, you should expose a specific service interface, with specific operations intended for remote usage. Besides internal callback interfaces, the target might implement multiple business interfaces, with only one of them intended for remote exposure. For these reasons, we require such a service interface to be specified.

This is a trade-off between configuration convenience and the risk of accidental exposure of internal methods. Always specifying a service interface is not too much effort and puts you on the safe side regarding controlled exposure of specific methods.

2.2. Considerations when Choosing a Technology
Each and every technology presented here has its drawbacks. When choosing a technology, you should carefully consider your needs, the services you expose, and the objects you send over the wire.

When using RMI, you cannot access the objects through the HTTP protocol, unless you tunnel the RMI traffic. RMI is a fairly heavy-weight protocol, in that it supports full-object serialization, which is important when you use a complex data model that needs serialization over the wire. However, RMI-JRMP is tied to Java clients. It is a Java-to-Java remoting solution.

Spring’s HTTP invoker is a good choice if you need HTTP-based remoting but also rely on Java serialization. It shares the basic infrastructure with RMI invokers but uses HTTP as transport. Note that HTTP invokers are not limited only to Java-to-Java remoting but also to Spring on both the client and the server side. (The latter also applies to Spring’s RMI invoker for non-RMI interfaces.)

Hessian might provide significant value when operating in a heterogeneous environment, because they explicitly allow for non-Java clients. However, non-Java support is still limited. Known issues include the serialization of Hibernate objects in combination with lazily-initialized collections. If you have such a data model, consider using RMI or HTTP invokers instead of Hessian.

JMS can be useful for providing clusters of services and letting the JMS broker take care of load balancing, discovery, and auto-failover. By default, Java serialization is used for JMS remoting, but the JMS provider could use a different mechanism for the wire formatting, such as XStream to let servers be implemented in other technologies.

Last but not least, EJB has an advantage over RMI, in that it supports standard role-based authentication and authorization and remote transaction propagation. It is possible to get RMI invokers or HTTP invokers to support security context propagation as well, although this is not provided by core Spring. Spring offers only appropriate hooks for plugging in third-party or custom solutions.

2.3. Java Web Services
Spring provides full support for the standard Java web services APIs:

Exposing web services using JAX-WS

Accessing web services using JAX-WS

In addition to stock support for JAX-WS in Spring Core, the Spring portfolio also features Spring Web Services, which is a solution for contract-first, document-driven web services — highly recommended for building modern, future-proof web services.

2.3.1. Exposing Servlet-based Web Services by Using JAX-WS
Spring provides a convenient base class for JAX-WS servlet endpoint implementations: SpringBeanAutowiringSupport. To expose our AccountService, we extend Spring’s SpringBeanAutowiringSupport class and implement our business logic here, usually delegating the call to the business layer. We use Spring’s @Autowired annotation to express such dependencies on Spring-managed beans. The following example shows our class that extends SpringBeanAutowiringSupport:

/**
 * JAX-WS compliant AccountService implementation that simply delegates
 * to the AccountService implementation in the root web application context.
 *
 * This wrapper class is necessary because JAX-WS requires working with dedicated
 * endpoint classes. If an existing service needs to be exported, a wrapper that
 * extends SpringBeanAutowiringSupport for simple Spring bean autowiring (through
 * the @Autowired annotation) is the simplest JAX-WS compliant way.
 *
 * This is the class registered with the server-side JAX-WS implementation.
 * In the case of a Java EE server, this would simply be defined as a servlet
 * in web.xml, with the server detecting that this is a JAX-WS endpoint and reacting
 * accordingly. The servlet name usually needs to match the specified WS service name.
 *
 * The web service engine manages the lifecycle of instances of this class.
 * Spring bean references will just be wired in here.
 */
import org.springframework.web.context.support.SpringBeanAutowiringSupport;

@WebService(serviceName="AccountService")
public class AccountServiceEndpoint extends SpringBeanAutowiringSupport {

    @Autowired
    private AccountService biz;
    
    @WebMethod
    public void insertAccount(Account acc) {
        biz.insertAccount(acc);
    }
    
    @WebMethod
    public Account[] getAccounts(String name) {
        return biz.getAccounts(name);
    }
}
Our AccountServiceEndpoint needs to run in the same web application as the Spring context to allow for access to Spring’s facilities. This is the case by default in Java EE environments, using the standard contract for JAX-WS servlet endpoint deployment. See the various Java EE web service tutorials for details.

2.3.2. Exporting Standalone Web Services by Using JAX-WS
The built-in JAX-WS provider that comes with Oracle’s JDK supports exposure of web services by using the built-in HTTP server that is also included in the JDK. Spring’s SimpleJaxWsServiceExporter detects all @WebService-annotated beans in the Spring application context and exports them through the default JAX-WS server (the JDK HTTP server).

在这种情况下，终结点实例被定义和管理为Spring bean本身。它们已在JAX-WS引擎中注册，但是它们的生命周期取决于Spring应用程序上下文。这意味着您可以将Spring功能（例如显式依赖项注入）应用于端点实例。注释驱动的注入也@Autowired同样有效。以下示例显示了如何定义这些bean：

<bean class="org.springframework.remoting.jaxws.SimpleJaxWsServiceExporter">
    <property name="baseAddress" value="http://localhost:8080/"/>
</bean>

<bean id="accountServiceEndpoint" class="example.AccountServiceEndpoint">
    ...
</bean>

...
该AccountServiceEndpoint可以，但并没有从Spring的推导SpringBeanAutowiringSupport，因为在这个例子中，端点是完全春季管理的bean。这意味着端点实现可以如下所示（不声明任何超类，并且@Autowired仍采用Spring的配置注释）：

@WebService(serviceName="AccountService")
public class AccountServiceEndpoint {

    @Autowired
    private AccountService biz;
    
    @WebMethod
    public void insertAccount(Account acc) {
        biz.insertAccount(acc);
    }
    
    @WebMethod
    public List<Account> getAccounts(String name) {
        return biz.getAccounts(name);
    }
}
2.3.3。使用JAX-WS RI的Spring支持导出Web服务
作为GlassFish项目的一部分开发的Oracle JAX-WS RI，将Spring支持作为其JAX-WS Commons项目的一部分。这允许将JAX-WS端点定义为Spring管理的Bean，类似于上一节中讨论的独立模式，  但是这次是在Servlet环境中。

这在Java EE环境中不可移植。它主要适用于将JAX-WS RI嵌入为Web应用程序一部分的非EE环境，例如Tomcat。
与导出基于servlet的终结点的标准样式的不同之处在于，终结点实例本身的生命周期由Spring管理，并且仅定义了一个JAX-WS servlet web.xml。使用标准的Java EE样式（如前所述），每个服务端点都有一个servlet定义，每个端点通常都委派给Spring Bean（@Autowired如前所述，通过使用）。

有关 设置和使用样式的详细信息，请参见https://jax-ws-commons.java.net/spring/。

2.3.4。使用JAX-WS访问Web服务
Spring provides two factory beans to create JAX-WS web service proxies, namely LocalJaxWsServiceFactoryBean and JaxWsPortProxyFactoryBean. The former can return only a JAX-WS service class for us to work with. The latter is the full-fledged version that can return a proxy that implements our business service interface. In the following example, we use JaxWsPortProxyFactoryBean to create a proxy for the AccountService endpoint (again):

<bean id="accountWebService" class="org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean">
    <property name="serviceInterface" value="example.AccountService"/> 
    <property name="wsdlDocumentUrl" value="http://localhost:8888/AccountServiceEndpoint?WSDL"/>
    <property name="namespaceUri" value="https://example/"/>
    <property name="serviceName" value="AccountService"/>
    <property name="portName" value="AccountServiceEndpointPort"/>
</bean>
Where serviceInterface is our business interface that the clients use.
wsdlDocumentUrl is the URL for the WSDL file. Spring needs this at startup time to create the JAX-WS Service. namespaceUri corresponds to the targetNamespace in the .wsdl file. serviceName corresponds to the service name in the .wsdl file. portName corresponds to the port name in the .wsdl file.

Accessing the web service is easy, as we have a bean factory for it that exposes it as an interface called AccountService. The following example shows how we can wire this up in Spring:

<bean id="client" class="example.AccountClientImpl">
    ...
    <property name="service" ref="accountWebService"/>
</bean>
From the client code, we can access the web service as if it were a normal class, as the following example shows:

public class AccountClientImpl {

    private AccountService service;
    
    public void setService(AccountService service) {
        this.service = service;
    }
    
    public void foo() {
        service.insertAccount(...);
    }
}
The above is slightly simplified in that JAX-WS requires endpoint interfaces and implementation classes to be annotated with @WebService, @SOAPBinding etc annotations. This means that you cannot (easily) use plain Java interfaces and implementation classes as JAX-WS endpoint artifacts; you need to annotate them accordingly first. Check the JAX-WS documentation for details on those requirements.
2.4. RMI (Deprecated)
As of Spring Framework 5.3, RMI support is deprecated and will not be replaced.
By using Spring’s support for RMI, you can transparently expose your services through the RMI infrastructure. After having this set up, you basically have a configuration similar to remote EJBs, except for the fact that there is no standard support for security context propagation or remote transaction propagation. Spring does provide hooks for such additional invocation context when you use the RMI invoker, so you can, for example, plug in security frameworks or custom security credentials.

2.4.1. Exporting the Service by Using RmiServiceExporter
Using the RmiServiceExporter, we can expose the interface of our AccountService object as RMI object. The interface can be accessed by using RmiProxyFactoryBean, or via plain RMI in case of a traditional RMI service. The RmiServiceExporter explicitly supports the exposing of any non-RMI services via RMI invokers.

We first have to set up our service in the Spring container. The following example shows how to do so:

<bean id="accountService" class="example.AccountServiceImpl">
    <!-- any additional properties, maybe a DAO? -->
</bean>
Next, we have to expose our service by using RmiServiceExporter. The following example shows how to do so:

<bean class="org.springframework.remoting.rmi.RmiServiceExporter">
    <!-- does not necessarily have to be the same name as the bean to be exported -->
    <property name="serviceName" value="AccountService"/>
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
    <!-- defaults to 1099 -->
    <property name="registryPort" value="1199"/>
</bean>
In the preceding example, we override the port for the RMI registry. Often, your application server also maintains an RMI registry, and it is wise to not interfere with that one. Furthermore, the service name is used to bind the service. So, in the preceding example, the service is bound at 'rmi://HOST:1199/AccountService'. We use this URL later on to link in the service at the client side.

The servicePort property has been omitted (it defaults to 0). This means that an anonymous port is used to communicate with the service.
2.4.2. Linking in the Service at the Client
Our client is a simple object that uses the AccountService to manage accounts, as the following example shows:

public class SimpleObject {

    private AccountService accountService;
    
    public void setAccountService(AccountService accountService) {
        this.accountService = accountService;
    }
    
    // additional methods using the accountService
}
To link in the service on the client, we create a separate Spring container, to contain the following simple object and the service linking configuration bits:

<bean class="example.SimpleObject">
    <property name="accountService" ref="accountService"/>
</bean>

<bean id="accountService" class="org.springframework.remoting.rmi.RmiProxyFactoryBean">
    <property name="serviceUrl" value="rmi://HOST:1199/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
That is all we need to do to support the remote account service on the client. Spring transparently creates an invoker and remotely enables the account service through the RmiServiceExporter. At the client, we link it in by using the RmiProxyFactoryBean.

2.5. Using Hessian to Remotely Call Services through HTTP (Deprecated)
As of Spring Framework 5.3, Hessian support is deprecated and will not be replaced.
Hessian offers a binary HTTP-based remoting protocol. It is developed by Caucho, and you can find more information about Hessian itself at https://www.caucho.com/.

2.5.1. Hessian
Hessian communicates through HTTP and does so by using a custom servlet. By using Spring’s DispatcherServlet principles (see webmvc.html), we can wire up such a servlet to expose your services. First, we have to create a new servlet in our application, as shown in the following excerpt from web.xml:

<servlet>
    <servlet-name>remoting</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>remoting</servlet-name>
    <url-pattern>/remoting/*</url-pattern>
</servlet-mapping>
If you are familiar with Spring’s DispatcherServlet principles, you probably know that now you have to create a Spring container configuration resource named remoting-servlet.xml (after the name of your servlet) in the WEB-INF directory. The application context is used in the next section.

Alternatively, consider the use of Spring’s simpler HttpRequestHandlerServlet. Doing so lets you embed the remote exporter definitions in your root application context (by default, in WEB-INF/applicationContext.xml), with individual servlet definitions pointing to specific exporter beans. In this case, each servlet name needs to match the bean name of its target exporter.

2.5.2. Exposing Your Beans by Using HessianServiceExporter
In the newly created application context called remoting-servlet.xml, we create a HessianServiceExporter to export our services, as the following example shows:

<bean id="accountService" class="example.AccountServiceImpl">
    <!-- any additional properties, maybe a DAO? -->
</bean>

<bean name="/AccountService" class="org.springframework.remoting.caucho.HessianServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
Now we are ready to link in the service at the client. No explicit handler mapping is specified (to map request URLs onto services), so we use BeanNameUrlHandlerMapping used. Hence, the service is exported at the URL indicated through its bean name within the containing DispatcherServlet instance’s mapping (as defined earlier): https://HOST:8080/remoting/AccountService.

Alternatively, you can create a HessianServiceExporter in your root application context (for example, in WEB-INF/applicationContext.xml), as the following example shows:

<bean name="accountExporter" class="org.springframework.remoting.caucho.HessianServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
In the latter case, you should define a corresponding servlet for this exporter in web.xml, with the same end result: The exporter gets mapped to the request path at /remoting/AccountService. Note that the servlet name needs to match the bean name of the target exporter. The following example shows how to do so:

<servlet>
    <servlet-name>accountExporter</servlet-name>
    <servlet-class>org.springframework.web.context.support.HttpRequestHandlerServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>accountExporter</servlet-name>
    <url-pattern>/remoting/AccountService</url-pattern>
</servlet-mapping>
2.5.3. Linking in the Service on the Client
By using the HessianProxyFactoryBean, we can link in the service at the client. The same principles apply as with the RMI example. We create a separate bean factory or application context and mention the following beans where the SimpleObject is by using the AccountService to manage accounts, as the following example shows:

<bean class="example.SimpleObject">
    <property name="accountService" ref="accountService"/>
</bean>

<bean id="accountService" class="org.springframework.remoting.caucho.HessianProxyFactoryBean">
    <property name="serviceUrl" value="https://remotehost:8080/remoting/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
2.5.4. Applying HTTP Basic Authentication to a Service Exposed through Hessian
One of the advantages of Hessian is that we can easily apply HTTP basic authentication, because both protocols are HTTP-based. Your normal HTTP server security mechanism can be applied through using the web.xml security features, for example. Usually, you need not use per-user security credentials here. Rather, you can use shared credentials that you define at the HessianProxyFactoryBean level (similar to a JDBC DataSource), as the following example shows:

<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping">
    <property name="interceptors" ref="authorizationInterceptor"/>
</bean>

<bean id="authorizationInterceptor"
        class="org.springframework.web.servlet.handler.UserRoleAuthorizationInterceptor">
    <property name="authorizedRoles" value="administrator,operator"/>
</bean>
In the preceding example, we explicitly mention the BeanNameUrlHandlerMapping and set an interceptor, to let only administrators and operators call the beans mentioned in this application context.

The preceding example does not show a flexible kind of security infrastructure. For more options as far as security is concerned, have a look at the Spring Security project at https://projects.spring.io/spring-security/.
2.6. Spring HTTP Invoker (Deprecated)
As of Spring Framework 5.3, HTTP Invoker support is deprecated and will not be replaced.
As opposed to Hessian, Spring HTTP invokers are both lightweight protocols that use their own slim serialization mechanisms and use the standard Java serialization mechanism to expose services through HTTP. This has a huge advantage if your arguments and return types are complex types that cannot be serialized by using the serialization mechanisms Hessian uses (see the next section for more considerations when you choose a remoting technology).

Under the hood, Spring uses either the standard facilities provided by the JDK or Apache HttpComponents to perform HTTP calls. If you need more advanced and easier-to-use functionality, use the latter. See hc.apache.org/httpcomponents-client-ga/ for more information.

Be aware of vulnerabilities due to unsafe Java deserialization: Manipulated input streams can lead to unwanted code execution on the server during the deserialization step. As a consequence, do not expose HTTP invoker endpoints to untrusted clients. Rather, expose them only between your own services. In general, we strongly recommend using any other message format (such as JSON) instead.

If you are concerned about security vulnerabilities due to Java serialization, consider the general-purpose serialization filter mechanism at the core JVM level, originally developed for JDK 9 but backported to JDK 8, 7 and 6 in the meantime. See https://blogs.oracle.com/java-platform-group/entry/incoming_filter_serialization_data_a and https://openjdk.java.net/jeps/290.

2.6.1. Exposing the Service Object
Setting up the HTTP invoker infrastructure for a service object closely resembles the way you would do the same by using Hessian. As Hessian support provides HessianServiceExporter, Spring’s HttpInvoker support provides org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter.

To expose the AccountService (mentioned earlier) within a Spring Web MVC DispatcherServlet, the following configuration needs to be in place in the dispatcher’s application context, as the following example shows:

<bean name="/AccountService" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
Such an exporter definition is exposed through the DispatcherServlet instance’s standard mapping facilities, as explained in the section on Hessian.

Alternatively, you can create an HttpInvokerServiceExporter in your root application context (for example, in 'WEB-INF/applicationContext.xml'), as the following example shows:

<bean name="accountExporter" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
In addition, you can define a corresponding servlet for this exporter in web.xml, with the servlet name matching the bean name of the target exporter, as the following example shows:

<servlet>
    <servlet-name>accountExporter</servlet-name>
    <servlet-class>org.springframework.web.context.support.HttpRequestHandlerServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>accountExporter</servlet-name>
    <url-pattern>/remoting/AccountService</url-pattern>
</servlet-mapping>
2.6.2. Linking in the Service at the Client
Again, linking in the service from the client much resembles the way you would do it when you use Hessian. By using a proxy, Spring can translate your calls to HTTP POST requests to the URL that points to the exported service. The following example shows how to configure this arrangement:

<bean id="httpInvokerProxy" class="org.springframework.remoting.httpinvoker.HttpInvokerProxyFactoryBean">
    <property name="serviceUrl" value="https://remotehost:8080/remoting/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
As mentioned earlier, you can choose what HTTP client you want to use. By default, the HttpInvokerProxy uses the JDK’s HTTP functionality, but you can also use the Apache HttpComponents client by setting the httpInvokerRequestExecutor property. The following example shows how to do so:

<property name="httpInvokerRequestExecutor">
    <bean class="org.springframework.remoting.httpinvoker.HttpComponentsHttpInvokerRequestExecutor"/>
</property>
2.7. JMS (Deprecated)
As of Spring Framework 5.3, JMS remoting support is deprecated and will not be replaced.
You can also expose services transparently by using JMS as the underlying communication protocol. The JMS remoting support in the Spring Framework is pretty basic. It sends and receives on the same thread and in the same non-transactional Session. As a result, throughput is implementation-dependent. Note that these single-threaded and non-transactional constraints apply only to Spring’s JMS remoting support. See JMS (Java Message Service) for information on Spring’s rich support for JMS-based messaging.

The following interface is used on both the server and the client sides:

package com.foo;

public interface CheckingAccountService {

    public void cancelAccount(Long accountId);
}
The following simple implementation of the preceding interface is used on the server-side:

package com.foo;

public class SimpleCheckingAccountService implements CheckingAccountService {

    public void cancelAccount(Long accountId) {
        System.out.println("Cancelling account [" + accountId + "]");
    }
}
The following configuration file contains the JMS-infrastructure beans that are shared on both the client and the server:

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="connectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://ep-t43:61616"/>
    </bean>
    
    <bean id="queue" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg value="mmm"/>
    </bean>

</beans>
2.7.1. Server-side Configuration
On the server, you need to expose the service object that uses the JmsInvokerServiceExporter, as the following example shows:

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="checkingAccountService"
            class="org.springframework.jms.remoting.JmsInvokerServiceExporter">
        <property name="serviceInterface" value="com.foo.CheckingAccountService"/>
        <property name="service">
            <bean class="com.foo.SimpleCheckingAccountService"/>
        </property>
    </bean>
    
    <bean class="org.springframework.jms.listener.SimpleMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="destination" ref="queue"/>
        <property name="concurrentConsumers" value="3"/>
        <property name="messageListener" ref="checkingAccountService"/>
    </bean>

</beans>
package com.foo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Server {

    public static void main(String[] args) throws Exception {
        new ClassPathXmlApplicationContext("com/foo/server.xml", "com/foo/jms.xml");
    }
}
2.7.2. Client-side Configuration
The client merely needs to create a client-side proxy that implements the agreed-upon interface (CheckingAccountService).

The following example defines beans that you can inject into other client-side objects (and the proxy takes care of forwarding the call to the server-side object via JMS):

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="checkingAccountService"
            class="org.springframework.jms.remoting.JmsInvokerProxyFactoryBean">
        <property name="serviceInterface" value="com.foo.CheckingAccountService"/>
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="queue" ref="queue"/>
    </bean>

</beans>
package com.foo;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Client {

    public static void main(String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("com/foo/client.xml", "com/foo/jms.xml");
        CheckingAccountService service = (CheckingAccountService) ctx.getBean("checkingAccountService");
        service.cancelAccount(new Long(10));
    }
}
3. Enterprise JavaBeans (EJB) Integration
As a lightweight container, Spring is often considered an EJB replacement. We do believe that for many, if not most, applications and use cases, Spring, as a container, combined with its rich supporting functionality in the area of transactions, ORM and JDBC access, is a better choice than implementing equivalent functionality through an EJB container and EJBs.

However, it is important to note that using Spring does not prevent you from using EJBs. In fact, Spring makes it much easier to access EJBs and implement EJBs and functionality within them. Additionally, using Spring to access services provided by EJBs allows the implementation of those services to later transparently be switched between local EJB, remote EJB, or POJO (plain old Java object) variants, without the client code having to be changed.

In this chapter, we look at how Spring can help you access and implement EJBs. Spring provides particular value when accessing stateless session beans (SLSBs), so we begin by discussing this topic.

3.1. Accessing EJBs
This section covers how to access EJBs.

3.1.1. Concepts
To invoke a method on a local or remote stateless session bean, client code must normally perform a JNDI lookup to obtain the (local or remote) EJB Home object and then use a create method call on that object to obtain the actual (local or remote) EJB object. One or more methods are then invoked on the EJB.

To avoid repeated low-level code, many EJB applications use the Service Locator and Business Delegate patterns. These are better than spraying JNDI lookups throughout client code, but their usual implementations have significant disadvantages:

Typically, code that uses EJBs depends on Service Locator or Business Delegate singletons, making it hard to test.

In the case of the Service Locator pattern used without a Business Delegate, application code still ends up having to invoke the create() method on an EJB home and deal with the resulting exceptions. Thus, it remains tied to the EJB API and the complexity of the EJB programming model.

Implementing the Business Delegate pattern typically results in significant code duplication, where we have to write numerous methods that call the same method on the EJB.

The Spring approach is to allow the creation and use of proxy objects (normally configured inside a Spring container), which act as codeless business delegates. You need not write another Service Locator, another JNDI lookup, or duplicate methods in a hand-coded Business Delegate unless you actually add real value in such code.

3.1.2. Accessing Local SLSBs
Assume that we have a web controller that needs to use a local EJB. We follow best practice and use the EJB Business Methods Interface pattern, so that the EJB’s local interface extends a non-EJB-specific business methods interface. We call this business methods interface MyComponent. The following example shows such an interface:

public interface MyComponent {
    ...
}
One of the main reasons to use the Business Methods Interface pattern is to ensure that synchronization between method signatures in local interface and bean implementation class is automatic. Another reason is that it later makes it much easier for us to switch to a POJO (plain old Java object) implementation of the service if it makes sense to do so. We also need to implement the local home interface and provide an implementation class that implements SessionBean and the MyComponent business methods interface. Now, the only Java coding we need to do to hook up our web tier controller to the EJB implementation is to expose a setter method of type MyComponent on the controller. This saves the reference as an instance variable in the controller. The following example shows how to do so:

private MyComponent myComponent;

public void setMyComponent(MyComponent myComponent) {
    this.myComponent = myComponent;
}
We can subsequently use this instance variable in any business method in the controller. Now, assuming we obtain our controller object out of a Spring container, we can (in the same context) configure a LocalStatelessSessionProxyFactoryBean instance, which is the EJB proxy object. We configure the proxy and set the myComponent property of the controller with the following configuration entry:

<bean id="myComponent"
        class="org.springframework.ejb.access.LocalStatelessSessionProxyFactoryBean">
    <property name="jndiName" value="ejb/myBean"/>
    <property name="businessInterface" value="com.mycom.MyComponent"/>
</bean>

<bean id="myController" class="com.mycom.myController">
    <property name="myComponent" ref="myComponent"/>
</bean>
A lot of work happens behind the scenes, courtesy of the Spring AOP framework, although you are not forced to work with AOP concepts to enjoy the results. The myComponent bean definition creates a proxy for the EJB, which implements the business method interface. The EJB local home is cached on startup, so there is only a single JNDI lookup. Each time the EJB is invoked, the proxy invokes the classname method on the local EJB and invokes the corresponding business method on the EJB.

The myController bean definition sets the myComponent property of the controller class to the EJB proxy.

Alternatively (and preferably in case of many such proxy definitions), consider using the <jee:local-slsb> configuration element in Spring’s “jee” namespace. The following example shows how to do so:

<jee:local-slsb id="myComponent" jndi-name="ejb/myBean"
        business-interface="com.mycom.MyComponent"/>

<bean id="myController" class="com.mycom.myController">
    <property name="myComponent" ref="myComponent"/>
</bean>
This EJB access mechanism delivers huge simplification of application code. The web tier code (or other EJB client code) has no dependence on the use of EJB. To replace this EJB reference with a POJO or a mock object or other test stub, we could change the myComponent bean definition without changing a line of Java code. Additionally, we have not had to write a single line of JNDI lookup or other EJB plumbing code as part of our application.

Benchmarks and experience in real applications indicate that the performance overhead of this approach (which involves reflective invocation of the target EJB) is minimal and is undetectable in typical use. Remember that we do not want to make fine-grained calls to EJBs anyway, as there is a cost associated with the EJB infrastructure in the application server.

There is one caveat with regards to the JNDI lookup. In a bean container, this class is normally best used as a singleton (there is no reason to make it a prototype). However, if that bean container pre-instantiates singletons (as do the various XML ApplicationContext variants), you can have a problem if the bean container is loaded before the EJB container loads the target EJB. That is because the JNDI lookup is performed in the init() method of this class and then cached, but the EJB has not been bound at the target location yet. The solution is to not pre-instantiate this factory object but to let it be created on first use. In the XML containers, you can control this by using the lazy-init attribute.

Although not of interest to the majority of Spring users, those doing programmatic AOP work with EJBs may want to look at LocalSlsbInvokerInterceptor.

3.1.3. Accessing Remote SLSBs
Accessing remote EJBs is essentially identical to accessing local EJBs, except that the SimpleRemoteStatelessSessionProxyFactoryBean or <jee:remote-slsb> configuration element is used. Of course, with or without Spring, remote invocation semantics apply: A call to a method on an object in another VM in another computer does sometimes have to be treated differently in terms of usage scenarios and failure handling.

Spring’s EJB client support adds one more advantage over the non-Spring approach. Normally, it is problematic for EJB client code to be easily switched back and forth between calling EJBs locally or remotely. This is because the remote interface methods must declare that they throw RemoteException, and client code must deal with this, while the local interface methods need not. Client code written for local EJBs that needs to be moved to remote EJBs typically has to be modified to add handling for the remote exceptions, and client code written for remote EJBs that needs to be moved to local EJBs can either stay the same but do a lot of unnecessary handling of remote exceptions or be modified to remove that code. With the Spring remote EJB proxy, you can instead not declare any thrown RemoteException in your Business Method Interface and implementing EJB code, have a remote interface that is identical (except that it does throw RemoteException), and rely on the proxy to dynamically treat the two interfaces as if they were the same. That is, client code does not have to deal with the checked RemoteException class. Any actual RemoteException that is thrown during the EJB invocation is re-thrown as the non-checked RemoteAccessException class, which is a subclass of RuntimeException. You can then switch the target service at will between a local EJB or remote EJB (or even plain Java object) implementation, without the client code knowing or caring. Of course, this is optional: Nothing stops you from declaring RemoteException in your business interface.

3.1.4. Accessing EJB 2.x SLSBs Versus EJB 3 SLSBs
Accessing EJB 2.x Session Beans and EJB 3 Session Beans through Spring is largely transparent. Spring’s EJB accessors, including the <jee:local-slsb> and <jee:remote-slsb> facilities, transparently adapt to the actual component at runtime. They handle a home interface if found (EJB 2.x style) or perform straight component invocations if no home interface is available (EJB 3 style).

Note: For EJB 3 Session Beans, you can effectively use a JndiObjectFactoryBean / <jee:jndi-lookup> as well, since fully usable component references are exposed for plain JNDI lookups there. Defining explicit <jee:local-slsb> or <jee:remote-slsb> lookups provides consistent and more explicit EJB access configuration.

4. JMS (Java Message Service)
Spring provides a JMS integration framework that simplifies the use of the JMS API in much the same way as Spring’s integration does for the JDBC API.

JMS can be roughly divided into two areas of functionality, namely the production and consumption of messages. The JmsTemplate class is used for message production and synchronous message reception. For asynchronous reception similar to Java EE’s message-driven bean style, Spring provides a number of message-listener containers that you can use to create Message-Driven POJOs (MDPs). Spring also provides a declarative way to create message listeners.

The org.springframework.jms.core package provides the core functionality for using JMS. It contains JMS template classes that simplify the use of the JMS by handling the creation and release of resources, much like the JdbcTemplate does for JDBC. The design principle common to Spring template classes is to provide helper methods to perform common operations and, for more sophisticated usage, delegate the essence of the processing task to user-implemented callback interfaces. The JMS template follows the same design. The classes offer various convenience methods for sending messages, consuming messages synchronously, and exposing the JMS session and message producer to the user.

The org.springframework.jms.support package provides JMSException translation functionality. The translation converts the checked JMSException hierarchy to a mirrored hierarchy of unchecked exceptions. If any provider-specific subclasses of the checked javax.jms.JMSException exist, this exception is wrapped in the unchecked UncategorizedJmsException.

The org.springframework.jms.support.converter package provides a MessageConverter abstraction to convert between Java objects and JMS messages.

The org.springframework.jms.support.destination package provides various strategies for managing JMS destinations, such as providing a service locator for destinations stored in JNDI.

The org.springframework.jms.annotation package provides the necessary infrastructure to support annotation-driven listener endpoints by using @JmsListener.

The org.springframework.jms.config package provides the parser implementation for the jms namespace as well as the java config support to configure listener containers and create listener endpoints.

Finally, the org.springframework.jms.connection package provides an implementation of the ConnectionFactory suitable for use in standalone applications. It also contains an implementation of Spring’s PlatformTransactionManager for JMS (the cunningly named JmsTransactionManager). This allows for seamless integration of JMS as a transactional resource into Spring’s transaction management mechanisms.

As of Spring Framework 5, Spring’s JMS package fully supports JMS 2.0 and requires the JMS 2.0 API to be present at runtime. We recommend the use of a JMS 2.0 compatible provider.

If you happen to use an older message broker in your system, you may try upgrading to a JMS 2.0 compatible driver for your existing broker generation. Alternatively, you may also try to run against a JMS 1.1 based driver, simply putting the JMS 2.0 API jar on the classpath but only using JMS 1.1 compatible API against your driver. Spring’s JMS support adheres to JMS 1.1 conventions by default, so with corresponding configuration it does support such a scenario. However, please consider this for transition scenarios only.

4.1. Using Spring JMS
This section describes how to use Spring’s JMS components.

4.1.1. Using JmsTemplate
The JmsTemplate class is the central class in the JMS core package. It simplifies the use of JMS, since it handles the creation and release of resources when sending or synchronously receiving messages.

Code that uses the JmsTemplate needs only to implement callback interfaces that give them a clearly defined high-level contract. The MessageCreator callback interface creates a message when given a Session provided by the calling code in JmsTemplate. To allow for more complex usage of the JMS API, SessionCallback provides the JMS session, and ProducerCallback exposes a Session and MessageProducer pair.

The JMS API exposes two types of send methods, one that takes delivery mode, priority, and time-to-live as Quality of Service (QOS) parameters and one that takes no QOS parameters and uses default values. Since JmsTemplate has many send methods, setting the QOS parameters have been exposed as bean properties to avoid duplication in the number of send methods. Similarly, the timeout value for synchronous receive calls is set by using the setReceiveTimeout property.

Some JMS providers allow the setting of default QOS values administratively through the configuration of the ConnectionFactory. This has the effect that a call to a MessageProducer instance’s send method (send(Destination destination, Message message)) uses different QOS default values than those specified in the JMS specification. In order to provide consistent management of QOS values, the JmsTemplate must, therefore, be specifically enabled to use its own QOS values by setting the boolean property isExplicitQosEnabled to true.

For convenience, JmsTemplate also exposes a basic request-reply operation that allows for sending a message and waiting for a reply on a temporary queue that is created as part of the operation.

Instances of the JmsTemplate class are thread-safe, once configured. This is important, because it means that you can configure a single instance of a JmsTemplate and then safely inject this shared reference into multiple collaborators. To be clear, the JmsTemplate is stateful, in that it maintains a reference to a ConnectionFactory, but this state is not conversational state.
As of Spring Framework 4.1, JmsMessagingTemplate is built on top of JmsTemplate and provides an integration with the messaging abstraction — that is, org.springframework.messaging.Message. This lets you create the message to send in a generic manner.

4.1.2. Connections
The JmsTemplate requires a reference to a ConnectionFactory. The ConnectionFactory is part of the JMS specification and serves as the entry point for working with JMS. It is used by the client application as a factory to create connections with the JMS provider and encapsulates various configuration parameters, many of which are vendor-specific, such as SSL configuration options.

When using JMS inside an EJB, the vendor provides implementations of the JMS interfaces so that they can participate in declarative transaction management and perform pooling of connections and sessions. In order to use this implementation, Java EE containers typically require that you declare a JMS connection factory as a resource-ref inside the EJB or servlet deployment descriptors. To ensure the use of these features with the JmsTemplate inside an EJB, the client application should ensure that it references the managed implementation of the ConnectionFactory.

Caching Messaging Resources
The standard API involves creating many intermediate objects. To send a message, the following 'API' walk is performed:

ConnectionFactory->Connection->Session->MessageProducer->send
Between the ConnectionFactory and the Send operation, three intermediate objects are created and destroyed. To optimize the resource usage and increase performance, Spring provides two implementations of ConnectionFactory.

Using SingleConnectionFactory
Spring provides an implementation of the ConnectionFactory interface, SingleConnectionFactory, that returns the same Connection on all createConnection() calls and ignores calls to close(). This is useful for testing and standalone environments so that the same connection can be used for multiple JmsTemplate calls that may span any number of transactions. SingleConnectionFactory takes a reference to a standard ConnectionFactory that would typically come from JNDI.

Using CachingConnectionFactory
The CachingConnectionFactory extends the functionality of SingleConnectionFactory and adds the caching of Session, MessageProducer, and MessageConsumer instances. The initial cache size is set to 1. You can use the sessionCacheSize property to increase the number of cached sessions. Note that the number of actual cached sessions is more than that number, as sessions are cached based on their acknowledgment mode, so there can be up to four cached session instances (one for each acknowledgment mode) when sessionCacheSize is set to one . MessageProducer and MessageConsumer instances are cached within their owning session and also take into account the unique properties of the producers and consumers when caching. MessageProducers are cached based on their destination. MessageConsumers are cached based on a key composed of the destination, selector, noLocal delivery flag, and the durable subscription name (if creating durable consumers).

4.1.3. Destination Management
Destinations, as ConnectionFactory instances, are JMS administered objects that you can store and retrieved in JNDI. When configuring a Spring application context, you can use the JNDI JndiObjectFactoryBean factory class or <jee:jndi-lookup> to perform dependency injection on your object’s references to JMS destinations. However, this strategy is often cumbersome if there are a large number of destinations in the application or if there are advanced destination management features unique to the JMS provider. Examples of such advanced destination management include the creation of dynamic destinations or support for a hierarchical namespace of destinations. The JmsTemplate delegates the resolution of a destination name to a JMS destination object that implements the DestinationResolver interface. DynamicDestinationResolver is the default implementation used by JmsTemplate and accommodates resolving dynamic destinations. A JndiDestinationResolver is also provided to act as a service locator for destinations contained in JNDI and optionally falls back to the behavior contained in DynamicDestinationResolver.

Quite often, the destinations used in a JMS application are only known at runtime and, therefore, cannot be administratively created when the application is deployed. This is often because there is shared application logic between interacting system components that create destinations at runtime according to a well-known naming convention. Even though the creation of dynamic destinations is not part of the JMS specification, most vendors have provided this functionality. Dynamic destinations are created with a user-defined name, which differentiates them from temporary destinations, and are often not registered in JNDI. The API used to create dynamic destinations varies from provider to provider since the properties associated with the destination are vendor-specific. However, a simple implementation choice that is sometimes made by vendors is to disregard the warnings in the JMS specification and to use the method TopicSession createTopic(String topicName) or the QueueSession createQueue(String queueName) method to create a new destination with default destination properties. Depending on the vendor implementation, DynamicDestinationResolver can then also create a physical destination instead of only resolving one.

The boolean property pubSubDomain is used to configure the JmsTemplate with knowledge of what JMS domain is being used. By default, the value of this property is false, indicating that the point-to-point domain, Queues, is to be used. This property (used by JmsTemplate) determines the behavior of dynamic destination resolution through implementations of the DestinationResolver interface.

You can also configure the JmsTemplate with a default destination through the property defaultDestination. The default destination is with send and receive operations that do not refer to a specific destination.

4.1.4. Message Listener Containers
One of the most common uses of JMS messages in the EJB world is to drive message-driven beans (MDBs). Spring offers a solution to create message-driven POJOs (MDPs) in a way that does not tie a user to an EJB container. (See Asynchronous reception: Message-Driven POJOs for detailed coverage of Spring’s MDP support.) Since Spring Framework 4.1, endpoint methods can be annotated with @JmsListener — see Annotation-driven Listener Endpoints for more details.

A message listener container is used to receive messages from a JMS message queue and drive the MessageListener that is injected into it. The listener container is responsible for all threading of message reception and dispatches into the listener for processing. A message listener container is the intermediary between an MDP and a messaging provider and takes care of registering to receive messages, participating in transactions, resource acquisition and release, exception conversion, and so on. This lets you write the (possibly complex) business logic associated with receiving a message (and possibly respond to it), and delegates boilerplate JMS infrastructure concerns to the framework.

There are two standard JMS message listener containers packaged with Spring, each with its specialized feature set.

SimpleMessageListenerContainer

DefaultMessageListenerContainer

Using SimpleMessageListenerContainer
This message listener container is the simpler of the two standard flavors. It creates a fixed number of JMS sessions and consumers at startup, registers the listener by using the standard JMS MessageConsumer.setMessageListener() method, and leaves it up the JMS provider to perform listener callbacks. This variant does not allow for dynamic adaption to runtime demands or for participation in externally managed transactions. Compatibility-wise, it stays very close to the spirit of the standalone JMS specification, but is generally not compatible with Java EE’s JMS restrictions.

While SimpleMessageListenerContainer does not allow for participation in externally managed transactions, it does support native JMS transactions. To enable this feature, you can switch the sessionTransacted flag to true or, in the XML namespace, set the acknowledge attribute to transacted. Exceptions thrown from your listener then lead to a rollback, with the message getting redelivered. Alternatively, consider using CLIENT_ACKNOWLEDGE mode, which provides redelivery in case of an exception as well but does not use transacted Session instances and, therefore, does not include any other Session operations (such as sending response messages) in the transaction protocol.
The default AUTO_ACKNOWLEDGE mode does not provide proper reliability guarantees. Messages can get lost when listener execution fails (since the provider automatically acknowledges each message after listener invocation, with no exceptions to be propagated to the provider) or when the listener container shuts down (you can configure this by setting the acceptMessagesWhileStopping flag). Make sure to use transacted sessions in case of reliability needs (for example, for reliable queue handling and durable topic subscriptions).
Using DefaultMessageListenerContainer
This message listener container is used in most cases. In contrast to SimpleMessageListenerContainer, this container variant allows for dynamic adaptation to runtime demands and is able to participate in externally managed transactions. Each received message is registered with an XA transaction when configured with a JtaTransactionManager. As a result, processing may take advantage of XA transaction semantics. This listener container strikes a good balance between low requirements on the JMS provider, advanced functionality (such as participation in externally managed transactions), and compatibility with Java EE environments.

You can customize the cache level of the container. Note that, when no caching is enabled, a new connection and a new session is created for each message reception. Combining this with a non-durable subscription with high loads may lead to message loss. Make sure to use a proper cache level in such a case.

This container also has recoverable capabilities when the broker goes down. By default, a simple BackOff implementation retries every five seconds. You can specify a custom BackOff implementation for more fine-grained recovery options. See api-spring-framework/util/backoff/ExponentialBackOff.html[ExponentialBackOff] for an example.

Like its sibling (SimpleMessageListenerContainer), DefaultMessageListenerContainer supports native JMS transactions and allows for customizing the acknowledgment mode. If feasible for your scenario, This is strongly recommended over externally managed transactions — that is, if you can live with occasional duplicate messages in case of the JVM dying. Custom duplicate message detection steps in your business logic can cover such situations — for example, in the form of a business entity existence check or a protocol table check. Any such arrangements are significantly more efficient than the alternative: wrapping your entire processing with an XA transaction (through configuring your DefaultMessageListenerContainer with an JtaTransactionManager) to cover the reception of the JMS message as well as the execution of the business logic in your message listener (including database operations etc).
The default AUTO_ACKNOWLEDGE mode does not provide proper reliability guarantees. Messages can get lost when listener execution fails (since the provider automatically acknowledges each message after listener invocation, with no exceptions to be propagated to the provider) or when the listener container shuts down (you can configure this by setting the acceptMessagesWhileStopping flag). Make sure to use transacted sessions in case of reliability needs (for example, for reliable queue handling and durable topic subscriptions).
4.1.5. Transaction Management
Spring provides a JmsTransactionManager that manages transactions for a single JMS ConnectionFactory. This lets JMS applications leverage the managed-transaction features of Spring, as described in Transaction Management section of the Data Access chapter. The JmsTransactionManager performs local resource transactions, binding a JMS Connection/Session pair from the specified ConnectionFactory to the thread. JmsTemplate automatically detects such transactional resources and operates on them accordingly.

In a Java EE environment, the ConnectionFactory pools Connection and Session instances, so those resources are efficiently reused across transactions. In a standalone environment, using Spring’s SingleConnectionFactory result in a shared JMS Connection, with each transaction having its own independent Session. Alternatively, consider the use of a provider-specific pooling adapter, such as ActiveMQ’s PooledConnectionFactory class.

You can also use JmsTemplate with the JtaTransactionManager and an XA-capable JMS ConnectionFactory to perform distributed transactions. Note that this requires the use of a JTA transaction manager as well as a properly XA-configured ConnectionFactory. (Check your Java EE server’s or JMS provider’s documentation.)

Reusing code across a managed and unmanaged transactional environment can be confusing when using the JMS API to create a Session from a Connection. This is because the JMS API has only one factory method to create a Session, and it requires values for the transaction and acknowledgment modes. In a managed environment, setting these values is the responsibility of the environment’s transactional infrastructure, so these values are ignored by the vendor’s wrapper to the JMS Connection. When you use the JmsTemplate in an unmanaged environment, you can specify these values through the use of the properties sessionTransacted and sessionAcknowledgeMode. When you use a PlatformTransactionManager with JmsTemplate, the template is always given a transactional JMS Session.

4.2. Sending a Message
The JmsTemplate contains many convenience methods to send a message. Send methods specify the destination by using a javax.jms.Destination object, and others specify the destination by using a String in a JNDI lookup. The send method that takes no destination argument uses the default destination.

The following example uses the MessageCreator callback to create a text message from the supplied Session object:

import javax.jms.ConnectionFactory;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Queue;
import javax.jms.Session;

import org.springframework.jms.core.MessageCreator;
import org.springframework.jms.core.JmsTemplate;

public class JmsQueueSender {

    private JmsTemplate jmsTemplate;
    private Queue queue;
    
    public void setConnectionFactory(ConnectionFactory cf) {
        this.jmsTemplate = new JmsTemplate(cf);
    }
    
    public void setQueue(Queue queue) {
        this.queue = queue;
    }
    
    public void simpleSend() {
        this.jmsTemplate.send(this.queue, new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                return session.createTextMessage("hello queue world");
            }
        });
    }
}
In the preceding example, the JmsTemplate is constructed by passing a reference to a ConnectionFactory. As an alternative, a zero-argument constructor and connectionFactory is provided and can be used for constructing the instance in JavaBean style (using a BeanFactory or plain Java code). Alternatively, consider deriving from Spring’s JmsGatewaySupport convenience base class, which provides pre-built bean properties for JMS configuration.

The send(String destinationName, MessageCreator creator) method lets you send a message by using the string name of the destination. If these names are registered in JNDI, you should set the destinationResolver property of the template to an instance of JndiDestinationResolver.

If you created the JmsTemplate and specified a default destination, the send(MessageCreator c) sends a message to that destination.

4.2.1. Using Message Converters
To facilitate the sending of domain model objects, the JmsTemplate has various send methods that take a Java object as an argument for a message’s data content. The overloaded methods convertAndSend() and receiveAndConvert() methods in JmsTemplate delegate the conversion process to an instance of the MessageConverter interface. This interface defines a simple contract to convert between Java objects and JMS messages. The default implementation (SimpleMessageConverter) supports conversion between String and TextMessage, byte[] and BytesMesssage, and java.util.Map and MapMessage. By using the converter, you and your application code can focus on the business object that is being sent or received through JMS and not be concerned with the details of how it is represented as a JMS message.

沙盒当前包含一个MapMessageConverter，使用反射在JavaBean和一个之间进行转换MapMessage。您可能自己实现的其他流行实现选择是使用现有XML编组程序包（例如JAXB或XStream）创建TextMessage表示对象的的转换器。

为了适应消息属性，标头和正文的设置，这些设置通常不能封装在转换器类中，因此，MessagePostProcessor接口可以在消息转换后但发送之前访问消息。以下示例显示将ajava.util.Map转换为消息后如何修改消息头和属性 ：

public void sendWithConversion() {
    Map map = new HashMap();
    map.put("Name", "Mark");
    map.put("Age", new Integer(47));
    jmsTemplate.convertAndSend("testQueue", map, new MessagePostProcessor() {
        public Message postProcessMessage(Message message) throws JMSException {
            message.setIntProperty("AccountID", 1234);
            message.setJMSCorrelationID("123-00001");
            return message;
        }
    });
}
这将导致以下形式的消息：

MapMessage = {
    标头= {
        ...标头...
        CorrelationID = {123-00001}
    }
    属性= {
        AccountID = {整数：1234}
    }
    字段= {
        名称= {String：Mark}
        年龄= {整数：47}
    }
}
4.2.2。使用SessionCallback和ProducerCallback
尽管发送操作涵盖了许多常见的使用场景，但是您有时可能希望在JMSSession或上执行多个操作MessageProducer。的 SessionCallback和ProducerCallback暴露的JMSSession和Session/ MessageProducer分别对，。这些execute()方法将JmsTemplate运行这些回调方法。

4.3。接收讯息
这描述了如何在Spring中使用JMS接收消息。

4.3.1。同步接收
虽然JMS通常与异步处理相关联，但是您可以同步使用消息。重载的receive(..)方法提供了此功能。在同步接收期间，调用线程将阻塞，直到消息可用为止。这可能是危险的操作，因为调用线程可能会无限期地被阻塞。该receiveTimeout属性指定接收者在放弃等待消息之前应该等待多长时间。

4.3.2。异步接收：消息驱动的POJO
Spring还通过使用@JmsListener 注释支持带注释的侦听器端点，并提供了开放的基础结构以编程方式注册端点。到目前为止，这是设置异步接收器的最便捷方法。有关更多详细信息，请参见启用侦听器端点注释。
消息驱动POJO（MDP）以类似于EJB世界中的消息驱动Bean（MDB）的方式充当JMS消息的接收者。MDP上的一个限制（但请参见 使用MessageListenerAdapter）是它必须实现javax.jms.MessageListener接口。注意，如果您的POJO在多个线程上接收消息，那么确保您的实现是线程安全的，这一点很重要。

以下示例显示了MDP的简单实现：

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

public class ExampleListener implements MessageListener {

    public void onMessage(Message message) {
        if (message instanceof TextMessage) {
            try {
                System.out.println(((TextMessage) message).getText());
            }
            catch (JMSException ex) {
                throw new RuntimeException(ex);
            }
        }
        else {
            throw new IllegalArgumentException("Message must be of type TextMessage");
        }
    }
}
一旦实现了MessageListener，就可以创建一个消息侦听器容器了。

以下示例显示了如何定义和配置Spring附带的消息侦听器容器（在本例中为DefaultMessageListenerContainer）：

<!-- this is the Message Driven POJO (MDP) -->
<bean id="messageListener" class="jmsexample.ExampleListener"/>

<!-- and this is the message listener container -->
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
</bean>
See the Spring javadoc of the various message listener containers (all of which implement MessageListenerContainer) for a full description of the features supported by each implementation.

4.3.3. Using the SessionAwareMessageListener Interface
The SessionAwareMessageListener interface is a Spring-specific interface that provides a similar contract to the JMS MessageListener interface but also gives the message-handling method access to the JMS Session from which the Message was received. The following listing shows the definition of the SessionAwareMessageListener interface:

package org.springframework.jms.listener;

public interface SessionAwareMessageListener {

    void onMessage(Message message, Session session) throws JMSException;
}
You can choose to have your MDPs implement this interface (in preference to the standard JMS MessageListener interface) if you want your MDPs to be able to respond to any received messages (by using the Session supplied in the onMessage(Message, Session) method). All of the message listener container implementations that ship with Spring have support for MDPs that implement either the MessageListener or SessionAwareMessageListener interface. Classes that implement the SessionAwareMessageListener come with the caveat that they are then tied to Spring through the interface. The choice of whether or not to use it is left entirely up to you as an application developer or architect.

Note that the onMessage(..) method of the SessionAwareMessageListener interface throws JMSException. In contrast to the standard JMS MessageListener interface, when using the SessionAwareMessageListener interface, it is the responsibility of the client code to handle any thrown exceptions.

4.3.4. Using MessageListenerAdapter
The MessageListenerAdapter class is the final component in Spring’s asynchronous messaging support. In a nutshell, it lets you expose almost any class as an MDP (though there are some constraints).

Consider the following interface definition:

public interface MessageDelegate {

    void handleMessage(String message);
    
    void handleMessage(Map message);
    
    void handleMessage(byte[] message);
    
    void handleMessage(Serializable message);
}
Notice that, although the interface extends neither the MessageListener nor the SessionAwareMessageListener interface, you can still use it as an MDP by using the MessageListenerAdapter class. Notice also how the various message handling methods are strongly typed according to the contents of the various Message types that they can receive and handle.

现在考虑MessageDelegate接口的以下实现：

public class DefaultMessageDelegate implements MessageDelegate {
    // implementation elided for clarity...
}
特别要注意，MessageDelegate接口（ DefaultMessageDelegate该类）的先前实现是如何完全没有JMS依赖性的。这确实是一个POJO，我们可以通过以下配置将其制作为MDP：

<!-- this is the Message Driven POJO (MDP) -->
<bean id="messageListener" class="org.springframework.jms.listener.adapter.MessageListenerAdapter">
    <constructor-arg>
        <bean class="jmsexample.DefaultMessageDelegate"/>
    </constructor-arg>
</bean>

<!-- and this is the message listener container... -->
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
</bean>
下一个示例显示另一个MDP，它只能处理接收到的JMS TextMessage消息。请注意，实际上是如何调用消息处理方法的receive（消息处理方法 的名称MessageListenerAdapter 默认为handleMessage），但是它是可配置的（如本节后面所述）。还请注意，如何对receive(..)方法进行强类型设置以仅接收和响应JMS TextMessage消息。以下清单显示了TextMessageDelegate接口的定义：

public interface TextMessageDelegate {

    void receive(TextMessage message);
}
以下清单显示了实现该TextMessageDelegate接口的类：

public class DefaultTextMessageDelegate implements TextMessageDelegate {
    // implementation elided for clarity...
}
话务员的配置MessageListenerAdapter如下：

<bean id="messageListener" class="org.springframework.jms.listener.adapter.MessageListenerAdapter">
    <constructor-arg>
        <bean class="jmsexample.DefaultTextMessageDelegate"/>
    </constructor-arg>
    <property name="defaultListenerMethod" value="receive"/>
    <!-- we don't want automatic message context extraction -->
    <property name="messageConverter">
        <null/>
    </property>
</bean>
需要注意的是，如果messageListener接收到一个JMSMessage以外的类型的TextMessage，一个IllegalStateException被抛出（随后吞咽）。MessageListenerAdapter该类的另一个功能是，Message如果处理程序方法返回非空值，则能够自动发送回响应。考虑以下接口和类：

public interface ResponsiveTextMessageDelegate {

    // notice the return type...
    String receive(TextMessage message);
}
public class DefaultResponsiveTextMessageDelegate implements ResponsiveTextMessageDelegate {
    // implementation elided for clarity...
}
If you use the DefaultResponsiveTextMessageDelegate in conjunction with a MessageListenerAdapter, any non-null value that is returned from the execution of the 'receive(..)' method is (in the default configuration) converted into a TextMessage. The resulting TextMessage is then sent to the Destination (if one exists) defined in the JMS Reply-To property of the original Message or the default Destination set on the MessageListenerAdapter (if one has been configured). If no Destination is found, an InvalidDestinationException is thrown (note that this exception is not swallowed and propagates up the call stack).

4.3.5. Processing Messages Within Transactions
Invoking a message listener within a transaction requires only reconfiguration of the listener container.

You can activate local resource transactions through the sessionTransacted flag on the listener container definition. Each message listener invocation then operates within an active JMS transaction, with message reception rolled back in case of listener execution failure. Sending a response message (through SessionAwareMessageListener) is part of the same local transaction, but any other resource operations (such as database access) operate independently. This usually requires duplicate message detection in the listener implementation, to cover the case where database processing has committed but message processing failed to commit.

Consider the following bean definition:

<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
    <property name="sessionTransacted" value="true"/>
</bean>
要参与外部管理的事务，您需要配置一个事务管理器并使用支持外部管理的事务的侦听器容器（通常为DefaultMessageListenerContainer）。

要为XA事务参与配置消息侦听器容器，您需要配置一个JtaTransactionManager（默认情况下，它委托给Java EE服务器的事务子系统）。请注意，底层的JMSConnectionFactory需要具有XA功能，并已向您的JTA事务协调器正确注册。（检查Java EE服务器的JNDI资源配置。）这使消息接收以及（例如）数据库访问成为同一事务的一部分（具有统一的提交语义，但以XA事务日志开销为代价）。

以下bean定义创建一个事务管理器：

<bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
然后，我们需要将其添加到我们之前的容器配置中。容器负责其余的工作。以下示例显示了如何执行此操作：

<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
    <property name="transactionManager" ref="transactionManager"/> 
</bean>
我们的交易经理。
4.4。支持JCA消息端点
从2.5版开始，Spring还提供了对基于JCA的 MessageListener容器的支持。在JmsMessageEndpointManager尝试自动确定ActivationSpec从提供的类名 ResourceAdapter类名。因此，通常可以提供Spring的generic JmsActivationSpecConfig，如以下示例所示：

<bean class="org.springframework.jms.listener.endpoint.JmsMessageEndpointManager">
    <property name="resourceAdapter" ref="resourceAdapter"/>
    <property name="activationSpecConfig">
        <bean class="org.springframework.jms.listener.endpoint.JmsActivationSpecConfig">
            <property name="destinationName" value="myQueue"/>
        </bean>
    </property>
    <property name="messageListener" ref="myMessageListener"/>
</bean>
或者，您可以JmsMessageEndpointManager使用给定的 ActivationSpec对象设置一个。该ActivationSpec对象也可能来自JNDI查找（使用<jee:jndi-lookup>）。以下示例显示了如何执行此操作：

<bean class="org.springframework.jms.listener.endpoint.JmsMessageEndpointManager">
    <property name="resourceAdapter" ref="resourceAdapter"/>
    <property name="activationSpec">
        <bean class="org.apache.activemq.ra.ActiveMQActivationSpec">
            <property name="destination" value="myQueue"/>
            <property name="destinationType" value="javax.jms.Queue"/>
        </bean>
    </property>
    <property name="messageListener" ref="myMessageListener"/>
</bean>
使用Spring的ResourceAdapterFactoryBean，您可以在ResourceAdapter 本地配置目标，如以下示例所示：

<bean id="resourceAdapter" class="org.springframework.jca.support.ResourceAdapterFactoryBean">
    <property name="resourceAdapter">
        <bean class="org.apache.activemq.ra.ActiveMQResourceAdapter">
            <property name="serverUrl" value="tcp://localhost:61616"/>
        </bean>
    </property>
    <property name="workManager">
        <bean class="org.springframework.jca.work.SimpleTaskWorkManager"/>
    </property>
</bean>
指定的对象WorkManager也可以指向特定于环境的线程池-通常通过SimpleTaskWorkManager实例的asyncTaskExecutor属性。ResourceAdapter如果碰巧使用多个适配器，请考虑为所有实例定义一个共享线程池。

在某些环境（例如WebLogic 9或更高版本）中，您可以改为ResourceAdapter使用JNDI获取整个对象<jee:jndi-lookup>。然后，基于Spring的消息侦听器可以与服务器托管的服务器交互，该服务器ResourceAdapter也使用服务器的内置服务器WorkManager。

有关，和 的更多信息JmsMessageEndpointManager， 请参见javadoc 。JmsActivationSpecConfigResourceAdapterFactoryBean

Spring还提供了一个不受JMS约束的通用JCA消息端点管理器 org.springframework.jca.endpoint.GenericMessageEndpointManager。该组件允许使用任何消息侦听器类型（例如JMS MessageListener）和任何提供程序特定的ActivationSpec对象。请参阅JCA提供程序的文档以了解连接器的实际功能，并请参阅 GenericMessageEndpointManager javadoc以获取特定于Spring的配置详细信息。

基于JCA的消息端点管理与EJB 2.1消息驱动Bean非常相似。它使用相同的基础资源提供者合同。与EJB 2.1 MDB一样，您也可以在Spring上下文中使用JCA提供程序支持的任何消息侦听器接口。尽管如此，Spring为JMS提供了显式的“便利”支持，因为JMS是JCA端点管理协定中最常用的端点API。
4.5。注释驱动的侦听器端点
异步接收消息的最简单方法是使用带注释的侦听器端点基础结构。简而言之，它使您可以将托管Bean的方法公开为JMS侦听器端点。以下示例显示了如何使用它：

@Component
public class MyService {

    @JmsListener(destination = "myDestination")
    public void processOrder(String data) { ... }
}
前面示例的思想是，只要消息可用，就会 相应地调用javax.jms.Destination myDestination该processOrder方法（在这种情况下，使用JMS消息的内容，类似于MessageListenerAdapter 提供的内容）。

带注释的端点基础结构通过使用，为每种带注释的方法在幕后创建一个消息侦听器容器JmsListenerContainerFactory。这样的容器不会针对应用程序上下文进行注册，但是可以通过使用JmsListenerEndpointRegistryBean方便地定位用于管理目的。

@JmsListener是Java 8上的可重复注释，因此您可以通过@JmsListener 向其添加其他声明来将多个JMS目标与同一方法相关联。
4.5.1。启用侦听器端点注释
要启用对@JmsListener注释的支持，可以将其添加@EnableJms到一个@Configuration类中，如以下示例所示：

@Configuration
@EnableJms
public class AppConfig {

    @Bean
    public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
        DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory());
        factory.setDestinationResolver(destinationResolver());
        factory.setSessionTransacted(true);
        factory.setConcurrency("3-10");
        return factory;
    }
}
默认情况下，基础结构会寻找一个名为BeanjmsListenerContainerFactory 的工厂供其用来创建消息侦听器容器的源。在这种情况下（并忽略JMS基础结构设置），您可以processOrder 使用三个线程的核心轮询大小和十个线程的最大池大小来调用该方法。

您可以自定义用于每个注释的侦听器容器工厂，也可以通过实现JmsListenerConfigurer接口来配置显式默认值。仅当至少一个端点在没有特定容器工厂的情况下注册时，才需要使用默认值。有关JmsListenerConfigurer 详细信息和示例，请参见实现的类的javadoc 。

如果您更喜欢XML配置，那么可以使用<jms:annotation-driven> 元素，如以下示例所示：

<jms:annotation-driven/>

<bean id="jmsListenerContainerFactory"
        class="org.springframework.jms.config.DefaultJmsListenerContainerFactory">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destinationResolver" ref="destinationResolver"/>
    <property name="sessionTransacted" value="true"/>
    <property name="concurrency" value="3-10"/>
</bean>
4.5.2。程序化端点注册
JmsListenerEndpoint提供JMS端点的模型，并负责为该模型配置容器。除了JmsListener注释所检测的端点之外，基础结构还允许您以编程方式配置端点。以下示例显示了如何执行此操作：

@Configuration
@EnableJms
public class AppConfig implements JmsListenerConfigurer {

    @Override
    public void configureJmsListeners(JmsListenerEndpointRegistrar registrar) {
        SimpleJmsListenerEndpoint endpoint = new SimpleJmsListenerEndpoint();
        endpoint.setId("myJmsEndpoint");
        endpoint.setDestination("anotherQueue");
        endpoint.setMessageListener(message -> {
            // processing
        });
        registrar.registerEndpoint(endpoint);
    }
}
在前面的示例中，我们使用SimpleJmsListenerEndpoint了提供实际 MessageListener调用的。但是，您也可以构建自己的端点变体来描述自定义调用机制。

请注意，您可以@JmsListener完全跳过使用，而可以通过编程方式仅注册端点JmsListenerConfigurer。

4.5.3。带注释的端点方法签名
到目前为止，我们已经String在端点中注入了一个简单的方法，但实际上它可以具有非常灵活的方法签名。在以下示例中，我们将其重写为Order使用自定义标头注入：

@Component
public class MyService {

    @JmsListener(destination = "myDestination")
    public void processOrder(Order order, @Header("order_type") String orderType) {
        ...
    }
}
您可以在JMS侦听器端点中注入的主要元素如下：

原始javax.jms.Message或其任何子类（前提是它与传入的消息类型匹配）。

该javax.jms.Session可选访问本地JMS API（例如，用于发送一个自定义的回复）。

的org.springframework.messaging.Message，它表示传入的JMS消息。请注意，此消息同时包含自定义标头和标准标头（由定义JmsHeaders）。

@Header-带注释的方法参数，以提取特定的标头值，包括标准的JMS标头。

一个带@Headers注释的参数，也必须可分配给该参数，以java.util.Map访问所有标头。

不是支持的类型（Message或 Session）之一的非注释元素被视为有效负载。您可以通过使用注释参数来使其明确@Payload。您还可以通过添加额外的来启用验证 @Valid。

注入Spring的Message抽象功能特别有用，它可以受益于存储在特定于传输的消息中的所有信息，而无需依赖于特定于传​​输的API。以下示例显示了如何执行此操作：

@JmsListener(destination = "myDestination")
public void processOrder(Message<Order> order) { ... }
方法参数的处理由提供DefaultMessageHandlerMethodFactory，您可以进一步自定义以支持其他方法参数。您也可以在那里自定义转换和验证支持。

例如，如果要Order在处理之前确保我们的有效，则可以使用注释有效负载@Valid并配置必要的验证器，如以下示例所示：

@Configuration
@EnableJms
public class AppConfig implements JmsListenerConfigurer {

    @Override
    public void configureJmsListeners(JmsListenerEndpointRegistrar registrar) {
        registrar.setMessageHandlerMethodFactory(myJmsHandlerMethodFactory());
    }
    
    @Bean
    public DefaultMessageHandlerMethodFactory myHandlerMethodFactory() {
        DefaultMessageHandlerMethodFactory factory = new DefaultMessageHandlerMethodFactory();
        factory.setValidator(myValidator());
        return factory;
    }
}
4.5.4。反应管理
现有的支持MessageListenerAdapter 已经使您的方法具有非void返回类型。在这种情况下，调用的结果将封装在中javax.jms.Message，发送JMSReplyTo到原始消息的标头中指定的目标位置或侦听器上配置的默认目标位置。现在，您可以使用@SendTo消息传递抽象的注释来设置默认目标。

假设我们的processOrder方法现在应该返回OrderStatus，我们可以将其编写为自动发送响应，如以下示例所示：

@JmsListener(destination = "myDestination")
@SendTo("status")
public OrderStatus processOrder(Order order) {
    // order processing
    return status;
}
如果您有多个带有 注释的@JmsListener方法，则还可以将@SendTo注释放在类级别以共享默认的答复目标。
如果需要以与传输无关的方式设置其他标头，则可以Message使用类似于以下方法的方法返回a ：

@JmsListener(destination = "myDestination")
@SendTo("status")
public Message<OrderStatus> processOrder(Order order) {
    // order processing
    return MessageBuilder
            .withPayload(status)
            .setHeader("code", 1234)
            .build();
}
如果需要在运行时计算响应目标，则可以将响应封装在一个JmsResponse实例中，该实例还提供要在运行时使用的目标。我们可以如下重写前一个示例：

@JmsListener(destination = "myDestination")
public JmsResponse<Message<OrderStatus>> processOrder(Order order) {
    // order processing
    Message<OrderStatus> response = MessageBuilder
            .withPayload(status)
            .setHeader("code", 1234)
            .build();
    return JmsResponse.forQueue(response, "status");
}
最后，如果需要为响应指定一些QoS值，例如优先级或生存时间，则可以进行相应的配置JmsListenerContainerFactory，如以下示例所示：

@Configuration
@EnableJms
public class AppConfig {

    @Bean
    public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
        DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory());
        QosSettings replyQosSettings = new QosSettings();
        replyQosSettings.setPriority(2);
        replyQosSettings.setTimeToLive(10000);
        factory.setReplyQosSettings(replyQosSettings);
        return factory;
    }
}
4.6。JMS命名空间支持
Spring提供了一个XML名称空间来简化JMS配置。要使用JMS名称空间元素，您需要引用JMS模式，如以下示例所示：

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jms="http://www.springframework.org/schema/jms" 
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/jms https://www.springframework.org/schema/jms/spring-jms.xsd">

    <!-- bean definitions here -->

</beans>
引用JMS模式。
这个命名空间由三个顶级元素：<annotation-driven/>，<listener-container/> 和<jca-listener-container/>。<annotation-driven/>启用注释驱动的侦听器终结点的使用。<listener-container/>并<jca-listener-container/> 定义共享的侦听器容器配置，并且可以包含<listener/>子元素。以下示例显示了两个侦听器的基本配置：

<jms:listener-container>

    <jms:listener destination="queue.orders" ref="orderService" method="placeOrder"/>
    
    <jms:listener destination="queue.confirmations" ref="confirmationLogger" method="log"/>

</jms:listener-container>
前面的示例等效于创建两个不同的侦听器容器bean定义和两个不同的MessageListenerAdapterbean定义，如Using中MessageListenerAdapter所示。除了前面示例中显示的属性之外，该listener元素还可以包含几个可选属性。下表描述了所有可用属性：

表3. JMS <listener>元素的属性
属性	描述
id

托管侦听器容器的Bean名称。如果未指定，将自动生成Bean名称。

destination （需要）

此侦听器的目标名称，通过DestinationResolver 策略解析。

ref （需要）

处理程序对象的bean名称。

method

要调用的处理程序方法的名称。如果ref属性指向MessageListener 或Spring SessionAwareMessageListener，则可以忽略此属性。

response-destination

向其发送响应消息的默认响应目标的名称。在请求消息不包含JMSReplyTo字段的情况下适用。此目标的类型由listener-container的 response-destination-type属性确定。请注意，这仅适用于具有返回值的侦听器方法，为此，每个结果对象都将转换为响应消息。

subscription

持久订阅的名称（如果有）。

selector

此侦听器的可选消息选择器。

concurrency

要启动此侦听器的并发会话或使用者的数量。此值可以是表示最大数的简单数字（例如5），也可以是表示下限和上限的范围（例如3-5）。请注意，指定的最小值仅是一个提示，在运行时可能会被忽略。默认值为容器提供的值。

该<listener-container/>元素还接受几个可选属性。这允许自定义各种策略（例如taskExecutor和 destinationResolver）以及基本的JMS设置和资源引用。通过使用这些属性，您可以定义高度自定义的侦听器容器，同时仍然受益于命名空间的便利性。

您可以JmsListenerContainerFactory通过指定id通过factory-id属性公开的Bean来自动将这些设置显示为，如以下示例所示：

<jms:listener-container connection-factory="myConnectionFactory"
        task-executor="myTaskExecutor"
        destination-resolver="myDestinationResolver"
        transaction-manager="myTransactionManager"
        concurrency="10">

    <jms:listener destination="queue.orders" ref="orderService" method="placeOrder"/>
    
    <jms:listener destination="queue.confirmations" ref="confirmationLogger" method="log"/>

</jms:listener-container>
下表描述了所有可用属性。有关AbstractMessageListenerContainer 各个属性的更多详细信息，请参见和其具体子类的类级javadoc 。Javadoc还讨论了事务选择和消息重新交付方案。

表4. JMS <listener-container>元素的属性
属性	描述
container-type

此侦听器容器的类型。可用选项为default，simple， default102或者simple102（默认选项default）。

container-class

自定义侦听器容器实现类，作为完全限定的类名称。根据属性，默认值为Spring的标准DefaultMessageListenerContainer或 。SimpleMessageListenerContainercontainer-type

factory-id

将此元素定义的设置公开为JmsListenerContainerFactory 带有指定的，id以便它们可以与其他端点重用。

connection-factory

对JMS ConnectionFactoryBean的引用（默认Bean名称为 connectionFactory）。

task-executor

TaskExecutor对JMS侦听器调用程序的Spring的引用。

destination-resolver

DestinationResolver对解决JMSDestination实例的策略的引用。

message-converter

MessageConverter对将JMS消息转换为侦听器方法参数的策略的参考。默认值为SimpleMessageConverter。

error-handler

引用一种ErrorHandler策略，用于处理在执行期间可能发生的任何未捕获的异常MessageListener。

destination-type

这个监听器的JMS目的地类型：queue，topic，durableTopic，sharedTopic，或sharedDurableTopic。这潜在地使pubSubDomain，subscriptionDurable 与subscriptionShared所述容器的性质。默认值为queue（禁用这三个属性）。

response-destination-type

响应的JMS目标类型：queue或topic。默认值为该destination-type属性的值 。

client-id

此侦听器容器的JMS客户端ID。使用持久订阅时必须指定它。

cache

对JMS资源的高速缓存级别：none，connection，session，consumer，或 auto。默认情况下（auto），consumer除非指定了外部事务管理器，否则缓存级别是有效的-在这种情况下，有效的默认值将是none（假设Java EE风格的事务管理，其中给定的ConnectionFactory是可识别XA的池）。

acknowledge

本机JMS确认模式：auto，client，dups-ok，或transacted。值transacted激活本地交易Session。或者，您可以指定transaction-manager属性，如表中稍后所述。默认值为auto。

transaction-manager

对外部PlatformTransactionManager（通常是基于XA的事务协调器，例如Spring的JtaTransactionManager）的引用。如果未指定，则使用本机确认（请参阅acknowledge属性）。

concurrency

每个侦听器启动的并发会话或使用者的数量。它可以是表示最大数的简单数字（例如5），也可以是表示下限和上限的范围（例如3-5）。请注意，指定的最小值只是一个提示，在运行时可能会被忽略。默认值为1。如果1主题侦听器或队列顺序很重要，则应将并发限制在一定范围内。考虑将其提高到一般队列。

prefetch

加载到单个会话中的最大消息数。请注意，增加此数字可能会导致并发消费者饥饿。

receive-timeout

用于接听电话的超时时间（以毫秒为单位）。默认值为1000（一秒钟）。-1表示没有超时。

back-off

指定BackOff用于计算恢复尝试间隔的实例。如果BackOffExecution实现返回BackOffExecution#STOP，则侦听器容器不会进一步尝试恢复。recovery-interval 设置此属性后，将忽略该值。缺省值为a FixedBackOff，间隔为5000毫秒（即5秒）。

recovery-interval

指定两次恢复尝试之间的时间间隔（以毫秒为单位）。它提供了一种方便的方法来创建FixedBackOff具有指定间隔的。有关更多恢复选项，请考虑指定一个BackOff实例。缺省值为5000毫秒（即5秒）。

phase

该容器应在其中启动和停止的生命周期阶段。值越低，此容器启动越早，而其停止越晚。默认值为 Integer.MAX_VALUE，表示容器尽可能晚地启动，并尽快停止。

使用jms模式支持配置基于JCA的侦听器容器非常相似，如以下示例所示：

<jms:jca-listener-container resource-adapter="myResourceAdapter"
        destination-resolver="myDestinationResolver"
        transaction-manager="myTransactionManager"
        concurrency="10">

    <jms:listener destination="queue.orders" ref="myMessageListener"/>

</jms:jca-listener-container>
下表描述了JCA变体的可用配置选项：

表5. JMS <jca-listener-container />元素的属性
属性	描述
factory-id

将此元素定义的设置公开为JmsListenerContainerFactory 带有指定的，id以便它们可以与其他端点重用。

resource-adapter

对JCA ResourceAdapterbean的引用（默认bean名称为 resourceAdapter）。

activation-spec-factory

对的引用JmsActivationSpecFactory。默认设置是自动检测JMS提供程序及其ActivationSpec类（请参阅参考资料DefaultJmsActivationSpecFactory）。

destination-resolver

DestinationResolver解决JMS的策略的参考Destinations。

message-converter

MessageConverter对将JMS消息转换为侦听器方法参数的策略的参考。默认值为SimpleMessageConverter。

destination-type

这个监听器的JMS目的地类型：queue，topic，durableTopic，sharedTopic。或sharedDurableTopic。这潜在地使pubSubDomain，subscriptionDurable和subscriptionShared容器的性质。默认值为queue（禁用这三个属性）。

response-destination-type

响应的JMS目标类型：queue或topic。默认值为该destination-type属性的值 。

client-id

此侦听器容器的JMS客户端ID。使用持久订阅时需要指定它。

acknowledge

本机JMS确认模式：auto，client，dups-ok，或transacted。值transacted激活本地交易Session。或者，您可以指定transaction-manager稍后描述的属性。默认值为auto。

transaction-manager

对SpringJtaTransactionManager或a的 引用，javax.transaction.TransactionManager用于为每个传入消息启动XA事务。如果未指定，则使用本机确认（请参阅 acknowledge属性）。

concurrency

每个侦听器启动的并发会话或使用者的数量。它可以是表示最大值的简单数字（例如5），也可以是表示上下限的范围（例如3-5）。请注意，指定的最小值只是一个提示，通常在运行时使用JCA侦听器容器时将被忽略。预设值为1。

prefetch

加载到单个会话中的最大消息数。请注意，增加此数字可能会导致并发消费者饥饿。

5. JMX
Spring中的JMX（Java管理扩展）支持提供了一些功能，使您可以轻松，透明地将Spring应用程序集成到JMX基础结构中。

JMX？
本章不是JMX的介绍。它没有试图解释为什么您可能要使用JMX。如果您不熟悉JMX，请参阅本章末尾的其他资源。

具体来说，Spring的JMX支持提供了四个核心功能：

将任何Spring bean自动注册为JMX MBean。

用于控制bean的管理界面的灵活机制。

通过远程JSR-160连接器以声明方式公开MBean。

本地和远程MBean资源的简单代理。

这些功能旨在在不将应用程序组件耦合到Spring或JMX接口和类的情况下起作用。实际上，在大多数情况下，您的应用程序类无需了解Spring或JMX即可利用Spring JMX功能。

5.1。将您的Bean导出到JMX
Spring的JMX框架的核心类是MBeanExporter。此类负责获取您的Spring bean并向JMX注册它们MBeanServer。例如，考虑以下类：

package org.springframework.jmx;

public class JmxTestBean implements IJmxTestBean {

    private String name;
    private int age;
    private boolean isSuperman;
    
    public int getAge() {
        return age;
    }
    
    public void setAge(int age) {
        this.age = age;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
    
    public int add(int x, int y) {
        return x + y;
    }
    
    public void dontExposeMe() {
        throw new RuntimeException();
    }
}
要将此Bean的属性和方法公开为MBean的属性和操作，可以MBeanExporter在配置文件中配置该类的实例并传入Bean，如以下示例所示：

<beans>
    <!-- this bean must not be lazily initialized if the exporting is to happen -->
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter" lazy-init="false">
        <property name="beans">
            <map>
                <entry key="bean:name=testBean1" value-ref="testBean"/>
            </map>
        </property>
    </bean>
    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>
</beans>
前面的配置片段中相关的bean定义是exporter bean。该beans属性MBeanExporter确切地告诉您必须将哪些bean导出到JMX MBeanServer。在默认配置中，中的每个条目的键beans Map都用作ObjectName相应条目值所引用的Bean。您可以更改此行为，如控制ObjectNameBean的实例中所述。

使用此配置，该testBeanBean在下方显示为MBean ObjectName bean:name=testBean1。默认情况下，publicbean的所有属性都公开为属性，所有public方法（从Object类继承的方法除外 ）都公开为操作。

MBeanExporter是一个Lifecyclebean（请参阅启动和关闭回调）。默认情况下，MBean在应用程序生命周期中尽可能晚地导出。您可以phase通过设置autoStartup标志来配置导出发生的位置或禁用自动注册。
5.1.1。创建一个MBeanServer
上一节中显示的配置假定该应用程序正在一个（并且只有一个）MBeanServer 已经运行的环境中运行。在这种情况下，Spring尝试找到正在运行的MBeanServer服务器，并将您的bean注册到该服务器（如果有）。当您的应用程序在具有自己的容器（例如Tomcat或IBM WebSphere）中运行时，此行为很有用MBeanServer。

但是，这种方法在独立环境中或在未提供的容器中运行时没有用MBeanServer。为了解决这个问题，您可以MBeanServer通过将org.springframework.jmx.support.MBeanServerFactoryBean类的实例添加到配置中来声明性地创建 实例 。您还可以MBeanServer通过将MBeanExporter实例server属性的MBeanServer值设置为所返回的值 来确保使用特定对象 MBeanServerFactoryBean，如以下示例所示：

<beans>

    <bean id="mbeanServer" class="org.springframework.jmx.support.MBeanServerFactoryBean"/>
    
    <!--
    this bean needs to be eagerly pre-instantiated in order for the exporting to occur;
    this means that it must not be marked as lazily initialized
    -->
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
            <map>
                <entry key="bean:name=testBean1" value-ref="testBean"/>
            </map>
        </property>
        <property name="server" ref="mbeanServer"/>
    </bean>
    
    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

</beans>
在前面的示例中，的实例由MBeanServer创建，MBeanServerFactoryBean并MBeanExporter通过server属性提供给。提供您自己的 MBeanServer实例时，MBeanExporter不会尝试查找正在运行 MBeanServer的MBeanServer实例并使用提供的实例。为了使其正常工作，您必须在类路径上具有JMX实现。

5.1.2。重用现有的MBeanServer
如果未指定服务器，则MBeanExporter尝试自动检测正在运行 MBeanServer。在大多数仅使用一个MBeanServer实例的环境中，这是可行的。但是，当存在多个实例时，导出器可能选择了错误的服务器。在这种情况下，应使用MBeanServer agentId指示要使用的实例，如以下示例所示：

<beans>
    <bean id="mbeanServer" class="org.springframework.jmx.support.MBeanServerFactoryBean">
        <!-- indicate to first look for a server -->
        <property name="locateExistingServerIfPossible" value="true"/>
        <!-- search for the MBeanServer instance with the given agentId -->
        <property name="agentId" value="MBeanServer_instance_agentId>"/>
    </bean>
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="server" ref="mbeanServer"/>
        ...
    </bean>
</beans>
对于平台或现有的MBeanServer具有agentId通过查找方法检索到的动态（或未知）动态的 情况，应使用 factory-method，如以下示例所示：

<beans>
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="server">
            <!-- Custom MBeanServerLocator -->
            <bean class="platform.package.MBeanServerLocator" factory-method="locateMBeanServer"/>
        </property>
    </bean>

    <!-- other beans here -->

</beans>
5.1.3。延迟初始化的MBean
如果使用MBeanExporter还配置了延迟初始化的来配置Bean ，MBeanExporter则不会破坏该协定，并避免实例化该Bean。相反，它向注册了一个代理，MBeanServer并推迟从容器中获取Bean，直到对该代理进行第一次调用为止。

5.1.4。自动注册MBean
通过导出的MBeanExporter，已经是有效MBean的所有Bean都将按原样注册，MBeanServer而无需Spring的进一步干预。MBeanExporter通过将autodetect 属性设置为true，可以使MBean被自动检测，如以下示例所示：

<bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
    <property name="autodetect" value="true"/>
</bean>

<bean name="spring:mbean=true" class="org.springframework.jmx.export.TestDynamicMBean"/>
在前面的示例中，被调用的beanspring:mbean=true已经是有效的JMX MBean，并由Spring自动注册。默认情况下，自动检测到JMX注册的bean的bean名称用作ObjectName。您可以覆盖此行为，如控制ObjectNameBean的实例中所述。

5.1.5。控制注册行为
考虑这样一个春天的情景MBeanExporter尝试注册一个MBean 与MBeanServer使用ObjectName bean:name=testBean1。如果MBean 实例已经在该实例下注册ObjectName，则默认行为是失败（并抛出InstanceAlreadyExistsException）。

您可以精确控制将MBean进行注册时发生的情况MBeanServer。Spring的JMX支持允许三种不同的注册行为来控制注册行为，当注册过程发现anMBean已在相同的之下注册时ObjectName。下表总结了这些注册行为：

表6.注册行为
注册行为	说明
FAIL_ON_EXISTING

这是默认的注册行为。如果一个MBean实例已经在相同的注册ObjectName，将MBean正在注册不注册，而InstanceAlreadyExistsException被抛出。现有 MBean不受影响。

IGNORE_EXISTING

如果MBean已经在同一实例下注册了实例ObjectName，那么 MBean正在注册的实例不会被注册。现有MBean不受影响，并且不会Exception抛出任何异常。这在多个应用程序要共享一个共享MBean中的一个共享的设置中很有用MBeanServer。

REPLACE_EXISTING

如果某个MBean实例已经在同一实例下注册ObjectName，那么MBean先前已注册的现有实例将被取消注册，并且新 实例将MBean在其位置注册（新MBean实例将有效替换先前的实例）。

上表中的值被定义为RegistrationPolicy该类上的枚举。如果要更改默认注册行为，则需要将定义registrationPolicy上的属性值设置 MBeanExporter为这些值之一。

下面的示例显示如何从默认注册行为更改为REPLACE_EXISTING行为：

<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
            <map>
                <entry key="bean:name=testBean1" value-ref="testBean"/>
            </map>
        </property>
        <property name="registrationPolicy" value="REPLACE_EXISTING"/>
    </bean>
    
    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

</beans>
5.2。控制Bean的管理接口
在上一节的示例中，您几乎没有控制bean的管理接口。public 每个导出的bean的所有属性和方法分别作为JMX属性和操作公开。为了对出口的Bean的哪些属性和方法实际上作为JMX属性和操作公开而进行更细粒度的控制，Spring JMX提供了一种全面且可扩展的机制来控制Bean的管理接口。

5.2.1。使用MBeanInfoAssembler介面
在后台，MBeanExporter代表org.springframework.jmx.export.assembler.MBeanInfoAssembler接口的实现，该 接口负责定义每个公开的bean的管理接口。默认实现 org.springframework.jmx.export.assembler.SimpleReflectiveMBeanInfoAssembler定义了一个管理接口，该接口公开了所有公共属性和方法（如前面各节中的示例所示）。Spring提供了MBeanInfoAssembler接口的两个附加实现，使您可以使用源级元数据或任何任意接口来控制生成的管理接口。

5.2.2。使用源级元数据：Java注释
通过使用MetadataMBeanInfoAssembler，您可以使用源级元数据为bean定义管理界面。org.springframework.jmx.export.metadata.JmxAttributeSource接口封装了元数据的读取。Spring JMX提供了使用Java注释的默认实现，即 org.springframework.jmx.export.annotation.AnnotationJmxAttributeSource。您必须MetadataMBeanInfoAssembler使用JmxAttributeSource接口的实现实例配置，才能使其正常运行（没有默认值）。

要标记要导出到JMX的bean，应使用注释对bean类进行 ManagedResource注释。您必须使用ManagedOperation注释将要公开的每个方法标记为一个操作，并使用注释将要公开的每个属性标记为ManagedAttribute。标记属性时，可以省略getter或setter的注释，以分别创建只写或只读属性。

带ManagedResource注释的Bean必须是公共的，公开操作或属性的方法也必须是公共的。
以下示例显示了JmxTestBean我们在创建MBeanServer中使用的类的带注释版本：

package org.springframework.jmx;

import org.springframework.jmx.export.annotation.ManagedResource;
import org.springframework.jmx.export.annotation.ManagedOperation;
import org.springframework.jmx.export.annotation.ManagedAttribute;

@ManagedResource(
        objectName="bean:name=testBean4",
        description="My Managed Bean",
        log=true,
        logFile="jmx.log",
        currencyTimeLimit=15,
        persistPolicy="OnUpdate",
        persistPeriod=200,
        persistLocation="foo",
        persistName="bar")
public class AnnotationTestBean implements IJmxTestBean {

    private String name;
    private int age;
    
    @ManagedAttribute(description="The Age Attribute", currencyTimeLimit=15)
    public int getAge() {
        return age;
    }
    
    public void setAge(int age) {
        this.age = age;
    }
    
    @ManagedAttribute(description="The Name Attribute",
            currencyTimeLimit=20,
            defaultValue="bar",
            persistPolicy="OnUpdate")
    public void setName(String name) {
        this.name = name;
    }
    
    @ManagedAttribute(defaultValue="foo", persistPeriod=300)
    public String getName() {
        return name;
    }
    
    @ManagedOperation(description="Add two numbers")
    @ManagedOperationParameters({
        @ManagedOperationParameter(name = "x", description = "The first number"),
        @ManagedOperationParameter(name = "y", description = "The second number")})
    public int add(int x, int y) {
        return x + y;
    }
    
    public void dontExposeMe() {
        throw new RuntimeException();
    }

}
在前面的示例中，您可以看到JmxTestBean该类标记有 ManagedResource注释，并且该ManagedResource注释配置了一组属性。这些属性可用于配置MBean生成的MBean的各个方面，MBeanExporter稍后将在Source-level Metadata Types中进行详细说明。

无论是age和name属性注释与ManagedAttribute 注释，但是，在的情况下age财产，只标记了getter。这导致这两个属性都作为属性包含在管理界面中，但是该age属性是只读的。

最后，该add(int, int)方法带有ManagedOperation属性标记，而dontExposeMe()方法则没有。使用时，这将导致管理界面仅包含一个操作（add(int, int)）MetadataMBeanInfoAssembler。

以下配置显示了如何配置MBeanExporter来使用 MetadataMBeanInfoAssembler：

<beans>
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="assembler" ref="assembler"/>
        <property name="namingStrategy" ref="namingStrategy"/>
        <property name="autodetect" value="true"/>
    </bean>

    <bean id="jmxAttributeSource"
            class="org.springframework.jmx.export.annotation.AnnotationJmxAttributeSource"/>
    
    <!-- will create management interface using annotation metadata -->
    <bean id="assembler"
            class="org.springframework.jmx.export.assembler.MetadataMBeanInfoAssembler">
        <property name="attributeSource" ref="jmxAttributeSource"/>
    </bean>
    
    <!-- will pick up the ObjectName from the annotation -->
    <bean id="namingStrategy"
            class="org.springframework.jmx.export.naming.MetadataNamingStrategy">
        <property name="attributeSource" ref="jmxAttributeSource"/>
    </bean>
    
    <bean id="testBean" class="org.springframework.jmx.AnnotationTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>
</beans>
在前面的示例中，MetadataMBeanInfoAssembler已经为Bean配置了AnnotationJmxAttributeSource该类的实例，并通过MBeanExporter 汇编程序属性将其传递给。这是利用Spring公开的MBean的元数据驱动的管理接口所需要的全部。

5.2.3。源级元数据类型
下表描述了可在Spring JMX中使用的源级别元数据类型：

表7.源级元数据类型
目的	注解	注释类型
将a的所有实例标记Class为JMX托管资源。

@ManagedResource

类

将方法标记为JMX操作。

@ManagedOperation

方法

将一个getter或setter标记为JMX属性的一半。

@ManagedAttribute

方法（仅getter和setter）

定义操作参数的描述。

@ManagedOperationParameter 和 @ManagedOperationParameters

方法

下表描述了可在这些源级元数据类型上使用的配置参数：

表8.源级元数据参数
参数	描述	适用于
ObjectName

用于MetadataNamingStrategy确定ObjectName托管资源的。

ManagedResource

description

设置资源，属性或操作的友好描述。

ManagedResource，ManagedAttribute，ManagedOperation，或者ManagedOperationParameter

currencyTimeLimit

设置currencyTimeLimit描述符字段的值。

ManagedResource 要么 ManagedAttribute

defaultValue

设置defaultValue描述符字段的值。

ManagedAttribute

log

设置log描述符字段的值。

ManagedResource

logFile

设置logFile描述符字段的值。

ManagedResource

persistPolicy

设置persistPolicy描述符字段的值。

ManagedResource

persistPeriod

设置persistPeriod描述符字段的值。

ManagedResource

persistLocation

设置persistLocation描述符字段的值。

ManagedResource

persistName

设置persistName描述符字段的值。

ManagedResource

name

设置操作参数的显示名称。

ManagedOperationParameter

index

设置操作参数的索引。

ManagedOperationParameter

5.2.4。使用AutodetectCapableMBeanInfoAssembler介面
为了进一步简化配置，Spring包含了该 AutodetectCapableMBeanInfoAssembler接口，该MBeanInfoAssembler 接口扩展了该接口以添加对自动检测MBean资源的支持。如果您MBeanExporter使用实例配置 AutodetectCapableMBeanInfoAssembler，则可以对包含的Bean进行“投票”以暴露给JMX。

该AutodetectCapableMBeanInfo接口的唯一实现是MetadataMBeanInfoAssembler，该投票将包括该ManagedResource属性标记的任何bean 。在这种情况下，默认方法是将Bean名称用作ObjectName，这将导致类似于以下内容的配置：

<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <!-- notice how no 'beans' are explicitly configured here -->
        <property name="autodetect" value="true"/>
        <property name="assembler" ref="assembler"/>
    </bean>
    
    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>
    
    <bean id="assembler" class="org.springframework.jmx.export.assembler.MetadataMBeanInfoAssembler">
        <property name="attributeSource">
            <bean class="org.springframework.jmx.export.annotation.AnnotationJmxAttributeSource"/>
        </property>
    </bean>

</beans>
请注意，在上述配置中，没有将任何bean传递给MBeanExporter。但是，由于JmxTestBean仍使用ManagedResource 属性标记并MetadataMBeanInfoAssembler检测到该属性并对其进行表决，因此该属性仍处于注册状态。这种方法的唯一问题是JmxTestBeannow的名称具有商业意义。您可以通过更改“控制Bean实例”中ObjectName 定义的默认创建行为来解决此问题。ObjectName

5.2.5。使用Java接口定义管理接口
除了之外MetadataMBeanInfoAssembler，Spring还包括 InterfaceBasedMBeanInfoAssembler，您可以使用来约束基于接口集合中定义的方法集公开的方法和属性。

尽管公开MBean的标准机制是使用接口和简单的命名方案，InterfaceBasedMBeanInfoAssembler但是通过消除对命名约定的需要，允许您使用多个接口以及消除对实现MBean接口的bean的需求，扩展了此功能。

考虑以下接口，该接口用于为JmxTestBean我们前面显示的类定义管理接口 ：

public interface IJmxTestBean {

    public int add(int x, int y);
    
    public long myOperation();
    
    public int getAge();
    
    public void setAge(int age);
    
    public void setName(String name);
    
    public String getName();

}
该接口定义在JMX MBean上作为操作和属性公开的方法和属性。以下代码显示了如何配置Spring JMX以使用此接口作为管理接口的定义：

<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
            <map>
                <entry key="bean:name=testBean5" value-ref="testBean"/>
            </map>
        </property>
        <property name="assembler">
            <bean class="org.springframework.jmx.export.assembler.InterfaceBasedMBeanInfoAssembler">
                <property name="managedInterfaces">
                    <value>org.springframework.jmx.IJmxTestBean</value>
                </property>
            </bean>
        </property>
    </bean>
    
    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

</beans>
在前面的示例中，将InterfaceBasedMBeanInfoAssembler其配置为IJmxTestBean在构造任何bean的管理接口时使用该 接口。重要的是要理解，InterfaceBasedMBeanInfoAssembler 不需要由处理的bean来实现用于生成JMX管理接口的接口。

在上述情况下，该IJmxTestBean接口用于为所有bean构造所有管理接口。在许多情况下，这不是理想的行为，您可能想对不同的bean使用不同的接口。在这种情况下，您可以通过该 属性传递 InterfaceBasedMBeanInfoAssembler一个Properties实例interfaceMappings，其中每个条目的键是Bean名称，每个条目的值是一个逗号分隔的接口名称列表，用于该Bean。

如果没有通过managedInterfaces或 interfaceMappings属性指定管理接口，则InterfaceBasedMBeanInfoAssemblerref会反映在bean上，并使用该bean实现的所有接口来创建管理接口。

5.2.6。使用MethodNameBasedMBeanInfoAssembler
MethodNameBasedMBeanInfoAssembler使您可以指定作为属性和操作公开给JMX的方法名称的列表。以下代码显示了示例配置：
```xml
<bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
    <property name="beans">
        <map>
            <entry key="bean:name=testBean5" value-ref="testBean"/>
        </map>
    </property>
    <property name="assembler">
        <bean class="org.springframework.jmx.export.assembler.MethodNameBasedMBeanInfoAssembler">
            <property name="managedMethods">
                <value>add,myOperation,getName,setName,getAge</value>
            </property>
        </bean>
    </property>
</bean>
```
在前面的例子中，可以看到的是，add和myOperation方法被公开为JMX操作，和getName()，setName(String)和getAge()被暴露为JMX属性的适当的一半。在前面的代码中，方法映射适用于公开给JMX的bean。要逐个bean地控制方法公开，可以使用methodMappingsof属性MethodNameMBeanInfoAssembler将bean名称映射到方法名称列表。
### 5.3. 控制ObjectName您的Bean的实例
在幕后，MBeanExporter委托的实现为 它注册的每个beanObjectNamingStrategy获取ObjectName实例。默认情况下，默认实现KeyNamingStrategy使用的键 beans Map作为ObjectName。此外，KeyNamingStrategy可以将的键映射beans Map到一个Properties文件（或多个文件）中的条目以解决 ObjectName。除了之外KeyNamingStrategy，Spring还提供了两个附加的 ObjectNamingStrategy实现：（基于bean的JVM身份IdentityNamingStrategy构建一个 ObjectName）和MetadataNamingStrategy（使用源级元数据获取ObjectName）。
#### 5.3.1. ObjectName从属性读取实例
您可以配置自己的KeyNamingStrategy实例，并将其配置为ObjectName从Properties实例读取 实例，而不使用Bean键。在 KeyNamingStrategy试图定位在入门Properties用钥匙对应于bean的关键。如果没有找到条目，或者Properties实例是 null，则使用bean密钥本身。

以下代码显示的示例配置KeyNamingStrategy：