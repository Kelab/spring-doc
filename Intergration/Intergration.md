# Intergration

## 1.REST端点（REST Endpoints）

### 1.1. `RestTemplate`

在`RestTemplate`通过HTTP客户端库提供了一个更高水平的API。它使在一行中轻松调用REST端点变得容易。它公开了以下几组重载方法：

| 方法组            | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
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

```
RestTemplate template = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
```

每个都`ClientHttpRequestFactory`公开特定于基础HTTP客户端库的配置选项-例如，用于凭证，连接池和其他详细信息。

```
RestTemplate template = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
```

##### 统一资源标识符(URIs)

许多`RestTemplate`方法都接受URI模板和URI模板变量（作为`String`变量参数或）`Map<String,String>`。

下面的示例使用一个`String`变量参数：

```
String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/bookings/{booking}", String.class, "42", "21");
```

以下示例使用`Map<String, String>`：

```
Map<String, String> vars = Collections.singletonMap("hotel", "42");

String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/rooms/{hotel}", String.class, vars);
```

请注意，URI模板是自动编码的，如以下示例所示：

```
restTemplate.getForObject("https://example.com/hotel list", String.class);

// Results in request to "https://example.com/hotel%20list"
```

您可以使用的`uriTemplateHandler`属性来自`RestTemplate`定义URI的编码方式。或者，您可以准备一个`java.net.URI`并将其传递到`RestTemplate`接受的方法之一`URI`。

有关使用和编码URI的更多详细信息，请参阅[URI链接](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-uri-building)。

##### 标头(Headers)

您可以使用这些`exchange()`方法指定请求标头，如以下示例所示：

```
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

```
URI location = template.postForLocation("https://example.com/people", person);
```

您无需显式设置请求的Content-Type标头。在大多数情况下，您可以根据源`Object`类型找到兼容的消息转换器，并且所选的消息转换器会相应地设置内容类型。如有必要，可以使用这些 `exchange`方法显式提供`Content-Type`请求标头，进而影响选择哪个消息转换器。

在GET上，响应的主体反序列化为output `Object`，如以下示例所示：

```
Person person = restTemplate.getForObject("https://example.com/people/{id}", Person.class, 42);
```

`Accept`不需要显式设置请求的标头。在大多数情况下，可以根据预期的响应类型找到兼容的消息转换器，这有助于填充`Accept`标头。如有必要，可以使用这些`exchange` 方法`Accept`显式提供标头。

默认情况下，根据有助于确定存在哪些可选转换库的类路径检查，`RestTemplate`注册所有内置 [消息转换器](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-message-conversion)。您还可以将消息转换器设置为显式使用。

#### 1.1.3. 信息转换(Message Conversion)

该`spring-web`模块包含了`HttpMessageConverter`用于阅读和写作，通过HTTP请求和响应的身体合同`InputStream`和`OutputStream`。 `HttpMessageConverter`在客户端（例如在中`RestTemplate`）和服务器端（例如在Spring MVC REST控制器中）使用实例。

框架中提供了主要媒体（MIME）类型的具体实现，默认情况下，已`RestTemplate`在客户端和`RequestMethodHandlerAdapter`服务器端注册了具体实现 （请参阅 [配置消息转换器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-message-converters)）。

`HttpMessageConverter`以下各节介绍了的实现。对于所有转换器，都使用默认媒体类型，但是您可以通过设置`supportedMediaTypes`bean属性来覆盖它 。下表描述了每种实现：

| 消息转换器                               | 描述                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
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

```
MappingJacksonValue value = new MappingJacksonValue(new User("eric", "7!jd#h23"));
value.setSerializationView(User.WithoutPasswordView.class);

RequestEntity<MappingJacksonValue> requestEntity =
    RequestEntity.post(new URI("https://example.com/user")).body(value);

ResponseEntity<String> response = template.exchange(requestEntity, String.class);
```

##### 多部分(Multipart)

要发送多部分数据，您需要提供`MultiValueMap<String, Object>`其值可以是`Object`部分内容，`Resource`文件部分或`HttpEntity`带有标题的部分内容的。例如：

```
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

```
MultiValueMap<String, Object> parts = ...;
template.postForObject("https://example.com/upload", parts, Void.class);
```

如果`MultiValueMap`包含至少一个非`String`值时，`Content-Type`被设定为`multipart/form-data`通过`FormHttpMessageConverter`。如果`MultiValueMap`有 `String`值`Content-Type`默认为`application/x-www-form-urlencoded`。如有必要，`Content-Type`也可以明确设置。

### 1.2使用`AsyncRestTemplate`（不推荐使用）

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

```
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

### 2.3. Java Web服务

Spring提供了对标准Java Web服务API的全面支持：

- 使用JAX-WS公开Web服务
- 使用JAX-WS访问Web服务

除了在Spring Core中对JAX-WS的库存支持之外，Spring产品组合还具有[Spring Web Services](http://www.springframework.org/spring-ws)，它是合同优先，文档驱动的Web服务的解决方案-强烈建议构建现代的，面向未来的Web服务。

#### 2.3.1. 使用JAX-WS公开基于Servlet的Web服务

Spring提供了JAX-WS servlet的端点实现提供了一个方便的基类： `SpringBeanAutowiringSupport`。为了公开我们的内容`AccountService`，我们扩展了Spring的 `SpringBeanAutowiringSupport`类并在此处实现我们的业务逻辑，通常将调用委派给业务层。我们使用Spring的`@Autowired` 注释来表达对Spring管理的bean的依赖。以下示例显示了扩展的类`SpringBeanAutowiringSupport`：

```
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

#### 2.3.2. 使用JAX-WS导出独立的Web服务

Oracle JDK随附的内置JAX-WS提供程序通过使用JDK中也包含的内置HTTP服务器来支持Web服务公开。Spring `SimpleJaxWsServiceExporter`会`@WebService`在Spring应用程序上下文中检测所有带注释的bean，并通过默认的JAX-WS服务器（JDK HTTP服务器）导出它们。

在这种情况下，终结点实例被定义和管理为Spring bean本身。它们已在JAX-WS引擎中注册，但是它们的生命周期取决于Spring应用程序上下文。这意味着您可以将Spring功能（例如显式依赖项注入）应用于端点实例。注释驱动的注入也`@Autowired`同样有效。以下示例显示了如何定义这些bean：

