## 概述 ##

- 不可变的对象是指一经创建就处于不变的状态，可以被安全共享的对象
	- 如果必需属性丢失就无法创建
	- 传递给其他代码时不会被偷偷修改
- 不可变对象天生就是线程安全的，因此可以被不同的线程共享
	- 不需要过度复制
	- 不需要过度同步
- 能很轻松愉快地读写对象的定义
	- 不需要公式化的setters和getters
	- 不需要在源代码中存储由IDE生成的蹩脚的`hashCode`，`equals`和`toString`方法。

## 关键概念 ##

### 抽象价值类型 ###

本文提到的抽象价值类型是由人编写的非final（往往是抽象）类或接口（甚至是注解类型），他们负责定义值的类型，使用`org.immutables.value.Value.Immutable`注解来标注。他们可以包含属性和其他元数据，例如正常的Java方法（和必要的字段）。强烈建议抽象价值类不要出现可见的可变状态。他们将做为源模型来生成代码。

### 特性（attribute） ###

特性保存了一个在所属对象创建后就无法更改的值。这里使用特性来与“字段”（field）或JavaBean的属性（property）进行区分，同时也显示了他与Java的注解特性的类似性。通过一个没有参数，非void返回类型的访问方法来定义一个特性。不需要在访问方法上使用注解来标注它定义了一个特性。但是某些特性，例如那些拥有默认值的特性使用非抽象方法来定义，方法体里计算它们的默认值，这些特性就需要使用特殊的注解来与其他一般方法区分来开。

### 不可变的实现类 ###

生成的final类扩展了人工编写的抽象价值类，并实现了所有的访问方法，同时也支持字段，一般方法，构造方法和相应的建造者类。不可变实现类为数值类型的基本数据类型以及引用类型的访问方法提供了实现，也提供了对集合以及其他类型的特殊支持。它重写了`java.lang.Object`的`equals`，`hashCode`和`toString`方法，这些方法的行为依赖于该类型的特性的值而不是对象的个体本身。

## 特点 ##

### 值 ###

注解处理器把被注解的抽象价值类型作为模型来生成不可变的实现类。产生的不可变类继承（抽象类型是抽象类）或实现（抽象类型是接口）了该抽象类型。如果类里不包含抽象的访问方法，则该类不必是抽象类。

		@Value.Immutable
		interface ValueInterface {}

		@Value.Immutable
		class ValueClass {}

		@Value.Immutable
		@interface ValueAnnotation {}

		...

		ValueInterface valueInterface = ImmutableValueInterface.builder().build();

		ValueClass valueClass = ImmutableValueClass.builder().build();

		ValueAnnotation valueAnnotation = ImmutableValueAnnotation.builder().build();

可以自定义生成的类的名字，使用`Immutable*`以外的其他前缀，也可以完全不使用前缀，具体参阅styles文档。

嵌套的抽象价值类型如果被声明为内部类，则必须显式声明为静态内部类。接口和注解自动隐式声明为静态类型。

可以使用其他包里的抽象价值类型来生成不可变的实现类。这需要使用`@Value.Include`注解（可以用在类和包上），它也可以搭配`@Value.Enclosing`注解一起使用。这在需要生成和DI类库，如Guice，一起使用的注解的实现时非常有用。

		package my.package;
		import java.lang.annotation.Retention;
		import java.lang.annotation.RetentionPolicy;
		import org.immutables.value.Value;

		@Value.Include({ Retention.class })
		interface IncludedAnnotations {}
		...

		ImmutableRetention.build()
			.value(RetentionPolicy.CLASS);
			.build();

### 建造者类 ###

每个不可变的实现类都会默认生成一个相应的建造者类。建造者类使用有名字的特性初始化方法来显式构造对象。要使用相应的建造者类，必须调用生成的不可变实现类的`builder()`方法来返回相应的建造者类实例，然后调用特性初始化方法来设置这些特性，最后再调用`build()`方法来返回初始化完毕的不可变类对象。如果略过了必需参数的初始化，则最后的`build()`方法不会成功。

		// builder methods illustrated
		ImmutableValue.builder()
			.foo(1442)
			.bar(true)
			.addBuz("a")
			.addBuz("1", "2")
			.addAllBuz(iterable)
			.putQux(key, value)
			.build();

可以自定义初始化方法，给他们加上类似`set`或`with`的前缀，或者通过构造方法来生成建造者类实例，以及添加`create`方法。具体参阅styles文档。

建造者类默认会包含一个`from`方法。该方法允许使用一个已存在的不可变类对象的特性值来生成并初始化一个新的不可变类对象。此方法可以用来提取多个值对象的特性值来进行计算。

		ImmutableValue.builder()
			.from(otherValue) // merges attribute value into builder
			.addBuz("b")
			.build();

在严格的建造者模式下，不会自动生成`from`方法。如果值对象从父类型中继承了抽象访问方法的定义，则它可以从父类型的实例或者其他父类型的实现类（也继承了该特性）的实例中复制该特性的值。自动生成的`from`方法会针对每种允许复制特性值的父类型生成相应的重载方法。注意这种从父类型复制特性值的方式不能用于参数化的泛型类性以及使用协变返回类型（covariant)重写了访问方法的子类型。

如果使用构造方法而不是建造者模式来构建对象，则可以使用`@Value.Immutable(builder = false)`注解来取消生成建造者类。

如果需要在某些高级应用中，需要让建造者返回同一接口的不同实现类的实例，则可以声明一个静态嵌套类并命名为`Builder`，这个类将被自动生成的建造者类继承。因此可以让这个`Builder`类的返回类型为抽象的接口类型，从而实现多态。

		interface Vehicle {
		}

		interface VehicleBuilder {
		  // Generated builders will implement this method
		  // It is compatible with signature of generated builder methods where
		  // return type is narrowed to Scooter or Automobile
		  Vehicle build();
		}

		@Value.Immutable
		public abstract class Scooter implements Vehicle {
		  public abstract static class Builder implements VehicleBuilder {}
		}

		@Value.Immutable
		public abstract class Automobile implements Vehicle {
		  public abstract static class Builder implements VehicleBuilder {}
		}

		class Builders {
		  void buildIt(VehicleBuilder builder) {
			Vehicle vehicle = builder.build();
		  }

		  void use() {
			buildIt(ImmutableScooter.builder());
			buildIt(ImmutableAutomobile.builder());
		  }
		}

显式声明的抽象建造者类可以指定所需继承的类或实现的接口，以及包含一些便利方法，这些都将出现在最后生成的建造者实现类中。但要留意维持声明了建造者类的父类型和所生成建造者类在结构上的兼容性。

使用转调工厂方法和抽象建造者类，可以将生成的实现类型以及它的建造者类从API中隐藏起来。

