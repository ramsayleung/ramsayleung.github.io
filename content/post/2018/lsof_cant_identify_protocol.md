+++
title = "lsof can't identify protocol"
date = 2018-04-11T15:24:00+08:00
lastmod = 2022-02-24T23:04:29+08:00
tags = ["linux"]
categories = ["linux"]
draft = false
toc = true
+++

Socket 泄漏引起的Tomcat 宕机问题分析

在2018年4月9号下午，收到反馈：测试集群部分接口访问有问题，请求时而正常，时而超时。

最近的测试环境真的是问题多多，可是测试环境就是我搭建的，冏。查看日志发现87 这台 服务器的Tomcat 无法访问：

```log
2018-04-09 17:41:31,568 - [ERROR] - from org.apache.tomcat.util.net.NioEndpoint in http-nio-47001-Acceptor-0
Socket accept failed
java.io.IOException: Too many open files
at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)
at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:422)
at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:250)
at org.apache.tomcat.util.net.NioEndpoint$Acceptor.run(NioEndpoint.java:825)
at java.lang.Thread.run(Thread.java:745)

2018-04-09 17:41:33,168 - [ERROR] - from org.apache.tomcat.util.net.NioEndpoint in http-nio-47001-Acceptor-0
Socket accept failed
java.io.IOException: Too many open files
at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)
at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:422)
at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:250)
at org.apache.tomcat.util.net.NioEndpoint$Acceptor.run(NioEndpoint.java:825)
at java.lang.Thread.run(Thread.java:745)
```


## <span class="section-num">1</span> Linux 文件句柄限制 {#linux-文件句柄限制}

报错看起来像是进程打开文件句柄的个数达到了linux的限制。而这种限制是分为系统层面的和用户层面的


### <span class="section-num">1.1</span> 系统层面 {#系统层面}

系统层面的在：/proc/sys/fs/file-max里设置

```sh
cat /proc/sys/fs/file-max
2442976
```


### <span class="section-num">1.2</span> 用户层面 {#用户层面}

用户层面的限制在：/etc/security/limits.conf里设定。通过`ulimit -a` 查看系统允许单个进程打开的最大文件数：

```sh
ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 192059
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 65536
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 10240
cpu time               (seconds, -t) unlimited
max user processes              (-u) 65535
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

单个进程可以打开的最大文件数是 65536


## <span class="section-num">2</span> lsof 显示大量open file {#lsof-显示大量open-file}

按照Tomcat 给出的报错信息，登录87 这台服务器检查打开的文件数，发现打开的文件超过70000:

```sh
lsof |wc -l
75924
```

然后找出打开文件数最多的进程，按文件数降序排列，左边是 open file 的数量，右边是进程ID：

```sh
lsof -n|awk '{print $2}'| sort | uniq -c | sort -nr | head
65966 25204
5374 20179
184 27275
65 5361
61 29421
16 22177
14 19751
12 22181
12 22179
12 22178
```

发现 `25204` 这个进程打开了大量的文件，已经超过了单个进程的最大文件数限制。而这个进程就是部署的java 应用对应的进程。打开的文件句柄数量已经超过Linux 限制，
Tomcat 无法创建新的socket 连接。


## <span class="section-num">3</span> can't identify protocol {#can-t-identify-protocol}

用 lsof 查看 java 应用打开的文件的时候，发现有非常多奇怪的输出：

```sh
java    25204 nemo *516u  sock                0,6       0t0 215137625 can't identify protocol
java    25204 nemo *517u  sock                0,6       0t0 215137626 can't identify protocol
java    25204 nemo *518u  sock                0,6       0t0 215137627 can't identify protocol
java    25204 nemo *519u  sock                0,6       0t0 215137628 can't identify protocol
java    25204 nemo *520u  sock                0,6       0t0 215137629 can't identify protocol
java    25204 nemo *521u  sock                0,6       0t0 215137630 can't identify protocol
java    25204 nemo *522u  sock                0,6       0t0 215137631 can't identify protocol
java    25204 nemo *523u  sock                0,6       0t0 215137634 can't identify protocol
java    25204 nemo *524u  sock                0,6       0t0 215137635 can't identify protocol
java    25204 nemo *525u  sock                0,6       0t0 215137636 can't identify protocol
java    25204 nemo *526u  sock                0,6       0t0 215137637 can't identify protocol
java    25204 nemo *527u  sock                0,6       0t0 215137638 can't identify protocol
java    25204 nemo *528u  sock                0,6       0t0 215137639 can't identify protocol
java    25204 nemo *529u  sock                0,6       0t0 215137640 can't identify protocol
java    25204 nemo *530u  sock                0,6       0t0 215137641 can't identify protocol
java    25204 nemo *531u  sock                0,6       0t0 215137642 can't identify protocol
java    25204 nemo *532u  sock                0,6       0t0 215137644 can't identify protocol
java    25204 nemo *533u  sock                0,6       0t0 215137646 can't identify protocol
```

统计之后发现， `can't identify protocol` 这样的文件数量非常多：

