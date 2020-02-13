Spring 框架的核心机制是 Spring 容器（Spring Core Container）。它主要由 `org.springframework.core`，`org.springframework.beans`，`org.springframework.context`和`org.springframework.expression` 4个包及其子包组成，提供 Spring IoC 支持。

Spring 框架的本质就是通过 XML 配置文件驱动 Java 代码的运行。把原本由Java代码管理的耦合关系提取到XML配置文件中管理。

Spring 把容器中的一切 Java 对象统称为 Bean。推荐面向接口编程。

Spring 对 XML 配置文件的名称没有任何要求。

- 配置文件中的 `<bean .../>`元素默认以反射的方式调用指定类的无参数构造器，`class`属性指定类时必须使用全限定类名，不能是接口，也不能是抽象类。

Spring调用构造器创建对象和调用setter方法是先后执行的，中间几乎没有间隔。

`ApplicationContext`是Spring框架里最常用的接口，有两个实现类：

- `ClassPathXmlApplicationContext`：使用类加载路径搜索配置文件并创建Spring容器。（推荐使用）
- `FileSystemXmlApplicationContext`：使用文件系统的相对或绝对路径搜索配置文件并创建Spring容器。

### Spring核心机制：依赖注入 ###

Spring中的依赖关系：如果A对象需要调用B对象的方法，则称A对象B对象，这种调用关系也称为依赖关系。

Spring框架的核心功能有2个：

- Spring容器作为超级大工厂，负责创建和管理所有的Java对象，这些对象在Spring容器中称为Bean；
- Spring容器使用依赖注入管理容器中Bean之间的依赖关系。

#### 理解依赖注入 ####

IoC(Inversion of Control) 控制反转/Dependency Injection 依赖注入：以配置文件来管理Java实例之间的协作关系。不同的名字代表理解这种管理方式的不同角度：

- IoC(Inversion of Control) 控制反转：传统模式下调用A对象调用B对象的方法，需要A对象主动在内部创建一个B对象，或者通过简单工厂模式，主动去B对象的工厂类/对象获取B对象；Spring框架则是通过配置文件来建立2个对象之间的关联。A对象不再主动创建或去获取B对象，而是被动地接受配置文件为其指定的B对象。换句话说，A对象由主动变为被动，因此称为控制反转。
- Dependency Injection 依赖注入：Spring容器负责管理依赖关系，通过将被依赖对象赋值给调用者的成员变量，将被依赖对象（如上例中的B对象）传递给依赖对象（上例中的A对象），从而完成为调用者注入它所依赖的实例，因此称为依赖注入。

注入方式有两种：

- 设值注入：底层使用无参数的构造器创建被依赖对象，再使用成员变量的setter方法完成初始化。在配置文件中使用`<property.../>`子元素指定需要赋值的成员变量的名字和值；
- 构造注入：底层直接使用被依赖对象的带参数构造器完成对成员变量的初始化。在配置文件中使用`<constructor-arg.../>`子元素指定构造器参数的值。

`<peroperty.../>`和`<constructor-arg.../>`都可以使用`value`，`ref`和`type`属性指定传入的值。
	- `value`属性：传入参数是基本类型及其包装类，`String`，`date`类型。
	- `type`属性：和`value`属性配合使用，显式指定传入值的类型。
	- `ref`属性：以容器中的其他Bean作为传入参数，属性值是传入Bean的`id`属性值。

建议采用设值注入为主，构造注入为辅的注入策略。对于依赖关系无须变化的注入，尽量采用构造注入；而采用设值注入负责其他依赖关系的注入。

### 使用Spring容器 ###

Spring的核心接口是`BeanFactory`和`ApplicationContext`接口，其中`ApplicationContext`接口是`BeanFactory`接口的子接口，都可以代表Spring容器。Spring容器也成为Spring上下文。

`BeanFactory`中的基本方法：

- boolean containsBean(String name)
- <T> T getBean(Class<T> requiredType)
- Object getBean(String name)
- <T> T getBean(String name, Class<T> requiredType)
- Class<?> getType(String name)

`BeanFactory`接口的常用实现类是`DefaultListableBeanFactory`。

`ApplicationContext`作为`BeanFactory`的子接口，常用的实现类包括：

- `ClassPathXmlApplicationContext`：使用类加载路径搜索配置文件并创建Spring容器。（推荐使用）
- `FileSystemXmlApplicationContext`：使用文件系统的相对或绝对路径搜索配置文件并创建Spring容器。
- `AnnotationConfigApplicationContext`：
- `XmlWebApplicationContext`：面向Web应用
- `AnnotationConfigWebApplicationContext`：面向Web应用

`ApplicationContext`提供以下额外功能：

- 默认预初始化所有的singleton Bean（配置文件中的Bean默认就是单例Bean）；
- 继承`MessageSource`接口，提供国际化支持；
- 允许访问资源，如访问URL和文件；
- 提供事件机制；
- 允许同时加载多个配置文件；
- 可以以声明方式启动并创建Spring容器。

#### ApplicationContext的国际化支持 ####

`ApplicationContext`接口继承了`MessageSource`接口，使用该接口的以下2个方法来获取国际化消息：

- `String getMessage(String code, Object[] args, Locale loc)`
- `String getMessage(String code, Object[] args, String default, Locale loc)`

当程序创建`ApplicationContext`容器时，Spring自动查找配置文件中名为`messageSource`的Bean实例，并由该实例负责以上2个方法的调用；如果没有该Bean，则`ApplicationContext`会查找并使用其父容器的`messageSource`Bean，如果最终没有找到该Bean，则系统创建一个空的`StaticMessageSource` Bean，并由它来处理以上2个方法的调用。
配置文件中的`messageSource` Bean通常使用`ResourceBundleMessageSource`实现类。

#### ApplicationContext的事件机制 ####

ApplicationContext的事件机制是观察者模式的实现，通过`ApplicationEvent`类和`ApplicationListener`接口处理事件。

- `ApplicationEvent`：容器事件类，由`ApplicationContext`发布；
- `ApplicationListener`：监听器接口，由容器中的任意监听器Bean担任。

自定义事件需要继承`ApplicationEvent`类，监听器类则需要实现`ApplicationListener`接口，该接口有一个方法：

- `onApplicationEvent(ApplicationEvent event)`

`ApplicationContext`可以调用`publishEvent(ApplicationEvent evt)`方法来主动发布事件从而触发监听器。如果希望容器中的Bean也能够主动发布事件，就需要通过`ApplicationContextAware`或`BeanFactoryAware`接口在Bean内部添加对容器的引用。

监听器会监听到所有的事件，包括用户自定义事件和容器的内置事件。

容器的内置事件有以下几种：

- `ContextRefreshedEvent`：`ApplicationContext`容器初始化或刷新时触发该事件； 
- `ContextStartedEvent`：当使用`ConfigurableApplicationContext`接口的`start()`方法启动`ApplicationContext`容器时触发该事件；
- `ContextClosedEvent`：当使用`ConfigurableApplicationContext`接口的`close()`方法关闭`ApplicationContext`容器时触发该事件；
- `ContextStoppedEvent`：当使用`ConfigurableApplicationContext`接口的`stop()`方法停止`ApplicationContext`容器时触发该事件；
- `RequestHandledEvent`：只能应用于使用`DispatcherServlet`的Web应用中。当Spring处理完用户请求后触发该事件。

以下三个事件用于处理WebSocket功能服务：

- `SessioConnectedEvent`
- `SessionConnectEvent`
- `SessionDisconnectEvent`

#### 让Bean获取Spring容器 ####

通过让Bean实现`BeanFactoryAware`或`ApplicationContextAware`接口来为Bean内部添加对容器的引用。这两个接口各包含一个方法：

- `BeanFactoryAware`: `setBeanFactory(BeanFactory factory)`
- `ApplicationContextAware`: `setApplicationContext(ApplicationContext context)`

Spring容器会检查容器中所有的Bean，如果发现某个Bean实现了以上接口，Spring会在创建该Bean之后，自动调用这个方法将容器本身传递给该Bean。

### Spring容器中的Bean ###

`<beans.../>`元素是Spring配置文件中的根元素，具有以下属性：

- `default-lazy-init`：所有Bean默认的延迟初始化行为；
- `default-merge`：所有Bean默认的merge行为；
- `default-autowire`：所有Bean默认的自动装配行为；
- `default-autowire-candidates`：所有Bean默认是否作为自动装配的候选Bean；
- `default-init-method`：所有Bean默认的初始化方法；
- `default-destroy-method`：所有Bean默认的回收方法。

以上属性都可以在`<bean.../>`子元素中使用，只需要把属性名中的default去掉即可。

`<bean.../>`子元素用于定义一个Bean，每个Bean对应Spring容器中的一个Java实例。通常需要指定以下2个属性：

- `id`：该Bean的唯一标识，其他Bean的`<property.../>`或`<constructor-arg.../>`通过设置`ref`属性来引用此Bean。必须遵守XML文档的id属性命名规则，不能使用某些特殊字符如"/"作为属性值。如需使用特殊字符，可以通过设置Bean的别名并使用别名来访问Bean。2种方法设置别名：
	- 使用`<bean.../>`的`name`属性，多个别名可以使用逗号，冒号或者空格来分隔；
	- 使用`<alias.../>`元素设置：`<alias name="person" alias="jack"/> <alias name="jack" alias="jackee"/>`
- `class`：指定Bean的具体实现类，不能是接口或抽象类。

#### 容器中Bean的作用域 ####

Spring支持六种作用域，通过`<bean.../>`的`scope`属性设置：

- `singleton`：单例模式（默认模式），在整个容器中，该Bean将只有一个实例。容器负责跟踪Bean实例的状况，维护Bean实例的生命周期行为；
- `prototype`：每次通过容器的`getBean()`方法获取Bean时，都会生成一个新的Bean实例。容器不跟踪Bean实例的状况，也不维护Bean实例的生命周期行为；
- `request`：针对Web应用，每一次的HTTP请求都会生成一个新的Bean实例，在同一次HTTP请求内，每次请求该Bean都会得到同一个实例；
- `session`：针对Web应用中的每一次HTTP会话，该Bean都只会生成一个实例；
- `application`：针对整个Web应用，该Bean都只会生成一个实例；
- `websocket`：整个WebSocket通信过程中，该Bean都只会生成一个实例。

#### 配置依赖 ####

通常建议只使用配置文件管理容器中Bean与Bean之间的依赖关系，使用Java代码管理基本类型的属性值。

`BeanFactory`容器和`ApplicationContext`容器创建Bean的时机不同：

- `BeanFactory`在程序需要Bean实例时才会创建；
- `ApplicationContext`则在创建容器时就会预初始化容器中所有的单例模式的Bean。

#### 属性值类型-普通类型 ####

`<value.../>`元素用于指定基本类型及其包装类，字符串类型的属性值。

#### 属性值类型-其他Bean ####

`<ref.../>`元素指定属性值是容器中的另一个Bean。指定的属性值会覆盖自动装配的依赖。

自动装配注入Bean：通过`<beans.../>`元素的`default-autowire`属性或`<bean.../>`的`autowire`属性指定是否使用自动装配。其属性值包括：

- `no`：默认值，不使用自动装配。Bean依赖必须通过`ref`定义。
- `byName`：根据setter方法进行自动装配。将setter方法名去掉set前缀，并将首字母小写得到需要的Bean的标识，如果容器中没有该Bean，则不会进行任何注入。
- `byType`：根据setter方法的形参类型自动装配。在容器中查找类型和形参类型匹配的Bean，如果有多个Bean符合，则抛出异常，如果没有Bean符合，则不进行注入。父类或接口都有效。
- `constructor`：与`byType`类似，但根据构造器的形参类型自动装配。在容器中查找类型和构造器形参类型匹配的Bean，如果没有Bean符合，则抛出异常。
- `autodetect`：Spring容器根据Bean内部结构，自行决定使用`constructor`还是`byType`策略。如果该Bean的类型有一个默认的构造器，则使用`byType`策略。

当一个Bean既使用自动装配依赖，又使用`ref`属性显式指定依赖，则显式指定的依赖覆盖自动装配。

如果需要把某个Bean排除在自动装配机制之外（该Bean不被其他Bean引用），可以将该Bean的`autowire-candidate`属性设置为`false`。 
也可以在`<beans.../>`元素的`default-autowire-candidates`属性中使用模式字符串指定需要排除的Bean（可以使用多个模式字符串）。
项目较大时不鼓励使用自动装配。

#### 属性值类型-嵌套Bean ####

嵌套Bean类似Java中的内部类，只在其外部的`<property.../>`或`<constructor-arg.../>`元素作用域内有效，因此不能被其他Bean引用，因此也不需要指定`id`属性。

#### 属性值类型-集合类型 ####

如果需要调用形参类型是集合类型的setter方法或构造器，可以使用如下元素来设置：

- `<list.../>`：`List`和数组类型
- `<set.../>`：`Set`类型
- `<map.../>`：`Map`类型
- `<props.../>`：`Properties`类型

 `<list.../>`和`<set.../>`以及`<key.../>`元素又可以递归包含以上所有类型的子元素。
 
`<map.../>`元素中每一对key-value对都对应于一个`<entry.../>`子元素，可以包含`key`，`key-ref`，`value`，`value-ref`属性，因此其格式可以为：

		<map>
			<entry key="key1" value="value1"/>
			<entry key="key2" value-ref="bean1"/>
			<entry key-ref="bean2" value="value2"/>
			<entry key-ref="bean3" value-ref="bean4"/>
		</map>
			

 `<props.../>`元素设置key-value对的`Properties`类型，其格式为：

		<props key="key">value</props>
		<!-- 或者使用如下简写格式 -->
		<!-- 此格式中的属性名和属性值都只能是英文字母和数字，不能出现中文 -->
		<peroperty>
			<value>
				key1=value1
				key2=value2
			</value>
		</peroperty>

Spring的Bean之间也可以有继承关系，但这里的继承与Java中的继承不同，并没有产生新的类。父Bean和子Bean是同一个的不同实例，其中子Bean沿用了父Bean的属性值，也就是说，子Bean中的集合属性值可以从其父Bean的集合属性继承和覆盖而来，其最终值是父Bean和子Bean合并后的最终结果。而且子Bean集合中元素可以覆盖父Bean集合中的对应的元素。

如果Java代码中使用了泛型，则Spring会根据泛型信息把配置文件中的集合参数值转换为相应的数据类型。

#### 组合属性 ####

在配置文件中可以为如`foo.bar.name`的属性设置参数值。此时相当于调用`foo.getBar().setName()`方法。此时除最后一个属性外，其他属性值不能为null。也就是说`getBar()`不能返回null。否则抛出`NullPointerException`。

#### Spring的Bean和JavaBean区别 ####

Spring中的Bean建议遵循以下原则：

- 尽量为每一个Bean实现类提供无参数的构造器；
- 接受构造注入的Bean，应提供对应的，带参数的构造器；
- 接受设值注入的Bean，应提供相应的setter方法（getter方法不是必需）

Spring的Bean和JavaBean区别：

用处不同：JavaBean更多作为值对象传递参数；Spring中Bean是万金油，可以用于任何用途；
写法不同：JavaBean要求为每个属性都提供setter和getter方法；Spring的Bean只要求为设值注入的属性提供setter方法即可；
生命周期不同：JavaBean不接受任何容器的管理其生命周期；Spring的Bean则由容器管理其生命周期行为。

### 使用Java类来管理配置 ###

除了传统的XML配置文件，也可以使用Java类来做补充配置。

Java配置类中常用的注解：

- `@Configuration`：修饰一个Java配置类；
- `@Bean`：修饰一个方法，方法的返回值将定义为容器中的一个Bean；
- `@Value`：修饰一个Field，用于为该Field配置一个值，相当于配置了一个变量；
- `@Import`：修饰一个Java配置类，用于向当前Java配置类中导入其他Java配置类；
- `@Scope`：修饰一个方法，指定该方法对应的Bean的作用域；
- `@Lazy`：修饰一个方法，指定该方法对应的Bean是否需要延迟初始化；
- `@DependOn`：修饰一个方法，指定在初始化该方法对应的Bean之前需要先初始化的其他Bean。

XML配置文件和Java配置类可以结合使用：

1. 以XML配置文件为主，Java配置类为辅，需要在XML配置文件中添加以下代码：

		<context:annotation-config/>
		<!-- 加载Java配置类 -->
		<bean class="path.to.java.ConfigClass"/>

在代码中还是以此XML配置文件来创建Spring容器：

		ApplicationContext context = new ClassPathXmlApplicationContext("path.to.xmlConfig.xml");

