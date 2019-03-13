### 1. 使用静态工厂方法代替构造方法 ###

和构造方法对比所具有的优点：

1. 每个静态方法都可以有自己的具有说明性的方法名
2. 让不变类（immutable classes）不用每次都创建一个新的实例，可以利用缓存机制重复使用已创建的实例。
	**instance-controlled Class:** 通过调用静态工厂方法返回相同的对象，从而实现对在任何时候能存在的实例进行严格控制的类。
	实例化控制可以保证一个类是singleton类或不可实例化类，也可以保证让不可变类绝不出现两个相同的实例。即：`a.equals(b) only if a == b`。枚举类型就可以提供这种保证。
3. 可以返回其声明返回类型的任何子类或实现类（如果声明的返回类型是接口）类型的对象。
	一个典型的应用就是静态工厂方法声明返回一个接口或者抽象类，从而把实际的实现类或子类信息这些实现细节隐藏起来，从而让API更精简。
	通常会把一个接口`Type`的静态工厂方法放在一个名为`Types`的不可实例化类中，这个`Types`类作为接口`Type`的工具类使用。如`Collections`对应于`Collection`。
	Java8之前接口中不能有静态方法，必须采用上面所说的工具类来定义接口的静态工厂方法；Java8取消了这个限制，但要求所有的静态成员都必须是公有的；Java9则允许接口有自己的私有的静态方法，但静态成员变量和静态成员类都要求必须是公有的。因此有可能需要把具体的实现方法细节封装到一个包私有的类中，然后在接口内部的静态工厂方法中调用该类的方法从而进一步隐藏实现细节。
4. 返回对象的实际类型可以根据传入参数的不同而不同，只要保证是声明的返回类型的子类或实现类即可。
5. 在编写静态工厂方法时，返回对象的实际类型所对应的接口的实现类可以并不存在（因为签名中返回的类型是接口）。

**service provider framework**：在一个系统中，由单独的服务提供者来提供实现了具体服务内容的服务实现类，而系统负责将这些服务暴露给用户，从而将用户与服务实现代码剥离。是面向接口编程的一种体现。

**service provider framework**有3个主要和1个可选组成部分，以JDBC为例：

- **service interface**：服务接口，代表服务（方法）的所有实现。`Connection`
- **provider registration API**：服务提供类注册，服务提供类用来注册具体的实现。`DriverManager.registerDriver`
- **service access API**：服务访问API，用来取得服务的实例，也就是上面所说的静态工厂方法，用户可以通过这个API来设定选择实现的条件。如果没有设定条件，则返回一个默认实现的实例或者让用户遍历所有可用的实现。服务访问API就是灵活的静态工厂方法，也是服务提供者框架的基础。`DriverManager.getConnection`
- **service provider interface**（可选）：服务提供者接口，描述用于创建服务对象的工厂对象，也就是服务提供类对象，如果没有此工厂对象，就要使用反射机制来实例化服务对象。`Driver`

	个人理解：在Java中，万物皆对象，包括服务，虽然服务实际对应的是一个个方法，但这些方法必须依托于某个对象才能被调用，所以这个对象也就是抽象服务接口的实现类的对象。在**service interface framework**中，服务接口定义服务的内容，代表了所有的实现；服务提供者负责提供具体的实现，不同的实现由不同的服务提供者提供。服务提供者接口就是用来描述这些提供者的行为（包括提供/返回一个具体的服务实现），也是他们的类型代表。而服务提供类注册则是告诉系统有哪些服务提供类可以使用。之所以会出现多个服务提供类是因为每个服务提供类所对应的业务领域或技术不同，如JDBC中的Driver可能是基于MySQL的，也可能是基于Oracle的。服务访问API则是用户取得服务对象（是服务对象而不是服务提供类对象，系统应该根据用户提供的条件选择合适的服务提供者来提供相应的服务对象）的方法。

静态工厂方法的缺点：

- 没有`public`或`protected`构造方法的类是无法被继承的。
- 静态工厂方法不像构造方法在javadoc中有明显提示

静态工厂方法常用命名规则：

- from：类型转换方法，根据一个其他类型的参数返回此类相应的实例

		Date d = Date.from(instant);

- of：一个聚合方法，根据多个参数返回此类相对应的实例

		Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);

- valueOf：from和of方法的综合版

		BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);

- instace/getInstance：根据参数返回一个此类的实例，但不保证返回的实例都是同一个实例

		StackWalker luke = StackWalker.getInstance(options);

- create/newInstance：和getInstance类似，但每次都会返回一个新的实例

		Object newArray = Array.newInstance(classObject, arrayLen);

- getType：和getInstance类似，但用于在其他类中创建本类实例

		FileStore fs = Files.getFileStore(path);

- newType：和newInstance类似，但用于在其他类中创建本类实例

		BufferedReader br = Files.newBufferedReader(path);

- type：getType和newType的简写

		List<Complaint> litany = Collections.list(legacyLitany);

### 2. 当构造方法有很多参数时，使用Builder ###

当构造方法可能有很多参数时，方法之一是使用`telescoping constructor`：先编写最复杂的构造方法，其他构造方法都是通过调用此构造方法，并为删减的参数提供默认值的方式来实现。

		public Person(){this("Max");}
		public Person(String name){this(name, 20);}
		public Person(String name, int age){this(name, age, Sex.MALE);}
		public Person(String name, int age, Sex sex){}

另一种方法是使用`JavaBean`规范：先使用无参数的构造方法创建对象，再调用成员变量的setter方法来初始化成员变量值。但这种方法会造成对象在初始化过程中处于非一致状态。并且这种方法无法应用于不可变类。

第三种方法就是Builder方法：使用必需参数调用该类对应的Builder类的构造方法，使用返回的Builder对象的setter方法来设置可选参数的值，这些setter方法返回的都是该Builder对象，因此可以链式调用，最后再调用该Builder对象的`build()`方法来生成对应的类的对象。这个Builder类通常是以要构建的类的公共静态成员类的形式存在。

在建造者的构造方法和其他方法中对传入的参数进行校验，把涉及到多个参数的合法性校验放在`build()`方法调用的需构造的类的构造方法中进行。这些合法性校验应该基于传入参数的防御性拷贝进行。一旦发现错误，及时抛出`IllegalArgumentException`异常，并在异常的详细消息中指出是哪些参数不合法。

建造者模式也适用于类的层次结构。每个类，包括抽象类，都可以有自己相应的建造者类，一般以静态成员类的形式存在，抽象类的建造者类也是抽象的。

在Java中没有自类型(self type)，但可以通过使用带有递归类型参数的泛型类型+抽象的`self()`方法来实现可在继承中使用的方法链。

协变返回类型(covariant return typing)：子类方法返回的类型是父类中该方法返回类型的子类。

建造者模式优于构造方法的一个地方在于建造者可以指定多个varargs参数，因为每个参数都通过自己的方法来设置。因此可以在通过对同一方法的多次调用传入多个相同类型的参数，再用这些参数的值来设置一个成员变量的值，例如根据传入的多种配料生成自己的披萨配方。

一个建造者对象可以用来生成多个被建造者对象。

建造者模式因为要创建建造者对象，所以在对性能要求非常苛刻的环境可能会受到限制。

小结：当构造方法或静态工厂方法需要接受很多参数来生成实例时，可以优先考虑使用建造者模式。

### 3. 通过私有构造方法或枚举类型来强化单例属性 ###

通常有2种方式来实现单例类，2种方式都需要把构造方法私有化并暴露一个公有的静态成员，不同之处就在于暴露的成员的类型不同：

1. 暴露的是final静态成员变量；
2. 暴露的是静态工厂方法。

静态工厂方式的优点：

- 可以在不改变API的情况下改变类是否是单例类；
- 如果需要，可以写成泛型单例工厂；
- 方法引用可以直接作为供应类使用，如`Type::instance`可以作为`Supplier<Type>`

如果没有对上述优点的需求，可以考虑优先使用final静态成员变量的方法。

如果需要把单例类可序列化，必须把所有的实例成员变量设为`transient`并提供一个`readResolve()`方法来返回那个唯一的实例。

还有第三种方式，也是最简单有效的方法是使用只有一个值的枚举类型，这样就可以直接实现可序列化，也可有效对付反射攻击。前提是不需要继承除`Enum`类之外的其他类（接口的实现没有限制）。

### 4. 使用私有构造方法强化不可实例化的类 ###

工具类是一些只包含静态成员变量和静态方法的类，通常用于：

- 打包和基本类型和数组有关的方法
- 打包处理某些接口实现类对象的静态方法
- 打包处理不可变类的方法

工具类通常都是不能实例化的。可以通过显式提供一个私有构造方法来达到无法实例化的目的。在该构造方法中抛出一个`AssertionError`来避免在类内部不小心调用该构造方法，并用注释注明此构造方法是为了达到无法实例化的目的。这种方法还有一个副作用就是该类将因为没有公共的构造方法而无法被继承。

### 5. 用依赖注入来取代在代码中写死资源 ###

静态工具类和单例类并不适用于自身行为依赖于某个下层资源的类。这样的类应该使用依赖注入方式，在创建实例时为构造方法或静态工厂方法或建造者传入所依赖的资源或资源工厂，并根据传入的资源生成不同的实例。传入的资源应该使用防御性拷贝实现资源的不可更改。

资源工厂的典型应用是Java8中的`Supplier<T>`接口，使用该接口作为输入的方法通常应该使用带边界的通配符来限制工厂的类型参数，从而让该工厂只能产生特定类型及其子类的对象。

小结：不要把依赖于一个或多个下层资源的类变成单例类或静态工具类，也不要在这些类中直接创建这些资源。应该把这些资源或相关的资源工厂作为输入传递给这些类的构造方法，静态工厂方法或建造者。

### 6. 避免创建不必要的对象 ###

一个不可更改的对象总是能够被重复使用。如果一个不可变类同时提供了构造方法和静态工厂方法，通常静态工厂方法可以避免创建不必要的对象，因为构造方法总是会创建一个新的对象，而静态工厂方法不需要如此，很多实现都提供了缓存功能。

可变类的对象如果不再被更改，也可以安全地被重用。

有一些方法，如`String.matches()`方法会在方法内部创建`Pattern`类的实例并仅使用一次就丢弃，如果创建对象的成本过高，并且有可能多次调用该方法，则有可能会导致性能的下降。因此，显式创建一个该对象的引用，缓存它，并在需要的时候重用它可以改善性能。

不推荐延后初始化(lazily initializing)。

适配器模式中的适配器只是为所封装的对象提供一个不同的接口，除了被包装对象的状态，本身并不具备新的状态，因此通常没有必要为一个给定的对象创建多个同类型的适配器。例如`Map`接口的`keySet()`方法返回的`Set`对象。

另一种可能被无意义地创建的对象是基本类型的封装类对象，尤其是在系统自动封装和解封时。处理的原则是尽量使用基本类型，同时注意无意中造成的自动封装。

通常不需要自己创建对象池来避免创建对象，但一些创建成本很高的类的对象可能需要一个对象池来作为缓存，如数据库连接。

### 7. 清除废弃的对象引用 ###

废弃的对象引用(obselete references)就是那些再也不会被引用的引用。

如果一个对象的引用无意间被保留了下来，那么不仅仅该对象不会被垃圾回收，该对象所使用的对象引用也不会被回收。当这种对象越来越多时就可能造成内存泄露。解决的方法就是让所有废弃的引用指向`null`。这样做的另一个好处是如果有引用不小心重新使用了该引用，就会抛出`NullPointerException`，从而发现错误。

注意，使引用指向`null`应该作为一种特例，而不是成为一种正常的编程习惯。更正确的做法应该是尽量缩小类似引用的作用范围，让他们在使用后自然失效从而被回收。

什么时候使用赋空操作呢？

- 当类自己管理内存的时候，我们应该小心处理内存泄露。比如说数组的内容并不是全部都需要，只有部分是有效数据，其他的都可以废弃时。
- 当使用缓存时，需要定期或者在加入新数据时对旧有内容进行清理。
- 使用监听器或者其他回调时。可以通过只使用弱引用来解决回收问题。

### 8. 避免使用finalizers和cleaners ###

Finalizers是不可预测的，通常也是危险和不必要的。Cleaner比finalizers好一些，但同样是不可预测，执行缓慢以及不必要的。

Finalizers和cleaners的一个很大的缺陷是无法保证他们一定会被及时运行，甚至是否一定会被运行都无法保障，因此不要把时间紧要的操作放在他们之内，也不要依靠他们去更新持续化的状态。

另一个缺陷是在finalization过程中如果抛出未捕获的异常，该异常会被忽略，但finalization会中止。cleaner不会发生这种情况。正常的异常处理是中止线程并打印异常跟踪信息。

第三个缺陷是finalization和cleaner会带来性能上的惩罚。

第四个缺陷是会引发安全问题，使类面临finalizer攻击。

finalizer攻击：如果在一个类的构造方法或序列化/反序列化过程中抛出异常，恶意的子类finalizer代码可以对这个创建了一半的对象进行操作，从而阻止对他的回收。

在构造方法中抛出异常通常可以阻止一个对象的创建，但如果该类有finalizer，则不一定能阻止成功。为了避免这种攻击，可以在可继承类中加入final的`finalize()`空方法。

最好的替代finalizers和cleaners的方法：
	让类实现`AutoCloseable`接口，要求类的使用者必须在每个实例使用完之后显式调用它的`close()`方法，通常配合`try-with-resources`代码块使用。另外要注意实例应该知道自己是否已被关闭，通常是`close()`方法会在一个成员变量中标记实例已被关闭，其他方法在执行时必须检测该标记，如果标记为已关闭则应该抛出`IllegalStateException`。

可能需要用到finalizers和cleaners的地方：

- 作为一道备用安全网来处理用户忘记使用`close()`方法关闭资源
- 用于关闭native peers。

cleaners用起来也很麻烦，尽量避免使用。

### 9. 使用try-with-resources替代try-finally ###

在try-with-resources代码块中使用的资源必须实现`AutoCloseable`接口。传统的try-finally模块在代码块内部产生的异常会掩盖初始化资源和关闭资源可能产生的异常。

一句话：尽量使用try-with-resources替代try-finally来处理需要关闭的资源。

### 10. 在重写`equal`方法时遵守通用规则 ###

默认的`equals`方法判断两个变量是否相等的唯一条件就是两个变量必须指向同一个实例。

不需要重写类的`equals`方法的情况：

- 该类的每一个实例本身都是唯一的；
- 不需要为该类提供一个判断是否相等的逻辑；
- 该类的父类已经重写了`equals`方法，并且该方法也适用于此子类；
- 该类的访问权限是私有或者包内访问，并且该类的`equals`方法永远都不会被调用。为了确定`equals`方法不会被调用，可以重写该方法如下：

		@Override
		public boolean equals(Object o) {
			throw new AssertionError();	
		}

只有在默认的或者父类的等价判断不适用于该类，从而必须提供自己的等价判断逻辑时才需要重写`equals`方法。这些类通常是价值类value classes（该类代表了一种值的类型，如Integer和String）。

有且仅有一个实例的价值类，如枚举类，不需要重写`equals`方法。

重写`equals`方法需要遵循的原则：

- 自反性reflexive：对于所有非空变量x，`x.equals(x)`必须返回`true`
- 对称性symmetric：对于所有非空变量x，y，`x.equals(y)`在且仅在`y.equals(x)`返回`true`的情况下才返回`true`
- 传递性transitive：对于所有非空变量x，y，z，如果`x.equals(y)`和`y.equals(z)`都返回`true`，则`x.equals(z)`必须返回`true`
- 一致性consistent：对于所有非空变量x，y，在没有改变`equals`方法所用参数值的情况下，多次调用`x.equals(y)`的返回结果必须保持一致
- 对于所有非空变量x，`x.equals(null)`必须返回`false`

当需要为一个价值类添加新的内容时，可以通过使用聚合来代替继承，从而实现扩展而不是改写原有的代码。

如果父类是不可实例化类，如抽象类，则重写`equals`方法不会引起任何问题。

不要让`equals`方法依赖于不稳定的资源。URL类的`equals`方法需要将url地址转换为IP地址后再进行比较，所以要依赖于网络连接，这是不推荐的。应该让`equals`方法使用驻留在内存中的资源。

重写`equals`方法的步骤：

1. 使用`==`判断输入参数是否与此实例指向同一个对象
2. 使用`instanceof`来检查输入参数与此实例的类型是否相同
3. 将输入参数强制转换为同一类型
4. 挨个检查类型中每一个有意义的成员变量是否相同。如果步骤2中的类型是接口，则应该使用接口中的方法来访问成员变量
	- 对比除`float`和`double`以外的基本类型时，直接使用`==`；
	- 对比引用类型的变量时，递归调用他们的`equals`方法；
	- 对比`float`和`double`类型的变量时，分别使用`Float.compare(float, float)`和`Double.compare(double, double)`方法。如果直接使用`Float.equals`和`Double.equals`方法会因为自动拆装箱导致性能下降。
	- 对比数组类型的变量时，使用以上原则对数组里的每一个有意义的元素进行对比。如果数组里的所有元素都有意义，则直接调用`Arrays.equals`重载方法之一来对比。

有些`Object`类型的变量可能允许`null`值，此时应该调用`Objects.equals(Object, Object)`方法来对比。

为了优化性能，应该先比较最有可能发生变化，或者最容易比较的变量，最理想的状况是两者兼备。有些类如果比较所有的属性会对系统带来沉重的负担，此时可以为该类提供一个规范模式（canonical form），如使用一个将所有属性的`toString`结果合并起来的字符串，然后直接通过比较该字符串来比较其对象。

不要对比不是对象逻辑状态组成部分的成员变量，例如用于同步锁。
不需要对比由关键数据计算生成的衍生数据。除非该衍生数据的对比能用来取代所有的关键数据的对比。

一些额外的注意事项：

- 重写`equals`方法时也一定要重写`hashCode`方法
- 不要自作聪明把对比复杂化
- 不要在`equals`方法的签名中把输入参数从`Object`改为其他类型。否则，这是重载，而不是重写。使用`Override`注解来避免这个错误。
- 考虑使用google的`AutoValue`框架来生成和测试这些方法。

### 11. 重写`equals`方法时也重写`hashCode`方法 ###

在每一个重写了`equals`方法的类中都必须重写`hashCode`方法

- 在同一个程序运行期间，如果`equals`方法中使用的信息没有发生变化，则多次调用一个对象的`hashCode`方法的返回值应该保持一致。不同程序执行期间`hashCode`方法的返回值可以不同。
- 如果两个对象通过`equals`方法判定是相等的，则他们应该返回相同的hashCode值。
- 如果两个对象通过`equals`方法判定是不等的，他们不一定必须返回不同的hashCode值。

一个简单的计算hashCode值的方法：

1. 声明一个`int`类型的变量`result`，它的初始值等于该对象第一个关键成员变量的hashCode值`c`；
2. 对于剩下的每一个关键成员变量`f`，按如下方法计算其hashCode值：
	1. 计算该成员变量的`int`类型的hashCode值：
		1. 如果成员变量是基本类型，使用`Type.hashCode(f)`，其中`Type`是成员变量`f`对应的基本类型的封装类型
		2. 如果是引用类型的变量并且该类型的`equals`方法递归调用其成员变量的`equals`方法，则递归调用其成员变量的`hashCode`方法。如果需要更复杂的比较，则使用该成员变量的标准形式的hashCode值。如果该成员变量的值是`null`，通常使用0来表示。
		3. 如果该成员变量是数组类型，则把它的每一个关键数组元素作为一个单独的成员变量来处理。也就是按照上述方法来计算每一个关键元素的hashCode值并把它们叠加。如果该数组没有关键元素，通常返回0.如果该数组中所有的元素都是关键元素，则可以直接使用`Arrays.hashCode`方法来计算。
	2. 把按照上述方法计算得到的hashCode值按照以下公式叠加并赋给`result`

		result = 31 * result + c;

3. 返回`result`
		
计算hashCode值时，应该排除那些由关键数据计算生成的衍生数据。换句话说，如果某个成员变量的值是从其他成员变量的值经过计算产生的，则可以将其排除。另外，也应该排除那些没有在`equals`方法中使用的成员变量。

`Objects`类有一个静态方法`hash`，接受多个输入参数并按照上述方法来计算hashCode值。但这个方法因为使用了数组和自动封装，可能造成性能下降，所以只适用于性能不重要的场合。

如果某个类是不可变类并且计算hashCode值的成本较高，则可以考虑缓存该类型对象的hashCode值，而不用每次都在需要的时候重新计算。如果该类型的大多数对象都会作为哈希键值，则应该在创建对象时就计算它的哈希值。否则，可以使用延后初始化的方法在第一次使用到它的哈希值时再计算，但要小心确保线程安全。

不要为了提高性能就故意忽略对关键数据的计算。

不要为返回的哈希值提供详细的规范，这样该类的用户就不会依赖该返回值，从而可以在必要的时候改变该方法的实现。

### 12. 总是重写`toString`方法 ###

`toString`方法应该返回对象中所有有意义的信息。理想状况下，返回值应该能够自我说明。

重写价值类的`toString`方法时，推荐规范化返回的字符串格式，并在文档中进行说明。如果指定了格式，那么最好提供一个匹配的静态工厂或构造方法从而可以轻易在对象和字符串之间进行转换。不管是否规范字符串格式，都应该在文档中说明这么做的意图。

对在`toString`方法返回的字符串中包含的信息都应该提供相应的访问方法，这样可以避免用户只能通过解析字符串来获得想要的信息。

可以在抽象类中实现`toString`方法，如果它的子类使用同一种字符串表达方式。

### 13. 谨慎重写`clone`方法 ###

`Cloneable`接口设计为表示类是可以被克隆的混元接口（mixin）。它的缺点是没有一个`clone`方法，而`Object`类的`clone`方法是protected的，只能通过反射来调用对象的`clone`方法，并且不能保证成功，因为该对象可能没有一个可调用的`clone`方法。

`Cloneable`接口虽然没有包含任何方法，但它指明了`Object`的`clone`方法所实现的行为：如果某个类实现了`Cloneable`接口，则`Object`的`clone`方法返回一个把成员变量逐个拷贝的对象复本，否则会抛出`CloneNotSupportedException`异常。这是接口的非常规用法，不推荐模仿。

在实际使用过程中，一个实现了`Cloneable`接口的类应该提供一个能有效工作的`public clone`方法。需要注意的是，使用克隆会不调用构造方法而创建新的对象。

如果一个类的`clone`方法使用该类的构造方法创建对象的复本，而不是调用`super.clone`方法，那么该类的子类在调用`super.clone`方法时返回的将是错误的类型。虽然final类因为无法继承不会出现这个问题，但一个不在自己的`clone`方法中调用`super.clone`方法的final类也就完全没有必要取实现`Cloneable`接口，因为它的`clone`方法的实现与`Object`类的`clone`方法完全无关。

如果一个类的父类有行为良好的`clone`方法，则可以声明这个类实现了`Cloneable`接口并调用`super.clone`来确定对象复本。如果该类的每个成员变量都是基本数据类型或指向一个不可变的对象的引用类型，则返回的复本不需要再做处理，可以直接使用。事实上，不可变类完全不需要提供`clone`方法，因为它的对象都是不变的，不需要再去生成一个复本。

实现了`Cloneable`接口的类的`clone`方法其实是重写了`Object`类的`clone`方法，因此应该用`@Override`注解标识，而它的访问权限和返回类型必须遵照重写的两同两小一大原则。

- 方法名同，形参列表同
- 子类方法返回值类型，声明抛出的异常类型要比父类方法的更小或相等
- 子类方法的访问权限要比父类方法的大

另外，`Object`的`clone`方法抛出了`CloneNotSupportedException`异常，需要进行处理。但如果此类声明实现了`Cloneable`接口，则调用`super.clone`方法一定会成功而不会抛出异常，此时应该在处理异常的catch代码块中抛出一个异常报警。

		@Override 
		public PhoneNumber clone() {
			try {
				return (PhoneNumber) super.clone();
			} catch (CloneNotSupportedException e) {
				throw new AssertionError(); 
			}
		}

实际上，`clone`方法的作用类似于构造方法。必须确保它不会影响对象原件以及生成了不受影响的复本。

调用数组的`clone`方法会返回一个与被克隆数组的编译时和运行时类型都相同的数组。这是复制数组的最好方法。

和序列化类似，`Cloneable`架构与引用可变对象的final成员变量的使用不兼容，除非这个可变对象可以在原对象和复本间安全地共享。因为使用`clone`方法后，复本的final变量已经赋值了，无法为了深度复制将一个新创建的成员变量复本赋给它。

深度复制有时会因为大量的递归而导致性能下降，此时可以使用迭代（iteration）来替代。

和构造方法类似，`clone`方法不应该调用任何可被重写的方法

公共的`clone`方法应该在方法体内对可能抛出的`CloneNotSupportedException`异常进行处理，从而避免在声明中抛出该异常。

如果一个类被设计为用于继承，则该类不应该实现`Cloneable`接口。此时有两种处理方法：

- 在该类的内部模仿`Object`类创建一个声明抛出`CloneNotSupportedException`异常的`protected clone`方法。那么该类的子类就可以像继承了`Object`类一样自主选择是否实现`Cloneable`接口。
- 使用如下的模板创建一个无效的`clone`方法，从而阻止子类实现`Cloneable`接口

		//阻止子类实现`Cloneable`接口的父类的`clone`方法
		@Override
		protected final Object clone() throws CloneNotSupportedException {
			throw new CloneNotSupportedException();
		}

如果一个线程安全的类需要实现`Cloneable`接口，则必须同步它的`clone`方法

重写`clone`方法的正常步骤：

1. 所有实现了`Cloneable`接口的类都应该重写`clone`方法，把它的访问权限变为public，返回类型变为该类本身。
2. 先调用`super.clone`方法，然后修正需要修正的成员变量。这通常意味着深度复制那些可变对象，然后让指向原件的成员变量重新定向到这些新生成的复本。

另一种更好的复制对象的方法是提供一个复制构造方法或复制工厂。即接受一个自身类型的参数作为输入参数的构造方法或工厂方法。

		//复制构造方法
		public Type(Type type){};
		//复制工厂
		public static Type newInstance(Type type) {};

把接口作为输入参数的复制构造方法和复制工厂称为转换构造方法和转换工厂。因为使用他们可以在实现了同一个接口的类型之间进行转换。

除了数组之外，尽量使用复制构造方法和复制工厂来实现对象的复制。

### 14.  考虑实现`Comparable` ###

一个类实现了`Comparable`接口表示它可以排序。将此类的某个对象与指定的对象进行比较以获得顺序。返回负整数，零或正整数，分别表示此对象小于，等于或大于指定对象。如果指定对象的类型阻止将其与此对象进行比较，则抛出`ClassCastException`。

在以下描述中，符号sgn（表达式）指定数学符号函数，其被定义为根据表达式的值是负，零还是正而返回-1,0或1。

- 实现者必须保证`sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`适用于所有x和y。如果 `x.compareTo(y)` 抛出异常，那么 `y.compareTo(x)` 也应该抛出异常。
- 同时实现者也应该保证关系是可传递的： `(x.compareTo(y) > 0 && y.compareTo(z) > 0)` 暗示着 `x.compareTo(z) > 0`
- 最后，实现者必须保证 `x.compareTo(y) == 0` 暗示着对于所有 z，`sgn(x.compareTo(z)) == sgn(y.compareTo(z))`都成立。
- 强烈推荐但不是必须的，`(x.compareTo(y) == 0) == (x.equals(y))`。 通常来讲，任何实现了`Comparable`接口并违反此条件的类都应清楚地在文档中说明：“此类具有与equals不一致的自然顺序。”

如果数组的元素实现了`Comparable`接口，则可以使用`Arrays.sort(array);`来排序。

基本上Java类库中所有的价值类，包括所有的枚举类都实现了`Comparable`接口。

和`equals`方法类似，添加一个关键数据应该影响`compareTo`方法的结果。因此，使用聚合而不是继承来添加关键数据，这样就可以不用修改代码，而是扩展代码。

如果一个成员变量没有实现`Comparable`接口或者需要非标准的排序方法，则可以使用`Comparator`来替代自然排序。

`Comparable`接口是泛型接口，在声明实现该接口时如果指定了泛型参数类型，则意味着该类只能和这个参数类型的对象进行比较。

不推荐使用关系运算符`<`和`>`来进行比较，应该使用封装类的`compare`方法来比较。

如果一个类有多个关键数据，则比较这些数据的顺序非常重要。应该按照数据的重要性从高到低依次比较。

从Java8开始，`Comparator`接口提供了一系列的对比构造方法，从而可以串联使用比较器，但会带来性能上的损耗。当使用这种方法，考虑使用静态导入机制来简化方法的名字。

		//使用对比构造方法
		private static final Comparator<PhoneNumber> COMPARATOR = 
			comparingInt((PhoneNumber pn) -> pn.areaCode).
			thenComparingInt(pn -> pn.prefix).
			thenComparingInt(pn -> pn.lineNum);
		
		public int compareTo(PhoneNumber pn) {
			return COMPARATOR.compare(this, pn);
		}

上例中，`comparingInt`方法使用一个键值提取函数（把一个对象的引用映射为一个`int`类型的键值）作为输入参数并返回一个以此键值排序的比较器。当比较结果是相等时，再依次使用后面两个比较器进行比较。

