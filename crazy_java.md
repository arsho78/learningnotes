## Chapter 3 数据类型和运算符 ##





javadoc工具默认只处理*public*或*protected*修饰的类, 接口, 方法, 成员变量, 构造器和内部类之前的文档注释.

protected定义的成员变量被子类访问时有如下限制：

- 当子类和父类在同一包中时，protected相当于public
- 当子类和父类不在同一包时，在子类内部可以直接使用父类的protected成员，或者通过子类自身实例访问继承自父类的protected成员，但是不能通过父类的实例来访问父类的protected成员。

java的标识符: 字母, 数字(不能打头), 下划线(_), 美元符($).
java的关键字都是小写

java语句可以跨行书写，但是字符串和变量名不能跨行

java使用int作为整型的默认类型,超出int范围时必须在整数的后面添加l或L来表示长整型.
当数值处于合法范围内,整数可以直接赋值给byte,short;
当数值处于int范围内,可以直接赋值给long;
当超出int范围,必须在数字后添加l或L来赋值给long.

正数的补码与原码相同,负数的补码是反码+1,符号位保持不变
根据负数的补码求值方法（举例用-5）：
1. 补码-1，取反，求值，ex： ...11011, -1得11010，取反得00101，求值得5，所以是-5
2. 补码取反，求值，+1， ex： ...11011, 取反得00100，求值得4，+1得5，所以是-5

所有char类型的值都可以参与算术运算，其值就是字符对应的unicode编号

表达式类型的自动提升

- 所有的byte，short，char都会被提升到int；
- 整个算数表达式的数据类型自动提升到与表达式中最高等级的操作数同样的类型。

正无穷大都相等，负无穷大也都相等，NaN与任何数值都不相等包括它自己
只有浮点数除0会得出正无穷大和负无穷大，整数除0会抛出ArithmeticException:/by zero

自动运算符++，--只能用于单个数值型变量，不能操作常量或表达式

赋值运算符支持连续赋值但不推荐


移位运算符规则：  

- 最低类型为int。char，byte，short都会自动转为int再运算
- int类型整数a>>b，当b大于32时，先用b对32求余，其结果才是真正移位的位数
- long类型整数a>>b，当b大于64时，先用b对64求余，其结果才是真正移位的位数
- 如果没有发生有效位丢失，左移n位相当于乘以2的n次方，右移相当于除以2的n次方
- 操作数不会发生变化，只会得到一个新的运算结果
- 负数按位非得得到的正数是其绝对值-1，ex: -5得到4



		byte a = 5;
		//会出错，右边自动提升为int
		a = a + 5;
		//不会出错
		a += 5;
		
等于比较符：数值类型操作数即使不是同一类型，只要值相等就返回true；两个引用类型之间如果没有父子继承关系，就不能用==比较

逻辑运算符的操作数必须是布尔型的变量或常量。
短路与和或会根据前面表达式的结果来判断是否执行后续表达式，非短路与和或一定会执行所有的表达式
异或逻辑运算符：相同为false，不同为true
异或位运算符：相同为0，不同为1

三目运算符不支持多个语句

使用if条件控制时，范围较小的条件优先处理

P69 表3.4 运算符优先级

## Chapter 4 流程控制与数组 ##

switch语句后面的控制表达式的数据类型只能是byte，short，char，int四种整数类型，枚举类型和String类型，不能是boolean；case后面只能跟常量；一旦遇到相等的值就会执行以后的代码，不再判断与后面的case，default标签的条件是否相等，除非碰到break才会结束。

while后面仅跟一个分号表明循环体是空循环体，只有一个分号。

for循环的迭代语句没有像while和do...while放在循环体内，一旦出现在continue语句后就可能不会执行而陷入死循环，for语句的迭代语句在括号内一定会被执行。

for循环可以初始化多个变量但只能有一个声明语句，因此这些变量都必须是同一种数据类型。

for循环括号内的三个语句都可以为空，也可以把初始化语句和迭代语句分别放在循环之前和循环体内。

for循环括号内只有2个分号是必须的，其他都是可以省略的。

java中的标签只有放在循环语句之前才有作用。

数组的静态初始化是指初始化数组时指定数组元素值。
只有在定义数组的同时执行数组初始化才支持使用简化的静态初始化。
不能同时使用静态初始化和动态初始化，即不能即指定长度也分配初始值。

使用foreach循环迭代访问数组元素时，并不能改变数组元素的值。

所有方法的局部变量（相当于指针）都保存在栈内存中，对象则保持在堆内存中。

## Chapter 5 面向对象（上） ##

变量存放在栈内存中，实际对象存放在堆内存中

`this`所指代的的对象在编译期无法确定

静态成员不能访问非静态成员。

个数可变的形参可以代表0-n个参数，只能处于形参列表的最后，也就是说一个方法最多只能有一个个数可变的形参。个数可变的形参本质上是一个数组类型的形参，因此可以直接传入一个数组。

方法递归一定要向可知方向递归，即一定会终结递归

方法重载的要求，两同一不同：

- 同一个类
- 方法名相同
- 参数列表不同

使用固定数量形参的方法总是会比使用可变个数形参的方法优先执行，因此不推荐重载形参个数可变的方法

成员变量无需显示初始化，局部变量除了形参必须显示初始化。

定义局部变量后，系统并未为这个变量分配内存空间，直到程序为这个变量赋初始值时，系统才会为其分配内存，并将初始值保存在这块内存中。局部变量保存在其所属方法的栈内存中。栈内存中的变量无需系统垃圾回收，随方法或代码块的运行结束而结束。

一个类里不能有同名的成员变量；一个方法里不能有同名的方法局部变量和形参；一个代码块内不能有和已有方法局部变量同名的代码块局部变量；不同代码块内可以有重名的代码块局部变量；方法局部变量可以和成员变量重名

外部类只能有2种访问控制级别：public和默认

如果一个java源文件里定义的所有类都没有使用public修饰，则这个java源文件可以是一切合法的文件名。

带包的可执行类在执行时必须使用全路径，而不能人工进入包目录，直接使用类名执行。java在执行可执行类时，会在classpath指定路径下寻找包目录，然后在包目录中寻找该类。同一个包的类可以存储在不同的位置，但必须具备相同的与包结构对应的目录结构。

父包和子包在用法上不存在任何关系，如果父包中的类要是用子包中的类，必须使用子包的全名，不能省略父包部分。

Java默认为所有源文件导入`java.lang`包下所有的类

当程序员调用构造器时，系统会先为该对象分配内存空间并执行默认初始化(此时对象已经创建，但只能被构造器通过`this`调用)，然后才会执行构造器。

构造器中使用`this`关键字调用其他构造器的语句必须放在第一条。

java的子类不会继承（可以调用）父类的构造器。

父类的private成员对子类也不可见。

方法的重写遵循两同两小一大：

- 方法名同，形参列表同
- 子类方法返回值类型，声明抛出的异常类型要比父类方法的更小或相等
- 子类方法的访问权限要比父类方法的大

重写方法和被重写方法必须属于同种方法：类方法或者实例方法，否则编译错误。

super和this关键字都不能出现在类方法中，也不会同时出现。因为都要求出现在第一行。

子类的构造器一定会调用父类构造器一次：

- 子类构造器第一行使用`super`关键字显式调用父类构造器
- 子类构造器没有显式调用父类构造器，也没有使用`this`关键字调用其他构造器，则隐式调用父类无参数的构造器

**父类必须拥有一个无参数的构造器。**

当程序创建一个子类对象时，系统不仅会为该类中定义的实例变量分配内存，也会为它从父类继承得到的所有实例变量分配内存。当系统创建一个java对象时，如果该java类有两个父类（直接父类和间接父类），那么这个java对象保存着这三个类中定义的所有的实例变量。

- 编译时类型：声明该变量时使用的类型
- 运行时类型：实际赋给变量的对象的类型


引用变量只能调用声明该变量时所用类型包含的方法，但方法体是运行时类型所对应的方法体。

通过引用变量访问他的实例变量，总是试图访问他编译时类型所定义的成员变量而不是运行时类型所定义的成员变量。

类型转换时，两个类之间必须有继承关系，否则，运行时出现ClassCastException异常

instanceof运算符前面操作数的编译时类型要么与后面的类相同，要么与后面的类具有父子继承关系，才能运行，返回值是`true`或`false`，如果没有继承关系，则编译出错。

尽量不要在父类构造器中调用将要被子类重写的方法

- 子类需要额外增加属性，而不仅仅是属性值的改变；
- 子类需要增加自己独有的行为方式（包括重写或者增加新的方法） 

初始化块在创建java对象时依次执行并在执行构造器之前执行，只能被static修饰，

当java创建一个对象时，系统先为该对象的所有实例变量分配内存（前提是该类已经被加载过了），接着程序开始对这些实例变量执行初始化：先执行初始化块或声明实例变量时指定的的初始值，再执行构造器里指定的初始值。

静态初始化块和普通初始化块都会从顶级父类向下递归执行：

		顶级父类静态初始化块
		次级父类静态初始化块
		...
		本类静态初始化块
		顶级父类普通初始化块
		顶级父类无参数构造器
		次级父类普通初始化块
		次级父类无参数构造器
		...
		本类普通初始化块
		本类无参数|带参数构造器
		

静态初始化块在类加载的初始化阶段执行，而不是在创建对象时才执行，因此比普通初始化块先执行。而且只能处理类变量，不能处理非静态的实例变量。


## Chapter 6 面向对象（下） ##

把字符串类型的值转换为基本类型：

- 包装类的parseXxx(String s)静态方法
- 包装类的XXX(String s)构造器

把基本类型转换为字符串类型：

- String类的value()方法

-128~127之间的同一个整数自动装箱成Integer实例时，永远都是引用cache数组里的同一个数组元素，所以他们全部相等，但每次把一个在此范围之外的整数自动装箱成Integer实例时，系统总是重新创建一个Integer实例，所以同一整数生成的Integer实例可能并不是同一对象。

重写`toString()`方法的一般格式：

		类名[field1=value1, field2=value2, ...]

当java程序使用new String("hello")来创建String对象时，会先在常量池里创建"hello"直接量，然后在堆内存中创建相应的String对象，所以总共创建了2个字符串对象。

