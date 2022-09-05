# Hadoop环境搭建

环境为WIN10+VMWare，配置4个节点，一主三从。

### 1.配置虚拟机

做完下述操作后可以保证虚拟机使用NAT时都在192.168.1.x这个网段：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled.png)

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%201.png)

再修改windows本地网卡：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%202.png)

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%203.png)

### 2.配置Linux环境

准备一个Centos8+Java8的环境，这里我为了美观还添加了zsh：

首先用yum下载安装好git、zsh，然后跟着[Oh My Zsh官网](https://ohmyz.sh/#install)指示慢慢下载安装，如果失败可以换国内源，我用的“bira”主题。

准备JDK1.8，将openJdk包解压后注意正确设置环境路径即可：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%204.png)

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%205.png)

最后将虚拟机完整克隆3次，得到4个节点，通过命令`ifconfig`，得到节点地址为：

192.168.1.6~192.168.1.9

在windows本地host文件中配置上述节点，并在powershell中测试是否ping通。

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%206.png)

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%207.png)

如果可以的话，此刻就只需要用xftp和xshell去取代vmware窗口操作了。

### 3.配置Hadoop集群

首先，我们给四个节点分配角色：

| node1 | nameNode | resourceManager |
| --- | --- | --- |
| node2 | secondaryNameNode、dataNode | nodeManager |
| node3 | dataNode | nodeManager |
| node4 | dataNode | nodeManager |

 给四个节点分配主机名和Hosts映射：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%208.png)

关闭防火墙（4个节点），检测状态为inactivate即成功：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%209.png)

ssh免密登录，这里我们主要是实现让node1能免密登录其他节点，用相同方法连通node1~4：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2010.png)

准备文件夹，并将jdk转移，再将hadoop解压缩到指定目录：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2011.png)

修改Hadoop配置文件，这里很多很杂：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2012.png)

hadoop-env.sh

```bash
export JAVA_HOME=/export/server/java/jdk1.8.0_342

export HDFS_NAMENODE_USER=zion
export HDFS_DATANODE_USER=zion
export HDFS_SECONDARYNAMENODE_USER=zion
export YARN_RESOURCEMANAGER_USER=zion
export YARN_NODEMANAGER_USER=zion
```

core-site.xml

```xml
<configuration>

<property>
        <name>fs.defaultFS</name>
        <value>hdfs://node1:8020</value>
</property>

<property>
        <name>hadoop.tmp.dir</name>
        <value>/export/data/hadoop</value>
</property>

<property>
        <name>hadoop.http.staticuser.user</name>
        <value>zion</value>
</property>

<!-- with hive proxy setting. by zion  -->
<property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
</property>

<property>
        <name>fs.trash.interval</name>
        <value>1440</value>
</property>

</configuration>
```

hdfs-site.xml

```xml
<configuration>
<property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node2:9868</value>
</property>
</configuration>
```

mapred-site.xml

```xml
<configuration>
<property>
	<name>mapreduce.framework.name</name>
	<value>yarn</value>
</property>

<!-- MR程序历史服务器地址-->
<property>
	<name>mapreduce.jobhistory.address</name>
	<value>node1:10020</value>
</property>

<!-- MR历史服务器web端地址 -->
<property>
	<name>mapreduce.jobhistory.webapp.address</name>
	<value>node1:19888</value>
</property>

<property>
	<name>yarn.app.mapreduce.am.env</name>
	<value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>

<property>
	<name>mapreduce.map.env</name>
	<value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
</configuration>
```

yarn-site.xml

```xml
<configuration>
<property>
　　<name>yarn.resourcemanager.hostname</name>
　　<value>node1</value>
</property>

<property>
　　<name>yarn.nodemanager.aux-services</name>
　　<value>mapreduce_shuffle</value>
</property>

<!-- 是否将对容器实施物理内存限制 -->
<property>
　　<name>yarn.nodemanager.pmem-check-enabled</name>
　　<value>false</value>
</property>

<!-- 是否将对容器实施虚拟内存限制 -->
<property>
　　<name>yarn.nodemanager.vmem-check-enabled</name>
　　<value>false</value>
</property>

<!-- 开启日志聚集 -->
<property>
　　<name>yarn.log-aggregation-enable</name>
　　<value>true</value>
</property>

<!-- 设置yarn历史服务器地址 -->
<property>
　　<name>yarn.log.server.url</name>
　　<value>http://node1:19888/jobhistory/logs</value>
</property>

<!-- 历史日志保存时间：7天 -->
<property>
　　<name>yarn.log-aggregation.retain-seconds</name>
　　<value>604800</value>
</property>
</configuration>
```

