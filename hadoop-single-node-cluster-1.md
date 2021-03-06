# `Hadoop单节点`

### `目的`

目的是能快速搭建单节点的HADOOP，然后就能够使用MapReduce和HDFS的简单操作。

### `前置条件`
#### `操作系统`
```
GNU/Linux系统，Hadoop可在Linux上部署2000节点
&
Windows系统也是支持的
```
#### `其他环境`

```
Java JDK8
&
ssh
```
#### `安装ssh和pdsh`

```
sudo apt-get install ssh
sudo apt-get install pdsh
```
### `下载合适的hadoop版本(hadoop-x.x.x.tar.gz)`
### `准备开启Hadoop集群`

解压`hadoop-x.x.x.tar.gz`,编辑`etc/hadoop/hadoop-env.sh`去定义`JAVA_HOME`。

```
export JAVA_HOME=/usr/java/latest
```

进入`Hadoop`的解压目录，运行`bin/hadoop`查看`Hadoop`的所有命令。

目前为止，已经可以开启你的`Hadoop`集群，通过一下三种形式:

* 本地(Standalone)模式。
* 伪分布式模式。
* 全分布式模式。

### `单机模式操作`

默认，`hadoop`被配置为非分布式模式，作为一个单独的`Java`进程。这对于调试很有用。

以下例子将复制`etc/hadoop/*.xml`文件到`input`文件夹作为输入，输出将会被输入到`output`文件夹。

配置输入

```
$ mkdir input
$ cp etc/hadoop/*.xml input/
```
运行小例子，并显示结果

```
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.3.jar grep input output 'dfs[a-z.]+'
$ cat output/*		
//result: 1	dfsadmin
```

### `伪分布式模式操作`

`Hadoop`支持在`伪分布式`的情况下运行单节点。每一个守护进程都是一个独立的`Java`进程。

#### `配置`

- `etc/hadoop/core-site.xml`

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <!--配置临时文件夹的路径-->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/lllyyyggg/tmp</value>
    </property>
</configuration>
```
表示默认的defaultFS的地址为`http://localhost:9000`

- `etc/hadoop/hdfs-site.xml`

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <!--配置namenode和datanode临时文件夹-->
	<property>
       <name>dfs.namenode.name.dir</name>
       <value>/home/hadoop/lllyyyggg/tmp/dfs/name</value>
	</property>
    <property>
       <name>dfs.datanode.data.dir</name>
       <value>/home/hadoop/lllyyyggg/tmp/dfs/data</value>
    </property>