**常量池(constant pool)**：管理在编译时被确定并保存在已编译的.class文件中的一些数据，包括类，方法，接口中的常量以及字符串常量。

		String s1 = "crazy";			//生成一个变量，一个常量
		String s2 = "java";				//生成一个变量，一个常量
		String s3 = "crazyjava";		//生成一个变量，一个常量
		String s4 = "crazy" + "java";	//生成一个变量，"crazyjava"在常量池中已存在，所以不再生成
		String s5 = s1 + s2;"			//生成一个变量，s1和s2都是变量，不能在编译期取值
		
重写equals()方法模板：

		pubic boolean equals(Object obj) {
			//如果两个对象为同一对象
			if (this == obj) {
				return true;
			}
			//只有当obj时Person对象
			if (obj != null && obj.getClass() == Person.class) {
				Person personObj = (Person) obj;
				//若2个对象的idStr相等则两个对象相等
				if (this.getIdStr().equals(personObj.getIdStr())) {
					return ture;
				}
			}
			return false;
		}

正确重写equals()方法的原则：

- 自反性：对任意x，x.equals(x)一定返回true
- 对称性：对任意x和y，x.equals(y)与y.equals(x)的结果一定相同
- 传递性：对任意x，y，z，如果x.equals(y)和y.equals(z)都返回true，则x.equals(z)一定为true
- 一致性：对任意x，y，如果对象中用于等价比较的信息没有改变，那么x.equals(y)的返回值一直保持一致
- 对任何不是null的x，x.equals(null)一定返回false

系统在第一次加载类时为类变量分配内存存储空间，创建该类对象时就不再为类变量分配空间，也不会对其初始化。一旦该类初始化结束后，静态初始化块将永远不会获得执行的机会。

即使某个类的实例为null，也可以通过他访问所属类的类成员，但不能访问实例成员，会引发NullPointerException。

Singleton代码示例：
				//使用一个类变量缓存创建的唯一实例
			private static Singleton instance;
				//构造器使用private隐藏
			private Singleton() {}
		
		class Singleton {
		//提供一个静态方法，返回实例
				//加入自定义控制，保证只有一个对象
			public static Singleton getInstance() {
				//如果instance为null，说明还没创建对象，创建一个返回
				//如果instance不为null，说明已创建，直接返回
				if (instance == null) {
					instance = new Singleton();
				}
				return instance;
			}
		}
		

final修饰的成员变量必须由程序员显示指定初始值。final类变量在初始化类时已经被指定初始值，所以不能在普通初始化块中重新赋值。

final修饰的局部变量可以不在声明时赋值，但在之后必须且只能被赋值一次。

final变量在满足以下条件时相当于一个直接量：

- 使用final修饰
- 定义该变量时指定了初始值
- 该初始值在编译时就能确定下来（表达式只是基本的算术表达式或字符串连接运算，没有访问普通变量，调用方法）

编译器会把程序中所有用到该变量的地方直接替换成该变量的值。如果被赋的是基本的算术表达式或字符串连接，没有访问任何普通变量，调用方法，编译器同样会把这种final变量作为直接量(宏变量)处理。

final实例变量只有在定义时就指定初始值才会有宏变量效果。

创建自定义的不可变类规则：

- 使用private和final修饰符来修饰该类的成员变量
- 提供带参数的构造器，用于根据传入参数来初始化类里的成员变量
- 仅为该类的成员变量提供getter方法，不提供setter方法
- 如果有必要，重写Object类的hashCode()和equals()方法。
- 如果要完全不可变类，则要利用构造器中传入的参数重新生成一个新的同类型对象保存在一个private final实例变量中，并且在该变量的get方法中生成一个新的完全相同的对象返回，从而保证这个不可变类里的实例变量只在类内可见。

#### 带缓冲的不可变类 ####
- 使用数组保存已生成实例
- 使用private修饰构造器
- 创建`valueOf()`方法，用于创建未生成实例并加入数组或从数组中取出已生成实例
- 根据需要，重写`equals()`和`hashCode`方法



抽象类即使不包含抽象方法也不能用于创建实例。

抽象类可以包含成员变量，方法，构造器，初始化块，内部类5种成分。抽象类的构造器不能用于创建实例，主要用于被子类调用。

抽象类必须包含一个不带参数的构造器，否则无法被继承。

final和abstract永远不能同时使用；abstract不能用于修饰变量，也不能用于修饰构造器；static和abstract不能同时修饰同一个方法，但可以同时修饰内部类；private和abstract不能同时修饰同一个方法；

接口不能包含构造器和初始化块，可以包含成员变量（只能是静态常量），方法（只能是抽象实例方法，类方法或默认方法），内部类（包括内部接口和枚举）。


- 接口里的所有成员，包括常量，方法，内部类和内部枚举都是public访问权限。
- 接口里的常量总是用public static final修饰的（也是默认修饰符），因为接口中没有构造器和初始化块，所以只能在定义时赋值。
- 接口中的普通方法总是使用public abstract来修饰，不能有方法体。类方法和默认方法都必须有方法体。
- 接口里定义的内部类，内部接口，内部枚举默认都采用public static来修饰。
- 接口的默认方法必须是用default修饰，不能使用static修饰，而且总是public的，可以有多个默认方法。
- 接口的类方法必须使用static修饰，不能用default，而且总是public的

接口和抽象类的区别

- 接口只能包含抽象方法，静态方法和默认方法，不能为普通方法提供实现；抽象类则可以包含普通方法
- 接口里只能定义静态常量，不能定义普通成员变量，抽象类则都可以
- 接口不包含构造器，抽象类可以
- 饥渴不包含初始化块，抽象类可以
- 一个类只能有一个直接父类，但可以实现多个接口

简单工厂模式：在应用中使用接口来代替具体的实现类，用单独一个工厂类来完成接口到具体实现类的转接

命令模式：在方法签名中使用接口包装对数据的处理，具体的数据处理在调用方法时传入具体的实现了该接口的命令类实例，参考Comparator接口的使用

非静态内部类不能拥有静态成员

当在非静态内部类的方法内访问某个变量时，寻找顺序如下：


1. 系统优先在该方法内查找是否存在该名字的局部变量
2. 该方法所在内部类是否有该名字的成员变量
3. 该内部类所在的外部类是否有该名字的成员变量
4. 都没有，编译错误，提示找不到该变量

如果外部类成员变量，内部类成员变量与内部类里方法的局部变量同名，则可以通过this，外部类名.this作为限定来区分

如果外部类需要访问非静态内部类的成员，必须显式创建非静态内部类的对象来调用访问其实例成员。

外部类的静态成员不能直接使用非静态内部类。

非静态内部类里不能有静态成员，包含静态变量，静态初始化块和静态方法。

静态内部类可以包含静态成员，也可以包含非静态成员。但在其内部只能访问外部类的静态成员，不能访问实例成员。

外部类不能直接访问静态内部类的成员，但可以通过内部类名来访问其静态成员或者内部类对象来访问其实例成员。

静态内部类和非静态内部类的成员都不能被外部类直接使用。

- 静态内部类：
	- 内部类对外部类成员的访问受限，只能访问外部类的静态成员
	- 外部类对内部类的使用没有限制，所有方法和初始化块都可以使用内部类声明变量或创建对象
- 非静态内部类：
	- 内部类对外部类的访问没有限制，可以直接访问外部类的所有成员，包括private成员
	- 外部类对内部类的使用受限，外部类的静态成员不能使用内部类声明变量或创建实例
	- 不能包含任何静态成员

接口里定义的内部类只能是静态内部类

内部类的使用：

1. 外部类内可以直接使用，与其他普通类没有区别
2. 在外部类以外使用非静态内部类：必须先创建外部类的对象，再通过该对象创建其内部类对象，即非静态内部类对象必须依附于某一个其所在外部类的对象。非静态内部类的子类的对象也必须依附于某一个其父类所在外部类的对象。
3. 在外部类以外使用内部静态类：通过外部类名.内部类名来使用，类似于包空间

局部内部类的class文件比成员内部类的class文件多了一个数字

匿名内部类必须继承一个父类或者实现一个接口，但最多只能继承一个父类或实现一个接口。

- 实现接口：只能通过接口名()来调用匿名内部类的隐式无参数构造器
- 继承父类：可以使用与父类构造器拥有相同形参列表的构造器

匿名内部类不能：

- 是抽象类
- 包含构造器

被匿名内部类访问的局部变量必须是final（系统会自动限制-effectively final）

当Lambda表达式需要返回值，而他的代码块仅有一条省略了return的语句，Lambda会自动返回该语句的值

函数式接口：只有一个抽象方法的接口（可以含有类方法和默认方法）。

Lambda表达式的目标类型必须是明确的函数式接口。

Java8，`java.util.function`中常见的函数式接口：
- XxxFunction：通常包含一个`apply()`方法，用于对参数进行转换处理，然后返回一个新值
- XxxConsumer：通常包含一个`accept()`方法，也是对参数进行处理，但不返回处理结果
- XxxPredicate：通常包含一个`test()`方法，对参数进行判断，然后返回一个boolean值
- XxxSupplier：通常包含一个`getAsXxx()`方法，不需要输入参数，会按实现的逻辑算法返回一个数据

如果Lambda表达式的代码块只有一条代码，还可以在代码块中使用方法引用和构造器引用：p216 表6.2

- 类名::类方法
(a, b, ...) -> 类名.类方法(a, b, ...)
- 特定对象::实例方法
(a, b, ...) -> 特定对象.实例方法(a, b, ...)
- 类名::实例方法
(a, b, ...) -> a.实例方法(b, ...)
- 类名::new
(a, b, ...) -> new 类名(a, b, ...)


lambda表达式和匿名内部类的相同点：

- 都可以直接访问“effectively final”的局部变量，以及外部类的成员变量
- 创建的对象都可以直接调用从接口中继承的默认方法

不同点：

- 匿名内部类可以为任意接口创建实例，lambda只能为函数式接口服务
- 匿名内部类可以为抽象类甚至普通类创建实例，lambda只能为函数式接口服务
- 匿名内部类实现的抽象方法的方法体允许调用接口中定义的默认方法，lambda表达式的代码块则不允许调用


枚举类

- 使用enum定义的枚举类默认继承了java.lang.Enum类，因此不能显式继承其他父类；
- 非抽象的枚举类默认使用final修饰，因此不能派生子类
- 枚举类的构造器只能使用private访问控制符
- 枚举类的所有实例必须在枚举类的第一行显式列出，以,分开，;结束，且总是public static final。

当switch控制表达式使用枚举类型时，case表达式中的值可以直接使用枚举值，无需添加枚举类作为限定。

枚举类通常应该设计为不可变类，因此建议将枚举类的成员变量都使用private final修饰，并在构造器里为这些final成员变量赋值，也就是说必须显式定义带参数的构造器。