```sh
lsof -p 25204|grep "can't identify protocol"|wc -l
64214
```

也就是大部份打开的文件都是属于 `cant' identify  protocol` 的文件。


## <span class="section-num">4</span> 问题定位 {#问题定位}

Google 搜索之后发现，这个 `cant' identify protocol` 的东东出现的原因是因为 这些
sockets 处于 `CLOSED` 的状态，但是却没有真正close 掉，正处于 `half-close` 状态。因此，如果使用 `netstat` 来查看socket 状态，是不会显示这些 `half-close`的 socket 的：

```sh
netstat  -nat |wc -l
881
```

使用 `netstat` 的改进版本 `ss` 就能发现大量处于 `Closed` 状态的 socket:

```sh
ss -s
Total: 76052 (kernel 76254)
TCP:   75924 (estab 123, closed 75524, orphaned 0, synrecv 0, timewait 173/0), ports 104

Transport Total     IP        IPv6
*         76254    -         -
RAW       0         0         0
UDP       9         6         3
TCP       116       80        36
INET      125       86        39
FRAG      0         0         0
```

接着查看内核的 socket 情况：

```sh
cat /proc/net/sockstat
sockets: used 75724
TCP: inuse 886 orphan 0 tw 0 alloc 72134 mem 222
UDP: inuse 5 mem 0
UDPLITE: inuse 0
RAW: inuse 0
FRAG: inuse 0 memory 0
```

很多的 socket 处于 `alloc`, 只有少量的 socket 处于 `inuse`. 可以确认是 java 应用出现了 socket fd 的泄漏。 但是为什么会有那么多的socket 泄漏呢？


## <span class="section-num">5</span> 大胆假设 {#大胆假设}

现在可以确定的是 java应用出现了问题，导致了socket 泄漏，让 Tomcat 无法建立新连接，最终宕机。既然导致问题出现的是 java 应用，那么就应该去检查应用日志。

