+++
title = "记一次Hbase 宕机原因分析"
date = 2018-04-04T19:54:00+08:00
lastmod = 2022-02-24T22:58:14+08:00
tags = ["hbase"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 背景 {#背景}

在2018年4月4号早上，业务方反应Hbase 读超时，无法读取当前数据。然后发现测试环境的
Hbase region server 全部宕机，已经无可用Region Server. 因为公司的机器的Ip 和Host
不便在博文展示，所以我会用：

```nil
192.168.2.1: node-master
192.168.2.2: node1
192.168.2.3: node2
```

来代替


## <span class="section-num">2</span> Region Server 宕机原因分析 {#region-server-宕机原因分析}

经查看日志，发现三台部署了Hbase 的服务器，分别是`node-master 192.168.2.1`, `node1 192.168.2.2`,=node2 192.168.2.3=. node1 机器在2018-03-13 14:47:55 收到了Shutdown Message, 停了Region Server. node-master这台机器在2018-03-20
10:13:07收到了Shutdown Message, 停掉了Region Server.

也就是说在3月下旬到昨天，Hbase 一直只有一台Region Server 在运行。而在昨天，2018-04-03 23:19:35, 剩下的最后一台机器也收到了Shutdown Message, 因此把剩下的最后一台Region Server 停掉，测试 环境的Hbase 全部下线。那么，为什么这三台服务器会收到Shutdown Message 呢？


### <span class="section-num">2.1</span> node1 {#node1}

先从 node1这台机器开始分析，关于 Region Server 退出的日志显示如下：

```log
2018-03-13 14:47:49,665 INFO  [main-SendThread(node-master:2181)] zookeeper.ClientCnxn:
Unable to reconnect to ZooKeeper service, session 0x161d6c1ae910001 has expired, closing socket connection
2018-03-13 14:47:49,706 FATAL [main-EventThread] regionserver.HRegionServer: ABORTING region server node1,60020,1519732610839:
regionserver:60020-0x161d6c1ae910001, quorum=node-master:2181,node1:2181,node2:2181,
baseZNode=/hbase regionserver:60020-0x161d6c1ae910001 received expired from ZooKeeper, aborting
org.apache.zookeeper.KeeperException$SessionExpiredException: KeeperErrorCode = Session expired
    at org.apache.hadoop.hbase.zookeeper.ZooKeeperWatcher.connectionEvent(ZooKeeperWatcher.java:700)
    at org.apache.hadoop.hbase.zookeeper.ZooKeeperWatcher.process(ZooKeeperWatcher.java:611)
    at org.apache.zookeeper.ClientCnxn$EventThread.processEvent(ClientCnxn.java:522)
    at org.apache.zookeeper.ClientCnxn$EventThread.run(ClientCnxn.java:498)
2018-03-13 14:47:49,718 FATAL [main-EventThread] regionserver.HRegionServer: RegionServer abort:
loaded coprocessors are: [org.apache.hadoop.hbase.coprocessor.MultiRowMutationEndpoint]
2018-03-13 14:47:50,705 WARN  [DataStreamer for file /hbase-nemo/WALs/node1,60020,1519732610839/node1%2C60020%2C1519732610839.default.1520922158622 block BP-1296874721-192.168.2.1-1519712987003:blk_1073743994_3170] hdfs.DFSClient: DataStreamer Exception
org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.LeaseExpiredException):
No lease on /hbase-nemo/oldWALs/node1%2C60020%2C1519732610839.default.1520922158622 (inode 18837):
File is not open for writing. [Lease.  Holder: DFSClient_NONMAPREDUCE_551822027_1, pendingcreates: 1]
    at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkLease(FSNamesystem.java:3612)
    at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.getAdditionalDatanode(FSNamesystem.java:3516)
    at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.getAdditionalDatanode(NameNodeRpcServer.java:711)
    at org.apache.hadoop.hdfs.server.namenode.AuthorizationProviderProxyClientProtocol.getAdditionalDatanode(AuthorizationProviderProxyClientProtocol.java:229)
    at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.getAdditionalDatanode(ClientNamenodeProtocolServerSideTranslatorPB.java:508)
    at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
    at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:617)
    at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1073)
    at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2086)
    at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2082)
    at java.security.AccessController.doPrivileged(Native Method)
    at javax.security.auth.Subject.doAs(Subject.java:422)
    at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1693)
    at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2080)

    at org.apache.hadoop.ipc.Client.call(Client.java:1471)
    at org.apache.hadoop.ipc.Client.call(Client.java:1408)
    at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:230)
    at com.sun.proxy.$Proxy16.getAdditionalDatanode(Unknown Source)
    at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.getAdditionalDatanode(ClientNamenodeProtocolTranslatorPB.java:429)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:256)
    at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:104)
    at com.sun.proxy.$Proxy17.getAdditionalDatanode(Unknown Source)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.apache.hadoop.hbase.fs.HFileSystem$1.invoke(HFileSystem.java:279)
    at com.sun.proxy.$Proxy18.getAdditionalDatanode(Unknown Source)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.addDatanode2ExistingPipeline(DFSOutputStream.java:1228)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.setupPipelineForAppendOrRecovery(DFSOutputStream.java:1404)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.processDatanodeError(DFSOutputStream.java:1119)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.run(DFSOutputStream.java:622)
2018-03-13 14:47:53,803 FATAL [regionserver/node1/192.168.2.2:60020] regionserver.HRegionServer: ABORTING region server node1,60020,1519732610839: org.apache.hadoop.hbase.YouAreDeadException: Server REPORT rejected; currently processing node1,60020,1519732610839 as dead server
```

从 14:47:49 开始， Hbase 没法和 Zookeeper 通信，连接时间超时。翻查 Zookeeper 的日志，发现Zookeeper 的日志有如下内容：

```log
2018-03-13 14:47:46,926 [myid:1] - INFO  [QuorumPeer[myid=1]/0.0.0.0:2181:ZooKeeperServer@588]
- Invalid session 0x161d6c1ae910002 for client /192.168.2.2:51611, probably expired
2018-03-13 14:47:49,612 [myid:1] - INFO  [QuorumPeer[myid=1]/0.0.0.0:2181:ZooKeeperServer@588]
- Invalid session 0x161d6c1ae910001 for client /192.168.2.2:51612, probably expired
```

说明Hbase 和 ZooKeeper 的通信的确去了问题。连接出问题以后，集群就会认为 这个
Hbase 的节点出了故障，宕机，然后就把这个节点当作 `DeadNode`, 这个节点的
RegionServer 就下线了。


### <span class="section-num">2.2</span> node-master {#node-master}

现在再来看看node-master这台机器的日志

```log
2018-03-20 10:12:19,986 INFO  [main-SendThread(node-master:2181)] zookeeper.ClientCnxn:
Unable to read additional data from server sessionid 0x361d65049260001, likely server has closed socket, closing socket connection and attempting reconnect
2018-03-20 10:12:20,841 INFO  [main-SendThread(node1:2181)] zookeeper.ClientCnxn:
Opening socket connection to server node1/192.168.2.2:2181. Will not attempt to authenticate using SASL (unknown error)
2018-03-20 10:12:43,747 INFO  [regionserver/node-master/192.168.2.1:60020-SendThread(node1:2181)] zookeeper.ClientCnxn:
Client session timed out, have not heard from server in 60019ms for sessionid 0x161d65049590000, closing socket connection and attempting reconnect
2018-03-20 10:12:44,574 INFO  [regionserver/node-master/192.168.2.1:60020-SendThread(node-master:2181)] zookeeper.ClientCnxn:
Opening socket connection to server node-master/192.168.2.1:2181. Will not attempt to authenticate using SASL (unknown error)
2018-03-20 10:12:44,575 INFO  [regionserver/node-master/192.168.2.1:60020-SendThread(node-master:2181)] zookeeper.ClientCnxn:
Socket connection established, initiating session, client: /192.168.2.1:58042, server: node-master/192.168.2.1:2181
2018-03-20 10:12:44,577 INFO  [regionserver/node-master/192.168.2.1:60020-SendThread(node-master:2181)] zookeeper.ClientCnxn:
Session establishment complete on server node-master/192.168.2.1:2181, sessionid = 0x161d65049590000, negotiated timeout = 90000
2018-03-20 10:12:49,625 INFO  [main-SendThread(node1:2181)] zookeeper.ClientCnxn:
Socket connection established, initiating session, client: /192.168.2.1:46815, server: node1/192.168.2.2:2181
2018-03-20 10:12:53,258 WARN  [ResponseProcessor for block BP-1296874721-192.168.2.1-1519712987003:
blk_1073747108_6286] hdfs.DFSClient: Slow ReadProcessor read fields took 70070ms (threshold=30000ms);
ack: seqno: -2 reply: 0 reply: 1 downstreamAckTimeNanos: 0, targets: [DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.2:50010,DS-4eb97418-f0a1-45a7-b335-83f77e4d6a7b,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]]
2018-03-20 10:12:53,259 WARN  [ResponseProcessor for block BP-1296874721-192.168.2.1-1519712987003:blk_1073747108_6286] hdfs.DFSClient: DFSOutputStream ResponseProcessor exception  for block BP-1296874721-192.168.2.1-1519712987003:blk_1073747108_6286
java.io.IOException: Bad response ERROR for block BP-1296874721-192.168.2.1-1519712987003:blk_1073747108_6286 from datanode DatanodeInfoWithStorage[192.168.2.2:50010,DS-4eb97418-f0a1-45a7-b335-83f77e4d6a7b,DISK]
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer$ResponseProcessor.run(DFSOutputStream.java:1002)
2018-03-20 10:12:53,259 WARN  [DataStreamer for file /hbase-nemo/WALs/node-master,60020,1519720160721/node-master%2C60020%2C1519720160721.default.1521509628323 block BP-1296874721-192.168.2.1-1519712987003:blk_1073747108_6286] hdfs.DFSClient: Error Recovery for block BP-1296874721-192.168.2.1-1519712987003:blk_1073747108_6286 in pipeline DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.2:50010,DS-4eb97418-f0a1-45a7-b335-83f77e4d6a7b,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]: bad datanode DatanodeInfoWithStorage[192.168.2.2:50010,DS-4eb97418-f0a1-45a7-b335-83f77e4d6a7b,DISK]
2018-03-20 10:12:53,264 WARN  [DataStreamer for file /hbase-nemo/WALs/node-master,60020,1519720160721/node-master%2C60020%2C1519720160721.default.1521509628323 block BP-1296874721-192.168.2.1-1519712987003:blk_1073747108_6286] hdfs.DFSClient: DataStreamer Exception
java.io.IOException: Failed to replace a bad datanode on the existing pipeline due to no more good datanodes being available to try. (Nodes: current=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]], original=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]]). The current failed datanode replacement policy is DEFAULT, and a client may configure this via 'dfs.client.block.write.replace-datanode-on-failure.policy' in its configuration.
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.findNewDatanode(DFSOutputStream.java:1162)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.addDatanode2ExistingPipeline(DFSOutputStream.java:1236)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.setupPipelineForAppendOrRecovery(DFSOutputStream.java:1404)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.processDatanodeError(DFSOutputStream.java:1119)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.run(DFSOutputStream.java:622)
2018-03-20 10:12:53,265 WARN  [sync.4] hdfs.DFSClient: Error while syncing
java.io.IOException: Failed to replace a bad datanode on the existing pipeline due to no more good datanodes being available to try. (Nodes: current=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]], original=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]]). The current failed datanode replacement policy is DEFAULT, and a client may configure this via 'dfs.client.block.write.replace-datanode-on-failure.policy' in its configuration.
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.findNewDatanode(DFSOutputStream.java:1162)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.addDatanode2ExistingPipeline(DFSOutputStream.java:1236)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.setupPipelineForAppendOrRecovery(DFSOutputStream.java:1404)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.processDatanodeError(DFSOutputStream.java:1119)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.run(DFSOutputStream.java:622)
2018-03-20 10:12:53,266 ERROR [sync.4] wal.FSHLog: Error syncing, request close of WAL
java.io.IOException: Failed to replace a bad datanode on the existing pipeline due to no more good datanodes being available to try. (Nodes: current=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]], original=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]]). The current failed datanode replacement policy is DEFAULT, and a client may configure this via 'dfs.client.block.write.replace-datanode-on-failure.policy' in its configuration.
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.findNewDatanode(DFSOutputStream.java:1162)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.addDatanode2ExistingPipeline(DFSOutputStream.java:1236)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.setupPipelineForAppendOrRecovery(DFSOutputStream.java:1404)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.processDatanodeError(DFSOutputStream.java:1119)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.run(DFSOutputStream.java:622)
2018-03-20 10:12:53,266 INFO  [sync.4] wal.FSHLog: Slow sync cost: 474 ms, current pipeline: [DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]]
2018-03-20 10:13:05,816 INFO  [regionserver/node-master/192.168.2.1:60020.logRoller] wal.FSHLog: Slow sync cost: 12546 ms, current pipeline: [DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]]
2018-03-20 10:13:05,817 ERROR [sync.0] wal.FSHLog: Error syncing, request close of WAL
java.io.IOException: Failed to replace a bad datanode on the existing pipeline due to no more good datanodes being available to try. (Nodes: current=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]], original=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]]). The current failed datanode replacement policy is DEFAULT, and a client may configure this via 'dfs.client.block.write.replace-datanode-on-failure.policy' in its configuration.
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.findNewDatanode(DFSOutputStream.java:1162)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.addDatanode2ExistingPipeline(DFSOutputStream.java:1236)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.setupPipelineForAppendOrRecovery(DFSOutputStream.java:1404)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.processDatanodeError(DFSOutputStream.java:1119)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.run(DFSOutputStream.java:622)
2018-03-20 10:13:05,817 ERROR [regionserver/node-master/192.168.2.1:60020.logRoller] wal.FSHLog: Failed close of WAL writer hdfs://node-master:19000/hbase-nemo/WALs/node-master,60020,1519720160721/node-master%2C60020%2C1519720160721.default.1521509628323, unflushedEntries=1
org.apache.hadoop.hbase.regionserver.wal.FailedSyncBeforeLogCloseException: java.io.IOException: Failed to replace a bad datanode on the existing pipeline due to no more good datanodes being available to try. (Nodes: current=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]], original=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]]). The current failed datanode replacement policy is DEFAULT, and a client may configure this via 'dfs.client.block.write.replace-datanode-on-failure.policy' in its configuration.
    at org.apache.hadoop.hbase.regionserver.wal.FSHLog$SafePointZigZagLatch.waitSafePoint(FSHLog.java:1615)
    at org.apache.hadoop.hbase.regionserver.wal.FSHLog.replaceWriter(FSHLog.java:833)
    at org.apache.hadoop.hbase.regionserver.wal.FSHLog.rollWriter(FSHLog.java:699)
    at org.apache.hadoop.hbase.regionserver.LogRoller.run(LogRoller.java:148)
    at java.lang.Thread.run(Thread.java:748)
Caused by: java.io.IOException: Failed to replace a bad datanode on the existing pipeline due to no more good datanodes being available to try.
(Nodes: current=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK],
DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]],
original=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK],
DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]]).
The current failed datanode replacement policy is DEFAULT, and a client may configure this
via 'dfs.client.block.write.replace-datanode-on-failure.policy' in its configuration.
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.findNewDatanode(DFSOutputStream.java:1162)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.addDatanode2ExistingPipeline(DFSOutputStream.java:1236)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.setupPipelineForAppendOrRecovery(DFSOutputStream.java:1404)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.processDatanodeError(DFSOutputStream.java:1119)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.run(DFSOutputStream.java:622)
2018-03-20 10:13:05,818 FATAL [regionserver/node-master/192.168.2.1:60020.logRoller]
regionserver.HRegionServer: ABORTING region server node-master,60020,1519720160721: Failed log close in log roller
org.apache.hadoop.hbase.regionserver.wal.FailedLogCloseException: hdfs://node-master:19000/hbase-nemo/WALs/node-master,60020,1519720160721/node-master%2C60020%2C1519720160721.default.1521509628323, unflushedEntries=1
    at org.apache.hadoop.hbase.regionserver.wal.FSHLog.replaceWriter(FSHLog.java:882)
    at org.apache.hadoop.hbase.regionserver.wal.FSHLog.rollWriter(FSHLog.java:699)
    at org.apache.hadoop.hbase.regionserver.LogRoller.run(LogRoller.java:148)
    at java.lang.Thread.run(Thread.java:748)
Caused by: org.apache.hadoop.hbase.regionserver.wal.FailedSyncBeforeLogCloseException: java.io.IOException: Failed to replace a bad datanode on the existing pipeline due to no more good datanodes being available to try. (Nodes: current=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]], original=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]]). The current failed datanode replacement policy is DEFAULT, and a client may configure this via 'dfs.client.block.write.replace-datanode-on-failure.policy' in its configuration.
    at org.apache.hadoop.hbase.regionserver.wal.FSHLog$SafePointZigZagLatch.waitSafePoint(FSHLog.java:1615)
    at org.apache.hadoop.hbase.regionserver.wal.FSHLog.replaceWriter(FSHLog.java:833)
    ... 3 more
Caused by: java.io.IOException: Failed to replace a bad datanode on the existing pipeline due to no more good datanodes being available to try. (Nodes: current=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]], original=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]]). The current failed datanode replacement policy is DEFAULT, and a client may configure this via 'dfs.client.block.write.replace-datanode-on-failure.policy' in its configuration.
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.findNewDatanode(DFSOutputStream.java:1162)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.addDatanode2ExistingPipeline(DFSOutputStream.java:1236)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.setupPipelineForAppendOrRecovery(DFSOutputStream.java:1404)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.processDatanodeError(DFSOutputStream.java:1119)
    at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.run(DFSOutputStream.java:622)
2018-03-20 10:13:05,818 FATAL [regionserver/node-master/192.168.2.1:60020.logRoller] regionserver.HRegionServer: RegionServer abort: loaded coprocessors are: []
2018-03-20 10:13:05,997 INFO  [regionserver/node-master/192.168.2.1:60020.logRoller] regionserver.HRegionServer: Dump of metrics as JSON on abort:
```

从上面的日志可以看到 node-master与node1机器通信，获取node1 的响应失败，认为node1 是 bad DataNode，接着集群想要把出现问题的DataNode 下掉，却发现没有多余DataNode 来替换， 紧接着在Syncing 时出错，关闭 WAL 失败, 最后就停掉了Region Server. 比较关键的时机如下：

```log
2018-03-20 10:12:53,265 WARN  [sync.4] hdfs.DFSClient: Error while syncing

2018-03-20 10:12:53,266 ERROR [sync.4] wal.FSHLog: Error syncing, request close of WAL

2018-03-20 10:13:05,817 ERROR [sync.0] wal.FSHLog: Error syncing, request close of WAL

2018-03-20 10:13:06,397 ERROR [regionserver/node-master/192.168.2.1:60020] regionserver.HRegionServer: Shutdown / close of WAL failed: java.io.IOException: Failed to replace a bad datanode on the existing pipeline due to no more good datanodes being available to try. (Nodes: current=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]], original=[DatanodeInfoWithStorage[192.168.2.1:50010,DS-84998b22-8294-44ed-90fd-9c1a78d0f558,DISK], DatanodeInfoWithStorage[192.168.2.3:50010,DS-d94668c9-66f4-40f6-b38f-83f14b26c2b4,DISK]]). The current failed datanode replacement policy is DEFAULT, and a client may configure this via 'dfs.client.block.write.replace-datanode-on-failure.policy' in its configuration.
```

期间HDFS 同步出错，尝试关闭WAL, 失败。失败的原因是无法用健康的节点替换出了问题的节点， 应该是健康的节点数太少了。最后在多次尝试关闭WAL都因为IOException 失败之后，
RegionServer 下线。只是为什么尝试关闭WAL 失败需要关闭Region Server 依然存疑。


### <span class="section-num">2.3</span> node2 {#node2}

node2 是Hbase 集群最后一台机器，当node2 倒下了，Hbase 就真的完全宕机了。

```log
2018-04-03 23:19:33,472 FATAL [regionserver/node2/192.168.2.3:60020.logRoller] regionserver.LogRoller: Aborting
java.io.IOException: cannot get log writer
    at org.apache.hadoop.hbase.wal.DefaultWALProvider.createWriter(DefaultWALProvider.java:365)
    at org.apache.hadoop.hbase.regionserver.wal.FSHLog.createWriterInstance(FSHLog.java:724)
    at org.apache.hadoop.hbase.regionserver.wal.FSHLog.rollWriter(FSHLog.java:689)
    at org.apache.hadoop.hbase.regionserver.LogRoller.run(LogRoller.java:148)
    at java.lang.Thread.run(Thread.java:748)
Caused by: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.SafeModeException): Cannot create file/hbase-nemo/WALs/node2,60020,1519732668326/node2%2C60020%2C1519732668326.default.1522768773233. Name node is in safe mode.
Resources are low on NN. Please add or free up more resources then turn off safe mode manually. NOTE:  If you turn off safe mode before adding resources, the NN will immediately return to safe mode. Use "hdfs dfsadmin -safemode leave" to turn safe mode off.
    at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkNameNodeSafeMode(FSNamesystem.java:1418)
    at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.startFileInt(FSNamesystem.java:2674)
    at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.startFile(FSNamesystem.java:2561)
    at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.create(NameNodeRpcServer.java:593)
    at org.apache.hadoop.hdfs.server.namenode.AuthorizationProviderProxyClientProtocol.create(AuthorizationProviderProxyClientProtocol.java:111)
    at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.create(ClientNamenodeProtocolServerSideTranslatorPB.java:393)
    at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
    at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:617)
    at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1073)
    at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2086)
    at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2082)
    at java.security.AccessController.doPrivileged(Native Method)
    at javax.security.auth.Subject.doAs(Subject.java:422)
    at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1693)
    at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2080)

    at org.apache.hadoop.ipc.Client.call(Client.java:1471)
    at org.apache.hadoop.ipc.Client.call(Client.java:1408)
    at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:230)
    at com.sun.proxy.$Proxy16.create(Unknown Source)
    at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.create(ClientNamenodeProtocolTranslatorPB.java:296)
    at sun.reflect.GeneratedMethodAccessor30.invoke(Unknown Source)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:256)
    at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:104)
    at com.sun.proxy.$Proxy17.create(Unknown Source)
    at sun.reflect.GeneratedMethodAccessor30.invoke(Unknown Source)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.apache.hadoop.hbase.fs.HFileSystem$1.invoke(HFileSystem.java:279)
    at com.sun.proxy.$Proxy18.create(Unknown Source)
    at org.apache.hadoop.hdfs.DFSOutputStream.newStreamForCreate(DFSOutputStream.java:1897)
    at org.apache.hadoop.hdfs.DFSClient.create(DFSClient.java:1738)
    at org.apache.hadoop.hdfs.DFSClient.create(DFSClient.java:1698)
    at org.apache.hadoop.hdfs.DistributedFileSystem$7.doCall(DistributedFileSystem.java:450)
    at org.apache.hadoop.hdfs.DistributedFileSystem$7.doCall(DistributedFileSystem.java:446)
    at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
    at org.apache.hadoop.hdfs.DistributedFileSystem.createNonRecursive(DistributedFileSystem.java:446)
    at org.apache.hadoop.fs.FileSystem.createNonRecursive(FileSystem.java:1124)
    at org.apache.hadoop.fs.FileSystem.createNonRecursive(FileSystem.java:1100)
    at org.apache.hadoop.hbase.regionserver.wal.ProtobufLogWriter.init(ProtobufLogWriter.java:90)
    at org.apache.hadoop.hbase.wal.DefaultWALProvider.createWriter(DefaultWALProvider.java:361)
    ... 4 more
2018-04-03 23:19:33,501 FATAL [regionserver/node2/192.168.2.3:60020.logRoller] regionserver.HRegionServer: ABORTING region server node2,60020,1519732668326: IOE in log roller
java.io.IOException: cannot get log writer
    at org.apache.hadoop.hbase.wal.DefaultWALProvider.createWriter(DefaultWALProvider.java:365)
    at org.apache.hadoop.hbase.regionserver.wal.FSHLog.createWriterInstance(FSHLog.java:724)
    at org.apache.hadoop.hbase.regionserver.wal.FSHLog.rollWriter(FSHLog.java:689)
    at org.apache.hadoop.hbase.regionserver.LogRoller.run(LogRoller.java:148)
    at java.lang.Thread.run(Thread.java:748)
Caused by: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.SafeModeException): Cannot create file/hbase-nemo/WALs/node2,60020,1519732668326/node2%2C60020%2C1519732668326.default.1522768773233. Name node is in safe mode.
Resources are low on NN. Please add or free up more resources then turn off safe mode manually. NOTE:  If you turn off safe mode before adding resources, the NN will immediately return to safe mode. Use "hdfs dfsadmin -safemode leave" to turn safe mode off.
    at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkNameNodeSafeMode(FSNamesystem.java:1418)
    at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.startFileInt(FSNamesystem.java:2674)
    at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.startFile(FSNamesystem.java:2561)
    at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.create(NameNodeRpcServer.java:593)
    at org.apache.hadoop.hdfs.server.namenode.AuthorizationProviderProxyClientProtocol.create(AuthorizationProviderProxyClientProtocol.java:111)
    at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.create(ClientNamenodeProtocolServerSideTranslatorPB.java:393)
    at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
    at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:617)
    at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1073)
    at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2086)
    at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2082)
    at java.security.AccessController.doPrivileged(Native Method)
    at javax.security.auth.Subject.doAs(Subject.java:422)
    at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1693)
    at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2080)

    at org.apache.hadoop.ipc.Client.call(Client.java:1471)
    at org.apache.hadoop.ipc.Client.call(Client.java:1408)
    at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:230)
    at com.sun.proxy.$Proxy16.create(Unknown Source)
    at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.create(ClientNamenodeProtocolTranslatorPB.java:296)
    at sun.reflect.GeneratedMethodAccessor30.invoke(Unknown Source)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:256)
    at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:104)
    at com.sun.proxy.$Proxy17.create(Unknown Source)
    at sun.reflect.GeneratedMethodAccessor30.invoke(Unknown Source)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.apache.hadoop.hbase.fs.HFileSystem$1.invoke(HFileSystem.java:279)
    at com.sun.proxy.$Proxy18.create(Unknown Source)
    at org.apache.hadoop.hdfs.DFSOutputStream.newStreamForCreate(DFSOutputStream.java:1897)
    at org.apache.hadoop.hdfs.DFSClient.create(DFSClient.java:1738)
    at org.apache.hadoop.hdfs.DFSClient.create(DFSClient.java:1698)
    at org.apache.hadoop.hdfs.DistributedFileSystem$7.doCall(DistributedFileSystem.java:450)
    at org.apache.hadoop.hdfs.DistributedFileSystem$7.doCall(DistributedFileSystem.java:446)
    at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
    at org.apache.hadoop.hdfs.DistributedFileSystem.createNonRecursive(DistributedFileSystem.java:446)
    at org.apache.hadoop.fs.FileSystem.createNonRecursive(FileSystem.java:1124)
    at org.apache.hadoop.fs.FileSystem.createNonRecursive(FileSystem.java:1100)
    at org.apache.hadoop.hbase.regionserver.wal.ProtobufLogWriter.init(ProtobufLogWriter.java:90)
    at org.apache.hadoop.hbase.wal.DefaultWALProvider.createWriter(DefaultWALProvider.java:361)
    ... 4 more
```

可以看到上面的日志出现IO 出现异常，无法获取 `log writer`:

```nil
2018-04-03 23:19:33,472 FATAL [regionserver/node2/192.168.2.3:60020.logRoller] regionserver.LogRoller: Aborting
java.io.IOException: cannot get log writer

2018-04-03 23:19:33,501 FATAL [regionserver/node2/192.168.2.3:60020.logRoller] regionserver.HRegionServer: ABORTING region server node2,60020,1519732668326: IOE in log roller
java.io.IOException: cannot get log writer
```

而无法获取 `log writer`, 对日志进行写入的原因是：

```nil
Caused by: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.SafeModeException): Cannot create file/hbase-nemo/WALs/node2,60020,1519732668326/node2%2C60020%2C1519732668326.default.1522768773233. Name node is in safe mode.
Resources are low on NN. Please add or free up more resources then turn off safe mode manually. NOTE:  If you turn off safe mode before adding resources, the NN will immediately return to safe mode. Use "hdfs dfsadmin -safemode leave" to turn safe mode off.
```

NameNode 进入了safe-mode, 关于safe-mode 的描述：
&gt;During start up the NameNode loads the file system state from the fsimage and the edits log file. It then waits for DataNodes to report their blocks so that it does not prematurely start replicating the blocks though enough replicas already exist in the cluster. During this time NameNode stays in Safemode. Safemode for the NameNode is essentially a read-only mode for the HDFS cluster, where it does not allow any modifications to file system or blocks. Normally the NameNode leaves Safemode automatically after the DataNodes have reported that most file system blocks are available. If required, HDFS could be placed in Safemode explicitly using bin/hadoop dfsadmin -safemode command. NameNode front page shows whether Safemode is on or off. A more detailed description and configuration is maintained as JavaDoc for setSafeMode().

NameNode 进入safe-mode 的原因是因为 `node-master`这台Master 机器的磁盘被应用日志打满了，导 致 NameNode 进入了只读的 `safe-mode`. 因为NameNode 进入readonly 的safe-mode 就无 法写入日志, 所以 Hbase 在出现异常之后，就开始把Hbase 的信息 dump 出来，并关闭
Region Server, 导致整个Hbase 集群宕机。

对于`node2` Region Server 下线的原因，猜测是 NameNode 服务器的磁盘用完，导致NameNode 进入read-only 的safe-mode, 又因为Hbase 存储的核心之一是WAL(write-ahead-log, 预写日志),较长时间无法写入日志，最终导致 Region Server 下线。


## <span class="section-num">3</span> 分析小结 {#分析小结}

经过这样的一翻排查，可以得出结论，最开始 `node1` 因为Hbase 和 ZooKeeper 的通信出现问题， 被认为是问题节点，下线了Region Server；

一个星期之后，`node-master`这台机器在同步的时候 出现问题，想要关闭WAL, 但是却因为没有充足的健康节点来替换出现问题的`node1`, 导致关闭 WAL 失败，也下线了Region Server. `node2` 这台机器因为作为 NameNode 的`node-master`服务器的磁盘用 完，导致NameNode 进入read-only 的safe-mode, 又因为Hbase 存储的核心之一是
WAL(write-ahead-log, 预写日志),较长时间无法写入日志，最终导致 Region Server 下线。


## <span class="section-num">4</span> 其他 {#其他}

还有一个关键点是为什么Hbase 和Zookeeper 的连接超时，Zookeeper 的日志只是简单地说明：

```nil
2018-03-13 14:47:46,926 [myid:1] - INFO  [QuorumPeer[myid=1]/0.0.0.0:2181:ZooKeeperServer@588] - Invalid session 0x161d6c1ae910002 for client /192.168.2.2:51611, probably expired
2018-03-13 14:47:49,612 [myid:1] - INFO  [QuorumPeer[myid=1]/0.0.0.0:2181:ZooKeeperServer@588] - Invalid session 0x161d6c1ae910001 for client /192.168.2.2:51612, probably expired
```

为什么 session 会无效，日志并没有给出说明，个人猜测可能是因为在部署了 Hbase/Zookeeper 的服务器上还部署了应用。

应用或者是Hbase 导致的长GC 导致ZooKeeper 停顿，并且导致session 超时无效。


## <span class="section-num">5</span> 结语 {#结语}

和同事交流之后，觉得以上的分析只是基于日志的猜测，可能Hbase 宕机的原因正如我所说， 或者另有原因，所以现在最关键的措施是加上对Hbase 的各种监控。

在Hbase 宕机的时候， 参考日志和详细的监控，比如连接数，CPU 使用率，内存，集群负载情况，每个节点情况。不然再遇到一次宕机，还是只能看日志，猜原因。

话分两头，现在的分析主要是基于Hbase 和ZooKeeper 的日志进行分析，简而言之就是捞日 志，查看信息; 捞日志，查看信息；通过工具找出日志中隐藏的关键时机，然后对时机前后发生的事情进行分析，这也是一个有趣的过程。

只是从1G 多的日志里面找出想要的内容，也不是一个容易的过程。

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

