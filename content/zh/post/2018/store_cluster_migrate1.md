+++
title = "记存储集群的一次迁移过程（上）"
date = 2018-03-05T17:39:00+08:00
lastmod = 2022-02-24T22:25:27+08:00
tags = ["mysql", "hbase"]
categories = ["hbase"]
draft = false
toc = true
showQuote = true
+++

搭建和配置 Hadoop, Zookeeper, Hbase


## <span class="section-num">1</span> 前言 {#前言}

最近负责公司测试环境的迁移，主要包括 Hbase+Mysql 存储集群的迁移，消息队列，缓存组件的迁移, 而我打算说说存储集群的迁移。因为公司的机器的Ip 和Host 不便在博文展示，所以我会用：

```cfg
192.168.2.1: node-master
192.168.2.2: node1
192.168.2.3: node2
```

来代替公司的机器和域名。


## <span class="section-num">2</span> 搭建新环境 {#搭建新环境}


### <span class="section-num">2.1</span> Hadoop 搭建流程 {#hadoop-搭建流程}


#### <span class="section-num">2.1.1</span> Hadoop 集群的架构 {#hadoop-集群的架构}

在配置 Hadoop 的主从节点(master/slave)之前，先来了解一下 Hadoop 集群的组件作用；

-   `master` 节点负责担任管理分布式文件系统以及进行相应的资源调度的角色:
    -   NameNode: 管理分布式文件系统并且感知数据块在集群的存储位置
    -   ResourceManager: 管理 `YARN` 任务，并且负责在 `slave` 节点调度和处理
-   `slave` 节点负责担任存储真实的数据并且提供运算 `YARN` 任务的能力的角色：
    -   DataNode: 负责物理存储真实的数据
    -   NodeManager: 管理在该节点 `YARN` 任务的具体执行。

`master` 和 `slave` 的角色不一定像上面划分得泾渭分明，比如 `master` 节点也可以是 `dataNode`,这个就看具体配置了。


#### <span class="section-num">2.1.2</span> 配置JDK {#配置jdk}

Hadoop 集群需要JAVA 环境，而Linux 的发行版本一般都是默认带有JDK 的，只是OpenJDK 而不是 Oracle JDK, 如果需要修改JDK 的版本，可以自行修改，网上已经有很多安装JDK 的教程，我就不一一讲解了。


#### <span class="section-num">2.1.3</span> 修改host {#修改host}

因为需要不同的机器之间通信，所以需要先配置好Ip 和域名的映射。修改每台机器的 `/etc/hosts` 文件，加上以下内容：

```nil
192.168.2.1: nodw-master
192.168.2.2: node1
192.168.2.3: node2
```


#### <span class="section-num">2.1.4</span> 新建 hadoop 用户 {#新建-hadoop-用户}

虽说我可以用我自己的登录名来配置和运行 hadoop, 但是出于安全的考虑，还是在每个节点创建一个专门用来运行 hadoop 集群的用户比较好。

```shell
useradd hadoop
passwd hadoop
```


#### <span class="section-num">2.1.5</span> SSH 免密码登录 {#ssh-免密码登录}

因为在 Hadoop 集群中， `node-master` 节点会通过SSH连接和其他节点进行通信，所以需要为
Hadoop 集群配置免密码校验的通信。首先以 `hadoop` 用户身份登录到 `node-master` 节点，然后生成 SSH的公私钥：

```shell
ssh-keygen -b 4096
```

然后把公钥复制到其他的节点，如果你想要把 `node-master`也当作`dataNote`的话，就需要把公钥也复制到
`master`节点：

```nil
ssh-copy-id -i $HOME/.ssh/id_rsa.pub hadoop@node-master
ssh-copy-id -i $HOME/.ssh/id_rsa.pub hadoop@node1
ssh-copy-id -i $HOME/.ssh/id_rsa.pub hadoop@node2
```

谨记：复制的是”公钥”,不是”私钥”.


#### <span class="section-num">2.1.6</span> 安装hadoop 1. 下载hadoop 安装包，以 `hadoop`登录`node-master`，采用 wget命令下载： {#安装hadoop-1-dot-下载hadoop-安装包-以-hadoop-登录-node-master-采用-wget命令下载}

wget <http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.7.1.tar.gz>

