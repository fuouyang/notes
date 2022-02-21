
# 4.4 日志

Spring Boot使用[Commons Logging](https://commons.apache.org/logging)进行所有内部日志记录，但是使底层日志实现保持打开状态。提供了[Java Util Logging](https://docs.oracle.com/javase/8/docs/api//java/util/logging/package-summary.html)，[Log4J2](https://logging.apache.org/log4j/2.x/)和[Logback](https://logback.qos.ch/)的默认配置。 在每种情况下，记录器都已预先配置为使用控制台输出，同时还提供可选文件输出。

默认情况下，如果使用"Starters"，则使用Logback进行日志记录。 还包括适当的Logback路由，以确保使用Java Util Logging，Commons Logging，Log4J或SLF4J的依赖库都能正常工作。

>有许多可用于Java的日志记录框架。 如果上面的列表看起来令人困惑，请不要担心。 通常，您不需要更改日志记录依赖项，并且Spring Boot默认值可以正常工作。

>将应用程序部署到servlet容器或应用程序服务器时，通过Java Util Logging API执行的日志记录不会路由到应用程序的日志中。 这样可以防止容器或其他已部署到容器中的应用程序执行的日志记录出现在应用程序的日志中。

## 4.4.1 日志格式

Spring Boot的默认日志输出类似于以下示例：
```
2019-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2019-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2019-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```

输出以下项目：
- 日期和时间：毫秒精度，易于分类。
- 日志级别：ERROR, WARN, INFO, DEBUG,或者TRACE。
- 进程ID。
- 一个---分隔符，用于区分实际日志消息的开始。
- 线程名称：用方括号括起来（对于控制台输出可能会被截断）。
- 记录器名称：通常是类名（经常是缩写）。
- 日志信息

>Logback没有FATAL级别，对应为ERROR

## 4.4.2 控制台输出

默认日志配置在写入消息时将消息回显到控制台。 默认情况下，将记录ERROR级，WARN级和INFO级消息。 您还可以通过使用--debug标志启动应用程序来启用“调试”模式。
```
$ java -jar myapp.jar --debug
```

>你也可以在application.properties中指定debug=true

启用调试模式后，将配置一些核心记录器（嵌入式容器，Hibernate和Spring Boot）以输出更多信息。 启用调试模式不会将您的应用程序配置为记录所有具有DEBUG级别的消息。

另外，您可以通过使用--trace标志（或application.properties中的trace = true）启动应用程序来启用“跟踪”模式。 这样做可以为某些核心记录器（嵌入式容器，Hibernate模式生成以及整个Spring产品组合）启用跟踪记录。

**彩色代码输出**

如果您的终端支持ANSI，则使用彩色输出来提高可读性。 您可以将spring.output.ansi.enabled设置为[支持的值](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/ansi/AnsiOutput.Enabled.html)，以覆盖自动检测。

使用％clr转换字配置颜色编码。 转换器以最简单的形式根据对数级别为输出着色，如以下示例所示：
```
%clr(%5p)
```

下表描述了日志级别到颜色的映射：

Level|Color
-|-
FATAL|Red
ERROR|Red
WARN|Yellow
INFO|Green
DEBUG|Green
TRACE|Green

另外，你可以为转换提供一个选项来指定颜色和样式。例如，使文字变黄，使用下面的配置：
```
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

支持下列颜色样式：
- blue
- cyan
- faint
- green
- magenta
- red
- yellow

## 4.4.3 文件输出

默认情况下，Spring Boot只记录到控制台，不写日志文件。 如果除了控制台输出外还想写日志文件，则需要设置logging.file.name或logging.file.path属性（例如，在application.properties中）。

下表显示了logging.\*属性如何一起使用：

**Table 7 日志属性**

logging.file.name|logging.file.path|Example|descripton
-|-|-|-
 (none)| (none)||仅控制台记录。
 Specific file|(none)|my.log|写入指定的日志文件。名称可以是确切的位置，也可以相对于当前目录。
(none)|Specfic directory|/var/log|将spring.log写入指定目录。名称可以是确切的位置，也可以相对于当前目录。

日志文件达到10 MB时会循环，并且与控制台输出一样，默认情况下会记录ERROR级别，WARN级别和INFO级别的消息。 可以使用logging.file.max-size属性更改大小限制。 除非已设置logging.file.max-history属性，否则默认情况下将保留最近7天的循环日志文件。 可以使用logging.file.total-size-cap限制日志归档文件的总大小。 当日志归档的总大小超过该阈值时，将删除备份。 要在应用程序启动时强制清除日志存档，请使用logging.file.clean-history-on-start属性。

>日志记录属性独立于实际的日志记录基础结构。因此，指定的配置键（例如Logback的logback.configurationFile）不是由Spring Boot管理的。

## 4.4.4 日志级别

通过使用logging.level.<logger-name> = <level>，可以在Spring环境中（例如，在application.properties中）设置所有受支持的日志记录器级别，其中level是TRACE，DEBUG，INFO，WARN，ERROR,FATAL或OFF。可以使用logging.level.root配置根记录器。

以下示例显示了application.properties中的潜在日志记录设置：
```
logging.level.root=warn
logging.level.org.springframework.web=debug
logging.level.org.hibernate=error
```

也可以使用环境变量设置日志记录级别。 例如，LOGGING_LEVEL_ORG_SPRINGFRAMEWORK_WEB = DEBUG会将org.springframework.web设置为DEBUG。

>以上方法仅适用于程序包级别的日志记录。 由于宽松的绑定总是将环境变量转换为小写，因此无法以这种方式为单个类配置日志记录。 如果需要为类配置日志记录，则可以使用[SPRING_APPLICATION_JSON](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features-external-config-application-json)变量。

## 4.4.5 日志分组

通常将相关记录器组合在一起可以很有用，以便可以同时配置它们。 例如，您可能通常会更改所有与Tomcat相关的记录器的日志记录级别，但是您不容易记住顶级软件包。

为了解决这个问题，Spring Boot允许您在Spring Environment中定义日志记录组。 例如，以下是通过将“ tomcat”组添加到application.properties来定义它的方法：
```
logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
```

定义后，您可以使用一行更改该组中所有记录器的级别：
```
logging.level.tomcat=TRACE
```

Spring Boot包含以下预定义的日志记录组，它们可以直接使用：

Name|Loggers
-|-
web|org.springframework.core.codec, org.springframework.http, org.springframework.web, org.springframework.boot.actuate.endpoint.web, org.springframework.boot.web.servlet.ServletContextInitializerBeans
sql|org.springframework.jdbc.core, org.hibernate.SQL, org.jooq.tools.LoggerListener

## 4.4.6 自定义日志配置

可以通过在类路径上包括适当的库来激活各种日志记录系统，并可以通过在类路径的根目录中或在以下Spring Environment属性：logging.config指定的位置中提供适当的配置文件来进一步自定义日志文件。

您可以通过使用org.springframework.boot.logging.LoggingSystem系统属性来强制Spring Boot使用特定的日志记录系统。 该值应该是LoggingSystem实现的完全限定的类名。 您也可以使用none完全禁用Spring Boot的日志记录配置。

>由于日志记录是在创建ApplicationContext之前初始化的，因此无法从Spring @Configuration文件中的@PropertySources控制日志记录。 更改日志记录系统或完全禁用它的唯一方法是通过系统属性

根据您的日志记录系统，将加载以下文件：

Logging System|Customization
-|-
Logback|logback-spring.xml, logback-spring.groovy, logback.xml, 或 logback.groovy
Log4J2|log4j2-spring.xml or log4j2.xml
JDK(Java Util Logging)|logging.properties

>如果可能，我们建议您在日志配置中使用-spring变体（例如，logback-spring.xml而不是logback.xml）。 如果使用标准配置位置，Spring将无法完全控制日志初始化。

>从“可执行jar”运行时，Java Util Logging存在一些已知的类加载问题，这些问题会引起问题。 我们建议您尽可能从“可执行jar”运行时避免使用它。

为了帮助定制，将其他一些属性从Spring Environment转移到System属性，如下表所述：

Spring Environment|System Property|Comments
:-|:-|:-
logging.exception-conversion-word|LOG_EXCEPTION_CONVERSION_WORD|记录异常时使用的转换字
logging.file.clean-history-on-start|LOG_FILE_CLEAN_HISTORY_ON_START|是否在启动时清除存档日志文件(如果启用了LOG_FILE)(仅Logback默认配置支持)
logging.file.name|LOG_FILE|如果定义，它将在默认日志配置中使用。
logging.file.max-asize|LOG_FILE_MAX_SIZE|日志文件的最大大小(如果启用了LOG_FILE)(仅Logback默认配置支持)
logging.file.max-history|LOG_FILE_MAX_HISTORY|要保留的最大归档日志文件数(如果启用了LOG_FILE)(仅Logback默认配置支持)
logging.file.path|LOG_PATH|如果定义，它将在默认日志配置中使用。
logging.file.total-size-cap|LOG_FILE_TOTAL_SIZE_CAP|要保留的日志备份的总大小(如果启用了LOG_FILE)(仅Logback默认配置支持)
logging.pattern.console|CONSOLE_LOG_PATTERN|控制台上要使用的日志模式(stdout)(仅Logback默认配置支持)
logging.pattern.dateformat|LOG_DATEFORMAT_PATTERN|记录日期格式的附加模式(仅Logback默认配置支持)
logging.pattern.file|FILE_LOG_PATTERN|文件中使用的日志模式(如果启用了LOG_FILE)(仅Logback默认配置支持)
logging.pattern.level|LOG_LEVEL_PATTERN|呈现日志级别时使用的格式(默认为％5p)(仅Logback默认配置支持)
logging.pattern.rolling-file-name|ROLLING_FILE_NAME_PATTERN|滚动日志文件名的模式(默认$ {LOG_FILE}.％d {yyyy-MM-dd}.％i.gz)(仅Logback默认配置支持)
PID|PID|当前进程ID（如果可能，并且尚未将其定义为OS环境变量时，将被发现）

所有受支持的日志记录系统在解析其配置文件时都可以查阅系统属性。 有关示例，请参见spring-boot.jar中的默认配置：
- [Logback](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)
- [Log4J2](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml)
- [Java Util logging](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/java/logging-file.properties)

>如果要在日志记录属性中使用占位符，则应使用Spring Boot的语法而不是基础框架的语法。 值得注意的是，如果使用Logback，则应使用：作为属性名称与其默认值之间的分隔符，而不应使用:-。

>您可以通过仅覆盖LOG_LEVEL_PATTERN（或带Logback的logging.pattern.level）来将MDC和其他临时内容添加到日志行。 例如，如果使用logging.pattern.level = user：％X {user}％5p，则默认日志格式包含“ user”的MDC条目（如果存在），如以下示例所示。
>
2019-08-30 12：30：04.031用户：某人INFO 22174-[[nio-8080-exec-0] demo.Controller
处理已认证的请求

## 4.4.7 Logback扩展

Spring Boot包含许多Logback扩展，可以帮助进行高级配置。 您可以在logback-spring.xml配置文件中使用这些扩展名。

>由于标准logback.xml配置文件加载得太早，因此无法在其中使用扩展名。 您需要使用logback-spring.xml或定义logging.config属性。

>这些扩展不能与Logback的配置扫描一起使用。 如果尝试这样做，则对配置文件进行更改将导致类似于以下记录之一的错误：

```
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProperty], current ElementPath is [[configuration][springProperty]]
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProfile], current ElementPath is [[configuration][springProfile]]
```

**特定文件配置**

通过<springProfile>标记，您可以根据活动的Spring概要文件有选择地包括或排除配置部分。 在<configuration>元素内的任何位置都支持概要文件部分。 使用name属性指定哪个配置文件接受配置。 <springProfile>标记可以包含简单的配置文件名称（例如，暂存）或配置文件表达式。 配置文件表达式允许表达更复杂的配置文件逻辑，例如生产＆（eu-central | eu-west）。 有关更多详细信息，请参阅[参考指南](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java)。 以下清单显示了三个样本概要文件：
```
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

