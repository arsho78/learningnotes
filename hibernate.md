## ORM和Hibernate ##

ORM(Object/Relation Mapping)：对象/关系数据库映射。ORM工具的唯一作用就是把持久化对象的保存，删除，修改等操作转换成对数据库的操作。

### 基本映射方式 ###

- 数据表映射类
- 数据表的行映射对象（即实例）
- 数据表的列（字段）映射对象的属性

## Hibernate入门 ##

Hibernate框架中的PO（Persistent Object）- 持久化对象，完全采用普通的Java对象（POJO），无须继承任何父类，或者实现任何接口，代码不会受到污染（依赖其他类库），因此属于低侵入式的设计。

		// Hibernate框架中的PO
		PO = POJO + 持久化注解

Hibernate的配置信息可以使用properties属性文件或XML文件来配置。XML配置文件的默认文件名为`hibernate.cfg.xml`，当程序调用`Configuration`对象的`configure()`方法时，会自动加载该文件。

配置文件的根元素是`<hibernate-configuration.../>`，根元素里有`<session-factory.../>`子元素，该元素依次有很多`<property.../>`子元素，这些子元素负责配置Hibernate连接数据的必要信息。

Hibernate不推荐使用`DriverManager`连接数据库，而是推荐使用数据源来管理数据库连接，如C3P0数据源。数据源是一种提高数据库连接性能的常规手段，负责维持一个数据库连接池，当抽象创建数据源实例时，系统会一次性创建多个数据库连接，并将这些连接放入连接池中管理。当程序需要访问数据库时，无须重新创建数据库连接，而是从连接池中取出一个空闲的数据库连接使用。当访问结束后，无须关闭该连接，而是将连接退还给连接池即可。

使用Hibernate进行持久化操作通常需要以下步骤：

1. 开发持久化类，由POJO+持久化注解组成；
2. 获取`SessionFactory`
	1. 传统方法：使用`Configuration`

		// 使用hibernate.cfg.xml配置文件创建Configuration对象
		Configuration config = new Configuration().configure();
		使用Configuration对象创建SessionFactory对象
		SessionFactory sf = config.buildSessionFactory();
	
	2. Hibernate5.x推荐的方法：使用`Metadata`

		// 使用hibernate.cfg.xml配置文件创建ServiceRegistry对象
		StandardServiceRegistry registry = new StandardServiceRegistryBuilder().configure().build();
		// 使用ServiceRegistry对象创建Metadata对象
		Metadata metadata = new MetadataSources(registry).buildMetadata();
		// 使用Metadata创建SessionFactory对象
		SessionFactory sf = metadata.buildSessionFactory();
		
3. 获取`Session`并用其打开事务

		// 使用SessionFactory对象获取Session对象
		Session session = sf.openSession();
		// 打开事务
		Transaction tx = session.beginTransaction();

4. 用面向对象的方式操作持久化类实体；
5. 将完成操作的持久化类实体保存在`Session`对象中，提交事务，关闭`Session`和`SessionFactory`。

		// 保存实体
		session.save(obj);
		// 提交事务
		tx.commit();
		// 关闭Session
		session.close();
		// 关闭SessionFactory
		sf.close();

根据PO和Session的关系，PO可处于以下三种状态：

- 瞬态：从未与Session关联过的PO实例处于瞬态；
- 持久化：PO实例与Session关联起来，且该实例对应到数据库记录，则该实例处于持久化状态；
- 脱管：PO实例曾经与Session关联过，但因为Session的关闭或其他原因，实例脱离了Session的管理，这种状态称为脱管状态。

对PO对象的操作必须在`Session`的管理下才能同步到数据库；
`Session`由`SessionFactory`工厂产生；
`SessionFactory`是数据库编译后的内存镜像，通常一个应用对应一个`SessionFactory`对象；
`SessionFactory`由`Configuration`对象生成，该对象负责加载Hibernate的配置文件。

## Hibernate体系结构 ##

- `SessionFactory`：单个数据库映射关系通过编译后的内存镜像。是线程安全的，也是生成`Session`的工厂，本身需要依赖于`ConnectionProvider`。对象可以在进程或集群的级别上，为事务之间可重用的数据提供可选的二级缓存。
- `Session`：应用程序与持久存储层之间交互操作的一个单线程对象。所有的持久化对象都必须在`Session`的管理下才能进行持久化操作。在底层封装了JDBC连接，也是`Transaction`的工厂。`Session`对象持有必选的一级缓存，在显式执行flush之前，所有持久化操作的数据都在缓存中的`Session`对象这里。
- 持久化对象（Persistent Object）：系统创建的POJO实例，与普通的POJO实例的区别是它们正与一个`Session`关联。
- 瞬态对象和脱管对象：新创建的未与任何Session关联的Java实例此时处于瞬态状态，或者说是新创建的还没有开始持久化的实例；一旦实例曾经执行过持久化操作，而相关联的Session一旦被关闭，则该对象进入脱管状态。
- 事务（Transaction）：代表一次原子操作。是对底层具体的JDBC，JTA以及CORBA事务的抽象。某些情况下，一个`Session`之内可能有多个`Transaction`对象。所有的持久化操作都应该在事务管理下进行，即使是只读操作。
- 连接提供者（ConnectionProvider）：生成JDBC连接的工厂，通过抽象将应用程序和底层的`DataSource`或`DriverManager`隔离开。
- 事务工厂（TransactionFactory）：生成`Transaction`对象的工厂

## 深入Hibernate配置文件 ##

每个Hibernate配置文件对应一个`Configuration`对象。

`org.hibernate.cfg.Configuration`实例代表了应用程序到SQL数据库的配置信息，提供了一个`buildSessionFactory()`方法，负责产生一个不可变的`SessionFactory`对象。

所有Hibernate属性的名字和语义都在`org.hibernate.cfg.Environment`中定义。

- 可以在配置文件中指定所处理的Hibernate持久化类，然后使用该配置文件创建的`Configuration`对象就可以处理这些持久化类。
- 或者，先创建`Configuration`对象，然后通过`addAnnotatedClass()`方法或`addPackage()`方法添加指定包下的所有持久化类。

有以下几种配置Hibernate的方式：

- 使用`hibernate.properties`文件作为配置文件；
- 使用`hibernate.cfg.xml`文件作为配置文件；
- 不使用任何配置文件，在代码中以编码方式创建`Configuration`对象

`Configuration`实例的唯一作用就是创建`SessionFactory`实例，一旦`SessionFactory`实例创建完毕，`Configuration`对象也就被丢弃了。

**1. 使用`hibernate.properties`配置文件（不推荐）**

这种配置方式没有在配置文件中提供添加Hibernate持久化类的方式。因此必须调用`Configuration`实例的`addAnnotatedClass()`或`addPackage()`方法来添加持久化类。

在Hibernate发布包的`project/etc`路径下，提供了一个`hibernate.properties`样板文件，内有配置的注释说明。

使用这种方式创建`Configuration`实例的代码如下：

		// 实例化Configuration
		Configuration conf = new Configuration();
		// 通过多次调用addAnnotatedClass方法添加持久化类
		conf.addAnnotatedClass(Item.class);
		conf.addAnnotatedClass(ShopCart.class);
		
**2. 使用`hibernate.cfg.xml`配置文件**

使用这种方式创建`Configuration`实例的代码如下：

		// 实例化Configuration
		// configure()方法负责加载hibernate.cfg.xml文件
		Configuration conf = new Configuration().configure();
		
**3. 不使用配置文件创建`Configuration`对象（极端方式，通常不会使用）**

这种方式需要使用`Configuration`类的API来在代码中手工配置该类的实例。常用的API包括：

- `Configuration addAnnotatedClass(Class annotatedClass)`：添加一个持久化类；
- `Configuration addPackage(String packageName)`：添加指定包下的所有持久化类；
- `Configuration setProperties(Properties properties)`：设置一系列属性，这些属性通过`Properties`实例传入；
- `Configuration setProperty(String PropertyName, String value)`：设置一个单独的属性。

这种方式在代码中采用建造者模式，通过链式调用的方法来实例化`Configuration`对象。在实际应用中因繁琐用处不大，但可以在特殊情况下将敏感且不会改变的数据硬编码在代码中。

### JDBC连接属性 ###

- `hibernate.connection.driver_class`：设置数据库的驱动；
- `hibernate.connection.url`：设置数据库的地址；
- `hibernate.connection.username`：设置数据库的用户名；
- `hibernate.connection.password`：设置数据库的密码；
- `hibernate.connection.pool_size`：设置Hibernate数据库连接池的最大并发连接数，推荐使用c3p0的连接池来代替；
- `hibernate.dialect`：设置数据库的方言。

Hibernate自带的连接池不推荐使用，应使用c3p0或Proxool连接池来代替：

        <!--数据库URL，其中hibernate是本应用连接的数据库名-->
        <property name="connection.url">
            jdbc:mysql://localhost:3306/hibernate?useSSL=true
        </property>
        <!--连接数据库所用的驱动-->
        <property name="connection.driver_class">
            com.mysql.jdbc.Driver
        </property>
        <!--连接数据库的用户名-->
        <property name="connection.username">
            root
        </property>
        <!--连接数据库的密码-->
        <property name="connection.password">
            mysql78
        </property>
        <!--连接池的最大连接数-->
        <property name="hibernate.c3p0.max_size">20</property>
        <!--连接池的最小连接数-->
        <property name="hibernate.c3p0.min_size">1</property>
        <!--连接池里连接的超时时长-->
        <property name="hibernate.c3p0.timeout">5000</property>
        <!--连接池里最大缓存多少个Statement对象-->
        <property name="hibernate.c3p0.max_statements">100</property>
        <property name="hibernate.c3p0.idle_test_period">3000</property>
        <property name="hibernate.c3p0.acquire_increment">2</property>
        <property name="hibernate.c3p0.validate">true</property>
		
### 数据库方言 ###

数据库方言是指不同数据库对标准SQL语言所做的专有扩展，这些扩展在语法上会有细微差异。

### JNDI数据源的连接属性 ###

Hibernate可以不用自己管理数据源，而是直接访问容器管理数据源，此时使用JNDI（Java Naming Directory Interface，Java命名目录接口）配置数据源。

- `hibernate.connection.datasource`：指定JNDI数据源的名字；
- `hibernate.jndi.url`：指定JNDI提供者的URL，可选属性，如果JNDI与Hibernate持久化访问的代码处于同一个应用中，无须指定；
- `hibernate.jndi.class`：指定JNDI `InitialContextFactory`的实现类，可选属性，如果JNDI与Hibernate持久化访问的代码处于同一个应用中，无须指定；
- `hibernate.connection.username`：指定数据库的用户名，可选属性；
- `hibernate.connection.password`：指定数据库的密码，可选属性。

即使使用JNDI数据源，也可以指定数据库的方言来实现持久化层访问的优化。

如果数据源所在容器支持跨事务资源的全局事务管理，从JNDI数据源获得的JDBC连接，可自动参与容器管理的全局事务，而不仅仅是Hibernate的局部事务。

### Hibernate事务属性 ###

- `jta.UserTransaction`：属性值是一个JNDI名，Hibernate将使用`JTATransactionFactory`从应用服务器获取JTA `UserTransaction`；
- `hibernate.transaction.factory_class`：Hibernate所用的事务工厂的类型，属性值必须是`TransactionFactory`的直接或间接子类；
- `hibernate.transaction.manager_lookup_class`：属性值应为一个`TransactionManagerLookup`类名，当使用JVM级别的缓存时，或者在JTA环境中使用hilo生成器策略时，需要该类；
- `hibernate.transaction.flush_before_completion`：`Session`是否在事务完成后自动将数据刷新到底层数据库。最好使用`Context`相关的`Session`管理；
- `hibernate.transaction.auto_close_session`：是否在事务结束后自动关闭`Session`。最好使用`Context`相关的`Session`管理。

### 二级缓存相关属性 ###

- `hibernate.cache.use_second_level_cache`：设置是否启用二级缓存；
- `hibernate.cache.region.factory_class`：设置二级缓存`RegionFactory`实现类的类名；
- `hibernate.cache.region_prefix`：设置二级缓存区名称的前缀；
- `hibernate.cache.use_minimal_puts`：以频繁的读操作为代价，优化二级缓存以实现最小化写操作；
- `hibernate.cache.use_query_cache`：设置是否允许查询缓存；
- `hibernate.cache.query_cache_factory`：设置查询缓存工厂的类名，查询工厂必须实现`QueryCache`接口，默认值为内建的`StandardQueryCache`；
- `hibernate.cache.use_structured_entries`：设置是否强制Hibernate以可读性更好的格式将数据存入缓存。

### 外连接抓取属性 ###

外连接抓取通过在单个select语句中使用`outer join`来一次抓取多个数据表里的数据。

外连接抓取允许在单个select语句中，通过`@ManyToOne`，`@OneToMany`，`@ManyToMany`和`OneToOne`等关联获取连接对象的整个对象图。

将`hibernate.max_fetch_depth`设为0，将在全局范围内禁止外连接抓取，设为1或更高值能启用N-1或1-1的外连接抓取。除此之外，还应该在持久化注解中通过`fetch=FetchType.EAGER`来指定这种外连接抓取。

### 其他常用的配置属性 ###

- `hibernate.show_sql`：是否在控制台输出Hibernate持久化操作底层所使用的SQL语句，true或者false；
- `hibernate.format_sql`：是否将SQL语句转换成格式良好的SQL，true或者false；
- `hibernate.use_sql_comments`：是否在Hibernate生成的SQL语句中添加有助于调试的注释，true或者false；
- `hibernate.jdbc.fetch_size`：JDBC抓取数量的大小，接受一个整数值，实质是调用`Statement.setFetchSize()`方法；
- `hibernate.jdbc.batch_size`：Hibernate使用JDBC2的批量更新的大小，接受一个整数值，建议取5~30之间的值；
- `hibernate.connection.autocommit`：是否自动提交，通常不建议自动提交；
- `hibernate.hbm2ddl.auto`：当创建`SessionFactory`实例时，是否根据持久化类的映射关系自动创建数据库表。属性值可以是：
	- `validate`：
	- `update`：如果库中没有对应的数据表才会创建新表，否则使用已有的数据表进行操作；
	- `create`：每次都会重新建表，因此前面插入的数据会丢失；
	- `create-drop`：每次显式关闭`SessionFactory`时，都会自动drop刚刚创建的数据表。

## 深入理解持久化对象 ##

### 持久化类的要求 ###

Hibernate中的持久化类基本上都是POJO，但仍应该遵守以下规则：

- 提供一个无参数的构造器。构造器可以不采用`public`访问控制符，Hibernate使用`Constructor.newInstance()`方法来创建实例。通常为了能够为持久化类生成代理，构造器的访问控制符应设为至少包可见，即大于等于默认的访问控制符。
- 提供一个标识属性。标识属性通常映射数据库表的主键字段。该属性可以使用任意名称，类型可以是任何的基本类型（建议使用基本类型的包装类代替），`java.util.String`或`java.util.Date`，自定义类（如果使用了数据库表的联合主键）。建议使用可以为空的类型来作为标识属性的类型，因此应尽量避免基本类型。
- 为持久化类的每个成员变量提供setter和getter方法。Hibernate认可如下形式的方法名：`getXxx(), isXxx(), setXxx()`。如果需要，可以切换属性的访问策略。
- 使用非final的类。在运行时生成代理是Hibernate的一个重要功能。如果持久化类没有实现任何接口，Hibernate使用Javassist生成代理，该代理对象是持久化类的子类的实例，因此持久化类不能声明为final，同时应该避免在非final类中声明`public final`的方法，否则必须通过设置`lazy=false`来禁用代理。另一种生成代理的方法是让持久化类实现一个接口，此时将使用JDK的动态代理。
- 重写`equals()`和`hashCode()`方法。如果需要把持久化类放入Set中（当需要进行关联映射时，推荐这样做），或需要重用脱管实例，则应该为该持久化类重写`equals()`和`hashCode()`方法。最简单的实现逻辑是比较2个对象的标识属性的值，但这种方法不适用于自动生成标识属性值的对象。Hibernate仅为持久化对象提供标识值，一个瞬态的实例将不会有任何标识值。因此，如果一个瞬态的实例在持久化前就放入Set中，则持久化后该实例的标识值会发生变化，从而影响hashCode的返回值，这将违反Set的规则。通常建议使用业务键值相等来实现这两个方法。业务键值是指可以唯一标识一个实例的单个或多个逻辑字段的组合。