实现了接口的枚举类可以为每个枚举值提供不同的抽象方法实现。拥有抽象方法的枚举类可以衍生子类，每个实现了抽象方法的枚举值都是他的匿名子类

包含抽象方法的枚举类也不能显式使用abstract修饰，但是定义每一个枚举值时要为抽象方法提供实现，否则出现编译错误。

垃圾回收机制只负责回收堆内存中的对象，不负责回收物理资源；在回收任何对象之前，都会调用它的`finalize()`方法。

可以调用以下方法来督促系统进行垃圾回收：
- `System`类的静态方法`System.gc()`
- `Runtime`对象的`gc()`实例方法：`Runtime.getRuntime().gc()`

当jvm执行finalize()方法出现异常时，垃圾回收机制不会报告异常，程序继续执行

p233 表6.3 Java修饰符适用范围总表

abstract和final永远不能同时使用；abstract和static不能同时修饰方法，但可以同时修饰内部类；abstract和private不能同时修饰方法，可以同时修饰内部类；private和final可以混用但没有意义。



## Chapter 7 java基础类库 ##

为Scanner设置分隔符使用useDelimiter(String pattern)

通过identityHashCode(Object x)可以获得对象的identityHashCode值，这个特殊的identityHashCode值可以唯一地标识对象。默认情况下Object类的hashCode()方法得到的就是identityHashCode。

Object类的clone()方法只是一种浅克隆，只克隆该对象的所有成员变量值，不会对引用类型的成员变量值所引用的对象进行克隆。

Objects类的requireNonNull()方法，当传入的参数不为null时，返回参数，否则引发NullPointerException异常。用于对方法形参进行输入校验。

Objects类的`hashCode(Object obj)`返回的hashcode值和`obj.hashcode()`返回值不一样。

Objects类的`requireNonNull(Object obj, Supplier<String> messageSupplier)`比`requireNonNull(Object obj, String message)`效率要高


StringBuilder和StringBuffer的用法完全，只是StringBuffer是线程安全的。两者都实现了`CharSequence`接口，这个接口可以作为字符串的协议接口

使用StringBuilder和StringBuffer的`reverse()`方法时，(high-low)的surrogate对不会翻转，作为一个字符处理，而(low-high)的surrogate对会翻转，成为一个新的字符。

`String`类的`compareTo(String anotherStr)`方法的可能返回值：
- 0：当`this.equals(str) == true`
- `this.charAt(k) - str.charAt(k)`：如果2个字符串中有不同字符，其中`k`是出现不同字符的最低索引位置
- `this.length() - str.length()`：如果2个字符串没有不同字符，但长度不同。

`Math.round(double)`方法返回`long`，而不是`double`

使用默认种子构造Random对象时，属于同一个种子。

只要两个Random对象的种子相同，方法的调用顺序也相同，他们会产生相同的数字序列。

推荐使用当前系统时间作为Random对象的种子：

    Random rand =  new Random(System.getCurrentTimeMills());

建议优先使用基于String的构造器构造BigDecimal类的实例，如果必须使用double浮点数创建BigDecimal对象，而是通过BigDecimal.valueOf(double value)静态方法来创建BigDecimal对象。

`BigDecimal`的对象时不可变的，因此使用`setScale()`系列方法不会改变修改原对象，会返回一个具有新标度的对象，这个对象可能不是新创建的。

Calendar类的add,roll,set方法区别

- add()与set()方法类似，当修改的字段超出允许范围，会发生进位，如果下一级字段也需要修改，该字段会修正到变化最小的值
- roll()方法不会发生向上进位，对下级字段处理与add方法相同

Calendar类有2种模式：

- lenient模式：允许字段超出允许范围，做进位处理
- non-lenient模式：不允许字段超出范围，否则抛出异常

典型的正则表达式的使用：

		Pattern p = Pattern.compile("a*b");
		Matcher m = p.matcher("aaaaabb");
		boolean b = m.matches();

java的国际化：

1. 准备basename.properties，basename_language.properties或basename_language_country.properties文件。其中language和country必须使用java的Locale中的language和country。
2. 使用native2ascii工具转换包含非西欧字符的资源文件

		native2ascii basename.properties basename_language_country.properties

3. 使用以下类似代码从相应资源文件中取得所需语音文字：

		public class Hello {
			public static void main(String[] args) {
				Locale myLocale = Locale.getDefault(Locale.Category.FORMAT);
				ResourceBundle bundle = ResourceBundle.getBundle("mess", myLocale);
				System.out.println(bundle.getString("hello"));
			}
		}

## Chapter 8 Java集合 ##

所有的集合类都位于java.util包下

数组里既可以存储基本类型，也可以存储对象，集合里只能存储对象。

当使用Iterator对集合元素迭代时，Iterator并不是把集合元素本身（也就是指向对象的引用）传递给了迭代变量，而是把集合元素的值传给了迭代变量，所以修改迭代变量的值不会对集合元素有任何影响。参考基本数据类型的值复制。

当使用Iterator迭代访问集合元素时，集合里的元素不能被改变，只能通过Iterator的remove()方法删除上一次next()方法返回的集合元素，否则会引发java.util.ConcurrentModificationException异常。

Iterator采用快速失败（fail-fast）机制，一旦在迭代过程中检测到集合被修改，就会立刻引发java.util.ConcurrentModificationException异常。

与使用Iterator接口迭代访问集合元素类似的是，foreach循环中的迭代变量也不是集合元素本身，因此在foreach循环中修改迭代变量的值不会影响集合元素，该集合也不能被改变，否则将引发ConcurrentModificationException异常

#### Stream 操作 ####
独立使用Stream的步骤：

1. 使用Stream或XxxStream的builder()类方法来创建该Stream对应的builder
2. 重复使用Builder类的add()方法来向该流中添加多个元素
3. 调用Builder的build()方法来获取对应的Stream对象
4. 调用Stream类的聚集方法

Stream类的聚集方法可以分为：

- 中间方法： 中间操作允许流保持打开状态，并允许直接调用后继方法，返回值时另外一个流
- 末端方法： 对流的最终操作，该流会被消耗，并不能再用。

Stream类的聚集方法也可以是：

- 有状态的方法： 会给流增加一些新的属性，如元素的唯一性，最大数量，完成排序等处理。
- 短路方法： 尽早结束对流的操作，不必检查所有的元素。

HashSet的特点：

- 不能保证元素的排列顺序
- HashSet不是同步
- 集合元素值可以是null

当加入新元素时，HashSet调用元素的hashCode()方法来计算该元素的hashCode值，然后根据该值决定新元素在集合中的位置。

如果两个元素通过equals()方法比较返回true，但hashCode值不同，HashSet会把他们存储在不同的位置。

HashSet判断两个元素相等的标准是他们的hashCode相等且equals()返回true。如果：

- equals相等，hashCode不等，则为不同元素，存入不同位置
- equals不等，hashCode相等，则为不同元素，但以链表形式存入相同位置(bucket)，造成性能下降
- equals和hashCode都相等，则为同一元素 
- equals和hashCode都不等，则为不同元素

重写hashCode()的基本原则：

- 同一对象多次调用hashCode()方法应该返回相同的值
- 两个对象使用equals()方法对比的结果应该和他们的hashCode对比结果相同
- 所有用于equals比较标准的实例变量也应该用于计算hashCode

实例变量类型						|计算方式
--------------------------------|-------
boolean							|f?0:1
整数类型(byte, short, char, int)	|(int)f
long							|(int)(f^(f>>>32))
float							|Float.floatToIntBits(f)
double							|long l = Double.doubleToLongBits(f); (int)(l^(l>>>32))
引用类型							|f.hashCode()

常用计算hashcode值的方法：

1. 使用上表中的计算方式计算与hashcode值有关的实例变量的hashcode
2. 把每个相关实例变量的hashcode值与一个质数相乘再相加得到最终的hashcode值

在把对象加入到HashSet后，尽量不要修改影响equals和hashCode的实例变量的值，否则会引起HashSet混乱

加入TreeSet的类必须实现Comparable接口，而且必须是同一个类的对象，否则抛出ClassCastException异常

TreeSet使用compareTo()方法来判断两个对象是否相等，如果两个对象通过compareTo(Object obj)方法比较相等，则新对象无法加入TreeSet中。因此，在改写放入TreeSet的类的equals()方法时，应注意他的返回值与compareTo()方法保持一致。

TreeSet可以删除没有被修改实例变量，且不与其他被修改实例变量的对象重复的对象，但不能删除改变了实例变量的对象，也不能删除未修改但与修改过元素相等的元素。这个限制会在成功删除某个元素后解除，因为TreeSet会重新索引所有元素。

TreeSet的自然排序采用升序排列；如果使用定制排序，则采用指定的Comparable接口实现类(通常是Lambda表达式)来负责排序，而元素本身不必实现Comparable接口

EnumSet不允许添加null元素，会抛出NullPointerException异常。但查询和删除null元素不会抛出NullPointerException，删除操作会返回false。EnumSet类没有显示构造器，只能通过静态方法来创建对象。

EnumSet是所有Set类中性能最好的，但只能保存同一个枚举类的枚举值作为集合元素。

HashSet，TreeSet和EnumSet都是线程不安全的，如果需要保证Set集合的同步性，可以再创建Set时使用Collections的synchronizedSortedSet方法包装该集合。

		SortedSet set = Collections.synchronizedSortedSet(new TreeSet(...));

List判断两个对象相等只需equals()返回true即可。

List的set(int index, Object element)的index必须是List的有效索引。

Arrays类的asList(Object... a)方法生成的List集合是Arrays类内部类ArrayList的实例，程序只能遍历访问该集合里的元素，不能增加或删除内容。

Queue接口定义的方法：

- 加入元素到队尾
	- void add(Object e) 队列若满则抛出异常
	- boolean offer(Object e) 队列若满则返回false，通常比add更好
- 获取队列头部的元素
	- Object element() 获取但不删除，为空则抛出异常
	- Object peek() 获取但不删除，为空则返回null
	- Object poll() 获取并删除，为空则返回null
	- Object remove() 获取并删除，为空则抛出异常

PriorityQueue类不允许插入null元素，还需要对队列元素进行排序，因此显示顺序可能不同于提出数据的顺序。

使用List的建议：

- 如果需要遍历List集合元素，对于ArrayList和Vector集合，应使用随机访问方法（get）来遍历集合元素，对于LinkedList集合，可以使用迭代器（Iterator）遍历
- 如果需要经常插入，删除包含大量数据的的List集合的大小，可考虑使用LinkedList集合
- 如果需要保证同步性，需要用Collections工具类包装

