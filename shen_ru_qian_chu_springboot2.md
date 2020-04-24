## Chapter 3 全注解下的 Spring IoC ##

Ioc 是一种通过描述来生成或获取对象的技术。

在 Spring 中，把每个需要管理的对象称为 Spring Bean，而 Spring 管理这些 Bean 的容器称为 Spring IoC 容器。 IoC 容器需要具备2个基本的功能：

- 通过描述管理 Bean，包括发布和获取 Bean
- 通过描述完成 Bean 之间的依赖关系

Spring 中，所有的 IoC 容器都需要实现接口 `BeanFactory`。它是一个顶级容器接口。

`SpringBootApplication` 提供的 `exclude` 和 `excludeName` 两个方法只对其内部的自动配置类有效，如果需要排除其他类，需要加入 `@ComponentScan` 并配置它的 `excludeFilters` 属性。

`@Autowired` 注解匹配 Bean 的规则：  
首先会根据类型找到对应的 Bean，如果找到多个候选者，则根据其属性名和 Bean 的名称进行匹配并返回匹配者。  
另外，该注解默认一定会找到匹配者，如果没有则会抛出异常。但可以通过设置其 `required` 属性为 `false` 来允许其返回 null ，而不是抛出异常。

可以使用 `@Qualifier` 注解，通过配置它的 `value` 属性值（字符串类型）来指定需要的 Bean 的名称。   
如果不使用该注解，默认要查找的 Bean 的名称根据 `@Autowired` 标注的内容类型不同而不同：

- 标注属性：要查找的 Bean 的名称是属性名头字母小写
- 标注 setter 方法：方法名去掉 `set` 后头字母小写

这些规则和 `@Bean` 注解指定所生成的 Bean 的名称的规则相同。

`@Autowired` 和 `@Qualifier` 也可用于标注构造方法的参数。

Bean 的生命周期按顺序包括：

1. Bean 的定义
2. Bean 的初始化
3. Bean 的生存期
4. Bean 的销毁

Bean 的定义过程大致如下：

1. Spring 通过我们的配置，如 `@ComponentScan` 定义的扫描路径去找到带有 `@Component` 的类，即资源定位；
2. 一旦找到了资源，就开始解析，并将定义的信息保存起来。此时还没有开始初始化 Bean，也就没有 Bean 的实例，仅有 Bean 的定义；
3. 把 Bean 定义发布到 Spring IoC 容器中。此时 IoC 容器中也只有 Bean 的定义，还是没有生成任何 Bean。

`BeanPostProcessor` 是针对所有 Bean 都有效的，包括它的 `postProcessBeforeInitialization()` 和 `postProcessAfterInitialization()` 方法。

只有实现了 `ApplicationContext` 接口的容器，才会在生命周期调用 `ApplicationContextAware` 所定义的 `setApplicationContext()` 方法。

属性文件中的属性值通过 `${...}` 的方式来引用。

可以使用 `@ConfigurationProperties` 注解，该注解的值将与 POJO 的属性名称一起组成属性的全限定名去配置文件中查找。  
还可以通过 `@PropertySource` 注解定义对应的属性文件。其 `value` 值可以配置多个配置文件。当 `value` 值使用 `classpath` 前缀时，表示去类库文件路径下寻找属性文件。 其 `ignoreResourceNotFound` 属性值默认为 `false`，表示如果没有找到属性文件会报错，如果为 `true` 则会忽略，不报错。

`@Conditional` 需要配合 `@Condition` 使用。

Bean 的作用域：

作用域类型    | 使用范围         | 作用域描述
---           | ---              | ---
singleton     | 所有 Spring 应用 | 默认值， IoC 容器中只存在单例
prototype     | 所有 Spring 应用 | 每次从 IoC 容器中取出的 Bean，都是一个新的 Bean 实例
session       | Spring Web 应用  | HTTP 会话
application   | Spring Web 应用  | Web 工程生命周期
request       | Spring Web 应用  | Web 工程单次请求
globalSession | Spring Web 应用  | 在一个全局的 HTTP Session 中，一个 Bean 定义对应一个实例，一般不使用


Spring 中可以使用 `@Profile` 注解来切换不同的配置环境。   
有2个参数用来修改启动的 profile ：

- `spring.profiles.active`：优先级高
- `spring.profiles.default`：优先级低

以上都没有设置，则被 `@Profile` 标注的 Bean 不会被 Spring 装配到 IoC 容器中。

如果启动 Spring 应用的参数 `-Dspring.profiles.active` 的值记为 `{profile}`，则他会用 `application-{profile}.properties` 配置文件去代替原来默认的 `application.properties` 文件。

