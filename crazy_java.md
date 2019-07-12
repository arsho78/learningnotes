## Chapter 3 数据类型和运算符 ##

javadoc工具默认只处理*public*或*protected*修饰的类, 接口, 方法, 成员变量, 构造器和内部类之前的文档注释。

只有对处于多个包下的源文件生成API文档时，才会有概述页面。

常见的javadoc标识：

- @author
- @version
- @deprecated
- @param
- @return
- @see
- @exception
- @throws

javadoc工具默认不会处理@author和@version标识，需要在执行javadoc命令时显式加入这些选项

API文档中的包描述不是直接放在源文件中，而是单独的一个HTML5文件，通常称为包描述文件，文件名通常是package.html，并与该包的所有Java源文件放在一起。

分号; 花括号{} 方括号[] 圆括号() 逗号, 圆点. 空格 统称为分隔符

protected定义的成员变量被子类访问时有如下限制：

- 当子类和父类在同一包中时，protected相当于public
- 当子类和父类不在同一包时，在子类内部可以直接使用父类的protected成员，或者通过子类自身实例访问继承自父类的protected成员，但是不能通过父类的实例来访问父类的protected成员。

java的标识符: 字母, 数字(不能打头), 下划线(`_`不能单独使用), 美元符($).
不能包含空格
java的关键字都是小写

java语句可以跨行书写，但是字符串和变量名不能跨行

java有50个关键字和保留字

Java中每个变量和每个表达式都有一个在编译期就确定的类型。

Java的数据类型分类：

- 基本类型(primitive type)： 
	- 数值类型
		- 整型：
			- byte: -128~127 (1 byte, 8 bits)
			- short: -32768~32767 (2 bytes, 16 bits)
			- int: -2,147,483,648~2,147,483,647 (4 bytes, 32 bits)
			- long: -2^31~2^31-1 (8 byt^s, 64 bits)
		- 字符型(char):(2 bytes, 16 bits)
		- 浮点型：
			- float: (4 bytes, 32 bits)
			- double: (8 bytes, 64 bits)
	- 布尔类型(boolean)
- 引用类型(reference type)
	- 类
	- 接口
	- 数组
	- null type

整数表示法：

- 二进制：0b/0B开头
- 八进制：0开头
- 十六进制：0x/0X开头

java使用int作为整型的默认类型,超出int范围时必须在整数的后面添加l或L来表示长整型.
当数值处于合法范围内,整数可以直接赋值给byte,short;
当数值处于int范围内,可以直接赋值给long;
当超出int范围,必须在数字后添加l或L来赋值给long.

正数的补码与原码相同,负数的补码是反码+1,符号位保持不变
根据负数的补码求值方法（举例用-5）：
1. 补码-1，取反，求值，ex： ...11011, -1得11010，取反得00101，求值得5，所以是-5
2. 补码取反，求值，+1， ex： ...11011, 取反得00100，求值得4，+1得5，所以是-5

所有char类型的值都可以参与算术运算，其值就是字符对应的unicode编号，范围0~65535

只有浮点型的数值才能使用科学计数法表示

Java中浮点类型的默认类型是double

浮点类型的特殊值：

- 正无穷大(POSITIVE_INFINITY)：正数除以0.0，都一样大
- 负无穷大(NEGATIVE_INFINITY)：负数除以0.0，都一样大
- 非数(NaN)：0.0除以0.0， 或者对负数开方，或者有浮点数参与的对0取余，与任何数值都不等，包括NaN本身。

只有浮点数除0会得出正无穷大和负无穷大，整数除0会抛出ArithmeticException:/by zero

表达式类型的自动提升

- 所有的byte，short，char都会被提升到int；
- 整个算数表达式的数据类型自动提升到与表达式中最高等级的操作数同样的类型。

支持直接量的数据类型：

- 基本类型
- 字符串类型
- null type

由多个字符串常量连接形成的字符串也是字符串常量，存储在常量池中

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
		
等于比较符：数值类型操作数即使不是同一类型，只要值相等就返回true；两个引用类型之间如果没有父子继承关系，就不能用==或!=比较

逻辑运算符的操作数必须是布尔型的变量或常量。
短路与和或会根据前面表达式的结果来判断是否执行后续表达式，非短路与和或一定会执行所有的表达式
异或逻辑运算符：相同为false，不同为true
异或位运算符：相同为0，不同为1

三目运算符不支持多个语句

使用if条件控制时，范围较小的条件优先处理

P69 表3.4 运算符优先级

## Chapter 4 流程控制与数组 ##

使用if..else语句时总是优先把包含范围小的条件放在前面处理

switch语句后面的控制表达式的数据类型只能是byte，short，char，int四种整数类型，枚举类型和String类型，不能是boolean；case后面只能跟常量；一旦遇到相等的值就会执行以后的代码，不再判断与后面的case，default标签的条件是否相等，除非碰到break才会结束。

while后面仅跟一个分号表明循环体是空循环体，只有一个分号。此循环也是一个死循环，因为没有迭代语句改变循环条件。

do..while循环条件后面必须有一个分号，表示循环结束

for循环的迭代语句没有像while和do...while放在循环体内，一旦出现在continue语句后就可能不会执行而陷入死循环，for语句的迭代语句在括号内一定会被执行。

for循环可以初始化多个变量但只能有一个声明语句，因此这些变量都必须是同一种数据类型。

for循环括号内的三个语句都可以为空，也可以把初始化语句和迭代语句分别放在循环之前和循环体内。

for循环括号内只有2个分号是必须的，其他都是可以省略的。

在for循环体中不要直接修改循环变量的值，应该重新定义一个临时变量，将循环变量的值赋给临时变量或者循环之前就已定义的变量，再修改这些变量的值。

java中的标签只有放在循环语句之前才有作用。并且在break所在循环的外层循环之前定义才有意义。

数组本身也是一种引用类型。定义数值时不能指定数组的长度。

数组的静态初始化是指初始化数组时指定数组元素值。
只有在定义数组的同时执行数组初始化才支持使用简化的静态初始化。
不能同时使用静态初始化和动态初始化，即不能即指定长度也分配初始值。

使用foreach循环迭代访问数组元素时，并不能改变数组元素的值。其循环变量只是一个与数组元素值相同的临时变量，并不是数组元素本身。

所有在方法中定义的局部变量（相当于指针）都保存在栈内存中，对象则保持在堆内存中。

多维数组的维数层级是从左到右逐渐加深的，例如int[3][4]是一个长度为3的，元素类型为int[4]的数组，初始化多维数组时，可以只指定最左边维的长度，或者所有维的长度。

常用的java.util.Arrays类的方法：

- int binarySearch(type[] a, type key)
- int binarySearch(type[]a, int fromIndex, int toIndex, type key)
- type[] copyOf(type[] original, int length)
- type[] copyOfRange(type[] original, int from, int to)
- boolean equals(type[] a, type[] a2)
- void fill(type[] a, type val)
- void fill(type[] a, int fromIndex, int toIndex, type val)
- void sort(type[] a)
- void sort(type[] a, int fromIndex, int toIndex)
- String toString(type[] a)
- void parallePrefix(xxx[] array, XxxBinaryOperator op)
- void parallePrefix(xxx[] array, int fromIndex, int toIndex, XxxBinaryOperator op)
- void setAll(xxx[] array, IntToXxxFunction generator)
- void paralleSetAll(xxx[] array, IntToXxxFunction generator)
- void paralleSort(xxx[] a)
- void paralleSort(xxx[] a, int fromIndex, int toIndex)
- Spliterator.OfXxx spliterator(xxx[] array)
- Spliterator.OfXxx spliterator(xxx[] array, int startInclusive, int endExclusive)
- XxxStream stream(xxx[] array)
- XxxStream stream(xxx[] array, int startInclusive, int endExclusive)



## Chapter 5 面向对象（上） ##

在多个构造器中重复出现的初始化代码可以提取出来放在初始化代码块中，初始化代码块总是在构造器执行前被调用。

		[修饰符] class 类名 {
			[修饰符] 类名(构造器形参列表) {}
			...
			[修饰符] 类型 成员变量名 [= 默认值];
			...
			[修饰符] 返回值类型 方法名(形参列表) {}
			...
		}

Java类名通则：每个单词首字母大写，其他字母小写，单词之间没有分隔符。

static修饰的成员不能访问非static的成员。

如果没有手工添加构造器，系统自动为该类提供一个默认的无参数构造器。

成员变量名通则：第一个单词首字母小写，其余单词首字母大写，其他字母小写，单词间不使用间隔符。 

变量存放在栈内存中，实际对象存放在堆内存中

`this`所指代的的对象在编译期无法确定，只能确定类型。

static修饰的方法中不能使用this引用，静态成员不能访问非静态成员。

个数可变的形参可以代表0-n个参数，只能处于形参列表的最后，也就是说一个方法最多只能有一个个数可变的形参。个数可变的形参本质上是一个数组类型的形参，因此可以直接传入一个数组。

方法递归一定要向可知方向递归，即一定会终结递归

方法重载的要求，两同一不同：

- 同一个类
- 方法名相同
- 参数列表不同

使用固定数量形参的方法总是会比使用可变个数形参的方法优先执行，因此不推荐重载形参个数可变的方法

成员变量无需显式初始化，局部变量除了形参必须显式初始化。

一个类里不能有同名的成员变量；一个方法里不能有同名的方法局部变量和形参；一个代码块内不能有和已有方法局部变量同名的代码块局部变量；不同代码块内可以有重名的代码块局部变量；方法局部变量可以和成员变量重名

实例变量实在创建实例时分配内存空间并指定初始值。

定义局部变量后，系统并未为这个变量分配内存空间，直到程序为这个变量赋初始值时，系统才会为其分配内存，并将初始值保存在这块内存中。局部变量保存在其所属方法的栈内存中。栈内存中的变量无需系统垃圾回收，随方法或代码块的运行结束而结束。

一下情况考虑使用成员变量：

- 变量用于描述某个类或某个对象的固有信息；
- 变量用于保存该类或者实例运行时状态信息；
- 变量用于保存某个在多个方法之间共享的消息。

外部类只能有2种访问控制级别：public和默认

如果某个类主要用于做其他类的父类，该类包含的大部分方法可能仅希望被其子类重写，而不是被外界直接调用，此时应该使用protected修饰这些方法，在子类中重写这些方法并扩大权限至public

如果一个java源文件里定义的所有类都没有使用public修饰，则这个java源文件可以是一切合法的文件名。

带包的可执行类在执行时必须使用全路径，而不能人工进入包目录，直接使用类名执行。java在执行可执行类时，会在classpath指定路径下寻找包目录，然后在包目录中寻找该类。因此，同一个包的类的class文件可以存储在不同的位置，但必须具备相同的与包结构对应的目录结构并在classpath指定路径下。如果编译时不适用-d选项，编译器不会为class文件自动生成相应的文件结构。

父包和子包在用法上不存在任何关系，如果父包中的类要是用子包中的类，必须使用子包的全名，不能省略父包部分。

import static 和import 语句之间没有顺序要求。import可以省略包名，import static可以进一步省略类名。

Java默认为所有源文件导入`java.lang`包下所有的类

当程序员调用构造器时，系统会先为该对象分配内存空间并执行默认初始化(此时对象已经创建，但只能被构造器通过`this`调用)，然后才会执行构造器。

构造器中使用`this`关键字调用其他构造器的语句必须放在第一条。

java的子类不会继承（可以调用）父类的构造器。

子类虽然继承了父类的private成员，但不可以直接访问，必须通过父类提供的public访问方法。

方法的重写遵循两同两小一大：

- 方法名同，形参列表同
- 子类方法返回值类型，声明抛出的异常类型要比父类方法的更小或相等
- 子类方法的访问权限要比父类方法的大

重写方法和被重写方法必须属于同种方法：类方法或者实例方法，否则编译错误。

子类无法访问父类的private方法，因此也不能重写该方法，即使是完全相同的方法定义也只是相当于重新定义了一个新方法。

系统查找变量的顺序：

1. 首先查找当前方法中是否有同名的局部变量
2. 然后查找当前类中是否有同名的成员变量
3. 上一级父类中是否有同名的成员变量，迭代上溯直到找到该变量，如果找不到，编译出错。

super和this关键字都不能出现在类方法中，也不会同时出现。因为都要求出现在第一行。

子类的构造器一定会调用父类构造器一次：

- 子类构造器第一行使用`super`关键字显式调用父类构造器
- 子类构造器使用`this`显式调用其他没有使用`this`的构造器，参考下一条
- 子类构造器没有显式调用父类构造器，也没有使用`this`关键字调用其他构造器，则隐式调用父类无参数的构造器

**父类必须拥有一个无参数的构造器。**

创建任何对象总是从该类所在继承树最顶层类的构造器开始执行，然后依次向下执行，最后才执行本类的构造器。如果某个类通过this调用了同类中重载的构造器，就会依次执行此类的多个构造器。

当程序创建一个子类对象时，系统不仅会为该类中定义的实例变量分配内存，也会为它从父类继承得到的所有实例变量分配内存。当系统创建一个java对象时，如果该java类有两个父类（直接父类和间接父类），那么这个java对象保存着这三个类中定义的所有的实例变量。

- 编译时类型：声明该变量时使用的类型
- 运行时类型：实际赋给变量的对象的类型


引用变量只能调用声明该变量时所用类型包含的方法，但方法体是运行时类型所对应的方法体。

通过引用变量访问他的实例变量，总是试图访问他编译时类型所定义的成员变量而不是运行时类型所定义的成员变量。

引用类型强制转换时，两个类之间必须有继承关系，否则，运行时出现ClassCastException异常

instanceof运算符前面操作数的编译时类型要么与后面的类相同，要么与后面的类具有父子继承关系，才能运行，返回值是`true`或`false`，如果没有继承关系，则编译出错。

尽量不要在父类构造器中调用将要被子类重写的方法

- 子类需要额外增加属性，而不仅仅是属性值的改变；
- 子类需要增加自己独有的行为方式（包括重写或者增加新的方法） 

初始化块在创建java对象时依次执行并在执行构造器之前执行，只能被static修饰，

当java创建一个对象时，系统先为该对象的所有实例变量分配内存（前提是该类已经被加载过了），接着程序开始对这些实例变量执行初始化：先执行初始化块或声明实例变量时指定的的初始值，再执行构造器里指定的初始值。

当JVM第一次使用某个类时，系统会在类的准备阶段为该类的所有静态成员变量分配内存，在初始化阶段负责这些静态成员变量的初始化，也就是执行静态初始化块或者声明类变量时指定的初始值，执行顺序与在源代码中出现的顺序相同。

出现在初始化块和静态初始化块的变量必须是在代码块外声明的变量，并且可以放在代码块后声明。因为执行初始化块时，已经创建了该变量并为其分配了内存空间，具体原因参考上面2段说明。

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
		

静态初始化块在类加载的初始化阶段执行，而不是在创建对象时才执行，因此比普通初始化块先执行。而且只能访问静态成员，不能访问非静态的实例变量和实例方法。


## Chapter 6 面向对象（下） ##

把字符串类型的值转换为基本类型：

- 包装类的parseXxx(String s)静态方法
- 包装类的valueOf(String s)静态方法
- 包装类的Xxx(String s)构造器

把基本类型转换为字符串类型：

- String类的valueOf()方法
- 与空字符串""连接

包装类变量与基本类型变量的比较是直接比较值；
包装类变量之间的比较是比较两者是否指向同一个包装类对象

-128~127之间的同一个整数自动装箱成Integer实例时，永远都是引用cache数组里的同一个数组元素，所以他们全部相等，但每次把一个在此范围之外的整数自动装箱成Integer实例时，系统总是重新创建一个Integer实例，所以同一整数生成的Integer实例并不是同一对象。

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

- final修饰的类变量必须在声明该变量时指定初始值或在静态初始化块中指定初始值，二者只能选其一
- final修饰的实例变量必须在声明该实例变量或构造器中，或非静态初始化块中指定初始值，三者只能选其一

final成员变量在显式初始化之前不能直接访问，但可以通过方法来访问，此时该成员变量会被系统自动赋予默认值。

final修饰的局部变量可以不在声明时赋值，但在之后必须且只能被赋值一次。

final变量在满足以下条件时相当于一个直接量：

- 使用final修饰
- 定义该变量时指定了初始值
- 该初始值在编译时就能确定下来（表达式只是基本的算术表达式或字符串连接运算，没有访问普通变量，调用方法）

编译器会把程序中所有用到该变量的地方直接替换成该变量的值。如果被赋的是基本的算术表达式或字符串连接，没有访问任何普通变量，调用方法，编译器同样会把这种final变量作为直接量(宏变量)处理。

final实例变量只有在定义时就指定初始值才会有宏变量效果。

final方法不能被子类重写；但由于子类不会重写父类的private方法，因此父类的private方法即使用final修饰，在子类中仍然可以看到与其完全相同的子类方法，但这不是重写，而算是一个只属于子类的新方法。

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

通过new创建对象不会启用缓存

抽象类即使不包含抽象方法也不能用于创建实例。

抽象类可以包含成员变量，方法，构造器，初始化块，内部类5种成分。抽象类的构造器不能用于创建实例，主要用于被子类调用。

抽象类必须包含一个不带参数的构造器，否则无法被继承。

final和abstract永远不能同时使用；
abstract不能用于修饰变量，也不能用于修饰构造器；
static和abstract不能同时修饰同一个方法，但可以同时修饰内部类；
private和abstract不能同时修饰同一个方法

接口定义的是多个类共同的公共行为规范，通常是定义一组公用方法。

Java9允许在接口中定义默认方法，类方法和私有方法，都可以提供方法实现。