2. 以Java配置类为主，以XML配置文件为辅：使用`@ImportResource`注解导入指定的XML配置文件。

		@Configuration
		//导入XML配置文件
		@ImportResource("classpath:/xmlConfig.xml")
		public Class SpringConfig {...}

相应地，应该在代码中使用该配置类来创建Spring容器：

		ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);

### 创建Bean的3种方式 ###

- 调用构造器
- 调用静态工厂方法
- 调用实例工厂方法

#### 调用构造器创建Bean ####

最常见的情况，必须指定`class`属性的值。如果没有使用构造注入，则必须为该Bean的类型提供无参数的构造器，其所有属性都会执行默认初始化。容器根据配置文件决定依赖关系，先初始化被依赖的Bean，然后为需要依赖的Bean注入被依赖的Bean。

#### 使用静态工厂方法创建Bean ####

同样必须指定`class`属性的值，但该值并不是Bean的实现类，而是负责生成该类实例的静态工厂类。另外还需要使用`factory-method`属性指定使用的静态工厂方法。

- `class`：指定使用的静态工厂类；
- `factory-method`：指定使用的静态工厂方法的名字。如果需要传入参数，使用`<constructor-arg.../>`元素。

通过这种方法创建的Bean实例仍然由Spring容器负责管理。

#### 使用实例工厂方法创建Bean ####

使用静态工厂方法创建Bean需要指定静态工厂类和使用的静态工厂方法即可，而使用实例工厂方法则需要工厂实例和使用的工厂方法。

- `factory-bean`：指定所使用的工厂Bean的标识；
- `factory-method`：指定实例工厂的工厂方法，如果需要传入参数，使用`<constructor-arg.../>`元素。

使用静态工厂方法和使用实例工厂方法的区别：使用实例工厂方法必须创建实例工厂类的Bean（实例）；静态工厂方法则不用创建Bean，直接通过`class`属性指定工厂类即可；

二者的相同之处：

- 都需要通过`factory-method`属性指定使用的工厂方法；
- 在需要时都通过`<constructor-arg.../>`元素传递参数；
- 普通的设值注入都是用`<property.../>`元素设定参数。

### 详解容器中的Bean ###

#### 抽象Bean和子Bean ####

把多个`<bean.../>`中相同的配置提取出来集中成配置模板，该模板不能被实例化，也不需要指定`class`属性，但必须指定其`abstract`属性为true，也就是抽象Bean。

抽象Bean因为不能被实例化，所以不能通过容器的`getBean()`方法来获取，也不能被其他Bean所依赖。其作用就是通过衍生子Bean来复用其配置信息。

子Bean可以从父Bean继承实现类，构造器参数，属性值等配置信息，并添加自己的新配置信息，新信息可以覆盖父Bean中的配置信息。
子Bean不能从父Bean继承以下属性，这些属性必须在子Bean中显式指定或者直接采取默认值：

- `depend`-on
- `autowire`
- `singleton`
- `scope`
- lazy-`init`

子Bean通过设置其`parent`属性指定其父Bean。子Bean和父Bean可以是不同类型。

#### 容器中的工厂Bean ####

工厂Bean和前面说的通过静态工厂类或者实例工厂创建Bean都不一样。
静态工厂类和实例工厂类都是工厂模式的实现，都是普通的Java类，没有特殊的要求。而工厂Bean是Spring框架中一种特殊的Bean，必须实现`FactoryBean`接口。

`FactoryBean`接口有3个方法：

- `T getObject()`：返回该工厂Bean生成的Java实例；
- `Class<?> getObjectType()`：返回该工厂Bean生成的Java实例的类型；
- `boolean isSingleton()`：该工厂Bean生成的Java实例是否是单例模式。

当容器调用`getBean()`方法获取该Bean时，得到的实例不是该Bean的类型的实例，而是其内部`getObject()`返回的Java类实例，起到类似转接器的作用。具体会获得什么类型的返回值完全取决于开发者在`getObject()`方法中的实现代码。如果需要返回该工厂Bean本身的实例，则需要在为`getBean()`方法指定所需Bean的标识前加`&`符号，如`getBean("&factoryBean")`。

需要注意的是，为工厂Bean设置的属性是在工厂Bean的类里定义的属性，而与工厂Bean实际返回的类型无关。

工厂Bean的执行流程可以理解为：

1. 实例化工厂Bean类
2. 使用setter方法设置`<property.../>`指定的属性值
3. 调用所生成工厂Bean实例的`getObject()`方法，将返回值作为容器中的Bean

#### 获取Bean本身的id ####

使用依赖注入通常不需要使用`getBean()`方法来获得Bean，因为Bean的ID在XML配置文件中指定，Java实例并不知道自己所代表的Bean的标识。为了获取该标识，该类型必须实现`BeanNameAware`接口，该接口只有一个方法`setBeanName(String name)`。与前面介绍的`ApplicationContextAware`接口类似，此方法是由Spring容器在创建Bean实例时自动调用，将配置中指定的Bean的ID传递给该实例（该实例内部应该有一个`String`类型的字段用于存放ID值）。

#### 强制初始化Bean ####

当依赖关系不够直接，Spring容器无法自动识别所有需要预先初始化的Bean时（如在某个Bean类的初始化块中使用了其他Bean），需要使用`depend-on`属性显式指定需要提前初始化的Bean实例。

### 容器中Bean的生命周期 ###

Spring容器只负责管理`singleton`作用域的Bean实例的生命周期行为。管理的时机主要有如下2个：

- 依赖注入完成
- Bean销毁之前

#### 依赖注入完成 ####

通过以下两种方式在Bean全部属性设置完成后执行特定行为：

- 使用`init-method`属性，配置在XML文件中完成，代码污染小；
- 实现`InitializingBean`接口（包含一个方法`void afterPropertiesSet() throws Exception`）。

如果同时使用了这两种方法，则先执行接口中的`afterPropertiesSet`方法，然后再执行`init-method`属性指定的方法。

#### Bean销毁之前 ####

通过以下两种方式在销毁Bean之前执行特定行为：

- 使用`destroy-method`属性，配置在XML文件中完成，代码污染小；
- 实现`DisposableBean`接口（包含一个方法`void destroy() throws Exception`）。

如果同时使用了这两种方法，则先执行接口中的`destroy`方法，然后再执行`destroy-method`属性指定的方法。

基于Web的`ApplicationContext`实现中，系统已经提供了相应的代码保证关闭Web应用时恰当地关闭Spring容器。非Web应用需要在JVM中注册一个关闭挂钩（shutdown hook），从而保证正确关闭Spring容器。注册方法：调用`AbstractApplicationContext`类中的`registerShutdownHook()`方法。

可以在根元素`<beans.../>`中通过设置`default-init-method`属性值为所有的Bean指定默认的`init-method`方法。

#### 协调作用域不同步的Bean ####

当被依赖的Bean的作用域是singleton时，不会产生作用域不同步的问题。这里的作用域不同步指的是Bean指定的作用域和它实际表现出的行为不一致。比如singleton的Bean bean1 依赖于prototype的Bean bean2，由于前者只有一个实例，且在初始化该实例时所有的属性已经设置完毕，依赖注入已完成，因此每次使用bean1其属性值也是固定的。也就是说，尽管其属性使用了prototype的bean2，但每次使用bean1时，bean2并不会生成新的实例，而是仍然使用在初始化bean1实例时赋给它的bean2实例。这违背了prototype的作用域的定义（每次使用都产生一个新的实例）。可以使用方法注入解决这个问题：

1. 将调用者Bean的实现类定义为抽象类，并为其定义一个抽象方法来获取被依赖的Bean；
2. 在`<bean.../>`元素中添加`<lookup-method.../>`子元素让Spring为调用者Bean的实现类实现指定的抽象方法。

`<lookup-method.../>`子元素需要指定以下两个属性：

- `name`：需要让Spring实现的方法；
- `bean`：指定Spring实现该方法的返回值。属性值是容器中定义的一个作用域为prototype的Bean的ID。

Spring实现该方法的逻辑是固定的，假设调用者Bean定义如下：

		<!-- 假设调用者Bean中定义如下： -->
		<bean .....>
			<lookup-method name="getDog" bean="gunDog"/>
		</bean>
		<bean id="gunDog" class="new.xiaoluo.service.GunDog" scope="prototype">
			<property name="name" value="puppet"/>
		</bean>
		
Spring容器总是采用如下代码实现该方法：

		// 假设抽象方法定义为
		// public Dog getDog();
		public Dog getDog(){
			//获取Spring容器ctx
			...
			return ctx.getBean("gunDog");
		}

通过这种方法，每次使用调用者Bean实例时，都会生成一个新的被依赖Bean实例传递给它的属性。

### 高级关系依赖配置 ###

通常组件和组件之间的耦合关系（依赖）才使用依赖注入管理，基本类型的成员变量值则直接在代码中设置。

在Spring中不仅可以通过依赖注入执行Java代码中的无参数或有参数构造器以及setter方法，还可以：

- 调用getter方法：使用`PropertyPathFactoryBean`
- 调用其他普通方法：使用`MethodInvokingFactoryBean`
- 访问类或对象的Field：使用`FieldRetrivingFactoryBean`

#### 获取其他Bean的属性值 ####

`PropertyPathFactoryBean`用来获取目标Bean的属性值（也就是该属性getter方法的返回值），获取的值可以注入给其他Bean，也可以定义为一个新的Bean。常用属性包括：

- `setTargetObject(Object targetObject)`：调用哪个对象，不能直接使用Bean的标识，必须通过`ref`。
- `setTargetBeanName(String targetBeanName)`：调用哪个Bean实例
- `setPropertyPath(String propertyPath)`：调用哪个getter方法

使用举例：

		<!-- 将指定Bean实例的getter方法返回值定义为新的Bean -->
		<bean id="bean2" class="org.springframework.beans.factory.config.PropertyPathFactoryBean">
			<!-- 确定目标Bean -->
			<property name="targetBeanName" value="bean1"/>
			<!-- 确定目标Bean的getter方法，
			可以使用组合属性如attr1.attr2,
			相当于： bean1.getAttr1().getAttr2() -->
			<property name="propertyPath" value="attr1"/>
		</bean>

配置`PropertyPathFactoryBean`工厂Bean时指定的`id`属性并不是该Bean的标识，因为返回的并不是该Bean类的实例，而是通过`propertyPath`指定的属性的getter方法的返回值。如果需要把返回值注入到其他Bean中，可以用`id`属性的值直接指定目标属性值:

		<property name="age">
			<!-- 访问指定Bean的getter方法的简化写法，直接使用id属性值指定目标方法
			相当于： person.getSon().getAge() -->
			<bean id="person.son.age" class="org.springframework.beans.factory.config.PropertyPathFactoryBean"/>
		</property>

目标Bean既可以是容器中已有的Bean，也可以是嵌套Bean。

可以使用`<util:property-path.../>`元素来简化`PropertyPathFactoryBean`的配置：

- 作为一个新的Bean：`<util:property-path id="bean2" path="bean1.attr1"/>`
- 注入到其他Bean：
		
		<bean id="age">
			<property>
				<util:property-path path="person.son.age"/>
			</property>
		</bean>

该元素可指定以下两个属性：

- `id`：指定新Bean（新Bean使用getter方法返回值定义）的标识；
- `path`：指定使用哪个Bean实例，哪个属性（支持组合属性）。

#### 获取Field值 ####

通过`FieldRetrivingFactoryBean`可以访问类的静态Field或对象的实例Field值。

- 访问静态Field：
	- `setTargetClass(String targetClass)`：调用哪个类
	- `setTargetField(String targetField)`：访问哪个Field
	- `setStaticField(String staticField)`：使用全限定类名+Field组成的字符串直接指定需要的Field
- 访问实例Field：
	- `setTargetObject(String targetObject)`：调用哪个对象
	- `setTargetField(String targetField)`：访问哪个Field

使用`<util:constant.../>`元素简化配置，可以指定以下属性：

- `id`：指定新Bean（新Bean使用Field值定义）的标识；
- `static-field`：指定使用哪个类的哪个静态Field。

#### 获取普通方法返回值 ####

`MethodInvokingFactoryBean`可以调用任意类方法和实例方法。如果调用的方法有返回值，既可以将返回值定义为新的Bean，也可以注入到其他Bean中。

- 静态方法：
	- `setTargetClass(String targetClass)`：调用哪个类
	- `setTargetMethod(String targetMethod)`：访问哪个Method
	- `setArgument(Object[] arguments)`：调用方法的参数
- 实例方法：
	- `setTargetObject(String targetObject)`：调用哪个对象
	- `setTargetMethod(String targetMethod)`：访问哪个Method
	- `setArgument(Object[] arguments)`：调用方法的参数
	
#### 使用Spring配置文件管理Java代码的总结 ####

- 调用构造器创建对象（包括使用工厂方法创建对象），用`<bean.../>`元素
- 调用setter方法，使用`<property.../>`元素
- 调用getter方法，使用`PropertyPathFactoryBean`工厂Bean或`<util:property-path.../>`元素
- 调用普通方法，使用`MethodInvokingFactoryBean`工厂Bean
- 获取Field的值，使用`FieldRetrivingFactoryBean`工厂Bean或`<util:constant.../>`元素。虽然可以访问实例成员变量，但通常都是对类成员变量取值，因为推荐是通过getter方法来访问实例成员变量。

通常只应该把以下两类信息放入配置文件中管理：

- 项目升级，维护需要经常改动的信息
- 控制项目内各组件耦合关系的代码

### 基于XML Schema的简化配置方式 ###

#### 使用p:命名空间简化设值注入配置 ####

		<!-- 导入命名空间 -->
		<beans ...
			xmlns:p="http://www.springframework.org/schema/p"
			...>
		<!-- 使用命名空间简化对属性setter方法的调用 -->
		<!-- 如果值不是基本类型而是Bean，则必须在属性名后加上-ref -->
		<!-- 所以Bean的名称本身不能含有-ref -->
		<bean id="person" class="net.xiaoluo.service.Person" p:name="xiao" p:address-ref="adress1"/>

#### 使用c:命名空间简化构造注入配置 ####

		<!-- 导入命名空间 -->
		<beans ...
			xmlns:p="http://www.springframework.org/schema/c"
			...>
		<!-- 使用命名空间简化对构造器的调用 -->
		<bean id="person" class="net.xiaoluo.service.Person" c:name="xiao" c:address-ref="adress1"/>

在`c:`后除了使用参数名指定参数，还可以通过索引来配置构造器参数：`c:_N`，其中N代表参数在参数列表中的索引（从0开始）。

另外，可以使用`java.beans`包内的`@ConstructorProperties`注解为构造器参数显式命名，然后在配置文件中使用它们指定参数。

		//Java类
		public class Person{
			private String name;
			private int age;
			@ConstructorProperties("personName", "age")
			public Person(String name, int age) {
				this.name=name;
				this.age=age;
			}
		}

		<!-- XML配置文件 -->
		<bean id="person" class="net.xiaoluo.service.Person">
			<constructor-arg name="age" value="23"/>
			<constructor-arg name="personName" value="xiao"/>
		</bean>
		<!-- 或者使用c:命名空间简化配置 -->
		<bean id="person" class="net.xiaoluo.service.Person" c:age="23" c:personName="xiao"/>

#### 使用util:命名空间简化配置 ####

`xmlns:util="http://www.springframework.org/schema/util`

util命名空间提供了如下几个元素：
		
- `constant`：获取静态Field值，是`FieldRetrivingFactoryBean`工厂Bean的简化配置
- `property-path`：获取getter方法的返回值，是`PropertyPathFactoryBean`工厂Bean的简化配置
- `list`：定义一个`List`Bean，支持使用`<value.../>`，`<ref.../>`，`<bean.../>`等子元素来定义`List`集合元素，支持以下3个属性：
	- `id`：Bean的标识
	- `list-class`：使用哪个`List`实现类创建Bean实例，默认使用`ArrayList`
	- `scope`：指定该Bean实例的作用域
- `set`：定义一个`Set`Bean，支持使用`<value.../>`，`<ref.../>`，`<bean.../>`等子元素来定义`Set`集合元素，支持以下3个属性：
	- `id`：Bean的标识
	- `set-class`：使用哪个`Set`实现类创建Bean实例，默认使用`HashSet`
	- `scope`：指定该Bean实例的作用域
- `map`：定义一个`Map`Bean，支持使用`<entry.../>`定义`Map`的key-value对，支持以下3个属性：
	- `id`：Bean的标识
	- `map-class`：使用哪个`Map`实现类创建Bean实例，默认使用`HashMap`
	- `scope`：指定该Bean实例的作用域
