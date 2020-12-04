1. Spring Web MVC 框架

Spring Web MVC是**基于Servlet API**构建的原始Web框架，从一开始就已包含在Spring Framework中。 正式名称“ Spring Web MVC”来自其源模块的名称 ([`spring-webmvc`](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc))，但**通常称为“ Spring MVC”**。

`与`Spring Web MVC并行，Spring Framework 5.0引入了一个**反应式堆栈Web框架**，其名称“ Spring WebFlux”也基于其源模块([`spring-webflux`](https://github.com/spring-projects/spring-framework/tree/master/spring-webflux))。 本节介绍Spring Web MVC。 [下一节](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#spring-web-reactive)将介绍Spring WebFlux。 

有关基线信息以及与Servlet容器和Java EE版本范围的兼容性，请参见Spring Framework  [Wiki](https://github.com/spring-projects/spring-framework/wiki/Spring-Framework-Versions)。

## 1.1.  DispatcherServlet 分发器



[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-dispatcher-handler)

与其他许多Web框架一样，Spring MVC==围绕前端控制器模式==进行设计，在该模式下，中央Servlet即DispatcherServlet提供了用于请求处理的共享算法，而实际工作是由可配置的委托组件执行的。 该模型非常灵活，并支持多种工作流程。 

与任何Servlet一样，都需要使用Java配置或在web.xml中根据Servlet规范声明和映射DispatcherServlet。 反过来，DispatcherServlet使用Spring配置发现请求映射，视图解析，异常处理等所需的委托组件,更多请参考[特殊类的类型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet-special-bean-types)。 

以下Java配置示例注册并初始化DispatcherServlet，该容器由Servlet容器自动检测到（请参阅 [Servlet Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-container-config)）：

Java：

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register(AppConfig.class);

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic registration = servletContext.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```



| ![image-20201130175103639](Spring%20Web%20MVC.assets/image-20201130175103639.png) | ***注意*** :  除了直接使用ServletContext API外，您还可以扩展AbstractAnnotationConfigDispatcherServletInitializer并覆盖特定方法（请参见[Context Hierarchy](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet-context-hierarchy)下的示例）。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

以下web.xml配置示例注册并初始化DispatcherServlet：

```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>

</web-app>
```

| ![image-20201130175115567](Spring%20Web%20MVC.assets/image-20201130175115567.png) | ***注意*** ：Spring Boot==遵循不同的初始化顺序==。 Spring Boot并没有陷入Servlet容器的生命周期，而是使用Spring配置来引导自身和嵌入式Servlet容器。 在Spring配置中检测到过滤器和Servlet声明，并在==Servlet容器中注册==。 有关更多详细信息，请参见[Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-embedded-container)。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |



### 1.1.1. Context Hierarchy 上下文层次

DispatcherServlet希望**WebApplicationContext**（纯ApplicationContext的扩展）为其自身的配置。 WebApplicationContext具有**指向ServletContext和与其关联的Servlet的链接**。它还绑定到ServletContext，以便应用程序可以在RequestContextUtils上使用**静态方法**来查找WebApplicationContext（如果需要访问它们）。

对于许多应用程序来说，拥有一个WebApplicationContext很简单并且足够。也**可能具有上下文层次结构**，其中一个根WebApplicationContext在多个DispatcherServlet（或其他Servlet）实例之间**共享**，每个实例都有其自己的**子WebApplicationContext配置**。有关上下文层次结构功能的更多信息，请参见[ApplicationContext的其他功能](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction)。

根WebApplicationContext通常包含**基础结构bean**，例如需要在多个Servlet实例之间共享的数据存储库和业务服务。这些Bean是有效继承的，可以在Servlet特定子WebApplicationContext中重写（即重新声明），该子WebApplicationContext通常包含给定Servlet本地的Bean。下图显示了这种关系：

![image-20201130173413845](Spring%20Web%20MVC.assets/image-20201130173413845.png)

下面的示例配置一个WebApplicationContext层次结构：

Java:

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { App1Config.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/app1/*" };
    }
}
```

| ![image-20201130174957967](Spring%20Web%20MVC.assets/image-20201130174957967.png) | ***注意*** ： 如果不需要应用程序上下文层次结构，则应用程序可以==通过getRootConfigClasses（）返回所有配置==，并从getServletConfigClasses（）返回null。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

以下示例显示了web.xml等效项：

```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app1</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app1-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app1</servlet-name>
        <url-pattern>/app1/*</url-pattern>
    </servlet-mapping>

</web-app>
```

| ![image-20201130174931690](Spring%20Web%20MVC.assets/image-20201130174931690.png) | ***注意*** :  如果不需要应用程序上下文层次结构，则应用程序可以仅配置“根”上下文，并将contextConfigLocation Servlet参数保留为空。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

### 1.1.2. Special Bean Types 特殊的Bean类型

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-special-bean-types)

DispatcherServlet委托给**特殊的bean**处理请求并呈现适当的响应。 所谓“特殊bean”，是指**实现框架协定的Spring管理对象实例**。 这些**通常带有内置合同**，但是您可以**自定义**它们的属性并**扩展或替换**它们。

下表列出了DispatcherServlet检测到的特殊bean：

|                         **Bean类型**                         | **说明**                                                     |
| :----------------------------------------------------------: | :----------------------------------------------------------- |
|                        HandlerMapping                        | 将请求与[拦截器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-handlermapping-interceptor)列表一起映射到处理程序，以进行预处理和后期处理。 映射基于某些条件，其细节因HandlerMapping实现而异。 <br />HandlerMapping的两个主要实现是**RequestMappingHandlerMapping**（支持@RequestMapping带注释的方法）和**SimpleUrlHandlerMapping**（维护向处理程序的URI路径模式的显式注册）。 |
|                        HandlerAdapter                        | **帮助DispatcherServlet调用映射**到请求的处理程序，而不管实际上如何调用该处理程序。 例如，调用带注释的控制器需要解析注释。 HandlerAdapter的主要目的是使DispatcherServlet免受此类细节的影响。 |
| [`HandlerExceptionResolver`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-exceptionhandlers) | **解决异常的策略**，可能将它们映射到处理程序，HTML错误视图或其他目标。请参阅[异常](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-exceptionhandlers)。 |
| [`ViewResolver`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-viewresolver) | 解析从处理程序返回的实际基于**字符串**的基于**逻辑**的视图名称，以实际的视图呈现给响应。请参阅查看[分辨率](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-viewresolver)和[查看技术](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view)。 |
| [`LocaleResolver`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-localeresolver), [LocaleContextResolver](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-timezone) | 解析客户正在使用的**语言环境以及可能的时区**，以便能够提供国际化的视图。请参阅[区域设置](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-localeresolver)。 |
| [`ThemeResolver`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-themeresolver) | 解决Web应用程序可以使用的主题，例如，以提供个性化的布局。请参阅[主题](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-themeresolver)。 |
| [`MultipartResolver`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multipart) | 借助一些多部分解析库来解析多部分请求的抽象（例如，浏览器表单文件上传）。请参见[多部分分解器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multipart)。 |
| [`FlashMapManager`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-flash-attributes) | **存储和检索**“输入”和“输出” FlashMap，可用于将属性从一个请求传递到另一个请求，通常跨重定向。请参见[Flash属性](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-flash-attributes)。 |

### 1.1.3. Web MVC Config (Web MVC配置)

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-framework-config)

应用程序可以声明处理请求所需的[特殊Bean类型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet-special-bean-types)中列出的基础结构Bean。 DispatcherServlet检查每个特殊bean的WebApplicationContext。 如果没有匹配的bean类型，它将使用[`DispatcherServlet.properties`](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/resources/org/springframework/web/servlet/DispatcherServlet.properties)中列出的==默认类型==。 

在大多数情况下， [MVC Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config)是最佳起点。 它使用Java或XML声明所需的bean，并提供更高级别的配置回调API对其进行自定义。

| ![image-20201130182010866](Spring%20Web%20MVC.assets/image-20201130182010866.png) | Spring Boot依靠==MVC Java==配置来配置Spring MVC，并提供许多额外的方便选项。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

### 1.1.4. Servlet Config (Servlet配置)

在Servlet 3.0+环境中，您可以选择**以编程方式配置Servlet容器**，以作为**替代方案**或与web.xml文件**结合使用**。下面的示例注册一个DispatcherServlet：

Java:

```java
import org.springframework.web.WebApplicationInitializer;

public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet(appContext));
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }
}
```



WebApplicationInitializer是Spring MVC提供的**接口**，可确保检测到您的实现并将其==自动==用于初始化任何Servlet 3容器。 WebApplicationInitializer的抽象基类实现名为**AbstractDispatcherServletInitializer**，它通过覆盖**指定**Servlet映射和DispatcherServlet配置位置的方法，使注册DispatcherServlet更容易。

 对于使用基于**Java**的Spring配置的应用程序，建议这样做，如以下示例所示：

Java:

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { MyWebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

如果使用基于**XML**的Spring配置，则应直接从AbstractDispatcherServletInitializer进行扩展，如以下示例所示：

Java:

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        XmlWebApplicationContext cxt = new XmlWebApplicationContext();
        cxt.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");
        return cxt;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

AbstractDispatcherServletInitializer还提供了一种方便的方法来**添加Filter实例**，并将其**自动映射**到DispatcherServlet，如以下示例所示：

Java:

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    // ...

    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] {
            new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
    }
}
```

每个过滤器都会根据其具体类型添加一个默认名称，并**自动映射**到DispatcherServlet。

AbstractDispatcherServletInitializer的isAsyncSupported受保护方法提供了一个位置，以==启用==对DispatcherServlet及其映射的所有过滤器的**异步支持**。 默认情况下，此标志设置为true。 

最后，如果您需要进一步自定义DispatcherServlet本身，则可以覆盖createDispatcherServlet方法。

### 1.1.5. Processing 处理

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-dispatcher-handler-sequence)

DispatcherServlet处理请求的方式如下：

- **搜索WebApplicationContext**并将其绑定在请求中，作为控制器和流程中其他元素可以使用的==属性==。默认情况下，它绑定在DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE键下。
- **语言环境解析器绑定到请求**，以使流程中的元素解析在处理请求（呈现视图，准备数据等）时要使用的语言环境。 如果不需要语言环境解析，则不需要语言环境解析器。
- **主题解析器绑定到请求**，以使诸如视图之类的元素确定要使用的主题。如果不使用主题，则可以将其忽略。
- 如果**指定多部分文件解析器**，则将检查请求中是否有多部分。 如果找到多部分，则将该请求包装在MultipartHttpServletRequest中，以供流程中的其他元素进一步处理。 有关多部分处理的更多信息，请参见[Multipart Resolver](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multipart) 。
- **搜索适当的处理程序**。 如果找到处理程序，则将运行与该处理程序（预处理器，后处理器和控制器）关联的执行链，以准备要渲染的模型。 另外，对于==带注释的控制器，可以呈现响应==（在HandlerAdapter中），而不是返回视图。
- 如果**返回模型**，则***呈现视图***。如果未返回任何模型（可能是由于预处理器或后处理器拦截了该请求，可能出于安全原因），则不会呈现任何视图，因为该请求可能已被满足。

WebApplicationContext中声明的==HandlerExceptionResolver Bean==用于解决在请求处理期间引发的异常。 这些异常解析器**允许定制逻辑**以解决异常。 有关更多详细信息，请参见 [异常](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-exceptionhandlers)。 

Spring DispatcherServlet还支持Servlet API所指定的**last-modification-date**的返回。 确定特定请求的最后修改日期的过程很简单：DispatcherServlet查找适当的处理程序映射，并测试找到的处理程序是否实现了LastModified接口。 如果是这样，则将LastModified接口的**long getLastModified（request）方法的值**返回给客户端。

您可以通过将Servlet初始化参数（init-param元素）添加到web.xml文件中的Servlet声明中，来定制各个DispatcherServlet实例。 下表列出了受支持的参数：

**表1. DispatcherServlet初始化参数**

|             **参数**             | **说明**                                                     |
| :------------------------------: | :----------------------------------------------------------- |
|          `contextClass`          | 实现**ConfigurableWebApplicationContext**的类，将由此Servlet实例化并在本地配置。默认情况下，使用**XmlWebApplicationContext**。 |
|     `contextConfigLocation`      | 传递给上下文实例的字符串（由contextClass指定），以指示可以在哪里找到上下文。 该字符串可能包含多个字符串（使用逗号作为分隔符）以支持多个上下文。 对于具有两次定义的bean的多个上下文位置，以**最新位置**为准。 |
|           `namespace`            | **WebApplicationContext的命名空间**。默认为[servlet-name] -servlet。 |
| `throwExceptionIfNoHandlerFound` | 在找不到请求的处理程序时是否引发NoHandlerFoundException。 然后可以使用**HandlerExceptionResolver**捕获该异常（例如，通过使用@ExceptionHandler控制器方法），然后将其作为其他任何异常进行处理。 <br /><br />**默认情况下，它设置为false**，在这种情况下，DispatcherServlet将响应状态设置为404（NOT_FOUND），而不会引发异常。 <br /><br />请注意，如果还配置了[默认servlet处理](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-default-servlet-handler)，则始终将未解决的请求转发到**默认servlet**，并且永远不会引发404。 |

### 1.1.6. Interception 拦截

**所有**HandlerMapping实现都支持处理程序拦截器，当您要==将特定功能应用于某些请求时==（例如，检查主体），该拦截器很有用。 拦截器必须使用三种方法从org.springframework.web.servlet包中**实现HandlerInterceptor**，这三种方法应具有足够的灵活性来执行各种预处理和后处理：

- `preHandle(..)`: **在实际的处理程序运行之前**
- `postHandle(..)`: **处理程序运行后**
- `afterCompletion(..)`: **完整的请求完成后**

preHandle（..）方法返回一个布尔值。您可以使用此方法来**中断或继续执行链的处理**。当此方法返回true时，处理程序执行链将继续。当返回false时，DispatcherServlet假定拦截器本身已经处理了请求（例如，渲染了适当的视图），并且**不会继续执行其他**拦截器和执行链中的实际处理程序。

有关如何配置拦截器的示例，请参见MVC配置部分中的[拦截器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-interceptors) 。您还可以通过在各个HandlerMapping实现上使用setter直接注册它们。

请注意，在@ResponseBody和ResponseEntity方法中，postHandle的用处不大，在HandlerAdapter内和postHandle之前写入和提交响应的@ResponseBody和ResponseEntity方法。这意味着对响应进行任何更改为时已晚，例如添加额外的标头。对于此类情况，您可以**实现ResponseBodyAdvice**并将其声明为[Controller Advice](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-controller-advice) Bean，或直接在**RequestMappingHandlerAdapter**上对其进行配置。

### 1.1.7. Exceptions 异常

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-dispatcher-exceptions)

如果异常在请求映射期间发生或从请求处理程序（例如@Controller）抛出，则DispatcherServlet**委托给HandlerExceptionResolver Bean链**来解决该异常并提供替代处理，通常是错误响应。 

下表列出了可用的HandlerExceptionResolver实现：

**表2. HandlerExceptionResolver实现**

|                       处理器异常解析器                       | 描述                                                         |
| :----------------------------------------------------------: | :----------------------------------------------------------- |
|               `SimpleMappingExceptionResolver`               | **异常类名称和错误视图名称之间的映射**。对于在浏览器应用程序中呈现错误页面很有用。 |
| [`DefaultHandlerExceptionResolver`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/mvc/support/DefaultHandlerExceptionResolver.html) | 解决Spring MVC引发的异常，并将其**映射**到HTTP状态代码。另请参见备用ResponseEntityExceptionHandler和[REST API异常](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-rest-exceptions)。 |
|              `ResponseStatusExceptionResolver`               | 使用@ResponseStatus批注解决异常，并根据批注中的值将其**映射到HTTP状态代码。** |
|             `ExceptionHandlerExceptionResolver`              | 通过在@Controller或@ControllerAdvice类中调用**@ExceptionHandler**方法来解决异常。请参见[@ExceptionHandler方法](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler)。 |

#### Chain of Resolvers 解析器链

您可以通过在Spring配置中声明多个HandlerExceptionResolver bean并根据需要设置其**order属性**来形成异常解析器链。 order属性**越高**，异常解析器的定位就**越晚**。 

HandlerExceptionResolver的约定指定它可以返回：

- 指向错误视图的ModelAndView。
- 如果在解析器中处理了异常，则为空的ModelAndView。
- 如果该异常仍未解决，则为null，以供后续解析器尝试；如果该异常仍在末尾，则允许其==冒泡到Servlet容器==。

[MVC Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config)自动为默认的Spring MVC异常，@ ResponseStatus注释的异常以及对@ExceptionHandler方法的支持声明**内置解析器**。您可以自定义该列表或替换它。

#### Container Error Page 容器错误页面

如果任何HandlerExceptionResolver都无法解决异常，因此该异常可以**传播**，或者如果响应状态**设置为错误状态**（即4xx，5xx），则Servlet容器可以在HTML中呈现默认错误页面。 要自定义容器的默认错误页面，可以在web.xml中声明错误页面映射。 以下示例显示了如何执行此操作：

```xml
<error-page>
    <location>/error</location>
</error-page>
```

给定前面的示例，当异常冒出气泡或响应具有错误状态时，Servlet容器在容器内向配置的URL（例如`/error`）进行**ERROR调度**。然后由进行处理`DispatcherServlet`，可能将其映射到`@Controller`，可以实现该模型以使用模型返回==错误视图名称或呈现JSON响应==，如以下示例所示：

Java:

```java
@RestController
public class ErrorController {

    @RequestMapping(path = "/error")
    public Map<String, Object> handle(HttpServletRequest request) {
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("status", request.getAttribute("javax.servlet.error.status_code"));
        map.put("reason", request.getAttribute("javax.servlet.error.message"));
        return map;
    }
}
```

| ![image-20201201081455294](Spring%20Web%20MVC.assets/image-20201201081455294.png) | Servlet API没有提供在Java中创建错误页面映射的方法。但是，您可以同时使用WebApplicationInitializer和最小的web.xml。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

### 1.1.8. View Resolution 查看分辨率

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-viewresolution)

Spring MVC定义了**`ViewResolver`**和**`View`**接口，使您可以在浏览器中呈现模型，而无需将您绑定到特定的视图技术。`ViewResolver` 提供**视图名称**和实际视图之间的**映射**。`View`在移交给特定的视图技术之前，先解决数据准备问题。

下表提供了有关`ViewResolver`层次结构的更多详细信息：

**表3. ViewResolver实现**

|            视图解析器            | 描述                                                         |
| :------------------------------: | :----------------------------------------------------------- |
|  `AbstractCachingViewResolver`   | `AbstractCachingViewResolver`它们解析的缓存视图实例的子类。缓存可以提高某些视图技术的性能。您可以通过将==`cache`==属性设置为来关闭缓存`false`。此外，如果必须在运行时刷新某个视图（例如，当修改FreeMarker模板时），则可以使用该`removeFromCache(String viewName, Locale loc)`方法。 |
|      `UrlBasedViewResolver`      | `ViewResolver`接口的简单实现会影响将逻辑视图名称**直接解析为URL**而没有显式映射定义。如果您的逻辑名称以直接的方式与视图资源的名称匹配，而不需要任意映射，则这是适当的。 |
|  `InternalResourceViewResolver`  | 的方便子类`UrlBasedViewResolver`支持`InternalResourceView`（实际上，Servlet和JSP）和子类，如`JstlView`和`TilesView`。您可以使用来为此解析器生成的所有视图指定视图类`setViewClass(..)`。有关[`UrlBasedViewResolver`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/reactive/result/view/UrlBasedViewResolver.html) 详细信息，请参见javadoc。 |
|     `FreeMarkerViewResolver`     | 方便的子类的`UrlBasedViewResolver`支持`FreeMarkerView`和他们的自定义子类。 |
| `ContentNegotiatingViewResolver` | `ViewResolver`基于**请求文件名**或**`Accept`头解析视图的接口**的实现。请参阅[内容协商](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multiple-representations)。 |
|      `BeanNameViewResolver`      | `ViewResolver`在当前应用程序上下文中将**视图名称**解释为Bean名称的接口的实现。这是一个非常灵活的变体，它允许根据不同的视图名称来混合和匹配不同的视图类型。每个此类`View`都可以定义为bean，例如在XML或配置类中。 |

#### Handling 处理方式

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-viewresolution-handling)

您可以通过声明多个解析器bean以及必要时通过设置`order`属性以**指定顺序**来链接视图解析器。请记住，order属性**越高**，视图解析器在链中的定位就**越晚**。

ViewResolver的协定指定它可以返回null，以指示找不到该视图。但是，对于JSP和`InternalResourceViewResolver`，**确定JSP是否存在的唯一方法**是通过进行调度 `RequestDispatcher`。因此，您必须始终将`InternalResourceViewResolver` View解析器的总体顺序配置为**末尾**。

配置**视图分辨率**就像将`ViewResolver`bean添加到Spring配置中一样简单。所述[MVC配置](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config)提供了一种专用配置API [视图解析器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-view-resolvers)和用于将逻辑较少 [视图控制器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-view-controller)，其可用于HTML模板呈现有用而不控制器逻辑。

#### Redirecting 重定向

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-redirecting-redirect-prefix)

`redirect:`视图名称中的==特殊前缀==使您可以执行重定向。的 `UrlBasedViewResolver`（和它的子类）识别为是需要重定向的指令。视图名称的其余部分是**重定向URL**。

最终效果与控制器返回RedirectView的效果相同，但是现在控制器本身可以**根据逻辑视图名称**进行操作。逻辑视图名称（例如`redirect:/myapp/some/resource`）相对于当前Servlet上下文`redirect:https://myhost.com/some/arbitrary/path` 重定向，而名称（例如）重定向到绝对URL。

请注意，如果使用注释控制器方法`@ResponseStatus`，则**注释值优先于设置的响应状态**`RedirectView`。

#### Forwarding 转寄

您还可以`forward:`对视图名称使用**特殊的前缀**，这些视图名称最终由==`UrlBasedViewResolver`和子类解析==。这将创建一个 `InternalResourceView`，并执行一个`RequestDispatcher.forward()`。因此，此前缀在`InternalResourceViewResolver`和 `InternalResourceView`（对于JSP）中没有用，但是如果您使用另一种视图技术，但仍然希望强制转发由Servlet / JSP引擎处理的资源，则该前缀很有用。请注意，您也可以改为**链接多个视图解析器**。

#### Content Negotiation

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-multiple-representations)

[`ContentNegotiatingViewResolver`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/view/ContentNegotiatingViewResolver.html) 不会解析视图本身，而是委托其他视图解析器，并选择类似于客户端请求的表示形式的视图。可以根据`Accept`**标题或查询参数**（例如`"/path?format=pdf"`）来确定表示形式。

通过比较请求的媒体类型和与其相关联的媒体类型支持的媒体类型（也称为），`ContentNegotiatingViewResolver`选择合适的内容`View`来处理请求 。列表中的第一个具有**兼容**属性的列表将表示形式返回给客户端。如果链不能提供兼容的视图，那么将查询通过该属性指定的视图列表。后一个选项适用于单例，该单例可以呈现当前资源的适当表示形式，而与**逻辑视图名称无关**。Accept标头可以包含**通配符**（例如text / *），在这种情况下，其 是为相容的匹配。Content-Type	View  ViewResolvers	View	Content-Type	ViewResolver	DefaultViews	Views	Accept	text / *	ViewContent-Type	text / xml

有关配置详细信息，请参见[MVC Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config)下的[查看解析器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-view-resolvers)。

### 1.1.9. Locale 语言环境

正如Spring Web MVC框架所做的那样，Spring体系结构的大多数部分都支持国际化。`DispatcherServlet`使您可以使用客户端的语言环境**自动解析**消息。这是通过`LocaleResolver`对象完成的。

收到请求时，将`DispatcherServlet`查找**语言环境解析器**，如果找到一个，则尝试使用它来设置语言环境。通过使用该`RequestContext.getLocale()` 方法，您始终可以**检索**由语言环境解析器解析的语言环境。

除了自动的语言环境解析之外，您还可以在处理程序映射上附加一个拦截器（有关处理程序映射拦截[器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-handlermapping-interceptor)的更多信息，请参见拦截），以在**特定情况**下（例如，基于请求中的参数）更改语言环境。

语言环境解析器和拦截器在`org.springframework.web.servlet.i18n`程序包中定义， 并以常规方式在应用程序上下文中配置。Spring包含以下选择的语言环境解析器。

- [Time Zone](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-timezone)
- [Header Resolver](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-localeresolver-acceptheader)
- [Cookie Resolver](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-localeresolver-cookie)
- [Session Resolver](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-localeresolver-session)
- [Locale Interceptor](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-localeresolver-interceptor)

#### Time Zone 时区

除了获取客户的语言环境外，了解其时区通常也很有用。该`LocaleContextResolver`界面提供了扩展`LocaleResolver`，使解析程序可以提供更丰富的内容`LocaleContext`，其中可能包含**时区信息**。

如果可用，则`TimeZone`可以使用`RequestContext.getTimeZone()`方法获得 用户的信息。Spring的注册的任何日期/时间`Converter`和`Formatter`对象 都会==自动使用时区信息==`ConversionService`。

#### Header Resolver 标头解析器

此语言环境解析器检查`accept-language`客户端（例如，Web浏览器）发送的请求中的标头。通常，此**头字段**包含客户端操作系统的语言环境。请注意，此解析器**不支持时区信息**。

####  Cookie Resolver（Cookie解析器）

此语言环境解析器检查`Cookie`客户端上可能存在的，以查看是否 指定`Locale`或`TimeZone`。如果是这样，它将使用指定的详细信息。通过使用此**语言环境解析器的属性**，您可以指定cookie的名称以及**最长期限**。以下示例定义了一个`CookieLocaleResolver`：

```xml
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">

    <property name="cookieName" value="clientlanguage"/>

    <!-- in seconds. If set to -1, the cookie is not persisted (deleted when browser shuts down) -->
    <property name="cookieMaxAge" value="100000"/>

</bean>
```

下表描述了这些属性`CookieLocaleResolver`：

**表4. CookieLocaleResolver属性**

|      属性      |      默认       | 描述                                                         |
| :------------: | :-------------: | :----------------------------------------------------------- |
|  `cookieName`  |  类名+ LOCALE   | Cookie的名称                                                 |
| `cookieMaxAge` | Servlet容器默认 | Cookie在客户端上保留的**最长时间**。如果`-1`指定，则cookie将不会保留。它仅在客户端关闭浏览器之前可用。 |
|  `cookiePath`  |        /        | 将Cookie的可见性限制为网站的特定部分。当`cookiePath`被指定，cookie是**仅**对于该路径和它下面的路径。 |

#### Session Resolver 会话解析器

将`SessionLocaleResolver`允许您检索`Locale`和`TimeZone`从可能与用户的请求相关的会话。相反 `CookieLocaleResolver`，此策略将本地选择的语言环境设置存储在Servlet容器的中`HttpSession`。结果，这些设置对于每个会话都是**临时的**，因此在每个会话结束时会丢失。

请注意，与外部会话管理机制（例如Spring Session项目）**没有直接关系**。这将针对current`SessionLocaleResolver`评估和修改相应的`HttpSession`属性`HttpServletRequest`。

#### Locale Interceptor 区域拦截器

您可以通过将定义添加`LocaleChangeInterceptor`到其中之一 来**启用语言环境**的更改`HandlerMapping`。它检测到请求中的参数并相应地更改语言环境，`setLocale`从而`LocaleResolver`在调度程序的应用程序上下文中调用方法。下一个示例显示，调用`*.view`包含`siteLanguage`现在名为参数的参数的所有资源都会更改语言环境。因此，例如，对URL的请求`https://www.sf.net/home.view?siteLanguage=nl`将站点语言更改为荷兰语。以下示例显示如何拦截语言环境：

```xml
<bean id="localeChangeInterceptor"
        class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
    <property name="paramName" value="siteLanguage"/>
</bean>

<bean id="localeResolver"
        class="org.springframework.web.servlet.i18n.CookieLocaleResolver"/>

<bean id="urlMapping"
        class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="interceptors">
        <list>
            <ref bean="localeChangeInterceptor"/>
        </list>
    </property>
    <property name="mappings">
        <value>/**/*.view=someController</value>
    </property>
</bean>
```

###  1.1.10. Themes 主题

您可以应用==Spring Web MVC框架主题==来设置应用程序的整体外观，从而增强用户体验。主题是**静态资源（通常是样式表和图像）的集合**，这些资源会影响应用程序的视觉样式。

####  Defining a theme 定义主题

要在Web应用程序中使用主题，必须设置`org.springframework.ui.context.ThemeSource`接口的实现 。该`WebApplicationContext` 接口可以**扩展**，`ThemeSource`但将其职责委托给专用的实现。默认情况下，委托是`org.springframework.ui.context.support.ResourceBundleThemeSource`从**类路径的根**加载属性文件的 实现。要使用自定义`ThemeSource` 实现或配置的基本名称前缀`ResourceBundleThemeSource`，可以在应用程序上下文中使用保留名称注册Bean `themeSource`。Web应用程序**上下文会自动检测**到具有该名称的bean并使用它。

使用时`ResourceBundleThemeSource`，将在一个简单的属性文件中定义一个主题。属性文件列出了组成**主题的资源**，如以下示例所示：

```
styleSheet = / themes / cool / style.css 
background = / themes / cool / img / coolBg.jpg
```

属性的**键**是从视图代码引用主题元素的**名称**。对于JSP，通常使用`spring:theme`自定义标签（与该`spring:message`标签非常相似）来执行此操作。以下JSP片段使用上一示例中定义的主题来自定义外观：