接口不能包含构造器和初始化块，可以包含成员变量（只能是静态常量），方法（只能是抽象实例方法，类方法或默认方法，私有方法(Java9)），内部类（包括内部接口和枚举）。

- 接口里的所有成员，包括常量，方法，内部类和内部枚举都是public访问权限。
- 接口里的常量总是用public static final修饰的（也是默认修饰符），因为接口中没有构造器和初始化块，所以只能在定义时赋值。
- 接口的私有方法主要作用是作为工具方法为默认方法和类方法提供支持，不能用default修饰，可以用static修饰
- 接口中的普通方法总是使用public abstract来修饰，不能有方法体。类方法，默认方法和私有方法都必须有方法体。
- 接口里定义的内部类，内部接口，内部枚举默认都采用public static来修饰。
- 接口的默认方法必须是用default修饰，不能使用static修饰，而且总是public的，可以有多个默认方法。实际上就是实例方法。
- 接口的类方法必须使用static修饰，不能用default，而且总是public的

接口支持多继承；可以实现多个接口，implements必须房子extends的后面。
如果继承的多个父接口有相同的默认方法，子接口必须重写该默认方法。

接口不能显式继承任何类，但所有接口类型的变量可以直接赋给Object类型的变量。

接口和抽象类的区别

- 接口只能包含抽象方法，静态方法，默认方法和私有方法，不能为普通方法提供实现；抽象类则可以包含普通方法
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

非静态内部类里不能有静态成员，包含静态变量，静态初始化块和静态方法。但可以包含普通初始化块。

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
2. 在外部类以外使用非静态内部类：必须先创建外部类的对象，再通过该对象创建其内部类对象，即非静态内部类对象必须依附于某一个其所在外部类的对象。非静态内部类的子类的对象也必须依附于某一个其父类所在外部类的对象。非静态内部类的构造器必须通过外部类对象来调用。非静态内部类的子类的构造器必须有一个形参指向其父类所在外部类的对象。

		OuterClass.InnerClass var = OuterInstance.new InnerConstructor();

3. 在外部类以外使用内部静态类：通过外部类名.内部类名来使用，类似于包空间

使用内部类时，应该优先考虑使用静态内部类。子类无法覆盖父类的内部类，因为内部类的完整类名也包含外部类的类名。

所有局部成员都不能使用访问控制符修饰。

局部内部类的class文件比成员内部类的class文件多了一个数字

匿名内部类必须继承一个父类或者实现一个接口，但最多只能继承一个父类或实现一个接口。

- 实现接口：只能通过接口名()来调用匿名内部类的隐式无参数构造器
- 继承父类：可以使用与父类构造器拥有相同形参列表的构造器

匿名内部类不能：

- 是抽象类
- 包含构造器

匿名内部类必须实现它的抽象父类或者接口里包含的所有抽象方法。也可以重写父类中的普通方法。

被匿名内部类访问的局部变量必须是final（系统会自动限制-effectively final）

当Lambda表达式需要返回值，而他的代码块仅有一条省略了return的语句，Lambda会自动返回该语句的值;
Lambada表达式的参数还是形参。

函数式接口：只有一个抽象方法的接口（可以含有类方法和默认方法）。可以包含多个默认方法，类方法，但只能声明一个抽象方法。

Lambda表达式的目标类型必须是明确的函数式接口。

Java8，`java.util.function`中常见的函数式接口：
- XxxFunction：通常包含一个`apply()`方法，用于对参数进行转换处理，然后返回一个新值
- XxxConsumer：通常包含一个`accept()`方法，也是对参数进行处理，但不返回处理结果
- XxxPredicate：通常包含一个`test()`方法，对参数进行判断，然后返回一个boolean值
- XxxSupplier：通常包含一个`getAsXxx()`方法，不需要输入参数，会按实现的逻辑算法返回一个数据

如果Lambda表达式的代码块只有一条代码，还可以在代码块中使用方法（这里的方法指的不是函数式接口的抽象方法，而是在实现该抽象方法时所调用的具有方法体的其他类的类方法或实例方法）引用和构造器引用：p216 表6.2

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
- 匿名内部类实现的抽象方法的方法体允许调用接口中定义的默认方法，lambda表达式的代码块则不允许调用。注意与上一段中不同的地方在于，上一段指的是使用lambda创建的对象，这里指的是Lambda本身的代码块。


枚举类

- 使用enum定义的枚举类默认继承了java.lang.Enum类，因此不能显式继承其他父类，Enum类实现了Serializable和Comparable接口；
- 非抽象的枚举类默认使用final修饰，因此不能派生子类
- 枚举类的构造器只能使用private访问控制符
- 枚举类的所有实例必须在枚举类的第一行显式列出，以,分开，;结束，且总是public static final。

当switch控制表达式使用枚举类型时，case表达式中的值可以直接使用枚举值，无需添加枚举类作为限定。

枚举类通常应该设计为不可变类，因此建议将枚举类的成员变量都使用private final修饰，并在构造器里为这些final成员变量赋值，也就是说必须显式定义带参数的构造器。一旦定义了带参数的构造器，在枚举类中第一行列出所有枚举值时就必须使用该构造器为每个枚举值传入实际参数值。

非抽象的枚举类才默认使用final修饰，而实现了接口，拥有抽象方法的枚举类就是抽象枚举类型，默认使用abstract修饰，可以衍生子类，从而为每个枚举值提供不同的抽象方法实现，每个实现了抽象方法的枚举值都是他的匿名子类的实例。

包含抽象方法的枚举类不能显式使用abstract修饰（已经默认是abstract），但是定义每一个枚举值时要为抽象方法提供实现，否则出现编译错误。

枚举类型中出现抽象方法的原因：

- 实现了接口：
	- 可以在枚举类中直接实现接口的抽象方法，此时所有的枚举值共享该实现方法；
	- 也可以在列举每个枚举值时加入代码块单独实现该抽象方法，此时每个枚举值相当于该枚举类的一个匿名子类的实例。每个枚举值拥有不同的实现方法。
- 枚举类自带抽象方法：列出每个枚举值时必须单独提供该抽象方法的实现。

垃圾回收机制只负责回收堆内存中的对象，不负责回收物理资源；在回收任何对象之前，都会调用它的`finalize()`方法。

对象在内存中的状态：

- 可达状态：有指向该对象的引用
- 可恢复状态：没有指向该对象的引用，但还没允许该对象的finalize()方法
- 不可达状态：运行了该对象的finalize()方法后，仍没有指向该对象的引用。

可以调用以下方法来督促系统进行垃圾回收：
- `System`类的静态方法`System.gc()`
- `Runtime`对象的`gc()`实例方法：`Runtime.getRuntime().gc()`

永远不要主动调用某个对象的finalize()方法；
当jvm执行finalize()方法出现异常时，垃圾回收机制不会报告异常，程序继续执行

不要使用finalize()方法清理某个类打开的资源，因为不能确定finalize()是否会被执行。

对象的引用：

强引用(StrongReference) ：通常的赋值都是强引用
软引用(SoftReference)   ：使用SoftReference类实现，内存不足时，才会回收只有软引用指向的对象
弱引用(WeakReference)   ：使用WeakReference类实现，内存足不足都会回收只有弱引用指向的对象
虚引用(PhantomReference)：使用PhantomReference类实现，不能单独使用（构造时必须指定一个引用队列），主要用于和引用队列配合跟踪对象被垃圾回收的状态。系统无法通过虚引用获得指向的对象。

引用队列使用java.lang.ref.ReferenceQueue类表示，用于保存已经被回收对象的引用。
对于软引用，弱引用，其引用的对象被回收后，会把所对应的引用添加到关联的引用队列中；
虚引用则是在对象被回收之前，将指向他的虚引用添加到关联的引用队列中。

使用弱引用和软引用可以节省内存空间，但相应会增加重新创建对象的损耗。

		//取出弱引用指向的对象
		obj = weakReference.get();
		//如果取出的对象为空
		if (obj == null) {
			//重新创建对象并使用强引用指向他
			obj = recreateObj();
			//将弱引用指向新建立的对象
			wr = new WeakReference(obj);
		}
		//操作obj对象
		...
		//切断obj和对象之间的关联
		obj = null;

p233 表6.3 Java修饰符适用范围总表

abstract和final永远不能同时使用；
abstract和static不能同时修饰方法，但可以同时修饰内部类；
abstract和private不能同时修饰方法，可以同时修饰内部类；
private和final可以混用但没有意义。

解压缩jar文件时不能使用jar的-C选项来指定解压的目标目录，因为-C选项只能在创建或更新jar时才能使用。

## Chapter 7 java基础类库 ##

Scanner是一个基于正则表达式的文本扫描器，可以从文件，输入流，字符串中解析出基本类型值和字符串值。不同的构造器使用不同类型的参数作为数据源。使用空白（包含空格，Tab空白，回车）作为分隔符。为Scanner设置分隔符使用useDelimiter(String pattern)。 主要使用以下4个方法扫描输入：

- boolean hasNextXxx()
- boolean hasNextLine()
- Xxx nextXxx()
- String nextLine()

has方法的阻塞和next方法的阻塞无关

System类代表当前Java程序的运行平台

通过identityHashCode(Object x)可以获得对象的identityHashCode值，这个特殊的identityHashCode值可以唯一地标识对象。默认情况下Object类的hashCode()方法得到的就是identityHashCode。

RunTime类代表Java程序的运行时环境，每个Java程序都有一个与之对应的RunTime实例，应用程序通过该对象与其运行时环境相连。

Object类常用方法：

- boolean equals(Object obj)
- protected void finalize()
- Class<?> getClass()
- int hashCode()
- String toString()

自定义类实现“克隆”的方法：

1. 自定义类实现Cloneable接口
2. 自定义类实现自己的clone()方法
3. 实现clone()方法时调用super.clone()实现浅克隆

Object类的clone()方法只是一种浅克隆，只克隆该对象的所有成员变量值，不会对引用类型的成员变量值所引用的对象进行克隆。也就是说只克隆了引用，没有创建新对象。

Ojbects类的toString(Object obj)方法，不会因传入的参数为null而引发NullPointerException异常，而是会返回一个"null"字符串。

Objects类的requireNonNull()方法，当传入的参数不为null时，返回参数，否则引发NullPointerException异常。用于对方法形参进行输入校验。

Objects类的`hashCode(Object obj)`返回的hashcode值和`obj.hashcode()`返回值不一样。

Objects类的`requireNonNull(Object obj, Supplier<String> messageSupplier)`比`requireNonNull(Object obj, String message)`效率要高

String类时不可变类，一旦一个String对象被创建，包含在其中的字符序列是不可改变的；
StringBuffer代表一个字符序列可变的字符串，一旦通过StringBuffer生成了最终想要的字符串，就可以调用它的toString()方法将其转换为一个String对象。

StringBuilder和StringBuffer的用法完全，只是StringBuffer是线程安全的。两者都实现了`CharSequence`接口，这个接口可以作为字符串的协议接口

Java9改进了字符串的实现，包括String，StringBuffer，StringBuilder。
Java9以前字符串采用char[]数组保存字符，因此每个字符占2个字节，Java9的字符串采用byte[]数组再加一个encoding-flag字段保存字符。使用方法不受影响。

String类常用方法：

- char charAt(int index)
- int compareTo(String anotherString)：
	- 如果两个字符串的字符序列相等，则返回0，不相同时，从两个字符串第0个字符开始比较，返回第一个不相等的字符差（调用者-被调用者）；
	- 另一种情况，较长字符的前面部分恰巧是较短的字符串，则返回他们的长度差。
- String concat(String str)
- boolean contenEquals(StringBuffer sb)
- static String copyValueOf(char[] data)
- static String copyValueOf(char[] data, int offset, int count)
- boolean endsWith(String suffix)
- boolean equals(Object anObject)
- boolean equalsIgnoreCase(String str)
- byte[] getBytes()
- void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin)
- int indexOf(int ch)
- int indexOf(int ch, int fromIndex)
- int indexOf(String str)
- int indexOf(String str, int fromIndex)
- int lastIndexOf(int ch)
- int lastIndexOf(int ch, int fromIndex)
- int lastIndexOf(String str)
- int lastIndexOf(String str, int fromIndex)
- int length()
- String replace(char oldChar, char newChar)：替换第一个oldChar
- boolean startsWith(String prefix)
- boolean startsWith(String prefix, int toffset)
- String substring(int beginIndex)
- String substring(int beginIndex, int endIndex)
- char[] toCharArray()
- String toLowerCase()
- String toUpperCase()
- static String valueOf(Xxx xxx)

StringBuffer和StringBuilder的length属性表示所包含的字符序列的长度，capacity表示容量，通常容量比长度大。

使用StringBuilder和StringBuffer的`reverse()`方法时，(high-low)的surrogate对不会翻转，作为一个字符处理，而(low-high)的surrogate对会翻转，成为一个新的字符。

`String`类的`compareTo(String anotherStr)`方法的可能返回值：
- 0：当`this.equals(str) == true`
- `this.charAt(k) - str.charAt(k)`：如果2个字符串中有不同字符，其中`k`是出现不同字符的最低索引位置
- `this.length() - str.length()`：如果2个字符串没有不同字符，但长度不同。

`Math.round(double)`方法返回`long`，而不是`double`

ThreadLocalRandom类是Random类的增强版，可以减少多线程资源竞争。

使用默认种子构造Random对象时，属于同一个种子。

只要两个Random对象的种子相同，方法的调用顺序也相同，他们会产生相同的数字序列。

推荐使用当前系统时间作为Random对象的种子：

    Random rand =  new Random(System.getCurrentTimeMills());

建议优先使用基于String的构造器构造BigDecimal类的实例，如果必须使用double浮点数创建BigDecimal对象，而是通过BigDecimal.valueOf(double value)静态方法来创建BigDecimal对象。

`BigDecimal`的对象时不可变的，因此使用`setScale()`系列方法不会改变修改原对象，会返回一个具有新标度的对象，这个对象可能不是新创建的。

Calendar类本身是一个抽象类，是所有日历类的模板，并提供了一些所有日历通用的方法，本身不能实例化。Calendar.MONTH字段代表月份，起始值不是1，而是0

Calendar类的add,roll,set方法区别

- add()与set()方法类似，当修改的字段超出允许范围，会发生进位，如果下一级字段也需要修改，该字段会修正到变化最小的值
- roll()方法不会发生向上进位，对下级字段处理与add方法相同

Calendar类有2种模式：

- lenient模式：允许字段超出允许范围，做进位处理
- non-lenient模式：不允许字段超出范围，否则抛出异常，可以用于严格的参数检查。

通过Calendar类set(f, value)方法将日历字段f更改为value，尽管f的值是立即更改的，但Calendar所代表的时间只会在下次调用get(), getTime(), getTimeInMillis(), add(), roll()方法时才会使用f的新值重新计算日历的时间。

Java8新增的日期，时间包：

- Clock
- Duration
- Instant
- LocalDate
- LocalTime
- LocalDateTime
- MonthDay
- Year
- YearMonth
- ZonedDateTime
- ZoneId
- DayOfWeek
- Month

典型的正则表达式的使用：

		Pattern p = Pattern.compile("a*b");
		Matcher m = p.matcher("aaaaabb");
		boolean b = m.matches();

String类里使用正则表达式的方法：

- boolean matches(String regex)
- String replaceAll(String regex, String replacement)
- String replaceFirst(String regex, String replacement)
- String[] split(String regex)

如果某个正则表达式仅需一次使用，则可直接使用Pattern类的静态matches()方法，此方法自动把指定字符串编译成匿名的Pattern对象。

Pattern类是不可变类，可供多个并发线程安全使用。

Matcher类常用方法：

- find()：返回目标字符串中是否包含与Pattern匹配的子串
- group()：返回上一次与Pattern匹配的子串
- start()：返回上一次与Pattern匹配的子串在目标字符串中的开始位置
- end()：返回上一次与Pattern匹配的子串在目标字符串中的结束位置+1
- lookingAt()：返回目标字符串前面部分与Pattern是否相配
- matches()：返回整个目标字符串与Pattern是否相配
- reset()：将现有Matcher对象应用于一个新的字符序列

MethodHandle和VarHandle用于动态调用和处理方法和变量。通过MethodHandles工具类的静态方法和内部工厂类来生成相应的实例。VarHandle主要用于动态操作数组的元素或对象的成员变量。

java的国际化涉及的类：

- java.util.ResourceBundle：用于加载国家，语言资源包
- java.util.Locale：用于封装特定的国家，区域的语言环境
- java.text.MessageFormat：用于格式化带占位符的字符串

国际化步骤：

1. 准备basename.properties，basename_language.properties或basename_language_country.properties文件。其中language和country必须使用java的Locale中的language和country。
2. Java9已可直接使用UTF-8字符集的属性文件，Java8及以前必须使用native2ascii工具转换包含非西欧字符的资源文件

		native2ascii basename.properties basename_language_country.properties

3. 使用以下类似代码从相应资源文件中取得所需语言文字：

		public class Hello {
			public static void main(String[] args) {
				Locale myLocale = Locale.getDefault(Locale.Category.FORMAT);
				ResourceBundle bundle = ResourceBundle.getBundle("mess", myLocale);
				System.out.println(bundle.getString("hello"));
			}
		}

属性文件必须显示保存为UTF-8字符集

带占位符的字符串可以使用MessageFormat的format(String pattern, Object... values)来用后面的多个参数值填充前面的pattern字符串，其中pattern字符串不是一个正则表达式，而是一个带占位符的字符串。 

ResourceBundle搜索资源的顺序：

1. basename_zh_CN.class
2. baseName_zh_CN.properties
3. basename_zh.class
4. baseName_zh.properties
5. basename.class
6. baseName.properties