```log
2018-04-09 17:41:31,491 - [ERROR] - from com.alibaba.druid.pool.DruidDataSource in Druid-ConnectionPool-CreateScheduler--4-thread-214
create connection error, url: jdbc:mysql://test-server-host:3306/db_name?readOnlyPropagatesToServer=false&rewriteBatchedStatements=true&failOverReadOnly=false&socketTimeout=6000&connectTimeout=20000&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true&characterEncoding=utf-8&autoReconnect=true
com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Could not create connection to database server. Attempted reconnect 3 times. Giving up.
at sun.reflect.GeneratedConstructorAccessor169.newInstance(Unknown Source)
at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
at com.mysql.jdbc.Util.getInstance(Util.java:408)
at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:918)
at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:897)
at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:886)
at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:860)
at com.mysql.jdbc.ConnectionImpl.connectWithRetries(ConnectionImpl.java:2163)
at com.mysql.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:2088)
at com.mysql.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:806)
at com.mysql.jdbc.JDBC4Connection.<init>(JDBC4Connection.java:47)
at sun.reflect.GeneratedConstructorAccessor152.newInstance(Unknown Source)
at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
at com.mysql.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:410)
at com.mysql.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:328)
at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:148)
at com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:211)
at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:142)
at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1423)
at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1477)
at com.alibaba.druid.pool.DruidDataSource$CreateConnectionTask.runInternal(DruidDataSource.java:1884)
at com.alibaba.druid.pool.DruidDataSource$CreateConnectionTask.run(DruidDataSource.java:1849)
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
at java.util.concurrent.FutureTask.run(FutureTask.java:266)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
at java.lang.Thread.run(Thread.java:745)
Caused by: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.
at sun.reflect.GeneratedConstructorAccessor157.newInstance(Unknown Source)
at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:989)
at com.mysql.jdbc.MysqlIO.<init>(MysqlIO.java:341)
at com.mysql.jdbc.ConnectionImpl.coreConnect(ConnectionImpl.java:2251)
at com.mysql.jdbc.ConnectionImpl.connectWithRetries(ConnectionImpl.java:2104)
... 21 common frames omitted
Caused by: java.net.SocketException: Too many open files
at java.net.Socket.createImpl(Socket.java:460)
at java.net.Socket.getImpl(Socket.java:520)
at java.net.Socket.setTcpNoDelay(Socket.java:980)
at com.mysql.jdbc.StandardSocketFactory.configureSocket(StandardSocketFactory.java:132)
at com.mysql.jdbc.StandardSocketFactory.connect(StandardSocketFactory.java:203)
at com.mysql.jdbc.MysqlIO.<init>(MysqlIO.java:300)
... 23 common frames omitted

2018-04-09 17:41:31,568 - [ERROR] - from org.apache.tomcat.util.net.NioEndpoint in http-nio-47001-Acceptor-0
Socket accept failed
java.io.IOException: Too many open files
at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)
at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:422)
at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:250)
at org.apache.tomcat.util.net.NioEndpoint$Acceptor.run(NioEndpoint.java:825)
at java.lang.Thread.run(Thread.java:745)

2018-04-09 17:41:33,168 - [ERROR] - from org.apache.tomcat.util.net.NioEndpoint in http-nio-47001-Acceptor-0
Socket accept failed
java.io.IOException: Too many open files
at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)
at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:422)
at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:250)
at org.apache.tomcat.util.net.NioEndpoint$Acceptor.run(NioEndpoint.java:825)
at java.lang.Thread.run(Thread.java:745)

2018-04-09 17:41:34,470 - [ERROR] - from com.alibaba.druid.pool.DruidDataSource in Druid-ConnectionPool-CreateScheduler--4-thread-216
create connection error, url: jdbc:mysql://test-server-url:3306/db_name?readOnlyPropagatesToServer=false&rewriteBatchedStatements=true&failOverReadOnly=false&socketTimeout=6000&connectTimeout=20000&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true&characterEncoding=utf-8&autoReconnect=true
com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Could not create connection to database server. Attempted reconnect 3 times. Giving up.
at sun.reflect.GeneratedConstructorAccessor169.newInstance(Unknown Source)
at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
at com.mysql.jdbc.Util.getInstance(Util.java:408)
at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:918)
at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:897)
at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:886)
at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:860)
at com.mysql.jdbc.ConnectionImpl.connectWithRetries(ConnectionImpl.java:2163)
at com.mysql.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:2088)
at com.mysql.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:806)
at com.mysql.jdbc.JDBC4Connection.<init>(JDBC4Connection.java:47)
at sun.reflect.GeneratedConstructorAccessor152.newInstance(Unknown Source)
at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
at com.mysql.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:410)
at com.mysql.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:328)
at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:148)
at com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:211)
at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:142)
at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1423)
at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1477)
at com.alibaba.druid.pool.DruidDataSource$CreateConnectionTask.runInternal(DruidDataSource.java:1884)
at com.alibaba.druid.pool.DruidDataSource$CreateConnectionTask.run(DruidDataSource.java:1849)
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
at java.util.concurrent.FutureTask.run(FutureTask.java:266)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
at java.lang.Thread.run(Thread.java:745)
Caused by: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.
at sun.reflect.GeneratedConstructorAccessor157.newInstance(Unknown Source)
at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:989)
at com.mysql.jdbc.MysqlIO.<init>(MysqlIO.java:341)
at com.mysql.jdbc.ConnectionImpl.coreConnect(ConnectionImpl.java:2251)
at com.mysql.jdbc.ConnectionImpl.connectWithRetries(ConnectionImpl.java:2104)
... 23 common frames omitted
Caused by: java.net.SocketException: Too many open files
at java.net.Socket.createImpl(Socket.java:460)
at java.net.Socket.getImpl(Socket.java:520)
at java.net.Socket.setTcpNoDelay(Socket.java:980)
at com.mysql.jdbc.StandardSocketFactory.configureSocket(StandardSocketFactory.java:132)
at com.mysql.jdbc.StandardSocketFactory.connect(StandardSocketFactory.java:203)
at com.mysql.jdbc.MysqlIO.<init>(MysqlIO.java:300)
... 25 common frames omitted

2018-04-09 17:41:34,769 - [ERROR] - from org.apache.tomcat.util.net.NioEndpoint in http-nio-47001-Acceptor-0
Socket accept failed
java.io.IOException: Too many open files
at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)
at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:422)
at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:250)
at org.apache.tomcat.util.net.NioEndpoint$Acceptor.run(NioEndpoint.java:825)
at java.lang.Thread.run(Thread.java:745)
```

