# Flume 设置

当你按照 [User Guide][1] 跑第一个程序的时候可能会跑不起来，这里有一些问题的解决方案。

## jar 包冲突

> SLF4J: Class path contains multiple SLF4J bindings.
>
> SLF4J: Found binding in \[jar:file:/C:/Users/admin/.m2/repository/org/slf4j/slf4j-log4j12/1.6.4/slf4j-log4j12-1.6.4.jar!/org/slf4j/impl/StaticLoggerBinder.class]
>
> SLF4J: Found binding in \[jar:file:/C:/Users/admin/.m2/repository/org/slf4j/slf4j-log4j12/1.6.1/slf4j-log4j12-1.6.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]

这是由于 Hadoop 和 flume 引用了相同的 jar 包导致的，你可以去 flume 的对应文件夹下删除 jar 包即可。值得一提的是，这种情况很常见，后续安装部分遇到不再赘述。

## 内存不足

> Exception in thread "main"

当你看到如下的错误时，请将默认的 `flume-env.sh` 中的一行注释去掉。

```sh
# Give Flume more memory and pre-allocate, enable remote monitoring via JMX
export JAVA_OPTS="-Xms100m -Xmx2000m -Dcom.sun.management.jmxremote"
```

执行命令时请仿照如下：

```sh
flume-ng agent --conf /usr/local/Cellar/flume/1.9.0_1/libexec/conf --conf-file example.conf --name a3 -Dflume.root.logger=INFO,console
```

## 其它

请注意这里的环境应该是设置好 java 和 Hadoop 的环境变量之后才进行的操作，故这里不对该环境变量进行设置。

[1]: https://flume.apache.org/releases/content/1.9.0/FlumeUserGuide.html