```xml
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<html>
    <head>
        <link rel="stylesheet" href="<spring:theme code='styleSheet'/>" type="text/css"/>
    </head>
    <body style="background=<spring:theme code='background'/>">
        ...
    </body>
</html>
```

默认情况下，`ResourceBundleThemeSource`使用**空的基本名称**前缀。结果，从类路径的根加载属性文件。因此，您可以将 `cool.properties`主题定义放在类路径根目录中的目录中（例如in中`/WEB-INF/classes`）。在`ResourceBundleThemeSource`使用标准的Java资源包加载机制，允许主题的国际化。例如，我们可以使用`/WEB-INF/classes/cool_nl.properties`引用带有荷兰文字的特殊背景图片。

#### Resolving Themes 解决主题

定义主题后，如[上一节所述](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-themeresolver-defining)，您可以决定使用哪个主题。在`DispatcherServlet`查找名为类`themeResolver` 找出哪个`ThemeResolver`使用实施。主题解析器的工作方式与`LocaleResolver`。它可以==检测==用于特定请求的主题，还可以更改请求的主题。下表描述了Spring提供的主题解析器：

**表5. ThemeResolver实现**

|           类           | 描述                                                         |
| :--------------------: | :----------------------------------------------------------- |
|  `FixedThemeResolver`  | 选择通过使用`defaultThemeName`属性设置的固定主题。           |
| `SessionThemeResolver` | 主题在用户的HTTP会话中维护。每个会话**只需设置一次**，但在会话之间不会保留。 |
| `CookieThemeResolver`  | 所选主题存储在客户端的**cookie**中。                         |

Spring还提供了一个`ThemeChangeInterceptor`允许使用简单的请求参数对每个请求进行主题更改的功能。

### 1.1.11. Multipart Resolver 多部分解析器

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-multipart)

`MultipartResolver`从`org.springframework.web.multipart`软件包中提取是一种分析多部分请求（包括文件上传）的策略。有一种基于[Commons FileUpload的实现](https://jakarta.apache.org/commons/fileupload)，另一种基于**Servlet 3.0多部分**请求解析。

要启用多部分处理，您需要`MultipartResolver`在`DispatcherServlet`Spring配置中声明一个名称为的bean `multipartResolver`。在`DispatcherServlet`检测到它，并将其应用于传入请求。当与内容类型的POST`multipart/form-data`被接收时，分解器解析该**内容**和包装了当前`HttpServletRequest`作为`MultipartHttpServletRequest`在揭除它们作为请求参数提供到解析部件的访问。

####  Apache Commons FileUpload

要使用Apache Commons `FileUpload`，可以配置`CommonsMultipartResolver`名称为的类型的Bean `multipartResolver`。您还需要`commons-fileupload`依赖于类路径。

#### Servlet 3.0

需要通过Servlet容器配置启用Servlet 3.0多部分解析。为此：

- 在Java中，在Servlet注册上==设置MultipartConfigElement==。
- 在中`web.xml`，`"<multipart-config>"`向Servlet声明中添加一个部分。

以下示例显示了如何在Servlet注册上设置MultipartConfigElement：

Java:

```java
public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    // ...

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {

        // Optionally also set maxFileSize, maxRequestSize, fileSizeThreshold
        registration.setMultipartConfig(new MultipartConfigElement("/tmp"));
    }

}
```

Servlet 3.0配置到位后，您可以添加`StandardServletMultipartResolver`名称为的类型的Bean `multipartResolver`。

### 1.1.12. Logging 日志

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-logging)

Spring MVC中的**DEBUG级别**的日志被设计为紧凑，**最少**且**人性化**的。它侧重于一遍又一遍有用的高价值信息，而其他信息仅在调试特定问题时才有用。

**TRACE级别**的日志记录通常遵循与DEBUG相同的原则（例如，也不应是消防水带），但可用于调试任何问题。此外，某些日志消息在TRACE和DEBUG上可能显示不同级别的详细信息。

良好的日志记录来自使用日志的经验。如果发现任何不符合既定目标的东西，请告诉我们。

#### Sensitive Data 敏感数据

调试和跟踪日志记录可能会记录敏感信息。这就是==默认情况下屏蔽请求参数和标头==，并且必须通过`enableLoggingRequestDetails`on属性显式启用其完整登录的原因`DispatcherServlet`。

以下示例显示了如何通过使用Java配置来执行此操作：

Java:

```java
public class MyInitializer
        extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return ... ;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return ... ;
    }

    @Override
    protected String[] getServletMappings() {
        return ... ;
    }

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {
        registration.setInitParameter("enableLoggingRequestDetails", "true");
    }

}
```

##  1.2. Filters 筛选器

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-filters)

该`spring-web`模块提供了一些有用的过滤器：

- [Form Data](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-http-put)
- [Forwarded Headers](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-forwarded-headers)
- [Shallow ETag](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-shallow-etag)
- [CORS](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-cors)

### 1.2.1. Form Data 表格数据

浏览器只能通过HTTP GET或HTTP POST提交表单数据，但非浏览器客户端也可以使用HTTP PUT，PATCH和DELETE。Servlet API要求`ServletRequest.getParameter*()` 方法**仅支持**HTTP POST的表单字段访问。

该`spring-web`模块提供`FormContentFilter`了拦截内容类型为**HTTP PUT，PATCH和DELETE**的请求，`application/x-www-form-urlencoded`从请求主体读取表单数据，并包装`ServletRequest`使其通过`ServletRequest.getParameter*()`一系列方法可用的表单数据。

### 1.2.2. Forwarded Headers	转发的标题

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-forwarded-headers)

当请求通过**代理（例如负载平衡器）**进行处理时，主机，端口和方案可能会更改，这给从客户端角度指向正确的主机，端口和方案的链接创建带来了挑战。

[RFC 7239](https://tools.ietf.org/html/rfc7239)定义了`Forwarded`代理可用来提供有关原始请求的信息的HTTP标头。还有其他一些非标头，也包括`X-Forwarded-Host`，`X-Forwarded-Port`， `X-Forwarded-Proto`，`X-Forwarded-Ssl`，和`X-Forwarded-Prefix`。

ForwardedHeaderFilter是一个**Servlet筛选器**，用于修改请求，以便a）基于Forwarded标头更改主机，端口和方案，b）删除那些标头以**消除**进一步的影响。过滤器依赖于包装请求，因此必须先于其他过滤器（如）**订购**，该过滤器`RequestContextFilter`应适用于修改后的请求，而**不适用**于原始请求。

对于转发的标头，存在安全方面的考虑，因为应用程序无法知道标头是由代理添加的，还是由恶意客户端添加的。这就是为什么应配置信任边界处的代理以删除`Forwarded` 来自外部的**不受信任的标头**的原因。您还可以配置`ForwardedHeaderFilter` with `removeOnly=true`，在这种情况下，它会删除但不使用标题。

为了支持[异步请求](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async)和错误调度，此过滤器应与`DispatcherType.ASYNC`和也映射`DispatcherType.ERROR`。如果使用Spring Framework的`AbstractAnnotationConfigDispatcherServletInitializer` （请参阅[Servlet Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-container-config)），则会为所有调度类型**自动注册所有过滤器**。但是，如果通过注册该过滤器`web.xml`通过或在春季启动 `FilterRegistrationBean`时一定要包括`DispatcherType.ASYNC`和 `DispatcherType.ERROR`除`DispatcherType.REQUEST`。

### 1.2.3. Shallow ETag  弱*ETag*

所述`ShallowEtagHeaderFilter`过滤器通过**缓存**写入响应的内容，并从它计算MD5哈希创建一个“浅”的ETag。客户端下一次发送时，将执行相同的操作，但还会将计算出的值与`If-None-Match` 请求标头进行比较，==如果二者相等，则返回304==（NOT_MODIFIED）。

这种策略可以节省网络带宽，但不能节省CPU，因为必须为每个请求计算完整响应。如前所述，控制器级别的其他策略可以避免计算。请参阅[HTTP缓存](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching)。

该过滤器具有一个`writeWeakETag`参数，该参数将过滤器配置为写入弱ETag，类似于以下内容：（`W/"02a2d595e6ed9a0b24f027f2b63b134d6"`如[RFC 7232第2.3节中](https://tools.ietf.org/html/rfc7232#section-2.3)所定义 ）。

为了支持[异步请求，](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async)必须对此过滤器进行映射，`DispatcherType.ASYNC`以便过滤器可以延迟并成功生成ETag到最后一个异步调度的末尾。如果使用Spring Framework的 `AbstractAnnotationConfigDispatcherServletInitializer`（请参阅[Servlet Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-container-config)），则会为所有调度类型自动注册所有过滤器。但是，如果通过`web.xml`或在Spring Boot中通过注册过滤器，请`FilterRegistrationBean`确保包含 `DispatcherType.ASYNC`。

###  1.2.4. CORS 跨域资源共享

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-filters-cors)

Spring MVC通过控制器上的注释为CORS配置提供了**细粒度**的支持。但是，当与Spring Security一起使用时，我们建议依赖于 `CorsFilter`必须在Spring Security的过滤器链之前订购的内置组件。

有关更多详细信息，请参见有关[CORS](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors)和[CORS过滤器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors-filter)的部分。

##  1.3. Annotated Controllers 带注释的控制器

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-controller)

Spring MVC提供了一个==基于注释的编程模型==，其中`@Controller`和 `@RestController`组件使用注释来表达请求映射，请求输入，异常处理等。带注释的控制器具有**灵活的方法签名**，无需扩展基类或实现特定的接口。以下示例显示了由注释定义的控制器：

Java:

```java
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String handle(Model model) {
        model.addAttribute("message", "Hello World!");
        return "index";
    }
}
```

在前面的示例中，该方法接受Model并**以String的形式**返回视图名称，但是还存在许多其他选项，本章稍后将对其进行说明。

| ![image-20201201085421250](Spring%20Web%20MVC.assets/image-20201201085421250.png) | [spring.io](https://spring.io/guides) 上的指南和教程使用本节中描述的基于注释的编程模型。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

### 1.3.1. Declaration 声明

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-controller)

您可以通过使用Servlet的标准Spring bean定义来定义控制器bean `WebApplicationContext`。该`@Controller`原型允许自动检测，使用Spring普遍支持检测对准`@Component`在类路径中类和自动注册bean定义他们。它还充当带注释类的**构造型**，表明其作为**Web组件**的作用。

要启用对此类`@Controller`bean的自动检测，可以将**组件扫描**添加到Java配置中，如以下示例所示：

```java
@Configuration
@ComponentScan("org.example.web")
public class WebConfig {

    // ...
}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example.web"/>

    <!-- ... -->

</beans>
```

`@RestController`是一个[组合式注释](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-meta-annotations)，它本身带有元注释`@Controller`并`@ResponseBody`表示一个控制器，该控制器的每个方法都继承了类型级别的`@ResponseBody`注释，因此，与视图分辨率和HTML模板渲染相比，它**直接写入响应主体**。

####  AOP Proxies（AOP代理）

在某些情况下，您可能需要在运行时用AOP代理装饰控制器。一个示例是，如果您选择`@Transactional`直接在控制器上具有批注。在这种情况下，特别是对于控制器，我们建议使用基于类的代理。这通常是控制器的默认选择。但是，如果一个控制器必须实现一个接口，这不是一个Spring上下文回调（例如`InitializingBean`，`*Aware`和其他人），您可能需要显式配置基于类的代理。例如，使用`<tx:annotation-driven/>`可以更改为`<tx:annotation-driven proxy-target-class="true"/>`，使用 `@EnableTransactionManagement`可以更改为 `@EnableTransactionManagement(proxyTargetClass = true)`。

### 1.3.2. Request Mapping

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestmapping)

您可以使用`@RequestMapping`批注将请求映射到控制器方法。它具有各种属性，可以通过**URL，HTTP**方法，请求参数，标头和媒体类型进行匹配。您可以在类级别使用它来表示==共享的映射==，也可以在方法级别使用它来**缩小**到特定的端点映射。

还有HTTP方法特定的快捷方式变体`@RequestMapping`：

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

快捷方式是提供的“[自定义注释”](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-composed)，因为可以说，**大多数控制器方法**应映射到特定的HTTP方法，而不是使用using `@RequestMapping`（默认情况下，它与所有HTTP方法匹配）。同时，在类级别仍需要@RequestMapping来表示**共享映射**。

以下示例具有类型和方法级别的映射：

```java
@RestController
@RequestMapping("/persons")
class PersonController {

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        // ...
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void add(@RequestBody Person person) {
        // ...
    }
}
```

#### URI patterns  (URI模式)

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestmapping-uri-templates)

`@RequestMapping`可以==使用URL模式映射方法==。有两种选择：

- `PathPattern` —与URL路径匹配的预解析模式，该路径也预解析为 `PathContainer`。该解决方案专为Web使用而设计，可有效处理**编码和路径参数**，并有效匹配。
- `AntPathMatcher` —将字符串模式与字符串路径匹配。这是在Spring配置中还用于选择类路径，文件系统和其他位置上的资源的原始解决方案。它效率较低，并且字符串路径输入对于有效处理URL的编码和其他问题是一个挑战。

`PathPattern`是Web应用程序的推荐解决方案，它是Spring WebFlux的**唯一选择**。在5.3之前的版本中，它`AntPathMatcher`是Spring MVC中的唯一选择，并且继续是默认设置。但是`PathPattern`可以在[MVC配置中](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-path-matching)启用 。

`PathPattern`支持与相同的模式语法`AntPathMatcher`。另外，它还支持**捕获模式**，例如`{*spring}`，用于匹配路径末端的**0个或更多**路径段。`PathPattern`还限制了`**`用于匹配多个路径段的用法，以使其仅在**模式末尾**才允许使用。当为给定请求选择最佳匹配模式时，这消除了很多歧义。有关完整模式的语法，请参阅 [PathPattern](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/util/pattern/PathPattern.html)和 [AntPathMatcher](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/util/AntPathMatcher.html)。

一些示例模式：

- `"/resources/ima?e.png"` -匹配路径段中的一个字符
- `"/resources/*.png"` -匹配路径段中的零个或多个字符
- `"/resources/**"` -匹配多个路径段
- `"/projects/{project}/versions"` -匹配路径段并将其捕获为变量
- `"/projects/{project:[a-z]+}/versions"` -使用正则表达式匹配并捕获变量

捕获的URI变量可以使用访问`@PathVariable`。例如：

```java
@GetMapping("/owners/{ownerId}/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
    // ...
}
```

您可以在类和方法级别声明URI变量，如以下示例所示：

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class OwnerController {

    @GetMapping("/pets/{petId}")
    public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
        // ...
    }
}
```

URI变量会自动转换为适当的类型或`TypeMismatchException` 引发。简单类型（`int`，`long`，`Date`，等）默认支持，你可以注册任何其它数据类型的支持。请参阅[类型转换](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-typeconversion)和[`DataBinder`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-initbinder)。

您可以**显式地命名URI变量**（例如`@PathVariable("customId")`），但是如果名称相同并且您的代码是通过调试信息或`-parameters`Java 8上的编译器标志进行编译的，则可以省去该细节。

语法`{varName:regex}`使用**正则表达式**声明语法为的URI变量`{varName:regex}`。例如，给定URL `"/spring-web-3.0.5 .jar"`，以下方法提取名称，版本和文件扩展名：

```java
@GetMapping("/{name:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{ext:\\.[a-z]+}")
public void handle(@PathVariable String name, @PathVariable String version, @PathVariable String ext) {
    // ...
}
```

URI路径模式还可以具有**嵌入式**`${…}`占位符，这些占位符在启动时通过`PropertyPlaceHolderConfigurer`针对本地，系统，环境和其他属性源进行解析。例如，您可以使用它来基于一些外部配置参数化基本URL。

#### Pattern Comparison 模式比较

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestmapping-pattern-comparison)

当多个模式与URL匹配时，必须选择最佳匹配。根据是否启用了已解析的`PathPattern's，使用以下方法之一完成此操作：

- [`PathPattern.SPECIFICITY_COMPARATOR`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/util/pattern/PathPattern.html#SPECIFICITY_COMPARATOR)
- [`AntPathMatcher.getPatternComparator(String path)`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/util/AntPathMatcher.html#getPatternComparator-java.lang.String-)

两者都==有助于对模式进行排序==，并在上面放置更具体的模式。如果模式的URI变量（计数为1），单通配符（计数为1）和双通配符（计数为2）的数量较少，则模式的含义不太明确。给定相等的分数，则选择更长的模式。给定相同的分数和长度，将选择URI变量多于通配符的模式。

默认映射模式（`/**`）从评分中**排除**，并且始终排在最后。另外，前缀模式（例如`/public/**`）被认为比没有双通配符的其他模式更具体。

有关完整的详细信息，请单击上面的链接到模式比较器。

#### Suffix Match 后缀匹配

从5.3开始，**默认情况下**，Spring MVC不再执行`.*`后缀模式匹配，其中映射到的控制器`/person`也**隐式映射**到 `/person.*`。因此路径延伸不再用于解释所请求的内容类型为响应-例如，`/person.pdf`，`/person.xml`，等。

当浏览器用来发送`Accept`难以一致解释的标头时，以这种方式使用**文件扩展名**是必要的。目前，这已不再是必须的，使用`Accept`标头应该是**首选**。

随着时间的流逝，文件扩展名的使用已经以各种方式证明是有问题的。当使用URI变量，路径参数和URI编码进行覆盖时，可能会导致歧义。关于基于URL的授权和安全性的推理（请参阅下一部分以了解更多详细信息）也变得更加困难。

要完全禁用5.3之前版本中的路径扩展，请设置以下内容：

- `useSuffixPatternMatching(false)`，请参阅[PathMatchConfigurer](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-path-matching)
- `favorPathExtension(false)`，请参阅[ContentNegotiationConfigurer](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-content-negotiation)

除了通过`"Accept"`标头以外，请求其他内容类型的方法仍然很有用，例如在浏览器中键入URL时。路径扩展的一种安全替代方法是==使用查询参数策略==。如果必须使用文件扩展名，请考虑通过[ContentNegotiationConfigurer](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-content-negotiation)的`mediaTypes`属性 将它们限制为显式注册的扩展名列表。

#### Suffix Match and RFD （后缀匹配和RFD）

反射文件下载（RFD）攻击与XSS相似，它依赖于反映在响应中的**请求输入**（例如，查询参数和URI变量）。但是，**RFD攻击**不是将JavaScript插入HTML，而是依靠浏览器切换来执行下载，并在以后双击时将响应视为可执行脚本。

在Spring MVC中，`@ResponseBody`和`ResponseEntity`方法是有风险的，因为它们可以呈现**不同**的内容类型，客户端可以通过URL路径扩展要求。**禁用**后缀模式匹配并使用路径扩展进行内容协商可以**降低风险**，但不足以防止RFD攻击。

为了防止RFD攻击，Spring MVC在呈现响应主体之前添加了 `Content-Disposition:inline;filename=f.txt`标头，以建议固定和安全的下载文件。仅当URL路径包含既不被认为安全也不被明确注册用于内容协商的文件扩展名时，才执行此操作。但是，当直接在浏览器中键入URL时，它可能会产生**副作用**。

默认情况下，许多常见路径扩展都被**视为安全**。具有自定义`HttpMessageConverter`实现的应用程序 可以==显式注册文件扩展名==以进行内容协商，以避免`Content-Disposition`为这些扩展名添加头。请参阅[内容类型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-content-negotiation)。

有关RFD的其他建议，请参见[CVE-2015-5211](https://pivotal.io/security/cve-2015-5211)。

#### Consumable Media Types 消耗媒体类型

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestmapping-consumes)

您可以根据请求的来缩小请求映射`Content-Type`，如以下示例所示：

```java
@PostMapping(path = "/pets", consumes = "application/json") 注释: 1
public void addPet(@RequestBody Pet pet) {
    // ...
}
```

| ![image-20201201091420034](Spring%20Web%20MVC.assets/image-20201201091420034.png) | 使用`consumes`属性按内容类型缩小映射。 |
| ------------------------------------------------------------ | -------------------------------------- |
|                                                              |                                        |

该`consumes`属性还**支持否定表达式**-例如，`!text/plain`表示以外的任何内容类型`text/plain`。

您可以`consumes`在类级别声明共享属性。但是，与大多数其他请求映射属性不同，在**类级使用时**，方法级`consumes`属性将覆盖而**不是扩展类级声明**。

| ![image-20201201090712327](Spring%20Web%20MVC.assets/image-20201201090712327.png) | `MediaType`提供常用媒体类型（例如`APPLICATION_JSON_VALUE`和）的常量 `APPLICATION_XML_VALUE`。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

#### Producible Media Types 可生产的媒体类型

您可以根据`Accept`==请求标头和控制器方法==生成的内容类型列表来**缩小**请求映射，如以下示例所示：

```java
@GetMapping(path = "/pets/{petId}", produces = "application/json") 注释：1
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```

| ![image-20201201091408290](Spring%20Web%20MVC.assets/image-20201201091408290.png) | 使用`produces`属性按内容类型缩小映射。 |
| ------------------------------------------------------------ | -------------------------------------- |
|                                                              |                                        |

媒体类型可以指定**字符集**。支持否定表达式-例如， `!text/plain`表示除**“文本/纯文本”**之外的任何内容类型。

您可以`produces`在类级别声明共享属性。但是，与大多数其他请求映射属性不同，在类级使用时，方法级`produces`属性将覆盖而不是扩展类级声明。

| ![image-20201201090810521](Spring%20Web%20MVC.assets/image-20201201090810521.png) | `MediaType`提供常用媒体类型（例如`APPLICATION_JSON_VALUE`和）的常量 `APPLICATION_XML_VALUE`。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

#### Parameters, headers (参数，标题)

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestmapping-params-and-headers)

您可以根据请求参数条件来缩小请求映射。您可以测试是否存在请求参数（`myParam`），是否存在请求参数（）`!myParam`或特定值（`myParam=myValue`）。以下示例显示如何测试特定值：

```java
@GetMapping(path = "/pets/{petId}", params = "myParam=myValue") 注释: 1
public void findPet(@PathVariable String petId) {
    // ...
}
```

| ![image-20201201091356586](Spring%20Web%20MVC.assets/image-20201201091356586.png) | 测试是否`myParam`相等`myValue`。 |
| ------------------------------------------------------------ | -------------------------------- |
|                                                              |                                  |

您还可以将其与请求标头条件一起使用，如以下示例所示：

```java
@GetMapping(path = "/pets", headers = "myHeader=myValue") 注释: 1
public void findPet(@PathVariable String petId) {
    // ...
}
```

| ![image-20201201091333938](Spring%20Web%20MVC.assets/image-20201201091333938.png) | 测试是否`myHeader`相等`myValue`。 |
| ------------------------------------------------------------ | --------------------------------- |
|                                                              |                                   |

| ![image-20201201090940705](Spring%20Web%20MVC.assets/image-20201201090940705.png) | 您可以匹配`Content-Type`并`Accept`与标头条件匹配，但最好改用 [消耗](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-consumes)和[生产](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-produces) 。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

#### HTTP HEAD, OPTIONS （HTTP头，选项）

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestmapping-head-options)

`@GetMapping`（和`@RequestMapping(method=HttpMethod.GET)`）**透明地支持**HTTP HEAD以进行请求映射。控制器方法不需要更改。应用于的响应包装器`javax.servlet.http.HttpServlet`确保将`Content-Length` 标头设置为写入的**字节数**（实际上未写入响应）。

`@GetMapping`（和`@RequestMapping(method=HttpMethod.GET)`）被隐式映射到并**支持HTTP HEAD**。像处理HTTP GET一样处理HTTP HEAD请求，不同的是，不是写入正文，而是计算**字节数**并设置`Content-Length` 标头。

默认情况下，通过将`Allow`响应标头设置为所有`@RequestMapping`具有**匹配URL模式**的方法中列出的HTTP方法列表来==处理HTTP OPTIONS==。

对于`@RequestMapping`不使用HTTP方法声明的情况，`Allow`标头设置为 `GET,HEAD,POST,PUT,PATCH,DELETE,OPTIONS`。控制器方法应该**总是声明**支持HTTP方法（例如，通过使用HTTP方法具体变体： `@GetMapping`，`@PostMapping`，及其他）。

您可以将`@RequestMapping`方法显式映射到HTTP HEAD和HTTP OPTIONS，但这在通常情况下不是必需的。

#### Custom Annotations 自定义注释

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#mvc-ann-requestmapping-head-options)

Spring MVC支持将[组合注释](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-meta-annotations) 用于请求映射。这些注解本身是**元注解**， `@RequestMapping`并组成它们以`@RequestMapping` **更狭窄**，更具体的目的重新声明属性的子集（或全部）。

`@GetMapping`，`@PostMapping`，`@PutMapping`，`@DeleteMapping`，和`@PatchMapping`由注解的例子。之所以提供它们，是因为**大多数控制器**方法应该映射到特定的HTTP方法，而不是使用using `@RequestMapping`（默认情况下与所有HTTP方法匹配）。如果需要组合注释的示例，请查看如何声明它们。

Spring MVC还支持带有自定义请求匹配逻辑的自定义请求映射属性。这是一个**更高级**的选项，它需要`RequestMappingHandlerMapping`对`getCustomMethodCondition`方法进行子类化 和覆盖，您可以在其中检查custom属性并返回您自己的方法`RequestCondition`。

#### Explicit Registrations 明确注册

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestmapping-registration)

您可以以编程方式注册处理程序方法，这些方法==可用于动态注册或高级案例==，例如同一处理程序在不同URL下的不同实例。下面的示例注册一个处理程序方法：

```java
@Configuration
public class MyConfig {

    @Autowired
    public void setHandlerMapping(RequestMappingHandlerMapping mapping, UserHandler handler)  注释: 1
            throws NoSuchMethodException {

        RequestMappingInfo info = RequestMappingInfo
                .paths("/user/{id}").methods(RequestMethod.GET).build();  注释: 2

        Method method = UserHandler.class.getMethod("getUser", Long.class);  注释: 3

        mapping.registerMapping(info, handler, method);  注释： 4
    }
}
```



| ![image-20201201091106842](Spring%20Web%20MVC.assets/image-20201201091106842.png) | 注入目标处理程序和控制器的处理程序映射。 |
| :----------------------------------------------------------: | ---------------------------------------- |
| ![image-20201201091122514](Spring%20Web%20MVC.assets/image-20201201091122514.png) | 准备请求映射元数据。                     |
| ![image-20201201091138103](Spring%20Web%20MVC.assets/image-20201201091138103.png) | 获取处理程序方法。                       |
| ![image-20201201091148097](Spring%20Web%20MVC.assets/image-20201201091148097.png) | 添加注册。                               |

### 1.3.3. Handler Methods 处理程序方法

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-methods)

`@RequestMapping` 处理程序方法具有**灵活**的签名，可以从支持的==控制器方法参数和返回值的范围==中进行选择。

#### Method Arguments 方法参数

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-arguments)

下表描述了受支持的控制器方法参数。任何参数均**不支持**反应性类型。

JDK 8次的`java.util.Optional`被支撑作为组合的方法的参数与具有注解`required`的属性（例如，`@RequestParam`，`@RequestHeader`，和其它物质）和相当于`required=false`。