另一种结构上的自定义方式是通过指定`@Value.Style(visibility = ImplementationVisibility.PRIVATE)`来生成一个私有的不可变实现类，该类隐藏在生成的在同一包内的独立顶层建造者类中。

		@Value.Immutable
		@Value.Style(visibility = ImplementationVisibility.PRIVATE)
		public interface Person {
		  String getName();
		  String getAddress();
		}

		Person person = new PersonBuilder()
		  .name("Jim Boe")
		  .address("P.O. box 0001, Lexington, KY")
		  .build();

从2.0.17版本开始，可以使用如下代码来扩展“还未生成”的建造者类。

		@Value.Style(visibility = ImplementationVisibility.PACKAGE, overshadowImplementation = true)
		//...

		@Value.Immutable
		public interface Person {
		  String name();
		  String address();
		  // static inner class Builder extends generated or yet to be generated Builder
		  class Builder extends ImmutablePerson.Builder {}
		}

		Person person = new Person.Builder()
		  .name("Jim Boe")
		  .address("P.O. box 0000, Lexington, KY")
		  .build();

虽然`ImmutablePerson`（以及相应的`ImmutablePerson.Builder`）不能被包外访问，但`Person.Builder`继承并暴露了在`ImmutablePerson.Builder`中定义的所有公共方法。`overshadowImplementation = ture`风格特性确保`build()`方法将被声明返回抽象价值类`Person`，而不是实现类`ImmutablePerson`。有趣的事实是，调用方法的字节码只是引用`Person.Builder`，而不是`ImmutablePerson.Builder`。在上个例子中`INVOKEVIRTUAL`的目标是`Perosn.Builder.name`，`Person.Builder.address`和`Person.Builder.build()`。关键的是，生成的类成为了没有太多公式化代码的实现细节，而这些细节是把实现完全隐藏在用户手写代码之后所需要的。

### 严格的建造者类 ###

通过设定风格参数`@Value.Style(strictBuilder = true, ...))`，可以生成严格的建造者类。在这种模式下，只能向前初始化，换句话说，对于集合特性，只能执行增添操作，而普通特性则只能设置一次。严格建造者模式默认是关闭的，因此默认的建造者类是允许重新设置已初始化的特性的值的。

		@Value.Immutable
		@Value.Style(strictBuilder = true)
		interface Aar {
		  boolean a();
		  Integer b();
		}

		ImmutableAar.builder()
			.a(true)
			.b(1)
			.b(2) // IllegalStateException will be thrown here, 'b' cannot be reinitialized
			.build();

严格建造者模式下，不会为集合特性提供重置方法，也不会生成`from`方法。

注意不推荐直接在抽象价值类型上使用`@Value.Style`。`Style`注解的推荐用法是创建元注解，具体参阅styles文档。

### 阶段建造者 ###

在2.3.2版本中引进了阶段建造者模式。该模式通过设置`@Value.Style(stagedBuilder = true)`来开启。阶段建造者提供了编译期间的安全保障：确保所有必需的特性都要完成初始化。API包含了阶段接口，这些接口强制一步步地初始化所有的必需的特性，这些初始化操作可以通过代码自动补全来完成，如此以来，在完成所有必需特性初始化之前是无法调用`build()`方法的，从而确保最终的`build()`方法不会因为忘记初始化某个必需特性而抛出`IllegalStateException`异常。所有剩余的可选的，可为空的以及集合类型的特性的初始化都可以在最终阶段以任意顺序完成。如何对必需特性声明的修改，包括添加增减新特性，改变特性的顺序都会造成编译出错。

		@Value.Style(stagedBuilder = true)
		@Value.Immutable
		public interface Person {
			String name();
			int age();
			boolean isEmployed();
		}
		...
		ImmutablePerson.builder()
			.name("Billy Bounce")
			.age(33)
			.isEmployed(false)
			.build();
		...
		// under the hood
		public final class ImmutablePerson implements Person {
		  ...
		  public static NameBuildStage builder() { ... }
		  public interface NameBuildStage { AgeBuildStage name(String name); }
		  public interface AgeBuildStage { IsEmployedBuildStage age(int age); }
		  public interface IsEmployedBuildStage { BuildFinal isEmployed(boolean isEmployed); }
		  public interface BuildFinal { ImmutablePerson build(); }
		}

提高编译期安全性的代价是新增大量的接口，每个需要的特性都有一个接口相对应。阶段建造者模式同时也隐式遵守严格建造者规则。

### 构造方法 ###

作为建造者类的替代，可以提供简易的构造工厂方法。构造工厂方法在生成的不可变实现类中，以名为`of`的静态方法的形式出现。

把需要传递给构造工厂方法作为参数的特性用`org.immutables.value.Value.Parameter`注解标注起来。

		@Value.Immutable
		public abstract class HostWithPort {
		  @Value.Parameter
		  public abstract String hostname();
		  @Value.Parameter
		  public abstract int port();
		}
		...
		HostWithPort hostWithPort = ImmutableHostWithPort.of("localhost", 8081);

		boolean willBeTrue = hostWithPort.equals(
			ImmutableHostWithPort.builder()
				.hostname("localhost")
				.port(8081)
				.build());

可以通过设定`order`注解来指定特性作为参数传递给构造工厂方法的顺序。目前该功能对`javac`和Eclipse JDT编译器有效。

		@Value.Immutable
		public abstract class HostWithPort {
		  @Value.Parameter(order = 2)
		  public abstract int port();
		  @Value.Parameter(order = 1)
		  public abstract String hostname();
		}
		...
		HostWithPort hostWithPort = ImmutableHostWithPort.of("localhost", 8081);

如果需要自动把所有特性都作为构造工厂方法的参数，可以通过设定风格来实现，具体参阅元组风格模式。

#### 传统的公共构造方法 ####

如果需要传统的公共构造方法，而不是静态工厂方法，可以通过设置风格将`of`方法重命名为`new`方法。

		@Value.Style(
		  of = "new", // renames "of" method to "new", which is interpreted as plain constructor
		  allParameters = true // unrelated to the line above: every attribute becomes parameter
		  // reminder: don't get used to inline styles, read style guide!
		)
		@Value.Immutable
		public interface HostWithPort {
		  String host();
		  int port();
		}

		HostWithPort hostWithPort = new ImmutableHostWithPort("localhost", 8081);

需要注意：如果没有把所有的必需特性用`@Value.Parameter`注解标注，则编译器会报错：

		Compilation error: could not generate default value for mandatory field.

### 数组，集合和Map特性 ###

Immutables支持以下集合类型：

		T[]

		java.util.List<T>

		java.util.Set<T>

		java.util.Map<K, V>

		com.google.common.collect.Multiset<T>

		com.google.common.collect.Multimap<K, V> (ListMultimap, SetMultimap)

		com.google.common.collect.BiMap<K, V>

		com.google.common.collect.Immutable* variants for collections above