- `properties`：加载一份资源文件，并以此创建一个`Properties`Bean实例
	- `id`：Bean的标识
	- `location`：指定资源文件的位置
	- `scope`：指定该Bean实例的作用域

其他常用的简化Schema：

- spring-aop.xsd
- spring-jee.xsd
- spring-jms.xsd
- spring-lang.xsd
- spring-tx.xsd

### Spring的表达式语言SpEL ###

SpEL可以只当成简单的表达式语言独立于Spring容器使用，也可以在注解或XML配置中使用。

#### 使用Expression接口进行表达式求值 ####

SpEL 主要提供了如下3个接口：

- `ExpressionParser`：负责解析一个SpEL表达式，返回一个`Expression`对象；
- `Expression`：代表一个表达式；
- `EvaluationContext`：代表计算表达式的上下文。当SpEL表达式中含有变量时，需要使用该API计算表达式的值。

`Expression`实例代表一个表达式，包含如下方法用于计算表达式的值：

- `Object getValue()`：计算表达式的值；
- `<T> T getValue(Class<T> desiredResultType)`：计算表达式的值，并尝试将该值当做`desiredResultType`类型处理；
- `Object getValue(EvaluationContext context)`：使用指定的`EvaluationContext`计算表达式的值；
- `<T> T getValue(EvaluationContext context, Class<T> desiredResultType)`：使用指定的`EvaluationContext`计算表达式的值，并尝试将该值当做`desiredResultType`类型处理；
- `Object getValue(Object rootObject)`：以`rootObject`作为表达式的根对象来计算表达式的值；
- `<T> T getValue(Object rootObject, Class<T> desiredResultType)`：以`rootObject`作为表达式的根对象来计算表达式的值，并尝试将该值当做`desiredResultType`类型处理；

`EvaluationContext`代表SpEL计算表达式值的上下文，可以包含多个对象，但只能有一个根对象，类似于OGNL中的`Stack Context`，当表达式中包含变量时，会根据`EvaluationContext`中变量的值对表达式求值。使用以下方法往`EvaluationContext`中放入对象：

- `setVariable(String name, Object value)`

在SpEL中访问这些对象是，使用与OGNL类似的格式：`#name`

访问根对象的属性时，可以省略根对象的前缀。`StandardEvaluationContext`提供了以下方法来设置根对象：

- `setRootObject(Object rootObject)`

#### Bean定义中的表达式语言支持 ####

在XML配置文件和注解中使用SpEL时，需要在表达式外面添加`#{}`包围即可。在表达式内部可以调用方法，创建对象（代替嵌套Bean语法），访问其他Bean的属性等。

#### SpEL语法 ####

**直接量表达式**

在表达式中直接使用Java语言支持的直接量，如字符串，日期，数值，boolean和null。

		Expression exp = parser.parseExpression("'Hello World'");
		System.out.println(exp.getValue(String.class));
		exp = parser.parseExpression("0.23");
		System.out.println(exp.getValue(Double.class));

**在表达式中创建数组**

支持使用静态初始化，动态初始化两种语法创建数组：

		Expression exp = parser.parseExpression("new String[]{'java', 'Struts', 'Spring'}");
		System.out.println(exp.getValue());
		exp = parser.parseExpression("new int[2][4]");
		System.out.println(exp.getValue());
		
**在表达式中创建List集合**

使用如下语法直接创建`List`集合：`{ele1, ele2, ele3...}`

		Expression exp = parser.parseExpression("{'java', 'Struts', 'Spring'}");
		System.out.println(exp.getValue());
		exp = parser.parseExpression("{{'China', 'France'}, {'Apple', 'Orange'}}");
		System.out.println(exp.getValue());

**在表达式中访问List，Map集合元素**

- 访问`List`集合元素：`#listName[index]`
- 访问`Map`集合元素：`#mapName[index]`

**调用方法**

和普通Java代码调用方法一样，只是需要在变量名前加上`#`表示引用的是`EvaluationContext`中的变量。

**算数，比较，逻辑，赋值，三目等运算符**

SpEL表达式的赋值运算符可以直接改变表达式所引用的实际对象。

**类型运算符**

`T()`运算符：将运算符括号内的字符串当成类处理，避免Spring对其进行其他解析。在调用静态方法时尤其有用。

		System.out.println(parser.parseExpression("T(java.lang.Math).random()").getValue());

在表达式中使用类时，如果使用的是`java.lang`包下的类，可以省略包名，否则推荐使用该类的全限定类名。

**变量**

参见前面关于`EvaluationContext`变量的说明。SpEL中有2个特殊变量：

- `#this`：正在计算的对象
- `#root`：`EvaluationContext`的根对象

**自定义函数**

就是把Java代码中的方法注册为自定义函数。因为可以直接调用Java方法，所以意义不大。使用`StandardEvaluationContext`中的`registerFunction(String name, Method m)`注册。

**Elvis运算符**

三目运算符的特殊写法：

		//三目运算符
		name != null ? name : "newVal"
		//Elvis运算符
		name?:"newVal"

**安全导航运算符**

在SpEL表达式中使用如下语句可能导致`NullPointerException`异常（当`foo`为null时）：`#foo.bar`。为了避免发生异常，使用`#foo?.bar`，当`foo`为null时，结算结果直接返回null，而不是抛出异常。

**集合元素筛选**

表达式中可以对集合元素设置筛选条件，符合条件的集合元素才会返回：`#collection.?[conditon_expression]`

- 当集合是`List`类型时，`conditon_expression`中访问的每个属性和方法都是以集合元素为调用者的。
- 当集合是`Map`类型时，需要显式地用`key`引用Map Entry的key，用`value`引用Map Entry的value。

**集合投影**

将集合里的每个元素使用指定的表达式进行计算得出一个新的结果，这些新结果组成的新集合就是该表达式返回的值：`#collection.![conditon_expression]`

**表达式模板**

类似于带占位符的国际化消息。在直接量表达式中插入一个或多个`#{expr}`占位符，在动态解析时，`expr`的值会被动态计算出来。

使用`ExpressionParser`解析字符串模板时需要传入一个`TemplateParserContext`参数，`expr`必须和对象的属性名相同，从而能够将该属性的值在运行时传递给表达式。

		Person p = new Person("xiaoluo", 23);
		Expression expr = parser.parseExpression("我的名字是#{name}，我的年龄是#{age}", new TemplateParserContext());
		System.out.println(expr.getValue(p));
		
## 深入使用Spring ##

### 两种后处理器 ###

两种常用的后处理器类型：

- Bean后处理器：对容器中的Bean进行后处理，进行额外增强；
- 容器后处理器：对容器进行后处理，进行额外增强；

#### Bean后处理器 ####

Bean后处理器是一种特殊的Bean，不对外提供服务，无须id属性，负责对容器中的其他Bean执行后处理。会在Bean实例创建成功之后，对Bean实例进行进一步的增强。

Bean后处理器必须实现`BeanPostProcessor`接口，包含以下两个方法：

- `Object postProcessBeforeInitialization(Object bean, String name) throws BeanException`：在目标Bean初始化之前执行，第一个参数是系统即将进行后处理的Bean实例，第二个参数是该Bean的id。
- `Object postProcessAfterInitialization(Object bean, String name) throws BeanException`：在目标Bean初始化之后执行，第一个参数是系统即将进行后处理的Bean实例，第二个参数是该Bean的id。

Bean实例化的过程总结：

1. 执行Bean的构造器
2. 执行Bean的依赖注入（setter方法）
3. 执行后处理器的`postProcessBeforeInitialization`方法
4. 执行`InitializingBean`接口的`afterPropertiesSet`方法
5. 执行在xml配置文件中指定的`init-method`方法
6. 执行后处理器的`postProcessAfterInitialization`方法

`ApplicationContext`作为Spring容器会自动检测容器中的所有Bean，如果发现某个Bean实现了`BeanPostProcessor`接口，就会自动将其注册为Bean后处理器。如果使用`BeanFactory`作为Spring容器，则必须手动注册Bean后处理器，因此需要在XML配置文件中为Bean后处理器的Bean指定id，从而允许在程序中取得并注册它的Bean实例。

		public class BeanTest {
			public static void main(String[] args) {
				// 搜索类加载路径下的beans.xml文件创建Resource对象
				Resource isr = new ClassPathResource("beans.xml");
				// 创建默认的BeanFactory
				DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
				// 让默认的BeanFactory容器加载isr对应的XML配置文件
				new XmlBeanDefinitionReader(beanFactory).loadBeanDefinitions(isr);
				// 获取容器中的Bean后处理器，bp必须是在XML配置文件中定义的Bean的id
				BeanPostProcessor bp = (BeanPostProcessor) beanFactory.getBean("bp");
				// 注册Bean后处理器
				beanFactory.addBeanPostProcessor(bp);
			}
		}

#### Bean后处理器的用处 ####

Spring中常用的2种Bean后处理器：

- `BeanNameAutoProxyCreator`：根据Bean实例的name属性，创建Bean实例的代理；
- `DefaultAdvisorAutoProxyCreator`：根据提供的`Advisor`，为容器中的所有Bean实例创建代理。

Bean的代理就是对目标Bean进行增强，在其基础上进行修改得到新的Bean。

#### 容器后处理器 ####

容器后处理器负责对容器本身进行增强，必须实现`BeanFactoryPostProcessor`接口，包含一个方法：

- `postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)`

和Bean后处理器类似，`ApplicationContext`作为容器会自动检测并注册实现了`BeanFactoryPostProcessor`接口的的Bean，而如果使用`BeanFactory`作为容器，则需要手动调用该容器后处理器处理容器。Spring没有提供`ApplicationContextPostProcessor`，因此`ApplicationContext`也使用`BeanFactoryPostProcessor`作为容器后处理器。

Spring中常用的容器后处理器：

- `PropertyPlaceholderConfigurer`：属性占位符配置器
- `PropertyOverrideConfigurer`：重写占位符配置器
- `CustomAutowireConfigurer`：自定义自动装配的配置器
- `CustomScopeConfigurer`：自定义作用域的配置器

容器后处理器总是在容器实例化任何其他Bean之前，读取配置文件的元数据，并有可能修改这些元数据。可以配置多个容器后处理器，通过设置`order`属性（要求容器后处理器必须实现`Ordered`接口，推荐让所有容器后处理器都实现该接口）来控制容器后处理器的执行顺序。

#### 属性占位符配置器 ####

`PropertyPlaceholderConfigurer`负责读取`Properties`属性文件里的属性值，并将这些属性值配置成Spring配置文件中的数据。这样可以将Spring配置文件中的特定数据分离出来放在指定属性文件中。支持使用多个属性文件，但推荐不要过多地将Spring配置信息抽离出来以免降低配置文件的可读性。

采用基于XML Schema的配置文件可以导入`context:`命名空间，并使用如下方式配置属性占位符：

		<!-- location指定Properties文件位置 -->
		<context:property-placeholder location="classpath:db1.properties, classpath:db2.properties"/>

#### 重写占位符配置器 ####

`PropertyOverrideConfigurer`指定的属性文件中的信息可以覆盖Spring配置文件中的元数据。此时可以认为Spring配置信息是XML配置文件和属性文件的总和，如果出现冲突，属性文件的信息优先。

属性文件中的每条属性应保持如下格式：`beanId.property=value`，其中`beanId`必须是容器中实际存在的Bean的id，否则会出错。如果多于一个`PropertyOverrideConfigurer`对同一属性进行了覆盖，最后一次为准。

采用基于XML Schema的配置文件可以导入`context:`命名空间，并使用如下方式配置重写占位符：

		<!-- location指定Properties文件位置 -->
		<context:property-override location="classpath:db1.properties, classpath:db2.properties"/>

### Spring零配置支持 ###

#### 搜索Bean类 ####

Spring没有采用约定优于配置的策略，需要显式指定搜索Bean类的路径。通过引入`context`Schema来指定简单的搜索路径：

		<!-- 搜索包含子包 -->
		<context:component-scan base-package="org.xiaoluo.net.service"/>

可以为`<context:component-scan.../>`标签添加`<include-filter.../>`和`<exclude-filter.../>`子元素来指定Spring Bean类。

- `<include-filter.../>`：强制将指定类作为Bean类处理，即使他们没有使用Spring注解标注；
- `<exclude-filter.../>`：强制不将指定类作为Bean类处理，即使他们使用了Spring注解标注；

这两个子元素都要求指定如下两个属性：

- `type`：过滤器类型
- `expression`：过滤器所需的表达式

Spring內建支持如下4种过滤器：

- `annotation`：注解过滤器，需要指定一个注解名
- `assignable`：类名过滤器，直接指定一个类
- `regex`：正则表达式过滤器，指定类需要满足的正则表达式
- `aspectj`：AspectJ过滤器

Spring中用来标注Spring Bean的注解：

- `@Component`：普通Bean类
- `@Controller`：控制器组件类
- `@Serivce`：业务逻辑组件类
- `@Repository`：DAO组件类

尽量使用后面三个注解为标注的Bean类添加语义。

使用XML配置方式时，每个Bean实例的名称都是由其id属性指定；使用注解方式时，Spring约定使用这些Bean类的首字母小写作为这些Bean实例的名字，其他部分不变。也可以在使用注解时显式指定Bean实例的名称。

`@Lookup`注解的作用和`<lookup-method.../>`元素一样，直接标注需要执行`lookup`方法注入的方法，需要指定一个value属性，该属性等同于`<lookup-method.../>`元素的`bean`属性，即`lookup`方法注入所返回的Bean的id。

#### 指定Bean的作用域 ####

使用`@Scope`注解指定作用域的范围。

自定义的解析器需要实现`ScopeMetadataResolver`接口，并提供自定义的作用域解析策略，然后在配置扫描器时指定解析器的全限定类名。

		<context:component-scan base-package="net.xiaoluo.app" scope-resovler="net.xiaoluo.app.util.MyScopeResolver"/>

从Spring4.3开始添加了`@ApplicationScope`，`@SessionScope`，`@RequestScope`注解，且`proxyMode`属性被设置为`ScopedProxyMode.TARGET_CLASS`。

#### 使用@Resource和@Value配置依赖 ####

`@Resource`注解位于`javax.annotation`包下，通过该注解为目标Bean指定协作Bean。

`@Resource`有一个`name`属性，默认情况下，该值是需要被注入的`Bean`实例的`id`，换句话说，使用`@Resource`与`<property.../>`元素的`ref`属性有相同的效果。

`@Value`则相当于`<property.../>`元素的`value`属性，用于为`Bean`的基本类型属性配置属性值。该注解可以使用表达式。

`@Resource`和`@Value`不仅可以修饰`setter`方法，还可以直接修饰实例变量，此时`Spring`将会直接使用`JavaEE`规范的Field注入。

当使用省略了`name`属性的`@Resource`标注setter方法时，`name`属性的值默认为该setter方法去掉set前缀后首字母小写所得到的字符串；当标注实例变量时，默认值与该实例变量同名。

#### 使用@PostConstruct和@PreDestroy定制生命周期行为 ####

@PostConstruct和@PreDestroy同样位于javax.annotation包下。他们的行为和XML配置文件中指定`<bean.../>`元素的`init-method`和`destroy-Method`属性的效果大致一样，分别用于标准初始化方法和销毁方法。

#### 使用@DependOn和@Lazy改变初始化行为 ####

- `@DependOn`用于强制初始化其他Bean，可以标注类或方法，使用该注解时可以指定一个字符串数组作为参数，每个数组元素对应于一个强制初始化的Bean；
- `@Lazy`指定该Bean是否取消预初始化，可指定一个`boolean`类型的`value`属性。

#### 自动装配和精确装配 ####

`@Autowired`注解指定自动装配，可以修饰setter方法，普通方法，实例变量和构造器等。  
标注setter方法时，默认采用`byType`自动装配策略。  
修饰实例变量时，会把容器中与该实例变量类型匹配的Bean设置为该实例变量的值。  
如果标注的是数组类型的成员变量，则会自动搜索容器中所有该类型的Bean实例，并将这些Bean实例作为数组元素来创建数组，然后将该数组赋给该数组类型的变量。  
对集合类型的实例变量或形参为集合的方法的处理与数组完全相同，但要求这些集合必须使用泛型指定了集合元素的类型。

如果容器中出现多个可匹配类型，会导致异常。Spring提供了一个`@Primary`注解，用于将指定的候选Bean作为当选者。