|                        控制器方法参数                        | 描述                                                         |
| :----------------------------------------------------------: | :----------------------------------------------------------- |
|              `WebRequest`， `NativeWebRequest`               | 通用访问请求参数以及请求和会话属性，而无需直接使用Servlet API。 |
| `javax.servlet.ServletRequest`， `javax.servlet.ServletResponse` | 选择**任何特定的请求或响应类型**-例如`ServletRequest`，`HttpServletRequest`或春天的`MultipartRequest`，`MultipartHttpServletRequest`。 |
|               `javax.servlet.http.HttpSession`               | 强制会话的存在。结果，这种论据永远**不会`null`**。请注意，会话访问不是线程安全的。考虑将`RequestMappingHandlerAdapter`实例的`synchronizeOnSession`标志设置 为`true`是否允许多个请求**同时**访问会话。 |
|               `javax.servlet.http.PushBuilder`               | 用于程序化HTTP / 2资源推送的Servlet 4.0推送构建器API。请注意，根据Servlet规范，`PushBuilder`如果客户端**不支持**HTTP / 2功能，则注入的实例可以为null。 |
|                  `java.security.Principal`                   | 当前经过身份验证的用户-可能是特定的`Principal`实现类（如果已知）。 |
|                         `HttpMethod`                         | 请求的HTTP方法。                                             |
|                      `java.util.Locale`                      | 当前请求的语言环境，由最具体的`LocaleResolver`可用语言（实际上是配置的`LocaleResolver`或`LocaleContextResolver`）确定。 |
|          `java.util.TimeZone` + `java.time.ZoneId`           | 与当前请求关联的时区，由决定`LocaleContextResolver`。        |
|           `java.io.InputStream`， `java.io.Reader`           | 用于访问Servlet API公开的原始**请求**正文。                  |
|          `java.io.OutputStream`， `java.io.Writer`           | 用于访问Servlet API公开的原始响应正文。                      |
|                       `@PathVariable`                        | 用于访问**URI模板变量**。请参阅[URI模式](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-uri-templates)。 |
|                      `@MatrixVariable`                       | 用于访问**URI路径段**中的名称/值对。请参阅[矩阵变量](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-matrix-variables)。 |
|                       `@RequestParam`                        | 用于访问**Servlet请求参数**，包括多部分文件。参数值将转换为声明的方法参数类型。参见[`@RequestParam`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestparam)以及[Multipart](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multipart-forms)。请注意，`@RequestParam`对于简单参数值，使用是可选的。请参阅此表末尾的“其他任何参数”。 |
|                       `@RequestHeader`                       | 用于访问请求标头。标头值将转换为声明的方法参数类型。请参阅[`@RequestHeader`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestheader)。 |
|                        `@CookieValue`                        | 用于访问cookie。**Cookies值**将转换为声明的方法参数类型。请参阅[`@CookieValue`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-cookievalue)。 |
|                        `@RequestBody`                        | 用于访问HTTP请求正文。正文内容通过使用`HttpMessageConverter`实现转换为声明的方法参数类型。请参阅[`@RequestBody`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestbody)。 |
|                       `HttpEntity<B>`                        | 用于访问请求标头和正文。**主体**用转换`HttpMessageConverter`。参见[HttpEntity](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-httpentity)。 |
|                        `@RequestPart`                        | 要访问`multipart/form-data`请求中的零件，请使用转换零件的主体`HttpMessageConverter`。参见[多部分](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multipart-forms)。 |
| `java.util.Map`，`org.springframework.ui.Model`，`org.springframework.ui.ModelMap` | 用于访问HTML控制器中使用的模型，并作为**视图渲染**的一部分公开给模板。 |
|                     `RedirectAttributes`                     | 指定**在重定向的情况下**使用的属性（即追加到查询字符串中），并指定要临时存储的属性，直到重定向后的请求为止。请参阅[重定向属性](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-redirecting-passing-data)和[Flash属性](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-flash-attributes)。 |
|                      `@ModelAttribute`                       | 用于**访问**已应用数据绑定和验证的模型中现有的属性（如果不存在，则进行实例化）。参见[`@ModelAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-modelattrib-method-args)以及 [模型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-modelattrib-methods)和[`DataBinder`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-initbinder)。请注意，使用of`@ModelAttribute`是可选的（例如，设置其属性）。请参阅此表末尾的“其他任何参数”。 |
|                  `Errors`， `BindingResult`                  | 用于**访问**命令对象（即，`@ModelAttribute`自变量）的验证和数据绑定中的错误，`@RequestBody`或访问a或自 `@RequestPart`变量的验证中的错误。您必须在经过**验证**的方法参数之后立即声明`Errors`或`BindingResult`参数。 |
|          `SessionStatus` +班级 `@SessionAttributes`          | 为了**标记表单处理完成**，将触发清除通过类级`@SessionAttributes`注释声明的会话属性。请参阅 [`@SessionAttributes`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-sessionattributes)以获取更多详细信息。 |
|                    `UriComponentsBuilder`                    | 用于**准备**相对于当前请求的主机，端口，方案，上下文路径以及servlet映射的文字部分的URL。请参阅[URI链接](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-uri-building)。 |
|                     `@SessionAttribute`                      | 与访问由于类级`@SessionAttributes`声明而存储在会话中的模型属性**相反**，用于访问任何会话属性。请参阅 [`@SessionAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-sessionattribute)以获取更多详细信息。 |
|                     `@RequestAttribute`                      | 用于访问请求属性。请参阅[`@RequestAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestattrib)以获取更多详细信息。 |
|                         任何其他论点                         | 如果方法参数与该表中的任何较早值都不匹配，并且为简单类型（由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定 ，则将其解析为`@RequestParam`。否则，将其解析为`@ModelAttribute`。 |

####  Return Values 返回值

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-return-types)

下表描述了受支持的控制器方法返回值。所有返回值都支持反应性类型。

|                       控制器方法返回值                       | 描述                                                         |
| :----------------------------------------------------------: | :----------------------------------------------------------- |
|                       `@ResponseBody`                        | 返回值通过`HttpMessageConverter`实现**进行转换并写入响应**。请参阅[`@ResponseBody`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-responsebody)。 |
|            `HttpEntity<B>`， `ResponseEntity<B>`             | 指定完整响应（包括HTTP标头和正文）的返回值将通过`HttpMessageConverter`实现进行转换，并写入响应中。参见[ResponseEntity](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-responseentity)。 |
|                        `HttpHeaders`                         | 用于返回不包含标题的响应。                                   |
|                           `String`                           | 一个**视图名称**，将通过`ViewResolver`实现来解析，并与隐式模型一起使用-通过命令对象和`@ModelAttribute`方法确定。处理程序方法还可以通过声明`Model`参数来以**编程方式**丰富模型（请参见[Explicit Registrations](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-registration)）。 |
|                            `View`                            | 甲`View`实例以使用用于与所述隐式模型一起渲染-通过命令对象和确定`@ModelAttribute`方法。处理程序方法还可以通过声明`Model`参数来以编程方式丰富模型（请参见[Explicit Registrations](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-registration)）。 |
|       `java.util.Map`， `org.springframework.ui.Model`       | 要添加到隐式模型的属性，视图名称**通过隐式确定**`RequestToViewNameTranslator`。 |
|                      `@ModelAttribute`                       | 要添加到模型的属性，视图名称通过隐式确定`RequestToViewNameTranslator`。请注意，这`@ModelAttribute`是可选的。请参阅此表末尾的“其他任何返回值”。 |
|                     `ModelAndView` 目的                      | 要使用的视图和模型属性，以及响应状态（可选）。               |
|                            `void`                            | 如果`void`返回类型（或`null`返回值）的方法也具有`ServletResponse`，`OutputStream`参数或`@ResponseStatus`注释，则认为该方法**已完全**处理了响应。如果控制器进行了肯定`ETag`或`lastModified`**时间戳**检查，则情况也是如此 （有关详细信息，请参阅[控制器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching-etag-lastmodified)）。如果以上条件都不成立，则`void`返回类型还可以为REST控制器指示“无响应正文”，或者为HTML控制器指示默认视图名称选择。 |
|                     `DeferredResult<V>`                      | 从**任何线程异步**生成任何上述返回值-例如，由于某些事件或回调的结果。请参阅[异步请求](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async)和[`DeferredResult`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-deferredresult)。 |
|                        `Callable<V>`                         | 在Spring MVC管理的线程中异步产生上述任何返回值。请参阅[异步请求](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async)和[`Callable`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-callable)。 |
| `ListenableFuture<V>`， `java.util.concurrent.CompletionStage<V>`， `java.util.concurrent.CompletableFuture<V>` | `DeferredResult`为方便起见，替代，（例如，当基础服务返回其中之一时）。 |
|             `ResponseBodyEmitter`， `SseEmitter`             | **异步发出对象流**，以将其写入 `HttpMessageConverter`实现中。也支持作为的主体`ResponseEntity`。请参阅[异步请求](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async)和[HTTP流](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-http-streaming)。 |
|                   `StreamingResponseBody`                    | `OutputStream`异步写入响应。也支持作为的主体 `ResponseEntity`。请参阅[异步请求](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async)和[HTTP流](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-http-streaming)。 |
| 反应类型-Reactor，RxJava或其他类型 `ReactiveAdapterRegistry` | 替代收集到的`DeferredResult`多值流（例如`Flux`，`Observable`）`List`。对于流传输方案（例如，`text/event-stream`和`application/json+stream`）， 可以使用`SseEmitter`和`ResponseBodyEmitter`代替，其中`ServletOutputStream` 在**Spring MVC管理的线程上**执行阻塞I / O并在每次写入完成时施加背压。请参阅[异步请求](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async)和[响应类型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-reactive-types)。 |
|                        任何其他返回值                        | 如果返回值不是由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定的简单类型，则该返回值与该表中的任何较早值都不匹配且为a`String`或被`void`视为视图名称（通过`RequestToViewNameTranslator`应用默认视图名称选择 ），前提是该返回值不是简单类型 。简单类型的值仍然无法解析。 |

#### Type Conversion 类型转换

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-typeconversion)

表示一些注解的控制器方法的参数`String`为基础的==请求输入==（如 `@RequestParam`，`@RequestHeader`，`@PathVariable`，`@MatrixVariable`，和`@CookieValue`）可以要求类型转换如果参数被声明为比其它的东西`String`。

在这种情况下，将根据配置的转换器自动应用类型转换。默认情况下，简单的类型（`int`，`long`，`Date`，和其他人）的支持。您可以通过自定义类型转换`WebDataBinder`（见[`DataBinder`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-initbinder)），或者通过注册 `Formatters`与`FormattingConversionService`。参见[Spring字段格式化](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#format)。

#### Matrix Variables 矩阵变量

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-matrix-variables)

[RFC 3986](https://tools.ietf.org/html/rfc3986#section-3.3)讨论路径段中的名称/值对。在Spring MVC中，我们根据Tim Berners-Lee的[“旧帖子”](https://www.w3.org/DesignIssues/MatrixURIs.html)将其称为“矩阵变量” ，但它们也可以称为**URI路径参数**。

矩阵变量可以出现在任何路径段中，每个变量用**分号分隔**，多个值用**逗号分隔**（例如`/cars;color=red,green;year=2012`）。也可以==通过重复的变量名称==（例如`color=red;color=green;color=blue`）来指定多个值 。

如果期望URL包含**矩阵变量**，则控制器方法的请求映射必须使用URI变量来屏蔽该变量内容，并确保可以成功地匹配请求，而与矩阵变量的**顺序和状态无关**。以下示例使用矩阵变量：

```java
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11
}
```

鉴于所有路径段都可能包含矩阵变量，因此有时您可能需要消除矩阵变量应位于哪个路径变量的歧义。下面的示例演示了如何做到这一点：

```java
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22
}
```

可以将矩阵变量定义为可选变量，并**指定默认值**，如以下示例所示：

```java
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1
}
```

要获取所有矩阵变量，可以使用`MultiValueMap`，如以下示例所示：

```java
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable MultiValueMap<String, String> matrixVars,
        @MatrixVariable(pathVar="petId") MultiValueMap<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 22, "s" : 23]
}
```

请注意，您需要启用矩阵变量的使用。在MVC Java配置中，您需要通过 [路径匹配](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-path-matching)来设置`UrlPathHelper`with 。在MVC XML名称空间中，可以设置 。`removeSemicolonContent=false``<mvc:annotation-driven enable-matrix-variables="true"/>`

#### @RequestParam

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestparam)

您可以使用`@RequestParam`批注将**Servlet请求参数**（即查询参数或表单数据）绑定到控制器中的方法参数。

以下示例显示了如何执行此操作：

```java
@Controller
@RequestMapping("/pets")
public class EditPetForm {

    // ...

    @GetMapping
    public String setupForm(@RequestParam("petId") int petId, Model model) {  注释: 1
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...

}
```

| ![image-20201201094638995](Spring%20Web%20MVC.assets/image-20201201094638995.png) | 使用`@RequestParam`绑定`petId`。 |
| ------------------------------------------------------------ | -------------------------------- |
|                                                              |                                  |

默认情况下，使用此**批注**的方法参数是必需的，但是您可以通过将`@RequestParam`批注的`required`标志设置为 `false`或通过使用`java.util.Optional`包装器声明参数来指定方法参数是可选的。

如果目标方法参数类型不是，则类型转换将**自动应用** `String`。请参阅[类型转换](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-typeconversion)。

将参数类型声明为数组或列表，可以为同一参数名称解析多个参数值。

如果将`@RequestParam`注释声明为`Map<String, String>`或 `MultiValueMap<String, String>`，而注释中未指定参数名称，则将使用每个给定参数名称的请求参数值填充映射。

请注意，使用of`@RequestParam`是==可选的==（例如，设置其属性）。默认情况下，任何简单值类型的参数（由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定 ）且未由任何其他参数解析器解析，就如同使用注释一样`@RequestParam`。

#### @RequestHeader

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-cookievalue)

您可以使用`@RequestHeader`注释将请求标头**绑定**到控制器中的方法参数。

考虑以下带有标头的请求：

```
主机localhost：8080
接受text / html，application / xhtml + xml，application / xml; q = 0.9 
Accept-Language fr，en-gb; q = 0.7，en; q = 0.3 
Accept-Encoding gzip，deflate 
Accept-Charset ISO -8859-1，utf-8; q = 0.7，*; q = 0.7 
Keep-Alive 300
```

以下示例获取`Accept-Encoding`和`Keep-Alive`标头的值：

```java
@GetMapping("/demo")
public void handle(
        @RequestHeader("Accept-Encoding") String encoding, 注释: 1
        @RequestHeader("Keep-Alive") long keepAlive) {  注释: 2
    //...
}
```

| ![image-20201201094912548](Spring%20Web%20MVC.assets/image-20201201094912548.png) | 获取`Accept-Encoding`标头的值。 |
| ------------------------------------------------------------ | ------------------------------- |
| ![image-20201201094919181](Spring%20Web%20MVC.assets/image-20201201094919181.png) | 获取`Keep-Alive`标头的值。      |

如果目标方法的参数类型不是`String`，则将 自动应用类型转换。请参阅[类型转换](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-typeconversion)。

当`@RequestHeader`注解上的使用`Map<String, String>`， `MultiValueMap<String, String>`或`HttpHeaders`参数，则地图被填充有所有标头值。

| ![image-20201201095016982](Spring%20Web%20MVC.assets/image-20201201095016982.png) | 内置支持可用于将逗号分隔的字符串转换为数组或字符串集合或类型转换系统已知的其他类型。例如，用注释的方法参数`@RequestHeader("Accept")`可以是type `String`，也可以是 `String[]`or `List<String>`。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

#### @CookieValue

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-cookievalue)

您可以使用@CookieValue批注将HTTP cookie的值绑定到控制器中的方法参数。 考虑带有以下cookie的请求：

```
JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
```

```java
@GetMapping("/demo")
public void handle(@CookieValue("JSESSIONID") String cookie) { 注释：1
    //...
}
```

| ![image-20201201173557658](Spring%20Web%20MVC.assets/image-20201201173557658.png) | 获取JSESSIONID cookie的值。 |
| ------------------------------------------------------------ | --------------------------- |
|                                                              |                             |

如果目标方法的参数类型不是`String`，则类型转换将自动应用。请参阅[类型转换](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-typeconversion)。

#### @ModelAttribute

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-modelattrib-method-args)

您可以`@ModelAttribute`在方法参数上使用注释，以从模型访问属性，或者将其**实例化**（如果不存在）。model属性还覆盖了名称与字段名称匹配的HTTP Servlet请求参数中的值。这称为**数据绑定**，它使您不必处理解析和转换单个查询参数和表单字段的工作。以下示例显示了如何执行此操作：

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) { } 注释：1
```

| ![image-20201201173739112](Spring%20Web%20MVC.assets/image-20201201173739112.png) | 绑定的实例`Pet`。 |
| ------------------------------------------------------------ | ----------------- |
|                                                              |                   |

`Pet`上面的实例解析如下：

- 从模型（如果已使用[Model](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-modelattrib-methods)添加）。
- 通过使用HTTP会话[`@SessionAttributes`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-sessionattributes)。
- 从URI路径变量通过传递`Converter`（请参见下一个示例）。
- 从**默认构造函数**的调用开始。
- 从调用具有与Servlet请求参数匹配的参数的“主要构造函数”开始。参数名称是通过**JavaBeans** `@ConstructorProperties`或字节码中运行时保留的参数名称确定的。

虽然通常使用[模型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-modelattrib-methods)来用属性填充模型，但另一种替代方法是依赖于`Converter<String, T>`URI路径变量约定的组合。在下面的示例中，模型属性名称 `account`匹配URI路径变量`account`，并且`Account`通过将`String`帐号传递给已注册的来加载`Converter<String, Account>`：

```java
@PutMapping("/accounts/{account}")
public String save(@ModelAttribute("account") Account account) {
    // ...
}
```

获取模型属性实例后，将应用数据绑定。所述 `WebDataBinder`类servlet请求参数名（查询参数和表单字段）以在目标字段名称匹配`Object`。应用类型转换后，如有必要，将填充匹配字段。有关数据绑定（和验证）的更多信息，请参见 [验证](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation)。有关自定义数据绑定的更多信息，请参见 [`DataBinder`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-initbinder)。

数据绑定可能导致错误。默认情况下，`BindException`引发a。但是，要检查controller方法中的此类错误，可以在`BindingResult`旁边紧紧添加一个参数`@ModelAttribute`，如以下示例所示：

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) { 注释: 1
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

| ![image-20201201173904470](Spring%20Web%20MVC.assets/image-20201201173904470.png) | 在`BindingResult`旁边添加一个`@ModelAttribute`。 |
| ------------------------------------------------------------ | ------------------------------------------------ |
|                                                              |                                                  |

在某些情况下，您可能希望访问没有数据绑定的模型属性。在这种情况下，您可以将注入`Model`到控制器中并==直接访问它==，或者==设置set== @ModelAttribute(binding=false)，如以下示例所示：

```java
@ModelAttribut
public AccountForm setUpForm() {
    return new AccountForm();
}

@ModelAttribute
public Account findAccount(@PathVariable String accountId) {
    return accountRepository.findOne(accountId);
}

@PostMapping("update")
public String update(@Valid AccountForm form, BindingResult result,
        @ModelAttribute(binding=false) Account account) { 注释: 1
    // ...
}
```

| ![image-20201201173954191](Spring%20Web%20MVC.assets/image-20201201173954191.png) | 设置`@ModelAttribute(binding=false)`。 |
| ------------------------------------------------------------ | -------------------------------------- |
|                                                              |                                        |

您可以在**数据绑定**之后通过添加`javax.validation.Valid`注释或Spring的`@Validated`注释（ [Bean Validation](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation-beanvalidation)和 [Spring validate](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation)）自动应用验证 。以下示例显示了如何执行此操作：

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@Valid @ModelAttribute("pet") Pet pet, BindingResult result) { 注释： 1
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

| ![image-20201201174128139](Spring%20Web%20MVC.assets/image-20201201174128139.png) | 验证`Pet`实例。 |
| ------------------------------------------------------------ | --------------- |
|                                                              |                 |

请注意，using`@ModelAttribute`是可选的（例如，设置其属性）。默认情况下，任何不是简单值类型（由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定 ）且未被其他任何参数解析器解析的参数都将**被视为`@ModelAttribute`**。

####  @SessionAttributes

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-sessionattributes)

`@SessionAttributes`用于在**请求之间的HTTP Servlet会话中**存储模型属性。它是**类型级别**的注释，用于声明特定控制器使用的**会话属性**。这通常列出应透明地存储在会话中以供后续访问请求的模型属性的名称或模型属性的类型。

以下示例使用`@SessionAttributes`注释：

```java
@Controller
@SessionAttributes("pet") 注释：1
public class EditPetForm {
    // ...
}
```

| ![image-20201201174753802](Spring%20Web%20MVC.assets/image-20201201174753802.png) | 使用`@SessionAttributes`注释。 |
| ------------------------------------------------------------ | ------------------------------ |
|                                                              |                                |

在第一个请求上，将名称为的模型属性`pet`添加到模型时，该属性会**自动提升**到**HTTP Servlet会话**并保存在该会话中。它会一直保留在那里，直到另一个控制器方法使用`SessionStatus`方法参数来**清除存储**，如以下示例所示：

```java
@Controller
@SessionAttributes("pet") 
public class EditPetForm {

    // ...

    @PostMapping("/pets/{id}")
    public String handle(Pet pet, BindingResult errors, SessionStatus status) {
        if (errors.hasErrors) {
            // ...
        }
            status.setComplete(); 
            // ...
        }
    }
}
```

| ![image-20201201174832000](Spring%20Web%20MVC.assets/image-20201201174832000.png) | 将`Pet`值存储在Servlet会话中。   |
| ------------------------------------------------------------ | -------------------------------- |
| ![image-20201201174838091](Spring%20Web%20MVC.assets/image-20201201174838091.png) | `Pet`从Servlet会话中**清除值。** |

#### @SessionAttribute

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-sessionattribute)

如果您需要访问**全局存在**（即在控制器外部（例如，通过过滤器）管理）并且可能存在或可能不存在的预先存在的**会话属性**，则可以`@SessionAttribute`在方法参数上使用注释，如下所示示例显示：

```java
@RequestMapping("/")
public String handle(@SessionAttribute User user) { 注释：1
    // ...
}
```

| ![image-20201201174919841](Spring%20Web%20MVC.assets/image-20201201174919841.png) | 使用`@SessionAttribute`注释。 |
| ------------------------------------------------------------ | ----------------------------- |
|                                                              |                               |

对于用例需要**添加或删除**会话属性，可以考虑注入 `org.springframework.web.context.request.WebRequest`或 `javax.servlet.http.HttpSession`到控制器的方法。

要将模型属性临时存储在会话中作为**控制器工作流**的一部分，请考虑使用`@SessionAttributes`中所述 [`@SessionAttributes`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-sessionattributes)。

#### @RequestAttribute

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestattrib)

与相似`@SessionAttribute`，您可以使用`@RequestAttribute`批注来访问先前创建的==预先存在的请求属性==（例如，通过Servlet`Filter` 或`HandlerInterceptor`）：

```java
@GetMapping("/")
public String handle(@RequestAttribute Client client) { 注释: 1
    // ...
}
```

| ![image-20201201175026040](Spring%20Web%20MVC.assets/image-20201201175026040.png) | 使用`@RequestAttribute`注释。 |
| ------------------------------------------------------------ | ----------------------------- |
|                                                              |                               |

#### Redirect Attributes 重定向属性



默认情况下，**所有模型属性**均被视为在重定向URL中作为URI模板变量**公开**。在其余属性中，那些属于原始类型或原始类型的集合或数组的属性会**自动附加**为查询参数。

如果专门为重定向准备了模型实例，则将原始类型属性作为查询参数附加可能是**理想的结果**。但是，在带注释的控制器中，模型可以包含为渲染目的添加的其他属性（例如，下拉字段值）。为避免此类属性出现在URL中的可能性，`@RequestMapping`方法可以声明类型的自变量，`RedirectAttributes`并使用它来指定要提供给的确切属性`RedirectView`。如果该方法确实**重定向**，`RedirectAttributes`则使用的内容。否则，将使用模型的内容。

在`RequestMappingHandlerAdapter`提供了一个名为标志 `ignoreDefaultModelOnRedirect`，你可以用它来表示**默认的内容** `Model`不应该，如果一个控制器方法重定向使用。相反，控制器方法应该声明一个类型的属性，`RedirectAttributes`或者如果没有声明，则不应将任何属性传递给`RedirectView`。MVC命名空间和MVC Java配置都将此**标志设置为`false`**，以保持向后兼容性。但是，对于新应用程序，我们建议将其==设置为`true`==。

请注意，展开重定向URL时，本请求中的URI模板变量会自动变为可用，并且您无需通过`Model`或显式添加它们`RedirectAttributes`。以下示例显示了如何定义重定向：

```java
@PostMapping("/files/{path}")
public String upload(...) {
    // ...
    return "redirect:files/{path}";
}
```

将数据传递到重定向目标的另一种方法是==使用闪存属性==。与其他重定向属性不同，==Flash属性==保存在HTTP会话中（因此不会出现在URL中）。有关更多信息，请参见[Flash属性](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-flash-attributes)。

#### Flash Attributes (Flash属性)

Flash属性为一个请求提供了一种存储打算在另一个请求中使用的属性的方式。重定向时最常需要此功能，例如Post-Redirect-Get模式。Flash属性在重定向之前（通常在会话中）被**临时保存**，以便在重定向之后可供请求使用，并**立即**被删除。

Spring MVC有两个主要的**抽象**来支持Flash属性。`FlashMap`用于保存Flash属性，而`FlashMapManager`用于存储，检索和管理 `FlashMap`实例。

Flash属性支持始终处于“打开”状态，无需显式启用。但是，如果不使用它，则永远不会导致HTTP会话创建。在每个请求上，都有一个“输入” `FlashMap`，该属性具有从前一个请求（如果有）传递过来的属性，而“输出”则`FlashMap`具有为后一个请求保存的属性。`FlashMap` 可以通过==Spring中的静态方法==从Spring MVC中的任何位置访问这两个实例 `RequestContextUtils`。

**带注释**的控制器通常不需要`FlashMap`直接使用。取而代之的是， `@RequestMapping`方法可以接受类型的参数，`RedirectAttributes`并使用它为重定向方案添加Flash属性。通过添加的Flash属性将 `RedirectAttributes`自动传播到“输出” FlashMap。同样，重定向后，来自**“输入”**的属性`FlashMap`会自动添加到 `Model`服务于**目标URL的控制器**的。

```
												将请求与Flash属性匹配
												
Flash属性的概念存在于许多其他Web框架中，并已证明有时会遇到并发问题。这是因为根据定义，闪存属性将存储到下一个请求。但是，“下一个”请求可能不是预期的接收者，而是另一个异步请求（例如，轮询或资源请求），在这种情况下，过早删除了闪存属性。

为了减少此类问题的可能性，请使用目标重定向URL的路径和查询参数RedirectView自动“标记” FlashMap实例。反过来，默认值会FlashMapManager在查找“输入”时将该信息与传入请求进行匹配FlashMap。

这不能完全消除并发问题的可能性，但是可以通过重定向URL中已经可用的信息大大减少并发问题。因此，我们建议您主要将Flash属性用于重定向方案。
```

#### Multipart 多部分

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-multipart-forms)

一个后`MultipartResolver`已经[启用](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multipart)，POST的内容与要求`multipart/form-data`进行解析，并==定期请求参数进行访问==。以下示例访问一个常规表单字段和一个上载文件：

Java:

```java
@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(@RequestParam("name") String name,
            @RequestParam("file") MultipartFile file) {

        if (!file.isEmpty()) {
            byte[] bytes = file.getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```

将参数类型声明为List <MultipartFile>允许解析相同参数名称的**多个文件**。

如果将`@RequestParam`注释声明为`Map<String, MultipartFile>`或 `MultiValueMap<String, MultipartFile>`，而未在注释中指定**参数名称**，则将使用每个给定参数名称的多部分文件来**填充映射**。

| ![image-20201201175416375](Spring%20Web%20MVC.assets/image-20201201175416375.png) | 使用Servlet 3.0多部分解析时，您还可以声明`javax.servlet.http.Part` Spring而不是Spring`MultipartFile`作为方法参数或集合值类型。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

您还可以将多部分内容用作数据绑定到 [命令对象的一部分](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-modelattrib-method-args)。例如，前面示例中的表单字段和文件可以是表单对象上的字段，如以下示例所示：

```java
class MyForm {

    private String name;

    private MultipartFile file;

    // ...
}

@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(MyForm form, BindingResult errors) {
        if (!form.getFile().isEmpty()) {
            byte[] bytes = form.getFile().getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```

在RESTful服务方案中，也可以从**非浏览器客户端**提交多部分请求。以下示例显示了一个带有JSON的文件：

```
POST / someUrl
内容类型：multipart / mixed 

--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
内容处置：form-data; name =“元数据” 
Content-Type：application / json; charset = UTF-8 
Content-Transfer-Encoding：8bit 

{ 
    “ name”：“ value” 
} 
--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp 
Content-Disposition：表格数据；name =“文件数据”; filename =“ file.properties”
内容类型：text / xml
内容传输编码：8位
...文件数据...
```

您可以通过访问==“元数据”部分==`@RequestParam`的`String`，但你可能会想从JSON反序列化（类似`@RequestBody`）。在使用[HttpMessageConverter](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-message-conversion)`@RequestPart`将注释转换为多部分后，可使用 注释来访问它 ：

```java
@PostMapping("/")
public String handle(@RequestPart("meta-data") MetaData metadata,
        @RequestPart("file-data") MultipartFile file) {
    // ...
}
```

您可以`@RequestPart`与`javax.validation.Valid`Spring的`@Validated`注释==结合使用或一起使用== ，这两种注释都会导致应用**标准Bean验证**。默认情况下，验证错误会导致`MethodArgumentNotValidException`，并变成400（BAD_REQUEST）响应。或者，您可以通过`Errors`或`BindingResult`参数在控制器内本地处理验证错误，如以下示例所示：

```java
@PostMapping("/")
public String handle(@Valid @RequestPart("meta-data") MetaData metadata,
        BindingResult result) {
    // ...
}
```

#### @RequestBody

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestbody)

您可以使用`@RequestBody`注释有请求体**读取和反序列化**到一个 `Object`通过[`HttpMessageConverter`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-message-conversion)。下面的示例使用一个`@RequestBody`参数：

```java
@PostMapping("/accounts")
public void handle(@RequestBody Account account) {
    // ...
}
```

您可以使用[MVC Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config)的“[消息转换器”](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-message-converters)选项来配置或自定义消息转换。

可以`@RequestBody`与`javax.validation.Valid`或Spring的 `@Validated`注释结合使用，这两种注释都会导致应用标准Bean验证。默认情况下，验证错误会导致`MethodArgumentNotValidException`，并变成400（BAD_REQUEST）响应。或者，您可以通过`Errors`或`BindingResult`参数在控制器内本地处理验证错误，如以下示例所示：

```java
@PostMapping("/accounts")
public void handle(@Valid @RequestBody Account account, BindingResult result) {
    // ...
}
```

#### HttpEntity （http实体）

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestbody)

`HttpEntity`与使用大致相同，[`@RequestBody`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestbody)但基于==公开请求标头和正文的容器对象==。以下清单显示了一个示例：

```java
@PostMapping("/accounts")
public void handle(HttpEntity<Account> entity) {
    // ...
}
```

#### @ResponseBody

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestbody)

您可以`@ResponseBody`在方法上使用批注，以通过[HttpMessageConverter](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-message-conversion)将返回序列化为响应主体 。以下清单显示了一个示例：

```java
@GetMapping("/accounts/{id}")
@ResponseBody
public Account handle() {
    // ...
}
```