数组特性通过克隆保障安全性（Java数组是可变的）。如果类库路径中包含了Guava框架，则集合特性使用Guava的不可变集合类型。否则，他们将被安全地复制并封装在标准JDK库中不可更改的集合类中。

使用枚举类型作为键值类型的`java.util.Set`和`java.util.Map`将由更有效的`EnumSet`和`EnumMap`替代。

自然顺序或反序的maps和sets需要使用`Value.NaturalOrder`和`@Value.ReverseOrder`注解来标注。

		java.util.SortedSet<T>

		java.util.NavigableSet<T>

		java.util.SortedMap<K, V>

		java.util.NavigableMap<K, V>

如果没有使用排序注解，有序sets和maps特性将作为普通特性生成以支持使用自定义的比较器来构造对象。

所有上面提及的集合类型都可以使用Guava框架中相应的不可变集合类型`com.google.common.collect.Immutable*`来代替。注意其他的集合实现类型如`java.util.ArrayList`不会作为特殊集合特性被处理。

通过建造者类构建集合特性时，可以不用指定它的内容，但可以设置验证条件，例如使用前置条件检查方法来确保他们包含指定数量的元素。

建造者类具有特殊的方法来初始化集合特性：

- 对于名为`foo`的，元素类型为`T`的数组特性

		foo(T...)

- 对于元素类型为`T`的名为`foo`的`List`，`Set`，或`Multiset`特性：

		foo(Iterable<? extends T>) — not available in strict mode
		addFoo(T)
		addFoo(T...)
		addAllFoo(Iterable<? extends T>)

- 对于键值类型为`K`，值类型为`V`的名为`bar`的`Map`或`BiMap`特性：

		bar(Map<? extends K, ? extends V>) — not available in strict mode
		putBar(K, V)
		putBar(Map.Entry<? extends K, ? extends V>)
		putAllBar(Map<? extends K, ? extends V>)

- 对于名为`bar`的`Multimap`特性：

		bar(Multimap<? extends K, ? extends V>) — not available in strict mode
		putBar(K, V)
		putBar(Map.Entry<? extends K, ? extends V>)
		putBar(K, ...V)
		putAllBar(K, Iterable<V>)
		putAllBar(Multimap<? extends K, ? extends V>)

从版本0.16开始，不再为建造者类生成`clear*`方法，所以不会为集合特性生成`clearFoo`和`clearBar`方法。如果需要清除集合的内容，使用一个重置方法`bar(Collections.emptyList())`，或者在实例被创建后立刻使用复制方法。

从版本2.1.11开始，可以使用可选的去复数化功能来为名为`foos`的集合属性生成名为`addFoo`或`putFoo`的方法。

自动生成的方法集是为了能方便使用而精心挑选的。但也可以使用如ProGuard之类的工具排查可能无用的方法。

其他的容器类型如`java.util.Collection`或`java.util.Queue`等都由于过于宽泛或过于特定而不适用与不可变对象建模。但从积极的方面看，所有的类型都可以作为特性的类型来使用。

		@Value.Immutable
		public abstract class DoItYourselfContainer {
		   public abstract Iterable<String> iterable();
		}
		...
		ImmutableDoItYourselfContainer.builder()
			.iterable(ImmutableSet.of("a", "b", "c"))
			.build();

### `Optional`特性 ###

声明类型为`com.google.common.base.Optional<T>`的特性定义了一个类型为`T`的optional特性。

从版本2.0开始也支持Java 8的`java.util.Optional`，`java.util.OptionalInt`，`java.util.OptionalLong`和`java.util.OptionalDouble`。

从版本2.1.1开始也支持`com.atlassian.fugue.Option`和`io.atlassian.fugue.Option`。

构建对象时可以跳过可选的值，它们将默认使用`Optional.absent()`（或Java 8的`Optional.empty()`）来初始化。生成的建造者类针对可选特性有专用的初始化方法：

对于元素类型为`T`，名为`opt`的可选特性：

- `opt(T)`设置当前值
- `opt(Optional<T>)`设置当前值（可以为空值）

		import java.util.*;

		@Value.Immutable
		public interface AllOptional {
		  com.google.common.base.Optional<Integer> v1();
		  Optional<Integer> v2();
		  OptionalInt i1();
		  OptionalLong l1();
		  OptionalDouble d1();
		}
		...
		// No error as all values are optional
		ImmutableAllOptional.builder().build();

		ImmutableAllOptional.builder()
			.v1(1)
			.v2(2)
			.i1(1)
			.l1(1L)
			.d1(1.0)
			.build();

### 特性默认值 ###

可以为特性设置默认值：创建一个该特性的非抽象的初始化方法并使用`org.immutables.value.Value.Default`注解标注该方法。如果在构建过程中没有指定该特性的值，就会使用这个方法来计算它的默认值。

		@Value.Immutable public abstract class PlayerInfo {

		  @Value.Parameter
		  public abstract long id();

		  @Value.Default
		  public String name() {
			return "Anonymous_" + id();
		  }

		  @Value.Default
		  public int gamesPlayed() {
			return 0;
		  }
		}
		...

		PlayerInfo veteran = ImmutablePlayerInfo.builder()
			.id(1)
			.name("Fiddler")
			.gamesPlayed(99)
			.build();

		PlayerInfo anonymous44 = ImmutablePlayerInfo.of(44);

		String name = anonymous44.name(); // Anonymous_44

从版本2.1开始，特性的默认初始化方法可以调用其他默认或衍生特性的方法，前提是不会造成初始化环路，否则会在初始化这些特性时抛出`IllegalStateException`异常。

不需要为集合特性设置默认值，因为如果没有显式初始化集合特性，则会自动返回一个空的集合。从版本2.2开始，集合特性也可以使用`@Value.Default`和`@Nullable`注解来标注，这些集合可以在没有显式初始化的情况下返回提供的默认集合或`null`值。

不可变的注解类型中，带有默认值的特性通过关键字`default`来设置相应的默认值。

带有默认值的特性也可以和Java8中接口的默认方法配合使用，不过特性需要用`@Default`注解来标注。

		@Value.Immutable
		interface Val {
		  int anAttribute();
		  @Value.Default default int otherAttribute() {
			return 0;
		  }
		}

### 衍生特性 ###

衍生特性的值不能手工设置，只能从已存在的不可变实例中读取出来。

创建衍生特性的方法：创建一个非抽象的特性初始化方法，并用`org.immutables.value.Value.Derived`注解来标注它。和默认值特性类似，方法体内部都应该计算并返回特性的值。衍生特性和能够计算并返回值的普通方法类似，最重要的区别是：衍生特性的值只会被计算并存储一次（在创建对象的最终阶段）。

		@Value.Immutable
		public abstract class Order {

		  public abstract List<Item> items();

		  @Value.Derived
		  public int totalCount() {
			int count = 0;

			for (Item i : items())
			  count += i.count();

			return count;
		  }
		}

		Order order = ImmutableOrder.builder()
			.addItems(Item.of("item1", 11))
			.addItems(Item.of("item2", 22))
			.build();

		// total count will be already computed
		int totalCount33 = order.totalCount();