</configuration>
```
表示数据在hdfs中只保存一份。

#### `设置免ssh密码登录`

先试试看你本地需要密码不。

```
$ ssh localhost
```
如果你不能在本地免密登录，执行以下命令。

```
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```

#### `执行`

接下来将要执行本地的`MapReduce`作业。当然作业也可以用YARN调度，这里不做讲解。

- 1 格式化文件系统
 
```
$ bin/hdfs namenode -format			
```	
- 2 开启`NameNode`进程和`DataNode`进程

```
$ sbin/start-dfs.sh
```
接下来会看到

```
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes [Slave1]
```
使用jps命令查看NameNode和DataNode是否已经开启

```
$ jps
```

```
33907 Jps
33748 SecondaryNameNode
33542 DataNode
33423 NameNode
```
说明守护进程都开启成功。

> 如果出现DataNode没启动，很可能是`-format`的时候保留了原来的`DataNodeId`,导致ID不一致，那么此时可以删除`dfs.datanode.data.dir`下的数据，重新格式化，重新启动就OK了。

`Hadoop`的日志会被输出到`$HADOOP_LOG_DIR`目录(默认是`$HADOOP_HOME/logs`)。

- 3 此时在浏览器里面访问NameNode的接口，默认是

```
NameNode - http://localhost:9870/
```
- 4 使HDFS目录执行MapReduce工作。

```
$ bin/hdfs dfs -mkdir /user
$ bin/hdfs dfs -mkdir /user/<username>
```
> 如果这里出现`Call from xxx to localhost:9000 failed on connection exception`，一般是因为`NameNode`启动不正常。也就是`$ sbin/start-dfs.sh`.

- 5 将input文件夹复制到hdfs中

```
$ bin/hdfs dfs -mkdir input
$ bin/hdfs dfs -put etc/hadoop/*.xml input
```
> 如果在创建文件夹或者复制文件的时候出现文件夹找不到的情况，`$ bin/hadoop fs -mkdir -p input`

- 6 开始执行提供的例子

```
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.3.jar grep input output 'dfs[a-z.]+'
```

此时的input就是hdfs中的input,output就是hdfs中的output了。

- 7 测试输出文件夹：将output文件夹从分布式文件系统复制到本地系统，然后进行查看。

```
$ bin/hdfs dfs -get output output
$ cat output/*
```
查看分布式文件系统中的输出文件夹的结果。

```
$ bin/hdfs dfs -cat output/*
```

- 8 当你完成后，停止守护进程。

```
$ sbin/stop-dfs.sh
```

> 注意的是，如果你想使用Standalone模式，你必须注释掉以下配置。

```
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
</property>
```
#### `在单节点的伪分布式模式下使用YARN`

你可以在伪分布式模式下基于`YARN`运行`MapReduce`任务，仅仅通过设置一些参数并同时运行`ResourceManager`守护进程和`NodeManager`守护进程。

接下来的命令假设上面`执行`的指令的前4步已经执行了。

- 1 首先配置参数如下：

`etc/hadoop/mapred-site.xml`
 
```
 <configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```	
> 不过在此之前必须将`HADOOP_HOME`配置到`/etc/profile`中去，`export HADOOP_HOME=xxx`，然后`$ source /etc/profile`使环境变量生效。

`etc/hadoop/yarn-site.xml`

```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```
- 2 开启ResourceManager和NodeManager

```
$ sbin/start-yarn.sh
```

> 此处如果ResourceManager和NodeManager没有开启，请到`log/xxx-resourcemanager-xx.log`查看日志。如果是`Illegal to have multiple roots`错误，那么就可能是你的配置文件里面有两个根目录如`<configuration>`。

- 3 在浏览器的web接口浏览ResourceManager，默认的浏览路径为

```
ResourceManager - http://localhost:8088/
```

- 4 执行MapReduce任务

```
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.3.jar grep input output 'dfs[a-z.]+'
```
然后你会看到结果

```
...
2018-11-18 10:54:38,326 INFO mapreduce.Job:  map 0% reduce 0%
2018-11-18 10:54:46,413 INFO mapreduce.Job:  map 0% reduce 100%
2018-11-18 10:54:46,422 INFO mapreduce.Job: Job job_1542509260827_0002 completed successfully
2018-11-18 10:54:46,510 INFO mapreduce.Job: Counters: 40
...
```

通过如下命令查看任务执行结果

```
bin/yarn application -status application_1542509260827_0002
```

会有如下显示

```
Application Report : 
	Application-Id : application_1542509260827_0002
	Application-Name : grep-sort
	Application-Type : MAPREDUCE
	User : hadoop
	Queue : default
	Application Priority : 0
	Start-Time : 1542509662974
	Finish-Time : 1542509685077
	Progress : 100%
	State : FINISHED
	Final-State : SUCCEEDED
	Tracking-URL : http://Slave1:19888/jobhistory/job/job_1542509260827_0002
	RPC Port : 35825
	AM Host : Slave1
	Aggregate Resource Allocation : 55010 MB-seconds, 29 vcore-seconds
	Aggregate Resource Preempted : 0 MB-seconds, 0 vcore-seconds
	Log Aggregation Status : DISABLED
	Diagnostics : 
	Unmanaged Application : false
	Application Node Label Expression : <Not set>
	AM container Node Label Expression : <DEFAULT_PARTITION>
	TimeoutType : LIFETIME	ExpiryTime : UNLIMITED	RemainingTime : -1seconds
```


- 5 当你执行完毕，关闭守护进程

```
$ sbin/stop-yarn.sh
```
### `小结`
到此单节点上运行Standalone模式和伪分布式模式已经讲完了，大家应该有比较深刻的理解了。

主要的进程有如下:

* `NameNode`
* `DataNode`
* `SecondaryNameNode`
* `ResourceManager`
* `NodeManager`