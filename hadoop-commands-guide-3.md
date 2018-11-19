# `HADOOP命令手册`

### <strong>`概述`</strong>

所有的`hadoop`命令和子项目都遵循以下基本格式：

```
shellcommand [SHELL_OPTIONS] [COMMAND] [GENERIC_OPTIONS] [COMMAND_OPTIONS]
```

其中`shellcommand`是例如`hadoop`,`hdfs`和`yarn`等命令

`SHELL_OPTIONS`是shell语句优先于Java执行的可选命令

`COMMAND`是要做的操作

`GENERIC_OPTIONS`是被多个命令所支持的选项的集合。

`COMMAND_OPTIONS`

##### `Shell选项`

所有的shell命令都接收一系列的选项。但是对于某些命令，这些选项会被忽略。例如，--hostnames在单主机上执行的命令是会被忽略的。

```
--config confdir  用于覆盖$HADOOP_HOME/etc/hadoop/这个路径。
--darmon mode  	对于不支持守护进程的命令，这个选项会忽略。
--debug	开启shell级别的日志
--help	shell脚本使用信息
--hostnames	当--workers使用了，覆盖workers文件使用一系列的空格分隔的hostname，如果--workers没有开启，这个就会被忽略。
--hosts	当--workers被使用了，覆盖workers通过另外一个文件，这个文件需要有一系列的hostnames的集合，如果--workers没有被使用，将会忽略。
--loglevel loglevel	覆盖日志等级，合法的日志等级有FATAL, ERROR, WARN, INFO, DEBUG, and TRACE. Default is INFO。
--workers	如果可能，执行这个命令在所有的worker节点上。
```

##### `通用选项`

许多子命令都支持一系列的选项来改变他们的行为。

```
-archives	指定存档，逗号分隔，只适用于Job
-conf	指定特定的配置文件
-D	使用所给定的属性，如key:value
-files	指定特定的文件复制到map reduce集群上，只对Job有用
-fs	<files://>或<hdfs://namenode:port>	指定默认的文件系统，会覆盖fs.defaultFS的值
-jt	指定ResourceManager，仅仅对Job生效
-libjars	逗号分隔的jar文件会被加载到classpath，仅仅对Job生效。
```
# `Hadoop常用命令`

所有的命令被切分到了`User Commands`和`Administrator Commands`里去了。

### `User Command`

```
archive
创建一个hadoop存档
```

```
checknative
-a	查看所有库是否可用
-h 打印帮助
查询hadoop本地代码的可用性
```

```
classpath
--global 扩展通配符
--jar path 将类路径以jar名称路径写入
-h	打印帮助
```

```
credential
用于安全管理
```

```
distch
一次性改变多个文件的所有者和权限
```

```
distcp
递归的复制目录文件
```

```
dtutil
用于获取安全凭证的工具
```

```
fs
hdfs dfs启用后的代名词
```
```
gridmix
是hadoop集群的标准工具
```
```
jar
运行一个jar文件，建议用yarn jar替换
```
```
jnipath
打印计算过的 java.library.path路径
```
```
kerbname
改变命名主题
```
```
key
通过KeyProvider管理keys
```

```
kms
Run KMS, the Key Management Server.
```
```
trace
查看和修改Hadoop跟踪设置
```
```
version
查看版本
```
```
CLASSNAME
运行一个类
```
```
envvars
展示计算过后的hadoop环境变量
```

### `Administrator Commands`

```
daemonlog
设置或获取守护进程的日志级别
```

### `Files`

```
etc/hadoop/hadoop-env.sh
这个文件存储了全局的设置，这些设置将会被hadoop的shell命令脚本所使用
```

```
etc/hadoop/hadoop-user-functions.sh
使得高级用户去覆盖一些shell功能
```

```
~/.hadooprc
保存着个人的环境对于个体用户,在前面两个文件之后处理，可以存在相同的配置。
```