`@ResponseBody`在**类级别**也受支持，在这种情况下，它由所有控制器方法继承。这就是的效果`@RestController`，无非就是带有`@Controller`和标记的元注释`@ResponseBody`。

您可以使用`@ResponseBody`反应类型。有关更多详细信息，[请](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async)参见[异步请求](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async)和[响应类型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-reactive-types)。

您可以使用[MVC Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config)的“[消息转换器”](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-message-converters)选项来配置或自定义消息转换。

您可以将`@ResponseBody`方法与**JSON序列化视图**结合使用。有关详细信息，请参见[Jackson JSON](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-jackson)。

#### ResponseEntity (响应实体)

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestbody)

`ResponseEntity`就像[`@ResponseBody`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-responsebody)但带有**状态和标题**。例如：

```java
@GetMapping("/something")
public ResponseEntity<String> handle() {
    String body = ... ;
    String etag = ... ;
    return ResponseEntity.ok().eTag(etag).build(body);
}
```

Spring MVC支持使用单值[反应类型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-reactive-types) 来`ResponseEntity`为**主体**生成异步和/或单值和多值反应类型。

####  Jackson JSON ( 杰克逊JSON)

Spring提供了对Jackson JSON库的支持。

#### JSON Views (**JSON视图**)

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-jsonview)

Spring MVC为[Jackson的序列化视图](https://www.baeldung.com/jackson-json-view-annotation)提供了内置支持 ，该[视图](https://www.baeldung.com/jackson-json-view-annotation)仅可**呈现.NET中所有字段**的一部分`Object`。要将其与 `@ResponseBody`或`ResponseEntity`控制器方法一起使用，可以使用Jackson的 `@JsonView`注释来激活序列化视图类，如以下示例所示：

```java
@RestController
public class UserController {

    @GetMapping("/user")
    @JsonView(User.WithoutPasswordView.class)
    public User getUser() {
        return new User("eric", "7!jd#h23");
    }
}

public class User {

    public interface WithoutPasswordView {};
    public interface WithPasswordView extends WithoutPasswordView {};

    private String username;
    private String password;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @JsonView(WithoutPasswordView.class)
    public String getUsername() {
        return this.username;
    }

    @JsonView(WithPasswordView.class)
    public String getPassword() {
        return this.password;
    }
}
```

| ![image-20201201181200514](Spring%20Web%20MVC.assets/image-20201201181200514.png) | `@JsonView`允许一组视图类，但是每个控制器方法只能指定一个。如果需要激活多个视图，则可以使用==复合界面==。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

如果要以编程方式执行上述操作，而不是声明`@JsonView`注释，则将返回值包装为`MappingJacksonValue`并用于提供**序列化视图**：

```java
@RestController
public class UserController {

    @GetMapping("/user")
    public MappingJacksonValue getUser() {
        User user = new User("eric", "7!jd#h23");
        MappingJacksonValue value = new MappingJacksonValue(user);
        value.setSerializationView(User.WithoutPasswordView.class);
        return value;
    }
}
```

对于**依赖视图分辨率**的控制器，可以将序列化视图类添加到模型中，如以下示例所示：

```java
@Controller
public class UserController extends AbstractController {

    @GetMapping("/user")
    public String getUser(Model model) {
        model.addAttribute("user", new User("eric", "7!jd#h23"));
        model.addAttribute(JsonView.class.getName(), User.WithoutPasswordView.class);
        return "userView";
    }
}
```

### 1.3.4. Model 模型

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-modelattrib-methods)

您可以使用`@ModelAttribute`注释：

- 在[方法](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-modelattrib-method-args)中的[方法参数](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-modelattrib-method-args)`@RequestMapping`上创建或访问`Object`模型的，并将其通过绑定到请求 `WebDataBinder`。
- 作为`@Controller`或方法`@ControllerAdvice`类中的方法级别注释，有助于在任何`@RequestMapping`方法调用之前初始化模型。
- `@RequestMapping`标记其返回值的方法是**模型属性。**

本节讨论`@ModelAttribute`方法-前面列表中的第二项。控制器可以有多种`@ModelAttribute`方法。`@RequestMapping`在同一个控制器中，所有这些方法均在方法之前被调用。`@ModelAttribute` 也可以通过跨控制器共享一种方法`@ControllerAdvice`。有关更多详细信息，请参见“[控制器建议](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-controller-advice)”部分 。

`@ModelAttribute`方法具有灵活的方法签名。它们支持许多与`@RequestMapping`方法相同的参数，除了`@ModelAttribute`自身或与请求主体相关的任何东西。

以下示例显示了一种`@ModelAttribute`方法：

```java
@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountRepository.findAccount(number));
    // add more ...
}
```

以下示例仅添加一个属性：

```java
@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountRepository.findAccount(number);
}
```

| ![image-20201201181419621](Spring%20Web%20MVC.assets/image-20201201181419621.png) | 如果未明确指定名称，则根据`Object` 类型选择默认名称，如javadoc中对的解释[`Conventions`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/core/Conventions.html)。您始终可以使用重载`addAttribute`方法或通过`name`on的属性`@ModelAttribute`（用于返回值）来分配==显式名称==。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

您还可以`@ModelAttribute`在方法上用作方法级注释`@RequestMapping`，在这种情况下，方法的返回值将`@RequestMapping`解释为模型属性。通常不需要这样做，因为这是**HTML控制器**的默认行为，除非返回值是a `String`，否则它将被解释为视图名称。 `@ModelAttribute`还可以自定义模型属性名称，如以下示例所示：

```java
@GetMapping("/accounts/{id}")
@ModelAttribute("myAccount")
public Account handle() {
    // ...
    return account;
}
```

### 1.3.5. DataBinder 数据绑定器

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-initbinder)

`@Controller`或`@ControllerAdvice`类可以具有`@InitBinder`初始化的实例的方法`WebDataBinder`，而这些实例又可以：

- 将请求参数（即表单或查询数据）绑定到模型对象。
- 将基于字符串的请求值（例如请求参数，路径变量，标头，Cookie等）转换为控制器方法参数的**目标类型**。
- `String`呈现HTML表单时，将模型对象值格式化为值。

`@InitBinder`方法可以==注册控制器特异性==`java.beans.PropertyEditor`或Spring`Converter`和`Formatter`组件。此外，您可以使用 [MVC配置](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-conversion) 在全局共享中注册`Converter`和`Formatter`键入`FormattingConversionService`。

`@InitBinder``@RequestMapping`除了`@ModelAttribute`（命令对象）参数外，方法还支持许多与方法相同的参数。通常，它们使用一个`WebDataBinder`参数（用于注册）和一个`void`返回值进行声明。以下清单显示了一个示例：

```java
@Controller
public class FormController {

    @InitBinder  注释： 1
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }

    // ...
}
```

| ![image-20201201181610966](Spring%20Web%20MVC.assets/image-20201201181610966.png) | 定义`@InitBinder`方法。 |
| ------------------------------------------------------------ | ----------------------- |
|                                                              |                         |

另外，当您`Formatter`通过**shared**使用基于设置时 `FormattingConversionService`，可以**重新**使用相同的方法并注册特定于控制器的`Formatter`实现，如以下示例所示：

```java
@Controller
public class FormController {

    @InitBinder 注释： 1
    protected void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
    }

    // ...
}
```

| ![image-20201201181649482](Spring%20Web%20MVC.assets/image-20201201181649482.png) | `@InitBinder`在自定义格式化程序上定义方法。 |
| ------------------------------------------------------------ | ------------------------------------------- |
|                                                              |                                             |

### 1.3.6. Exceptions 异常

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-controller-exceptions)

`@Controller`和[@ControllerAdvice](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-controller-advice)类可以具有 `@ExceptionHandler`处理控制器方法异常的方法，如以下示例所示：

```java
@Controller
public class SimpleController {

    // ...

    @ExceptionHandler
    public ResponseEntity<String> handle(IOException ex) {
        // ...
    }
}
```

该异常可能与正在传播的**顶级异常**（即，直接`IOException`引发的异常）匹配，也可能与顶级包装程序异常（例如，`IOException`包装在内`IllegalStateException`）的**直接原因**匹配 。

对于匹配的异常类型，如前面的示例所示，最好将目标异常声明为方法参数。当多个异常方法匹配时，==根源异常匹配==通常比原因异常匹配更可取。更具体地说，`ExceptionDepthComparator`用来根据异常从引发的异常类型的深度对异常进行排序。

另外，注释声明可以缩小异常类型以使其匹配，如以下示例所示：

```java
@ExceptionHandler({FileSystemException.class, RemoteException.class})
public ResponseEntity<String> handle(IOException ex) {
    // ...
}
```

您甚至可以使用带有非常通用的参数签名的特定异常类型的列表，如以下示例所示：

```java
@ExceptionHandler({FileSystemException.class, RemoteException.class})
public ResponseEntity<String> handle(Exception ex) {
    // ...
}
```

| <img src="Spring%20Web%20MVC.assets/image-20201201181911281.png" alt="image-20201201181911281" style="zoom:200%;" /> | 根和原因异常匹配之间的区别可能令人惊讶。在`IOException`前面显示的变体中，通常以实际`FileSystemException`或`RemoteException`实例作为参数来调用该方法，因为这两个方法都从扩展`IOException`。但是，如果任何此类匹配异常都在本身为的包装器异常中传播`IOException`，则传入的异常实例就是该包装器异常。在`handle(Exception)`变体中，行为甚至更简单。在包装方案中，总是使用包装程序异常来调用此方法，`ex.getCause()`在这种情况下，将找到实际匹配的异常。仅当将它们作为顶级异常抛出时，传入的异常才是实际的`FileSystemException`或 `RemoteException`实例。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

我们通常建议您在==参数签名==中尽可能具体，以减少根类型和原因异常类型之间不匹配的可能性。考虑将多重匹配方法分解为单个`@ExceptionHandler` 方法，每个方法都通过其签名匹配单个特定的异常类型。

在多种`@ControllerAdvice`安排中，我们建议声明`@ControllerAdvice`优先级并以相应顺序声明您的**主根异常映射**。尽管根异常匹配优先于原因，但这是在给定控制器或`@ControllerAdvice`类的方法之间定义的。这意味着优先级较高的`@ControllerAdvice`Bean上的原因匹配优于优先级 较低的`@ControllerAdvice`Bean上的任何匹配（例如，根） 。

最后但并非最不重要的一点是，`@ExceptionHandler`方法实现可以选择通过以**原始形式**重新抛出异常来退出处理给定异常实例。在仅对根级别匹配或无法静态确定的特定上下文中的匹配感兴趣的情况下，这很有用。重新抛出的异常会在其余的解决方案链中传播，就像给定的`@ExceptionHandler`方法最初不会匹配一样。

`@ExceptionHandler`Spring MVC中对方法的支持建立在[HandlerExceptionResolver](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-exceptionhandlers)机制`DispatcherServlet` 级别上。

####  Method Arguments 方法参数

`@ExceptionHandler` 方法支持以下参数：

| 方法参数                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 异常类型                                                     | 用于访问引发的异常。                                         |
| `HandlerMethod`                                              | 用于访问引发异常的控制器方法。                               |
| `WebRequest`， `NativeWebRequest`                            | 对请求参数以及请求和会话属性的常规访问，而**无需直接**使用Servlet API。 |
| `javax.servlet.ServletRequest`， `javax.servlet.ServletResponse` | 选择任何特定的请求或响应类型（例如`ServletRequest`或 `HttpServletRequest`或Spring的`MultipartRequest`或`MultipartHttpServletRequest`）。 |
| `javax.servlet.http.HttpSession`                             | **强制会话的存在**。结果，这种论据永远不会`null`。 请注意，会话访问不是线程安全的。考虑将`RequestMappingHandlerAdapter`实例的`synchronizeOnSession`标志设置 为`true`是否允许多个请求同时访问会话。 |
| `java.security.Principal`                                    | 当前经过身份验证的用户-可能是特定的`Principal`实现类（如果已知）。 |
| `HttpMethod`                                                 | 请求的HTTP方法。                                             |
| `java.util.Locale`                                           | 当前请求的语言环境，取决于最具体的`LocaleResolver`可用语言（实际上是配置的`LocaleResolver`或）`LocaleContextResolver`。 |
| `java.util.TimeZone`， `java.time.ZoneId`                    | 与当前请求关联的时区，由决定`LocaleContextResolver`。        |
| `java.io.OutputStream`， `java.io.Writer`                    | 用于访问原始响应主体，如**Servlet API**所公开。              |
| `java.util.Map`，`org.springframework.ui.Model`，`org.springframework.ui.ModelMap` | 用于访问模型以进行错误响应。永远是空的。                     |
| `RedirectAttributes`                                         | 指定在重定向的情况下要使用的属性（将附加到查询字符串中）和flash属性，这些属性将临时存储直到重定向后的请求。请参阅[重定向属性](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-redirecting-passing-data)和[Flash属性](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-flash-attributes)。 |
| `@SessionAttribute`                                          | 与访问由于类级`@SessionAttributes`声明而存储在会话中的模型属性相反，用于访问任何会话属性。请参阅[`@SessionAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-sessionattribute)以获取更多详细信息。 |
| `@RequestAttribute`                                          | 用于访问请求属性。请参阅[`@RequestAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestattrib)以获取更多详细信息。 |

#### Return Values 返回值

`@ExceptionHandler` 方法支持以下返回值：

| 返回值                                           | 描述                                                         |
| :----------------------------------------------- | :----------------------------------------------------------- |
| `@ResponseBody`                                  | 返回值通过`HttpMessageConverter`实例转换并写入响应。请参阅[`@ResponseBody`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-responsebody)。 |
| `HttpEntity<B>`， `ResponseEntity<B>`            | 返回值指定完整的响应（包括HTTP标头和正文）将通过`HttpMessageConverter`实例转换并写入响应。参见[ResponseEntity](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-responseentity)。 |
| `String`                                         | 一个视图名称，将通过`ViewResolver`实现来解析，并与隐式模型一起使用-通过命令对象和`@ModelAttribute`方法确定。该处理程序方法还可以通过声明一个`Model` 参数（如前所述）以编程方式丰富模型。 |
| `View`                                           | 甲`View`实例以使用用于与所述**隐式模型**一起渲染-通过命令对象和确定`@ModelAttribute`方法。该处理程序方法还可以通过声明一个自`Model`变量（如前所述）以**编程方式**丰富模型。 |
| `java.util.Map`， `org.springframework.ui.Model` | 要添加到隐式模型的属性，其视图名称是通过隐式确定的`RequestToViewNameTranslator`。 |
| `@ModelAttribute`                                | 要添加到模型的属性，其视图名称通过**隐式**确定`RequestToViewNameTranslator`。请注意，这`@ModelAttribute`是可选的。请参见表末尾的“其他任何返回值”。 |
| `ModelAndView` 目的                              | 要使用的**视图和模型属性**，以及**响应状态**（可选）。       |
| `void`                                           | 用的方法`void`返回类型（或`null`返回值）被认为已经完全处理的响应，如果它也具有`ServletResponse`一个`OutputStream`参数，或`@ResponseStatus`注释。如果控制器进行了肯定`ETag`或`lastModified`时间戳检查，则情况也是如此 （有关详细信息，请参阅[控制器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching-etag-lastmodified)）。如果以上条件都不成立，则`void`返回类型还可以为**REST控制器**指示“无响应正文”，或者为HTML控制器指示默认视图名称选择。 |
| 任何其他返回值                                   | 如果返回值与上述任何一个都不匹配且不是简单类型（由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定 ），则会将其视为要添加到模型的模型属性。如果它是简单类型，则仍然无法解析。 |

#### REST API exceptions (REST API例外)

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-rest-exceptions)

REST服务的常见要求是在**响应正文中**包含错误详细信息。Spring框架不会自动执行此操作，因为响应主体中错误详细信息的表示是特定于应用程序的。但是，a `@RestController`可以使用`@ExceptionHandler`带有`ResponseEntity`返回值的方法来**设置响应的状态和主体**。也可以在`@ControllerAdvice`类中声明此类方法以将其**全局应用**。

在响应主体中实现具有错误详细信息的全局异常处理的应用程序应考虑extend [`ResponseEntityExceptionHandler`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/mvc/method/annotation/ResponseEntityExceptionHandler.html)，它提供对Spring MVC引发的异常的处理，并提供用于自定义响应主体的钩子。要使用此功能，请创建的子类 `ResponseEntityExceptionHandler`，用进行注释`@ControllerAdvice`，**覆盖**必要的方法，然后将其声明为Spring bean。

### 1.3.7. Controller Advice (增强控制器)

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-controller-advice)

通常`@ExceptionHandler`，，`@InitBinder`和`@ModelAttribute`方法适用于`@Controller`声明它们的类（或类层次结构）。如果要使此类方法更全局地应用（跨控制器），则可以在带有`@ControllerAdvice`或注释的类中声明它们`@RestControllerAdvice`。

`@ControllerAdvice`带有注释`@Component`，这意味着可以通过[组件扫描](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-instantiating-container-scan)将此类注册为Spring Bean 。`@RestControllerAdvice`是用`@ControllerAdvice`和注释的组合注释`@ResponseBody`，从本质上讲意味着 `@ExceptionHandler`方法是通过消息转换（与视图分辨率或模板渲染相比）呈现给**响应主体**的。

在启动时，用于`@RequestMapping`和`@ExceptionHandler` 方法的基础结构类检测使用注释的Spring bean `@ControllerAdvice`，然后在运行时应用其方法。全局`@ExceptionHandler`方法（来自`@ControllerAdvice`）适用*于*局部方法（来自`@Controller`）。相比之下，全局`@ModelAttribute` 和`@InitBinder`方法*先于*局部方法。

默认情况下，`@ControllerAdvice`方法适用于每个请求（即**所有控制器**），但是您可以通过使用批注上的属性将其范围缩小到**控制器的子集**，如以下示例所示：

```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```

前面示例中的选择器在运行时进行评估，如果广泛使用，可能会==对性能产生负面影响==。有关[`@ControllerAdvice`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/bind/annotation/ControllerAdvice.html) 更多详细信息，请参见 javadoc。

## 1.4. Functional Endpoints 功能端点

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn)

Spring Web MVC包含WebMvc.fn，这是一个==轻量级的函数编程模型==，其中的函数用于路由和处理请求，而**契约**则是为不变性而设计的。它是**基于注释**的编程模型的替代方案，但可以在同一[DispatcherServlet](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet)上运行。

### 1.4.1. Overview 总览

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn-overview)

在WebMvc.fn中，HTTP请求使用来处理`HandlerFunction`：一个接受`ServerRequest`并返回的函数 `ServerResponse`。请求和响应对象都具有**不可变的协定**，这些协定为JDK 8提供了对HTTP请求和响应的友好访问。 与基于注释的编程模型中方法`HandlerFunction`的主体等效`@RequestMapping`。

传入的请求使用：**路由到处理程序函数**，`RouterFunction`该函数采用`ServerRequest`并返回可选`HandlerFunction`（即`Optional<HandlerFunction>`）。当路由器功能匹配时，返回处理程序功能。否则为空的Optional。 `RouterFunction`与`@RequestMapping`注解等效，但**主要区别**在于路由器功能不仅提供数据，还提供行为。

`RouterFunctions.route()` 提供了一个有助于构建路由器的路由器构建器，如以下示例所示：

Java:

```java
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.servlet.function.RequestPredicates.*;
import static org.springframework.web.servlet.function.RouterFunctions.route;

PersonRepository repository = ...
PersonHandler handler = new PersonHandler(repository);

RouterFunction<ServerResponse> route = route()
    .GET("/person/{id}", accept(APPLICATION_JSON), handler::getPerson)
    .GET("/person", accept(APPLICATION_JSON), handler::listPeople)
    .POST("/person", handler::createPerson)
    .build();


public class PersonHandler {

    // ...

    public ServerResponse listPeople(ServerRequest request) {
        // ...
    }

    public ServerResponse createPerson(ServerRequest request) {
        // ...
    }

    public ServerResponse getPerson(ServerRequest request) {
        // ...
    }
}
```

如果将RouterFunction注册为Bean（例如，通过将其暴露在@Configuration类中），则Servlet将**自动检测**它，如[运行服务器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#webmvc-fn-running)中所述。

###  1.4.2. HandlerFunction 处理函数

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn-handler-functions)

ServerRequest和ServerResponse是==不可变的接口==，它们提供**JDK 8**友好的HTTP请求和响应访问，包括标头，正文，方法和状态代码。

#### ServerRequest  服务器请求

`ServerRequest`提供对HTTP方法，URI，标头和查询参数的访问，而通过这些`body`方法提供对正文的访问。

以下示例将请求正文提取到`String`：

```java
String string = request.body(String.class);
```

以下示例将主体提取到`List<Person>`，其中`Person` objects are decoded from a serialized form, such as JSON or XML:

```java
List<Person> people = request.body(new ParameterizedTypeReference<List<Person>>() {});
```

以下示例显示如何访问参数：

```java
MultiValueMap<String, String> params = request.params();
```

#### ServerResponse	服务器响应

`ServerResponse`提供对HTTP响应的访问，并且由于它是不可变的，因此可以使用一种**`build`方法**来创建它。您可以使用构建器来设置**响应状态**，添加响应标题或提供正文。以下示例使用JSON内容创建**200**（确定）响应：

```java
Person person = ...
ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(person);
```

下面的示例演示如何构建`Location`不带标头的201（已创建）响应：

```java
URI location = ...
ServerResponse.created(location).build();
```

您还可以将异步结果用作主体，形式为CompletableFuture，Publisher或ReactiveAdapterRegistry支持的任何其他类型。例如：

```java
Mono<Person> person = webClient.get().retrieve().bodyToMono(Person.class);
ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(person);
```

如果不只是身体，而且**状态或标题**是基于异步类型，你可以使用静态`async`的方法`ServerResponse`，它接受`CompletableFuture<ServerResponse>`，`Publisher<ServerResponse>`或通过支持的任何其他异步类型`ReactiveAdapterRegistry`。例如：

```java

Mono<ServerResponse> asyncResponse = webClient.get().retrieve().bodyToMono(Person.class)
  .map(p -> ServerResponse.ok().header("Name", p.name()).body(p));
ServerResponse.async(asyncResponse);
```

#### Handler Classes 处理程序类

我们可以==将处理程序函数编写为lambda==，如以下示例所示：

```java
HandlerFunction<ServerResponse> helloWorld =
  request -> ServerResponse.ok().body("Hello World");
```

这很方便，但是在应用程序中我们需要多个功能，并且**多个内联lambda**可能会变得凌乱。因此，将相关的处理程序功能分组到一个处理程序类中很有用，该类具有与`@Controller`基于注释的应用程序类似的作用。例如，以下类公开了反应式`Person`存储库：

Java:

```java
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.reactive.function.server.ServerResponse.ok;

public class PersonHandler {

    private final PersonRepository repository;

    public PersonHandler(PersonRepository repository) {
        this.repository = repository;
    }

    public ServerResponse listPeople(ServerRequest request) { 注释: 1
        List<Person> people = repository.allPeople();
        return ok().contentType(APPLICATION_JSON).body(people);
    }

    public ServerResponse createPerson(ServerRequest request) throws Exception { 注释: 2
        Person person = request.body(Person.class);
        repository.savePerson(person);
        return ok().build();
    }

    public ServerResponse getPerson(ServerRequest request) { 注释: 3
        int personId = Integer.parseInt(request.pathVariable("id"));
        Person person = repository.getPerson(personId);
        if (person != null) {
            return ok().contentType(APPLICATION_JSON).body(person);
        }
        else {
            return ServerResponse.notFound().build();
        }
    }

}
```

| ![image-20201201194742002](Spring%20Web%20MVC.assets/image-20201201194742002.png) | listPeople是一个处理函数，它以以下形式返回在存储库中找到的所有Person对象： JSON。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image-20201201194841749](Spring%20Web%20MVC.assets/image-20201201194841749.png) | `createPerson`是一个**处理程序功能**，用于存储`Person`请求正文中包含的新内容。 |
| ![image-20201201194902786](Spring%20Web%20MVC.assets/image-20201201194902786.png) | `getPerson`是一个**处理程序函数**，该函数返回由`id`path变量标识的一个人。我们`Person`从存储库中检索到该内容，并**创建一个JSON响应**（如果找到）。如果找不到，我们将返回404 Not Found响应。 |

#### Validation 验证方式

功能端点可以使用Spring的[验证工具](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation)将验证应用于**请求主体**。例如，给定一个定制的Spring [Validator](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation)实现`Person`：

```java
public class PersonHandler {

    private final Validator validator = new PersonValidator(); 注释：1

    // ...

    public ServerResponse createPerson(ServerRequest request) {
        Person person = request.body(Person.class);
        validate(person); 注释：2
        repository.savePerson(person);
        return ok().build();
    }

    private void validate(Person person) {
        Errors errors = new BeanPropertyBindingResult(person, "person");
        validator.validate(person, errors);
        if (errors.hasErrors()) {
            throw new ServerWebInputException(errors.toString()); 注释：3
        }
    }
}
```

| ![image-20201201195421676](Spring%20Web%20MVC.assets/image-20201201195421676.png) | 创建`Validator`实例。 |
| ------------------------------------------------------------ | --------------------- |
| ![image-20201201195453104](Spring%20Web%20MVC.assets/image-20201201195453104.png) | Apply validation.     |
| ![image-20201201195509464](Spring%20Web%20MVC.assets/image-20201201195509464.png) | 引发400响应的异常。   |

处理程序还可以通过创建和注入`Validator`基于的**全局实例**来使用标准bean验证API（JSR-303）`LocalValidatorFactoryBean`。请参阅[Spring验证](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation-beanvalidation)。

### 1.4.3. RouterFunction 路由器功能

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn-router-functions)

路由器功能用于将请求路由到相应的`HandlerFunction`。通常，您不是自己编写路由器功能，而是使用`RouterFunctions`实用程序类上的方法 创建一个。 `RouterFunctions.route()`（无参数）为您提供了一个**流畅的生成器**来创建路由器功能，而`RouterFunctions.route(RequestPredicate, HandlerFunction)`提供了一种直接的方式来创建路由器。

通常，建议==使用`route()`构建器==，因为它为典型的映射场景提供了便捷的快捷方式，而无需发现**静态导入**。例如，路由器功能构建器提供了`GET(String, HandlerFunction)`为**GET请求**创建映射的方法。和`POST(String, HandlerFunction)`POST。

除了基于HTTP方法的映射外，路由构建器还提供了一种在**映射**到请求时引入其他谓词的方法。对于每个HTTP方法，都有一个以a`RequestPredicate`作为参数的重载变体，不过可以表达其他约束。

####  Predicates 谓词

您可以编写自己的`RequestPredicate`，但是`RequestPredicates`实用程序类根据请求路径，HTTP方法，内容类型等提供常用的实现。以下示例使用请求谓词基于`Accept` 标头创建约束：

```java
RouterFunction<ServerResponse> route = RouterFunctions.route()
    .GET("/hello-world", accept(MediaType.TEXT_PLAIN),
        request -> ServerResponse.ok().body("Hello World")).build();
```

```java
        request -> ServerResponse.ok().body("Hello World")).build();
```

您可以使用以下命令组合多个请求谓词：

- `RequestPredicate.and(RequestPredicate)` -两者必须匹配。
- `RequestPredicate.or(RequestPredicate)` -两者都可以匹配。

的许多谓词`RequestPredicates`组成。例如，`RequestPredicates.GET(String)`由`RequestPredicates.method(HttpMethod)` 和组成`RequestPredicates.path(String)`。上面显示的示例还使用了两个请求谓词，因为构建器在`RequestPredicates.GET`内部使用 并将其与`accept`谓词组合在一起。

####  Routes 路线

路由器功能**按顺序**评估：如果第一个路由不匹配，则评估第二个路由，依此类推。因此，在通用路由之前声明更具体的路由是有意义的。请注意，此行为==不同于==基于注释的编程模型，在该模型中，将自动选择“最特定”的控制器方法。

使用路由器功能生成器时，所有定义的路由都组成一个`RouterFunction`从中返回的路由 `build()`。还有其他方法可以将多个路由器功能组合在一起：

- `add(RouterFunction)`在`RouterFunctions.route()`建造者上
- `RouterFunction.and(RouterFunction)`
- `RouterFunction.andRoute(RequestPredicate, HandlerFunction)` —`RouterFunction.and()`嵌套的快捷方式 `RouterFunctions.route()`。

以下示例显示了四种路线的组成：

```java
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.servlet.function.RequestPredicates.*;

PersonRepository repository = ...
PersonHandler handler = new PersonHandler(repository);

RouterFunction<ServerResponse> otherRoute = ...

RouterFunction<ServerResponse> route = route()
    .GET("/person/{id}", accept(APPLICATION_JSON), handler::getPerson) 注释: 1
    .GET("/person", accept(APPLICATION_JSON), handler::listPeople) 注释：2
    .POST("/person", handler::createPerson) 注释：3
    .add(otherRoute) 注释：4
    .build();
```

| ![image-20201202160028832](Spring%20Web%20MVC.assets/image-20201202160028832.png) | `GET /person/{id}`与`Accept`匹配JSON头被路由到 `PersonHandler.getPerson` |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image-20201202160043450](Spring%20Web%20MVC.assets/image-20201202160043450.png) | `GET /person`与`Accept`**匹配JSON头**被路由到 `PersonHandler.listPeople` |
| ![image-20201202160053004](Spring%20Web%20MVC.assets/image-20201202160053004.png) | `POST /person`没有其他谓词的映射到 `PersonHandler.createPerson`，并且 |
| ![image-20201202160103883](Spring%20Web%20MVC.assets/image-20201202160103883.png) | `otherRoute` 是在其他地方**创建并添加**到所建立路由的路由器功能。 |