检查日志发现，在 Tomcat 彻底挂机之前，曾经有比较大量的数据源连接池出错，无法访问 Mysql, 但是非常奇怪的是，在87 这台机器上面，是可以使用 mysql 命令行连接到测试数据库的，说明 Mysql 的连接是没有问题。

但是数据源连接就会出错！！ 真的是很奇怪，为什么连接池会报错，有没有可能是这些异常导致 socket 泄漏呢？后来，在本地运行应用，有时候会发现IDE 的控制台报错：

```log
2018-04-11 09:43:48,363 - [ERROR] - from com.alibaba.druid.pool.DruidDataSource in poolTaskScheduler-11
discard connection
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet successfully received from the server was 100,610 milliseconds ago.  The last packet sent successfully to the server was 0 milliseconds ago.
at sun.reflect.GeneratedConstructorAccessor108.newInstance(Unknown Source)
at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:989)
at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3556)
at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3456)
at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3897)
at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2524)
at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2677)
at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2545)
at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2503)
at com.mysql.jdbc.StatementImpl.executeQuery(StatementImpl.java:1369)
at com.alibaba.druid.filter.FilterChainImpl.statement_executeQuery(FilterChainImpl.java:2363)
at com.alibaba.druid.filter.FilterAdapter.statement_executeQuery(FilterAdapter.java:2481)
at com.alibaba.druid.filter.FilterEventAdapter.statement_executeQuery(FilterEventAdapter.java:302)
at com.alibaba.druid.filter.FilterChainImpl.statement_executeQuery(FilterChainImpl.java:2360)
at com.alibaba.druid.proxy.jdbc.StatementProxyImpl.executeQuery(StatementProxyImpl.java:211)
at com.alibaba.druid.pool.DruidPooledStatement.executeQuery(DruidPooledStatement.java:138)
at com.taobao.tddl.atom.jdbc.TStatementWrapper.executeQuery(TStatementWrapper.java:315)
at com.taobao.tddl.group.jdbc.TGroupStatement.executeQueryOnConnection(TGroupStatement.java:549)
at com.taobao.tddl.group.jdbc.TGroupStatement$4.tryOnDataSource(TGroupStatement.java:633)
at com.taobao.tddl.group.jdbc.TGroupStatement$4.tryOnDataSource(TGroupStatement.java:615)
at com.taobao.tddl.group.dbselector.AbstractDBSelector.tryOnDataSourceHolder(AbstractDBSelector.java:155)
at com.taobao.tddl.group.dbselector.OneDBSelector.tryExecuteInternal(OneDBSelector.java:52)
at com.taobao.tddl.group.dbselector.AbstractDBSelector.tryExecute(AbstractDBSelector.java:405)
at com.taobao.tddl.group.dbselector.AbstractDBSelector.tryExecute(AbstractDBSelector.java:412)
at com.taobao.tddl.group.jdbc.TGroupStatement.executeQuery(TGroupStatement.java:488)
at com.taobao.tddl.group.jdbc.TGroupStatement.executeInternal(TGroupStatement.java:131)
at com.taobao.tddl.group.jdbc.TGroupStatement.execute(TGroupStatement.java:101)
at com.taobao.tddl.repo.mysql.spi.My_JdbcHandler.executeQuery(My_JdbcHandler.java:521)
at com.taobao.tddl.repo.mysql.spi.My_Cursor.init(My_Cursor.java:106)
at com.taobao.tddl.repo.mysql.handler.QueryMyHandler.handle(QueryMyHandler.java:89)
at com.taobao.tddl.executor.AbstractGroupExecutor.executeInner(AbstractGroupExecutor.java:47)
at com.taobao.tddl.executor.AbstractGroupExecutor.execByExecPlanNode(AbstractGroupExecutor.java:36)
at com.taobao.tddl.executor.TopologyExecutor.execByExecPlanNode(TopologyExecutor.java:66)
at com.taobao.tddl.executor.MatrixExecutor.execByExecPlanNodeByOne(MatrixExecutor.java:670)
at com.taobao.tddl.executor.MatrixExecutor.execByExecPlanNode(MatrixExecutor.java:659)
at com.taobao.tddl.executor.MatrixExecutor.execute(MatrixExecutor.java:137)
at com.taobao.tddl.matrix.jdbc.TConnection.executeSQL(TConnection.java:241)
at com.taobao.tddl.matrix.jdbc.TPreparedStatement.executeSQL(TPreparedStatement.java:64)
at com.taobao.tddl.matrix.jdbc.TStatement.executeInternal(TStatement.java:133)
at com.taobao.tddl.matrix.jdbc.TPreparedStatement.execute(TPreparedStatement.java:49)
at sun.reflect.GeneratedMethodAccessor148.invoke(Unknown Source)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at org.apache.ibatis.logging.jdbc.PreparedStatementLogger.invoke(PreparedStatementLogger.java:59)
at com.sun.proxy.$Proxy102.execute(Unknown Source)
at org.apache.ibatis.executor.statement.PreparedStatementHandler.query(PreparedStatementHandler.java:63)
at org.apache.ibatis.executor.statement.RoutingStatementHandler.query(RoutingStatementHandler.java:79)
at org.apache.ibatis.executor.SimpleExecutor.doQuery(SimpleExecutor.java:63)
at org.apache.ibatis.executor.BaseExecutor.queryFromDatabase(BaseExecutor.java:325)
at org.apache.ibatis.executor.BaseExecutor.query(BaseExecutor.java:156)
at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:109)
at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:83)
at sun.reflect.GeneratedMethodAccessor146.invoke(Unknown Source)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at org.apache.ibatis.plugin.Invocation.proceed(Invocation.java:49)
at fastfish.interceptor.DbLogInterceptor.intercept(DbLogInterceptor.java:49)
at org.apache.ibatis.plugin.Plugin.invoke(Plugin.java:61)
at com.sun.proxy.$Proxy100.query(Unknown Source)
at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:148)
at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:141)
at sun.reflect.GeneratedMethodAccessor145.invoke(Unknown Source)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:434)
at com.sun.proxy.$Proxy87.selectList(Unknown Source)
at org.mybatis.spring.SqlSessionTemplate.selectList(SqlSessionTemplate.java:231)
at org.apache.ibatis.binding.MapperMethod.executeForMany(MapperMethod.java:128)
at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:68)
at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:53)
at com.sun.proxy.$Proxy124.selectAll(Unknown Source)
at fastfish.services.BusinessService.getAll(BusinessService.java:73)
at fastfish.services.BusinessService.loadDB(BusinessService.java:38)
at sun.reflect.GeneratedMethodAccessor190.invoke(Unknown Source)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at org.springframework.scheduling.support.ScheduledMethodRunnable.run(ScheduledMethodRunnable.java:65)
at org.springframework.scheduling.support.DelegatingErrorHandlingRunnable.run(DelegatingErrorHandlingRunnable.java:54)
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
at java.util.concurrent.FutureTask.runAndReset$$$capture(FutureTask.java:308)
at java.util.concurrent.FutureTask.runAndReset(FutureTask.java)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:180)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:294)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at java.lang.Thread.run(Thread.java:748)
Caused by: java.io.EOFException: Can not read response from server. Expected to read 4 bytes, read 0 bytes before connection was unexpectedly lost.
at com.mysql.jdbc.MysqlIO.readFully(MysqlIO.java:3008)
at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3466)
... 83 common frames omitted
```