和具有默认值特性的特性类似，衍生特性也可以调用其他的默认值特性或衍生特性，但同样需要注意不能形成初始化环路。

### 可空特性 ###

通常不推荐使用可空的特性。如果真的需要，在抽象的特性访问方法上使用`@Nullable`注解来标注。可空特性不需要使用建造者类来设置值，也可以使用`null`来初始化他们。不支持可空集合和其他特殊类型

		@Value.Immutable
		interface NullAccepted {
		  @Nullable Integer i1();
		  @Nullable Long l2();
		}

		NullAccepted obj = ImmutableNullAccepted.builder()
			.i1(null)
			.build();

		obj.toString(); // NullAccepted{i1=null, l2=null}

#### 集合中的空值 ####

集合本身及其元素都不应该为空值。但为了和第三方类库或服务兼容，可能需要允许或跳过空值。集合特性本身可以被标注为`Nullable`，也可使用`@AllowNulls`或`@SkipNulls`注解来标注其元素。注解Guava的不可变集合类型不支持空值，所以这个功能只能在使用标准JDK库里的集合类型时有效。

		@Value.Style(jdkOnly = true)
		@Value.Immutable
		public interface NullElements {
		  // collection elements
		  @AllowNulls List<Void> al();
		  @SkipNulls List<String> sk();
		  // map values (but not keys)
		  @AllowNulls Map<String, Integer> bl();
		  @SkipNulls Map<String, Integer> sm();
		}

根据使用的编译器的不同（ECJ和ErrorProne有效，标准Javac无效），`@Nullable`，`@AllowNulls`和`@SkipNulls`可以作为Java 8中的类型注解来使用，如`List<@Nullable obj>`。

### 延迟特性 ###

延迟特性是指它的初始化方法会延迟计算它的值，但只计算一次。

声明一个延迟特性，需要创建一个非抽象的特细初始化方法，并使用`org.immutables.value.Value.Lazy`注解标注它。和衍生特性类似，方法体内应该计算并返回特性的值。延迟特性的值只在第一次访问该特性时会被计算，在以后的访问中将返回相同的值（已被记录），而不再重新计算。

需要注意的是：延迟特性不会被用于`equals`方法和`hashCode`方法的实现。

		@Value.Immutable
		public abstract class Order {

		  public abstract List<Item> items();

		  @Value.Lazy
		  public int totalCost() {
			int cost = 0;

			for (Item i : items())
			  cost += i.count() * i.price();

			return cost;
		  }
		}

		Order order = ImmutableOrder.builder()
			.addItems(Item.of("item1", 11, 1))
			.addItems(Item.of("item2", 22, 2))
			.build();

		// total cost will be computed now
		int lazilyComputedCost = order.totalCost();
		// total cost is already computed and stored value is returned
		lazilyComputedCost = order.totalCost();

延迟特性的值是线程安全的，会被且仅会被计算一次。

与默认和衍生特性不同，延迟特性的访问方法的方法体可以引用任意特性，如果在初始化默认或衍生特性的过程中调用了延迟特性，它将被立刻初始化，此时它与衍生特性完全一样。

当前延迟特性的实现采用了和老版本的Scala非常相似的方式，所以可能会遭遇同样的问题，具体参阅`Scala SIP-20`。但只要避免把不可变对象和可变的/静态的/本地线程的状态混杂在一起（即在不同不可变对象之间出现了依赖环路），就不会产生问题。

### 前置条件检查方法 ###

不可变对象的一个核心优点是使用合理特性创建的不可变对象将处于持久化状态，并且再也不会被改变。但是偶尔也会需要对这些特性的值单独或组合在一起做一些检查来保证它们的正确性（比有效性更进一步）。通常会在手写的类的构造方法中执行这些检查，但由于在生成的不可变实现类中没有任何手写代码，所以必须在一个被`@Value.Check`注解标注的非私有方法中执行这些检查。

		@Value.Immutable
		public abstract class NumberContainer {
		  public abstract List<Number> nonEmptyNumbers();

		  @Value.Check
		  protected void check() {
			Preconditions.checkState(!nonEmptyNumbers().isEmpty(),
				"'nonEmptyNumbers' should have at least one number");
		  }
		}
		...
		// will throw IllegalStateException("'nonEmptyNumbers' should have at least one number")
		ImmutableNumberContainer.builder().build();

但是应该注意这种检查和其他类型的对象状态检验不同。其他类型的对象状态检验是指使用一些值创建了对象，并在稍后根据所处场景的商业规则来检验对象的正确性。前置条件检查不应该针对这些规则进行检验，而是应该用于维持对象的持久性并确保这些实例处于可用状态。

前置条件检查方法的执行应该在已创建了不可变对象的实例，所有特性也已被初始化，但还未将对象返回给调用者的时候。一旦检查失败，将会抛出异常，而所生成的对象也不会被返回给调用者。

#### 标准化处理 ####

`@Value.Check`有另外一个用途。当校验方法声明的返回类型是抽象价值类型时，该校验方法也可以返回一个经过标准化处理的替代实例。该实例的类型总是不可变的实现类，否则会在构建过程中抛出`ClassCastException`异常。注意如果没有执行正确的检查或提供冲突检查，就很容易引起无法解决的递归。如果值不需要被标准化处理，那么就应该总是返回`this`。

		@Value.Immutable
		public interface Normalized {
		  int value();

		  @Value.Check
		  default Normalized normalize() {
			if (value() == Integer.MIN_VALUE) {
			  return ImmutableNormalized.builder()
				  .value(0)
				  .build();
			}
			if (value() < 0) {
			  return ImmutableNormalized.builder()
				  .value(-value())
				  .build();
			}
			//前两种值，即value为0或负数的情况都被标准化处理了，正数不需要被处理，所以返回this
			return this;
		  }
		}

		int shouldBePositive2 = ImmutableNormalized.builder()
			.value(-2)
			.build()
			.value();

### 复制方法 ###

`with*`方法允许在其他特性保持不变的情况下，为特定特性指定新的值，并以此生成新的不可变对象返回。

		counter = counter.withValue(counter.value() + 1)

一个简易的引用等价检查`==`被加入在实现中，通过返回`this`防止复制相同值。基本数据类型通过`==`来检查等价性。基本数据类型中的`float`和`double`通过使用严格的`Float.floatToIntBits`和`Double.doubleToLongBits`来做等价检查。字符串和基本数据类型的封装类使用`Object.equals`方法来检查等价。但通常会跳过完全的等价检查，因为在实践中，创建一个值的复本的成本可能还要低于对它做深度等价检查的成本。

