# `Hadoop全分布式`

### `一 目的`

使你能够安装并且将Hadoop部署在多节点以至于更多的上千节点上。

### `前置条件`

```
Java JDK
& 
Hadoop-x.x.x.tar.gz
```

### `二 安装`

安装Hadoop集群首先要把文件打包到集群中所有的机器上并解压。根据硬件划分功能是非常重要的。

通常有一台机器为`NameNode`，另一台被指定为`ResourceManager`，这些被称为`Master`节点。其他服务，例如Web应用代理服务器和`MapReduce Job History Server`，通常要不是运行在专用的硬件上就是运行在共享的基础设施上，取决于负载。

其他的集群中的机器通常同时充当`DataNode`和`NodeManager`。这是两个工作线程。

##### `修改/etc/hostname和/etc/hosts`

在每台主机上修改这两个文件，其中/etc/hostname是修改主机名。而/etc/hosts是配置主机关系。

```
115.157.200.78  master
115.157.200.79  slave-001
115.157.200.80  slave-002
```
然后使用如下命令失效:

```
$ service network-manager restart
```
##### `然后将hadoop-x.x.x.tar.gz复制到所有机器上`

```
$ scp hadoop-x.x.x.tar.gz hadoop@slave-001:/home/lllyyyggg/
```

##### `然后在master和所有slave之间建立ssh互信`

- 在所有机器上运行`$ ssh-keygen -t rsa`
- 接下来将master的公匙放到authorization_keys里。`sudo cat id_rsa.pub >> authorization_keys`
- 将authorization_keys放到其他机器的～/.ssh目录下`$ sudo scp authorization_keys hadoop@slave-001:~/.ssh/`
- 然后将authorized_key的权限修改为644 `$ chmod 644 authorized_key`
- 然后`ssh slave-001`，首次需要输入密码，然后exit，然后再次`ssh slave-001`，不需要密码就成功了。

##### `在每台服务器上都配置JAVA_HOME`

```
$ sudo vim /etc/profile
&
export JAVA_HOME=/xxx
&
$ source /etc/profile
```

##### `在每台服务器上都配置HADOOP_HOME`
```
$ sudo vim /etc/profile
&
export HADOOP_HOME=/xxx
&
$ source /etc/profile
```

##### `然后开始在所有机器上配置HDFS,YARN和MAPREDUCE，必须是一样的`

具体所有的配置都在下第三部分。



### `三 配置Hadoop在非安全模式`

`Hadoop`的Java配置分为两个重要方面，这两个方面对应的配置文件如下:

- 默认只读配置 - `core-default.xml`, `hdfs-default.xml`, `yarn-default.xml` 和 `mapred-default.xml`.

- 取决于站点的配置 - `etc/hadoop/core-site.xml`, `etc/hadoop/hdfs-site.xml`, `etc/hadoop/yarn-site.xml` 和 `etc/hadoop/mapred-site.xml`.

另外，你可以控制Hadoop在bin/directory里的脚本，一般是通过在`etc/hadoop/hadoop-env.sh`和`etc/hadoop/yarn-env.sh`中设定站点定制的值。

配置Hadoop集群，你将需要去配置环境，在这个环境里，Hadoop守护进程在这个环境`environment`里运行且配置的参数也都在该环境里生效。

`HDFS`的守护进程有`DataNode`,`NameNode`和`SecondaryNameNode`。YARN守护进程有`ResourceManager`,`NodeManager`和`WebAppProxy`。如果MapReduce被使用，MapReduce Job History Server也将会启动。对于大集群的安装，这些进程通常是运行在不同的主机上的。

##### `配置Hadoop守护进程的environment`

管理员必须在必选的`etc/hadoop/hadoop-env.sh`和可选的`etc/hadoop/mapred-env.sh`和`etc/hadoop/yarn-env.sh`脚本中做站点定制的参数配置，以此作为Hadoop的运行环境。

最少的配置就是，你必须在每台机器上指定`JAVA_HOME`。

管理员可以配置每个守护进程通过使用如下配置选项。

|守护进程|环境变量|
|:------:|:------:|
|NameNode|HDFS_NAMENODE_OPTS|
|DataNode|HDFS_DATANODE_OPTS|
|Secondary NameNode|HDFS_SECONDARYNAMENODE_OPTS|
|ResourceManager|YARN_RESOURCEMANAGER_OPTS|
|NodeManager|YARN_NODEMANAGER_OPTS|
|WebAppProxy|YARN_PROXYSERVER_OPTS|
|Map Reduce Job History Server|MAPRED_HISTORYSERVER_OPTS|