HashMap，Hashtable判断两个value相等的标准是两个对象通过equals()方法比较返回true即可。
如果使用自定义类作为HashMap,Hashtable的key，必须重写该类的equals()方法和hashCode()方法以获得一致的判断结果。

如果使用自定义类作为TreeMap的key，必须重写该类的equals()方法和compareTo()方法以获得一致的返回结果。

在IdentityHashMap中，当且仅当两个key严格相等（key1 == key2）时，IdentityHashMap才认为两个key相等，对于普通的HashMap，只要key1和key2通过equals()方法比较返回true，且它们的hashCode值相等即可。

当TreeMap被填充后，可以调用keySet()，取得由key组成的set，然后使用toArray()方法生成key的数组，在使用Arrays的binarySearch()方法在已排序的数组中快速地查询对象。


## Chapter 9 泛型 ##
调用方法时必须为所有的数据形参传入参数值，与调用方法不同，使用类，接口时也可以不为类型形参传入实际的类型参数。编译器会发出警告，并自动把类型参数设置为Object

不存在泛型类：不管为泛型的类型形参传入哪一种类型实参，java都会将他们作为同一个类处理，在内存中也只占用一块内存空间，因此在静态方法，静态初始化或者静态变量的声明和初始化中不允许使用类型参数，instanceof后面也不能使用泛型类。

Java不支持泛型数组

带通配符的List仅表示它是各种泛型List的父类，并不能把元素加入其中，使用get()方法取出的元素时Object类型。

		List<?> list = new ArrayList<String>();
		//compile error
		list.add(new Object());

程序可以为类型形参设定多个上限，但至多只能有一个父类上限，可以有多个接口上限，用&连接，父类上限必须放在第一位。

		class <T extends SuperClass & Interface1 & Interface2 ...>

泛型方法：声明方法时定义一个或多个类型形参

		修饰符 <T, S> 返回值类型 方法名(形参列表) {}

泛型方法和类型通配符的区别：

使用类型通配符的参数或变量是无法在编译阶段确定其具体类型的。


- 通配符是被设计用来支持灵活的子类化的，适合只需要在不同的调用点传入不同实际类型。
- 如果某个方法中一个形参(a)的类型或返回值的类型依赖于另一个形参(b)的类型，则应该使用泛型方法在方法签名中声明类型参数。
- 如果无需向集合中添加元素或删除元素，可以使用类型通配符，无需使用泛型方法
- 类型通配符既可以在方法签名中定义形参的类型，也可用于定义变量的类型（List<? extends Number> c），但泛型方法中的类型形参必须在方法中显式声明。

如果程序指定了泛型构造器中声明的类型形参的实际类型，则不可以使用“菱形”语法。

		class MyClass<E> {
			public <T> MyClass(T t) {...}
		}
		...
		MyClass<String> mc = new <Integer> MyClass<String>(5); 
		//MyClass<String> mc = new <Integer> MyClass<>(5); wrong 


使用类型通配符的方法可以重载但可能无法调用

		public static <T> void copy(Collection<T> dest, Collection<? extends T> src) {}
		public static <T> void copy(Collection<? super T> dest, Collection<T> src) {} 

如果没有为泛型类指定实际的类型参数，则该类型参数被当成raw type处理，默认为声明该类型参数时指定的第一个上限类型。
当把一个具有泛型信息的对象赋给一个没有泛型信息的变量时，所有在尖括号内的类型信息都会丢失，变成raw type。

如果一段代码在编译时没有提出“[unchecked]未经检查的转换”警告，则程序运行时不会引发ClassCastException异常。所以不能在创建数组时使用类型变量或类型形参，除非是无上限的类型通配符，但可以在声明这样的数组。比如可以声明List<String>[]类型的数组，但不能创建ArrayList<String>[10]这样的数组对象。

在应用中，可以无上限的通配符来声明和创建数组对象，然后使用添加了instanceof判断的强制类型转换取出数组元素。


## Chapter 10 异常处理 ##

通常情况下，如果try块被执行一次，则try块后只有一个catch块会被执行，绝不可能有多个catch块被执行。

捕获异常的原则是先捕获小异常，再捕获大异常

捕获多种类型的异常时，多种异常类型之间用竖线(|)隔开

捕获多种类型的异常时，异常变量有隐式的final修饰，因此程序不能对异常变量重新赋值。

如果try块的某条语句抛出异常，该语句后的其他语句都不会获得执行的机会。

即使try块或catch块中执行了return语句，finally块总会被执行，而且catch块和finally块至少出现其中之一

如果在异常处理代码中调用了System.exit(1)来退出虚拟机，则finally块将失去执行的机会。

通常情况下，不要在finally块中使用return或throw等会导致方法终止的语句，否则会造成try块，catch块中的return，throw语句失效。

当java程序执行try块，catch块时遇到了return或throw语句，方法不会立即结束，系统会先去寻找该异常处理流程中是否包含finally块，如果没有，程序立刻执行这两个语句，方法终止；如果有finally块，系统立即开始执行finally块，然后才跳回来执行return和throw语句。所以如果finally块中也有return或throw语句，将不会再跳回try块和catch块执行任何代码
。

通常没有必要使用超过2层的嵌套异常处理

可以再catch块中使用throw抛出新的异常以便于方法调用者对异常进行二次处理

方法签名中声明的抛出异常可以比捕捉的异常更具体，如方法声明throws FileNotFoundException，但方法体内catch Exception

异常链：捕获一个异常然后接着抛出另一个异常，并把原始异常信息保存下来。

如果main方法没有处理捕捉到的异常，jvm会中止程序运行，并打印异常的跟踪栈信息。

通常处理异常方式：

- 捕获并保留诊断信息
- 通知合适的人员
- 采取合适的方式结束异常活动

处理异常注意事项：

- 不要过度使用异常。不要用异常处理代替流程控制，也不要和普通错误混在一起
对于完全已知的错误，应该编写处理这种错误的代码，增加程序的健壮性。只有对外部的，不能确定和预知的运行时错误才使用异常。
- 不适用过于庞大的try块
- 避免使用catch all语句
- 不要忽略捕获到的异常


## Chapter 13 MySQL 数据库与JDBC编程 ##

JDBC： Java Database Connectivity， Java数据库连接。

数据库驱动程序是JDBC程序和数据库之间的转换层，数据库驱动程序辅助将JDBC调用映射成特定的数据库调用。

### SQL语法 ###
DBMS： Database Management System， 数据库管理系统。

DBMS的数据字典（系统表）用于存储它拥有的每个事务的相关信息。

元数据：关于数据的数据。

数据表中每行称为一条记录，每列称为一个字段。

每条MySQL命令结束后都应该有一个英文分号。

		create database [IF NOT EXISTS] 数据库名;
		drop database 数据库名;
		use 数据库名;
		show tables;
		desc 表名;

SQL： Structured Query Language，结构化查询语言。

SQL语句分类：

- 查询语句：select语句
- DML： Data Manipulation Language，数据操作语言。 insert，update， delete
- DDL： Data Definition Language，数据定义语言。create，alter，drop，truncate
- DCL： Data Control Language，数据控制语言。grant， revoke
- 事务控制语句：commit， rollback， savepoint

SQL语句的关键字不区分大小写。

SQL的标识符命名规则：

- 以字母开头；
- 可以包括字母，数字和三个特殊字符#, $, _
- 不要使用当前数据库系统的关键字，保留字，单词之间以下划线`_`分隔
- 同一个外模式下的对象不应该同名

常见的数据库对象：表13.1，P584
MySQL支持的列类型：表13.2， P585

		create table [模式名.]表名 （
			columnName datatype [default expr],
			...
		);

		# 新表中的字段列表必须和子查询中的字段列表数量匹配，如果省略了字段列表，则新表的列名和选择结果完全相同
		create table [模式名.]表名 [column1[, column...]]
		as subquery;

		alter table 表名
		add (
			column_name1 datatype [default expr],
			...
		);

SQL语句中的字符串值不是用双引号引起，而是用单引号。

增加字段时，如果数据表中已有数据记录，那么除非给新增的列制定了默认值，否则新增的数据列不可指定非空约束。

		alter table 表名
		modify column_name datatype [default expr] [first|after col_name];
		# first 或者 after col_name指定需要将目标修改到指定位置
		# add新增的列名必须是原表中不存在的，而modify修改的列名必须时原表中已存在的。
		# 在MySQL中modify命令一次只能修改一个列定义，如果需要一次修改多个列定义，则可以在alter table后面使用多个modify命令
		#修改默认值的操作只会对以后的插入操作起作用。

		# 删除列
		alter table 表名
		drop 列名;

		# 修改表名
		alter table 表名
		renanme to 新表名;

		# modify增强版，可修改列名，如果不需要改列名，还是使用modify
		alter table 表名
		change 旧列名 新列名 datatype [defautl expr] [first|after col_name];

		# 删除表
		drop table 表名;

		# 删除表中所有记录，但保留表结构
		truncate 表名;

约束：在表上强制执行的数据校验规则。

约束的类型：

- NOT NULL： 非空约束。只能作为列级约束使用。
	- 所有的数据类型的值都可以是null，包括int，float，boolean
	- 空字符串不等于null，0也不等于null
- UNIQUE： 唯一约束
	- 可以出现多个null值
	- 为某列创建唯一约束时，MySQL会为该列相应地创建索引。如果不给唯一约束起名，该唯一约束默认与列名相同。多列约束的默认约束名与第一列的列名相同，如果出现重名，则从2开始在约束名尾添加数字
	- 如果需要为多列创建唯一约束，或者需要为唯一约束指定约束名，则只能用表级约束语法。
	- 建立唯一约束时，MySQL在唯一约束所在列或列组合上建立对应的唯一索引
	- 表级约束既可以放在create table中与列定义并列，也可以在alter table中使用add关键字添加
	- 使用modify关键字为单列采用列级约束语法来增加唯一约束
	- 标准SQL在alter table语句后使用`drop constraint 约束名`来删除约束，MySQL使用`drop index 约束名`来删除唯一约束。
		
		[constraint 约束名] 约束定义

- PRIMARY KEY：主键
	- 相当于非空约束+唯一约束
	- 单列主键不允许出现重复值，也不允许出现null值；
	- 多列主键每一列都不能为空，但只要求列组合值不能重复。
	- 每个表中最多只能有一个主键
	- MySQL中所有主键约束名都是PRIMARY
	- 如果某主键列的类型为整型，则可以指定该列具有自增长功能。 `auto_increment`
