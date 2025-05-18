---
title: 傲来大数据方向HBase组优化日报-Day7
date: 2025-05-14 14:01:42 +08:00
filename: 2025-05-14-EulixOS-HBase-Day7
categories:
  - Study
  - HBase
tags:
  - BigData
  - DataBase
  - EulixOS
dir: Study/HBase
share: true
---
尝试使用JNI修改布隆过滤器部分组件，验证可行性

以下命令分别是在MacOS和Linux平台上生成对应动态链接库的命令，后续可能考虑要通过cmake构建动态链接库。

```shell
    clang++ -shared -std=c++11 \
    -I"$JAVA_HOME/include" \
    -I"$JAVA_HOME/include/darwin" \
    -o hbase-server/src/main/native/libhbase_bloom_filter_jni.dylib \
    hbase-server/src/main/native/BloomFilterUtilJni.cpp
```

```
g++ -shared -std=c++11 -fPIC \
    -I"$JAVA_HOME/include" \
    -I"$JAVA_HOME/include/linux" \
    -o hbase-server/src/main/native/libhbase_bloom_filter_jni.so \
    hbase-server/src/main/native/BloomFilterUtilJni.cpp
```

如果java代码没有涉及到更改的话，只更改cpp代码是不需要用mvn重新构建的，只需要在此运行上述命令即可。（毕竟动态链接库）

在hbase-server的pom.xml中添加头文件自动生成的字段（第一个plugin包裹），还需要动态的指定库文件的path（第二个plugin包裹）（重新生成头文件需要先用hbase-server里的pom clean一下 再compile）

```xml
 <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
            <compilerArgs>
                <arg>-h</arg>
                <arg>/root/hb/hbase-2.5.11/hbase-server/src/main/native</arg> <!-- 或者希望的输出目录 -->
            </compilerArgs>
        </configuration>
      </plugin>
          <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <configuration>
            <argLine>-Djava.library.path=/root/hb/hbase-2.5.11/hbase-server/src/main/native</argLine>
            <!-- 或者 -->
            <!-- <systemPropertyVariables>
                <java.library.path>/root/hb/hbase-2.5.11/hbase-server/src/main/native</java.library.path>
            </systemPropertyVariables> -->
        </configuration>
      </plugin>
```


```shell
//conf/hbase-env.sh
export HBASE_OPTS="-Djava.library.path=/root/hb/hbase-2.5.11/hbase-server/src/main/native"
```

记得加上我们的共享链接库路径，不然运行起来会报如下的错误

```shell
ERROR: Zookeeper GET could not be completed in 10000 ms
```


记得指定rootdir和tmpdir，方便每次测试完之后清除数据，不然在多版本测试中会出现缓存没更新导致的错误。

```xml
<property>
    <name>hbase.tmp.dir</name>
    <value>/root/hb/hbase-2.5.11/hbase-data/tmp</value>
</property>
<property>
  <name>hbase.master.hostname</name>
  <value>localhost</value>
</property>
<property>
  <name>hbase.regionserver.hostname</name>
  <value>localhost</value>
</property>
<property>
    <name>hbase.rootdir</name>
		<value>file:///root/hb/hbase-2.5.11/hbase-data/hbase</value>
</property>
```

同样的，也需要指定HBase运行的hostname，不然在docker中运行，很可能会错误的解析hostname为docker内的网卡配置。
##

### 坑点 1：HBase 服务注册的主机名与实际监听地址不一致或客户端无法正确访问

- 具体表现与报错信息：

1. HBase Performance Evaluation (PE) 工具启动后，长时间卡住，最终报错 hbase:meta,,1 is not online on legion-eulix,...。
    