持久化类可以拥有子类，子类从父类继承标识属性。

### 持久化类的状态 ###

- 瞬态：通过new新创建且尚未与Hibernate Session关联的实例处于瞬态。瞬态对象不会被持久化到数据库中，也不会被赋予持久化标识。如果程序中失去了瞬态对象的引用，对象将被垃圾回收机制销毁。
- 持久化：持久化实例在数据库中有对应的记录，并拥有一个持久化标识。持久化实例可以是刚刚保存的，也可以是刚刚被加载的，无论哪一种，都必须与Hibernate Session关联。Hibernate会检测到处于持久化对象的改动，在当前操作执行完成时将对象数据写回数据库，开发者不需要手动执行update。
- 脱管：某个实例曾经处于持久化状态，但随着与之关联的Session被关闭，该对象就变成脱管状态。脱管对象的引用仍然有效，可继续被修改。如果让脱管对象重新与某个Session关联，则该对象将重新转换为持久化状态，而脱管期间的改动不会丢失，也可被写入数据库。应用程序事务就利用此功能，让对象处于处于脱管状态，此时对该对象的操作无须锁定数据库，不会造成性能的下降。

### 改变持久化类状态的方法 ###

#### 持久化实体 ####

瞬态->持久化：

- `Serializable save(Object obj)`：立即将持久化对象对应的数据插入数据库，并返回该持久化对象的标识属性值（即对应记录的主键值）；
- `void persist(Object obj)`：当在一个事务外部被调用时，不会立即转换成insert语句，也没有任何返回值。

把瞬态实体转为持久化状态时，Hibernate会在底层对应地生成一条insert语句，负责把该实体对应的数据记录插入数据库。

如果实例的标识属性是`generated`的类型，则Hibernate会在执行`save()`方法时自动生成标识属性值，并将该值分配给该实例；
如果实例的标识属性是`assigned`的类型，或者是复合主键（composite key），则应在执行`save()`方法前手工将标识属性值赋给该实例。

#### 根据主键加载持久化实体 ####

通过`load()`方法加载一个持久化实例，就是根据持久化类的标识属性值加载持久化类实例，其实质就是根据主键值从数据表中加载一条新记录。

		<T> T load(Class<T> theClass, Serializable id)

		Return the persistent instance of the given entity class with the given identifier, assuming that the instance exists. This method might return a proxied instance that is initialized on-demand, when a non-identifier method is accessed.

		You should not use this method to determine if an instance exists (use get() instead). Use this only to retrieve an instance that you assume exists, where non-existence would be an actual error.

		Parameters:
				theClass - a persistent class
				id - a valid identifier of an existing persistent instance of the class
		Returns:
				the persistent instance or proxy 

如果没有匹配的数据库记录，可能抛出`HibernateException`异常。但如果在持久化注解中指定了延时加载，则会返回一个未初始化的代理对象，该代理对象并没有加载记录，只有在程序调用该代理对象的某个方法时，Hibernate才会取访问数据库。非常适用于当希望在某对象中创建一个指向另一个对象的关联，又不想在从数据库中加载该对象的同时立即装载所关联的全部对象。

`get()`方法和`load()`方法类似，区别在于是否延迟加载，`get()`方法会立即访问数据库，如果没有相应的记录，返回null，而不是一个代理对象。

当程序通过以上两个方法加载持久化实例时，Hibernate会在底层生成一条对应的select语句，该语句带有`where <主键列>=<标识属性值>`子句，表示根据主键加载。还可以使用重载方法里的`LockOptions`参数来指定锁模式，该参数可有2个值：`READ`和`UPGRADE`

#### 更新持久化实体 ####

对处于持久化的实体所做的修改会在Session flush之前被自动保存到数据库中，无须程序调用其他方法（不需要调用update()方法）。

如果调用持久化实体的setter方法修改了某个属性，则Hibernate会在Session Flush之前生成一条update语句，该语句带有`where <主键列>=<标识属性值>`子句，表示根据主键值修改特定记录。

#### 更新脱管实体 ####

当程序修改了脱管实体的属性后，程序应该显式地使用新的Session来保存这些修改。Hibernate提供了以下方法来保存这些修改：

- `update()`：脱管->持久化；
- `merge()`：生成对象的副本，该副本处于持久化状态，原对象还是处于脱管状态；
- `updateOrSave()`：如果实例处于瞬态，则执行`save()`；如果处于脱管状态，则执行`update()`；
- `Session.LockRequest`的`lock()`方法也可以将某个脱管对象重新持久化，但该对象必须没有被修改过。

#### 删除持久化实体 ####

`delete()`方法删除持久化实体时，该实体对应的数据记录也会被删除。

## 深入Hibernate映射 ##

Hibernate提供了三种方式将POJO变为PO类：

- 使用持久化注解。以JPA标准注解为主，特殊需要时仍需使用Hibernate本身提供的注解，主流用法。
- 使用JPA提供的XML配置描述文件（XML deployment descpritor）：让Hibernate的PO类与JPA实体类兼容，较少使用。
- 使用Hibernate传统的XML映射文件（.hbm.xml文件的形式）：传统方式。

Hibernate的PO类通常采用以下两个注解来修饰：

- `@Entity`：被该注解修饰的POJO就是一个实体。可通过`name`属性指定该实体类的名称。系统默认以类的类名作为实体类的名称。
- `@Table`：指定持久化类所映射的表，支持以下属性：

属性                | 是否必需 | 说明
---                 | ----     | ---
`catalog`           | 否       | 将持久化类所映射的表放入指定的catalog中。没有指定则放入默认的catalog中
`indexes`           | 否       | 为持久化类所映射的表设置索引，属性值是一个`@Index`注解的数组
`name`              | 否       | 持久化类所映射的表名。默认值与持久化类的类名相同
`schema`            | 否       | 将持久化类所映射的表放入指定的schema中。没有指定则放入默认的schema中
`uniqueConstraints` | 否       | 为持久化类所映射的表设置唯一约束，属性值是一个`@UniqueConstraint`注解的数组

- `@UniqueConstraint`：用于为数据表定义唯一约束，只有一个属性:
	- 'columnNames`：属性值是一个字符串数组，每个字符串元素代表一个数据列，使用这些数据列的组合作为唯一约束。
- `@Index`：为数据表定义索引。具有如下属性：

属性         | 是否必需 | 说明
---          | ----     | ---
`columnList` | 是       | 设置对哪些列建立索引，该属性的值可指定多个数据列的列名
`name`       | 否       | 设置索引的名称
`unique`     | 否       | 设置该索引是否具有唯一性，只能是boolean值

- `@Access`：标注持久化类，需要改变属性的访问策略时使用。该注解的`value`属性的值有两个：
	- `AccessType.PROPERTY`：通过getter和setter方法访问属性；
	- `AccessType.FIELD`：直接通过成员变量访问属性（不推荐）。

其他特殊的注解：

- `@Proxy`：该注解的`proxyClass`属性指定一个接口，在延迟加载时作为代理使用，也可以在这里指定该类自己的名字。
- `@DynamicInsert`：指定用于插入记录的insert语句是否在运行时动态生成，并且只插入那些非空字段。默认false，开启该属性将导致Hibernate需要更多时间来生成SQL语句。
- `@DynamicUpdate`：指定用于插入记录的update语句是否在运行时动态生成，并且只更新那些改变过的字段。默认false，开启该属性将导致Hibernate需要更多时间来生成SQL语句。打开此注解后，可指定如下几种乐观锁定的策略：
	- `OptimisticLockType.VERSION`：检查version/timestamp字段，强烈建议使用此策略，性能上是最好的选择，也是唯一能够处理在Session外进行脱管操作的策略
	- `OptimisticLockType.ALL`：检查全部字段
	- `OptimisticLockType.DIRTY`：只检查修改过的字段
	- `OptimisticLockType.NONE`：不使用乐观锁定
- `@SeclectBeforeUpdate`：指定Hibernate在更新（update）某个持久化对象之前是否需要先进行一次查询（select）。如果该注解的`value`值设为true，则Hibernate可以保证只有当持久化对象的状态被修改过才会使用update语句来保存其状态。默认值为false。如果程序中某个持久化对象的状态经常会发生改变，那么该属性应该设置为false；如果该持久化对象的状态很少发生改变，而程序又经常要保存该对象，则可将该属性设置为true。
- `@PolymorphismType`：当采用`TABLE_PER_CLASS`继承映射策略时，该注解用于指定是否需要采用隐式多态查询。默认值是`PolymorphismType.IMPLICIT`，即支持隐式多态查询-如果查询时给出的是任何超类，该类实现的接口或者该类的名字，那么会返回该类（及其子类）的实例，如果查询时给出的是子类的名字，则只返回子类的实例。否则，只有在查询时明确给出某个类名时，才会返回这个类的实例。
- `@Where`：该注解的`clause`属性可指定一个附加的SQL语句过滤条件。此时只要试图加载该持久化类的实例时，该条件都会生效。也就是说，只有符合该条件的记录才会被加载。
- `@BatchSize`：当Hibernate抓取集合属性或延迟加载的实体时，该注解的`size`属性指定每批抓取的实例数。
- `@OptimisticLocking`：该注解的`type`属性指定乐观锁定策略。支持4个属性值：`OptimisticLockType.ALL, OptimisticLockType.VERSION, OptimisticLockType.ALL, OptimisticLockType.DIRTY`。默认为`OptimisticLockType.VERSION`。
- `@Check`：该注解可通过constraints指定一个SQL表达式，用于为该持久化类所对应的表指定一个`Check`约束。
- `@Subselect`：用于映射不可变的，只读实体。就是将数据库的子查询映射成Hibernate持久化对象。当需要使用视图（实质是一个查询）来代替数据表时，比较有用。

### 映射属性 ###

默认情况下，被`@Entity`标注的持久化类的所有属性都会被映射到底层数据表。可以使用`@Column`注解标注某个属性，从而指定该属性所对应的数据列的详细信息。该注解支持以下属性：

属性               | 是否必需 | 说明
---                | ----     | ---
`columnDefinition` | 否       | 属性值是一个代表列定义的的SQL字符串（列名后面部分），指定创建该数据列的SQL语句
`insertable`       | 否       | 该列是否包含在Hibernate生成的insert语句的列列表中，默认值：true
`length`           | 否       | 该列所能保存的数据的最大长度，默认值225
`name`             | 否       | 该列的列名。默认列名与`@Column`标注的成员变量名相同
`nullable`         | 否       | 该列是否允许为null，默认值true
`precision`        | 否       | 该列是decimal类型时，所支持的最大有效数字位
`scale`            | 否       | 该列是decimal类型时，所支持的最大小数位数
`table`            | 否       | 该列所属的表名，当需要多个表来保存一个实体时，需要指定该属性
`unique`           | 否       | 该列是否具有唯一约束，默认值false
`updatable`        | 否       | 该列是否包含在Hibernate生成的update语句的列列表中，默认值true

其他为属性映射提供的特殊注解：

- `@Generated`：设置该属性映射的数据列的值是否由数据库生成，可接受以下三个值：
	- `GenerationTime.NEVER`：不由数据库生成
	- `GenerationTime.INSERT`：在执行insert语句时生成，但不会在执行update语句时重新生成
	- `GenerationTime.ALWAY`：在执行insert和update语句时都会被重新生成

- `@Formula`：该注解的`value`属性可指定一个SQL表达式，指定该属性的值将根据表达式来计算。持久化类对应的表中没有和计算属性对应的数据列，因为该属性的值是动态计算出来的，无需保存到数据库。`value`属性的表达式中可以运用`sum, average, max`函数求值的结果，也可以根据另外一个表的查询结果来计算属性值。此外，还需注意以下几点：
	- `value="(sql)"`的英文括号不能少；
	- `value="()"`的括号里面是SQL表达式，其中的列名和表名都应该和数据库表对应，而不是和持久化实例的属性对应；
	- 如果需要在SQL表达式中使用参数，则直接使用类似`where cur.id=currencyID`的形式，其中`currencyID`就是参数，当前持久化实例的`currencyID`属性将作为参数传入。

如果持久化对象有任何属性不是由Java程序提供，而是由数据库生成，包括使用`timestamp`数据类型或触发器来为该列自动插入值等，都可以使用`@Generated`标注持久化类的属性。如果持久化类拥有被`@Generated`标注的属性，则每当Hibernate执行一条insert语句或update语句时，Hibernate会立刻执行一条select语句来获得该数据列的值，并将该值赋给由`@Generated`标注的属性。

#### 使用@Transient标注不想持久保存的属性 ####

#### 使用@Enumerated标注枚举类型的属性 ####

实体类如果有枚举类型的属性，其属性值以及对应的底层数据库中既可以使用枚举值的名字，也可以使用枚举值的序号（从0开始）。这可以通过`@Enumerated`的`value`属性来指定：

- `EnumType.STRING`：使用枚举值的值；
- `EnumType.ORDINAL`：使用枚举值的序号。

#### 使用@Lob，@Basic标注大数据类型的属性 ####

数据库通常采用`Blob`或`Clob`类型的数据列保存大数据，如图片，影音等；而JDBC会采用`java.sql.Blob`和`java.sql.Clob`来表示这些大数据类型的值。

Hibernate使用`@Lob`来标注这种大数据类型，当持久化类的属性为`byte[]`，`Byte[]`或`java.io.Serializable`类型时，标注的属性将映射为底层的`Blob`列；当持久化类的属性为`char[]`，`Character[]`或`java.lang.String`类型时，标注的属性将映射为底层的`Clob`列。

Hibernate加载对象时，并不会立即加载该对象中由`@Basic`标注的属性，而是只加载一个虚拟的代理，等到程序真正需要pic属性时才从底层数据表中加载数据。

`@Basic`注解可以指定以下属性：

- `fetch`：是否延迟加载该属性。该属性可接受`FetchType.EAGER`，`FetchType.LAZY`两个值之一；
- `optinal`：该属性映射的数据列是否允许使用null值。

#### 使用@Temporal标注日期类型的属性 ####

Java程序中，表示日期，时间的类型只有2种：`java.util.Date`和`java.util.Calendar`；但数据库中表示日期，时间的类型比较多：`date`，`time`，`datetime`，`timestamp`等。此时可以使用`@Temporal`来标注这种类型的属性，该注解有一个`value`属性，属性值可以是以下三个值之一：

- `TemporalType.DATE`
- `TemporalType.TIME`
- `TemporalType.TMIESTAMP`

### 映射主键 ###

不推荐使用具有实际意义的字段组合而成的物理主键；应该使用没有任何实际意义的逻辑主键。即设置一个单独的数据列作为记录的唯一标识。Hibernate为这种逻辑主键提供了主键生成器，负责为每个持久化实例生成唯一的逻辑主键值。

如果实体类的标识属性（映射成主键列）是基本数据类型，基本类型的包装类，`String`，`Date`等类型，可以简单地使用`@Id`标注该实体属性即可。使用`@Id`注解时无须指定任何属性。Hibernate可以使用`@GeneratedValue`来标注实体的标识属性，从而为逻辑主键自动生成主键值。该注解有如下属性：

属性      | 是否必需 | 说明
---       | ----     | ---
strategy  | 否       | 指定Hibernate生成主键的策略。
generator | 否       | 当使用`GenerationType.SEQUENCE`，`GenerationType.TABLE`主键策略时，该属性引用`@SequenceGenerator`，`@TableGenerator`所定义的生成器的名称。

`strategy`的属性值可以是以下值之一：

- `GenerationType.AUTO`：默认值，Hibernate自动选择最合适底层数据库的主键生成策略。
- `GenerationType.IDENTITY`：对于MySQL，SQL Server这样的数据库，选择自增长的主键生成策略。
- `GenerationType.SEQUENCE`：对于Oracle这样的数据库，选择使用基于`Sequence`的主键生成策略，应与`@SequenceGenerator`一起使用。
- `GenerationType.TABLE`：使用辅助表来生成主键，应与`@TableGenerator`一起使用。