- FOREIGN KEY：外键
	- 外键确保相关的两个字段的参照关系：从表外键列的值必须在主表被参照列的值范围内，或者为空（如果没有非空约束） 
	- 主表的记录只有在删除所有参照它的从表记录后才能被删除，也可以强制级联删除达到同样目的。
	- 从表只能参照主表的唯一键列（包括主键列）
	- 同一表内可以有多个外键
	- 建立外键时，MySQL也会为该列建立索引
	- 列级约束使用`references`关键字，MySQL中可以使用列级约束，但不会生效。表级约束配合`foreign key`一起使用
	- 如果创建外键约束时没有指定约束名，则MySQL会为该外键约束命名为`table_name_ibfk_n`，n初始值为1
	- 多列外键必须使用表级约束定义，且从表的组合字段数量和类型必须和参照的主表中的多列主键中的字段数量以及类型一一对应。
	- 建立外键约束时，使用`on delete cascade`来激活级联删除，即当主表记录删除时，参照该记录的从表记录也自动删除；或者使用`on delete set null`把从表的相应外键值设置为null
- CHECK：布尔表达式检查，MySQL中无效支持

约束也是数据库对象，并被存储在系统表中，也有自己的名字。

指定约束的时机：

- 建表的同时为相应的数据列指定约束；
- 建表后创建，以修改表的方式来增加约束。

### 索引 ###
索引是存放在模式（schema）中的一个数据库对象，在数据字典中独立存放，但不能独立存在，必须属于某个表。

MySQL中使用information_schema数据库里的STATISTICS表存放所有的索引信息。

创建索引的两种方式：

- 自动：在表示定义主键约束，唯一约束和外键约束时，系统自动为该数据列创建索引
- 手动：通过`create index`语句创建

		create index index_name on table_name (column[, column]...);
		drop index 索引名 on 表名

### 视图 ###
创建视图就是创建视图名和查询语句的关联。

		create or replace view 视图名
		as subquery
		# MySQL允许通过with check option子句使视图不能被修改;Oracle使用的是with read only子句
		[with check option];

		drop view 视图名

### DML语法 ###

#### insert into：插入记录 ####

		insert into table_name [(column [, column...])]
		values(value [, value...]);

如果不在表名后列出列名，则必须为所有的列指定值，如果某列的值不确定，则分配一个null值，如果该列有非空约束，则会出错。

外键约束保证被参照的记录一定存在，但也可以使用null表示没有对应任何主表的记录。可以通过添加非空约束来限定子表的记录必须对应某个主表记录。

MySQL允许在values后面使用多个括号插入多条记录，括号之间以英文逗号隔开。

#### update 更新记录 ####

		update table_name
		set column1 = value1[, column2 = value2...]
		[where condition];

#### delete from 删除记录 ####

		delete from table_name
		[where condition];

### 单表查询 ###

		select [distinct] column1, column2...
		from 数据源
		[where condition];
		# 没有where子句，则选出所有行；也可以使用*表示所有的列
		# distinct紧跟select，只根据所选列而不是所有列的组合值来判断重复值

在select语句中使用算术表达式的规则：

- 数值型数据列，变量，常量可以使用算术表达符（+，-，×，/）创建表达式
- 日期型数据列，变量，常量可以使用部分算术表达式（+，-）创建表达式，两个日期之间也可以进行减法计算，日期和数值之间也可以进行加减运算
- 运算符也可以用于两列间的运算
- MySQL不能使用+来连接字符串，但可以通过concat函数连接2个字符串
- MySQL中如果在算术表达式中或concat函数中使用null，则整个表达式或函数的返回值都会是null
- 如果列别名中使用特殊字符（如空格），或者需要强制大小写敏感，可以通过用双引号包装别名来实现
- 如果为多列起别名，则列与列之间以逗号隔开，列和别名之间以空格隔开

如果select后跟的是一个常量，则结果是显示该常量的值n次，n是满足where条件的记录的个数。

Oracle和MySQL都支持使用一个虚表来显示一次常量：
		
		select 5 from dual;

可以使用distinct关键字从查询结果中消除重复行

SQL中判断两个值是否相等的比较运算符是单等号，不相等的运算符时<>，赋值运算符时:=

特殊的比较运算符：表13.3 p602

- 使用between val1 and val2 必须保证val1小于val2，这两个值不仅可以是常量，也可以是变量，或者是列名
- in后的括号里必须列出一个或多个值

SQL语句中可以使用两个通配符：

- 下划线`_`：代表任意一个字符
- 百分号`%`：代表任意多个字符

MySQL使用反斜线\作为转义字符，标准SQL中没有转义字符，使用escape关键字显式转义。

判断值是否为空不要用等号，因为null=null返回null

逻辑运算符：and/or/not

使用order by对查询结果排序，默认升序，可以使用desc指定降序排列，多列排序时，必须为每列单独设定排序方法。当前一个列出现重复值时，再用下一列排序。

		order by col_name [desc], col_name2...col_name2

#### 数据库函数 ####

单行函数对每行输入单独计算，每行得到一个计算结果；多行函数对多行输入值计算，最后只会得到一个结果。

MySQL处理null的函数

- ifnull(expr1, expr2)：如果expr1为null，则返回expr2，否则返回expr1
- nullif(expr1, expr2)：如果expr1和expr2相等，则返回null，否则返回expr1
- if(expr1, expr2, expr3)：类似?:三目运算符，如果expr1为true，不等于0，且不等于null，则返回expr2，否则返回expr3
- isnull(expr1)：判断expr1是否为null，为null返回true，否则返回false

case函数

		case value
		when compare_value1 then result1
		when compare_value2 then result2
		...
		else resutl
		end

		case
		when condition1 then result1
		when condition2 then result2
		...
		else result
		end

#### 分组和组函数 ####

- `avg([distinct|all]expr)`：计算多行expr的平均值，expr可以是变量，常量，数据列，但其数据类型必须是数值型，distinct表示不计算重复值
- `count({*|[distinct|all]expr})`：计算多行expr的总条数，可以是任意类型，用星号表示统计该表内的记录行数。null不会被计算在内。distinct和`*`不能同时使用
- `max(expr)`：计算多行expr的最大值，任意数据类型
- `min(expr)`：计算多行expr的最小值，任意数据类型
- `sum([distinct|all]expr)`：计算多行expr的总和，必须是数值类型

使用`group by 列名`对结果进行分组

很多数据库要求：如果查询列表中使用了组函数，或者select语句中使用了group by子句，则要求出现在select列表中的字段，要么使用组函数包起来，要么必须出现在group by子句中。

MySQL没有这个要求，他会显示该组中的第一条记录在此列的值。

对分组过滤使用having子句。where和having的区别在于：

- 不能在where子句中过滤组，where子句仅用于过滤行
- 不能在where子句中使用组函数，having子句中可以

#### 多表连接查询 ####

*SQL 92*

		select column1, column2 ...
		from table1, table2 ...
		[where join_condition]
		# 如果出现相同的列名，则需要在列名前面加上表名或表的别名来区别
		# MySQL不支持SQL 92 中的外连接
		# 自连接就是为同一个表使用不同的别名以利用该表的自关联（同一个表中的不同列之间存在主，外键约束关联）

*SQL 99*

- cross join 交叉连接：SQL 92中的笛卡尔积
- natural join 自然连接：以两个表中的同名列作为连接条件，如果没有同名列，就是笛卡尔积
- using子句连接：显示指定使用哪些同名列作为连接条件
- on子句连接：每个on子句只能指定一个连接条件，如果需要连接n个表，则需要有n-1个join...on对
- 左，右，全外连接：left[outer] join, right[outer] join, full[outer] join 。 SQL 99的外连接和SQL 92的外连接结果相反，99的左连接会把左边表中所有不满足连接条件的记录全部列出，右连接则列出右边表中不符合连接条件的记录。92相反。

#### 子查询 ####

子查询可以出现在2个地方：

- from语句的后面当成数据表，即行内视图，实质是一个临时视图
- where条件后面作为过滤条件

注意点：

- 子查询要用括号括起来
- 把子查询当数据表时，可以为该子查询起别名作为前缀来限定数据列
- 把子查询作为过滤条件时，子查询放在比较运算符的右边
- 把子查询作为过滤条件时，单行子查询使用单行运算符，多行子查询使用多行运算符。
- 如果子查询返回单行单列值，则被当成一个标量值使用。如果子查询返回多个值，则需要使用in，any和all关键字

in关键字可以单独使用，all和any关键字可以和>, >=, <, <=, <>, =等运算符结合使用

理解<any, >any, <all, >all的区别

- <any：小于最大值即可
- >any：大于最小值即可
- <all：小于最小值才行
- >all：大于最大值才行

#### 集合运算 ####

对两个结果集进行集合运算的条件：

- 两个结果集所包含的数据列的数量必须相等
- 两个结果集所包含的数据列的数据类型必须一一对应

*union并运算*：`select 语句 union select 语句`

*minus差运算*：`select语句 minus select语句`

*intersect交运算*：`select语句 intersect select语句`

MySQL不支持intersect运算，但可以使用多表连接查询来实现。如果2个select语句都包括了where条件，那么改写的多表连接查询需要将他们的where条件进行and运算

### JDBC的典型用法 ###

- DriverManager：管理JDBC驱动的服务类，主要功能是获取Connection对象
	- public static synchronized Connection getConnection(String url, String user, String pass) throws SQLException
- Connection接口：数据库连接对象，每个Connection代表一个物理连接对话。
	- Statement createStatement() throws SQLException
	- PreparedStatement prepareStatement(String sql) throws SQLException
	- CallableStatement prepareCall(String sql) throws SQLException
	- Savepoint setSavepoint()
	- Savepoint setSavepoint(String name)
	- void setTransactionIsolation(int level)
	- void rollback()
	- void rollback(Savepoint savepoint)
	- void setAutoCommit(boolean autoCommit)
	- void commit()
	- setSchema(String schema)
	- getSchema()
	- setNetworkTimeout(Executor executor, int milliseconds)
	- getNetworkTimeout()
- Statement：执行SQL语句的工具接口。
	- ResultSet executeQuery(String sql) throws SQLException
	- int executeUpdate(String sql) throws SQLException
	- boolean execute(String sql) throws SQLException
	- closeOnCompletion()
	- isCloseOnCompletion()
	- long executeLargeUpdate()