不要使用两个关键数据的差值作为判断大小的依据，应该使用封装类的静态`compare`方法或者对比构造方法来比较。

		//利用差来比较，不推荐
		static Comparator<Object> hashCodeOrder = new Comparator<>() {
			public int compare(Object o1, Object o2) {
				return o1.hashCode() - o2.hashCode();
			}
		};

		//静态比较方法
		static Comparator<Object> hashCodeOrder = new Comparator<>() {
			public int compare(Object o1, Object o2) {
				return Integer.compare(o1.hashCode(), o2.hashCode());
			}
		};

		//对比构造方法
		static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());

小结：在编写任何可排序的价值类时，都应该实现`Comparable`接口。`compareTo`方法中不要使用关系运算符`<`和`>`以及数值差来作为排序的依据，使用基本类型的封装类的静态`compare`方法或者对比构造方法来进行排序。

### 15. 最小化类及其成员的访问权限 ###

尽量把每一个类及其成员的可视范围降至最低。

如果一个顶级类或接口不是公共API的一部分，就应该把它们的访问权限局限于包内；如果他们只被一个类使用，应该考虑将他们变成该类的私有静态嵌套类。

在设计完一个类的公共API之后，应该先把剩下的成员都变为私有成员，再在开发的过程中根据需要做出调整。

受保护的成员也是对外API的一部分，会暴露实现细节。重写方法时需要遵守两小两同一大原则。

不要为了方便测试随便提升成员的访问权限，将公共类的私有成员提成为包私有是可以接受的，更高等级（会成为API的一部分）就不行了。

公共类的实例成员很少拥有公共访问权限，因为拥有可变的公共成员的类不是线程安全的。

常量必须是基本数据类型或者不可变类。

非零长度数组总是可变的，所以不能作为公共的静态final成员使用，也不应该提供相应的访问方法。如果外部需要使用数组中的数据，可以使用以下2种方法：

- 将该数组变为私有和final，再用该数组生成一个公共不可变的列表

		private static final Thing[] PRIVATE_VALUES = {...};
		public static final List<Thing> VALUES = 
			Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));

- 将该数组变为私有和final，再添加一个公共方法返回该数组的拷贝。

		private static final Thing[] PRIVATE_VALUES = {...};
		public static final Thing[] values() {
			return PRIVATE_VALUES.clone();
		}

从Java9开始可以使用模块来组织包。在模块声明中可以通过导出声明指定需要导出的包，没有被导出的包的公共成员或受保护的成员是不能被其他模块访问的。但是如果程序的类库路径包含了该模块的jar包的位置，则模块的导出机制无效，按Java传统的访问权限机制处理。

### 16. 在公共类中使用存取方法（setter/getter），不要使用公共成员变量 ###


如果一个类会有来自包外的访问，那么就应该为其成员变量提供访问方法，而不是赋予它们公共权限。但如果一个类是包私有类或者某个类的私有嵌套类，则可以将其成员变量变为公共变量直接使用，不必提供访问方法。

### 17. 最小化可变性 ###

不可变类是指一个类的对象一旦创建就不能再做更改。在对象的生命周期内其所包含的信息都是固定的。

创建不可变类的5个原则：

- 不要提供会改变对象状态的方法（mutators）
- 此类不能被继承
- 所有成员变量都是final的
- 所有成员变量都是私有的
- 排除一切对可变内容的访问。确保用户不会获得任何可变内容的引用，不要让这些变量的初始化依赖于用户提供的数据，也不要提供访问方法返回这些引用。在构造方法，访问方法和`readObject`方法中使用防御性拷贝。

函数式方法（functional approach）：一个方法使用它的输入参数得出并返回了结果，同时又没有改变输入参数的状态。方法名往往是介词，而不是动词。
过程式方法（procedural approach）：一个方法对它的输入参数进行了处理，从而改变了它的状态。

不可变对象是线程安全的，不需要进行同步，可以被安全地共享。所以不可变类应该鼓励它的用户尽可能重用已经存在的实例。一种方法是为经常用到的值提供public static final常量。或者更进一步，提供静态工厂来缓存被频繁需要的实例。

不需要拷贝不可变对象，因此不需要为不可变类提供`clone`方法或拷贝用构造方法。

不可变对象非常适合用作map的键值或者set的元素。它提供了免费的原子失败机制（参阅76），它们的状态永远不会改变，所以不可能出现临时的不一致。

不可变类的主要缺点是每一个可能值都对应一个单独的对象，所以可能造成开销过大或性能下降。解决方法是提供一个相应的可变的伙伴类，这个类负责执行一些常见的，开销较大的操作。在操作过程中可以通过改变自身对象的状态来模拟对应不可变类的另一个对象，从而避免使用不可变类执行这些操作时每次都需要重新创建新的不可变对象以应对变化。也就是说，通过复用可变的伙伴类对象，可以减少不可变类创建对象的次数，从而降低系统开销。

如果可以预测哪些多步操作是经常需要的，不需要用户干预，就可以把伙伴类声明为包私有，把这些多步操作组合为一个新的基础操作并发布。如果无法预测，则将伙伴类声明为公有，让用户来实现这些复杂操作。

除了把不可变类声明为final外，也可以通过把类的构造方法全部限制为私有或包私有，另外提供公共静态工厂的方法来确保其不能被继承。这样该类不仅可以灵活地拥有多个实现类，它的每个静态工厂方法提供不同的实现，返回不同的类型（但必须是该类的子类或实现类），还可以在将来的版本中通过优化静态工厂方法中的对象缓存机制来提高性能。

`BigInteger`类和`BigDecimal`类是不规范的不可变类，他们可以被继承，所以在接受来自不信任源的这两类参数时，应该对他们进行类型检查，如果是他们的子类，则应该防御性拷贝这些参数以免这些子类中有可变的成员变量。

严格意义的不可变类的状态是完全不能改变的，但实际上只要满足对外的状态不会发生变化就可以，其内部状态是可以改变的。例如在不可变类内部声明私有变量用于缓存某些开销较大的执行结果。之所以可以这么做是因为其不可变性决定了每次执行的结果都是一样的，所以不需要再重新执行，直接使用缓存的结果就可以，从而节省了开销。

当序列化一个包含了可变内容的不可变类时，需要显式提供`readObject`或`readResolve`方法，或者使用`ObjectOutputStream.writeUnshared`方法和`ObjectInputStream.readUnshared`方法。

小结：避免为每个访问方法都提供设置方法，除非真的有必要。把那些小的价值类设计为不可变类并尽量让那些大的价值类也成为不可变类。当且仅当对性能的要求非常高时，再为不可变类提供相应的公共可变伙伴类。尽量减少对象可能出现的状态。将所有成员变量优先声明为private final，然后再根据需要调整。构造方法应该创建完全初始化的对象。不要提供一个再初始化方法来模拟新建一个不同状态的对象。

### 18. 组合优先于继承 ###

在包中使用继承是安全的，其中子类和父类的实现都在同一个程序员的控制之下。对于专门为了继承而设计的，并且有文档说明的类来说（参阅19），使用继承也是安全的。跨越包级边界继承普通的具体类是非常危险的。

继承打破了封装，一个子类依赖于其父类的实现细节来保证其正确的功能。父类的实现可能会从发布版本不断变化，子类可能因此而被破坏。

与其选择继承一个类，可以在新类中增加一个私有变量，该变量是现有类的实例引用，这种设计被称为组合（composition），因为现有的类将成为新类的组成部分。新类中的实例方法调用其所包含的现有类的实例的相应方法并返回结果，这被称为转发（forwarding），而新类中的方法被称为转发方法。

包装类不适合在回调框架（callback frameworks）中使用，其中回调对象是将自我引用传递出来以用于后续调用（“回调”）。而被包装的对象不知道它外面的包装对象，它只能传递一个指向自身的引用（this），因此无法通过外面的包装对象回调。这被称为SELF问题。

只有在子类真的是父类的子类型的情况下，继承才是合适的。换句话说，只有在两个类之间存在“is-a”关系的情况下，B类才能继承A类。如果试图让B类继承A类时，问自己这个问题：每个B都是A吗？如果答案不确定，那么B就不应该继承A。如果答案是否定的，那么B通常包含一个类型为A的私有实例，并且暴露一个不同的API：A不是B的重要部分 ，只是其实现细节。

如果在适用组合的地方使用继承，则会不必要地公开实现细节。

小结：只有在子类和父类之间存在真正的子类型关系时才适用。如果子类与父类不在同一个包中，并且父类不是为继承而设计的，继承可能会导致脆弱性。为了避免这种脆弱性，使用组合和转发代替继承，特别是如果存在一个合适的接口来实现包装类。

### 19. 继承需要有专门的设计和文档，否则就不应该使用 ###

首先，这个类必须在文档中准确地描述重写任何方法带来的影响。换句话说，该类必须在文档中说明它的可重写方法的自用性（self-use）。对于每个公共或受保护的方法，文档必须指明该方法需要调用哪些可重写方法，以何种顺序以及每次调用的结果将如何影响后续处理。更一般地说，一个类必须在文档中说明任何可能调用可重写方法的情况。

调用可重写方法的方法在文档注释的结尾必须包含对这些调用的描述。这些描述在规范中的特定部分，标记为“Implementation Requirements”，由Javadoc标签@implSpec生成。下面是从`java.util.AbstractCollection`类的规范中拷贝的例子：

		public boolean remove(Object o);
		从该集合中删除指定元素的单个实例（如果存在，optional实例操作）。
		更正式地说，如果这个集合包含一个或多个元素e，且Objects.equals(o, e)返回为true，则将其删除。
		如果此集合包含指定的元素（或者等同于此集合因调用而发生了更改），则返回true。

		实现要求：这个实现迭代遍历集合查找指定元素。 
		如果找到元素，则使用迭代器的remove方法从集合中删除元素。 
		请注意，如果此集合的iterator方法返回的迭代器未实现remove方法，并且此集合包含指定的对象，
		则此实现将引发UnsupportedOperationException异常。

这个文档明确说明了重写`iterator`方法会影响`remove`方法的行为。它还描述了`iterator`方法返回的`Iterator`行为将如何影响`remove`方法的行为。

`@implSpec`标签是在Java 8中添加的，并且在Java 9中被大量使用。这个标签应该默认启用，但是从Java 9开始，除非通过命令行开关-tag “apiNote:a:API Note:”，否则Javadoc实用工具仍然会忽略它。

设计继承不仅仅需要在文档中说明自用的模式。为了让程序员能够写出有效的子类，一个类可能必须提供受保护的方法甚至在某些极端情况下，提供受保护的属性以方便其子类使用这些方法或属性来实现或优化某些操作。例如`java.util.AbstractCollection`的`removeRange`方法就是为了让其子类能够提供快速执行的`clear`方法。

测试为继承而设计的类的唯一方法是编写子类。在其发布之前，必须通过编写子类来测试它。经验表明，通常三个子类足以测试一个可继承的类。这些子类应该由父类作者以外的人编写。

还有一些设计可继承类时必须遵守的限制：

构造方法绝不能直接或间接调用可重写的方法。否则，因为父类构造方法在子类构造方法之前运行，所以在子类构造方法运行之前，子类中的重写方法就会被调用。而如果重写方法依赖于子类构造方法执行的任何初始化，则此方法将不会按预期运行，形成环路。

		public class Super {
			// Broken - constructor invokes an overridable method
			public Super() {
				overrideMe();
			}
			public void overrideMe() {
			}
		}

以下是一个重写overrideMe方法的子类，Super类的唯一构造方法会错误地调用它：

		public final class Sub extends Super {
			// Blank final, set by constructor
			private final Instant instant;

			Sub() {
				instant = Instant.now();
			}

			// Overriding method invoked by superclass constructor
			@Override public void overrideMe() {
				System.out.println(instant);
			}

			public static void main(String[] args) {
				Sub sub = new Sub();
				sub.overrideMe();
			}
		}

这个程序的期望效果是打印两次instant实例，但是它第一次打印出`null`，因为在Sub构造方法有机会初始化instant属性之前，`overrideMe`被Super构造方法调用。 请注意，这个程序观察两个不同状态的final属性！ 还要注意的是，如果`overrideMe`方法调用了instant实例中任何方法，那么当父类构造方法调用`overrideMe`时，它将抛出一个`NullPointerException`异常。这个程序不会抛出`NullPointerException`的唯一原因是`println`方法容忍null参数。

从构造方法中调用私有方法，final方法和静态方法是安全的，因为其中任何一个方法都不可重写。

设计让用于继承的类实现`Cloneable`和`Serializable`接口时需要加倍小心。相关的一些操作在条目13和条目86中有描述。

如果决定在为继承而设计的类中实现`Cloneable`或`Serializable`接口，那么和构造方法类似，`clone`和`readObject`都不能直接或间接调用可重写的方法。

`readObject`方法中，重写方法将在子类的状态被反序列化之前运行。
`clone`方法中，重写方法将在子类的`clone`方法有机会修复克隆的状态之前运行。可能会损坏原始对象以及克隆对象本身。

最后，如果决定让为继承设计的类实现`Serializable`接口，并且该类有一个`readResolve`或`writeReplace`方法，则必须把`readResolve`或`writeReplace`方法设置为受保护而不是私有。如果这些方法是私有的，它们将被子类无声地忽略。

保障继承的健壮性的最好办法是，只允许继承那些专门为继承而设计并有完备文档的类。 有两种方法禁止子类化。两者中较容易的是声明类为final。另一种方法是使所有的构造方法都是私有的或包级私有的，并且添加公共静态工厂来代替构造方法（推荐）。

如果必须允许继承普通类，那么要确保类从不调用任何可重写的方法，并在文档中说明这个事实。换句话说，完全消除类的自使用（self-use）可重写的方法。

一种消除自使用的方法是：将每个可重写的方法的方法体移动到一个私有的“帮助方法”，并让每个可重写的方法调用其私有的帮助方法。然后在需要调用可重写方法的地方用直接调用其专用帮助方法即可。

### 20. 接口优先于抽象类 ###

一个类只能继承一个父类，但可以实现多个接口。一个接口也可以继承多个接口。任何存在的类都可以通过实现接口来轻易的改装。

接口是定义混合类型（mixin）的理想选择。一般来说，mixin是一个类，除了它的“主类型”之外，还可以声明它提供了一些可选的行为。

例如，`Comparable`是一个混合接口，它的实现类的实例是可通过比较来排序的。这样的接口被称为mixin，因为它允许可选功能被“混合”到原有类型的主要功能，换句话说就是为原有类型提供了新的功能。

抽象类不能用于定义混合类，这是因为它们不能被加载到现有的类中：一个类不能有多个父类，并且在类层次结构中没有合理的位置来插入一个混合类型。

接口允许构建非层级类型的框架。传统的类框架需要针对每个受支持的属性组合，创建一个单独的类从而造成臃肿的类层级结构。如果类型系统中有n个属性，则可能需要支持2^n种可能的组合。这就是所谓的组合爆炸（combinatorial explosion）。臃肿的类层级结构可能会导致具有许多方法的臃肿类，这些方法仅在参数类型上有所不同，因为类层级结构中没有类型来捕获通用行为。

如果接口中的某些方法的实现非常明显，会被许多它的实现类共享，则考虑把它们作为默认方法来使用，并确保使用`@implSpec` Javadoc标记（参阅19）将在文档中说明它们会被继承。

默认方法可以提供的帮助是有限制的。尽管许多接口指定了`Object`类中方法（如`equals`和`hashCode`）的行为，但不允许为它们提供默认方法。此外，接口不允许包含实例变量或非公共静态成员（私有静态方法除外）。 最后，不能在不受控制的接口中添加默认方法。

抽象的实现骨架类（abstract skeletal implementation class）将接口和抽象类的优点结合了起来。接口定义了该骨架类的类型，还可以提供一些默认方法，而实现骨架类实现了剩余的接口方法。在骨架类的基础上扩展可以复用骨架类中已实现的方法，避免每次都要重新实现接口的所有方法。这就是模板方法设计模式。

按照惯例，实现骨架类被称为Abstract***Interface***，其中Interface是它所实现接口的名称。例如，集合框架（Collections Framework）提供了一个框架实现以配合每个主要集合接口：`AbstractCollection`，`AbstractSet`，`AbstractList`和`AbstractMap`。

例如，下面是一个静态工厂方法，在`AbstractList`的顶层包含一个完整的功能齐全的`List`实现：

		// Concrete implementation built atop skeletal implementation
		static List<Integer> intArrayAsList(int[] a) {
			Objects.requireNonNull(a);
			// The diamond operator is only legal here in Java 9 and later
			// If you're using an earlier release, specify <Integer>
			return new AbstractList<>() {
				@Override public Integer get(int i) {
					return a[i]; // Autoboxing (Item 6)
				}
				@Override public Integer set(int i, Integer val) {
					int oldVal = a[i];
					a[i] = val; // Auto-unboxing
					return oldVal; // Autoboxing
				}
				@Override public int size() {
					return a.length;
				}
			};
		}

实现骨架类的优点在于，它实际是使用实现的接口来作为它的类型，并提供了最通用的实现模板，因此所有此接口的实现类都可以通过继承它来取得默认的实现并在此基础上进行扩展。如果某个实现类无法继承该骨架类，也可以直接声明为实现了该接口，如果还想使用骨架类中的实现，可以在内部创建一个骨架类类型的变量，然后通过该变量来实现方法跳转。这种方法可以看成是一种对多继承的模拟。

编写一个实现骨架类：

1. 首先确定接口中哪些方法是最基本的，而其他方法都是可以立刻实现的。这些基本方法是实现骨架类中的抽象方法。
2. 接下来，所有可以直接在基本方法之上实现的方法将作为接口中的默认方法
3. 如果还有剩下的方法，就编写一个声明实现接口的类作为骨架类，实现所有剩下的接口方法。此类可能包含任何的非公共成员。

由于实现骨架类是为了继承而设计的，所以应该遵循条目19中的所有设计和文档说明。

与骨架实现有稍许不同的是简单实现，以`AbstractMap.SimpleEntry`为例。一个简单的实现就像一个骨架实现，它实现了一个接口，并且是为了继承而设计的，但是它的不同之处在于它不是抽象的：它是最简单的工作实现。可以按照情况直接使用它，也可以根据情况进行子类化。

小结：接口通常是定义允许多个实现类型的最佳方式。如果需要公布一个重要的接口，应该强烈考虑提供一个抽象的实现骨架类。在可能的情况下，应该通过接口默认方法提供骨架实现，以便接口的所有实现者都可以使用它。

### 21. 为后代设计接口 ###

虽然可以通过添加默认方法为已创建的接口添加新的功能，但这些新的默认方法无法保证总是能够维护那些已有实现类中的不变量。

例如，考虑在Java 8中添加到`Collection`接口的`removeIf`方法。此方法删除给定布尔方法（或`Predicate`函数式接口）返回true的所有元素。默认实现使用迭代器遍历集合，在每个元素上使用给定的布尔式或`Predicate`函数式接口进行判断，并使用迭代器的`remove`方法删除返回为`true`的元素。因此该方法的声明应该看起来像这样：

		// Default method added to the Collection interface in Java 8
		default boolean removeIf(Predicate<? super E> filter) {
			Objects.requireNonNull(filter);
			boolean result = false;
			for (Iterator<E> it = iterator(); it.hasNext(); ) {
				if (filter.test(it.next())) {
					it.remove();
					result = true;
				}
			}
			return result;
		}

这是可能为`removeIf`方法编写的最好的通用实现，但它在一些实际的`Collection`实现中却失败了。

例如，`org.apache.commons.collections4.collection.SynchronizedCollection`类。 这个类出自Apache Commons类库中，与`java.util`包中的静态工厂`Collections.synchronizedCollection`方法返回的类相似。Apache版本提供了使用客户端提供的对象来代替集合本身进行锁定的功能。换句话说，它是一个包装类（参阅18），它们的所有方法在委托给所包装的集合类之前需要在一个锁定对象上进行同步。

Apache的`SynchronizedCollection`类仍然在积极维护，但在撰写本文时，并未重写`removeIf`方法。如果这个类与Java 8一起使用，它将继承`removeIf`的默认实现，但实际上不能保证该类的基本承诺：自动同步每个方法调用。 `remove`默认实现对同步一无所知，并且不能访问用于锁定的对象。如果客户端在另一个线程同时修改集合的情况下调用`SynchronizedCollection`实例上的`removeIf`方法，则可能会导致`ConcurrentModificationException`异常或其他未指定的行为。

如果为已有接口添加了新的默认方法，该接口的现有实现类可能在没有错误或警告的情况下编译，但在运行时失败。所以应该避免使用默认方法向现有的接口添加新的方法，如非常有必要，也应该确定现有的接口实现是否会被所添加的默认方法破坏。

默认方法不是被设计用来从接口中移除方法或者改变现有方法的签名。因为这样会破坏现有的实现类。因此，在发布之前测试每个新接口是非常重要的。多个程序员应该以不同的方式实现每个接口。至少应该准备三种不同的实现。

### 22. 接口仅用来定义类型 ###

当一个类实现了某个接口时，该接口将作为这个类的实例的引用类型来被使用。基于其他任何目的来使用接口都是不正确的。

一种不符合此要求的不良示范是：constant interface（常量接口）。这种接口不包含任何方法; 它仅由静态final变量组成，每个变量都代表一个常量。类通过实现此接口来使用这些常量。

		// Constant interface antipattern - do not use!
		public interface PhysicalConstants {
			// Avogadro's number (1/mol)
			static final double AVOGADROS_NUMBER = 6.022_140_857e23;
			// Boltzmann constant (J/K)
			static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
			// Mass of the electron (kg)
			static final double ELECTRON_MASS = 9.109_383_56e-31;
		}

类在内部使用常量完全属于实现细节。实现一个常量接口会导致这个实现细节泄漏到类的公共API中。如果在将来的版本中修改了类，不再需要使用这些常量，但它仍然必须实现该接口，以确保版本的兼容性。如果一个非final类实现了常量接口，那么其所有子类都会受到该问题的影响。

如果你想导出常量，有几个合理的选择方案。

- 如果常量与现有的类或接口紧密相关，则应将其添加到该类或接口中。例如，所有数字基本类型的包装类，如`Integer`和`Double`，都会导出`MIN_VALUE`和`MAX_VALUE`常量。
- 如果常量最好被看作枚举类型的成员，则应该使用枚举类型导出它们。
- 用一个不可实例化的工具类来导出常量（参阅4）。 

前面所示的PhysicalConstants示例的工具类的版本：

		// Constant utility class
		package com.effectivejava.science;

		public class PhysicalConstants {
		  private PhysicalConstants() { }  // Prevents instantiation

		  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
		  public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;
		  public static final double ELECTRON_MASS    = 9.109_383_56e-31;
		}

注意在数字文字中合理使用下划线字符（`_`）。从Java 7开始，合法的下划线对数字字面量的值没有影响，但是如果使用得当的话可以使它们更容易阅读。无论是固定的浮点数，如果他们包含五个或更多的连续数字，考虑将下划线添加到数字字面量中。对于底数为10的数字，无论是整型还是浮点型的，都应该用下划线将数字分成三个数字组，表示一千的正负幂。

通常要求客户端使用工具类的类名来限定常量名，例如`PhysicalConstants.AVOGADROS_NUMBER`。如果大量使用实用工具类导出的常量，则通过使用静态导入来限定具有类名的常量：

		// Use of static import to avoid qualifying constants
		import static com.effectivejava.science.PhysicalConstants.*;

		public class Test {
			double  atoms(double mols) {
				return AVOGADROS_NUMBER * mols;
			}
			...
			// Many more uses of PhysicalConstants justify static import
		}

### 23. 类层次结构优于标记类 ###


标记类是指一个类的实例可能有不同的风格，而该类在内部使用一个变量来标记这些风格。

		// Tagged class - vastly inferior to a class hierarchy!
		class Figure {
			
			enum Shape { RECTANGLE, CIRCLE };
			
			// Tag field - the shape of this figure
			final Shape shape;
			
			// These fields are used only if shape is RECTANGLE
			double length;
			double width;
			
			// This field is used only if shape is CIRCLE
			double radius;
			
			// Constructor for circle
			Figure(double radius) {
				shape = Shape.CIRCLE;
				this.radius = radius;
			}
			
			// Constructor for rectangle
			Figure(double length, double width) {
				shape = Shape.RECTANGLE;
				this.length = length;
				this.width = width;
			}
			
			double area() {
				switch(shape) {
					case RECTANGLE:
						return length * width;
					case CIRCLE:
						return Math.PI * (radius * radius);
					default:
						throw new AssertionError(shape);
				}
			}
		}

标记类的缺点：

- 杂乱无章，包含了枚举声明，标记变量和switch语句，可读性差。
- 内存占用量增加，因为实例包含了属于其他风格的不相关变量。构造方法初始化不相关的变量以便将这些变量设为final。
- 构造方法必须设置标记变量并在没有编译器帮助的情况下初始化正确的属于相应风格的数据变量：如果初始化错误的变量，程序将在运行时失败。
- 需要添加或更改风格时，必须修改源代码。如果添加了一个风格，则必须在每个`switch`语句中添加一个`case`，否则将在运行时失败。
- 实例的数据类型没有给出其所属风格的线索。

事实上，标记类只是对类层次结构的一种低效模仿。

使用类层次结构来改写标记类的步骤：

1. 定义抽象类。标记类中每个依赖于风格（标记值）的方法在该抽象类中都有一个抽象方法相对应。在`Figure`类中，只有一个这样的方法，即`area`。此抽象类是类层次结构的根。将任何行为不依赖于标记值的方法放在此类中，并且将所有风格都使用的数据变量也放在此类中。
2. 定义子类。为原始标记类的每个风格定义具体子类。在示例中，有两个：圆和矩形。在每个子类中包含特定于其风格的数据变量。在示例中，半径特定于圆，长度和宽度特定于矩形。在每个子类中正确实现抽象父类中的抽象方法。

改写后的代码如下：

		// Class hierarchy replacement for a tagged class
		abstract class Figure {
			abstract double area();
		}

		class Circle extends Figure {
			final double radius;
			Circle(double radius) { this.radius = radius; }
			@Override double area() { return Math.PI * (radius * radius); }
		}

		class Rectangle extends Figure {
			final double length;
			final double width;
			Rectangle(double length, double width) {
				this.length = length;
				this.width = width;
			}
			@Override double area() { return length * width; }
		}

每种风格都有相应的子类来实现，而这些类都没有包含无关的数据变量。所有的变量也都是final的。编译器确保每个类的构造方法初始化其数据属性，并且每个类都有属于自己的父类抽象方法的实现。消除了由于缺少switch-case语句而导致的运行时失败的可能性。
多个程序员可以独立地继承类层次，无需访问根类的源代码。每种风格都有一个独立的数据类型与之相关联，允许程序员指出变量的类型，并将变量和输入参数限制为特定的类型。
反映了类型之间的自然层次关系，提高了编译时类型检查的效率。

小结：带有明显标签属性的类，可以考虑使用类层次结构将其重构。

### 24. 静态成员类优于非静态成员类 ###

嵌套类是指在其他类内部定义的类，它应该只为其宿主类提供服务，否则应该将其取出作为顶层类使用。

有四种嵌套类：静态成员类，非静态成员类，匿名类和本地类。除第一种之外的其他类型都被称为内部类。

静态成员类可以看作是恰好在其他类中声明的普通类，可以访问宿主类的所有成员包括私有成员。

静态成员类的最常见用途是作为公共帮助类，仅在与其外部类一起使用时才有用。例如，考虑一个描述计算器所支持操作的枚举类型（参阅34）。 `Operation` 枚举应该是`Calculator`类的公共静态成员类。`Calculator`客户端可以使用`Calculator.Operation.PLUS`和`Calculator.Operation.MINUS`等名称来引用操作。

非静态成员类和静态成员类之间的唯一区别是静态成员类在其声明中具有`static`修饰符。非静态成员类的每个实例都与其宿主类的某个实例相关联。在非静态成员类的实例方法中，可以调用宿主实例的方法，或者使用`OutterClass.this`获得对宿主实例的引用。如果嵌套类的实例可以与其宿主类的实例分开单独存在，那么该嵌套类必须是静态成员类。不可能在没有宿主实例的情况下创建非静态成员类的实例。

非静态成员类实例和其宿主实例之间的关联是在创建成员类实例时建立的，并且之后不能被修改。通常情况下，在宿主类的实例方法中调用非静态成员类构造方法会自动建立关联。

非静态成员类的常见用法是定义一个`Adapter`，它可以让外部类的实例以不相关类型的实例的形式展现出来。例如，`Map`接口的实现通常使用非静态成员类来实现它们的集合视图，这些视图由`Map`的`keySet`，`entrySet`和`values`方法返回。同样，集合接口（如`Set`和`List`）的实现通常使用非静态成员类来实现它们的迭代器：

		// Typical use of a nonstatic member class
		public class MySet<E> extends AbstractSet<E> {
			... // Bulk of the class omitted

			@Override public Iterator<E> iterator() {
				return new MyIterator();
			}

			private class MyIterator implements Iterator<E> {
				...
			}
		}

不需要访问宿主实例的成员类应该声明为静态成员类。 

私有静态成员类的常见用法是用来表示宿主类对象中的组件。例如，将键与值相关联的`Map`实例。许多`Map`实现对每一对键-值对都有一个内部的`Entry`对象相对应。尽管每个entry对象都与一个`Map`实例相关联，entry上的方法(`getKey`，`getValue`和`setValue`)却不需要访问该`Map`实例，因此应该使用私有静态成员类。