例如，配置NameNode使其使用parallelGC和4G的Java堆内存，你需要在hadoop-env.sh中配置:

```
export HDFS_NAMENODE_OPTS="-XX:+UseParallelGC -Xmx4g"
```
你可以到`etc/hadoop/hadopp-env.sh`中查看其他例子。

其他有用的参数还有如下:

- HADOOP_PID_DIR - 守护进程ID文件的存储路径
- HADOOP_LOG_DIR - 守护进程日志文件存储目录。日志文件如果不存在会自动创建。
- HADOOP_HEAPSIZE_MAX - 最大使用的Java堆内存。这个值可以被每个守护进程的基础配置覆盖掉，同时用合适的上面所提到的定制的_OPTS变量。例如，设置`HADOOP_HEAPSIZE_MAX=1g`并且设置`HADOOP_NAMENODE_OPTS="-Xmx5g"`将会配置NameNode 5G的堆内存。

在大多数情况下，你需要指定`HADOOP_PID_DIR` 和 `HADOOP_LOG_DIR`，那么这样一来，他们只能被将使用`Hadoop`守护进程的使用者写入。否则，可能出现符号链接攻击(`symlink attack`)。

通常你也需要配置HADOOP_HOME，在系统级别的环境配置文件中，如:/etc/profile.d。

```
HADOOP_HOME=/path/to/hadoop
export HADOOP_HOME
```

然后通过`$ source /etc/profile`执行生效。

##### `配置守护进程`

这个小节涉及一些重要的配置参数，在特定的配置文件里。

- `etc/hadoop/core-site.xml`

|参数|参数值|说明|
|------|------|------|
|fs.defaultFS|NameNode访问地址|`hdfs://host:port/`|
|io.file.buffer.size|131072|序列文件中读写缓冲区的大小|

- `etc/hadoop/hdfs-site.xml`	

	- `配置NameNode`

	|参数|参数值|说明|
	|------|------|------|
	|dfs.namenode.name.dir|本地系统NameNode用来存储namespace和事务持久化日志的目录|如果这是逗号分隔的目录列表，则为了冗余，将名称表复制到所有目录中。|
	|dfs.hosts/dfs.hosts.exclude|允许/不允许的DataNodes|如果可以，用这个文件控制允许的DataNode|
	|dfs.blocksize|268435456|HDFS中对于大文件系统的块大小|
	|dfs.namenode.handler.count|100|更多NameNode线程用来处理从大量DataNode来的RPCs|

	- `配置DataNode`

	|参数|参数值|说明|
	|------|------|------|
	|dfs.datanode.data.dir|用逗号分隔的本地系统的路径的集合，用于存储数据块|如果是逗号分隔，那么数据会被存储在所有目录中，通常在不同的设备上|