`@SequenceGenerator`注解支持以下属性：

属性             | 是否必需 | 说明
---              | ----     | ---
`name`           | 是       | 主键生成器的名称
`allocationSize` | 否       | 底层Sequence每次生成主键值的个数。对于Oracle，该属性指定的整数值将作为定义Sequence时`increment by`的值
`catalog`        | 否       | 将底层Sequence放入指定catalog中，如果不指定该属性，该Sequence将放入默认的catalog中
`schema`         | 否       | 将底层Sequence放入指定schema中，如果不指定该属性，该Sequence将放入默认的schema中
`initialValue`   | 否       | 底层Sequence的初始值。对于Oracle，该属性指定的整数值将作为定义Sequence时`start with`的值
`sequenceName`   | 否       | 底层Sequence的名称

`@TableGenerator`注解支持以下属性：

属性                | 是否必需 | 说明
---                 | ----     | ---
`name`              | 是       | 主键生成器的名称
`allocationSize`    | 否       | 底层辅助表每次生成主键值的个数
`catalog`           | 否       | 将辅助表放入指定catalog中，如果不指定该属性，该辅助表将放入默认的catalog中
`schema`            | 否       | 将底层辅助表放入指定schema中，如果不指定该属性，该辅助表将放入默认的schema中
`table`             | 否       | 辅助表的表名
`initialValue`      | 否       | 该属性指定的整数值将作为辅助表的初始值，默认值为0
`pkColumnName`      | 否       | 指定存放主键名的列名
`pkColumnValue`     | 否       | 指定主键名
`valueColumnName`   | 否       | 指定存放主键值的列名
`indexes`           | 否       | 属性值是一个`@Index`数组，用于为辅助表定义索引
`uniqueConstraints` | 否       | 属性值是一个`@UniqueConstraint`数组，用于为辅助表创建唯一约束

### 使用Hibernate的主键生成器 ###

JPA标准只支持`AUTO`，`IDENTITY`，`SEQUENCE`，和`TABLE`这4种主键生成策略。但Hibernate支持更多的策略，这些Hibernate特有的策略需要使用Hibernate提供的`@GenericGenerator`注解。该注解支持以下2个属性：

属性       | 是否必需 | 说明
---        | ----     | ---
`name`     | 是       | 主键生成器的名称
`strategy` | 否       | 生成器使用的主键生成策略

`strategy`属性的值即可以是Hibernate自带的主键生成类，也可以是自定义的主键生成类（只需实现Hibernate的`IdentifierGenerator`接口）。

Hibernate自带的主键生成类：

- `IncrementGenerator`：为`long`，`short`或者`int`类型的主键生成唯一标识。只有在没有其他进程往同一个表中插入数据时才能使用。在集群下不要使用！
- `IdentityGenerator`：在DB2，MySQL，Microsoft SQL Server，Sybase和HypersonicSQL等提供自增长主键支持的数据表中使用。返回的标识值是`long`，`short`或者`int`类型的。
- `SequenceStyleGenerator`：在DB2，Oracle，PostgreSQL，SAP DB，McKoi等提供Sequence支持的数据表中使用。返回的标识值是`long`，`short`或者`int`类型的。
- `MultipleHiloPerTableGenerator`：使用一个高低位算法高效地生成`long`，`short`或者`int`类型的标识符。给定一个表和字段（默认分别为hibernate_sequence和next_hi）作为高位值的来源。高低位算法生成的标识值只在一个特定的数据库中是唯一的。
- `UUIDGenerator`：用一个128位的UUID算法生成字符串类型的标识符，这在网络是唯一的（IP地址也作为算法的数据源）。UUID被编码为一个32位的十六进制的字符串。
- `GUIDGenerator`：在Microsoft SQL Server中使用数据库生成的GUID字符串。
- `SelectGenerator`：通过数据库触发器选择某个唯一主键的行，并返回其主键值作为标识属性值。
- `ForeignGenerator`：直接使用另一个关联的对象的标识属性值（即本持久化对象不能生成主键）。只在基于主键的1-1关联映射中才有用。

如果准备只使用`IdentityGenerator`或`SequenceStyleGenerator`主键生成器，则应该直接使用标准的`@GeneratedValue`注解即可。采用`@GenericGenerator`都是需要使用Hibernate特定的生成策略实现类。

### 映射集合属性 ###

集合属性大致有两种：

- 单纯的集合属性，如`List`，`Set`或数组等集合属性；
- Map结构的集合属性，每个属性值都有对应的key映射。

Hibernate要求持久化集合值字段必须声明为接口。

集合类实例具有值类型的行为。当持久化对象被保存时，这些集合属性会被自动持久化；当持久化对象被删除时，这些集合属性对应的记录也会被自动删除。假设集合元素被从一个持久化对象传递到另一个持久化对象，该集合元素对应的记录会从一个表转移到另一个表。
两个持久化对象不能共享同一个集合元素的引用。

统一使用`@ElementCollection`注解进行映射。该注解具有以下属性：

属性          | 是否必需 | 说明
---           | ----     | ---
`fetch`       | 否       | 该实体对集合属性的抓取策略（当程序初始化该实体时，是否立即从数据库中抓取该实体的集合属性中的所有元素）。该属性支持`FetchType.EAGER`和`FetchType.LAZY`（默认值）。
`targetClass` | 否       | 指定集合属性中集合元素的类型。

因为集合属性总需要保存到另一个数据表中，所以保存集合属性的数据表必须包含一个外键列，用于参照到主键列，该外键列使用`@JoinColumn`进行映射。

Hibernate使用标准的`@CollectionTable`映射保存集合属性的表。该注解支持以下属性：

属性                | 是否必需 | 说明
---                 | ----     | ---
`name`              | 否       | 保存集合属性的数据表的表名
`catalog`           | 否       | 将保存集合属性的数据表放入指定catalog中，如果不指定该属性，该辅助表将放入默认的catalog中
`schema`            | 否       | 将保存集合属性的数据表放入指定schema中，如果不指定该属性，该辅助表将放入默认的schema中
`indexes`           | 否       | 为持久化类所映射的表设置索引，该属性值是一个`@Index`注解数组
`joinColumns`       | 否       | 该属性值为`@JoinColumn`数组，每个`@JoinColumn`映射一个外键列（通常只需要一个外键列，但如果主实体采用了复合主键，保存集合属性的表就需要定义更多个外键列）
`uniqueConstraints` | 否       | 为持久化类所映射的表设置唯一约束。该属性值是一个`@UniqueConstraint`注解数组

`@JoinColumn`注解专门用于定义外键列。该注解具有以下属性：

属性                   | 是否必需 | 说明
---                    | ----     | ---
`name`                 | 否       | 外键列的列名
`columnDefinition`     | 否       | Hibernate使用该属性值指定的SQL片段来创建外键列
`insertable`           | 否       | 该列是否包含在Hibernate生成的insert语句的列列表中。默认值：true
`updatable`            | 否       | 该列是否包含在Hibernate生成的update语句的列列表中。默认值：true
`nullable`             | 否       | 该列是否允许为null，默认值：true
`table`                | 否       | 该列所在数据表的表名
`unique`               | 否       | 是否为该列添加唯一约束
`referencedColumnName` | 否       | 该列所参照的主键列的列名

当集合元素是基本数据类型，字符串类型，日期类型或其他复合类型时，因为这些集合元素都是从属于持久化对象，而且这些数据类型在数据表中只需要一列就可以保存，因此使用`@Column`定义集合元素列即可，并且应该将映射外键列的`@JoinColumn`的`nullable`属性设置为false；对于一对多的关联，由于外键列所在的数据表也对应于独立的持久化对象，因此`@JoinColumn`的`nullable`属性既可以设置为true，也可以设置为false。

如果要映射带索引的集合（List，数组，Map），需要为集合元素所在的数据表指定一个索引列-用于保存这些数据类型的索引。用于映射索引列的注解有两个：

- `@OrderColumn`：定义List，数组的索引列；
- `@MapKeyColumn`：映射Map集合的索引列。

这两个注解的属性参考普通`@Column`注解的属性。

程序可使用`@MapKeyClass`显式指定Map key的类型。该注解只有一个`value`属性，用于指定Map key的类型。

总体而言，集合元素的类型大致可分为以下几种情况：

- 集合元素是基本类型及其包装类，字符串类型和日期类型：此时使用`@ElementCollection`映射集合属性，并使用普通的`@Column`映射集合元素对应的列；
- 集合元素是组件（非持久化实体的复合类型）：此时使用`@ElementCollection`映射集合属性，然后使用`@Embeddable`标注非持久化实体的复合类；
- 集合元素是关联的持久化实体：此时已不再是集合属性了，应使用`@OneToMany`或`@ManyToMany`进行关联映射。

#### List集合属性 ####

`List`和`Set`集合类型的属性必须显式地初始化，否则程序运行时会抛出`NullPointerException`异常。

集合属性可以使用泛型来限制集合元素的类型，这样可以无须指定`@ElementCollection`注解的`targetClass`属性。但仍然推荐显式设置`targetClass`属性，从而减轻Hibernate的识别负担。`List`集合和数组类型的属性可以用关联持久化对象的外键和集合索引列作为联合主键。

#### 数组属性 ####

Hibernate对`List`和数组的处理几乎完全一样。

#### Set集合属性 ####

与前两种集合属性不同，`Set`集合是无序，不可重复的集合，所以无须使用`@OrderColumn`注解映射索引列。但同样需要`@ElementCollection`和`@CollectionTable`来分别映射集合属性和保存集合元素的表，如果集合元素是基本类型及其包装类，`String`，`Date`等类型，也可使用`@Column`映射保存集合元素的数据列。

`List`集合和数组的元素有索引，总是用关联持久化类的外键列和集合元素索引列作为联合主键；而`Set`集合的元素没有索引，因此没有索引列，如果元素列不为空（`@Column`注解的`nullable=false`），则以关联持久化类的外键列和元素列作为联合主键，否则该表没有主键。

#### Map集合属性 ####

需要使用`@MapKeyColumn`映射保存Map key的数据列，使用`@MapKeyClass`指定Map key的类型，Map value的类型由`@ElementCollection`注解的`targetClass`属性值指定。

Hibernate将以外键列和key列作为联合主键。

同样推荐显式指定Map key和Map value的类型，从而减轻Hibernate自动识别的负担。

### 集合属性的性能分析 ###

- 有序集合：拥有一个有外键列和集合元素索引列组成的联合主键，集合里的元素可以通过联合主键快速定位，因此CRUD操作有较好的性能（数组类型除外，因为数组长度不可变，无法使用延迟加载）；
- 无序集合：集合里的元素只能遍历。

如果仅仅是对`Set`集合中的已有元素进行了修改，则Hibernate不会立即更新（update）该元素所对应的数据行。只有在执行插入（insert）和删除（delete）操作时，这些更改才生效。

Hibernate中的关联映射都是采用`Set`集合来管理。在设计良好的Hibernate模型中，1-N关联的1的一端通常不控制关联关系，所有的更新操作都放在N的一端进行处理，从而避免考虑集合的更新性能。此时该集合的映射作为反向集合使用，`List`集合属性将有较好的性能，因为可以在为初始化集合元素的情况下直接向`List`集合中添加新元素（`add()`和`addAll()`方法总是返回true）。`Set`集合则需要先加载所有的元素，并和新元素比较以确定没有重复，因此添加新元素的性能较低。

当需要删除集合元素时，如果是删除全部元素，Hibernate会执行一个delete语句来实现；但如果只是删除部分元素，Hibernate则需要一个一个元素的删除。此时可以考虑从持久化实例中取出集合，然后清空集合属性值，基于取出的集合进行删减操作（此时的集合与数据库已脱钩，不会对数据库进行任何操作），然后将修改后的集合重新赋值给持久化类的集合属性。

集合属性表里的记录完全从属于主表的实体，当主表的记录被删除时，集合属性表里从属于该记录的数据将会被删除；Hibernate无法直接加载，查询集合数据表中的记录，只能先加载主表实体，再通过主表实体去获取集合属性表中对应的记录。

### 有序集合映射 ###

Hibernate还支持使用`SortedSet`和`SortedMap`两个有序集合，需要使用Hibernate本身提供的`@SortNatural`（自然排序）和`@SortComparator`（使用`value`属性指定`Comparator`实现类从而实现定制排序）注解。

Hibernate可以使用`@OrderBy`注解来实现在查询时对集合元素排序。此时排序会在SQL查询中完成，而不是在内存中完成。该注解的`value`属性指定了一个SQL排序子句，即通过持久化实体获取集合属性时所生成的SQL语句总会添加这个SQL排序子句（`order by 子句`）。

### 映射数据库对象 ###

映射数据库对象无法用注解来实现，只能通过Hibernate传统的`*.hbm.xml`映射文件的方式来实现。

`<database-object.../>`元素在映射文件中创建和删除触发器，存储过程等数据库对象。有以下两种形式：

- 在映射文件中显式声明`create`和`drop`命令。
	
		<hibernate-mapping>
			...
			<database-object>
				<create>create trigger t_full_content_gen ...</create>
				<drop>create trigger t_full_content_gen ...</drop>
			</database-object>
		</hibernate-mapping>

每个`<database-object.../>`元素中只有一组`<create.../>`，`<drop.../>`对

- 提供一个类，这个类必须实现`org.hibernate.mapping.AuxiliaryDatabaseObject`接口，负责组织`create`和`drop`命令。

		<hibernate-mapping>
			...
			<database-object>
				<definition class="MyTriggerDefinition"/>
			</database-object>
		</hibernate-mapping>

在`<database-object.../>`元素中使用`<dialect-scope.../>`子元素来指定某些数据库对象仅在特定的方言中才能使用。

		<hibernate-mapping>
			...
			<database-object>
				<create>create trigger t_full_content_gen ...</create>
				<drop>create trigger t_full_content_gen ...</drop>
				<!-- 指定仅对MySQL数据库有效 -->
				<dialect-scope name="org.hibernate.dialect.MySQL5Dialect"/>
				<dialect-scope name="org.hibernate.dialect.MySQL5InnoDBDialect"/>
			</database-object>
		</hibernate-mapping>

所有能在`java.sql.Statement.execute()`方法中执行的SQL语句都能在`create`和`drop`中使用。

为了让Hibernate根据`<database-object.../>`元素创建数据表，一定要将Hibernate配置文件里的`hbm2ddl.auto`属性值修改成`create`。Hibernate提供的`SchemaExport`工具可以根据映射文件来生成数据库对象，则无须将`hbm2ddl.auto`属性值修改成`create`，使用`update`即可。

## 映射组件属性 ##

组件属性：持久化类的属性不是基本数据类型，也不是字符串，日期等标量类型的变量，而是一个复合类型的对象，在持久化过程中，它仅仅被作为值类型，而并非引用另一个持久化实体。

作为组件属性使用的POJO使用`@Embeddable`注解，该注解与`@Entity`类似。区别是：

- `@Entity`：持久化类
- `@Embeddable`：持久化类的组件类。

`@Embeddable`标注的类同样可以使用`@Column`来指定这些属性对应的数据列。该类内可以拥有一个指向该组件所属实体的属性，这个属性使用`@Parent`标注。

只包含普通标量属性的组件类型，组件里的每个属性映射组件所属持久化类对应的数据表中的一个数据列即可，组件类本身在数据库中并没有单独对应的数据表，因此无法让其他实体类与其建立关联关系，换句话说，实体类和组件类之间的关联只能是单向关联，也只能在组件类中控制。

Hibernate为组件映射提供了另一种映射策略：无须在组件类上使用`@Embeddable`，而是直接在持久化类中使用`@Embedded`标注组件属性。此时可以使用`@AttributeOverrides`和`@AttributeOverride`，每个`@AttributeOverride`指定一个属性的映射配置，支持以下属性。

属性     | 是否必需 | 说明
---      | ----     | ---
`name`   | 是       | 对组件类的哪个属性进行配置
`column` | 是       | 属性所映射的数据列的列名

### 组件属性为集合 ###