匿名类在使用时同时声明和实例化。当且仅当出现在非静态上下文中时，匿名类才会有宿主类实例。但是，即使出现在静态上下文中，它们也不能拥有除常量型变量之外的任何静态成员，这些常量型变量包括final的基本数据类型或者使用常量表达式初始化的字符串变量。

匿名类的适用性有很多限制：

- 除了声明的时候之外，不能实例化它们。
- 不能使用`instanceof`方法测试或者做任何其他需要使用到类名的操作。
- 不能声明一个匿名类来实现多个接口，或者继承一个类并同时实现一个接口。
- 匿名类的客户端不能调用除继承自父类型的成员以外的任何成员。
- 因为匿名类在表达式中出现，所以它们必须保持短——约十行或更少。

匿名类的常见用途是实现静态工厂方法。

一个局部类可以在任何可以声明局部变量的地方声明，并遵守相同的作用域规则。局部类与其他类型的嵌套类具有共同的属性。像成员类一样，他们有名字，可以重复使用。就像匿名类一样，只有在非静态上下文中定义它们时，它们才会包含实例，并且它们不能包含静态成员。像匿名类一样，应该保持简短，以免损害可读性。

小结：如果一个成员类的每个实例都需要一个对其宿主实例的引用，则将其声明为非静态的，否则声明为静态。如果这个类只属于某个方法的内部且只需要从一个地方创建实例，并且存在一个预置类型来作为该类的类型，那么使用匿名类，否则使用局部类。

### 25. 每个源代码文件只应该包含一个顶层类 ###


在源文件中定义多个顶层类使得为类提供多个定义成为可能。具体使用哪个定义会受到源文件传递给编译器的顺序的影响。可以使用静态成员类（参阅24）作为将多个类拆分为单独的源文件的替代方法。

如果这些类都服务于另一个类，那么将它们变成静态成员类通常是更好的选择，因为它提高了可读性，并且可以通过将它们声明为私有（参阅15）来减少类的可访问性。

永远不要将多个顶级类或接口放在一个源文件中。

### 26. 不要使用原生类型  ###

一个在定义声明中拥有一个或多个类型参数的类或接口称之为泛型类或泛型接口。二者统称为泛型类型。`List<E>`

每一种泛型类型都定义了一组参数化类型，即在类名或接口名后用一对尖括号为其定义中的形式类型参数指定相对应的实际类型参数，`List<String>`。每一种泛型类型也定义了一个相应的原生类型（raw type），表现为不带实际类型参数的泛型类型。原生类型事实上已经退出了泛型的使用，它的存在纯粹是为了向上兼容的考虑。`List`

`List<String>`是原生类型`List`的子类，但不是`List<Object>`的子类。

如果需要使用泛型类型，但又不知道类型参数的实际类型或根本不关心它的实际类型，就可以使用无限制通配符`?`代替。

但是不能往一个`Collection<?>`里写入除`null`外的任何元素，也不能判定从该集合中取出的元素的实际类型，如果想克服这些局限，可以使用泛型方法或带边界的通配符类型。

必须使用原生类型的场景：

- 在使用类对象的直接量时。类对象的直接量可以接受数组和基本类型，但不接受泛型类型。即`Stirng[].class`, `int.class`, `List.class`都合法，但`List<String>.class`不合法。
- 在使用`instanceof`操作符时，只能接受原生类型和无边界的通配符类型，即<?>。但对二者的处理没有太大差别，因此推荐的使用方法是：

		if (o instanceof Set) { 	//raw type
			Set<?> s = (Set<?>) o; 	//wildcard type
		}

注意一旦确定了o的类型是Set，就应该使用无边界的通配符强制转型，在这里不要使用原生类型，从而避免编译器报错。

`Set<Object>`, `Set<?>`, `Set`的区别：

- `Set<Object>`：表示Set中可存储任何类型的对象，也就是说可能有不同类型的对象存储在该Set中
- `Set<?>`：Set中存储且仅仅存储了某种未知类型的对象，不可能有其他类型的对象存储在该Set中
- `Set`：原生类型，完全没有使用泛型

前两种是类型安全的，第三种不是。

术语                    | 例子                               | 相关条目
-----                   | -----                              | -----
Parameterized type      | `List<String>`                     | 26
Actural type parameter  | `String`                           | 26
Generic type            | `List<E>`                          | 26, 29
Formal type parameter   | `E`                                | 26
unbounded wildcard type | `List<?>`                          | 26
raw type                | `List`                             | 26
bounded type parameter  | `<E extends Number>`               | 29
recursive type bound    | `<T extends Comparable<T>>`        | 30
bounded wildcard type   | `List<? extends Number>`           | 31
generic method          | `static <E> List<E> asList(E[] a)` | 30
type token              | `String.class`                     | 33

### 27. 解决未检测警告  ###

尽可能解决每一个产生的未检测的警告。

如果无法解决产生的unchecked warning，而我们又可以确认产生警告的代码是类型安全的，就可以使用`@SuppressWarnings`注解来消除这些警告。

`@SuppressWarnings`可以用在任何声明上，从单独的局部变量到整个类。但尽量将它的作用范围控制在最小，通常是一个变量的声明，或者一个简短的方法或构造方法。永远不要将它作用于整个类上。

如果发现必须在一个拥有超过1行代码的带有返回值的方法或构造方法上使用`@SuppressWarnings`注解，你可能可以将其作用范围缩小到方法中的某个局部变量的声明上，虽然你可能因此要声明一个新的局部变量（来引用引发警告的方法的返回值）。

不能在return语句上使用`@SuppressWarnings`注解。

每次使用`@SuppressWarnings("unchecked")`时，都应该添加注释说明这样做的原因。

小结：每个unchecked warning都代表了一种在运行时出现`ClassCastException`的可能。尽量解决发现的每个unchecked warning，如果无法解决，且确定引起该警告的代码是类型安全的，就在最小的作用范围内使用`@SuppressWarnings("unchecked")`注解来消除该警告，并在注释中说明原因。

### 28. 列表优先于数组 ###

数组是协变类型(covariant)。也就是说如果`Sub`类是`Super`类的子类，那么`Sub[]`数组也是`Super[]`数组的子类；泛型则相反，是不可变类型(invariant)，即对于任意两种不同的类型`Type1`和`Type2`，不管他们之间是否有继承关系，`List<Type1>`都不可能是`List<Type2>`的子类。

第二个主要差别在于数组是具体化(reified)的，数组在运行期是知道并确保它所包含的元素的类型。而泛型则是通过擦除(erasure)来实现，即泛型只会在编译期确保所包含元素的类型，在运行期就会删除所有的元素类型信息。

数组和泛型无法在一起混合使用，比如不能创建一个泛型类型的数组(List<E>[])，一个带实际参数的泛型类型的数组(List<String>[])，一个形式类型参数的数组(E[])。这些都会导致在运行期发生错误。

`List<E>`/`List<String>`/`E`都被称之为不可具体化类型(non-reifiable types)，即这些类型在运行期中所包含的信息少于他们在编译期中所包含的信息。唯一合法的可具体化的参数化类型是使用无边界通配符`<?>`的类型，如`List<?>`。然而，虽然合法，但很少创建使用无限制通配符的类型的数组，因为这样的集合无法写入，读出的数据也无法判断其真实类型，只能调用集合本身的方法做一些基于集合自身而不是元素的操作。

通常不可能让一个泛型集合返回一个所包含元素类型的数组。如果在方法的可变参数中使用泛型类型也会发出警告，因为可变参数背后的实现机制就是创建了一个数组来保存这些参数。可以使用`SafeVarargs`注解来消除这个警告。

当你遇上一个创建泛型数组错误或者一个未检测的数组转型警告，最好的解决方法就是使用集合`List<E>`代替数组`E[]`。

小结：数组是协变和具体化的，提供了运行期安全而不是编译期安全；泛型是不可变和被擦除的，提供了编译期安全，而不是运行期安全。二者通常不能混用，如果在使用二者时发生错误，应该首先考虑使用列表代替数组。

### 29. 多使用泛型类型 ###

泛型化一个类的方法：

1. 在这个类的声明中加入一个或多个形式类型参数
2. 在类的内部把所有对`Object`类型的引用都替换成相应的形式类型参数。

当类中需要使用泛型参数类型的数组`E[]`时的解决方法：

- （推荐）在确保类型安全的前提下，直接将`Object[]`数组强制转型为相应的形式类型数组`E[]`，然后使用`@SuppressWarnings`注解来关闭警告。
- 在确保类型安全的前提下，将每次从`Object[]`数组中取出的元素强制转型为相应的形式类型`E`，然后使用`@SuppressWarnings`注解来关闭警告。

第一种方法因为只需要强制转换类型一次，所以更常用，但也可能造成堆污染：数组在运行时的类型和在编译时的类型不一致。

不能把基本类型作为实际类型参数传给泛型类型。

### 30. 多使用泛型方法 ###

操作参数化类型的静态工具方法通常都是泛型方法。

泛型方法的类型参数列表处在方法的修饰符和返回类型之间。

泛型单例工厂：使用一个不变的对象来代表不同的参数化类型，但需要一个静态方法来为需要的参数化类型返回该对象。

		//generic singleton factory pattern
		private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

		@SuppressWarnings("unchecked")
		pubic static <T> UnaryOperator<T> identityFunction() {
			//这里的强制类型转换会发出警告，因为UnarayOperator<Object>不一定能转换成UnaryOperator<T>
			return (UnaryOperator<T>) IDENTITY_FN;
		}

		//sample program to exercise generic singleton
		public static void main(String[] args) {
			String[] strings = {"tom", "john", "jack"};
			UnaryOperator<String> sameString = identityFunction();
			for (String s : strings) 
				System.out.println(sameString.apply(s));
			
			Number[] numbers = {1, 2.0, 3L};
			UnaryOperator<Number> sameNumber = identityFunction();
			for (Number n : numbers)
				System.out.println(sameNumber.apply(n));
		}

递归类型限定：类型参数被包含了类型参数自身的表达式限定。最常见的例子是和`Comparable<T>`接口联合使用，如许多方法会对一个其元素实现了该接口的集合进行排序，搜索，计算最大最小值等操作。这就需要集合的元素能够互相比较：

		//使用递归类型限定来体现互比性
		public static <E extends Comparable<E>> E max(Collection<E> c);

限定边界`<E extends Comparable<E>>`读为“所有可以和自身比较的类型E”。

### 31. 使用边界通配符来增加API的灵活性 ###

PECS(Producer-extends, Consumer-super)：

在使用参数化类型如`Collection<T>`时，

- 生产者：如果只读取了类型参数T所代表类型的实例，则类型化参数使用`extends`，如`Collection<? extends T>`，类型参数代表的类型的数据只外流而不会内流。
- 消费者：如果只需要写入类型参数T所代表类型的实例，则类型化参数使用`super`，如`Collection<? super T>`，类型参数代表的类型的数据只内流而不会外流。
- 如果既是生产者，也是消费者，则不能使用通配符，必须使用确切的类型`T`。
- 所有的可比较类和所有的比较器都是消费者。

不要在返回类型中使用边界通配符。合理地使用通配符类型可以让类自然接受应该接受的参数，同时拒绝应该拒绝的参数。如果一个类的用户必须思考为什么某方法的参数使用通配符类型，那么这个API的设计一定有问题。

如果一个类没有直接实现某个接口，而是继承了该接口的某个实现类，则需要使用通配符来支持使用和该接口有关的方法。

		//例如ScheduledFuture类没有直接实现Comaparable<ScheduledFuture>，
		//而是继承了Delayed类，该类实现了Comparable<Delayed>
		//因此，下面这个定义的方法将不起作用
		public static <T extends Comparable<T>> T max(List<T> list)
		//下面是改良过的方法
		/public static <T extends Comparable<? super T>> T max(List<? extends T> list)

如果类型参数只在方法的声明中出现了一次，则使用通配符来代替它:

- `T` -> `<?>`
- `T extends Type` -> `<? extends Type>`

但由于我们不能往使用无限制通配符`<?>`的集合中存放除了`null`的任何元素，我们需要一个私有的辅助方法来捕获该通配符所的实际类型。

		public static void swap(List<?> list, int i, int j) {
			swapHelper(list, i, j);	
		}

		//用于捕获通配符类型的辅助方法
		private static <E> void swapHelper(List<E> list, int i, int j) {
			list.set(i, list.set(j, list.get(i)));
		}

`swapHelper()`方法使用了类型参数，所以知道通过`get()`取出的元素的类型和把该类型的元素再放回该集合是类型安全的。

### 32. 谨慎搭配泛型和可变参数 ###

无论是在方法声明中使用不可具体化的参数类型，还是在调用方法时传入不可具体化类型的参数，编译器都会发出警告说可能产生堆污染。

堆污染(Heap pollution)：一个参数化类型的变量指向了一个非本类型的对象。

尽管无法显式创建一个泛型数组，但声明一个使用泛型可变参数的非法却是合法的，虽然将值保存在泛型可变参数中并不安全。

Java7以前，只能使用`@SuppressWarnings("unchecked")`来消除因此产生的警告。Java7以后方法的作者可以使用`@SafeVarargs`注解来注明该方法是类型安全的。

如何判断一个使用了泛型可变参数的方法是类型安全的？
	在调用该方法时，会创建一个泛型数组来保存传入参数，如果该方法没有对该数组做任何写操作（可能改变参数值），也没有让任何指向该数组的引用外泄（可能被不信任的外部代码访问），则该方法是安全的。换句话说，如果可变参数数组仅仅是方法调用者用来向方法传递数量可变的输入参数，该数组仅在方法内部可见并且只读不写，则该方法是安全的。

通常让其他方法可以访问泛型可变参数数组是不安全的，二种情况例外：

- 将数组传递给另外一个被`@SafeVarargs`注解了的同样使用可变参数的方法；
- 将数组传递给一个未使用可变参数的方法，该方法仅仅使用了数组元素的某些不改变元素状态的功能。

每一个使用了泛型可变参数的方法都应该确保自己是类型安全的，并使用`@SafeVarargs`加以注解。注意此注解只用于不能被重写的方法上。Java8中，该注解只能用于静态方法和final实例方法；Java9中则也可以用于私有的实例方法。

一种替代`@SafeVarargs`注解的方案是参照原则28，使用列表代替可变参数数组。

小结：虽然泛型可变参数数组是不安全的，会造成堆污染，但却是合法的。因此设计使用泛型可变参数的方法时，都应该通过不在方法体内往该数组中再存放数据以及不将其暴露给其他方法来确保该方法是类型安全的，并用`@SafeVarargs`注解标明。

### 33. 考虑使用类型安全的混合的容器 ###

在容器的声明中直接使用的类型参数通常指代了能够在容器中存储或使用的数据的类型，虽然这些类型参数在容器被使用时会被替换成实际类型参数，但一旦指定，容器中就只能存放或处理同一种类型的数据，也就是说每次使用容器实际处理的数据类型是有限的。如果需要在容器中存放各种不同类型的数据，则可以考虑参数化键值而不是参数化容器本身，然后通过参数化键值来插入或删除对应的数据。

		//类型安全的混合的容器的API模板
		public class Favorites {
			public <T> void putFavorite(Class<T> type, T instance);
			public <T> T getFavorite(Class<T> type);
		}
		//类型安全的混合的容器的实现模板
		public class Favorites {
			private Map<Class<?>, Object> favorites = new HashMap<>();

			public <T> void putFavorite(Class<T> type, T instance) {
				favorites.put(Objects.requireNonNull(type), instance);
			}
			public <T> T getFavorite(Class<T> type) {
				//使用Class的cast()方法来动态转型，执行效果：
				//(anInstance instanceof Type ? ((Type) anInstance) : (throws ClassCastException)
				return type.cast(favorites.get(type));	
			}
		}

		//使用举例
		public static void main(String[] args) {
			Favorites f = new Favorites();
			f.putFavorite(String.class, "java");
			f.putFavorite(Integer.class, 0xcafebabe);
			f.putFavorite(Class.class, Favorites.class);
			String favoriteString = f.getFavorite(String.class);
			int favoriteInteger = f.getFavorite(Integer.class);
			Class<?> favoriteClass = f.getFavorite(Class.class);
			System.out.println("%s, %x %s%n", 
								favoriteString, 
								favoriteInteger, 
								favoriteClass.getName());
		}

`Favorites`类有两个限制：

- 恶意代码可以通过使用`Class`的原生类型对象来破坏`Favorites`类的实例的类型安全。解决方法是在跑`putFavorite()`方法中加入动态转型从而确保类型安全。
	
		//通过动态转型确保运行时类型安全
		public <T> void putFavorite(Class<T> type, T instance) {
			favorites.put(type, type.cast(instance));
		}

在`java.util.Collections`类中有类似功能的集合封装方法`checkedSet, checkedList, checkedMap`等等，这些方法接受1个或2个`Class`对象和一个集合，确保`Class`对象和集合元素的类型在编译时一致，从而保证集合的类型安全，让泛型集合可具体化。

- 第二个限制是不能使用`Favorite`类来处理不可具体化类型，即可以存储`String`，`String[]`，但不能存储`List<String>`，因为没有泛型类型的`Class`对象。

类型安全的混合的容器的一个应用：一个被注解的元素就是一个类型安全的混合容器，它的键值就是注解的类型。

小结：泛型在集合的API中经常使用，用于限制每个容器可以存放的数据的类型。但是我们可以通过参数化键值而不是直接参数化容器本身来打破这种限制。这个参数化键值通常是`Class`对象。此时，该`Class`对象被称为类型标记(type token)。

### 34. 使用枚举代替int常量 ###


Java枚举类型背后的基本思想很简单：通过公共静态final属性为每个枚举常量导出一个实例。由于没有可访问的构造方法，枚举类型实际上是final的。由于客户既不能创建枚举类型的实例也不能继承它，除了声明的枚举常量外，不能有任何实例。它们是单例类的泛化，单例类基本上就是单元素的枚举。

枚举的优点：

- 枚举提供了编译时类型的安全性。尝试传递错误类型的值，或将一个枚举类型的表达式分配给另一个类型的变量，或使用==运算符来比较不同枚举类型的值，将导致编译时错误。
- 具有相同名称常量的枚举类型可以和平共存，因为每种类型都有其自己的名称空间。可以在枚举类型中添加或重新排序常量，而无需重新编译其客户端。
- 可以通过调用其`toString`方法将枚举转换为可打印的字符串。
- 允许添加任意方法和属性并实现任意接口。它们提供了所有Object方法的高质量实现，并实现了Comparable和Serializable。

枚举类型可以作为枚举常量的简单集合，并随着时间的推移而进化出更丰富的功能。太阳系的八颗行星就是一个很好的例子。每个行星都有质量和半径，从这两个属性可以计算出它的表面重力。从而在给定物体的质量下，计算出一个物体在行星表面上的重量。下面是这个枚举类型。每个枚举常量之后的括号中的数字是传递给其构造方法的参数。在这种情况下，它们是星球的质量和半径：

		// Enum type with data and behavior
		public enum Planet {
			MERCURY(3.302e+23, 2.439e6),
			VENUS  (4.869e+24, 6.052e6),
			EARTH  (5.975e+24, 6.378e6),
			MARS   (6.419e+23, 3.393e6),
			JUPITER(1.899e+27, 7.149e7),
			SATURN (5.685e+26, 6.027e7),
			URANUS (8.683e+25, 2.556e7),
			NEPTUNE(1.024e+26, 2.477e7);

			private final double mass;           // In kilograms
			private final double radius;         // In meters
			private final double surfaceGravity; // In m / s^2
			// Universal gravitational constant in m^3 / kg s^2
			private static final double G = 6.67300E-11;

			// Constructor
			Planet(double mass, double radius) {
				this.mass = mass;
				this.radius = radius;
				surfaceGravity = G * mass / (radius * radius);
			}

			public double mass()           { return mass; }
			public double radius()         { return radius; }
			public double surfaceGravity() { return surfaceGravity; }

			public double surfaceWeight(double mass) {
				return mass * surfaceGravity;  // F = ma
			}
		}

如果需要让枚举值带有属性，可以声明实例属性并编写相应的构造方法，构造方法中设置这些属性。所有的属性都应该是final的。属性可以是公开的，但最好将它们设置为私有并提供公共访问方法。

在上例中，构造方法还计算和存储了表面重力，但这只是一种优化。可以在重力被`SurfaceWeight`方法使用时再从质量和半径重新计算出来。

所有的枚举类都有一个静态`values`方法，该方法以声明的顺序返回其值的数组。

如果从枚举类型中移除了一个枚举值，没有使用该枚举值的已有客户端程序将继续正常工作。使用了该枚举值的客户段会在重新编译时失败并在引用该枚举值的地方提供有用的错误消息; 如果无法重新编译客户端，它将在运行时从此行中引发有用的异常。

一些与枚举常量相关的行为如果只需要在定义该枚举类型的类或包中使用，那么它们最好以私有或包级私有方式实现。这样，每个枚举常量都会有自己的行为集合。

如果一个枚举是广泛使用的，它应该是一个顶级类; 如果它的使用与特定的顶级类绑定，它应该是该顶级类的成员类。

如果需要为每个枚举值定义自己的行为，一种方法是使用`switch`代码块。例如，假设正在编写枚举类型来表示计算器上的四种基本操作，并且希望提供一种方法来执行由每个常量表示的算术运算。

		// Enum type that switches on its own value - questionable
		public enum Operation {
			PLUS, MINUS, TIMES, DIVIDE;

			// Do the arithmetic operation represented by this constant
			public double apply(double x, double y) {
				switch(this) {
					case PLUS:   return x + y;
					case MINUS:  return x - y;
					case TIMES:  return x * y;
					case DIVIDE: return x / y;
				}
				throw new AssertionError("Unknown op: " + this);
			}
		}

这种方法的问题是：

- 如果没有throw语句，就不能编译，因为该方法的结束在技术上是可达到的，尽管它永远不会被达到。
- 如果添加新的枚举常量，但忘记向`switch`语句添加相应的条件，仍然会编译成功，但在尝试应用新操作时，将在运行时失败。

一种更好的方法可以将不同的行为与每个枚举常量关联起来：在枚举类型中声明一个抽象的`apply`方法，并在常量自己的类体中重写它。这种方法被称为特定于常量（constant-specific）的方法实现：

		// Enum type with constant-specific method implementations
		public enum Operation {
		  PLUS  {public double apply(double x, double y){return x + y;}},
		  MINUS {public double apply(double x, double y){return x - y;}},
		  TIMES {public double apply(double x, double y){return x * y;}},
		  DIVIDE{public double apply(double x, double y){return x / y;}};

		  public abstract double apply(double x, double y);
		}

此时添加新的常量，不可能会忘记提供`apply`方法，因为该方法紧跟在每个常量声明之后。

特定于常量的方法实现可以与特定于常量的数据结合使用。例如，下面这个`Operation`类重写了`toString`方法以返回通常与该操作关联的符号：

		// Enum type with constant-specific class bodies and data
		public enum Operation {
			PLUS("+") {
				public double apply(double x, double y) { return x + y; }
			},
			MINUS("-") {
				public double apply(double x, double y) { return x - y; }
			},
			TIMES("*") {
				public double apply(double x, double y) { return x * y; }
			},
			DIVIDE("/") {
				public double apply(double x, double y) { return x / y; }
			};
			private final String symbol;
			Operation(String symbol) { this.symbol = symbol; }
			@Override public String toString() { return symbol; }
			public abstract double apply(double x, double y);
		}

枚举类型具有自动生成的valueOf(String)方法，该方法将常量名称转换为常量本身。如果在枚举类型中重写了`toString`方法，也应该考虑编写`fromString`方法将自定义字符串表示转换为相应的枚举值。 下面的代码（类型名称被适当地改变）对任何枚举都有效，只要每个常量具有唯一的字符串表示形式：

		// Implementing a fromString method on an enum type
		private static final Map<String, Operation> stringToEnum =
				Stream.of(values()).collect(
					toMap(Object::toString, e -> e));

		// Returns Operation for string, if any
		public static Optional<Operation> fromString(String symbol) {
			return Optional.ofNullable(stringToEnum.get(symbol));
		}

`Operation`枚举常量在静态变量`stringToEnum`被初始化时放在了此map中，此变量的初始化是在完成了创建所有的枚举值之后进行的 。前面的代码在`values()`方法返回的数组上使用了流；在Java 8之前的传统做法是创建一个空的`hashMap`并遍历该数组，再将字符串到枚举值的映射放入map中。需要注意：每个常量无法通过其构造方法来将自己放入map中，这会导致编译错误。枚举类型的构造方法不允许访问除常量以外的其他静态变量。因为静态变量在枚举构造方法运行时尚未初始化。枚举常量也不能在构造方法中相互访问。

`fromString` 方法返回一个`Optional`。这允许表示传入的字符串不代表有效的操作，从而强迫客户端处理这种情况。

特定于常量的方法实现的一个缺点是它们使得难以在枚举常量之间共享代码。例如，考虑一个代表工资包中的工作天数的枚举。该枚举有一个方法，根据工人的基本工资（每小时）和当天工作的分钟数计算当天工人的工资。在五个工作日内，任何超过正常工作时间的工作都会产生加班费；在两个周末的日子里，所有工作都会产生加班费。

方法1：使用`switch`语句，通过在代码段上应用将多个`case`标签来实现代码复用：

		// Enum that switches on its value to share code - questionable
		enum PayrollDay {
			MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
			SATURDAY, SUNDAY;

			private static final int MINS_PER_SHIFT = 8 * 60;

			int pay(int minutesWorked, int payRate) {
				int basePay = minutesWorked * payRate;
				int overtimePay;
				switch(this) {
				  case SATURDAY: case SUNDAY: // Weekend
					overtimePay = basePay / 2;
					break;
				  default: // Weekday
					overtimePay = minutesWorked <= MINS_PER_SHIFT ?
					  0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
				}

				return basePay + overtimePay;
			}
		}

缺点是当增加了新的枚举值时，必须在`switch`代码块中加入相应的处理。

方法2： 使用特定于常量的方法实现。此时必须在每个常量的类体中重复计算加班工资的代码，或将计算代码移至两个辅助方法，分别用于工作日和周末，再在每个常量的类体中并调用适当的辅助方法。缺点是都会产生相当数量的样板代码，降低了可读性并增加了出错机会。

一种优化方法是用计算工作日加班费用的方法实现来代替抽象的`overtimePay`方法，这样只有周末的日子必须重写该方法。但是，这与`switch`语句具有相同的缺点：如果在不重写`overtimePay`方法的情况下添加新的枚举常量，就会默认继承工作日的计算方式。

如果需要每次添加枚举常量时都必须指定加班费策略，则可以将加班费计算移入私有嵌套枚举中，并将此策略枚举的枚举值作为参数传递给`PayrollDay`枚举值的构造方法。然后，`PayrollDay`枚举值将加班工资计算委托给策略枚举值来执行，此种方法可以看做是特定于常量的方法实现的变型，它把特定的方法绑定到了私有嵌套枚举类的枚举值当中：

		// The strategy enum pattern
		enum PayrollDay {
			MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
			SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

			private final PayType payType;

			PayrollDay(PayType payType) { this.payType = payType; }
			PayrollDay() { this(PayType.WEEKDAY); }  // Default

			int pay(int minutesWorked, int payRate) {
				return payType.pay(minutesWorked, payRate);
			}

			// The strategy enum type
			private enum PayType {
				WEEKDAY {
					int overtimePay(int minsWorked, int payRate) {
						return minsWorked <= MINS_PER_SHIFT ? 0 :
						  (minsWorked - MINS_PER_SHIFT) * payRate / 2;
					}
				},
				WEEKEND {
					int overtimePay(int minsWorked, int payRate) {
						return minsWorked * payRate / 2;
					}
				};

				abstract int overtimePay(int mins, int payRate);
				private static final int MINS_PER_SHIFT = 8 * 60;

				int pay(int minsWorked, int payRate) {
					int basePay = minsWorked * payRate;
					return basePay + overtimePay(minsWorked, payRate);
				}
			}
		}

基于枚举常量的`switch`语句适合于当无法直接修改枚举类型源代码时，使用特定于常量的行为来增加枚举类型。

		// Switch on an enum to simulate a missing method 
		public static Operation inverse(Operation op) { 
			switch(op) { 
				case PLUS: return Operation.MINUS; 
				case MINUS: return Operation.PLUS; 
				case TIMES: return Operation.DIVIDE; 
				case DIVIDE: return Operation.TIMES; 
				default: throw new AssertionError(“Unknown op: “ + op); 
			} 
		}

如果可以修改枚举类型的源代码，但要调用的方法却不属于该枚举类型时，也可以使用`switch`代码块。

小结：在任何时候需要使用一组可以在编译期就确定值的常量时就可以使用枚举类型。在部分而不是全部枚举值共享常用的行为时，考虑使用策略枚举模式来实现。

### 35. 使用实例变量代替序数 ###

所有枚举类型都有一个`ordinal`方法，它返回每个枚举常量在类型中的位置。不要使用这个序数作为获取枚举值的索引。

		// Abuse of ordinal to derive an associated value - DON'T DO THIS
		public enum Ensemble {
			SOLO,   DUET,   TRIO, QUARTET, QUINTET,
			SEXTET, SEPTET, OCTET, NONET,  DECTET;

			public int numberOfMusicians() { return ordinal() + 1; }

		}

如果需要为枚举值添加一个序号，应将其保存在一个实例变量中：

		public enum Ensemble {

			SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
			SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
			NONET(9), DECTET(10), TRIPLE_QUARTET(12);

			private final int numberOfMusicians;
			Ensemble(int size) { this.numberOfMusicians = size; }
			public int numberOfMusicians() { return numberOfMusicians; }

		}