#### Nested Routes 嵌套路线

一组路由器功能通常具有**共享谓词**，例如共享路径。在上面的示例中，共享谓词将是与其中`/person`三个路由使用的match匹配的路径谓词。使用注释时，您可以通过使用`@RequestMapping` 映射到的**类型级别**的注释来删除此重复项`/person`。在WebMvc.fn中，可以通过`path`路由器功能构建器上的方法**共享路径谓词**。例如，可以通过使用**嵌套路由**以以下方式改进上面示例的最后几行：

```java
RouterFunction<ServerResponse> route = route()
    .path("/person", builder -> builder 注释：1
        .GET("/{id}", accept(APPLICATION_JSON), handler::getPerson)
        .GET(accept(APPLICATION_JSON), handler::listPeople)
        .POST("/person", handler::createPerson))
    .build();
```

| ![image-20201202160150827](Spring%20Web%20MVC.assets/image-20201202160150827.png) | 请注意，的第二个参数`path`是使用路由器构建器的使用者。 |
| ------------------------------------------------------------ | ------------------------------------------------------ |
|                                                              |                                                        |

尽管基于路径的嵌套是最常见的，但是您可以通过使用`nest`构建器上的方法来嵌套在**任何种类的谓词**上。上面的内容仍然包含一些**共享头**`Accept`谓词形式的重复项。通过结合使用该`nest`方法，我们可以进一步改进`accept`：

```java
RouterFunction<ServerResponse> route = route()
    .path("/person", b1 -> b1
        .nest(accept(APPLICATION_JSON), b2 -> b2
            .GET("/{id}", handler::getPerson)
            .GET(handler::listPeople))
        .POST("/person", handler::createPerson))
    .build();
```

### 1.4.4. Running a Server

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn-running)

通常，您可以[`DispatcherHandler`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet)通过[MVC Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config)在基于的设置中运行路由器功能，该 [配置](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config)使用**Spring配置**来声明处理请求所需的组件。MVC Java配置声明以下基础结构组件以支持功能端点：

- `RouterFunctionMapping`：`RouterFunction<?>`在Spring配置中检测一个或多个bean，将它们组合在一起`RouterFunction.andOther`，并将请求路由到生成的composition `RouterFunction`。
- `HandlerFunctionAdapter`：简单的适配器，可以`DispatcherHandler`调用`HandlerFunction`映射到请求的。

前面的组件使功能端点适合于`DispatcherServlet`请求==处理生命周期==，并且（如果有）声明的控制器也可以（可能）与**带注释**的控制器并排运行。这也是Spring Boot Web启动程序如何**启用功能端点**的方式。

以下示例显示了WebFlux Java配置：

```java
@Configuration
@EnableMvc
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public RouterFunction<?> routerFunctionA() {
        // ...
    }

    @Bean
    public RouterFunction<?> routerFunctionB() {
        // ...
    }

    // ...

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        // configure message conversion...
    }

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // configure CORS...
    }

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        // configure view resolution for HTML rendering...
    }
}
```

### 1.4.5. Filtering Handler Functions 过滤处理程序功能

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn-handler-filter-function)

您可以通过使用过滤处理器功能`before`，`after`或`filter`在路由功能生成器方法。使用注释，您可以通过使用实现类似的功能`@ControllerAdvice`，一个`ServletFilter`，或两者兼而有之。该过滤器将应用于构建器构建的所有路由。这意味着在嵌套路由中定义的过滤器不适用于“顶级”路由。例如，考虑以下示例：

```java
RouterFunction<ServerResponse> route = route()
    .path("/person", b1 -> b1
        .nest(accept(APPLICATION_JSON), b2 -> b2
            .GET("/{id}", handler::getPerson)
            .GET(handler::listPeople)
            .before(request -> ServerRequest.from(request) 注释：1
                .header("X-RequestHeader", "Value")
                .build()))
        .POST("/person", handler::createPerson))
    .after((request, response) -> logResponse(response)) 注释：2
    .build();
```

| ![image-20201202160431443](Spring%20Web%20MVC.assets/image-20201202160431443.png) | `before`添加自定义请求标头的过滤器仅应用于两个GET路由。 |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| ![image-20201202160441300](Spring%20Web%20MVC.assets/image-20201202160441300.png) | `after`记录响应的过滤器将应用于所有路由，包括嵌套路由。 |

在`filter`路由器上构建器方法需要`HandlerFilterFunction`：一个函数，接受`ServerRequest`和`HandlerFunction`并返回`ServerResponse`。**handler函数**参数代表链中的下一个元素。这通常是路由到的处理程序，但是如果应用了多个，它也可以是另一个过滤器。

现在，我们可以在路由中添加一个简单的**安全过滤器**，假设我们有一个`SecurityManager`可以确定是否允许**特定路径**的。以下示例显示了如何执行此操作：

```java
SecurityManager securityManager = ...

RouterFunction<ServerResponse> route = route()
    .path("/person", b1 -> b1
        .nest(accept(APPLICATION_JSON), b2 -> b2
            .GET("/{id}", handler::getPerson)
            .GET(handler::listPeople))
        .POST("/person", handler::createPerson))
    .filter((request, next) -> {
        if (securityManager.allowAccessTo(request.path())) {
            return next.handle(request);
        }
        else {
            return ServerResponse.status(UNAUTHORIZED).build();
        }
    })
    .build();
```

前面的示例演示了调用`next.handle(ServerRequest)`是可选的。我们只允许在允许访问时运行处理程序函数。

除了使用`filter`路由器功能构建器上的方法之外，还可以通过将过滤器应用于现有路由器功能`RouterFunction.filter(HandlerFilterFunction)`。

| ![image-20201202160810522](Spring%20Web%20MVC.assets/image-20201202160810522.png) | 通过专用CORS支持功能端点 [`CorsFilter`](https://docs.spring.io/spring-framework/docs/current/reference/html/webmvc-cors.html#mvc-cors-filter)。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

## 1.5. URI Links (URI链接)

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-uri-building)

本部分介绍了Spring框架中可用于URI的各种选项。

### 1.5.1. UriComponents (URI组件)

Spring MVC和Spring WebFlux

`UriComponentsBuilder` 有助于从带有变量的URI模板构建URI，如以下示例所示：

```java
UriComponents uriComponents = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")  注释：1
        .queryParam("q", "{q}")  注释：2
        .encode() 注释：3
        .build(); 注释：4

URI uri = uriComponents.expand("Westin", "123").toUri();  注释：5
```

| ![image-20201202161106723](Spring%20Web%20MVC.assets/image-20201202161106723.png) | 带有URI模板的静态工厂方法。      |
| ------------------------------------------------------------ | -------------------------------- |
| ![image-20201202161114537](Spring%20Web%20MVC.assets/image-20201202161114537.png) | **添加或替换URI**组件。          |
| ![image-20201202161121571](Spring%20Web%20MVC.assets/image-20201202161121571.png) | 请求对URI模板和URI变量进行编码。 |
| ![image-20201202161129283](Spring%20Web%20MVC.assets/image-20201202161129283.png) | 建立一个`UriComponents`。        |
| ![image-20201202161135913](Spring%20Web%20MVC.assets/image-20201202161135913.png) | 展开变量并获得`URI`。            |

可以将前面的示例合并为一个链，并通过进行缩短`buildAndExpand`，如以下示例所示：

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")
        .queryParam("q", "{q}")
        .encode()
        .buildAndExpand("Westin", "123")
        .toUri();
```

您可以通过**直接转到URI**（这意味着编码）来进一步缩短它，如以下示例所示：

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")
        .queryParam("q", "{q}")
        .build("Westin", "123");
```

您可以使用完整的URI模板进一步**缩短**它，如以下示例所示：

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}?q={q}")
        .build("Westin", "123");
```

### 1.5.2. UriBuilder (URI构建器)

Spring MVC和Spring WebFlux

[`UriComponentsBuilder`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#web-uricomponents)实施`UriBuilder`。您可以`UriBuilder`依次使用创建 一个`UriBuilderFactory`。一起，`UriBuilderFactory`并 `UriBuilder`提供一个可插入的机构以从URI模板，基于**共享的配置**，构建的URI诸如基本URL，编码偏好，以及其他细节。

您可以配置`RestTemplate`和`WebClient`使用`UriBuilderFactory` 自定义的URI的准备。`DefaultUriBuilderFactory`是内部`UriBuilderFactory`使用`UriComponentsBuilder`并公开共享配置选项的==默认实现==。

以下示例显示了如何配置`RestTemplate`：

```java
// import org.springframework.web.util.DefaultUriBuilderFactory.EncodingMode;

String baseUrl = "https://example.org";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl);
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

RestTemplate restTemplate = new RestTemplate();
restTemplate.setUriTemplateHandler(factory);
```

以下示例配置了`WebClient`：

```java
// import org.springframework.web.util.DefaultUriBuilderFactory.EncodingMode;

String baseUrl = "https://example.org";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl);
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
```

另外，您也可以`DefaultUriBuilderFactory`直接使用。它与使用类似， `UriComponentsBuilder`但不是**静态工厂方法**，它是一个包含配置和首选项的实际实例，如以下示例所示：

```java
String baseUrl = "https://example.com";
DefaultUriBuilderFactory uriBuilderFactory = new DefaultUriBuilderFactory(baseUrl);

URI uri = uriBuilderFactory.uriString("/hotels/{hotel}")
        .queryParam("q", "{q}")
        .build("Westin", "123");
```

### 1.5.3. URI Encoding (URI编码)

Spring MVC和Spring WebFlux

`UriComponentsBuilder` 在两个级别公开编码选项：

- [UriComponentsBuilder＃encode（）](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/util/UriComponentsBuilder.html#encode--)：首先对URI模板进行预编码，然后在扩展时严格对URI变量进行编码。
- [UriComponents＃encode（）](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/util/UriComponents.html#encode--)：扩展URI变量*后，*[对](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/util/UriComponents.html#encode--)URI组件*进行*编码。

这两个选项都使用转义的八位字节替换**非ASCII和非法字符**。但是，第一个选项还会替换出现在URI变量中的具有**保留含义**的字符。

| ![image-20201202161618394](Spring%20Web%20MVC.assets/image-20201202161618394.png) | 考虑“;”，这在路径上是合法的，但具有保留的含义。第一个选项代替“;” URI变量中带有“％3B”，但URI模板中没有。相比之下，第二个选项永远不会替换“;”，因为它是路径中的合法字符。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

在大多数情况下，第一个选项可能会产生预期的结果，因为它将URI变量视为要完全编码的**不透明数据**，而选项2仅在URI变量有意包含保留字符的情况下才有用。

以下示例使用第一个选项：

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}")
        .queryParam("q", "{q}")
        .encode()
        .buildAndExpand("New York", "foo+bar")
        .toUri();

// Result is "/hotel%20list/New%20York?q=foo%2Bbar"
```

您可以通过直接转到URI（这意味着编码）来缩短前面的示例，如以下示例所示：

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}")
        .queryParam("q", "{q}")
        .build("New York", "foo+bar")
```

您可以使用完整的URI模板进一步缩短它，如以下示例所示：

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}?q={q}")
        .build("New York", "foo+bar")
```

在`WebClient`与`RestTemplate`**扩大**和编码URI通过内部模板`UriBuilderFactory`策略。两者都可以使用**自定义策略**进行配置。如下例所示：

```java
String baseUrl = "https://example.com";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl)
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

// Customize the RestTemplate..
RestTemplate restTemplate = new RestTemplate();
restTemplate.setUriTemplateHandler(factory);

// Customize the WebClient..
WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
```

该`DefaultUriBuilderFactory`实现在`UriComponentsBuilder`内部使用以扩展和编码URI模板。作为工厂，它提供了一个位置，可以根据以下一种编码模式来配置编码方法：

- `TEMPLATE_AND_VALUES`：使用`UriComponentsBuilder#encode()`，对应于先前列表中的第一个选项，对URI模板进行预编码，并在**扩展**时严格编码URI变量。
- `VALUES_ONLY`：不对URI模板进行编码，而是在将URI变量`UriUtils#encodeUriUriVariables`扩展到模板之前对其进行严格编码。
- `URI_COMPONENT`：使用`UriComponents#encode()`，对应于先前列表中的第二个选项，在扩展URI变量*后使用*URI组件值进行编码。
- `NONE`：未应用编码。

出于历史原因和向后兼容性，`RestTemplate`将设置`EncodingMode.URI_COMPONENT`为。在`WebClient`依赖于缺省值`DefaultUriBuilderFactory`，将其从变化`EncodingMode.URI_COMPONENT`在5.0.x的到`EncodingMode.TEMPLATE_AND_VALUES`5.1。

### 1.5.4. Relative Servlet Requests (相对Servlet请求)

您可以`ServletUriComponentsBuilder`用来创建相对于当前请求的URI，如以下示例所示：

```java
HttpServletRequest request = ...

// Re-uses host, scheme, port, path and query string...

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromRequest(request)
        .replaceQueryParam("accountId", "{id}").build()
        .expand("123")
        .encode();
```

您可以创建相对于上下文路径的URI，如以下示例所示：

```java
// Re-uses host, port and context path...

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromContextPath(request)
        .path("/accounts").build()
```

您可以创建与Servlet相关的URI（例如`/main/*`），如以下示例所示：

```java
// Re-uses host, port, context path, and Servlet prefix...

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromServletMapping(request)
        .path("/accounts").build()
```

| ![image-20201202161853365](Spring%20Web%20MVC.assets/image-20201202161853365.png) | 从5.1开始，`ServletUriComponentsBuilder`忽略来自`Forwarded`和 `X-Forwarded-*`标头的信息，该标头指定了客户端起源的地址。考虑使用 [`ForwardedHeaderFilter`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-forwarded-headers)提取和使用或丢弃此类标头。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

### 1.5.5. Links to Controllers 链接到控制器

Spring MVC提供了一种准备到控制器方法的链接的机制。例如，以下MVC控制器允许创建链接：

```java
@Controller
@RequestMapping("/hotels/{hotel}")
public class BookingController {

    @GetMapping("/bookings/{booking}")
    public ModelAndView getBooking(@PathVariable Long booking) {
        // ...
    }
}
```

您可以通过按名称引用方法来准备链接，如以下示例所示：

```java
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodName(BookingController.class, "getBooking", 21).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

在前面的示例中，我们提供了实际的方法参数值（在本例中为long值`21`：）用作路径变量并插入到URL中。此外，我们提供了值，`42`以填充所有剩余的URI变量，例如`hotel`从类型级别请求映射继承的变量。如果该方法具有更多参数，则可以为URL不需要的**参数提供null**。通常，只有`@PathVariable`和`@RequestParam`参数与**构造URL有关**。

还有其他使用方法`MvcUriComponentsBuilder`。例如，您可以使用一种类似于代理的测试技术来避免按名称**引用控制器方法**，如以下示例所示（该示例假定静态导入`MvcUriComponentsBuilder.on`）：

```java
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodCall(on(BookingController.class).getBooking(21)).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

| ![image-20201202162027042](Spring%20Web%20MVC.assets/image-20201202162027042.png) | 当控制器方法签名可以用于链接创建时，控制器方法签名的设计受到限制`fromMethodCall`。除了需要适当的参数签名外，返回类型还存在技术限制（即，为链接生成器调用生成运行时代理），因此返回类型一定不能为`final`。特别是，`String`视图名称的通用返回类型在这里不起作用。您应该改用`ModelAndView` 甚至是纯文本`Object`（带有`String`返回值）。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

前面的示例在中使用静态方法`MvcUriComponentsBuilder`。在内部，它们依靠`ServletUriComponentsBuilder`当前请求的方案，主机，端口，上下文路径和Servlet路径准备基本URL。在大多数情况下，这种方法效果很好。但是，有时可能不足。例如，您可能不在请求的上下文之内（例如，准备链接的批处理过程），或者您可能需要插入路径前缀（例如，从请求路径中删除并需要重新设置的语言环境前缀）。**插入链接**）。

在这种情况下，您可以使用`fromXxx`接受的静态重载方法 `UriComponentsBuilder`来使用基本URL。另外，您可以`MvcUriComponentsBuilder` 使用基本URL创建的实例，然后使用基于实例的`withXxx`方法。例如，以下清单使用`withMethodCall`：

```java
UriComponentsBuilder base = ServletUriComponentsBuilder.fromCurrentContextPath().path("/en");
MvcUriComponentsBuilder builder = MvcUriComponentsBuilder.relativeTo(base);
builder.withMethodCall(on(BookingController.class).getBooking(21)).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

| ![image-20201202162127040](Spring%20Web%20MVC.assets/image-20201202162127040.png) | 从5.1开始，`MvcUriComponentsBuilder`忽略来自`Forwarded`和 `X-Forwarded-*`标头的信息，该标头指定了客户端起源的地址。考虑使用 [ForwardedHeaderFilter](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-forwarded-headers)提取和使用或丢弃此类标头。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

### 1.5.6. Links in Views 视图中的链接

在**Thymeleaf，FreeMarker或JSP之类**的视图中，您可以通过引用每个请求映射的==隐式或显式==分配的名称来构建到带注释的控制器的链接。

考虑以下示例：

```java
@RequestMapping("/people/{id}/addresses")
public class PersonAddressController {

    @RequestMapping("/{country}")
    public HttpEntity<PersonAddress> getAddress(@PathVariable String country) { ... }
}
```

给定前面的控制器，您可以按照以下步骤准备来自JSP的链接：

```java
<%@ taglib uri="http://www.springframework.org/tags" prefix="s" %>
...
<a href="${s:mvcUrl('PAC#getAddress').arg(0,'US').buildAndExpand('123')}">Get Address</a>
```

上面的示例依赖于`mvcUrl`Spring标记库中声明的函数（即META-INF / spring.tld），但是很容易定义您自己的函数或为其他模板技术准备类似的函数。

这是这样的。在启动时，每个人`@RequestMapping`都通过分配了一个默认名称`HandlerMethodMappingNamingStrategy`，其默认实现使用类的**大写字母和方法名称**（例如，中的`getThing`方法 `ThingController`变为“ TC＃getThing”）。如果名称冲突，则可以使用 `@RequestMapping(name="..")`来分配一个明确的名称或实现自己的名称 `HandlerMethodMappingNamingStrategy`。

## 1.6. Asynchronous Requests 异步请求

[与WebFlux相比](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-vs-webflux)

Spring MVC与Servlet 3.0异步请求[处理](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-processing)具有广泛的集成 ：

- [`DeferredResult`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-deferredresult)和[`Callable`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-callable) 控制器方法中的返回值，并为单个异步返回值提供基本支持。
- 控制器可以[流式传输](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-http-streaming)多个值，包括 [SSE](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-sse)和[原始数据](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-output-stream)。
- 控制器可以使用反应式客户端并返回 [反应式类型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-reactive-types)以进行响应处理。

###  1.6.1. DeferredResult 异步请求处理

[与WebFlux相比](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-vs-webflux)

一旦 在Servlet容器中[启用](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-configuration)了异步请求处理功能，**控制器方法**就可以使用来包装任何受支持的控制器方法返回值`DeferredResult`，如以下示例所示：

```java
@GetMapping("/quotes")
@ResponseBody
public DeferredResult<String> quotes() {
    DeferredResult<String> deferredResult = new DeferredResult<String>();
    // Save the deferredResult somewhere..
    return deferredResult;
}

// From some other thread...
deferredResult.setResult(result);
```

控制器可以从==另一个线程异步生成返回值==，例如，响应外部事件（JMS消息），计划任务或其他事件。

### 1.6.2. Callable 可召回

[与WebFlux相比](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-vs-webflux)

控制器可以使用来包装任何受支持的返回值`java.util.concurrent.Callable`，如以下示例所示：

```java
@PostMapping
public Callable<String> processUpload(final MultipartFile file) {

    return new Callable<String>() {
        public String call() throws Exception {
            // ...
            return "someView";
        }
    };
}
```

然后可以通过[configure](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-configuration-spring-mvc) 来运行给定任务来获取返回值 `TaskExecutor`。

###  1.6.3. Processing 处理中

[与WebFlux相比](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-vs-webflux)

这是Servlet异步请求处理的非常简洁的概述：

- `ServletRequest`可以通过调用将A置于异步模式`request.startAsync()`。这样做的主要作用是可以退出Servlet（以及所有过滤器），但是响应保持打开状态，以便以后完成处理。
- 要调用`request.startAsync()`的回报`AsyncContext`，您可以使用超过异步处理的进一步控制。例如，它提供了`dispatch`与Servlet API中的转发类似的方法，不同之处在于，它允许应用程序恢复Servlet容器线程上的请求处理。
- 在`ServletRequest`提供对电流`DispatcherType`，它可以使用处理该初始请求，**异步调度之间**进行区分，前向，以及其他的调度类型。

`DeferredResult` 处理工作如下：

- 控制器返回a`DeferredResult`并将其保存在一些可以访问它的内存队列或列表中。
- Spring MVC调用`request.startAsync()`。
- 同时，`DispatcherServlet`和所有配置的过滤器退出请求处理线程，但**响应保持打开状态。**
- 应用程序`DeferredResult`从某个线程设置，Spring MVC将请求分派回Servlet容器。
- 将`DispatcherServlet`被再次调用，并且处理与异步生产返回值恢复。

`Callable` 处理工作如下：

- 控制器返回`Callable`。
- Spring MVC调用`request.startAsync()`并将其提交`Callable`到`TaskExecutor`一个单独的线程中进行处理。
- 同时，`DispatcherServlet`和所有过滤器退出Servlet容器线程，但是响应保持打开状态。
- 最终`Callable`产生一个结果，Spring MVC将请求分派回Servlet容器以完成处理。
- 将`DispatcherServlet`被再次调用，并且处理从所述异步生产返回值恢复`Callable`。

有关更多背景知识，您还可以阅读 在Spring MVC 3.2中引入了异步请求处理支持[的博客文章](https://spring.io/blog/2012/05/07/spring-mvc-3-2-preview-introducing-servlet-3-async-support)。

#### Exception Handling 异常处理

使用时`DeferredResult`，您可以选择呼叫`setResult`还是 `setErrorResult`例外。在这两种情况下，Spring MVC都将请求**分派回Servlet容器**以完成处理。然后将其视为控制器方法返回了给定值，或者好像它产生了给定的异常。然后，异常将通过常规的异常处理机制（例如，调用 `@ExceptionHandler`方法）进行处理。

当您使用时`Callable`，会发生类似的处理逻辑，**主要区别**是从中返回了结果，`Callable`或者引发了异常。

####  Interception 拦截

`HandlerInterceptor`实例的类型可以为`AsyncHandlerInterceptor`，以接收`afterConcurrentHandlingStarted`启动异步处理的初始请求（而不是`postHandle`和`afterCompletion`）上的 回调。

`HandlerInterceptor`实现也可以注册`CallableProcessingInterceptor` 或`DeferredResultProcessingInterceptor`，以与异步请求的生命周期更深入地**集成**（例如，处理超时事件）。请参阅 [`AsyncHandlerInterceptor`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/AsyncHandlerInterceptor.html) 以获取更多详细信息。

`DeferredResult`提供`onTimeout(Runnable)`和`onCompletion(Runnable)`回调。有关 更多详细信息，请参见的[Javadoc`DeferredResult`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/context/request/async/DeferredResult.html)。`Callable`可以替换为`WebAsyncTask`暴露超时和完成回调的其他方法。

####  Compared to WebFlux （与WebFlux相比）

Servlet API**最初**是为==通过Filter-Servlet链==进行一次传递而构建的。Servlet 3.0中添加了异步请求处理，使应用程序可以==退出Filter-Servlet链==，但保留响应以进行进一步处理。Spring MVC异步支持围绕该机制构建。当控制器返回a时`DeferredResult`，退出Filter-Servlet链，并释放Servlet容器线程。稍后，当`DeferredResult`设置了时，将进行一次`ASYNC`调度（到相同的URL），在此期间再次映射控制器，但是`DeferredResult`使用该值（就像控制器返回了它一样）而不是调用它来恢复处理。

相比之下，Spring WebFlux**既不是基于Servlet API构建**的，也**不需要这种异步请求处理功能**，因为它在设计上是异步的。异步处理已内置在所有框架协定中，并在请求处理的所有阶段得到内在支持。

从**编程模型**的角度来看，Spring MVC和Spring WebFlux都支持异步和响应[类型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-reactive-types)作为控制器方法中的返回值。Spring MVC甚至支持流，包括反应背压。但是，与WebFlux不同，WebFlux依赖于非阻塞I / O，并且每次写入都不需要额外的线程，因此对响应的单个写入仍然处于**阻塞状态**（并在单独的线程上执行）。

另一个根本区别在于Spring MVC的**不支持**在控制器方法参数异步或反应性类型（例如，`@RequestBody`，`@RequestPart`，和其它物质），也不会具有用于异步和反应类型作为模型属性的任何显式支持。Spring WebFlux确实支持所有这些。

###  1.6.4. HTTP Streaming （HTTP流）

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs-streaming)

您可以将`DeferredResult`和`Callable`用于单个异步返回值。如果要产生多个异步值并将那些值写入响应，该怎么办？本节介绍如何执行此操作。

#### Objects 对象

您可以使用`ResponseBodyEmitter`返回值生成一个对象流，其中每个对象都使用进行序列化 [`HttpMessageConverter`](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-message-conversion)并写入响应，如以下示例所示：

```java
@GetMapping("/events")
public ResponseBodyEmitter handle() {
    ResponseBodyEmitter emitter = new ResponseBodyEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();
```

您还可以`ResponseBodyEmitter`在中将其用作**主体**，以`ResponseEntity`==自定义响应的状态和标题==。

当`emitter`抛出异常时`IOException`（例如，如果远程客户端消失了），应用程序==不负责清理连接==，因此不应调用`emitter.complete` 或`emitter.completeWithError`。取而代之的是，Servlet容器自动启动 `AsyncListener`错误通知，Spring MVC在其中进行`completeWithError`调用。依次，此调用`ASYNC`对应用程序**执行最后的调度**，在此期间，Spring MVC调用**已配置**的异常解析器并完成请求。

#### SSE

`SseEmitter`（的子类`ResponseBodyEmitter`）提供对[服务器发送事件的](https://www.w3.org/TR/eventsource/)支持 ，其中从服务器[发送的事件](https://www.w3.org/TR/eventsource/)根据W3C SSE规范进行格式化。要从控制器生成SSE流，请返回`SseEmitter`，如以下示例所示：

```java
@GetMapping(path="/events", produces=MediaType.TEXT_EVENT_STREAM_VALUE)
public SseEmitter handle() {
    SseEmitter emitter = new SseEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();
```

尽管SSE是流式传输到浏览器的主要选项，但请注意==Internet Explorer不支持服务器发送事件==。考虑将Spring的 [WebSocket消息](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket)与 针对各种浏览器的[SockJS后备](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-fallback)传输（包括SSE）一起使用。

另请参阅[上一节](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-objects)以获取有关异常处理的注释。

#### Raw Data 原始数据

有时，**绕过消息转换并直接**流式传输到响应很有用 `OutputStream`（例如，用于文件下载）。您可以使用`StreamingResponseBody` 返回值类型来执行此操作，如以下示例所示：

```java
@GetMapping("/download")
public StreamingResponseBody handle() {
    return new StreamingResponseBody() {
        @Override
        public void writeTo(OutputStream outputStream) throws IOException {
            // write...
        }
    };
}
```

您可以`StreamingResponseBody`在中用作正文来自`ResponseEntity`定义响应的状态和标题。

### 1.6.5. Reactive Types 反应类型

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs-streaming)

Spring MVC支持在控制器中使用==反应式客户端库==（另请参阅 WebFlux部分中的[反应式库](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-reactive-libraries)）。这包括`WebClient`from`spring-webflux`和其他，例如Spring Data反应数据存储库。在这种情况下，能够从控制器方法返回反应类型是很方便的。

反应性返回值的处理方式如下：

- 与使用相似，单值承诺也适用`DeferredResult`。示例包括`Mono`（Reactor）或`Single`（RxJava）。
- 与使用或 类似，适用于具有**流媒体类型**（例如`application/x-ndjson` 或`text/event-stream`）的多值流。示例包括（Reactor）或（RxJava）。应用程序也可以返回或。`ResponseBodyEmitter``SseEmitter``Flux``Observable``Flux<ServerSentEvent>``Observable<ServerSentEvent>`
- `application/json`与使用相似，适用于其他任何媒体类型（例如）的多值流`DeferredResult<List<?>>`。

| ![image-20201202163543249](Spring%20Web%20MVC.assets/image-20201202163543249.png) | Spring MVC通过[`ReactiveAdapterRegistry`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/core/ReactiveAdapterRegistry.html)from来 支持Reactor和RxJava `spring-core`，这使其可以适应多个反应式库。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

为了流式传输到响应，**支持了反应性背压**，但响应的写仍处于**阻塞状态**，并通过[configure](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-configuration-spring-mvc) `TaskExecutor`，在单独的线程上运行 ，以避免阻塞上游源（例如`Flux`从返回的源`WebClient`）。默认情况下，`SimpleAsyncTaskExecutor`用于阻止写操作，但是在**负载下**不适合使用。如果计划使用响应类型进行流传输，则应使用 [MVC配置](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-configuration-spring-mvc)来配置任务执行程序。

### 1.6.6. Disconnects 断开连接

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs-streaming)

当远程客户端离开时，Servlet API不提供任何通知。因此，在通过[SseEmitter](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-sse) 或[反应性类型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-reactive-types)流式传输到响应时，**定期发送数据**非常重要，因为如果客户端断开连接，写入将失败。发送可以采取空（仅评论）SSE事件或另一端必须将其解释为心跳和忽略的任何其他数据的形式。

或者，考虑使用具有**内置心跳机制**的Web消息传递解决方案（例如，基于 [WebSocket的STOMP](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp)或具有[SockJS的](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-fallback)WebSocket ）。

###  1.6.7. Configuration 配置

[与WebFlux相比](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-vs-webflux)

必须在Servlet容器级别**启用异步请求**处理功能。MVC配置还为异步请求提供了多个选项。

#### Servlet Container （Servlet容器）

Filter和Servlet声明具有一个`asyncSupported`标志，需要将其设置`true` 为启用异步请求处理。此外，应**声明过滤器映射**以处理`ASYNC` `javax.servlet.DispatchType`。

在Java配置中，当您用于`AbstractAnnotationConfigDispatcherServletInitializer` 初始化Servlet容器时，这是**自动完成**的。

在`web.xml`配置中，您可以添加`<async-supported>true</async-supported>`到 `DispatcherServlet`和`Filter`声明以及添加 `<dispatcher>ASYNC</dispatcher>`到**过滤器映射**。

####  Spring MVC

MVC配置公开了以下与异步请求处理相关的选项：

- Java配置：在`configureAsyncSupport`上使用回调`WebMvcConfigurer`。
- XML名称空间：使用中的`<async-support>`元素`<mvc:annotation-driven>`。

您可以配置以下内容：

- 异步请求的==默认超时值==（如果未设置）取决于基础Servlet容器。
- `AsyncTaskExecutor`用于在使用[响应类型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-reactive-types)进行流式传输时阻止写操作 以及用于执行`Callable`从控制器方法返回的实例。我们强烈建议您配置此属性，如果您使用**响应类型进行流式处理或具有返回的控制器方法**，则`Callable`默认情况下为a `SimpleAsyncTaskExecutor`。
- `DeferredResultProcessingInterceptor`实施和`CallableProcessingInterceptor`实施。

请注意，您还可以在a `DeferredResult`，a`ResponseBodyEmitter`和an上设置默认超时值`SseEmitter`。对于`Callable`，您可以使用 `WebAsyncTask`提供超时值。

## 1.7. CORS  跨域资源共享

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-cors)

Spring MVC使您可以处理CORS（跨源资源共享）。本节介绍如何执行此操作。

###  1.7.1. Introduction 介绍

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-cors-intro)

出于安全原因，**浏览器禁止AJAX调用**当前来源以外的资源。例如，您可以将您的银行帐户放在一个标签中，将evil.com放在另一个标签中。来自evil.com的脚本不能使用您的凭据向您的银行API发出AJAX请求，例如从您的帐户中提取资金！

跨域资源共享（CORS）是 由[大多数浏览器](https://caniuse.com/#feat=cors)实现的[W3C规范](https://www.w3.org/TR/cors/)，可让您指定授权哪种类型的跨域请求，而不是使用基于IFRAME或JSONP的安全性较低且功能较弱的变通办法。

###  1.7.2. Processing 处理中

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-cors-processing)

CORS规范区分飞行前，简单和实际要求。要了解CORS的工作原理，您可以阅读 [本文](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)以及其他内容，或者参阅规范以获取更多详细信息。

Spring MVC`HandlerMapping`实现为CORS提供**内置支持**。成功将请求映射到处理程序后，`HandlerMapping`实现会检查给定请求和处理程序的CORS配置并采取进一步的措施。飞行前请求直接处理，而简单和实际的CORS请求被拦截，验证并设置了必需的CORS响应标头。

为了**启用跨域请求**（即，`Origin`标头存在并且与请求的主机不同），您需要具有一些**显式声明**的CORS配置。如果找不到匹配的CORS配置，则飞行前请求将被拒绝。没有将CORS标头添加到简单和实际CORS请求的响应中，因此，浏览器拒绝了它们。

每个`HandlerMapping`都可以 使用**基于URL模式**的映射进行 单独[配置](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/handler/AbstractHandlerMapping.html#setCorsConfigurations-java.util.Map-)`CorsConfiguration`。在大多数情况下，应用程序使用**MVC Java配置或XML名称空间**声明此类映射，这导致将**单个全局映射**传递给所有`HandlerMappping`实例。

您可以将全局CORS配置`HandlerMapping`与**更细粒度**的处理**程序级CORS**配置结合使用。例如，带注释的控制器可以使用类或方法级的`@CrossOrigin`注释（其他处理程序可以实现 `CorsConfigurationSource`）。

组合全局和本地配置的规则通常是相加的，例如，所有全局和所有本地来源。对于那些只能接受单个值的属性，例如`allowCredentials`和`maxAge`，局部变量将覆盖全局值。请参阅 [`CorsConfiguration#combine(CorsConfiguration)`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/cors/CorsConfiguration.html#combine-org.springframework.web.cors.CorsConfiguration-) 以获取更多详细信息。

 ![image-20201202164251494](Spring%20Web%20MVC.assets/image-20201202164251494.png)要从源中了解更多信息或进行高级自定义，请查看后面的代码：

* `CorsConfiguration`

- `CorsProcessor`， `DefaultCorsProcessor`
- `AbstractHandlerMapping`

###  1.7.3. @CrossOrigin

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors-controller)

该[`@CrossOrigin`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html) 注释能够对带注释的控制器方法跨域请求，如下面的示例所示：

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @GetMapping("/{id}")
    public Mono<Account> retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public Mono<Void> remove(@PathVariable Long id) {
        // ...
    }
}
```

默认情况下，`@CrossOrigin`允许：

- 所有起源。
- 所有标题。
- 控制器方法映射到的所有HTTP方法。

`allowCredentials`默认情况下不启用，因为这会建立一个信任级别，以**公开敏感**的用户特定信息（例如cookie和CSRF令牌），并且仅在适当的地方使用。启用后，`allowOrigins`必须将其设置为**一个或多个特定域**（而不是特殊值`"*"`），或者`allowOriginPatterns`可以使用该属性来匹配动态的一组原点。

`maxAge` 设置为30分钟。

`@CrossOrigin`在类级别也受支持，并且由**所有方法继承**。下面的示例指定一个特定的域并将其设置`maxAge`为一个小时：

```java
@CrossOrigin(origins = "https://domain2.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @GetMapping("/{id}")
    public Mono<Account> retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public Mono<Void> remove(@PathVariable Long id) {
        // ...
    }
}
```

您可以`@CrossOrigin`在**类和方法级别上**使用，如以下示例所示：

```java
@CrossOrigin(maxAge = 3600) 注释：1
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin("https://domain2.com") 注释：2
    @GetMapping("/{id}")
    public Mono<Account> retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public Mono<Void> remove(@PathVariable Long id) {
        // ...
    }
}
```

| ![image-20201202164707636](Spring%20Web%20MVC.assets/image-20201202164707636.png) | 使用`@CrossOrigin`类级别。   |
| ------------------------------------------------------------ | ---------------------------- |
| ![image-20201202164716242](Spring%20Web%20MVC.assets/image-20201202164716242.png) | 使用`@CrossOrigin`方法级别。 |

### 1.7.4. Global Configuration 全局配置

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors-global)

除了细粒度的控制器方法级配置外，您可能还想定义一些==全局CORS配置==。您可以`CorsConfiguration` 在任何上分别设置基于URL的映射`HandlerMapping`。但是，大多数应用程序都使用WebFlux Java配置来执行此操作。

默认情况下，全局配置启用以下功能：

- 所有起源。
- 所有标题。
- `GET`，`HEAD`和`POST`方法。

`allowedCredentials`默认情况下不会启用，因为这会建立一个信任级别，该级别**公开敏感**的用户特定信息（例如cookie和CSRF令牌），并且仅在适当的地方使用。启用后，`allowOrigins`必须将其设置为**一个或多个特定域**（而不是特殊值`"*"`），或者`allowOriginPatterns`可以使用该属性来匹配动态的一组原点。

`maxAge` **设置为30分钟**。

要在WebFlux Java配置中启用CORS，可以使用`CorsRegistry`回调，如以下示例所示：

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/api/**")
            .allowedOrigins("https://domain2.com")
            .allowedMethods("PUT", "DELETE")
            .allowedHeaders("header1", "header2", "header3")
            .exposedHeaders("header1", "header2")
            .allowCredentials(true).maxAge(3600);

        // Add more mappings...
    }
}
```