如果组件类包括了`List`，`Set`，`Map`等集合属性，则可直接在组件类中使用`@ElementCollection`标注集合属性，并使用`@CollectionTable`指定保存集合属性的数据表-与普通实体类中映射集合属性的方式基本相同。

### 集合属性的元素为组件 ###

元素为组件类型的集合属性仍然使用`@ElementCollection`，`@CollectionTable`，`@OrderColumn`，`@MapKeyColumn`标注。
与普通集合属性的区别是不再使用`@Column`映射保存集合元素（组件类型）的数据列（Hibernate无法使用单独的数据列保存集合元素），程序只要使用`@Embeddable`修饰组件类即可。

### 组件作为Map的索引 ###

与普通Map集合属性的使用一样，使用`@MapKeyClass`指定Map key的类型。只是要重写这个组件类的`equals()`和`hashCode()`两个方法。此外，还应该使用`@Embeddable`标注该组件类。组件类的属性被映射到集合属性表里的数据列，而Hibernate会将外键列，组件类属性所映射的多列作为联合主键。

### 组件作为复合主键 ###

使用组件作为复合主键，也就是使用组件作为持久化类的标识符，该组件类必须满足以下要求：

- 有无参数的构造器
- 必须实现`java.io.Serializable`接口
- 正确重写`equals()`和`hashCode()`方法

当使用组件作为复合主键时，Hibernate无法为这种复合主键自动生成主键值，所以程序必须为持久化实例分配这种组件标识符，即为持久化类的实例的主键属性显式赋值。当持久化类使用组件作为复合主键时，程序需要使用`@EmbeddedId`来标注该主键。该注解和`@Embedded`的区别在于：

- `@Embedded`：标注普通的组件属性；
- `@EmbeddedId`：标注组件类型的主键。

### 多列作为联合主键 ###

持久化类的多个属性可以映射成联合主键（直接使用多个`@Id`标注这些属性），该持久化类必须满足如下条件：

- 有无参数的构造器；
- 必须实现`java.io.Serializable`接口
- 根据联合主键所映射的属性重写`equals()`和`hashCode()`方法

## 使用传统的映射文件 ##

采用XML映射文件来管理实体类的映射关系：

`PO = POJO + 映射文件`

映射文件名为：`*.hbm.xml`。映射文件的根元素是`<hibernate-mapping.../>`，该元素下有`<class.../>`子元素，每个子元素映射一个持久化类。`<class.../>`元素中的`<composite-id.../>`元素用于映射复合主键，`<property.../>`元素用于映射普通属性，`<component.../>`元素用于映射组件属性。

在Hibernate的配置文件中使用如下配置持久化类：

		<!-- 使用注解配置 -->
		<mapping class="net.xiaoluo.practice.domain.Person"/>
		<!-- 使用传统的XML配置文件 -->
		<mapping resource="net/xiaoluo/practice/domain/Person.hbm.xml"/>

## Hibernate的关联映射 ##

关联关系大致有如下两个分类：

- 单向关系：只需单向访问关联端；
	- 单向1-1
	- 单向1-N
	- 单向N-1
	- 单向N-N
- 双向关系：关联的两端可以互相访问。
	- 双向1-1
	- 双向1-N
	- 双向N-N

### 单向N-1关联 ###

单向的N-1关联只需从N的一端可以访问1的一端，因此程序应该在N的一端的持久化类中增加一个属性，该属性引用1的一端的关联实体。

N-1关联（不管是单向关联，还是双向关联），都需要在N的一端使用`@ManyToOne`标注代表关联实体的属性。该注解可以指定以下属性：

属性           | 是否必需 | 说明
---            | ----     | ---
`cascade`      | 否       | Hibernate对关联实体采用的级联策略：<br/> - `CascadeType.ALL`：将所有的持久化操作都级联到关联实体 <br/> - `CascadeType.MERGE`：将`merge`操作级联到关联实体 <br/> - `CascadeType.PERSIST`：将`persist`操作级联到关联实体 <br/> - `CascadeType.REFRESH`：将`refresh`操作级联到关联实体 <br/> - `CascadeType.REMOVE`：将`remove`操作级联到关联实体
`fetch`        | 否       | 抓取关联实体时的抓取策略：<br/> - `FetchType.EAGER`：默认值，立即抓取关联实体；<br/> - `FetchType.LAZY`：延迟抓取关联实体，等真正用到关联实体时才去抓取
`optional`     | 否       | 关联关系是否可选
`targetEntity` | 否       | 关联实体的类名。默认情况下，Hibernate将通过反射来判断关联实体的类名

如果用于表示关联实体的集合不带泛型信息，就必须指定`targetEntity`属性。

#### 无连接表的N-1关联 ####

无连接表的N-1关联只要在N的一端增加一列外键，让外键值记录该对象所属的实体即可，Hibernate可使用`@JoinColumn`来标注代表关联实体的属性，用于映射底层的外键列。

当需要在N端的表中插入一条记录时，如果与该记录关联的1端的实体还没有持久化，可能发生以下三种情况：

- 系统抛出`TransientPropertyValueException`异常：`Not-null property references a transient value`，因为1端记录还没有插入，导致N端的记录也无法插入；
- 系统先自动级联插入1端表的记录，在插入N端表的记录；
- 如果在N端使用`@JoinColumn`标注的代表关联实体的属性映射的外键列允许为null（通过`nullable`属性设置），Hibernate会先插入一条外键列值为null的记录，当该实体关联的1端的实体被持久化后，Hibernate使用一条额外的update语句来修改N端所对应记录的外键值，从而建立1端持久类实体和N端持久类实体之间的关联。与前面相比，需要多执行一条update语句。

因此，在所有基于外键约束的关联关系中，要么总是先持久化1端表记录对应的实体，要么设置级联操作；否则当Hibernate试图插入N端表记录时，如果发现N端表记录关联的1端表记录不存在，就会引发异常，或者先插入一条外键值为null的记录，在稍后条件满足时再多执行一条update语句修改该外键列的值。

#### 有连接表的N-1关联 ####

对于使用连接表的单向N-1关联，程序需要显式使用`@JoinTable`来映射连接表。该注解支持以下属性：

属性                 | 是否必需 | 说明
---                  | ----     | ---
`name`               | 否       | 连接表的表名
`catalog`            | 否       | 将连接表放入指定catalog中，如果不指定该属性，该连接表将放入默认的catalog中
`schema`             | 否       | 将连接表放入指定schema中，如果不指定该属性，该连接表将放入默认的schema中
`targetEntity`       | 否       | 指定关联实体的类名，默认情况下，Hibernate通过反射来判断关联实体的类名
`indexes`            | 否       | 为该连接表设置索引，该属性值是一个`@Index`注解数组
`joinColumns`        | 否       | 该属性值为`@JoinColumn`数组，每个`@JoinColumn`映射一个外键列，这些外键列参照当前实体对应表的主键列，因为是映射N-1关联的1端，所以需要为`@JoinColumn`增加`unique=true`。
`inverseJoinColumns` | 否       | 该属性值为`@JoinColumn`数组，每个`@JoinColumn`映射一个外键列，这些外键列参照当前实体的关联实体对应表的主键列
`uniqueConstraints`  | 否       | 为连接表增加唯一约束

使用连接表的N-1关联的两个实体对应的数据表都无须增加外键列，两个实体对应的数据表不存在主，从关系，因此程序持久化实体的顺序并不重要，不会像无连接表的N-1关联引起问题。

### 单向1-1关联 ###

单向1-1关联需要在持久化类里增加代表关联实体的成员变量，并为该成员变量增加setter和getter方法。从持久化类的代码上看，单向1-1与单向N-1没有丝毫区别。对于1-1关联（不管是单向关联，还是双向关联），都需要使用`@OneToOne`标注代表关联实体的属性。该注解可以指定以下属性：

属性            | 是否必需 | 说明
---             | ----     | ---
`cascade`       | 否       | Hibernate对关联实体采用的级联策略：<br/> - `CascadeType.ALL`：将所有的持久化操作都级联到关联实体 <br/> - `CascadeType.MERGE`：将`merge`操作级联到关联实体 <br/> - `CascadeType.PERSIST`：将`persist`操作级联到关联实体 <br/> - `CascadeType.REFRESH`：将`refresh`操作级联到关联实体 <br/> - `CascadeType.REMOVE`：将`remove`操作级联到关联实体
`fetch`         | 否       | 抓取关联实体时的抓取策略：<br/> - `FetchType.EAGER`：默认值，立即抓取关联实体；<br/> - `FetchType.LAZY`：延迟抓取关联实体，等真正用到关联实体时才去抓取
`mappedBy`      | 否       | 合法的属性值为关联实体的属性名，指定关联实体中哪个属性可引用到当前实体
`orphanRemoval` | 否       | 是否删除孤儿实体-即实体所关联的的父实体不存在（即该实体对应记录的外键为null）。
`optional`      | 否       | 关联关系是否可选
`targetEntity`  | 否       | 关联实体的类名。默认情况下，Hibernate将通过反射来判断关联实体的类名

#### 基于外键的单向1-1关联 ####

基于外键的1-1关联需要先使用`@OneToOne`标注代表关联实体的属性，再使用`@JoinColumn`映射外键列，此外，还必须为`@JoinColumn`增加`unique=true`。

很少会使用有连接表的单向1-1关联。

### 单向1-N关联 ###

单向1-N关联中只需要在1的一端增加`Set`类型的成员变量，该成员变量记录当前实体所有的关联实体，还需要为这个`Set`类型的属性增加setter和getter方法。这与前面的集合属性非常相似，只是此时集合里的元素是关联实体。Hibernate使用`@OneToMany`映射1-N关联。该注解支持以下属性：


属性            | 是否必需 | 说明
---             | ----     | ---
`cascade`       | 否       | Hibernate对关联实体采用的级联策略：<br/> - `CascadeType.ALL`：将所有的持久化操作都级联到关联实体 <br/> - `CascadeType.MERGE`：将`merge`操作级联到关联实体 <br/> - `CascadeType.PERSIST`：将`persist`操作级联到关联实体 <br/> - `CascadeType.REFRESH`：将`refresh`操作级联到关联实体 <br/> - `CascadeType.REMOVE`：将`remove`操作级联到关联实体
`fetch`         | 否       | 抓取关联实体时的抓取策略：<br/> - `FetchType.EAGER`：默认值，立即抓取关联实体；<br/> - `FetchType.LAZY`：延迟抓取关联实体，等真正用到关联实体时才去抓取
`mappedBy`      | 否       | 合法的属性值为关联实体的属性名，指定关联实体中哪个属性可引用到当前实体
`orphanRemoval` | 否       | 是否删除孤儿实体-即实体所关联的的父实体不存在（即该实体对应记录的外键为null）。
`targetEntity`  | 否       | 关联实体的类名。默认情况下，Hibernate将通过反射来判断关联实体的类名

#### 无连接表的单向1-N关联 ####

无连接表的1-N单向关联使用`@JoinColumn`在1的一端控制关联关系，该注解映射外键列，这里的外键列并没有增加到当前实体对应的数据表中，而是增加到关联实体对应的数据表中。

应尽量使用双向1-N关联代替单向1-N关联，并在N的一端控制关联关系。

#### 有连接表的单向1-N关联 ####

有连接表的1-N关联同样需要使用`@OneToMany`标注代表关联实体的集合属性，此外，还应使用`@JoinTable`显式指定连接表。
与有连接表的N-1关联类似，实体的持久化顺序不影响系统的性能。

### 单向N-N关联 ###

单向N-N关联和1-N关联的持久化类代码完全相同，但需要使用`@ManyToMany`标注代表关联实体的集合属性。该注解支持的属性如下：

属性            | 是否必需 | 说明
---             | ----     | ---
`cascade`       | 否       | Hibernate对关联实体采用的级联策略：<br/> - `CascadeType.ALL`：将所有的持久化操作都级联到关联实体 <br/> - `CascadeType.MERGE`：将`merge`操作级联到关联实体 <br/> - `CascadeType.PERSIST`：将`persist`操作级联到关联实体 <br/> - `CascadeType.REFRESH`：将`refresh`操作级联到关联实体 <br/> - `CascadeType.REMOVE`：将`remove`操作级联到关联实体
`fetch`         | 否       | 抓取关联实体时的抓取策略：<br/> - `FetchType.EAGER`：默认值，立即抓取关联实体；<br/> - `FetchType.LAZY`：延迟抓取关联实体，等真正用到关联实体时才去抓取
`mappedBy`      | 否       | 合法的属性值为关联实体的属性名，指定关联实体中哪个属性可引用到当前实体
`targetEntity`  | 否       | 关联实体的类名。默认情况下，Hibernate将通过反射来判断关联实体的类名

N-N关联必须使用连接表。使用`@JoinTable`映射连接表，但必须去掉`@JoinTable`的`inverseJoinColumns`属性所指定的`@JoinColumn`中的`unique=true`。

### 双向1-N关联 ###

Hibernate推荐使用双向关联处理1-N关联，而且使用N的一端，而不是1的一端控制关联关系。两端都需要增加对关联属性的访问，N的一端增加引用到关联实体的属性，1的一端增加集合属性，集合元素为关联实体。

#### 无连接表的双向1-N关联 ####

无连接表的双向1-N关联，在N的一端需要增加`@ManyToOne`标注代表关联实体的属性，而1的一端则需要使用`@OneToMany`标注代表关联实体的属性。

底层数据库为了记录1-N关联关系，实际上只需要在N的一端的数据表里增加一个外键列即可，因此应该在使用`@ManyToOne`的同时，使用`@JoinColumn`来映射外键列。

因为通常需要在N的一端控制关联关系，所以应该在使用`@OneToMany`指定`mappedBy`属性。一旦为`@OneToMany`，`@ManyToMany`指定了该属性，则表明当前实体不能控制关联关系。而当`@OneToMany`，`@ManyToMany`，`@OneToOne`所在的当前实体放弃控制关联关系之后，Hibernate就不允许使用`@JoinColumn`或`@JoinTable`标注代表关联实体的属性了。对于指定了`mappedBy`属性的`@OneToMany`，`@ManyToMany`，`@OneToMany`注解，都不能与`@JoinColumn`或`@JoinTable`同时标注代表关联实体的属性。

控制关联关系和主从表是2个不同的概念：

- 控制关联关系：通过在PO类中使用`@JoinColumn`或`@JoinTable`来控制关联关系；
- 主从表：
	- 主表：1的一端对应的数据表，不具备外键列；
	- 从表：N的一端对应的数据表，具备外键列；

通过连接表管理关联关系的2个表没有主，从表之分。

主程序需要注意如下几点：

- 最好先持久化1端的对象，或者该对象本身已处于持久化状态，因此程序希望持久化N端的对象时，Hibernate可为该对象的外键属性分配值。
- 先设置1端和N端的关联关系，再保存持久化N端对象。
- 不要通过1端对象设置关联关系，因为在此持久化类中的`@OneToMany`中指定了`mappedBy`属性，也表明该持久化实体不能控制关联关系。

#### 有连接表的双向1-N关联 ####

有连接表的双向1-N关联可以让1的一端无须任何改变，此时1的一端仍然不控制关联关系，只需要在N的一端使用`@JoinTable`显式指定连接表即可。

有连接表的双向1-N关联可以使用1的一端控制关联关系，不会影响系统性能。此时需要删除`@OneToMany`的`mappedBy`属性，并要求两端的`@JoinTable`的`name`属性相同（表示使用同一个连接表），负责指定外键列的属性也相互对应：两端的`@JoinTable`中的`joinColumns`和`inverseJoinColumns`的值是互反的。（a.joinColumns=b.inverseJoinColumns， a.inverseJoinColumns=b.joinColumns）。

### 双向N-N关联 ###

双向N-N关联只能使用连接表建立两个实体之间的关联关系。在两端的实体中分别使用`@ManyToMany`标注`Set`集合属性，并都使用`@JoinTable`显式映射连接表，连接表的表名相同，所指定的外键列也相互对应。可以在`@ManyToMany`中使用`mappedBy`属性来放弃此端对关联关系的控制，如此以来，这一端也就不需要再使用`@JoinTable`来映射连接表了。

### 双向1-1关联 ###

双向1-1关联需要修改两端的持久化类代码，让两个持久化类都增加引用关联实体的属性，并为属性提供setter和getter方法。