是数据池连接出错。但是我本地的应用确实是可以访问测试数据库的， 比较有趣的异常就是

```log
Caused by: java.io.EOFException: Can not read response from server. Expected to read 4 bytes, read 0 bytes before connection was unexpectedly lost.
```

数据还没有读完， Connection 就丢了。为什么会 lost connection 呢，可能数据库出问题，也可能是网络出了问题。

我还能从数据库读到数据，说明数据库没问题的，兼之这个异常只是偶尔出现，所以可能就是网络出问题了。

如此说来，是否可能是因为测试环境网络不稳定，连接池无法和 Mysql 保持连接，在丢掉 Connection 之后，连接池重新发起连接，但是因为网络不稳定又丢掉了Connection, 不断循环这个过程，导致建立的 socket 连接越 来越多，但是建立的 socket 很快就被Close 掉了，内核又没有把这些 Close 掉的 socket 资源回收掉，因此打开的 socket 文件越来越多，最后导致 Tomcat 因为打开的文件过多无法建立新的 socket 连接。


## <span class="section-num">6</span> 小心求证 {#小心求证}

如果连接池真的不断尝试连接Mysql 的话，必定会建立很多的连接，而Mysql 是会将这些记录保存下来的，检查Mysql 的变量：

{{< figure src="https://imgur.com/gAxepYH.png" >}}

