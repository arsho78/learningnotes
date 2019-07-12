在国际化时，如果使用`ResourceBundelMessageSource`来指定国际化资源，必须注意其默认的properties文件的编码是iso-8859-1，因此对于中文等语言必须使用native2ascii工具转换才能正确显示。另一种解决方法是将其默认的编码配置为UTF-8：

		<bean id="messageSource"
					class="org.springframework.context.support.ResourceBundelMessageSource">
			<property name="basename" value="message"/>
			<property name="defaultEncoding" value="UTF-8"/>
		</bean>

使用JSR303校验数据时，方法参数里的Errors参数必须声明在紧跟使用`@Valid`注解标注的参数后面，否则会出现400错误：

		public String validate(@Valid @ModelAttribute Customer customer,
													Errors errors,
													Modle model){
		....
		}


## Chapter 3 Spring MVC的常用注解 ##

### @Controller ###

完成以下2件事以保证Spring能够找到Controller类

- 在 SpringMVC 的配置文件的头文件中引入 spring-context
- 使用`<context:component-scan/>`元素，它的`base-package`属性指定了需要扫描的类包（子包也会被递归扫描）

#### SpringMVC 配置文件中： ####

- `<mvc:annotation-driven>`是一种简写形式，会自动注册`RequestMappingHandlerMapping`和`RequestMappingHandlerAdapter`两个Bean

- `<mvc:default-servlet-handler/>`是SpringMVC 的静态资源处理器，在`web.mxl`中，如果将`DispatcherServlet`请求映射配置为`/`，则 SpringMVC 将捕获包括静态资源的所有请求。
在引入类似`<script type="text/javascript" src="js/jquery-1.11.0.min.js"/>`这种静态资源文件时，`DispatcherServlet`会将`/`看做请求路径，找不到时就会导致提示404错误。
在配置文件中配置`<mvc:default-servlet-handler/>`之后，会在SpringMVC的上下文中定义一个`org.springframework.web.servlet.reousrce.DefaultServletHttpRequestHandler`，
它会对请求的URL进行筛查，静态资源的请求将转由Web应用服务器默认的Servlet处理，其他请求将由该`DispatcherServlet`继续处理。

- 视图解析器`InternalResourceViewResolver`负责解析视图，将View呈现给用户。它的`prefix`属性表示视图的前缀（所在目录路径），`suffix`属性表示视图的后缀（视图文件的扩展名）。

### 用于参数绑定的注解 ###

都在`org.springframework.web.bind.annotation`包中

共有六类：

- 处理请求参数和内容部分的注解：
	- `@RequestMapping`
	- `@RequestParam`
	- `@GetMapping`
	- `@PostMapping`
	- `@PutMapping`
	- `@DeleteMapping` 
	- `@PatchMapping`
	- `@RequestBody`
	- `@ResponseBody`
	- `@RequestPart`
	- `@restController`
- 处理请求URL部分的注解
	- `@PathVariable`
	- `@MatrixVariable`
	- `@CrossOrigin`
- 处理请求头部分的注解
	- `@RequestHeader`
	- `@CookieValue`
- 处理属性类型的注解
	- `@RequestAttribute`
	- `@SessionAttribute`
	- `@SessionAttributes`
	- `@ModelAttribute`
- 处理异常类型的注解
	- `@ExceptionHandler`
	- `@ControllerAdvice`
	- `@RestControllerAdvice`
	- `@ResponseStatus`
- 绑定表单数据的注解
	- `@InitBinder`

### @RequestMapping ###

`@RequestMapping`可以用于类或方法。

用于类时，相当于为该类中所有被`@RequestMapping`注解的方法所对应的映射路径添加了一个前缀。
例如用于该类的`@RequestMapping`的value值是`/user`，其中某个方法的`@RequestMapping`注解的value值是`/login`，则该方法实际对应的URL是`/user/login`。

`@RequestMapping`注解支持的属性：

属性     | 类型            | 是否必需
-------- | ---------       | -----------
value    | String[]        | no
name     | String[]        | no
method   | RequestMethod[] | no
consumes | String[]        | no
produces | String[]        | no
params   | String[]        | no
headers  | String[]        | no
path     | String[]        | no

- `value`属性负责将URL映射到方法或类上，只有一个字符串做为属性值时，默认是此属性的值；属性值是空字符串时，该方法被映射到该类/应用的根路径；
- `method`属性指示该方法仅处理哪些HTTP请求方式，若无指定，则默认处理所有方式；
- `consumes`属性指定处理的请求的提交内容类型（Content-Type）；
- `produces`属性指定返回的内容类型（必须是request请求头Accept中所包含的类型）
- `params`属性指定request中必须包含指定的参数值，才由该方法处理
- `headers`属性指定request中必须包含指定的header值，才由该方法处理

