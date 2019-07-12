如果在`.m2/settings.xml`或项目的`pom.xml`中设置了repository，而在IntelliJ IDEA中`Settings/Maven/Repositories`中该repository显示不可用，有error，update提示`FileNotFoundException: Resource nexus-maven-repository-index.properties does not exist in IntelliJ`，则说明在nexus服务器上忘记发布索引文件了，登录nexus服务器，在`system/task`中新建一个`Maven - Publish Maven Indexer files`任务并立刻执行，返回IntelliJ，update成功。

webapp程序的`pom.xml`文件中应该指定打包方式为war，否则在IntelliJ中不会自动为其创建web子模块和artifact



## 创建自己的archetype组 ##

第一步：创建父项目xiaoluo-archetype-bundles

主目录下3个文件：

		.
		├── .gitignore
		├── plugin-versions.properties
		└── pom.xml

`pom.xml`文件内容如下：

		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
			<modelVersion>4.0.0</modelVersion>

			<groupId>net.xiaoluo.archetypes</groupId>
			<artifactId>xiaoluo-archetype-bundles</artifactId>
			<version>1.0-SNAPSHOT</version>
			<packaging>pom</packaging>

			<name>XiaoLuo's Archetypes</name>
			<description>Archetypes created by XiaoLuo</description>
			<url>https://xiaoluo.net</url>

			<distributionManagement>
				<repository>
					<id>net.xiaoluo.local.release</id>
					<name>Xiao's Release Repository</name>
					<url>http://localhost:8090/repository/net-xiaoluo-releases/</url>
				</repository>
				<snapshotRepository>
					<id>net.xiaoluo.local.snapshots</id>
					<name>Xiao's Snapshot Repository</name>
					<url>http://localhost:8090/repository/net-xiaoluo-snapshots/</url>
				</snapshotRepository>
			</distributionManagement>

			<properties>
				<maven.archetype.plugin.version>3.0.1</maven.archetype.plugin.version>
				<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
				<maven.compiler.source>11</maven.compiler.source>
				<maven.compiler.target>11</maven.compiler.target>
			</properties>

			<build>
				<extensions>
					<extension>
						<groupId>org.apache.maven.archetype</groupId>
						<artifactId>archetype-packaging</artifactId>
						<version>${maven.archetype.plugin.version}</version>
					</extension>
				</extensions>
				<resources>
					<resource>
						<directory>src/main/resources</directory>
					</resource>
					<resource>
						<directory>src/main/resources-filtered</directory>
						<filtering>true</filtering>
					</resource>
				</resources>
				<filters>
					<filter>../plugin-versions.properties</filter>
				</filters>
				<pluginManagement>
					<plugins>
						<plugin>
							<groupId>org.apache.maven.plugins</groupId>
							<artifactId>maven-archetype-plugin</artifactId>
							<version>${maven.archetype.plugin.version}</version>
							<configuration>
								<ignoreEOLStyle>true</ignoreEOLStyle>
							</configuration>
						</plugin>
						<plugin>
							<groupId>org.apache.maven.plugins</groupId>
							<artifactId>maven-resources-plugin</artifactId>
							<configuration>
								<addDefaultExcludes>false</addDefaultExcludes>
								<escapeString>\</escapeString>
							</configuration>
						</plugin>
					</plugins>
				</pluginManagement>
			</build>
		</project>

`plugin-versions.properties`内容如下：

		clean     3.1.0
		site      3.7.1
		install   2.5.2
		deploy    2.8.2
		resources 3.1.0
		compiler  3.8.0
		surefire  2.22.1
		jar       3.1.1
		ejb       3.0.1
		plugin    3.6.0
		war       3.2.2
		ear       3.0.1
		rar       2.4
		archetype 3.0.1
		invoker   3.1.0
		pir       3.0.0
		javadoc   3.1.0
		testng    7.0.0-beta3

第二步：创建子项目xiaoluo-archetype-archetype

该项目用于创建一个archetype项目，即创建archetype项目的archetype项目。

此时，父项目目录如下：

		.
		├── plugin-versions.properties
		├── pom.xml
		└── xiaoluo-archetype-archetype
		   ├── pom.xml
		    └── src
		        └── main
		            ├── resources
		            │   ├── archetype-resources
		            │   │   └── src
		            │   │       └── main
		            │   │           ├── resources
		            │   │           │   ├── archetype-resources
		            │   │           │   │   └── src
		            │   │           │   │       ├── main
		            │   │           │   │       │   └── java
		            │   │           │   │       │       └── App.java
		            │   │           │   │       └── test
		            │   │           │   │           └── java
		            │   │           │   │               └── AppTest.java
		            │   │           │   └── META-INF
		            │   │           │       └── maven
		            │   │           │           └── archetype-metadata.xml
		            │   │           └── resources-filtered
		            │   │               └── archetype-resources
		            │   │                   └── pom.xml
		            │   └── META-INF
		            │       └── maven
		            │           └── archetype-metadata.xml
		            └── resources-filtered
		                └── archetype-resources
		                    └── pom.xml

## 从本地仓库中移除artifact ##

- 进入需要移除的artifact的项目目录，如`~/works/project/archetype-demo`。（注意不是本地仓库目录下的artifact目录，如`~/.m2/repository/demo/archetype-demo`）；
- 运行`mvn build-helper:remove-project-artifact`完成从本地仓库中删除artifact内容；
- 编辑`~/.m2/repository/archetype-catalog.xml`，删除相应的artifact条目。这样才能把该项目从archetype列表中删除（使用`mvn archetype:generate`时显示的列表）。

## 在生成的archetype中包含指定文件 ##

在创建自定义archetype时，如果需要在生成的项目中包含一些默认会排除的文件，如`.gitignore`，则需要做以下设置：

1. 在`archetypedir/pom.xml`文件中加入：

		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>	
				<artifactId>maven-resources-plugin</artifactId>
				<configuration>
					<addDefaultExcludes>false</addDefaultExcludes>
				</configuration>
			</plugin>
		</plugins>

2. 在`archetypedir/src/main/resources/META-INF/maven/archetype-metadata.xml`中加入下列代码：

		<fileSets>
			<fileSet>
				<!-- indicate the directory where locates the .gitignore -->
				<directory></directory>
				<includes>
					<include>.gitignore</include>
				</includes>
			</fileSet>
		</fileSets>

完成设置后，运行`mvn clean install`即可使用新生成的archetype来创建项目了。

## 如何读取classpath和resources目录下的文件 ##

    // get file from classpath, resources folder
    private File getFileFromResources(String fileName) {

        ClassLoader classLoader = getClass().getClassLoader();

        URL resource = classLoader.getResource(fileName);
        if (resource == null) {
            throw new IllegalArgumentException("file is not found!");
        } else {
            return new File(resource.getFile());
        }

    }
		
参考：
https://kodejava.org/how-do-i-load-file-from-resources-directory/
https://howtodoinjava.com/java/io/read-file-from-resources-folder/