1.  创建一个hadoop目录，将各个组件都安装在这个目录。

<!--listend-->

```nil
mdkir ~/hadoop
tar -zxvf hadoop-2.6.0-cdh5.7.1.tar.gz -C ~/hadoop
```


#### <span class="section-num">2.1.7</span> 修改配置文件 {#修改配置文件}

所有修改的 hadoop配置文件都位于 ~/hadoop/etc/hadoop/ 目录

```nil
cd ~/hadoop/etc/hadoop
```

<!--list-separator-->

1.  设置 NameNode 位置

    `vim core-site.xml`: 修改成以下内容：

    ```xml
    <configuration>
      <property>
        <name>fs.defaultFS</name>
        <value>hdfs://node-master:19000</value>
      </property>
      <property>
        <name>fs.trash.interval</name>
        <value>10080</value>
      </property>
      <property>
        <name>fs.trash.checkpoint.interval</name>
        <value>10080</value>
      </property>
    </configuration>
    ```

<!--list-separator-->

2.  设置 HDFS 路径

    `vim hdfs-site.xml`,修改内容为：

    ```xml
    <configuration>
      <property>
        <name>dfs.replication</name>
        <value>1</value>
      </property>
      <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/hadoop/data/temp</value>
      </property>
      <property>
        <name>dfs.namenode.http-address</name>
        <value>node-master:50070</value>
      </property>
      <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node1:50090</value>
      </property>
      <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
      </property>
      <property>
        <name>dfs.data.dir</name>
        <value>/home/hadoop/hadoop/data/hdfs</value>
      </property>
    </configuration>
    ```

<!--list-separator-->

3.  将 YARN 设置成任务调度器(Job Scheduler)

    ```shell
    cp mapred-site.xml.template mapred-site.xml
    ```

    然后修改配置，将 `yarn` 设置成 `MapReduce` 操作的默认框架：
    `vim mapred-site.xml`:

    ```xml
    <configuration>
      <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
      </property>
      <property>
        <name>mapreduce.jobhistory.address</name>
        <value>node-master:10020</value>
      </property>
      <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>node-master:19888</value>
      </property>
    </configuration>
    ```

<!--list-separator-->

4.  配置 YARN

    `vim yarn-site.xml`:

    ```xml
    <configuration>
      <property>
        <name>yarn.acl.enable</name>
        <value>0</value>
      </property>

      <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>node-master</value>
      </property>

      <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
      </property>
    </configuration>
    ```

<!--list-separator-->

5.  配置 master 节点

    因为公司的机器内存较大，兼之机器数不多，所以我就把两台机器都都当作 `master`:
    `vim masters`修改文件为：

    ```cfg
    node-master
    node1
    ```

<!--list-separator-->

6.  配置 slave 节点

    把全部节点都当作数据几点(dataNode):
    `vim slaves`:

    ```cfg
    node-master
    node1
    node2
    ```

<!--list-separator-->

7.  修改 Hadoop 环境变量配置

    这个配置是否修改就视情况而定了。
    `vim hadoop-env.sh`:

    ```sh
    #因为ssh的端口不是默认的22,需要重新指定
    export HADOOP_SSH_OPTS="-p 9922"
    #如果报错java_home找不到，可在这里重新指定
    #export JAVA_HOME=/usr/java/jdk1.8.0_161/
    ```

<!--list-separator-->

8.  在其它的节点安装并且解压，不要修改配置文件

<!--list-separator-->

9.  将配置文件同步到slave主机

    ```shell
    scp -r -P 9922 /home/hadoop/hadoop/hadoop-2.6.0-cdh5.7.1/etc/hadoop/  node1:/home/hadoop/hadoop/hadoop-2.6.0-cdh5.7.1/etc
    scp -r -P 9922 /home/hadoop/hadoop/hadoop-2.6.0-cdh5.7.1/etc/hadoop/  node2:/home/hadoop/hadoop/hadoop-2.6.0-cdh5.7.1/etc
    ```

<!--list-separator-->