请求处理方法可以接受的形参类型：

- `javax.servlet.ServletRequest`或`javax.servlet.http.HttpServletRequest`
- `javax.servlet.ServletResponse`或`javax.servlet.http.HttpServletResponse`
- `javax.servlet.http.HttpSession`
- `org.springframework.web.context.request.WebRequest`或`org.springframework.web.context.request.NativeWebRequest`
- `java.util.Locale`
- `java.io.InputStream`或`java.io.Reader`
- `java.io.OutputStream`或`java.io.Writer`
- `java.security.Principal`
- `HttpEntity<?>`
- `java.util.Map`
- `org.springframework.ui.Model`
- `org.springframework.ui.ModelMap`
- `org.springframework.web.servlet.mvc.support.RedirectAttributes`
- `org.springframework.validation.Errors`
- `org.springframework.validation.BindingResult`
- `org.springframework.web.bind.support.SessionStatus`
- `org.springframework.web.util.UriComponentsBuilder`
- `@PathVariable`, `@MatrixVariable`注解
- `@RequestParam`, `@RequestHeader`, `@RequestBody`, `@RequestPart`注解

`WebRequest`是SpringMVC提供的同意请求访问接口，不仅可以访问请求的相关数据，还可以访问请求作用域和会话作用域中的数据，但不能访问Cookie区的数据。有如下重要方法：

- `getParameters(String paramName)`
- `getHeader(String headerName)`
- `setAttribute(String attName, Object value, int scope)`
- `getAttribute(String attName, int scope)`

`scope`的值应该是在`WebRequest`中定义的两个常量`SCOPE_REQUEST`和`SCOPE_SESSION`

请求处理方法可以返回的类型：

- `org.springframework.web.portlet.ModelAndView`：使用`addObject`方法添加模型，使用`setViewName`方法来指定视图
- `org.springframework.ui.Model`：使用`addAttribute`方法添加模型，范围是在`requestScope`
- `java.util.Map<k, v>`
- `org.springframework.web.servlet.View`
- `java.lang.String`
- `HttpEntity或ResponseEntity`
- `java.util.concurrent.Callable`
- `org.springframework.web.context.request.async.DeferredResult`
- `void`

SpringMVC在内部使用`org.springframework.ui.Model`接口来存储模型数据，在调用请求处理方法前会创建一个隐含的模型对象，作为模型数据的存储容器，如果处理方法的形参是`Model`或者`ModelAndView`，则会把隐含模型的引用作为实参传入。

#### 页面转发 ####

请求转发有两种：

- 服务器内部跳转（forward）
- 客户端重定向（redirect）

根据目的地的不同，又可以分为转发到JSP页面或其他请求处理方法。

当请求处理方法返回的是字符串时：

- 默认跳转到JSP页面：`return "welcome"`，跳转到`welcome.jsp`页面
- 重定向到JSP页面：`return "redirect:/welcome.jsp"`
- 跳转到其他请求处理方法：`return "forward:/welcome"`，跳转到被映射为`welcome`的处理方法
- 重定向到其他请求处理方法：`return "redirect:/welcome"`

当请求处理方法返回的是`ModelAndView`类型对象`mv`时

- 跳转到JSP页面：`mv.setViewName("welcome.jsp")`，跳转到`welcome.jsp`页面
- 重定向到JSP页面：`mv.setViewName("redirect:/welcome.jsp")`
- 跳转到其他请求处理方法：`mv.setViewName("forward:/welcome")`，跳转到被映射为`welcome`的处理方法
- 重定向到其他请求处理方法：`mv.setViewName("redirect:/welcome")`

客户重定向相当于在浏览器重新发送请求，所以不能访问`WEB-INF`下的资源文件，此时SpringMVC配置文件中通过视图解析器设置的前缀和后缀都将无效。

### @RequestParam注解 ###

`org.springframework.web.bind.annotation.RequestParam`将指定的请求参数赋值给方法中的形参。

属性         | 类型      | 是否必需
--------     | --------- | -----------
name         | String    | no
value        | String    | no
required     | boolean   | no
defaultValue | String    | no

如果通过`name`或`value`指定的请求参数不存在，则会产生异常。

### @PathVariable注解 ###

`org.springframework.web.bind.annotation.PathVariable`获取请求参数中的动态参数


属性         | 类型      | 是否必需
--------     | --------- | -----------
name         | String    | no
value        | String    | no
required     | boolean   | no

		@RequestMapping(value="/pathVariableTest/{userId}")
		public void pathVariableTest(@PathVariable Integer userId)

