# `HDFS架构`

HDFS可运行在廉价的硬件上。高容错，高吞吐，大数据。

愿景:

- 从硬件失败快速恢复
- 流数据处理数据集，批处理。
- 大数据集
- 简单一致性模型，MapReduce程序和网络爬虫非常实用
- 容易移动
- 容易移植

`NameNode` and `DataNodes`：

主从架构。一个`NameNode`和多个`DataNode`。文件被切分多份，保存在一系列`DataNode`上。

架构如下：

![](https://hadoop.apache.org/docs/r3.0.3/hadoop-project-dist/hadoop-hdfs/images/hdfsarchitecture.png)

NameNode管理和数据分块的所有决策。NameNode和DataNode通过心跳保持连接。接收心跳代表DataNode健康。

![](https://hadoop.apache.org/docs/r3.0.3/hadoop-project-dist/hadoop-hdfs/images/hdfsdatanodes.png)

