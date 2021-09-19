# hive 设置

基本的安装还是使用 brew 来进行，主要介绍后续的设置。

## 创建文件夹

参考 [官方起始文档][1]，我们需要创建指定的文件夹并赋予权限。

```sh
hadoop fs -mkdir       /tmp
hadoop fs -mkdir       /user/hive/warehouse
hadoop fs -chmod g+w   /tmp
hadoop fs -chmod g+w   /user/hive/warehouse
```

但是经过我的尝试，可能需要进行如下的权限设置，否则在后续进行数据库操作时会出现 `Permission denied`。

```sh
hadoop fs -chmod -R 777 /user/hive
hadoop fs -chmod 777 /tmp
```

参考 [Permission Denied error while creating database in hive][2]

## 设置 mysql 和 hive 配合

### 创建数据库

首先需要安装好 mysql，这里就不赘述了，注意进行设置之前先启动服务。然后创建一个数据库供 hive 使用，并赋予 hive 用户对应的权限。

```sql
CREATE DATABASE metastore;
USE metastore;
CREATE USER 'hive'@'localhost' IDENTIFIED BY 'hive';
GRANT ALL ON metastore.* TO 'hive'@'localhost';
```

### 下载驱动

前往 [MySQL Community Downloads][5]，选择 Platform Independent，下载压缩包，解压出 jar 文件，复制到 `/usr/local/Cellar/hive/3.1.2_3/libexec/lib` 文件夹下。

### 配置 hive-site.xml

主要是设置数据库的连接，注意驱动选择 `com.mysql.cj.jdbc.Driver`。

> Loading class \`com.mysql.jdbc.Driver'. This is deprecated. The new driver class is \`com.mysql.cj.jdbc.Driver'.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://localhost/metastore</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.cj.jdbc.Driver</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hive</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>hive</value>
  </property>
</configuration>
```

### 设置 Hadoop 的 core-site.xml

参考 [启动 hiveserver2 服务后使用 beeline 报错 Could not open client transport with JDBC Uri: jdbc:hive2://localhost:100][8]

我们需要对于访问的用户进行权限设置，在配置文件中加入如下的部分（注意 root 换成你当前的用户名）

```xml
<property>
    <name>hadoop.proxyuser.root.hosts</name>
    <value>*</value>
</property>
<property>
    <name>hadoop.proxyuser.root.groups</name>
    <value>*</value>
</property>
```

## 启动

```sh
schematool -dbType mysql -initSchema
hiveserver2
beeline -u jdbc:hive2://localhost:10000
```

请等到 hiveserver2 启动后日志显示 4 行 `Hive Session ID`，再通过 beeline 进行连接，否则可能会出现如下的错误：

> 21/09/19 12:24:23 [main]: WARN jdbc.HiveConnection: Failed to connect to localhost:10000
>
> Could not open connection to the HS2 server. Please check the server URI and if the URI is correct, then ask the administrator to check the server status.
>
> Error: Could not open client transport with JDBC Uri: jdbc:hive2://localhost:10000: java.net.ConnectException: Connection refused (Connection refused) (state=08S01,code=0)

## 退出

请使用 ctrl + c 进行退出，exit 和 quit 都是无效的。

[1]: https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-RunningHive
[2]: https://stackoverflow.com/a/32211881/15548365
[3]: https://zhuanlan.zhihu.com/p/55798418
[4]: https://gist.github.com/akuksin/42e9848baa491b74c6c1c7e659de9bce
[5]: https://dev.mysql.com/downloads/connector/j/
[6]: https://stackoverflow.com/q/33201551/15548365
[7]: https://stackoverflow.com/a/25515367/15548365
[8]: https://blog.csdn.net/weixin_43713105/article/details/111932704