4.0后的Spring的`@Autowired`注解可以根据泛型形参自动装配。

		// DAO组件类
		public class BaseDaoImpl<T> implements BaseDao<T> {
			public void save(T e) {
				System.out.println("Save Object:" + e);
			}
		}
		-------------
		@Component("userDao")
		public class UserDaoImpl extends BaseDaoImpl<User> implements UserDao {}
		-------------
		@Component("itemDao")
		public class ItemDaoImpl extends BaseDaoImpl<Item> implements ItemDao {}
		--------------------
		// Service组件类
		public class BaseServiceImpl<T> implements BaseService<T> {
			// 自动装配会根据T的运行时类型注入相应的BaseDao类型或其子类的Bean实例
			@Autowired
			private BaseDao<T> dao;
			public void addEntity(T entity) {
				System.out.println("call " + dao + ", save " + entity);
			}
		}
		-------------
		@Component("userService")
		public class UserServiceImpl extends BaseServiceImpl<User> implements UserService {}
		-------------
		@Component("itemService")
		public class ItemServiceImpl extends BaseServiceImpl<Item> implements ItemService {}

`@Qualifier`注解允许根据指定Bean的`id`来执行自动装配。可以标注实例变量或方法形参。

`@Autowired`和`@Qualifier`组合实现精确自动装配可以用`@Resource`注解直接执行依赖注入。

#### Spring5新增的注解 ####

使用`@Autowired`注解可以指定一个`required`属性，默认为`true`，表示该注解修饰的Field或setter方法必须被依赖注入，否则Spring在初始化容器时报错。

`@Autowired`和在XML配置文件中指定`autowire="byType"`的自动装配不一样，如果找不到自动装配的候选Bean，XML配置的容器只是不执行注入，不会报错，但使用`@Autowired`配置就会报错。有两种方法可以避免报错，其实只是不执行依赖注入：

- 将`@Autowired`的required属性设置为false
- 使用Spring5新增的注解`@Nullable`

		@Autowired(required=false)
		public void setName(@Nullable String name){}

两种方法任选其一。

Spring5还新增了如下注解：

- `@NonNull`：可标注参数，返回值和Field；
- `@NonNullApi`：标注包，表明该包内API的参数，返回值都不能为null，如果允许某些参数为null，则单独使用`@Nullable`标注他们；
- `@NonNullFields`：标注包，表明该包内API的Field都不能为null，如果允许某些Field为null，则单独使用`@Nullable`标注他们；

#### 使用@Required检查注入 ####

`@Required`注解标注setter方法，表示必须显式通过`<property.../>`元素或自动装配为其配置依赖注入，否则Spring容器会抛出`BeanInitializationException`异常。

### 资源访问 ###

资源访问通常是由`java.net.URL`和文件IO完成。Spring为资源访问提供了一个`Resource`接口，是具体资源访问策略的抽象，也是所有资源访问类所实现的接口。主要包含以下几个方法：

- `getInputStream()`：打开资源，返回资源对应的输入流。每次调用都会返回新的输入流，调用者必须负责关闭输入流；
- `exists()`：返回资源是否存在；
- `isOpen()`：返回资源是否打开，如果资源不能多次读取，每次读取结束时都应该显式关闭；
- `getDescription()`：返回资源的描述信息，用于资源处理出错时输出该信息，通常是全限定文件名或实际URL；
- `getFile()`：返回资源对应的File对象；
- `getURL()`：返回资源对应的URL对象。

`Resource`可以在Spring项目之外单独作为资源访问的工具使用。

#### Resource的实现类 ####

- `UrlResource`：访问网络资源的实现类
- `ClassPathResource`：访问类加载路径下资源的实现类
- `FileSystemResource`：访问文件系统里的资源的实现类
- `ServletContextResource`：访问ServletContext路径下资源的实现类
- `InputStreamResource`：访问输入流资源的实现类
- `ByteArrayResource`：访问字节数组资源的实现类

**网络资源的访问：**

`UrlResource`是`java.net.URL`类的包装，用于访问之前通过URL类访问的资源对象，支持的前缀与URL类所支持的前缀完全相同。如：`file:`用于文件系统，`http:`用于通过HTTP协议访问资源，`ftp:`用于通过FTP协议访问资源等。

**类加载路径下的资源的访问**

`ClassPathResource`用来访问类加载路径下的资源。虽然可以使用构造器显式创建该类的实例，但更多时候则是隐式创建：当执行Spring的某个方法，而该方法的参数包含一个代表资源路径的字符串参数，那么Spring会根据该字符串是否包含`classpath:`前缀来自动创建`ClassPathResource`的实例。

**文件系统资源的访问**

`FileSystemResource`用于访问文件系统中的资源。虽然可以使用构造器显式创建该类的实例，但更多时候则是隐式创建：当执行Spring的某个方法，而该方法的参数包含一个代表资源路径的字符串参数，那么Spring会根据该字符串是否包含`file:`前缀来自动创建`FileSystemResource`的实例。

**Web应用资源的访问**

`ServletContextResource`用于访问Web应用相对路径下的资源，其构造器接受一个代表资源位置路径的字符串参数，该资源位置是相对于Web应用根路径的位置。

使用`ServletContextResource`访问的资源，也可以通过文件IO或URL访问，通过`java.io.File`访问要求资源被解压缩，而且必须在本地文件系统中；使用`ServletContextResource`则无须关心资源是否已被解压，还是直接存放在jar文件中，都可以通过Servlet容器访问。

如果程序视图通过File来访问Web应用相对路径下的资源时，必须先使用`ServletContext`的`getRealPath()`方法来获得资源绝对路径，再以该绝对路径来创建File对象。

**字节数组资源的访问**

`InputStreamResource`可以访问二进制输入流资源，只有当没有合适的`Resource`实现类时，才考虑使用`InputStreamResource`。通常优先考虑使用`ByteArrayResource`，或者基于文件的`Resource`实现。

与其他Resource的实现类不同，`InputStreamResource`是一个总是被打开的资源，所以`isOpen()`方法总是返回true。如果需要多次读取某个流，就不要使用`InputStreamResource`。

`ByteArrayResource`用于直接访问字节数组资源。对于需要采用`InputStreamResource`访问的资源，可以先从`InputStream`流中读出字节数组，然后以该字节数组创建`ByteArrayResource`实例，通过这种转换实现对该资源的多次读取。

#### ResourceLoader接口和ResourceLoaderAware接口 ####

- `ResourceLoader`：该接口实现类的实例可以获得一个`Resource`实例
- `ResourceLoaderAware`：该接口实现类的实例可以获得一个`ResourceLoader`的引用

`ResourceLoader`接口仅包含如下方法：

- `Resource getResource(String location)`：返回一个`Resource`实例。`ApplicationContext`的实现类都实现了`ResourceLoader`接口，因此`ApplicationContext`可以直接获取`Resource`实例，默认采取和该`ApplicationContext`实例相同的资源访问策略：
	- `FileSystemXmlApplicationContext`对应`FileSystemResource`
	- `ClassPathXmlApplicationContext`对应`ClassPathResource`
	- `XmlWebApplicationContext`对应`ServletContextResource`

Spring应用需要访问资源时，并不需要直接使用`Resource`实现类，而是调用`ResourceLoader`实例的`getResource()`方法来获得资源，`ResourceLoader`负责选择`Resource`的实现类，确定具体的访问策略，从而将应用和具体的资源访问策略分离。

使用`ResourceLoader`的`getResource()`方法获取资源时，可以在代表资源路径的字符串前加上前缀来强制使用某种策略（`Resource`实现类）：

- `classpath:`：`ClassPathResource`实现类
- `file:`：`UrlResource`实现类
- `http:`：`UrlResource`实现类
- 无前缀：由`ResourceLoader`实现类来决定访问策略

`ResourceLoaderAware`和`BeanFactoryAware`，`BeanNameAware`接口类似，提供了一个`setResourceLoader()`方法，该方法由Spring容器负责调用，并将一个`ResourceLoader`实例作为方法的参数传入，因此该接口的实现类必须拥有一个`ResourceLoader`类型的成员变量。如果`ApplicationContext`容器中含有`ResourceLoaderAware`接口的实现类的Bean，则容器会把自身作为参数传给该Bean的`setResourceLoader()`方法。

#### Bean需要访问资源 ####

有2种方法：

- 前面介绍的在代码中获取`Resource`实例。因为必须在代码中提供资源位置，所以资源的物理位置被耦合到了代码中，如果资源位置发生了变化，必须改写程序。
- 推荐使用依赖注入：声明`Resource`类型的成员变量，并为其提供setter方法。这样可以将资源位置的设置从代码中迁移到XML配置文件中，从而实现了二者的解耦。

#### 在ApplicationContext中使用资源 ####

无论以什么方式创建`ApplicationContext`实例，都需要为其指定配置文件（可以使用一份或多份配置文件）。这些配置文件也是以`Resource`的方式来访问，因此支持使用所有的`Resource`实现类（策略）。指定策略通常有2种方法：

- 使用`ApplicationContext`实现类指定访问策略；
	- `FileSystemXmlApplicationContext`对应`FileSystemResource`
	- `ClassPathXmlApplicationContext`对应`ClassPathResource`
	- `XmlWebApplicationContext`对应`ServletContextResource`
- 使用前缀指定访问策略，该策略只对当次访问有效。因此推荐使用`ApplicationContext`实现类来指定访问策略。

`classpath*:`前缀的使用：允许加载多个配置文件，这些配置文件散布在类加载路径中，Spring会找出所有名称匹配指定过滤规则（可以使用通配符）的配置文件，合并这些文件中的配置并创建Spring容器。`classpath:`只加载搜索到的第一个匹配的配置文件，搜索的顺序取决于类加载路径的顺序。

`classpath*:`只能用于`ApplicationContext`，不能用于`Resource`，也即不能使用`classpath*:`一次性访问多个资源。

**`file:`前缀的使用：**

当使用`FileSystemXmlApplicationContext`作为`ResourceLoader`使用时，Spring容器会让所有绑定的`FileSystemResource`实例把绝对路径当做相对路径处理，而不管是否以斜杠开头。即以下两行代码的效果完全相同：

		ApplicationContext ctx = new FileSystemXmlApplicationContext("beans.xml");
		ApplicationContext ctx = new FileSystemXmlApplicationContext("/beans.xml"); // 貌似绝对路径，但仍然当做相对路径处理

如果需要使用绝对路径，不要依靠使用`FileSystemXmlApplicationContext`或`FileSystemResource`来指定绝对路径，建议强制使用`file:`前缀来区分相对路径和绝对路径：

		ApplicationContext ctx = new FileSystemXmlApplicationContext("file:beans.xml");
		ApplicationContext ctx = new FileSystemXmlApplicationContext("file:/beans.xml");
		
### Spring的AOP ###

AOP(Aspect Orient Programming)：面向切面编程。
OOP是从静态角度考虑程序结构，AOP是从动态角度考虑程序的运行过程。

AOP专门用于处理系统中分布于各个模块（不同方法）中的交叉关注点的问题。在JavaEE应用中，常常通过AOP处理一些具有横切性质的系统级服务。

#### 使用AspectJ实现AOP ####

AspectJ是Java语言的AOP实现，主要包括2个部分：

- 语法规范，定义如何表达，定义AOP编程中的语义
- 工具部分，包括编译器，调试工具等。

AspectJ在编译时增强了Java类的功能，因此AspectJ常被称为编译时增强的AOP框架。

AOP的主要功能是保证程序员不改变源代码的前提下，为系统中业务组件的多个业务方法添加某种通用功能。但实质是AOP仍然要取修改这些业务方法的源代码，只是这个过程不再由程序员手工完成，而是由AOP框架来负责完成。

AOP的实现分为2类（按AOP框架修改源代码的实际）：

- 静态AOP实现：AOP框架在编译阶段对程序进行修改，实现对目标类的增强，生成静态的AOP代理类（生成的.class文件是修改后的版本，需要使用特定的编译器），代表就是AspectJ；
- 动态AOP实现：AOP框架在运行阶段动态生成AOP代理（在内存中以JDK动态代理或cglib动态地生成AOP代理类），以实现对目标对象的增强，代表是Spring AOP。

#### AOP的基本概念 ####

AOP从程序运行的角度考虑程序的流程，提取业务处理过程的切面。AOP框架并不与特定的代码耦合，能处理程序执行中特定的切入点（Pointcut)，而不与某个具体类耦合。AOP框架具有以下2个特征：

- 各步骤之间的良好隔离性
- 源代码无关性

基本术语：

- 切面（Aspect）：切面用于组织多个Advice，Advice在切面中定义；
- 连接点（Joinpoint）：程序执行过程中明确的点，如方法的调用，异常的抛出。在Spring AOP中，连接点总是方法的调用；
- 增强处理（Advice）：AOP框架在特定的切入点执行的增强处理。
- 切入点（Pointcut）：可以插入增强处理的连接点。即当某个连接点满足指定条件时，该连接点将添加增强处理，该连接点也变成了切入点。如何使用表达式来定义切入点是AOP的核心，Spring默认使用AspectJ切入点语法。 
- 引入：将方法或字段添加到被处理的类中。Spring允许将新的接口引入到被处理的对象中；
- 目标对象：被AOP框架进行增强处理的对象。如果AOP框架采取的是动态实现，则该对象就是被代理的对象；
- AOP代理：AOP框架创建的对象，代理对象就是目标对象的加强版。Spring中的AOP代理可以是JDK动态代理（为实现接口的目标对象代理），也可以是cglib代理（为不实现接口的对象代理）；
- 织入（Weaving）：将增强处理添加到目标对象中，并创建以个被增强的对象（AOP代理）的过程就是织入。织入有两种实现方式：编译时增强（如AspectJ）和运行时增强（如Spring AOP）。

AOP代理就是有AOP框架动态生成的一个对象，该对象可作为目标对象使用。AOP代理包含了目标对象的全部方法，但AOP代理中的方法是目标对象方法的增强版，在特定切入点添加了增强处理并回调了目标对象的方法。

#### Spring的AOP支持 ####

Spring中的AOP代理可以是JDK动态代理（为实现接口的目标对象代理），也可以是cglib代理（为不实现接口的对象代理）。Spring目前只支持将方法调用作为连接点，如果需要把对成员变量的访问和更新也作为连接点，可以考虑使用AspectJ。

Spring的AOP侧重于AOP实现了Spring容器的整合，因此其AOP通常和容器一起使用。

AOP框架只需要程序员参与3件事：

- 定义普通业务组件
- 定义切入点，一个切入点可以横切多个业务组件
- 定义增强处理，即AOP框架为普通业务组件织入的处理动作

		AOP代理的方法 = 增强处理 + 目标对象的方法

通常建议使用AspectJ方式来定义切入点和增强处理，Spring可以使用下面两种方式：

- 基于注解的零配置方式：使用`@Aspect`，`@Pointcut`等注解来标注切入点和增强处理
- 基于XML配置文件的管理方式：使用Spring配置文件来定义切入点和增强处理

#### 基于注解的零配置方式 ####

在配置文件中开启对AOP注解的支持：

		<?xml version-"1.0" encodeing="utf-8"?>
		<beans ...
					 xmlns:aop="http://www.springframework.org/schema/aop"
					 xsi:schemaLocation="http://www.springframework.org/schema/beans
					 ...
					 http://www.springframework.org/schema/aop
					 http://www.springframework.org/schema/aop/spring-aop.xsd">
			<!-- 使用Schema启用@AspectJ支持 -->
			<aop:aspectj-autoproxy/>
			<!-- 不使用Schema启用@AspectJ支持 -->
			<bean class="org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator"/>
		</beans>

**1. 定义切面Bean**

在容器的搜索路径中配置一个用`@Aspect`注解标注的Java类，Spring会自动识别该Bean，并将其作为切面Bean处理。

切面类和其他类一样可以有方法，成员变量，还可能包括切入点，增强处理定义。切面Bean不会作为组件Bean处理，因此自动增强的后处理Bean将会略过该Bean，不对其做任何增强处理。

**2. 定义Before增强处理**

在一个切面类里使用`@Before`修饰一个方法，该方法将作为Before增强处理。通常需要指定一个`value`属性，属性值指定一个切入点表达式（可以是已定义的切入点，也可以使用新的切入点表达式），用于该方法的增强处理将被织入哪些切入点。

使用Before增强处理只能在目标方法执行之前织入增强，如果Before增强处理没有特殊处理，则目标方法总会自动执行，如果需要阻止目标方法的执行，可通过抛出一个异常来实现。另外，因为Before增强处理总是先于目标方法执行，因此无法在增强处理中访问目标方法的返回值。

**3. 定义AfterReturning增强处理**

`@AfterReturning`标注的方法将在目标方法执行完后作为增强处理被织入执行。可指定以下2个常用属性：