- PreparedStatement：预编译的Statement对象，是Statement的子接口
	- void setXxx(int parameterIndex, Xxx value)
- ResultSet：结果集对象，可以通过列索引或列名获取列数据
	- void close()
	- boolean absolute(int row)：如果row是负数，则移动到倒数第row行
	- void beforeFirst()
	- boolean first()
	- boolean previous()
	- boolean next()
	- boolean last()
	- void afterLast()
	- <T> T getObject(int columnIndex, Class<T> type)
	- <T> T getObject(String columnLabel, Class<T> type)

#### JDBC编程步骤 ####

1. 加载数据库驱动，通常使用Class类的forName()静态方法来加载
		
		Class.forName(driverClass);
		Class.forName("com.mysql.jdbc.Driver");

2. 通过DriverManager获取数据库连接。

		DriverManager.getConnection(String url, String user, String pass);
		# mysql url
		jdbc:mysql://hostname:port/databasename
		# oracle url
		jdbc:oracle:thin:@hostname:port:databasename
	
3. 通过Connection对象创建Statement对象

		createStatement();
		prepareStatement();
		prepareCall();

4. 使用Statement执行SQL语句

		execute();
		executeUpdate();
		executeQuery();

5. 操作结果集

		next(); previous(); beforeFirst(); afterLast(); first(); last(); absolute();
		getXxx();

6. 回收数据库资源，包括关闭ResultSet，Statement和Connection等资源

### 执行SQL语句的方式 ###

`executeUpdate()`和`executeQuery()`执行DDL语句返回0，执行DML语句返回受影响的记录条数。

使用`execute()`方法执行SQL语句的返回值只能是boolean值。然后使用`getResultSet()`获取返回的ResultSet对象。

ResultSet的getString()方法几乎可以获取除Blob之外的任意类型列的值。

推荐使用带占位符(?)参数的SQL语句

		insert into student_table values(null, ?, ?);

如果程序清楚PreparedStatement预编译SQL语句中各参数的类型，则使用相应的setXxx()方法来传入参数即可，如果程序不清楚预编译SQL语句中各参数的类型，则可以使用setObject()方法来传入参数，由PreparedStatement负责类型转换。

- PreparedStatement预编译SQL语句的性能更好
- PreparedStatement无须拼接SQL语句，编程更简单
- PreparedStatement可以防止SQL注入，安全性更好

占位符参数只能代替普通值，不能使用占位符参数代替表名，列名等数据库对象，也不要使用占位符代替SQL语句中的insert，select等关键字。

### 管理结果集 ###

以默认方式打开的ResultSet是不可更新的，必须在创建Statement或PreparedStatement时传入额外的参数来创建可更新的ResultSet。

- resultSetYpe：控制ResultSet的类型
	- ResultSet.TYPE_FORWARD_ONLY：记录指针只能向前移动
	- ResultSet.TYPE_SCROLL_INSENSITIVE：记录指针可以自由移动，但底层数据的改变不会影响ResultSet的内容
	- ResultSet.TYPE_SCROLL_SENSITIVE：记录指针可以自由移动，且底层数据的改变会影响ResultSet的内容
- resultSetConcurrency：控制ResultSet的并发类型
	- ResultSet.CONCUR_READ_ONLY：只读，不可更新
	- ResultSet.CONCUR_UPDATABLE：可更新，但需要满足下面两个条件：
		- 所有数据都应该来自一个表
		- 选出的数据集必须包含主键列

可以调用可滚动，可更新的ResultSet的updateXxx(int columnIndex, Xxx value)方法来修改记录指针所指记录，特定列的值，然后调用ResultSet的updateRow()方法来提交修改。

java8添加了updateObject(int columnIndex, Object x, SQLType targetSqlType)和updateObject(String columnLabel, Object x, SQLType targetSqlType)两个默认方法。

#### 处理Blob类型数据 ####

必须使用PreparedStatement来往数据库中插入Blob数据：`setBinaryStream(int parameterIndex, InputStream x)`

从ResultSet中取出Blob数据时，调用ResultSet的getBlob(int columnIndex)方法取出Blob对象，然后调用Blob对象的getBinaryStream()来获取该Blob数据的输入流，也可以调用该Blob对象的getByte()方法来取出封装的二进制数据

#### 使用ResultSetMetaData分析结果集 ####

调用ResultSet的getMetaData()方法返回该ResultSet对应的ResultSetMetaData对象，然后可以使用该对象获取ResultSet的描述信息，常用的有一下3个：

- int getColumnCount()：返回该ResultSet的列数量
- String getColumnName(int column)：返回指定索引的列名
- int getColumnType(int column)：返回指定索引的列类型

### Java的RowSet ###

RowSet接口继承了ResultSet接口，子接口包括JdbcRowSet, CachedRowSet, FilteredRowSet, JoinRowSet和WebRowSet。除了JdbcRowSet外都是离线RowSet，无须保持与数据库的连接。

RowSet是可滚动，可更新，可序列化的结果集，并且作为JavaBean使用。 

#### Java 7 新增的RowSetFactory与RowSet ####
Java 7 中新增了RowSetProvider类和RowSetFactory接口，RowSetProvider负责创建RowSetFactory，RowSetFactory负责创建RowSet实例：

- CachedRowSet createCachedRowSet()
- FilteredRowSet createFilteredRowSet()
- JdbcRowSet createJdbcRowSet()
- JoinRowSet createJoinRowSet()
- WebRowSet createWebRowSet()

使用RowSetFactory创建的RowSet实例没有装填数据，必须设置相关信息

- setUrl(String url)
- setUsername(String name)
- setPassword(String password)
- setCommand(String sql)
- execute()

#### 离线RowSet ####

1. 获取ResultSet对象
2. 创建CachedRowSet对象
3. 使用CachedRowSet包装ResultSet对象，RowSet对象的populate(ResultSet rs)
4. 关闭Connection对象
5. 操作CachedRowSet对象
6. 重新建立连接
7. 接受CachedRowSet对象的修改,RowSet对象的acceptChanges()方法。

在包装ResultSet对象前应该先设置CachedRowSet对象的pageSize。

### 事务处理 ###

事务：一步或几步数据库操作序列组成的逻辑执行单元，要么全部执行，要么全部放弃执行。具备4个特性：ACID

- 原子性（Atomicity）：事务是应用中最小的执行单位
- 一致性（Consistency）：事务执行的结果必须使数据库从一个一致性状态，变到另一个一致性状态。
- 隔离性（Isolation）：各个事务的执行互不干扰
- 持续性（Durability/Persistence）：事务一旦提交，对数据所做的任何改变都要记录到永久存储器中。

事务由：

- 0到多条DML语句
- 1条DDL语句或DCL语句，最多只能有一条，因为会导致事务立即提交。

提交有2种：

- 显式提交：使用commit
- 自动提交：执行DDL或DCL语句

回滚（rollback）有两种：

- 显式回滚：使用rollback
- 自动回滚：系统错误或强行退出


`set autocommit = (0|1) 0为关闭自动提交，即开启事务`

一个MySQL的命令行窗口代表一次连接session，在该窗口里设置autocommit=0， 只是关闭了该连接session的自动提交，不会影响其他的连接。

临时开启事务： 使用`start transaction`或`begin`，后面紧跟的DML语句不会立即生效，直到显示提交或自动提交。

设置回滚中间点：`savepoint a`；回滚到指定中间点：`rollback a`

提交和不使用savepoint的回滚都会结束当前事务，使用savepoint的回滚不会结束当前事务。

在JDBC中使用`conn.setAutoCommit(false|true)`来设置自动提交。

当Connection遇到未处理的SQLException异常时，系统会非正常退出，事务也会自动回滚，但如果使用catch块捕捉了该异常，则需要在异常处理块中显式地回滚。

Connection设置中间点的方法：

- Savepoint setSavepoint()：匿名中间点
- Savepoint setSavepoint(String name)：有名字的中间点，其实没有必要设置名字，因为回滚时不是根据名字，而是根据指定的Savepoint对象：`rollback(Savepoint savepoint)`

使用Statement对象的addBatch()方法添加批量操作的SQL语句，然后执行executeBatch()或executeLargeBatch()方法来实现批量更新。
批量更新不支持select查询语句。返回的是一个long[]数组，每个数值是对应sql语句的返回值。

程序应该在开始批量操作之前先关闭自动提交，然后开始添加更新语句，当批量操作执行结束后，提交事务，并恢复之前的自动提交模式。

### 分析数据库信息 ###

CRUD操作：Create增加，Retrieve读取，Update更新，Delete删除

#### 使用DatabaseMetaData分析数据库信息 ####

通过Connection对象的getMetaData()方法获取所连接数据库对应的DatabaseMetaData对象

DatabaseMetaData的很多方法需要传入一个xxxPattern模式字符串，这里的xxxPattern不是正则表达式，而是SQL里的模式字符串，用百分号`%`代表任意多个字符串，用下划线`_`代表一个字符。如果把该模式字符串参数值设置为null，表明该参数不作为过滤条件。

#### 使用系统表分析数据库信息 ####

用户只能参看系统表的数据，不能直接修改。

- 如果需要获得数据库信息，如数据库驱动提供了哪些功能，或者需要查询与JDBC驱动有关的信息，应该利用DatabaseMetaData；
- 如果仅仅纯粹地分析数据库的静态对象，如包含多少数据库，数据表，视图，索引等，则利用系统表更合适

可以通过DatabaseMetaData的getDatabaseProductName(), getDatabaseProductVersion()方法来获得底层数据库系统的名称和版本号，也可以使用getDriverName(), getDriverVersion()方法获得驱动程序的名称和版本号。

### 使用连接池管理连接 ###

为了避免频繁地打开/关闭数据库连接，可以使用数据库连接池方案：
当应用程序启动时，系统主动建立足够的数据库连接，并将这些连接组成一个连接池。每次程序请求数据库连接时，无须重新打开连接，而是从连接池中取出已有连接使用，使用完后不再关闭连接，而是讲连接归还给连接池。

JDBC的数据库连接池使用javax.sql.DataSource接口来表示，通常由应用服务器来提供实现。

DataSource通常被称为数据源，包含连接池和连接池管理两个部分。

数据源无须创建多个，它仅仅时产生数据库连接的工厂，因此整个应用只需要一个数据源即可。建议将其设为static成员变量，并在应用开始时立即初始化数据源对象。通过数据源获得的数据库连接执行close()时并不会真的关闭连接，而是将其释放回归到连接池中。