- `etc/hadoop/yarn-site.xml`

	- `配置ResourceManager和NodeManager`
	
	|参数|参数值|说明|
	|------|------|------|
	|yarn.acl.enable|true/false|默认是false,开启ACL|
	|arn.admin.acl|Admin ACL|默认是*，如果是特定值或空格，以为着没人有权限|
	|yarn.log-aggregation-enable|false|配置日志是否聚合|
	
	- `配置ResourceManager`
	
	|参数|参数值|说明|
	|------|------|------|
	|yarn.resourcemanager.address|host:port，用于客户端提交作业|如果设置了，将会覆盖hostname在yarn.resourcemanager.hostname中设置的值|
	|yarn.resourcemanager.scheduler.address|host:port，用于Masters和调度者进行沟通去获取资源|如果设置了，将会覆盖hostname在yarn.resourcemanager.hostname中设置的值|
	|yarn.resourcemanager.resource-tracker.address|host:port，用于DataNode使用|如果设置了，将会覆盖hostname在yarn.resourcemanager.hostname中设置的值|
	|yarn.resourcemanager.admin.address|host:port，用于管理员命令|如果设置了，将会覆盖hostname在yarn.resourcemanager.hostname中设置的值|
	|yarn.resourcemanager.webapp.address|host:port，用于Web-UI|如果设置了，将会覆盖hostname在yarn.resourcemanager.hostname中设置的值|
	|yarn.resourcemanager.hostname|ResourceManager的主机名|暂无|
	|yarn.resourcemanager.scheduler.class|调度者类型|CapacityScheduler (recommended), FairScheduler (also recommended), or FifoScheduler. Use a fully qualified class name, e.g., org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler.|
	|yarn.scheduler.minimum-allocation-mb|每个容器请求的最小内存限制|使用兆MBs的形式|
	|yarn.scheduler.maximum-allocation-mb|每个容器请求的最大内存限制|使用兆MBs的形式|
	|yarn.resourcemanager.nodes.include-path / yarn.resourcemanager.nodes.exclude-path|允许或不允许的NodeManagers|如果可以，使用这些文件去控制NodeManagers|

	- `配置NodeManager`	
	
	|参数|参数值|说明|
	|------|------|------|
	|yarn.nodemanager.resource.memory-mb|NodeManager可用物理内存|总共可用的资源是的NodeManager运行容器|
	|yarn.nodemanager.vmem-pmem-ratio|虚拟内存最大超过实际内存的大小的比例|无|
	|yarn.nodemanager.local-dirs|用逗号隔开的用于本地文件系统存储中间文件的目录|无|
	|yarn.nodemanager.log-dirs|本地文件系统日志存储目录，逗号分离|无|
	|yarn.nodemanager.log.retain-seconds|10800|日志集成disabled之后默认的用来在NodeManager上获取日志文件的时间|
	|yarn.nodemanager.remote-app-log-dir|/logs|HDFS存储日志目录，应用程序完成后会将日志从本地移到该目录|
	|yarn.nodemanager.remote-app-log-dir-suffix|logs||
	|yarn.nodemanager.aux-services|mapreduce_shuffle|对于MapReduce应用的Shuffle服务|
	|yarn.nodemanager.env-whitelist|系统属性从NodeManagers中继承的|对于MapReduce程序，HADOOP\_MAPRED\_HOME应该被添加，有这么多JAVA\_HOME,HADOOP\_COMMON\_HOME,HADOOP\_HDFS\_HOME,HADOOP\_CONF_DIR,CLASSPATH\_PREPEND\_DISTCACHE,HADOOP\_YARN\_HOME,HADOOP\_MAPRED\_HOME|
	
	- `配置History Server`

	|参数|参数值|说明|
	|------|:------:|------|
	|yarn.log-aggregation.retain-seconds|-1|集成日志的保存时间，-1表示禁止，这里要小心，如果设置的小了，就会将namenode垃圾化|
	|yarn.log-aggregation.retain-check-interval-seconds|-1|check继承日志保留的时间间隔，如果设置负数或者0将会是1/10的保留时间，太小的话会垃圾化namenode|

- etc/hadoop/mapred-site.xml

   - `配置MapReduce应用程序`

   	|参数|参数值|说明|
	|------|:------:|------|
	|mapreduce.framework.name|yarn|执行框架设为yarn|
	|mapreduce.map.memory.mb|1536|对于maps更大资源的限制|
	|mapreduce.map.java.opts|-Xmx1024M|maps子虚拟机更大的堆内存|
	|mapreduce.reduce.memory.mb|3072|reducer更大资源的限制|
	|mapreduce.reduce.java.opts|-Xmx2560M|reducers子虚拟机更大堆内存|
	|mapreduce.task.io.sort.mb|512|数据排序更大内存|
	|mapreduce.task.io.sort.factor|100|merge更多的stream在做排序文件的时候|
	|mapreduce.reduce.shuffle.parallelcopies|50|更大的并行拷贝由reducers去获取output从更大的数量的maps|
	
	- `配置MapReduce JobHistory Server`

	|参数|参数值|说明|
	|------|:------:|------|
	|mapreduce.jobhistory.address|host:port|默认是10020|
	|mapreduce.jobhistory.webapp.address|Web UI host:port|默认是19888|
	|mapreduce.jobhistory.intermediate-done-dir|/mr-history/tmp|历史文件存储目录，历史文件由MapReduce任务产生|
	|mapreduce.jobhistory.done-dir|/mr-history/done|历史文件被MR JobHistory Server管理的目录|
	
### `四 监测NodeManagers的健康`

Hadoop提供一种机制，能够周期性的使NodeManagers运行管理员提供的脚本，去查看NodeManager是否健康。