### 36. 使用`EnumSet`代替位变量 ###

一种表达一种类型的实例可能有多种组合状态的方法是为类提供几个位变量来表示不同的状态。下面将2的不同倍数赋值给每个常量：

		// Bit field enumeration constants - OBSOLETE!
		public class Text {
			public static final int STYLE_BOLD          = 1 << 0;  // 1
			public static final int STYLE_ITALIC        = 1 << 1;  // 2
			public static final int STYLE_UNDERLINE     = 1 << 2;  // 4
			public static final int STYLE_STRIKETHROUGH = 1 << 3;  // 8

			// Parameter is bitwise OR of zero or more STYLE_ constants
			public void applyStyles(int styles) { ... }
		}

这种表示方式允许使用按位或（or）运算将几个常量合并到一个称为位属性（bit field）的集合中：

		text.applyStyles(STYLE_BOLD | STYLE_ITALIC);

位属性表示还允许使用位运算符有效地执行集合运算，如并集和交集。

使用`java.util`包提供的`EnumSet`类来代替这种位属性，因为它能更有效地表示从单个枚举类型中提取的值集合。这个类实现了`Set`接口，提供了所有其他所有`Set`实现的丰富性，类型安全性和互操作性。在内部，每个`EnumSet`则都表示为一个位矢量（bit vector）。

如果底层的枚举类型有64个或更少的元素（大多数情况下都是如此），整个`EnumSet`用单个`long`表示，此时它的性能与位属性的性能相当。

		// EnumSet - a modern replacement for bit fields
		public class Text {
			public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

			// Any Set could be passed in, but EnumSet is clearly best
			public void applyStyles(Set<Style> styles) { ... }
		}

下面的客户端代码将`EnumSet`实例传递给`applyStyles`方法。

		text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));

`EnumSet`的缺点是它不能创建一个不可变的`EnumSet`，虽然可以用`Collections.unmodifiableSet`封装一个`EnumSet`，但是简洁性和性能会受到影响，希望在发布的版本中可能会得到加强。

### 37. 使用`EnumMap`代替序数索引 ###

有时可能会看到使用枚举类型的`ordinal`方法来索引到相关数组或列表元素的代码。最好用`EnumMap`替换这种做法。

例如，下面这个类代表一种植物：

		class Plant {

			enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

			final String name;
			final LifeCycle lifeCycle;
			
			Plant(String name, LifeCycle lifeCycle) {
				this.name = name;
				this.lifeCycle = lifeCycle;
			}
			
			@Override public String toString() {
				return name;
			}
		}

现在假设有一组植物代表一个花园，想要列出这些由生命周期组织的植物(一年生，多年生，或双年生)。为此，需要构建三个集合对应三个生命周期，然后遍历整个花园，将每个植物放置在适当的集合中。下面的代码使用枚举值的序数来索引相关的数组：

		// Using ordinal() to index into an array - DON'T DO THIS!
		Set<Plant>[] plantsByLifeCycle =
			(Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

		for (int i = 0; i < plantsByLifeCycle.length; i++)
			plantsByLifeCycle[i] = new HashSet<>();

		for (Plant p : garden)
			plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

		// Print the results
		for (int i = 0; i < plantsByLifeCycle.length; i++) {
			System.out.printf("%s: %s%n",
				Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
		}

这种方法虽然有效，但问题多多。首先，因为数组不兼容泛型，所以需要一个未经检查的转换。而且数组不知道索引代表什么，因此必须手动标记输出的结果是什么。最后，当访问一个使用枚举序数作为索引的数组时，必须使用正确的int值；int也不像枚举类型一样保障类型安全性。

在这个应用中，数组是作为枚举值到另一类值的映射使用的，所以可以使用`Map`。`java.util.EnumMap`专门用于键值是枚举类型的情况。

		// Using an EnumMap to associate data with an enum
		Map<Plant.LifeCycle, Set<Plant>>  plantsByLifeCycle =
			new EnumMap<>(Plant.LifeCycle.class);

		for (Plant.LifeCycle lc : Plant.LifeCycle.values())
			plantsByLifeCycle.put(lc, new HashSet<>());

		for (Plant p : garden)
			plantsByLifeCycle.get(p.lifeCycle).add(p);

		System.out.println(plantsByLifeCycle);

`EnumMap`构造方法的参数是其键值类型的`Class`对象：这是一个有限定的类型令牌（bounded type token），它提供运行时的泛型类型信息。

还可以使用stream来进一步优化：

		// Using a stream and an EnumMap to associate data with an enum
		System.out.println(Arrays.stream(garden)
				.collect(groupingBy(p -> p.lifeCycle,
			() -> new EnumMap<>(LifeCycle.class), toSet()))); 

基于stream版本的行为与`EmumMap`版本的行为略有不同。`EnumMap`版本总是会为每个生命周期生成一个嵌套map类，而基于流的版本只有在生命周期在花园中有一个或多个对应的植物时，才会生成嵌套map类。 例如，如果花园包含一年生和多年生植物但没有两年生的植物，`plantByLifeCycle`的大小在`EnumMap`版本中为三个，在基于流的版本中为两个。

有时还会需要使用多个枚举类型做交叉组合表示阶段，例如，下面这个程序使用序数和数组来把两个阶段映射到一个阶段转换（phase transition）（液体到固体表示凝固，液体到气体表示沸腾等等）：

		// Using ordinal() to index array of arrays - DON'T DO THIS!
		public enum Phase {
			SOLID, LIQUID, GAS;

			public enum Transition {
				MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
				
				// Rows indexed by from-ordinal, cols by to-ordinal
				private static final Transition[][] TRANSITIONS = {
					{ null,    MELT,     SUBLIME },
					{ FREEZE,  null,     BOIL    },
					{ DEPOSIT, CONDENSE, null    }
				};

				// Returns the phase transition from one phase to another
				public static Transition from(Phase from, Phase to) {
					return TRANSITIONS[from.ordinal()][to.ordinal()];
				}
			}
		}

这段代码的问题是，编译器无法知道序数和数组索引之间的关系。如果在转换表中出错或者在更改`Phase`或`Phase.Transition`枚举类型后忘记更新转换表，则程序将会运行失败。

使用`EnumMap`更为安全。因为每个阶段转换都是通过一对阶段枚举来索引，所以最好将关系表示为从一个枚举（from 阶段）映射到一个map的map，其中第二个map是从一个枚举（to阶段）映射到结果（阶段转换）。 与阶段转换相关的两个阶段（from 和 to）通过阶段转换枚举的构造方法将它们三个相关联起来，然后用它来初始化嵌套的`EnumMap`：

		// Using a nested EnumMap to associate data with enum pairs
		public enum Phase {
		   SOLID, LIQUID, GAS;

		   public enum Transition {
			  MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
			  BOIL(LIQUID, GAS),   CONDENSE(GAS, LIQUID),
			  SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

			  private final Phase from;
			  private final Phase to;

			  Transition(Phase from, Phase to) {
				 this.from = from;
				 this.to = to;
			  }

			  // Initialize the phase transition map
			  private static final Map<Phase, Map<Phase, Transition>>
				  m = Stream.of(values()).collect(groupingBy(t -> t.from,
					  () -> new EnumMap<>(Phase.class),
					  toMap(t -> t.to, t -> t,
						(x, y) -> y, () -> new EnumMap<>(Phase.class))));

			  public static Transition from(Phase from, Phase to) {
				 return m.get(from).get(to);
			  }
		   }
		}

map的类型是Map`<Phase, Map<Phase, Transition>>`，意思是“从（源）阶段映射到从（目标）阶段到阶段转换的映射。”这个map的map使用两个collector的级联序列进行初始化。 第一个collector按源阶段对转换进行分组，第二个collector使用从目标阶段到阶段转换的映射创建一个`EnumMap`。 第二个collector的参数中的lambda表达式`((x, y) -> y))`并未使用，仅仅是因为我们需要指定一个map工厂才能获得一个`EnumMap`。

现在假设想为系统添加一个新阶段：等离子体或电离气体。 这个阶段有两个转变：电离，将气体转化为等离子体; 和去离子，将等离子体转化为气体。更新基于`EnumMap`的版本，只需将PLASMA添加到阶段枚举中，并将`IONIZE(GAS, PLASMA)`和`DEIONIZE(PLASMA, GAS)`添加到阶段转换枚举中：

		// Adding a new phase using the nested EnumMap implementation
		public enum Phase {

			SOLID, LIQUID, GAS, PLASMA;

			public enum Transition {
				MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
				BOIL(LIQUID, GAS),   CONDENSE(GAS, LIQUID),
				SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
				IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);

				... // Remainder unchanged
			}
		}

小结：使用枚举值在枚举类型中的序数来索引数组，从而为两者建立关联是非常不合适的，应该使用`EnumMap`。如果需要表达的关系是多维的，使用`EnumMap <…，EnumMap <… >..>`。

### 38. 使用接口模拟可扩展的枚举 ###


通常，没有必要考虑枚举类型的可扩展性。但有时候也会有这种需求，例如操作码（operation codes），也称为opcodes。操作码是枚举类型，用来表示某些机器上的操作，例如Item 34中的`Operation`类型表示简单计算器上的功能。有时需要让API的用户提供他们自己的操作，从而有效地扩展API提供的操作集。

解决方法是利用枚举类型可以实现任意接口的原理，为opcode类型定义一个接口，并创建一个枚举类型作为该接口的标准实现。

		// Emulated extensible enum using an interface 
		public interface Operation { 
			double apply(double x, double y); 
		} 

		public enum BasicOperation implements Operation { 
			PLUS("+") { 
				public double apply(double x, double y) { return x + y; } 
			}, 
			MINUS("-") { 
				public double apply(double x, double y) { return x - y; } 
			}, 
			TIMES("*") { 
				public double apply(double x, double y) { return x * y; } 
			}, 
			DIVIDE("/") { 
				public double apply(double x, double y) { return x / y; } 
			}; 
			
			private final String symbol; 
			
			BasicOperation(String symbol) { 
				this.symbol = symbol; 
			} 
			
			@Override public String toString() { 
				return symbol; 
			} 
		} 

虽然枚举类型（`BasicOperation`）不可扩展，但接口（`Operation`）是可以扩展的，并且它是用于描述API中的操作。所以可以在需要的时候定义另一个实现此接口的枚举类型，并使用这个新类型的实例来代替基本类型。

例如，假设想要定义前面所示的操作类型的扩展，包括指数运算和余数运算。所要做的就是编写一个实现`Operation`接口的枚举类型：

		// emulated extension enum
		public enum ExtendedOperation implements Operation {
			EXP("^") {
				public double apply(double x, double y) {
				return Math.pow(x, y);
				}
			},
			REMAINDER("%") {
				public double apply(double x, double y) {
				return x % y;
				}
			};
			
			private final String symbol;
			ExtendedOperation(String symbol) {
				this.symbol = symbol;
			}
			@Override public String toString() {
				return symbol;
			}
		}

不仅可以在任何需要“基本枚举”的地方使用“扩展枚举”的单个实例，还可以传入整个扩展的枚举类型，并使用其元素。

		public static void main(string[] args) {
			double x = Double.parseDouble(args[0]);
			double y = Double.parseDouble(args[1]);
			test(ExtendedOperation.class, x, y);
		}

		private static <T extends Enum<T> & Operation> void test(
			Class<T> opeNumType, double x, double y) {
			for (Operation op : opeNumType.getEnumConstants())
				System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
		}

`opeNumType`参数中复杂的声明（`<T extends Enum<T> & Operation>`）确保了`Class<T>`类型既是枚举又是`Operation`接口的实现类。

第二种方式是传递一个`Collection<? extends Operation>`，而不是传递一个`Class`对象：

		public static void main(String[] args) {
			double x = Double.parseDouble(args[0]);
			double y = Double.parseDouble(args[1]);
			test(Arrays.asList(ExtendedOperation.values()), x, y);
		}
		private static void test(Collection<? extends Operation> opset,
			double x, double y) {
			for (Operation op : opset)
				System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
		}

这种方法允许调用者将来自多个接口实现类的操作组合在一起。但另一方面，也放弃了在指定操作上使用`EnumSet`和`EnumMap`的能力。

使用接口来模拟可扩展枚举的一个小缺点是不能让一个枚举类型继承另一个枚举类型。 如果实现代码不依赖于任何状态，则可以将其放在接口的默认方法中。

### 39. 注解优先于命名模式 ###

命名模式（naming pattern）：为程序里需要特殊处理的元素的名字制定相应的命名规则。

注解能提供更好的处理这种需要特殊对待的元素。以下都是使用注解的示范，不感兴趣可以跳过。

假设想定义一个注解类型来指定自动运行的简单测试，并且如果它们抛出一个异常就判断执行失败。

		// Marker annotation type declaration
		import java.lang.annotation.*;
		/**
		* Indicates that the annotated method is a test method.
		* Use only on parameterless static methods.
		*/
		@Retention(RetentionPolicy.RUNTIME)
		@Target(ElementType.METHOD)
		public @interface Test {
		}

`Test`注解类型的声明本身使用了`Retention`和`Target`注解进行标记。用在注解类型声明上的这种注解称为元注解（meta-annotations）。`@Retention（RetentionPolicy.RUNTIME）`元注解指示`Test`注解应该在运行时保留。没有它，测试工具就会看不到`Test`注解。`@Target.（ElementType.METHOD）`元注解表明`Test`注解只能用于方法的声明，不能应用于类声明，属性声明或其他程序元素。

在`Test`注解声明之前的注释说：“仅在无参数静态方法中使用”。必须编写自己的注解处理器来实现次限定。否则，该限定无效。

以下是`Test`注解在实践中的应用。它是一个标记注解（marker annotations），即没有参数，只能用于标记被注解元素。

		// Program containing marker annotations
		public class Sample {
			@Test 
			public static void m1() { }  // Test should pass
			public static void m2() { }
			@Test 
			public static void m3() {     // Test should fail
				throw new RuntimeException("Boom");
			}
			public static void m4() { }
			@Test 
			public void m5() { } // INVALID USE: nonstatic method
			public static void m6() { }
			@Test 
			public static void m7() {    // Test should fail
				throw new RuntimeException("Crash");
			}
			public static void m8() { }
		}

`Sample`类有七个静态方法，其中四个被标注为`Test`。其中`m3`和`m7`引发异常；`m1`和`m5`不引发异常，但是`m5`是实例方法，因此注解无效。所以，`Sample`包含四个测试：一个会通过，两个会失败，一个是无效的。未使用`Test`注解标注的四种方法将被测试工具忽略。

`Test`注解对`Sample`类的语义没有直接影响。注解不会改变注解代码的语义，但可以通过诸如简单的测试运行器等工具对其进行特殊处理：

		// Program to process marker annotations
		import java.lang.reflect.*;

		public class RunTests {
			public static void main(String[] args) throws Exception {
				int tests = 0;
				int passed = 0;
				Class<?> testClass = Class.forName(args[0]);
				for (Method m : testClass.getDeclaredMethods()) {
					if (m.isAnnotationPresent(Test.class)) {
						tests++;
						try {
							m.invoke(null);
							passed++;
						} catch (InvocationTargetException wrappedExc) {
							Throwable exc = wrappedExc.getCause();
							System.out.println(m + " failed: " + exc);
						} catch (Exception exc) {
							System.out.println("Invalid @Test: " + m);
						}
					}
				}
				System.out.printf("Passed: %d, Failed: %d%n",
				passed, tests - passed);
			}
		}

测试运行器工具在命令行上接受类的全名，并通过调用`Method.invoke`来反射地运行所有类标记有`Test`注解的方法。`isAnnotationPresent`方法提供需要运行哪些方法的判断条件。如果测试方法引发异常，则反射机制将其封装在`InvocationTargetException`中。该工具捕获此异常并打印故障报告，该报告包含了故障的源头，即由`test`方法抛出的起始异常。该异常是使用`getCause`方法从`InvocationTargetException`中提取的。

如果抛出了除`InvocationTargetException`之外的其他异常，则表示发现了在编译期没有捕获到的`Test`注解的无效使用：包括注解了实例方法，或者具有一个或多个参数的方法或不可访问的方法。

这是在`RunTests`在`Sample`上运行时打印的输出：

		public static void Sample.m3() failed: RuntimeException: Boom
		Invalid @Test: public void Sample.m5()
		public static void Sample.m7() failed: RuntimeException: Crash
		Passed: 1, Failed: 3

现在，添加对仅在抛出特定异常时才成功的测试的支持。为此需要添加一个新的注解类型：

		// Annotation type with a parameter
		import java.lang.annotation.*;
		/**
		* Indicates that the annotated method is a test method that
		* must throw the designated exception to succeed.
		*/
		@Retention(RetentionPolicy.RUNTIME)
		@Target(ElementType.METHOD)
		public @interface ExceptionTest {
			Class<? extends Throwable> value();
		}

此注解的参数类型是`Class<? extends Throwable>`。

		// Program containing annotations with a parameter
		public class Sample2 {
			@ExceptionTest(ArithmeticException.class)
			public static void m1() { // Test should pass
				int i = 0;
				i = i / i;
			}
			@ExceptionTest(ArithmeticException.class)
			public static void m2() { // Should fail (wrong exception)
				int[] a = new int[0];
				int i = a[1];
			}
			@ExceptionTest(ArithmeticException.class)
				public static void m3() { } // Should fail (no exception)
			}
		}

现在修改测试运行器工具来处理新的注解。将以下代码添加到`main`方法中：

		if (m.isAnnotationPresent(ExceptionTest.class)) {
			tests++;
			try {
				m.invoke(null);
				System.out.printf("Test %s failed: no exception%n", m);
			} catch (InvocationTargetException wrappedEx) {
				Throwable exc = wrappedEx.getCause();
				Class<? extends Throwable> excType =
					m.getAnnotation(ExceptionTest.class).value();
				if (excType.isInstance(exc)) {
					passed++;
				} else {
					System.out.printf(
						"Test %s failed: expected %s, got %s%n",
						m, excType.getName(), exc);
				}
			} catch (Exception exc) {
				System.out.println("Invalid @Test: " + m);
			}
		}

如果注解的参数在编译时有效，但代表指定异常类型的类文件在运行时不再存在，则将抛出`TypeNotPresentException`异常。

现在进一步细化测试工作，只要抛出几个指定的异常中的任何一个，就会通过测试。将`ExceptionTest`注解的参数类型更改为`Class`对象数组：

		// Annotation type with an array parameter
		@Retention(RetentionPolicy.RUNTIME)
		@Target(ElementType.METHOD)
		public @interface ExceptionTest {
			Class<? extends Exception>[] value();
		}

注解中数组类型作为参数使用很灵活。它针对单元素数组进行了优化。所有以前的`ExceptionTest`注解仍然适用于新的使用数组作为参数的`ExceptionTest`，并且会自动生成单元素数组。要指定一个多元素数组时使用花括号将这些元素包括起来，并用逗号分隔它们：

		// Code containing an annotation with an array parameter
		@ExceptionTest({ IndexOutOfBoundsException.class,
						NullPointerException.class })
		public static void doublyBad() {
			List<String> list = new ArrayList<>();
			// The spec permits this method to throw either
			// IndexOutOfBoundsException or NullPointerException
			list.addAll(5, null);
		}
		修改测试运行器工具以处理新版本的ExceptionTest是相当简单的。 此代码替换原始版本：

		if (m.isAnnotationPresent(ExceptionTest.class)) {
			tests++;
			try {
				m.invoke(null);
				System.out.printf("Test %s failed: no exception%n", m);
			} catch (Throwable wrappedExc) {
				Throwable exc = wrappedExc.getCause();
				int oldPassed = passed;
				Class<? extends Exception>[] excTypes =
					m.getAnnotation(ExceptionTest.class).value();
				for (Class<? extends Exception> excType : excTypes) {
					if (excType.isInstance(exc)) {
						passed++;
						break;
					}
				}
				if (passed == oldPassed)
					System.out.printf("Test %s failed: %s %n", m, exc);
			}
		}

从Java 8开始，还有另一种方法来实现重复注解。使用`@Repeatable`元注解来标示可重复注解的声明，就不必再声明注解类型使用数组参数。 该元注解只有一个参数，该参数是可重复注解的容器类的类对象，这个容器类的唯一参数是一个可重复注解类型的数组。注意必须为该容器类指定合理的retention policy和target，否则声明将无法编译：

		// Repeatable annotation type
		@Retention(RetentionPolicy.RUNTIME)
		@Target(ElementType.METHOD)
		@Repeatable(ExceptionTestContainer.class)
		public @interface ExceptionTest {
			Class<? extends Exception> value();
		}
		@Retention(RetentionPolicy.RUNTIME)
		@Target(ElementType.METHOD)
		public @interface ExceptionTestContainer {
			ExceptionTest[] value();
		}

下面是使用重复注解代替基于数组值注解的例子:

		// Code containing a repeated annotation
		@ExceptionTest(IndexOutOfBoundsException.class)
		@ExceptionTest(NullPointerException.class)
		public static void doublyBad() { ... }

处理可重复的注解需要注意： 重复注解会生成该注解类型的容器类注解。`getAnnotationsByType`方可用于访问可重复注解类型的重复了的和未重复的注解。但isAnnotationPresent明确指出重复注解不是注解类型，而是包含注解类型。 如果某个元素具有某种类型的重复注解，并且使用isAnnotationPresent方法检查元素是否具有该类型的注释，则会发现它没有。 使用此方法检查注解类型的存在会因此导致程序默默忽略重复的注解。 同样，使用此方法检查包含的注解类型将导致程序默默忽略不重复的注释。 要使用isAnnotationPresent检测重复和非重复的注解，需要检查注解类型及其包含的注解类型。 以下是RunTests程序的相关部分在修改为使用ExceptionTest注解的可重复版本时的例子：

		// Processing repeatable annotations
		if (m.isAnnotationPresent(ExceptionTest.class)
			|| m.isAnnotationPresent(ExceptionTestContainer.class)) {
			tests++;
			try {
				m.invoke(null);
				System.out.printf("Test %s failed: no exception%n", m);
			} catch (Throwable wrappedExc) {
				Throwable exc = wrappedExc.getCause();
				int oldPassed = passed;
				ExceptionTest[] excTests =
					m.getAnnotationsByType(ExceptionTest.class);
				for (ExceptionTest excTest : excTests) {
					if (excTest.value().isInstance(exc)) {
						passed++;
						break;
					}
				}
				if (passed == oldPassed)
					System.out.printf("Test %s failed: %s %n", m, exc);
			}
		}

鼓励使用Java提供的预定义注解类型。

### 40. 坚持使用`Override`注解 ###

此注解只能在方法声明上使用。应该在重写了父类方法的子类方法上使用`Override`注解。

`Override`注解也可用来标记重写了接口的默认方法。如果某个接口没有默认方法，也可以不必使用`Override`注解。

然而，在一个抽象类或接口中，所有被认为重写了父类或父接口的方法都应该使用`Override`注解标记，不论方法是具体的还是抽象的。

### 41. 使用标记接口定义类型 ###

标记接口（marker interface）：不包含任何方法声明，只是用来标记它的实现类具有一些共性，可以视为同一类型。例如，通过实现`Serializable`接口，一个类表明它的实例可以被序列化。

标记注解并不能取代标记接口。标记接口与标记注解相比具有两个优点。首先，标记接口定义了一个由被标记类实例实现的类型；标记注解则不会。标记接口允许在编译时捕获错误，如果使用标记注解，则直到运行时才能捕获这些错误。

Java的序列化机制使用`Serializable`标记接口来指示某个类型是可序列化的。负责序列化的`ObjectOutputStream.writeObject`方法就要求其参数实现`Serializable`接口类型，并在编译时会通过类型检查检测到序列化不适当对象的尝试。这就符合标记接口在编译时就检测错误意图，但`ObjectOutputStream.write`API没有利用`Serializable`接口：它的参数被声明为`Object`类型，所以尝试序列化一个不可序列化的对象会直到运行时才失败。

标记接口对于标记注解的另一个优点是它们更精确。如果使用标记注解`ElementType.TYPE`，则只能说明可应用于任何类或接口。而使用标记接口就可以将范围缩小到具体的接口类型。

标记注解优于标记接口的主要优点是它们是整个注解机制的一部分。因此，标记注解允许在基于注解的框架中保持一致性。

所以什么时候应该使用标记注解，什么时候应该使用标记接口？ 

- 如果标记适用于除类或接口以外的任何程序元素，则必须使用注解，因为只能类和接口才能实现或扩展接口。
- 如果标记仅适用于类和接口，那么自问一个问题：“是否会编写一个或多个只接受受此标记的对象的方法呢？”
	- 如果是，则应该优先使用标记接口而不是注解。从而可以使用接口作为方法的参数类型，并受益于编译时的类型检查。
	- 如果否，永远不会写一个只接受带有标记的对象的方法，那么就使用标记注解。尤其当该标记是大量使用注解的框架的一部分的时候，标记注解更是明确的选择。

如果想定义一个没有任何新方法与之关联的的类型，应该考虑使用标记接口。 如果要标记除类和接口以外的程序元素，或者要将标记整合到一个已大量使用注解的框架中，那么标记注解是正确的选择。

### 42. lambda表达式优先于匿名类 ###

只有单个抽象方法的接口（或很少见的抽象类）被用作函数类型（function types）。 它们的实例称为函数对象（function objects)，代表函数或动作。下面这个例子使用匿名类创建排序的比较函数，用于按长度顺序对字符串列表进行排序：

		// Anonymous class instance as a function object - obsolete!
		Collections.sort(words, new Comparator<String>() {
			public int compare(String s1, String s2) {
				return Integer.compare(s1.length(), s2.length());
			}
		});

匿名类适用于经典的需要功能对象的面向对象的设计模式，特别是策略模式（strategy pattern）。`Comparator`接口表示用于排序的抽象策略；使用的匿名类是排序字符串的策略的具体实现。

从Java 8开始，具有单一抽象方法的接口被称为函数式接口（functional interfaces），允许使用lambda表达式创建这些接口的实例。把上面的代码用lambda改写，用lambda替换其中的匿名类。

		// Lambda expression as function object (replaces anonymous class)
		Collections.sort(words,
				(s1, s2) -> Integer.compare(s1.length(), s2.length()));

应该省略所有lambda参数的类型，除非它们能使程序更清晰。如果编译器出错，无法推断lambda参数的类型，就需要人工指定它。

尽量使用泛型类型或泛型方法，这将有助于lambda表达式做出正确的类型推断。

使用方法引用来代替lambda可以让程序更简洁：

		Collections.sort(words, comparingInt(String::length));

实际上，通过使用Java 8中添加到`List`接口的`sort`方法，可以使代码段更简短：

		words.sort(comparingInt(String::length));

lambda使函数对象的应用更广泛。例如，Item 34中的`Operation`枚举类型。每个枚举值对其`apply`方法需要不同的行为，所以使用常量特定的类体来实现每个枚举常量中的`apply`方法：

		// Enum type with constant-specific class bodies & data (Item 34)
		public enum Operation {
			PLUS("+") {
				public double apply(double x, double y) { return x + y; }
			},
			MINUS("-") {
				public double apply(double x, double y) { return x - y; }
			},
			TIMES("*") {
				public double apply(double x, double y) { return x * y; }
			},
			DIVIDE("/") {
				public double apply(double x, double y) { return x / y; }
			};
			private final String symbol;
			Operation(String symbol) { this.symbol = symbol; }
			@Override public String toString() { return symbol; }
			
			public abstract double apply(double x, double y);
		}

第34项曾提到应该优先使用enum实例字段，而非常量特定的类体。Lambdas可以轻松完成此项工作。为枚举类型声明一个函数式接口类型的变量代表需要指定的操作，然后在每个枚举值的构造方法中传入实现这个操作的lambda。构造方法将lambda存储在该实例变量中，`apply`方法将调用转发给lambda：

		// Enum with function object fields & constant-specific behavior
		public enum Operation {
			PLUS ("+", (x, y) -> x + y),
			MINUS ("-", (x, y) -> x - y),
			TIMES ("*", (x, y) -> x * y),
			DIVIDE("/", (x, y) -> x / y);
			
			private final String symbol;
			private final DoubleBinaryOperator op;
			Operation(String symbol, DoubleBinaryOperator op) {
				this.symbol = symbol;
				this.op = op;
			}
			@Override public String toString() { return symbol; }
			
			public double apply(double x, double y) {
				return op.applyAsDouble(x, y);
			}
		}

与方法和类不同，lambdas缺少名称和文档；如果代码不能自我解释，或超过几行，则不要将它放在lambda中。单行代码最适用于lambda，三行代码是合理的最大长度。此外，传递给枚举构造函数的参数是在静态上下文中进行估值，所以枚举构造函数中的lambdas无法访问枚举的实例成员。如果枚举类型具有复杂的常量特定行为，无法在几行中实现，或者需要访问实例变量或方法，则仍然可以采用特定于常量的类体。

和匿名类相比：

- lambdas仅限于函数式接口。如果要创建抽象类的实例，可以使用匿名类，但不能使用lambda。
- 也可以使用匿名类来创建具有多个抽象方法的接口实例。
- 最后，lambda无法获得对自身的引用。在lambda中，`this`关键字引用的是外包实例。在匿名类中，`this`关键字引用的是匿名类实例。 如果需要从其体内访问函数对象，则必须使用匿名类。