```
<bean class="org.springframework.remoting.jaxws.SimpleJaxWsServiceExporter">
    <property name="baseAddress" value="http://localhost:8080/"/>
</bean>

<bean id="accountServiceEndpoint" class="example.AccountServiceEndpoint">
    ...
</bean>

...
```

该`AccountServiceEndpoint`可以，但并没有从Spring的推导`SpringBeanAutowiringSupport`，因为在这个例子中，端点是完全春季管理的bean。这意味着端点实现可以如下所示（不声明任何超类，并且`@Autowired`仍采用Spring的配置注释）：

```
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

#### 2.3.3. 使用JAX-WS RI的Spring支持导出Web服务

作为GlassFish项目的一部分开发的Oracle JAX-WS RI，将Spring支持作为其JAX-WS Commons项目的一部分。这允许将JAX-WS端点定义为Spring管理的Bean，类似于上[一节中](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-web-services-jaxws-export-standalone)讨论的独立模式，  但这次是在Servlet环境中。

|      | 这在Java EE环境中不可移植。它主要适用于将JAX-WS RI嵌入为Web应用程序一部分的非EE环境，例如Tomcat。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

与导出基于servlet的终结点的标准样式的不同之处在于，终结点实例本身的生命周期由Spring管理，并且在中仅定义了一个JAX-WS servlet `web.xml`。使用标准的Java EE样式（如前所述），每个服务端点都有一个servlet定义，每个端点通常都委派给Spring Bean（`@Autowired`如前所述，通过使用）。

有关 设置和使用样式的详细信息，请参见https://jax-ws-commons.java.net/spring/。

#### 2.3.4. 使用JAX-WS访问Web服务

Spring提供了两个工厂bean来创建JAX-WS Web服务代理，分别是 `LocalJaxWsServiceFactoryBean`和`JaxWsPortProxyFactoryBean`。前者只能返回一个JAX-WS服务类供我们使用。后者是完整版本，可以返回实现我们的业务服务接口的代理。在以下示例中，我们`JaxWsPortProxyFactoryBean`用于`AccountService`（再次）为端点创建代理 ：

```
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

```
<bean id="client" class="example.AccountClientImpl">
    ...
    <property name="service" ref="accountWebService"/>
</bean>
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

### 2.4. RMI（不建议使用）

从Spring Framework 5.3开始，RMI支持已被弃用，不会被取代。

通过使用Spring对RMI的支持，您可以通过RMI基础结构透明地公开服务。进行了此设置之后，除了不存在对安全上下文传播或远程事务传播的标准支持这一事实之外，您基本上具有与远程EJB相似的配置。当您使用RMI调用程序时，Spring确实为此类附加调用上下文提供了挂钩，因此，例如，您可以插入安全框架或自定义安全凭证。

#### 2.4.1 （ Exporting the Service by Using）通过使用导出服务`RmiServiceExporter`

使用`RmiServiceExporter`，我们可以将AccountService对象的接口公开为RMI对象。可以使用`RmiProxyFactoryBean`或通过传统RMI服务通过普通RMI访问该接口。在`RmiServiceExporter`明确支持通过RMI调用任何非RMI业务。

我们首先必须在Spring容器中设置服务。以下示例显示了如何执行此操作：

```
<bean id="accountService" class="example.AccountServiceImpl">
    <!-- any additional properties, maybe a DAO? -->
</bean>
```

接下来，我们必须使用公开我们的服务`RmiServiceExporter`。以下示例显示了如何执行此操作：

```
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

#### 2.4.2. 在客户端链接服务（ Linking in the Service at the Client）

我们的客户是一个使用`AccountService`来管理帐户的简单对象，如以下示例所示：

```
public class SimpleObject {

    private AccountService accountService;

    public void setAccountService(AccountService accountService) {
        this.accountService = accountService;
    }

    // additional methods using the accountService
}
```

为了在客户端上链接服务，我们创建了一个单独的Spring容器，其中包含以下简单对象和服务链接配置位：

```
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

```
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

#### 2.5.2. （Exposing Your Beans by Using）使用公开您的bean `HessianServiceExporter`

在名为的新创建的应用程序上下文中`remoting-servlet.xml`，我们创建一个 `HessianServiceExporter`来导出我们的服务，如以下示例所示：

```
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

```
<bean name="accountExporter" class="org.springframework.remoting.caucho.HessianServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

在后一种情况下，您应该在中为该导出器定义一个相应的servlet `web.xml`，并得到相同的最终结果：导出器被映射到位于的请求路径 `/remoting/AccountService`。注意，servlet名称需要与目标导出器的bean名称匹配。以下示例显示了如何执行此操作：

```
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

```
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

```
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

#### 2.6.1. 公开服务对象。（Exposing the Service Object）

为服务对象设置HTTP调用程序基础结构与使用Hessian进行设置的方法非常相似。正如Hessian支持提供的那样 `HessianServiceExporter`，Spring的HttpInvoker支持提供了 `org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter`。

为了`AccountService`在Spring Web MVC中公开（前面提到的） `DispatcherServlet`，需要在调度程序的应用程序上下文中进行以下配置，如以下示例所示：

```
<bean name="/AccountService" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