**环境属性**

<springProperty>标记使您可以从Spring Environment中公开属性，以在Logback中使用。 如果要访问Logback配置中的application.properties文件中的值，则这样做很有用。 该标签的工作方式类似于Logback的标准<property>标签。 但是，您没有指定直接值，而是指定了属性的来源（来自环境）。 如果需要将属性存储在本地范围以外的其他位置，则可以使用scope属性。 如果需要后备值（如果未在环境中设置该属性），则可以使用defaultValue属性。 以下示例显示如何公开在Logback中使用的属性：
```
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
        defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
    <remoteHost>${fluentHost}</remoteHost>
    ...
</appender>
```

>	必须在kebab情况下指定来源（例如my.property-name）。 但是，可以使用宽松的规则将属性添加到环境中。

# 4.5  国际化

Spring Boot支持本地化消息，因此您的应用程序可以迎合不同语言首选项的用户。 默认情况下，Spring Boot在类路径的根目录下查找消息资源包的存在。
>当已配置资源包的默认属性文件可用时（即默认情况下为messages.properties），将应用自动配置。 如果您的资源包仅包含特定于语言的属性文件，则需要添加默认文件。 如果找不到与任何配置的基本名称匹配的属性文件，将没有自动配置的MessageSource。

可以使用spring.messages命名空间配置资源包的基本名称以及其他几个属性，如以下示例所示：
```
spring.messages.basename=messages,config.i18n.messages
spring.messages.fallback-to-system-locale=false
```

>spring.messages.basename支持以逗号分隔的位置列表，可以是包限定符，也可以是从类路径根目录解析的资源。

有关更多受支持的选项，请参见[MessageSourceProperties](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/context/MessageSourceProperties.java)。

# 4.6 JSON

Spring Boot提供了与三个JSON映射库的集成：
- Gson
- Jackson
- JSON-B

Jackson是首选的默认库。

## 4.6.1 Jackson

提供了Jackson的自动配置，并且Jackson是spring-boot-starter-json的一部分。 当Jackson放在类路径上时，将自动配置ObjectMapper Bean。 提供了几个配置属性，用于[自定义ObjectMapper的配置](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#howto-customize-the-jackson-objectmapper)。

## 4.6.2 GSon

提供了Gson的自动配置。 当Gson在类路径上时，将自动配置Gson bean。 提供了几个spring.gson.\*配置属性用于自定义配置。 为了获得更多控制权，可以使用一个或多个GsonBuilderCustomizer bean。

## 4.6.3 JSON-B

提供了JSON-B的自动配置。 当JSON-B API和实现位于类路径上时，将自动配置Jsonb bean。 首选的JSON-B实现是提供依赖管理的Apache Johnzon。