NumberFormat和DateFormat都包含

- format()：把数值和日期格式化为字符串
- parse()：把字符串解析成数值或日期

NumberFormat是个抽象基类，使用类方法获得NumberFormat对象：

- getCurrenyInstance()
- getIntegerInstance()
- getNumberInstance()
- getPercentInstance()

DateFormat也是一个抽象基类，常用方法：

- getDateInstance()
- getTimeInstance()
- getDateTimeInstance()

DateFormat对象可以调用setLenient(boolean lenient)方法设置是否采取严格语法。DateFormat的parse()方法要求被解析的字符串必须符合日期字符串的要求，否则可能抛出ParseException异常。

Java8新增了DateTimeFormatter类，通常使用下面三种方法获取该类的实例：

- 直接引用该类的静态常量作为DateTimeFormatter格式器
- 使用代表不同风格的枚举值来创建DateTimeFormatter
- 使用模式字符串来创建DateTimeFormatter

使用DateTimeFormatter将日期时间格式化为字符串的方法：

- 调用DateTimeFormatter的format(TemporalAccessor temporal)方法执行格式化，LocalDate，LocalTime，LocalDateTime等都是TemporalAccessor接口的实现类
- 调用LocalDate, LocalTime, LocalDateTime等类的对象的format(DateTimeFormatter formatter)方法执行格式化

可通过日期事件对象的parse(CharSequence text, DateTimeFormatter formatter)方法使用DateTimeFormatter将字符串解析成日期时间对象

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

实例变量类型                     | 计算方式
-------------------------------- | -------
boolean                          | f?0:1
整数类型(byte, short, char, int) | (int)f
long                             | (int)(f^(f>>>32))
float                            | Float.floatToIntBits(f)
double                           | long l = Double.doubleToLongBits(f); (int)(l^(l>>>32))
引用类型                         | f.hashCode()

常用计算hashcode值的方法：

1. 使用上表中的计算方式计算与hashcode值有关的实例变量的hashcode
2. 把每个相关实例变量的hashcode值与一个质数相乘再相加得到最终的hashcode值

在把对象加入到HashSet后，尽量不要修改影响equals和hashCode的实例变量的值，否则会引起HashSet混乱

LinkedHashSet集合里的元素按照添加顺序排列，但依然不允许集合元素重复

加入TreeSet的类必须实现Comparable接口，而且必须是同一个类的对象，否则抛出ClassCastException异常

TreeSet使用compareTo()方法来判断两个对象是否相等，如果两个对象通过compareTo(Object obj)方法比较相等，则新对象无法加入TreeSet中。因此，在改写放入TreeSet的类的equals()方法时，应注意他的返回值与compareTo()方法保持一致。

TreeSet可以删除没有被修改实例变量，且不与其他被修改实例变量的对象重复的对象，但不能删除改变了实例变量的对象，也不能删除未修改但与修改过元素相等的元素。这个限制会在成功删除某个元素后解除，因为TreeSet会重新索引所有元素。

TreeSet的自然排序采用升序排列；如果使用定制排序，则采用指定的Comparable接口实现类(通常是Lambda表达式)来负责排序，而元素本身不必实现Comparable接口

EnumSet不允许添加null元素，会抛出NullPointerException异常。但查询和删除null元素不会抛出NullPointerException，删除操作会返回false。EnumSet类没有显示构造器，只能通过静态方法来创建对象。

EnumSet是所有Set类中性能最好的，但只能保存同一个枚举类的枚举值作为集合元素。

当试图复制一个Collection集合里的元素来创建EnumSet集合时，必须保证Collection集合里的所有元素都是同一个枚举类型的枚举值。

HashSet，TreeSet和EnumSet都是线程不安全的，如果需要保证Set集合的同步性，可以再创建Set时使用Collections的synchronizedSortedSet方法包装该集合。

		SortedSet set = Collections.synchronizedSortedSet(new TreeSet(...));

List判断两个对象相等只需equals()返回true即可。而且调用的是用于判断是否被包含的对象的equals方法，而不是list中元素的equals方法。

List的set(int index, Object element)的index必须是List的有效索引。

ArrayList和Vector类都是基于数组实现的List类，其内部数组的默认是10，可以使用void ensureCapacity(int minCapacity)方法来调整数组的长度，也可以使用void trimToSize()方法来缩减数组长度至实际包含元素的个数

不推荐使用Vector类以及其子类Stack，可以使用ArrayDeque来实现栈结构。

Arrays类的asList(Object... a)方法生成的List集合是Arrays类内部类ArrayList的实例，程序只能遍历访问该集合里的元素，不能增加或删除内容。

Queue接口定义的方法：

- 加入元素到队尾
	- void add(Object e) 队列若满则抛出异常
	- boolean offer(Object e) 队列若满则返回false，当使用有容量限制的队列时，通常比add更好
- 获取队列头部的元素
	- Object element() 获取但不删除，为空则抛出异常
	- Object peek() 获取但不删除，为空则返回null
	- Object poll() 获取并删除，为空则返回null
	- Object remove() 获取并删除，为空则抛出异常

PriorityQueue类不允许插入null元素，还需要对队列元素进行排序，排序的方法可参考TreeSet的自然排序和定制排序。PriorityQueue的显示顺序可能不同于数据存入以及提出数据的顺序，即其显示顺序可能既不是存入的顺序，也可不能不是排列后的顺序。

ArrayList和ArrayDeque两个集合类的实现机制基本相似，都是采用一个动态的，可重新分配的Object[]数组来存储集合元素，当集合元素超出了该数组的容量时，系统会在底层重新分配一个Object[]数组，并把原来数组里的内容拷贝过去。

对于所有内部基于数组的集合实现，如ArrayList，ArrayDeque等，使用随机访问的性能比使用Iterator迭代访问的性能要好。

使用List的建议：

- 如果需要遍历List集合元素，对于ArrayList和Vector集合，应使用随机访问方法（get）来遍历集合元素，对于LinkedList集合，可以使用迭代器（Iterator）遍历
- 如果需要经常插入，删除包含大量数据的的List集合的大小，可考虑使用LinkedList集合
- 如果需要保证同步性，需要用Collections工具类包装

Map的key和value可以是任何引用类型的数据，key不允许重复，通过equals()方法返回的总是true。

如果把Map里的key放在一起，就组成了一个Set集合。Map的实现类和子接口中key集的存储形式和对应Set集合中元素的存储形式完全相同。Java实际上是先实现了Map，然后通过包装一个所有value都是null的Map实现了Set。

把Map里的value放在一起，就类似一个List。

Map的Object put(Object key, Object value)方法返回的时被覆盖的value值或是null。

Map中还包括一个内部类Entry，该类封装了一个key-value对。

HashMap和Hashtable的区别：

- Hashtable是线程安全的，而HashMap不是
- Hashtable不能使用null作为key和value，会引发NullPointerException异常，而HashMap可以，但最多只能有一个key是null

HashMap，Hashtable判断两个value相等的标准是两个对象通过equals()方法比较返回true即可。

与HashSet一样，HashMap和Hashtable也不能保证其中的key-value对的顺序
 
如果使用自定义类作为HashMap,Hashtable的key，必须重写该类的equals()方法和hashCode()方法以获得一致的判断结果。

如果使用自定义类作为TreeMap的key，必须重写该类的equals()方法和compareTo()方法以获得一致的返回结果。

LinkedHashMap使用双向链表来维护key-value对，使其顺序与插入顺序一致

Properties相当于一个key和value都是String类型的Map，有get和set属性的方法，也有读取和存储属性文件的方法

在IdentityHashMap中，当且仅当两个key严格相等（key1 == key2）时，IdentityHashMap才认为两个key相等，对于普通的HashMap，只要key1和key2通过equals()方法比较返回true，且它们的hashCode值相等即可。

IdentityHashMap也可以使用null作为key和value，但也不保证key-value对的顺序

TreeMap和TreeSet一样也有两种排序方式：

- 自然排序：所有key必须是同一个实现了Comparable接口的类，否则会出现ClassCastException异常
- 定制排序：创建TreeMap时传入一个Comparator对象，负责对所有的key进行排序，此时不要求key实现Comparable接口

WeakHashMap和HashMap的区别：

- HashMap的key使用的是对象的强引用，此时对象是不会被回收的
- WeakHashMap的key使用的是对象的弱引用，对象是可以被系统回收的，此时WeakHashMap也会自动删除该key所对应的key-value对

当TreeMap被填充后，可以调用keySet()，取得由key组成的set，然后使用toArray()方法生成key的数组，在使用Arrays的binarySearch()方法在已排序的数组中快速地查询对象。

EnumMap的特点：

- 内部以数组形式存储
- 根据key的自然顺序，也就是枚举值在枚举类中的定义顺序来维护key-value对的顺序
- 不允许使用null作为key，但可以作为value使用

HashMap和Hashtable的hash表都包含如下属性：

- 容量(capacity)：hash表中桶(bucket)的数量
- 初始化容量(initial capacity)：创建hash表时桶的数量，在构造器中指定初始化容量大小
- 尺寸(size)：当前hash表中记录的数量
- 负载因子(load factor)：等于size/capacity，可以理解为当前hash表的使用率，0~1
- 负载极限：等于(prefered max size)/capacity，0~1，默认0.75，当负载因子大于极限时，自动创建一个2倍容量的新hash表，并把原hash表的内容放入到新的hash表中

使用Collections类的synchronizedXxx()方法将指定的集合包装成线程同步的集合

使用Collections类的emptyXxx(), singletonXxx(), unmodifiableXxx()方法创建集合的只读版本

在java9中可以直接调用Set，List，Map的of()方法创建包含N个元素的不可变集合。不可变集合中既不能添加新元素，也不能删除元素

创建不可变的Map集合有两种方法：

- 使用of()方法依次传入多个key-value对
- 使用ofEntries()方法传入多个Map.Entry对象

## Chapter 9 泛型 ##
Java9中允许在创建匿名内部类时使用菱形语法，系统会根据上下文推断内部类中泛型的类型

泛型：就是在定义类，接口和方法时使用类型参数，这个类型形参将在声明变量，创建对象，调用方法时根据传入的实际类型参数，即类型实参来动态地指定所使用的实际类型。相当于生成了逻辑上的子类，但该子类在物理上并不存在

当实现使用泛型的接口，或者继承使用泛型的类时，不能在接口和父类名中使用类型形参，必须指定实际类型参数或者不传入任何类型参数

调用方法时必须为所有的数据形参传入参数值，与调用方法不同，使用类，接口时也可以不为类型形参传入实际的类型参数。编译器会发出警告，这种省略了泛型的形式称为原始类型(raw type)



不存在泛型类：不管为泛型的类型形参传入哪一种类型实参，java都会将他们作为同一个类处理，在内存中也只占用一块内存空间，因此在静态方法，静态初始化或者静态变量的声明和初始化中不允许使用类型参数，instanceof后面也不能使用泛型类。

如果Foo是Bar的一个子类型（子类或者子接口），而G是具有泛型声明的类或接口，G<Foo>不是G<Bar>的子类型。数组和泛型不同，Foo[]依然是Bar[]的子类型。

Java不支持泛型数组

问号?称为通配符，可以代表任何类型。

使用raw type(MyType)和使用通配符?(MyType<?>)的区别：二者都可以用于表示泛型形参是任意类型的类或接口，但实现机制不同，raw type抹去了所有的泛型信息，也就关闭了泛型类型的检测，所以可以接受任意类型作为泛型参数的实参，因为它们并不起作用。此时，如果出现泛型类型污染，编译器不会发现报错，只有在运行时才会报错。而使用通配符?的接口和类型仍然保持了泛型类型的信息，?表示可以接受任何类型的泛型参数类型，但注意一旦为该使用了通配符的类或接口的实例指定了实际的泛型形参类型，该实例将使用该类型来做泛型类型检测，防止泛型类型污染。

如果编译器在编译阶段不能确定使用了通配符?的集合中?所代表的具体的泛型参数类型，则不能往集合中添加任何元素。如该集合作为方法的参数的时候：

		static void appendNewObject(List<?> list) {
			list.append(new Object());  //compile error!
		}

带通配符的List仅表示它是各种泛型List的父类，并不能把元素加入其中，唯一的例外是null，它是所有引用类型的实例。使用get()方法取出的元素是Object类型。

		List<?> list = new ArrayList<String>();
		//compile error
		list.add(new Object());

指定通配符上限的集合只能从中取元素，取出的元素的类型是上限的类型，不能向集合中添加元素。

程序可以为类型形参设定多个上限，但至多只能有一个父类上限，可以有多个接口上限，用&连接，父类上限必须放在第一位。

		class <T extends SuperClass & Interface1 & Interface2 ...>

指定通配符下限的集合可以向其中添加元素，取出的元素只能被当成Object类型处理。

如果Foo是Bar的一个子类型（子类或者子接口），

- G<Foo>是G<? extends Bar>的子类
- G<Bar>和G<Object>是G<? super Foo>的子类

泛型方法：声明方法时定义一个或多个类型形参

		修饰符 <T, S> 返回值类型 方法名(形参列表) {}

方法中的泛型参数无须显式传入实际类型参数，编译器根据实参推断出最直接的类型（上限类型），推断时只比较方法签名中的使用了泛型的参数

泛型方法和类型通配符的区别：

使用类型通配符的参数或变量是无法在编译阶段确定其具体类型的。

- 通配符是被设计用来支持灵活的子类化的，适合只需要在不同的调用点传入不同实际类型。
- 如果某个方法中一个形参(a)的类型或返回值的类型依赖于另一个形参(b)的类型，则应该使用泛型方法在方法签名中声明类型参数(b)。
- 如果无需向集合中添加元素或删除元素，可以使用类型通配符，无需使用泛型方法
- 类型通配符既可以在方法签名中定义形参的类型，也可用于定义变量的类型（List<? extends Number> c），但泛型方法中的类型形参必须在方法中显式声明。

泛型构造器使用的泛型参数与所属类的泛型参数没有关系。
如果程序指定了泛型构造器中声明的类型形参的实际类型，则不可以使用“菱形”语法。

		class MyClass<E> {
			public <T> MyClass(T t) {...}
		}
		...
		MyClass<String> mc = new <Integer> MyClass<String>(5); 
		//MyClass<String> mc = new <Integer> MyClass<>(5); wrong 


使用类型通配符的方法可以重载但无法调用

		public static <T> void copy(Collection<T> dest, Collection<? extends T> src) {}
		public static <T> void copy(Collection<? super T> dest, Collection<T> src) {} 

Java8中改进了泛型方法的类型推断能力：

- 通过调用方法的上下文来推断泛型的目标类型
- 在方法调用链中，将推断得到的泛型传递到最后一个方法

如果没有为泛型类指定实际的类型参数，则该类型参数被当成raw type处理，默认为声明该类型参数时指定的第一个上限类型。
当把一个具有泛型信息的对象赋给一个没有泛型信息的变量时，所有在尖括号内的类型信息都会丢失，变成raw type。

Java允许直接把List对象赋给一个List<Type>(Type可以是任意类型)类型的变量，但会发出“[unchecked]未经检查的转换”警告，对list变量的实际上的引用时List<Type>，所以当试图把集合里的元素当成OtherType类型的对象使用时将引发ClassCastException异常。

如果一段代码在编译时没有提出“[unchecked]未经检查的转换”警告，则程序运行时不会引发ClassCastException异常。所以不能在创建数组时使用类型变量或类型形参，除非是无上限的类型通配符，但可以在声明这样的数组。比如可以声明List<String>[]类型的数组，但不能创建ArrayList<String>[10]这样的数组对象。

在应用中，可以使用无上限的通配符来声明和创建数组对象，然后使用添加了instanceof判断的强制类型转换取出数组元素。

创建元素类型是泛型类型的数组对象将导致编译错误。

## Chapter 10 异常处理 ##

通常情况下，如果try块被执行一次，则try块后只有一个catch块会被执行，绝不可能有多个catch块被执行。

捕获异常的原则是先捕获小异常，再捕获大异常

捕获多种类型的异常时

- 多种异常类型之间用竖线(|)隔开
- 异常变量有隐式的final修饰，因此程序不能对异常变量重新赋值。

如果try块的某条语句抛出异常，该语句后的其他语句都不会获得执行的机会。

即使try块或catch块中执行了return语句，finally块总会被执行，而且catch块和finally块至少出现其中之一，finally块必须位于所有的catch块之后。

如果在异常处理代码中调用了System.exit(1)来退出虚拟机，则finally块将失去执行的机会。

通常情况下，不要在finally块中使用return或throw等会导致方法终止的语句，否则会造成try块，catch块中的return，throw语句失效。

当java程序执行try块，catch块时遇到了return或throw语句，方法不会立即结束，系统会先去寻找该异常处理流程中是否包含finally块，如果没有，程序立刻执行这两个语句，方法终止；如果有finally块，系统立即开始执行finally块，然后才跳回来执行return和throw语句。所以如果finally块中也有return或throw语句，将不会再跳回try块和catch块执行任何代码。

自动关闭的try语句允许在try关键字后面紧跟一对圆括号，括号内可以声明，初始化一个或多个资源，此处的资源指的是那些必须在该语句结束时显式关闭的资源。这些资源类必须实现AutoCloseable或Closeable接口。Closeable接口是AutoCloseable接口的子接口，close()方法抛出IOException异常，AutoCloseable接口的close()方法抛出Exception异常。Java9不要求在括号中声明并创建资源，只需要自动关闭的资源有final修饰或者是有效的final(effectively final)即可。因此可以在try块之前声明并创建final资源，在try后面的括号中直接引用。