Lambdas与匿名类都无法在实现中可靠地序列化和反序列化。如果一个可序列化的函数对象，例如`Comparator`，应该使用私有静态嵌套类的实例（参阅24）。

### 43. 方法引用优先于lambda ###

lambda和方法引用示例：
	
		map.merge(key, 1, (count, incr) -> count + incr);
		map.merge(key, 1, Integer::sum);

如果lambda很长或很复杂，可以将lambda中的代码提取到一个新的方法中，然后用对该方法的引用代替lambda。这样就可以给这个功能起一个好名字，并能为其添加文档。

IDE往往提供了将lambda替换为方法引用的功能，推荐使用。

偶尔，lambda会比方法引用更简洁。经常发生在方法与lambda处于相同的类中。

例如，考虑这段代码，它被假定出现在一个名为`GoshThisClassNameIsHumongous`的类中：

		service.execute(GoshThisClassNameIsHumongous::action);
		service.execute(() -> action());

类似地，`Function`接口提供了一个通用的静态工厂方法来返回标识函数`Function.identity()`。但通常使用更短，更简洁的等效lambda：`x - > x`。

许多方法引用是指引用静态方法，但有四种方法引用不是。其中两个是绑定（bound）和非绑定（unbound）对象的实例方法引用。在特定对象引用中，接收对象在方法引用中指定。特定对象引用在本质上与静态引用类似：函数对象与引用的方法具有相同的参数。在非绑定对象方法引用中，接收对象在应用函数对象时通过在方法声明的参数之前的一个附加参数指定。非绑定对象的方法引用通常用作流管道（pipelines）中的映射和过滤方法（参阅45）。最后，对于类和数组，有两种构造方法引用。构造方法引用用作工厂对象。

Method Ref Type   | Example                | Lambda Equivalent
---               | ---                    | ---
static            | Integer::parseInt      | str -> Integer.parseInt(str)
bound             | Instant.now()::isAfter | Instant then = Instant.now(); t -> then.isAfter(t)
unbound           | String::toLowerCase    | str -> str.toLowerCase()
class constructor | TreeMap<K, V>::new     | () -> new TreeMap<K, V>
array constructor | int[]::new             | len -> new int[len]

### 44. 多使用标准的函数式接口 ###

了解并使用`java.util.function`包提供的大量标准函数式接口。

API的编写随着lambda的引入发生了很大的变化，一些传统的模式已经被取代。例如，传统的模板方法模式（template method pattern）中，父类负责提供基本的实现模板，子类可以重写这些方法以定制自己的行为。现代的替代方法是提供一个静态工厂或构造方法，它接受一个函数对象来实现相同的效果。广而泛之，现在会编写更多以函数对象作为参数的构造方法和普通方法。选择正确的函数参数类型需要谨慎。

以`LinkedHashMap`为例，它包含一个受保护的`removeEldestEntry`方法，该方法在每次通过`put`方法添加新键值时会被调用。可以通过重写此方法将此类用作缓存。当此方法返回`true`时，map将删除作为参数传递给此方法的最早的entry。以下代码允许保留最近的100个entry：

		protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
			//调用map对象的size方法返回map的大小
			return size() > 100;
		}

可以用lambda改善这段代码。首先`LinkedHashMap`将有一个带有函数对象的静态工厂或构造方法。如果参考`removeEldestEntry`方法的声明，可能认为函数对象应该使用`Map.Entry <K，V>`作为参数并返回一个布尔值，但这却行不通。因为`removeEldestEntry`方法调用`size()`来获取map中的entry的数目，可以这样做是因为`removeEldestEntry`是map上的实例方法。而传递给构造函数的函数对象不是 map 上的实例方法，也无法获取该map对象，因为在调用其工厂或构造函数时该map尚不存在。因此必须把这个map对象和要删除的最早的entry一起传递给函数对象。

		// Unnecessary functional interface; use a standard one instead.  
		@FunctionalInterface interface EldestEntryRemovalFunction<K,V> {
			boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
		}

此接口虽然可以工作，但却与标准函数式接口产生了重叠。而如果有标准函数式接口能胜任所需工作，就应该优先使用它们，而不是专门构建一个新的函数式接口。例如，`Predicate`接口提供了可以组合判断条件的方法。对于`LinkedHashMap`示例，应优先使用标准的`BiPredicate<Map<K,V>,Map.Entry<K,V>>`接口，而不是自定义`EldestEntryRemovalFunction`接口。

`java.util.Function`中有四十三个接口。其中有六个基本接口，基于引用类型进行操作。

- `Operator`接口表示结果和参数类型相同的函数。
- `Predicate`接口表示一个接受参数并返回布尔值的函数。
- `Function`接口表示其参数和返回类型不同的函数。
- `Supplier`接口表示不带参数却有返回（或“提供”）值的函数。
- `Consumer`表示一个函数，它接受一个参数但什么都不返回。

六个基本函数式接口总结如下：

interface           | function signature     | example
---                 | ---                    | ---
`UnaryOperator<T>`  | `T apply(T t)`         | `String::toLowerCase`
`BinaryOperator<T>` | `T applay(T t1, T t2)` | `BigInteger::add`
`Predicate<T>`      | `boolean test(T t)`    | `Collection::isEmpty`
`Function<T, R>`    | `R apply(T t)`         | `Arrays::asList`
`Supplier<T>`       | `T get()`              | `Instant::now`
`Consumer<T>`       | `void accept(T t)`     | `System.out::println`

六种基本接口中的每一种都有三种变体分别针对基本类型`int`，`long`和`double`进行操作。它们的名称以基本接口名称为基础，加上基本类型的名称作为前缀，例如`IntPredicate`，带有两个long值并返回long的二元运算符是`LongBinaryOperator`。除`Function`的变体外，这些针对基本数据的函数式接口量类型都没有参数化，`Function`变体接口则会参数化返回类型。例如，`LongFunction<int[]>` 取一个`long`并返回一个`int[]`。

`Function`接口还有九个额外的变体，对应返回结果类型是基本数据类型的情况。注意，`Function`接口的输入和输出类型总是不同的，如果相同则应该使用`UnaryOperator`接口。

如果输入和输出都是基本数据类型，则使用`SrcToResult`作为`Function`的前缀，例如`LongToIntFunction`（六个变体）。如果输入是基本数据类型而输出是引用类型，则使用`SrcToObj`作为`Function`的前缀，例如`DoubleToObjFunction`（三个变体）。三个基本函数式接口有各自的双参数版本：`BiPredicate<T，U>`，`BiFunction<T，U，R>`和`BiConsumer<T，U>`。还有`BiFunction`变体用于返回三个基本类型：`ToIntBiFunction<T，U>`，`ToLongBiFunction<T，U>`和`ToDoubleBiFunction<T，U>`。`Consumer`的双参数变体使用一个引用类型和一个基本数据类型：`ObjDoubleConsumer`，`ObjIntConsumer`和`ObjLongConsumer`。总共有九个基本接口的双参数版本。

最后，还有`BooleanSupplier`接口，这是`Supplier`的一个返回布尔值的变体。这是所有标准函数式接口名称中唯一明确提到布尔类型的接口，但是可以通过`Predicate`及其四种变体来支持返回布尔值。`BooleanSupplier`接口和前面所描述的四十二个接口一起组成四十三个标准函数式接口。

大多数标准函数式接口仅用于提供对原始类型的支持。 不要试图使用基本数据类型的封装类作为参数的基本函数式接口来替代直接使用基本数据类型作为参数的函数式接口。

需要自己编写函数式接口的原因：

- 它将被普遍使用，并需要有意义的描述性名称。
- 它与之相关的合同很强。
- 它将受益于自定义默认方法。

如果您选择编写自己的函数式接口，请记住它是一个接口，因此应该非常谨慎地设计（参阅21）。

始终使用`@FunctionalInterface`注解标记自定义的函数式接口。

不要为了使用不同的函数式接口而通过在相同参数位置使用不同类型参数的方法来重载方法。

### 45. 谨慎使用流 ###

在Java 8中添加了Stream API，以简化顺序或并行执行批量操作的任务。该API提供了两个关键的抽象：流(Stream)，表示有限或无限的数据元素序列，以及流管道(stream pipeline)，表示对这些元素的多级计算。

Stream中的元素可以来自任何地方。常见的源包括集合，数组，文件，正则表达式的模式匹配器，伪随机数生成器和其他流。流中的数据元素可以是对象引用或基本类型。只支持三种基本类型：`int`，`long`和`double`。

流管道由源流（source stream）的零或多个中间操作和一个终结操作组成。每个中间操作都以某种方式转换流，例如将每个元素映射到该元素的一个功能或过滤掉所有不满足某些条件的元素。中间操作都将一个流转换为另一个流，其元素类型可能与输入流相同或不同。终结操作对最后一次中间操作产生的流做最终处理，如将其元素存储到集合中、返回某个元素或打印其所有元素。

流管道采用延迟（lazily）求值：求值操作直到终结操作被调用才开始进行，而完成终结操作不需要的数据元素永远不会被计算出来。

Stream API是流式的（fluent）：组成一个管道的多个操作可以连接在一起组成一个操作链表达式，而多个管道也可以连接在一起形成一个表达式。

默认情况下，流管道按顺序(sequentially)运行。 在管道中的任何流上调用`parallel`方法可以使管道并行执行，但很少这样做（参阅48）。

Java缺乏对原始字符流的支持，应该避免使用流来处理char值。

在使用流时创建适当的帮助方法可以提高代码的可读性，因为流管道缺乏显式的类型信息并会创建临时变量。

只在必要的时候使用流改写已有代码。

流管道使用函数对象(通常为lambdas或方法引用)实现重复计算；而迭代代码使用代码块实现重复计算，在代码块中可以做使用函数对象不能做的事情:

- 在代码块中，可以读取或修改有效范围内的任何局部变量; lambda中，只能读取final或effective finald的变量，并且无法修改任何局部变量。
- 在代码块中，可以从外包方法中返回，中断或继续外包循环，或抛出此方法声明的任何已检查异常; lambda不能做这些事情。

流技术适合以下场景：

- 统一转换元素序列
- 过滤元素序列
- 使用单个操作组合一系列元素(例如添加、连接或计算最小值)
- 将元素序列累积到一个集合中，可以利用一些共有属性将它们分组
- 在元素序列中搜索满足指定条件的元素

对于流来说，很难做到的一件事是同时访问处于管道的不同阶段中的相应元素。因为一旦将流转换到其它流，原始流就会丢失。一种解决方法是：如果允许，可以将某个阶段的流回转为之前的流。

迭代代码和流的选择没有铁律，更多的是根据个人喜好，如果不确定，可以都尝试以下。以下是一个扑克牌的静态工厂方法，分别用迭代和流来实现。

		// Iterative Cartesian product computation
		private static List<Card> newDeck() {
			List<Card> result = new ArrayList<>();
			for (Suit suit : Suit.values())
				for (Rank rank : Rank.values())
					result.add(new Card(suit, rank));
			return result;
		}

		// Stream-based Cartesian product computation
		//使用了中间操作flatMap方法。将一个流中的所有元素映射到另一个流中的每一个元素，然后将所有这些新流连接到一个流(或展平它们)。
		//注意，这个实现包含一个嵌套的lambda表达式（rank -> new Card(suit, rank))）:
		private static List<Card> newDeck() {
			return Stream.of(Suit.values())
				.flatMap(suit ->
					Stream.of(Rank.values())
				.map(rank -> new Card(suit, rank)))
				.collect(toList());
		}

### 46. 无副作用函数优先于流 ###

流范式中最重要的概念是将计算组织为一系列转换，每个转换阶段的结果都尽可能是上一个转换阶段结果的纯函数（pure function）。纯函数的结果仅取决于其输入，它不依赖于任何可变状态，也不更新任何状态。因此，传递给流操作的任何函数对象（中间操作和终结操作）都应该没有副作用。

来看一段不良示范，这段代码构建了文本文件中单词的频率表:

		// Uses the streams API but not the paradigm--Don't do this!
		Map<String, Long> freq = new HashMap<>();
		try (Stream<String> words = new Scanner(file).tokens()) {
			words.forEach(word -> {
				freq.merge(word.toLowerCase(), 1L, Long::sum);
			});
		}

这段代码的问题是，简而言之，它根本不是流代码，而是伪装成流代码的迭代代码；它没有从流API中获益，并且比相应的迭代代码更长，更难读，并且更难于维护。具体来说，这段代码在一个终结操作`forEach`中完成了所有工作，没有任何中间阶段；使用了一个改变外部状态（频率表）的lambda。

`forEach`操作应该仅用于报告流计算的结果，而不是用于执行计算。但如果只用了流的`forEach`操作，并且仅用来表示流执行的计算结果，就和一个改变状态的lambda一样，都是流编程中的不良代码。下面是改良过的代码

		// Proper use of streams to initialize a frequency table
		Map<String, Long> freq;
		try (Stream<String> words = new Scanner(file).tokens()) {
			freq = words
				.collect(groupingBy(String::toLowerCase, counting()));
		}

改进后的代码使用了收集器（collector），这是使用流必须学习的新概念。`Collectors`有39个方法，其中一些方法有多达5个类型参数。使用`Collectors`时可以静态导入其所有成员，从而使流管道更易于阅读。对于初学者来说，可以忽略收集器接口，将收集器看作是封装缩减策略（ reduction strategy）的不透明对象。此时，reduction意味着将流的元素组合为单个对象。收集器生成的对象通常是一个集合（这也是收集器的名字的来源）。

用于将流的元素收集到真正的`Collection`中的收集器有三个:`toList()`、`toSet()`和`toCollection(collectionFactory)`。它们分别返回集合、列表和程序员指定的集合类型。

		// Pipeline to get a top-ten list of words from a frequency table
		List<String> topTen = freq.keySet().stream()
			.sorted(comparing(freq::get).reversed())
			.limit(10)
			.collect(toList());

`Collectors`里的大多数方法都是用于将流收集到map中的，每个流元素都与一个键和一个值相关联，多个流元素可以与同一个键相关联。

最简单的映射收集器是`toMap(keyMapper、valueMapper)`，它接受两个函数，一个将流元素映射到键，另一个映射到值。在条目34中的`fromString`实现中，我们使用这个收集器从enum的字符串形式映射到enum本身:

		// Using a toMap collector to make a map from string to enum
		private static final Map<String, Operation> stringToEnum =
			Stream.of(values()).collect(
				toMap(Object::toString, e -> e));

如果流中的每个元素都映射到唯一键，则这种简单的`toMap`形式是完美的。但如果多个流元素映射到同一个键，则管道将以`IllegalStateException`终止。

`toMap`以及`groupingBy`方法的更复杂的格式，，提供了处理此类冲突(collisions)的各种方法。一种方法是为`toMap`方法添加一个参数，这第三个参数是除键和值映射器(mappers)之外的`merge`函数功能参数。`merge`是一个`BinaryOperator<V>`，其中`V`是map的值的类型（不是键的类型）。与键值关联的任何额外值都使用`merge`方法与现有值相结合。

简单来说，三个参数的`toMap`方法创建了一个map，这个map使用指定的mapper参数建立键和值，当一个键对应多个值时，第三个参数提供的函数方法就用来处理这些值，其处理结果将作为该键对应的唯一值。最终所得map的键和值是一一对应的关系，不会出现一键对多值的情况。例如，假设有一系列不同艺术家（artists）的唱片集（albums），想要得到艺术家到其最畅销专辑的map。

		// Collector to generate a map from key to chosen element for key
		Map<Artist, Album> topHits = albums.collect(
		   toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));

`maxBy`是从`BinaryOperator`静态导入的。此方法使用比较器构造方法`comparing`返回的比较器来取得键所对应值的最大值。这个比较器比较的元素是通过`Album::sales`提取出来的。

另一个应用的例子是当发生多值冲突时强制执行last-write-wins策略。

		// Collector to impose last-write-wins policy
		toMap(keyMapper, valueMapper, (oldVal, newVal) ->newVal)

`toMap`方法还有一个四个参数的版本，第四个参数是一个map工厂，用于指定特定的map实现，例如`EnumMap`或`TreeMap`。

`toMap`的前三个版本各有名为`toConcurrentMap`的变体形式，能高效并行运行并生成`ConcurrentHashMap`实例。

除了`toMap`方法之外，`Collectors`API还提供了`groupingBy`方法，该方法返回收集器，该收集器用于生成使用指定分类器函数(classifier function)将元素分组到不同类别中的map。分类器函数接受一个元素并返回它所属的类别。此类别来用作map的键。`groupingBy`方法的最简单版本仅使用一个分类器并返回一个map，map中的值是每个类别（键值）对应的所有元素的列表。下面是在条目45中的`Anagram`程序中使用的收集器，用于生成从一个字符序列到该序列所能生成单词的列表的map：

		words.collect(groupingBy(word -> alphabetize(word)));

如果想让生成的map的值的类型不是列表，则除了分类函数外，还需要为groupingBy方法提供一个downstream collector作为参数。一个downstream收集器负责根据一个流生成值，该流包含了一个类别里的所有元素。这个参数最简单的实参就是`toSet()`方法，也可以传入`toCollection(collectionFactory)`方法来创建指定用来存放值的集合。这样就可以把值存放在不同类型的集合里了。另外一个简单的例子是传入`counting()`方法作为downstream收集器，该方法不会生成任何集合，但会统计该键值所对应值的数量，相当于使用前面的downstream收集器所生成的集合的`size()`方法，生成的map的值是int类型。

		Map<String, Long> freq = words
				.collect(groupingBy(String::toLowerCase, counting()));

`groupingBy`的第三个版本在指定一个downstream收集器之外还要指定一个map工厂。此方法违反了标准的可伸缩参数列表模式(standard telescoping argument list pattern)，即新添加参数应在原有参数之后：`mapFactory`参数位于`downStream`参数之前，而不是之后。此版本的`groupingBy`可以返回的收集器中使用的`Map`中值的类型，默认类型是`List`，例如，可以指定收集器返回一个`TreeMap`，其值是`TreeSet`。

`groupingByConcurrent`方法提供了`groupingBy`的三个重载的变体。这些变体能高效并行运行并生成`ConcurrentHashMap`实例。还有一个很少使用的`groupingBy`的亲戚称为`partitioningBy`。它使用`Predicate`代替了分类器方法，从而返回键为布尔值的map。此方法有两种重载，另一个除了`predicate`之外，还需要提供`downstream`收集器。

`Collectors`的`counting`方法仅是为了生成downstream收集器。如果需要计数功能，可以直接在`Stream`上调用`count`来实现，而不是使用`collect(counting())`。还有十五种收集器方法也属于这种情况。其中九个方法的名称以`summing`，`averaging`和`summarizing`开头，其逻辑功能都可以调用相应的原始流的方法实现）。此外，还有`reduce`方法的所有重载，以及`filter`，`mapping`，`flatMapping`和`collectingAndThen`方法。这些方法的存在是为了尝试在收集器中部分复制流的功能，以便可以把downstream收集器充当“迷你流(ministreams)”。

`Collectors`中的`join`仅对`CharSequence`实例（如字符串）的流进行操作。无参数版本返回一个简单地连接元素的收集器。单参数版本使用名为`delimiter`的单个`CharSequence`参数作为分隔符，返回的连接流元素的收集器在相邻元素之间插入指定分隔符。三参数版本除了分隔符之外还指定前缀和后缀。生成的收集器会生成类似于打印集合时获得的字符串，例如[came, saw, conquered]。

### 47. 作为返回类型，`Collection`优先于`Stream` ###

和`Stream`相比，返回类型最好是`Collection`。

许多方法返回元素序列（sequence）。在Java 8之前，这些方法的返回类型是集合接口`Collection`，`Set`和`List`这些接口；还包括`Iterable`和数组类型。通常返回的都是是集合接口。但如果该方法仅是为了使用for-each循环，或者返回的序列不能实现某些`Collection`方法(通常是`contains(Object)`)，则使用迭代（`Iterable`）接口。如果返回的元素是基本类型或有严格的性能要求，则使用数组。

虽然`Stream`接口包含了`Iterable`接口中定义的唯一的抽象方法，而且`Stream`中此方法的规范也与`Iterable`兼容。但`Stream`却没有继承`Iterable`，所以不能在流上使用for-each循环。

如果直接在for-each循环中使用`Stream`的`iterator`方法引用作为要循环的对象：

		// Won't compile, due to limitations on Java's type inference
		for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
			// Process the process
		}

编译时出错：

		Test.java:6: error: method reference not expected here
		for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
								^

为了使代码编译，必须将方法引用强制转换为相应的参数化的`Iterable`类型：

		// Hideous workaround to iterate over a stream
		for  (ProcessHandle ph : (Iterable<ProcessHandle>)
								 ProcessHandle.allProcesses()::iterator)

更好的解决方法是使用适配器方法。在适配器方法中不需要强制转换，因为Java的类型推断在这里能够正常工作：

		// Adapter from  Stream<E> to Iterable<E>
		public static <E> Iterable<E> iterableOf(Stream<E> stream) {
			return stream::iterator;
		}

使用此适配器，可以使用for-each语句迭代任何流：

		for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
			// Process the process
		}

Item 34中的`Anagrams`程序的流版本使用`Files.lines`方法读取字典，而迭代版本使用了`scanner`。`Files.lines`方法优于`scanner`，因为`scanner`在读取文件时会无声地吞噬所有异常。理想情况下，在迭代版本中也应该使用`Files.lines`。如果API只提供了对序列的流访问，而又希望使用for-each语句遍历序列，那么就要使用这个适配器模式折中。

相反，也可以提供一个适配器为只提供Iterable的API添加流管道处理支持：

		// Adapter from Iterable<E> to Stream<E>
		public static <E> Stream<E> streamOf(Iterable<E> iterable) {
			return StreamSupport.stream(iterable.spliterator(), false);
		}

在编写处理返回的对象序列的API时，应该根据需要选择使用流管道处理还是使用迭代处理，如果不确定，最好两者都提供。

`Collection`接口是`Iterable`的子接口，并且具有stream方法，同时提供了迭代和流访问，因此，`Collection`及其子类通常是返回序列的公共方法的最佳返回类型。数组也使用`Arrays.asList`和`Stream.of`方法提供了简单的迭代和流访问。如果返回的序列小到可以轻易放入内存中，最好返回一个标准集合实现，例如`ArrayList`或`HashSet`。但不要只是为了返回一个集合就在内存中存储大的序列。

如果需要返回的序列很大但有某种简洁的表示方法，就考虑实现一个专用集合。例如，假设返回给定集合的幂集（power set：就是原集合中所有的子集（包括全集和空集）构成的集族）。{a，b，c}的幂集为({}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c})。拥有n个元素的集合，其幂集拥有2的n次方个元素。不应考虑将幂集存储在标准集合实现中，应该在AbstractList的帮助下，实现自定义集合。

诀窍是把幂集中每个元素的索引作为位向量（bit vector），索引从`int`转换为2进制后，第n位可以用来指示在该元素（幂集）中是否包含源集合中的是第n个元素。这是因为从0到2的（n-1）次方的二进制数和n元素集合的幂集之间存在自然映射。

		// Returns the power set of an input set as custom collection
		public class PowerSet {
		   public static final <E> Collection<Set<E>> of(Set<E> s) {
			  List<E> src = new ArrayList<>(s);
			  if (src.size() > 30)
				 throw new IllegalArgumentException("Set too big " + s);
			  return new AbstractList<Set<E>>() {
				 @Override public int size() {
					return 1 << src.size(); // 2 to the power srcSize
				 }
				 @Override public boolean contains(Object o) {
					return o instanceof Set && src.containsAll((Set)o);
				 }
				 @Override public Set<E> get(int index) {
					Set<E> result = new HashSet<>();
					for (int i = 0; index != 0; i++, index >>= 1)
					   if ((index & 1) == 1)
						  result.add(src.get(i));
					return result;
				 }
			  };
		   }
		}

注意：如果输入集合超过30个元素，则`PowerSet.of`方法会引发异常。这也突出了使用`Collection`作为返回类型而不是`Stream`或`Iterable`的缺点：`Collection`有返回类型为`int`的`size`方法，该方法将返回序列的长度限制为`Integer.MAX_VALUE`或`pow(2, 31) - 1`。如果集合更大，甚至无限，`size`方法返回的也是`pow(2, 31) - 1`，因此这个方法并不是一个完全令人满意的解决方案。

在`AbstractCollection`上编写`Collection`实现，除了实现`Iterable`所需的方法之外，只需要实现另外两种方法：`contains`和`size`。

应用示例：

编写一个方法，该方法返回输入列表的所有(连续的)子列表。生成这些子列表并将它们放到标准集合中只需要三行代码，但是保存这个集合所需的内存是源列表大小的二次方。

实现输入列表的所有子列表的流较为直接。首先，把所有包含第一个元素的子列表称之为列表的前缀（prefix）。例如，（a，b，c）的前缀是（a），（a，b）和（a，b，c）。类似地，把所有包含最后一个元素的子列表称为列表的后缀（suffix），因此（a，b，c）的后缀是（a，b，c），（b，c）和（c）。那么一个列表的所有子列表是它所有前缀的后缀（或所有的后缀的前缀）加上空列表。

		// Returns a stream of all the sublists of its input list
		public class SubLists {
		   public static <E> Stream<List<E>> of(List<E> list) {
			  return Stream.concat(Stream.of(Collections.emptyList()),
				 prefixes(list).flatMap(SubLists::suffixes));
		   }
		   private static <E> Stream<List<E>> prefixes(List<E> list) {
			  return IntStream.rangeClosed(1, list.size())
				 .mapToObj(end -> list.subList(0, end));
		   }
		   private static <E> Stream<List<E>> suffixes(List<E> list) {
			  return IntStream.range(0, list.size())
				 .mapToObj(start -> list.subList(start, list.size()));
		   }
		}

`Stream.concat`方法用于将空列表添加到返回的流中，或者在`rangeClosed`调用中用`(int) Math.signum(start)`替换1。`flatMap`方法（参阅45）用于生成由所有前缀的所有后缀组成的单个流。`IntStream.range`和`IntStream.rangeClosed`返回的连续int值流来生成前缀和后缀，这个习惯用法，等价于在整数流上使用标准的for循环。

		for (int start = 0; start < src.size(); start++)
			for (int end = start + 1; end <= src.size(); end++)
				System.out.println(src.subList(start, end));

### 48. 把流并行化时要加倍小心 ###

如果一个流管道的源来自`Stream.iterate`方法，或者在中间操作使用了`limit`方法，则并行化管道也不能提高其性能。因此不要无差别地并行化流管道（stream pipelines）。

并行化带来的性能收益在`ArrayList`、`HashMap`、`HashSet`和`ConcurrentHashMap`实例、数组、`int`类型的范围和`long`类型的范围的流上最能体现。这些数据结构的共同之处在于，它们都可以精确而廉价地分割成任意大小的子范围，从而让在并行线程之间划分工作变得很容易。在流的类库中，用于执行此任务的抽象是`spliterator`，它由`Stream`和`Iterable`上的`spliterator`方法返回。

这些数据结构的另一个重要共同点是它们在被顺序处理时提供了良好到极好的引用位置（locality of reference）：顺序元素引用在内存中存储在一块。这些引用所引用的对象在内存中可能彼此不接近，这就降低了引用的局部性。对于并行化批量操作而言，如果没有引用的局部性，线程大部分时间都处于空闲状态，等待数据从内存传输到处理器的缓存中。具有最佳引用局部性的数据结构是基本数据类型的数组，因为数据本身被连续存储在内存中。

流管道终端操作的性质也会影响并行执行的有效性。如果与管道的整体工作相比，在终端操作中完成了大量的工作，并且这个终端操作本质上是连续的，则并行化此管道的有效性是有限的。最适用于并行化的终端操作是缩减操作（reductions），即使用流的`reduce`方法之一或者预先打包的缩减操作(如`min`、`max`、`count`和`sum`)来提取管道里的所有元素并将它们组合在一起。短路操作如`anyMatch`、`allMatch`和`noneMatch`也适用于支持并行性。由`Stream`的`collect`方法执行的操作，称为可变缩减（mutable reductions），不适合并行性。

如果需要编写自己的`Stream`，`Iterable`或`Collection`实现，并且希望获得良好的并行性能，则必须重写`spliterator`方法并广泛测试生成的流的并行性能。

如果要并行化随机数流，使用`SplittableRandom`实例，而不是`ThreadLocalRandom`（或基本上过时的`Random`）。`SplittableRandom`专为此用途而设计。`ThreadLocalRandom`设计用于单个线程，虽然在作为并行流源时会自我调整，但性能不会像`SplittableRandom`一样快。

### 49. 验证方法参数的有效性 ###

如果对方法的参数有限制，必须在文档中加以说明，并在方法主体的开头就对传入参数的有效性进行检测，如果检测失败应该抛出适当的异常。

对公共方法和受保护的方法，使用javadoc的`@throws`标签来注明验证失败时所抛出的异常。通常这些异常是`illegalargumentexception`，`indexoutofboundsexception`或者`nullpointerexception`。

类一级注释中描述的异常适用于该类所有的公共方法的参数校验。如果有某个方法的某个参数允许为空，则可以使用`@Nullable`注解标记。

使用`objects.requirenonnull()`方法来做参数的非空校验，该方法的返回值就是参数本身，也可以自定义抛出异常的消息内容。

		this.strategy = Objects.requireNonNull(strategy, "strategy must not be null");

Java 9中加入了3个新的范围校验：

- `objects.checkfromindexsize(int fromIndex, int size, int length)`
- `objects.checkfromindextoindex(int fromIndex, int toIndex, int length)`
- `objects.checkindex(int index, int length)`

