## JSP/Servlet及相关技术详解 ##

一个典型的tomcat下的web应用文件结构如下：

		webDemo           //web应用的名称
		├── <a.jsp> 			//可以有多个JSP文件
		└── WEB-INF
				├── classes   //存放web应用所需的Java类文件
				│   └── xiao
				│       └── Person.class
				├── lib       //存放web应用所需的Java类文件，与classes目录的区别是这里存放的是.jar文件
				├── src       //web应用需要的Java类文件的源文件
				│   └── xiao
				│       └── Person.java
				└── web.xml

`WEB-INF`文件夹是一个特殊文件夹，web容器会包含该文件夹下的内容，但客户端浏览器无法访问该文件夹下的任何内容。其中的`web.xml`文件被称为配置描述符，负责配置和管理如下内容：

- 配置JSP
- 配置和管理Servlet
- 配置和管理Listener
- 配置和管理Filter
- 配置标签库
- 配置JSP属性
- 配置和管理JAAS授权认证
- 配置和管理资源引用
- web应用首页

`web.xml`文件的根元素是`<web-app.../>`元素，Servlet3中加入了`metadata-complete`属性，该属性设置为true时，web应用不会加载注解配置的web组件。

使用`welcome-file-list`元素设置web应用的首页：

		<!-- 配置web应用的首页列表 -->
		<welcome-file-list>
			<welcome-file>index.html</welcome-file>
			<welcome-file>index.htm</welcome-file>
			<welcome-file>index.jsp</welcome-file>
		</welcome-file-list>

每个web容器都会提供一个系统的web.xml文件，用于描述所有web应用共同的配置属性。

### JSP的基本原理 ###

JSP页面的内容由如下2部分组成：

- 静态部分：标准的HTML标签，静态的页面内容。这些内容与静态HTML页面相同。
- 动态部分：受Java程序控制的内容，由Java脚本动态生成。

JSP的本质依然是Servlet，每个JSP页面就是一个Servlet实例。因为web应用中的每个JSP页面都会由Servlet容器生成对应的Servlet。生成的Servlet文件放置在tomcat目录下的`work/Catalina/localhost/jspPrinciple/org/apache/jsp`。

- JSP文件必须在JSP服务器内运行；
- JSP文件必须生成Servlet才能执行；
- 第一次访问一个JSP页面的速度会有些慢，因为必须等待JSP编译成Servlet；
- JSP页面的访问者无须安装任何客户端，也不需要可以运行Java运行环境，因为输送到客户端的是标准HTML页面。

#### JSP注释 ####

		<%-- JSP注释内容 --%>
		<!-- HTML注释内容 -->

HTML注释可以通过源代码查看到，JSP注释无法通过源代码查看，这也意味着JSP注释在编译成Servlet时已被抛弃，不会被发送到客户端。

#### JSP声明 ####

		<%！声明部分%>

JSP声明语法定义的变量和方法对应于Servlet类的成员变量和方法，所以这些变量和方法可以使用`private`，`public`等访问控制符修饰，也可以使用`static`修饰符将其变成类属性和方法，但不能使用`abstract`修饰声明的方法。

JSP页面被编译成一个Servlet类，每个Servlet在容器中只有一个实例；在JSP中声明的变量是成员变量，成员变量只在创建实例时初始化，变量的值一直保存到实例被销毁。

#### JSP输出表达式 ####

		<%=expression%>

输出表达式语法可以用来代替`out.println()`输出语句。语句后不能有分号。

#### JSP小脚本 ####

		<%脚本内容%>

小脚本的内容将转换为Servlet里`_jspService`方法里的可执行代码。因此在小脚本内部声明的变量是局部变量，不能使用`private`，`public`等访问控制符来修饰，也不能使用`static`修饰。小脚本里也不能定义方法。

### JSP的3个编译指令 ###

常见的编译指令有3个：

- `page`：针对当前页面的指令；
- `include`：指定包含另一个页面；
- `taglib`：定义和访问自定义标签。

使用编译指令的语法格式：

		<%@ 编译指令名 属性名="属性值"...%>

#### `page`指令 ####

		<%@page
		[language="Java"]
		[extend="package.class"]
		[import="package.class | package.*, ..."]
		[session="true | false"]
		[buffer="none | 8KB | size Kb"]
		[autoFlush="true | false"]
		[isThreadSafe="true | false"]
		[info="text"]
		[errorPage="relativeURL"]
		[contentType="mimeType[;charset=characterSet]" | "text/html;charset=ISO-8859-1"]
		[pageEncoding="ISO-8859-1"]
		[isErrorPage="true | false"]
		%>

`page`指令通常位于JSP页面的顶端，一个JSP页面可以使用多条`page`指令。

JSP不处理异常，包括checked异常。如果JSP页面在运行时抛出未处理的异常，系统自动跳转到`errorPage`属性指定的页面，如果没有指定该页面，则会把异常栈信息直接呈献给客户端浏览器。

#### `include`指令 ####

		<%@include file="relativeURLSpec"%>

`include`指令可以将一个外部文件嵌入到当前JSP文件中。这是静态的include语句，会把目标页面的其他编译指令也包含进来，如果两个页面的编译指令冲突，页面就会出错，动态include则不会。

include既可以包含静态的文本，也可以包含动态的JSP页面。静态的include编译指令会在编译时把被包含的页面合并到当前页面，形成一个页面，因此被包含页面不需要是一个完成的页面。
































































6 directories, 17 files