```shell
2025-05-14T09:07:54,037 INFO  [main] client.RpcRetryingCallerImpl: ion: hbase:meta,,1 is not online on localhost,16020,1747213592524
        at org.apache.hadoop.hbase.regionserver.HRegionServer.getRegionByEncodedName(HRegionServer.java:3539)
        at org.apache.hadoop.hbase.regionserver.HRegionServer.getRegion(HRegionServer.java:3517)
        at org.apache.hadoop.hbase.regionserver.RSRpcServices.getRegion(RSRpcServices.java:1489)
        at org.apache.hadoop.hbase.regionserver.RSRpcServices.get(RSRpcServices.java:2556)
        at org.apache.hadoop.hbase.shaded.protobuf.generated.ClientProtos$ClientService$2.callBlockingMethod(ClientProtos.java:45002)
        at org.apache.hadoop.hbase.ipc.RpcServer.call(RpcServer.java:415)
        at org.apache.hadoop.hbase.ipc.CallRunner.run(CallRunner.java:124)
        at org.apache.hadoop.hbase.ipc.RpcHandler.run(RpcHandler.java:102)
        at org.apache.hadoop.hbase.ipc.RpcHandler.run(RpcHandler.java:82)
, details=row 'TestTable' on table 'hbase:meta' at region=hbase:meta,,1.1588230740, hostname=legion-eulix,16020,1747209674622, seqNum=-1, see https://s.apache.org/timeout
```
    

2. 后续的错误日志显示客户端尝试连接 Legion-Eulix/172.17.0.2:16020 但连接被拒绝 (Connection refused)。
    
```shell
2025-05-14T09:16:44,494 WARN  [RPCClient-NioEventLoopGroup-1-1] ipc.NettyRpcConnection: Exception encountered while connecting to the server Legion-Eulix:16020
org.apache.hbase.thirdparty.io.netty.channel.AbstractChannel$AnnotatedConnectException: Connection refused: Legion-Eulix/172.17.0.2:16020
Caused by: java.net.ConnectException: Connection refused
        at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method) ~[?:1.8.0_452]
        at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:716) ~[?:1.8.0_452]
        at org.apache.hbase.thirdparty.io.netty.channel.socket.nio.NioSocketChannel.doFinishConnect(NioSocketChannel.java:337) ~[hbase-shaded-netty-4.1.5.jar:?]
        at org.apache.hbase.thirdparty.io.netty.channel.nio.AbstractNioChannel$AbstractNioUnsafe.finishConnect(AbstractNioChannel.java:334) ~[hbase-shaded-netty-4.1.5.jar:?]
        at org.apache.hbase.thirdparty.io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:776) ~[hbase-shaded-netty-4.1.5.jar:?]
        at org.apache.hbase.thirdparty.io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:724) ~[hbase-shaded-netty-4.1.5.jar:?]
        at org.apache.hbase.thirdparty.io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:650) ~[hbase-shaded-netty-4.1.5.jar:?]
        at org.apache.hbase.thirdparty.io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:562) ~[hbase-shaded-netty-4.1.5.jar:?]
        at org.apache.hbase.thirdparty.io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:997) ~[hbase-shaded-netty-4.1.5.jar:?]
        at org.apache.hbase.thirdparty.io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) ~[hbase-shaded-netty-4.1.5.jar:?]
        at org.apache.hbase.thirdparty.io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30) ~[hbase-shaded-netty-4.1.5.jar:?]
        at java.lang.Thread.run(Thread.java:750) ~[?:1.8.0_452]
```
    

3. netstat 命令显示 HBase RegionServer (Java 进程) 监听在 172.17.0.2:16020。
    
```shell
root@Legion-Eulix [09:14:39] [~/hb/hbase-cmp] 
-> #  netstat -npl|grep 16020
tcp6       0      0 172.17.0.2:16020        :::*                    LISTEN      121562/java   
```

- 原因分析：

- HBase Master 和 RegionServer 在启动时，如果没有在 conf/hbase-site.xml 中明确配置 hbase.master.hostname 和 hbase.regionserver.hostname，它们会尝试自动检测主机名。在这个案例中，它们可能将主机名解析为 legion-eulix，对应 IP 172.17.0.2，并以此地址向 Zookeeper 注册。

- 客户端从 Zookeeper 获取到的是 legion-eulix:16020 这个地址。