#### 基于外键的双向1-1关联 ####

两端都需要使用`@OneToOne`进行映射。外键可放置在任何一端，在该端使用`@JoinColumn`映射外键列，还应该为该注解增加`unique=true`表示该实体是1的一端。此时，增加了外键的实体将作为从表，而另一个表则成为主表。
作为主表的实体不应该用于控制关联关系，所以主表所对应的实体中使用`@OneToOne`时，应增加`mappedBy`属性。

有连接表的双向1-1关联非常罕见。

### 包含关联实体的组件属性 ###

如果组件的属性是关联实体，则可以使用`@OneToOne`，`@OneToMany`，`@ManyToOne`，或者`@ManyToMany`标注代表关联实体的属性。  
如果程序采用基于外键的映射策略，还需要配合在从表（N端）的一端使用`@JoinColumn`映射外键列；  
如果程序采用基于连接表的映射策略，还需要配合在N的一端使用`@JoinTable`映射连接表。

一般来说，如果需要在组件类中和其他持久化实体建立关联关系，程序应该将该组件映射成持久化实体，而不是组件类，这样就可以将单向1-N关联改写为双向1-N关联，从而提供较好的性能。  
因为，组件类在数据库中没有属于自己的单独的数据表，它关联的（不是被关联）的持久化实体只能使用该组件所属实体的主键列作为外键列，关联的逻辑出现了混乱。  
举例：持久化实体A中关联了组件类B，而B中又关联了持久化实体C，因为B没有单独的数据表，所以C无法直接映射与B的关联，必须通过使用A的主键作为C的外键列来映射与B的关联（通过A的主键来定位B的记录）。

### 基于复合主键的关联关系 ###

在1的一端使用复合主键，不需要特殊的改变；N的一端使用`@JoinColumns`组合多个`@JoinColumn`来将主表中的复合（多个）主键映射为多个外键列。

### 复合主键中某个属性是关联实体 ###

如果持久化实体A的复合主键中的某个属性的类型是持久化实体B，则A会将实体B的主键列映射为自己的外键列，并将此外键列作为复合主键列之一。

### 持久化的级联 ###

从数据库建模的角度，两个表之间的1-N关联关系总是用外键约束来表示，其中保留外键的数据表称为从表，被从表参照的数据表称为主表。Hibernate使用2种策略来映射这种关系：

- 将从表记录映射成持久化类的组件，也就是集合属性的元素类型是组件，这些组件的生命周期总是依赖于父对象。  
Hibernate默认启用级联操作，父对象的保存和删除会导致组件也被保存和删除；
- 将从表记录也映射为持久化实体，这样两者就发生了关联。  
从表实体拥有自己的生命周期，允许其他实体共享对它的引用；  
Hibernate默认不启用实体到其他关联实体之间的级联操作，父对象的保存和删除不会导致关联的实体也被保存和删除；

可以通过为`@OneToOne`，`@OneToMany`，`@ManyToOne`，`@ManyToMany`的`cascade`属性指定以下值来启用相应的级联策略：

- `CascadeType.ALL`：将所有的持久化操作都级联到关联实体  
- `CascadeType.MERGE`：将`merge`操作级联到关联实体  
- `CascadeType.PERSIST`：将`persist`操作级联到关联实体  
- `CascadeType.REFRESH`：将`refresh`操作级联到关联实体  
- `CascadeType.REMOVE`：将`remove`操作级联到关联实体

此外，可在`@OneToOne`和`@OneToMany`中通过`orphanRemoval`属性来启用删除孤儿记录。从注解名可看出，该策略只能当实体处于1的一端，且底层数据表为主表的时候有效。其作用是当程序通过主表实体切断了与从表实体的关联关系时，虽然此时主表实体对应的记录并没有删除，但由于从表实体失去了对主表实体的引用，变为孤儿记录，Hibernate会自动删除这些记录。

Hibernate对设定级联策略的建议：

- 通常不要在`@ManyToOne`中指定级联策略。级联通常在`@OneToOne`，`@OneToMany`中比较有用。因为级联策略通常应该是由主表记录传播到从表记录，而不是从从表记录到主表记录。
- 如果从表记录被完全限制在主表记录内（当主表记录被删除，从表记录就失去存在意义），则可以指定`cascade=CascadeType.ALL`和`orphanRemoval=true`级联策略，将从表实体的生命周期完全交给主表实体管理。
- 如果经常在某个事务中同时使用主表实体和从表实体，则可以考虑指定`cascade={CascadeType.PERSIST, CascadeType.MERGE}`级联策略。

`CascadeType.ALL`策略的详细解释：

- 如果主表实体被`persist()`，那么关联的从表实体也会被`persist()`；
- 如果主表实体被`merge()`，那么关联的从表实体也会被`merge()`；
- 如果主表实体被`save(), update(), saveOrUpdate()`，那么关联的从表实体也会被`saveOrUpdate()`；
- 如果把持久化状态下的主表实体与处于瞬态或脱管状态的从表实体相关联，则从表实体会被自动持久化，进入持久化状态；
- 如果主表实体被删除，那么关联的从表实体也会被删除；
- 如果没有把主表实体删除，仅仅是切断了二者之间的关联，则从表实体不会被删除，只是将从表实体的外键列设为null

所有的操作都是在调用期（call time）或写入期（flush time）级联到关联实体上的，Hibernate通常会尽量在调用持久化操作（调用期）将操作级联到关联实体。但是save-update和orphanRemoval的操作实在Session flush时（写入期）才级联到关联实体的。

## 继承映射 ##

Hibernate提供了3种策略映射继承：

- 整个类层次对应一张表；
- 连接子类的映射策略；
- 每个具体类对应一个表。

### 整个类层次对应一个表的映射策略 ###

父类和所有子类都映射为一张数据表。该表的数据列是整个类层次中所有实体的全部属性的总和。此外还有额外的一列，用于区分每行记录到底是哪个类的实例，该列被称为辨别者列（discriminator），需要在根父类中使用`@DiscriminatorColumn`来配置。该注解支持以下属性：

属性                 | 是否必需 | 说明
---                  | ----     | ---
`name`               | 否       | 辨别者列的名称，默认为`DTYPE`
`columnDefinition`               | 否       | 使用该属性值指定的SQL片段来创建辨别者列
`discriminatorType`               | 否       | 指定辨别者列的数据类型。支持以下几种类型，默认类型为`DiscriminatorType.STRING`：<br/>  - `DiscriminatorType.CHAR`：字符类型，该列只接受单个字符<br/> - `DiscriminatorType.INTEGER`：整数类型，该列只接受整数值<br/> - `DiscriminatorType.STRING`：字符串类型，该列只接受字符串值。 
`length`               | 否       | 辨别者列的字符长度

除此之外，继承树上的每个类（包括父类和所有子类）还需要使用`@DiscriminatorValue`来标注，该注解只需要设置一个`value`属性，用于指定不同该实体在辨别者列上的值。

这种策略有一个缺点：子类中增加的属性映射的字段不能有非空约束，因为父类的实例没有这些属性，只能传递null值。

### 连接子类的映射策略 ###

此策略不是Hibernate默认的继承映射策略，因此需要在继承树的根父类中使用`@Inheritance`指定使用。该注解必须设置`strategy`属性，支持以下3个值：

- `InheritanceType.SINGLE_TABLE`：整个类层次对应一个表，也是默认值；
- `InheritanceType.JOINED`：连接子类的映射策略；
- `InheritanceType.TABLE_PER_CLASS`：每个具体类对应一个表的策略。

这种策略下，无须使用辨别者列。父类实体保存在父类表中；子类实体则由父类表和子类表共同存储，其中父类表中保存父类和子类公有的属性，子类增加的属性则保存在子类表中，可以为这些属性添加非空约束。

父类和子类之间的关联属于自关联，所有的自关联中的外键列都不能有非空约束。

在底层数据库中，父类和子类的数据表通过一个共有主键互相关联，在查询子类记录时，通过一个多表连接（join）实现拼接。

### 每个具体类对应一个表的映射策略 ###

顾名思义，子类实例的数据仅保存在子类表中，和父类表没有任何关系。子类表中的数据列是该类的直接和间接父类，以及自身增加的属性的总和。父类表和子类表在结构上看不出继承关系，只是他们的主键值具有某种连续性，因此不能让数据库为这些数据表自动生成主键值，所以不能使用`GenerationType.AUTO`和`GenerationType.IDENTITY`这两种主键生成策略。

采用这种策略的类继承数，只需要在根父类中将`@Inheritance`的`strategy`属性值设置为`InheritanceType.TABLE_PER_CLASS`即可，其直接和间接子类都不需要做额外标注。

这种策略很难生成高效的SQL语句，因此通常建议尽量避免使用这种策略。

## 批量处理策略 ##

### 批量插入 ###

插入的新纪录都将在Session级别的缓存区进行缓存，如果一次插入的记录过多，将抛出`OutOfMemoryException`异常。所以应该定时将Session缓存的数据刷入数据库，而不是一直在Session级别缓存。可以考虑设计一个累加器，根据累加器的值决定是否需要将缓存中的数据刷入数据库。除了手动清空Session级别的缓存，最好关闭`SessionFactory`级别的二级缓存。

### JPA与Hibernate ###

JPA本质上是一种ORM规范，提供的只是一些接口，并没有提供实现。

使用Hibernate编程的核心API是`SessionFactory`，`Session`，`Transaction`，而JPA也提供了相应的API，分别是`EntityManagerFactory`，`EntityManager`，`EntityTransaction`。

### JPA的批量插入 ###

使用JPA的编程步骤：

1. 获取`EntityManagerFactory`
2. 获取`EntityManager`，打开事务
3. 用面向对象的方式操作数据库
4. 关闭事务，关闭`EntityManager`

使用JPA与使用Hibernate的一个区别在于配置文件。

- Hibernate使用类路径下的`hibernate.cfg.xml`作为配置文件；
- JPA使用类路径下的`META-INF`子目录下的`persistence.xml`作为配置文件。

直接调用`Persistence`类的`createEntityManagerFactory()`静态方法即可，该方法需要传入一个字符串参数，也就是`<persistence-unit.../>`元素的`name`属性名字。

### 批量更新 ###

批量更新数据时如果需要返回多行数据，应该使用`scroll()`方法。但这种方法的效能不高。

### DML风格的批量更新/删除 ###

批量更新/删除的HQL语句：

		update | delete from? <ClassName> [where where_conditions]

- 在`from`子句中，`from`关键字是可选的；
- 在`from`子句中只能有一个类名，可以在该类名后指定别名；
- 不能在批量HQL语句中使用连接，显式或者隐式的都不行。但可以在`where`子句中使用子查询；
- 整个`where`子句是可选的。`where`子句的语法和HQL语句中`where`子句的语法完全相同。

这种语法类似与`PreparedStatement`的`executeUpdate()`语法，返回一个整数值，该值是受此操作影响的记录数量。如果该方法在底层数据库需要转换成多条`update`或`delete`语句，则只能返回最后一条SQL语句影响的记录行数。

### JPA的DML支持 ###

JPA和Hibernate类似，只是需要将Hibernate的`Session`换成JPA的`EntityManager`即可。

## HQL查询和JPQL查询 ##

### HQL查询 ###

HQL（Hibernate Query Language），是一种面向对象的查询语言，支持继承，多态等特性。  
SQL语言操作的对象是数据表，列等数据库对象；  
HQL的操作对象是类，实例，属性等。

HQL查询依赖于`Query`类，每个`Query`实例对应一个查询对象。使用HQL查询按如下步骤进行：

1. 获取Hibernate Session对象；
2. 编写HQL语句；
3. 以HQL语句为参数，调用Session的`createQuery()`方法创建查询对象；
4. 如果HQL语句包含参数，则调用`Query`的`setParameter()`方法为参数赋值；
5. 调用`Query`对象的`getResultList()`或`getSingleResult()`方法返回查询结果列表或单条结果。

HQL的占位符既可以使用英文问号+索引的JPQL占位符语法形式（?N）（推荐使用）；  
也可以使用有名字的占位符（英文冒号+占位符名称）。

`Query`的`setParameter(String name, Xxx value)`方法既可以使用参数名，也可以使用参数索引设置参数值。

`Query`对象可以链式调用`setParameter()`方法来为多个参数赋值。

`Query`还包含如下2个方法：

- `setFirstResult(int firstResult)`：设置返回的结果集从第几条记录开始；
- `setMaxResults(int maxResults)`：设置本次查询返回的最大结果数目。

HQL本身不分大小写，但HQL语句中所使用的包名，类名，实例名，属性名都区分大小写。

### JPQL查询 ###

与HQL相对应，JPA提供了JPQL。JPA为`EntityManager`提供了`createQuery()`方法，返回JPA的`Query`对象，程序用此对象完成JPQL查询。

### from子句 ###

`from`关键字后紧跟持久化类的类名，表明从该持久化类对应的表中选出全部的实例。  
可以为持久化类起个别名，例如：`from Person as p`。`as`关键字可选，但建议保留提高可读性。

### 关联和连接 ###

HQL（JPQL）支持2种关联连接（join）形式：隐式（implicit）和显式（explicit）。

隐式连接不使用`join`关键字，使用英文点号来隐式连接关联实体，由Hibernate底层自动进行关联查询。  
显式连接需要使用`xxx join`关键字。使用显式连接可以为相关联的实体，甚至是关联集合中的全部元素指定一个别名。

HQL（JPQL）支持使用的多表查询的关键字如下：

- `inner join`（内连接），可简写为`join`
- `left outer join`（左外连接），可简写为`left join`
- `right outer join`（右外连接），可简写为`right join`
- `full join`（全连接），不常用

使用显式连接时，还可以通过`with`关键字提供额外的连接条件。该关键字和SQL99中的`on`关键字作用差不多，都用于指定连接条件。

隐式和显式连接还有以下两点区别：

- 隐式连接底层将转换SQL99的交叉连接，显式连接底层将转换为SQL99的`inner join, left join, right join`等连接；
- 隐式连接和显式连接查询后返回的结果不同：
	- 省略`select`关键字时，隐式连接查询返回的结果是多个被查询实体组成的集合；
	- 省略`select`关键字时，显式连接返回的也是集合，但集合元素是由被查询的持久化对象，所有被关联的持久化对象所组成的数组。

Hibernate 3.2.3以后的版本中，如果关联实体是单个实体或单个的组件属性，HQL依然可以使用英文点号来隐式连接关联实体或组件；  
但如果关联实体是集合（包括1-N关联，N-N关联和集合元素是组件等），则必须使用`xxx join`来显式连接关联实体或组件。

Hibernate默认采用延迟加载集合属性。2种方法改变加载策略：

- 在持久化类中使用`fetch=FetchType.EAGER`指定加载策略为立即加载；
- 在HQL语句中使用`join fetch`立即加载关联的集合属性（或关联实体）。

使用`join fetch`时无须指定别名，因为相关联的对象不应当在`where`子句或其他任何子句中使用，被关联的对象也不会在被查询的结果中直接返回，而是应该通过父对象来访问。  
另外，在使用`fetch`关键字时还需要注意几点：

- `fetch`不应该与`setMaxResults()`或`setFirstResult()`方法同时使用；
- `fetch`不能与独立的`with`条件一起使用；
- 如果在一次查询中`fetch`多个集合，查询可能返回笛卡尔积；
- `full join fetch`与`right join fetch`没有任何意义。

如果在持久化类的注解中使用`fetch=FetchType.LAZY`启用了延迟加载，但在程序里又想预加载这些原本应该延迟加载的属性，则可以通过`fetch all properties`来强制Hibernate立即抓取这些属性。

### 查询的select子句 ###

select子句用于选择指定的属性或直接选择某个实体。  
select还可以选择组件属性包含的属性。

通常情况下，使用select子句查询的结果是集合，集合元素是select后的实例，属性等组成的数组；也可以通过`list(attr...)`显式将集合元素从数组改为List对象；或者调用某个类的构造器，使用选择出来的属性作为构造器参数创建该类的对象。  
即使select子句选择了某个持久化类的全部属性，Hibernate也只会将他们作为属性处理，而不会组装成该持久化类的对象。  
select支持给选中的表达式命名别名，通常和`new map`结合使用：

		// HQL语句返回的结果是集合，其中集合元素是Map对象，以personName作为Map的key，实际选出的值作为Map的value
		select new map(p.name as personName) from Person as p;