但这三个方法都不支持自定义异常消息，也只能用于列表和数组的校验并且范围不包括首尾

非公共方法可以直接使用断言（assertions）来校验参数，失败时抛出`AssertionError`异常。在java命令中使用`-ea`或者`-enableassertions`来激活断言语句，否则不会起作用，也不会占用资源。

对于那些没有在方法体中立刻使用，而是把传入参数存储在内部组件中留作后用的方法，更应该对其参数进行校验，最典型的例子就是构造方法和静态工厂方法。

参数校验的例外是当校验参数的开销很大，**并且**校验工作会在数据计算或处理过程中隐式完成时，可以不用校验参数，但如果隐式校验出错所抛出的异常与方法签名中声明的异常不同时，仍然要使用异常翻译（参阅76）将其转换为声明的异常。

尽量在设计方法时减少对输入参数的限制。

### 50. 需要时使用防御性拷贝 ###

首先有一个小tip：不要使用`java.util.date`类应该使用`java.time`包下的类

如果传给某个类的构造方法的参数是可变的，应该创建参数的防御性拷贝来进行操作，而不是直接操作传入的参数。对参数的验证也应该基于这些拷贝，而不是原始参数。这样做是为了防御toctou(time-of-check/time-of-use)攻击。

不要使用`clone()`方法对可能被不可信任方继承的类的对象做防御性拷贝

`accessor`方法返回值时也应该考虑使用防御性拷贝，虽然也可以使用`clone()`方法，但最好还是使用构造方法或静态工厂方法来创建防御性拷贝。

如果一个对象在构建过程中传入了可变参数，并且这个参数的变化会影响到该对象的状态（成员变量）或服务（方法），则应该在传入参数时建立该参数的防御性拷贝，并且只使用这些拷贝，而不是使用原始参数。使用内部组件作为返回值的方法也应该考虑使用该成员变量的防御性拷贝作为返回值，以防止外部程序对其非法操作。

需要注意的是非零长度的数组总是可变的。因此，在传入或传出非零数组时都应该考虑创建防御性拷贝，或者创建它的不可变视图。

在类的内部尽可能使用不可变类的对象作为它的组件，从而避免使用防御性拷贝。

小结：如果类含有可变的组件，该组件的值来自于外部或将返回外部，则必须创建并使用它的防御性拷贝完成这些工作。如果创建防御性拷贝的成本过高，**并且**类的使用者不会对其有不适当操作，则可以不使用防御性拷贝，但必须在文档中注明类的使用者不能修改哪些组件。

### 51. 谨慎设计方法的声明 ###

设计公共api时，不要过于热衷提供便利方法。类或接口所支持的每个行为都应该提供功能完备的方法。只有在某段代码会被经常用到才考虑为他单独设计一个方法。

避免过长的参数列表，最好4个以下。三种方法来缩减参数列表的长度：

- 将方法分解为多个方法，从而让每个方法都只接受一部分参数；
- 创建一个帮助类，这个类通常是静态成员类。当不同的参数组合可以代表不同的个体时，如纸牌的大小和花色组合可以唯一指定一张牌，推荐使用这种方法；
- 套用建造者模式，当可能需要很多参数，但其中某些参数只是可选参数时，可以先创建一个包含了所有参数的对象（可选参数可以使用默认值），然后再为可选参数分别提供设置方法，当需要的时候，通过调用这些设置方法来提供可选参数的值，最后再调用此对象的`执行`方法，这个执行方法会对所有的参数做最后的验证并执行真正的操作。

参数类型应该优先使用接口而不是类。

不要使用布尔类型的参数来代表只有两个值的类型，除非这个方法就是需要接受布尔类型的参数，否则应该为其创建一个枚举类。

### 52. 谨慎使用重载 ###


- 选择执行哪个重载方法是静态的，是在编译期间决定的
- 选择执行哪个重写方法是动态的，是在运行期间决定的

java无法根据参数在运行期间的实际类型来选择调用正确的重载方法。解决方法是不使用方法的重载，而是在方法内部对传入参数的类型进行判断，根据判断结果执行不同的代码段或方法。

		public static String classify(Collection<?> c) {
			return c instanceof Set ? "Set" :
				c instanceof List ? "List" : "Unknown Collection";
		}

一个保守但安全的重载方法的原则：永远不要重载2个含有相同数量参数的方法，也不要重载使用varargs的方法。可以为使用不同的方法名来取代重载，如在方法名中加入对参数类型的描述`writeInt`，`readLong`，`comparingInt`。

当参数的类型是不同族的时候，即两个类型的非空值不能互相转型时，就可以让方法接受相同数量的参数。

不要在相同的参数位置使用不同的函数式接口来实现方法的重载。

数组类型和除了`object`类的其他类型都是不同族的，也和除了`cloneable`和`serializable`之外的接口不同族。当两个类互不为对方的后代时，这两个类是无亲属关系的，也是不同族的。

如果在特殊情况下出现了重载方法参数同族的情况，可以让接受更具体参数的方法在内部调用接受更广泛参数的方法来减少可能造成的困扰。

	//通过方法转调确保2个重载的两个方法拥有一致的行为
	public boolean contentEquals(StringBuffer sb) {
		return contentEquals((CharSequence) sb);
	}

小结：最好避免重载拥有相同数量参数的方法，如果避免不了，则应该使用转型来避免不同的方法可以接受同一种类型，如果还是避免不了，则应该确保这些不同方法的内部对这个参数的处理保持一致。

### 53. 谨慎使用可变数量参数varargs ###

可变数量参数是先创建一个长度等于参数数量的数组，然后把每个参数放入数组中，最后把该数组传递给方法。

因为数量可变的参数总是从0开始计数，所以当方法需要1个以上数量的可变数量参数时，需要声明两个参数，一个参数作为此类型的第一个参数，另一个参数是该类型的可变数量参数。

因为数量可变的参数总是会创建一个数组，所以为了避免非必要的性能损耗，可以使用以下窍门：为最经常出现的参数组合分别创建重载方法，然后再提供一个带可变数量参数的方法应对不经常出现的情况。

### 54. 返回空集合或数组，而不是null值 ###

如果方法返回`null`值，它的使用者就必须对返回值做非空判断，因此应该考虑返回空集合或空数组。

如果不想每次都创建新的空集合或空数组，可以利用如下代码返回不可变的空集合或空数组（不可变的对象是可以被安全共享的）

		private final list<product> productinstock = ...;
		//返回空集合
		public list<product> getproduct() {
			return new arraylist<>(productinstock);
		}

		//使用可共享的空集合优化的返回空集合
		public list<product> getproduct() {
			return productinstock.isempty() ? 
				collections.emptylist() :
				new arraylist<>(productinstock);
		}

		//返回空数组，传入空数组用于指定返回类型
		public product[] getproducts() {
			return productinstock.toarray(new product[0]);
		}

		//使用不可变的空数组优化的返回空数组
		private static final product[] empty_product_array = new product[0];
		public product[] getproducts() {
			reeturn productinstock.toarray(empty_product_array);
		}

		//不要在使用toarray方法转换数组时创建提前分配好长度的数组，因为会影响性能
		//以下方法为错误示例
		public product[] getproducts() {
			reeturn productinstock.toarray(new product[productinstock.size()]);
		}
		
### 55. 小心返回optionals ###

在Java 8以前，有两种方法处理无法返回正常值的情况：

- 抛出一个异常
- 返回一个`null`值

Java 8中的`Optional<T>`代表了一个可以最多包含一个非空`T`类型的引用或者什么都没有的不可变容器（空容器）。也可以把它当成是一个最多只能包含一个元素的集合，虽然它并没有实现`collection`接口。

一个正常情况下应该返回一个`T`类型对象的方法，也可能在某些特殊情况下无法返回值，此时该方法可以使用`Optional<T>`声明它的返回类型。当方法无法返回值时，就返回一个空`Optional<T>`对象，这比抛出异常的方法更灵活，更易用，也比返回`null`的方法更不容易出错。

具体方法是在方法内部使用相应的静态工厂方法生成`Optional<T>`对象。比如使用`Optional.empty()`返回一个空容器，使用`Optional.of(value)`返回带值的对象。注意`Optional.of(value)`方法不接受`null`值，但`Optional.ofNullable(value)`方法可以接受`null`值并返回一个空容器。因此永远不要在返回类型是`Optional<T>`的方法中直接返回一个`null`值。

示例代码如下：

		//使用抛出异常的方法来处理空值
		public static <E extends Comparable<E>> E max(Collection<E> c) {
			if (c.isEmpty())
				throw new IllegalArgumentExcepton("Empty collection");
			E result = null;
			for (E e : c)
				if (result == null || e.compareTo(result) > 0)
					result = Objects.requireNonNull(e);
			return result;
		}

		//使用Optional<E>来处理空值
		public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
			if (c.isEmpty())
				return Optional.empty();
			E result = null;
			for (E e : c)
				if (result == null || e.compareTo(result) > 0)
					result = Objects.requireNonNull(e);
			return Optional.of(result);
		}

基于流的很多终端操作都返回`Optional`。

optionals有些类似检测的异常，他们都强制要求api的使用者必须面对无法返回值的情况。抛出未检测异常或者返回`null`值都让用户可以忽略这种情况从而埋下隐患。而抛出检测的异常又会要求用户增加额外的处理代码。

可以为`Optional<T>`指定默认值或者默认抛出一个异常。当默认抛出异常时要注意传递的参数不是一个实际的异常，而是一个异常工厂方法，这样可以只在它需要被抛出的时候才创建该异常。

		//为`Optional<T>`提供默认值
		String lastWordInLexicon = max(words).orElse("no words...");

		//为`Optional<T>`提供默认抛出异常
		Toy mytoy = max(toys).orElseThrow(TemperTantrumException::new);

在没有指定如何处理空值的空容器上调用`get`方法会抛出`NoSuchElementException`异常

如果创建默认值的开销过大，可以使用`orElseGet`方法，该方法接受一个`Supplier<T>`对象并只在需要的时候才会使用该对象创建默认值，从而避免不必要的开销。

如果以上方法都无法满足需要，用户想自己定义返回结果的值，可以和`isPresent()`方法结合定制自己的返回值。但在使用这个方法前先确定是否有其他方法已经能够满足需求。

在Java 9以前，如果需要把`Stream<Optional<T>>`对象转换为`Stream<T>`对象，可以使用如下代码：

		streamOfOptionals.
			filter(Optional::isPresent).
			map(Optional::get);

Java 9以后，可以使用`Optional`的`stream`方法来把一个`Optional`对象转换为只含一个元素或0个元素的`Stream`对象，再用`flatMap`方法将他们合并成一个流：

		streamOfOptionals.flatMap(Optional::stream);

所有的容器类型，包括collections，maps，streams，arrays和optionals都不能封装在optionals中。

什么时候使用`Optional<T>`而不是`T`作为方法的返回类型：
当该方法可能没有返回值并且该方法的使用者必须对这一情况做特殊处理。

直接使用基本数据类型作为optionals的封装类型。`int`，`long`，`double`类型有对应的`OptionalInt`，`OptionalLong`和`OptionalDouble`类。其他的基本类型的封装类虽然可以被optionals使用，但应该小心使用。

optionals不能作为maps的键值使用，事实上，optionals永远都不要作为集合或数组的key，value，或元素来使用。

optionals可以作为成员变量的类型使用，表示该成员变量是一个可选属性，可能没有在构造方法中被赋值，或者可能在某个子类中没有被使用。当该变量的类型本应该是基本数据类型时，无法有效表示该值为空，此时就可以使用`Optional`作为变量的类型。

总之，尽量不要把optionals用于除返回类型以外其他的用途。

### 56. 为所有发布的api元素撰写文档 ###

在每一个发布的类，接口，构造方法，方法和成员变量的声明前添加文档注释。如果一个类是可序列化，也应该在文档注释中说明它的序列化格式。公共类不应该使用默认的构造方法因为无法为其添加文档注释。

方法的文档注释应该简洁明了地阐述该方法和其使用者之间应该遵守的协议。该协议定义了该方法能做什么而不是怎么做。文档注释应该列出方法的前置条件和后置条件。前置条件往往通过使用`@throws`注解所抛出的未检测异常来表示，每个未检测异常表示违反了一个前置条件。前置条件也可以在它所影响的参数的`@param`注解中说明。方法的副作用是指在运行过程中可被观察到的系统状态的变化，虽然副作用并不是满足后置条件的必需条件，但还是应该记录在方法的文档中。

文档注释应该为每一个参数提供`@param`注解，为除了`void`以外的返回类型提供`@return`注解，以及为每一个抛出的异常（包括检测的和未检测的）提供`@throws`注解。

- `@param`和`@return`注解应该使用名词词组作为注释，偶尔也会用算数表达式；
- `@throws`应该包含一个if从句来说明异常产生的条件；
- `@param`和`@return`以及`@throws`通常不会在结尾使用句号。

使用`{@code}`注解可以加入代码块，让其以代码的格式在生成的文档中显示。在花括号内的html标签也会失去作用。如果需要多行代码，可以把该注解用html标签`<pre>`包装起来：`<pre>{@code code...}</pre>`，这样就可以保留代码中的回车换行，同时也不需要转义除`@`之外的其他html元字符。

当设计继承用的父类时，必须在其文档中说明它自调用模式。这些自调用模式说明应该用`@implspec`注解标明。需要注意的是，通常的注释是阐述方法和其使用者之间的协议；而`@implspec`则是阐述方法和该类的子类之间的协议，也即子类在继承或通过`super`关键字调用该方法时所依赖的父类方法实现。

`{@literal}`注解和`{@code}`注解除了不以代码格式显式所注解的内容外，功能完全一样，都可以用于显示html元字符。

文档注释在源代码中和生成的文档中都应该具有良好的可读性，如果无法兼顾，以生成的文档为先。

每个文档注释的第一句都应该是该注释所说明的元素的总结性描述。类或接口两个成员或构造方法的注释不应该有相同的总结描述。总结描述总会以遇到的第一个后跟空白（包括空格，tab，换行符）的的句号作为结束标志，如果这样的句号需要作为描述的内容时，需要使用`{@literal}`把它包装起来。

- 传统上，方法和构造方法的总结描述应该是一个描述了方法所执行的操作的动词词组（包括宾语）；
- 类，接口和成员变量的总结描述应该是一个该类或接口的对象或者该成员变量本身所代表的事物。

从Java 9开始，可以使用`{@index}`注解将注释的内容加入到javadoc的索引当中便于搜索。

- 泛型类型或方法的注释应该对所有的泛型类型参数做出说明
- 枚举类型的注释需要对每个枚举值，该类型以及公共方法做出说明，文档注释可以只占一行。
- 注解类型的注释需要对它的每一个成员，以及注解类型本身做出说明。成员的总结描述用名词词组，注解类型本身的注释需要使用动词词组来说明什么时候使用该注解。

包一级的注释应该放在一个名为`package-info.java`的文件中；模块一级的注释应该放在一个名为`module-info.java`的文件中。

不管一个类或静态方法是不是线程安全的，都应该在文档中说明它的线程安全等级。如果一个类是可序列化的，应该在文档中说明它的序列化格式。

javadoc可以继承方法的注释。如果一个api元素没有想用的文档注释，则javadoc会寻找最适合的文档注释，其中实现的接口优于父类。也可以使用`{@inheritDoc}`注解来继承父类的部分注释。

如果API很复杂，包含了很多相互关联的类，往往就需要一个单独的外部文件来描述API的整体结构，而与之相关的类或包的注释里都应该包含此文件的链接。

Java 9后可以使用`-html5`来将生成文档的格式从html4.01转换为html5。

### 57. 最小化局部变量的作用域 ###

最好的最小化局部变量的作用域的方法是在第一次使用它时再声明。

几乎所有的局部变量的声明都应该包含一个初始语句。如果还没有足够的信息来初始化该变量，可以等稍后有了足够的信息再声明并初始化该变量。一个例外是try-catch语句块。如果使用了可能抛出异常的表达式来给该变量赋值，则该变量必须在try代码块中初始化。如果该变量必须在try代码块以外的地方使用，那么它必须在try代码块之前声明。

如果循环变量的值不会在循环结束后使用，则和while循环相比，优先使用for循环。如果需要访问迭代器，如需要调用它的`remove`方法，则应该优先使用传统的for循环而不是for-each循环。

在传统的for循环中，如果循环变量的上限需要调用一个方法或进行计算而且每次返回的结果都相同，则可以在初始化循环变量的时候分配两个循环变量同时指定初始值和上限值。

		for (int i = 0, n = expensiveComputation(); i < n; i++) {
			... //do something with i;
		}

如果有一个方法在循环体内被反复调用且每次调用的返回值都相同，则可以考虑创建一个局部变量存储其返回值，减少方法调用次数。

尽量将方法精简并专注于一个任务。

### 58. 优先使用for-each循环，而不是传统for循环 ###

for-each循环中的`:`可读为"in"，循环条件可念为"fro each element e in elements"

三种通常不能使用for-each循环而必须使用传统for循环的情况：

- 破坏性过滤：当需要遍历一个集合以删除选定的元素时，需要使用一个显式的迭代器以便调用它的`remove`方法。Java 8以后可以通过使用`Collection`的`removeIf`方法来避免显式的遍历。
- 更改内容：如果需要遍历一个列表或数组并且替换某些或全部元素的值，则需要该列表的迭代器或数组的索引来替换元素的值。
- 同步迭代：如果需要同步遍历多个集合，则需要显式控制迭代器或索引变量，这样就可以在同步语句中同时递进该迭代器或数组索引。

		//使用传统for循环创建牌组
		for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
			Suit suit = i.next();
			for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
				deck.add(new Card(suit, j.next()));
		//使用for-each循环创建牌组
		for (Suit suit : suits)
			for (Rank rank : ranks)
				deck.add(new Card(suit, rank));

如果需要自己编写一个类来代表一组元素，则不管它是否实现`collection`，都应该考虑实现`iterable`接口。

### 59. 了解并使用已有类库 ###

Java 7以后都不应该再使用`Random`类。无并行需要则使用`ThreadLocalRandom`类；需要并行则使用`SplittableRandom`类。

必须要掌握了解的类库：`java.lang`，`java.util`，`java.io`以及他们的子包。同时也需要了解集合框架和流类库以及负责同步的`java.util.concurrent`包。

### 60. 避免在需要精确答案时使用`float`和`double`类型 ###

`float`和`double`类型不适合用于需要精准数值的计算，如金融领域。此时可以使用`BigDecimal`，`int`和`long`。

注意`BigDecimal`类的构造方法中，使用`String`作为输入参数类型的构造方法比使用`double`作为输入参数的构造方法更常用。

使用`BigDecimal`类有2个缺陷：不如基本数据方便；处理起来比较慢。

`BigDecimal`类的替代方案是使用`int`或`long`，即把小数转换为整数再做处理，如0.23变为23

小结：在需要精准计算的时候不要使用`float`和`double`类型。如果不介意使用引用类型导致的不方便和开销，则使用`BigDecimal`类来获得准确的舍入计算结果。如果性能很重要，数值也不大，且不介意自己控制小数点的位置，则可以使用`int`和`long`类型。如果数值不超过9位，则使用`int`，如果不超过18位，则使用`long`，如果超过18位，则使用`BigDecimal`。

### 61. 优先使用基本数据类型，而不是他们的封装类型 ###

基本数据类型和封装类型的3个主要区别：

- 基本类型只有值，而封装类型对象的个体识别与其值无关。2个封装类型的对象可能具有相同的值，却是2个不同的个体；
- 基本数据类型只有有效值，而封装类型除了有有效值之外还有一个非有效值`null`；
- 基本数据类型比封装类型在时间和空间上都更有效率。

不能在封装类型上使用`==`运算符来判断两者是否相等，因为封装类型是引用类型，该运算符比较的是2个变量是否引用的是同一个对象，而不是对二者的值进行比较。

如果需要一个比较器来描述一个类型的自然排序，可以调用`Comparator.naturalOrder()`方法；如果定制比较器，可以使用比较器构造方法或者在基本数据类型上使用静态比较方法。

当在一个操作中混合使用了基本数据类型和封装类型，则封装类型会被自动拆箱，因此如果该变量的值是`null`，就会在自动拆箱过程中引发`NullPointerException`异常。

使用封装类型的时机：

- 封装类型作为集合的elements，keys和values，因为基本数据类型不能放入集合中；
- 在泛型类和泛型方法中使用封装类型作为类型参数；
- 在使用反射机制调用方法时；

### 62. 当有其他类型适合时避免使用`String`类表示一个类 ###

- `String`不适合用来表示其他价值类。如果有一个现有的价值类，无论是基本数据类型还是引用类型，就用该类来表示此对象；如果没有，就创建一个类型，不要使用`String`类。
- `String`类不适合用来表示枚举类。
- `String`类不适合用来表示聚合类。如果该类有多个组件，则不应该只用一个字符串来表示这个对象。更好的方法是创建一个私有静态成员类来表示该聚合。
- `String`类不适合用来表示不可伪造的键值capability-作为维持对象唯一性的，不可伪造的密匙。

		//inappropriate use of Stirng as capability
		public class ThreadLocal {
			private ThreadLocal() {} //Noninstantiable

			//set the current thread's value for the named variable
			public static void set(String key, Object value);

			//return the current thread's value for the named variable
			public static Object get(String key);
		}

		//imporved version - replace the string with an unforgeable key
		public class ThreadLocal {
			private ThreadLocal() {} //Noninstantiable

			public static class Key { //(capability)
				Key() {}
			}

			//generate a unique, unforgeable key
			public static Key getKey() {
				return new Key();
			}

			public static void set(Key key, Object value);
			public static Object get(Key key);
		}

		//even better - convert the key to a thread-local variable directly with typesafe
		public final calss ThreadLocal<T> {  //this used to be the nested class - Key
			public ThreadLocal();
			public void set(T value);
			public T get();
		}

小结：尽量避免在有更好的数据类型存在时或可以创建一个类型时使用字符串来表示一个对象。

### 63. 注意合并字符串操作的性能 ###

重复合并n个字符串需要2的n次方的时间。当需要合并的字符串过多时，使用`StringBuilder`来代替`String`，或者使用字符数组，或者一次性处理该字符串，而不是多次合并。

### 64. 使用接口作为对象的引用类型 ###

如果有合适的接口存在，那么参数，返回类型，局部变量和成员变量都应该使用接口来声明。

不使用接口而使用类声明变量的3种情况：

- 如果没有合适的接口来表示声明类型，就直接使用该类的类型声明。价值类通常都是final的，没有实现接口，不会有多个实现，因此可以直接使用价值类来作为参数，变量和返回类型。
- 如果某个框架是基于类而不是接口，那么属于该框架的对象应该使用框架中相应的基类（通常是抽象类）来声明。
- 如果一个类在实现了接口的同时也提供了额外的方法并且它的使用者依赖于这些额外的方法。

小结：如果有合适的接口，就用接口作为参数，变量，返回值的类型。如果没有合适的接口，则在继承树中使用提供了所需要的功能的最接近的父类来声明。

### 65. 优先使用接口，而不是反射 ###

使用反射的缺点：

- 丧失了编译期类型检查，包括异常检测，所带来的好处。如果程序尝试使用反射调用不存在或无法访问的方法，程序只能等到运行时才会出错。
- 使用反射的代码复杂而冗长
- 性能开销大

如果程序需要使用一个在编译期不存在的类，而该类实现或继承了可以在编译期确定的接口或父类，则使用者可以通过反射来创建该类型的对象并通过这些接口或父类来访问该对象。

		//使用接口访问通过反射创建的对象
		public static void main(String[] args) {
			//根据类名生成类对象
			Class<? extends Set<String> cl = null;
			try {
				//未检测的类型转换，forName方法的返回类型是Class<?>
				cl = (Class<? extends Set<String>>) Class.forName(args[0]);
			} catch (ClassNotFoundException e) {
				fatalError("Class not found");
			}

			//获取构造方法
			Constructor<? extends Set<String>> cons = null;
			try {
				cons = cl.getDeclaredConstructor();
			} catch (NoSuchMethodException e) {
				fatalError("No parameterless constructor");
			}

			//创建实例
			Set<String> s = null;
			try {
				s = cons.newInstance();
			} catch (IllegalAccessException e) {
				fatalError("Constructor not accessible");
			} catch (InstantiationException e) {
				fatalError("Class not instantiable");
			} catch (InvocationTargetException e) {
				fatalError("Constructor threw " + e.getCause());
			} catch (ClassCastException e) {
				fatalError("Class doesn't implement Set");
			}

			//使用创建的set实例
			s.addAll(Arrays.asList(args).subList(1, args.length));
			System.out.pringln(s);
		}

		private static void fatalError(String msg) {
			System.err.println(msg);
			System.exit(1);
		}

上述代码使用的技巧可以用来实现服务提供者框架（参阅1），通常这就是需要用到反射的全部地方。

注意那个未检测的类型转换，`forName`方法的返回类型是`Class<?>`，所以强制类型转换的结果有可能不是`Class<? extends Set<String>>`类型。

使用反射主要是为了管理类对其他可能在编译期间不可用的类，方法或者成员变量的依赖。比如某个包A依赖于另一个有不同版本的包B来运行，则可以让这个包A依赖于最简单的B包版本B1来编译，然后通过反射来使用新版本B包中新添加的类或方法。总之，尽可能只用反射创建对象，然后通过编译期就可以确定的接口或父类使用这个对象。

### 66. 小心使用本地方法（native methods） ###

不推荐为了提高性能而使用本地方法。

### 67. 谨慎执行优化 ###

努力编写好的程序而不是快速的程序。

在设计系统，尤其是API，互动协议以及持久数据格式的时候，就应该考虑性能上的要求。只有在仔细设计了程序，并且简洁明了地完成了架构良好的实现之后才应该开始考虑优化。而且在优化前后都应该使用profiler对程序做性能测量。

优化性能的第一步应该是对使用的算法重新评估，看是否有更好的选择。

### 68. 坚持使用广泛接受的命名规则 ###

书写规则:

- 包名和模块名应该使用句点和他们的组件分隔开以体现层次。组件名使用小写字母或数字（比较少见），通常8个字符或更短，可以使用有意义的简写或缩写。在单位外使用的包名应该以单位的互联网域名的反向写法命名。用户不能使用以`java`和`Javax`开头的包名或模块名。
- 类名，接口名，包括枚举类名和注解类名都应该包含一个或多个单词，每个单词的首字母大写。应该避免不常见的简写，但可以使用缩写，缩写的首字母大写。
- 方法和成员变量名遵循类名，接口名的命名规则，除了第一个单词的首字母要小写。常量名使用一个或多个全部大写的单词，每个单词以下划线分隔，这也是唯一在命名中使用下划线的地方。
- 局部变量名遵循成员变量命名规则，但是可以使用简写。作为方法参数的局部变量名尽量使用有意义的单词，因为他们将作为方法文档的一部分出现。
- 泛型类或泛型方法的类型参数通常是一个大写的字符，常用的有5种：
	- T(ype)：表示任意类型。当需要表示多个类型时，可以使用T，U，V或者T1，T2，T3
	- E(lement)：表示集合中的元素类型
	- K(key)，V(alue)：分别表示map中的键值和值
	- (e)X(ception)：表示异常
	- R(eturn)：表示返回类型

语法规则：

- 可实例化类名，包括枚举类名，通常使用一个或多个名词短语；
- 非实例化的工具类名，通常使用一个名称的复数形式，如`Collections`；
- 接口名类似类名，或者使用一个形容词性的结尾`able`或`ible`；
- 注解名可以使用任意词性的词，名词，动词，介词，形容词都可以；
- 执行某个操作的方法名通常使用动词或欧动词短语（包含宾语）；
- 返回布尔类型的方法通常以`is`或`has`（较少见）开头，后跟名词或名词短语，或任何可作为形容词的单词或短语；
- 返回非布尔类型的方法通常直接以返回的属性的名称，或在其前面加上`get`来命名。对于没有setter方法的属性可以直接使用属性名作为getter方法名使用；
- 变量名通常以名词或名词短语命名。布尔类型的变量以相应的getter方法去掉开头的`is`或`has`来命名。

特殊方法的命名规则：

- 返回一个独立的，不同类型的对象的方法。通常命名为`toType`，如`toArray`，`toString`
- 返回一个依赖于此对象，不同类型的对象（view）的方法。通常命名为`asType`，如`asList`
- 返回一个与对象等值的基本数据类型对象的方法。通常命名为`typeValue`，如`intValue`
- 静态工厂方法常见的名称有
	- `from`
	- `of`
	- `valueOf`
	- `instance`
	- `getInstance`
	- `newInstance`
	- `getType`
	- `newType`
	
### 69. 只在例外的情况下使用异常 ###

异常应该只用来处理例外的情况，而不应该用于正常的流程控制

如果一个类有一个依赖于状态的方法（即根据状态的不同会执行不同的方法体），并且该方法只会在一些不可预测的特定条件下才会被调用，那么该类通常会有一个相应的用于判断状态的方法来检测是否满足调用条件。最典型的例子是`Iterator`接口的`next()`和`hasNext()`方法。

另一种方法是让这个依赖于状态的方法在无法按预期执行时，返回一个空的`Optional`对象或者一个可轻易辨识的值，如`null`值。