`${...}` 代表占位符，会读取上下文的属性值装配到属性中；  
`#{...}` 代表启用 Spring 表达式，将具有运算功能，如 `#{T(System).currentTimeMillis()}` 。其中 `T(...)` 代表的是引入类，`System` 是 Java 默认加载的包中的类，因此不必写全限定名，其他的包则需要写出类的全限定名；  
此外还可以给属性直接赋值，或获取其他 Spring Bean 的属性值来给当前的 Bean 属性赋值，如 `@Value("#{beanName.attr}")`，其中 `beanName` 是引用的 Bean 在 Spring IoC 容器中的名称， `attr` 是其属性，代表引用 Bean `beanName` 的 `attr` 属性的值为当前属性赋值。  
也可以调用 `attr` 属性的方法来为当前属性赋值， 如 `@Value("#{beanName.attr?.method()}")`，  `?`表示需要判断该属性是否为空，不为空才会调用它的方法。

## Chapter 4 约定编程 ##

JDK 中的 `Proxy` 提供了获得代理对象的静态方法： 

```java
newProxyInstance(ClassLoader classLoader, Class<?>[] interfaces, InvocationHandler invocationHandler)
```

- `classLoader`：类加载器
- `interfaces`：绑定的接口，也就是把代理对象绑定到哪些接口下，必须是代理对象实现的接口
- `invocationHandler`：绑定代理对象逻辑实现，是一个接口 `InvocationHandler` 对象，定义了一个 `invoke` 方法，该方法负责实现代理对象的逻辑。

```java
public Object invoke(Object proxy, Method method, Object[] args);
```

- `proxy`：代理对象
- `method`：要通过代理对象调用的方法
- `args`：要调用的方法的参数

一般常见的数据库操作流程为：

1. 打开数据库连接，并对其属性进行设置
2. 执行 SQL 语句
3. 如果没有异常，提交事务
4. 如果出现异常，回滚事务
5. 关闭数据库事务连接

其中 1, 3，4，5 都可以提供默认实现来减少开发人员工作量，这也是 Spring AOP 实际做的工作之一。

Spring AOP 是基于方法的 AOP，只能应用于方法上。它的相关术语：

- **连接点（join point）**：对应被拦截的对象，在 Spring 中指的就是特定的方法。
- ** 切点（point cut）**：当切面不仅适用于单个方法，而是多个类的不同方法时，可以通过正则表达式和指示器的规则去定义适配连接点的条件。这些条件组合就是切点。
- **通知（advice）**：按照约定的流程下的方法，分为前置通知（before advice），后置通知（after advice），环绕通知（around advice），事后返回通知（afterReturning advice）和异常通知（afterThrowing advice）。
- **目标对象（target）**：被代理对象
- **引入（introduction）**：引入新的类和其方法，增强现有 Bean 的功能
- **织入（weaving）**：通过动态代理技术，为原有服务对象生成代理对象，然后将与切点匹配的连接点拦截，并按约定将各类通知织入约定流程的过程
- **切面（aspect）**：可以定义切点，各类通知和引入的内容，通常是一个类， Spring AOP 通过它的信息来增强 Bean 的功能或将相应的方法织入流程。

AOP  开发的一般过程：

- 确定连接点
- 开发切面， Spring 以 `@Aspect` 作为切面的声明
- 定义切点，使用 `@PointCut` 注解定义切点，标注在某个方法上，后面的通知注解中就可以使用该方法名称引用切点

切点表达式举例：

```Java
execution(* net.xiaoluo.practice.qualified.ClassName.methodName(..))
```

- `execution`：表示在执行的时候，拦截里面的正则匹配的方法
- `*`：表示任意返回类型的方法
- `net.xiaoluo.practice.qualified.ClassName`：指定目标对象类的全限定名
- `methodName`：指定目标对象的方法
- `(..)`：表示任意参数进行匹配


AspectJ 关于 Spring AOP 切点的指示器：

类型          | 　描述
---           | ---
arg()         | 限定连接点方法参数
@args()       | 　通过连接点方法参数上的注解进行限定
execution()   | 用于匹配是连接点的执行方法
this()        | 限制连接点匹配 AOP 代理 Bean 引用为指定的类型
target        | 目标对象（被代理对象）
@target()     | 限制目标对象配置了指定的注解
within        | 限制连接点匹配指定的类型
@within()     | 限制连接点带有匹配的注解类型
@annotation() | 限定带有指定注解的连接点

使用环绕通知的场景一般是在需要大幅度修改原有目标对象的服务逻辑时，否则应该尽量使用其他的通知。环绕通知的方法拥有一个 `ProceedingJoinPoint` 类型的参数，该类有一个实例方法 `proceed()`，通过该方法回调被代理对象的方法。

`@DeclareParents` 用于引入新的类来增强服务，有两个必须的属性 `value` 和 `defaultImpl`

- `value`：要增强的目标对象，可以是接口
- `defaultImpl`：要引入的带有增强功能的类

通过 `args()` 和 `JoinPoint` 类配合使用可以为非环绕通知方法传递参数，环绕通知方法则可以通过前面提到的 `ProceedingJoinPoint` 类来获得参数。

```Java
// 把名为 user 的参数传递进来
@Before("pointCut() && args(user)")
public void beforeParam(JoinPoint point, User user) {
	Object[] args = point.getArgs();
	System.out.println("before ...");
}
```

`@Order` 或 `Ordered` 接口可以在有多个切面时为它们排序，值越低则优先级越高。