10.  修改环境变量（所有节点都要配置）

    编辑 `.bash_profile`：
    `vim ~/.bash_profile`
    加入：

    ```shell
    export JAVA_HOME=/usr/java/jdk1.8.0_161/
    export JRE_HOME=/usr/java/jdk1.8.0_161/jre
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
    export HADOOP_HOME=/home/hadoop/hadoop/hadoop-2.6.0-cdh5.7.1
    export HBASE_HOME=/home/hadoop/hadoop/hbase-1.2.0-cdh5.7.1
    export HADOOP_MAPRED_HOME=${HADOOP_HOME}
    export HADOOP_COMMON_HOME=${HADOOP_HOME}
    export HADOOP_HDFS_HOME=${HADOOP_HOME}
    export YARN_HOME=${HADOOP_HOME}
    export HADOOP_YARN_HOME=${HADOOP_HOME}
    export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
    export HDFS_CONF_DIR=${HADOOP_HOME}/etc/hadoop
    export YARN_CONF_DIR=${HADOOP_HOME}/etc/hadoop
    PATH=$PATH:$HOME/bin:$JAVA_HOME/bin:$HADOOP_HOME/sbin:$HBASE_HOME/bin:$HADOOP_HOME/bin
    export PATH
    ```

    然后加载 `~/.bash_profile`:
    `source ~/.bash_profile`

<!--list-separator-->

11.  格式化 HDFS

    就像其它的文件系统那样，在使用之前需要格式化，HDFS 这个分布式文件系统也不例外。在 `node-master`,运行：

    ```nil
    hdfs namenode -format
    ```

    那么，到目前为止， Hadoop 就已经安装和配置好了。


#### <span class="section-num">2.1.8</span> 运行和监控 HDFS {#运行和监控-hdfs}

<!--list-separator-->

1.  启动HDFS

    在 `node-master` 的 `/home/hadoop/hadoop/sbin/` 目录运行下面的命令以启动 HDFS:

    ```nil
    ./start-dfs.sh
    ```

    然后就会启动 `NameNode` 和 `SecondaryNameNode`, 然后继续启动 `DataNode`.

<!--list-separator-->

2.  验证HDFS

    可以在各个节点通过 `jps` 检查HDFS 的运行状态。比如在 `node-master` 运行 `jps`:

    ```cfg
    12243 NameNode
    2677 ResourceManager
    19593 Jps
    15036 DataNode
    ```

    `node1`:

    ```cfg
    30464 DataNode
    13094 Jps
    28589 SecondaryNameNode
    ```

<!--list-separator-->

3.  停止HDFS

    在 `node-master` 的 `/home/hadoop/hadoop/sbin/` 目录运行下面的命令以停止HDFS:

    ```nil
    ./stop-dfs.sh
    ```

<!--list-separator-->