throws声明抛出只能在方法签名中使用，throws可以抛出多个异常类，每个类之间以逗号隔开。

子类方法声明抛出的异常应该时父类方法声明抛出的异常的子类或相同，不允许比父类方法声明抛出的异常多。

大部分时候推荐使用RunTime异常，因为无须在方法签名中声明抛出异常。

如果throw语句抛出的异常是Checked异常，则该throw语句要么处于try块内，显式捕获该异常，要么放在一个带throws声明抛出的方法中；
如果throw语句抛出的异常是RunTime异常，则该语句无须放在try块里，也无须放在带throws声明抛出的方法中，可以完全不理会该异常，由方法调用者自行处理。

自定义异常类时通常需要提供两个构造器：

- 无参数构造器
- 带字符串参数构造器，字符串作为该异常对象的描述信息

通常没有必要使用超过2层的嵌套异常处理

可以在catch块中使用throw抛出新的异常以便于方法调用者对异常进行二次处理。大型企业应用中，通常需要对异常分两部处理：

1. 应用后台需要通过日志来记录异常发生的详细情况
2. 应用还需要根据异常向使用者传达某种提示

方法签名中声明的抛出异常可以比捕捉的异常更具体，如方法声明throws FileNotFoundException，但方法体内catch Exception， 前提是方法体内抛出的异常只能是这个范围更小的异常。

异常转译：程序先捕捉原始异常，然后抛出一个新的业务异常，其中包含了对用户的提示信息
异常链：捕获一个异常然后接着抛出另一个异常，并把原始异常信息保存下来。

如果main方法没有处理捕捉到的异常，jvm会中止程序运行，并打印异常的跟踪栈信息。

通常处理异常方式：

- 捕获并保留诊断信息
- 通知合适的人员
- 采取合适的方式结束异常活动

处理异常注意事项：

- 不要过度使用异常。不要用异常处理代替流程控制，也不要和普通错误混在一起
对于完全已知的错误，应该编写处理这种错误的代码，增加程序的健壮性。只有对外部的，不能确定和预知的运行时错误才使用异常。
不要使用异常处理代替正常的业务逻辑判断
不要使用异常处理代替正常的程序流程控制
- 不适用过于庞大的try块，把大块的try块分割成多个可能出现异常的程序块，并把他们放在单独的try块中，从而分别捕获并处理异常
- 避免使用catch all语句
- 不要忽略捕获到的异常，处理它，或者抛出它然后在合理的层处理它。


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

**APT(Annotation Processing Tool)**： 访问和处理Annotation的工具，检测并找出源代码所包含的注解信息，然后针对注解信息进行额外的处理

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

标识修饰的类，方法等已经过时，如果被使用，编译器将报警。Java9加入了2个属性：

- forRemoval：该API在将来是否会被删除
- since：指定该API从哪个版本被标记过时

#### @SuppressWarnings ####

对该注解修饰的元素及其所有子元素都有效，关闭编译器警告，在`@SuppressWarnings`的后面加括号，里面使用`value="警告类型"`来指定关闭警告类型

    @SuppressWarnings(value="unchecked")

#### @SafeVarargs ####

**堆污染(Heap pollution)**：当把一个不带泛型的对象赋给一个带泛型的变量时，会造成堆污染，因为该对象内的数据类型可能与该变量指定的泛型类型不相符。如果方法具有个数可变的形参，而该形参的类型又是泛型，则更容易出现堆污染，因为Java中没有泛型数组，而个数可变的形参实质就是数组。

三种方法抑制堆污染警告：

- 通过@SafeVarargs修饰方法或构造器可以关闭该方法或构造器的堆污染警告，Java9允许使用该注解修饰私有实例方法。
- 使用`@SuppressWarnings("Unchecked")`修饰
- 编译时使用`-Xlint:varargs`选项

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
包含一个`ElementType`的value成员变量，使用时必须指定为以下值的组合，之间用逗号隔开：

- `ElementType.ANNOTATION_TYPE`： 只能修饰其他Annotation
- `ElementType.CONSTRUCTOR`： 只能修饰构造器
- `ElementType.FIELD`： 只能修饰成员变量
- `ElementType.LOCAL_VARIABLE`： 只能修饰局部变量
- `ElementType.METHOD`： 只能修饰方法定义
- `ElementType.PACKAGE`： 只能修饰包定义
- `ElementType.PARAMETER`： 可以修饰参数
- `ElementType.TYPE`： 可以修饰类，接口，其他注解或枚举定义

#### @Documented ####
只能修饰其他Annotation，用于表明被他修饰的Annotation一旦修饰了程序元素，则该注解也会出现在该元素的API文档中

#### @Inherited ####
只能修饰其他Annotation，如果某个类使用了被此注解修饰的注解，则这个类的子类将自动被同样的注解修饰

### 自定义Annotation ###

- **标记Annotation**： 没有定义成员变量的Annotation
- **元数据Annotation**： 包含成员变量的Annotation

注解的成员变量在注解定义中以无参数的方法形式来声明，其方法名和返回值定义了该成员变量的名字和类型。

    public @interface MyAnnotation {
    	String name() default "Martin";
    	int age() default 30;
    }

使用自定义Annotation时，有默认值的成员变量可以不用显式指定值，没有默认值的成员变量必须显式指定值。

Java使用java.lang.Annotation接口来代表所有的注解，它是所有注解的父接口。

java.lang.reflect.AnnotatedElement代表了所有可以接受注解的程序元素。该接口的实现类（Class，Constructor，Filed，Method，Package）都有相应的方法调取相关的（包含或者修饰自己）Annotation的信息

- <A extends Annotation> A getAnnotion(Class<A> annotationClass)：
- <A extends Annotation> A getDeclaredAnnotion(Class<A> annotationClass)
- Annotation[] getAnnotations()
- Annotation[] getDeclaredAnnotion()
- boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)
- <A extends Annotation>A[] getAnnotationsByType(Class<A> annotationClass)：Java8新增，功能类似getAnnotations()，但支持重复注解 
- <A extends Annotation>A[] getDeclaredAnnotationsByType(Class<A> annotationClass)：Java8新增，功能类似getDeclaredAnnotations()，但支持重复注解

#### 使用注解实现监听器模式 ####

1. 定义一个带有ActionListener属性的注解类
2. 定义一个监听器安装类，该类有一个带Object类型形参的类方法，负责为传入的对象的成员变量添加监听器
3. 在某个类中声明需要监听器的变量时使用该注解类，并指定所需的监听器类型
4. 在初始化类的对象时调用监听器安装类的类方法，把此对象作为实参传入

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

File类是java.io包下代表与平台无关的文件和目录。File类不能访问文件内容本身，需要使用输入/输出流。

#### 访问文件名相关的方法 ####

- String getName()
- String getPath()
- File getAbsoluteFile()
- String getAbsolutePath()
- String getParent()
- boolean renameTo(File newName)

#### 文件检测相关的方法 ####

- boolean exist()
- boolean canWrite()
- boolean canRead()
- boolean isFile()
- boolean isDirectory()
- boolean isAbsolute()

#### 获取常规文件信息 ####

- long lastModified()
- long length()

#### 文件操作相关的方法 ####

- boolean createNewFile()
- boolean delete()
- static File createTempFile(String prefix, String suffix)
- static File createTempFile(String prefix, String suffix, File Directory)
- void deleteOnExit()

#### 目录操作相关的方法 ####

- boolean mkdir()
- String[] list()
- File[] listFiles()
- static File[] listRoots()

`getAbsoluteFile`和`getAbsolutePath`的区别在于前者返回的是File对象，后者返回的是String对象。

当使用相对路径的File对象来获取父路径时可能引起错误，返回null，因为该方法返回的是将该File对象对应的相对路径的目录名，文件名里的最后一个子目录名，子文件名删除后的结果。

在File类的list方法中可以接受一个FilenameFilter参数，通过该参数列出符合条件的文件。
FilenameFilter是一个函数式接口，包含了一个accept(File dir, String name)方法，该方法将依次对指定File的所有子目录和文件进行迭代，如果返回true，则list()方法会列出该子目录或文件。