#### 这两种方法的选择 ###
#
如果一个对象能够在没有使用外部同步监视器的情况下被并发访问，或者会遇到由外部因素引起的状态改变，则应该使用返回一个空的`Optional`对象或者一个可轻易辨识的值。因为如果使用判断状态的方法，在两个方法调用的间隙，对象的状态可能发生改变，从而导致先前的判断无效。另外，如果使用状态判断的方法使用了大量与配对方法重复的代码，也应该使用直接在功能方法中返回空`Optional`值来处理。其他情况都可以优先搭配状态判断方法来使用。从而提高代码的可读性以及便于在发生错误时及时捕捉。

### 70. 使用检测异常处理可恢复情况，使用运行时异常处理编程错误 ###

如果程序在运行出问题时可以恢复到正常状态，则使用检测异常，这也是告诉用户这是API可能出现的“正常”问题，用户需要进行处理。

使用运行时异常来标识编程错误。当一个程序抛出运行时异常时，通常表示程序无法恢复到正常状态，继续运行可能引发更严重错误。最常见的运行时异常是`precondition violations`，即违反代码运行先决条件，比如`ArrayIndexOutOfBoundsExcetpion`。所有自定义的运行时异常都应该是`RunTimeException`的子类，也不应该抛出除`AssertionError`以外的任何`Error`及其子类。

异常类中也可以定义属于自己的方法，这些方法可以用于描述异常，比如说明异常产生的原因，来源等。对于检测异常，可能还要提供额外的信息以便于使用者恢复程序的状态。

小结：

- 使用检测异常处理可恢复情况，使用未检测异常处理编程错误。
- 如果不确定，则优先使用未检测异常。
- 不要定义任何非检测异常或非运行时异常。
- 在检测异常中提供用于帮助恢复程序状态的方法。

### 71. 避免过度使用检测异常 ###

在Java 8中，抛出检测异常的方法不能在流中直接使用。

同时满足以下两个条件才考虑使用检测异常：

- 异常需要处理的意外情况即使在正确使用API时也会出现；
- API的使用者可以采取有效措施来处理这些异常。

如果不满以上两个条件，就应该使用未检测异常。

如果一个方法被放入`try`代码块中或者不能在`stream`中直接使用只是因为该方法抛出了一个检测异常，就应该考虑是否可以有方法避免该异常。

方法一：让方法返回一个该类型的`Optional`值。缺点是无法像异常一样提供问题的相关信息。

方法二：创建一个相应的判断方法用于判断是否会出现问题，然后使用未检测异常来处理问题。缺点是在两个方法调用的间隙，判断条件可能发生变化，或者判断条件的代码开销大且和主方法重复（参阅69）。

		//使用检测异常的调用
		try {
			obj.action(args);
		} catch (TheCheckedException e) {
			...//处理产生的异常
		}

		//使用状态检测方法和未检测异常的调用
		if (obj.actionPermitted(args)) {
			obj.action(args);
		} else {
			...//处理产生异常的条件
		}

小结：如果用户不能恢复程序的运行状态，则使用未检测异常。如果运行可恢复，且想强迫用户必须处理该意外情况，则优先考虑返回一个`Optional`值。只有当这种方法无法提供足够必须的信息时，再考虑使用检测异常。

### 72. 尽量使用已有标准异常 ###

尽量使用Java已有的标准异常。

常用异常包括：

**异常**                        | **使用场景**
------------                    | ---------------------------
IllegalArgumentException        | 方法的非空参数的值不合法
IllegalStateException           | 方法要处理的对象的状态不能用于调用该方法
NullPointerException            | 为非空参数传入`null`值
IndexOutOfBoundsException       | 索引大小超出集合或数组边界
ConcurrentModificationException | 在一个为单线程设计，禁止并发操作的对象上执行了并发操作
UnsupportedOperationException   | 对象不支持此方法，如没有实现接口的某些可选方法

还有在处理复杂和关系性数字时常用的`ArithmeticException`和`NumberFormatException`异常。

不要直接使用`Exception`，`RunTimeException`，`Throwable`或`Error`。

使用已有异常时应该基于异常文档中的语义说明，而不是异常名称。可以通过继承创建自己的异常类，但切记异常必须实现序列化。

当同时可以使用`IllegalArgumentException`和`IllegalStateException`时，如果参数的所有可能值都无法生效，就使用`IllegalStateException`异常，否则使用`IllegalArgumentException`异常。

### 73. 抛出的异常必须符合相应的抽象 ###

不要在抽象等级较高的方法中直接抛出他所调用的抽象等级较低的方法所抛出的低级异常。

异常转型(exception translation)：抽象等级较高的层次应该捕捉抽象层次较低的异常，并且抛出对应的符合他所在层次的异常。可以理解为捕捉到抽象层次较低的异常后，用抽象层次较高的异常将其包装，然后抛出该高级异常。

异常链(exception chaining)：把抽象层次较低的异常传递给抽象层次较高的异常，然后提供一个访问方法来访问该低级异常。最常见的方法是把低级异常作为参数传递给高级异常，表示该低级异常是该高级异常的来源，然后使用高级继承自`Throwable`的`getCause()`方法来访问该低级异常。如果高级异常没有带参数的构造方法，则可以使用继承自`Throwable`的`initCause()`方法来设置。

不要滥用异常转型，最好的处理低级层次的异常的方法是尽量通过让低层次方法成功执行来避免产生低层次异常。比如高层次方法在调用低层次方法前，对传递的参数进行校验。如果实在无法避免产生低层次的异常，最好的处理方法是在高层次方法的内部就完成对低层次异常的处理，而不是将它包装后再次抛出。这样就可以将最终用户和低层次问题隔离开。在这种情况下，考虑使用适当的日志机制，如`java.util.logging`，将发生的问题和采取的解决方案记录下来。

小结：如果不能阻止或者处理来自底层的异常，并且该异常与方法所在的抽象层次不符，则使用异常转型将其包装为与抽象层次相符的异常再抛出。异常链可以有效实现该功能。

### 74. 为方法抛出的所有异常编写文档 ###

总是单独声明抛出检测异常并使用Javadoc的`@throws`注解来注明其产生的原因。不要用声明所抛出异常的共同父类的方法来减少文档的说明。唯一的例外是`main()`方法。

一个好的API文档也应该对方法可能抛出的未检测异常的产生条件做出说明，从而说明该方法能够正确执行的先决条件。所以每一个公共方法的文档都应该阐明它的前置条件。接口里的方法也应该为其可能抛出的未检测异常编写文档。

使用javadoc的`@throws`注解为方法可能抛出的所有异常，包括检测异常和未检测异常编写文档。但在代码中方法的签名里不要使用`throws`关键字抛出未检测异常。

如果一个异常在一个类中被多个方法以相同的理由抛出，可以将对它的说明放在该类的文档注释中。

小结：使用`@throws`注解为方法可能抛出的每一种异常编写文档，无论该异常是检测异常还是未检测异常，也无论该方法是抽象方法还是实际方法。但是在方法的声明中只用`throws`关键字抛出检测异常。

### 75. 在详细的异常消息中包含失败时捕捉到的相关信息 ###

在异常的详细消息里应该包含所有导致此次异常的参数或成员变量的值。如和`IndexOutOfBoundsException`异常有关的索引，索引上限，索引下限。

不要在异常消息中包含包括密码，加密密匙等类似敏感信息。

一种保证异常会在它的详细消息中包含必须信息的的方法：把必须的信息作为参数传递给异常的构造方法，然后在异常内部使用这些参数生成详细的消息，然后调用父类或者通过重载的带消息的构造方法创建异常对象。可能也需要提供访问这些信息的访问方法。

### 76. 争取实现失败原子性 ###

理想状态：一个失败的方法调用应该将对象还原到方法调用前的状态。

可采取的方法：

1. 设计并使用不可变类的对象；
2. 在执行操作前校验传入参数的合法性；
3. 把任何可能导致异常的操作放在更改对象状态的操作前执行。；
	2,3方法都是尽量让异常在对象发生变化之前产生；
4. 为对象建立一个临时拷贝并使用它来进行执行操作，等所有操作成功完成后再用临时拷贝替换对象；
5. 最后一种方法是编写恢复代码，让对象回滚到调用方法前的状态，但这种方法不常用。主要用于基于磁盘的数据结构。

如果两个线程尝试在没有适当同步的情况下同时修改同一个对象，则该对象可能处于不一致状态。因此，在捕捉到`ConcurrentModificationException`异常后必须假设该对象的状态已发生改变。

错误是无法恢复的，所以不必尝试恢复，例如`AssertionError`。

小结：任何在方法签名中声明的异常都应该将对象恢复到方法调用前的状态。如果无法实现，应该在文档说明中清楚指明对象可能处于的状态。（虽然经常看到`unpredictable`）。

### 77. 不要忽略异常 ###

空的`catch`块会违背设计异常的目的。

如果一定要忽略一个异常，一定要在相应的`catch`块中注明原因，并且把异常变量命名为`ignored`。

		try {
			...
		} catch (TimeoutException | ExecutionException ignored) {
			//explain why ignore this exception
		}

请记住，不向外扩散异常可以保证在程序发生错误的地方就可以得到及时处理，至少可以保存错误的相关信息以便纠错。

### 78. 同步对共享可变数据的访问 ###

`synchronized`关键字确保一个方法或代码块同一时间只能被一个线程执行。

同步不仅可以阻止线程观察处于非持续状态的对象，也确保了每一个线程在进入被同步的方法或代码块的时候会看到之前在同一个所控制下的更改。

Java语言规范规定除`long`和`double`以外类型的变量的读和写都是原子性的。换句话说，读取一个除`long`和`double`以外类型变量的值的返回结果一定是先前某个线程存放进去的，即使有多个线程在同时修改该变量但并没有使用同步。

同步起到2个重要的作用：

- 线程对公共资源访问的互斥
- 线程之间以共享资源为媒介的通信。

一个线程不要使用`Thread.stop`方法来让其他线程停止执行。更好的方法是让需要停止的线程B查看一个初始化值为`false`的布尔型变量，该变量可以被提出停止要求的线程A修改为`true`，从而使线程B可以对线程A的要求作出响应。

如果没有同步，则下列代码中的第一个while循环会被虚拟机转变为第二个循环的形式，此时就可能导致死循环，因为系统不知道后台监督该标志的线程什么时候能看到对该标志作出的修改。此时该标志产生了隐身错误。

		//第一个while循环
		while (!stopRequest)
			i++;

		//第二个循环
		if (!stopRequest)
			while (true)
				i++;

一个解决方法是同步对`stopRequest`的访问

		public class StopThread {
			private static boolean stopRequested;

			private static synchronized void requestStop() {
				stopRequest = true;
			}

			private static synchronized boolean stopRequested() {
				return stopRequested;
			}

			public static void main(String[] args) {
				Thread backgroundThread = new Thread(() -> {
					int i = 0;
					while (!stopRequested)
						i++;
				});
				backgroundThread.start();

				TimeUnit.SECOND.sleep(1);
				requestStop();
			}
		}

正确有效的同步应该是对变量的读，写操作都同步。只同步一个操作无法起到完全的同步作用。

使用`volatile`修饰变量虽然无法起到让访问它的方法互斥的作用，但是可以保证所有读取它的操作都能看到最后一次修改所写入的值。

`++`运算符的操作不是原子性的，它分为两步，第一步读取数据，第二步修改并写入数据。所以如果有一个线程在这两步操作之间访问了它所操作的变量，就会导致安全错误。

一个解决方法是使用`synchronized`关键字来修饰更改该变量值的方法的声明，并取消使用`volatile`关键字修饰该变量。
第二种方法是使用`java.util.concurrent.atomic`包中的`AtomicLong`类。这个包提供了一些基本数据类型用于单个变量的不需要锁的线程安全的编程。该包不仅像`volatile`一样提供了同步的通信功能，也提供了同步所需的原子性操作。

最佳的避免同步可能出现的问题的方法是根本不要共享可变数据。换句话说，就是可变数据应该只被单线程使用。如果采用了这个策略，则应该在文档中说明，作为将来的参考。

如果一个线程在一段时间内修改了数据对象，然后再将其共享给其他线程，那么哪怕只同步了这个共享步骤，也是可以接受的。而且如果该数据对象再不会被更改，那么其他线程就可以不需要同步对他的访问。这个对象也被认为是可以当做不可变对象使用。把这种对象从一个线程传递到另一个线程的过程称之为对该变量的安全发布。安全发布有很多种形式：

- 在类的初始化阶段使用静态成员变量存储该对象
- 使用`volatile`成员变量，最终成员变量，以及需要通过正常锁才能访问的成员变量来存储该对象
- 将该对象存储在一个并发集合中

当有多个线程分享可变数据时，每个线程对该数据的读和写都应该要实现同步。否则就会导致该数据产生隐身错误和安全错误。如果仅需要使用该数据作为线程之间通信的标志，而不需要操作互斥，则可以使用`volatile`关键字来实现片面的同步。

### 79. 避免过度的同步 ###

永远不要在同步方法或代码块中把控制权交给用户。换句话说，就是不能在同步的方法或代码块中调用被设计为用来重写的方法，或者由用户来提供的功能对象。

当系统已有的函数式接口的名称或者其方法名不能准确地表达我们所需要的功能时，可以考虑自定义一个函数式接口，该接口可以包装一个标准的函数式接口，如将其作为方法的参数，来实现方法的回调。

如果有一个线程正在遍历某个列表，那么另外一个线程就不能对该列表做出更改，包括增加，删除和修改元素的值。

Java7以后，可以在`catch`语句中捕捉多个异常，以`|`隔开

把对界外方法（alien methods）的调用放到同步代码块之外。比如在同步代码块中为共享数据建立一个快照，然后就可以在代码块的外部使用这个快照来进行相关操作而不需要同步该数据了。除此之外，一个更好的解决方法是使用并发集合，如`CopyOnwriteArrayList`类，该类是`ArrayList`类的变体，该类所有涉及更改操作的方法都是在内部创建一个底层数组的新鲜复本并在此复本基础上进行操作的。由于这个内部数组从来都不会改变，因此遍历它不需要加锁，速度也很快。虽然这个类的性能有些低，但非常适合用作观察者列表，因为该列表很少会被改动但却需要频繁地遍历。在同步代码块外部被调用的界外方法被称为开放调用（open call）。

应该在同步域内安排尽量少的工作。如果某个工作比较耗费时间，就应该找一个方法将其移出同步域内。

在编写可变类时有2个选择：

- 完全忽略同步，让用户根据需要从外部控制同步
- 在内部实现同步，从而让该类做到线程安全

通常只有在内部实现同步会比从外部通过锁住整个对象来实现同步带来非常大的并发性能的提升时，才考虑第二种方案。如果不确定，就采取第一种方案，并在文档中注明该类不是线程安全的。

如果一个方法修改了某个静态成员变量，并且该方法还有可能被多个线程同时调用，那么就必须在内部同步对该成员变量的访问。

### 80. 倾向于用executors， tasks， streams来替代threads ###

`java.util.concurrent`包中包含了一个`Executor Framework`，这是一个灵活的基于接口的任务执行机制。以下代码使用该机制运行和结束任务：

		//创建一个执行任务的服务
		ExecutorService exec = Executors.newSingleThreadExecutor();
		//使用服务运行一个任务
		exec.execute(runnable);
		//关闭服务
		exec.shutdown();

这个服务机制还有很多其他功能，比如：

- 使用`get`方法来等待特定任务完成；
- 使用`invokeAny`或`invokeAll`方法来等待一个集合内的任意一个或全部方法都结束；
- 使用`awaitTermination`方法来等待关闭一个执行服务；
- 使用`ExecutorCompletionService`来在任务一个一个结束后获取任务的执行结果；
- 使用`ScheduledThreadPoolExecutor`来安排任务在特定时间或固定间隔之间运行。

`java.util.concurrent.Executors`类提供了不同的静态工厂方法来创建不同的执行服务。如果需要多个线程来处理请求队列中的请求时，可以使用其中的一个静态工厂方法来创建一个称之为线程池（thread pool）的执行服务。如果需要定制自己的线程池对象，可以直接使用`ThreadPoolExecutor`类来创建线程池对象。

对于小程序或者轻负载服务器，可以`Executors.newCachedThreadPool`方法来创建一个简单的带缓冲功能的线程池对象。但这个对象并不适用于高负载服务器，因为该线程池内，提交的任务不会被列队，而是立刻被一个线程执行。如果没有可用的线程，则会创建一个新的线程。高负载服务器最好使用`Executors.newFixedThreadPool`类来创建一个容量固定的线程池，或者直接使用`ThreadPoolExecutor`类来实现对该线程池的控制最大化。

不仅应该避免编写自己的工作队列，通常也应该避免直接操作线程。当直接操作线程时，一个`Thread`将作为一个工作单位和执行该工作单位的机制来使用。

而在执行者框架中，工作单位和执行机制是分开的。

- 工作单位（a unit of work）的最关键的抽象是任务（task）。有两种类型的任务：`Runnable`和`Callable`（和`Runnable`类似，但它有返回值并可能抛出异常）。
- 执行任务的执行机制通常是执行者服务（executor service）。

分支-合并任务（fork-join taskhs）可以被分解成更小的子任务，由`ForkJoinTask`类的对象代表并由被称为分支-合并池（fork-join pool）的服务来执行。该服务所使用的线程会在完成自己的分配任务后，从其他线程（同一服务控制的线程）“窃取”任务，让所有的线程都处于忙碌状态，从而实现高CPU使用率，低延迟。并发流就是基于分支-合并池编写的。

### 81. 倾向使用并发工具来代替`wait`和`notify`方法 ###

在`java.util.concurrent`包中的高级并发工具分为三大类：

- 执行者框架
- 并发集合
- 同步器

并发集合是普通集合接口的高并发性能实现，因为自带内部并发管理，所以不可能从并发集合中排除并发活动，对它们加锁只会减慢程序。

因为不能排除并发集合上的并发活动，所以也不能以原子方式组合对它们的方法调用。也就是说，每个方法的调用都可能引发该集合的并发活动，所以将多个基于该集合的方法调用通过原子方式组合是无法达到排除并发活动而实现线程安全的目的的。因此，并发集合接口配备了依赖于状态的修改操作，这些操作将几个原语组合成单个原子操作。

		// Concurrent canonicalizing map atop ConcurrentMap - not optimal
		private static final ConcurrentMap<String, String> map = new ConcurrentHashMap<>();

		public static String intern(String s) {
			String previousValue = map.putIfAbsent(s, s);
			return previousValue == null ? s : previousValue;
		}

		//ConcurrentHashMap针对检索操作进行了优化，例如get。 因此，如果get表明有必要，最初只需调用get并调用putIfAbsent
		// Concurrent canonicalizing map atop ConcurrentMap - faster!
		public static String intern(String s) {
			String result = map.get(s);
			if (result == null) {
				result = map.putIfAbsent(s, s);
				if (result == null)
					result = s;
			} 
			return result;
		}
	
使用`ConcurrentHashMap`优先于`Collections.synchronizedMap`。

一些集合接口使用阻塞操作进行扩展，这些操作等待（或阻塞）直到可以成功执行。例如适用于工作队列（也被称为生产者-消费者队列）并被大多数`ExecutorService`实现所使用的`BlockingQueue`接口。该接口中的`take`默认方法从队列中删除并返回head元素，如果队列为空则等待。

同步器是使线程能够彼此等待的对象，允许它们协调它们的活动。最常用的同步器是`CountDownLatch`和`Semaphore`。不太常用的是`CyclicBarrier` 和`Exchanger`。最强大的同步器是`Phaser`。

`CountDownLatch`是一次性使用的屏障，允许一个或多个线程等待一个或多个其他线程执行某些操作。`CountDownLatch`的唯一构造函数接受一个`int`，它是在允许所有等待的线程继续之前必须在`CountDownLatch`对象上调用`countDown`方法的次数。

		// Simple framework for timing concurrent execution
		public static long time(Executor executor, int concurrency,
									Runnable action) throws InterruptedException {
			CountDownLatch ready = new CountDownLatch(concurrency);
			CountDownLatch start = new CountDownLatch(1);
			CountDownLatch done = new CountDownLatch(concurrency);
			
			for (int i = 0; i < concurrency; i++) {
				executor.execute(() -> {
					ready.countDown(); // 告诉计数器完成准备
					try {
						start.await(); // 等待所有的工作准备就绪
						action.run();
					} catch (InterruptedException e) {
						Thread.currentThread().interrupt();
					} finally {
						done.countDown(); // 告诉计数器工作完成
					} 
				});
			}
			
			ready.await(); // 等待所有的工作准备就绪
			long startNanos = System.nanoTime();
			start.countDown(); // 开始工作
			done.await(); // 等待所有工作完成
			return System.nanoTime() - startNanos;
		}

注意传递给`time`方法的执行者必须允许创建至少与给定并发级别一样多的线程，否则测试将永远不会完成。

对于间隔计时，请始终使用`System.nanoTime`而不是`System.currentTimeMillis`。`System.nanoTime`更准确，更精确，不受系统实时时钟调整的影响。

始终使用wait循环模式来调用`wait`方法；永远不要在循环之外调用它。循环用于测试等待前后的状况。

		// 通常使用wait方法的模板
		synchronized (obj) {
			while (<condition does not hold>)
				obj.wait(); // 释放锁并在被唤醒时再次获取他
			... // 根据条件执行相应的操作
		}

在等待之前测试条件并在条件满足的情况下跳过等待。如果条件已经存在并且在线程等待之前已经调用了`notify`（或`notifyAll`）方法，则无法保证线程将从等待中唤醒。

在等待后再次测试条件并在条件不满足的时候再次等待是确保安全必须要做的事情。

当条件不成立时，线程可能会被唤醒的原因有多种：

- 另一个线程可以获得锁并在线程调用`notify`和等待线程醒来之间改变了保护状态。也就是说通知线程没有改变条件的状态，但是在它调用`notify`方法前，另一个线程取得锁并改变了条件，再将锁还给通知线程让他继续执行`notify`方法来唤醒等待线程。
- 当条件不成立时，另一个线程可能会意外或恶意地调用`notify`。在公共可访问对象的同步方法中中的任何`wait`方法都容易受到此问题的影响。
- 通知线程在唤醒等待线程时可能过于“慷慨”。例如，即使只有一些等待的线程满足条件，通知线程也可以调用`notifyAll`。
- 等待线程可以（很少）在没有通知的情况下唤醒。这被称为虚假唤醒。

总是使用`notifyAll`方法来代替`notify`方法。在新代码中尽量不要使用`wait`和`notify`方法。

### 82. 在文档中注明线程安全性 ###

方法声明中的`synchronized`关键字是实现细节，而不是API的一部分。

要启用安全的并发使用，类必须清楚地记录它支持的线程安全级别。常见的等级划分为：

- Immutable 不变的：此类的实例显示为常量。无需外部同步。包括`String`，`Long`和`BigInteger`（参阅17）。
- Unconditionally thread-safe 无条件线程安全：此类的实例是可变的，但该类具有足够的内部同步，以便可以同时使用其实例而无需任何外部同步。包括`AtomicLong`和`ConcurrentHashMap`。
- Conditionally thread-safe 有条件线程安全：像无条件线程安全一样，除了某些方法需要外部同步以便安全地并发使用。包括`Collections.synchronized`包装器返回的集合，其迭代器需要外部同步。
- Not thread-safe 不是线程安全：这个类的实例是可变的。要同时使用它们，客户端必须使用客户端选择的外部同步来包围每个方法调用（或调用序列）。包括通用集合实现，例如`ArrayList`和`HashMap`。
- Thread-hostile 线程敌对：即使每个方法调用都被外部同步包围，此类对于并发使用也是不安全的。线程敌意通常是在没有同步的情况下修改静态数据。

记录有条件的线程安全类需要小心。必须指明哪些调用序列需要外部同步，以及必须获取哪个锁（或在极少数情况下，多个锁）才能执行这些序列。通常它是实例本身的锁，但也有例外。例如，`Collections.synchronizedMap`的文档说明了这一点：

		当迭代任何返回的map对象的集合视图时，用户必须手动同步该map对象。
		Map<K, V> m = Collections.synchronizedMap(new HashMap<>());
		Set<K> s = m.keySet(); // 不需要放在同步代码块中
		...
		synchronized(m) { // 同步m，而不是s
			for (K key : s)
				key.f();
		}

类的线程安全性的描述通常属于类的doc注释，但具有特殊线程安全属性的方法应在其自己的文档注释中描述这些属性。没有必要记录枚举类型的不变性。除非从返回类型中显而易见，否则静态工厂必须记录返回对象的线程安全性，如`Collections.synchronizedMap`（上文）所示。

当类承诺使用可公开访问的锁时，它允许客户端以原子方式执行一系列方法调用，但它与并发集合（如`ConcurrentHashMap`）使用的高性能内部并发控制不兼容。此外，客户端可以通过长时间保持可公开访问的锁来启动拒绝服务攻击。

要防止此类拒绝服务攻击，可以使用私有锁对象而不是使用`synchronized`方法（这意味着可公开访问的锁）：

		// Private lock object idiom - thwarts denial-of-service attack
		private final Object lock = new Object();

		public void foo() {
			synchronized(lock) {
				...
			}
		}

锁应始终声明为final。无论使用的是普通的监视器锁（如上所示）还是使用`java.util.concurrent.locks`包中的锁，都是如此。

私有锁对象类的这个习惯用法只能用于无条件的线程安全类。有条件的线程安全类不能使用这个习惯用法，因为它们必须在文档中说明在执行某些方法调用序列时客户端要获取的锁，从而暴露了实现细节。

使用私有锁对象特别适合用于被设计用来继承的类。如果这样的类要使用其实例作为锁，则其子类很容易在无意中干扰父类的操作，反之亦然。通过为不同的目的使用相同的锁，子类和父类可能最终互相干扰到彼此。

小结：每一个类应该清楚地以措辞谨慎的文字描述或线程安全有关的注解说明它的线程安全性。有条件的线程安全类必须记录哪些方法调用序列需要外部同步，在执行这些序列时需要获取哪些锁。如果编写无条件的线程安全类，请考虑使用私有锁对象代替同步方法。这可以保护您免受客户端和子类的同步干扰。

### 83. 谨慎使用延迟初始化 ###

在存在多个线程的情况下，延迟初始化会很麻烦。如果两个或多个线程共享一个延迟初始化的字段，则必须采用某种形式的同步，否则可能导致严重的错误（参阅78）。

在大多数情况下，常规初始化优于延迟初始化。以下是常规初始化的实例字段的典型声明。注意使用`final`修饰符（参阅17）：

		// 常规初始化
		private final FieldType field = computeFieldValue();

如果使用延迟初始化是为了破坏初始化环路（类A的构造器中创建了类B的实例，类B的构造器中创建了类A的实例），则应该使用同步的访问器访问该变量，因为它是最简单，最清晰的替代方法：

		// Lazy initialization of instance field - synchronized accessor
		private FieldType field;

		private synchronized FieldType getField() {
			if (field == null)
				field = computeFieldValue();
			return field;
		}

如果需要在静态成员变量上使用延迟初始化来提高性能，请使用下面这个惯用方法-延迟初始化持有者类（lazy initialization holder class）：

		// Lazy initialization holder class idiom for static fields
		private static class FieldHolder {
			static final FieldType field = computeFieldValue();
		}

		private static FieldType getField() { return FieldHolder.field; }

当第一次调用`getField`时，它首次读取`FieldHolder.field`时导致`FieldHolder`类的初始化。这个用法的优点在于`getField`方法不是同步的，只执行字段访问，因此延迟初始化几乎不会增加访问成本。

如果需要在实例变量上使用延迟初始化来提高性能，请使用双重检查法。这个方法避免了初始化后访问变量加锁的成本（参阅79）。该方法检查变量的值两次（因此称为双重检查）：第一次仅仅是查看变量是否被初始化，不会改变变量的状态，所以没有加锁。如果变量没有被初始化，则在第二次访问该变量前加锁，然后检查该字段此时是否被初始化，如果没有，则开始初始化该变量。因为变量初始化后就释放了锁，所以必须将该变量声明为`volatile`。

		// Double-check idiom for lazy initialization of instance fields
		private volatile FieldType field;

		private FieldType getField() {
			FieldType result = field;
			if (result == null) { // First check (no locking)
				synchronized(this) {
					if (field == null) // Second check (with locking)
						field = result = computeFieldValue();
				}
			}
			return result;
		}

需要反复初始化的实例变量可以省去第二次检查。

		// Single-check idiom - can cause repeated initialization!
		private volatile FieldType field;

		private FieldType getField() {
			FieldType result = field;
			if (result == null)
				field = result = computeFieldValue();
			return result;
		}

基本数据类型中数字类型的变量可以使用`0`作为未初始化的判断依据（需要注意故意初始化为0的情况）。

小结：优先使用常规化初始化方法。如果必须延迟初始化以满足性能需要或者打破初始化环路，则应该：

- 对于静态变量，使用初始化持有者类；
- 对于实例变量，使用双重检查方法；可以重复初始化的变量，可以考虑单一检查方法。

### 84. 不要依赖线程调度器 ###

任何依赖于线程调度程序以获得正确性或性能的程序都可能是不可移植的。

编写健壮、响应迅速的可移植程序的最佳方法是确保可运行线程的平均数量不会明显大于处理器数量。注意可运行线程的数量与线程总数不同，后者可能要高得多，因为包含了等待的线程。

保持可运行线程数量较少的主要方法是让每个线程做一些有用的工作，然后等待更多的工作。就Executor Framework而言（参阅80），这意味着适当地调整线程池的大小并保持任务简短但不会太短。