### HQL查询的聚集函数 ###

HQL支持与SQL完全相同的聚集函数：

- `avg`：计算属性平均值；
- `count`：统计选择对象的数量；
- `max`：统计属性值的最大值；
- `min`：统计属性值的最大值；
- `sum`：计算属性值的总和。

还支持字符串连接符，算术运算符，`distinct`和`all`关键字，以及SQL函数。

### 多态查询 ###

HQL支持多态查询，from后跟持久化类，不仅会查出该持久化类的全部实例，还会查询出该类的子类的全部实例。

### HQL查询的where子句 ###

如果没有为持久化实例命名别名，则可以直接使用属性名来引用属性；  
如果为持久化实例命名了别名，则应该使用完整的属性名。

Hibernate 3.2.3以后，只有当属性引用的是普通组件属性或者单独的关联实体时，才可接着在后面使用点号引用属性的属性，如果属性是集合属性，则不支持这种用法。 

`=`运算符不仅可以用来比较属性的值，也可以用来比较实例。特殊属性（小写）`id`可以用来表示一个对象的标识符（也可以使用该对象的属性名，使用`id`是一种简洁的写法）。  
`id`还可以用来代表组件类型的标识符。

在进行多态持久化的情况下，`class`关键字用来存取一个实例的辨别值（discriminator value）。嵌入`where`子句中的Java类名，将被作为该类的辨别值。当`where`子句中的运算符只支持基本类型或者字符串时，`where`子句中的属性表达式必须以基本类型或者字符串结尾，不要使用组件类型属性结尾。

### 表达式 ###

`where`子句中允许使用大部分SQL支持的表达式：

- 数学运算符：`+`，`-`，`*`，`/`等；
- 二进制比较运算符：`=`，`>=`，`<=`，`<>`，`!=`，`like`等；
- 逻辑运算符：`and`，`or`，`not`等；
- `in`，`not in`，`between`，`is null`，`is not null`，`is empty`，`is not empty`，`member of and not member of`等；
- 简单的`case`，`case...when...then...else...end`和`case`，`case when...then...else...end`等；
- 字符串连接符：如`value1 || value2`，或使用字符串连接函数`concat(value1, value2)`；
- 时间操作函数：`current_date()`，`current_time()`，`current_timestamp()`，`second()`，`minute()`，`hour()`，`day()`，`month()`，`year()`等；
- HQL还支持EJB-QL 3.0所支持的函数或操作：`substring()`，`trim()`，`lower()`，`upper()`，`length()`，`locate()`，`abs()`，`sqrt()`，`bit_length()`，`coalesce()`，和`nullif()`等；
- 支持数据库的类型转换函数，如`cast(...as...)`，第二个参数是Hibernate的类型名，或者`extract(...from...)`，前提是底层数据库支持`ANSI cast()`和`extract()`；
- 如果底层数据库支持单行函数：`sign()`，`trunc()`，`rtrim()`，`sin()`，则HQL语句也完全支持；
- HQL语句支持使用命名参数作为占位符，方法是在参数名前加英文冒号（:），例如`:start_date`，`:x1`等；也支持使用英文问号（?）+数字的形式（?N）的参数作为占位符。

可以在`where`子句中使用SQL常量，还可以在HQL语句中使用Java中的`public static final`类型的常量。除此之外，`where`子句还支持如下特殊关键字用法：

- HQL `index()`函数，作用于`join`的有序集合的别名；
- HQL 函数，把集合作为参数：`size(), minelement(), maxelement(), minindex(), maxindex()`，还有特别的`elements()`和`indices()`函数，可以用数量词加以限定，如`some, all, exists, any, in`。
- `in`与`between...and`可按如下方法使用：
		
		from DomesticCat cat where cat.name between 'A' and 'B';
		from DomesticCat cat where cat.name in ('Foo', 'Bar', 'Baz');

- `not in`和`not between...and`的使用：

		from DomesticCat cat where cat.name not between 'A' and 'B';
		from DomesticCat cat where cat.name not in ('Foo', 'Bar', 'Baz');

- `is null`与`is not null`可以被用来测试空值：

		from DomesticCat cat where cat.name where cat.name is null;
		from DomesticCat cat where cat.name where cat.name is not null;

- 如果在Hibernate配置文件中进行如下声明：

		<!-- HQL转换SQL语句时，将使用字符1和0来代替关键字true和false，然后就可以在表达式中使用1和0来构建布尔表达式了 -->
		<property name="hibernate.query.substitutions">true 1, false 0</property>

		from Cat cat where cat.alive = 0;

- `size`关键字用于返回一个集合的大小：

		from Cat cat where cat.kittens.size > 0;
		from Cat cat where size(cat.kittens) > 0;
		
- 对于有序集合，还可以使用`minindex()`与`maxindex()`函数代表最小和最大的索引序数。同理，可以使用`minelement()`与`maxelement()`函数代表集合中最小与最大的元素。

		from Calendar cal where maxelement(cal.holidays) > current_date;
		from Order order where maxindex(order.items) > 100;
		from Order order where minelement(order.items) > 10000;
		
- `elements()`和`indices()`函数，用于返回指定集合的所有元素和所有索引。还可以使用`any, some, all, exists, in`等SQL函数操作集合里的元素。

		// 操作集合元素
		select mother from Cat as mother, Cat as kit
		where kit in elements(foo.kittens);
		// p的name属性等于集合中某个元素的name属性
		select p from NameList list, Person p
		where p.name = some elements(list.names);
		// 操作集合元素
		from Cat cat where exists elements(cat.kittens);
		from Player p where 3 > all elements(p.scores);
		from Show show where 'fizard' in indices(show.acts);

  这些结构变量：`size, elements, indices, minindex, maxindex, minelement, maxelement`等，只能在`where`子句中使用。

- 在`where`子句中，有序集合（数组，List集合，Map对象）的元素可以通过`[]`运算符访问。

		// items是有序集合属性，items[0]代表第一个元素
		from Order order where order.items[0].id = 1234;
		// holidays是Map集合属性，holidays['national']代表key为national的元素
		select person from Person person, Calendar calendar
		where calendar.holidays['national day'] = person.birthday
		and person.nationality.calendar = calendar;
		// 下面同时使用List集合和Map集合属性
		select item from Item item, Order order
		where order.items[order.deliveredItemIndices[0]] = item and order.id = 11;
		select item from Item item, Order order
		where order.items[maxindex[order.items]] = item and order.id = 11;

- 在`[]`中的表达式可以是一个算术表达式。

		select item from Item item, Order order
		where order.items[size(order.items) - 1] = item;

### order by 子句 ###

查询返回的集合可以根据类或组件属性的任何属性进行排序：

		from Person as p 
		order by p.name, p.age;

还可以使用`asc`或`desc`关键字指定升序或降序的排序规则，默认采用升序。

		from Person as p
		order by p.name asc, p.age desc;

### group by 子句 ###

返回聚集值的查询可以对持久化类或组件属性的属性进行分组，分组使用`group by`子句

		select cat.color, sum(cat.weight), count(cat)
		from Cat cat
		group by cat.color;

出现在`select`后的属性，要么出现在聚集函数中，要么出现在`group by`的属性列表中。

		// select后出现的id处于group by之后，而name属性则出现在聚集函数中
		select foo.id, avg(name), max(name)
		from Foo foo join foo.names name
		group by foo.id

`having`子句用于对分组进行过滤

		select cat.color, sum(cat.weight), count(cat)
		from Cat cat
		group by cat.color
		having cat.color in (eg.Color.TABBY, eg.Color.BLACK)

`having`子句只能在有`group by`子句时才可以使用。`group by`子句与`order by`子句中都不能包含算术表达式。

### 子查询 ###

HQL子查询需要底层数据库支持，也需要用英文括号括起来。如果子查询是多行结果集，则应该使用多行运算符。  
HQL子查询只可以在`select`子句或`where`子句中出现。如果`select`子查询后的列表中包含多项，则在HQL中需要使用一个元组构造符（括号）。

### 命名查询 ###

HQL查询支持将查询所用的HQL语句放入注解中，而不是代码中。  
Hibernate支持使用标准的`@NamedQuery`注解来配置命名查询，支持如下属性：

属性    | 是否必需 | 说明
---     | ----     | ---
`name`  | 否       | 命名查询的名字
`query` | 否       | 命名查询所执行的HQL查询语句

当需要多个`@NamedQuery`定义命名查询时，可使用`@NamedQuerys`来组合多个`@NamedQuery`。

`Session`里提供了一个`getNamedQuery(String name)`方法，用于使用名字为`name`命名查询创建一个`Query`对象。

可以在XML映射文件中的`<hibernate-mapping.../>`元素中使用`<query.../>`子元素定义命名查询。

## 动态条件查询 ##

动态条件查询通过如下三个类完成：

- `CriteriaBuilder`：用于创建`CriteriaQuery`，`CriteriaUpdate`，`CriteriaDelete`，`Predicate`，`Expression`：
	- `CriteriaQuery`，`CriteriaUpdate`，`CriteriaDelete`：第一个API代表查询，后两个代表DML操作
	- `Expression`：代表表达式
	- `Predicate`：代表查询条件。使用`CriteriaBuilder`的运算符方法组合两个`Expression`来生成一个`Predicate`。
- `Root`：代表要查询的根实体；
- `Join`：代表一个关联。

执行条件查询的步骤如下：

1. 获得JPA的`EntityManager`对象，并打开事务；
2. 以`EntityManager`对象创建`CriteriaBuilder`对象；
3. 调用`CriteriaBuilder`对象的`createQuery()`方法创建`CriteriaQuery`查询，传入的类型参数代表该条件查询返回结果集中的元素类型；
4. 调用`CriteriaQuery`对象的`from()`方法设置查询的根实体，该方法返回一个`Root`对象，如果需要查询多个`Root`对象，则需要执行多次；
5. 调用`CriteriaQuery`对象的`select()`方法设置查询语句的`select`部分；
6. 调用`CriteriaQuery`对象的`where()`方法设置查询语句的`where`部分；
7. 以`CriteriaQuery`为参数，调用`EntityManager`的`createQuery()`方法创建`Query`对象，该对象执行`getResultList()`方法；  

`CriteriaUpdate`和`CriteriaDelete`生成的`Query`对象执行`executeUpdate`和`executeDelete`方法，返回`int`类型结果，代表影响的记录数。

`CriteriaQuery`代表一次查询，用于组织查询语句的子句，提供的方法大致对应了`select`语句的各种子句：

- `select(Selection<? extends T> selection)`：只需要select单个实体或单个属性时使用；
- `multiselect(Selection<?>... selections)`：需要select多个实体或多个属性时使用；
- `distinct(boolean distinct)`：去除重复行，对应查询语句中的`distinct`关键字；
- `from(Class<X> entityClass)`：设置查询的根实体，对应查询语句中的`from`子句；
- `where(Predicate... restrictions)`：设置查询条件，对应查询语句中的`where`子句；
- `groupBy(Expression<?>... grouping)`：设置分组，对应查询语句中的`group by`子句；
- `having(Predicate... restrictions)`：过滤分组，对应查询语句中的`having`子句；
- `orderBy(Order... o)`：设置排序，对应查询语句中的`order by`子句；

上面方法所需的各种`Selection`，`Expression`，`Predicate`，`Order`等参数通常都由`CriteriaBuilder`对象创建。

`Predicate`，`Expression`都是`Selection`的子接口，`Root`也是`Expression`的子接口。

JPA为每一个实体类都提供了一个元模型类，其类名通常就是实体类的类名添加一个下划线。  
JPA元模型类的优势在于能更好地利用泛型。

`Predicate`接口代表一个查询条件，该查询条件由`CriteriaBuilder`负责产生，包含了以下常用的方法：

- `equals|notEquals|gt|greaterThan|ge|greaterThanOrEqualTo|lt|lessThan|le|lessThanOrEqualTo`：判断表达式的值是否等于|不等于|大于|大于等于|小于|小于等于指定值或表达式的值；
- `between(Expression<? extends Y> v, Y x, Y y)`：判断表达式的值是否在某个区间之内；
- `like|not like(Expression<String> x, String pattern)`：判断表达式的值是否匹配某个字符串；
- `in(Expression<? extends T> expression)`：判断表达式的值是否等于系列值之一；
- `isEmpty|isNotEmpty(Expression<C> collection)`：判断集合表达式指定的集合是否为空|不为空；
- `isNull|isNotNull(Expression<C> collection)`：判断属性值是否为空|不为空；
- `not(Expression<Boolean> expression)`：表达式求否；
- `and(Predicate... restrictions)`：对多个表达式求与；
- `or(Predicate... restrictions)`：对多个表达式求或。

### 执行DML语句 ###

条件查询提供了`CriteriaDelete`和`CriteriaUpdate`方法来执行DML语句。这两类对象对应的`delete`语句和`update`语句的子句较少，所以方法也较少：

- `from(Class<T> entityClass)`：设置修改和删除的实体，对应于`from`子句；
- `where(Expression<Boolean> restriction)`：为修改和删除设置筛选条件，对应于`where`子句；
- `set(Path<Y> attribute, X value)`：`CriteriaUpdate`专用方法，为更新语句设置新的值，对应于`set`子句。

### select的用法 ###

如果需要查询多个属性和多个实体，或者查询多个属性和多个实体的组合，则可以使用`CriteriaQuery`的`multiselect()`方法，也可先用`CriteriaBuilder`的`array()`方法包装多个表达式，然后再调用`CriteriaQuery`的`select()`方法。  
此外，还可以将查询出来的多个属性封装成DTO对象，此时结果集的元素不再是数组，而是DTO对象，需要利用`CriteriaBuilder`的`construct()`方法创建`CompoundSelection`（`Selection`的子接口）对象。需要注意：使用`construct()`方法时，第一个参数是指定的DTO对象的类，后面的参数是该类的构造器的参数，因此JPA要求该类必须有一个可以使用这些参数的构造器。

### 元组查询 ###

除了上面介绍的利用DTO封装multiselect得到的多个属性，JPA还提供了一个`Tuple`接口，专门用于封装多个属性，这样就无须额外的DTO类。此时，查询的结果还是集合，但集合的元素类型不是数组或DTO，而是`Tuple`。

### 多Root查询 ###

`Root`代表条件查询要查询的目标实体。 
多`Root`查询的实现需要多次调用`CriteriaQuery`的`from()`方法，并使用`multiselect()`指定需要查询的目标实体。

当条件查询包含多个`Root`实体时，JPA要求显式使用`multiselect()`或`select()`方法指定要查询的实体，否则JPA会报异常。

### 关联和动态关联 ###

`Root`对象是获取关联的基础，JPA提供了`Join`接口来代表关联，`Root`和`Join`都是`From`的子接口。`From`接口提供了以下方法来建立关联：

- `join(XxxAttribute<? super X, Y> xxx)`：建立基于`XxxAttribute`属性的关联，该方法必须使用JPA元模型支持，因此可充分利用泛型，方法中的`Xxx`可以是`Collection`，`List`，`Set`，`Map`，`Singular`，分别代表关联属性的类型。默认使用内连接；
- `join(XxxAttribute<? super X, Y> xxx, JoinType jt)`：与前一方法类似，但可以通过`JoinType`参数指定左连接，右连接；
- `join(String attributeName)`：建立基于`attributeName`属性的关联，直接使用`String`类型的属性名，对泛型的支持不好；
- `join(String attributeName, JoinType jt)`：与前一方法类似，但可以通过`JoinType`参数指定左连接，右连接；
- `joinXxx(String attributeName)`：上上个方法的增强版，用方法名指定了关联属性的集合类型。其中`Xxx`可以是`Collection`，`List`，`Set`，`Map`，`Singular`，分别代表关联属性的类型；
- `joinXxx(String attributeName, JoinType jt)`：与前一方法类似，但可以通过`JoinType`参数指定左连接，右连接；