## Chapter 14 Annotation(注解) ##

**APT(Annotation Processing Tool)**： 访问和处理Annotation的工具

5个基本的Annotation：

- @Override
- @Deprecated
- @SuppressWarnings
- @SafeVarargs
- @FunctionalInterface

#### @Override ####

只能用于修饰子类的方法，强制必须满足以下2个条件，否则编译出错

1. 父类必须有此方法
2. 子类方法必须重写覆盖父类的方法

#### @Deprecated ####

标识修饰的类，方法等已经过时，如果被使用，编译器将报警

#### @SuppressWarnings ####

关闭编译器警告，在`@SuppressWarnings`的后面加括号，里面使用`value="警告类型"`来指定关闭警告类型

    @SuppressWarnings(value="unchecked")

#### @SafeVarargs ####

**堆污染(Heap pollution)**：当把一个不带泛型的对象赋给一个带泛型的变量时，会造成堆污染，因为该对象内的数据类型可能与该变量指定的泛型类型不相符

通过@SafeVarargs修饰方法或构造器可以关闭该方法或构造器的堆污染警告

#### @FunctionalInterface ####

只能用于修饰接口，表明该接口为函数式接口

### JDK的元Annotation ###

#### @Retention ####
只能修饰其他Annotation定义，用于指定被修饰的Annotation可以保留多长时间。
包含一个`RetentionPolicy`的value成员变量，使用@Retention时必须指定其值为以下三者之一：

- `RetentionPolicy.CLASS`： 编译器把Annotation记录在class文件中。但运行时JVM不能获取Annotation信息，默认值；
- `RetentionPolicy.RUNTIME`： 同上，但运行时JVM可以通过反射获取Annotation信息
- `RetentionPolicy.SOURCE`： Annotation只保存在源代码中，编译器不记录。

如果某个Annotation只需要为value成员变量指定值，则可以使用简化格式，在括号中只写入其值即可，不用使用`value=值`格式

#### @Target ####
只能修饰其他Annotation定义，用于指定被修饰的Annotation能用于修饰哪些元素。
包含一个`ElementType`的value成员变量，使用时必须指定为以下值的组合：

- `ElementType.ANNOTATION_TYPE`： 只能修饰其他Annotation
- `ElementType.CONSTRUCTOR`： 只能修饰构造器
- `ElementType.FIELD`： 只能修饰成员变量
- `ElementType.LOCAL_VARIABLE`： 只能修饰局部变量
- `ElementType.METHOD`： 只能修饰方法定义
- `ElementType.PACKAGE`： 只能修饰包定义
- `ElementType.PARAMETER`： 可以修饰参数
- `ElementType.TYPE`： 可以修饰类，接口，其他注解或枚举定义

#### @Documented ####
只能修饰其他Annotation，用于表明被他修饰的Annotation一旦修饰了程序元素，将出现在该元素的API文档中


#### @Inherited ####
只能修饰其他Annotation，被修饰的Annotation在使用时自动被其修饰的类型的子类所继承。

### 自定义Annotation ###

- **标记Annotation**： 没有定义成员变量的Annotation
- **元数据Annotation**： 包含成员变量的Annotation

    public @interface MyAnnotation {
    	String name() default "Martin";
    	int age() default 30;
    }

使用自定义Annotation时，有默认值的成员变量可以不用显式指定值。

Java使用java.lang.Annotation接口来代表所有的注解，它是所有注解的父接口。

java.lang.reflect.AnnotatedElement接口的实现类（Class，Constructor，Filed，Method，Package）都有相应的方法调取相关的（包含或者修饰自己）Annotation的信息

#### Java8重复注解@Repeatable ####

从Java8开始，同一种注解可以使用简化格式修饰同一个元素多次。步骤：

1. 正常定义可多次使用的注解A
2. 定义一个注解B作为注解A的容器，注解B包含一个类型为A[]的value成员变量
3. 在注解A的定义前加上`@Repeatable(B.class)`

作为容器的注解B的保留期(Retention)必须比注解A的长，否则编译器报错

#### Java8添加的Type Annotation ####
- `ElementType.TYPE_PARAMETER`
- `ElementType.TYPE_USE`

**类型注解(Type Annotation)**： 使用`@Target(ElementType.TYPE_USE)`修饰的注解

Java8以前，注解只能用于各种程序元素的定义；Java8开始，类型注解可以用于任何用到类型的地方，比如：

- 使用`new`关键字创建对象
- 类型转换
- 使用`implement`实现接口
- 使用`throws`抛出异常



		@Target(ElementType.TYPE_USE)
	    @interface NotNull{}
	    
	    @NotNull
	    public class TypeAnnotationTest implements @NotNull Serializable { //implements 使用Type Annotation
	    		//方法形参中使用Type Annotation
	    	public static void main(@NotNull String[] args) throws @NotNull FileNotFoundException { //throws时使用Type Annotation
	    		Object obj = "java.org";
	    		//强制类型转换时使用Type Annotation
	    		String str = (@NotNull String)obj;
	    		//创建对象时使用Type Annotation
	    		Object win = new @NotNull JFrame("Crazy Software");
	    	}
	    	//泛型中使用Type Annotation
	    	public void foo(List<@NotNull String> info) {}
	    }

开发者需要自己实现Type Annotation检查框架

#### 编译时处理Annotation ####

每个Annotation处理器都必须实现`javax.annotation.processing`包中的Processor接口，通常会采用继承`AbstractProcessor`的方式来实现Annotation处理器

## Chapter 15 输入/输出 ##

File类不能访问文件内容本身，需要使用输入/输出流。

`getAbsoluteFile`和`getAbsolutePath`的区别在于前者返回的是File对象，后者返回的是String对象。