workers

```xml
node2
node3
node4
```

配置路径（4台机器），敲入hadoop命令后显示提示即表示配置成功：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2013.png)

在第一台机器上执行，这里我是在root权限下执行的：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2014.png)

启动hdfs服务，用jps验证：

node1:

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2015.png)

node2:

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2016.png)

我们可以通过[http://node1:9870](http://node1:9870)访问hdfs，可以通过[http://node1:8088](http://node1:8088)访问yarn。

### 4.配置Spark

部署Python，先安装anaconda，安装好后配置国内镜像源，并创建一个名为“pyspark”的环境：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2017.png)

然后解压spark安装包，并配置spark环境变量：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2018.png)

此时，可以在spark的bin目录中找到pyspark执行程序去测试：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2019.png)

修改Spark配置文件，

workers：

```bash
node2
node3
node4
```

spark-env.sh:

```bash
# Java
JAVA_HOME=/export/server/java/jdk1.8.0_342

# Hadoop
HADOOP_CONF_DIR=/export/server/hadoop/etc/hadoop
YARN_CONF_DIR=/export/server/hadoop/etc/hadoop

# Others
export SPARK_MASTER_HOST=node1
export SPARK_MASTER_PORT=7077
SPARK_MASTER_WEBUI_PORT=8080

SPARK_WORKER_CORES=1
SPARK_WORKER_MEMORY=1g
SPARK_WORKER_PORT=7078
SPARK_WORKER_WEBUI_PORT=8081

SPARK_HISTORY_OPTS="-Dspark.history.fs.logDirectory=hdfs://node1:8020/sparklog/ -Dspark.history.fs.cleaner.enabled=true"
```

这里由于将历史信息存在hdfs的sparklog中，因此我们要先去检查是否存在这个目录，不存在要先创建:

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2020.png)

spark.default.conf:

```bash
# 开启spark日期记录功能
spark.eventlog.enabled true
spark.eventlog.dir hdfs://node1:8020/sparklog/
spark.eventlog.compress true
```

log4j2.properties（可选）修改打印级别:

```bash
rootLogger.level = WARN, console
```

配置完毕，将spark发送到其他节点的export/server目录。

启动hadoop历史服务器（不然后面spark运行在yarn采取cluster模式时无法查看日志）：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2021.png)

启动spark历史服务器：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2022.png)

启动spark集群：

主节点：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2023.png)

再看看从节点：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2024.png)

也可以打开其web页面检查（端口是之前配置的SPARK_MASTER_WEBUI_PORT，我这里是8080）：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2025.png)

再重复之前的测试，如何让pyspark的shell工作在集群中，我们只需要将其和master节点（node1）连上：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2026.png)

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2027.png)

将spark部署在yarn中，之前我们做的是让spark复用hdfs和yarn的节点，但这样就很混乱和复杂，具体操作也很简单（如果我们之前配置的没问题的话）：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2028.png)

这个时候我们回到Web-UI，不难发现spark没有程序运行，但hadoop里有一个相同的app_id在执行：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2029.png)

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2030.png)

另外，当部署在cluster模式时，我们在终端无法看到输出，需要去任务日志中查看，我们默认是clien模式：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2031.png)

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2032.png)

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2033.png)

### 5.PySpark

使用Anaconda安装PySpark：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2034.png)

注意，我把Anaconda的一个环境命名为了“pyspark”，而spark的bin目录下有个shell也叫pyspark，而python有个库也叫pyspark，要注意区分它们。

配置pycharm远程解释器：

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2035.png)

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2036.png)

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2037.png)

![Untitled](Hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%20cd93459b2620404fbe275b067beac1f8/Untitled%2038.png)

由于我使用的是zsh，因此必须在文件中声明一些设置。