JPA支持Fetch关联查询，为`Root`提供了`fetch()`方法，返回一个`Fetch`对象。该方法来自`Root`和`Fetch`共同的父接口`FetchParent`。该接口提供了以下几种`fetch()`方法：

- `fetch(XxxAttribute<? super X, ?, Y> attribute, JoinType jt)`：设置使用Fetch关联，需要使用JPA元模型。其中`JoinType`参数用于指定连接类型，该参数可以省略，默认使用内连接；
- `fetch(XxxAttribute<String attributeName, JoinType jt)`：设置使用Fetch关联，直接使用`String`类型的属性名。其中`JoinType`参数用于指定连接类型，该参数可以省略，默认使用内连接；

### 分组，聚集和排序 ###

`CriteriaQuery`可以使用`groupBy()`方法进行分组；可以使用`min()`，`max()`，`avg()`，`count()`，`sum()`，`least()`，`greatest()`方法，前5个方法对应于SQL中的聚集函数，而`greatest()`相当于`max()`，不过`max()`只能作用域`Number`类型的表达式，而`greatest()`可以作用域字符串，日期等各种能比较大小的类型的表达式。`least()`和`min()`同理。

排序可以使用`orderBy()`方法实现，该方法需要传入`Order`对象 - 通过`CriteriaBuilder`的`asc()`和`desc()`方法来创建。

## 原生SQL查询 ##

原生SQL查询通过`NativeQuery`接口表示。该接口是`Query`的子接口，有两个特有的重载的方法：

- `addEntity()`：将查询到的记录与特定的实体关联；
- `addScalar()`：将查询的记录关联成标量值。

执行SQL查询的步骤如下：

1. 获取Hibernate Session对象；
2. 编写SQL语句；
3. 以SQL语句作为参数，调用`Session`的`createQuery()`方法创建查询对象；
4. 调用`NativeQuery`对象的`addScalar()`或`addEntity()`方法将选出的结果与标量值或实体关联，分别用于进行标量查询和实体查询；
5. 如果SQL语句包含参数，则调用`Query`的`setXxx()`方法为参数赋值；
6. 调用`Query`的`getResultList()`或`getSingleResult()`方法返回查询的结果。

### 标量查询 ###

最基本的SQL查询就是获得一个标量列表，如

		Query.createQuery("select * from student_inf").getResultList();

在默认情况下，将返回一个`Object`数组组成的`List`，数组里的每个元素都是`student_inf`表的列值，Hibernate会通过`ResultSetMetadata`来判定返回数据列的实际顺序和类型。  
如果`select`后面只有一个字段，那么返回的`List`集合元素就不是数组，而是单个的变量值。

在JDBC中过多地使用`ResultSetMetadata`会降低性能，因此建议通过`addScalar()`为这些数据列指定更明确的返回值类型:

		Query.createQuery("select * from student_inf")
			.addScalar("name", StandardBasicTypes.STRING)
			.getResultList();

该语句指定了：

- SQL字符串；
- 查询返回的字段列表；
- 查询返回的各字段类型；

此时，虽然在SQL字符串中使用了`*`作为查询的字段列表，但依然只会返回`name`字段所组成的列表。  
如果希望选出某个字段的值，但又不想明确指出该字段的数据类型，则可使用`addScalar(String columnAlias)`方法。通常只有在程序无法知道所查询数据列的数据类型时，才使用这个方法。否则还是使用可指定数据类型的重载方法。

总而言之，标量查询中的`addScalar()`方法有两个作用：

- 指定查询结果包含哪些数据列 - 没有通过该方法选出的列将不会包含在查询结果中；
- 指定查询结果中数据列的数据类型。

### 实体查询 ###

如果查询返回了某个数据表的全部数据列，且该数据表有对应的持久化类映射，则可以通过`Query`的多个重载的`addEntity()`方法把查询结果转换为实体。

必须选出数据表的所有数据列才可以被转换为持久化实体。

在原生SQL语句中也支持使用参数。这些参数可以使用问号+索引占位符（?N），或者直接使用名字。

如果在SQL语句中显式使用了多表连接，则可选出多个数据表的数据，Hibernate可以将查询结果转换为多个实体，此时SQL字符串中应该为不同的数据表指定不同的别名，并调用`addEntity(String alias, Class entityClass)`将不同的数据表转换成不同的实体。

此外，Hibernate支持将查询结果转换为非持久化实体，即普通的JavaBean，只要该JavaBean为这些数据列提供了对应的setter和getter方法即可。`Query`提供了一个`setResultTransformer()`方法（已标记过时），该方法接受一个`Transformer`对象，通过调用该对象的`aliasToBean(Class beanClass)`方法完成转换。JPA没有提供相应的功能。

### 处理关联和继承 ###

只要原生SQL选出了足够的数据列，程序除了可以将指定数据列转换为持久化实体之外，还可以将实体的关联实体（通常以属性的形式存在）转换成查询结果。使用的方法是`NativeQuery addJoin(String alias, String path)`，第一个参数是转换后的实体名，第二个参数是待转换的实体属性。

如果使用原生SQL查询的结果实体是继承树的一部分，则查询的SQL字符串必须包含基类和所有子类的全部属性。

### 命名SQL查询 ###

可以将SQL语句不放在程序而是在注解中管理。

Hibernate允许使用`@NamedNativeQuery`注解来定义命名的原生SQL查询，如果程序有多个命名的原生SQL查询需要定义，则可使用`@NamedNativeQuery`注解，用于组合多个命名的原生SQL查询。该注解支持以下属性：

属性               | 是否必需 | 说明
---                | ----     | ---
`name`             | 是       | 命名的原生SQL查询的名字
`query`            | 否       | 命名查询所执行的SQL查询语句
`resultClass`      | 否       | 指定一个实体类的类名，指定将查询结果集映射成该实体类的实例
`resultSetMapping` | 否       | 指定一个SQL结果映射（使用`@SqlResultSetMapping`定义）的名称，用于指定使用该SQL结果映射来转换查询结果集

`@SqlResultSetMapping`的作用是将查询得到的结果集转换为标量查询或实体查询 - 基本等同于`NativeQuery`对象的`addScalar()`或`addEntity()`方法的功能。该方法支持以下属性：

属性       | 是否必需 | 说明
---        | ----     | ---
`name`     | 是       | SQL结果映射的名称
`columns`  | 否       | 属性值为`@ColumnResult`注解数组，每个`@ColumnResult`定义一个标量查询
`entities` | 否       | 属性值为`@EntityResult`注解数组，每个`@EntityResult`定义一个实体查询
`classes`  | 否       | 属性值为`@ConstructorResult`注解数组，每个`@ConstructorResult`负责将指定的多列转换为普通类的对应属性

- `@ColumnResult`注解类似于`NativeQuery`的`addScalar()`方法，程序为`@SqlResultSetMapping`的`columns`属性指定了几个`@ColumnResult`，就相当于调用了`NativeQuery`的`addScalar()`方法几次；
- `@EntityResult`注解类似于`NativeQuery`的`addEntity()`方法，程序为`@SqlResultSetMapping`的`columns`属性指定了几个`@ColumnResult`，就相当于调用了`NativeQuery`的`addEntity()`方法几次；
- `@ConstructorResult`注解类似于`Query`的`setResultTransformer()`方法，把查询结果转换为普通JavaBean。

普通的SQL查询只要通过`resultClass`属性即可将查询结果转换为指定实体，如果查询包含的数据列比较多，且程序希望同时进行标量查询，实体查询，就必须借助于`@SqlResultSetMapping`来定义SQL结果映射。

### 调用存储过程 ###

从Hibernate3开始，可以通过命名SQL查询来调用存储过程或函数。函数必须返回一个结果集；存储过程的第一个参数必须是传出参数，且其数据类型必须是结果集。

Hibernate当前仅支持存储过程返回标量和实体，同时还需注意：

- 建议采用的调用方式是标准SQL92语法，如`{? = call functionName(<parameters>)}`或`{call procedureName(<parameters>)}`，不支持原生的调用语法；
- 因为存储过程本身完成了查询的全部操作，因此调用存储过程进行的查询无法使用`setFirstResult()/setMaxResults()`进行分页；

对于Oracle，有如下规则：

- 函数必须返回一个结果集，存储过程的第一个参数必须是`OUT`，它返回一个结果集，这个结果集由Oracle9或Oracle10的`SYS_REFCURSOR`类型来完成。也就是在Oracle存储过程中需要定义一个`REF CURSOR`类型，并使用该类型来定义函数返回值或者存储过程的第一个参数。

对于Sybase或者MS SQL Server有如下规则：

- 存储过程必须返回一个结果集，虽然这些数据库可能返回多个结果集或记录的更新条数，但Hibernate只能取出第一个结果集作为它的返回值，其他的将被丢弃；
- 如果可以在存储过程里设定`SET NOCOUNT ON`，可能会让效率更高（但并不是必需）。

### 使用定制SQL ###

当Hibernate需要保存，更新，删除持久化实体时，默认通过一套固定的SQL语句来完成这些功能，如果需要改变这套默认的SQL语句，可以使用Hibernate提供的定制SQL功能。Hibernate为定制SQL提供了如下注解：

- `@SQLInsert`：定制插入记录的SQL语句；
- `@SQLUpdate`：定制更新记录的SQL语句；
- `@SQLDelete`：定制删除记录的SQL语句；
- `@SQLDeleteAll`：定制删除所有记录的SQL语句。

这些注解都有一个`sql`属性，该属性值为执行相应操作的SQL语句。  
如果需要使用存储过程来执行这些操作，只需将这些注解的`callable`属性设置为true即可。注意调用存储过程的顺序必须和Hibernate所期待的顺序相同。  
程序可以将`org.hibernate.persister.entity`日志设为debug级别，从而允许查看Hibernate所期待的顺序 - 查看前先不要使用定制SQL功能，等记录了Hibernate静态SQL所期待的顺序后，再到持久化注解中配置定制SQL。

因为Hibernate会检查SQL语句是否执行成功，所以应该让存储过程能返回该过程所影响的记录条数。Hibernate通常把CUD操作语句的第一个参数注册为数值型输出参数，所以应使用存储过程的第一个传出参数记录该存储过程所影响的记录数。

此外，由于命名的SQL查询可用于查询指定实体，因此还可以使用命名查询来实现定制加载，即对查询结果进行额外处理后，再以此生成实体。该功能需要：

- `@Loader`：通过`namedQuery`属性指定使用的命名查询；
- `@NamedNativeQuery`：定义一个命名查询，在该注解中指定实现后处理的SQL语句。

### JPA的原生SQL查询 ###

步骤如下：

1. 获取`EntityManager`对象；
2. 编写SQL语句；
3. 以SQL语句作为参数，调用`EntityManager`对象的`createNativeQuery()`方法创建`Query`对象；
4. 如果SQL语句包含参数，则调用`Query`的`setXxx()`方法为参数赋值；
5. 调用`Query`的`getResultList()`或`getSingleResult()`方法得到查询的结果。

JPA的原生SQL查询是在`PreparedStatement`的基础上进一步封装，得到的查询结果不是`ResultSet`结果集，而是一个`List`集合。

JPA执行原生SQL查询时不会跟踪脱管实体的状态，所以应该避免在原生SQL中使用`insert, update, delete`语句，这些语句会导致数据库的数据发生改变，但这种改变不会反映到实体中，从而导致脱管实体和底层数据库的不一致。  
因此，如果使用JPA作为持久化解决方案，应该尽量避免使用原生SQL查询，尤其是避免使用原生SQL的`insert, update, delete`语句。

JPA的原生SQL查询返回的结果是一个`List`集合，集合元素的类型取决于实现框架，可以是数组类型（如Hibernate），也可以是`List`集合类型（如TopLink）。

JPA的原生SQL查询也支持将查询结果转换为实体：在选择了某个实体所需的全部数据列后，在调用`createNativeQuery()`方法时传入该实体类的类名即可。

JPA的原生SQL查询并没有提供`addEntity()`，`addScalar()`，`addJoin()`方法来处理标量查询，实体查询以及关联。这些功能可以通过`@SqlResultSetMapping`和`@SqlResultSetMappings`（用于组织多个`SqlResultSetMapping`）来实现。

`@SqlResultSetMapping`支持如下属性：

属性       | 是否必需 | 说明
---        | ----     | ---
`name`     | 是       | SQL结果映射的名称
`columns`  | 否       | 属性值为`@ColumnResult`注解数组，指定查询结果应该包含的数据列，每个`@ColumnResult`定义一个查询列。
`entities` | 否       | 属性值为`@EntityResult`注解数组，指定将结果集映射成一个或多个实体，每个`@EntityResult`定义一个实体映射。
`classes`  | 否       | 属性值为`@ConstructorResult`注解数组，指定将结果集映射成一个或多个DTO对象，每个`@ConstructorResult`定义一个DTO映射。

其中`columns`，`entites`，`classes`至少需要指定一个。

`@ColumnResult`支持的属性如下：

属性       | 是否必需 | 说明
---        | ----     | ---
`name`     | 是       | 数据列的名称
`type`     | 否       | 数据列的类型

`@EntityResult`支持的属性如下：

属性       | 是否必需 | 说明
---        | ----     | ---
`entityClass`     | 是       | 映射结果集的实体类的类名
`discriminatorColumn`     | 否       | 在查询的数据列中作为辨别者列的列名，只有在查询的数据表中使用了继承映射时才需要指定该属性
`fields`     | 否       | 将选出的数据列映射为实体的属性，该属性的属性值是`@FieldResult`数组，每个`@FieldResult`完成一个数据列和一个实体属性之间的映射

`@FieldResult`支持的属性如下：

属性     | 是否必需 | 说明
---      | ----     | ---
`name`   | 是       | 需要映射到的实体属性
`column` | 是       | 需要映射的数据列

`@ConstructorResult`支持的属性如下：

属性          | 是否必需 | 说明
---           | ----     | ---
`columns`     | 是       | 将选出的数据列映射为DTO对象的属性，该属性的属性值是`@ColumnResult`数组，每个`@ColumnResult`定义一个数据列
`targetClass` | 是       | DTO类的类名

## 数据过滤 ##

一旦启用了数据过滤器，则不管数据查询还是数据加载，该过滤器将自动作用于所有数据，只有满足过滤条件的记录才会被选出来。

过滤器与修饰持久化类的`@Where`注解非常相似。区别是过滤器可以带参数，应用程序可以在运行时决定是否启用指定的过滤器，使用怎样的参数值；而修饰持久化类的`@Where`注解将一直生效，且无法动态传入参数。

过滤器的用法很像数据库视图，区别是视图在数据库中已经定义完成，而过滤器还需要在应用程序中确定参数值。

过滤器的使用分为三步：

1. 定义过滤器。使用Hibernate提供的`@FilterDef`定义过滤器，也可以使用`@FilterDefs`组合多个`@FilterDef`。
2. 使用过滤器。使用`@Filter`元素应用过滤器。
3. 在代码中通过`Session`启用过滤器。

`@FilterDef`通常用于修饰持久化类，用于定义过滤器；`@Filter`则通常用于修饰持久化类或集合属性（包括关联实体），表示对指定持久化类或集合属性应用过滤器。过滤器和持久化类或集合属性是多对多的关系。

`@FilterDef`支持的属性：

属性               | 是否必需 | 说明
---                | ----     | ---
`name`             | 是       | 过滤器的名称
`defaultCondition` | 否       | 属性值为带参数的SQL条件表达式，用于指定该过滤器默认的过滤条件
`parameters`       | 否       | 指定过滤器中SQL条件表达式支持的参数

`@Filter`既可以修饰持久化类，此时该过滤器将对整个持久化类的所有实例进行过滤；  
也可修饰集合属性，此时该过滤器对集合属性进行过滤。 
`condition`属性的值是一个SQL风格的where子句，指定的过滤条件应该根据表名，列名进行过滤。

系统默认不启用过滤器，必须通过`Session`的`enableFilter(String filterName)`才可以启用。该方法返回一个`Filter`实例，该实例使用`setParameter()`来为过滤器参数赋值。 
一旦启用了过滤器，将在整个Session内有效，所有的数据加载都将自动应用该过滤条件，直到调用`disableFilter()`方法。