- `pointcut/value`：都用于指定该切入点对应的切入表达式。当指定了`pointcut`属性后，`value`属性值被覆盖
- `returning`：指定一个形参名，用于表示Advice方法中可定义与此同名的形参，该形参用于访问目标方法的返回值。在Advice方法中定义该形参时指定的类型将用于过滤目标方法返回值的类型。例如如果增强处理方法定义的形参类型为`String`，则该切入点只会匹配指定包下返回值类型为`String`或没有返回值的方法。一个特殊情况是形参类型为`Object`，则说明该切入点可匹配任何返回值类型的方法。

虽然AfterReturning增强处理可以访问但无法修改目标方法的返回值。

**4. 定义AfterThrowing增强处理**

`@AfterThrowing`用于处理程序中为处理的异常。可指定如下两个常用属性：

- `pointcut/value`：参考`@AfterReturning`注解的同名属性；
- `throwing`：指定一个形参名，用于表示Advice方法中可定义与此同名的形参，该形参用于访问目标方法抛出的异常。在Advice方法中定义该形参时指定的类型将用于过滤目标方法抛出异常的类型。例如如果增强处理方法定义的形参类型为`NullPointerException`，则该切入点只会匹配指定包下抛出异常类型为`NullPointerException`的方法。一个特殊情况是形参类型为`Throwable`，则说明该切入点可匹配抛出任何类型异常的方法。

AOP的AfterThrowing的异常处理和直接使用catch处理异常不同，catch处理意味着完全处理该异常，如果catch块中没有抛出新的异常，则该方法可正常结束；而AfterThrowing虽然处理了异常，但该异常仍然会传递给调用者，由他来进一步处理。

**5. After增强处理**

与AfterReturning的区别：

- AfterReturning只有在目标方法成功完成后才被织入；
- After则不管目标方法如何结束（成功/遇到异常中止）都会被织入，因此必须准备处理正常返回和异常返回两种情况。通常用于资源回收，有点类似final块。

`@After`注解也有一个`value`属性，用于指定需要织入该增强处理的切入点。

**6. Around增强处理**

Around增强处理类似Before增强处理和AfterReturning增强处理的总和。与这两种增强处理不同的是，Around增强处理可以在内部决定目标方法在什么时候执行，如何执行，甚至完全阻止它的执行；Around增强处理也可以改变目标方法执行的参数值和返回值。

Around增强处理必须在线程安全的环境下使用，如果可以使用普通的Before增强处理和AfterReturning增强处理就不必使用Around增强处理了，但如果需要目标方法在执行前和执行后共享某种状态数据，或者需要改变目标方法返回值，则只能使用Around增强处理了。

使用`@Around`注解也需要通过`value`属性值指定需要织入的切入点。并且该增强方法的第一个形参必须是`ProceedingJoinPoint`类型（所以该方法至少有一个形参），在增强方法内部，调用该参数的`proceed()`方法才会执行目标方法。增强方法就是通过这个参数的使用来控制目标方法执行的时机。调用`proceed()`方法时还可以传入一个`Object[]`对象作为参数，该数组中的值将作为实参传入目标方法；如果数组元素的个数或类型与对应的目标方法的形参不匹配，则会抛出异常。

**7. 访问目标方法的参数**

访问目标方法的最简单方法就是定义增强处理方法时将第一个参数定义为`JoinPoint`类型，该参数就代表了织入增强处理的连接点。该类型包含以下常用方法：

- `Object[] getArgs()`：返回执行目标方法时的参数
- `Signature getSignature()`：返回目标方法的声明信息
- `Object getTarget()`：返回被织入增强处理的目标对象
- `Object getThis()`：返回AOP框架为目标对象生成的代理对象

Spring AOP采用和AspectJ一样的优先顺序来织入增强处理：在进入连接点是，优先级高的增强处理优先被织入；离开连接点时，优先级高的增强处理后被织入。

如果同一个切入点有来自不同切面的多个增强处理，则Spring AOP将按随机顺序来织入。如果需要指定执行顺序，则可以采取以下解决方法：

- 让切面类实现`org.springframework.core.Ordered`接口。该接口只有一个方法`int getOrder()`，返回值越小，优先级越高。
- 直接使用`@Order`注解标注一个切面类，该注解可指定一个`int`类型的`value`属性，属性值越小，优先级越高。

如果有来自同一个切面的多个相同类型的增强处理，Spring没有办法控制他们的执行顺序，只能以随机的顺序来织入这些增强处理。因此如果对他们的执行顺序有要求，可以考虑将他们合并为一个增强处理，在内部处理他们的执行顺序，或者将他们分配到不同的切面类，再使用上面描述的方法进行排序。

如果只需要访问目标方法的形参，则可以在程序中使用`args`切入点表达式来绑定目标方法的参数。该表达式有2个作用：

- 提供了更简单的方式来访问目标方法的参数
- 对切入表达式增加额外的限制

		// ..表示后面还可以定义其他形参（可有可无），但这里的argName1和argName2是必须定义的形参
		args(argName1, argName2, ..)

**8. 定义切入点**

定义切入点就是为一个切入点表达式起一个别名，从而允许在多个增强处理中引用该表达式。

Spring AOP只支持将Spring Bean的方法作为连接点，因此可以把切入点看成所有能和切入点表达式匹配的Bean方法。

切入点定义包含以下两个部分：

- 一个切入点表达式：指定该切入点和哪些方法匹配；
- 一个包含名字和任意参数的方法签名：作为切入点的名称。

`@AspectJ`风格的AOP中，切入点签名采用一个普通的方法定义（方法体通常为空）来提供，且该方法的返回值必须是`void`；切入点表达式需要使用`@Pointcut`注解来标注。该方法签名中的访问控制符将决定该切入点是否能被其他切面类和其他包中的切面类使用。如果需要在其他切面类中使用该切入点，则访问控制符不能使`private`，且在`@Before`，`@AfterReturning`，`@Around`等注解中通过`pointcut`或`value`属性引用该切入点时，必须添加类名前缀。

**9. 切入点指示符**

Spring AOP仅支持AspectJ的切入点指示符（非全部）外加一个`bean`切入点指示符，而且这些指示符只会匹配方法执行的连接点。Spring AOP不支持的AspectJ切入点指示符：`call, get, set, preinitialization, staticinitialization, initialization, handler, adviceexecution, withincode, cflow, cflowbelow, if, @this, @withincode`。在Spring AOP中使用这些指示符会导致抛出`IllegalArgumentException`异常。

Spring AOP支持的切入点指示符：

- `execution`：匹配执行方法的连接点，格式如下：
	- `execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)`：各部分的解释如下：
	- `modifiers-pattern`：方法的修饰符，支持通配符，可省略；
	- `ret-type-pattern`：方法的返回值类型，支持通配符，可使用`*`通配符匹配所有的返回类型；
	- `declaring-type-pattern`：方法所属的类，支持通配符，可省略；
	- `name-pattern`：方法名，支持通配符，可使用`*`通配符匹配所有的方法；
	- `param-pattern`：方法签名中的形参列表，支持2个通配符：`*`和`..`，`*`代表一个任意类型的参数；`..`代表零个或多个任意类型的参数；
	- `throws-pattern`：方法声明抛出的异常，支持通配符，可省略。
- `within`：限定匹配特定范围内的连接点；
- `this`：限定AOP代理必须是指定类型的实例；
- `target`：限定目标对象必须是指定类型的实例；
- `args`：对连接点的参数类型进行限制，参数类型必须是指定类型；
- `bean`：只匹配指定Bean内的连接点，需要指定Bean的`id`或`name`，支持使用`*`通配符。

使用`args`指示符时需注意以下两种表达式的含义是不同的：

- `args(java.io.Serializable)`：匹配所有在运行时只传入一个`Serializable`类型的参数的方法，方法签名中该参数的声明类型并不重要，可以是该接口的任意实现类；
- `execution(* *(java.io.Serializable))`：匹配在方法签名中只包含一个声明为`Serializable`类型形参的方法，声明类型被限制为`Serializable`，不能是它的实现类。

**10. 组合切入点表达式**

Spring支持使用如下三个逻辑运算符组合切入点表达式：

- `&&`：同时满足
- `||`：满足之一即可
- `!`：不匹配指定表达式

#### 基于XML配置文件的配置方式 ####

使用Spring的XML配置方式仅支持`singleton`切面Bean，不能在XML中组合多个命名连接点的声明。

在Spring配置文件中，所有切面，切入点和增强处理都必须定义在`<aop:config.../>`元素内部。`<beans.../>`内部可以有多个`<aop:config.../>`元素，每个`<aop:config.../>`元素可以包含`pointcut`，`advisor`，`aspect`元素，且这三个元素必须按照此顺序来定义。

**1. 配置切面**

`<aop:aspect.../>`将一个已有的Bean转换为切面Bean，因此必须先定义该Bean。可指定三个属性：

- `id`：该切面Bean的标识；
- `ref`：该切面Bean引用的普通Bean的标识；
- `order`：指定该切面Bean的优先级，越小优先级越高。

**2. 配置增强处理**

在`<aop:aspect.../>`元素中使用如下子元素定义：

- `<before.../>`：配置Before增强处理；
- `<after.../>`：配置After增强处理；
- `<after-returning.../>`：配置AfterReturning增强处理；
- `<after-throwing.../>`：配置AfterThrowing增强处理；
- `<around.../>`：配置Around增强处理。

以上这些元素都不支持子元素，但可以指定如下属性：

- `pointcut/pointcut-ref`：指定切入点表达式，`pointcut-ref`属性指定已有的切入点名称，二者任用其一；
- `method`：指定该切面Bean中作为增强处理的方法名；
- `throwing`：只对`<after-throwing.../>`元素有效，指定一个形参名，AfterThrowing增强处理方法通过该形参访问目标方法抛出的异常；
- `returning`：只对`<after-returning.../>`元素有效，指定一个形参名，AfterReturning增强处理方法通过该形参访问目标方法的返回值。

XML配置文件使用如下运算符组合切入点表达式：

- `and`：相当于`&&`；
- `or`：相当于`||`；
- `not`：相当于`!`。

**3. 配置切入点**

`<aop:pointcut.../>`定义切入点，作为`<aop:aspect.../>`的子元素时，表示该切入点只在该切面Bean中有效；作为`<aop:config.../>`子元素时，表示该切入点可被多个切面共享。通常需要指定以下两个属性：

- `id`：该切入点的标识；
- `expression`：该切入点关联的切入点表达式。

### Spring的缓存机制 ###

Spring的缓存机制可以对容器中的任意Bean或Bean的任意方法进行缓存，因此可以在JavaEE应用的任何层次上进行缓存。Spring的缓存机制不是一种具体的缓存实现方案，底层仍然依赖EhCache，Guava等具体缓存工具。

#### 启用Spring缓存 ####

Spring配置文件专门为缓存提供了一个`cache`命名空间，导入之后，还需要：

1. 在Spring配置文件中添加`<cache:annotation-driven cache-manager="缓存管理器id"/>`元素，`cache-manager`属性的默认值是`cacheManager`，所以如果缓存管理器的名称也是`cacheManager`，就可以跳过设置该属性；
2. 针对不同的缓存实现配置对一个的缓存管理器。

**1. Spring内置缓存实现的配置**

Spring内置缓存的实现只是一种内存中的缓存，并非真正的缓存实现，通常只用于简单的测试环境，不建议在实际项目中使用。

Spring内置缓存实现使用`SimpleCacheManager`作为缓存管理器，底层直接使用了JDK的`ConcurrentMap`来实现缓存，`SimpleCacheManager`使用`ConcurrentMapCacheFactoryBean`作为缓存区，每个`ConcurrentMapCacheFactoryBean`配置一个缓存区，可以为每个缓存区设置名字，从而在后面使用注解驱动缓存时将缓存数据放入指定的缓存区。

一般情况下，应用中有多少个组件需要缓存，就配置多少个缓存区。

**2. EhCache缓存实现的配置**

需要在类加载路径下添加EhCache的配置文件`ehcache.xml`，并在Spring配置文件中添加如下配置：

		<!--
			 - 配置EhCache的CacheManager
			 - 通过configLocation指定ehcache.xml文件的位置
			 -->
		<bean id="ehcacheManager"
					class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean"
					p:configLocation="classpath:ehcache.xml"
					p:shared="false"/>
		<!-- 为Spring容器配置基于EhCahe的缓存管理器，并将EhCache的CacheManager注入到该缓存管理器Bean -->
		<bean id="cacheManager"
					class="org.springframework.cache.ehcache.EhCacheCacheManager"
					p:cacheManager-ref="ehcacheManager"/>

#### 使用@Cacheable执行缓存 ####

`@Cacheable`可以用于标注类和方法：

- 标注类时：在类级别上缓存，该类的实例的任何方法都需要缓存，并且共享同一个缓存区；
- 标注方法时：在方法级别上缓存，调用该方法时才需要缓存。

使用了缓存区结果的方法是不会执行其方法体的。

**1. 类级别的缓存**

程序调用该类的任意方法时，只要传入的参数相同，Spring就会使用缓存。  
具体来说，当程序第一次调用该类的实例的某个方法时，Spring缓存机制会将该方法返回的数据放入指定缓存区-`@Cacheable`注解的`value`属性值指定的缓存区（该缓存区必须在之前已经配置过）。  
以后程序调用该类实例的任何方法时，只要传入的参数相同，Spring就不会真的执行该方法，而是直接使用缓存区中的数据。 
因此，当两个方法的参数完全相同，但返回类型却不同时，可能会引发异常：第二个方法会将第一个方法存入缓存的结果取出并尝试强制类型转换为自己的返回类型，从而可能引发类型转换异常。