### 1.7.5. CORS WebFilter (CORS Web过滤器)

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors-filter)

您可以通过内置应用CORS支持 [`CorsWebFilter`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/cors/reactive/CorsWebFilter.html)，这非常适合[功能性端点](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn)。

| ![image-20201202164927116](Spring%20Web%20MVC.assets/image-20201202164927116.png) | 如果您尝试在`CorsFilter`Spring Security中使用，请记住Spring Security [内置了](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#cors) 对CORS的[支持](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#cors)。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

要配置过滤器，可以声明一个`CorsWebFilter`bean并将其传递 `CorsConfigurationSource`给其**构造函数**，如以下示例所示：

```java
@Bean
CorsWebFilter corsFilter() {

    CorsConfiguration config = new CorsConfiguration();

    // Possibly...
    // config.applyPermitDefaultValues()

    config.setAllowCredentials(true);
    config.addAllowedOrigin("https://domain1.com");
    config.addAllowedHeader("*");
    config.addAllowedMethod("*");

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);

    return new CorsWebFilter(source);
}
```

##  1.8. Web Security (网络安全)

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-web-security)

在[Spring Security的](https://projects.spring.io/spring-security/)项目提供了保护Web应用程序免受恶意攻击的支持。请参阅Spring Security参考文档，包括：

- [WebFlux安全](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#jc-webflux)
- [WebFlux测试支持](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#test-webflux)
- [CSRF保护](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#csrf)
- [安全响应标头](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#headers)

[HDIV](https://hdiv.org/)是另一个与Spring MVC集成的Web安全框架。

## 1.9. HTTP Caching  HTTP缓存

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-caching)

HTTP缓存可以显着提高Web应用程序的性能。HTTP缓存围绕`Cache-Control`响应标头和随后的条件请求标头（例如`Last-Modified`和`ETag`）展开。`Cache-Control`为私有（例如浏览器）和公共（例如代理）缓存提供有关如何缓存和重用响应的建议。一个`ETag`头用于使没有身体可能导致一个304（NOT_MODIFIED）一个条件请求，如果内容没有改变。`ETag`可以看作是`Last-Modified`标题的更复杂的后继者。

本节描述了Spring Web MVC中与HTTP缓存相关的选项。

### 1.9.1. CacheControl 缓存控制

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-caching-cachecontrol)

[`CacheControl`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/http/CacheControl.html)提供对配置与`Cache-Control`标头相关的设置的支持，并在许多地方作为参数被接受：

- [`WebContentInterceptor`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/mvc/WebContentInterceptor.html)
- [`WebContentGenerator`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/support/WebContentGenerator.html)
- [控制器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching-etag-lastmodified)
- [静态资源](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching-static-resources)

尽管[RFC 7234](https://tools.ietf.org/html/rfc7234#section-5.2.2)描述了`Cache-Control`响应标头的所有可能的指令，但该`CacheControl`类型采用面向用例的方法，重点关注常见方案：

```java
// Cache for an hour - "Cache-Control: max-age=3600"
CacheControl ccCacheOneHour = CacheControl.maxAge(1, TimeUnit.HOURS);

// Prevent caching - "Cache-Control: no-store"
CacheControl ccNoStore = CacheControl.noStore();

// Cache for ten days in public and private caches,
// public caches should not transform the response
// "Cache-Control: max-age=864000, public, no-transform"
CacheControl ccCustom = CacheControl.maxAge(10, TimeUnit.DAYS).noTransform().cachePublic();
```

`WebContentGenerator`还接受如下更简单的`cachePeriod`属性（以秒为单位定义）：

- 甲`-1`值不产生`Cache-Control`响应头。
- 甲`0`值可以防止通过使用缓存`'Cache-Control: no-store'`指令。
- 一个`n > 0`值缓存用于给定响应`n`通过使用秒 `'Cache-Control: max-age=n'`指令。

### 1.9.2. Controllers (控制器)

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-caching-etag-lastmodified)

控制器可以添加==对HTTP缓存的显式支持==。我们建议您这样做，因为需要先计算资源的 `lastModified`or`ETag`值，然后才能将其==与条件请求标头==进行比较。控制器可以将`ETag`标头和`Cache-Control` 设置添加到中`ResponseEntity`，如以下示例所示：

```java
@GetMapping("/book/{id}")
public ResponseEntity<Book> showBook(@PathVariable Long id) {

    Book book = findBook(id);
    String version = book.getVersion();

    return ResponseEntity
            .ok()
            .cacheControl(CacheControl.maxAge(30, TimeUnit.DAYS))
            .eTag(version) // lastModified is also available
            .body(book);
}
```

如果与条件请求标头的比较表明内容**未更改**，则前面的示例发送带有空正文的304（NOT_MODIFIED）响应。否则， `ETag`和`Cache-Control`标头将添加到响应中。

您还可以在控制器中针对条件请求标头进行检查，如以下示例所示：

```java
@RequestMapping
public String myHandleMethod(WebRequest request, Model model) {

    long eTag = ... 注释：1

    if (request.checkNotModified(eTag)) {
        return null; 注释：2
    }

    model.addAttribute(...); 注释：3
    return "myViewName";
}
```

| ![image-20201202170004862](Spring%20Web%20MVC.assets/image-20201202170004862.png) | 特定于应用程序的计算。                           |
| ------------------------------------------------------------ | ------------------------------------------------ |
| ![image-20201202170012413](Spring%20Web%20MVC.assets/image-20201202170012413.png) | 响应已设置为304（NOT_MODIFIED）-无需进一步处理。 |
| ![image-20201202170019405](Spring%20Web%20MVC.assets/image-20201202170019405.png) | 继续进行请求处理。                               |

有三种变体，用于根据`eTag`值和`lastModified` /或值检查条件请求。对于条件`GET`和`HEAD`请求，可以将**响应设置为304**（NOT_MODIFIED）。对于有条件的`POST`，`PUT`和`DELETE`，可以改为将响应设置为412（PRECONDITION_FAILED），以防止并发修改。

### 1.9.3. Static Resources 静态资源

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-caching-static-resources)

您应该为静态资源提供`Cache-Control`和条件响应标头，以实现最佳性能。请参阅“配置[静态资源](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-static-resources)”部分。

###  1.9.4. `ETag` Filter  （ETag过滤器）

您可以使用`ShallowEtagHeaderFilter`来添加`eTag`根据响应内容计算出的“浅”值，从而节省带宽，但不节省CPU时间。请参阅[浅ETag](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-shallow-etag)。

 ## 1.10. View Technologies 查看技术

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view)

Spring MVC中视图技术的使用是可插入的。是否**决定使用Thymeleaf，Groovy标记模板**，JSP或其他技术，主要取决于配置更改。本章介绍与Spring MVC集成的视图技术。我们假设您已经熟悉[View Resolution](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-viewresolver)。

| ![image-20201202170733738](Spring%20Web%20MVC.assets/image-20201202170733738.png) | Spring MVC应用程序的视图位于该应用程序的内部信任范围内。视图可以访问应用程序上下文中的所有bean。因此，不建议在外部源可编辑模板的应用程序中使用Spring MVC的模板支持，因为这可能会带来安全隐患。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

### 1.10.1. Thymeleaf

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view-thymeleaf)

**Thymeleaf**是一种==现代的服务器端Java模板引擎==，它强调可以通过双击在浏览器中预览的**自然HTML模板**，这对于独立处理UI模板（例如，由设计人员）非常有用，而无需使用正在运行的服务器。如果要替换JSP，Thymeleaf提供了最广泛的功能集之一，以使这种**过渡**更加容易。Thymeleaf是积极开发和维护的。有关更完整的介绍，请参见 [Thymeleaf](https://www.thymeleaf.org/)项目主页。

Thymeleaf与Spring MVC的集成由Thymeleaf项目管理。**配置涉及几个bean声明**，如 `ServletContextTemplateResolver`，`SpringTemplateEngine`和`ThymeleafViewResolver`。有关更多详细信息，请参见[Thymeleaf + Spring](https://www.thymeleaf.org/documentation.html)。

### 1.10.2. FreeMarker

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view-freemarker)

[Apache FreeMarker](https://freemarker.apache.org/)是一个模板引擎，用于生成从HTML到电子邮件等的任何类型的文本输出。Spring框架具有**内置的集**成，可以将**Spring MVC与FreeMarker**模板一起使用。

#### View Configuration 查看配置

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view-freemarker-contextconfig)

以下示例显示了如何将FreeMarker配置为一种视图技术：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.freeMarker();
    }

    // Configure FreeMarker...

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("/WEB-INF/freemarker");
        return configurer;
    }
}
```

以下示例显示了如何在XML中进行配置：

```xml
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:freemarker/>
</mvc:view-resolvers>

<!-- Configure FreeMarker... -->
<mvc:freemarker-configurer>
    <mvc:template-loader-path location="/WEB-INF/freemarker"/>
</mvc:freemarker-configurer>
```

另外，您也可以声明`FreeMarkerConfigurer`Bean以完全控制所有属性，如以下示例所示：

```xml
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
</bean>
```

您的模板需要存储在`FreeMarkerConfigurer` 上例所示的目录中。给定上述配置，如果您的控制器返回的视图名称为`welcome`，则解析器将查找 `/WEB-INF/freemarker/welcome.ftl`模板。

#### FreeMarker Configuration （FreeMarker配置）

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-views-freemarker)

您可以通过`Configuration`在bean上设置适当的bean属性，将FreeMarker的“设置”和“ SharedVariables”直接传递给**FreeMarker 对象**（由Spring管理）`FreeMarkerConfigurer`。该`freemarkerSettings`属性需要一个`java.util.Properties`对象，而该`freemarkerVariables`属性需要一个 `java.util.Map`。以下示例显示了如何使用`FreeMarkerConfigurer`：

```xml
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
    <property name="freemarkerVariables">
        <map>
            <entry key="xml_escape" value-ref="fmXmlEscape"/>
        </map>
    </property>
</bean>

<bean id="fmXmlEscape" class="freemarker.template.utility.XmlEscape"/>
```

有关设置和变量应用于`Configuration`对象的详细信息，请参见FreeMarker文档。

#### Form Handling 表格处理

Spring提供了一个供JSP使用的标签库，其中包含一个 `<spring:bind/>`元素。该元素主要允许表单显示来自表单支持对象的值，并显示来自`Validator`Web或业务层中验证失败的结果。Spring还支持FreeMarker中的相同功能，并带有用于生成表单输入元素本身的**附加便利宏。**

#### The Bind Macros 绑定宏

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view-bind-macros)

`spring-webmvc.jar`FreeMarker的文件中保留了一组标准的宏，因此它们始终可用于经过适当配置的应用程序。

Spring模板库中定义的某些宏被视为内部（私有）宏，但是在宏定义中**不存在这种范围**，使所有宏对调用代码和用户模板可见。以下各节仅关注您需要从模板内==直接调用的宏==。如果您希望直接查看宏代码，则将调用该文件`spring.ftl`并将其包含在 `org.springframework.web.servlet.view.freemarker`包中。

#### Simple Binding 简单绑定

在基于FreeMarker模板的HTML表单中，这些模板充当Spring MVC控制器的表单视图，您可以使用类似于下一个示例的代码绑定到字段值，并以类似于JSP的方式显示每个输入字段的错误消息。以下示例显示一个`personForm`视图：

```xml
<!-- FreeMarker macros have to be imported into a namespace.
    We strongly recommend sticking to 'spring'. -->
<#import "/spring.ftl" as spring/>
<html>
    ...
    <form action="" method="POST">
        Name:
        <@spring.bind "personForm.name"/>
        <input type="text"
            name="${spring.status.expression}"
            value="${spring.status.value?html}"/><br />
        <#list spring.status.errorMessages as error> <b>${error}</b> <br /> </#list>
        <br />
        ...
        <input type="submit" value="submit"/>
    </form>
    ...
</html>
```

`<@spring.bind>`需要一个'path'参数，该参数包含命令对象的名称（除非您在控制器配置中对其进行了更改，否则为'command'），后跟一个句点以及所需的命令对象上的字段名称绑定。您还可以使用嵌套字段，例如`command.address.street`。该`bind`宏假定默认的HTML转由指定的行为`ServletContext`参数 `defaultHtmlEscape`在`web.xml`。

宏的另一种形式称为`<@spring.bindEscaped>`第二个参数，该参数明确指定在状态错误消息或值中应使用==HTML转义==。您可以根据需要将其设置为`true`或`false`。附加的表单处理宏可**简化**HTML转义的使用，并且您应尽可能使用这些宏。下一节将对它们进行说明。

#### Input Macros 输入宏

FreeMarker的其他便利宏可简化绑定和表单生成（包括验证错误显示）。从来**没有必要**使用这些宏来生成表单输入字段，并且您可以将它们与简单的HTML混合或匹配，或者直接调用我们之前强调的Spring绑定宏。

下表列出了可用的宏，其中显示了FreeMarker模板（FTL）定义和每个参数采用的参数列表：

| 巨集                                                         | FTL定义                                            |
| :----------------------------------------------------------- | :------------------------------------------------- |
| `message` （根据code参数从资源包中输出一个字符串）           | <@ spring.message代码/>                            |
| `messageText` （根据code参数从资源包中输出一个字符串，回退到默认参数的值） | <@ spring.message文本代码，文本/>                  |
| `url` （使用应用程序的上下文根作为相对URL的前缀）            | <@ spring.url relativeUrl />                       |
| `formInput` （用于收集用户输入的标准输入字段）               | <@ spring.formInput路径，属性，fieldType />        |
| `formHiddenInput` （用于输入非用户输入的隐藏输入字段）       | <@ spring.formHiddenInput路径，属性/>              |
| `formPasswordInput` （用于收集密码的标准输入字段。请注意，此类型的字段中不会填充任何值。） | <@ spring.formPasswordInput路径，属性/>            |
| `formTextarea` （大文本字段，用于收集长而自由格式的文本输入） | <@ spring.formTextarea路径，属性/>                 |
| `formSingleSelect` （选项的下拉框允许选择一个必需的值）      | <@ spring.formSingleSelect路径，选项，属性/>       |
| `formMultiSelect` （选项列表框，允许用户选择0个或多个值）    | <@ spring.formMultiSelect路径，选项，属性/>        |
| `formRadioButtons` （一组单选按钮，可从可用选项中进行单个选择） | <@ spring.formRadioButtons路径，选项分隔符，属性/> |
| `formCheckboxes` （一组允许选择0个或多个值的复选框）         | <@ spring.formCheckboxes路径，选项，分隔符，属性/> |
| `formCheckbox` （一个复选框）                                | <@ spring.formCheckbox路径，属性/>                 |
| `showErrors` （简化了绑定字段验证错误的显示）                | <@ spring.showErrors分隔符，classOrStyle />        |

| ![image-20201202171138889](Spring%20Web%20MVC.assets/image-20201202171138889.png) | 在FreeMarker的模板，`formHiddenInput`而`formPasswordInput`实际上并不是必需的，因为你可以使用正常的`formInput`宏，指定`hidden`或`password` 作为值`fieldType`参数。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

以上任何宏的参数都具有一致的含义：

- `path`：要绑定的字段名称（即“ command.name”）
- `options`：`Map`可在输入字段中选择的所有可用值中的A。**映射的键**表示从表单回发并绑定到命令对象的值。针对键存储的地图对象是在表单上显示给用户的标签，并且可能与表单回发的相应值不同。通常，这种地图由控制器作为参考数据提供。您可以使用任何`Map`实现，具体取决于所需的行为。对于严格排序的地图，可以使用带有适当符号的`SortedMap`（例如`TreeMap`）`Comparator`，对于应按插入顺序返回值的**任意地图**，请使用`LinkedHashMap`或`LinkedMap`from `commons-collections`。
- `separator`：如果多个选项可用作离散元素（单选按钮或复选框），则用于分隔列表中每个字符的字符序列（例如`<br>`）。
- `attributes`：HTML标记本身包含的任意标记或文本的附加字符串。该字符串实际上是由宏回显的。例如，在一个 `textarea`字段中，您可以提供属性（例如'rows =“ 5” cols =“ 60”'），或者可以传递样式信息，例如'style =“ border：1px solid silver”'。
- `classOrStyle`：对于`showErrors`宏，`span` 包装每个错误的元素使用的CSS类的名称。如果未提供任何信息（或该值为空），则将错误包装在`<b></b>`标签中。

以下各节概述了宏的示例。

输入栏位

该`formInput`宏带有`path`参数（`command.name`）和一个附加`attributes` 参数（在后面的示例中为空）。该宏与所有其他表单生成宏一起，对path参数执行隐式Spring绑定。绑定将保持有效，直到发生**新的绑定为止**，因此该`showErrors`宏**无需**再次传递path参数-它对最后创建绑定的字段进行操作。

的`showErrors`宏需要一个**分离器参数**（其用于在给定的场中分离多个错误的字符），还接受第二个参数-此时**，类名或风格属性**。请注意，FreeMarker可以为attributes参数指定默认值。以下示例显示了如何使用`formInput` 和`showErrors`宏：

```xml
<@spring.formInput "command.name"/>
<@spring.showErrors "<br>"/>
```

下一个示例显示表单片段的输出，生成名称**字段**，并在提交表单后在==该字段中没有值的情况下==显示验证错误。验证通过Spring的**Validation框架**进行。

生成的HTML类似于以下示例：

```jsp
Name:
<input type="text" name="name" value="">
<br>
    <b>required</b>
<br>
<br>
```

该`formTextarea`宏的工作方式相同的`formInput`宏观和接受相同的参数列表。常见的是，第二个参数（`attributes`）被用于传递样式信息或`rows`和`cols`该属性`textarea`。

选择字段

您可以使用四个选择字段宏在HTML表单中生成常见的UI值选择输入：

- `formSingleSelect`
- `formMultiSelect`
- `formRadioButtons`
- `formCheckboxes`

四个宏中的每个宏都接受一个`Map`选项，这些选项包含表单字段的值以及与该值对应的标签。值和标签可以相同。

下一个示例是FTL中的单选按钮。支持表单的对象为此字段指定==默认值“London”==，因此无需验证。呈现表单时，将在模型中以“ cityMap”为名称提供可供选择的整个城市列表作为**参考数据**。以下清单显示了示例：

```jsp
...
Town:
<@spring.formRadioButtons "command.address.town", cityMap, ""/><br><br>
```

上面的清单呈现了一行单选按钮，每个单选按钮用于中的每个值`cityMap`，并使用的分隔符`""`。没有提供其他属性（缺少该宏的最后一个参数）。在`cityMap`使用相同的`String`在图中的每个键-值对。映射键是表单实际作为`POST`请求参数提交的键。映射值是用户看到的标签。在前面的示例中，给定三个知名城市的列表以及表单支持对象中的**默认值**，HTML类似于以下内容：

```jsp
Town:
<input type="radio" name="address.town" value="London">London</input>
<input type="radio" name="address.town" value="Paris" checked="checked">Paris</input>
<input type="radio" name="address.town" value="New York">New York</input>
```

如果您的应用程序希望通过内部代码处理城市（例如），则可以使用合适的键创建代码地图，如以下示例所示：

```java
protected Map<String, ?> referenceData(HttpServletRequest request) throws Exception {
    Map<String, String> cityMap = new LinkedHashMap<>();
    cityMap.put("LDN", "London");
    cityMap.put("PRS", "Paris");
    cityMap.put("NYC", "New York");

    Map<String, Object> model = new HashMap<>();
    model.put("cityMap", cityMap);
    return model;
}
```

现在，该代码将产生输出，其中无线电值是相关代码，但是用户仍然可以看到更加用户友好的城市名称，如下所示：

```jsp
Town:
<input type="radio" name="address.town" value="LDN">London</input>
<input type="radio" name="address.town" value="PRS" checked="checked">Paris</input>
<input type="radio" name="address.town" value="NYC">New York</input>
```

#### HTML Escaping (HTML转义)

前面描述的表单宏的默认用法导致HTML元素符合HTML 4.01，并且使用`web.xml`文件中定义的HTML转义的默认值（如Spring的绑定支持所使用）。要使元素符合XHTML或**覆盖**默认的HTML转义值，您可以在模板（或模型中对模板可见的位置）中指定两个变量。在模板中指定它们的优点是，可以在稍后的模板处理中将它们更改为不同的值，以为表单中的不同字段提供不同的行为。

要为您的标记**切换到XHTML兼容性**，请`true`为名为的模型或上下文变量指定的值`xhtmlCompliant`，如以下示例所示：

```jsp
<#-- for FreeMarker -->
<#assign xhtmlCompliant = true>
```

处理完此指令后，Spring宏生成的任何元素现在都符合XHTML。

以类似的方式，您可以指定每个字段的HTML转义，如以下示例所示：

```jsp
<#-- until this point, default HTML escaping is used -->

<#assign htmlEscape = true>
<#-- next field will use HTML escaping -->
<@spring.formInput "command.name"/>