如果只是临时的数据筛选，可以直接使用常规查询；  
如果某个筛选条件使用的非常频繁，可以将其设为过滤器；  
对于在SQL语句中使用行内表达式，视图的地方，也可以考虑使用过滤器。

在一个持久化类中定义的过滤器，可以在其他不同的持久化类中使用，前提是他们都由同一个`SessionFactory`负责加载并管理。

## 事务控制 ##

当多个数据库原子访问绑定在一起实现某个业务逻辑时，这些访问应该被绑定为一个整体 - 即事务。  
事务是一个最小的逻辑执行单元，一个事务的执行不能分开执行，具有原子性。

### 事务的概念 ###

事务通常具备4个特性（被称为ACID）：

- 原子性（Atomicity）：事务是应用中的最小执行单位。
- 一致性（Consistency）：事务执行的结果，必须使数据库从一种一致性状态，变到另一种一致性状态。一致性是通过原子性来保障的。
- 隔离性（Isolation）：各个事务的执行互不干扰，并发执行的事务之间不能互相影响。
- 持续性（Durability）：事务一旦提交，对数据所做的任何改变都要记录到永久存储器中，通常是保存进物理数据库。

### Session与事务 ###

Hibernate的事务（`Transaction`对象）通过`Session`的`beginTransaction()`方法显式打开，Hibernate本身不提供事务控制行为，底层直接使用JDBC连接，JTA资源或其他资源的事务。  
从编程角度，Hibernate的事务由`Session`对象开启；  
从底层实现角度，Hibernate的事务由`TransactionFactory`的实例来产生。

`TransactionFactory`是一个事务工厂接口，Hibernate为不同的事务环境提供了不同的实现类，如`CMTTransactionFactory`（针对容器管理事务环境的实现类），`JDBCTransactionFactory`（针对JDBC局部事务环境的实现类），`JTATransactionFactory`（针对JTA全局事务环境的实现类）。

应用程序无须手动操作`TransactionFactory`产生事务，因为`SessionFactory`已经在底层封装了`TransactionFactory`。`SessionFactory`对象的创建代价很高，是线程安全的对象，可以被所有线程共享。通常，`SessionFactory`对象在应用程序启动时创建，一旦创建就不会轻易关闭，只有当退出应用时才会关闭。

`Session`对象是轻量级的，也是线程不安全的。单个的业务进程，工作单元只会使用一次`Session`对象。  
创建`Session`时，并不会立即打开与数据库之间的连接，只有需要进行数据库操作时，`Session`才会获取JDBC连接，所以打开和关闭`Session`不会对性能造成重大影响。由此可见，长`Session`对应用性能的影响不大，只要它没有长时间打开数据库连接。  
即使如此，也应该让数据库事务尽可能地短，从而降低数据库锁定造成的资源争用。

Hibernate的所有持久化访问都必须在`Session`的管理下进行，但并不推荐因为一次简单的数据库原子操作，就打开和关闭一次`Session`，数据库事务也是如此。因为事务往往是由多个原子操作组合而成的一个逻辑整体。

Hibernate建议采用每个请求对应一次`Session`的模式，因此一次请求通常表示需要执行一个完整的业务功能，这个功能由一系列数据库原子操作组成，是一个逻辑上的整体。

每次HTTP Session 对应一次Hibernate Session 的模式会导致应用程序由于数据库事务和Session一直处于打开状态，导致数据库被锁定而无法扩展并发用户的数量，因此不推荐使用。

如果需要处理应用程序长事务，可以采用以下模式来解决：

- 自动版本化：Hibernate能自动进行乐观并发控制，如果在用户思考过程中持久化实体发生并发修改，Hibernate能够自动检测到；
- 脱管对象：如果采用每次用户请求对应一次Session的模式，那么前面载入的实例在用户思考过程中，始终与Session脱离，处于脱管状态。Hibernate可以在稍后将脱管对象重新关联到Session上，并对修改进行持久化。此模式下，自动版本化用来隔离并发修改。使用脱管对象的每次请求都对应一个Hibernate Session；
- 长生命周期Session：Session在数据库事务提交之后，断开和底层JDBC连接。当新的客户端请求到来时，又重新连接上底层的JDBC连接。每个应用程序事务对应一个Session。因为应用程序事务一般都相当长，所以也被称为长生命周期Session。

Session缓存了处于持久化状态的每个对象，如果Session打开的时间过长，或载入了过多的数据，会造成内存不足，抛出`OutOfMemoryException`异常。因此，程序应该定期调用`clean()`和`evict()`方法来清理Session的缓存。大批量的数据处理推荐使用DML风格的HQL语句完成。

如果在Session范围之外访问为初始化的集合或代理，Hibernate会抛出`LazyInitializationException`异常。也就是说，在脱管状态下，访问一个实体所拥有的集合，或者访问其指向代理的属性时，都会引发该异常。

为了保证在Session关闭前初始化代理属性或集合属性，可以使用`Hibernate`的`initilize(Object proxy)`静态方法来强制初始化某个集合或代理。

如果需要保证Session一直处于打开状态，可以采用以下方法：

- 在一个Web应用中，可以利用过滤器，在用户请求结束，页面生成结束时关闭Session。也就是保证在视图显示层一直打开Session，即所谓的Open Session in View 模式。这种模式下，必须保证所有的异常得到正确处理，在呈现视图界面之前，或者在生成视图界面的过程中一旦发生异常，必须保证可以正确关闭Session，并结束事务。

- 让业务逻辑层来负责准备数据，业务逻辑层在返回数据之前，对每个所需集合调用`Hibernate.initialze()`方法，或者使用带`fetch`子句或`FetchMode.JOIN`的查询，实现取得所有数据，并将这些数据封装成VO（值对象）集合，然后程序就可以关闭Session了。业务逻辑层将VO集合传入视图层，让视图层只负责简单的逻辑显示。

### 上下文的Session ###

`HibernateUtil`工具类可以保证将线程不安全的Session绑定限制在当前线程内 - 也就是实现一种“上下文相关”的Session。

从Hibernate 3.1 开始，`SessionFactory.getCurrentSession()`的底层实现是可插拔的，Hibernate引入了`CurrentSessionContext`接口，并通过`hibernate.current_session_context_class`参数来管理上下文相关的`Session`的底层实现。

`CurrentSessionContext`接口有如下三个实现类：

- `org.hibernate.context.JTASessionContext`：根据JTA来跟踪和界定上下文相关的Session；
- `org.hibernate.context.ThreadLoacalSessionContext`：通过当前正在执行的线程来跟踪和界定上下文相关的Session，和`HibernateUtil`的Session维护模式相似，Hibernate的Session会随着`getCurrentSession()`方法自动打开，并随着事务提交自动关闭；
- `org.hibernate.context.ManagedSessionContext`：通过当前正在执行的线程来跟踪和界定上下文相关的Session，但程序需要使用这个类的静态方法将Session实例绑定，取消绑定，它并不会自动打开，flush或关闭任何Session。

如果在容器中使用Hibernate，通常会采用第一种方式；  
而独立的Hibernate应用通常采用第二种方式。

在`hibernate.cfg.xml`配置文件中设置策略：

		// 可以使用thread, jta, managed之一
		<property name="hibernate.current_session_context_class">thread/jta/managed</property>

## 二级缓存和查询缓存 ##

Hibernate包括2个级别的缓存：

- 默认总是启用的Session级别的一级缓存；
- 可选的`SessionFactory`级别的二级缓存。

一级缓存不需要开发者关心，默认总是有效。当应用保存，修改持久化实体时，Session不会立即把这种改变flush到数据库，而是缓存在当前Session的一级缓存中，除非程序显式调用Session的`flush()`方法，或程序关闭Session时才会把这些改变一次性地flush到数据库。

二级缓存是全局性的，应用的所有Session都共享这个二级缓存，默认是关闭的，必须由程序显式开启。一旦开启，当Session需要抓取数据时，会先查找一级缓存，再查找二级缓存，只有当2个缓存中都没有所需数据时，才会去查找底层数据库。

### 开启二级缓存 ###

在`hibernate.cfg.xml`中做如下配置：

		<!-- 开启二级缓存 -->
		<property name="hibernate.cache.use_second_level_cache">true</property>
		<!-- 设置二级缓存的实现类 -->
		<property name="hibernate.cache.region.factory_class">org.hibernate.cache.ehache.EhCacheRegionFactory</property>

一旦开启二级缓存，并且设置了对某个持久化类启用缓存，`SessionFactory`就会缓存应用访问过的该实体类的每个对象，除非缓存的数据超出缓存空间。

Hibernate支持以下的缓存实现：

缓存                | 缓存实现类                                               | 类型                         | 集群安全 | 查询缓存支持
---                 | ---                                                      | ---                          | ---      | ---
`ConcurrentHashMap` | `org.hibernate.testing.cache.CachingRegionFactory`       | 内存                         |          
`EhCache`           | `org.hibernate.cache.ehcache.EhCacheRegionFactory`       | 内存，磁盘，事务性，支持集群 | 是       | 是
`Infinispan`        | `org.hibernate.cache.infinispan.InfinispanRegionFactory` | 事务性，支持集群             | 是       | 是

`ConcurrentHashMap`只是一种内存级别的缓存实现，是作为测试使用的缓存实现，不推荐在实际项目中使用。

EhCache为例，Hibernate的二级缓存的用法：

1. 在`hibernate.cfg.xml`配置文件中开启二级缓存并设定实现类为EhCache；
2. 将缓存实现所需要的配置文件添加到系统的类加载路径中。EhCache需要一个`ehcache.xml`配置文件；
3. 设置对哪些实体类，实体的哪些集合属性启用二级缓存。2种方式：
	- （推荐使用）修改要使用缓存的持久化类文件，使用Hibernate提供的`@Cache`标注该持久化类或标注集合属性；
	- 在`hibernate.cfg.xml`文件中使用`<class-cache.../>`或`<collection-cache.../>`元素指定需要启用二级缓存的持久化类或集合属性。

`ehcache.xml`样本：

		<?xml version="1.0" encoding="utf-8"?>
		<ehcache>
			<diskStore path="java.io.tmpdir"/>
			<defaultCache
				maxElementsInMemory="10000" // 设置缓存中最多方多少个对象
				eternal="false" 	// 设置缓存是否永久有效
				overflowToDisk="true"  
	 			timeToIdleSeconds="120"  // 设置缓存的对象多少秒没有被使用就会被清理调
				timeToLiveSeconds="120" // 设置缓存对象在过期之前可以缓存多少秒
	 			diskPersistent="false"/>  // 设置缓存是否被持久化到硬盘中，保存路径由<diskStore.../>元素指定
		</ehcache>

通过`@Cache`的`usage`属性指定缓存的策略，支持以下几种：

- 只读（`CacheConcurrencyStrategy.READ_ONLY）：应用程序只需读取持久化实体的对象而无须对其修改；
- 读/写（`CacheConcurrencyStrategy.READ_WRITE）：应用程序需要更新数据，与序列化事务（serializable transaction）不兼容。<br/> 如果在JTA环境下使用，则必须指定`hibernate.transaction.manager_lookup_class`属性的值。<br/> 在其他环境中，程序必须在`Session.close()`或`Session.disconnect()`之前结束整个事务。<br/> 在集群环境中，必须保证底层的缓存实现支持锁定（locking）。Hibernate内置的缓存策略不支持锁定功能；
- 非严格读/写（`CacheConcurrencyStrategy.NONSTRICT_READ_WRITE）：应用程序只需要偶尔更新数据（两个事务同时更新同一记录的情况很少），也不需要十分严格的事务隔离。<br/> 如果在JTA环境下使用，则必须指定`hibernate.transaction.manager_lookup_class`属性的值。
- 事务缓存（`CacheConcurrencyStrategy.TRANSACTIONAL）：提供了全事务的缓存支持，只能在JTA环境中其作用，必须指定`hibernate.transaction.manager_lookup_class`属性的值。

### 管理缓存和统计缓存 ###

一级缓存是局部缓存，只对当前Session有效；  
二级缓存是全局缓存，对所有Session都有效。

`Session`中处理一级缓存的方法：

- `evict(Object obj)`：将指定对象或集合从一级缓存中清除
- `clear()`：清除一级缓存中的所有对象
- `contains(Object obj)`：一级缓存中是否包含指定对象或集合

`SessionFactory`提供了`getCache()`方法，返回一个`Cache`对象，代表二级缓存，通过该对象实现对二级缓存的操作，参见Hibernate API。

开启二级缓存的统计功能需要在`hibernate.cfg.xml`中作如下配置：

		<!-- 开启二级缓存的统计功能 -->
		<property name="hibernate.generate_statistics">true</property>
		<!-- 设置使用结构化方式来维护缓存 -->
		<property name="hibernate.cache.use_structured_entries">true</property>

调用`SessionFactory`对象的`getStatistics().getSecondLevelCacheStatistics()`方法返回一个`SecondLevelCacheStatistics`对象，使用该对象提供的API来分析统计二级缓存。

### 使用查询缓存 ###

一级二级缓存都是缓存整个实体，不会缓存普通属性。  
使用查询缓存来实现缓存普通属性，但可能造成性能下降，慎用！

查询缓存的key就是查询所用的HQL或SQL语句，判断key相同的标准不仅要求所使用的HQL或SQL语句相同，传入的参数也要求相同。只有经常使用相同的查询语句和相同的查询参数才能从查询缓存获利，查询缓存的生命周期知道属性被修改了为止。  

查询缓存默认是关闭的，开启需要在`hibernate.cfg.xml`中做如下配置：

		<!-- 开启查询缓存 -->
		<property name="hibernate.cache.use_query_cache">true</property>

此外，还必须调用`Query`对象的`setCacheable(true)`才能缓存查询结果。

## 事件机制 ##

Hibernate的事件框架分为2个部分：

- 拦截器机制：拦截特定动作，回调应用中的特定动作；
- 事件系统：重写Hibernate的事件监听器。

### 拦截器 ###

通过`Interceptor`接口，可以从Session中回调应用程序的特定方法，从而在持久化对象被保存，更新，删除或加载之前，检查并修改其属性；  
还可以在数据进入数据库之前，对数据进行最后的检查，将非法数据摒除或修改为合法数据。

使用拦截器的步骤：

1. 定义实现`Interceptor`接口的拦截器类，最好通过继承`EmptyInterceptor`类来实现；
2. 通过`Session`启用拦截器，或者通过`Configuration`启用全局拦截器：
	- 通过`SessionFactory`的`openSession(Interceptor interceptor)`方法打开一个带局部拦截器的Session；
	- 通过`Configuration`的`setInterceptor(Interceptor interceptor)`方法设置全局拦截器。

### 事件系统 ###

事件系统可以代替拦截器或作为拦截器的补充使用。

基本上，`Session`接口的每个方法都有对应的事件，如`LoadEvent`，`FlushEvent`等。当Session调用某个方法时，Hibernate Session会生成相应的事件，并激活对应的事件监听器。

监听器是单例模式对象，所有同类型的事件处理共享同一个监听器实例，因此监听器不能保存任何状态，即不应该使用任何成员变量。

使用事件系统的步骤：

1. 实现自己的事件监听器类；
2. 注册自定义的事件监听器，代替系统默认的事件监听器。

实现自定义事件监听器的三种方法：

- 实现对应的监听器接口：必须实现接口中的所有方法，关键是必须实现Hibernate对应的持久化操作，即数据库访问，这就意味着开发一套完全替代Hibernate底层操作的方法；
- 继承事件适配器：选择性地实现需要关注的方法，但仍然试图取代Hibernate完成数据库的访问；
- 继承系统默认的事件监听器：扩展特定方法，推荐使用。扩展自定义事件监听器时，一定要在方法中调用父类的对应方法。

Hibernate提供了一个`EventListenerRegistry`接口，该接口提供了三种方法来注册自定义的事件监听器：

- `appendListeners()`：该方法有两个重载的版本，都用于将自定义的事件监听器追加到系统默认的事件监听器序列的后面；
- `prependListeners()`：该方法有两个重载的版本，都用于将自定义的事件监听器追加到系统默认的事件监听器序列的前面；
- `setListeners()`：该方法有两个重载的版本，都用于使用自定义的事件监听器代替系统默认的事件监听器序列。

	




