- 如果 HBase 服务实际监听的 IP 接口与客户端尝试连接的 IP 不符，或者该 IP 对于客户端不可达（例如，172.17.0.2 是一个内部 Docker/WSL 网络 IP，而客户端从另一个网络视角访问），就会导致连接失败。

- 解决方案：

1. 在 conf/hbase-site.xml 文件中，明确指定 HBase Master 和 RegionServer 绑定的主机名。对于单机部署，推荐使用 localhost：

```xml
<property>
	<name>hbase.master.hostname</name>
	<value>localhost</value>
</property>
<property>
	<name>hbase.regionserver.hostname</name>
	<value>localhost</value>
</property>
```


2. 彻底停止并重启 HBase 服务 (./bin/stop-hbase.sh 后接 ./bin/start-hbase.sh)，以确保 HBase 使用新的配置启动并向 Zookeeper 注册。

### 坑点 2：Zookeeper 中存在陈旧的 HBase元数据

- 具体表现与报错信息：

- 即使 HBase 服务通过 netstat 确认已在 127.0.0.1:16000 (Master) 和 127.0.0.1:16020 (RegionServer) 上正确监听，PE 工具的错误日志依然显示它尝试连接旧的、错误的主机名 Legion-Eulix/172.17.0.2:16020。
    
```shell
2025-05-14T09:16:44,494 WARN  [RPCClient-NioEventLoopGroup-1-1] ipc.NettyRpcConnection: Exception encountered while connecting to the server Legion-Eulix:16020
org.apache.hbase.thirdparty.io.netty.channel.AbstractChannel$AnnotatedConnectException: Connection refused: Legion-Eulix/172.17.0.2:16020
```

- 错误详情中仍然包含旧的主机名信息：hostname=legion-eulix,16020,1747209674622。

- 原因分析：

- 客户端通过 Zookeeper 查找 hbase:meta Region 的位置。

- 如果 HBase 服务在配置更改（例如，从 legion-eulix 改为 localhost）后重启，但 Zookeeper 中关于 hbase:meta Region Server 的信息没有被正确更新或清除，Zookeeper 就会向客户端返回这些陈旧的、错误的地址信息。

- 解决方案：

1. 清理 HBase 的临时数据目录 hbase.tmp.dir (您配置的路径为 /root/hb/hbase-cmp/hbase-data/tmp) 和数据目录 hbase.rootdir（如果这是全新的测试并且可以容忍数据丢失）。这有助于确保一个完全干净的启动。

### 坑点 3：测试成功前出现 "NoNode for /hbase/hbaseid" 或 "No meta znode available"

- 具体表现与报错信息：