这些“wither”方法的实现使用了结构共享来完成复制。新生成的对象除了提供了新值的特性外，其他特性的值都维持不变，包括所有的不可变集合及其内部的值（重建开销比较大）。

目前，复制方法设置任意类型特性的新值的操作仅仅是简单的值替换。集合类型的新值一定是不可变的。

		Value changedValue =
			ImmutableValue.copyOf(existingValue)
				.withName("Changed Name")
				.withValues(Arrays.asList("Only new value")) // replacing any copied collection

复制方法默认是自动生成的，但必须基于不可变实现类的引用来使用它们，作为父类型的抽象价值类的引用无法调用它们。

可以通过设置`@Value.Immutable(copy = false)`注解来关闭自动生成复制方法。

### 单例类实例 ###

使用`@Value.Immutable(singleton = true)`注解来生成单例类，使用`of()`工厂方法来取得单例类实例。

		@Value.Immutable(singleton = true)
		public abstract class Data {
		  public abstract Set<String> chunks();
		}

		...
		boolean willBeTrue =
			ImmuableData.of() == ImmuableData.of();
		// true

		boolean willBeTrueAlso =
			ImmuableData.of().chunks().isEmpty();
		// true

单例类的抽象价值类型不应该包含任何主要特性，否则不可能生成单例类。可以使用默认或可选特性来代替主要特性。

如果所有的特性都是非强制性的，则空单例类实例可以和建造者类以及构造方法组合在一起。如果只想提供单例类实例，可以禁止建造者类和构造方法：

- 和`@Value.Immutable`注解一起使用`singleton = true`和`builder = false`；
- 避免使用`@Value.Parameter`，防止生成构造方法。

		@Value.Immutable(singleton = true, builder = false)
		public abstract class Singleton {
		  // Limit constructor accessibility to a package if needed.
		  // Private access will not work as ImmutableSingleton should
		  // be able to extend Singleton
		  Singleton() {}
		}

		...
		Singleton singleInstance = ImmutableSingleton.of();

注意如果抽象价值类型包含主要特性，同时又引入单例类注解，会造成编译期出错：`Compilation error: could not generate default value for mandatory field`

### 实例驻留（instance interning） ###

实例驻留是指让等价的两个实例引用同一个对象，而不是创建2个完全等价却不相等的对象。

使用`@Value.Immutable(intern = true)`注解来开启强驻留。

- 建造者类和构造方法返回的对象都会被驻留，一个规范化的实例会被返回；
- `equals`方法等效于`==`操作符。

默认支持强驻留。从版本2.6.0开始，也可以通过设置`@Value.Style(weakInterning = true)`来开启支持弱驻留。

其他形式的驻留需要在外部实现。但是，有一个模块`org.immutables:ordinal`支持在类似枚举类型的对象上实现基于域的驻留。具体参阅它的文档。

### 提前计算的hashCode ###

如果不可变类拥有很多特性，或者特性具有相当大的对象图，则哈希码的计算的开销会很大，此时应该采取提前计算并缓存哈希码的方法来减少操作开支。

使用`@Value.Immutable(prehash = true)`注解开启提前计算哈希码功能。

### 保密特性（redacted attributes） ###

从版本v2.5.0开始，可以使用`@Value.Redacted`注解来在自动生成的`toString`方法的输出中隐藏或掩盖特性的值。默认是不显示特性的值，但也可以通过`@Value.Style(redactedMask = "####")`注解来指定用于覆盖该特性值的字符（可以任意指定）。

		@Value.Style(redactedMask = "####")
		..
		@Value.Immutable
		public interface RedactedMask {
		  @Value.Redacted
		  String ssn();
		  @Value.Redacted
		  String secret();
		}
		// toString output: RedactedMask{ssn=####, secret=####}
		// without setting style it would be just: RedactedMask{}

可以使用更灵活，功能更强大，同时也保证类型安全的不透明容器来达到同样的目的。


### 次要特性 ###

使用`@Value.Auxiliary`标注次要特性。这些特性不会被用于`equals`，`hashCode`和`toString`方法。延迟特性总是作为次要特性处理。

		@Value.Immutable(intern = true)
		interface TypeDescriptor {
		  // Instances are interned only by qualified name
		  String qualifiedName();
		  @Value.Auxiliary
		  TypeElement element();
		}

`equals`，`hashCode`和`toString`方法也可以分开处理。

### 自定义`equals`，`hashCode`和`toString` ###

如果用户重写了这些方法，则`Immutables`处理器直接使用这些重写的方法。

		@Value.Immutable
		public abstract class OmniValue {
		  @Override
		  public boolean equals(Object object) {
			return object instanceof OmniValue;
		  }

		  @Override
		  public int hashCode() {
			return 1;
		  }

		  @Override
		  public String toString() {
			return "OmniValue{*}";
		  }
		}

		boolean willBeTrue =
			ImmutableOmniValue.builder().build()
				.equals(new OmniValue() {});

另外，人工编写的`equals`和`hashCode`方法会自动和实例驻留以及提前计算哈希码功能一起工作。

### 不可变的注解 ###

注解类型也可以用`@Value.Immutable`来标注。其使用和普通的不可变对象一样。

		@Value.Immutable
		public @interface MyAnnotation {
		  String[] value();
		  boolean enable() default true;
		}
		...
		ImmutableMyAnnotation.builder()
		  .addValue("a", "b")
		  .enable(true)
		  .build();

### 序列化 ###

通过以下方式提供对基本的Java二进制序列化的支持：

- 检查抽象价值类型是否实现了`java.lang.Serializable`接口
- 使用`transient`修饰延迟特性
- 把抽象价值类型的`serialVersionUID`复制到不可变的实现类
- 生成`readResolve`方法实现来维护单例类和驻留的实例。

`serial`模块提供了更高级的Java二进制序列化注解。

- `org.immutables:serial:2.7.4`
- `@Serial.Version` - 为包装类型提供序列版本
- `@Serial.Structural` - 开启特殊的结构性序列化。具体参阅模块文档
- 使用`@Serial.Version`或`@Serial.Structural`标注的抽象类型的实现类会自动实现`java.lang.Serializable`接口。

JSON的序列化参阅JSON文档。

### 可更改的类 ###

使用`@Value.Modifiable`注解（可选择是否与`@Value.Immutable`一起使用）标注的抽象价值类型会生成一个前缀为`Modifiable`的可变伙伴类。该伙伴类可作为缓存，uber-builder，或初始化了一部分的实现类来使用。这个伙伴类可以作为更好的替代方案替代以下做法：