检测到不健康后需要在标准输出中打印`ERROR`字样，然后这个NodeManager就是`unhealthy`的状态。然后这个`NodeManager`就会被`ResourceManager`拉入黑名单，并且管理员在WEB-UI上是可以看到的，不仅可以看到不健康的内容，健康的内容日志也是可以看到的。	
接下来的参数可以用来控制节点的健康监控脚本，在`etc/hadoop/yarn-site.xml`中配置。

|参数|参数值|说明|
|------|:------:|------|	
|yarn.nodemanager.health-checker.script.path|Node Health Script的路径|无|
|yarn.nodemanager.health-checker.script.opts|Node Health Scipt的可选配置|无|
|yarn.nodemanager.health-checker.interval-ms|执行间断时间|无|
|yarn.nodemanager.health-checker.script.timeout-ms|执行等待最大时长|无|

健康检测脚本不应该给出ERROR，如果仅仅是本地磁盘损坏。NodeManager有权限去周期性的检查本地磁盘的健康(nodemanager-local-dirs and nodemanager-log-dirs)和达到坏目录数量的阈值后，整个节点将会被标记为`unhealthy`状态。不管磁盘是突袭还是失败在启动磁盘的时候都会被脚本检测出来。

### `五 Slave文件`

用来列出所有的工作节点的hostname和ip地址，在`etc/hadoop/workers`文件中，帮助脚本将会使用`etc/hadoop/workers`文件去执行命令在一些主机上马上。它将不会对于任何基于Java的Hadoop配置。为了去使用这个功能，`ssh`互信必须建立起来。

### `六 Hadoop机架意识`

许多Hadoop组件都是有机架意识的，能够充分利用网络拓扑结构的到性能和安全上去。Hadoop守护进程获取机架信息通过唤起一个管理员配置的模块。这里就不做详细说明了，更多到官网去看更加详细的说明。

### `七 日志`

Hadoop利用Log4j来打印日志。可以编辑`etc/hadoop/log4j.properties`文件去定制守护进程的日志配置。

### `八 操作Hadoop集群`

当所有的必要的配置都完成之后，将文件分发到`HADOOP_CONF_DIR`目录下在集群中所有机器中，这个目录在所有机器上必须是一样的。

总体来说建议HDFS和YARN为不同的用户来运行。在主要的安装中，HDFS操作以`hdfs`用户来运行。YARN通常使用`yarn`账户来运行。

##### `开启HADOOP`

开启Hadoop集群，你需要同时开启HDFS和YARN集群。

- 1 首次使用HDFS的时候必须进行`格式化`。格式化一个新的文件系统as hdfs
	
```
$ bin/hdfs namenode -format <cluster_name>
```

- 2 开启HDFS NameNode通过以下命令在指定的节点上as hdfs

```
$ bin/hdfs --daemon start namenode
```

- 3 开启HDFS DataNode通过以下命令在每个指定的DataNode节点上as hdfs

```
$ bin/hdfs --daemon start datanode
```

- 4 如果`etc/hadoop/workers`和ssh互信配置好了之后，所有的HDFS进程就能够被开启通过工具脚本as hdfs。


```	
$ sbin/start-dfs.sh
```

- 5 开启YARN通过以下命令，开启ResourceManager以yarn用户身份。

```
$ bin/yarn --daemon start resourcemanager
```
- 6 开启NodeManager在每台指定的主机上以yarn身份。

```
$ bin/yarn --daemon start nodemanager
```
- 7 开启WebAppProxy，使用yarn的身份运行。如果多台服务器都被使用了负载均衡，那么每台上面都要运行以yarn身份。

```
$ bin/yarn --daemon start proxyserver
```
- 8 如果`etc/hadoop/workers`和ssh互信配置了，所有的YARN进程就能通过以下脚本进行开启以yarn身份。

```
$ sbin/start-yarn.sh
```

- 9 开启MapReduce JobHistory Server通过以下命令，以mapred身份。

```
$ bin/mapred --daemon start historyserver
```

### `九 Web Interfaces`

一旦你的Hadoop集群开启之后，你可以通过以下web-ui来访问不通组件:

|进程|web-ui|说明|
|------|:------:|------|
|NameNode|nm:port|默认是9870端口|
|ResourceManager|rm:port|默认是8088端口|
|MapReduce JobHistory Server|jhs:port|默认是19888端口|