<#assign htmlEscape = false in spring>
<#-- all future fields will be bound with HTML escaping off -->
```

### 1.10.3. Groovy Markup (Groovy标记)

在[Groovy的标记模板引擎](http://groovy-lang.org/templating.html#_the_markuptemplateengine) 的主要目的是生成XML类标记（XML，XHTML，HTML5等），但你可以用它来生成**任何基于文本的内容**。Spring框架具有内置的集成，可以将Spring MVC与Groovy标记一起使用。

| ![image-20201202171608157](Spring%20Web%20MVC.assets/image-20201202171608157.png) | Groovy标记模板引擎需要Groovy 2.3.1+。 |
| ------------------------------------------------------------ | ------------------------------------- |
|                                                              |                                       |

#### Configuration 配置

以下示例显示了如何配置Groovy标记模板引擎：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.groovy();
    }

    // Configure the Groovy Markup Template Engine...

    @Bean
    public GroovyMarkupConfigurer groovyMarkupConfigurer() {
        GroovyMarkupConfigurer configurer = new GroovyMarkupConfigurer();
        configurer.setResourceLoaderPath("/WEB-INF/");
        return configurer;
    }
}
```

以下示例显示了如何在XML中进行配置：

```xml
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:groovy/>
</mvc:view-resolvers>

<!-- Configure the Groovy Markup Template Engine... -->
<mvc:groovy-configurer resource-loader-path="/WEB-INF/"/>
```

####  Example （范例）

Unlike traditional template engines, Groovy Markup relies on a DSL that uses a builder syntax. The following example shows a sample template for an HTML page:

```GROOVy
yieldUnescaped '<!DOCTYPE html>'
html(lang:'en') {
    head {
        meta('http-equiv':'"Content-Type" content="text/html; charset=utf-8"')
        title('My page')
    }
    body {
        p('This is an example of HTML contents')
    }
}
```

### 1.10.4. Script Views 脚本视图

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view-script)