- 以查找特性和`isSet/isInitialized`方法来污染建造者类；
- 引入危险的`buildPartial()`方法来制造未完成的不可变实例。

		@Value.Immutable
		@Value.Modifiable
		interface Item {
		  String getName();
		  List<Integer> getCount();
		}
		...

		// When simple workflow of regular Builder is not enough

		ModifiableItem item = ModifiableItem.create()
			.setName("Super Item")
			.addCount(1)
			.addCount(2);

		item.getCount().add(3);

		if (item.isIntialized()) {
		  ImmutableItem immutableItem = item.toImmutable();
		  System.out.println(immutableItem);
		  // Item{name=Super Item, count=[1, 2, 3]}
		}

		item.clear()
			.from(ImmutableItem.builder().name("First").addCount(4, 5).build())
			.from(ImmutableItem.builder().name("Second").addCount(6).build());

		System.out.println(item);
		// ModifiableItem{name=Second, count=[4, 5, 6]}

可更改类的类名可以通过使用style来设置。

### 全面支持泛型 ###

从版本2.2开始支持泛型参数，也可以指定泛型的上限（upper）通配符。

		interface TreeElement<T> {}

		@Value.Immutable
		interface Node<T extends Serializable> extends TreeElement<T> {
		  List<TreeElement<T>> elements();
		}

		@Value.Immutable
		interface Leaf<T extends Serializable> extends TreeElement<T> {
		  @Value.Parameter T value();
		}

		TreeElement<String> tree =
			ImmutableNode.<String>builder()
				.addElements(ImmutableLeaf.of("A"))
				.addElements(ImmutableLeaf.of("B"))
				.addElements(
					ImmutableNode.<String>builder()
						.addElements(ImmutableLeaf.of("C"))
						.addElements(ImmutableLeaf.of("D"))
						.build())
				.build();

### 注解注入 ###

在版本2.6.0中加入了实验性质的注解注入功能：可以在字段，访问方法，初始化方法，不可变类及建造者类等元素上注入注解。具体参阅`org.immutables:annotation`模块。

### 警告 ###

`Immutables`注解处理器在发现有错误隐患或不鼓励用法的时候会发出错误提示或警告。例如，在基本数据类型上使用`@Nullable`注解，或让一个`Value.Immutable`类型扩展另一个`Value.Immutable`类型，会提示错误；缺乏，多余，忽略或者不鼓励的注解使用会引发警告。这些都可以通过设置`SuppressWarnings("immutables")`或`SuppressWarnings("all")`来关闭。参阅`Style.generateSuppressAllWarnings`来了解使用风格特性来调整警告级别。

## 模式 ##

这里描述的是一些`Immutables`常用的模式和技巧。

### 封装类类型 ###

经常会为基本数据类型，字符串，和某些常见的类型创建封装类来根本性地提高类型的安全性。但是也需要最小化由之引起的语句开销。

使用一个父类型和相应的风格来描述封装类型。

		// declare style as meta annotation as shown
		// or on package/top-level class
		// This is just an example, adapt to your taste however you like
		@Value.Style(
			// Detect names starting with underscore
				typeAbstract = "_*",
			// Generate without any suffix, just raw detected name
				typeImmutable = "*",
			// Make generated public, leave underscored as package private
				visibility = ImplementationVisibility.PUBLIC,
			// Seems unnecessary to have builder or superfluous copy method
				defaults = @Value.Immutable(builder = false, copy = false))
		public @interface Wrapped {}

		// base wrapper type
		abstract class Wrapper<T> {
		  @Value.Parameter
		  public abstract T value();
		  @Override
		  public String toString() {
			return getClass().getSimpleName() + "(" + value() + ")";
		  }
		}

		...
		// Declare wrapper types/domain values

		@Value.Immutable @Wrapped
		abstract class _LongId extends Wrapper<Long> {}

		@Value.Immutable @Wrapped
		abstract class _PersonName extends Wrapper<String> {}

		@Value.Immutable @Wrapped
		abstract class _VehicleMake extends Wrapper<String> {}

		...
		// Enjoy your wrapper value types

		LongId id = LongId.of(123L);

		PersonName name = PersonName.of("Vasilij Pupkin");

		VehicleMake make = VehicleMake.of("Honda");

如果使用了自定义不可变注解中的技巧，则可以只使用`@Wrapped`注解，而不再需要`@Value.Immutable`注解。

### 元组风格 ###

这种风格会生成只使用构造方法的类型，其构造方法会把所有的特性作为参数传入。它的关键字`allParameters`，会把所有特性看做构造方法的参数，而不管他们是否被`@Value.Parameter`注解标注。

		@Value.Style(
			// Generate construction method using all attributes as parameters
			allParameters = true,
			// Changing generated name just for fun
			typeImmutable = "*Tuple",
			// We may also disable builder
			defaults = @Value.Immutable(builder = false))
		public @interface Tuple {}
		...
		// declare type with "tuple" style
		@Value.Immutable @Tuple
		public interface Complex {
		  double re();
		  double im();
		}
		...

		Complex c = ComplexTuple.of(1d, 0d);

如果使用了自定义不可变注解中的技巧，则可以只使用`@Tuple`注解，而不再需要`@Value.Immutable`注解。

### 在深层不可变类探测的帮助下，封装类型，元组类型的初始化方法可以作为另类的设置方法被隐式调用（真绕口，看下面的代码更容易理解一些） ###

当封装类型（即单个）和元组类型（即多个）价值对象在另一个`@Immutable`类型中被同时使用。可以在创建外部包装对象时，使用如下代码所示的快捷方式来避免显式构建被包装的内部价值对象。

个人理解：某个抽象价值类型应用了元组风格（意味着只有构造方法），而它包含的特性又使用了封装类型（在此例中就是`int`对应的`Integer`）。如果在另外一个不可变类内部使用了此类型，则在创建外部类的对象（例如调用建造者类的设置方法）时会自动隐式地调用该类型的构造方法来避免显式构建内部类对象。

		@Value.Immutable
		@Value.Style(deepImmutablesDetection = true, depluralize = true)
		public interface Line {
		  List<Point> points();
		}

		@Value.Immutable
		@Value.Style(allParameters = true)
		public interface Point {
		  int x();
		  int y();
		}

		ImmutableLine line = ImmutableLine.builder()
		  .addPoint(1, 2) // implicit addPoint(ImmutablePoint.of(1, 2))
		  .addPoint(4, 5)
		  .build();
		}

### 表述性的工厂方法 ###

有些可能需要自定义构造方法的名字以及提供构造钩子。下面就是2个互相关联的需要：

- 使用工厂方法参数值的衍生值（不同于参数值）来构建对象；
- 为工厂方法起一个可以清晰表达如何使用参数创建对象的描述性名字。