Java支持使用`/`作为平台无关的路径分隔符，否则在windows平台下必须使用两条`\`来作为路径分隔符， 如`d:\\temp\\tmp.txt`

- **输入流：**只能读数据，不能写数据
- **输出流：**只能写数据，不能读数据

输入/输出流是从程序运行所在的内存角度判断，例如，内存写到硬盘是输出流。
输入流主要以InputStream和Reader作为基类，输出流主要以OutputStream和Writer作为基类，这些都是抽象基类，无法直接创建实例。

- **字节流：**操作的数据单元是8位的字节
- **字符流：**操作的数据单元是16位的字符

字节流以InputStream和OutputStream作为基类，字符流则以Reader和Writer作为基类。

- **节点流（低级流/low level stream）：**可以从/向一个特定的IO设备读/写数据的流，直接连接到实际的数据源，和实际的输入输出节点连接
- **处理流（高级流/high level stream）：**对一个已存在的流进行连接或封装，通过封装后的流来实现数据的读/写功能。不直接连接到实际的数据源，也没有和实际的输入输出节点连接。

处理流是典型的装饰器模式。 

处理流的功能：
- 提高性能：主要以增加缓冲的方式提高输入/输出的效率
- 方便操作：提供一系列便捷方法来操作大量内容

### InputStream & Reader ###
#### 读取数据的方法 ####

- `read()` 返回所读取的字节/字符数据
- `read(byte[] b)`/`read(char[] cbuf)` 返回实际读取的字节数/字符数
- `read(byte[] b, int off, int len)`/`read(char[] cbuf, int off, int len)` 返回实际读取的字节数/字符数
- 如果返回-1，说明到达输入流的终点

#### 操作记录指针的方法 ####

- void mark(int readAheadLimit)：在记录指针当前位置设置一个标记
- boolean markSupport()：判断此输入流是否支持mark()操作
- void reset()：将此流的记录指针重新定位到上一次记录标记mark的位置
- long skip(long n)：记录指针向前移动n个字节/字符。

### OutputStream & Writer ###

- void write(int c)
- void write(byte[]/char[] buf)
- void write(buye[]/char[], int off, int len)
- void write(String s)                    //Writer类专有
- void write(String s, int off, int len)  //Writer类专有

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

- void unread(byte[]/char[] buf)
- void unread(byte[]/char[] buf, int off, int len)
- void unread(int b)

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

- long getFilePointer()
- void seek(long pos)

`RandomAccessFile`的访问模式：

- "r"： 以只读方式打开文件，执行写入操作将抛出IOException
- "rw"： 以读写方式打开文件，如果该文件不存在，将尝试创建该文件
- "rws"： 以读写方式打开文件，还要求对文件的内容或元数据的每个更新都同步写入到底层存储设备
- "rwd"： 以读写方式打开文件，还要求对文件的内容的每个更新都同步写入到底层存储设备

`RandomAccessFile`不能直接往文件中间插入新的内容，因为新插入的内容会覆盖原有内容，如果需要插入新的内容，则要把该位置后的内容存入到缓冲区，在写完新内容之后，再从缓冲区中读入原来的内容追加在新内容之后。

### 对象序列化 ###

对象序列化机制允许把内存中的Java对象转换成平台无关的二进制流，从而允许把这种二进制流保存在磁盘上，或者通过网络进行传输。

**对象的序列化Serialize**：将一个Java对象写入IO流中
**对象的反序列化Deseialize**：从IO流中恢复该Java对象

可序列化的类必须实现如下2个接口之一：

- `Serializable`：仅仅是一个标记接口，不用实现任何方法。
- `Externalizable`

Java9允许对读入的序列化数据进行过滤，这样就可以在反序列化之前对数据执行校验。

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

如果序列化了多个Java对象到一个文件，反序列化时必须按照实际写入的顺序读取。为了解决EOFException的问题，可以将多个对象放入一个容器对象中，如数组或集合，然后把这个容器序列化，读取时只用读取该容器，再从容器中取出其他对象。这样就不用判断是否读到了最后一个对象。44

当一个可序列化类有多个父类时，这些父类必须满足以下条件之一，否则反序列化时会抛出InvalidClassException:

- 父类也是可序列化的
- 父类有无参数的构造器，如果不是可序列化的，则父类的成员变量值不会序列化到二进制流中

可序列化的类的成员变量必须都是可序列化的，即基本类型，String或者其他可序列化的类

Java的序列化算法：

- 使用序列化了对象都会有一个序列化编号
- 当序列化一个对象时，程序会检查该对象是否第一次被序列化，如果是，则将其转换成字节序列并赋予一个序列化编号
- 如果该对象曾经被序列化，则输出该对象的序列化编号，而不是再次重新序列化

因此，如果一个对象的某个成员变量的值在该对象被序列化后发生改变，即使再次执行`writeObject()`方法，该变化也不会被记录，在反序列化时读取出来的依然是改变前的原值

#### Java9新增的过滤功能 ####
Java9为ObjectInputStream新增了setObjectInputFilter(), getObjectInputFilter()方法。当程序通过ObjectInputStream反序列化对象时，过滤器的checkInput()方法被自动激发，该方法有3中返回值：

- Status.REJECTED：拒绝恢复
- Status.ALLOWED：允许恢复
- Status.UNDECIDED：未做决定，继续检查

**递归序列化**：序列化对象时，所有的成员变量都会被序列化，以此类推

`transient`关键字只可用于实例变量，表示该实例变量无须序列化，不可修饰其他成分

如果需要在序列化某类对象时对数据特殊处理，可以在该类中提供特殊签名的方法：

- `private void writeObject(java.io.ObjectOutputStream out) throws IOException`
- `private void readObject(java.io.ObjectInputStream in)throws IOException, ClassNotFoundException`
- `private void readObjectNoData() throws ObjectStreamException`

`writeObject()`方法和`readObject()`方法对对象的处理顺序必须保持一致

当序列化流不完整时，readObjectNoData()方法可以用来正确地初始化反序列化的对象，例如接收方使用的反序列化的类的版本不同于发送方，或者接收方版本扩展的类不是发送方版本扩展的类，又或者序列化流被篡改时，系统都会调用readObjectNoData()方法来初始化反序列化的对象。

可以用另一个对象来完全替代某个可序列化对象：

    ANY-ACCESS-MODIFIER Object writeReplace() throws ObjectStreamException
    ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException

在序列化对象时，系统会递归调用`writeReplace()`方法，直到所引用对象不再具有`writeReplace()`方法，此时调用该对象的`writeObject()`方法输出对象。

`readResolve()`方法会紧接着`readObject()`方法之后被调用，该方法的的返回值会取代反序列化恢复的对象，而恢复的对象将被抛弃。该方法在序列化单例类，枚举类时尤其有用。

反序列化机制在恢复java对象时无须调用构造器来初始化Java对象，所以可以用来克隆对象

`final`类重写`readResolve()`方法没有问题，否则应该尽量使用`private`修饰该方法

使用`Externalizable`机制反序列化对象时，程序会先使用public的无参数构造器创建实例，然后才执行readExternal()方法进行反序列化，因此实现此接口的类必须提供一个无参数的构造器，否则编译出错：`no valid construction`

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

传统的输入输出流都是阻塞式的流，一次只能处理一个字节，新IO采用内存映射文件的方式来处理输入/输出，主要包含以下几个包：

- java.nio
- java.nio.channels
- java.nio.charset
- java.nio.channels.spi
- java.nio.charset.spi

- **通道(Channel)**：在新IO系统中所有的数据都必须通过通道传输，与传统InputStream，OutputStream最大的区别是它提供了一个`map()`方法，可以把一块数据映射到内存中
- **缓冲(Buffer)**：本质是一个数组，发送到Channel中的所有的对象都必须放在Buffer中，而从Channel中读取的数据也必须先放到Buffer中

新IO还提供了将Unicode字符串映射成字节序列以及逆映射操作的Charset类，和用于支持非阻塞式输入输出的Selector类。

Buffer是个抽象咧，除boolean外的基本类型都要对应的XxxBuffer类，这些Buffer类都没有构造器，获得Buffer对象的方法：

	static XxxBuffer allocate(int capacity)

常用的主要时ByteBuffer类和CharBuffer类，其中ByteBuffer类还有一个MappedByteBuffer子类，用于表示Channle将磁盘文件的部分或者全部内容映射到内存中后得到的结果，由Channle的map()方法返回。

- **容量(capacity)**： 表示该buffer的最大数据容量，即最多可以存储多少数据，不可能为负值，创建后不能改变
- **界限(limit)**： 第一个不该被读出或写入的缓冲区位置索引，位于limit后的数据既不可被读，也不可被写
- **位置(position)**： 下一个可以被读出或者写入的缓冲区位置索引，当使用buffer从channel中读取数据时，position的值=已读数据的数量，但指向下一个buffer的位置，参考数组的索引。

	0<=mark<=position<=limit<=capacity

如果position或者limit < mark，则mark会被抛弃

开始时Buffer的Position为0，limit为capacity，程序可以通过put()方法向Buffer中放入数据，Buffer的position也会相应地向后移动。当buffer装入数据结束后，调用buffer的`flip()`方法，该方法将limit设置为position的位置，并将position归零，从而使buffer做好输出数据的准备；当bufffer输出数据结束后，buffer调用`clear()`方法，该方法并不清空buffer的数据，仅仅将position归零，将limit设为capacity，这样为再次向buffer中装入数据做好准备。

Buffer常用方法：

- int capacity()
- boolean hasRemaining()：position和limit之间是否还有元素
- int limit()
- Buffer limit(int newLt)
- Buffer mark()：在0和position之间做mark
- int position()
- Buffer position(int newPs)
- int remaining()
- Buffer reset()：position转到mark所在位置
- Buffer rewind()：position归零，删除mark

使用`put()`和`get()`访问buffer中的数据时，分为相对和绝对：

- 相对：从buffer的当前position处开始读取或写入数据，然后将position的值按照处理元素的个数增加。
如果要求的数据超出边界
	- `get()`抛出`BufferUnderflowException`异常
	- `put()`抛出`BufferOverflowException`异常
- 绝对： 直接根据索引向buffer中读取或写入数据，并不会影响position的值。如果index非法，则抛出`IndexOutofBoundsException`异常

只有`ByteBuffer`才能通过allocateDirect()方法创建直接buffer，而且直接buffer只适用于长生存期的buffer，不适用于短生存期的buffer

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

Charset类提供了一个avaailableCharsets()类方法来获取当前JDK所支持的所有字符集。

Charset类常用方法：

- CharBuffer decode(ByteBuffer bb)
- ByteBuffer encode(CharBuffer cb)
- ByteBuffer encode(String str)

String类也提供了一个getBytes(String charset)方法用于返回一个使用指定字符集将字符串转换的byte[]

使用`FileChannel`中的`lock()`和`trylock()`方法获得`FileLock`对象，两个方法的区别是：如果不能获得文件锁，`lock()`方法会阻塞程序，`trylock()`方法会返回null

锁定部分内容：

- `lock(long position, long size, boolean shared)`
- `trylock(long position, long size, boolean shared)`

`shared`为true，表明该锁是一个共享锁，允许多个进程读取该文件，但阻止其他进程获得该文件的排它锁；
`shared`为false，表明该锁是一个排它锁，将锁住对该文件的读写

无参数的`lock()`和`trylock()`方法获取的都是排它锁

通过FileLock的isShared()方法判断获得的文件锁是否是排它锁，通过FileLock的release()方法释放文件锁。

### Java7的NIO.2 ###

Path接口代表一个平台无关的平台路径；
Files类包含了大量用于操作文件的类方法；
Paths类包含了两个返回Path的静态工厂方法。

`getNameCount()`方法返回Path路径所包含的路径名的数量，也就是路径的深度

#### 使用`FIleVisitor`遍历文件和目录： ####

Files类用于遍历文件和目录的方法：

- walkFileTree(Path start, FileVisitor<? super Path> visitor)：遍历start路径下的所有文件和目录
- walkFileTree(Path start, Set<FileVisitOption> options, int maxDepth, FileVisitor<? super Path> visitor)

FileVisitor代表一个文件访问器，遍历文件盒子目录会触发FileVisitor中相应的4中方法之一

- FileVisitResult postVisitDirectory(T dir, IOException exc)  //访问子目录后触发
- FileVisitResult preVisitDirectory(T dir, IOException exc)  //访问子目录前触发
- FileVisitResult visitFile(T file, BasicFileAttributes attrs)  //访问文件时触发
- FileVisitResult visitFileFailed(T file, IOException exc)  //访问文件失败时触发

FileVIsitResult是一个枚举类，代表了访问之后的后续行为：

- CONTINUE
- SKIP_SIBLINGS
- SKIP_SUBTREE
- TERMINATE

实际编程时可以通过继承`SimpleFileVisitor`来实现自己的文件访问器

#### 使用WatchService监控文件变化 ####
watcher用于监听变化，每个变化对应一个event，WatchKey相当于在监听但未查询期间所发生的event集合。
类似于：watcher相当于监听记录本，WatchKey就是监听记录本上的未读记录，event就是监听记录本上的每条记录，
    register(WatchService watcher, WatchEvent.kind<?>... events)
	//WatchService代表一个文件系统监听服务
    //使用WatchService的三个方法来获得被监听的目录的文件变化事件
    WatchKey poll()  //获取下一个WatchKey, 没有则立刻返回null
    WatchKey poll(long timeout, TimeUnit unit)  //尝试等待timeout时间去获取下一个WatchKey
    WatchKey take()  //获取下一个WatchKey，没有则一直等待
如果程序需要一直监控，应使用`take()`方法；如果只需要监控指定时间，则使用`poll()`方法

#### 访问文件属性 ####

使用java.nio.file.attribute包下的工具类操作文件的属性，主要分为两大类：

- XxxAttribtueView：代表某种文件属性的视图，FileAttributeView是其他视图类的父接口
- XxxAttributes：代表某种文件属性的集合，一般通过XxxAttribtueView对象来获取XxxAttributes

常用视图类：

- AclFileAttributeView：设置文件的ACL（Access Control List），getAcl()方法返回List<AclEntry>对象
- BasicFileAttributeView：readAttributes()方法返回BasicFileAttributes对象
- DosFileAttributeView：readAttributes()方法返回DosFileAttributes对象
- FileOwnerAttributeView：getOwner()方法返回一个UserPrincipal对象代表文件所有者，setOwner(UserPrincipal owner)方法改变文件的所有者
- PosixFileAttributeView：readAttributes()方法返回PosixFileAttributes对象
- UserDefinedFileAttributeView

## Chapter 16 多线程##

### 线程概述 ###

操作系统可以同时执行多个任务，每个任务就是一个进程；
进程可以同时执行多个任务，每个任务就是一个线程。

进程是处于运行过程中的程序，并且具有一定的独立能力，进程是系统进行资源分配和调度的一个独立单位。

进程包含以下三个特征：

- 独立性：进程是系统中独立存在的实体，可以拥有自己独立的资源，每个进程都有自己私有的地址空间。在没有经过进程本身允许的情况下，进程不能访问其他进程的地址空间
- 动态性：程序只是一个静态的指令集合，进程是一个正在系统中活动的指令集合。进程具有自己的生命周期和各种不同的状态。
- 并发性：多个进程可以在单个处理器上并发执行。

并发性(concurrency)和并行性(parallel)是两个概念：

- 并行：在同一时刻，有多条指令在多个处理器上同时执行；
- 并发：在同一时刻，只有一条指令在执行，但多个进程指令被快速轮换执行，从而在宏观上具有多个进程同事执行的效果

线程/轻量级进程(lightweight process)：是进程的执行单元，在程序中是独立的，并发的执行流。

当进程被初始化后，主线程就被创建了，但也可以在该进程内创建多条顺序执行流，这些顺序执行流就是线程，每个线程也是相互独立的。

一个进程可以拥有多个线程，一个线程必须拥有一个父进程。线程可以拥有自己的堆栈，程序计数器和自己的局部变量，但不拥有系统资源，而是与父进程的其他线程共享该进程所拥有的全部资源。

线程是独立运行的，不知道进程中是否还有其他线程存在。它的执行时抢占式的，运行中的线程在任何时候都可能被挂起，以便另一个线程可以运行。

一个线程可以创建和撤销另一个线程，同一个进程中的多个线程之间可以并发执行。

线程比进程具有更高的性能，因为同一个进程中的线程都有共性，共享同一个进程的虚拟空间。

### 线程的创建和启动 ###

Java使用Thread类代表线程，所有的线程对象都必须是Thread类或其子类的实例。

#### 继承Thread类创建线程类 ####
通过继承Thread类来创建并启动多线程的步骤：

- 定义Thread类的子类，并重写该类的run()方法，其方法体就代表了线程需要完成的任务
- 创建Thread子类的实例，即线程对象
- 调用线程对象的start()方法启动该线程

主线程的线程执行体不是由run()方法确定的，而是由main()方法确定的。

`Thread.currentThread()`：Thread类的类方法，返回当前正在执行的线程对象。

使用继承Thread类的方法创建线程类时，多个线程之间无法共享线程类的实例变量。

#### 实现Runnable接口创建线程类 ####
步骤：

- 定义Runnable接口的实现类，实现该接口的run()方法
- 创建Runnable实现类的实例，并以此实例作为Thread的target来创建Thread对象
- 调用线程对象的start()方法启动线程

Runnable对象仅仅作为Thread对象的target，Runnable实现类里包含的run()方法仅作为线程执行体，实际的线程对象依然是Thread实例，只是该Thread线程负责执行其target的run()方法。

通过继承Thread类来获得当前线程对象可以直接使用`this`关键字；通过事项Runnable接口来获得当前线程对象必须使用`Thread.currentThread()`方法。

采用Runnable接口创建的多个线程可以共享线程类的实例变量，因为此时多个线程之间是共享同一个target，即同一个Runnable实现类的对象。

#### 使用Callable和Future创建线程 ####
Callable接口是一个函数式接口，提供了一个call()作为线程执行体，该方法可以：

- 具有返回值，返回值类型与接口的泛型形参参数类型相同
- 声明抛出异常

Future接口代表Calla接口里的call()方法的返回值，其实现类FutureTask也实现了Runnable接口

Future接口常用方法：

- boolean cancel(boolean mayInterruptIfRunning)：尝试取消该Future里关联的Callable任务
- V get()：返回Callable任务里call()方法的返回值，调用该方法将导致程序阻塞，必须等到子线程结束返回返回值
- V get(long timeout, TimeUnit unit)：在指定时间内等待返回值，超过时间就抛出`TimeoutException`异常
- boolean isCancelled()：如果任务正常完成前被取消，返回`true`
- boolean isDone()：如果任务正常完成，返回`true`

创建并启动有返回值的线程的步骤：

- 创建Callable接口的实现类，并实现cal()方法，然后创建该实现类的实例
- 使用FutureTask类包装Callable对象
- 使用FutureTask对象作为Thread对象的target创建并启动新线程
- 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值

#### 创建线程的三种方式的比较 ####
通过实现Runnable和Callable接口创建的线程类可以继承其他类，创建的线程对象可以共享同一个target对象，从而实现资源共享，推荐使用。

### 线程的生命周期 ###
新建(new)，就绪(runnable)，运行(running)，阻塞(blocked)，死亡(dead)

#### 新建和就绪 ####
当程序使用`new`关键字创建了一个新线程后，该线程就处于新建状态，由Java虚拟机为其分配内存并初始化其成员变量的值。
当程序调用了`start()`方法后，线程处于就绪状态，Java虚拟机为其创建方法调用栈和程序计数器，处于这个状态的线程并没有开始运行，只是表示可以运行，具体什么时候由jvm的线程调度器决定。

永远不要调用线程对象的run()方法。因此这会把线程对象当成普通的对象处理，run()当成普通的方法而不是线程执行体。此时run()方法会一直执行下去，直到任务完成或出现错误，异常而中止。

创建新线程对象后，调用`run()`方法或`start()`方法将使其从新建状态进入就绪状态，此时不能再次调用`start()`方法，否则会抛出`IllegalThreadStatusException`异常。

#### 运行和阻塞状态 ####
如果处于就绪状态的线程获得了cpu，开始执行`run()`方法的线程执行体，该线程就处于运行状态。

- 抢占式策略：系统给每个可执行的线程一个时间段来处理任务，超过时间系统就会剥夺该线程所占用的资源，让其他线程获得执行机会。
- 协作式策略：只有当一个线程主动调用了它的`sleep()`或`yield()`方法后才会放弃所占用的资源

发生以下情况时，线程进入阻塞状态：

- 线程调用`sleep()`方法主动放弃所占用的资源，睡眠时间结束解除阻塞
- 线程调用了一个阻塞式IO方法，在该方法返回之前，该线程被阻塞，方法返回后解除阻塞
- 线程试图获得一个同步监视器，但该同步监视器被其他线程持有，取得该同步计数器后解除阻塞
- 线程在等待某个通知，获得其他线程的通知后解除阻塞
- 程序调用了线程的`suspend()`方法将该线程挂起，调用`resume()`方法解除阻塞，容易引起死锁，应尽量避免使用该方法

#### 线程死亡 ####
线程以以下三种方式结束后处于死亡状态：

- `run()`或`call()`方法执行完成，线程正常结束
- 线程抛出一个未捕获的Exception或Error
- 直接调用该线程的stop()方法结束该线程，容易造成死锁

当主线程结束时，其他线程不收任何影响，不会随之结束。一旦子线程启动，它就拥有与主线程相同的地位，不会受主线程影响。

调用线程对象的`isAlive()`方法判断某个线程是否已经死亡。调用已死亡的线程对象的start()方法会抛出`IllegalThreadStatusException`异常

### 控制线程 ###

#### join线程 ####
通过在线程A的线程执行体中调用线程B的join()方法，使得线程B加入到线程A的执行，线程A必须等线程B执行完毕或者指定等待时间结束后才能继续执行。

- join()
- join(long millis)
- join(long millis, int nanos)

#### 后台线程 ####
后台线程(Daemon Thread)：在后台运行，为其他线程提供服务的线程。如果所有的前台线程都死亡，后台线程会自动死亡。

调用`Thread`对象的`setDaemon(true)`可将指定线程设置成后台线程，但必须在线程启动，即调用`start()`方法之前设置，否则引发`IllegalThreadStatusException`异常

前台线程创建的子线程默认就是前台线程，后台线程创建的子线程默认就是后台线程。

#### 线程睡眠sleep ####
在线程睡眠期间，即使系统中没有任何其他可执行的线程，该线程也不会获得任何执行的机会

- sleep(long millis)
- sleep(long millis, int nanos)

Thread类的`yield()`类方法不会让线程进入睡眠，而是转入就绪状态。此时，只有优先级大于等于当前线程的处于就绪状态的线程（包括才被转入就绪状态的当前线程）才会获得执行机会。

`sleep()`与`yield()`方法的区别：

- `sleep()`方法暂停当前线程后，所有处于就绪状态的线程都有可能获得执行机会，不用考虑优先级；`yield()`方法则要求新执行的线程的优先级要大于等于当前线程。
- `sleep()`方法有指定的休眠时间，在此期间该线程处于阻塞状态，而`yield()`方法则是立刻转入就绪状态
- `sleep()`方法声明抛出了`InterruptedException`异常，必须对其进行处理，而`yield()`方法没有声明抛出任何异常
- `sleep()`方法拥有更好的可移植性

#### 改变线程优先级 ####
每个线程默认的优先级都与创建它的父线程的优先级相同。默认情况下，main线程具有普通优先级。

Thread类的`setPriority(int newPriority)`方法可以设置指定线程的优先级。优先级是一个1~10之间的整数。有3个相关的类变量：

- MAX_PRIORITY：10    
- MIN_PRIORITY：1
- NORMAL_PRIORITY：5

由于各操作系统的优先级定义可能不同，尽量使用上面的静态常量来设置优先级。

### 线程同步 ###
	
#### 同步代码块 ####

		//obj是同步监视器
		synchronized(obj) { 
			... //需要同步的代码块
		}
#### 同步方法 ####
使用`synchronized`修饰的实例方法（非静态方法）无须显式指定同步监视器，其值就是`this`，也就是调用该方法的对象。

`synchronized`可以修饰方法，代码块，但不能修饰构造器，成员变量等。

领域驱动设计(Domain Driven Design)：设计的每个类都应该是完备的领域对象，应该具备完整的领域功能，如银行账户类应该提供取钱和存钱功能。

使用同步代码块和同步方法的策略：

不要对线程安全类的所有方法都进行同步，只对那些会改变竞争资源的方法进行同步
如果可变类有单线程，多线程两种运行环境，则应该为其提供线程不安全和线程安全两种版本，参考`StringBuilder`和`StringBuffer`类

#### 释放同步监视器的锁定 ####
程序无法显式释放对同步监视器的锁定，线程会在以下情况自动释放：

- 当前线程的同步方法，同步代码块执行结束
- 当前线程在同步代码块，同步方法中遇到`break`，`return`终止了该代码块，方法的进行执行
- 当前线程在同步代码块，同步方法中出现了未处理的Error或Exception，导致该代码块，方法异常结束
- 当前线程执行同步代码块或同步方法时，程序调用了同步监视器对象的`wait()`方法

以下情况不会释放同步监视器：

- 线程执行同步代码块或同步方法时，程序调用Thread.sleep()，Thread.yield()方法来在听当前线程的执行
- 线程执行同步代码块时，其他线程调用了该线程的suspend()方法将该线程挂起

#### 同步锁 ####
Lock时控制多个线程对共享资源进行访问的工具。提供了对共享资源的独占访问，每次只能有一个线程对Lock对象加锁，线程开始访问共享资源之前要先获得Lock对象。

Lock，ReadWriteLock是Java5提供的两个根接口。ReentrantLock是Lock的实现类，ReentrantReadWriteLock是ReadWriteLock的实现类。Java8新增了StampedLock类，可以替代传统的ReentrantReadWriteLock类

使用ReentrantLock的代码模板：

		class X {
			//定义锁对象
			private final ReentrantLock lock = nwe ReentrantLock();
			...
			//定义需要保证线程安全的方法
			public void m() {
				//加锁
				lock.lock();
				try {
					//需要保证线程安全的代码块
					//method body
				}
				//使用finally块来保证释放锁
				finally {
					lock.unlock();
				}
			}
		}

同步方法或同步代码块使用与竞争资源相关的，隐式的同步监视器，并强制要求加锁和释放锁要在同一个块结构中，并且当获取了多个锁时，必须以相反的顺序释放，且必须在与所有锁被获取时相同的范围内释放所有锁。而使用Lock对象则可以通过使用finally块实现加锁和解锁出现在不同的作用范围内。

ReentrantLock锁具有可重入性，也就是锁可以嵌套加锁。

#### 死锁 ####
死锁和阻塞不同，死锁是因为阻塞出现了回路，即线程A和线程B都因为需要等待对方的执行结果而处于阻塞状态。

### 线程通信 ###
#### 传统的线程通信 ####
使用同步监视器对象的`wait()`，`notify()`，`notifyAll()`方法，这三个方法继承自Object类

- 同步方法：同步监视器就是该类的实例，所以可以直接在方法中调用这三个方法
- 同步代码块：同步监视器是`synchronized`后面括号里的对象，所以必须调用该对象的这三个方法

- `wait()`：导致当前线程等待，直到其他线程调用该同步监视器的`notify()`或`notifyAll()`方法来唤醒该线程，或者指定等待时间结束后自动唤醒
- `notify()`：随机唤醒在此同步监视器上等待的单个线程
- `notifyAll()`：唤醒所有在此同步监视器上等待的线程

#### 使用`Condition`控制线程通信 ####
使用`Lock`对象保证同步时，使用`Condition`类来保持协调。`Condition`类的实例被绑定在一个`Lock`对象上，需要通过`Lock`对象的`newCondition()`方法来获得相应的`Condition`实例。

`Condition`常用方法：

- `await()`：导致当前线程等待，直到其他线程调用该`Condition`对象的`signal()`或`signalAll()`方法来唤醒该线程，或者指定等待时间结束后自动唤醒，有很多变体，具体参考API
- `signal()`：随机唤醒在此`Lock`对象上等待的单个线程
- `signalAll()`：唤醒所有在此`Lock`对象上等待的线程

#### 使用阻塞队列(BlockingQueue)控制线程通信 ####
`BlockingQueue`接口是`Queue`的子接口，当生产者试图向已满的`BlockingQueue`中放入元素时，线程被阻塞，当消费者试图从空的`BlockingQueue`中取出元素时，线程被阻塞。

`BlockingQueue`常用的方法：

- 在队列尾部插入元素以及当队列已满时的处理：
	- `add(E e)`：抛出异常
	- `offer(E e)`：返回false
	- `put(E e)`：阻塞队列
- 在队列头部删除并返回删除的元素，以及队列已空时的处理
	- `remove()`：抛出异常
	- `poll()`：返回false
	- `take()`：阻塞队列
- 在队列头部取出但不删除元素以及队列为空时的处理：
	- `element()`：抛出异常
	- `peek()`：返回`false`

`BlockingQueue`的5个实现类：

- `ArrayBlockingQueue`：基于数组实现的`BlockingQueue`
- `LinkedBlockingQueue`：基于链表实现的`BlockingQueue`
- `PriorityBlockingQueue`：元素按自然排序或定制排序的顺序取出
- `SynchronousQueue`：同步队列，对该队列的存取操作必须交替进行
- `DelayQueue`：基于`PriorityBlockingQueue`类并实现`Delay`接口（该接口只有一个`long getDelay()`方法），根据`getDelay()`方法的返回值排序 

### 线程组和未处理的异常 ###
`ThreadGroup`代表线程组，用户创建的所有线程都属于某个线程组，如果没有显式指定所属线程组，则该线程属于默认线程组。默认情况下，子线程和创建它的父线程属于同一线程组。一旦某个线程加入指定线程组，则将一直属于该线程组，直至该线程死亡，线程运行中不能改变它所属的线程组。

`Thread`类中指定线程组的构造器：

- `Thread(ThreadGroup group, Runnable target)`
- `Thread(ThreadGroup group, Runnalbe target, String name)`
- `Thread(ThreadGroup group, String name)`

`ThreadGroup`类的构造器：

- `ThreadGroup(String name)`
- `ThreadGroup(ThreadGroup parent, String name)`

可以看出线程组总会有一个字符串类型的名字，可以通过`ThreadGroup`类的`getName()`方法获得，名字不能更改。

`ThreadGroup`类常用方法：

- `int activeCount()`：此线程组中活动线程的数目
- `interrupt()`：中断此线程组中的所有线程
- `isDaemon()`：判断是否为后台线程组
- `setDaemon(boolean daemon)`：设置成后台线程组，当线程组中再没有活动线程时自动销毁
- `setMaxPriority(int priority)`：设置线程组的最高优先级
- `void uncaughtException(Thread t, Throwable e)`：处理该线程组内的任意线程所抛出的未处理异常

如果线程在执行过程中抛出了一个未处理异常，JVM会在结束该线程之前自动查找是否有对应的Thread.UncaughtExceptionHandler对象，如果找到该处理器对象，则会调用该对象的uncaughtException（Thread t, Throwable e)方法处理异常。
`Thread.UncaughtExceptionHandler`是`Thread`类的一个静态内部接口，该接口只有一个方法：`void uncaughtException(Thread t, Throwable e)`。

`Thread`类提供了2个方法来设置异常处理器：

- `static setDefaultUncaughtExceptionHandler(Thread.UncaughtExceptionHandler eh)`：为该线程类的所有线程实例设置默认的异常处理器
- `setUncaughtExceptionHandler(Thread.UncaughtExceptionHandler eh)`：为指定的线程实例设置异常处理器

`ThreadGroup`类实现了`Thread.UncaughtExceptionHandler`接口，每个线程所属的线程组将作为默认的异常处理器。当一个线程抛出未处理异常时，JVM首先查找该异常所对应的异常处理器（`setUncaughtExceptionHandler()`方法设置的异常处理器），如果找到该异常处理器，则调用该异常处理器处理异常，否则，JVM将调用该线程所属的线程组对象的`uncaughtException()`方法来处理异常。

线程组处理异常的默认流程：

1. 如果该线程组有父线程组，调用父线程组的`uncaughtException()`方法来处理异常
2. 如果该线程实例所属的线程组有默认的异常处理器（由`setDefaultUncaughtExceptionHandler()`方法设置的异常处理器），那么调用该异常处理器处理异常
3. 如果该异常对象时`ThreadDeath`的对象，不做任何处理，否则，将异常跟踪栈的信息打印到`System.err`错误输出流，并结束该线程。

异常处理器与通过`catch`捕捉异常不同，当使用`catch`捕捉异常时，异常不会向上传播给上一级调用者，但使用异常处理器处理异常后，异常仍然会传递给上一级调用者。

### 线程池 ###
线程池在系统启动时创建大量空闲的线程，程序将一个`Runnable`对象或`Callable`对象传递给线程池，线程池就会启动一个空闲的线程来执行他们的线程执行体，当执行结束后，该线程不会死亡，而是再次返回线程池中成为空闲状态，等待下一个`Runnable`对象或`Callable`对象。

#### Java8改进的线程池 ####
使用`Executors`工厂类产生线程池：

- `newCachedThreadPool()`：创建带有缓存功能的线程池
- `newFixedThreadPool(int nThreads)`：创建一个可重用的，具有固定线程数的线程池
- `newSingleThreadPool()`：创建一个只有单线程的线程池，相当于`newFixedThreadPool(1)` 
- `newScheduledThreadPool(int corePoolSize)`：创建具有指定线程数的线程池，可以在指定延迟后执行线程任务
- `newSingleThreadScheduledExecutor()`：创建只有一个线程的线程池，在指定延迟后执行线程任务
- `ExecutorService newWorkStealingPool(int parallelism)`：创建持有足够线程的线程池来支持给定的并行级别
- `ExecutorService newWorkStealingPool()`：相当于`newWorkStealingPool(cpuCoresYouHave)`

后两种方法生成的work stealing线程池都相当于后台线程池

`ExecutorService`代表尽快执行线程的线程池，只要线程池中有空闲的线程，就会立即执行线程任务，提供了三个方法：

- `Future<?> submit(Runnable task)`：将一个Runnable对象提交给线程池，`Future`对象在`run()`方法执行结束后返回null
- `<T> Future<T> submit(Runnable task, T result)`：将一个`Runnable`对象提交给线程池，result显式指定线程执行结束后的返回值
- `<T> Future<T> submit(Callable task)`：将一个`Callable`对象提交给线程池，`Future`代表`Callable`对象里`call()`方法的返回值

`ScheduleExecutorService`代表可在指定延迟后或周期性地执行线程任务的线程池，有如下四个方法：

- `ScheduleFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit)`：指定callable任务在delay延迟后执行
- ScheduleFuture<?> schedule(Runnable command, long delay, TimeUnit unit)：指定command任务在delay延迟后执行
- `ScheduleFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)`：指定command任务将在首定延迟后开始执行，然后每隔一段时间就会执行新的任务（不管上一任务是否执行完毕）
- `ScheduleFuture<?> scheduleAtFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)`：创建并执行一个在给定初始延迟后首次启用的定期操作，随后在每次任务终止后，下一任务执行前都会延迟给定的时间，如果任务在任一次执行时遇到异常，就会取消后续执行，否则，只能通过程序显式取消或终止该任务

用完一个线程池后，应该调用该线程池的`shutdown()`方法启动线程池的关闭序列，此时不再接受新的任务，但会把已有任务执行完毕。`shutdownNow()`方法会试图停止所有正在执行的活动任务，暂停处理正在等待的任务，并返回等待执行的任务列表。

使用线程池执行线程任务的步骤：

1. 调用`Executor`类的静态工厂方法创建一个`ExecutorService`对象，该对象代表一个线程池
2. 创建`Runnable`实现类或`Callable`实现类的实例，作为线程执行任务
3. 调用`ExecutorService`对象的`submit()`方法提交`Runnable`实例或`Callable`实例
4. 当不再需要线程池执行任何新任务时，调用`ExecutorService`对象的`shutdown()`方法关闭线程池

#### Java8增强的ForkJoinPool ####
`ForkJoinPool`是`ExecutorService`接口的实现类，其常用构造器为：

- `ForkJoinPool(int parallelism)`：创建一个包含parallelism个并行线程的`ForkJoinPool`
- `ForkJoinPool()`：以`Runtime.availableProcessors()`方法返回值作为paralleism参数来创建`ForkJoinPool`

Java8通过下面两个静态方法提供通用池功能：

- ForkJoinPool commonPool()：返回一个通用池，通用池的运行状态不会受shutdown()或shutdownNow()的影响 
- int getCommonPoolParallelism()：返回通用池的并行级别

创建了`ForkJoinPool`实例后，就可以调用`ForkJoinPool`的`submit(ForkJoinTask task)`或`invoke(ForkJoinTask task)`方法来执行指定任务了。其中ForkJoinTask代表一个可以并行，合并的任务，是一个抽象类，具有两个抽象子类：`RecursiveAction`（代表没有返回值的任务）和`RecursiveTask`（代表有返回值的任务）。分解后的任务分别调用`fork()`方法开始并行执行。

### 线程相关类 ###

#### `ThreadLocal`类 ####
线程局部变量就是为每一个使用该变量的线程都提供一个变量值的副本，使每一个线程都可以独立地改变自己的副本，而不会和其他线程的副本冲突。

`ThreadLocal`类提供了三个public方法

- `T get()`：返回在当前线程中此线程局部变量的值
- `void remove()`：删除在当前线程中此线程局部变量的值
- `void set(T value)`：设置在当前线程副本中此线程局部变量的值

同步机制是为了同步多个线程对相同资源的并发访问，是多个线程之间进行通信的有效方式；而`ThreadLocal`是为了隔离多个线程的数据共享，从根本上避免多个线程之间对共享资源的竞争，也就不需要对多个线程进行同步。

- 使用同步机制：如果多个线程之间需要共享资源，以达到线程之间的通信功能
- 使用`ThreadLocal`：仅仅需要隔离多个线程之间的共享冲突

#### 包装线程不安全的集合 ####
在创建了普通集合之后应该立刻使用`Collections`类提供的静态方法包装成线程安全的集合：

- `static <T> Collection<T> synchronizedCollection(Collection<T> c)`
- `static <T> List<T> synchronizedList(List<T> list)`
- `static <T> Set<T> synchronizedSet(Set<T> s)`
- `static <K, V> Map<K, V> synchronizedMap(Map<K, V> m)`
- `static <K, V> SortedMap<K, V> synchronizedSortedMap(Map<K, V> m)`
- `static <T> SortedSet<T> synchronizedSortedSet(Set<T> s)`

线程安全的集合分为两类：

- 以Concurrent开头的集合类，如`ConcurrentHashMap`，`ConcurrentListMap`，`ConcurrentSkipListSet`，`ConcurrentLinkedQueue`和`ConcurrentLinkedDeque`
- 以CopyOnWrite开头的集合类，如`CopyOnWriteArrayList`，`CopyOnWriteArraySet`

以Concurrent开头的集合类代表了支持并发访问的集合，可以支持多个线程并发写入访问，这些写入线程的所有操作都是线程安全的，在并发写入时有较好的性能，而读取操作不必锁定。

`ConcurrentLinkQueue`适合当多个线程共享访问一个公共集合时使用，不允许使用null元素

默认情况下，`ConcurrentHashMap`支持16个线程并发写入，可以在创建实例时通过传入的`concurrencyLevel`构造参数设定支持的并发线程

由于`ConcurrentLinkedQueue`和`ConcurrentHashMap`支持多进程并发访问，当使用迭代器遍历集合元素时，可能无法反映出创建迭代器之后所做的修改，但程序不会抛出异常

在Java8中增强了的`ConcurrentHashMap`更适合作为缓存实现类使用，新增的方法大致可分为三类：

- forEach系列：`forEach`, `forEachKey`, `forEachValue`, `forEachEntry`
- search系列：`search`, `searchKeys`, `searchValues`, `searchEntries`
- reduce系列：`reduce`, `reduceToDouble`, `reduceToLong`, `reduceKeys`, `reduceValues`

使用`java.util`包下的`Collection`作为集合对象时，如果集合对象在创建了迭代器改变了集合元素，会引发`ConcurrenModificationException`异常

`CopyOnWriteArraySet`的底层封装了`CopyOnWriteArrayList`，因此使用方法完全类似

`CopyOnWriteArrayList`采用复制底层数组的方式来实现写操作。当执行读取操作时，线程会直接读取集合本身，无须加锁与阻塞。当执行写操作时，会在底层复制一份新的数组，并对新的数组执行写操作。适合用于读取操作远大于写操作的场景，如缓存。

#### Java9新增的发布-订阅框架 ####
该框架适合用于处理异步线程之间的流数据交换，主要使用`Flow`类的4个静态内部接口作为核心API：

- `Flow.Publisher`：代表数据发布者，生产者
- `Flow.Subscriber`：代表数据订阅者，消费者
- `Flow.Subscription`：代表发布者和订阅者之间的连接纽带。订阅者通过调用该对象的request()方法来获取数据项，也可通过调用该对象的cancel()方法取消订阅
- `Flow.Processor`：数据处理器，看同事作为发布者和订阅者使用

`Flow.Publisher`接口定义了如下方法来注册订阅者：

- `void subscribe(Flow.Subscriber<? super T> subscriber)`：程序调用此方法注册订阅者时，会触发订阅者的`onSubscribe()`方法，同时把`Flow.Subscription`对象作为参数传给`onSubcribe()`方法。如果注册失败，将触发订阅者的`onError()`方法

`Flow.Subscriber`接口定义了如下方法：

- `void onSubscribe(Flow.Subscription subscription)`：订阅者注册时自动触发该方法
- `void onComplete()`：订阅结束时触发该方法
- `void onError(Throwable throwable)`：订阅失败时触发该方法
- `void onNext(T item)`：订阅者从发布者获取数据时触发该方法，订阅者可以通过该方法获取数据项

Java9为`Flow.Publisher`提供了一个`SubmissionPublisher`实现类，程序创建`SubmissionPublisher`对象时，需要传入一个线程池作为底层支撑；或者默认使用`ForkJoinPool.commonPool()`方法来提交发布者

## Chapter 17 网络编程 ##
### 网络基础知识 ###
通信协议通常由三部分组成：

- 语义部分：决定对话的类型
- 语法部分：决定对话的格式
- 变换规则：决定双方的应答关系

开放系统互连参考模型(OSI: Open System Interconnection)将计算机网络分成七个层次：

- 应用层
- 表示层
- 会话层
- 传输层
- 网络层
- 数据链路层
- 物理层

TCP/IP模型将计算机网络分为四个层次：

- 应用层
- 传输层
- 网络层
- 物理和数据链路层

### IP地址和端口号 ###
IP地址时一个32位整数，通常分为4个8位的二进制数

端口是一个16位的整数，每个通信程序使用一个端口与外界交换并处理数据，同一天机器上不能有两个程序使用同一个端口。

端口号从0到65535，分为3类：

公认端口(Well Known Ports)：0~1023
注册端口(Registered Ports)：1024~49151
动态或私有端口(Dynamic and/or Private Ports)：49152~6553

### Java的基本网络支持 ###

#### 使用InetAddress ####
InetAddress类代表IP地址，有两个子类：`Inet4Address`和`Inet6Address`。没有提供构造器，使用两个静态方法来获取`InetAddress`类的实例

- getByName(String host)：根据主机获取对应的`InetAddress`对象
- getByAddress(byte[] addr)：根据原始IP地址来获取对应的`InetAddress`对象

`InetAddress`类常用方法：

- String getCanonicalHostName()：返回此IP地址的全限定域名
- String getHostAddress()：返回此InetAddress实例对应的IP地址字符串
- String getHostName()：返回此IP地址的主机名
- InetAddress getLocalHost()：返回本机IP地址对应的`InetAddress`实例
- boolean isReachable()：是否可到达改地址

#### 使用URLEncoder和URLDecoder ####
仅包含西欧字符的普通字符串和`application/x-www-form-urlencoded MIME`字符串之间无须转换，非西欧字符串需要使用URLEncoder和URLDecoder来进行转换

- String URLEncoder.encode(String s, String enc)
- String URLDecoder.decode(String s, String enc)

#### URL，URLConnection，和URLPermission ####
URI(Uniform Resource Identifiers)代表统一资源标识符，不能用于定位资源，唯一作用就是解析；
URL(Uniform Resource Locator)对象代表统一资源定位器，包含一个可以到达该资源的输入流，格式如下：

		protocol://host:port/resourceName

URL类常用方法：

- String getFile()
- String getHost()
- String getPath()
- String getPort()
- String getProtocol()
- String getQuery()
- URLConnection openConnection()
- String openStream()：返回一个用于读取该URL资源的InputStream

一种实现多线程下载的步骤：

1. 创建URL对象；
2. 获取指定URL对象所指向资源的大小；
3. 在本地磁盘上创建一个与网络资源具有相同大小的空文件；
4. 计算每个线程应该下载网络资源的哪个部分；
5. 依次创建，启动多个线程来下载网络资源的指定部分。

建立连接读取URL资源的步骤：

- 通过调用URL对象的`openConnection()`方法来创建`URLConnection`对象；
- 设置`URLConnection`的参数和普通请求属性；
- 如果只是发送GET方式请求，使用`connect()`方法建立和远程资源之间的实际连接，如果需要发送POST方式的请求，需要获取`URLConnection`实例对应的输出流来发送请求参数；
- 远程资源变为可用，程序可以访问远程资源的头字段或通过输入流读取远程资源的数据。

在建立和远程资源实际连接之前，可用如下方法设置请求头字段：

- `setAllowUserInteraction()`
- `setDoInput()`
- `setDoOutput()`
- `setIfModifiedSince()`
- `setUserCashes()`

可用如下方法设置或添加通用头字段：

- `setRequestProperty(String key, String value)`
- `addRequestProperty(String key, String value)`

当远程资源可用后，可用如下方法来访问头字段和内容：

- `Object getContent()`
- `getInputStream()`
- `getOutputStream()`
- `String getHeaderFiled(String name)`：获取指定响应头字段的值，也可以使用下面的方法直接获取常用的响应头字段的值
- `getContentEncoding()`
- `getContentLength()`
- `getContentType()`
- `getDate()`
- `getExpiration()`
- `getLastModified()`

当同时需要输入流读取`URLConnection`响应的内容以及输出流来发送请求参数时，一定要先使用输出流，再使用输入流。

### 基于TCP协议的网络编程 ###
#### TCP协议基础 ####
IP协议是支持网间互联的数据包协议，提供网间连接的完善功能，包括IP数据报规定互联网络范围内的地址格式，只保证计算机能发送和接收分组数据。

TCP协议负责收集信息包，并将其按适当的次序放好传送，接收端收到后再将其正确地还原。TCP协议保证了数据包在传送中准确无误。TCP协议使用重发机制，当一个通信实体发送一个消息给另一个通信实体后，需要收到另一个通信实体的确认信息，如果没有收到另一个通信实体的确认信息，则会再次重发刚才发送的信息。

#### 使用ServerSocket创建TCP服务器端 ####
在两个通信实体建立了虚拟链路之前，必须有一个通信实体主动接收来自其他通信实体的连接请求，这个通信实体在Java中以ServerSocket类代表。ServerSocket类的对象用于监听来自客户端的Socket连接，如果没有连接，将一直处于等待状态。常用方法包括：

- Socket accept()：如果接收到一个客户端的Socket连接请求，将返回一个与客户端Socket对应的Socket，否则该方法将一直处于等待状态，线程也被阻塞。

构造器：

- ServerSocket(int port)
- ServerSocket(int port, int backlog)：增加一个用来改变连接队列长度的参数backlog
- ServerSocket(int port, int backlog, InetAddress localAddr)：在机器存在多个IP地址的情况下，使用localAddr参数指定将ServerSocket绑定到指定的IP地址

#### 使用Socket进行通信 ####
构造器：

- Socket(InetAddress/String remoteAddress, int port)：默认使用本地主机的默认IP地址，使用系统自动分配的端口
- Socket(InetAddress/String remoteAddress, int port, InetAddress localAddr, int localPort)

Socket对象获取输入输出流的方法：

- InputStream getInputStream()：程序通过返回的输入流从Socket中取出数据
- OutputStream getOutputStream()：程序通过返回的输出流向Socket中输出数据

这里的输入输出都是从内存而不是流的角度来判断的

Socket提供了设置超时时长的方法：

- setSoTimeout(int timeout)

在使用Socket进行读，写操作完成之前超出该时间限制就会抛出`SocketTimeoutException`异常。值得注意的是，`Socket`类并没有提供指定超时时长的构造器，程序必须先创建一个无连接的`Socket`对象，再调用`Socket`的带有超时时长参数的`connect()`方法来连接远程服务器。

#### 加入多线程 ####
服务器端应该为每个Socket单独启动一个线程，每个线程负责与一个客户端进行通信。客户端也应该单独启动一个线程负责读取服务器端数据。

#### 半关闭的Socket ####
Socket类提供了了两个半关闭的方法，分别用于关闭输入流和输出流，表示完成输入或输出。关闭其中的一个不会影响另外一个的操作。

- shutdownInput()
- shutdownOutput()

注意，即使通过调用这两个方法将socket的输入和输出流全部关闭，Socket实例却没有关闭，只是不能输入，也不能输出。

还有两个方法用于判断是否已经关闭相应的流：

- isInputShutdown()
- isOutputShutdown()

当使用Socket的shutdownOutput和shutdownInput关闭了输出和输入流后，该Socket将无法再次打开输出，输入流，因此这种做法只适合一站式的通信协议，如HTTP协议，当客户端连接到服务器后，开始发送请求数据，发送完毕后无须再次发送数据，只需要读取服务器端的相应数据即可，当读取完毕后，该Socket连接也被关闭了。

#### 使用NIO实现非阻塞Socket通信 ####
按照POSIX（Portable Operating System Interface of UNIX，这个标准定义了操作系统应该为应用程序提供的接口标准）可以将IO分为两类：同步IO和异步IO。对于IO的操作可以分成两步：1. 程序发出IO请求；2. 完成实际的IO操作。阻塞IO和非阻塞IO都是针对第一步划分的，如果发出IO请求会阻塞线程，就是阻塞IO；如果发出请求没有阻塞线程就是非阻塞IO；同步IO和异步IO的区别在第二步：如果实际的IO操作由操作系统完成，再将结果返回给应用程序，这就是异步IO；如果实际的IO需要应用程序本身取执行，会阻塞线程，那就是同步IO。传统IO和基于Channel的非阻塞IO其实都是同步IO。

非阻塞IO有几个关键抽象：Selector，SelectableChannel，SelectionKey

Selector是SelectableChannel对象的多路复用器，所有希望采用非阻塞方式通信的Channel都应该注册到Selector对象。Selector可以同时监控多个SelectableChannel的IO状况，是非阻塞IO的核心。

SelectionKey对象代表了SelectableChannel和Selector之间的注册关系。一个Selector实例有三个SelectionKey集合：

- 所有的SelectionKey集合：代表了所有注册在该Selector上的Channel，该集合可以通过keys()方法返回。
- 被选择的SelectionKey集合：代表了所有可通过select()方法获取的，需要进行IO处理的Channel，该集合可以通过selectedKeys()返回。
- 被取消的SelectionKey集合：代表了所有被取消注册关系的Channel，在下一次执行select()方法时，这些Channel对应的SelectionKey会被彻底删除，系统通常无须直接访问该集合。

SelectableChannel代表可以支持非阻塞IO操作的Channel对象，可调用它的register()方法将其注册到指定Selector上。当该Selector上的某些SelectableChannel上有需要处理的IO操作时，程序可以调用Selector实例的select()方法获取它们的数量，并可以通过selectedKeys()方法返回他们对应的SelectionKey集合--通过该集合就可以获取所有有需要进行IO处理的SelectableChannel集。

SelectableChannel对象支持阻塞和非阻塞两种模式（默认是阻塞模式），下面两个方法分别用来设置和返回模式：

- SelectableChannel configureBlocking(boolean block)
- boolean isBlocking()

不同的SelectableChannel支持的操作不一样，在SelectionKey中，用静态常量定义了4种IO操作：`OP_READ(1)`，`OP_WRITE(4)`，`OP_CONNECT(8)`，`OP_ACCEPT(16)`，由于它们任意2个，3个，4个相加的结果总是互不相同，所以系统可以根据validOps()方法的返回值确定该SelectableChannel支持的操作。

此外，SelectableChannel提供了几个方法获取它的注册状态：

- boolean isRegistered()：返回该Channel是否已注册在一个或多个Selector。
- SelectionKey keyFor(Selector sel)：返回该Channel与Selector sel之间的注册关系，如果不存在注册关系，则返回null。

Selector提供了一系列和select()相关的方法：

- int select()：监控所有注册的Channel，返回有需要处理的IO操作的Channel的数量，并将对应的SelectionKey加入被选择的SelectionKey集合中。
- int select(long timeout)
- int selectNow()：执行一个立即返回的select()操作，相对于无参数的select方法，此方法不会阻塞线程。
- Selector wakeup()：是一个还未返回的select()方法立刻返回。

ServerSocketChannel支持非阻塞操作，对应于java.net.ServerSocket类，支持`OP_ACCEPT`操作，也提供了accept()方法，功能相当于ServerSocket的accept方法。

SocketChannel支持非阻塞操作，对应于java.net.Socket类，支持`OP_CONNECT`，`OP_READ`和`OP_WRITE`操作。这个类还实现了ByteChannel接口，ScatteringByteChannel接口和GatheringByteChannel接口，可以直接通过SocketChannel来读写ByteBuffer对象。

服务器上的所有Channel（包括ServerSocketChannel和SocketChannel）都需要向Selector注册，而该Selector则负责见识这些Socket的IO状态，当其中任意一个或多个Channel具有可用的IO操作时，该Selector的select()方法就会返回大于0的整数，其值就是该Selector上有多少个Channel具有也用的IO操作，并提供了selectedKeys()方法来返回这些Channel对应的SelectionKey集合。当Selector上注册的所欲的Channel都没有需要出来的IO操作时，select()方法将被阻塞，调用该方法的线程被阻塞。

ServerSocketChannel不能像ServerSocket一样直接指定监听的端口，而且也不能使用已有的ServerSocket的getChannel()方法来获取ServerSocketChannel的实例。程序必须先调用它的open()静态方法返回一个ServerSocketChannel的实例，再使用它的bind()方法指定它在某个端口监听。

		//使用open方法创建一个未绑定的ServerSocketChannel实例
		ServerSocketChannel server = ServerSocketChannel.open();
		//将该ServerSocketChannel实例绑定到指定的IP地址
		InetScoketAddress isa = new InetScoketAddress("127.0.0.1", 30000);
		server.bind(isa):

使用NIO来实现服务器端时，无须使用List来保存服务器端所有的SocketChannel，因为所有的SocketChannel都已注册到指定的Selector对象。

#### 使用Java7的AIO实现非阻塞通信 ####
Java7的NIO.2为AIO提供了2个以Asynchronous开头的接口和3个实现类。

AsynchronousServerSocketChannel是一个负责监听的Channel，使用它需要三步：

1. 调用它的open()静态方法创建一个未监听端口的AsynchronousServerSocketChannel。
2. 调用AsynchronousServerSocketChannel的bind()方法指定该Channel在指定地址，端口监听。
3. 调用AsynchronousServerSocketChannel的accept()方法接受连接请求。

AsynchronousServerSocketChannel的open方法有两个版本：

- open()
- open(AsynchronousChannelGroup group)

AsynchronousChannelGroup是异步Channel的分组管理器，可以实现资源共享。创建AsynchronousChannelGroup需要传入一个ExecutorService，也就是说，他会绑定一个线程池，该线程池负责2个任务：处理IO事件和触发CompletionHandler。

AIO的AsynchronousServerSocketChannel和AsynchronousSocketChannel都允许使用线程池进行管理，因此创建AsynchronousSocketChannel对象时也可以传入AsynchronousChannelGroup对象进行分组管理。

accept方法有两个版本：

- Future<AsynchronousSocketChannel> accept()：如果程序需要连接成功后返回的AsynchronousSocketChannel，则应该调用该方法返回的Future对象的get方法，但该方法会阻塞线程。
- <A> void accept(A attachment, CompletionHandler<AsynchronousSocketChannel, ? super A> handler)：接受来自客户端的请求，连接成功或失败都会触发CompletionHandler对象里相应的方法。

CompletionHandler是一个接口，定义了2个方法：

- completed(V result, A attachment)：当IO操作成功完成时触发，result代表IO操作返回的对象，attachment代表发起IO操作时传入的参数。
- failed(Throwable exc, A attachment)：当IO操作失败时触发，exc代表操作失败引发的异常或错误，attachment代表发起IO操作时传入的参数。

异步IO的实际IO操作是交给操作系统完成的，因此程序并不清楚异步IO操作什么时候能够完成。

AsynchronousSocketChannel的用法：

1. 调用open()静态方法创建AsynchronousSocketChannel对象（同时也可以指定一个AsynchronousChannelGroup对象作为分组管理器）；
2. 调用AsynchronousSocketChannel的connect方法连接到指定IP地址，指定端口的服务器；
3. 调用AsynchronousSocketChannel的read，write方法进行读写。

AsynchronousSocketChannel的read，write，connect方法都有两个版本，一个返回Future类型的对象，一个需要传入CompletionHandler参数。

### 基于UDP协议的网络编程 ###
#### UDP协议基础 ####
UDP协议，用户数据报协议（User Datagram Protocol）是一种不可靠的网络协议，也是一种面向非连接的协议。面向非连接的协议指的是在正式通信前不必与对方先建立连接，不管对方状态就直接发送。对方是否能收到这些数据内容，UDP协议无法控制。因此UDP协议只适用于一次只传送少量数据，对可靠性要求不高的应用环境。

UDP协议的主要作用是完成网络数据流和数据报之间的转换：

- 在信息的发送端，UDP协议将网络数据流封装成数据报，然后将数据报发送出去；
- 在信息的接收端，UDP协议将数据报转换成实际数据内容。

UDP协议在通信实例的两端各建立一个Socket，但这两个Socket之间并没有虚拟链路，这两个Socket只是发送，接受数据包对象。Java提供了DatagramSocket对象作为基于UDP协议的Socket，使用DatagramPacket代表发送，接受的数据。对于基于UDP协议的通信双方，没有所谓的客户端和服务器端的概念，但通常把固定IP地址，固定端口的DatagramSocket对象所在的程序称为服务器，因为该DatagramSocket可以主动接收数据。

TCP协议和UDP协议的简单对比：

- TCP协议：可靠，传输大小无限制，但是需要连接建立时间，差错控制开销大；
- UDP协议：不可靠，差错控制开销较小，传输大小限制在64KB以下，不需要建立连接。

#### 使用DatagramSocket发送，接受数据 ####
DatagramSocket的唯一作用就是发送和接受数据报。

常用的三个构造器：

- DatagramSocket()：绑定到本机默认地址，随机选择端口；
- DatagramSocket(int port)：绑定到本机默认地址，指定端口；
- DatagramSocket(int port, InetAddress laddr)：绑定到指定地址和端口。

接受，发送数据报的方法：

- receive(DatagramPacket p)
- send(DatagramPacket p)

使用DatagramSocket发送数据报时并不知道需要发送的目的地，由DatagramPacket自身决定数据报的目的地。

常用的DatagramPacket的构造器：

- DatagramPacket(byte[] buf, int length)：以空数组创建对象，用于接受数据；
- DatagramPacket(byte[] buf, int offset, int length)：以空数组创建对象，用于接受数据，接收到的数据从offset开始放入buf数组中，最多方length个字节；
- DatagramPacket(byte[] buf, int length, InetAddress addr, int port)：以包含数据的数组创建对象，同时指定了发送目的地的IP地址和端口；
- DatagramPacket(byte[] buf, int offset, int length, InetAddress addr, int port)：同上，但指定发送数组从offset开始，总共length个字节。

在接收数据之前，应该采用前两个构造器生成一个DatagramPacket对象，给出接收数据的字节数组及其长度。然后调用DatagramSocket的receive方法来等待数据报的到来，该方法将一直等待（会阻塞调用该方法的线程），直到收到一个数据报为止。

在发送数据之前，调用后两个构造器构造DatagramSocket对象，此时的字节数组里存放了想要发送的数据。除此之外，还要给出完整的目的地址，包括IP地址和端口号。

DatagramPacket提供了三个方法来获取发送者的IP地址和端口：

- InetAddress getAddress()：接收数据时，返回数据报来源主机的IP地址；发送数据时，返回数据报目的地主机的IP地址。
- int getPort()：接收数据时，返回数据报来源主机的端口；发送数据时，返回数据报目的地主机的端口。
- SocketAddress getSocketAddress()：接收数据时，返回数据报来源主机的SocketAddress；发送数据时，返回数据报目的地主机的SocketAddress。SocketAddress对象包含了IP地址和端口信息。

#### 使用MulticastSocket实现多点广播 ####
DatagramSocket只允许数据报发送给指定的目标地址，MulticastSocket可以将数据报以广播方式发送到多个客户端。

IP协议为多点广播提供了特俗的IP地址，范围是224.0.0.0至239.255.255.255.

MulticastSocket是实现多点广播的关键，当MulticastSocket把一个DatagramPacket发送到多点广播的IP地址时，该数据报将被自动广播到加入到该地址的所有MulticastSocket。

MulticastSocket事实上是DatagramSocket的子类，因此其常用的构造器也类似：

- public MulticastSocket()
- public MulticastSocket(int port)
- public MulticastSocket(SocketAddress bindaddr)

将MulticastSocket加入和删除到指定的多点广播IP地址的方法：

- joinGroup(InetAddress multicastAddr)
- leaveGroup(InetAddress multicastAddr)

仅用于发送数据报的MulticastSocket对象，可以只使用默认地址和随机端口即可；
接受用的MulticastSocket对象必须具有指定端口，否则发送方无法确定发送数据报的目标端口。

MulticastSocket用于发送和接受数据报的方法与DatagramSocket的完全一样，但MulticastSocket多了一个setTimeToLive(int ttl)方法。ttl参数用于设置数据报最多可以跨过多少个网络：0表示应停留在本地主机；1（默认值）表示数据报发送到本地局域网；32表示只能发送的本站点的网络；64表示发送到本地区；128表示本大洲；255表示可以发送到所有地方。

使用MulticastSocket进行多点广播时，所有的通信实体之间都是平等的。

### 使用代理服务器 ###
Java 5开始，java.net包下提供了Proxy和ProxySelector两个类。Proxy代表一个代理服务器，可以在打开URLConnection连接时指定Proxy，创建Socket连接时也可以指定Proxy；而ProxySelector代表一个代理选择器，它提供了对代理服务器更灵活的控制，可以对HTTP，HTTPS，FTP，SOCKS等进行分别设置，还可以设置不需要通过代理服务器的主机和地址。

#### 直接使用Proxy创建连接 ####
使用构造器`Proxy(Proxy.Type type, SocketAddress sa)`，sa指定代理服务器的地址，type指定该代理服务器的类型：

- Proxy.Type.DIRECT
- Proxy.Type.HTTP
- Proxy.Type.SOCKS

URL包含了一个使用代理服务器建立连接的方法，Socket则提供了使用代理服务器的构造器：

- URLConnection openConnection(Proxy proxy)
- Socket(Proxy proxy)

#### 使用ProxySelector自动选择代理服务器 ####
ProxySelector代表一个代理选择器，本身是一个抽象类，开发者通过继承该类来实现自己的代理选择器。该类有2个抽象方法：

- List<Proxy> select(URI uri)
- connectFailed(URI uri, SocketAddress sa, IOException ioe)

使用ProxySelector的setDefault(Proxy proxy)静态方法来注册默认的代理选择器。系统默认的代理选择器在使用系统设置的代理服务器失败时，会采用直连的方式连接远程资源。它的select方法会根据系统属性来决定使用哪个代理服务器。和代理服务器有关的常用系统属性名：

- http.proxyHost
- http.proxyPort：这两个属性前面的http都可以更改为https，ftp等；
- http.nonProxyHosts：不需要使用代理服务器的主机，支持使用`*`通配符；支持多个地址，之间用`|`分隔

## Chapter 18 类加载机制和反射 ##

### 类的加载，连接和初始化 ###
#### JVM和类 ####
当Java命令运行某个Java程序时，也就启动了与一个Java虚拟机进程，该程序启动的所有线程都处于该虚拟机进程里。同一个JVM的所有线程，所有变量都处于同一个进程内，使用该JVM进程的内存区。

JVM进程在以下情况被终止：

- 程序运行到最后正常结束
- 程序运行到使用'System.exit()'或'Runtime.getRuntime().exit()'代码处结束程序
- 程序执行过程中遇到未捕获的异常或错误而结束
- 程序所在平台强制结束了JVM进程

JVM进程被终止后，该进程在内存中的状态将会丢失。2个JVM进程间不会共享数据。

#### 类的加载 ####
当程序主动使用某个类时，如果该类还未被加载到内存中，系统会通过加载，连接，初始化三个步骤来对该类初始化。系统也可以预先加载某些类。

类的加载指的是将类的class文件读入内存，并为之创建一个java.lang.Class对象。

类的加载由类加载器完成，JVM提供的类加载器称为系统类加载器。可以通过继承ClassLoader基类来创建自己的类加载器。

类的class文件通常来源于：

- 本地文件系统中的class文件
- JAR包中的class文件
- 通过网络加载
- 把一个Java源文件动态编译并加载

#### 类的连接 ####
类的连接负责把类的二进制数据合并到JRE中，分为三个阶段：

- 验证：检验被加载的类是否有正确的内部结构，并和其他类协调一致
- 准备：负责为类的类变量分配内存，并设置默认初始值
- 解析：将类的二进制数据中的符号引用替换成直接饮用

#### 类的初始化 ####
类的初始化是由虚拟机负责通过2中方式对类变量进行初始化：

- 声明类变量时指定初始值
- 使用静态初始化块为类变量指定初始值

JVM将按照这些语句在程序中的排列顺序依次执行。

JVM初始化一个类的步骤：

1. 假如这个类还没有被加载和连接，则程序先加载并连接该类
2. 如果这个类的直接父类还没有被初始化，则先初始化其直接父类
3. 如果类中有初始化语句，则系统依次执行这些初始化语句

所以JVM最先初始化的总是`java.lang.Object`类，而且会保证一个类的直接父类和间接父类都会被初始化。

#### 类的初始化时机 ####
当Java程序首次通过下面6种方式来使用某个类或接口时，系统会初始化该类或接口：

- 创建类的实例，包括使用`new`关键字，通过反射，或通过反序列化的方法来创建实例
- 调用某个类的类方法
- 调用某个类或接口的类变量，或为该类变量赋值
- 使用反射方式来强制创建某个类或接口对应的`java.lang.Class`对象，如`Class.forname("aClassName")`
- 初始化某个类的子类
- 直接使用java.exe命令来运行某个类

对于一个final类型的类变量，如果该变量的值在编译期间就可以确定，则该变量相当于一个常量，编译期间编译器会用直接量替换所有出现该变量的地方，因此如果程序只使用某个类的这种类型的变量，该类不会被初始化。

当使用ClassLoader类的loadClass()方法来加载某个类时，该方法只是加载该类，并不会执行该类的初始化，使用Class的forName()方法才会导致强制初始化该类。

### 类加载器 ###
类加载器负责将类的class文件加载到内存，并生成相应的`java.lang.Class`对象。

在JVM中确保所加载的类的唯一性的方法：使用类的全限定类名（包含了包名的类名）和其类加载器作为唯一标识。这意味着被两个不同的类加载器加载的同一个类在JVM中被当作是不同的两个类。

当JVM启动时，会形成由三个类加载器组成的初始类加载器层次结构：

- Bootstrap ClassLoader：引导类加载器/根类加载器，负责加载Java的核心类，并不是由Java实现的，因此并不是ClassLoader抽象类的实现类。
- Extension ClassLoader：扩展类加载器，是ClassLoader抽象类的实现类PlatformClassLoader类的实例，但从类加载器层次的角度看，其父类是根类加载器
- System ClassLoader：系统类加载器，是ClassLoader抽象类的实现类AppClassLoader类的实例

JVM的类加载机制主要有三种：

- 全盘负责：当一个类加载器负责加载某个类时，该类所依赖的和引用的其他类也将由该类加载器负责载入，除非显式使用另一个类加载器来载入。
- 父类委托：先尝试让parent类加载器加载该类，如果无法加载再尝试从自己的类路径中加载该类
- 缓存机制：保证所有已经加载过的类都会被缓存，当程序中需要使用某个类时，类加载器先在缓存区中搜寻该类，如果找不到再加载该类的类文件并转换为Class对象，放入缓存区中

类加载器之间的父子关系并不是类继承上的父子关系，而是类加载器实例之间的关系。即由父到子为：根类加载器<-扩展类加载器<-系统类加载器<-用户类加载器。

类加载器加载类文件的步骤：

1. 检测此类文件是否被载入过，即是否能在缓存区中找到该类，如果有则直接跳到第8步
2. 如果父类加载器不存在（即要么父类加载器是根类加载器，或者本身就是根类加载器），跳到第4步
3. 请求使用父类加载器载入目标类，如果成功就直接跳到第8步
4. 请求使用根类加载器载入目标类，如果成功就直接跳到第8步
5. 当前类加载器尝试寻找class文件，如果找到则执行第6步，找不到就跳到第7步
6. 从文件中载入class文件，成功就跳到第8步
7. 抛出`ClassNotFoundException`异常
8. 返回相应的`java.lang.Class`对象

#### 创建并使用自定义的类加载器 ####
JVM中除根类加载器外的所有类加载器都是`ClassLoader`子类的实例。

`ClassLoader`有两个关键方法：

- loadClass(String name, boolean resolve)
- findClass(String name)

`loadClass()`方法执行步骤如下：

1. 用findLoadedClass(Sting)来检查是否已经加载过此类，是就直接取出返回
2. 在父类加载器上调用loadClass()方法，如果父类加载器为null，就使用根类加载器加载
3. 调用findClass(String)方法查找类

实现自定义的类加载器需要重写以上两个方法，通常推荐重写`findClass()`方法而不是`loadClass()`方法。因为可以避免覆盖上述第1,2步中的默认类加载器的父类委托，缓冲机制两种策略

`ClassLoader`类中还有一个重要的final方法：

- Class defineClass(String name, byte[] b, int off, int len)：负责将指定类的字节码文件读入到字节数组b中并将他转换为Class对象。

`ClassLoader`的其他方法：

- findSystemClass(String name)：从本地文件系统装入文件，如果存在，就使用`defineClass()`方法将字节码文件转换为`Class`对象
- static getSystemClassLoader()：返回系统类加载器
- getParent()：获取该类加载器的父类加载器
- resolveClass(Class<?> c)：连接指定的类
- findLoadedClass(String name)：如果此虚拟机已加载了名为name的类，则直接返回该类对应的Class对象，否则返回null

使用自定义的类加载器可以：

- 执行代码前自动验证数字签名
- 根据用户提供的密码解密代码，避免反编译
- 根据用户需求来动态地加载类
- 根据应用需求把其他数据以字节码的形式加载到应用中

#### URLClassLoader类 ####
Java为`ClassLoader`提供了一个URLClassLoader实现类，这也是系统类加载器和扩展类加载器的父类。他有两个构造器：

- URLClassLoader(URL[] urls)
- URLClassLoader(URL[] urls, ClassLoader parent)

### 通过反射查看类信息 ###

解决编译时类型和运行时类型不同的方法：

- 如果知道编译时和运行时的类型的具体信息，可以使用`instanceof`运算符进行判断，再利用强制类型转换将其转换为运行时类型
- 如果编译时无法知道运行时类型的具体类型，则只能利用反射

#### 获得Class对象 ####
Java程序中获得Class对象的方法：

- 使用`Class`类的`forName(String className)`类方法，其中`className`是带完整包名的全限定类名，此方法可能抛出`ClassNotFoundException`异常
- 调用某个类的class属性来获取该类对应的Class对象，推荐使用
- 调用某个对象的`getClass()`方法

#### 从Class中获得信息 ####

用于获取构造器的4个方法：

- Constructor<T> getConstructor(Class<?>... parameterType)：返回带有指定形参列表的public构造器
- Constructor<?>[] getConstructors()：返回所有public构造器
- Constructor<T> getDeclaredConstructor(Class<?>... parameterType)：返回带有指定形参列表的所有构造器，无论权限
- Constructor<?>[] getDeclaredConstructors()：返回所有权限的构造器


用于获取方法的4个方法：

- Method<T> getMethod(String name, Class<?>... parameterType)：返回带有指定形参列表的public方法
- Method<?>[] getMethods()：返回所有public方法
- Method<T> getDeclaredMethod(String name, Class<?>... parameterType)：返回带有指定形参列表的所有方法，无论权限，但仅限在本类中声明的或者通过实现接口获得的方法，通过继承获得的方法不算在内。
- Method<?>[] getDeclaredMethods()：返回所有权限的方法，但仅限在本类中声明的或者通过实现接口获得的方法，通过继承获得的方法不算在内。

只能通过一个方法的方法名和其形参类型（不是形参名称）来唯一确定一个方法。


用于获取成员变量的4个方法：

- Filed getFiled(String name)：返回带有指定形参列表的public成员变量
- Filed[] getFileds()：返回所有public成员变量
- Filed getDeclaredFiled(String name)：返回带有指定形参列表的所有成员变量，无论权限，但仅限在本类中声明的成员变量，通过继承或者通过实现接口获得的成员变量不算在内。
- Filed[] getDeclaredFileds()：返回所有权限的成员变量，但仅限在本类中声明的成员变量，通过继承或者通过实现接口获得的成员变量不算在内。

注意通过`getDeclaredFiled()`和`getDeclaredMethod()`分别获得成员变量和方法时对通过接口获得的成员处理方法的不同：

可以获取通过接口声明的方法
无法获取通过接口声明的成员变量

用于获取Annotation的方法：

- <A extends Annotation> A getAnnotation(Class<A> annotationClass)
- <A extends Annotation> A getDeclaredAnnotation(Class<A> annotationClass)
- Annotation[] getAnnotations()
- Annotation[] getDeclaredAnnotations()
- <A extends Annotation> A[] getAnnotationsByType(Class<A> annotationClass)：用于重复注解
- <A extends Annotation> A[] getDeclaredAnnotationsByType(Class<A> annotationClass)：用于重复注解

用于获取内部类，外部类的方法：

- Class<?>[] getDeclaredClasses()：返回全部的内部类
- Class<?> getDeclaringClass()：返回所在的外部类

用于获取所实现的接口：

- Class<?>[] getInterfaces()

用于获取所继承的父类：

- Class<? super T> getSuperClass()

用于获取类的修饰符，所在包，类名等基础信息：

- int getModifiers()：返回的整数需要使用Modifier工具类的方法来解码
- Package getPackage()
- String getName()：带包名
- String getSimpleName()：只有类名

用于判断类的类型的方法：

- boolean isAnnotation()
- boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)
)
- boolean isAnonymousClass()
- boolean isArray()
- boolean isEnum()
- boolean isInterface()
- boolean isInstance(Object obj)：完全可以替代`instanceof`操作符

对于只在源代码级别保留的注解（RentionPolicy.SOURCE），使用运行时获得的Class对象无法访问到该注解对象

#### Java8新增的方法参数反射 ####
Method和Constructor是Java8在`java.lang.reflect`包中新增的`Executable`抽象基类的子类。提供了2个方法来获得方法形参：

- int getParameterCount()：返回形参的个数
- Parameter[] getParameters()：返回所有形参

`Parameter`类代表了方法的形参，每个实例代表一个形参，有如下常用方法：

- getModifiers()：返回修饰该形参的修饰符
- String getName()：返回形参名
- Type getParameterizedType()：返回带泛型的形参类型
- Class<?> getType()：返回形参类型
- boolean isNamePresent()：该类的class文件中是否包含了方法的形参信息
- boolean isVarArgs()：判断该参数是否为个数可变的形参

使用默认javac命令编译出的class文件不包含方法的形参信息，必须为javac命令添加`-parameters`选项

### 使用反射生成并操作对象 ###

通过Class对象可以获得该类的方法（Method对象），构造器（Constructor对象），成员变量（Filed对象），这三个类都处于`java.lang.reflec`包下，并实现了`java.lang.reflect.Member`接口。

#### 创建对象 ####
通过Class对象获得指定的Constructor对象，再调用Constructor对象的newInstance()方法来创建该类的实例。通常只有在编译时不知道需要创建什么类型的对象，需要在运行时动态地创建某个类的时候才会考虑使用反射。

#### 调用方法 ####
通过Class对象的`getMethods()`或`getMethod()`方法获得全部方法或指定方法，每个Method对象对应一个方法，再调用Method对象的`invoke()`方法来执行该方法

- Object invoke(Object obj, Object... args)：obj是执行该方法的实例，args是执行该方法时传入的实参。

当通过Method的`invoke()`方法来执行相应的方法时，要求程序必须有调用该方法的权限。如果程序确实需要调用某个对象的private方法，可以先调用Method对象的`setAccessible()`方法：

- setAccessible(boolean flag)：设置是否取消Java的访问权限检查。此方法`AccessibleObject`接口中定义，所以Method，Constructor，Filed都可以使用。

#### 访问成员变量 ####
通过Class对象的`getFileds()`或`getFiled()`方法获取全部public的成员变量或指定的public成员变量

- Xxx getXxx(Object obj)
- setXxx(Object obj, Xxx value)
- set(Object obj, Object value)

#### 操作数组 ####
在`java.lang.reflec`包下的Array类的对象可以代表所有类型的数组。

static Object newInstance(Class<?> componentType, int... length)
static Xxx getXxx(Object array, int index)
static Object get(Object array, int index)
static void setXxx(Object array, int index, Xxx val)
static void set(Object array, index, Object val)

### 使用反射生成JDK动态代理 ###
#### 使用Proxy和InvocationHandler创建动态代理 ####
如果在程序中为一个或多个接口动态地生成实现类，就可以使用Proxy来创建动态代理类；
如果需要为一个或多个接口动态地创建实例，也可以使用Proxy来创建动态代理实例。

- static Class<?> getProxyClass(ClassLoader loader, Class<?>... Interfaces)
- static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)

执行代理对象的每个方法时都会被替换执行InvocationHandler对象的invoke方法。系统生成的每个代理对象都有一个与之关联的InvocationHandler对象。

- proxy：代表动态代理对象
- method：代表正在执行的方法
- args：代表调用目标方法时传入的参数

#### 动态代理和AOP ####
JDK动态代理只能为接口创建动态代理

AOP(Aspect Orient Programming)面向切面编程：AOP代理可以代替目标对象，AOP代理包含了目标对象的全部方法，AOP代理中的方法月目标对象的方法存在差异，AOP代理里的方法可以在执行目标方法之前，之后插入一些通用处理。

### 反射和泛型 ###
String.class的类型就是Class<String>。如果Class对应的类型未知，则使用Class<?>

使用`getType()`方法获取非泛型类型，使用`getGenericType()`方法获取带泛型的类型，然后将Type对象强制转换为ParameterizedType对象，有以下两个方法：

getRawType()：返回没有泛型信息的原始类型
getActualTypeArguments()：返回泛型参数的类型

Type也是`java.lang.reflect`包下的一个接口，代表所有类型的公共公共高级接口。






















