Java支持使用`/`作为平台无关的路径分隔符，否则在windows平台下必须使用两条`\`来作为路径分隔符， 如`d:\\temp\\tmp.txt`

- **输入流：**只能读数据，不能写数据
- **输出流：**只能写数据，不能读数据

输入/输出流是从程序运行所在的内存角度判断，例如，内存写到硬盘是输出流。

- **字节流：**操作的数据单元是8位的字节
- **字符流：**操作的数据单元是16位的字符

字节流以InputStream和OutputStream作为基类，字符流则以Reader和Writer作为基类。

- **节点流（低级流/low level stream）：**可以从/向一个特定的IO设备读/写数据的流
- **处理流（高级流/high level stream）：**对一个已存在的流进行连接或封装，通过封装后的流来实现数据的读/写功能

处理流的功能：
- 提高性能：主要以增加缓冲的方式提高输入/输出的效率
- 方便操作：提供一系列便捷方法来操作大量内容

### InputStream & Reader ###
- `read()` 返回所读取的字节/字符数据
- `read(byte[] b)`/`read(char[] cbuf)` 返回实际读取的字节数/字符数
- `read(byte[] b, int off, int len)`/`read(char[] cbuf, int off, int len)` 返回实际读取的字节数/字符数
- 如果返回-1，说明到达输入流的终点

### OutputStream & Writer ###
Writer 除了可以使用字符数组作为`write()`方法的参数，还可以使用字符串作为参数，即：`write(String s)`和`write(String s, int off, int len)`

通过java的IO流执行输出时，不要忘记关闭输出流，这样除了可以保证流的物理资源被回收之外，可能还可以将输出流缓冲区中的数据flush到物理节点中，因为在执行` `前会自动执行输出流的`flush()`方法


### 输入/输出流体系 ###

- 处理流的构造器参数都是已存在的流（可以是节点流或其他处理流）
- 节点流的构造器参数是物理IO节点

如果需要输出文本内容，都应该将输出流包装成PrintStream后输出

在使用处理流包装了底层节点流之后，关闭输入/输出流资源时，只需关闭最外层的处理流即可，系统会自动迭代关闭所有被包装的流。

分类			|字节输入流					|字节输出流					|字符输入流				|字符输出流
------------|---------------------------|---------------------------|-----------------------|----------------
抽象基类		|*InputStream*				|*OutputStream*				|*Reader*				|*Writer*
访问文件		|**FileInputStream**		|**FileOutputStream**		|**FileReader**			|**FileWriter**
访问数组		|**ByteArrayInputStream**	|**ByteArrayOutputStream**	|**CharArrayReader**	|**CharArrayWriter**
访问管道		|**PipedInputStream**		|**PipedOutputStream**		|**PipedReader**		|**PipedWriter**
访问字符串	|							|							|**StringReader**		|**StringWriter**
缓冲流		|BuffedInputStream			|BufferedOutputStream		|BufferedReader			|BufferedWriter
转换流		|							|							|InputStreamReader		|OutputStreamWriter
对象流		|ObjectInputStream			|ObjectOutputStream		
过滤抽象基类	|*FilterInputStream*		|*FilterOutputStream*		|*FilterReader*			|*FilterWriter*
打印流		|							|PrintOutputStream			|						|PrintWriter
推回输入流	|PushbackInputStream		|							|PushbackReader	
特殊流		|DataInputStream			|DataOutputStream		

经常把读取文本内容的输入流包装成`BufferedReader`，用来方便读取输入流的文本内容

### 推回输入流 ###

`PushbackInputStream`和`PushbackReader`都带有一个推回缓冲区，当程序调用这两个流的`unread()`方法时，系统将会把指定数组的内容推回到缓冲区里，而推回输入流每次调用`read()`方法时总是先从推回缓冲区读取，只有在完全读取了推回缓冲区的内容后仍然没有装满`read()`所需数组时才会从原输入流中读取。

如果程序中推回到推回缓冲区的内容超出了缓冲区的大小，将会引发`Pushback buffer overflow`的IOException异常。

### Java虚拟机读写其他进程的数据 ###

`Process`类的三个数据通信的方法：

- `InputStream getErrorStream()`：获取子进程的错误流，程序的输入流，子进程---数据--->程序
- `InputStream getInputStream()`：获取子进程的输出流，程序的输入流，子进程---数据--->程序
- `OuputStream getOutputStream()`：获取子进程的输入流，程序的输出流，子进程<---数据---程序

### RandomAccessFile ###

如果需要访问文件部分内容或在文件后追加内容，应该使用`RandomAccessFile`


`RandomAccessFile`只能读写文件，不能读写其他IO节点

`RandomAccessFile`的访问模式：

- "r"： 以只读方式打开文件，执行写入操作将抛出IOException
- "rw"： 以读写方式打开文件，如果该文件不存在，将尝试创建该文件
- "rws"： 以读写方式打开文件，还要求对文件的内容或元数据的每个更新都同步写入到底层存储设备
- "rwd"： 以读写方式打开文件，还要求对文件的内容的每个更新都同步写入到底层存储设备

`RandomAccessFile`不能直接往文件中间插入新的内容，因为新插入的内容会覆盖原有内容，如果需要插入新的内容，则要把该位置后的内容存入到缓冲区，在写完新内容之后，再从缓冲区中读入原来的内容追加在新内容之后。

### 对象序列化 ###

**对象的序列化Serialize**：将一个Java对象写入IO流中
**对象的反序列化Deseialize**：从IO流中恢复该Java对象

可序列化的类必须实现如下2个接口之一：

- `Serializable`
- `Externalizable`

所有可能在网络上传输或需要存储在磁盘里的对象的类都必须是可序列化的。

程序创建的每个JavaBean类都实现`Serializable`

实现了`Serializable`接口的类的序列化：

1. 使用其他节点流创建一个`ObjectOutputStream`，这是一个处理流
2. 调用`ObjectOutputStream`的`writeObject()`方法输出可序列化对象

实现了`Serializable`接口的类的反序列化：

1. 使用其他节点流创建一个`ObjectInputStream`，这是一个处理流
2. 调用`ObjectInputStream`的`readObject()`方法读取可序列化对象，返回一个`Object`对象，如果知道该对象的实际类型，可以使用强制类型将其转换成真实类型

反序列化读取的仅仅是Java对象的数据，而不是Java类，所以反序列化时必须提供该对象所属类的class文件，否则会引起`ClassNotFoundException`异常

反序列化机制无须通过构造器来初始化Java对象

如果序列化了多个Java对象到一个文件，反序列化时必须按照实际写入的顺序读取

当一个可序列化类有多个父类时，这些父类必须满足以下条件之一:

- 父类也是可序列化的
- 父类有无参数的构造器，如果不是可序列化的，则父类的成员变量值不会序列化到二进制流中

可序列化的类的成员变量必须都是可序列化的，即基本类型，String或者其他可序列化的类

Java的序列化算法：

- 使用序列化了对象都会有一个序列化编号
- 当序列化一个对象时，程序会检查该对象是否第一次被序列化，如果是，则将其转换成字节序列并赋予一个序列化编号
- 如果该对象曾经被序列化，则输出该对象的序列化编号，而不是再次重新序列化

因此，如果一个对象的某个成员变量的值在该对象被序列化后发生改变，即使再次执行`writeObject()`方法，该变化也不会被记录，在反序列化时读取出来的依然是改变前的原值

**递归序列化**：序列化对象时，所有的成员变量都会被序列化，以此类推

`transient`关键字只可用于实例变量，不可修饰其他成分

如果需要在序列化某类对象时对数据特殊处理，可以在该类中提供特殊签名的方法：

- `private void writeObject(java.io.ObjectOutputStream out) throws IOException`
- `private void readObject(java.io.ObjectInputStream in)throws IOException, ClassNotFoundException`
- `private void readObjectNoData() throws ObjectStreamException`

`writeObject()`方法和`readObject()`方法对对象的处理顺序必须保持一致

可以用另一个对象来完全替代某个可序列化对象：

    ANY-ACCESS-MODIFIER Object writeReplace() throws ObjectStreamException
    ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException

在序列化对象时，系统会递归调用`writeReplace()`方法，直到所引用对象不再具有`writeReplace()`方法，此时调用该对象的`writeObject()`方法输出对象。

`readResolve()`方法会紧接着`readObject()`方法之后被调用，该方法的的返回值会取代反序列化恢复的对象，而恢复的对象将被抛弃。该方法在序列化单例类，枚举类时尤其有用。

`final`类重写`readResolve()`方法没有问题，否则应该尽量使用`private`修饰该方法

使用`Externalizable`接口的类必须提供一个无参数的构造器，否则编译出错：`no valid construction`

`Externalizable`接口包含方法：

- `void readExternal(ObjectInput in)`
- `void writeExternal(ObjectOutput out)`

对象的类名，实例变量（包括基本类型，数组，对其他对象的引用）都会被序列化；方法，类变量，`transient`实例变量都不会被序列化

实现`Serializable`接口的类使用`transient`关键字来屏蔽不想被序列化的实例变量，不要使用`static`来实现此功能

反序列化对象时必须有序列化对象的class文件

读取被序列化的对象的顺序必须与实际写入的顺序一致

可序列化类的版本：

    private static final long serialVersionUID

可以用`serialver ClassName`来查看该类的版本值

类的修改对反序列化的影响：

- 如果仅仅修改了方法，反序列化不受影响，无须修改`serialVersionUID`的值
- 如果仅仅修改了静态变量或瞬态实例变量，反序列化不受影响，无须修改`serialVersionUID`的值
- 如果修改前后的类包含同名的实例变量，但是类型不同，则反序列化失败，应该更新`serialVersionUID`的值
- 如果修改前的类包含更多的实例变量，则多出的实例变量被忽略，序列化版本可以兼容，可以不更新`serialVersionUID`的值
- 如果修改后的类包含更多的实例变量，则序列化版本可以兼容，可以不更新`serialVersionUID`的值，但反序列化得到的对象中多出的实例变量值都是缺省值。


### NIO ###

- **通道(Channel)**：在新IO系统中所有的数据都必须通过通道传输，与传统InputStream，OutputStream最大的区别是它提供了一个`map()`方法，可以把一块数据映射到内存中
- **缓冲(Buffer)**：本质是一个数组，发送到Channel中的所有的对象都必须放在Buffer中，而从Channel中读取的数据也必须先放到Buffer中

获得Buffer对象的方法：

	static XxxBuffer allocate(int capacity)

- **容量(capacity)**： 表示该buffer的最大数据容量，即最多可以存储多少数据，不可能为负值，创建后不能改变
- **界限(limit)**： 第一个不该被读出或写入的缓冲区位置索引，位于limit后的数据既不可被读，也不可被写
- **位置(position)**： 下一个可以被读出或者写入的缓冲区位置索引，当使用buffer从channel中读取数据时，position的值=已读数据的数量，但指向下一个buffer的位置，参考数组的索引。

	0<=mark<=position<=limit<=capacity

如果position或者limit<mark，则mark会被抛弃

当buffer装入数据结束后，调用buffer的`flip()`方法，该方法将limit设置为position的位置，并将position归零，从而使buffer做好输出数据的准备；当bufffer输出数据结束后，buffer调用`clear()`方法，该方法并不清空buffer的数据，仅仅将position归零，将limit设为capacity，这样为再次向buffer中装入数据做好准备。

使用`put()`和`get()`访问buffer中的数据时，分为相对和绝对：

- 相对：从buffer的当前position处开始读取或写入数据，然后将position的值按照处理元素的个数增加。
如果要求的数据超出边界
	- `get()`抛出`BufferUnderflowException`异常
	- `put()`抛出`BufferOverflowException`异常
- 绝对： 直接根据索引向buffer中读取或写入数据，并不会影响position的值。如果index非法，则抛出`IndexOutofBoundsException`异常

只有`ByteBuffer`才能创建直接buffer，而且直接buffer只适用于长生存期的buffer，不适用于短生存期的buffer

Channel与传统IO区别：

- Channel可以将制定文件的部分或全部直接映射成Buffer
- Channel只能与Buffer交互，程序不能直接访问Channel中的数据，

所有的Channel都不应该通过构造器来直接创建，而是通过传统的节点`InputStream`，`OutputStream`的`getChannel()`方法来获取相应的Channel。

Channel中最常用的3个方法：

- `map()`：将Channel对应的数据映射成`ByteBuffer`
- `read()`及其重载：从Buffer中读取数据
- `write()`及其重载：向Buffer中写入数据

    MappedByteBuffer map(FileChannel.MapMode mode, long position, long size)

`FileOutputStream`返回的`FileChannel`只能写，`FileInputStream`返回的`FileChannel`只能读，`RandomAccessFile`返回的`FileChannel`得读写取决于`RandomAccessFile`打开文件的模式

`Charset`类不可变

**编码(Encode)**：把明文字符序列转换为二进制序列
**解码(Decode)**：把二进制序列转换为明文字符序列

使用`FileChannel`中的`lock()`和`trylock()`方法获得`FileLock`对象，两个方法的区别是：如果不能获得文件锁，`lock()`方法会阻塞程序，`trylock()`方法会返回null

锁定部分内容：

- `lock(long position, long size, boolean shared)`
- `trylock(long position, long size, boolean shared)`

`shared`为true，表明该锁是一个共享锁，允许多个进程读取该文件，但阻止其他进程获得该文件的排它锁；
`shared`为false，表明该锁是一个排它锁，将锁住对该文件的读写

无参数的`lock()`和`trylock()`方法获取的都是排它锁

### Java7的NIO.2 ###

`getNameCount()`方法返回Path路径所包含的路径名的数量，也就是路径的深度

#### 使用`FIleVisitor`遍历文件和目录： ####

    walkFileTree(Path start, FileVisitor<? super Path> visitor)
    walkFileTree(Path start, Set<FileVisitOption> options, int maxDepth, FileVisitor<? super Path> visitor)
    //遍历文件盒子目录会触发FileVisitor中相应的4中方法之一
    FileVisitResult postVisitDirectory(T dir, IOException exc)  //访问子目录后触发
    FileVisitResult preVisitDirectory(T dir, IOException exc)  //访问子目录前触发
    FileVisitResult visitFile(T file, BasicFileAttributes attrs)  //访问文件时触发
    FileVisitResult visitFileFailed(T file, IOException exc)  //访问文件失败时触发

实际编程时可以通过继承`SimpleFileVisitor`来实现自己的文件访问器

#### 使用WatchService监控文件变化 ####
watcher用于监听变化，每个变化对应一个event，WatchKey相当于在监听但未查询期间所发生的event集合。
类似于：watcher相当于监听记录本，WatchKey就是监听记录本上的未读记录，event就是监听记录本上的每条记录，
    register(WatchService watcher, WatchEvent.kind<?>... events)
    //使用WatchService的三个方法来获得被监听的目录的文件变化事件
    WatchKey poll()  //获取下一个WatchKey, 没有则立刻返回null
    WatchKey poll(long timeout, TimeUnit unit)  //尝试等待timeout时间去获取下一个WatchKey
    WatchKey take()  //获取下一个WatchKey，没有则一直等待
如果程序需要一直监控，应使用`take()`方法；如果只需要监控指定时间，则使用`poll()`方法