实现这个功能不需要任何特殊的注解或方法。只需在抽象价值类型中简单地声明一个新的具有描述性名字的工厂方法，然后在内部转调自动生成的不可变实现类的构造方法即可。

		@Value.Immutable
		public abstract class Point {
		  @Value.Parameter
		  public abstract double x();
		  @Value.Parameter
		  public abstract double y();

		  public static Point origin() {
			return ImmutablePoint.of(0, 0);
		  }

		  public static Point of(double x, double y) {
			return ImmutablePoint.of(x, y);
		  }

		  public static Point fromPolar(double r, double t) {
			return ImmutablePoint.of(r * Math.cos(t), r * Math.sin(t));
		  }
		}

注意在上面的例子中，通过转调工厂方法，没有将实现类`ImmutablePoint`暴露在抽象价值类型`Point`的API中，从而隐藏了实现细节。


### 隐藏实现类 ###

在上面例子的基础上，还可以使用内嵌的抽象建造者类来隐藏建造者类的实现。

		// Make generated class package private
		@Value.Style(visibility = ImplementationVisibility.PACKAGE)
		@Value.Immutable
		public abstract class Point {
		  @Value.Parameter public abstract double x();
		  @Value.Parameter public abstract double y();

		  public static Point of(double x, double y) {
			return ImmutablePoint.of(x, y);
		  }

		  public static Builder builder() {
			return ImmutablePoint.builder();
		  }
		  // Signatures of abstract methods should match to be
		  // overridden by implementation builder
		  public interface Builder {
			Builder x(double x);
			Builder y(double y);
			Point build();
		  }
		}

### 智能数据 ###

不可变对象非常适合承担“智能数据”的角色。除了作为单纯的数据容器之外，一个价值类对象还可以拥有一些和领域相关的知识和执行计算的能力。

		@Value.Immutable
		public abstract class OriginDestination {
		  @Value.Parameter
		  public abstract Airport origin();
		  @Value.Parameter
		  public abstract Airport destination();

		  public boolean isDomestic() {
			return origin().country().equals(destination().country());
		  }

		  public boolean isCrossCityTransit() {
			return origin().city().equals(destination().city());
		  }

		  public OriginDestination reverse() {
			return ImmutableOriginDestination.of(destination(), origin());
		  }
		  ...
		}

### 非公共的特性 ###

一些特定的特性可能不需要暴露在抽象价值类型的API中。调降他们的的可视范围可以将这些特性隐藏在API用户的视线之外，但它们仍然会在建造者类中以`public`的形式出现，或者作为构造方法的参数暴露出来。

		@Value.Immutable
		public abstract class Name {
		  @Value.Parameter
		  abstract String value();

		  public String toString() {
			return value();
		  }

		  public static Name of(String value) {
			return ImmutableName.of(value);
		  }
		}
		...

		Name name = Name.of("The Rose");
		String value = name.toString();
		// "The Rose"

### 特性值中的空-对象模式 ###

作为另一种使用`Optional<T>`特性的方式，可以使用空-对象模式。这不需要`Immutables`的特殊支持，使用默认特性就可以实现。

		public enum Stars {
		  NONE, ONE, TWO, THREE, FOUR, FIVE;
		}

		@Value.Immutable
		public abstract class Hotel {
		  @Value.Default
		  public Stars stars() {
			return Stars.NONE;
		  }
		}

### 不透明容器 ###

有时候可能希望阻止某些字段出现在`toString`方法的返回结果中，或者不希望在`hashCode`或`equals`中使用它们。那么可以为这些字段创建一个不透明的封装类来满足这些要求。

例如想避免`toString`方法暴露某些机密数据，可以为其创建一个不透明的封装类：

		@Value.Immutable(builder = false)
		abstract class Confidential {
		  @Value.Parameter
		  abstract String value();
		  public String toString() { return "<NON DISCLOSED>"; }
		}

然后，将创建的封装类作为特性在抽象价值类型中使用：

		@Value.Immutable
		interface Val {
		  int number();
		  Confidential confidential();
		}
		...
		// toString
		"Val{number=1, confidential=<NON DISCLOSED>}"

另外两种隐藏数据的方法参阅保密特性和次要特性

## 风格 ##

`Immutables`使用`@org.immutables.value.Value.Immutable`来定义生成什么代码，使用`@org.immutables.value.Value.Style`来定义如何生成这些代码。

### 定义风格 ###

`@Value.Style`注解使用它的特性来完成所生成API和实现的自定义：

- 自定义使用什么前缀或后缀来探测类型和特性的名字；
- 自定义生成类型和方法的前缀或后缀；
- 强迫使用构造方法而不是工厂方法来构建建造者类（通过把方法名称改为`new`）；
- 定义实现类的可视范围：公共，包私有，私有或和所对应的抽象价值类型一致；
- 使构造方法返回抽象价值类型而不是具体的实现类；
- 把不可变的实现类作为私有成员隐藏在顶级的建造者类中；
- 把不可变的实现类作为私有成员隐藏在顶级的外部类中；
- 生成严格的建造者类；
- 强迫生成只支持JDK的实现代码，即使Guava已经包含在类路径中；
- 为`@Value.Immutable`创建会被每一个不可变类使用的默认设置模板。

### 应用风格 ###

风格可以附加在以下元素上：

- 包，此时会影响所有该包内的类，也会影响嵌套的包（除非被重写）；
- 顶级类，此时会影响该类和所拥有的嵌套价值类型；
- 嵌套的价值类型，如果与顶级的风格不冲突（处于包装类中），则有效。
- 注解，然后此注解又可以作为风格的元注解来标注类型和包。

内置的`@Value.Style`总是会覆盖元注解风格。

推荐为项目创建一个或多个风格元注解从而减少维护和升级的工作量。

		import org.immutables.value.Value;
		import java.lang.annotation.RetentionPolicy;
		import java.lang.annotation.Retention;
		import java.lang.annotation.ElementType;
		import java.lang.annotation.Target;

		@Target({ElementType.PACKAGE, ElementType.TYPE})
		@Retention(RetentionPolicy.CLASS) // Make it class retention for incremental compilation
		@Value.Style(
			get = {"is*", "get*"}, // Detect 'get' and 'is' prefixes in accessor methods
			init = "set*", // Builder initialization methods will have 'set' prefix
			typeAbstract = {"Abstract*"}, // 'Abstract' prefix will be detected and trimmed
			typeImmutable = "*", // No prefix or suffix for generated immutable type
			builder = "new", // construct builder using 'new' instead of factory method
			build = "create", // rename 'build' method on builder to 'create'
			visibility = ImplementationVisibility.PUBLIC, // Generated class will be always public
			defaults = @Value.Immutable(copy = false)) // Disable copy methods by default
		public @interface MyStyle {}
		...


		@Value.Immutable // if no attributes are specified, then defaults will be used
		@MyStyle // This annotation could be also placed on package
		interface AbstractItem {
		  int getId();
		  boolean isEnabled();
		}
		...

		Item item = new Item.Builder()
		  .setId(1)
		  .setEnabled(true)
		  .create();