此类导出器定义通过`DispatcherServlet`实例的标准映射功能公开，如[有关Hessian的部分所述](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#remoting-caucho-protocols)。

或者，您可以`HttpInvokerServiceExporter`在根应用程序上下文中创建一个（例如，在中`'WEB-INF/applicationContext.xml'`），如以下示例所示：

```
<bean name="accountExporter" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

另外，您可以在中为该导出器定义一个相应的servlet，该`web.xml`servlet名称与目标导出器的bean名称匹配，如以下示例所示：

```
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

```
<bean id="httpInvokerProxy" class="org.springframework.remoting.httpinvoker.HttpInvokerProxyFactoryBean">
    <property name="serviceUrl" value="https://remotehost:8080/remoting/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

如前所述，您可以选择要使用的HTTP客户端。默认情况下，会 `HttpInvokerProxy`使用JDK的HTTP功能，但是您也可以`HttpComponents`通过设置`httpInvokerRequestExecutor`属性来使用Apache 客户端。以下示例显示了如何执行此操作：

```
<property name="httpInvokerRequestExecutor">
    <bean class="org.springframework.remoting.httpinvoker.HttpComponentsHttpInvokerRequestExecutor"/>
</property>
```

### 2.7. JMS（已弃用）

从Spring Framework 5.3开始，已弃用JMS远程支持，并且将不会替换。

您还可以通过使用JMS作为基础通信协议来透明地公开服务。Spring框架中的JMS远程支持非常基本。它在`same thread`和上以相同的非事务发送和接收`Session`。结果，吞吐量取决于实现方式。请注意，这些单线程和非事务性约束仅适用于Spring的JMS远程支持。有关Spring对基于JMS的消息传递的丰富支持的信息，请参见[JMS（Java消息服务）](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms)。

服务器和客户端均使用以下接口：

```
package com.foo;

public interface CheckingAccountService {

    public void cancelAccount(Long accountId);
}
```

在服务器端使用上述接口的以下简单实现：

```
package com.foo;

public class SimpleCheckingAccountService implements CheckingAccountService {

    public void cancelAccount(Long accountId) {
        System.out.println("Cancelling account [" + accountId + "]");
    }
}
```

以下配置文件包含在客户端和服务器上共享的JMS基础结构Bean：

```
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

#### 2.7.1. 服务器端配置

在服务器上，您需要公开使用的服务对象 `JmsInvokerServiceExporter`，如以下示例所示：

```
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

#### 2.7.2. 客户端配置

客户端只需要创建一个客户端代理即可实现约定的接口（`CheckingAccountService`）。

下面的示例定义了可以注入到其他客户端对象中的bean（代理负责通过JMS将调用转发到服务器端对象）：

```
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

## 3.企业JavaBeans（EJB）集成（Accessing EJBs）

### 3.1.1 概念（Concepts）

要在本地或远程无状态会话Bean上调用方法，客户机代码通常必须执行JNDI查找以获取（本地或远程）EJB Home对象，然后`create`对该对象使用方法调用以获取实际的（本地或远程）EJB对象。EJB对象。然后在EJB上调用一种或多种方法。

为了避免重复的低级代码，许多EJB应用程序都使用服务定位器和业务委托模式。这些比在整个客户端代码中喷射JNDI查找要好，但是它们的常规实现有很多缺点：

- 通常，使用EJB的代码取决于Service Locator或Business Delegate单例，因此很难进行测试。
- 在不使用业务委托的情况下使用服务定位器模式的情况下，应用程序代码仍然最终必须`create()`在EJB home上调用该方法并处理产生的异常。因此，它仍然与EJB API和EJB编程模型的复杂性联系在一起。
- 实现业务委托模式通常会导致大量的代码重复，我们必须编写许多在EJB上调用相同方法的方法。

Spring的方法是允许创建和使用代理对象（通常在Spring容器内配置），这些代理对象充当无代码的业务委托。除非您在此类代码中实际添加了实际价值，否则您无需在手动编码的业务委托中编写另一个Service Locator，另一个JNDI查找或重复方法。

#### 3.1.2. 访问本地SLSB（ Accessing Local SLSBs）

假设我们有一个Web控制器，需要使用本地EJB。我们遵循最佳实践，并使用EJB业务方法接口模式，以便EJB的本地接口扩展了非EJB特定的业务方法接口。我们称这种业务方法为接口`MyComponent`。以下示例显示了这样的接口：

```
public interface MyComponent {
    ...
}
```

使用业务方法接口模式的主要原因之一是确保本地接口中的方法签名与bean实现类之间的同步是自动的。另一个原因是，如果有必要的话，以后可以使我们更轻松地切换到服务的POJO（普通旧Java对象）实现。我们还需要实现本地home接口，并提供一个实现类`SessionBean`和`MyComponent`业务方法接口的实现类。现在，将Web层控制器连接到EJB实现所需的唯一Java编码是`MyComponent` 在控制器上公开类型的setter方法。这会将引用另存为控制器中的实例变量。以下示例显示了如何执行此操作：

```
private MyComponent myComponent;

public void setMyComponent(MyComponent myComponent) {
    this.myComponent = myComponent;
}
```

随后，我们可以在控制器中的任何业务方法中使用此实例变量。现在，假设我们从Spring容器中获取了控制器对象，我们可以（在相同上下文中）配置一个`LocalStatelessSessionProxyFactoryBean`实例，即EJB代理对象。我们配置代理，并`myComponent`使用以下配置条目设置控制器的 属性：

```
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

```
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

#### 3.1.3. 访问远程SLSB（Accessing Remote SLSBs）

访问远程EJB与访问本地EJB本质上相同，只是使用了 `SimpleRemoteStatelessSessionProxyFactoryBean`or`<jee:remote-slsb>`配置元素。当然，无论是否使用Spring，远程调用语义都适用：调用另一台计算机上另一台VM中的对象上的方法时，有时在使用情况和故障处理方面必须区别对待。

与非Spring方法相比，Spring的EJB客户端支持增加了另一优势。通常，在本地或远程调用EJB之间轻松地来回切换EJB客户端代码是有问题的。这是因为远程接口方法必须声明它们throw `RemoteException`，而客户端代码必须对此进行处理，而本地接口方法则不需要。通常需要修改为需要移至远程EJB的本地EJB编写的客户端代码，以添加对远程异常的处理，为需要移至本地EJB的远程EJB编写的客户端代码可以保持不变，但可以执行以下操作：许多不必要的远程异常处理，或进行修改以删除该代码。使用Spring远程EJB代理，您可以声明不抛出任何异常`RemoteException`在您的业务方法接口和实现EJB代码中，拥有一个完全相同的远程接口（除了它确实抛出`RemoteException`），并且依靠代理来动态地将这两个接口视为相同。也就是说，客户端代码不必处理已检查的`RemoteException`类。`RemoteException`在EJB调用期间抛出的所有实数都将重新抛出为未经检查的`RemoteAccessException`类，该类是的子类`RuntimeException`。然后，您可以在本地EJB或远程EJB（甚至纯Java对象）实现之间随意切换目标服务，而无需了解或关心客户端代码。当然，这是可选的：没有什么可以阻止您`RemoteException`在业务界面中进行声明。

#### 3.1.4. 访问EJB 2.x SLSB与EJB 3 SLSB（Accessing EJB 2.x SLSBs Versus EJB 3 SLSBs）

通过Spring访问EJB 2.x会话Bean和EJB 3会话Bean在很大程度上是透明的。Spring的EJB访问器（包括`<jee:local-slsb>`和 `<jee:remote-slsb>`设施）在运行时透明地适应实际组件。它们会处理Home接口（如果找到）（EJB 2.x样式），或者在没有Home接口可用时（EJB 3样式）执行直接组件调用。

注意：对于EJB 3会话Bean，您也可以有效地使用`JndiObjectFactoryBean`/ `<jee:jndi-lookup>`，因为公开了完全可用的组件引用以用于在那里的普通JNDI查找。定义显式`<jee:local-slsb>`或`<jee:remote-slsb>` 查找可提供一致且更显式的EJB访问配置。

## 4. JMS（Java消息服务）（Using Spring JMS）

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

### 4.1。使用Spring JMS（Using Spring JMS）

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

#### 4.1.2。连接数（Connections）

该`JmsTemplate`要求的一个参考`ConnectionFactory`。它`ConnectionFactory` 是JMS规范的一部分，并且是使用JMS的切入点。客户端应用程序将其用作工厂以创建与JMS提供程序的连接，并封装各种配置参数，其中许多是特定于供应商的，例如SSL配置选项。

当在EJB中使用JMS时，供应商提供JMS接口的实现，以便它们可以参与声明式事务管理并执行连接和会话的池化。为了使用此实现，Java EE容器通常要求您将JMS连接工厂声明为`resource-ref`EJB或Servlet部署描述符中的内部。为了确保在`JmsTemplate`EJB内部使用这些功能 ，客户端应用程序应确保其引用的托管实现`ConnectionFactory`。

##### 缓存消息资源（ Caching Messaging Resources）

标准API涉及创建许多中间对象。要发送消息，请执行以下“ API”遍历：

```
ConnectionFactory->连接->会话-> MessageProducer->发送
```

在`ConnectionFactory`和`Send`操作之间，创建并销毁了三个中间对象。为了优化资源使用并提高性能，Spring提供了两种实现`ConnectionFactory`。

##### 使用 `SingleConnectionFactory`

Spring提供了`ConnectionFactory`接口的实现，该接口 在所有调用`SingleConnectionFactory`中返回相同的内容`Connection`， `createConnection()`并忽略对的调用`close()`。这对于测试和独立环境非常有用，因此同一连接可用于`JmsTemplate`可能跨越任意数量事务的多个 调用。`SingleConnectionFactory` 引用了`ConnectionFactory`通常来自JNDI的标准。

##### 使用 `CachingConnectionFactory`

该`CachingConnectionFactory`扩展的功能`SingleConnectionFactory` ，并增加了缓存`Session`，`MessageProducer`和`MessageConsumer`实例。初始缓存大小设置为`1`。您可以使用该`sessionCacheSize`属性来增加缓存的会话数。请注意，由于根据会话的确认模式缓存会话，因此实际缓存的会话数大于该数量，因此当`sessionCacheSize`设置为one时，最多可以有四个缓存的会话实例（每个确认模式一个）。`MessageProducer`和`MessageConsumer`实例在其自己的会话中缓存，并且在缓存时还要考虑生产者和使用者的独特属性。MessageProducers根据其目的地进行缓存。基于由目标，选择器，noLocal传递标志和持久订阅名称（如果创建持久使用者）组成的键来缓存MessageConsumers。

#### 4.1.3。目的地管理（Destination Management）

作为`ConnectionFactory`实例，目标是可以在JNDI中存储和检索的JMS管理的对象。在配置Spring应用程序上下文时，可以使用JNDI`JndiObjectFactoryBean`工厂类或`<jee:jndi-lookup>`对对象对JMS目标的引用执行依赖项注入。但是，如果应用程序中有大量目标，或者JMS提供程序具有独特的高级目标管理功能，则此策略通常很麻烦。这种高级目标管理的示例包括动态目标的创建或对目标的分层名称空间的支持。将`JmsTemplate`目标名称的解析委托给实现`DestinationResolver`接口的JMS目标对象 。`DynamicDestinationResolver`是由...使用的默认实现`JmsTemplate`并适应解析的动态目标。`JndiDestinationResolver`还提供A 充当JNDI中包含的目的地的服务定位器，并且可以选择回退到中包含的行为 `DynamicDestinationResolver`。

通常，仅在运行时才知道JMS应用程序中使用的目的地，因此，在部署应用程序时无法通过管理方式创建。这通常是因为在交互的系统组件之间存在共享的应用程序逻辑，这些组件根据已知的命名约定在运行时创建目标。即使动态目标的创建不是JMS规范的一部分，但大多数供应商都提供了此功能。动态目标是使用用户定义的名称创建的，该名称将它们与临时目标区分开来，并且通常未在JNDI中注册。提供商之间使用的用于创建动态目标的API有所不同，因为与目标关联的属性是特定于供应商的。然而，`TopicSession` `createTopic(String topicName)`或`QueueSession` `createQueue(String queueName)`使用默认目标属性创建新目标的方法。`DynamicDestinationResolver`然后，根据供应商的实现，还可以创建一个物理目标，而不仅仅是解决一个目标。

布尔值属性`pubSubDomain`用于使用`JmsTemplate`正在使用的JMS域的知识来配置。默认情况下，此属性的值为false，表示`Queues`将使用点对点域。（由所使用`JmsTemplate`）此属性通过`DestinationResolver`接口的实现确定动态目标解析的行为。

您还可以`JmsTemplate`通过属性将设置为具有默认目的地`defaultDestination`。默认目标是带有不引用特定目标的发送和接收操作。

#### 4.1.4。消息侦听器容器（Message Listener Containers）

在EJB世界中，JMS消息最常见的用途之一就是驱动消息驱动的bean（MDB）。Spring提供了一种解决方案，以不将用户绑定到EJB容器的方式创建消息驱动的POJO（MDP）。（有关Spring对MDP支持的详细介绍，请参见[异步接收：消息驱动的POJO](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-receiving-async)。）从Spring Framework 4.1开始，可以使用以下方法对端点方法进行注释`@JmsListener` -有关更多详细信息，请参见[注释驱动的侦听器端点](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-annotated)。

消息侦听器容器用于从JMS消息队列接收消息并驱动`MessageListener`注入到其中的消息。侦听器容器负责消息接收的所有线程，并分派到侦听器中进行处理。消息侦听器容器是MDP与消息传递提供程序之间的中介，并负责注册接收消息，参与事务，资源获取和释放，异常转换等。这使您可以编写与接收消息（并可能响应消息）相关的（可能很复杂的）业务逻辑，并将样板JMS基础结构问题委托给框架。

Spring附带了两个标准的JMS消息侦听器容器，每个容器都有其专门的功能集。

- [`SimpleMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-mdp-simple)
- [`DefaultMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-mdp-default)

##### 使用 `SimpleMessageListenerContainer`

此消息侦听器容器是两种标准样式中的简单容器。它在启动时创建了固定数量的JMS会话和使用者，使用标准JMS`MessageConsumer.setMessageListener()`方法注册了侦听器，并将其留给JMS提供者来执行侦听器回调。此变体不允许动态适应运行时需求或参与外部管理的事务。在兼容性方面，它非常接近独立JMS规范的精神，但通常与Java EE的JMS限制不兼容。

|      | 尽管`SimpleMessageListenerContainer`不允许参与外部管理的事务，但它确实支持本机JMS事务。要启用此功能，可以将`sessionTransacted`标志切换为，`true`或者在XML名称空间中将`acknowledge`属性设置 为`transacted`。从您的侦听器抛出的异常会导致回滚，并重新传递消息。或者，考虑使用 `CLIENT_ACKNOWLEDGE`模式，该模式在出现异常的情况下也提供重新交付但不使用事务处理的`Session`实例，因此`Session`在事务协议中不包括任何其他 操作（例如发送响应消息）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 默认`AUTO_ACKNOWLEDGE`模式不能提供适当的可靠性保证。当侦听器执行失败时（由于提供者会在侦听器调用后自动确认每条消息，并且没有异常要传播到提供者），或者在侦听器容器关闭时（您可以通过设置`acceptMessagesWhileStopping`标志进行配置），消息可能会丢失。确保出于可靠性需求（例如，为了可靠的队列处理和持久的主题订阅）使用事务处理的会话。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 使用 `DefaultMessageListenerContainer`

大多数情况下使用此消息侦听器容器。与之相反 `SimpleMessageListenerContainer`，此容器变量允许动态适应运行时需求，并且能够参与外部管理的事务。配置为时，每个接收到的消息都会在XA事务中注册 `JtaTransactionManager`。结果，处理可以利用XA事务语义。该侦听器容器在JMS提供程序的低要求，高级功能（例如参与外部管理的事务）以及与Java EE环境的兼容性之间取得了良好的平衡。

您可以自定义容器的缓存级别。请注意，当未启用缓存时，将为每个消息接收创建一个新的连接和一个新的会话。将其与具有高负载的非持久订阅结合使用可能会导致消息丢失。在这种情况下，请确保使用适当的缓存级别。

当代理崩溃时，此容器还具有可恢复的功能。默认情况下，一个简单的`BackOff`实现每五秒钟重试一次。您可以`BackOff`为更细粒度的恢复选项指定自定义实现。有关`ExponentialBackOff`示例，请参见api-spring-framework / util / backoff / ExponentialBackOff.html [ ]。

|      | 与其兄弟（[`SimpleMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-mdp-simple)） 一样，它`DefaultMessageListenerContainer`支持本机JMS事务，并允许自定义确认模式。如果对您的方案可行，则强烈建议在外部管理的事务上使用此方法，也就是说，如果JVM死亡，您可以偶尔接收重复的消息。业务逻辑中的自定义重复消息检测步骤可以解决这种情况-例如，以业务实体存在检查或协议表检查的形式。任何这样的安排是显著比其他更有效的：用一个XA事务包裹整个处理（通过在配置 `DefaultMessageListenerContainer`有`JtaTransactionManager`）以覆盖所述JMS消息的接收，以及业务逻辑在您的消息侦听器的执行（包括数据库操作等）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 默认`AUTO_ACKNOWLEDGE`模式不能提供适当的可靠性保证。当侦听器执行失败时（由于提供者会在侦听器调用后自动确认每条消息，并且没有异常要传播到提供者），或者在侦听器容器关闭时（您可以通过设置`acceptMessagesWhileStopping`标志进行配置），消息可能会丢失。确保出于可靠性需求（例如，为了可靠的队列处理和持久的主题订阅）使用事务处理的会话。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 4.1.5。交易管理（Transaction Management）

Spring提供了一个`JmsTransactionManager`管理单个JMS事务的 `ConnectionFactory`。这使JMS应用程序可以利用Spring的托管事务功能，如 [“数据访问”一章的“事务管理”部分所述](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction)。在`JmsTransactionManager`执行本地资源交易，结合从指定的JMS连接/会话对`ConnectionFactory`给线程。 `JmsTemplate`自动检测此类交易资源并对其进行相应操作。

在Java EE环境中，`ConnectionFactory`池连接和实例实例，因此这些资源可在事务之间有效地重用。在独立环境中，使用Spring的`SingleConnectionFactory`结果在共享的JMS中`Connection`，每个事务都有自己独立的`Session`。或者，考虑使用提供程序专用的池适配器，例如ActiveMQ的`PooledConnectionFactory` 类。

您还可以`JmsTemplate`与`JtaTransactionManager`和具有XA功能的JMS一起使用 `ConnectionFactory`以执行分布式事务。请注意，这需要使用JTA事务管理器以及正确的XA配置的ConnectionFactory。（检查您的Java EE服务器或JMS提供程序的文档。）

跨托管和非托管的事务环境下重用代码可以使用JMS API来创建一个混乱的时候`Session`从一个`Connection`。这是因为JMS API只有一种工厂方法可以创建`Session`，并且它需要事务和确认模式的值。在托管环境中，设置这些值是环境的事务基础结构的责任，因此，供应商对JMS Connection的包装将忽略这些值。当您使用`JmsTemplate` 在非托管环境中，您可以通过使用属性来指定这些值`sessionTransacted`和`sessionAcknowledgeMode`。当使用 `PlatformTransactionManager`with时`JmsTemplate`，模板总是被赋予事务性JMS `Session`。

### 4.2。发送信息（ Sending a Message）

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

在前面的示例中，`JmsTemplate`通过将引用传递给来构造 `ConnectionFactory`。作为替代方案，提供了零参数构造函数，该构造函数 `connectionFactory`可以用于以JavaBean样式（使用`BeanFactory`Java或纯Java代码）构造实例。另外，考虑从Spring的`JmsGatewaySupport`便捷基类派生，该类为JMS配置提供了预构建的bean属性。

该`send(String destinationName, MessageCreator creator)`方法使您可以使用目标的字符串名称发送消息。如果这些名称已在JNDI中注册，则应将`destinationResolver`模板的属性设置为的实例 `JndiDestinationResolver`。

如果您创建`JmsTemplate`并指定了默认目标，则会向该目标 `send(MessageCreator c)`发送一条消息。

#### 4.2.1。使用消息转换器（ Using Message Converters）

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

```
MapMessage = {
    标头= { 
        ...标准标头... 
        CorrelationID = {123-00001} 
    }
    属性= { 
        AccountID = {Integer：1234} 
    }
    字段= { 
        Name = {String：Mark} 
        Age = {Integer：47} 
    } 
}
```

#### 4.2.2。使用`SessionCallback`和`ProducerCallback`

尽管发送操作涵盖了许多常见的使用场景，但是您有时可能希望在JMS`Session`或上执行多个操作`MessageProducer`。的 `SessionCallback`和`ProducerCallback`暴露的JMS`Session`和`Session`/ `MessageProducer`分别对，。这些`execute()`方法将`JmsTemplate`运行这些回调方法。

### 4.3。接收讯息（Receiving a Message）

这描述了如何在Spring中使用JMS接收消息。

#### 4.3.1。同步接收（Synchronous Reception）

虽然JMS通常与异步处理相关联，但是您可以同步使用消息。重载的`receive(..)`方法提供了此功能。在同步接收期间，调用线程将阻塞，直到消息可用为止。这可能是危险的操作，因为调用线程可能会无限期地被阻塞。该`receiveTimeout`属性指定接收者在放弃等待消息之前应该等待多长时间。

#### 4.3.2。异步接收：消息驱动的POJO（Asynchronous reception: Message-Driven POJOs）

|      | Spring还通过使用`@JmsListener` 注释支持带注释的侦听器端点，并提供了开放的基础结构以编程方式注册端点。到目前为止，这是设置异步接收器的最便捷方法。有关更多详细信息，请参见[启用侦听器端点注释](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-annotated-support)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

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

#### 4.3.3。使用`SessionAwareMessageListener` 接口

该`SessionAwareMessageListener`接口是特定于Spring的接口，该接口提供与JMS`MessageListener`接口相似的协定，但还允许消息处理方法访问`Session`从其`Message`接收JMS的JMS 。以下清单显示了`SessionAwareMessageListener`接口的定义：

```java
package org.springframework.jms.listener;

public interface SessionAwareMessageListener {

    void onMessage(Message message, Session session) throws JMSException;
}
```

`MessageListener`如果希望MDP能够响应任何接收到的消息（通过使用 方法中`Session`提供的信息），则可以选择让MDP实现此接口（优先于标准JMS接口`onMessage(Message, Session)`）。Spring附带的所有消息侦听器容器实现都支持实现`MessageListener`或 `SessionAwareMessageListener`接口的MDP 。`SessionAwareMessageListener`需要注意的是，实现该类的类 随后将它们通过接口绑定到Spring。是否使用它的选择完全取决于您是应用程序开发人员还是架构师。

请注意， 接口的`onMessage(..)`方法`SessionAwareMessageListener`throws `JMSException`。与标准JMS`MessageListener` 接口相反，使用该`SessionAwareMessageListener`接口时，客户端代码负责处理任何引发的异常。

#### 4.3.4。使用`MessageListenerAdapter`

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

特别要注意，`MessageDelegate`接口（ `DefaultMessageDelegate`该类）的先前实现是如何完全没有JMS依赖性的。这确实是一个POJO，我们可以通过以下配置将其制作为MDP：

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

以下清单显示了实现该`TextMessageDelegate`接口的类：

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

需要注意的是，如果`messageListener`接收到一个JMS`Message`以外的类型的`TextMessage`，一个`IllegalStateException`被抛出（随后吞咽）。`MessageListenerAdapter`该类的另一个功能是，`Message`如果处理程序方法返回非空值，则可以自动发送回响应。考虑以下接口和类：

```java
public interface ResponsiveTextMessageDelegate {

    // notice the return type...
    String receive(TextMessage message);
}
public class DefaultResponsiveTextMessageDelegate implements ResponsiveTextMessageDelegate {
    // implementation elided for clarity...
}
```

如果将`DefaultResponsiveTextMessageDelegate`a与a结合使用 `MessageListenerAdapter`，则从该`'receive(..)'`方法的执行返回的任何非null值（在默认配置下）都将转换为a `TextMessage`。`TextMessage`然后将结果发送到原始`Destination`JMS`Reply-To`属性中定义的（如果存在）`Message`或（如果已配置）默认`Destination`设置的JMS属性`MessageListenerAdapter`。如果未`Destination`找到，`InvalidDestinationException`则引发an （请注意，该异常不会被吞没，并且会在调用堆栈中传播）。

#### 4.3.5。处理事务中的消息（ Processing Messages Within Transactions）

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

|      | 我们的交易经理。 |
| ---- | ---------------- |
|      |                  |

### 4.4。支持JCA消息端点（Support for JCA Message Endpoints）

从2.5版开始，Spring还提供了对基于JCA的 `MessageListener`容器的支持。在`JmsMessageEndpointManager`尝试自动确定`ActivationSpec`从提供的类名 `ResourceAdapter`类名。因此，通常可以提供Spring的generic `JmsActivationSpecConfig`，如以下示例所示：

```xml
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

|      | 基于JCA的消息端点管理与EJB 2.1消息驱动Bean非常相似。它使用相同的基础资源提供者合同。与EJB 2.1 MDB一样，您也可以在Spring上下文中使用JCA提供程序支持的任何消息侦听器接口。尽管如此，Spring为JMS提供了显式的“便利”支持，因为JMS是JCA端点管理协定中最常用的端点API。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.5。注释驱动的侦听器端点（Annotation-driven Listener Endpoints）

异步接收消息的最简单方法是使用带注释的侦听器端点基础结构。简而言之，它使您可以将托管Bean的方法公开为JMS侦听器端点。以下示例显示了如何使用它：

```java
@Component
public class MyService {

    @JmsListener(destination = "myDestination")
    public void processOrder(String data) { ... }
}
```

前面示例的想法是，只要消息可用，就会 相应地调用`javax.jms.Destination` `myDestination`该`processOrder`方法（在这种情况下，使用JMS消息的内容，类似于[`MessageListenerAdapter`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-receiving-async-message-listener-adapter) 提供的内容）。

带注释的终结点基础结构通过使用，为每种带注释的方法在幕后创建一个消息侦听器容器`JmsListenerContainerFactory`。这样的容器不会针对应用程序上下文进行注册，但是可以通过使用`JmsListenerEndpointRegistry`Bean方便地定位用于管理目的。

|      | `@JmsListener`是Java 8上的可重复注释，因此您可以通过`@JmsListener` 向其添加其他声明来将多个JMS目标与同一方法相关联。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 4.5.1。启用侦听器端点注释（ Enable Listener Endpoint Annotations）

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

如果您更喜欢[XML配置](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-namespace)，则可以使用`<jms:annotation-driven>` 元素，如以下示例所示：

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

#### 4.5.2。程序化端点注册（Programmatic Endpoint Registration）

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

#### 4.5.3。带注释的端点方法签名（Annotated Endpoint Method Signature）

到目前为止，我们已经`String`在端点中注入了一个简单的方法，但实际上它可以具有非常灵活的方法签名。在以下示例中，我们将其重写为`Order`使用自定义标头注入：

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

#### 4.5.4。反应管理（Response Management）

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

|      | 如果您有多个带有 注释的`@JmsListener`方法，则还可以将`@SendTo`注释放在类级别以共享默认的答复目标。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

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

如果需要在运行时计算响应目标，则可以将响应封装在一个`JmsResponse`实例中，该实例还提供要在运行时使用的目标。我们可以如下重写前一个示例：

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

最后，如果您需要为响应指定一些QoS值，例如优先级或生存时间，则可以进行相应的配置`JmsListenerContainerFactory`，如以下示例所示：

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

### 4.6。JMS命名空间支持（JMS Namespace Support）

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

|      | 引用JMS模式。 |
| ---- | ------------- |
|      |               |

这个命名空间由三个顶级元素：<annotation-driven/>，<listener-container/> 和<jca-listener-container/>。<annotation-driven/>启用注释驱动的侦听器端点的使用。<listener-container/>并<jca-listener-container/> 定义共享的侦听器容器配置，并且可以包含<listener/>子元素。以下示例显示了两个侦听器的基本配置：

```xml
<jms:listener-container>

    <jms:listener destination="queue.orders" ref="orderService" method="placeOrder"/>

    <jms:listener destination="queue.confirmations" ref="confirmationLogger" method="log"/>

</jms:listener-container>
```

前面的示例等效于创建两个不同的侦听器容器bean定义和两个不同的`MessageListenerAdapter`bean定义，如[Using中`MessageListenerAdapter`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms-receiving-async-message-listener-adapter)所示。除了前面示例中显示的属性之外，该`listener`元素还可以包含几个可选属性。下表描述了所有可用属性：

| 属性                   | 描述                                                         |
| :--------------------- | :----------------------------------------------------------- |
| `id`                   | 托管侦听器容器的Bean名称。如果未指定，将自动生成Bean名称。   |
| `destination` （需要） | 此侦听器的目标名称，通过`DestinationResolver` 策略解析。     |
| `ref` （需要）         | 处理程序对象的bean名称。                                     |
| `method`               | 要调用的处理程序方法的名称。如果`ref`属性指向a`MessageListener` 或Spring `SessionAwareMessageListener`，则可以忽略此属性。 |
| `response-destination` | 向其发送响应消息的默认响应目标的名称。在请求消息不包含`JMSReplyTo`字段的情况下适用。此目标的类型由listener-container的 `response-destination-type`属性确定。请注意，这仅适用于具有返回值的侦听器方法，为此，每个结果对象都将转换为响应消息。 |
| `subscription`         | 持久订阅的名称（如果有）。                                   |
| `selector`             | 此侦听器的可选消息选择器。                                   |
| `concurrency`          | 要启动此侦听器的并发会话或使用者的数量。此值可以是表示最大数的简单数字（例如`5`），也可以是表示下限和上限的范围（例如`3-5`）。请注意，指定的最小值仅是一个提示，在运行时可能会被忽略。默认值为容器提供的值。 |

该`<listener-container/>`元素还接受几个可选属性。这允许自定义各种策略（例如`taskExecutor`和 `destinationResolver`）以及基本的JMS设置和资源引用。通过使用这些属性，您可以定义高度自定义的侦听器容器，同时仍然受益于命名空间的便利性。

您可以`JmsListenerContainerFactory`通过指定`id`通过`factory-id`属性公开的Bean来自动将这些设置显示为，如以下示例所示：

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

下表描述了所有可用属性。有关[`AbstractMessageListenerContainer`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/jms/listener/AbstractMessageListenerContainer.html) 各个属性的更多详细信息，请参见和其具体子类的类级javadoc 。Javadoc还讨论了事务选择和消息重新交付方案。

| 属性                        | 描述                                                         |
| :-------------------------- | :----------------------------------------------------------- |
| `container-type`            | 此侦听器容器的类型。可用选项为`default`，`simple`， `default102`或者`simple102`（默认选项`default`）。 |
| `container-class`           | 自定义侦听器容器实现类，作为完全限定的类名称。根据属性，默认值为Spring的标准`DefaultMessageListenerContainer`或 。`SimpleMessageListenerContainer``container-type` |
| `factory-id`                | 将此元素定义的设置公开为`JmsListenerContainerFactory` 带有指定的，`id`以便它们可以与其他端点重用。 |
| `connection-factory`        | 对JMS `ConnectionFactory`Bean的引用（默认Bean名称为 `connectionFactory`）。 |
| `task-executor`             | `TaskExecutor`对JMS侦听器调用程序的Spring的引用。            |
| `destination-resolver`      | `DestinationResolver`对解决JMS`Destination`实例的策略的引用。 |
| `message-converter`         | `MessageConverter`对将JMS消息转换为侦听器方法参数的策略的参考。默认值为`SimpleMessageConverter`。 |
| `error-handler`             | 引用一种`ErrorHandler`策略，用于处理在执行期间可能发生的任何未捕获的异常`MessageListener`。 |
| `destination-type`          | 这个监听器的JMS目的地类型：`queue`，`topic`，`durableTopic`，`sharedTopic`，或`sharedDurableTopic`。这潜在地使`pubSubDomain`，`subscriptionDurable` 与`subscriptionShared`所述容器的性质。默认值为`queue`（禁用这三个属性）。 |
| `response-destination-type` | 响应的JMS目标类型：`queue`或`topic`。默认值为该`destination-type`属性的值 。 |
| `client-id`                 | 此侦听器容器的JMS客户端ID。使用持久订阅时必须指定它。        |
| `cache`                     | 对JMS资源的高速缓存级别：`none`，`connection`，`session`，`consumer`，或 `auto`。默认情况下（`auto`），`consumer`除非指定了外部事务管理器，否则缓存级别是有效的-在这种情况下，有效的默认值将是`none`（假设Java EE风格的事务管理，其中给定的ConnectionFactory是可识别XA的池）。 |
| `acknowledge`               | 本机JMS确认模式：`auto`，`client`，`dups-ok`，或`transacted`。值`transacted`激活本地交易`Session`。或者，您可以指定`transaction-manager`属性，如表中稍后所述。默认值为`auto`。 |
| `transaction-manager`       | 对外部`PlatformTransactionManager`（通常是基于XA的事务协调器，例如Spring的`JtaTransactionManager`）的引用。如果未指定，则使用本机确认（请参阅`acknowledge`属性）。 |
| `concurrency`               | 每个侦听器启动的并发会话或使用者的数量。它可以是表示最大数的简单数字（例如`5`），也可以是表示下限和上限的范围（例如`3-5`）。请注意，指定的最小值只是一个提示，在运行时可能会被忽略。默认值为`1`。如果`1`主题侦听器或队列顺序很重要，则应将并发限制在一定范围内。考虑将其提高到一般队列。 |
| `prefetch`                  | 加载到单个会话中的最大消息数。请注意，增加此数字可能会导致并发消费者饥饿。 |
| `receive-timeout`           | 用于接听电话的超时时间（以毫秒为单位）。默认值为`1000`（一秒钟）。`-1`表示没有超时。 |
| `back-off`                  | 指定`BackOff`用于计算两次恢复尝试间隔的实例。如果`BackOffExecution`实现返回`BackOffExecution#STOP`，则侦听器容器不会进一步尝试恢复。`recovery-interval` 设置此属性时，将忽略该值。缺省值为a `FixedBackOff`，间隔为5000毫秒（即5秒）。 |
| `recovery-interval`         | 指定两次恢复尝试之间的时间间隔（以毫秒为单位）。它提供了一种方便的方法来创建`FixedBackOff`具有指定间隔的。有关更多恢复选项，请考虑指定一个`BackOff`实例。缺省值为5000毫秒（即5秒）。 |
| `phase`                     | 该容器应在其中启动和停止的生命周期阶段。值越低，此容器启动越早，而其停止越晚。默认值为 `Integer.MAX_VALUE`，表示容器尽可能晚地启动，并尽快停止。 |

使用`jms`模式支持配置基于JCA的侦听器容器非常相似，如以下示例所示：

```xml
<jms:jca-listener-container resource-adapter="myResourceAdapter"
        destination-resolver="myDestinationResolver"
        transaction-manager="myTransactionManager"
        concurrency="10">

    <jms:listener destination="queue.orders" ref="myMessageListener"/>

</jms:jca-listener-container>
```

下表描述了JCA变体的可用配置选项：

| 属性                        | 描述                                                         |
| :-------------------------- | :----------------------------------------------------------- |
| `factory-id`                | 将此元素定义的设置公开为`JmsListenerContainerFactory` 带有指定的，`id`以便它们可以与其他端点重用。 |
| `resource-adapter`          | 对JCA `ResourceAdapter`bean的引用（默认bean名称为 `resourceAdapter`）。 |
| `activation-spec-factory`   | 对的引用`JmsActivationSpecFactory`。默认设置是自动检测JMS提供程序及其`ActivationSpec`类（请参阅参考资料[`DefaultJmsActivationSpecFactory`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/jms/listener/endpoint/DefaultJmsActivationSpecFactory.html)）。 |
| `destination-resolver`      | `DestinationResolver`解决JMS的策略的参考`Destinations`。     |
| `message-converter`         | `MessageConverter`对将JMS消息转换为侦听器方法参数的策略的参考。默认值为`SimpleMessageConverter`。 |
| `destination-type`          | 这个监听器的JMS目的地类型：`queue`，`topic`，`durableTopic`，`sharedTopic`。或`sharedDurableTopic`。这潜在地使`pubSubDomain`，`subscriptionDurable`和`subscriptionShared`容器的性质。默认值为`queue`（禁用这三个属性）。 |
| `response-destination-type` | 响应的JMS目标类型：`queue`或`topic`。默认值为该`destination-type`属性的值 。 |
| `client-id`                 | 此侦听器容器的JMS客户端ID。使用持久订阅时需要指定它。        |
| `acknowledge`               | 本机JMS确认模式：`auto`，`client`，`dups-ok`，或`transacted`。值`transacted`激活本地交易`Session`。或者，您可以指定`transaction-manager`稍后描述的属性。默认值为`auto`。 |
| `transaction-manager`       | 对Spring`JtaTransactionManager`或a的 引用，`javax.transaction.TransactionManager`用于为每个传入消息启动XA事务。如果未指定，则使用本机确认（请参阅 `acknowledge`属性）。 |
| `concurrency`               | 每个侦听器启动的并发会话或使用者的数量。它可以是表示最大数的简单数字（例如`5`），也可以是表示下限和上限的范围（例如`3-5`）。请注意，指定的最小值只是一个提示，通常在运行时使用JCA侦听器容器时将被忽略。预设值为1。 |
| `prefetch`                  | 加载到单个会话中的最大消息数。请注意，增加此数字可能会导致并发消费者饥饿。 |