线程不应该忙于等待，即反复检查共享对象的状态是否改变。当程序因为有些线程无法获得相对于其他线程足够的CPU时间而几乎无法工作时，要抵制想调用`Thread.yield`来“修复”程序的诱惑。

通过设置线程的优先级来调度线程也是不可取的。因为也会导致移植的问题。

### 85. 优先使用Java序列化的替代方案 ###

在反序列化期间调用的执行潜在危险活动的方法称为小工具。可以一起使用多个小工具来形成小工具链。一个足够强大的小工具链允许攻击者在底层硬件上执行任意本机代码，只要有机会提交精心设计的字节流进行反序列化。

在不使用任何小工具的情况下，可以通过反序列化会导致需要很长时间完成反序列化的短流来轻松地发起拒绝服务攻击。这种流被称为反序列化炸弹。下面这个例子只使用了`Set`和字符串：

		// Deserialization bomb - deserializing this stream takes forever
		static byte[] bomb() {
			Set<Object> root = new HashSet<>();
			Set<Object> s1 = root;
			Set<Object> s2 = new HashSet<>();
			for (int i = 0; i < 100; i++) {
				Set<Object> t1 = new HashSet<>();
				Set<Object> t2 = new HashSet<>();
				t1.add("foo"); // Make t1 unequal to t2
				s1.add(t1); s1.add(t2);
				s2.add(t1); s2.add(t2);
				s1 = t1;
				s2 = t2;
			} 
			return serialize(root); // Method omitted for brevity
		}

对象图由201个`HashSet`实例组成，每个实例包含3个或更少的对象引用。整个流的长度只有5,744字节，但问题是反序列化`HashSet`实例需要计算其元素的哈希码。根哈希集的2个元素本身是包含2个哈希集元素的哈希集，每个哈希集元素包含2个哈希集元素，依此类推，深度为100个级别。因此，反序列化集会导致`hashCode`方法被调用超过2100次。

避免序列化漏洞利用的最佳方法是永远不要反序列化任何东西。没有理由在编写的任何新系统中使用Java序列化。应该使用其他的跨平台结构化数据表示。这些表示的共同之处在于它们比Java序列化简单得多。它们不支持任意对象图的自动序列化和反序列化。相反，它们支持由一组属性 - 值对组成的简单结构化数据对象。仅支持少数基本数据类型和数组数据类型。

领先的跨平台结构化数据表示是JSON和Protocol Buffers，也称为protobuf。 JSON由Douglas Crockford设计用于浏览器 - 服务器通信，并且协议缓冲器由Google设计用于在其服务器之间存储和交换结构化数据。即使这些表示有时被称为语言中性，但JSON最初是为JavaScript开发的，而protobuf是为C++开发的。

JSON和protobuf之间最显着的区别是JSON是基于文本的，人类可读的，而protobuf是二元的，效率更高; 并且JSON完全是数据表示，而protobuf提供模式schema（类型type）来记录和加强适当的用法。尽管protobuf比JSON更有效，但JSON对于基于文本的表示非常有效。虽然protobuf是二进制表示，但它提供了一种替代文本表示，用于需要人类可读性的用途（pbtxt）。

如果无法完全避免Java序列化，那么下一个最佳选择是永远不会反序列化不受信任的数据。特别是永远不应接受来自不受信任来源的RMI流。

如果无法避免序列化，并且不确定需要反序列化的数据的安全性，则应该使用Java 9中添加的对象反序列化过滤并向后移植到早期版本（java.io.ObjectInputFilter）。此工具允许指定在反序列化之前应用于数据流的过滤器。它以类为基本单位来进行控制，允许接受或拒绝某些类。默认接受类并拒绝潜在危险类列表称为黑名单；默认情况下拒绝类并接受假定安全的列表称为白名单。首选使用白名单而不是黑名单。`Serial Whitelist Application Trainer（SWAT）`的工具可用于为应用程序自动准备白名单。

小结：如果是从头开始设计系统，使用跨平台的结构化数据表示，例如JSON或protobuf。不要反序列化不受信任的数据。如果必须这样做，使用对象反序列化过滤，但这也不能保证阻止所有攻击。

### 86. 实现`Serializable`接口时加倍小心 ###

实现`Serializable`的一个主要成本是它降低了发布后再更改类的实现的灵活性。当类实现`Serializable`时，其字节流编码（或序列化格式）将成为其导出API的一部分。在广泛分发类之后，通常需要永久支持序列化格式，就像需要支持导出的API的所有其他部分一样。如果没有努力设计自定义序列化格式而接受了默认格式，则该类的序列化格式将永远与其原始内部表示相关联。换句话说，如果接受了默认的序列化格式，则该类的私有和包私有实例变量将成为其导出API的一部分，并且最小化对变量的访问（参阅15）的做法将失去其作为信息隐藏工具的有效性。

如果接受了默认的序列化格式并在稍后更改了类的内部表示，则将导致序列化格式的不兼容。尝试使用旧版本的类序列化实例并使用新版本对其进行反序列化（反之亦然）的客户将遇到程序失败。可以在保持原始序列化形式（使用`ObjectOutputStream.putFields`和`ObjectInputStream.readFields`）的同时更改内部表示，但这可能很困难并且在源代码中留下可见的瑕疵。如果选择将类序列化，应该仔细设计一个能长期使用的高质量序列化格式（参阅87，90）。

在类的进化过程中，一个由序列化带来的限制是关于流的唯一标识符（stream unique identifiers），通常称为系列版本UID（serial version UID）。每个可序列化的类都有一个与之关联的唯一标识号。如果未通过声明名为`serialVersionUID`的`long`类型的静态最终变量来指定此数字，则系统会在运行时通过将加密哈希函数（SHA-1）应用于类的结构来自动生成它。此值受类的名称，其所实现的接口及其大多数成员（包括编译器生成的合成成员）的影响。如果更改任何这些内容，例如，添加便捷方法，自动生成的系列版本UID会更改。如果声明串行版本UID失败，则兼容性将被破坏，从而导致运行时出现`InvalidClassException`。

实现`Serializable`的第二个成本是它增加了错误和安全漏洞的可能性（参阅85）。通常，对象是通过构造器来创建；但序列化也是一种创建对象的语言额外机制。无论是接受默认行为还是覆盖默认行为，反序列化都是一个“隐藏的构造函数”。 因为没有与反序列化相关联的显式构造函数，所以很容易忘记必须确保它保证构造函数建立的所有不变量，并且它不允许攻击者访问构造中的对象的内部。

实现`Serializable`的第三个成本是它增加了与发布新版本类相关的测试负担。修改可序列化类时，重要的是检查是否可以序列化新版本中的实例并在旧版本中反序列化，反之亦然。必须确保序列化 - 反序列化过程成功并且它会产生一个原始对象的忠实副本。

为继承而设计的类（参阅19）应该很少实现`Serializable`，接口也应该很少扩展它。

专为实现`Serializable`的继承而设计的类包括`Throwable`和`Component`。`Throwable`实现`Serializable`，因此RMI可以从服务器向客户端发送异常。`Component`实现`Serializable`，因此可以发送，保存和恢复GUI。

如果在实现一个可同时序列化及扩展具有实例变量的类，则需要注意：如果有任何实例变量存储的是不变量，则必须防止子类覆盖`finalize`方法，这可以通过在该类中重写`finalize`并将其声明为`final`来完成。如果类的实例变量被初始化为默认值（整数类型为零，布尔值为`false`，引用类型为`null`），则它的不变量可能会需要重新赋值（延迟初始化），此时必须添加此`readObjectNoData`方法：

		// readObjectNoData for stateful extendable serializable classes
		private void readObjectNoData() throws InvalidObjectException {
			throw new InvalidObjectException("Stream data required");
		}

如果为继承而设计的类不可序列化，则要求该类具有可访问的无参数构造器从而实现它的子类的正常反序列化。如果没有提供这样的构造器，则子类必须使用序列化代理模式（参阅90）。

内部类（参阅24）不应实现`Serializable`。但是静态内部类可以实现`Serializable`。

### 87. 考虑使用自定义的序列化格式 ###

不要在没有事先考虑是否合适的情况下使用默认的序列化格式。

对象的默认序列化格式是对以该对象为起点的对象图的物理表示进行合理有效编码。它描述了此对象中以及可从此对象访问的每个对象中包含的数据。它还描述了所有这些对象相互链接的拓扑。对象的理想序列化格式仅应该包含对象表示的逻辑数据，独立于物理表示。

如果对象的物理表示与其逻辑内容相同，则默认的序列化格式可能是合适的。例如，对于以下类，默认的序列化形式是合理的，这简单地表示一个人的姓名：

		// Good candidate for default serialized form
		public class Name implements Serializable {
		/**
		* Last name. Must be non-null.
		* @serial
		*/
		private final String lastName;
		/**
		* First name. Must be non-null.
		* @serial
		*/
		private final String firstName;
		/**
		* Middle name, or null if there is none.
		* @serial
		*/
		private final String middleName;
			... // Remainder omitted
		}

即使确定默认的序列化格式是合适的，通常也必须提供`readObject`方法以确保不变量和安全性。对于`Name`类，`readObject`方法必须确保字段`lastName`和`firstName`为非空值。

注意，对`lastName`，`firstName`和`middleName`变量有文档注释，因此即使这些变量是私有的，但它们被用于定义一个公共API，即该类的序列化格式，而公共API必须被文档说明。`@serial`注解的存在告诉Javadoc将此文档放在一个记录序列化格式的特殊页面上。这条规则也适用于私有方法。

当对象的物理表示与其逻辑数据内容明显不同时，使用默认的序列化格式有四个缺点：

- 它将导出的API永久绑定到当前内部表示。 
- 它会消耗过多的空间。
- 它可能会消耗过多的时间。
- 它可能导致堆栈溢出。

`transient`修饰符指示要从类的默认序列化格式中省略该实例变量：

`writeObject`方法首先应该调用`defaultWriteObject`方法，而`readObject`做的第一件事就是调用`defaultReadObject`。如果所有类的实例变量都是`transient`，可以不用调用`defaultWriteObject`和`defaultReadObject`，但通常都会调用它们，因为可以方便在以后的版本中添加非`transient`实例变量，并同时保持向后和向前兼容性。如果实例在更高版本中序列化并在早期版本中反序列化，则添加的变量将被忽略。如果早期版本的`readobject`方法无法调用`defaultReadObject`，则反序列化将因`StreamCorruptedException`而失败。

如果某个类的不变量与具体的实现相关联，则其不变量值可能在序列化-反序列化过程中被破坏。例如，哈希表的物理表示是包含键值条目的一系列桶。条目所在的桶是基于其密钥的哈希码的函数。通常，它不保证不同实现所产生的值是相同的。实际上，相同实现，不同的运行都不能保证相同。因此，接受哈希表的默认序列化格式将构成严重错误。序列化和反序列化哈希表可能会产生一个不变量严重损坏的对象。

无论是否接受默认的序列化格式，当调用`defaultWriteObject`方法时，每个未标记为`transient`的实例变量都将被序列化。因此，每个可以声明为`transient`的实例变量都应该声明（包括派生字段，其值可以从主数据字段计算，例如缓存的哈希值）。要确认每一个非transient变量的值都必须是对象逻辑状态的一部分。

如果使用了默认序列化格式并且有一个或多个变量标记为`transient`，那么在反序列化时，这些变量都会被初始化为其默认值：对象引用字段为`null`，数字基本字段为零，布尔字段为`false`。如果需要在反序列化时为他们提供非默认值，则必须提供一个`readObject`方法，该方法调用`defaultReadObject`方法，然后将transient变量设定为想要的值（参阅88）。或者，这些变量可以在第一次使用时进行延迟初始化（参阅83）。

无论是否使用默认的序列化格式，都必须像同步其他读取对象完整状态的方法一样强制同步对象的序列化过程。因此，如果有一个线程安全的对象（参阅82）通过同步每个方法来实现其线程安全，并且选择了使用默认序列化格式，应该使用以下`writeObject`方法：

		// writeObject for synchronized class with default serialized form
		private synchronized void writeObject(ObjectOutputStream s)
						throws IOException {
			s.defaultWriteObject();
		}

如果在`writeObject`方法中运用了同步，则必须确保它与其他活动使用了相同的锁定顺序约束，否则将面临资源排序死锁的风险。

无论选择哪种序列化格式，在编写的每个可序列化类中都应该显式声明系列版本UID，从而消除版本UID不兼容引发的问题（参阅86）。而且还可以避免系统自动生成版本UID带来的性能损耗。使用以下代码声明UID：

		private static final long serialVersionUID = randomLongValue;

如果是编写一个新类，则为`randomLongValue`选择的值无关紧要。可以通过在类上运行`serialver`工具方法来生成值，但也可以凭空挑选任一数字。版本UID可以不是唯一的。如果修改缺少自定义版本UID的现有类，并且希望新版本和现有的序列化实例兼容，则必须使用旧版本自动生成的UID。可以通过在类的旧版本上运行`serialver`工具方法来获取旧版本类的UID编号。

如果在新版本类中更改了版本UID的值，那么在尝试反序列化旧版本的实例时会抛出`InvalidClassException`。

### 88. 防御性编写`readObject`方法 ###

`readObject`方法实际上相当于另一个公共构造函数，因此需要与任何其他构造函数一样小心。正如构造函数必须检查其参数的有效性（参阅49）并在适当的地方制作参数的防御性复本（参阅50），`readObject`方法也必须如此做。

笼统地讲，`readObject`是一个使用字节流作为唯一参数的构造函数。在正常使用中，字节流是通过序列化正常构造的实例生成的。但如果该字节流被人工构造以生成破坏该类不变量的对象时，就可能创建一个正常情况下不可能生成的对象。

要解决此问题，在`readObject`方法中调用`defaultReadObject`，然后检查反序列化对象的有效性。如果有效性检查失败，则`readObject`方法将抛出`InvalidObjectException`，从而阻止反序列化完成：

		// readObject method with validity checking - insufficient!
		private void readObject(ObjectInputStream s)
					throws IOException, ClassNotFoundException {
			s.defaultReadObject();
			// Check that our invariants are satisfied
			if (start.compareTo(end) > 0)
				throw new InvalidObjectException(start +" after "+ end);
		}

虽然这可以防止攻击者创建无效的实例，但仍然存在潜在的问题。例如，可以通过构造以有效实例开头的字节流来创建可变实例，然后在这个可变实例中提供和所包含有效实例中的私有变量相同类型的公共变量，再通过修改字节流中这些公共变量对应的字节码，使它们指向私有变量的引用，从而将这些引用指向的对象通过这些额外的公共变量暴露出来。如下所示：

		public class MutablePeriod {
			// A period instance
			public final Period period;
			// period's start field, to which we shouldn't have access
			public final Date start;
			// period's end field, to which we shouldn't have access
			public final Date end;
			
			public MutablePeriod() {
				try {
					ByteArrayOutputStream bos = new ByteArrayOutputStream();
					ObjectOutputStream out = new ObjectOutputStream(bos);
					// Serialize a valid Period instance
					out.writeObject(new Period(new Date(), new Date()));
					/*
					* Append rogue "previous object refs" for internal
					* Date fields in Period. For details, see "Java
					* Object Serialization Specification," Section 6.4.
					*/
					byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // Ref #5
					bos.write(ref); // The start field
					ref[4] = 4; // Ref # 4
					bos.write(ref); // The end field
					// Deserialize Period and "stolen" Date references
					ObjectInputStream in = new ObjectInputStream( 
									new ByteArrayInputStream(bos.toByteArray()));
					period = (Period) in.readObject();
					start = (Date) in.readObject();
					end = (Date) in.readObject();
				} catch (IOException | ClassNotFoundException e) {
					throw new AssertionError(e);
				}
			}
		}

运行以下程序发动攻击：

		public static void main(String[] args) {
			MutablePeriod mp = new MutablePeriod();
			Period p = mp.period;
			Date pEnd = mp.end;
			// Let's turn back the clock
			pEnd.setYear(78);
			System.out.println(p);
			// Bring back the 60s!
			pEnd.setYear(69);
			System.out.println(p);
		}

问题的根源是`Period`的`readObject`方法没有做足够的防御性复制。对象反序列化时，必须防御性地复制客户端不得拥有的引用类型的变量。因此，每个包含私有可变组件的可序列化不可变类必须在其`readObject`方法中防御性地复制这些组件。以下`readObject`方法足以确保`Period`的不变量并保持其不变性：

		// readObject method with defensive copying and validity checking
		private void readObject(ObjectInputStream s)
						throws IOException, ClassNotFoundException {
			s.defaultReadObject();
			// Defensively copy our mutable components
			start = new Date(start.getTime());
			end = new Date(end.getTime());
			// Check that our invariants are satisfied
			if (start.compareTo(end) > 0)
				throw new InvalidObjectException(start +" after "+ end);
		}

注意复制防御性复本应在有效性检查之前执行，另外最终变量无法进行防御性复制。因此，在上例中，要使用`readObject`方法，必须使`start`和`end`变量为非最终变量。

一个简单的用于判断默认的`readObject`方法是否适用于某个类的方法：是否愿意添加一个公共构造函数，该构造函数将对象中每个非transient变量的值作为参数，然后将值存储在变量中而不进行任何验证。如果没有，则必须提供`readObject`方法，并且必须执行构造函数所需的所有有效性检查和防御性复制。或者，可以使用序列化代理模式（参阅90）。

`readObject`方法和构造函数都适用于非最终可序列化类。与构造函数一样，`readObject`方法中不能直接或间接调用可覆盖的方法（参阅19）。如果违反此规则，重写了某个方法，则重写的方法会在子类的状态被反序列化之前运行，进而导致程序失败。

小结：编写`readObject`方法时要采取编写公共构造函数时一样的思路：不管给出了什么字节流，它都必须生成有效的实例。同时不要假设所给字节流一定是合法的序列化实例。以下是编写`readObject`方法的一些指南：

- 对于具有必须保持私有的引用类型变量的类，防御性地复制此变量中的每个对象。 如不可变类的可变组件。
- 检查所有不变量，并在检查失败时抛出`InvalidObjectException`。 检查应紧跟防御性复制。
- 如果在反序列化后必须验证整个对象图，请使用`ObjectInputValidation`接口（本书未讨论）。
- 不要直接或间接调用类中的任何可被覆盖的方法。

### 89. 使用枚举类型代替`readResolve`实现对生成实例数量的控制 ###

任何`readObject`方法，无论是显式方法还是默认方法，都会返回一个新创建的实例，该实例与在类初始化时创建的实例不同，因此在序列化单例类时要小心。

`readResolve`功能允许将另一个实例替换为`readObject`创建的实例。如果被反序列化的对象的类使用适当的声明定义了`readResolve`方法，则反序列化后新创建的对象会调用此方法。然后用此方法返回的对象来代替这个新创建的对象。在大多数情况中，不会保留对新创建的对象的引用，因此它立即可被垃圾回收。

如果`Elvis`类用于实现`Serializable`，则以下`readResolve`方法足以保证它的单例属性：

		// readResolve for instance control - you can do better!
		private Object readResolve() {
			// Return the one true Elvis and let the garbage collector
			// take care of the Elvis impersonator.
			return INSTANCE;
		}

此方法忽略了反序列化时生成的对象，直接返回在初始化类时创建的唯一`Elvis`实例。因此，`Elvis`类的序列化格式不需要包含任何实际数据，可以把所有实例字段声明为`transient`。实际上，如果依赖`readResolve`进行实例控制，则所有引用类型的实例变量都应该声明为`transient`。否则，攻击者有可能在反序列化对象运行`readResolve`方法之前使用类似于第88项中的`MutablePeriod`的攻击技术。

以下是它详细的作案流程：首先，编写一个stealer类，它同时具有`readResolve`方法和一个实例变量，该变量指向窃取类所寄生的序列化的单例类。然后修改要反序列化的字节流，将单例类的非transient变量替换为窃取类的实例。现在出现了一个环路：单例类包含窃取类，窃取类中也有变量指向该单例类。

因为单例类包含窃取类，所以当单例类被反序列化时，窃取类的`readResolve`方法首先运行。而当窃取类的`readResolve`方法运行时，其实例变量仍指向那个已完成部分反序列化但还没有换身的单例类实例。

窃取类的`readResolve`方法将引用从这个实例变量复制到一个静态变量，以在`readResolve`方法运行后能通过该静态变量访问该单例类实例。然后，该方法返回一个和在单例类中它所替换的那个非transient变量相同类型的值。如果不这样做，当序列化系统尝试将窃取者类的引用存储到该变量时，两者的类型不一样，VM将抛出`ClassCastException`。换句话说，通过修改字节码流将单例类中的非transient变量的值强制改为窃取类的实例，再通过窃取类将单例类反序列化生成的实例半成品提取出来做修改，然后使用该窃取类的`readResolve`方法返回正确的类型，从而造成运行正常的假象。最后，单例类会执行自己的`readResolve`方法，但此时窃取类已经对单例类的反序列化对象作出了修改，最终得到的对象与实际对象可能是完全不同的。

		// Broken singleton - has nontransient object reference field!
		public class Elvis implements Serializable {
			public static final Elvis INSTANCE = new Elvis();
			private Elvis() { }
			private String[] favoriteSongs =
						{ "Hound Dog", "Heartbreak Hotel" };
			public void printFavorites() {
				System.out.println(Arrays.toString(favoriteSongs));
			}
			private Object readResolve() {
				return INSTANCE;
			}
		}

下面是一个窃取类：

		public class ElvisStealer implements Serializable {
			static Elvis impersonator;
			private Elvis payload;
			
			private Object readResolve() {
				// Save a reference to the "unresolved" Elvis instance
				impersonator = payload;
				// Return object of correct type for favoriteSongs field
				return new String[] { "A Fool Such as I" };
			}
			
			private static final long serialVersionUID = 0;
		}

最后这个程序将手工更改需要反序列化的流，以生成有缺陷的单例的两个不同实例。

		public class ElvisImpersonator {
			// Byte stream couldn't have come from a real Elvis instance!
			private static final byte[] serializedForm = {
			(byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
			0x45, 0x6c, 0x76, 0x69, 0x73, (byte)0x84, (byte)0xe6,
			(byte)0x93, 0x33, (byte)0xc3, (byte)0xf4, (byte)0x8b,
			0x32, 0x02, 0x00, 0x01, 0x4c, 0x00, 0x0d, 0x66, 0x61, 0x76,
			0x6f, 0x72, 0x69, 0x74, 0x65, 0x53, 0x6f, 0x6e, 0x67, 0x73,
			0x74, 0x00, 0x12, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x6c,
			0x61, 0x6e, 0x67, 0x2f, 0x4f, 0x62, 0x6a, 0x65, 0x63, 0x74,
			0x3b, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0c, 0x45, 0x6c, 0x76,
			0x69, 0x73, 0x53, 0x74, 0x65, 0x61, 0x6c, 0x65, 0x72, 0x00,
			0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x01,
			0x4c, 0x00, 0x07, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64,
			0x74, 0x00, 0x07, 0x4c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x3b,
			0x78, 0x70, 0x71, 0x00, 0x7e, 0x00, 0x02
			};
			public static void main(String[] args) {
				// Initializes ElvisStealer.impersonator and returns
				// the real Elvis (which is Elvis.INSTANCE)
				Elvis elvis = (Elvis) deserialize(serializedForm);
				Elvis impersonator = ElvisStealer.impersonator;
				elvis.printFavorites();
				impersonator.printFavorites();
			}
		}

最终输出结果如下，证明了可以创建两个不同的`Elvis`实例

		[Hound Dog, Heartbreak Hotel] [A Fool Such as I]

可以通过声明`favoriteSongs`变量为transient来解决这个问题，但最好是让`Elvis`成为只有一个值的枚举类型来修复它（参阅3）。

如果将需要控制实例数量的可序列化类编写为枚举类型，Java会保证除了声明的枚举值之外不会有其他实例。以下是`Elvis`类作为枚举类型的示例： 

		// Enum singleton - the preferred approach
		public enum Elvis {
			INSTANCE;
			private String[] favoriteSongs =
					{ "Hound Dog", "Heartbreak Hotel" };
			
			public void printFavorites() {
				System.out.println(Arrays.toString(favoriteSongs));
			}
		}

如果一个需要控制实例数量的可序列化类的实例在编译时是未知的，那么就无法将该类表示为枚举类型，只能使用`readResolve`方法。

`readResolve`的可访问性非常重要。如果在最终类上放置readResolve方法，它应该是私有的。如果将`readResolve`方法放在非最终类上，则不同的访问性会产生不同效果：

- 如果它是私有的，则不适用于任何子类。
- 如果它是包私有的，它将仅适用于同一包中的子类。
- 如果它是受保护的或公开的，它将适用于所有不覆盖它的子类。但反序列化子类实例将生成一个父类实例，这可能会导致`ClassCastException`。

小结：尽量使用枚举类型来控制实例的生成。如果这个需要控制实例生成以及可序列化的类的实例不能在编译期间确定，则使用`readResolve`方法并确定类的所有实例变量都是基本数据类型或transient的。

### 90. 考虑使用序列化代理代替序列化的实例 ###

序列化代理模式：首先，设计一个私有静态内部类，它简洁地代表外部类实例的逻辑状态。这个内部类称为外部类的序列化代理。它应该只有一个构造函数，其参数类型是外部类。此构造函数仅仅复制其参数中的数据：它不需要执行任何一致性检查或防御性复制。序列化代理的默认序列化格式是外部类的完美序列化格式。外部类及其序列化代理都必须实现`Serializable`。

例如，在技巧50中编写的不可变`Period`类，在技巧88中进行了序列化。下面是该类的序列化代理。`Period`非常简单，其序列化代理与该字段具有完全相同的字段：

		// Serialization proxy for Period class
		private static class SerializationProxy implements Serializable {
			private final Date start;
			private final Date end;
			SerializationProxy(Period p) {
				this.start = p.start;
				this.end = p.end;
			}
			private static final long serialVersionUID =
						234098243823485285L; // Any number will do (Item 87)
		}

接下来，将以下`writeReplace`方法添加到外部类中。可以将此方法逐字复制到具有序列化代理的任何类中：

		// writeReplace method for the serialization proxy pattern
		private Object writeReplace() {
			return new SerializationProxy(this);
		}

外部类上存在此方法会导致序列化系统产生的是序列化代理类的实例而不是外部类的实例。换句话说，`writeReplace`方法在序列化之前将外部类的实例转换为其序列化代理类的实例。

使用此`writeReplace`方法，序列化系统将永远不会生成外部类的序列化实例，但攻击者可能会构造一个试图违反类不变量的实例。要确保此类攻击失败，只需将此`readObject`方法添加到外部类：

		// readObject method for the serialization proxy pattern
		private void readObject(ObjectInputStream stream)
					throws InvalidObjectException {
			throw new InvalidObjectException("Proxy required");
		}

最后，序列化代理类提供`readResolve`方法，该方法返回外部类的逻辑等效实例。此方法会在反序列化时将序列化代理类的实例转换回外部类的实例。

这个`readResolve`方法只使用它的外部类的公共API创建一个它的实例，这样做的好处是反序列化时生成的实例是使用常规的创建实例的方法，即通过构造器，静态工厂方法来生成。这样就不必需要单独确保反序列化生成的实例要满足类的不变量要保持一致的约束。如果类的静态工厂或构造函数负责生成这些不变量并且使用实例方法来维护它们，那么这些不变量也会在序列化过程中得到维护。

以下是`Period.SerializationProxy`的`readResolve`方法：

		// readResolve method for Period.SerializationProxy
		private Object readResolve() {
			return new Period(start, end); // Uses public constructor
		}

序列化代理方法允许`Period`的变量为`final`，从而让`Period`类成为真正的不可变类。序列化代理模式还允许反序列化生成的实例具有与序列化生成实例属于不同的类型。

比如`EnumSet`（参阅36）。这个类没有公共构造函数，只有静态工厂。从客户端的角度来看，它们返回的是`EnumSet`实例，但在当前的OpenJDK实现中，它们返回两个子类中的一个，具体取决于底层枚举类型的容量。如果底层枚举类型包含64个或更少的元素，则静态工厂返回`RegularEnumSet`；否则，他们返回一个`JumboEnumSet`。

如果序列化时其底层枚举类型具有六十个元素，将生成`RegularEnumSet`实例，然后再向枚举类型添加五个元素，然后再反序列化先前生成的实例得到的结果不是`RegularEnumSet`实例，而是`JumboEnumSet`实例。

		// EnumSet's serialization proxy
		private static class SerializationProxy <E extends Enum<E>>
					implements Serializable {
			// The element type of this enum set.
			private final Class<E> elementType;
			// The elements contained in this enum set.
			private final Enum<?>[] elements;
			SerializationProxy(EnumSet<E> set) {
				elementType = set.elementType;
				elements = set.toArray(new Enum<?>[0]);
			}
			private Object readResolve() {
				EnumSet<E> result = EnumSet.noneOf(elementType); //noneof方法是返回不同类型EnumSet子类的关键
				for (Enum<?> e : elements)
					result.add((E)e);
				return result;
			}
			private static final long serialVersionUID = 362491234563181265L;
		}

序列化代理模式有两个限制。

- 与可扩展的类不兼容（参阅19）。
- 与某些对象图中包含环路的类不兼容：如果尝试从其序列化代理类的`readResolve`方法中调用外部类对象上的方法，将抛出`ClassCastException`，因为该外部类的对象还不存在。

小结：如果必须在不需要由客户端扩展的类上编写`readObject`或`writeObject`方法，就考虑使用序列化代理模式。