最简单的全局化风格设置的方法是在项目的顶级包或模块中应用风格。

		// com/mycompany/project/package-info.java
		@MyStyle
		package com.mycompany.project;

需要注意：

在同一级别应用多个元注解时，如果发生了冲突，那么会随机选择一个来应用，在这里没有任何可靠的选择可以依赖；
风格不会以任何形式合并；
风格应该在包，顶层类，或嵌套价值类自身上应用。尝试在特性或中间层的嵌套类上应用风格是无效的；
风格是把双面剑，如果没有正确地定义风格，如名字不正确，包含非法字符或使用了Java关键字，就会引发编译错误；
风格会被积极缓存，所以如果改变了元风格，或者父包的风格，可能无法看到更改立刻生效，必须重新构建项目，甚至重启IDE。

### 去复数化 ###

`Style.depluralization`风格特性可以开启自动去复数化功能。该功能是针对集合或map类型的特性，把为它们生成的方法名中的复数改为单数，如`add*`和`put*`。可以使用`Style.depluralizeDictionary`数组（内部元素是`"singular:plural"`形式的字符串）来指定特殊情况。

		@Value.Style(
		  depluralize = true, // enable feature
		  depluralizeDictionary = {"person:people", "foot:feet"}) // specifying dictionary of exceptions

如果使用上例中定义的风格，则生成的`add*`方法的名字对应如下：

		boats → addBoat
		people → addPerson
		feet → addFoot
		feetPeople → addFeetPerson
		peopleRepublics → addPeopleRepublic

### 使用导入简化注解名 ###

通过使用导入可以简化使用的注解名（全限定名称 -> 简单名称）。

		import org.immutables.value.Value.Immutable;
		import org.immutables.value.Value.Parameter;

		@Immutable interface Value {
		  @Parameter int getFirst();
		  @Parameter String getSecond();
		}

###  使用"is"前缀和自定义赋值方法 ###

`Immutables`默认没有使用和JavaBean兼容的命名方式。因此默认只支持使用`get-`作为访问方法的前缀，或者不使用任何前缀。`isEmpty()`方法对应的特性的名字是`isEmpty`，而不是`empty`。但是通过设置`Value.Style(get = {"get*", "is*"})`，可以让系统把`get`和`is`都作为访问方法的前缀来处理。该风格特性最好在项目顶级层或模块中设置，从而保证整个项目的一致性。

		@Value.Immutable
		@Value.Style(get = {"get*", "is*", "*Whatever"}, init = "set*")
		interface Val {
		  int getProp();
		  boolean isEmpty();
		  String fooWhatever();

		  static void demo() {
			Val v = ImmutableVal.builder()
			  .setProp(1)
			  .setEmpty(true)
			  .setFoo("whatever")
			  .build();

			v.toString();// Val{prop=1, empty=true, foo=whatever}
		  }
		}

### 包装类型 ###

当为消息或文档建模时，可能会在一个文件里包含大量的小价值类。Java中一般会把这些小类放在一个顶级总和类中。

`@Value.Enclosing`注解可以用在顶级类上，为该类内部抽象价值类型的实现类提供命名空间。这样可以避免同一包内可能出现的命名冲突。

带命名空间的实现类的名字是没有前缀的，可以使用星号来一次性导入。

		@Value.Enclosing
		class GraphPrimitives {
		  @Value.Immutable
		  interface Vertex {}
		  @Value.Immutable
		  static class Edge {}
		}
		...
		import ...ImmutableGraphPrimitives.*;
		...
		Edge.builder().build();
		Vertex.builder().build();

注意需要把风格注解应用在顶层的包装类或包上，因为要保持包装类和嵌套类的风格一致。

版本2.0以前，`@Value.Enclosing`的名字是`@Value.Nested`。从版本2.1开始不再建议使用`@Value.Enclosing`注解，虽然还没有废弃，但通常没有必要再使用。

### 自定义不可变的注解 ###

可以为自定义的风格创建新的注解作为代表，并让`Immutables`注解处理器能够识别它们。

		package org.example.annotation;

		import org.immutables.value.Value;
		import java.lang.annotation.ElementType;
		import java.lang.annotation.Target;

		/**
		 * Tupled annotation will be used to generate simple tuples in reverse-style,
		 * having construction methods of all annotations.
		 */
		@Value.Style( // Tupled annotation will serve as both immutable and meta-annotated style annotation
			typeAbstract = "*Def",
			typeImmutable = "*",
			allParameters = true, // all attributes will become constructor parameters
								  // as if they are annotated with @Value.Parameter
			visibility = Value.Style.ImplementationVisibility.PUBLIC, // Generated class will be always public
			defaults = @Value.Immutable(builder = false)) // Disable copy methods and builder
		public @interface Tupled {}
		...

		/**
		 * Builded annotation will generate builder which produces private implementations
		 * of abstract value type.
		 */
		@Target(ElementType.TYPE)
		@Value.Style(
			typeBuilder = "BuilderFor_*",
			defaultAsDefault = true, // java 8 default methods will be automatically turned into @Value.Default
			visibility = Value.Style.ImplementationVisibility.PRIVATE,
			builderVisibility = Value.Style.BuilderVisibility.PACKAGE) // We will extend builder to make it public
		public @interface Builded {}

仅有这些定义还不够，还必须创建一个文本文件，该文本文件的位置为项目的类库路径下的`/META-INF/annotations/org.immutables.value.immutable`，其中`org.immutables.value.immutable`是文件的名字。例如在maven默认的项目结构中，该文件位于`moudledir/src/main/resources/META-INF/annotations/org.immutables.value.immutable`。然后在该文件中添加自定义注解的全限定名称：

		org.example.annotation.Tupled
		org.example.annotation.Builded

现在重新编译，并为该文件生成单独的jar文件并放在和注解处理器相同的类库路径/视野下，以便在编译期间和一般的类库路径一样使用。该jar文件在运行时并不需要。然后就完成了编译/注解处理的依赖设置，可以正常使用这些自定义的注解了。在maven中，把视野（scope）设置为`provided`即可。

		package org.example.models;

		import org.example.annotation.Tupled;
		import org.example.annotation.Builded;

		// Look, custom annotation instead of @Value.Immutable
		// and the style is also attached!
		@Tupled interface RgbDef {
		  double red();
		  double green();
		  double blue();
		}
		// ...
		Rgb color = Rgb.of(0.4, 0.3, 1.0);

		// Custom annotation for builder with private immutable implementation.
		@Builded public interface Record {
		  long id();
		  String name();

		  default String notes() { // Works as default attribute!
			return "";
		  }

		  // Then we extend package-private builder with public nested builder and expose all
		  // public methods as methods of Record.Builder.
		  class Builder extends BuilderFor_Record {}
		}
		// ...
		Record record = new Record.Builder()
			.id(123L)
			.name("Named Record")
			.notes("Oh, nothing interesting, surely!")
			.build();



















































