`{userId}`在这里相当于参数的占位符，在实际发送的请求中会提取该位置的实际值并将其绑定在被`@PathVariable`修饰的方法同名参数上。

### @MatrixVariable注解 ###

`org.springframework.web.bind.annotation.MatrixVariable`扩展URL请求功能，允许多个变量使用分号隔开。在Spring MVC中默认是不启动的，需要在配置文件中开启该功能：`<mvc:annotation-driven enable-matrix-variables="true"/>

属性         | 类型      | 是否必需
--------     | --------- | -----------
name         | String    | no
value        | String    | no
pathVar      | String    | no
required     | boolean   | no
defaultValue | String    | no

		// 映射的请求为matrixVariableTest/{userId};name=jack;age=23
		@GetMapping("/matrixVariableTest/{userId}")
		public void matrixVariableTest(@PathVariable Integer userId,
																	 @MatrixVariable(value="name", pathVar="userId") String name,
																	 @MatrixVariable(value="age", pathVar="userId") Integer age)

### @CrossOrigin注解 ###

`org.springframework.web.bind.annotation.CrossOrigin`用于处理跨域请求。类上和方法上的`@CrossOrigin`注解会合并使用。


属性             | 类型            | 是否必需
--------         | ---------       | -----------
allowCredentials | String          | no
allowedHeaders   | String[]        | no
exposedHeaders   | Stringp[]       | no
maxAge           | long            | no
methods          | RequestMethod[] | no
origins          | String[]        | no
value            | String[]        | no

### @RequestHeader注解 ###

`org.springframework.web.bind.annotation.RequestHeader`用于将请求的头信息映射到方法的参数上。

属性         | 类型      | 是否必需
--------     | --------- | -----------
name         | String    | no
value        | String    | no
required     | boolean   | no
defaultValue | String    | no

### @CookieValue注解 ###

`org.springframework.web.bind.annotation.CookieValue`注解用于将请求的Cookie数据映射到方法的参数上。

属性         | 类型      | 是否必需
--------     | --------- | -----------
name         | String    | no
value        | String    | no
required     | boolean   | no
defaultValue | String    | no

### @RequestAttribute注解 ###

`org.springframework.web.bind.annotation.RequestAttribute`注解用于访问有请求处理方法，过滤器或拦截器创建的，预先存于request作用域中的属性，并将该属性的值映射到方法的参数上。

属性         | 类型      | 是否必需
--------     | --------- | -----------
name         | String    | no
value        | String    | no
required     | boolean   | no


### @SessionAttribute注解 ###

`org.springframework.web.bind.annotation.SessionAttribute`注解用于访问有请求处理方法，过滤器或拦截器创建的，预先存于session作用域中的属性，并将该属性的值映射到方法的参数上。

属性         | 类型      | 是否必需
--------     | --------- | -----------
name         | String    | no
value        | String    | no
required     | boolean   | no

### @SessionAttributes注解 ###

`org.springframework.web.bind.annotation.SessionAttributes`可以有选择性地将Model中的属性转存到`HttpSession`对象中。只能用来标注类，不能用于标注方法。

属性     | 类型       | 是否必需
-------- | ---------  | -----------
name     | String[]   | no
value    | String[]   | no
types    | Class<?>[] | no

`types`属性指定需要放入`HttpSession`对象中的类型。

### @ModelAttribute注解 ###

`org.springframework.web.bind.annotation.ModelAttribute`注解用于将请求参数绑定到对象。


属性     | 类型      | 是否必需
-------- | --------- | -----------
value    | String    | no

如果需要为`Model`添加多个属性，可以在方法内部使用`Model`对象的`addAttribute`方法添加。如果没有指定`value`，而注解的方法返回的是一个具体类，则`Model`属性的名称被隐式指定为由该类名小写而得到的名称。

被`@ModelAttribute`注解标注的方法会在`Controller`类里每个方法执行前被执行，因此在一个`Controller`类被映射到多个URL时，要小心使用。

当`@ModelAttribute`和`@RequestMapping`同时标注一个返回`String`类型的方法时，该方法返回的不是视图的名称，而是`model`属性的值，该属性的名称由`@ModelAttribute`的`value`值指定；视图名称则由`@RequestMapping`的`value`的值指定。

当`@ModelAttribute`用于标注方法的一个参数时，会将前台页面控件的值（如Spring表单中设置了`path`属性的`input`控件）自动传递给Model对象中与该注解所标注的方法参数同名的属性，如果该属性不存在，则先自动创建该属性。






































