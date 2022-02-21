# 4.7 开发Web应用

Spring Boot同样适合开发Web应用。您可以使用嵌入式Tomcat，Jetty，Undertow或Netty创建独立的HTTP服务器，大多数Web应用程序都使用spring-boot-starter-web模块来快速启动和运行。您还可以选择使用spring-boot-starter-webflux模块构建反应式Web应用程序。

如果尚未开发Spring Boot Web应用程序，则可以遵循入门章节中的“ Hello World！”的示例。

## 4.7.1  Spring Web Mvc框架

Spring Web MVC框架（通常简称为“ Spring MVC”）是一个丰富的“模型视图控制器” Web框架。 Spring MVC使您可以创建特殊的@Controller或@RestController Bean来处理传入的HTTP请求。 使用@RequestMapping注解将控制器中的方法映射到HTTP。

以下代码显示了提供JSON数据的典型@RestController：
```
@RestController
@RequestMapping(value="/users")
public class MyRestController {

    @RequestMapping(value="/{user}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
    List<Customer> getUserCustomers(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}", method=RequestMethod.DELETE)
    public User deleteUser(@PathVariable Long user) {
        // ...
    }

}
```
Spring MVC是核心Spring Framework的一部分，有关详细信息，请参阅[参考文档](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc)。 在[spring.io/guides](https://spring.io/guides)上还有一些涵盖Spring MVC的指南。

**Spring MVC自动配置**

Spring Boot为Spring MVC提供了自动配置，可与大多数应用程序完美配合。

自动配置在Spring的默认设置之上添加了以下功能：
- 包含ContentNegotiatingViewResolver和BeanNameViewResolver Bean。
- 支持提供静态资源，包括对WebJars的支持（在本文档的后面部分有介绍）。
- 自动注册Converter，GenericConverter和Formatter Bean。
- 对HttpMessageConverters的支持（在本文档后面介绍）。
- 自动注册MessageCodesResolver（在本文档后面介绍）。
- 静态index.html支持。
- 自定义Favicon支持（在本文档后面介绍）。
- 自动使用ConfigurableWebBindingInitializer Bean（在本文档后面介绍）。

如果要保留这些Spring Boot MVC定制并进行更多的MVC定制（拦截器，格式化程序，视图控制器和其他功能），则可以添加自己的类型为WebMvcConfigurer的@Configuration类，但不要使用@EnableWebMvc。

如果要提供RequestMappingHandlerMapping，RequestMappingHandlerAdapter或ExceptionHandlerExceptionResolver的自定义实例，并且仍然保留Spring Boot MVC自定义，则可以声明WebMvcRegistrations类型的bean，并使用它提供这些组件的自定义实例。

如果要完全控制Spring MVC，则可以添加用@EnableWebMvc注释的自己的@Configuration，或者按照@EnableWebMvc的Javadoc中的说明添加自己的@Configuration注释的DelegatingWebMvcConfiguration。

**HttpMessageConverters**

Spring MVC使用HttpMessageConverter接口转换HTTP请求和响应。 开箱即用中包含智能的默认设置。 例如，可以将对象自动转换为JSON（通过使用Jackson库）或XML（通过使用Jackson XML扩展，如果Jackson XML扩展可用，或者通过使用JAXB，如果Jackson XML扩展不可用）。 默认情况下，字符串以UTF-8编码。

如果您需要添加或自定义转换器，则可以使用Spring Boot的HttpMessageConverters类，如以下清单所示：
```
import org.springframework.boot.autoconfigure.http.HttpMessageConverters;
import org.springframework.context.annotation.*;
import org.springframework.http.converter.*;

@Configuration(proxyBeanMethods = false)
public class MyConfiguration {

    @Bean
    public HttpMessageConverters customConverters() {
        HttpMessageConverter<?> additional = ...
        HttpMessageConverter<?> another = ...
        return new HttpMessageConverters(additional, another);
    }

}
```

上下文中存在的所有HttpMessageConverter bean都将添加到转换器列表中。 您也可以用相同的方法覆盖默认转换器。

**自定义JSON序列化器和反序列化器**

如果使用Jackson序列化和反序列化JSON数据，则可能要编写自己的JsonSerializer和JsonDeserializer类。 自定义序列化程序通常是[通过模块向Jackson进行注册](https://github.com/FasterXML/jackson-docs/wiki/JacksonHowToCustomSerializers)的，但是Spring Boot提供了一种替代性的@JsonComponent批注，这使得直接注册Spring Bean更加容易。

您可以直接在JsonSerializer，JsonDeserializer或KeyDeserializer实现上使用@JsonComponent注解。 您还可以在包含序列化器/反序列化器作为内部类的类上使用它，如以下示例所示：
```
import java.io.*;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.*;
import org.springframework.boot.jackson.*;

@JsonComponent
public class Example {

    public static class Serializer extends JsonSerializer<SomeObject> {
        // ...
    }

    public static class Deserializer extends JsonDeserializer<SomeObject> {
        // ...
    }

}
```

ApplicationContext中的所有@JsonComponent bean都会自动向Jackson注册。 因为@JsonComponent用@Component进行元注释，所以适用通常的组件扫描规则。

Spring Boot还提供了JsonObjectSerializer和JsonObjectDeserializer基类，这些基类在序列化对象时为标准Jackson版本提供了有用的替代方法。 有关详细信息，请参见Javadoc中的JsonObjectSerializer和JsonObjectDeserializer。

**MessageCodesResolver**

Spring MVC有一个生成错误代码以从绑定错误中呈现错误消息的策略：MessageCodesResolver。 如果设置spring.mvc.message-codes-resolver-format属性PREFIX_ERROR_CODE或POSTFIX_ERROR_CODE，Spring Boot会为您创建一个（请参见DefaultMessageCodesResolver.Format中的枚举）。

**静态内容**

默认情况下，Spring Boot从类路径中的/ static目录（或/ public或/ resources或/ META-INF / resources）或ServletContext的根目录中提供静态内容。 它使用Spring MVC中的ResourceHttpRequestHandler，以便您可以通过添加自己的WebMvcConfigurer并覆盖addResourceHandlers方法来修改该行为。

在独立的Web应用程序中，还启用了容器中的默认Servlet，并将其用作后备，如果Spring决定不处理，则从ServletContext的根目录提供内容。 在大多数情况下，这不会发生（除非您修改默认的MVC配置），因为Spring始终可以通过DispatcherServlet处理请求。

默认情况下，资源映射在/\**上，但是您可以使用spring.mvc.static-path-pattern属性进行调整。 例如，将所有资源重定位到/ resources /\**可以实现如下：
```
spring.mvc.static-path-pattern=/resources/**
```
您还可以使用spring.resources.static-locations属性来自定义静态资源位置（用目录位置列表替换默认值）。 根Servlet上下文路径“ /”也会自动添加为位置。

除了前面提到的“标准”静态资源位置，Webjar内容也有特殊情况。 如果jar文件以Webjars格式打包，则从jar文件提供带有/ webjars /\**路径的所有资源。

>如果您的应用程序打包为jar，则不要使用src / main / webapp目录。 尽管此目录是一个通用标准，但它仅与war打包一起使用，并且如果生成jar，大多数构建工具都将其忽略。

Spring Boot还支持Spring MVC提供的高级资源处理功能，允许使用案例，例如缓存清除静态资源或对Webjars使用版本无关的URL。

要对Webjars使用版本无关的URL，请添加webjars-locator-core依赖项。 然后声明您的Webjar。 以jQuery为例，添加“ /webjars/jquery/jquery.min.js”将得到“ /webjars/jquery/x.y.z/jquery.min.js”，其中x.y.z是Webjar版本。

>如果使用JBoss，则需要声明webjars-locator-jboss-vfs依赖关系，而不是webjars-locator-core。 否则，所有Webjar都解析为404。

要使用缓存清除，以下配置为所有静态资源配置了缓存清除解决方案，从而有效地在URL中添加了内容哈希，例如<link href =“ /css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css”/>，
```
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
```

>借助为Thymeleaf和FreeMarker自动配置的ResourceUrlEncodingFilter，可以在运行时在模板中重写资源链接。 使用JSP时，您应该手动声明此过滤器。 当前不自动支持其他模板引擎，但可以与自定义模板宏/帮助器一起使用，以及使用ResourceUrlProvider。

例如，当使用JavaScript模块加载器动态加载资源时，不能重命名文件。 这就是为什么其他策略也受支持并且可以组合的原因。 “固定”策略在URL中添加静态版本字符串，而不更改文件名，如以下示例所示：
```
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
spring.resources.chain.strategy.fixed.enabled=true
spring.resources.chain.strategy.fixed.paths=/js/lib/
spring.resources.chain.strategy.fixed.version=v12
```

通过这种配置，位于“ /js/lib/”下的JavaScript模块使用固定的版本控制策略（“/v12/js/lib/mymodule.js”），而其他资源仍使用内容版本（<link href =“/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css“ />）。

有关更多受支持的选项，请参见[ResourceProperties](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ResourceProperties.java)。

**Welcom Page**

Spring Boot支持静态和模板欢迎页面。 它首先在配置的静态内容位置中查找index.html文件。 如果未找到，则寻找索引模板。 如果找到任何一个，它将自动用作应用程序的欢迎页面。

**自定义图标**

与其他静态资源一样，Spring Boot在已配置的静态内容位置中查找favicon.ico。 如果存在这样的文件，它将自动用作应用程序的图标。

**路径匹配和内容协商**

Spring MVC可以通过查看请求路径并将其匹配到应用程序中定义的映射（例如，Controller方法上的@GetMapping批注）来将传入的HTTP请求映射到处理程序。

Spring Boot默认选择禁用后缀模式匹配，这意味着“ GET /projects/spring-boot.json”之类的请求将不会与@GetMapping（“ /projects/spring-boot”）映射进行匹配。这被认为是[Spring MVC应用程序的最佳实践](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc-ann-requestmapping-suffix-pattern-match)。过去，此功能主要用于未发送正确的“ Accept”请求标头的HTTP客户端。 我们需要确保将正确的内容类型发送给客户端。 如今，内容协商已变得更加可靠。

还有其他处理HTTP客户端的方法，这些客户端不能始终发送正确的“ Accept”请求标头。除了使用后缀匹配，我们还可以使用查询参数来确保将诸如“ GET / projects / spring-boot？format = json”之类的请求映射到@GetMapping（“/projects/spring-boot”）：
```
spring.mvc.contentnegotiation.favor-parameter=true

# We can change the parameter name, which is "format" by default:
# spring.mvc.contentnegotiation.parameter-name=myparam

# We can also register additional file extensions/media types with:
spring.mvc.contentnegotiation.media-types.markdown=text/markdown
```
后缀模式匹配已被弃用，并将在以后的版本中删除。 如果您了解了注意事项，但仍希望您的应用程序使用后缀模式匹配，则需要以下配置：
```
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-suffix-pattern=true
```

另外，与其打开所有后缀模式，不如只支持已注册的后缀模式，这更安全：
```
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-registered-suffix-pattern=true

# You can also register additional file extensions/media types with:
# spring.mvc.contentnegotiation.media-types.adoc=text/asciidoc
```

**ConfigurableWebBindingInitializer**

Spring MVC使用WebBindingInitializer初始化特定请求的WebDataBinder。 如果创建自己的ConfigurableWebBindingInitializer @Bean，Spring Boot会自动配置Spring MVC以使用它。

**模板引擎**

除了REST Web服务之外，您还可以使用Spring MVC来提供动态HTML内容。 Spring MVC支持各种模板技术，包括Thymeleaf，FreeMarker和JSP。 同样，许多其他模板引擎包括它们自己的Spring MVC集成。

Spring Boot包括对以下模板引擎的自动配置支持：
- [FreeMarker](https://freemarker.apache.org/docs/)
- [Groovy](http://docs.groovy-lang.org/docs/next/html/documentation/template-engines.html#_the_markuptemplateengine)
- [Thymeleaf](https://www.thymeleaf.org/)
- [Mustache](https://mustache.github.io/)

>如果可能，应避免使用JSP。 将它们与嵌入式servlet容器一起使用时，存在几个已知的限制。

在默认配置下使用这些模板引擎之一时，将从src main/resources/templates中自动提取模板。

>根据您运行应用程序的方式，IntelliJ IDEA对类路径的排序方式不同。 与使用Maven或Gradle或从打包的jar运行应用程序时，从IDE的主要方法运行应用程序的顺序会有所不同。 这可能会导致Spring Boot无法在类路径上找到模板。 如果遇到此问题，可以在IDE中重新排序类路径，以首先放置模块的类和资源。 或者，您可以配置模板前缀以搜索类路径上的每个template目录，如下所示：classpath \*：/templates/。

**错误处理**

默认情况下，Spring Boot提供了一个/ error映射，以一种明智的方式处理所有错误，并且在servlet容器中被注册为“全局”错误页面。 对于机器客户端，它将生成JSON响应，其中包含错误，HTTP状态和异常消息的详细信息。 对于浏览器客户端，有一个“ whitelabel”错误视图以HTML格式呈现相同的数据（要对其进行自定义，请添加一个可解决错误的视图）。 要完全替换默认行为，可以实现ErrorController并注册该类型的bean定义，或者添加类型为ErrorAttributes的bean以使用现有机制，但替换其内容。

>BasicErrorController可用作自定义ErrorController的基类。 如果要为新的内容类型添加处理程序（默认是专门处理text / html并为其他所有内容提供后备功能），则此功能特别有用。 为此，请扩展BasicErrorController，添加具有Produces属性的@RequestMapping的公共方法，并创建新类型的bean。

您还可以定义一个带有@ControllerAdvice注释的类，以自定义JSON文档以针对特定的控制器和/或异常类型返回，如以下示例所示：
```
@ControllerAdvice(basePackageClasses = AcmeController.class)
public class AcmeControllerAdvice extends ResponseEntityExceptionHandler {

    @ExceptionHandler(YourException.class)
    @ResponseBody
    ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
        HttpStatus status = getStatus(request);
        return new ResponseEntity<>(new CustomErrorType(status.value(), ex.getMessage()), status);
    }

    private HttpStatus getStatus(HttpServletRequest request) {
        Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
        if (statusCode == null) {
            return HttpStatus.INTERNAL_SERVER_ERROR;
        }
        return HttpStatus.valueOf(statusCode);
    }

}
```
在前面的示例中，如果与AcmeController在同一包中定义的控制器抛出YourException，则将使用CustomErrorType POJO的JSON表示形式而不是ErrorAttributes表示形式。

**自定义错误页面**

如果要显示给定状态代码的自定义HTML错误页面，可以将文件添加到/ error文件夹。 错误页面可以是静态HTML（即添加到任何静态资源文件夹下），也可以使用模板来构建。 文件名应为确切的状态代码或系列掩码。

例如，要将404映射到静态HTML文件，您的文件夹结构如下：
```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- public/
             +- error/
             |   +- 404.html
             +- <other public assets>
```
要使用FreeMarker模板映射所有5xx错误，您的文件夹结构如下：
```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.ftlh
             +- <other templates>
```

对于更复杂的映射，还可以添加实现ErrorViewResolver接口的bean，如以下示例所示：
```
public class MyErrorViewResolver implements ErrorViewResolver {

    @Override
    public ModelAndView resolveErrorView(HttpServletRequest request,
            HttpStatus status, Map<String, Object> model) {
        // Use the request or status to optionally return a ModelAndView
        return ...
    }

}
```
您还可以使用常规的Spring MVC功能，例如@ExceptionHandler方法和@ControllerAdvice。 然后，ErrorController拾取所有未处理的异常。

**在Spring MVC外部映射错误页面**

对于不使用Spring MVC的应用程序，可以使用ErrorPageRegistrar接口直接注册ErrorPages。 此抽象直接与基础嵌入式servlet容器一起使用，即使您没有Spring MVC DispatcherServlet，它也可以使用。
```
@Bean
public ErrorPageRegistrar errorPageRegistrar(){
    return new MyErrorPageRegistrar();
}

// ...

private static class MyErrorPageRegistrar implements ErrorPageRegistrar {

    @Override
    public void registerErrorPages(ErrorPageRegistry registry) {
        registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
    }

}
```
如果您在错误页面上注册了一个最终由过滤器处理的路径（这在某些非Spring Web框架（如Jersey和Wicket）中很常见），则必须将过滤器显式注册为ERROR调度程序，如 下面的例子：
```
@Bean
public FilterRegistrationBean myFilter() {
    FilterRegistrationBean registration = new FilterRegistrationBean();
    registration.setFilter(new MyFilter());
    ...
    registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
    return registration;
}
```
请注意，默认的FilterRegistrationBean不包含ERROR调度程序类型。

注意：当Spring Boot部署到servlet容器时，将使用其错误页面过滤器将具有错误状态的请求转发到相应的错误页面。 如果尚未提交响应，则只能将请求转发到正确的错误页面。 缺省情况下，WebSphere Application Server 8.0及更高版本在成功完成servlet的服务方法后提交响应。 您应该通过将com.ibm.ws.webcontainer.invokeFlushAfterService设置为false来禁用此行为。

**Spring HATEOAS**

如果您开发使用超媒体的RESTful API，Spring Boot会为Spring HATEOAS提供自动配置，该配置可与大多数应用程序很好地兼容。 自动配置取代了使用@EnableHypermediaSupport的需要，并注册了许多bean来简化基于超媒体的应用程序的构建，包括一个LinkDiscoverers（用于客户端支持）和一个ObjectMapper，该对象被配置为将响应正确地解析为所需的表示形式。 通过设置各种spring.jackson.\*属性，或通过Jackson2ObjectMapperBuilder bean（如果存在）来定制ObjectMapper。

您可以使用@EnableHypermediaSupport来控制Spring HATEOAS的配置。 请注意，这样做会禁用前面所述的ObjectMapper定制。

**CORS Support**

跨域资源共享（CORS）是由大多数浏览器实施的W3C规范，使您可以灵活地指定授权哪种类型的跨域请求，而不是使用一些安全性和功能不强的方法（例如IFRAME或JSONP） 。

从4.2版本开始，Spring MVC支持CORS。 在Spring Boot应用程序中使用带有@CrossOrigin注解的控制器方法CORS不需要任何特定的配置。 可以通过使用自定义的addCorsMappings（CorsRegistry）方法注册WebMvcConfigurer bean来定义全局CORS配置，如以下示例所示：
```
@Configuration(proxyBeanMethods = false)
public class MyConfiguration {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**");
            }
        };
    }
}
```

# 4.7.2 “ Spring WebFlux框架”

Spring WebFlux是Spring Framework 5.0中引入的新的响应式Web框架。 与Spring MVC不同，它不需要Servlet API，是完全异步且无阻塞的，并通过Reactor项目实现Reactive Streams规范。

Spring WebFlux有两种形式：功能性的和基于注释的。 基于注释的模型非常类似于Spring MVC模型，如以下示例所示：
```
@RestController
@RequestMapping("/users")
public class MyRestController {

    @GetMapping("/{user}")
    public Mono<User> getUser(@PathVariable Long user) {
        // ...
    }

    @GetMapping("/{user}/customers")
    public Flux<Customer> getUserCustomers(@PathVariable Long user) {
        // ...
    }

    @DeleteMapping("/{user}")
    public Mono<User> deleteUser(@PathVariable Long user) {
        // ...
    }

}
```

功能变体“ WebFlux.fn”将路由配置与请求的实际处理分开，如以下示例所示：
```
@Configuration(proxyBeanMethods = false)
public class RoutingConfiguration {

    @Bean
    public RouterFunction<ServerResponse> monoRouterFunction(UserHandler userHandler) {
        return route(GET("/{user}").and(accept(APPLICATION_JSON)), userHandler::getUser)
                .andRoute(GET("/{user}/customers").and(accept(APPLICATION_JSON)), userHandler::getUserCustomers)
                .andRoute(DELETE("/{user}").and(accept(APPLICATION_JSON)), userHandler::deleteUser);
    }

}

@Component
public class UserHandler {

    public Mono<ServerResponse> getUser(ServerRequest request) {
        // ...
    }

    public Mono<ServerResponse> getUserCustomers(ServerRequest request) {
        // ...
    }

    public Mono<ServerResponse> deleteUser(ServerRequest request) {
        // ...
    }
}
```
WebFlux是Spring Framework的一部分，其[参考文档](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web-reactive.html#webflux-fn)中提供了详细信息。

>您可以根据需要定义任意数量的RouterFunction bean，以对路由器的定义进行模块化。 如果需要应用优先级，可以排序Bean。

首先，将spring-boot-starter-webflux模块添加到您的应用程序。
>在应用程序中添加spring-boot-starter-web和spring-boot-starter-webflux模块会导致Spring Boot自动配置Spring MVC，而不是WebFlux。 之所以选择这种行为，是因为许多Spring开发人员将spring-boot-starter-webflux添加到其Spring MVC应用程序中以使用反应式WebClient。 您仍然可以通过将所选应用程序类型设置为SpringApplication.setWebApplicationType（WebApplicationType.REACTIVE）来强制执行选择。

**Spring WebFlus自动配置**

Spring Boot为Spring WebFlux提供了自动配置，可与大多数应用程序完美配合。

自动配置在Spring的默认设置之上添加了以下功能：
- 为HttpMessageReader和HttpMessageWriter实例配置编解码器（在本文档后面介绍）。
- 支持服务静态资源，包括对WebJars的支持（在本文档的后面部分中介绍）。

如果您想保留Spring Boot WebFlux功能并想要添加其他WebFlux配置，则可以添加自己的类型为WebFluxConfigurer的@Configuration类，但不添加@EnableWebFlux。

如果要完全控制Spring WebFlux，则可以添加带有@EnableWebFlux注释的自己的@Configuration。

**带有HttpMessageReaders和HttpMessageWriters的HTTP编解码器**

Spring WebFlux使用HttpMessageReader和HttpMessageWriter来转换HTTP请求和响应，通过查看类路径中可用的库，使用CodecConfigurer将它们配置为具有合理的默认值。

Spring Boot为编解码器spring.codec.\*提供了专用的配置属性。 它还通过使用CodecCustomizer实例应用进一步的自定义。 例如，将spring.jackson.\*配置密钥应用于Jackson编解码器。

如果需要添加或自定义编解码器，则可以创建一个自定义CodecCustomizer组件，如以下示例所示：
```
import org.springframework.boot.web.codec.CodecCustomizer;

@Configuration(proxyBeanMethods = false)
public class MyConfiguration {

    @Bean
    public CodecCustomizer myCodecCustomizer() {
        return codecConfigurer -> {
            // ...
        };
    }

}
```
您还可以利用Boot的自定义JSON序列化器和反序列化器。

**静态内容**

默认情况下，Spring Boot从类路径中名为/ static（或/public或/resources或/META-INF/resources）的目录中提供静态内容。 它使用Spring WebFlux中的ResourceWebHandler，所以你可以通过添加自己的WebFluxConfigurer并覆盖addResourceHandlers方法来修改该行为。

默认情况下，资源映射在/\**上，但是您可以通过设置spring.webflux.static-path-pattern属性来对其进行调整。 例如，将所有资源重定位到/0resources/\**可以实现如下：
```
spring.webflux.static-path-pattern=/resources/**
```

您还可以使用spring.resources.static-locations自定义静态资源位置。 这样做会将默认值替换为目录位置列表。 如果这样做，默认的欢迎页面检测将切换到您的自定义位置。 因此，如果启动时您的任何位置都有index.html，则它是应用程序的主页。

除了前面列出的“标准”静态资源位置之外，Webjar内容也有特殊情况。 如果jar文件以Webjars格式打包，则从jar文件提供带有/webjars/\**路径的所有资源。

>Spring WebFlux应用程序不严格依赖Servlet API，因此不能将它们部署为war文件，也不使用src / main / webapp目录。

**模板引擎**

除了REST Web服务之外，您还可以使用Spring WebFlux来提供动态HTML内容。 Spring WebFlux支持多种模板技术，包括Thymeleaf，FreeMarker和Mustache。

Spring Boot包括对以下模板引擎的自动配置支持：
- [FreeMarker](https://freemarker.apache.org/docs/)
- [Thymeleaf](https://www.thymeleaf.org/)
- [Mustache](https://mustache.github.io/)

在默认配置下使用这些模板引擎之一时，将从src/main/resources/templates中自动提取模板。

**错误处理**

Spring Boot提供了一个WebExceptionHandler，以一种合理的方式处理所有错误。 它在处理顺序中的位置紧靠WebFlux提供的处理程序之前，该处理程序被认为是最后一个。 对于机器客户端，它将生成JSON响应，其中包含错误，HTTP状态和异常消息的详细信息。 对于浏览器客户端，有一个“ whitelabel”错误处理程序，以HTML格式呈现相同的数据。 您还可以提供自己的HTML模板来显示错误（请参阅下一节）。

定制此功能的第一步通常涉及使用现有机制，但替换或增加错误内容。 为此，您可以添加类型为ErrorAttributes的bean。

要更改错误处理行为，可以实现ErrorWebExceptionHandler并注册该类型的bean定义。 由于WebExceptionHandler的级别很低，因此Spring Boot还提供了一个方便的AbstractErrorWebExceptionHandler，可让您以WebFlux功能方式处理错误，如以下示例所示：
```
public class CustomErrorWebExceptionHandler extends AbstractErrorWebExceptionHandler {

    // Define constructor here

    @Override
    protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {

        return RouterFunctions
                .route(aPredicate, aHandler)
                .andRoute(anotherPredicate, anotherHandler);
    }

}
```

为了获得更完整的描述，您还可以直接将DefaultErrorWebExceptionHandler子类化并重写特定方法。

**自定义错误页**

如果要显示给定状态代码的自定义HTML错误页面，可以将文件添加到/ error文件夹。 错误页面可以是静态HTML（即添加到任何静态资源文件夹下），也可以使用模板构建。 文件名应为确切的状态代码或系列掩码。

例如，要将404映射到静态HTML文件，您的文件夹结构如下：
```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- public/
             +- error/
             |   +- 404.html
             +- <other public assets>
```

要使用Mustache模板映射所有5xx错误，您的文件夹结构如下：
```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.mustache
             +- <other templates>
```
**Web过滤器**

Spring WebFlux提供了一个WebFilter接口，可以实现该接口来过滤HTTP请求-响应交换。 在应用程序上下文中找到的WebFilter bean将自动用于过滤每个交换。

如果过滤器的顺序很重要，则可以实现Ordered或使用@Order进行注释。 Spring Boot自动配置可能会为您配置Web过滤器。 这样做时，将使用下表中显示的顺序：
Web Filter|Order
:-|:-
MEtricsWebFilter|Ordered.HIGHEST_PRECEDENCE + 1
WebFilterChainProxy (Spring Security)|-100
HttpTraceWebFilter|Ordered.LOWEST_PRECEDENCE - 10

# 4.7.3 JAX-RS and Jersey

如果您更喜欢REST端点使用JAX-RS编程模型，则可以使用可用的实现之一来代替Spring MVC。 Jersey和Apache CXF开箱即用。 CXF要求您在应用程序上下文中将其Servlet或Filter注册为@Bean。 Jersey有一些本机Spring支持，因此我们在Spring Boot中还与启动器一起为其提供了自动配置支持。

要开始使用Jersey，请将spring-boot-starter-jersey作为依赖项包括在内，然后需要一个ResourceConfig类型的@Bean，在其中注册所有端点，如以下示例所示：
```
@Component
public class JerseyConfig extends ResourceConfig {

    public JerseyConfig() {
        register(Endpoint.class);
    }

}
```
Jersey对扫描可执行存档的支持非常有限。 例如，在运行可执行的war文件时，它无法扫描在完全可执行的jar文件或WEB-INF / classs中找到的包中的端点。 为了避免这种限制，不应该使用packages方法，并且应该使用register方法分别注册端点，如前面的示例所示。

对于更多高级的自定义，您还可以注册任意数量的实现了ResourceConfigCustomizer的bean。

所有注册的端点都应该是带有HTTP资源注释（@GET和其他注释）的@Components，如以下示例所示：
```
@Component
@Path("/hello")
public class Endpoint {

    @GET
    public String message() {
        return "Hello";
    }

}
```
由于端点是Spring @Component，因此其生命周期由Spring管理，您可以使用@Autowired注入依赖项，并使用@Value注入外部配置。 默认情况下，Jersey servlet被注册并映射到/\*。 您可以通过将@ApplicationPath添加到ResourceConfig来更改映射。

默认情况下，Jersey被设置为名为jerseyServletRegistration的ServletRegistrationBean类型的@Bean中的Servlet。 默认情况下，该servlet延迟初始化，但是您可以通过设置spring.jersey.servlet.load-on-startup来自定义该行为。 您可以通过使用相同的名称创建自己的一个来禁用或覆盖该bean。 您还可以通过设置spring.jersey.type = filter（在这种情况下，要替换或覆盖的@Bean是jerseyFilterRegistration）来使用过滤器而不是servlet。 过滤器具有@Order，您可以使用spring.jersey.filter.order进行设置。 可以通过使用spring.jersey.init.\*来指定属性映射，从而为servlet和过滤器注册都赋予init参数。

# 4.7.4 内嵌Servlet容器支持

Spring Boot包括对嵌入式Tomcat，Jetty和Undertow服务器的支持。 大多数开发人员使用适当的“启动器”来获取完全配置的实例。 默认情况下，嵌入式服务器在端口8080上侦听HTTP请求。

**Servlets,过滤器和监听器**

使用嵌入式Servlet容器时，可以通过使用Spring Bean或扫描Servlet组件来注册Servlet规范中的Servlet，过滤器和所有侦听器（例如HttpSessionListener）。

**将Servlets,过滤器和监听器注册为Bean**

任何作为Spring Bean的Servlet，Filter或Servlet \*Listener实例都向嵌入式容器注册。 如果要在配置过程中引用application.properties中的值，这可能特别方便。

默认情况下，如果上下文仅包含单个Servlet，则将其映射到/。 如果有多个servlet bean，则将bean名称用作路径前缀。 过滤器映射到/\*。

如果基于约定的映射不够灵活，则可以使用ServletRegistrationBean，FilterRegistrationBean和ServletListenerRegistrationBean类进行完全控制。

通常可以安全的将Filter bean处于有序状态。 如果需要特定的顺序，则应使用@Order注释Filter或使其实现Ordered。 您不能通过使用@Order注释Filter bean中的方法来配置它的顺序。 如果您不能更改Filter类以添加@Order或实现Ordered，则必须为Filter定义一个FilterRegistrationBean并使用setOrder（int）方法设置注册bean的顺序。 避免配置一个以Ordered.HIGHEST_PRECEDENCE顺序读取请求正文的过滤器，因为它可能与应用程序的字符编码配置不符。 如果Servlet过滤器包装了请求，则应使用小于或等于OrderedFilter.REQUEST_WRAPPER_FILTER_MAX_ORDER的顺序来配置它。

>要查看应用程序中每个Filter的顺序，请为Web日志记录组（logging.level.web = debug）启用调试级别的日志记录。 然后，将在启动时记录已注册过滤器的详细信息，包括其顺序和URL模式。

>注册Filter Bean时要小心，因为它们是在应用程序生命周期中很早就初始化的。 如果需要注册与其他bean交互的Filter，请考虑改用DelegatingFilterProxyRegistrationBean。

**Servlet上下文初始化**

嵌入式Servlet容器不会直接执行Servlet 3.0+ javax.servlet.ServletContainerInitializer接口或Spring的org.springframework.web.WebApplicationInitializer接口。 这是一个有意的设计决定，旨在降低旨在在war中运行的第三方库可能破坏Spring Boot应用程序的风险。

如果需要在Spring Boot应用程序中执行servlet上下文初始化，则应该注册一个实现org.springframework.boot.web.servlet.ServletContextInitializer接口的bean。 单个onStartup方法提供对ServletContext的访问，并且在必要时可以轻松地用作现有WebApplicationInitializer的适配器。

**扫描Servlets,过滤器和监听器**

使用嵌入式容器时，可以通过使用@ServletComponentScan来自动注册带有@ WebServlet，@ WebFilter和@WebListener的类。

>@ServletComponentScan在独立容器中无效，而是使用容器的内置发现机制。

**The ServletWebServerApplicationContext**

在后台，Spring Boot使用另一种类型的ApplicationContext来支持嵌入式Servlet容器。 ServletWebServerApplicationContext是WebApplicationContext的一种特殊类型，它通过搜索单个ServletWebServerFactory bean来自我引导。 通常，已经自动配置了TomcatServletWebServerFactory，JettyServletWebServerFactory或UndertowServletWebServerFactory。

**自定义内嵌Servlet容器**

可以使用Spring Environment属性来配置常见的servlet容器设置。 通常，您将在application.properties文件中定义属性。

常用服务器设置包括：
- 网络设置：Http请求的监听端口（server.port）,绑定到server.address的接口地址等等。
- Session设置： Session是否持续（server.servlet.session.persistent）,session超时时间（server.servlet.session.timeout）,session数据的位置（sever.servlet.session.store-dir）,session的cookie配置（sever.servlet.session.cookie.\*）。
- 错误管理：错误页的位置（server.error.path）等等。
- [SSL](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#howto-configure-ssl)
- [HTTP compression](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#how-to-enable-http-response-compression)


 Spring Boot尝试尽可能多地公开通用设置，但这并不总是可能的。 对于这些情况，专用名称空间提供了特定于服务器的自定义项（请参阅server.tomcat和server.undertow）。 例如，可以使用嵌入式servlet容器的特定功能配置访问日志。

 >	See the [ServerProperties](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java) class for a complete list.

 **程序化定制**

 如果需要以编程方式配置嵌入式Servlet容器，则可以注册一个实现WebServerFactoryCustomizer接口的Spring Bean。 WebServerFactoryCustomizer提供对ConfigurableServletWebServerFactory的访问，其中包括许多自定义设置方法。 以下示例显示以编程方式设置端口：
```
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.stereotype.Component;

@Component
public class CustomizationBean implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

    @Override
    public void customize(ConfigurableServletWebServerFactory server) {
        server.setPort(9000);
    }

}
```
 >TomcatServletWebServerFactory，JettyServletWebServerFactory和UndertowServletWebServerFactory是ConfigurableServletWebServerFactory的专用变体，分别具有针对Tomcat，Jetty和Undertow的其他自定义设置方法。

 **直接自定义ConfigurableServletWebServerFactory**

 如果上述定制技术太有限，则可以自己注册TomcatServletWebServerFactory，JettyServletWebServerFactory或UndertowServletWebServerFactory bean。

```
 @Bean
public ConfigurableServletWebServerFactory webServerFactory() {
    TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
    factory.setPort(9000);
    factory.setSessionTimeout(10, TimeUnit.MINUTES);
    factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html"));
    return factory;
}
```

提供了许多配置选项的设置器。 如果您需要做一些更奇特的操作，还提供了几种受保护的方法“挂钩”。 有关详细信息，请参见源代码文档。

**JSP局限性**

当运行使用嵌入式servlet容器（并打包为可执行归档）的Spring Boot应用程序时，JSP支持存在一些限制。
- 对于Jetty和Tomcat，如果使用war包装，它应该可以工作。 与java -jar一起启动时，可执行的war将起作用，并且还将可部署到任何标准容器中。 使用可执行jar时，不支持JSP。
- Undertow不支持JSP。
- 创建定制的error.jsp页面不会覆盖默认视图以进行错误处理。 应改用自定义错误页面。

## 4.7.5 嵌入式响应式服务器支持

Spring Boot包含对以下嵌入式反应式Web服务器的支持：Reactor Netty，Tomcat，Jetty和Undertow。 大多数开发人员使用适当的“启动器”来获取完全配置的实例。 默认情况下，嵌入式服务器在端口8080上侦听HTTP请求。

## 4.7.6 响应式服务器资源配置

当自动配置Reactor Netty或Jetty服务器时，Spring Boot将创建特定的bean，这些bean将向服务器实例：ReactorResourceFactory或JettyResourceFactory提供HTTP资源。

默认情况下，这些资源还将与Reactor Netty和Jetty客户端共享，以实现最佳性能，具体如下：
- 服务器和客户端使用相同的技术
- 客户端实例是使用Spring Boot自动配置的WebClient.Builder bean构建的

通过提供自定义的ReactorResourceFactory或JettyResourceFactory bean，开发人员可以覆盖Jetty和Reactor Netty的资源配置-这将同时应用于客户端和服务器。

您可以在“ WebClient运行时”部分中了解有关客户端资源配置的更多信息。

# 4.8 RSocket

RSocket是一个用于字节流传输的二元协议，它通过单个连接传递的异步消息来启用对称交互模型。

Spring框架的spring-messaging模块在客户端和服务器端都支持RSocket请求者和响应者。 有关更多详细信息，请参见Spring Framework参考中的RSocket部分，其中包括RSocket协议的概述。

## 4.8.1 RScoket自动配置策略

Spring Boot自动配置一个RSocketStrategies bean，该bean提供用于编码和解码RSocket有效负载的所有必需的基础结构。 默认情况下，自动配置将尝试按顺序配置以下内容：
1. CBOR codecs with Jackson
2. JSON codecs with Jackson

spring-boot-starter-rocket启动器同时提供了两种依赖关系。 查阅Jackson支持部分，以了解有关定制可能性的更多信息。

开发人员可以通过创建实现RSocketStrategiesCustomizer接口的bean来自定义RSocketStrategies组件。 请注意，它们的@Order很重要，因为它确定编解码器的顺序。

## 4.8.2 RScoket服务器自动配置

Spring Boot提供了RSocket服务器自动配置。 所需的依赖关系由spring-boot-starter-rsocket提供。

Spring Boot允许从WebFlux服务器通过WebSocket公开RSocket，或支持独立的RSocket服务器。 这取决于应用程序的类型及其配置。

对于WebFlux应用程序（即WebApplicationType.REACTIVE类型），仅当以下属性匹配时，RSocket服务器才会插入Web服务器：
```
spring.rsocket.server.mapping-path=/rsocket # a mapping path is defined
spring.rsocket.server.transport=websocket # websocket is chosen as a transport
#spring.rsocket.server.port= # no port is defined
```
>由于RSocket本身是使用该库构建的，因此只有Reactor Netty支持将RSocket插入Web服务器。

另外，RSocket TCP或Websocket服务器也可以作为独立的嵌入式服务器启动。 除了依赖性要求之外，唯一需要的配置是为该服务器定义端口：
```
spring.rsocket.server.port=9898 # the only required configuration
spring.rsocket.server.transport=tcp # you're free to configure other properties
```
## 4.8.3 Spring Messaging RSocket支持

Spring Boot将为RSocket自动配置Spring Messaging基础结构。

这意味着Spring Boot将创建一个RSocketMessageHandler bean，该bean将处理对您的应用程序的RSocket请求。


## 4.8.4 使用RSocketRequester调用RSocket服务

在服务器和客户端之间建立RSocket通道后，任何一方都可以向另一方发送或接收请求。

作为服务器，您可以在RSocket @Controller的任何处理程序方法上注入RSocketRequester实例。 作为客户端，您需要首先配置和建立RSocket连接。 在这种情况下，Spring Boot会使用预期的编解码器自动配置RSocketRequester.Builder。

RSocketRequester.Builder实例是一个原型bean，这意味着每个注入点将为您提供一个新实例。 这样做是有目的的，因为此构建器是有状态的，因此您不应使用同一实例创建具有不同设置的请求者。
以下代码显示了一个典型示例：
```
@Service
public class MyService {

    private final Mono<RSocketRequester> rsocketRequester;

    public MyService(RSocketRequester.Builder rsocketRequesterBuilder) {
        this.rsocketRequester = rsocketRequesterBuilder
                .connectTcp("example.org", 9898).cache();
    }

    public Mono<User> someRSocketCall(String name) {
        return this.rsocketRequester.flatMap(req ->
                    req.route("user").data(name).retrieveMono(User.class));
    }

}
```
# 4.9 安全

如果[Spring Security](https://spring.io/projects/spring-security)在类路径上，则默认情况下Web应用程序是安全的。 Spring Boot依靠Spring Security的内容协商策略来确定是使用httpBasic还是formLogin。 要将方法级安全性添加到Web应用程序，还可以使用所需的设置添加@EnableGlobalMethodSecurity。 可以在[Spring Security参考指南](https://docs.spring.io/spring-security/site/docs/5.2.2.RELEASE/reference/htmlsingle/#jc-method)中找到更多信息。

默认的UserDetailService只有单个用户，用户名是user,密码是随机的，并且在应用启动时以INFO级别打印，例如下面的示例：
```
Using generated security password: 78fa095d-3f4c-48b1-ad50-e24c31d5cf35
```

>如果您微调日志记录配置，请确保将org.springframework.boot.autoconfigure.security类别设置为记录INFO级别的消息。否则，不会打印默认密码。


你可以通过提供一个spring.security.user.name和spring.security.user.password来改变用户名和密码。

Web应用中默认提供下列基础功能：
- 一个具有内存存储的UserDetailsService（如果是WebFlux应用程序，则为ReactiveUserDetailsService）Bean，一个具有生成的密码的用户（请参阅SecurityProperties.User以获取用户属性）。
- 整个应用程序（当actuator位于类路径时，包括actuator端点）的基于表单登录或HTTP的基本安全性（取决于请求中的Accept标头）。
- 一个用来发布授权时间的DefaultAuthenticationEventPubliser
你可以增加一个Bean来提供一个不同的AuthenticationEventPublisher。

## 4.9.1 MVC安全

默认的安全配置SecurityAutoConfiguration和UserDetailsServiceAutoConfiguration中实现。SecurityAutoConfiguration导入用于Web安的SpringBootWebSecurityConfiguration，而UserDetailsServiceAutoConfiguration配置身份验证,这也在非Web应用中相关。要完全关闭默认的Web应用程序安全性配置或合并多个Spring Security组件（例如OAuth 2客户端和资源服务器），请添加类型为WebSecurityConfigurerAdapter的bean（这样做不会禁用UserDetailsService配置或Actuator的安全性）。

要同时关闭UserDetailsService配置，可以添加类型为UserDetailsService，AuthenticationProvider或AuthenticationManager的bean。

通过添加自定义WebSecurityConfigurerAdapter可以覆盖访问规则。 Spring Boot提供了便利的方法，可用于覆盖执行器端点和静态资源的访问规则。 EndpointRequest可用于创建基于management.endpoints.web.base-path属性的RequestMatcher。 PathRequest可用于为常用位置中的资源创建RequestMatcher。

## 4.9.2 WebFlux安全

与Spring MVC应用程序类似，您可以通过添加spring-boot-starter-security依赖项来保护WebFlux应用程序。 默认的安全配置在ReactiveSecurityAutoConfiguration和UserDetailsServiceAutoConfiguration中实现。 ReactiveSecurityAutoConfiguration导入WebFluxSecurityConfiguration以获得Web安全，而UserDetailsServiceAutoConfiguration配置身份验证，这在非Web应用程序中也相关。 要完全关闭默认的Web应用程序安全配置，您可以添加WebFilterChainProxy类型的Bean（这样做不会禁用UserDetailsService配置或Actuator的安全性）。

要同时关闭UserDetailsService配置，您可以添加ReactiveUserDetailsService或ReactiveAuthenticationManager类型的Bean。

可以通过添加自定义SecurityWebFilterChain bean来配置访问规则以及使用多个Spring Security组件（例如OAuth 2 Client和Resource Server）。 Spring Boot提供了便利的方法，可用于覆盖执行器端点和静态资源的访问规则。 EndpointRequest可用于创建基于management.endpoints.web.base-path属性的ServerWebExchangeMatcher。

可以使用PathRequest为常用位置中的资源创建ServerWebExchangeMatcher。

例如，您可以通过添加以下内容来自定义安全配置：

```
@Bean
public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
    return http
        .authorizeExchange()
            .matchers(PathRequest.toStaticResources().atCommonLocations()).permitAll()
            .pathMatchers("/foo", "/bar")
                .authenticated().and()
            .formLogin().and()
        .build();
}
```
## 4.9.3 OAuth2

OAuth2是Spring支持的一种广泛使用的授权框架。

**Client**

如果您在类路径中具有spring-security-oauth2-client，则可以利用一些自动配置功能来轻松设置OAuth2 / Open ID Connect客户端。 此配置使用OAuth2ClientProperties下的属性。 相同的属性适用于servlet和反应式应用程序。

您可以在spring.security.oauth2.client前缀下注册多个OAuth2客户端和提供者，如以下示例所示：
```
spring.security.oauth2.client.registration.my-client-1.client-id=abcd
spring.security.oauth2.client.registration.my-client-1.client-secret=password
spring.security.oauth2.client.registration.my-client-1.client-name=Client for user scope
spring.security.oauth2.client.registration.my-client-1.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-1.scope=user
spring.security.oauth2.client.registration.my-client-1.redirect-uri=https://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-1.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-1.authorization-grant-type=authorization_code

spring.security.oauth2.client.registration.my-client-2.client-id=abcd
spring.security.oauth2.client.registration.my-client-2.client-secret=password
spring.security.oauth2.client.registration.my-client-2.client-name=Client for email scope
spring.security.oauth2.client.registration.my-client-2.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-2.scope=email
spring.security.oauth2.client.registration.my-client-2.redirect-uri=https://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-2.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-2.authorization-grant-type=authorization_code

spring.security.oauth2.client.provider.my-oauth-provider.authorization-uri=https://my-auth-server/oauth/authorize
spring.security.oauth2.client.provider.my-oauth-provider.token-uri=https://my-auth-server/oauth/token
spring.security.oauth2.client.provider.my-oauth-provider.user-info-uri=https://my-auth-server/userinfo
spring.security.oauth2.client.provider.my-oauth-provider.user-info-authentication-method=header
spring.security.oauth2.client.provider.my-oauth-provider.jwk-set-uri=https://my-auth-server/token_keys
spring.security.oauth2.client.provider.my-oauth-provider.user-name-attribute=name
```
对于支持OpenID Connect发现的OpenID Connect提供程序，可以进一步简化配置。提供者需要配置有一个issuer-uri，这是它声明为其发布者标识符的URI。例如，如果提供的issuer-uri是"https://example.com"， 则将向 “ https://example.com/.well-known/openid-configuration” 发出OpenID提供程序配置请求。结果应为OpenID提供程序配置响应。 以下示例显示了如何使用issuer-uri配置OpenID Connect提供程序：
```
spring.security.oauth2.client.provider.oidc-provider.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/
```

默认情况下，Spring Security的OAuth2LoginAuthenticationFilter仅处理与/ login/oauth2/code/\*匹配的URL。 如果要自定义redirect-uri以使用其他模式，则需要提供配置以处理该自定义模式。 例如，对于servlet应用程序，您可以添加自己的类似于以下内容的WebSecurityConfigurerAdapter：
```
public class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .anyRequest().authenticated()
                .and()
            .oauth2Login()
                .redirectionEndpoint()
                    .baseUri("/custom-callback");
    }
}
```
**普通提供商的OAuth2客户端注册**

对于常见的OAuth2和OpenID提供程序，包括Google，Github，Facebook和Okta，我们提供了一组提供程序默认值（分别为Google，github，facebook和okta）。

如果不需要自定义这些提供程序，则可以将提供程序属性设置为你需要的继承默认值的值。 另外，如果用于客户端注册的密钥与默认支持的提供程序匹配，Spring Boot也会进行推断。

换句话说，以下示例中的两种配置都使用Google提供程序：
```
spring.security.oauth2.client.registration.my-client.client-id=abcd
spring.security.oauth2.client.registration.my-client.client-secret=password
spring.security.oauth2.client.registration.my-client.provider=google

spring.security.oauth2.client.registration.google.client-id=abcd
spring.security.oauth2.client.registration.google.client-secret=password
```

**Resource Server**

如果您的类路径上有spring-security-oauth2-resource-server，则Spring Boot可以设置OAuth2资源服务器。 对于JWT配置，需要指定JWK设置URI或OIDC颁发者URI，如以下示例所示：
```
spring.security.oauth2.resourceserver.jwt.jwk-set-uri=https://example.com/oauth2/default/v1/keys
```

```
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/
```
>如果授权服务器不支持JWK设置URI，则可以使用用于验证JWT签名的公共密钥来配置资源服务器。 可以使用spring.security.oauth2.resourceserver.jwt.public-key-location属性完成此操作，该值需要指向包含PEM编码的x509格式的公钥的文件。

相同的属性适用于servlet和反应式应用程序。

另外，您可以为Servlet应用程序定义自己的JwtDecoder Bean，或者为反应性应用程序定义ReactiveJwtDecoder。

如果使用不透明令牌而不是JWT，则可以配置以下属性以通过自省来验证令牌：
```
spring.security.oauth2.resourceserver.opaquetoken.introspection-uri=https://example.com/check-token
spring.security.oauth2.resourceserver.opaquetoken.client-id=my-client-id
spring.security.oauth2.resourceserver.opaquetoken.client-secret=my-client-secret
```
再次强调，相同的属性适用于servlet和反应式应用程序。

另外，您可以为Servlet应用程序定义自己的OpaqueTokenIntrospector Bean，或者为反应性应用程序定义ReactiveOpaqueTokenIntrospector。

**授权服务器**

当前，Spring Security不提供实现OAuth 2.0授权服务器的支持。 但是，Spring Security OAuth项目提供了此功能，最终将被Spring Security完全取代。 在此之前，您可以使用spring-security-oauth2-autoconfigure模块轻松设置OAuth 2.0授权服务器； 有关说明，请参见其文档。

## 4.9.4 SAML 2.0

**Relying  Party**

如果您在类路径中具有spring-security-saml2-service-provider，则可以利用一些自动配置功能来轻松设置SAML 2.0依赖方。 此配置使用Saml2RelyingPartyProperties下的属性。

依赖方注册表示身份提供者IDP和服务提供者SP之间的配对配置。 您可以在spring.security.saml2.relyingparty前缀下注册多个依赖方，如以下示例所示：
```
spring.security.saml2.relyingparty.registration.my-relying-party1.signing.credentials[0].private-key-location=path-to-private-key
spring.security.saml2.relyingparty.registration.my-relying-party1.signing.credentials[0].certificate-location=path-to-certificate
spring.security.saml2.relyingparty.registration.my-relying-party1.identityprovider.verification.credentials[0].certificate-location=path-to-verification-cert
spring.security.saml2.relyingparty.registration.my-relying-party1.identityprovider.entity-id=remote-idp-entity-id1
spring.security.saml2.relyingparty.registration.my-relying-party1.identityprovider.sso-url=https://remoteidp1.sso.url

spring.security.saml2.relyingparty.registration.my-relying-party2.signing.credentials[0].private-key-location=path-to-private-key
spring.security.saml2.relyingparty.registration.my-relying-party2.signing.credentials[0].certificate-location=path-to-certificate
spring.security.saml2.relyingparty.registration.my-relying-party2.identityprovider.verification.credentials[0].certificate-location=path-to-other-verification-cert
spring.security.saml2.relyingparty.registration.my-relying-party2.identityprovider.entity-id=remote-idp-entity-id2
spring.security.saml2.relyingparty.registration.my-relying-party2.identityprovider.sso-url=https://remoteidp2.sso.url
```

## 4.9.5 Actuator安全

为了安全起见，默认情况下禁用/health和/info以外的所有执行器。 management.endpoints.web.exposure.include属性可用于启用执行器。

如果Spring Security位于类路径上，并且不存在其他WebSecurityConfigurerAdapter，则除/health和/info以外的所有执行器均由Spring Boot自动配置保护。 如果定义自定义WebSecurityConfigurerAdapter，则Spring Boot自动配置将退出，您将完全控制执行器访问规则。

>在设置management.endpoints.web.exposure.include之前，请确保裸露的执行器不包含敏感信息和/或通过将它们放置在防火墙后面或通过诸如Spring Security之类的方法进行保护。

**跨站点请求伪造保护**

由于Spring Boot依赖于Spring Security的默认值，因此CSRF保护默认情况下处于启用状态。 这意味着在使用默认安全配置时，需要POST（关闭和记录器端点），PUT或DELETE的执行器端点将收到403禁止错误。
>我们建议仅在创建非浏览器客户端使用的服务时完全禁用CSRF保护。

关于CSRF保护的其他信息可以在[Spring Security Reference Guide](https://docs.spring.io/spring-security/site/docs/5.2.2.RELEASE/reference/htmlsingle/#csrf)中找到。