在最终成功的测试运行日志中，开头部分出现了类似以下的警告：
```shell
2025-05-14T09:18:28,818 WARN  [main] client.ConnectionImplementation: Retrieve cluster id failed
java.util.concurrent.ExecutionException: org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /hbase/hbaseid
        at java.util.concurrent.CompletableFuture.reportGet(CompletableFuture.java:357) ~[?:1.8.0_452]
        at java.util.concurrent.CompletableFuture.get(CompletableFuture.java:1908) ~[?:1.8.0_452]
        at org.apache.hadoop.hbase.client.ConnectionImplementation.retrieveClusterId(ConnectionImplementation.java:666) ~[hbase-client-2.5.11.jar:2.5.11]
        at org.apache.hadoop.hbase.client.ConnectionImplementation.(ConnectionImplementation.java:325) ~[hbase-client-2.5.11.jar:2.5.11]
        at org.apache.hadoop.hbase.client.ConnectionImplementation.(ConnectionImplementation.java:272) ~[hbase-client-2.5.11.jar:2.5.11]
        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method) ~[?:1.8.0_452]
        at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62) ~[?:1.8.0_452]
        at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45) ~[?:1.8.0_452]
        at java.lang.reflect.Constructor.newInstance(Constructor.java:423) ~[?:1.8.0_452]
        at org.apache.hadoop.hbase.client.ConnectionFactory.lambda$null$0(ConnectionFactory.java:233) ~[hbase-client-2.5.11.jar:2.5.11]
        at java.security.AccessController.doPrivileged(Native Method) ~[?:1.8.0_452]
        at javax.security.auth.Subject.doAs(Subject.java:422) ~[?:1.8.0_452]
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1938) ~[hadoop-common-2.10.2.jar:?]
        at org.apache.hadoop.hbase.security.User$SecureHadoopUser.runAs(User.java:328) ~[hbase-common-2.5.11.jar:2.5.11]
        at org.apache.hadoop.hbase.client.ConnectionFactory.lambda$createConnection$1(ConnectionFactory.java:232) ~[hbase-client-2.5.11.jar:2.5.11]
        at org.apache.hadoop.hbase.trace.TraceUtil.trace(TraceUtil.java:216) ~[hbase-common-2.5.11.jar:2.5.11]
        at org.apache.hadoop.hbase.client.ConnectionFactory.createConnection(ConnectionFactory.java:218) ~[hbase-client-2.5.11.jar:2.5.11]
        at org.apache.hadoop.hbase.client.ConnectionFactory.createConnection(ConnectionFactory.java:131) ~[hbase-client-2.5.11.jar:2.5.11]
        at org.apache.hadoop.hbase.PerformanceEvaluation.runTest(PerformanceEvaluation.java:2589) ~[hbase-mapreduce-2.5.11-tests.jar:2.5.11]
        at org.apache.hadoop.hbase.PerformanceEvaluation.run(PerformanceEvaluation.java:3137) ~[hbase-mapreduce-2.5.11-tests.jar:2.5.11]
        at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:76) ~[hadoop-common-2.10.2.jar:?]
        at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:90) ~[hadoop-common-2.10.2.jar:?]
        at org.apache.hadoop.hbase.PerformanceEvaluation.main(PerformanceEvaluation.java:3171) ~[hbase-mapreduce-2.5.11-tests.jar:2.5.11]
Caused by: org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /hbase/hbaseid
        at org.apache.zookeeper.KeeperException.create(KeeperException.java:118) ~[zookeeper-3.8.4.jar:3.8.4]
        at org.apache.zookeeper.KeeperException.create(KeeperException.java:54) ~[zookeeper-3.8.4.jar:3.8.4]
        at org.apache.hadoop.hbase.zookeeper.ReadOnlyZKClient$ZKTask$1.exec(ReadOnlyZKClient.java:185) ~[hbase-client-2.5.11.jar:2.5.11]
        at org.apache.hadoop.hbase.zookeeper.ReadOnlyZKClient.run(ReadOnlyZKClient.java:386) ~[hbase-client-2.5.11.jar:2.5.11]
        at java.lang.Thread.run(Thread.java:750) ~[?:1.8.0_452]
2025-05-14T09:18:33,158 INFO  [main] client.RpcRetryingCallerImpl: Call exception, tries=6, retries=16, started=4206 ms ago, cancelled=false, msg=No meta znode available, details=row 'TestTable' on table 'hbase:meta' at null, see https://s.apache.org/timeout
```
- 原因分析：

- 这些信息通常出现在 HBase Master 正在初始化或刚刚完成 Zookeeper 状态重建的过程中。当客户端（PE 工具）尝试连接时，HBase Master 可能尚未完全在 Zookeeper 中创建 /hbase/hbaseid（集群唯一标识）或 /hbase/meta-region-server（指向 hbase:meta 表所在的 RegionServer）等关键 znode。

- 这在 HBase 刚启动或 Zookeeper 数据被清理后首次启动时是正常的，客户端会进行重试。

- 解决方案：

- 这通常不是一个需要用户直接干预的“错误”，而是 HBase 启动过程中的一个暂时状态。客户端的重试机制最终会成功连接。

- 确保 HBase Master 有足够的时间完成其在 Zookeeper 中的初始化过程。在自动化脚本中，可以在启动 HBase 后加入短暂的等待时间，然后再运行客户端程序。

### 验证JNI正确的被调用

在CPP文件中添加一些DEBUG信息，然后再去看LOG，如果有对应LOG产生，那么便可说明正确的被启用。