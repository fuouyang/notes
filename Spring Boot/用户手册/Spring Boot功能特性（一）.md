[全集](https://www.jianshu.com/p/a661eeaa2935)

这个章节深入挖掘Spring boot的细节，你可以了解到你想使用或定制的关键功能。如果你还没准备好，请先阅读入门指南和使用Spring Boot章节，以便您有良好的基础知识。

# 4.1 SpringApplication

SpringApplication类提供了一个快捷的方式来引导Spring Boot的启动，通过main方法。在许多情况下，您可以委派给静态SpringApplication.run方法，如以下示例所示：
```
public static void main(String[] args) {
    SpringApplication.run(MySpringConfiguration.class, args);
}
```
当你的应用启动时，你会看到和下面相似的输出：
```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v2.2.6.RELEASE

2019-04-31 13:09:54.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2019-04-31 13:09:54.166  INFO 56603 --- [           main] ationConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2019-04-01 13:09:56.912  INFO 41370 --- [           main] .t.TomcatServletWebServerFactory : Server initialized with port: 8080
2019-04-01 13:09:57.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658)
```
默认展示一些INFO级别的日志，包括一些相关的启动细节，例如启动应用的用户。如果你需要一个不是INFO级别的日志输出，你可以像在[日志级别](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features-custom-log-levels)中描述的一样设置它。主应用程序类包中使用的实现版本确定应用程序版本.启动日志信息可以通过设置spring.main.log-startup-info为false来关闭它。这还将关闭对应用程序活动配置文件的日志记录。
>要在启动期间添加其他日志记录，可以在SpringApplication的子类中重写logStartupInfo（boolean）。

## 4.1.1 启动失败
如果您的应用程序无法启动，则已注册的FailureAnalyzers将有机会提供专门的错误消息和解决该问题的具体措施。 例如，如果您在端口8080上启动Web应用程序，并且该端口已在使用中，则应该看到类似于以下消息的内容：
```
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```
>Spring Boot提供匿名的FailureAnalyzers实现，你也可以[添加自己的](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#howto-failure-analyzer)

如果没有故障分析器能够处理异常，您仍然可以显示完整情况报告以更好地了解出了什么问题。这样做你需要 org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener开启debug属性或者开启debug日志

举个例子，如果使用java -jar运行应用程序，则可以按以下方式启用debug属性：
```
 $ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```
## 4.1.2 懒加载

SpringApplication允许应用惰性初始化。如果应用开启了懒加载，Bean会在需要时创建而不是应用启动时创建。这样，就可以缩短应用启动花费的时间。在Web应用中，懒加载会是需要Web相关的Bean直到一个HTTP请求到来时才被初始化。

懒加载的一个缺点是会延迟发现应用问题的时间。如果配置错误的bean延迟初始化，则启动期间将不再发生故障，只有在初始化bean时问题才会显现。还必须注意确保JVM有足够的内存来容纳所有应用程序的bean，而不仅仅是启动期间初始化的那些bean。由于这些原因，默认情况下不会启用延迟初始化，因此建议在启用延迟初始化之前先对JVM的堆大小进行微调。

可以使用SpringApplicationBuilder上的lazyInitialization方法或SpringApplication上的setLazyInitialization方法以编程方式启用延迟初始化，或者，可以使用spring.main.lazy-initialization属性启用它，如以下示例所示：
```
 spring.main.lazy-initialization = true
```
>如果要在对应用程序其余部分使用延迟初始化时禁用某些Bean的延迟初始化，则可以使用@Lazy（false）批注将它们的延迟属性显式设置为false。

## 4.1.3 定制Banner

可以通过将banner.txt文件添加到类路径或将spring.banner.location属性设置为此类文件的位置来更改启动时打印的banner。如果该文件使用UTF-8之外的编码，可以设置spring.banner.charset属性。除了文本文件之外，您还可以将banner.gif，banner.jpg或banner.png图像文件添加到类路径中，或设置spring.banner.image.location属性。 图像将转换为ASCII艺术作品并打印在任何文本横幅上方。

在banner.txt文件中，你可以设置下列任何一个占位符：

**Table 4  Banner变量**

Variable|Desription
-|-
${application.version}|您的应用程序的版本号，在MANIFEST.MF中声明，例如，实现版本：1.0被打印为1.0。
${application.formatted-version}|您在MANIFEST.MF中声明的应用程序的版本号，其格式用于显示（用方括号括起来并以v开头）， 例如（v1.0）
${spring-boot.version}|您正在使用的Spring Boot版本。 例如2.2.6.RELEASE
${spring-boot.formatted-version}|您正在使用的Spring Boot版本，其格式用于显示（用方括号括起来并以v开头）， 例如（v2.2.6.RELEASE）
\${Ansi.NAME} (or \${AnsiColor.NAME}, \${AnsiBackground.NAME}, ${AnsiStyle.NAME})|其中NAME是ANSI转义代码的名称， 有关详细信息，请参见[AnsiPropertySource](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java)
${application.title}|您在MANIFEST.MF中声明的应用程序的标题。 例如，“实现标题”：MyApp被打印为MyApp

>如果要以编程方式生成banner，可以使用SpringApplication.setBanner()方法。 使用org.springframework.boot.Banner接口并实现自己的printBanner()方法。

您还可以使用spring.main.banner-mode属性来确定横幅是否必须在System.out（控制台）上打印，是否必须发送到配置的记录器（日志）或根本不打印（关闭）。

打印的横幅广告使用以下名称注册为单例bean：springBootBanner。

## 4.1.4 定制SpringApplication

如果默认的SpringApplication不符合您的要求，您可以生成一个本地实例并定制它作为替代，例如，关闭打印banner，你可以这样写：
```
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```
>传递给SpringApplication的构造函数参数是Spring bean的配置源。 在大多数情况下，它们是对@Configuration类的引用，但它们也可以是对XML配置或应扫描的程序包的引用。

可以在application.properties中配置你的SpringApplication,详情见参见[外部化配置](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features-external-config)

有关配置选项的完整列表，请参见[SpringApplication Javadoc](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/SpringApplication.html)。

## 4.1.5 流式构建API

如果您需要构建ApplicationContext层次结构（具有父/子关系的多个上下文），或者如果您更喜欢使用“流式的”构建器API，则可以使用SpringApplicationBuilder。

SpringApplicationBuilder允许您将多个方法调用链接在一起，并包括允许您创建层次结构的父方法和子方法，如以下示例所示：
```
new SpringApplicationBuilder()
        .sources(Parent.class)
        .child(Application.class)
        .bannerMode(Banner.Mode.OFF)
        .run(args);
```
>创建ApplicationContext层次结构时有一些限制。 例如，Web组件必须包含在子上下文中，并且相同的环境用于父上下文和子上下文。 有关完整的详细信息，请参见[SpringApplicationBuilder Javadoc](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/builder/SpringApplicationBuilder.html)。

## 4.1.6 应用程序事件和监听器

除了通常的Spring Framework事件（例如ContextRefreshedEvent）之外，SpringApplication还发送一些其他应用程序事件。

>一些事件在应用程序上下文创建前触发，所以你不能在这些事件中将一个Listener注册为一个Bean。你可以通过SpringApplication的addListener()方法或SpringApplicationBuilder的listeners()方法来注册它们。
>
>如果你希望这些Listeners被自动注册，而不用去管应用以何种方式创建，你可以在工程中添加一个META-INF/spring.factories文件并通过org.springframework.context.ApplicationListener来引用它们，像下面展示的这样：
org.springframework.context.ApplicationListener=com.example.project.MyListener

当应用启动时，应用事件按照下列顺序发送：
1. ApplicationStartingEvent在运行开始时发送，但在进行任何处理之前（侦听器和初始化程序的注册除外）发送。
2. 当已知要在上下文中使用的环境但在创建上下文之前，将发送ApplicationEnvironmentPreparedEvent。
3. 准备ApplicationContext并调用ApplicationContextInitializers之后但在加载任何bean定义之前，将发送ApplicationContextInitializedEvent。
4. 在刷新开始之前但在加载bean定义之后发送ApplicationPreparedEvent。
5. 在刷新上下文之后但在调用任何应用程序和命令行运行程序之前，发送ApplicationStartedEvent。
6. 在调用任何应用程序和命令行运行程序之后，将发送ApplicationReadyEvent。 它指示应用程序已准备就绪，可以处理请求。
7. 如果启动时发生异常，则发送ApplicationFailedEvent。

上面的列表仅包括绑定到SpringApplication的SpringApplicationEvents，除了上述那些，下面的事件也会在ApplicationPreparedEvent之后且在ApplicationStartedEvent之前发送。
1. 刷新ApplicationContext时，将发送ContextRefreshedEvent。
2. WebServer准备就绪后，将发送WebServerInitializedEvent。 ServletWebServerInitializedEvent和ReactiveWebServerInitializedEvent分别是servlet和反应式变量。

>您通常不需要使用应用程序事件，但是很容易知道它们的存在。 在内部，Spring Boot使用事件来处理各种任务。

应用程序事件是通过使用Spring Framework的事件发布机制发送的。 此机制的一部分确保在子级上下文中发布给侦听器的事件也可以在任何祖先上下文中发布给侦听器。 结果，如果您的应用程序使用SpringApplication实例的层次结构，则侦听器可能会收到同一类型的应用程序事件的多个实例。

为了使您的侦听器能够区分其上下文的事件和后代上下文的事件，它应请求注入其应用程序上下文，然后将注入的上下文与事件的上下文进行比较。 可以通过实现ApplicationContextAware来注入上下文，或者，如果侦听器是bean，则可以使用@Autowired注入上下文。

## 4.1.7 Web环境

SpringApplication尝试代表您创建正确的ApplicationContext类型，用于确定WebApplicationType的算法非常简单：
- 如果存在Spring MVC，则使用AnnotationConfigServletWebServerApplicationContext
- 如果不存在Spring MVC并且存在Spring WebFlux，则使用AnnotationConfigReactiveWebServerApplicationContext
-否则使用AnnotationConfigApplicationContext

这意味着，如果您在同一应用程序中使用Spring MVC和Spring WebFlux中的新WebClient，则默认情况下将使用Spring MVC。 您可以通过调用setWebApplicationType（WebApplicationType）轻松覆盖它。

也可以通过调用setApplicationContextClass(…​)来完全控制ApplicationContext类型。

>在JUnit测试中使用SpringApplication时，通常希望调用setWebApplicationType（WebApplicationType.NONE）。

## 4.1.8 接收应用参数

如果您需要访问传递给SpringApplication.run（...）的应用程序参数，则可以注入org.springframework.boot.ApplicationArguments bean。 ApplicationArguments接口提供对原始String []参数以及已解析的选项和非选项参数的访问，如以下示例所示：
```
import org.springframework.boot.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.stereotype.*;

@Component
public class MyBean {

    @Autowired
    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
    }

}
```
>Spring Boot还向Spring环境注册了CommandLinePropertySource。 这样，您还可以使用@Value批注注入单个应用程序参数。

## 4.1.9 使用ApplicationRunner和CommandLineRunner

如果您在应用启动之后立即执行一些特殊的代码，您可以实现ApplicationRunner或CommandLineRunner接口。这两个接口以相同的方式工作并提供一个单独的在SpingApplication.run()完成前调用的run()方法。

CommandLineRunner接口以简单的字符串数组提供对应用程序参数的访问，而ApplicationRunner使用前面讨论的ApplicationArguments接口。 以下示例显示了带有run方法的CommandLineRunner：
```
import org.springframework.boot.*;
import org.springframework.stereotype.*;

@Component
public class MyBean implements CommandLineRunner {

    public void run(String... args) {
        // Do something...
    }

}
```
如果多个CommandLineRunner或ApplicationRunner的Bean被定义，它们必须以顺序的方式调用，你可以实现org.springframework.core.ordered接口或是使用org.springframework.core.annotation.Order注解

## 4.1.10 应用退出

每个SpringApplication向JVM注册一个关闭钩子，以确保ApplicationContext在退出时正常关闭。 可以使用所有标准的Spring生命周期回调（例如DisposableBean接口或@PreDestroy批注）。

另外，如果bean希望在调用SpringApplication.exit（）时返回特定的退出代码，则可以实现org.springframework.boot.ExitCodeGenerator接口。 然后可以将此退出代码传递给System.exit（），以将其作为状态代码返回，如以下示例所示：
```
@SpringBootApplication
public class ExitCodeApplication {

    @Bean
    public ExitCodeGenerator exitCodeGenerator() {
        return () -> 42;
    }

    public static void main(String[] args) {
        System.exit(SpringApplication.exit(SpringApplication.run(ExitCodeApplication.class, args)));
    }

}
```
另外，ExitCodeGenerator接口可以通过异常实现。 遇到此类异常时，Spring Boot返回实现的getExitCode()方法提供的退出代码。
## 4.1.11 管理员功能
通过指定spring.application.admin.enabled属性，可以为应用程序启用与管理员相关的功能。 这将在平台MBeanServer上公开SpringApplicationAdminMXBean。 您可以使用此功能来远程管理Spring Boot应用程序。 对于任何服务包装器实现，此功能也可能很有用。

>如果您想知道应用程序在哪个HTTP端口上运行，请使用local.server.port键获取属性。

# 4.2 外部配置

Spring Boot使您可以外部化配置，以便可以在不同环境中使用相同的应用程序代码。 您可以使用properties文件，YAML文件，环境变量和命令行参数来外部化配置。 可以使用@Value批注将属性值直接注入到您的bean中，可以通过Spring的Environment抽象访问，也可以通过@ConfigurationProperties[绑定到结构化对象](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features-external-config-typesafe-configuration-properties)。

Spring Boot使用一个非常特殊的PropertySource顺序，该顺序旨在允许合理地覆盖值。属性应该按照下列的顺序：

1. 当使用devtools时，$ HOME / .config / spring-boot文件夹中的Devtools[全局设置属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#using-boot-devtools-globalsettings)。
2. 测试用例上的@TestPropertySource注解
3. 测试用例上的properties属性。在@SpringBootTest和测试注释上可用，用于测试应用程序的特定部分。
4. 命令行参数。
5. 来自SPRING_APPLICATION_JSON的属性（嵌入在环境变量或系统属性中的内联JSON）。
6. ServletConfig初始化参数。
7. ServletContext初始化参数。
8. java:comp/env中的JNDI属性。
9. Java系统属性(System.getProperties())。
10. 操作系统环境变量。
11. 一个RandomValuePropertySource，仅具有random.\*属性。
12. 打包的jar之外的[特定于配置文件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features-external-config-profile-specific-properties)的应用程序属性（application- {profile} .properties和YAML变体）。
13. 打包在jar中的[特定于配置文件])(https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features-external-config-profile-specific-properties)的应用程序属性（application- {profile} .properties和YAML变体）。
14. 打包的jar之外的应用程序属性（application.properties和YAML变体）。
15. 打包在jar中的应用程序属性（application.properties和YAML变体）。
16. @Configuration类上的@PropertySource注解。 请注意，在刷新应用程序上下文之前，不会将此类属性源添加到环境中。 现在配置某些属性（如loggin.\*和spring.main.\*）为时已晚，这些属性在刷新开始之前就已读取。
17. 默认属性（通过设置SpringApplication.setDefaultProperties指定）。

提供一个具体的例子，假设你正在开发一个使用了一个name属性的@Component，像下面这样：
```
import org.springframework.stereotype.*;
import org.springframework.beans.factory.annotation.*;

@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...

}
```

在您的应用程序类路径上（例如，在jar内），您可以拥有一个application.properties文件，该文件为名称提供合理的默认属性值。 在新环境中运行时，可以在jar外部提供一个覆盖名称的application.properties文件。 对于一次性测试，可以使用特定的命令行开关启动（例如，java -jar app.jar --name =“ Spring”）。

> 可以在命令行中使用环境变量来提供SPRING_APPLICATION_JSON属性。 例如，您可以在UN * X shell中使用以下行：<br/>
  $ SPRING_APPLICATION_JSON='{"acme":{"name":"test"}}' java -jar myapp.jar
>  
  在前面的示例中，您最终在Spring Environment中获得了acme.name = test。 您还可以在System属性中将JSON作为spring.application.json提供，如以下示例所示：<br/>
  $ java -Dspring.application.json='{"name":"test"}' -jar myapp.jar
>
>  您还可以使用命令行参数来提供JSON，如以下示例所示：
  $ java -jar myapp.jar --spring.application.json='{"name":"test"}'
>  
  您还可以将JSON作为JNDI变量提供，如下所示：<br/>
  java：comp / env / spring.application.json。

## 4.2.1 配置任意值

RandomValuePropertySource可用于注入随机值(例如，秘钥或测试案例)，它可以生成integers,longs,uuids，或者strings,如下所示：
```
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,35536]}
```

random.int\*语法是OPEN value(，max ) CLOSE，其中OPEN，CLOSE是任何字符，value，max是整数。 如果提供了max，则value是最小值，而max是最大值（不包括）。

## 4.2.2 获取命令行属性

默认SpringApplicatin转换任何命令行可选参数(参数以--开始，例如--server.port=9000)到一个属性并将其添加到Spring Environment抽象中。正如前面提到过的，命令行参数总数优先于其它属性源。

如果你不想将你的命令行参数添加到Environment,你可以通过SpringApplicaton.setAddCommandLineProperties(false)来关闭它。

## 4.2.3 应用程序文件

SpringApplicaton从下面位置的application.properties文件加载属性并将其添加到Spring环境中:
1. 在当前目录的一个/config子目录
2. 当前目录
3. 类路径中的/config包
4. 根类路径

该列表按照优先级排列（在优先级高的位置定义的属性会覆写那些在低优先级位置定义的）

> 您也可以用YAML文件来替代.properties文件

如果你不喜欢application.properties作为配置文件名称，你也可以指定spring.config.name环境属性切换到其它的文件。你同样可以通过spring.config.location环境属性来引用一个外部的位置（用逗号分隔的目录或文件路径列表）,下面示例展示了如何指定一个不同的文件名称：
```
 $ java -jar myproject.jar --spring.config.name=myproject
```
下面示例展示了如何指定两个位置：
```
 $ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
```

>spring.config.name和spring.config.location在最开始就被用来决定那些文件应被加载，它们必须作为环境属性被定义（通常是操作系统环境变量，一个系统属性，或是命令行参数）

如果Spring.config.location包含目录（而不是文件），那应该以/结束（并在运行之前，在从spring.config.name中生成的文件名被加载之前，附加在生成的文件之后，包含特定配置文件的文件名）。spring.config.location中指定的文件按原样使用，不支持特定配置文件的变体，并且被任何特定配置文件的属性覆盖。

配置目录按反向顺序搜索。默认配置文件位置是 classpath:/,classpath:/config/,file:./,file:./config/,结果搜索顺序如下：

1. file:/config/
2. file:./
3. classpath:/config/
4. classpath:/

使用spring.config.location配置自定义配置位置后，它们将替换默认位置。例如，如果spring.config.location按classpath:/custom-config/,file:./custom-config/配置，搜索顺序变成下列所示：

1. file:./custom-config/
2. classpath:custom-config/

另外，当使用spring.config.additional-location配置自定义配置位置时，除默认位置外，还会使用它们。 在默认位置之前搜索其他位置。 例如，如果配置了classpath:/ custom-config/，file:./custom-config/的其他位置，则搜索顺序如下：

1. file:/custom-config/
2. classpath:custom-config/
3. file:./config/
4. file:./
5. classpath:/config/
6. classpath:/

搜索顺序允许你在一个配置文件中指定一些默认值然后再其它配置文件中选择性的覆写那些值。您可以在默认位置之一的application.properties（或使用spring.config.name选择的其他任何基本名称）中为应用程序提供默认值。然后，可以在运行时使用自定义位置之一中的其他文件覆盖这些默认值。

>如果你使用环境变量而不是系统变量，大多数操作系统不允许使用句点分隔的键名，但是可以使用下划线代替（例如，使用SPRING_CONFIG_NAME代替spring.config.name）。

>如果您的应用程序在容器中运行，则可以使用JNDI属性（在java：comp / env中）或servlet上下文初始化参数代替环境变量或系统属性，也可以使用它。

## 4.2.4  特定于配置文件的属性

除了application.properties文件之外，还可以使用以下命名约定来定义特定于配置文件的属性：application- {profile} .properties。 如果没有设置活动配置文件，则环境具有一组默认配置文件（默认为[default]）。 换句话说，如果未显式激活任何配置文件，那么将加载application-default.properties中的属性。

特定于配置文件的属性是从与标准application.properties相同的位置加载的，特定于配置文件的文件总是会覆盖非特定文件，无论特定于配置文件的文件是位于打包jar的内部还是外部。

如果指定了多个配置文件，则采用后赢策略。 例如，在通过SpringApplication API配置了配置文件之后，添加了spring.profiles.active属性指定的配置文件，其具有优先权。

>如果您在spring.config.location中指定了任何文件，则不考虑这些文件的特定于配置文件的变体。 如果您还想使用特定于配置文件的属性，请使用spring.config.location中的目录。

## 4.2.5 属性中的占位符

使用application.properties中的值时，它们会通过现有环境进行过滤，因此您可以参考以前定义的值（例如，从System属性中）。
```
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

>您还可以使用此技术来创建现有Spring Boot属性的“简短”变体。 有关详细信息，请参见[使用“简短”命令行参数](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#howto-use-short-command-line-arguments)方法。

## 4.2.6 加密属性

Spring Boot不提供对加密属性值的任何内置支持，但是，它确实提供了修改Spring环境中包含的值所必需的挂钩点。EnvironmentPostProcessor界面允许您在应用程序启动之前操纵Environment。 有关详细信息，请参见[在启动前自定义环境或ApplicationContext](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#howto-customize-the-environment-or-application-context)。

如果您正在寻找一种安全的方式来存储凭据和密码，则[Spring Cloud Vault](https://cloud.spring.io/spring-cloud-vault/)项目提供了对在[HashiCorp Vault](https://www.vaultproject.io/)中存储外部化配置的支持。

## 4.2.7 使用YAML代替Properties

[YAML](https://yaml.org/)是JSON的超集，因此是一种用于指定层次结构配置数据的便捷格式。只要在类路径上具有SnakeYAML库，SpringApplication类就会自动支持YAML作为Properties的替代。

>如果你使用了“Starters”,spring-boot-starter已经自动提供了SnakeYAML。

**加载YAML**

Spring Framework提供了两个便捷的类可以用来加载YAML文档。YamlPropertiesFactoryBean将YAML作为Properties加载，而YamlMapFactoryBean将YAML作为Map加载。

例如，思考下面的YAML文档：
```
environments:
    dev:
        url: https://dev.example.com
        name: Developer Setup
    prod:
        url: https://another.example.com
        name: My Cool App
```
上述示例将被转换成下列属性：
```
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```
YAML列表用[index]解引用器表示为属性键。例如，思考下面的YAML:
```
my:
   servers:
       - dev.example.com
       - another.example.com
```
上述的示例将被转换为下述属性：
```
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```
要使用Spring Boot的Binder实用程序（@ConfigurationProperties所做的）绑定到类似的属性，您需要在目标bean中拥有一个类型为java.util.List（或Set）的属性，或者您需要提供一个setter 或使用可变值对其进行初始化。 例如，以下示例绑定到前面显示的属性：
```
@ConfigurationProperties(prefix="my")
public class Config {

    private List<String> servers = new ArrayList<String>();

    public List<String> getServers() {
        return this.servers;
    }
}
```

**在Spring环境中将YAML公开为属性**

YamlPropertySourceLoader类可用于在Spring环境中将YAML公开为PropertySource。 这样做可以让您使用@Value批注和占位符语法来访问YAML属性。


**多个特定配置的YAML文件**

您可以使用spring.profiles键在一个文件中指定多个特定于配置文件的YAML文档，以指示何时应用该文档，如以下示例所示：
```
server:
    address: 192.168.1.100
---
spring:
    profiles: development
server:
    address: 127.0.0.1
---
spring:
    profiles: production & eu-central
server:
    address: 192.168.1.120
```
在前面的示例中，如果开发配置文件处于活动状态，则server.address属性为127.0.0.1。 同样，如果生产和eu-central配置文件处于活动状态，则server.address属性为192.168.1.120。 如果未启用开发，生产和其他中心配置文件，则该属性的值为192.168.1.100。
>因此spring.profiles可以包含一个简单的概要文件名称（例如生产）或概要文件表达式。 配置文件表达式允许表达更复杂的配置文件逻辑，例如production＆（eu-central | eu-west）。 有关更多详细信息，请参阅[参考指南](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java)。

如果在启动应用程序上下文时未显式激活任何活动，则会激活默认配置文件。 因此，在以下YAML中，我们为spring.security.user.password设置了一个值，该值仅在“默认”配置文件中可用：
```
server:
  port: 8000
---
spring:
  profiles: default
  security:
    user:
      password: weak
```

而在以下示例中，始终设置密码是因为该密码未附加到任何配置文件，并且必须根据需要在所有其他配置文件中将其显式重置：
```
server:
  port: 8000
spring:
  security:
    user:
      password: weak
```

通过使用spring.profiles指定的Spring文件可以选择用'!'来否定。如果在一个文档中未否定和否定的profile同时存在的话，最后一个未否定文件必须匹配，并且可能没有否定文件可以匹配。

**YAML缺点**
无法使用@PropertySource注解加载YAML文件。 因此，在需要以这种方式加载值的情况下，需要使用属性文件。

在特定于配置文件的YAML文件中使用多YAML文档语法可能会导致意外行为。 例如，考虑文件中的以下配置：
```
server:
  port: 8000
---
spring:
  profiles: "!test"
  security:
    user:
      password: "secret"
```
如果使用参数--spring.profiles.active = dev运行应用程序，则可能希望将security.user.password设置为“ secret”，但事实并非如此。

嵌套文档将被过滤，因为主文件名为application-dev.yml。 它已经被认为是特定于配置文件的，并且嵌套文档将被忽略。

>我们建议您不要混用特定于配置文件的YAML文件和多个YAML文档。 坚持只使用其中之一。

## 4.2.8 类型安全的配置属性

使用@Value(${property}”)注解来注入配置属性有时会很麻烦，尤其是当您使用多个属性或数据本质上是分层的时。 Spring Boot提供了一种使用属性的替代方法，该方法使强类型的Bean可以管理和验证应用程序的配置。
>参见[@Value和类型安全配置的区别](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features-external-config-vs-value)

**JavaBean属性绑定**

可以绑定一个声明标准JavaBean属性的bean，如以下示例所示：
```
package com.example;

import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("acme")
public class AcmeProperties {

    private boolean enabled;

    private InetAddress remoteAddress;

    private final Security security = new Security();

    public boolean isEnabled() { ... }

    public void setEnabled(boolean enabled) { ... }

    public InetAddress getRemoteAddress() { ... }

    public void setRemoteAddress(InetAddress remoteAddress) { ... }

    public Security getSecurity() { ... }

    public static class Security {

        private String username;

        private String password;

        private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

        public String getUsername() { ... }

        public void setUsername(String username) { ... }

        public String getPassword() { ... }

        public void setPassword(String password) { ... }

        public List<String> getRoles() { ... }

        public void setRoles(List<String> roles) { ... }

    }
}
```
上述的POJO定义了下列属性：
- acme.enabled, 默认值为false。
- acme.remote-address, 具有可以从String强制转换的类型。
- acme.security.username,带有嵌套的“安全”对象，其名称由属性名称决定。 特别是，返回类型根本不使用，可能是SecurityProperties。
- acme.security.password。
- acme.security.roles,带有默认为"USER"的String集合。
>映射到Spring Boot中可用的@ConfigurationProperties类的属性（通过属性文件，YAML文件，环境变量等进行配置）是公共API，但是该类本身的访问器（获取器/设置器）不能直接使用。

>这种安排依赖于默认的空构造函数，并且getter和setter通常是强制性的，因为绑定是通过标准Java Beans属性描述符进行的，就像在Spring MVC中一样。 在以下情况下，可以忽略setter：
- Map,只要初始化，它们就需要使用getter，但不一定需要使用setter，因为它们可以被binder改变。
- 集合和数组可以通过下标（通常是YAML）或一个逗号分隔的值来访问，在后面的例子中，setter是必须的。我们建议你为这种类型总是添加一个setter。如果你初始化了一个集合，确保它是不可变的（像上述示例一样）
- 如果嵌套的POJO属性被初始化了（像上述示例中的Security一样），setter并不是必须的。如果希望binder通过使用其默认构造函数动态创建实例，则需要一个setter。
>
>一些人使用Lombok项目来自动添加getters和setters。确保Lombok不会为这样的类型添加人恶化特殊构造器，因为容器会自动使用它实例化对象。
>
>最后，只有标准Java Bean属性可以，绑定到静态属性是不被支持的。

**构造器绑定**

上一节中的示例可以以不变的方式重写，如下例所示：
```
package com.example;

import java.net.InetAddress;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.ConstructorBinding;
import org.springframework.boot.context.properties.bind.DefaultValue;

@ConstructorBinding
@ConfigurationProperties("acme")
public class AcmeProperties {

    private final boolean enabled;

    private final InetAddress remoteAddress;

    private final Security security;

    public AcmeProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }

    public boolean isEnabled() { ... }

    public InetAddress getRemoteAddress() { ... }

    public Security getSecurity() { ... }

    public static class Security {

        private final String username;

        private final String password;

        private final List<String> roles;

        public Security(String username, String password,
                @DefaultValue("USER") List<String> roles) {
            this.username = username;
            this.password = password;
            this.roles = roles;
        }

        public String getUsername() { ... }

        public String getPassword() { ... }

        public List<String> getRoles() { ... }

    }

}
```
在此设置中，@ConstructorBinding注解用于指示应使用构造函数绑定。 这意味着绑定器将期望找到带有您希望绑定的参数的构造函数。

@ConstructorBinding类的嵌套成员（例如上例中的Security）也将通过其构造函数进行绑定。

可以使用@DefaultValue指定默认值，并且将应用相同的转换服务将String值强制为缺少属性的目标类型。

>要使用构造函数绑定，必须使用@EnableConfigurationProperties或配置属性扫描来启用该类。 您不能对通过常规Spring机制创建的bean使用构造函数绑定（例如@Component bean，通过@Bean方法创建的bean或使用@Import加载的bean）

>如果您的类具有多个构造函数，则还可以直接在应绑定的构造函数上使用@ConstructorBinding。

**启用@ConfigurationProperties注解的类型**

Spring Boot提供了绑定@ConfigurationProperties类型并将其注册为Bean的基础架构。 您可以逐类启用配置属性，也可以启用与组件扫描类似的方式进行配置属性扫描。

有时，用@ConfigurationProperties注释的类可能不适合扫描，例如，如果您正在开发自己的自动配置，或者想要有条件地启用它们。 在这些情况下，请使用@EnableConfigurationProperties注解指定要处理的类型列表。 可以在任何@Configuration类上完成此操作，如以下示例所示：
```
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(AcmeProperties.class)
public class MyConfiguration {
}
```

要使用配置属性扫描，请将@ConfigurationPropertiesScan批注添加到您的应用程序。 通常，它被添加到以@SpringBootApplication注释的主应用程序类中，但是可以将其添加到任何@Configuration类中。 默认情况下，将从声明注释的类的包中进行扫描。 如果要定义要扫描的特定程序包，可以按照以下示例所示进行操作：
```
@SpringBootApplication
@ConfigurationPropertiesScan({ "com.example.app", "org.acme.another" })
public class MyApplication {
}
```
>使用配置属性扫描或通过@EnableConfigurationProperties注册@ConfigurationProperties Bean时，该Bean具有常规名称：<prefix>-<fqn>，其中<prefix>是@ConfigurationProperties批注和<fqn>中指定的环境键前缀。 是Bean的完全限定名称。 如果注释不提供任何前缀，则仅使用Bean的完全限定名称。
>
>上例中的bean名称是acme-com.example.AcmeProperties。

我们建议@ConfigurationProperties仅处理环境，尤其不要从上下文中注入其他bean。 对于极端情况，可以使用setter注入或框架提供的任何* Aware接口（例如，需要访问Environment的EnvironmentAware）。 如果仍要使用构造函数注入其他bean，则必须使用@Component注释配置属性bean，并使用基于JavaBean的属性绑定。

**使用@ConfigurationProperties注解的类型**

这种配置样式与SpringApplication外部YAML配置特别有效，如以下示例所示：
```
# application.yml

acme:
    remote-address: 192.168.1.1
    security:
        username: admin
        roles:
          - USER
          - ADMIN

# additional configuration as required
```
要使用@ConfigurationProperties Bean，可以像其他任何Bean一样注入它们，如以下示例所示：
```
@Service
public class MyService {

    private final AcmeProperties properties;

    @Autowired
    public MyService(AcmeProperties properties) {
        this.properties = properties;
    }

    //...

    @PostConstruct
    public void openConnection() {
        Server server = new Server(this.properties.getRemoteAddress());
        // ...
    }

}
```
>使用@ConfigurationProperties还可让您生成元数据文件，IDE可以使用这些元数据文件为您自己的键提供自动完成功能。 有关详细信息，请参见[附录](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#configuration-metadata)。

**第三方配置**

除了使用@ConfigurationProperties注释类外，还可以在公共@Bean方法上使用它。 当您要将属性绑定到控件之外的第三方组件时，这样做特别有用。

要从Environment属性配置Bean，请将@ConfigurationProperties添加到其Bean注册中，如以下示例所示：
```
@ConfigurationProperties(prefix = "another")
@Bean
public AnotherComponent anotherComponent() {
    ...
}
```
用"another"前缀定义的任何JavaBean属性都以类似于前面的AcmeProperties示例的方式映射到该AnotherComponent bean。

**宽松绑定**

Spring Boot使用一些宽松的规则将Environment属性绑定到@ConfigurationProperties bean，因此环境属性名称和bean属性名称之间不需要完全匹配。 有用的常见示例包括破折号分隔的环境属性（例如，context-path绑定到contextPath）和大写的环境属性（例如PORT绑定到端口）。

例如，考虑以下@ConfigurationProperties类：
```
@ConfigurationProperties(prefix="acme.my-project.person")
public class OwnerProperties {

    private String firstName;

    public String getFirstName() {
        return this.firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

}
```
使用前面的代码，可以全部使用以下属性名称：

**Table 5. relaxed binding**

| Property |Note     |
| :------------- | :------------- |
| acme.my-project.person.first-name       | Kebab大小写，建议在.properties和.yml文件中使用
|acme.myProject.person.firstName|标准驼峰语法
|acme.my_project.person.first_name|下划线表示法，是.properties和.yml文件中使用的另一种格式。
|ACME_MYPROJECT_PERSON_FIRSTNAME|大写格式，使用系统环境变量时建议使用。
>注释的前缀值必须为小写kebab格式,并用-分隔，例如acme.my-project.person。

绑定到Map属性时，如果键包含小写字母数字字符或-以外的任何其他字符，则需要使用方括号表示法，以便保留原始值。 如果键没有被[]包围，则所有非字母数字或-的字符都将被删除。 例如，考虑将以下属性绑定到Map：
```
acme:
  map:
    "[/key1]": value1
    "[/key2]": value2
    /key3: value3
```
上面的属性将以/ key1，/ key2和key3作为map中的键绑定到Map。
>对于YAML文件，方括号必须用引号引起来，以便正确解析键值。

**合并复杂属性**

如果在多个位置配置了列表，则通过替换整个列表来进行覆盖。

例如，假设MyPojo对象的名称和描述属性默认为空。 下面的示例从AcmeProperties公开MyPojo对象的列表：
```
@ConfigurationProperties("acme")
public class AcmeProperties {

    private final List<MyPojo> list = new ArrayList<>();

    public List<MyPojo> getList() {
        return this.list;
    }

}
```
考虑以下配置：
```
acme:
  list:
    - name: my name
      description: my description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
```
如果dev配置文件未处于活动状态，则AcmeProperties.list包含一个MyPojo条目，如先前所定义。 但是，如果启用了dev配置文件，则该列表仍仅包含一个条目（名称为我的另一个名称，并且描述为null）。 此配置不会将第二个MyPojo实例添加到列表中，并且不会合并项目。

在多个配置文件中指定列表时，将使用优先级最高的列表（并且仅使用那个列表）。 考虑以下示例：
```
acme:
  list:
    - name: my name
      description: my description
    - name: another name
      description: another description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
```
在前面的示例中，如果dev配置文件处于活动状态，则AcmeProperties.list包含一个MyPojo条目（其名称为my的另一个名称，以及为null的描述。对于YAML，可以使用逗号分隔的列表和YAML列表来完全覆盖列表的内容。

对于Map属性，可以绑定从多个来源绘制的属性值。 但是，对于多个源中的同一属性，将使用优先级最高的属性。 下面的示例从AcmeProperties公开Map <String，MyPojo>：
```
@ConfigurationProperties("acme")
public class AcmeProperties {

    private final Map<String, MyPojo> map = new HashMap<>();

    public Map<String, MyPojo> getMap() {
        return this.map;
    }

}
```
考虑以下配置：
```
acme:
  map:
    key1:
      name: my name 1
      description: my description 1
---
spring:
  profiles: dev
acme:
  map:
    key1:
      name: dev name 1
    key2:
      name: dev name 2
      description: dev description 2
```
如果开发人员配置文件未处于活动状态，则AcmeProperties.map包含一个键为key1的条目（名称为我的名字1，描述为my description 1）。但是，如果启用了开发配置文件，则map包含两个条目，其中键为key1（名称为dev name 1，其描述为my description 1）和key2（名称为dev name 2，其描述为dev description 2）。

>前述合并规则不仅适用于YAML文件，而且适用于所有属性源中的属性。

**属性转换**

当Spring Boot绑定到@ConfigurationProperties bean时，它尝试将外部应用程序属性强制为正确的类型。 如果需要自定义类型转换，则可以提供一个ConversionService bean（具有一个名为conversionService的bean）或自定义属性编辑器（通过CustomEditorConfigurer bean）或自定义Converters（具有定义为@ConfigurationPropertiesBinding的bean定义）。

>由于在应用程序生命周期中非常早就请求了此bean，因此请确保限制您的ConversionService使用的依赖项。 通常，您需要的任何依赖项在创建时可能都没有完全初始化。 如果配置键强转不需要自定义ConversionService，而仅依赖于具有@ConfigurationPropertiesBinding限定的自定义转换器，则可能需要重命名自定义ConversionService。

*转换Duration*

Spring Boot为表达持续时间提供了专门的支持。 如果公开java.time.Duration属性，则应用程序属性中的以下格式可用：
- 常规的长表示形式（使用毫秒作为默认单位，除非已指定@DurationUnit）
- java.time.Duration使用的标准ISO-8601格式
- 值和单位相结合的更易读的格式（例如10s表示10秒）

考虑下面的格式:
```
@ConfigurationProperties("app.system")
public class AppSystemProperties {

    @DurationUnit(ChronoUnit.SECONDS)
    private Duration sessionTimeout = Duration.ofSeconds(30);

    private Duration readTimeout = Duration.ofMillis(1000);

    public Duration getSessionTimeout() {
        return this.sessionTimeout;
    }

    public void setSessionTimeout(Duration sessionTimeout) {
        this.sessionTimeout = sessionTimeout;
    }

    public Duration getReadTimeout() {
        return this.readTimeout;
    }

    public void setReadTimeout(Duration readTimeout) {
        this.readTimeout = readTimeout;
    }

}
```
要指定30秒的会话超时，则30，PT30S和30s都是等效的。 可以使用以下任意形式指定500ms的读取超时：500，PT0.5S和500ms。

您也可以使用任何受支持的单位。 这些是：
- ns,纳秒
- us,微秒
- ms,毫秒
- s,秒
- m,分钟
- h,小时
- d,天

默认单位是毫秒，可以使用@DurationUnit覆盖，如上面的示例所示。

>如果您要从仅使用Long表示持续时间的先前版本进行升级，请确保在转换为Duration的旁边没有毫秒数的情况下定义单位（使用@DurationUnit）。 这样做可以提供透明的升级路径，同时支持更丰富的格式。

*转换数据大小*

Spring Framework具有DataSize值类型，以字节为单位表示大小。 如果公开DataSize属性，则应用程序属性中的以下格式可用：
- 常规的长表示形式（除非已指定@DataSizeUnit，否则使用字节作为默认单位）
- 值和单位耦合在一起的更易读的格式（例如10MB表示10兆字节）

考虑下面的格式:
```
@ConfigurationProperties("app.io")
public class AppIoProperties {

    @DataSizeUnit(DataUnit.MEGABYTES)
    private DataSize bufferSize = DataSize.ofMegabytes(2);

    private DataSize sizeThreshold = DataSize.ofBytes(512);

    public DataSize getBufferSize() {
        return this.bufferSize;
    }

    public void setBufferSize(DataSize bufferSize) {
        this.bufferSize = bufferSize;
    }

    public DataSize getSizeThreshold() {
        return this.sizeThreshold;
    }

    public void setSizeThreshold(DataSize sizeThreshold) {
        this.sizeThreshold = sizeThreshold;
    }

}
```
若要指定10MB的缓冲区大小，则10和10MB是等效的。256个字节的大小阈值可以指定为256或256B。

您也可以使用任何受支持的单位。 这些是：
- B, bytes
- KB, kilobytes
- MB, megabytes
- GB, gigabytes
- TB, terabytes

默认单位是字节，可以使用@DataSizeUnit覆盖，如上面的示例所示。

>如果您要从仅使用Long表示大小的先前版本进行升级，请确保在切换到DataSize旁边没有字节的情况下定义单位（使用@DataSizeUnit）。 这样做可以提供透明的升级路径，同时支持更丰富的格式。

**@ConfigurationProperties验证**

每当使用Spring的@Validated注解对@ConfigurationProperties类进行注释时，Spring Boot就会尝试对其进行验证。 您可以在配置类上直接使用JSR-303 javax.validation约束注释。 为此，请确保在类路径上有兼容的JSR-303实现，然后将约束注释添加到字段中，如以下示例所示：
```
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

    @NotNull
    private InetAddress remoteAddress;

    // ... getters and setters

}
```
>您还可以通过使用@Validated注释创建配置属性的@Bean方法来触发验证。

为了确保始终为嵌套属性触发验证，即使未找到任何属性，也必须使用@Valid注释关联的字段。 以下示例基于前面的AcmeProperties示例：
```
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

    @NotNull
    private InetAddress remoteAddress;

    @Valid
    private final Security security = new Security();

    // ... getters and setters

    public static class Security {

        @NotEmpty
        public String username;

        // ... getters and setters

    }

}
```

您还可以通过创建一个名为configurationPropertiesValidator的bean定义来添加自定义的Spring Validator。 @Bean方法应声明为静态。 配置属性验证器是在应用程序生命周期的早期创建的，并且将@Bean方法声明为static可以使创建该Bean而不必实例化@Configuration类。 这样做避免了由早期实例化引起的任何问题。

>spring-boot-actuator模块包括一个公开所有@ConfigurationProperties bean的端点。 将您的Web浏览器指向/ actuator / configprops或使用等效的JMX端点。 有关详细信息，请参见“[生产就绪功能](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#production-ready-endpoints)”部分。

**@ConfigurationProperties与@Value**

@Value批注是核心容器功能，它没有提供与类型安全的配置属性相同的功能。 下表总结了@ConfigurationProperties和@Value支持的功能：

Feature|@ConfigurationProperties|@Value
-|-|-
Relaxed binding|Yes|No
Meta-data support|Yes|No
SpEL evaluation|No|Yes

如果您为自己的组件定义了一组配置键，我们建议您将它们组合在以@ConfigurationProperties注释的POJO中。 您还应该意识到，由于@Value不支持宽松的绑定，因此如果您需要使用环境变量来提供值，则它不是一个很好的选择。

最后，尽管您可以在@Value中编写SpEL表达式，但不会从应用[程序属性文件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features-external-config-application-property-files)中处理此类表达式。

# 4.3 Profiles

Spring Profiles提供了一种隔离应用程序配置部分并使之仅在某些环境中可用的方法。 可以使用@Profile标记任何@ Component，@ Configuration或@ConfigurationProperties，以限制其加载时间，如以下示例所示：
```
@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```
> 如果@ConfigurationProperties Bean是通过@EnableConfigurationProperties而非自动扫描注册的，则需要在具有@EnableConfigurationProperties注解的@Configuration类上指定@Profile注解。在扫描@ConfigurationProperties的情况下，可以在@ConfigurationProperties类本身上指定@Profile。

你可以通过spring.profiles.active环境属性来指定哪个profile处于激活状态，你可以通过本章前面介绍的任何方式指定属性。 例如，你可以将其包含在application.properties中，如以下示例所示：
```
spring.profiles.avtive=dev,hsqldb
```
您也可以使用以下开关在命令行上指定它：--spring.profiles.avtive=dev,hsqldb

## 4.3.1

spring.profiles.active属性遵循与其他属性相同的排序规则：最高的PropertySource获胜。 这意味着您可以在application.properties中指定活动配置文件，然后使用命令行开关替换它们。

有时，将特定配置文件的属性添加到活动配置文件而不是替换它们是有用的。 spring.profiles.include属性可用于无条件添加活动配置文件。 SpringApplication入口点还具有Java API，用于设置其他配置文件（即，在由spring.profiles.active属性激活的配置文件之上）。 请参阅SpringApplication中的setAdditionalProfiles（）方法。

例如，使用开关--spring.profiles.active = prod运行具有以下属性的应用程序时，proddb和prodmq配置文件也会被激活：
```
---
my.property: fromyamlfile
---
spring.profiles: prod
spring.profiles.include:
  - proddb
  - prodmq
```

>请记住，可以在YAML文档中定义spring.profiles属性，以确定何时将该特定文档包括在配置中。 有关更多详细信息，请参见根据环境更改配置。

## 4.3.2 以编程方式设置配置文件

您可以在应用程序运行之前通过调用SpringApplication.setAdditionalProfiles（…）以编程方式设置活动配置文件。 也可以使用Spring的ConfigurableEnvironment接口来激活配置文件。

## 4.3.3 特定配置的配置文件

application.properties（或application.yml）和通过@ConfigurationProperties引用的文件的特定配置文件的变体都被视为文件并已加载。 有关详细信息，请参见“[特定配置文件的属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features-external-config-profile-specific-properties)”。
