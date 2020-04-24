## Chapter 2 理解独立的 Spring 应用 ##

从应用类型上划分， Spring Boot 应用包括：
	- Web 应用：
		- 在 Spring Boot 1.x 中仅有 Servlet 容器实现，包括传统的 Servlet 和 Spring Web MVC
		- 2.0 开始新增了 Reactive Web 容器实现，即 Spring 5.0 WebFlux。所以在 SpringApplication API 上新增了 `setWebApplicationType(WebApplicationType)` 方法。 `WebApplicationType` 是一个枚举类型，对应 Web 应用的类型：
			- `NONE`：非 Web 类型
			- `SERVLET`： Servlet Web 应用
			- `REACTIVE`： Reactive Web 应用 
	- 非 Web 应用；用于服务提供，调度任务，消息处理等。

Spring Boot 的错误分析报告器，用 API `FailureAnalysisReporter` 表示，是从 1.4 版之后加入的特性。

使用 `java -jar` 与 `mvn spring-boot:run` 运行应用基本没有分别，只是前者称为“生产环境运行方式”，后者称为“开发阶段运行方式”。

打包后的文件夹中有一个以 `original` 结尾的 jar 文件，例如 `first-app.jar.original` ，该文件仅包含应用的本地资源，如编译后的 `classes` 目录下的资源文件，不包含任何第三方依赖资源。而以 `.jar` 结尾的 jar 文件则包含了所有的依赖资源。其目录结构如下：

- `BOOT-INF/classes`：存放应用编译后的 class 文件
- `BOOT-INF/lib`：存放应用依赖的 jar 包
- `META-INF/`：存放应用相关的元信息，如 `MENIFEST.MF` 文件
- `org/`：存放 Spring Boot 相关的 class 文件

`URL` 的关联协议（Protocol）对应一种 `URLStreamHandler` 实现类， JDK 默认支持文件（file）， HTTP， JAR 等协议，所以 JDK 内建了对应协议的实现，这些实现类都存放在 `sun.net.www.protocol` 包中，并且类名必须为 `Handler`，其类全名为 `sun.net.www.protocol.${protocol}.Handler` ，其中 `${protocol}` 表示协议名。

`URL#getURLStreamHandler(String)` 先读取 Java 系统属性 `java.protocol.handler.pkgs`，然后再追加 `sun.net.www.protocol` 包，因此，前者具有更高的优先级，而后者是补充。

`JarLauncher` 实际上是同进程内调用 `Start-Class` 类的 `main(String[])` 方法，并在启动前准备好了 Class Path。

对于 Web 应用， `WEB-INF/Classes/` 和 `WEB-INF/Lib/` 是传统的 Servlet 应用的 Class Path 路径， 而 `WEB-INF/lib-provided/` 属于 Spring Boot WarLauncher 的定制实现，存放的是 `<scope>provided</scope>` 的 JAR 文件。事实上，打包 WAR 文件是一种兼容措施，既能被 WarLauncher 启动，又能兼容 Servlet 容器环境。换言之， WarLauncher 和 JarLauncher 并无本质区别，在使用非传统 Web 部署 Spring Boot 应用时，尽可能使用 JAR 归档方式。：

## Chapter 3 理解固化的 Maven 依赖 ##

单独引入 `spring-boot-maven-plugin` 插件时，需要配置 `<goal>repackage</goal>` 元素，否则不会添加 Spring Boot 引导依赖，进而无法引导当前应用。

通常不会使用 `spring-boot-dependencies` 作为 Maven 项目的 `<parent>`，而是使用 `spring-boot-starter-parent`。

## Chapter 4 理解嵌入式 Web 容器 ##

Tomcat Maven 插件并非嵌入式 Tomcat，仍旧利用了传统 Tomcat 容器部署方式，将 Web 应用打包为 `ROOT.war` 文件，然后在 Tomcat 应用启动的过程中，将 `ROOT.war` 文件解压至 `webapps` 目录中。  
此外，Tomcat Maven 插件打包时会压缩归档文件，因此生成的 JAR 或 WAR 文件属于非 FAT 模式，而 Spring Boot Maven 插件 `spring-boot-maven-plugin` 打包时采用零压缩模式，相当于在使用 `jar` 命令归档时添加了 `-0` 参数。

传统 Servlet 容器将压缩的 WAR 文件解压到对应目录，在加载该目录下的资源。而 Spring Boot 可执行 WAR 文件则需要在不解压当前 WAR 文件的前提下读取其中的资源。

嵌入式 Reactive Web 容器作为 Spring Boot 2.0 的特性，通常处于被动激活状态，如增加 `spring-boot-starter-webflux` 依赖，然而当它与 `spring-boot-starter-web` 同时存在时，`spring-boot-starter-webflux` 会被忽略。

## Chapter 5 理解自动装配 ##

Spring Framework 装配 `@Configuration` 类的三种方式：

- 通过 XML 元素 `<context:component-scan>`
- 通过注解 `@Import`
- 通过注解 `@ComponentScan`

这三种方式都需要 Spring 应用上下文引导，前者采用 `ClassPathXmlApplicationContext` 加载，后两者需要 `AnnotationConfigApplicationContext` 加载。

`@SpringBootApplication` 注解相当于以下三个注解的集合：

- `@EnableAutoConfiguration`：负责激活 Spring Boot 自动装配机制；
- `@Configuration`： 声明被标注类为配置类；
- `@ComponentScan`：激活 `@Component` 的扫描，在 Spring Boot 2.0 中，此注解指定了属性值：
	
	```java
	...
	@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class)
	})
	...

其中：

－ `TypeExcludeFilter` 在 Spring Boot 1.4 引入，用于查找 `BeanFactory` 中已注册的 `TypeExcludeFilter` Bean，作为代理执行对象；
- `AutoConfigurationExcludeFilter` 自 Spring Boot 1.4 开始支持，用于排除其他同时标注了 `@Configuration` 和 `@EnableAutoConfiguration` 的类。

自 Spring Boot 1.4 开始， `@SpringBootApplication` 不再被标注为 `@Configuration`， 而是 `@SpringBootConfiguration`，二者在运行上没有差别，类似于对象的继承关系。

`@SpringBootApplication` 可以通过属性别名来设置以上三个注解的属性，如 `scanBasePackages` 属性对应了 `@ComponentScan` 注解的 `basePackages` 属性。

尽管 `@EnableAutoConfiguration`（`@Component` 派生注解） 和 `@SpringBootApplication`（`@Configuration` 的派生注解） 在激活自动装配上没有区别，但被标注类对应的 Bean 类型存在差异。被 `@Configuration` 标注的类所对应的 Bean 类型属于完全模式，执行 CGLIB 提升的操作，而 `@Component` 标注的类的 Bean 类型则是轻量模式（Lite），不存在 CGLIB 处理。注意，CGLIB 提升并非为 `@Bean` 对象提供，而是为 `@Configuration` 类准备的。

`spring-boot-autoconfigure` 是 Spring Boot 的核心模块，提供了大量的内置自动装配 `@Configuration` 类，它们统一存放在 `org.springframework.boot.autoconfigure` 包或其子包下，同时，这些类都配置在 `META-INF/spring.factories` 资源中。该资源属于 Java Properties 文件格式，其中激活自动装配注解 `@EnableAutoConfiguration` 充当该 Properties 的 key ，而自动装配类为 key 的 value 。注意所有的类的命名均以 `AutoConfiguration` 作为后缀。
