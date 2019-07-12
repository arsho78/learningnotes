根据mysql网站的指南使用apt仓库来安装mysql。

安装完成后使用`sudo`或使用`su`命令以系统root身份运行登录进入mysql；

进入mysql后可以使用`select user,plugin from mysql.user;`命令看到mysql的root默认以`auth_socket`方式登录，使用以下命令将其改为使用传统的密码方式登录：

		update mysql.user set authentication_string=PASSWORD('newPwd'),plugin='mysql_native_password' where user='root';
		flush privileges;

重启服务，此时一般系统用户也可以使用mysql的root登录mysql了

## 更改mysql数据目录 ##

1. 停止mysql服务

		sudo systemctl stop mysql

2. 将当前数据目录拷贝到新的数据目录位置

		sudo cp -ra /var/lib/mysql /data/mysql

3. 修改`my.cnf`文件，mysql寻找该文件的先后顺序：`/etc/my.cnf`，`/etc/mysql/my.cnf`，`/usr/etc/my.cnf`，`~/.my.cnf`，以找到的第一个文件为准。
在`etc/mysql/my.cnf`中使用了`!includedir /etc/mysql/mysql.conf.d/`来包含该目录下`mysqld.cnf`配置文件中的内容。因此可以在该文件中设置数据目录的位置（当然也可以在`my.cnf`中直接设置）。

		...
		datadir=/data/mysql
		...

如果使用了apparmor来控制对mysql数据目录的访问，则还要执行如下操作：

4. 修改`/etc/apparmor.d/usr.sbin.mysqld`和`/etc/apparmor.d/abstractions/mysql`文件，将其中的旧数据目录更改为新的数据目录。

数据目录更改完成。

重启服务，如果出现`ERROR 2002（HY000）`错误，可能是新数据目录的权限出现了问题（如果拷贝时使用了-a参数，应该不存在该问题）。

## 连接MySQL数据库时显示时区错误 ##

		Caused by: java.sql.SQLException: The server time zone value 'EDT' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.

解决方法1：在连接数据库的地址的最后加上`serverTimezone=America/New_York`:

		jdbc:mysql://localhost:3306/webDemo/test.jsp?serverTimezone=America/New_York

解决方法2：登录进入MySQL，使用如下命令设置数据库的全局时区：

		set global time_zone = 'America/New_York';

如果提示错误`'America/New_York'不是合法的时区`，则应先在shell中使用如下命令将时区信息导入到数据库中：

		mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root -p mysql

然后再执行上面的set命令。

如果不想每次重启MySQL服务都执行一次上面的命令，可以修改`/etc/mysql/my.cnf`，在其中的`[mysqld]`分区中加入如下配置：

		default-time-zone='Amercia/New_York'

## 更新数据库到最新的5.7版本时，mysql.user数据表出现多余列 ##

更新到最新的版本后，不能创建新的用户，报错，列数不对，期待45列，实际找到48列，mysql.user数据表可能损坏，要求使用`mysql_upgrade`修复。

按照要求使用命令`mysql_upgrade -uroot -p --force`时，发现仍然有错误提示说数据表的列数不正确（但没有显示是哪张表），最后通过和网上可用的user表比较，手工删掉了`max_statement_time, default_role, is_role`三个数据列后恢复正常。

## 创建新用户 ##

使用`create`语句创建了新用户之后，可以通过`show grants`查看用户的权限。使用`grant`语句赋予用户权限。
