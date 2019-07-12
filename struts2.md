struts2老版本的convention插件不支持Java11，需要升级asm才能正确映射Java11编写的Action类

struts.convention.action.packages和struts.convention.package.locators.basePackage，以及struts.convention.package.locators在寻找映射Action类时的作用：

Action类必须同时满足以下2个方面的要求才会被自动映射：

- Action类本身（满足以下条件之一）
	- 该类实现了`com.opensymphony.xwork2.Action`接口
	- 该类以`struts.convention.action.suffix`指定的后缀名结尾，默认是`Action`
- Action类所在的包（以下条件按优先顺序排列，前者优先级更高）
	-  不是`struts.convention.exclude.packages`指定的包及其子包 
	-  是`struts.convention.action.packages`指定的包或其子包
	- 的全限定包名必须同时满足以下条件
		- 以`struts.convention.package.locators.basePackage`指定的包名开头，如果该常量没有设置，则忽略此条件
		- 包含`struts.convention.package.locators`中指定的任一包名，默认是`action, actions, struts, struts2`
	
使用convention的注解时，结果指向的页面最终地址是`@ResultPath + @NameSpace + @Result(location)`

一旦使用convention插件，在struts.xml文件中无法对该插件的约定进行修改，如result逻辑返回视图和物理视图的对应关系，必须使用注解在action类文件中修改。

package级别指定的namespace不会传给它的子包

一、EL简介

1.语法结构

${expression}

2.[]与.运算符

EL 提供.和[]两种运算符来存取数据。
当要存取的属性名称中包含一些特殊字符，如.或?等并非字母或数字的符号，就一定要使用 []。例如：
${user.My-Name}应当改为${user["My-Name"] }
如果要动态取值时，就可以用[]来做，而.无法做到动态取值。例如：
${sessionScope.user[data]}中data 是一个变量

3.变量

EL存取变量数据的方法很简单，例如：${username}。它的意思是取出某一范围中名称为username的变量。
因为我们并没有指定哪一个范围的username，所以它会依序从Page、Request、Session、Application范围查找。
假如途中找到username，就直接回传，不再继续找下去，但是假如全部的范围都没有找到时，就回传null。
属性范围在EL中的名称
Page PageScope
Request RequestScope
Session SessionScope
Application ApplicationScope

在使用'struts2'的'ModelDriven'机制时，如果驱动用的对象是'List'或'Map'类型，可以在页面使用'model'来引用该对象

		public class LoginAction implements ModelDriven<Map<String, User>>{...}

		--- 以下是网页内容 ---

		<s:textfield name="model['one'].name"/>


使用自定义的类型转换器，必须为要转换的`Action`类型的每一个属性提供setter方法，即便它可能是模型驱动的，如实现了`ModelDriven<User>`的`Action`类肯定有一个`User`类型的属性，所以必须提供该属性的setter方法。如果需要转换模型中的某个属性，需要在模型所在包中提供类型转换文件。

使用struts2.xml文件配置类型转换文件时，发现更改modelDriven.refreshModelBeforeResult参数值不起作用，甚至设置action的result也不起作用，以下代码不起作用。直接在Action类中使用注解反而有效。

		<package name="myInterceptor" namespace="/section4_01" extends="struts-default">
			<action name="loginto" class="net.xiaoluo.practice.struts2demo.actions.section4_01.LogintoAction">
				<interceptor-ref name="defaultStack">
					<param name="modelDriven.refreshModelBeforeResult">true</param>
				</interceptor-ref>
				<result>/WEB-INF/content/section4_01/login-success.jsp</result>
			</action>
		</package>

由于项目迁移，struts2的校验文件的文件头已发生改变，从原来的：

		<!DOCTYPE validators PUBLIC
			"-//OpenSymphony Group//XWork Validator 1.0.3//EN"
			"http://www.opensymphony.com/xwork/xwork-validator-1.0.3.dtd">

改变为：

		<!DOCTYPE validators PUBLIC
			"-//Apache Struts/XWork Validator 1.0.3//EN"
			"http://struts.apache.org/dtds/xwork-validator-1.0.3.dtd">


Struts2.5.14中的內建校验器`conversion`的`repopulateField`属性似乎不起作用，在struts2网站上有issue track: https://issues.apache.org/jira/browse/WW-3540