4.  监控HDFS

    如果你在启动 HDFS 之后，想要获取关于 HDFS 的详细信息，你可以使用 `hdfs dfsadmin -report` 命令， 例如在 `node-master` 运行 `hdfs dfsadmin -resport`, 输出如下：

    ```cfg
    Configured Capacity: 1201169780736 (1.09 TB)
    Present Capacity: 1129442681745 (1.03 TB)
    DFS Remaining: 1129442358161 (1.03 TB)
    DFS Used: 323584 (316 KB)
    DFS Used%: 0.00%
    Under replicated blocks: 0
    Blocks with corrupt replicas: 0
    Missing blocks: 0
    Missing blocks (with replication factor 1): 0

    -------------------------------------------------
    Live datanodes (3):

    Name: 192.168.2.3:50010 (node2)
    Hostname: node2
    Decommission Status : Normal
    Configured Capacity: 400389926912 (372.89 GB)
    DFS Used: 102400 (100 KB)
    Non DFS Used: 24759164513 (23.06 GB)
    DFS Remaining: 375630659999 (349.83 GB)
    DFS Used%: 0.00%
    DFS Remaining%: 93.82%
    Configured Cache Capacity: 0 (0 B)
    Cache Used: 0 (0 B)
    Cache Remaining: 0 (0 B)
    Cache Used%: 100.00%
    Cache Remaining%: 0.00%
    Xceivers: 11
    Last contact: Mon Mar 05 14:05:38 CST 2018


    Name: 192.168.2.2:50010 (node1)
    Hostname: node1
    Decommission Status : Normal
    Configured Capacity: 400389926912 (372.89 GB)
    DFS Used: 77824 (76 KB)
    Non DFS Used: 23483977479 (21.87 GB)
    DFS Remaining: 376905871609 (351.02 GB)
    DFS Used%: 0.00%
    DFS Remaining%: 94.13%
    Configured Cache Capacity: 0 (0 B)
    Cache Used: 0 (0 B)
    Cache Remaining: 0 (0 B)
    Cache Used%: 100.00%
    Cache Remaining%: 0.00%
    Xceivers: 7
    Last contact: Mon Mar 05 14:05:38 CST 2018


    Name: 192.168.2.1:50010 (node-master)
    Hostname: node-master
    Decommission Status : Normal
    Configured Capacity: 400389926912 (372.89 GB)
    DFS Used: 143360 (140 KB)
    Non DFS Used: 23483956999 (21.87 GB)
    DFS Remaining: 376905826553 (351.02 GB)
    DFS Used%: 0.00%
    DFS Remaining%: 94.13%
    Configured Cache Capacity: 0 (0 B)
    Cache Used: 0 (0 B)
    Cache Remaining: 0 (0 B)
    Cache Used%: 100.00%
    Cache Remaining%: 0.00%
    Xceivers: 7
    Last contact: Mon Mar 05 14:05:39 CST 2018
    ```

    或者可以使用更加友好的 Web 管理界面，在浏览器输入： <http://node-master-ip:50070>, 然后你就可以看到如下的监控界面：
    ![](https://imgur.com/jmIXxRy.jpg)

<!--list-separator-->

5.  使用 HDFS

    既然 HDFS 可以跑起来了，现在就需要添加一点数据以测试 HDFS 了。 在HDFS 的根目录下新建一个 `test` 目录：

    ```sh
    hdfs dfs -mkdir /test
    ```

    然后在本地创建一个 `helloworld` 文件，内容如下：

    ```sh
    Bye world!
    ```

    接着把 `helloworld` 文件放置到HDFS的 `/test` 目录下：

    ```sh
    hdfs dfs -put helloworld /test
    ```

    最后查看文件是否存在：

    ```shell
    hdfs dfs -ls /test
    ```


### <span class="section-num">2.2</span> 小结 {#小结}

至此，如果一切顺利的话， 那么Hadoop 集群就运行起来了。因为我是需要用Hbase 作存储集群，暂不需用`Yarn` 作计算，所以我就没有介绍启动 `Yarn` 的 流程了。


### <span class="section-num">2.3</span> ZooKeeper 搭建流程 {#zookeeper-搭建流程}

因为需要用 ZooKeeper 来管理集群，所以也需要安装 ZooKeeper. 而 ZooKeeper 的安装和 配置也是用 `hadoop` 用户进行操作的。


#### <span class="section-num">2.3.1</span> 安装zookeeper (每个节点同样的操作) 1. 下载zookeeper安装包，登录主机，采用wget命令下载： {#安装zookeeper--每个节点同样的操作--1-dot-下载zookeeper安装包-登录主机-采用wget命令下载}

wget <http://archive.cloudera.com/cdh5/cdh/5/zookeeper-3.4.5-cdh5.7.1.tar.gz>

1.  解压安装到hadoop目录，将各个组件都安装在这个目录。

<!--listend-->

```shell
tar -zxvf zookeeper-3.4.5-cdh5.7.1.tar.gz -C ~/hadoop
```


#### <span class="section-num">2.3.2</span> 配置 ZooKeeper {#配置-zookeeper}

修改zoo.cfg (所有机器一样的配置): `vim /home/hadoop/hadoop/zookeeper-3.4.5-cdh5.7.1/conf/zoo.cfg`, 配置文件内容如下：

```cfg
# The number of milliseconds of each tick
tickTime=2000
maxSessionTimeout=300000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/home/hadoop/hadoop/zookeeper-3.4.5-cdh5.7.1/data
# the port at which the clients will connect
clientPort=2181
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.1=node-master:2888:3888
server.2=node1:2888:3888
server.3=node2:2888:3888
```

<!--list-separator-->

1.  创建myid文件 （不同主机不同数字）

    ```sh
    cd {data_dir} # 按照上面的配置，就应该是/home/hadoop/hadoop/zookeeper-3.4.5-cdh5.7.1/data
    vim myid
    ```

    myid的数字要与zoo.cfg配置的一一对应。即要对应：

    ```cfg
    server.1=node-master:2888:3888
    server.2=node1:2888:3888
    server.3=node2:2888:3888
    ```

    也就是 `node-master` 的 `myid`是`1`, `node1`的 `myid`是 `2`,依次类推。需要注意的 是，数字前后都不能有空格！

<!--list-separator-->

2.  启动 ZooKeeper

    在 `node-master` 的 `/home/hadoop/hadoop/zookeeper-3.4.5-cdh5.7.1` 目录，运行以 下命令：

    ```shell
    sh bin/zkServer.sh start
    ```

    其它相应的命令如下：

    -   启动ZK服务: `sh bin/zkServer.sh start`
    -   查看ZK服务状态: `sh bin/zkServer.sh status`
    -   停止ZK服务: `sh bin/zkServer.sh stop`
    -   重启ZK服务: `sh bin/zkServer.sh restart`

<!--list-separator-->

3.  验证 ZooKeeper

    可以通过调用 `jps` 或者 `bin/zkCli.sh` 来验证 Zookeeper 的运行情况：

    ```shell
    ./zkCli.sh -server 192.168.2.1:2181
    ```


### <span class="section-num">2.4</span> HBase 搭建流程 {#hbase-搭建流程}


#### <span class="section-num">2.4.1</span> 安装Hbase {#安装hbase}

1.  下载hbase 安装包，登录主机，采用wget命令下载：
    wget <http://archive.cloudera.com/cdh5/cdh/5/hbase-1.2.0-cdh5.7.1.tar.gz>
    2、解压安装到hadoop目录（3台主机同样操作）

<!--listend-->

```sh
tar -zxvf hbase-1.2.0-cdh5.7.1.tar.gz -C ~/hadoop
```


#### <span class="section-num">2.4.2</span> 修改hbase的配置文件(所有主机一样的配置) {#修改hbase的配置文件--所有主机一样的配置}

修改 `hbase-1.2.0-cdh5.7.1/conf` 目录下的文件： 1. 修改 `hbase-site.xml` 内容如下：

```xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://node-master:19000/hbase-${user.name}</value>
  </property>
  <property>
    <name>hbase.master</name>
    <value>node-master</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.tmp.dir</name>
    <value>/home/hadoop/hadoop/data/hbase-${user.name}</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>node-master, node1, node2</value>
  </property>
</configuration>
```

1.  修改 `hbase-env.sh`:

<!--listend-->

```shell
export JAVA_HOME=/usr/java/jdk1.8.0_161/ (无法识别系统的环境变量，在这里直接指定)
export HBASE_MANAGES_ZK=false  （关闭自带的zookeeper）
export HBASE_SSH_OPTS="-p 9922" (端口号不是默认的22，要改为9922)
```

1.  修改 =regionservers=(指定regionservers的主机地址):

<!--listend-->

```shell
node-master
node1
node2
```


#### <span class="section-num">2.4.3</span> 启动 Hbase {#启动-hbase}

在 `node-master` 的 `/home/hadoop/hadoop/hbase-1.2.0-cdh5.7.1/bin` 目录下运行：

```sh
start-hbase.sh
```


#### <span class="section-num">2.4.4</span> 监控 Hbase {#监控-hbase}

可以在浏览器查看 HBase 集群的信息：

-   HMaster web 管理信息：<http://192.168.2.1:60010/master-status>
-   HregionServer 管理信息：<http://192.168.2.1:60030/> <http://192.168.2.2:60030/>

<http://192.168.2.3:60030/>


## <span class="section-num">3</span> 结语 {#结语}

整个 Hbase 集群应该搭建完了，关于Mysql 搭建的文章就太多了，我也不赘言了。至此，所有存储的组件就已经安装完毕并已启用，但是都是没有数据的，接下来我们需要做的是如何将旧的测试环境的数据迁移到新的测试环境。考虑到这篇内容已经很长了，剩下的内容我就另外写一篇博文了。


## <span class="section-num">4</span> 参考 {#参考}

-   <http://blog.csdn.net/u010824591/article/details/51174099>
-   <https://yq.aliyun.com/articles/26415>
-   <https://linode.com/docs/databases/hadoop/how-to-install-and-set-up-hadoop-cluster/>

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

