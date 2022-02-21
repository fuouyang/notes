[全集](https://www.jianshu.com/p/a661eeaa2935)

本节将详细介绍如何使用Spring Boot, 它包含了诸如构建系统，自动配置以及怎么运行你的程序这些主题。我们还将介绍一些Spring Boot的最佳实践。尽管Spring Boot没什么特别的（它只是您可以使用的另一个库），但是有一些建议可以使您的开发过程更轻松一些。

如果你是从Spring Boot开始的，那么在深入了解之前，您应该阅读[入门指南](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#getting-started)

# 3.1 构建系统

 我们强烈推荐您选择一个支持依赖管理的构建系统，可以让你使用发布到Maven中央仓库的组件。我们建议您使用Maven或Gradle。可以使Spring Boot与其它构建系统(例如Ant)一起使用，但是它们并没有得到很好的支持。

 ## 3.3.1 依赖管理

 每个Spring Boot版本都提供了他所支持的依赖关系列表，实践中，您不用在您的构建配置上提供任何依赖的版本，因为Spring Boot已经帮你管理了。当你升级Spring Boot时，它的依赖也一并升级了。
 > 当你需要时，你仍然可以声明版本来覆盖Spring Boot推荐的版本。

精选列表包含可与Spring Boot一起使用的所有spring模块以及完善的第三方库列表。 该列表是可与Maven和Gradle一起使用的标准依赖清单（spring-boot-dependencies）。

> 每个Spring Boot稳定版本都和一个Spring Framework基础版本相关联，我们强烈建议您不要声明它的版本。

## 3.1.2 Maven
Maven用户可以继承spring-boot-starter-parent来获取一些合理的默认值，该Parent项目提供了下列功能：
- Java 1.8作为默认编译级别
- UTF-8编码
- 一个继承自spring-boot-dependencies pom文件的依赖管理块，它管理了通用依赖的版本。这个依赖管理让你省略在pom文件使用的那些依赖的<version>标签
- 以repackage来实现重新打包目标的执行
- 智能的文件过滤
- 合理的插件配置（exec插件，Git提交Id和Shade）
- 对application.properties和application.yml进行智能的资源过滤，包括特定于配置文件的文件（例如，application-dev.properties和application-dev.yml）

注意，由于application.properties和application.yml文件接受Spring样式的占位符（${...}）,因此Maven过滤已更改为使用@...@占位符（您可以通过设置一个名为resource.delemiter的Maven属性来覆盖它）

**继承启动器**

通过继承spring-boot-starter-parent来配置你的项目，查看下面列出的parent:
```
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.6.RELEASE</version>
</parent>
```
> 你应该只在这个依赖中声明Spring Boot的版本。如果你导入其它的启动器，你可以安全的省略版本号

使用该设置，您还可以通过覆盖自己项目中的属性来覆盖各个依赖项。 例如，要升级到另一个Spring Data发布系列，您可以将以下内容添加到pom.xml中：
```
<properties>
    <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```
> 查看 spring-boot-dependencies [pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml)文件列出的属性

**不使用父POM来使用Spring Boot**

并不是每个人都喜欢继承spring-boot-starter-parent的POM文件。您可能需要使用自己的公司标准parent,或者可能希望显示声明所有Maven配置。

如果你不想使用spring-boot-starter-parent,你仍然可以通过使用一个 scope=import 的依赖来得到依赖管理的益处，如下所示：
```
<dependencyManagement>
    <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
上面的示例配置没有允许您通过使用属性来覆盖各个依赖项。为了获得相同的结果，需要在spring-boot-dependencies条目之前的项目的dependencyManagement中添加一个元素。例如，要升级到另一个Spring Data发布系列，您可以添加下面的元素到你的pom.xml文件中：
```
<dependencyManagement>
    <dependencies>
        <!-- Override Spring Data release train provided by Spring Boot -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-releasetrain</artifactId>
            <version>Fowler-SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
> 在生疏示例总，我们指定了一个BOM,但是任何依赖项类型都可以以相同方式覆盖

**使用Spring Boot Maven Plugin**

Spring Boot包含了一个将项目打包成可执行jar的Maven插件，若果要使用它就将这个插件像下面的示例一样添加到你的<plugins>中：
```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
> 如果你使用Spring Boot starter的父Pom,您只需要添加这个插件，并不需要任何配置除非你想改变在父Pom中定义的设置

## 3.1.3 Gradle
要将Spring Boot和Gradle一起使用，请参考Spring Boot的Gradle插件文档：
- Reference( [HTML](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/html/) 和 [PDF](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/pdf/spring-boot-gradle-plugin-reference.pdf) )
- [API](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/api/)
## 3.1.4 Ant
可以使用Apache Ant + Ivy构建Spring Boot项目。 spring-boot-antlib“ AntLib”模块也可用于帮助Ant创建可执行的jar。

要声明依赖关系，典型的ivy.xml文件看起来类似于以下示例：
```
<ivy-module version="2.0">
    <info organisation="org.springframework.boot" module="spring-boot-sample-ant" />
    <configurations>
        <conf name="compile" description="everything needed to compile this module" />
        <conf name="runtime" extends="compile" description="everything needed to run this module" />
    </configurations>
    <dependencies>
        <dependency org="org.springframework.boot" name="spring-boot-starter"
            rev="${spring-boot.version}" conf="compile" />
    </dependencies>
</ivy-module>
```
典型的build.xml类似于以下示例：
```
<project
    xmlns:ivy="antlib:org.apache.ivy.ant"
    xmlns:spring-boot="antlib:org.springframework.boot.ant"
    name="myapp" default="build">

    <property name="spring-boot.version" value="2.2.6.RELEASE" />

    <target name="resolve" description="--> retrieve dependencies with ivy">
        <ivy:retrieve pattern="lib/[conf]/[artifact]-[type]-[revision].[ext]" />
    </target>

    <target name="classpaths" depends="resolve">
        <path id="compile.classpath">
            <fileset dir="lib/compile" includes="*.jar" />
        </path>
    </target>

    <target name="init" depends="classpaths">
        <mkdir dir="build/classes" />
    </target>

    <target name="compile" depends="init" description="compile">
        <javac srcdir="src/main/java" destdir="build/classes" classpathref="compile.classpath" />
    </target>

    <target name="build" depends="compile">
        <spring-boot:exejar destfile="build/myapp.jar" classes="build/classes">
            <spring-boot:lib>
                <fileset dir="lib/runtime" />
            </spring-boot:lib>
        </spring-boot:exejar>
    </target>
</project>
```
> 如果您不想使用spring-boot-antlib模块，请参阅[不使用spring-boot-antlib“如何”而从Ant构建可执行档案](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#howto-build-an-executable-archive-with-ant)。

## 3.1.5 Starters
Starters是一组方便的依赖项描述符，您可以在应用程序中包括它们。 您可以一站式购买所需的所有Spring和相关技术，而不必搜寻示例代码和依赖描述符的复制粘贴负载。 例如，如果要开始使用Spring和JPA进行数据库访问，请在项目中包括spring-boot-starter-data-jpa依赖项。

Starters包含许多启动项目并快速运行所需的依赖项，并且具有一组受支持的受管传递性依赖项
> 名称包含什么<br/>
 所有官方Starters都遵循类似的命名方式。 spring-boot-starter-\*，其中\*是特定类型的应用程序。 这种命名结构旨在在您需要寻找入门者时提供帮助。 许多IDE中的Maven集成使您可以按名称搜索依赖项。 例如，在安装了适当的Eclipse或STS插件的情况下，您可以在POM编辑器中按ctrl-space并键入“ spring-boot-starter”以获取完整列表。

Spring Boot在org.springframework.boot组下提供了以下应用程序Starter:

**Table 1. Spring Boot应用程序Starter**

| Name                                       | Description                                                                                                   | Pom                                                                                                                                                                    |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| spring-boot-starter                        | 核心Starter,包含自动配置支持，日志和YAML                                                                      | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter/pom.xml)                         |
| spring-boot-starter-activemq               | 使用Apache ActiveMQ的JMS消息的Starter                                                                         | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-activemq/pom.xml)                |
| spring-boot-starter-amqp                   | 使用Spring AMQP和Rabbit MQ的Starter                                                                           | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-amqp/pom.xml)                    |
| spring-boot-starter-aop                    | 使用Spring AOP和AspectJ进行面向方面编程的Starter                                                              | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-aop/pom.xml)                     |
| spring-boot-starter-artemis                | 使用Apache Artemis的JMS消息传递Starter                                                                        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-artemis/pom.xml)                 |
| spring-boot-starter-batch                  | 使用Spring Batch的Starter                                                                                     | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-batch/pom.xml)                   |
| spring-boot-starter-cache                  | 使用Spring Framework的缓存支持的Starter                                                                       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cache/pom.xml)                   |
| spring-boot-starter-cloud-connectors       | 使用Spring Cloud Connectors的Starter，可简化与Cloud Foundry和Heroku等云平台中服务的连接。不推荐使用Java CFEnv | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cloud-connectors/pom.xml)        |
| spring-boot-starter-data-cassandra         | 使用Cassandra分布式数据库和Spring Data Cassandra Reactive的Starter                                            | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra-reactive/pom.xml) |
| spring-boot-starter-data-couchbase         | 使用Couchbase面向文档的数据库和Spring Data Couchbase的入门                                                    | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase/pom.xml)          |
| spring-boot-starter-data-elasticsearch     | 使用Elasticsearch搜索和分析引擎以及Spring Data Elasticsearch的Starter                                         | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-elasticsearch/pom.xml)      |
| spring-boot-starter-data-jdbc              | 使用Spring Data JDBC的Starter                                                                                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jdbc/pom.xml)               |
| spring-boot-starter-data-jpa               | 将Spring Data JPA与Hibernate结合使用的Starter                                                                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jpa/pom.xml)                |
| spring-boot-starter-data-ldap              | 使用Spring Data LDAP的Starter                                                                                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-ldap/pom.xml)               |
| spring-boot-starter-data-mongodb           | 使用MongoDB面向文档的数据库和Spring Data MongoDB的Starter                                                     | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb/pom.xml)            |
| spring-boot-starter-data-mongodb-reactive  | 使用MongoDB面向文档的数据库和Spring Data MongoDB Reactive的Starter                                            | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb-reactive/pom.xml)   |
| spring-boot-starter-data-neo4j             | 使用Neo4j图形数据库和Spring Data Neo4j的Starter                                                               | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-neo4j/pom.xml)              |
| spring-boot-starter-data-redis             | 使用Redis键值数据存储与Spring Data Redis和Lettuce客户端的Starter                                              | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis/pom.xml)              |
| spring-boot-starter-data-redis-reactive    | 将Redis键值数据存储与Spring Data Redis Reacting和Lettuce客户端一起使用的Starter                               | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis-reactive/pom.xml)     |
| spring-boot-starter-data-rest              | 使用Spring Data REST通过REST公开Spring数据存储库的Starter                                                     | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-rest/pom.xml)               |
| spring-boot-starter-data-solr              | 将Apache Solr搜索平台与Spring Data Solr结合使用的Starter                                                      | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-solr/pom.xml)               |
| spring-boot-starter-freemarker             | 使用FreeMarker视图构建MVC Web应用程序的Starter                                                                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-freemarker/pom.xml)              |
| spring-boot-starter-groovy-templates       | 使用Groovy模板视图构建MVC Web应用程序的Starter                                                                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-groovy-templates/pom.xml)        |
| spring-boot-starter-hateoas                | 使用Spring MVC和Spring HATEOAS构建基于超媒体的RESTful Web应用程序的Starter                                    | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-hateoas/pom.xml)                 |
| spring-boot-starter-integration            | 使用Spring Integration的Starter                                                                               | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-integration/pom.xml)             |
| spring-boot-starter-jdbc                   | 结合使用JDBC和HikariCP连接池的Starter                                                                         | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jdbc/pom.xml)                    |
| spring-boot-starter-jersey                 | 使用JAX-RS和Jersey构建RESTful Web应用程序的Starter,的替代品spring-boot-starter-web                            | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jersey/pom.xml)                  |
| spring-boot-starter-jooq                   | 使用jOOQ访问SQL数据库的Starter,spring-boot-starter-data-jpa或spring-boot-starter-jdbc的替代品                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jooq/pom.xml)                    |
| spring-boot-starter-json                   | 读写JSON的Starter                                                                                             | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-json/pom.xml)                    |
| spring-boot-starter-jta-atomikos           | 使用Atomikos的JTA事务的Starter                                                                                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-atomikos/pom.xml)            |
| spring-boot-starter-jta-bitronix           | 使用Bitronix的JTA事务的Starter                                                                                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-bitronix/pom.xml)            |
| spring-boot-starter-mail                   | 使用Java Mail和Spring Framework的电子邮件发送支持的Starter                                                    | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mail/pom.xml)                    |
| spring-boot-starter-mustache               | 使用Mustache视图构建Web应用程序的Starter                                                                      | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mustache/pom.xml)                |
| spring-boot-starter-oauth2-client          | 使用Spring Security的OAuth2 / OpenID Connect客户端功能的Starter                                               | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-oauth2-client/pom.xml)           |
| spring-boot-starter-oauth2-resource-server | 使用Spring Security的OAuth2资源服务器功能的Starter                                                            | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-oauth2-resource-server/pom.xml)  |
| spring-boot-starter-quartz                 | 使用Quartz Scheduler的Starter                                                                                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-quartz/pom.xml)                  |
| spring-boot-starter-rsocket                | 用于构建RSocket客户端和服务器的Starter                                                                        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-rsocket/pom.xml)                 |
| spring-boot-starter-security               | 使用Spring Security的Starter                                                                                  | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-security/pom.xml)                |
| spring-boot-starter-test                   | 用于使用包括JUnit，Hamcrest和Mockito在内的库测试Spring Boot应用程序的Starter                                  | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-test/pom.xml)                    |
| spring-boot-starter-thymeleaf              | 使用Thymeleaf视图构建MVC Web应用程序的Starter                                                                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-thymeleaf/pom.xml)               |
| spring-boot-starter-validation             | 通过Hibernate Validator使用Java Bean验证的Starter                                                             | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-validation/pom.xml)              |
| spring-boot-starter-web                    | 使用Spring MVC构建Web（包括RESTful）应用程序的Starter。使用Tomcat作为默认的嵌入式容器                         | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web/pom.xml)                     |
| spring-boot-starter-web-services           | 使用Spring Web Services的Starter                                                                              | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web-services/pom.xml)            |
| spring-boot-starter-webflux                | 使用Spring Framework的Reactive Web支持构建WebFlux应用程序的Starter                                            | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-webflux/pom.xml)                 |
| spring-boot-starter-websocket              | 使用Spring Framework的WebSocket支持构建WebSocket应用程序的Starter                                             | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-websocket/pom.xml)               |

除了应用程序启动程序，以下启动程序可用于添加生产就绪功能：

**Table 2. Spring Boot生产Starter**

| Name                         | Description                                                                          | Pom                                                                                                                                                     |
| ---------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| spring-boot-starter-actuator | 使用Spring Boot的Actuator的Starter，它提供了生产就绪功能，可帮助您监视和管理应用程序 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-actuator/pom.xml) |

最后，Spring Boot还包括以下启动程序，如果您想排除或交换特定的技术方面，可以使用这些启动程序：

**Table 3. Spring Boot技术入门**

| Name                              | Description                                                                                  | Pom                                                                                                                                                          |
| --------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| spring-boot-starter-jetty         | 使用Jetty作为嵌入式servlet容器的Starter,spring-boot-starter-tomcat的替代品                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jetty/pom.xml)         |
| spring-boot-starter-log4j2        | 使用Log4j2进行日志记录的Starter,spring-boot-starter-logging的替代品                          | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-log4j2/pom.xml)        |
| spring-boot-starter-logging       | 使用Logback进行日志记录的Starter,默认日志Starter                                             | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-logging/pom.xml)       |
| spring-boot-starter-reactor-netty | 使用Reactor Netty作为嵌入式反应式HTTP服务器的Starter                                         | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-reactor-netty/pom.xml) |
| spring-boot-starter-tomcat        | 用于将Tomcat用作嵌入式Servlet容器Starter,spring-boot-starter-web默认使用的servlet容器Starter | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-tomcat/pom.xml)        |
| spring-boot-starter-undertow      | 使用Undertow作为嵌入式servlet容器的Starter,spring-boot-starter-tomcat的替代品                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-undertow/pom.xml)      |

> 	有关社区贡献的其他入门者的列表，请参阅GitHub中spring-boot-starters模块中的[README文件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/README.adoc)

# 3.2 结构化代码

Spring Boot不需要任何特定的代码布局即可工作。 但是，有一些最佳做法可以帮助您

## 3.2.1 使用默认包

当类不包含程序包声明时，将其视为在“默认程序包”中。 通常不建议使用“默认程序包”，应避免使用。 对于使用@ ComponentScan，@ ConfigurationPropertiesScan，@ EntityScan或@SpringBootApplication注解的Spring Boot应用程序，这可能会导致特定的问题，因为每个jar中的每个类都会被读取。
>我们建议您遵循Java建议的程序包命名约定，并使用反向域名（例如com.example.project）。

## 3.2.2 放置应用程序主类

我们通常推荐您将应用程序主类放置在其它类下根包中。@SpringBootApplicaton注解经常放在你的主类中，它隐式的定义了某些条目的基础搜索路径。例如，你正在开发一个JPA应用，@SpringBootApplication注解的类的包被用来搜索@Entity,使用根包也可以将组件扫描应用于你的项目。

>如果你不想使用@SpringBootApplication注解,@EnableAutoConfiguration和@ComponentScan也定义了那些功能，所以你可以使用它们作为替代。

下面列出了一个经典的布局
```
com
 +- example
     +- myapplication
         +- Application.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```
Applicaton.java文件会声明main方法，以及基本的@SpringBootApplication注解,如下所示:
```
package com.example.myappmlication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
# 3.3 配置类
Spring Boot支持基于Java的配置，尽管可以将SpringApplicaton和XML源一起使用，我们通常建议您的主要代码为单个@Confuguration类。通常，定义Main方法的类是首选的@Configuration。

>网上有很多使用XML配置的Spring示例，如果可能，尽量使用基于Java的配置。搜索@Enable*注解会是一个很好的开始。

## 3.3.1 导入额外的配置类

你不需要在每个类上加@Configuration注解，@Import注解可以用来加载其他的配置类。你也可以使用@ComponentScan注解来扫描所有的Spring组件，包括@Configuration注解的类。

## 3.3.2 导入XML配置

如果你一定要是使用基于XML的配置,我们建议您依然从@Configuration类开始，然后使用@ImportResource注解来加载XML配置文件。

# 3.4 自动配置
Spring Boot的自动配置会尝试根据你导入的依赖来自动配置你的应用。例如，如果HSQLDB在你的类路径中，但你并没有手动配置任何数据库连接Bean,那么Spring Boot会自动配置一个内存型数据库。

您需要通过向@Configuration类之一添加@EnableAutoConfiguration或@SpringBootApplication注释来选择加入自动配置。
> 你只需要加入@EnableAutoConfiguration或@SpringBootApplication之一，我们通常推荐你添加其中一个到你的一个配置类中。

## 3.4.1 逐渐取代自动配置

自动配置是非侵入性的，在任何时候，你可以自定义你的配置来代替自动配置的特定部分。例如，当你添加你的数据库连接Bean,那么默认的内嵌数据库会被取代。

如果你想知道正在应用的自动配置是什么，为什么这么配置，那就以debug模式启动你的应用，这样做可以启用调试日志，以选择核心记录器，并将条件报告记录到控制台。

## 3.4.2 关闭特定的配置类

如果你发现一个正在被使用的配置类不是你想要的，你可以使用@SpringBootApplication注解的exclude属性来关闭它，正如下所示：
```
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;

@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
public class MyApplication {
}
```
如果这个类不在类路径中，你可用使用该注解的excludeName属性和使用类的全路径限定名来作为替代。如果你比起@SpringBootApplication更倾向于使用@EnableAutoConfiguration注解，exclude和excludeName也是可以使用的。最后，您还可以使用spring.autoconfigure.exclude属性控制要排除的自动配置类的列表。
> 您可以在注解级别和使用属性来定义排除项。

> 即使自动配置类是公共的，该类的唯一方面也被认为是公共API，该类的名称可以用于禁用自动配置。 这些类的实际内容（例如嵌套配置类或Bean方法）仅供内部使用，我们不建议直接使用它们。

# 3.5 SpringBean和依赖注入

你可以自由的选择使用标准Spring Framework技术来定义你的Bean和依赖注入。为简单起见，我们发现使用@ComponentScan（扫描你的Bean）和@Autowired（用来构造器注入）效果很好。

如果你按照上面的建议来组织你的代码（把你的应用启动类放在根包），你可以添加@ComponentScan而不使用任何参数，你的所有Spring组件（@Component,@Repository,@Service,@Controller等等）将会被自动注册为Bean。

以下示例显示了一个@Service Bean，它使用构造函数注入来获取所需的RiskAssessor Bean：
```
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```
如果bean具有一个构造函数，则可以省略@Autowired，如以下示例所示：
```
@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```
>请注意，使用构造函数注入使riskAssessor字段被标记为final，表明它随后无法更改。

# 3.6 使用@SpringBootApplication注解

许多Spring Boot开发人员喜欢他们的应用使用自动配置，组件扫描并可以添加额外的配置到应用启动类上，单个的@SpringBootApplication注解可以开启下面三个功能：
- @EnableAutoConfiguration 开启[Spring Boot的自动配置机制](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)
- @ComponentScan:在应用程序所在的软件包上启用@Component扫描（请参阅[最佳实践](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#using-boot-structuring-your-code)）
- @Configuration 允许注入额外的Bean到上下文中或导入额外的配置类。
```
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
>@SpringBootApplication还提供别名以自定义@EnableAutoConfiguration和@ComponentScan的属性。

>这些功能都不是强制性的，您可以选择用它启用的任何功能替换此单个注释。 例如，您可能不想在应用程序中使用组件扫描或配置属性扫描：
>```
>package com.example.myapplication;
>
>import org.springframework.boot.SpringApplication;
>import org.springframework.context.annotation.ComponentScan
>import org.springframework.context.annotation.Configuration;
>import org.springframework.context.annotation.Import;
>
>
>@Configuration(proxyBeanMethods = false)
>@EnableAutoConfiguration
>@Import({ MyConfig.class, MyAnotherConfig.class })
>public class Application {
>
>    public static void main(String[] args) {
>            SpringApplication.run(Application.class, args);
>    }
>
>}
>```
>在此示例中，除了不会自动检测到@Component注释的类和@ConfigurationProperties注释的类并且显式导入用户定义的Bean外，Application就像其他任何Spring Boot应用程序一样（请参阅@Import）。

# 3.7 运行你的应用

将应用打包成为Jar包并使用内嵌HTTP服务器的最大好处是你可以在任何时候运行你的应用。调试Spring Boot应用也变得非常容易，你不需要任何特殊的IDE插件或扩展。

## 3.7.1 在IDE中运行你的应用

你可以像一个简单的Java应用一样在IDE中运行Spring Boo,但是你首先需要导入你的项目，导入过程根据你的IDE和构建系统有所区别。大多数IDE可以直接导入Maven工程，例如，Eclipse用户可以从File菜单中选择Import->Existing Maven Projects导入。

如果你不能将应用直接导入你的IDE,你可能需要用构建插件来生成IDE元数据。Maven包含了Eclipse和IDEA的插件，Gradle也同样为一些IDE提供了插件。

> 如果突然将一个Web应用运行两次，会看到一个“Port is already in use”的错误。STS用户可以使用ReLaunch按钮而不是Run来确保存在的实例被关闭。

## 3.7.2 运行打包后的程序

如果你使用Spring Boot的Maven或Gradle插件来生成可执行jar，你可以像下面的示例一样，使用java -jar来运行你的应用
```
  $ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```
可以在运行一个打包的应用的同时开启远程调试，这样做可以将一个debugger和你的应用连接起来，像下面的示例一样：
```
 $ java -jar -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

## 3.7.3 使用Maven插件

Spring Boot的Maven插件包含了一个运行的目标，可以来快速的编译运行你的应用。应用可以像在IDE中一样以一种暴露的方式运行。下面的示例展示了一个用经典的Maven命令来运行应用：
```
 $ mvn spring-boot:run
```
 您可能还想使用MAVEN_OPTS操作系统环境变量，如以下示例所示：
```
 $ export MAVEN_OPTS=-Xmx1024m
```

## 3.7.4 使用Gradle插件

Spring Boot Gradle插件还包含一个bootRun任务，该任务可用于以暴露的方式运行您的应用程序。 每当您应用org.springframework.boot和java插件时，都会添加bootRun任务，并在以下示例中显示：
```
 $ gradle bootRun
```
您可能还想使用JAVA_OPTS操作系统环境变量，如以下示例所示：
```
 $ export JAVA_OPTS=-Xmx1024m
```

## 3.7.5 热交换
 因为Spring Boot应用是指普通的Java应用，JVM热交换应该可以立即使用。JVM热交换在一定程度上受到它可以替换的字节码的限制，对于更完善的解决方案，可以使用JRebel。

spring-boot-devtools也包含了应用快速启动的支持，有关详细信息，请参见本章稍后的“devtools”部分和热交换“操作方法”。

# 3.8 DevTools

Spring Boot包括一组额外的工具，这些工具可以使应用程序开发体验更加愉快。spring-boot-devtools模块可以添加到任何的工程中来提供额外的开发时特性。要包括devtools支持，请将模块依赖项添加到您的构建中，如以下Maven和Gradle清单所示：

**Maven**
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```
**Gradle**
```
configurations {
    developmentOnly
    runtimeClasspath {
        extendsFrom developmentOnly
    }
}
dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```
> 开发人员工具会在运行完全打包应用的时候被自动关闭。如果您的应用程序是从java -jar启动的，或者是从特殊的类加载器启动的，则将其视为“生产应用程序”。 如果这不适用于您（即，如果您从容器中运行应用程序），请考虑排除devtools或设置-Dspring.devtools.restart.enabled = false系统属性。

>在Maven中将依赖项标记为可选，或在Gradle中使用自定义developmentOnly配置（如上所示）是一种最佳做法，可防止将devtools过渡地应用到使用项目的其他模块。

>重新打包的存档默认情况下不包含devtools。 如果要使用某个远程devtools功能，则需要禁用excludeDevtools构建属性以包括它。 Maven和Gradle插件均支持该属性。

## 3.8.1 默认属性
Spring Boot支持的一些库使用缓存来提高性能。例如，模板引擎 [template engines](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-template-engines)缓存已编译的模板，以避免重复解析模板文件。 而且，Spring MVC可以在提供静态资源时向响应添加HTTP缓存标头。

尽管缓存在生产中非常有益，但在开发过程中可能适得其反，从而使您无法看到自己刚刚在应用程序中所做的更改。 因此，默认情况下，spring-boot-devtools禁用缓存选项。

缓存选项通常由application.properties文件中的设置配置。 例如，Thymeleaf提供spring.thymeleaf.cache属性。 spring-boot-devtools模块不需要自动设置这些属性，而是自动应用合理的开发时配置。

因为在开发Spring MVC和Spring WebFlux应用程序时需要有关Web请求的更多信息，所以devtools将为Web日志记录组启用DEBUG日志记录。 这将为您提供有关传入请求，处理程序正在处理的信息，响应结果等信息。如果您希望记录所有请求详细信息（包括潜在的敏感信息），则可以打开spring.http.log-request- details配置属性。

> 如果你不想使用默认属性，可以在application.properties文件中设置spring.devtool.add-properties为false

>有关devtools应用的属性的完整列表，请参见[DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)

## 3.8.2 启动重启

使用spring-boot-devtools的应用会在类路径下的文件改变时自动重启，在IDE中工作时，这可能是一个有用的功能，因为它为代码更改提供了非常快速的反馈循环。 默认情况下，将监视类路径上指向文件夹的任何条目的更改。 请注意，某些资源（例如静态资产和视图模板）不需要重新启动应用程序。

>  触发重启
当DevTools监视类路径资源时，触发重启的唯一方法是更新类路径。 导致类路径更新的方式取决于您使用的IDE。 在Eclipse中，保存修改后的文件将导致类路径被更新并触发重新启动。 在IntelliJ IDEA中，构建项目（Build +→+ Build Project）具有相同的效果。

>只要启用了fork，您还可以使用受支持的构建插件（Maven和Gradle）来启动应用程序，因为DevTools需要隔离的应用程序类加载器才能正常运行。 默认情况下，Gradle和Maven插件会fork应用程序进程。

>与LiveReload一起使用时，自动重启非常有效。 有关详细信息，请参见LiveReload部分。 如果使用JRebel，则禁用自动重新启动，而支持动态类重新加载。 其他devtools功能（例如LiveReload和属性替代）仍可以使用。

>DevTools依靠应用程序上下文的关闭挂钩在重新启动期间将其关闭。 如果您禁用了关闭挂钩（SpringApplication.setRegisterShutdownHook（false）），它将无法正常工作。

>在确定类路径上的条目是否应在更改后触发重新启动时，DevTools会自动忽略名为spring-boot，spring-boot-devtools，spring-boot-autoconfigure，spring-boot-actuator和spring-boot-starter的项目。

>DevTools需要自定义ApplicationContext使用的ResourceLoader。 如果您的应用程序已经提供了，它将被包装。 不支持在ApplicationContext上直接重写getResource方法。

>                                                          重启VS重加载
>    重启技术是通过Spring Boot使用两个类加载器来实现，不变的类（例如那些第三方Jar包中的）被加载到一个基本类加载器，你正在开发的类使用一个restart类加载器,当应用重启时，restart类加载器被丢掉并新建一个实例，这种方式意味着比传统的冷启动更快，因为基本类加载器一直可用并已填充
>
>如果发现重新启动对于您的应用程序而言不够快，或者遇到类加载问题，则可以考虑从ZeroTurnaround重新加载技术，例如JRebel。 这些方法通过在加载类时重写类来使它们更适合于重新加载。

**记录条件评估中的更改**

默认每次应用重启时，记录条件评估增量的报告。该报告显示了您进行更改（例如添加或删除Bean以及设置配置属性）时对应用程序自动配置的更改。

要禁用报告的日志记录，请设置以下属性：
```
spring.devtools.restart.log-condition-evaluation-delta=false
```

**排除资源**

某些资源在更改时不一定需要触发重新启动。 例如，Thymeleaf模板可以就地编辑。 默认情况下，更改/ META-INF / maven，/ META-INF / resources，/ resources，/ static，/ public或/ templates中的资源不会触发重新启动，但会触发实时重新加载。 如果要自定义这些排除项，则可以使用spring.devtools.restart.exclude属性。 例如，要仅排除/ static和/ public，可以设置以下属性：
```
spring.devtools.restart.exclude=static/**,public/**
```
>如果你想保持默认的配置并增加一些额外的排除项，请使用spring.devtools.restart.addtional-exclude属性作为替代

**监测额外路径**

当您对不在类路径上的文件进行更改时，您可能希望重新启动或重新加载应用程序。 为此，请使用spring.devtools.restart.additional-paths属性配置其他路径以监视更改。 您可以使用前面所述的spring.devtools.restart.exclude属性来控制其他路径下的更改是触发完全重启还是动态重新加载。

**关闭重启**

如果你不想使用重启功能，你可以设置spring.devtool.restart.enabled属性来关闭它。在大多数情况下，您可以在application.properties中设置此属性（这样做仍会初始化restart类加载器，但它不会监视文件更改）。

如果您需要完全禁用重启支持（例如，因为它不适用于特定的库），则需要在调用SpringApplication.run（...）之前将spring.devtools.restart.enabled系统属性设置为false，例如 如以下示例所示：
```
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```
**使用触发文件**

如果使用持续编译更改文件的IDE，则可能更喜欢仅在特定时间触发重新启动。 为此，您可以使用“触发文件”，这是一个特殊文件，当您要实际触发重新启动检查时必须对其进行修改。

>对文件的任何更新都会触发检查，但是只有在Devtools检测到有事情要做的情况下，重启才会真正发生。

要使用触发文件，请将spring.devtools.restart.trigger-file属性设置为触发文件的名称（不包括任何路径）。 触发器文件必须出现在类路径上的某个位置。

例如，如果您的项目具有以下结构：
```
src
+- main
   +- resources
      +- .reloadtrigger
```
那么你的trigger-file属性会是这样的：
```
spring.devtools.restart.trigger-file=.reloadtrigger
```
现在重启只会在src/main/resources/.reloadtrigger文件更新是发生

您可能希望将spring.devtools.restart.trigger-file设置为[全局设置](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#using-boot-devtools-globalsettings)，以便所有项目的行为均相同。

某些IDE具有使您不必手动更新触发器文件的功能。[Spring Tools for Eclipse](https://spring.io/tools)
和 [IntelliJ IDEA (Ultimate Edition)](https://www.jetbrains.com/idea/)都具有这种支持。 使用Spring Tools，您可以从控制台视图使用“重新加载”按钮（只要您的触发文件名为.reloadtrigger）。 对于IntelliJ，您可以按照其[文档](https://www.jetbrains.com/help/idea/spring-boot.html#configure-application-update-policies-with-devtools)中的说明进行操作。

**定制Restart类加载器**

如前面的“重新启动与重新加载”部分所述，重新启动功能是通过使用两个类加载器实现的。 对于大多数应用程序，此方法效果很好。 但是，有时可能会导致类加载问题。

默认情况下，IDE中任何打开的项目都使用“重新启动”类加载器加载，而任何常规.jar文件都使用“基本”类加载器加载。 如果您在多模块项目上工作，并且并非每个模块都导入到IDE中，则可能需要自定义内容。 为此，您可以创建一个META-INF / spring-devtools.properties文件。

spring-devtools.properties文件可以包含带有restart.exclude和restart.include前缀的属性。 include元素是应上拉到“重新启动”类加载器中的项目，而exclude元素是应下推到“基本”类加载器中的项目。 该属性的值是一个应用于类路径的正则表达式模式，如以下示例所示：
```
restart.exclude.companycommonlibs=/mycorp-common-[\\w\\d-\.]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w\\d-\.]+\.jar
```
>所有属性键都必须是唯一的。 只要属性以restart.include开头， 或restart.exclude开头，都会被考虑在内

>类路径中的所有META-INF / spring-devtools.properties将被加载。 您可以将文件打包到项目内部或项目使用的库中。

**已知局限性**

重新启动功能不适用于使用标准ObjectInputStream反序列化的对象。 如果您需要反序列化数据，则可能需要将Spring的ConfigurableObjectInputStream与Thread.currentThread().getContextClassLoader()结合使用。

如果您不想在应用程序运行时启动LiveReload服务，则可以将spring.devtools.livereload.enabled属性设置为false。

>一次只能运行一个LiveReload服务。 在启动应用程序之前，请确保没有其他LiveReload服务正在运行。 如果从IDE启动多个应用程序，则只有第一个具有LiveReload支持。

## 3.8.4 全局设置

您可以通过将以下任何文件添加到$HOME/.config / spring-boot文件夹来配置全局devtools设置：

1. spring-boot-devtools.properties
2. spring-boot-devtools.yaml
3. spring-boot-devtools.yml

添加到这些文件的任何属性都将应用于使用devtools的计算机上的所有Spring Boot应用程序。 例如，要将重新启动配置为始终使用触发文件，应添加以下属性：

**~/.config/spring-boot/spring-boot-devtools.properties**
```
spring.devtools.restart.trigger-file=.reloadtrigger
```
>如果在\$HOME/.config/spring-boot中找不到devtools配置文件，则在\$HOME文件夹的根目录中搜索是否存在.spring-boot-devtools.properties文件。 这使您可以与不支持\$HOME/.config/spring-boot位置的较旧版本的Spring Boot上的应用程序共享devtools全局配置。

>在上述文件中激活的配置文件不会影响特定配置文件的加载。

## 3.8.5 远程应用

Spring Boot开发人员工具不仅限于本地开发。 远程运行应用程序时，您还可以使用多种功能。 选择启用远程支持，可能会带来安全风险。 仅当在受信任的网络上运行或使用SSL保护时，才应启用它。 如果这两个选项都不可用，则不应使用DevTools的远程支持。 您永远不要在生产部署上启用支持。

要启用它，您需要确保重新打包的档案中包含devtools，如以下清单所示：
```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludeDevtools>false</excludeDevtools>
            </configuration>
        </plugin>
    </plugins>
</build>
```
然后，您需要设置spring.devtools.remote.secret属性。 像任何重要的密码或机密一样，该值应唯一且强壮，以免被猜测或强行使用。

远程devtools支持分为两个部分：接受连接的服务器端端点和在IDE中运行的客户端应用程序。 设置spring.devtools.remote.secret属性后，将自动启用服务器组件。 客户端组件必须手动启动。

**运行远程客户端应用**

远程客户端应用程序旨在在您的IDE中运行, 您需要使用与您连接到的远程项目相同的类路径来运行org.springframework.boot.devtools.RemoteSpringApplication。 该应用程序的唯一必需参数是它连接到的远程URL。

例如，如果您使用的是Eclipse或STS，并且有一个名为my-app的项目已部署到Cloud Foundry，则可以执行以下操作：
- 从Run菜单中选择Run Configuration
- 创建一个新的Java应用启动配置
- 选择my-app项目
- 使用org.springframework.boot.devtools.RemodeSpringApplication作为启动类
- 将https://myapp.cfapps.io（或你的URL）添加到程序参数中

正在运行的远程客户端可能类似于以下清单：
```
  .   ____          _                                              __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
 \\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
  '  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
 =========|_|==============|___/===================================/_/_/_/
 :: Spring Boot Remote :: 2.2.6.RELEASE

2015-06-10 18:25:06.632  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication on pwmbp with PID 14938 (/Users/pwebb/projects/spring-boot/code/spring-boot-project/spring-boot-devtools/target/classes started by pwebb in /Users/pwebb/projects/spring-boot/code)
2015-06-10 18:25:06.671  INFO 14938 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@2a17b7b6: startup date [Wed Jun 10 18:25:06 PDT 2015]; root of context hierarchy
2015-06-10 18:25:07.043  WARN 14938 --- [           main] o.s.b.d.r.c.RemoteClientConfiguration    : The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
2015-06-10 18:25:07.074  INFO 14938 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2015-06-10 18:25:07.130  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 0.74 seconds (JVM running for 1.105)
```
>因为远程客户端使用与真实应用程序相同的类路径，所以它可以直接读取应用程序属性。 这就是读取spring.devtools.remote.secret属性并将其传递给服务器进行身份验证的方式。

>始终建议使用https：//作为连接协议，以便对通信进行加密并且不能截获密码。

>如果需要使用代理来访问远程应用程序，请配置spring.devtools.remote.proxy.host和spring.devtools.remote.proxy.port属性

**远程更新**

远程客户端以与本地重新启动相同的方式监视应用程序类路径中的更改。 任何更新的资源都会推送到远程应用程序，并且（如果需要）会触发重新启动。 如果您迭代使用本地没有的云服务的功能，这将很有帮助。 通常，远程更新和重新启动比完整的重建和部署周期要快得多。
>仅在远程客户端正在运行时监视文件。 如果在启动远程客户端之前更改文件，则不会将其推送到远程服务器。

**配置文件系统观察器**

[FileSystemWatcher](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/filewatch/FileSystemWatcher.java)的工作方式是按一定时间间隔轮询类更改，然后等待预定义的静默期以确保没有更多更改。 然后将更改上传到远程应用程序。 在较慢的开发环境中，可能会发生静默期不够的情况，并且类中的更改可能会分为几批。 第一批类更改上传后，服务器将重新启动。 由于服务器正在重新启动，因此下一批不能发送到应用程序。

这通常通过RemoteSpringApplication日志中的警告来证明，即有关上载某些类失败的消息，然后进行重试。 但是，这也可能导致应用程序代码不一致，并且在上传第一批更改后无法重新启动。

如果您经常观察到此类问题，请尝试将spring.devtools.restart.poll-interval和spring.devtools.restart.quiet-period参数增加到适合您的开发环境的值：
```
spring.devtools.restart.poll-interval=2s
spring.devtools.restart.quiet-period=1s
```
现在每2秒轮询一次受监视的classpath文件夹是否有更改，并保持1秒钟的静默时间以确保没有其他类更改。

# 3.9  打包您的生产应用

可执行jar可以用于生产部署。 由于它们是独立的，因此它们也非常适合基于云的部署。

对于其他“生产准备就绪”功能，例如运行状况，审核和度量REST或JMX端点，请考虑添加spring-boot-actuator。 有关详细信息，请参见Spring Boot Actuator：[生产就绪功能](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#production-ready)。

# 3.10 接下来阅读什么

现在，您应该了解如何使用Spring Boot以及应遵循的一些最佳实践。 现在，您可以继续深入了解特定的[Spring Boot功能](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features)，或者可以跳过并阅读有关Spring Boot的“[生产就绪](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#production-ready)”方面的信息。
