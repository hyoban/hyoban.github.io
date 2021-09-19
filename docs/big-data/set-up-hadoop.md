# 在 macOS 上搭建 hadoop 环境

参考官方的 [起步文档][1]，推荐的系统为 Linux，但是 macOS 环境类似，应该也是可以尝试的。

## brew install hadoop

使用 [Homebrew][2] 来进行一键安装。注意它会依赖于 openjdk，但是 brew 会默认安装最新版本，而参考 [推荐版本][3]，最合适的是 java 8，所以请安装 [openjdk8][4]。安装完成后，如果你需要让系统识别安装的 jdk，请运行：

```sh
sudo ln -sfn $(brew --prefix)/opt/openjdk@8/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-8.jdk
```

这会在 JavaVirtualMachines 目录下创建软链接到你的 jdk 下，然后你的 IDEA 就可以识别到这个 jdk 了。而只是配置环境的话，我们环境变量里面设置一下 JAVA_HOME 即可

```sh
export JAVA_HOME="/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home"
```

另外必须有 ssh 的环境，macOS 一般默认自带，需要在设置选项 Sharing 中开启允许远程登录。以及安装 [pdsh][5] 来更好的管理 ssh 的资源。

## 初始化设置

对于 brew ，Hadoop 的安装目录是 `usr/local/Cellar/hadoop/<version>/libexec/etc/hadoop`

### 设置 JAVA_HOME

参考 `hadoop-env.sh` 文件，在 macOS 上我们不需要设置此项。

> The java implementation to use. By default, this environment variable is REQUIRED on ALL platforms except OS X!

### 其它

参考 [官方文档][1] 修改 `core-site.xml` 和 `hdfs-site.xml`，设置 ssh 免密连接。初始化格式文件系统：

```sh
hdfs namenode -format
```

### 设置快捷指令

仿照如下来设置启动和关闭 Hadoop 的快捷指令，添加到你的环境变量中。

```sh
alias hstart="/usr/local/Cellar/hadoop/3.3.1/sbin/start-dfs.sh"
alias hstop="/usr/local/Cellar/hadoop/3.3.1/sbin/stop-dfs.sh"
```

## 警告消除（可选）

启动完成之后你大概会看到如下的警告提示：

`2021-09-18 16:22:19,280 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable`

它并不影响正常工作，消除警告需要自己在 macOS 上编译源码，参考 [问题定位][8] 和 [Build Hadoop source code on macOS][7]。我就不编译了，消除强迫症的方法是解决的方法更加麻烦。

## 其它可能出现的错误

> Hadoop cluster setup - java.net.ConnectException: Connection refused

参考 [stackoverflow][9]，我们需要先关闭，然后重新格式化 namenode。

## 总结

如此一个可用的 hdfs 就搭建完成了，使用 `hstart` 启动之后可以访问 `http://localhost:9870/` 来查看效果，总体上比在 Linux 上还是方便不少。关于更多设置和使用，请看官方文档

[1]: https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html
[2]: https://formulae.brew.sh/formula/hadoop
[3]: https://cwiki.apache.org/confluence/display/HADOOP/Hadoop+Java+Versions
[4]: https://formulae.brew.sh/formula/openjdk@8
[5]: https://formulae.brew.sh/formula/pdsh
[6]: https://stackoverflow.com/a/51905683/15548365
[7]: https://medium.com/@zekexu/build-hadoop-source-code-on-macos-3f932780fd84
[8]: https://stackoverflow.com/q/19943766/15548365
[9]: https://stackoverflow.com/a/42281292/15548365
