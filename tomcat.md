tomcat的web应用的目录结构：

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

源文件和类文件必须放在一个包的下面，否则报错。

手工拷贝目录来部署web应用时，需要注意更改该目录的权限配合tomcat使用。


###Permission denied error from java.io.File.createTempFile###


***Resolution***

1. Find out if the JVM parameter -Djava.io.tmpdir is set on the java process 

On Linux: 

	a. Run this command 

		ps -ef | grep java

	b. Review the JVM parameters searching for -Djava.io.tmpdir

On Windows, Linux, or Unix:   

 a. Go to http://aem-host:aem-port/system/console/jmx/java.lang%3Atype%3DRuntime

 b. Search for java.io.tmpdir on the page.

2. Copy the value of java.io.tmpdir to the clipboard.

3. Go to that path on the Operating System and grant the user that owns the java process full read/write access to that folder.

If you didn't find a java.io.tmpdir parameter, then grant the user access to the default OS temp directory.  In Linux and Unix, this directory is /tmp by default.  In Windows, the directory is under the user's home directory, for example, C:\Users\aemuser\AppData\Local\Temp

### 查看tomcat日志 ###

在tomcat的安装目录下的logs目录中有一个`catalina.out`文件，如`/usr/local/tomcat/logs/catalina.out`，使用命令`tail -f catalina.out`可以查看最后的运行记录。