查看Mysql 的文档关于 `Connection` 和 `Thread_connected` 的说明：

-   Connections

    > The number of connection attempts (successful or not) to the MySQL server.

-   Threads_connected

    > The number of currently open connections.

也就是说，当时共有20000 多的连接请求，但是真正被 Mysql accpet 并且服务的只有 28 个连接。看来的确是因为连接池的连接导致 socket 泄漏


### <span class="section-num">6.1</span> 更新 {#更新}

和运维同学沟通之后，发现丢连接的原因不是网络不稳定，而是测试集群都是虚拟机，内存 用光，导致无法建立新的连接，内核释放一部分资源之后又可以建立连接了。内存用完，我能怎么办，我也很无奈。


## <span class="section-num">7</span> 解决方法 {#解决方法}

虽说基本确定了 socket 泄漏的源头，但是对于内核为什么无法回收已经关闭 socket 的原因依然不明确。

最令人百思不得其解的是，部署了应用的测试服务器有两台，另外一 台服务器也有同样的连接池问题，但是却没有出现 socket 泄漏问题， 出现泄漏的只有 87 这台机器。真的令人费解. 所以最后解决方法就是撤下 87 服务器的应用，换一台服务器来部署。

新的服务器部署应用之后虽说也有同样的数据库连接池异常，但是却没有出现 socket 泄漏，初步定位是 87这台机器的内核环境存在问题。


## <span class="section-num">8</span> 参考 {#参考}

-   [tcp-socket文件句柄泄漏/](http://mdba.cn/2015/03/10/tcp-socket%E6%96%87%E4%BB%B6%E5%8F%A5%E6%9F%84%E6%B3%84%E6%BC%8F/)
-   [lsof-cant-identify-protocol/](https://idea.popcount.org/2012-12-09-lsof-cant-identify-protocol/)