注意，类级别的缓存默认以所有方法参数作为key来缓存方法返回的数据，同一个类的不同方法，只要调用方法时传入的参数相同，Spring都会直接利用缓存区中的数据。例如

			pulbic User getUserByName(String name);
			public User getUser(Object name);
			---------
			User u1 = getUserByName("xiao");
			User u2 = getUser("xiao);

			return u1 == u2; // should be true

`@Cacheable`可以指定如下属性：

- `value`：必需属性，可指定多个缓存区名字，方法返回值将放入这些缓存区；
- `key`：通过SpEl表达式显式指定缓存的key；
- `condition`：指定一个返回boolean值的SpEL表达式，当该表达式返回true时，Spring才会缓存该方法的返回值；
- `unless`：指定一个返回boolean值的SpEL表达式，当该表达式返回true时，Spring就不会缓存该方法的返回值；

`@CachePut`注解和`@Cacheable`注解类似，但其标注的方法不会从缓冲区中读取数据，也就是说，每次调用该方法都会真正执行一次方法，并将方法返回值放入缓存区，但从不会使用缓存区中的数据。

**2. 方法级别的缓存**

每次调用该方法时，只要传入的参数相同，则直接使用缓存区中的数据。

#### 使用@CacheEvict清除缓存 ####

`@CacheEvict`标注的方法用于清除缓存，注解可指定如下属性：

- `value`：必需属性，指定清除哪个缓存区的数据；
- `allEntries`：指定是否清空整个缓存区；
- `beforeInvocation`：是否在执行方法前清除缓存。默认在方法成功完成之后才清除缓存。
- `condition`：指定一个SpEL表达式，当表达式为true时才清除缓存；
- `key`：通过SpEL表达式显式指定缓存的key。

### Spring的事务 ###

2种传统事务策略：

- 全局事务：由应用服务器管理，需要底层服务器的JTA支持，可以跨多个事务性资源。
- 局部事务：不能保证跨多个事务性资源的事务的正确性。和底层采用的持久化技术有关，采用JDBC持久化技术时，需要使用`Connection`对象操作事务；采用Hibernate持久化技术时，需要使用`Session`对象操作事务。

Spring事务策略是通过`PlatformTransactionManager`接口体现的，该接口是Spring事务策略的核心，有很多的实现类。应用程序面向与平台无关的接口编程，当底层采用不同的持久化技术时，系统只需使用不同的`PlatformTransactionManager`实现类即可，而这种转换通常有Spring容器负责管理，应用程序并不会和具体的事务API耦合，也无须与特定实现类耦合。

Spring的事务机制是典型的策略模式。`PlatformTransactionManager`接口代表事务管理接口，要求事务管理需要提供开始事务（`getTransaction()`），提交事务（`commit()`）和回滚事务（`rollback()`）三个方法，而不关心底层是具体如何实现这些方法的。

该接口包含的`getTransaction(TransactionDefinition definition)`方法根据传入的`TransactionDefinition`参数返回一个`TransactionStatus`对象。该对象表示一个事务，被关联到当前执行的线程上。如果当前执行的线程已经处于事务管理之下，则返回当前线程的事务对象；否则，系统将新建一个事务对象返回。

`TransactionDefinition`接口定义了一个事务规则，必须指定如下几个属性值：

- 事务隔离：当前事务和其他事务的隔离程度。例如这个事务是否能看到其他事务未提交的数据等。
- 事务传播：通常事务中执行的代码都会在当前事务中运行。但如果一个事务上下文已存在，有几个选项可指定该事务性方法的执行行为。Spring提供EJB CMT(Contain Manager Transaction, 容器管理事务)中所有的的事务传播选项。
- 事务超时：事务的最长持续时间，在超出该时间后，系统会自动回滚。
- 只读状态：只读事务不修改任何数据。

`TransactionStatus`代表事务本身，提供了简单的控制事务执行和查询事务状态的方法，这些方法在所有的事务API总都相同：

- `boolean isNewTransaction()`：判断事务是否为新建的事务
- `void setRollbackOnly()`：设置事务回滚
- `boolean isRollbackOnly()`：查询事务是否已有回滚标志

Spring具体的事务管理由`PlatformTransactionManager`的不同实现类完成。在Spring容器中配置`PlatformTransactionManager`时必须针对不同的环境提供不同的实现类。

JDBC数据源的局部事务管理器配置如下：

		<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
					 xmlns:p="http://www.springframework.org/schema/p"
					 xmlns="http://www.springframework.org/schema/beans"
					 xsi:schemaLocation="http://www.springframework.org/schema/beans 
															 http://www.springframework.org/schema/beans/spring-beans.xsd">
			<!-- 定义数据源Bean，使用C3P0数据源实现，并注入数据源的必要信息 -->
			<bean id="dataSource"
						class="com.mchange.v2.c3p0.ComboPooledDataSource"
						destory-method="close"
						p:driverClass="com.mysql.jdbc.Driver"
						p:jdbcUrl="jdbc:mysql://localhost/spring?userSSL=true"
						p:user="root"
						p:password="pass"
						p:maxPoolSize="40"
						p:minPoolSize="2"
						p:initialPoolSize="2"
						p:maxIdleTime="30"/>
			<!--
				 - 配置JDBC数据源的局部事务管理器，使用DataSourceTransactionManager类
				 - 该类实现PlatformTransactionManager接口，是针对采用数据源连接的特定实现
				 - 配置DataSourceTransactionManager时需要依赖注入DataSource的引用
				 -->
			<bean id="transactionManager"
						class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
						p:dataSource-ref="dataSource"/>
		</beans>

容器管理的JTA全局事务管理器的配置如下：

		<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
					 xmlns:p="http://www.springframework.org/schema/p"
					 xmlns="http://www.springframework.org/schema/beans"
					 xsi:schemaLocation="http://www.springframework.org/schema/beans 
															 http://www.springframework.org/schema/beans/spring-beans.xsd">
			<!-- 配置JNDI数据源Bean，其中jndiName指定容器管理数据源的JNDI -->
			<bean id="dataSource"
						class="org.springframework.jndi.JndiObjectFactoryBean"
						p:jndiName="jdbc/jpetstore"/>
			<!--
				 - 使用JtaTrasactionManager类，该类实现了PlatformTransactionManager接口
				 - 是针对采用全局事务管理的特定实现
				 -->
			<bean id="transactionManager"
						class="org.springframework.transaction.jta.JtaTrasactionManager"/>
		</beans>

配置`JtaTrasactionManager`全局事务管理策略时，只需指定事务管理器实现类即可，无须传入额外的事务性资源，因为全局事务的JTA资源有Java EE服务器提供，Spring容器可以自行从Java EE服务器中获取该事务性资源，所以无须使用依赖注入来配置。

采用Hibernate持久化技术时，局部事务策略的配置如下：

		<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
					 xmlns:p="http://www.springframework.org/schema/p"
					 xmlns="http://www.springframework.org/schema/beans"
					 xsi:schemaLocation="http://www.springframework.org/schema/beans 
															 http://www.springframework.org/schema/beans/spring-beans.xsd">
			<!-- 定义数据源Bean，使用C3P0数据源实现，并注入数据源的必要信息 -->
			<bean id="dataSource"
						class="com.mchange.v2.c3p0.ComboPooledDataSource"
						destory-method="close"
						p:driverClass="com.mysql.jdbc.Driver"
						p:jdbcUrl="jdbc:mysql://localhost/spring?userSSL=true"
						p:user="root"
						p:password="pass"
						p:maxPoolSize="40"
						p:minPoolSize="2"
						p:initialPoolSize="2"
						p:maxIdleTime="30"/>
			<!--
				 - 定义Hibernate的SessionFactory，SessionFactory需要依赖数据源，注入dataSource
				 - 并使用hibernate.cfg.xml文件配置Hibernate属性
				 -->
			<bean id="sessionFactory"
						class="org.springframework.orm.hibernate5.LocalSessionFactoryBean"
						p:dataSource-ref="dataSource"
						p:configLocation="classpath:hibernate.cfg.xml"/>
			<!--
				 - 配置Hibernate的局部事务管理器，使用HibernateTransactionManager类
				 - 该类是PlatformTransactionManager接口针对采用Hibernate的特定实现类
				 - 配置HibernateTransactionManager需要依赖注入SessionFactory
				 -->
			<bean id="transactionManager"
						class="org.springframework.orm.hibernate5.HibernateTransactionManager"
						p:sessionFactory-ref="sessionFactory"/>
		</beans>
			
底层采用Hibernate持久化技术，但事务采用JTA全局事务的配置如下：

		<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
					 xmlns:p="http://www.springframework.org/schema/p"
					 xmlns="http://www.springframework.org/schema/beans"
					 xsi:schemaLocation="http://www.springframework.org/schema/beans 
															 http://www.springframework.org/schema/beans/spring-beans.xsd">
			<!-- 配置JNDI数据源Bean，其中jndiName指定容器管理数据源的JNDI -->
			<bean id="dataSource"
						class="org.springframework.jndi.JndiObjectFactoryBean"
						p:jndiName="jdbc/jpetstore"/>
			<!--
				 - 定义Hibernate的SessionFactory，SessionFactory需要依赖数据源，注入dataSource
				 - 并使用hibernate.cfg.xml文件配置Hibernate属性
				 -->
			<bean id="sessionFactory"
						class="org.springframework.orm.hibernate5.LocalSessionFactoryBean"
						p:dataSource-ref="dataSource"
						p:configLocation="classpath:hibernate.cfg.xml"/>
			<!--
				 - 使用JtaTrasactionManager类，该类实现了PlatformTransactionManager接口
				 - 是针对采用全局事务管理的特定实现
				 -->
			<bean id="transactionManager"
						class="org.springframework.transaction.jta.JtaTrasactionManager"/>
		</beans>

可以看出，不管采用哪种持久层访问技术，只要使用JTA全局事务，Spring事务管理器的配置都完全一样，因为采用的都是相同的全局事务管理策略。但某些应用服务器可能需要使用`JtaTrasactionManager`的子类，如`WebLogicJtaTransactionManager`（Oracle提供的WebLogic），`WebSphereUowTransactionManager`（IBM提供的WebSphere）等，分别对应于不同的应用服务器。

实际上，Spring提供了如下两种事务管理方式：

- 编程式事务管理：面向`PlatformTransactionManager`接口编程，在代码中获取容器中的`transactionManager`Bean，该Bean总是`PlatformTransactionManager`的实例，所以可以调用它的3个方法来开始事务，提交事务和回滚事务；
- 声明式事务管理：无须在Java代码中写任何事务操作代码，通过在XML文件中为业务组件配置事务代理（AOP代理的一种），AOP为事务代理所织入的增强处理也有Spring提供：在目标方法执行前织入开始事务，在目标方法执行结束后织入结束事务。

#### 使用XML Schema配置事务策略（声明式事务管理） ####

Spring的XML Schema方式提供了简洁的事务配置策略，使用`tx:`命名空间来配置事务管理，使用`<tx:advice.../>`元素来配置增强处理，配置完后就可以使用`<aop:advisor.../>`来启用自动代理了。

`<tx:advice.../>`元素除了需要配置`transaction-manager`属性指定事务管理器外，还需要配置一个`<attributes.../>`子元素，该子元素里可包含多个`<method.../>`子元素。也是`<tx:advice.../>`元素配置的重点。每个`<method.../>`子元素都为一批方法指定了所需的事务定义，包括事务传播属性，事务隔离属性，事务超时属性，只读事务，对指定异常是否回滚等：

- `name`：必填属性，与该事务管理的方法名，支持使用通配符；
- `propagation`：指定事务传播行为，支持的传播行为如下：
	- `PROPAGATION_MANDATORY`：调用该方法的线程必须处于事务环境中，否则抛出异常；
	- `PROPAGATION_NESTED`：即使执行方法的线程已处于事务环境中，也依然启动新的事务，方法在嵌套的事务中执行；如果线程并未处于事务环境中，则处理方法和`PROPAGATION_REQUIRED`相同；
	- `PROPAGATION_NEVER`：不允许调用该方法的线程处于事务环境中，否则抛出异常；
	- `PROPAGATION_NOT_SUPPORTED`：如果调用该方法的线程处于事务环境中，则先暂停当前事务，然后执行该方法；
	- `PROPAGATION_REQUIRED`：默认值，要求在事务环境中执行该方法，如果执行线程已处于事务环境，则直接调用，否则启动新的事务后执行该方法；
	- `PROPAGATION_REQUIRES_NEW`：要求在新的事务环境中执行该方法，如果执行线程已处于事务环境，则暂停当前事务，启动新的事务后执行该方法；如果尚未处于事务环境中，则启动新的事务后执行方法；
	- `PROPAGATION_SUPPORTS`：如果当前执行线程处于事务环境中，则使用当前事务，否则不使用事务。
- `isolation`：指定事务隔离级别，可以是`Isolation`枚举类中的任一枚举值，默认值是`Isolation.DEFAULT`；
- `timeout`：指定事务超时的时间（秒为单位），`-1`意味着永不超时，也是默认值；
- `read-only`：指定事务是否只读，默认为false；
- `rollback-for`：指定触发事务回滚的异常类（使用全限定类名），多个异常类用逗号隔开；
- `no-rollback-for`：指定不触发事务回滚的异常类（使用全限定类名），多个异常类用逗号隔开；

如果事务管理器Bean（`PlatformTransactionManager`实现类）的名字是`transactionManager`，则配置`<tx:advice.../>`元素时可以省略指定`transaction-manager`属性。但如果为事务管理器Bean指定了其他名字，则需要为`<tx:advice.../>`元素指定`transaction-manager`属性。 

`<aop:advisor.../>`并不存在于标准的AOP机制里，它的作用就是将Advice（`advice-ref`属性的值）和切入点（`pointcut`或`pointcut-ref`属性的值）绑定在一起，从而保证Advice所包含的增强处理在对应的切入点被织入。

- `advice-ref`：对之前定义的`<tx:advice.../>`元素的引用；
- `pointcut/pointcut-ref`：`pointcut`现指定切入点表达式，`pointcut-ref`引用之前定义的切入点。

当使用`<aop:advisor.../>`元素将切入点和Advice绑定时，实际上是有Spring提供的Bean后处理器完成的。Spring提供了`BeanNameAutoProxyCreator`和`DefaultAdvisorAutoProxyCreator`两个Bean后处理器。

使用声明式事务管理还可以为不同的业务逻辑方法指定不同的事务策略。

默认情况下，只有当方法引发运行时异常和unchecked异常时，Spring事务机制才会自动回滚事务，即只有当抛出一个`RunTimeException`或其子类实例，或`Error`对象时，Spring才会自动回滚事务，如果事务方法抛出checked异常，则事务不会自动回滚。`rollback-for`属性可强制Spring遇到特定checked异常时自动回滚事务。

#### 使用@Transactional ####

`@Transactional`可以标注Spring Bean类，表明这些事务设置对整个Bean类起作用；也可以标注Bean类中的某个方法，表示这些事务设置只对该方法有效。可以指定如下属性：

- `isolation`：指定事务的隔离级别，默认为底层事务的隔离级别；
- `noRollbackFor`：指定遇到特定异常时强制不回滚事务；
- `noRollbackForClassName`：指定遇到特定的多个异常时强制不回滚事务，该属性值可以指定多个异常类名；
- `propagation`：指定事务传播行为；
- `readOnly`：指定事务是否只读；
- `rollbackFor`：指定遇到特定异常时强制回滚事务；
- `rollbackForClassName`：指定遇到特定的多个异常时强制回滚事务，该属性值可以指定多个异常类名；
- `timeout`：指定事务的超时时长。

此外，还要在Spring配置文件中增加如下的配置片段：

			<!--
				 - 配置JDBC数据源的局部事务管理器，使用DataSourceTransactionManager类
				 - 该类实现PlatformTransactionManager接口，是针对采用数据源连接的特定实现
				 - 配置DataSourceTransactionManager时需要依赖注入DataSource的引用
				 -->
			<bean id="transactionManager"
						class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
						p:dataSource-ref="dataSource"/>
			<!-- 根据注解来生成事务处理 -->
			<tx:annotation-driven transaction-manager="transactionManager"/>

如果需要对同一个包中的多个类配置相同的事务处理，推荐使用XML配置文件，如果需要为每个类配置不同的事务处理，则可以考虑使用`@Transactional`注解为每个类单独配置（也可以使用XML配置文件）。

### Spring整合Struts2 ###

#### 启动Spring容器 ####

在Web应用中创建Spring容器有如下两种方式：

- 直接在`web.xml`文件中配置创建Spring容器；
- 利用第三方MVC框架的扩展点创建Spring容器。

为了让Spring容器随Web应用的启动而自动启动，可以借助于`ServletContextListener`监听器，该监听器可以在Web应用启动时回调自定义方法-该方法就可以启动Spring容器。

Spring提供了一个`ContextLoaderListener`，该监听器实现了`ServletContextListener`接口，会在创建时自动查找`WEB-INF/`下的`applicationContext.xml`文件，因此如果只有一个且文件名为`applicationContext.xml`的配置文件，则只需在`web.xml`文件中添加如下配置：

		<listener>
			<listener-class>
			org.springframework.web.context.ContextLoaderListener
			</listener-class>
		</listener>

如果有多个配置文件需要载入，则考虑使用`<context-param.../>`元素来确定配置文件的文件名。`ContextLoaderListener`加载时，会查找名为`contextConfigLocation`的初始化参数，因此，配置`<context-param.../>`时应指定参数名为`contextConfigLocation`。如果没有使用`contextConfigLocation`指定配置文件，则Spring会自动查找`WEB-INF/`路径下的`applicationContext.xml`配置文件。如果无法找到合适的配置文件，则Spring将无法正常初始化。

Spring根据指定的配置文件创建`WebApplicationContext`对象，并将其保存在Web应用的`ServletContext`中。如果需要在应用中获取`ApplicationContext`实例，则可以通过以下代码获取：

		// 获取当前Web应用启动的Spring容器，如果没有相应的对象，则返回null而不会引发异常
		WebApplicationContext ctx = WebApplicationContextUtils.getWebApplicationContext(servletContext);

#### MVC框架与Spring的整合 ####

MVC框架中控制器和业务逻辑组件的整合可以采用工厂模式或服务定位器模式：

- 服务定位器模式：用于远程访问，业务逻辑组件已在某个容器中运行，并对外提供服务。控制器无须理会该业务逻辑组件的创建，直接调用服务即可，但在调用前，必须先找到该服务。经典JavaEE应用使用的就是这种结构。
- 工厂模式：所有的业务逻辑组件的创建和运行都由工厂负责，控制器只需要定位工厂实例即可。而在Spring框架中，Spring容器就是最大的工厂。

为了让Struts2的控制器可以访问到Spring的业务逻辑组件，有以下两种策略：

- Spring容器负责管理控制器Action，并利用依赖注入为控制器注入业务逻辑组件；
- 利用Spring的自动装配，Action将会自动从Spring容器中获取所需的业务逻辑组件。

#### 让Spring管理控制器 ####

Struts2提供了Spring插件，该插件提供了一种伪Action，允许在`struts.xml`文件中配置Action，指定其`class`属性（用于创建Action实例的实现类）时，不再指定Action的实际实现类，而是指定为Spring容器中的Bean的`id`。这样，Struts2不再自己负责创建Action实例，而是通过Spring容器获取Action对象。需要以下几个步骤：

1. 创建业务逻辑组件和Action类；
2. 在Spring配置文件中创建业务逻辑组件和Action类的Bean，并完成二者之间的依赖注入（Action类依赖于业务逻辑组件）；
3. 配置`struts.xml`文件，使用`<action.../>`元素定义处理请求的Action时，将其`class`属性值设置为Spring配置文件中定义的Action类型的Bean的标识；

从以上步骤可以看出，Spring和Struts2整合的实质就是让Spring容器作为工厂模式里的工厂来创建管理Action类实例。需要注意的是Spring配置的Action类Bean的`scope`可以是`prototype`或`request`，但一定不能设置为`singleton`，因为Action中包含了请求的状态信息，必须为每个请求创建一个新的Action实例来处理。

#### 使用自动装配 ####

利用Spring的自动装配策略将相应的业务逻辑组件注入到Action实例中-Action类中的setter方法名与Spring配置文件中的Bean的标识能够自动匹配。这种方法虽然配置文件简单，但因为必须在Spring配置文件中，创建与Action类中所需业务逻辑组件同名的业务逻辑组件Bean，从而让Action和业务逻辑组件在代码中又出现了耦合。

系统默认按`byName`自动装配，可以通过设置`struts.xml`配置文件中的`struts.objectFactory.spring.autoWire`常量改变Spring插件的自动装配策略，该值可接受以下常量：

- `name`：使用`byName`自动装配
- `type`：使用`byType`自动装配
- `auto`：Spring插件自动检测使用哪种装配方式
- `constructor`：与`type`类似，区别是`constructor`使用构造器来构造注入所需的参数，而不是设值注入。

### Spring整合Hibernate ###

#### Spring提供的DAO支持 ####

DAO模式的设计核心思想是，所有的数据库访问都通过DAO组件完成，DAO组件封装了数据库的增，删，改等原子操作。业务逻辑组件依赖于DAO组件提供的数据库原子操作完成系统业务逻辑的实现。

JavaEE应用大致都可分为以下三层：

- 表现层
- 业务逻辑层
- 数据持久层

轻量级JavaEE架构以Spring IoC容器为核心，承上启下：向上管理来自表现层的Action，向下管理业务逻辑层组件，同时负责管理业务逻辑层所需的DAO对象。

Spring提供了一系列的抽象类，这些抽象类将被作为应用中的DAO实现类的父类，从而简化DAO的开发步骤，并为不同的数据库访问技术提供一致的API。

此外，Spring还提供了一致的异常抽象，将原有的Checked异常转换包装成RunTime异常，从而在编码时无须捕捉各种技术中的特定异常。Spring DAO体系中的异常都继承了`DataAccessException`，该异常是RunTime的，无须显式捕捉。

#### 管理Hibernate的SessionFactory ####

通过Hibernate进行持久层访问时，必须先获得`SessisonFactory`对象，他是单个数据库映射关系编译后的内存镜像。大部分情况下，一个JavaEE应用对应一个数据库，即对应一个`SessisonFactory`对象。

在Spring配置文件中配置Hibernate SessisonFactory代码示例：

		<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
					 xmlns:p="http://www.springframework.org/schema/p"
					 xmlns="http://www.springframework.org/schema/beans"
					 xsi:schemaLocation="http://www.springframework.org/schema/beans 
															 http://www.springframework.org/schema/beans/spring-beans.xsd">
			<!-- 定义数据源Bean，使用C3P0数据源实现，并注入数据源的必要信息 -->
			<bean id="dataSource"
						class="com.mchange.v2.c3p0.ComboPooledDataSource"
						destory-method="close"
						p:driverClass="com.mysql.jdbc.Driver"
						p:jdbcUrl="jdbc:mysql://localhost/spring?userSSL=true"
						p:user="root"
						p:password="pass"
						p:maxPoolSize="40"
						p:minPoolSize="2"
						p:initialPoolSize="2"
						p:maxIdleTime="30"/>
			<!--
				 - 定义Hibernate的SessionFactory需要提供依赖数据源，注入dataSource
				 - 使用hibernate.cfg.xml作为Hibernate的配置文件
				 -->
			<bean id="sessionFactory"
						class="org.springframework.orm.hibernate5.LocalSessionFactoryBean"
						p:dataSource-ref="dataSource"
						p:configLocation="classpath:hibernate.cfg.xml"/>
		</beans>

除了JDBC数据源，也可以使用容器数据源：

		<!-- 配置JNDI数据源 -->
		<bean id="dataSource"
					class="org.springframework.jndi.JndiObjectFactoryBean"
					p:jndiName="java:comp/env/jdbc/myds"/>

#### 实现DAO组件的基类 ####

一个有效的实践是把所有DAO组件都需要提供的方法提取出来，由一个`BaseDao`来负责实现，让其他的DAO组件继承该`BaseDao`即可。通常所有的DAO组件都应该提供如下方法：

- 根据id加载持久化实体；
- 保存持久化实体；
- 更新持久化实体；
- 删除持久化实体，以及根据id删除持久化实体；
- 获取所有的持久化实体；
- 一些通用的查询方法。

		public interface BaseDao<T> {
			// 根据ID加载实体
			T get(Class<T> entityClazz, Serializable id);
			// 保存实体
			Serializable save(T entity);
			// 更新实体
			void update(T entity);
			// 删除实体
			void delete(T entity);
			// 根据ID删除实体
			void delete(Class<T> entityClazz, Serializable id);
			// 获取所有实体
			List<T> findAll(Class<T> entityClazz);
			// 获取实体总数
			long findCount(Class<T> entityClazz);
		}

Hibernate5的基类实现类范例：

		public class BaseDaoHibernate5<T> implements BaseDao<T> {
			// DAO组件进行持久化操作底层依赖的SessionFactory组件
			private SessionFactory sessionFactory;
			// 依赖注入SessionFactory所需的setter方法
			public void setSessionFactory(SessionFactory sessionFactory) {
				this.sessionFactory = sessionFactory;
			}
			public SessionFactory getSessionFactory() {
				return this.sessionFactory;
			}
			// 根据ID加载实体
			@SuppressWarnings("unchecked")
			public T get(Class<T> entityClazz, Serializable id) {
				return (T)getSessionFactory().getCurrentSession().get(entityClazz, id);
			}
			// 保存实体
			public Serializable save(T entity) {
				return getSessionFactory().getCurrentSession().save(entity);
			}
			// 更新实体
			public void update(T entity) {
				getSessionFactory().getCurrentSession().saveOrUpdate(entity);
			}
			// 删除实体
			public void delete(T entity) {
				getSessionFactory().getCurrentSession().delete(entity);
			}
			// 根据ID删除实体
			public void delete(Class<T> entityClazz, Serializable id) {
				getSessionFactory().getCurrentSession().
				createQuery("delete " + entityClazz.getSimpleName() + " en where en.id = ?0").
				setParameter("0", id).
				executeUpdate();
			}
			// 获取所有实体
			public List<T> findAll(Class<T> entityClazz) {
				return find("select en from " + entityClazz.getSimpleName() + " en");
			}
			// 获取实体总数
			public long findCount(Class<T> entityClazz) {
				List<?> list = find("select count(*) from " + entityClazz.getSimpleName());
				if (list != null && list.size() == 1) {
					return (Long)list.get(0);
				}
				return 0;
			}
			// 根据HQL语句查询实体
			@SuppressWarnings("unchecked")
			protected List<T> find(String hql) {
				return (List<T>) getSessionFactory().getCurrentSession().createQuery(hql).getResultList();
			}
			// 根据带占位符参数的HQL语句查询实体
			@SuppressWarnings("unchecked")
			protected List<T> find(String hql, Object.. params) {
				// 创建查询
				Query query = getSessionFactory().getCurrentSession().createQuery(hql);
				// 为包含占位符的HQL语句设置参数
				for (int i = 0, len = params.length; i < len; i++) {
					query.setParameter(i + "", params[i]);
				}
				return (List<T>) query.getResultList();
			}
			/**
			 * 使用HQL语句进行分页查询操作
			 * @param hql 需要查询的HQL语句
			 * @param pageNo 查询地pageNo页的记录
			 * @param pageSize 每页需要显式的记录数
			 * @return 当前页的所有记录
			 */
			@SuppressWarnings("unchecked")
			protected List<T> findByPage(String hql, int pageNo, int pageSize) {
				// 创建查询
				Query query = getSessionFactory().getCurrentSession().createQuery(hql).
											// 执行分页
											setFirstResult((pageNo - 1) * pageSize).
											setMaxResults(pageSize).
											getResultList();
			}
			/**
			 * 使用带占位符的HQL语句进行分页查询操作
			 * @param hql 需要查询的HQL语句
			 * @param pageNo 查询地pageNo页的记录
			 * @param pageSize 每页需要显式的记录数
			 * @param params 传入的占位符参数
			 * @return 当前页的所有记录
			 */
			@SuppressWarnings("unchecked")
			protected List<T> findByPage(String hql, int pageNo, int pageSize, Object.. params) {
				// 创建查询
				Query query = getSessionFactory().getCurrentSession().createQuery(hql);
				// 为包含占位符的HQL语句设置参数
				for (int i = 0, len = params.length; i < len; i++) {
					query.setParameter(i + "", params[i]);
				}
				// 执行分页，返回查询结果
				return query.setFirstResult((pageNo - 1) * pageSize).
							 			 setMaxResults(pageSize).
										 getResultList();
			}
		}

#### Hibernate Template和HibernateDaoSupport ####

注意！Spring不推荐使用这里介绍的方法，建议使用上一节介绍的通过`SessionFactory`的`getCurrentSession()`方法返回的`Session`实例来做持久化操作。

出于向上兼容的需要，Spring为整合Hibernate提供了`HibernateTemplate`和`HibernateDaoSupport`两个工具类。

`HibernateTemplate`通过内部的`SessionFactory`实例执行持久层的访问，该实例可以通过构造器传入，也可以通过设值注入传入。`HibernateTemplate`有三个构造函数：

- `HibernateTemplate()`：使用无参数的构造器后必须使用`setSessionFactory(SessionFactory factory)`方法注入依赖，然后才能进行持久化操作；
- `HibernateTemplate(org.hibernate.SessionFactory sessionFactory)`：构造时就传入`SessionFactory`对象；
- `HibernateTemplate(org.hibernate.SessionFactory sessionFactory, boolean allowCreate)`：`allowCreate`参数表明如果当前线程没有找到一个事务性的session，是否创建一个非事务性的session。

`HibernateTemplate`的常用方法：

- `void delete(Object entity)`：删除指定的持久化类实例；
- `deleteAll(Collection entities)`：删除集合内全部的持久化类实例；
- `find(String queryString)`：根据HQL查询字符串返回实例集合的一系列重载方法；
- `findByName(String QueryName)`：根据命名查询返回实例集合的一系列重载方法；
- `get(Class entityClass, Serializable id)`：根据主键加载特定的持久化类实例；
- `save(Object entity)`：保存新的实例；
- `update(Object entity)`：更新实例的状态，要求`entity`是持久状态；
- `setMaxResults(int maxResults)`：设置分页的大小。

`HibernateDaoSupport`工具类主要为实现DAO组件提供工具基类，有2个方法来简化DAO的实现：

- `public final HibernateTemplate getHibernateTemplate()`
- `public final void setSessionFactory(SessionFactory sessionFactory)`

`HibernateCallback`接口可以对`HibernateTemplate`进行增强，提高它的灵活性。该接口的实例可在任何有效的Hibernate数据访问中使用，包含一个`doInHibernate(org.hibernate.Session session)`方法，实现该方法时，可以通过`session`参数获得Hibernate Session的引用，该session是绑定到该线程的`Session`实例，然后就可以使用它来访问数据库了。

`HibernateTemplate`通过`<T> T execute(HibernateCallback<T> action)`方法来扩充自己的持久化操作。

#### 实现DAO组件 ####

通过继承上面的`BaseDao<T>`接口，并指定泛型类型来创建普通的DAO组件服务接口。

		public interface UserDao extends BaseDao<User> {
			// 定义业务相关的持久化操作
		}

这里仍然定义为接口是因为可能需要为这个DAO组件提供一些额外的业务相关的查询方法（服务），而根据面向接口编程的思想，对外提供的服务（API）都应该定义在接口中而不是直接在实例类中定义。

#### 使用IoC容器组装各种组件 ####

从用户角度看，用户发出HTTP请求，当MVC框架的控制组件拦截到用户请求后，将调用相应的业务逻辑组件来处理请求，业务逻辑组件则调用DAO组件来完成业务操作，而DAO组件则依赖于`SessionFactory`和`DataSource`等底层组件来实现数据库访问。

从系统实现角度看，IoC容器先创建`SessionFactory`和`DataSource`等底层组件，然后将这些底层组件注入给DAO组件，从而创建一个完整的DAO组件，并将此DAO组件注入给业务逻辑组件，进而创建一个完整的业务逻辑组件，再将该业务逻辑组件注入给控制器组件，此时的的控制器组件就具备了拦截并处理用户请求的功能。这一系列的操作都是由Spring的IoC容器提供实现的。

实际应用中，应该将DAO组件，业务逻辑组件以及控制器组件的配置放在不同的配置文件中。

#### 使用声明式事务 ####

为业务逻辑组件添加事务只需如下几个步骤：

1. 针对不同的事务策略配置对应的事务管理器；
2. 使用`<tx:advice.../>`元素配置事务增强处理Bean，可以使用多个`<method.../>`子元素为不同方法指定相应的事务语义；
3. 在`<aop:config.../>`元素中使用`<aop:pointcut.../>`定义切入点，并使用`<aop:advisor.../>`元素配置自动事务代理（关联pointcut和advice）。

### Spring整合JPA ###

#### 管理EntityManagerFactory ####

和使用Spring容器管理Hibernate的`SessionFactory`类似，Spring容器也可以管理JPA的`EntityManagerFactory`，通常有两种方式：

- 使用`LocalEntityManagerFactoryBean`
- 使用`LocalContainerEntityManagerFactoryBean`

**1. 使用LocalEntityManagerFactoryBean**

`LocalEntityManagerFactoryBean`创建的`EntityManagerFactory`不能使用Spring容器中已有的`DataSource`，也不能切换到全局事务。配置时必须注入持久化单元（PersistenceUnit），而该持久化单元必须在`persistence.xml`文件中已被配置，并自己管理数据库连接等信息。

**2. 推荐使用LocalContainerEntityManagerFactoryBean**

`LocalContainerEntityManagerFactoryBean`根据`persistence.xml`配置文件创建`PersistenceUnitInfo`，并提供`DataSourceLookup`策略，因此完全可以直接使用Spring容器中已有的数据源，并自己控制织入流程。`persistence.xml`文件在这种方法中不需要管理数据库连接信息。

#### 实现DAO组件基类 ####

使用Spring容器管理`EntityManagerFactory`之后，就可以通过它获得线程安全的`EntityManager`，并将其注入DAO组件。

1. 在DAO组件中使用`@PersistenceContext`标注`EntityManager`成员变量（使用Field注入），或标注设置`EntityManager`成员变量的setter方法（使用设值注入）；
2. 在Spring配置文件中配置`PersistenceAnnotationBeanPostProcessor`后处理器即可。该后处理器负责处理`@PersistenceContext`注解，并通过容器中的`EntityManagerFactory`获得线程安全的`EntityManager`，再将其注入到DAO组件。

使用JPA技术实现BaseDao组件：

		public class BaseDaoJpa<T> implements BaseDao<T> {
			@PersistenceContext
			protected EntityManager enetityManager;
			// 根据ID加载实体
			@SuppressWarnings("unchecked")
			public T get(Class<T> entityClazz, Serializable id) {
				return (T)enetityManager.find(entityClazz, id);
			}
			// 保存实体
			public Serializable save(T entity) {
				enetityManager.persist(entity);
				try {
					return (Serializable) entity.getClass().getMethod("getId").invoke(entity);
				} catch (Exception e) {
					e.printStackTrace();
					throw new RuntimeException(entity + "必须提供getId()方法！");
				}
			}
			// 更新实体
			public void update(T entity) {
				entityManager.merge(entity);
			}
			// 删除实体
			public void delete(T entity) {
				entityManager.remove(entity);
			}
			// 根据ID删除实体
			public void delete(Class<T> entityClazz, Serializable id) {
				entityManage.createQuery("delete " + entityClazz.getSimpleName() + " en where en.id = ?0").setParameter(0, id).executeUpdate();
			}
			// 获取所有实体
			public List<T> findAll(Class<T> entityClazz) {
				return find("select en from " + entityClazz.getSimpleName() + " en");
			}
			// 获取实体总数
			public long findCount(Class<T> entityClazz) {
				List<?> list = find("select count(*) from " + entityClazz.getSimpleName());
				if (list != null && list.size() == 1) {
					return (Long)list.get(0);
				}
				return 0;
			}
			// 根据JPQL语句查询实体
			@SuppressWarnings("unchecked")
			protected List<T> find(String jpql) {
				return (List<T>) enetityManager.createQuery(jpql).getResultList();
			}
			// 根据带占位符参数的JPQL语句查询实体
			@SuppressWarnings("unchecked")
			protected List<T> find(String jpql, Object.. params) {
				// 创建查询
				Query query = enetityManager.createQuery(jpql);
				// 为包含占位符的HQL语句设置参数
				for (int i = 0, len = params.length; i < len; i++) {
					query.setParameter(i + "", params[i]);
				}
				return (List<T>) query.getResultList();
			}
			/**
			 * 使用JPQL语句进行分页查询操作
			 * @param jpql 需要查询的JPQL语句
			 * @param pageNo 查询地pageNo页的记录
			 * @param pageSize 每页需要显式的记录数
			 * @return 当前页的所有记录
			 */
			@SuppressWarnings("unchecked")
			protected List<T> findByPage(String jpql, int pageNo, int pageSize) {
				// 创建查询
				Query query = enetityManager.createQuery(jpql).
											// 执行分页
											setFirstResult((pageNo - 1) * pageSize).
											setMaxResults(pageSize).
											getResultList();
			}
			/**
			 * 使用带占位符的JPQL语句进行分页查询操作
			 * @param jpql 需要查询的JPQL语句
			 * @param pageNo 查询地pageNo页的记录
			 * @param pageSize 每页需要显式的记录数
			 * @param params 传入的占位符参数
			 * @return 当前页的所有记录
			 */
			@SuppressWarnings("unchecked")
			protected List<T> findByPage(String jpql, int pageNo, int pageSize, Object.. params) {
				// 创建查询
				Query query = enetityManager.createQuery(jpql);
				// 为包含占位符的HQL语句设置参数
				for (int i = 0, len = params.length; i < len; i++) {
					query.setParameter(i + "", params[i]);
				}
				// 执行分页，返回查询结果
				return query.setFirstResult((pageNo - 1) * pageSize).
							 			 setMaxResults(pageSize).
										 getResultList();
			}
		}

为了让Spring将线程安全的`EntityManager`注入该成员变量，还需要在Spring配置文件中添加如下配置：

		<!-- 该Bean后处理器会告诉Spring处理DAO组件中的`@PersistenceContext`注解 -->
		<bean class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/>

#### 使用声明式事务 ####

和前面介绍的使用声明式事务一样，唯一需要改变的是事务管理器的实现类，在使用JPA进行持久化访问的环境中，事务管理器应配置如下：

		<!-- 配置针对JPA的局部管理事务 -->
		<bean id="transactionManager"
					class="org.springframework.orm.jpa.JpaTransactionManager"
					p:entityManagerFactory-ref="emf"/>

也可使用注解`@Transactional`来配置声明事务。一般都是在应用的业务逻辑层增加事务控制，因为业务逻辑组件里的一个方法往往代表一次业务操作，而只有对业务操作增加事务控制才有意义。对DAO组件的一次原子操作添加事务控制没有任何意义。
为业务逻辑组件添加了`@Transactional`注解后，还需要咋Spring配置文件中配置根据注解生成事务代理：

		<!-- 配置针对JPA的局部管理事务 -->
		<bean id="transactionManager"
					class="org.springframework.orm.jpa.JpaTransactionManager"
					p:entityManagerFactory-ref="emf"/>
		<!-- 根据事务注解来生成事务代理 -->
		<tx:annotation-driven transaction-manager="transactionManager"/>
		
## 企业应用开发的思考和策略 ##

优秀的企业级应用要求必须具备：

- 良好的可扩展性和可伸缩性
- 快捷，可控的开发
- 稳定性，高效性
- 花费最小化，利益最大化

### 开发建议 ###

- 使用建模工具
- 利用优秀的框架
- 选择性地扩展
- 使用代码生成器

### 常见的设计模式 ###

设计模式是对处于特定环境下，经常出现的某类软件开发问题的，一种相对成熟的设计方案。

设计模式常被分为三类：

- 创建型：创建对象时，不再直接实例化对象。主要包括：简单工厂模式，工厂方法，抽象工厂模式，单例模式，生成器模式和原型模式；
- 结构型：帮助将多个对象组织成更大的结构。主要包括：适配器模式，桥接模式，组合器模式，装饰器模式，门面模式，享元模式和代理模式；
- 行为型：帮助系统间各对象的通信，以及如何控制复杂系统中的流程。主要包括：命令模式，解释器模式，迭代器模式，中介者模式，备忘录模式，观察者模式，状态模式，策略模式，模板模式和访问者模式。

一段代码可能使用了多种设计模式。

#### 单例模式 ####

单例模式：单例类只能创建一个实例。

Spring容器中Bean的作用域默认就是`singleton`。Spring推荐将所有业务逻辑组件，DAO组件，数据源组件等配置成单例的行为方式，因为这些组件无须保存任何用户状态，所有客户端都可以共享这些业务逻辑组件，DAO组件。

单例模式实现的共同特点：

- 私有构造器
- `public static`修饰的对象创建方法
- 一个静态属性用于保存已创建的实例

#### 简单工厂模式 ####

2个对象，A对象需要调用B对象的方法。传统方法是让A对象使用`new`关键字创建一个B实例，然后调用该实例的方法。简单工厂模式是让B类实现一个接口IB，A类根据面向接口编程的原则和IB耦合，因为IB是接口，所以不能使用`new`关键字创建IB对象，而是定义一个工厂类`IBFactory`，负责创建IB实例。A类通过调用`IBFactory`工厂类的方法来获得IB实例，A类虽然必须和`IBFactory`工厂类和`IB`接口耦合，但可以通过该工厂类获得所有IB接口实现类的实例，具体获得哪种实现类的实例由工厂类内部的判断逻辑决定。从而实现了A类和具体的实现类之间的耦合。这种通过接口和工厂类将负责创建同接口不同实现类的对象的模式称为简单工厂模式。

这种模式让对象的调用者和对象的创建过程分离。Spring容器就是Spring框架中的工厂，应用的所有组件都由Spring容器复制创建，不仅如此，它还可以管理Bean实例之间的依赖关系；如果Bean实例具有单例作用域，容器还会缓存该Bean的实例，确保每次获取该Bean实例时，返回的都是同一个实例。

#### 工厂方法和抽象工厂 ####

简单工厂模式中，由工厂类负责逻辑判断和对象创建。如果不想把逻辑判断放在工厂类中，可以为每个具体的产品类提供一个单独的工厂类，然后为这些具体工厂类抽象出一个工厂类接口。

系统中将存在2个抽象：产品的抽象和工厂类的抽象，他们之间的关系是：

- 抽象工厂生产抽象产品
- 抽象工厂的实现类和抽象产品的实现类之间是一一对应的关系

这种模式下，对象调用者的代码需要与具体的抽象工厂实现类耦合。也就是说对象调用者必须指定具体的抽象工厂实现类来生产所需的具体产品的对象。

抽象工厂模式适用于一个产品X由多种抽象组件A，B，C等组合而成的场景。此时：

- 每个组件都是一种抽象产品；
- 为抽象产品X建立的工厂类里每个组件都有一个所属抽象产品对应的工厂抽象类或抽象方法。
- 每个产品X的实现类都必须为抽象组件返回具体的组件实现类对象。例如，产品型号X1包含的组件具体型号为（A1，B2，C1），型号X2包含的组件型号为（A3，B1，C2）等；

#### 代理模式 ####

代理模式：当客户端代码不能或不想直接访问被调用对象，而是额外创建一个代理对象返回给客户端使用。实质就是一个Java对象代表另一个Java对象来采取行动，代理对象可以在客户和目标对象之间起到中介作用。客户端不能分辨也无须分辨代理对象与真实对象的区别，客户端代码面向接口编程。

代理模式应用的场景：

- 出于性能考虑，把目标对象的创建推迟到真正使用它时才创建，从而减少对象在内存中的存活时间，节省系统的内存开销；
- 当目标对象的功能不足以满足客户端需求时，代理对象可以增强原目标对象的功能。

作为AOP的一种实现，Java的动态代理模板通常需要以下几个部分：

- 被代理类；
- 增强处理类；
- `java.lang.reflect.Proxy`类和`java.lang.reflect.InvocationHandler`接口

常见步骤：

1. 实现`java.lang.reflect.InvocationHandler`接口，在实现类中定义一个成员变量代表需要被代理的对象，然后重写`invoke()`方法，在方法内部加入增强处理后再通过该成员变量调用被代理对象的方法，从而完成增强被代理类的作用。在执行动态代理对象的所有方法时，就会被替换成执行重写的`invoke()`方法。
2. 创建一个代理工厂类，在内部创建一个上面定义的`InvocationHandler`接口实现类的实例，并将其作为参数传递给`java.lang.reflect.Proxy`类的`newProxyInstance()`方法，从而返回被代理对象的代理。
3. 在程序中创建被代理类的对象，然后使用代理工厂类为其创建代理对象。

#### 命令模式 ####

命令模式应用的场景：某个方法需要完成某一个功能，该功能的大部分步骤已经确定，但仍有部分内容需要在执行该方法时才可以确定，这些内容将作为参数传入该方法。在Java中，传入的参数是一个对象，通常是某个接口的匿名实现类的实例，该接口通常被称为命令接口，因此也称为命令模式。

在Hibernate框架中，`HibernateCallback`接口就是一个典型的`Command`接口，封装了自定义的持久化处理，`HibernateTemplate`的大部分持久化操作都可以通过一个方法来实现，该对象简化了Hibernate的持久化操作，但丢失了使用Hibernate持久化操作的灵活性。使用`HibernateTemplate`中的带`HibernateCallback`类型参数的`execute()`方法，就可以利用传入的`HibernateCallback`对象的`doInHibernate()`方法就是自定义的持久化处理。

应用步骤：

- 提供一个命令接口，该接口代表未来需要注入的命令；
- 在应用类中为需要注入命令的方法提供一个命令接口类型的参数。在实践中可以用匿名类或Lambda表达式来实现接口。

#### 策略模式 ####

策略模式用于封装系列的算法，这些算法通常被封装在一个被称为`Context`的类中，客户端可以自由选择其中一种算法，或让`Context`类为客户端选择一个最佳的算法。策略模式中客户端代码需要和不同的策略类耦合。可以考虑使用配置文件来指定具体的策略，从而分离客户端代码和具体打折策略类。

应用策略模式：

- 策略接口；
- 策略类，每个策略类都是策略接口的实现类；
- 决策类（通常以`-Context`结尾），该决策类负责选择执行合适的策略类。

#### 门面模式 ####

门面模式（Facade ）：被称为正面模式，外观模式，用于将一组复杂的类包装到一个简单的外部接口中。通过组合将业务关联的类组织在一起，通过调用它们的方法完成一个业务逻辑操作，并为该操作提供一个访问接口。简单地讲，就是把多个类对象的方法调用整合到一个类的方法调用中。

#### 桥接模式 ####

桥接模式是一种结构型模式，主要用于当某个类有两个或两个以上的维度变化。桥接模式的做法是把变化部分抽象出来，使变化部分与主类分离开来，从而将多个维度的变化彻底分离，最后提供一个管理类来组合不同维度的变化，通过这种组合来满足业务需求。

简单地讲：就是把变化的部分抽取出来，通过依赖注入将两部分关联，仍然是通过组合+接口的方式实现。

#### 观察者模式 ####

观察者模式定义了对象间的一对多依赖关系，让一个或多个观察者对象观察一个主题对象。当主题对象的状态发生变化时，系统能通知所有的依赖于此对象的观察者对象，从而使其能够自动做出反应。

在观察者模式中，被观察的对象常常也被称为目标和主题（Subject），依赖的对象被称为观察者（Observer）。

观察者接口应该包含一个更新方法，该方法接受一个被观察者接口类型的参数，该参数代表被观察对象，也就是主题或目标。

被观察者应该有一个共同的抽象基类，负责提供一个方法用于注册一个新的观察者，另一个方法来删除一个已注册的观察者。除此之外，还需要提供一个方法用来，当具体被观察对象的状态发生改变时，通知所有在册的观察者做出反应。

综上所述：观察者模式通常包含如下4个角色：

- 被观察者的抽象基类：通常会持有多个观察者对象的引用。Java提供了`java.util.Observable`基类来代表被观察者的抽象基类；
- 观察者接口：所有被观察对象应该实现的接口，通常只包含一个抽象方法`update()`。Java提供了`java.util.Observer`基类来代表观察者的抽象基类；；
- 被观察者实现类：该类继承`Observable`基类；
- 观察者实现类：实现`Observer`接口。

### 常见的架构设计策略 ###

#### 贫血模型 ####

贫血模型是指`Domain Object`只是单纯的数据类，不包含业务逻辑方法，即每个`Domain Object`类只包含相关属性，并为每个属性提供基本的setter和getter方法。所有的业务逻辑都由业务逻辑组件实现，这种`Domain Object`就是贫血模型中的领域对象。

贫血模型相当于抛弃了Java面向对象的性质，实际上是以数据结构代替了对象。贫血模型中的领域对象不具备该有的业务逻辑功能，仅仅是ORM框架持久化所需的持久化实体类，仅是数据载体。

系统DAO组件负责完成持久化操作，基本的CRUD操作都应该在DAO组件中实现，但根据业务逻辑的不同需要，不同的DAO组件可能有数量不等的查询方法。

在贫血模型中，业务逻辑组件作为DAO组件的门面，封装了全部的业务逻辑方法，Web层仅与业务逻辑组件交互即可，无须访问底层的DAO组件。Spring的声明式事务管理将负责业务逻辑方法的事务性。

#### 领域对象模型 ####

该模型中领域对象包含了业务相关的逻辑方法，但不要在领域对象中对消息回复完成持久化，如需完成持久化，必须调用DAO组件；一旦调用DAO组件，将造成DAO组件和领域对象的双向依赖；另外，领域对象中的业务逻辑方法还需要在业务逻辑组件中代理，才能真正实现持久化。

可重用度高，与领域对象密切相关的业务方法需要放在领域对象中实现。

## 简单工作流 ##

工作流：企业或组织日常工作的固定流程。

Web应用一般有如下分层：

- 表现层
- MVC层
- 业务逻辑层
- DAO层
- 领域对象层
- 数据库服务层

通常建议按细粒度的模块来设计Service组件，让业务逻辑组件作为DAO组件的门面，符合门面模式的设计。DAO组件负责持久化技术上的改变，业务逻辑组件负责业务逻辑上的改变。

### 设计持久化实体 ###

根据系统需求提取应用中的对象，将这些对象抽象成类，在抽取出需要持久化保存的类，这些需要持久化保存的类就是持久化对象（PO）。持久化对象之间的关联关系以成员变量的方式表现出来，通常对应数据库里的主，外键约束。

Hibernate只要求持久化对象提供无参数的构造器，如果需要将这些对象放入`HashSet`集合中，还应该根据实际需要重写`hashCode()`和`equals()`两个方法（不能使用标识属性），因为持久化对象处于瞬态时，这些对象的标识属性可能是null。

Hibernate支持将普通的POJO映射为PO，但这些POJO应遵循以下规则：

- 提供实现一个无参数构造器
- 提供一个标识属性（identifier property）用于标识该实例
- 使用非final的类。

对于所有的1-N的关联关系，不要使用“1”的一端控制关系，建议为`@OneToMany`注解增加`mappedBy`属性，让“N”的一端来控制关联关系。

Hibernate支持以下三种继承映射策略：

- 整个类层次对应一个表。所有父类，子类的记录都在一张表中，包含在父类和所有子类中定义的字段；
- 连接子类的映射策略。类表只包含属于自己的字段，子类表通过外键与父类表关联；
- 每个具体类对应一个表。所有类表都包含父类中定义的字段。

### 实现DAO层 ###

在Hibernate层（持久化层）之上可使用DAO组件再次封装数据库操作，每个DAO组件可对一个数据库表完成基本的CRUD等操作。

DAO的实现至少需要如下三个部分：

- DAO工厂类
- DAO接口
- DAO接口的实现类

DAO组件中通用的方法：

- `get(Serializable id)`：根据主键加载持久化实例
- `save(Object entity)`：保存持久化实例
- `update(Object entity)`：更新持久化实例
- `delete(Object entity)`：删除持久化实例
- `delete(Serializable id)`：根据主键删除持久化实例
- `findAll()`：获取数据表中全部的持久化实例





















