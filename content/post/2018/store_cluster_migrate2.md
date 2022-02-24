+++
title = "记存储集群的一次迁移过程（下）"
date = 2018-03-09T13:55:00+08:00
lastmod = 2022-02-24T22:29:04+08:00
tags = ["mysql", "hbase"]
categories = ["hbase"]
draft = false
toc = true
+++

从Mysql, Hbase 迁移数据


## <span class="section-num">1</span> Mysql 数据迁移 {#mysql-数据迁移}

搭建完之后就是数据迁移了，mysql 的数据迁移比较简单。在旧的机器用 mysqldump 把所 有的数据导出来，然后传到新的环境然后导出：

```sh
旧环境导出: mysqldump -u root -p --all-databases > all_dbs.sql
新环境导入: mysql -u root -p < all_dbs.sql
```

Mysql 集群在数据迁移的时候是在提供服务的，所以自然会有新数据写入，但因为是测试环 境，所以在迁移过程中可以忽略新数据写入的影响。不然这又会是一个大问题~


## <span class="section-num">2</span> Hbase 数据迁移 {#hbase-数据迁移}


### <span class="section-num">2.1</span> Hbase 集群复制(cluster replication) {#hbase-集群复制--cluster-replication}


#### <span class="section-num">2.1.1</span> 配置旧集群和新集群 {#配置旧集群和新集群}

在新的集群，创建和旧集群一样的表结构(table schema)和列族(column family),这样新的集群就知道在接收到旧集群数据时候怎么去保存。下面是具体的步骤：

1.  通过以下命令启动 Hbase Shell:

<!--listend-->

```shell
hbase shell
```

1.  通过下面的命令获取已有的表的元数据：

<!--listend-->

```shell
hbase> describe "content";
```

输入结果如下：

```sh
Table content is ENABLED
content, {TABLE_ATTRIBUTES => {coprocessor$1 => '|org.apache.phoenix.coprocessor.ScanRegionObserver|805306366|', co
processor$2 => '|org.apache.phoenix.coprocessor.UngroupedAggregateRegionObserver|805306366|', coprocessor$3 => '|or
g.apache.phoenix.coprocessor.GroupedAggregateRegionObserver|805306366|', coprocessor$4 => '|org.apache.phoenix.copr
ocessor.ServerCachingEndpointImpl|805306366|'}
COLUMN FAMILIES DESCRIPTION
{NAME => 'baseinfo', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REPLICATION_SCOPE => '0', COMPRESSION =>
'NONE', VERSIONS => '1', MIN_VERSIONS => '0', TTL => 'FOREVER', KEEP_DELETED_CELLS => 'FALSE', BLOCKSIZE => '65536'
, IN_MEMORY => 'false', BLOCKCACHE => 'true'}
{NAME => 'extrainfo', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REPLICATION_SCOPE => '0', COMPRESSION =>
 'NONE', VERSIONS => '1', MIN_VERSIONS => '0', TTL => 'FOREVER', KEEP_DELETED_CELLS => 'FALSE', BLOCKSIZE => '65536
', IN_MEMORY => 'false', BLOCKCACHE => 'true'}
2 row(s) in 0.3060 seconds
```

1.  复制以下内容到编辑器，并按要求进行修改：

<!--listend-->

```sh
{NAME => 'baseinfo', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REPLICATION_SCOPE => '0', COMPRESSION =>
'NONE', VERSIONS => '1', MIN_VERSIONS => '0', TTL => 'FOREVER', KEEP_DELETED_CELLS => 'FALSE', BLOCKSIZE => '65536'
, IN_MEMORY => 'false', BLOCKCACHE => 'true'}
{NAME => 'extrainfo', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REPLICATION_SCOPE => '0', COMPRESSION =>
 'NONE', VERSIONS => '1', MIN_VERSIONS => '0', TTL => 'FOREVER', KEEP_DELETED_CELLS => 'FALSE', BLOCKSIZE => '65536
', IN_MEMORY => 'false', BLOCKCACHE => 'true'}
```

-   将`TTL => 'FOREVER' with TTL` 修改成 `org.apache.hadoop.hbase.HConstants::FOREVER`
-   在列族的描述之间加上逗号`,`用来在新建的时候分隔列族
-   去掉文本中的换汉符(`\n, \r`), 这样文本就会变成单行文本

-   通过下面的语句在新的集群创建新的相同的表：


#### <span class="section-num">2.1.2</span> CopyTable {#copytable}

按照Apache 官方[文档](https://hbase.apache.org/book.html#ops.backup)的介绍，Hbase 支持两种形式的数据备份，分别是停服和不停服的。

我选择的是不停服的形式，停服的代价太大。而不停服的数据备份有三种方案，我选择是的 [CopyTable](https://hbase.apache.org/book.html#copy.table)方案。

<!--list-separator-->

1.  Add Peer

    因为 `CopyTable` 需要源机房和目标机房是网络连通，并且是目标集群在源集群的 `peer` list里面。所以要先在源集群添加 `peer`. 按照 Hbase [cluster replication](https://hbase.apache.org/book.html#_cluster_replication) 关于添加 `peer` 的说明：

    ```sh
    add_peer <ID> <CLUSTER_KEY>    Adds a replication relationship between two clusters.
           + ID — a unique string, which must not contain a hyphen.
           + CLUSTER_KEY: composed using the following template, with appropriate
    	 place-holders:
    	 `hbase.zookeeper.quorum:hbase.zookeeper.property.clientPort:zookeeper.znode.parent`
           + STATE(optional): ENABLED or DISABLED, default value is ENABLED
    ```

    而 `hbase.zookeeper.quorum` 可以在目标集群的 `$HBASE_HOME/conf/hbase-site.xml`
    目录找到设置的值； `hbase.zookeeper.property.clientPort` 可以在 `$HBASE_HOME/conf/hbase-site.xml`指定或者是在 `$ZOOKEEPER_HOME/conf/zoo.cfg`通 过 `clientPort` 指定； `zookeeper.znode.parent` 默认值是 `/hbase`

    所以，在 Hbase Shell 运行下面的命令来添加 `peer`

    ```shell
    add_peer '1', "node-master,node1,node2:2181:/hbase"
    ```


#### <span class="section-num">2.1.3</span> 复制表和数据 {#复制表和数据}

`CopyTable` 的命令说明如下：

```sh
./bin/hbase org.apache.hadoop.hbase.mapreduce.CopyTable --help
/bin/hbase org.apache.hadoop.hbase.mapreduce.CopyTable --help
Usage: CopyTable [general options] [--starttime=X] [--endtime=Y] [--new.name=NEW] [--peer.adr=ADR] <tablename>
```

可以指定需要复制的数据的时间间隔，也可以不指定。那么默认是全部数据，以 `cset_content` 表为例，复制一个小时的数据：

```shell
bin/hbase org.apache.hadoop.hbase.mapreduce.CopyTable --starttime=1265875194289 --endtime=1265878794289  --peer.adr=node-master,node1,node2:2181:/hbase --families=baseinfo,extrainfo cset_content
```

然后Hadoop 就会启动一个 MapReduce 的Job来运行这个 `CopyTable`任务。而我要复制所 有的数据，就需要把列族和表都列出来


## <span class="section-num">3</span> 结语 {#结语}

折腾一波之后，终于把环境弄好。如果目标机房和源机房不同的话，也可以尝试使用 Hbase 的 `Exporter` 和 `Importer`