Spring框架具有一个内置的集成，可以将Spring MVC与可以在[JSR-223](https://www.jcp.org/en/jsr/detail?id=223) Java脚本引擎之上运行的任何模板库一起使用 。我们已经在不同的脚本引擎上测试了以下模板库：

|                            脚本库                            |                       脚本引擎                        |
| :----------------------------------------------------------: | :---------------------------------------------------: |
|           [Handlebars](https://handlebarsjs.com/)            | [Nashorn](https://openjdk.java.net/projects/nashorn/) |
|           [Mustache](https://mustache.github.io/)            | [Nashorn](https://openjdk.java.net/projects/nashorn/) |
|          [React](https://facebook.github.io/react/)          | [Nashorn](https://openjdk.java.net/projects/nashorn/) |
|              [EJS](https://www.embeddedjs.com/)              | [Nashorn](https://openjdk.java.net/projects/nashorn/) |
|      [ERB](https://www.stuartellis.name/articles/erb/)       |            [JRuby](https://www.jruby.org/)            |
| [String templates](https://docs.python.org/2/library/string.html#template-strings) |           [Jython](https://www.jython.org/)           |
| [Kotlin Script templating](https://github.com/sdeleuze/kotlin-script-templating) |           [Kotlin](https://kotlinlang.org/)           |

| ![image-20201202171949601](Spring%20Web%20MVC.assets/image-20201202171949601.png) | 集成任何其他脚本引擎的基本规则是，它必须实现ScriptEngine和Invocable接口。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

####  Requirements 要求

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view-script-dependencies)

您需要在类路径上具有脚本引擎，其细节因脚本引擎而异：

- 在[纳斯霍恩](https://openjdk.java.net/projects/nashorn/)的**JavaScript引擎**提供的Java 8+。强烈建议使用可用的最新更新版本。
- 应该将[JRuby](https://www.jruby.org/)添加为**对Ruby支持的依赖**。
- 应该将[Jython](https://www.jython.org/)添加为对Python支持的依赖项。
- `org.jetbrains.kotlin:kotlin-script-util`依赖关系和`META-INF/services/javax.script.ScriptEngineFactory` 包含`org.jetbrains.kotlin.script.jsr223.KotlinJsr223JvmLocalScriptEngineFactory` 一行的文件应添加以==支持Kotlin脚本==。有关更多详细信息，请参 [见此示例](https://github.com/sdeleuze/kotlin-script-templating)。

您需要具有脚本模板库。针对Javascript的一种方法是通过[WebJars](https://www.webjars.org/)。

#### Script Templates 脚本模板

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-script-integrate)

您可以声明一个`ScriptTemplateConfigurer`bean，以指定要使用的脚本引擎，要加载的脚本文件，调用呈现模板的函数等等。以下示例使用Mustache模板和Nashorn JavaScript引擎：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.scriptTemplate();
    }

    @Bean
    public ScriptTemplateConfigurer configurer() {
        ScriptTemplateConfigurer configurer = new ScriptTemplateConfigurer();
        configurer.setEngineName("nashorn");
        configurer.setScripts("mustache.js");
        configurer.setRenderObject("Mustache");
        configurer.setRenderFunction("render");
        return configurer;
    }
}
```

以下示例显示了XML中的相同排列：

```xml
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:script-template/>
</mvc:view-resolvers>

<mvc:script-template-configurer engine-name="nashorn" render-object="Mustache" render-function="render">
    <mvc:script location="mustache.js"/>
</mvc:script-template-configurer>
```

对于Java和XML配置，控制器看起来没有什么不同，如以下示例所示：

```java
@Controller
public class SampleController {

    @GetMapping("/sample")
    public String test(Model model) {
        model.addAttribute("title", "Sample title");
        model.addAttribute("body", "Sample body");
        return "template";
    }
}
```

以下示例显示了Mustache模板：

```html
<html>
    <head>
        <title>{{title}}</title>
    </head>
    <body>
        <p>{{body}}</p>
    </body>
</html>
```

使用以下参数调用render函数：

- `String template`：模板内容
- `Map model`：视图模型
- `RenderingContext renderingContext`： [`RenderingContext`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/view/script/RenderingContext.html) 可以访问应用程序上下文，语言环境，模板加载器和URL（自5.0起）

`Mustache.render()` 与该签名本地兼容，因此您可以直接调用它。

如果您的模板技术需要一些自定义，则可以提供一个实现**自定义渲染功能**的脚本。例如，[Handlerbars](https://handlebarsjs.com/) 需要使用它们之前编译模板和需要 [填充工具](https://en.wikipedia.org/wiki/Polyfill)来模拟浏览器的一些设施，不可用在服务器端脚本引擎。

以下示例显示了如何执行此操作：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.scriptTemplate();
    }

    @Bean
    public ScriptTemplateConfigurer configurer() {
        ScriptTemplateConfigurer configurer = new ScriptTemplateConfigurer();
        configurer.setEngineName("nashorn");
        configurer.setScripts("polyfill.js", "handlebars.js", "render.js");
        configurer.setRenderFunction("render");
        configurer.setSharedEngine(false);
        return configurer;
    }
}

```

| ![image-20201202172309776](Spring%20Web%20MVC.assets/image-20201202172309776.png) | 当使用非线程安全脚本引擎和非并发设计的模板库（例如在Nashorn上运行的Handlebars或React）时`sharedEngine`，`false`需要 将该属性设置为。在这种情况下，由于[此bug](https://bugs.openjdk.java.net/browse/JDK-8076099)，需要Java SE 8 update 60 ，但通常建议在任何情况下都使用最新的Java SE修补程序版本。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

`polyfill.js`定义**仅**使`window`Handlebars正常运行所需的对象，如下所示：

```javas
var window = {};
```

此基本`render.js`实现在使用模板之前先对其进行编译。**生产就绪**的实现还应该存储任何重用的==缓存模板或预编译的模板==。您可以在脚本方面进行操作（并处理所需的任何自定义，例如，管理模板引擎配置）。以下示例显示了如何执行此操作：

```javascript
function render(template, model) {
    var compiledTemplate = Handlebars.compile(template);
    return compiledTemplate(model);
}
```

查看Spring Framework单元测试， [Java](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc/src/test/java/org/springframework/web/servlet/view/script)和 [资源](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc/src/test/resources/org/springframework/web/servlet/view/script)，以获取更多配置示例。

###  1.10.5. JSP and JSTL (JSP和JSTL)

Spring框架具有**内置的集成**，可以将Spring MVC与JSP和JSTL一起使用。

#### View Resolvers (视图解析器)

使用JSP进行开发时，通常会声明一个`InternalResourceViewResolver`bean。

`InternalResourceViewResolver`可以用于调度到任何Servlet资源，尤其是JSP。作为最佳实践，我们强烈建议您将JSP文件放在该目录下的`'WEB-INF'`目录中，以便客户端无法直接访问。

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

#### JSPs versus JSTL (JSP与JSTL)

使用JSP标准标记库（JSTL）时，必须使用特殊的视图类 `JstlView`，因为JSTL需要一些准备工作，然后I18N功能才能正常工作。

####  Spring’s JSP Tag Library (Spring的JSP标签库)

如前几章所述，Spring提供了请求参数到命令对象的数据绑定。为了促进结合这些数据绑定功能的JSP页面的开发，Spring提供了一些使事情变得更加容易的标记。==所有Spring标记都具有HTML转义功能==，以**启用或禁用**字符转义。

该`spring.tld`标签库描述符（TLD）包含在`spring-webmvc.jar`。有关单个标签的全面参考，请浏览 [API参考](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/tags/package-summary.html#package.description) 或查看标签库说明。

####  Spring’s form tag library (Spring的表单标签库)

从2.0版开始，Spring使用**JSP和Spring Web MVC**提供了一套全面的数据绑定感知标签，用于处理表单元素。每个标签都支持其对应的HTML标签对等物的属性集，从而使标签熟悉且易于使用。标记生成的HTML**符合HTML 4.01 / XHTML 1.0**。

与其他表单/输入标签库不同，Spring的表单标签库与Spring Web MVC集成在一起，使标签可以访问命令对象和控制器处理的参考数据。如下面的示例所示，表单标签使JSP易于开发，读取和维护。

我们浏览表单标签，并查看有关如何使用每个标签的示例。我们包含了生成的HTML代码段，其中某些标记需要进一步的注释。

##### Configuration组件

表单标签库捆绑在中`spring-webmvc.jar`。库描述符称为`spring-form.tld`。

要使用此库中的标记，请在JSP页面顶部添加以下指令：

```java
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
```

`form`您要用于此库中标签的标签名称前缀在哪里。

##### **The Form Tag **表单标签

此标记呈现HTML'form'元素，并向内部标记公开绑定路径以进行绑定。它将命令对象放入中，`PageContext`以便内部标签可以访问该命令对象。该库中的所有其他标签都是该`form`标签的嵌套标签 。

假设我们有一个名为的域对象`User`。它是具有诸如`firstName`和属性的JavaBean `lastName`。我们可以将其用作返回的表单控制器的表单支持对象`form.jsp`。以下示例显示了`form.jsp`可能的样子：

```xml
<form:form>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

该`firstName`和`lastName`值由放置在命令对象中检索`PageContext`由页控制器。继续阅读以查看内部标签如何与标签一起使用的更复杂的示例`form`。

下面的清单显示了生成的HTML，它看起来像标准格式：

```xml
<form method="POST">
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value="Harry"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value="Potter"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```

前面的JSP假定表单支持对象的变量名称为 `command`。如果已将表单支持对象以另一个名称（肯定是**最佳实践**）放入模型中，则可以将表单绑定到命名变量，如以下示例所示：

```xml
<form:form modelAttribute="user">
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

##### **The** `input` **Tag** 输入标签

默认情况下，此标记呈现具有`input`绑定值的HTML元素`type='text'`。有关此标签的示例，请参见[The Form标签](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view-jsp-formtaglib-formtag)。您还可以使用特定的HTML5类型，如`email`，`tel`，`date`，等。

##### **The** `checkbox` **Tag** 检查框标签

此标记呈现一个HTML`input`标记，其`type`设置为`checkbox`。

假设我们`User`有喜好，例如时事通讯订阅和兴趣爱好列表。以下示例显示了`Preferences`该类：

```java
public class Preferences {

    private boolean receiveNewsletter;
    private String[] interests;
    private String favouriteWord;

    public boolean isReceiveNewsletter() {
        return receiveNewsletter;
    }

    public void setReceiveNewsletter(boolean receiveNewsletter) {
        this.receiveNewsletter = receiveNewsletter;
    }

    public String[] getInterests() {
        return interests;
    }

    public void setInterests(String[] interests) {
        this.interests = interests;
    }

    public String getFavouriteWord() {
        return favouriteWord;
    }

    public void setFavouriteWord(String favouriteWord) {
        this.favouriteWord = favouriteWord;
    }
}
```

对应的内容`form.jsp`可能类似于以下内容：

```xml
<form:form>
    <table>
        <tr>
            <td>Subscribe to newsletter?:</td>
            <%-- Approach 1: Property is of type java.lang.Boolean --%>
            <td><form:checkbox path="preferences.receiveNewsletter"/></td>
        </tr>

        <tr>
            <td>Interests:</td>
            <%-- Approach 2: Property is of an array or of type java.util.Collection --%>
            <td>
                Quidditch: <form:checkbox path="preferences.interests" value="Quidditch"/>
                Herbology: <form:checkbox path="preferences.interests" value="Herbology"/>
                Defence Against the Dark Arts: <form:checkbox path="preferences.interests" value="Defence Against the Dark Arts"/>
            </td>
        </tr>

        <tr>
            <td>Favourite Word:</td>
            <%-- Approach 3: Property is of type java.lang.Object --%>
            <td>
                Magic: <form:checkbox path="preferences.favouriteWord" value="Magic"/>
            </td>
        </tr>
    </table>
</form:form>
```

`checkbox`标记有三种方法，应该可以满足您所有复选框的需求。

- 方法一：当界限值是type时`java.lang.Boolean`，将 `input(checkbox)`标记为`checked`界限值是`true`。该`value` 属性对应于`setValue(Object)`value属性的解析值。
- 方法二：当界限值的类型为`array`或时`java.util.Collection`，将 `input(checkbox)`标记为，`checked`好像`setValue(Object)`界限中存在配置的值`Collection`。
- 方法三：对于任何其他绑定值类型，将`input(checkbox)`标记为 `checked`配置的`setValue(Object)`值等于绑定值。

请注意，无论采用哪种方法，都会生成相同的HTML结构。以下HTML代码段定义了一些复选框：

```xml
<tr>
    <td>Interests:</td>
    <td>
        Quidditch: <input name="preferences.interests" type="checkbox" value="Quidditch"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
        Herbology: <input name="preferences.interests" type="checkbox" value="Herbology"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
        Defence Against the Dark Arts: <input name="preferences.interests" type="checkbox" value="Defence Against the Dark Arts"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
    </td>
</tr>
```

您可能不希望在每个复选框之后看到其他**隐藏字段**。如果未选中HTML页面中的复选框，则提交表单后，其值就不会作为HTTP请求参数的一部分发送到服务器，因此我们需要一种解决方法来使HTML中的这个问题生效，以使Spring表单数据绑定生效。该 `checkbox`标签包括如下**通过下划线**（前缀一个隐藏参数的Spring现有的惯例`_`用于每个复选框）。通过这样做，您可以有效地告诉Spring：“该复选框在表单中可见，并且我希望与表单数据绑定的对象能够反映**该复选框**的状态，无论如何。”

##### **The** `checkboxes` **Tag**  (checkboxes 标签)

此标记呈现设置为的多个HTML`input`标记。`type``checkbox`

本节以上一个`checkbox`标记节的示例为基础。有时，您宁愿不必在JSP页面中列出所有可能的爱好。您宁愿在运行时提供可用选项的列表，然后将其传递给标记。这就是`checkboxes`标签的目的。您可以传递一个`Array`，一个`List`，或者`Map`包含在可用的选项`items`属性。通常，bound属性是一个集合，因此它可以保存用户选择的多个值。以下示例显示了使用此标记的JSP：

```xml
<form:form>
    <table>
        <tr>
            <td>Interests:</td>
            <td>
                <%-- Property is of an array or of type java.util.Collection --%>
                <form:checkboxes path="preferences.interests" items="${interestList}"/>
            </td>
        </tr>
    </table>
</form:form>
```

这个例子假设`interestList`是`List`可用的为包含的值的字符串从被选择的模型的属性。如果使用`Map`，则将地图输入键用作值，并将地图输入的值用作要显示的标签。您还可以**使用自定义对象**，在其中您可以通过使用提供值的属性名称，并通过使用提供`itemValue`标签`itemLabel`。

##### **The** `radiobutton` **Tag** （radiobutton 标签）

此标记呈现一个HTML`input`元素，其`type`设置为`radio`。

典型的用法模式涉及绑定到相同属性但值不同的多个标记实例，如以下示例所示：

```xml
<tr>
    <td>Sex:</td>
    <td>
        Male: <form:radiobutton path="sex" value="M"/> <br/>
        Female: <form:radiobutton path="sex" value="F"/>
    </td>
</tr>
```

##### **The** `radiobuttons` **Tag** （radiobuttons 标签）

此标记呈现多个HTML`input`元素，其`type`设置为`radio`。

与[`checkboxes`标记一样](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view-jsp-formtaglib-checkboxestag)，您可能希望将可用选项作为运行时变量传递。对于这种用法，您可以使用 `radiobuttons`标签。你传递一个`Array`，一个`List`，或者`Map`包含在可用的选项`items`属性。如果使用`Map`，则将地图条目键用作值，并将地图条目的值用作要显示的标签。您还可以使用一个自定义对象，在其中可以使用来提供值的属性名称，并使用来提供`itemValue`标签`itemLabel`，如以下示例所示：

```xml
<tr>
    <td>Sex:</td>
    <td><form:radiobuttons path="sex" items="${sexOptions}"/></td>
</tr>
```

##### **The** `password` **Tag** （password标签）

此标记呈现`input`类型为`password`具有绑定值的HTML标记。

```xml
<tr>
    <td>Password:</td>
    <td>
        <form:password path="password"/>
    </td>
</tr>
```

请注意，默认情况下，不显示密码值。如果确实希望显示密码值，则可以将`showPassword`属性的值设置为 `true`，如以下示例所示：

```xml
<tr>
    <td>Password:</td>
    <td>
        <form:password path="password" value="^76525bvHGq" showPassword="true"/>
    </td>
</tr>
```

##### **The** `select` **Tag** （select标签）

此标记呈现HTML“ select”元素。它支持将数据绑定到所选选项以及使用嵌套`option`和`options`标记。

假设a`User`具有一系列技能。相应的HTML可能如下所示：

```xml
<tr>
    <td>Skills:</td>
    <td><form:select path="skills" items="${skills}"/></td>
</tr>
```

如果该`User’s`技能是 Herbology，则“Skills”行的HTML源可能如下：

```xml
<tr>
    <td>Skills:</td>
    <td>
        <select name="skills" multiple="true">
            <option value="Potions">Potions</option>
            <option value="Herbology" selected="selected">Herbology</option>
            <option value="Quidditch">Quidditch</option>
        </select>
    </td>
</tr>
```



##### **The** `option` **Tag** （option标签）

此标记呈现HTML`option`元素。它集`selected`的基础上限值。以下HTML显示了其典型输出：

```xml
<tr>
    <td>House:</td>
    <td>
        <form:select path="house">
            <form:option value="Gryffindor"/>
            <form:option value="Hufflepuff"/>
            <form:option value="Ravenclaw"/>
            <form:option value="Slytherin"/>
        </form:select>
    </td>
</tr>
```

如果`User’s` house位于Gryffindor，则'House' 行的HTML源代码如下：

```xml
<tr>
    <td>House:</td>
    <td>
        <select name="house">
            <option value="Gryffindor" selected="selected">Gryffindor</option> 注释：1
            <option value="Hufflepuff">Hufflepuff</option>
            <option value="Ravenclaw">Ravenclaw</option>
            <option value="Slytherin">Slytherin</option>
        </select>
    </td>
</tr>
```

| ![image-20201202174418764](Spring%20Web%20MVC.assets/image-20201202174418764.png) | 注意添加了一个选定的属性。 |
| ------------------------------------------------------------ | -------------------------- |
|                                                              |                            |

##### **The** `options` **Tag** （options标签）

此标记呈现HTML`option`元素列表。它`selected`**根据绑定值设置属性**。以下HTML显示了其典型输出：

```xml
<tr>
    <td>Country:</td>
    <td>
        <form:select path="country">
            <form:option value="-" label="--Please Select"/>
            <form:options items="${countryList}" itemValue="code" itemLabel="name"/>
        </form:select>
    </td>
</tr>
```

如果`User`居住在UK，则“Country”行的HTML来源如下：

```xml
<tr>
    <td>Country:</td>
    <td>
        <select name="country">
            <option value="-">--Please Select</option>
            <option value="AT">Austria</option>
            <option value="UK" selected="selected">United Kingdom</option> 注释：1
            <option value="US">United States</option>
        </select>
    </td>
</tr>
```

| ![image-20201202174513818](Spring%20Web%20MVC.assets/image-20201202174513818.png) | 注意添加了一个`selected`属性。 |
| ------------------------------------------------------------ | ------------------------------ |
|                                                              |                                |

如前面的示例所示，`option`标记与`options`标记的组合用法会生成相同的标准HTML，但可以让您在JSP中显式指定一个仅用于显示（该标记所属的位置）的值，例如示例中的==默认字符串： “ -- Please Select”==。

`items`通常，该属性填充有项目对象的集合或数组。 `itemValue`并`itemLabel`引用这些项目对象的bean属性（如果已指定）。否则，项目对象本身将变成字符串。或者，您可以指定一个`Map`项，在这种情况下，==映射键==将解释为选项值，并且映射值对应于选项标签。如果碰巧也指定了`itemValue`或`itemLabel`（或两者），则item value属性适用于地图键，item label属性适用于地图值。

##### **The** `textarea` **Tag** (textarea标签)

此标记呈现HTML`textarea`元素。以下HTML显示了其典型输出：

```xml
<tr>
    <td>Notes:</td>
    <td><form:textarea path="notes" rows="3" cols="20"/></td>
    <td><form:errors path="notes"/></td>
</tr>
```



##### **The** `hidden` **Tag** (hidden标签)

此标记`input`使用`type`设置`hidden`的绑定值呈现HTML标记。要提交==未绑定==的隐藏值，请使用设置为的HTML`input`标签。以下HTML显示了其典型输出：type hidden

```xml
<form:hidden path="house"/>
```

如果我们选择将`house`值作为隐藏值提交，则HTML如下所示：

```xml
<input name="house" type="hidden" value="Gryffindor"/>
```



##### **The** `errors` **Tag**  (errors标签)

此标记在HTML`span`元素中**呈现字段错误**。它提供对在控制器中创建的错误或由与控制器关联的任何**验证程序**创建的错误的访问。

假设 提交表单后，我们希望显示`firstName`和`lastName`字段的所有错误消息。我们有一个`User`名为的类实例的验证器`UserValidator`，如以下示例所示：

```java
public class UserValidator implements Validator {

    public boolean supports(Class candidate) {
        return User.class.isAssignableFrom(candidate);
    }

    public void validate(Object obj, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "required", "Field is required.");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "lastName", "required", "Field is required.");
    }
}
```

在`form.jsp`可能如下：

```xml
<form:form>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
            <%-- Show errors for firstName field --%>
            <td><form:errors path="firstName"/></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
            <%-- Show errors for lastName field --%>
            <td><form:errors path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

如果我们在`firstName`和`lastName`字段中提交具有空值的表单，则HTML如下所示：

```xml
<form method="POST">
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value=""/></td>
            <%-- Associated errors to firstName field displayed --%>
            <td><span name="firstName.errors">Field is required.</span></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value=""/></td>
            <%-- Associated errors to lastName field displayed --%>
            <td><span name="lastName.errors">Field is required.</span></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```

如果我们要显示给定页面的整个错误列表怎么办？下一个示例显示该`errors`标记还支持一些**基本的通配符功能**。

- `path="*"`：显示所有错误。
- `path="lastName"`：显示与该`lastName`字段相关的所有错误。
- 如果`path`省略，则仅显示对象错误。

以下示例在页面顶部显示错误列表，然后在字段旁边显示特定于字段的错误：

```xml
<form:form>
    <form:errors path="*" cssClass="errorBox"/>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
            <td><form:errors path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
            <td><form:errors path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

HTML将如下所示：

```xml
<form method="POST">
    <span name="*.errors" class="errorBox">Field is required.<br/>Field is required.</span>
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value=""/></td>
            <td><span name="firstName.errors">Field is required.</span></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value=""/></td>
            <td><span name="lastName.errors">Field is required.</span></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```

该`spring-form.tld`标签库描述符（TLD）包含在`spring-webmvc.jar`。有关单个标签的全面参考，请浏览 [API参考](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/tags/form/package-summary.html#package.description) 或查看标签库说明。

#### HTTP Method Conversion   **HTTP方法转换**

REST的一个关键原则是使用“统一接口”。这意味着可以使用相同的四种HTTP方法（GET，PUT，POST和DELETE）来操纵所有资源（URL）。对于每种方法，HTTP规范都定义了**确切的语义**。例如，GET应该始终是安全的操作，这意味着它没有副作用，而PUT或DELETE应该是幂等的，这意味着您可以一遍又一遍地重复这些操作，但是最终结果应该相同。虽然HTTP定义了这四种方法，但HTML仅支持两种：**GET和POST**。幸运的是，有两种可能的解决方法：您可以使用JavaScript进行**PUT或DELETE**，或者可以使用“ real”方法作为附加参数（在HTML表单中建模为**隐藏的输入字段**）进行POST。春天的`HiddenHttpMethodFilter`使用后一种技巧。该过滤器是一个普通的Servlet过滤器，因此，它可以与任何Web框架（不仅仅是Spring MVC）结合使用。将此过滤器添加到您的web.xml，然后将带有隐藏`method`参数的POST转换为相应的HTTP方法请求。

为了支持HTTP方法转换，Spring MVC表单标签已更新为支持设置HTTP方法。例如，以下代码片段来自“ the Pet Clinic”样本：

```xml
<form:form method="delete">
    <p class="submit"><input type="submit" value="Delete Pet"/></p>
</form:form>
```

前面的示例执行HTTP POST，并将“真实” DELETE方法**隐藏**在请求参数后面。它是由`HiddenHttpMethodFilter`web.xml中定义的拾取的，如以下示例所示：

```xml
<filter>
    <filter-name>httpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>httpMethodFilter</filter-name>
    <servlet-name>petclinic</servlet-name>
</filter-mapping>
```

以下示例显示了相应的`@Controller`方法：

```java
@RequestMapping(method = RequestMethod.DELETE)
public String deletePet(@PathVariable int ownerId, @PathVariable int petId) {
    this.clinic.deletePet(petId);
    return "redirect:/owners/" + ownerId;
}
```

#### **HTML5 Tags** (HTML5 标签)

Spring表单标签库允许**输入动态属性**，这意味着您可以输入**任何HTML5**特定的属性。

表单`input`标签支持输入以外的类型属性`text`。这是为了让渲染新的HTML5特定的输入类型，如`email`，`date`， `range`，等。请注意`type='text'`，由于`text` 默认类型，因此不需要输入。

### 1.10.6. Tiles

您可以像使用其他视图技术一样，将Tiles集成到使用Spring的Web应用程序中。本节将广泛介绍如何执行此操作。

| ![image-20201202183436532](Spring%20Web%20MVC.assets/image-20201202183436532.png) | 本节重点介绍Spring对`org.springframework.web.servlet.view.tiles3`软件包中Tiles版本3的支持 。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

#### Dependencies 依赖

为了能够使用Tiles，您必须在Tiles 3.0.1或更高版本上添加一个依赖项，并将[其传递依赖项添加](https://tiles.apache.org/framework/dependency-management.html) 到您的项目中。

####  Configuration 配置

为了能够使用Tiles，必须使用包含定义的文件对其进行配置（有关定义和其他Tiles概念的基本信息，请参见 [https://tiles.apache.org](https://tiles.apache.org/)）。在Spring中，这是通过使用来完成的`TilesConfigurer`。以下示例`ApplicationContext`配置显示了如何执行此操作：

```xml
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <property name="definitions">
        <list>
            <value>/WEB-INF/defs/general.xml</value>
            <value>/WEB-INF/defs/widgets.xml</value>
            <value>/WEB-INF/defs/administrator.xml</value>
            <value>/WEB-INF/defs/customer.xml</value>
            <value>/WEB-INF/defs/templates.xml</value>
        </list>
    </property>
</bean>
```

前面的示例定义了五个包含定义的文件。这些文件都位于`WEB-INF/defs`目录中。在初始化时`WebApplicationContext`，将加载文件，并初始化定义工厂。完成之后，**定义文件**中包含的Tiles可以用作Spring Web应用程序中的视图。为了能够使用视图，您必须`ViewResolver` 在Spring中拥有与其他任何视图技术一样的功能：通常很方便`TilesViewResolver`。

您可以通过添加下划线然后添加语言环境来指定特定于语言环境的Tiles定义，如以下示例所示：

```xml
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <property name="definitions">
        <list>
            <value>/WEB-INF/defs/tiles.xml</value>
            <value>/WEB-INF/defs/tiles_fr_FR.xml</value>
        </list>
    </property>
</bean>
```

使用前面的配置时，`tiles_fr_FR.xml`用于具有`fr_FR`语言环境的请求，并且`tiles.xml`默认情况下使用。

| ![image-20201202183541244](Spring%20Web%20MVC.assets/image-20201202183541244.png) | 由于下划线用于指示语言环境，因此建议不要在图块定义的文件名中使用下划线。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

##### UrlBasedViewResolver 基于视图解析器的url

该`UrlBasedViewResolver`实例化给出`viewClass`了它解决每个视图。以下bean定义了一个`UrlBasedViewResolver`：

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.UrlBasedViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.tiles3.TilesView"/>
</bean>
```

###### `SimpleSpringPreparerFactory` 和 `SpringBeanPreparerFactory`

作为一项**高级功能**，Spring还支持两种特殊的Tiles`PreparerFactory` 实现。有关如何`ViewPreparer`在Tiles定义文件中使用引用的详细信息，请参见Tiles文档 。

您可以指定根据指定的准备器类`SimpleSpringPreparerFactory`自动装配`ViewPreparer`实例，应用Spring的容器回调以及应用配置的Spring BeanPostProcessors。如果已经激活了Spring的上下文范围注释配置，则将`ViewPreparer`自动检测并应用类中的注释。请注意，这与默认设置`PreparerFactory`一样，期望Tiles定义文件中的**准备程序类**。

您可以指定`SpringBeanPreparerFactory`对指定的**准备器名称**（而不是类）进行操作，从而从DispatcherServlet的应用程序上下文中获取相应的Spring bean。在这种情况下，完整的bean创建过程由Spring应用程序上下文控制，从而允许使用**显式依赖项注入**配置，作用域bean等。请注意，您需要为每个准备器名称定义一个Spring bean定义（在Tiles定义中使用）。以下示例显示如何`SpringBeanPreparerFactory`在`TilesConfigurer`bean上定义属性：

```xml
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <property name="definitions">
        <list>
            <value>/WEB-INF/defs/general.xml</value>
            <value>/WEB-INF/defs/widgets.xml</value>
            <value>/WEB-INF/defs/administrator.xml</value>
            <value>/WEB-INF/defs/customer.xml</value>
            <value>/WEB-INF/defs/templates.xml</value>
        </list>
    </property>

    <!-- resolving preparer names as Spring bean definition names -->
    <property name="preparerFactoryClass"
            value="org.springframework.web.servlet.view.tiles3.SpringBeanPreparerFactory"/>

</bean>
```

### 1.10.7. RSS and Atom

二者`AbstractAtomFeedView`并`AbstractRssFeedView`从继承 `AbstractFeedView`基类和用于提供Atom和RSS提要视图，分别。它们基于[ROME](https://rometools.github.io/rome/)项目，位于软件包中`org.springframework.web.servlet.view.feed`。

`AbstractAtomFeedView`需要您实现该`buildFeedEntries()`方法并有选择地重写该`buildFeedMetadata()`方法（默认实现为空）。以下示例显示了如何执行此操作：

```java
public class SampleContentAtomView extends AbstractAtomFeedView {

    @Override
    protected void buildFeedMetadata(Map<String, Object> model,
            Feed feed, HttpServletRequest request) {
        // implementation omitted
    }

    @Override
    protected List<Entry> buildFeedEntries(Map<String, Object> model,
            HttpServletRequest request, HttpServletResponse response) throws Exception {
        // implementation omitted
    }
}
```

实施类似的要求`AbstractRssFeedView`，如以下示例所示：

```java
public class SampleContentRssView extends AbstractRssFeedView {

    @Override
    protected void buildFeedMetadata(Map<String, Object> model,
            Channel feed, HttpServletRequest request) {
        // implementation omitted
    }

    @Override
    protected List<Item> buildFeedItems(Map<String, Object> model,
            HttpServletRequest request, HttpServletResponse response) throws Exception {
        // implementation omitted
    }
}
```

该`buildFeedItems()`和`buildFeedEntries()`在HTTP请求方法传递，如果你需要访问的语言环境。**仅针对**Cookie或其他HTTP标头的设置传入HTTP响应。方法返回后，该提要将自动写入响应对象。

有关创建Atom视图的示例，请参见Alef Arendsen的Spring Team Blog [条目](https://spring.io/blog/2009/03/16/adding-an-atom-view-to-an-application-using-spring-s-rest-support)。

###  1.10.8. PDF and Excel

Spring提供了返回HTML以外的输出的方法，**包括PDF和Excel电子表格**。本节介绍如何使用这些功能。

#### Introduction to Document Views 文档视图简介

HTML页面并非始终是用户查看模型输出的最佳方法，而Spring使从模型数据==动态生成PDF文档或Excel电子表格==变得简单。该文档是视图，并从服务器以正确的内容类型进行流传输，以（有希望）使客户端PC能够运行其电子表格或PDF查看器应用程序作为响应。

为了使用Excel视图，您需要将Apache POI库添加到您的类路径中。为了生成PDF，您需要添加（最好是）OpenPDF库。

| ![image-20201202183924228](Spring%20Web%20MVC.assets/image-20201202183924228.png) | 如果可能，您应该使用基础文档生成库的最新版本。特别是，我们==强烈建议==您使用OpenPDF（例如，OpenPDF 1.2.12）而不是过时的原始iText 2.1.7，因为OpenPDF会得到积极维护并修复了不可信任PDF内容的重要漏洞。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

#### PDF Views (PDF视图)

单词列表的简单PDF视图可以扩展 `org.springframework.web.servlet.view.document.AbstractPdfView`和实现该 `buildPdfDocument()`方法，如以下示例所示：

```java
public class PdfWordList extends AbstractPdfView {

    protected void buildPdfDocument(Map<String, Object> model, Document doc, PdfWriter writer,
            HttpServletRequest request, HttpServletResponse response) throws Exception {

        List<String> words = (List<String>) model.get("wordList");
        for (String word : words) {
            doc.add(new Paragraph(word));
        }
    }
}
```

控制器可以从外部视图定义（按名称引用）或作为`View`处理程序方法的实例返回这种视图。

####  Excel Views

从Spring Framework 4.2开始， `org.springframework.web.servlet.view.document.AbstractXlsView`它作为Excel视图的基类提供。它基于Apache POI，具有取代过时类的专用子类（`AbstractXlsxView` 和`AbstractXlsxStreamingView`）`AbstractExcelView`。

编程模型类似于`AbstractPdfView`，`buildExcelDocument()` 作为**中央模板方法**，控制器能够从外部定义（按名称）或`View`从处理程序方法作为实例返回这种视图。

### 1.10.9. Jackson

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view-httpmessagewriter)

Spring提供了==对Jackson JSON库==的支持。

#### Jackson-based JSON MVC Views 基于Jackson的JSON MVC视图

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view-httpmessagewriter)

该`MappingJackson2JsonView`用途杰克逊库的`ObjectMapper`渲染响应内容为JSON。默认情况下，**模型映射**的所有内容（特定于框架的类除外）均编码为JSON。对于需要过滤地图内容的情况，可以使用属性指定一组特定的模型属性进行编码`modelKeys`。您还可以使用该`extractValueFromSingleKeyModel` 属性来直接提取和序列化单键模型中的值，而不是将其作为模型属性的映射。

您可以根据需要使用Jackson提供的注释来自定义JSON映射。当需要进一步控制时，可以为需要为特定类型提供自定义JSON序列化器和反序列化器的情况，`ObjectMapper` 通过`ObjectMapper`属性注入自定义。

####  Jackson-based XML Views  基于Jackson的XML视图

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view-httpmessagewriter)

`MappingJackson2XmlView`使用 [Jackson的XML扩展](https://github.com/FasterXML/jackson-dataformat-xml) `XmlMapper` 名将响应内容呈现为XML。如果模型包含多个条目，则应使用`modelKey`bean属性显式设置要序列化的对象。如果模型包含单个条目，则会**自动序列化**。

您可以根据需要使用JAXB或Jackson提供的注释自定义XML映射。当需要进一步控制时，可以`XmlMapper` 通过`ObjectMapper`属性注入自定义，对于自定义XML的情况，您需要为特定类型提供**序列化器和反序列化器**。

###  1.10.10. XML Marshalling （XML编组）

在`MarshallingView`使用XML `Marshaller`（在定义的`org.springframework.oxm` 包）来呈现响应内容为XML。您可以使用`MarshallingView`实例的`modelKey`bean属性来显式设置要编组的对象。或者，视图迭代所有模型属性，并封送所支持的第一个类型`Marshaller`。有关`org.springframework.oxm`包中功能的更多信息 ，请参见[使用O / X映射器编组XML](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#oxm)。

### 1.10.11. XSLT Views （ XSLT视图）

XSLT是XML的一种转换语言，在Web应用程序中作为视图技术而流行。如果您的应用程序自然地处理XML，或者如果您的模型可以轻松转换为XML，那么XSLT可以作为视图技术的不错选择。下一节显示了如何将XML文档生成为模型数据，以及如何在Spring Web MVC应用程序中使用XSLT对其进行转换。

这个示例是一个普通的Spring应用程序，它在中创建一个单词列表 `Controller`并将其添加到模型图中。返回该地图以及XSLT视图的视图名称。有关Spring Web MVC界面的详细信息， 请参见带[注释的控制器](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-controller)`Controller`。XSLT控制器将单词列表转换为准备转换的简单XML文档。

#### Beans 类

配置是简单Spring Web应用程序的标准配置：MVC配置必须定义一个`XsltViewResolver`bean和常规的MVC注释配置。以下示例显示了如何执行此操作：

```java
@EnableWebMvc
@ComponentScan
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public XsltViewResolver xsltViewResolver() {
        XsltViewResolver viewResolver = new XsltViewResolver();
        viewResolver.setPrefix("/WEB-INF/xsl/");
        viewResolver.setSuffix(".xslt");
        return viewResolver;
    }
}
```



####  Controller 控制器

我们还需要一个==封装词生成逻辑的控制器。==

控制器逻辑封装在一个`@Controller`类中，其中handler方法的定义如下：

```java
@Controller
public class XsltController {

    @RequestMapping("/")
    public String home(Model model) throws Exception {
        Document document = DocumentBuilderFactory.newInstance().newDocumentBuilder().newDocument();
        Element root = document.createElement("wordList");

        List<String> words = Arrays.asList("Hello", "Spring", "Framework");
        for (String word : words) {
            Element wordNode = document.createElement("word");
            Text textNode = document.createTextNode(word);
            wordNode.appendChild(textNode);
            root.appendChild(wordNode);
        }

        model.addAttribute("wordList", root);
        return "home";
    }
}
```

到目前为止，我们仅创建了一个DOM文档并将其添加到Model映射中。请注意，您还可以将XML文件作为加载`Resource`并使用它代替**自定义DOM文档**。

有可用的软件包自动“对象化”对象图，但是在Spring内，您可以完全灵活地以任何选择的方式从模型中创建DOM。这样可以防止XML转换在模型数据的结构中扮演过多的角色，这在使用工具来管理DOMification流程时是一种危险。

####  Transformation 转型

最后，`XsltViewResolver`解析“原始” XSLT模板文件并将DOM文档合并到其中以生成我们的视图。如`XsltViewResolver` 配置中所示，XSLT模板`war`位于`WEB-INF/xsl`目录中的文件中，并以`xslt`文件扩展名结尾。

以下示例显示了XSLT转换：

```xml
<?xml version="1.0" encoding="utf-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

    <xsl:output method="html" omit-xml-declaration="yes"/>

    <xsl:template match="/">
        <html>
            <head><title>Hello!</title></head>
            <body>
                <h1>My First Words</h1>
                <ul>
                    <xsl:apply-templates/>
                </ul>
            </body>
        </html>
    </xsl:template>

    <xsl:template match="word">
        <li><xsl:value-of select="."/></li>
    </xsl:template>

</xsl:stylesheet>
```

前面的转换呈现为以下HTML：

```html
<html>
    <head>
        <META http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Hello!</title>
    </head>
    <body>
        <h1>My First Words</h1>
        <ul>
            <li>Hello</li>
            <li>Spring</li>
            <li>Framework</li>
        </ul>
    </body>
</html>
```



## 1.11. MVC Config (MVC配置)

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config)

MVC Java配置和MVC XML名称空间提供适用于大多数应用程序的默认配置以及用于自定义它的配置API。

有关配置API中不可用的更多高级定制，请参阅[Advanced Java Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-advanced-java)和[Advanced XML Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-advanced-xml)。

您不需要了解由MVC Java配置和MVC名称空间创建的基础bean。如果要了解更多信息，请参见[特殊Bean类型](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet-special-bean-types) 和[Web MVC Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet-config)。

###  1.11.1. Enable MVC Configuration 启用MVC配置

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-enable)

在Java配置中，可以使用`@EnableWebMvc`注释启用MVC配置，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig {
}
```

在XML配置中，可以使用`<mvc:annotation-driven>`元素来启用MVC配置，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven/>

</beans>
```

前面的示例注册了许多Spring MVC [基础结构Bean，](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet-special-bean-types)并适应了类路径上可用的依赖项（例如，JSON，XML等的有效负载转换器）。

### 1.11.2. MVC Config API (MVC配置API)

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-customize)

在Java配置中，您可以实现该`WebMvcConfigurer`接口，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    // Implement configuration methods...
}
```

在XML中，您可以检查的属性和子元素`<mvc:annotation-driven/>`。您可以查看[Spring MVC XML模式](https://schema.spring.io/mvc/spring-mvc.xsd)或使用IDE的代码完成功能来发现可用的属性和子元素。

###  1.11.3. Type Conversion 类型转换

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-conversion)

默认情况下，将安装各种数字和日期类型的==格式化程序==，并支持通过`@NumberFormat`和`@DateTimeFormat`在字段上进行自定义。

要在Java配置中注册自定义格式器和转换器，请使用以下命令：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        // ...
    }
}
```

要在XML配置中执行相同的操作，请使用以下命令：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven conversion-service="conversionService"/>

    <bean id="conversionService"
            class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="org.example.MyConverter"/>
            </set>
        </property>
        <property name="formatters">
            <set>
                <bean class="org.example.MyFormatter"/>
                <bean class="org.example.MyAnnotationFormatterFactory"/>
            </set>
        </property>
        <property name="formatterRegistrars">
            <set>
                <bean class="org.example.MyFormatterRegistrar"/>
            </set>
        </property>
    </bean>

</beans>
```

默认情况下，Spring MVC在解析和格式化日期值时会考虑请求区域设置。这适用于使用“输入”表单字段将日期表示为字符串的表单。但是，对于“日期”和“时间”表单字段，浏览器使用HTML规范中定义的固定格式。在这种情况下，日期和时间格式可以按以下方式自定义：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
        registrar.setUseIsoFormat(true);
        registrar.registerFormatters(registry);
    }
}

```



| ![image-20201202185120715](Spring%20Web%20MVC.assets/image-20201202185120715.png) | 有关何时使用FormatterRegistrar实现的更多信息，请参见[FormatterRegistrar SPI](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#format-FormatterRegistrar-SPI)和FormattingConversionServiceFactoryBean。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

###  1.11.4. Validation 验证方式

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-validation)

默认情况下，如果[Bean验证](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation-beanvalidation-overview)存在于类路径中（例如，Hibernate Validator），则将`LocalValidatorFactoryBean`其注册为全局[验证器](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validator)，以`@Valid`与 `Validated`控制器方法参数一起使用。

在Java配置中，您可以自定义全局`Validator`实例，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public Validator getValidator() {
        // ...
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven validator="globalValidator"/>

</beans>
```

请注意，您还可以`Validator`在本地注册实现，如以下示例所示：

```java
@Controller
public class MyController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addValidators(new FooValidator());
    }
}
```

| ![image-20201202185345103](Spring%20Web%20MVC.assets/image-20201202185345103.png) | 如果需要在`LocalValidatorFactoryBean`某处进行注入，请创建一个bean并标记为Bean，`@Primary`以避免与MVC配置中声明的bean发生冲突。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

###  1.11.5. Interceptors 拦截器

在Java配置中，您可以注册拦截器以应用于传入的请求，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LocaleChangeInterceptor());
        registry.addInterceptor(new ThemeChangeInterceptor()).addPathPatterns("/**").excludePathPatterns("/admin/**");
        registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:interceptors>
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <mvc:exclude-mapping path="/admin/**"/>
        <bean class="org.springframework.web.servlet.theme.ThemeChangeInterceptor"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/secure/*"/>
        <bean class="org.example.SecurityInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

###  1.11.6. Content Types 内容类型

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-content-negotiation)

您可以配置Spring MVC如何根据请求确定请求的媒体类型（例如，`Accept`标头，URL路径扩展，查询参数等）。

默认情况下，URL路径扩展首先检查-有`json`，`xml`，`rss`，并`atom` 注册为已知扩展名（视路径依赖）。的`Accept`报头检查第二。

考虑将这些默认值更改为`Accept`仅标头，并且，如果必须使用基于URL的内容类型解析，请考虑对路径扩展使用查询参数策略。有关更多详细信息，请参见 [后缀匹配](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-suffix-pattern-match)和[后缀匹配以及RFD](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-rfd)。

在Java配置中，您可以自定义请求的内容类型解析，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.mediaType("json", MediaType.APPLICATION_JSON);
        configurer.mediaType("xml", MediaType.APPLICATION_XML);
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:annotation-driven content-negotiation-manager="contentNegotiationManager"/>

<bean id="contentNegotiationManager" class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="mediaTypes">
        <value>
            json=application/json
            xml=application/xml
        </value>
    </property>
</bean>
```

###  1.11.7. Message Converters 信息转换器

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-message-codecs)

您可以`HttpMessageConverter`通过覆盖[`configureMessageConverters()`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurer.html#configureMessageConverters-java.util.List-) （替换由Spring MVC创建的默认转换器）或覆盖 [`extendMessageConverters()`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurer.html#extendMessageConverters-java.util.List-) （定制默认转换器或向默认转换器添加其他转换器）来在Java配置中 进行自定义。

以下示例使用自定义的`ObjectMapper`而不是默认的添加了XML和Jackson JSON转换器 ：

```java
@Configuration
@EnableWebMvc
public class WebConfiguration implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
                .indentOutput(true)
                .dateFormat(new SimpleDateFormat("yyyy-MM-dd"))
                .modulesToInstall(new ParameterNamesModule());
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
        converters.add(new MappingJackson2XmlHttpMessageConverter(builder.createXmlMapper(true).build()));
    }
}
```

在前面的例子中， [`Jackson2ObjectMapperBuilder`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/http/converter/json/Jackson2ObjectMapperBuilder.html) 用于创建两种共同的构成`MappingJackson2HttpMessageConverter`和 `MappingJackson2XmlHttpMessageConverter`与缩进启用，定制的日期格式，和登记 [`jackson-module-parameter-names`](https://github.com/FasterXML/jackson-module-parameter-names)，这增加了用于访问参数名称（在Java中8增加了一个功能）的支持。

该构建器自定义Jackson的默认属性，如下所示：

- [`DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/DeserializationFeature.html#FAIL_ON_UNKNOWN_PROPERTIES) 被禁用。
- [`MapperFeature.DEFAULT_VIEW_INCLUSION`](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/MapperFeature.html#DEFAULT_VIEW_INCLUSION) 被禁用。

如果在类路径中检测到以下知名模块，它将自动注册以下知名模块：

- [jackson-datatype-joda](https://github.com/FasterXML/jackson-datatype-joda)：支持Joda-Time类型。
- [jackson-datatype-jsr310](https://github.com/FasterXML/jackson-datatype-jsr310)：支持Java 8日期和时间API类型。
- [jackson-datatype-jdk8](https://github.com/FasterXML/jackson-datatype-jdk8)：支持其他Java 8类型，例如`Optional`。
- [`jackson-module-kotlin`](https://github.com/FasterXML/jackson-module-kotlin)：支持Kotlin类和数据类。

| ![image-20201202185734173](Spring%20Web%20MVC.assets/image-20201202185734173.png) | 使用Jackson的XML支持启用缩进[`woodstox-core-asl`](https://search.maven.org/#search|gav|1|g%3A"org.codehaus.woodstox" AND a%3A"woodstox-core-asl") 除了需要依赖之外，还需要 依赖[`jackson-dataformat-xml`](https://search.maven.org/#search\|ga\|1\|a%3A"jackson-dataformat-xml")。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

其他有趣的Jackson模块也可用：

- [jackson-datatype-money](https://github.com/zalando/jackson-datatype-money)：支持`javax.money`类型（非官方模块）。
- [jackson-datatype-hibernate](https://github.com/FasterXML/jackson-datatype-hibernate)：支持特定于Hibernate的类型和属性（包括延迟加载方面）。

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper" ref="objectMapper"/>
        </bean>
        <bean class="org.springframework.http.converter.xml.MappingJackson2XmlHttpMessageConverter">
            <property name="objectMapper" ref="xmlMapper"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>

<bean id="objectMapper" class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean"
      p:indentOutput="true"
      p:simpleDateFormat="yyyy-MM-dd"
      p:modulesToInstall="com.fasterxml.jackson.module.paramnames.ParameterNamesModule"/>

<bean id="xmlMapper" parent="objectMapper" p:createXmlMapper="true"/>
```

###  1.11.8. View Controllers 视图控制器

这是用于定义的快捷方式，该快捷方式`ParameterizableViewController`在调用时立即转发到视图。当视图生成响应之前没有Java控制器逻辑要运行时，可以在静态情况下使用它。

以下Java配置示例将请求转发`/`到名为的视图`home`：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

以下示例通过使用`<mvc:view-controller>`元素，实现了与上一示例相同的操作，但使用XML ：

```xml
<mvc:view-controller path="/" view-name="home"/>
```

如果将`@RequestMapping`方法映射到任何HTTP方法的URL，则视图控制器不能用于处理相同的URL。这是因为通过URL与带注释的控制器的匹配被视为端点所有权的足够有力的指示，因此可以将405（METHOD_NOT_ALLOWED），415（UNSUPPORTED_MEDIA_TYPE）或类似的响应发送给客户端，以帮助进行调试。因此，建议避免在带注释的控制器和视图控制器之间拆分URL处理。

###  1.11.9. View Resolvers 视图解析器

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-view-resolvers)

MVC配置简化了视图解析器的注册。

以下Java配置示例通过使用JSP和Jackson作为`View`JSON呈现的默认配置来配置内容协商视图解析：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.jsp();
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:jsp/>
</mvc:view-resolvers>
```

但是请注意，FreeMarker，Tiles，Groovy标记和脚本模板也需要配置基础视图技术。

MVC命名空间提供了专用元素。以下示例适用于FreeMarker：

```xml
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:freemarker cache="false"/>
</mvc:view-resolvers>

<mvc:freemarker-configurer>
    <mvc:template-loader-path location="/freemarker"/>
</mvc:freemarker-configurer>
```

在Java配置中，您可以添加相应的`Configurer`bean，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.freeMarker().cache(false);
    }

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("/freemarker");
        return configurer;
    }
}
```

###  1.11.10. Static Resources 静态资源

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-static-resources)

此选项提供了一种方便的方法来从[`Resource`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/core/io/Resource.html)基于位置的列表中提供**静态资源** 。

在下一个示例中，给定一个以开头的请求，`/resources`相对路径用于相对于`/public`Web应用程序根目录下或之下的类路径查找和提供静态资源`/static`。这些资源的使用期限为一年，以确保最大程度地利用浏览器缓存并减少浏览器发出的HTTP请求。`Last-Modified`从中推导出该信息，`Resource#lastModified` 以便HTTP条件请求支持`"Last-Modified"`标头。

以下清单显示了如何使用Java配置进行操作：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
            .addResourceLocations("/public", "classpath:/static/")
            .setCacheControl(CacheControl.maxAge(Duration.ofDays(365)));
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:resources mapping="/resources/**"
    location="/public, classpath:/static/"
    cache-period="31556926" />
```

另请参见 [HTTP缓存对静态资源的支持](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching-static-resources)。

资源处理程序还支持一系列 [`ResourceResolver`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/resource/ResourceResolver.html)实现和 [`ResourceTransformer`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/resource/ResourceTransformer.html)实现，您可以使用它们来创建用于处理优化资源的工具链。

您可以`VersionResourceResolver`根据从内容，固定应用程序版本或其他版本计算出的MD5哈希值，使用for版本资源URL。一 `ContentVersionStrategy`（MD5哈希）是一个不错的选择-有一些明显的例外，如与模块加载程序使用的JavaScript资源。

以下示例显示了如何`VersionResourceResolver`在Java配置中使用：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/public/")
                .resourceChain(true)
                .addResolver(new VersionResourceResolver().addContentVersionStrategy("/**"));
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:resources mapping="/resources/**" location="/public/">
    <mvc:resource-chain resource-cache="true">
        <mvc:resolvers>
            <mvc:version-resolver>
                <mvc:content-version-strategy patterns="/**"/>
            </mvc:version-resolver>
        </mvc:resolvers>
    </mvc:resource-chain>
</mvc:resources>
```

然后，您可以`ResourceUrlProvider`用来重写URL并应用完整的解析器和转换器链-例如，插入版本。MVC配置提供了一个`ResourceUrlProvider` bean，以便可以将其注入其他对象。您也可以使用`ResourceUrlEncodingFilter`Thymeleaf，JSP，FreeMarker以及其他依赖于URL标签的URL使重写透明 `HttpServletResponse#encodeURL`。

请注意，在同时使用`EncodedResourceResolver`（例如，用于提供压缩或brotli编码的资源）和时`VersionResourceResolver`，必须按此顺序注册它们。这样可以确保始终基于未编码文件可靠地计算基于内容的版本。

`WebJarsResourceResolver`当`org.webjars:webjars-locator-core`类路径中存在库时，也会通过 自动注册 [WebJars](https://www.webjars.org/documentation)。解析程序可以重写URL以包括jar的版本，还可以与没有版本的传入URL进行匹配，例如from`/jquery/jquery.min.js`到 `/jquery/1.2.0/jquery.min.js`。

###  1.11.11. Default Servlet 默认Servlet



Spring MVC允许映射`DispatcherServlet`到`/`（从而覆盖了容器默认Servlet的映射），同时仍然允许容器**默认Servlet处理静态资源请求**。它配置的 `DefaultServletHttpRequestHandler`URL映射为`/**`，相对于其他URL映射具有最低的优先级。

该处理程序将所有请求转发到默认Servlet。因此，它必须按所有其他URL的顺序保留在最后`HandlerMappings`。如果使用，就是这种情况`<mvc:annotation-driven>`。另外，如果你设置你自己定制的`HandlerMapping`情况下，一定要设置其`order`属性的值比的降低`DefaultServletHttpRequestHandler`，这是`Integer.MAX_VALUE`。

下面的示例演示如何使用默认设置启用功能：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:default-servlet-handler/>
```

覆盖`/`Servlet映射的警告是，`RequestDispatcher`必须通过名称而不是通过路径来检索默认Servlet的。在 `DefaultServletHttpRequestHandler`尝试自动检测默认的Servlet在启动时的容器，使用大多数主要的Servlet容器（包括软件Tomcat，Jetty的GlassFish，JBoss和树脂中，WebLogic和WebSphere）已知名称的列表。如果已使用其他名称自定义配置了默认Servlet，或者在默认Servlet名称未知的情况下使用了不同的Servlet容器，则必须显式提供默认Servlet的名称，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable("myCustomDefaultServlet");
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:default-servlet-handler default-servlet-name="myCustomDefaultServlet"/>
```

###  1.11.12. Path Matching 路径匹配

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-path-matching)

您可以自定义与路径匹配和URL处理有关的选项。有关各个选项的详细信息，请参见 [`PathMatchConfigurer`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/config/annotation/PathMatchConfigurer.html)javadoc。

以下示例显示了如何在Java配置中自定义路径匹配：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer
            .setPatternParser(new PathPatternParser())
            .addPathPrefix("/api", HandlerTypePredicate.forAnnotation(RestController.class));
    }

    private PathPatternParser patternParser() {
        // ...
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:annotation-driven>
    <mvc:path-matching
        trailing-slash="false"
        path-helper="pathHelper"
        path-matcher="pathMatcher"/>
</mvc:annotation-driven>

<bean id="pathHelper" class="org.example.app.MyPathHelper"/>
<bean id="pathMatcher" class="org.example.app.MyPathMatcher"/>
```

###  1.11.13. Advanced Java Config 高级Java配置

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-advanced-java)

`@EnableWebMvc`进口`DelegatingWebMvcConfiguration`，其中：

- 为Spring MVC应用程序提供默认的Spring配置
- 检测并委托给`WebMvcConfigurer`实现以自定义该配置。

对于高级模式，您可以`@EnableWebMvc`直接从中删除和扩展 `DelegatingWebMvcConfiguration`而不是实现`WebMvcConfigurer`，如以下示例所示：

```java
@Configuration
public class WebConfig extends DelegatingWebMvcConfiguration {

    // ...
}
```

您可以将现有方法保留在中`WebConfig`，但是现在您还可以==覆盖基类中的bean声明==，并且`WebMvcConfigurer`在类路径上仍然可以具有许多其他实现。

### 1.11.14. Advanced XML Config 高级XML配置

MVC命名空间没有高级模式。如果您需要在bean上自定义一个不能更改的属性，则可以使用`BeanPostProcessor`Spring的生命周期挂钩`ApplicationContext`，如以下示例所示：

```java
@Component
public class MyPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object bean, String name) throws BeansException {
        // ...
    }
}
```

请注意，您需要以`MyPostProcessor`XML形式显式声明或通过`<component-scan/>`声明使其被检测为bean 。

##  1.12. HTTP/2

[WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-http2)

需要Servlet 4容器支持HTTP / 2，并且Spring Framework 5与Servlet API 4兼容。从**编程模型**的角度来看，应用程序不需要做任何特定的事情。但是，有一些与服务器配置有关的注意事项。有关更多详细信息，请参见 [HTTP / 2 Wiki页面](https://github.com/spring-projects/spring-framework/wiki/HTTP-2-support)。

Servlet API确实公开了一种==与HTTP / 2相关==的构造。您可以使用 `javax.servlet.http.PushBuilder`来将资源**主动**推送到客户端，并且它作为[方法的方法参数](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)而受支持`@RequestMapping`。



 