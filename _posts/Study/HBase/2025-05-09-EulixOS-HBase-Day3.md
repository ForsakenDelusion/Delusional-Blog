---
title: 傲来大数据方向HBase组优化日报-Day3
date: 2025-05-09 00:02:01 +08:00
filename: 2025-05-09-EulixOS-HBase-Day3
categories:
  - Study
  - HBase
tags:
  - DataBase
  - EulixOS
  - BigData
dir: Study/HBase
share: true
---
## 今日计划

熟悉HBase使用，寻找RVV优化示例。

## RISCV环境中部署HBase

### QEMU下RISCV环境HBase编译内存不足


![Day3-20250509.png](../../../assets/images/Day3-20250509.png)

### Maven Build错误

运行脚本首次编译libprotoc 25.4成功之后进行到mvn安装的环节的时候报错。

```shell
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------< org.apache.maven:standalone-pom >-------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] --------------------------------[ pom ]---------------------------------
[INFO]
[INFO] --- maven-install-plugin:2.4:install-file (default-cli) @ standalone-pom ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  17.358 s
[INFO] Finished at: 2025-05-09T00:31:55+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-install-plugin:2.4:install-file (default-cli) on projectstandalone-pom: The parameters 'file' for goal org.apache.maven.plugins:maven-install-plugin:2.4:install-file are missing or invalid -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginParameterException
./deploy_protobuf.sh: line 116: -DgroupId=com.google.protobuf: command not found
./deploy_protobuf.sh: line 118: -DartifactId=protoc: command not found
./deploy_protobuf.sh: line 120: -Dversion=3.25.4: command not found
./deploy_protobuf.sh: line 122: -Dclassifier=linux-riscv64: command not found
./deploy_protobuf.sh: line 124: -Dpackaging=exe: command not found
./deploy_protobuf.sh: line 125: -Dfile=/usr/local/bin/protoc: No such file or directory
this script is finished
```

以下这一段说明脚本的`\`换行并未能被正确解析，（回头看好像是每行之间又加了一个回车？）
```shell
mvn install:install-file \

            -DgroupId=com.google.protobuf \

            -DartifactId=protoc \

            -Dversion="${PROTOC_VERSION}" \

            -Dclassifier=linux-riscv64 \

            -Dpackaging=exe \

            -Dfile=/usr/local/bin/protoc
```

所以我们需要修改一下脚本的mvn安装命令。去掉`/`即可，牺牲一下可读性。

```shell
mvn install:install-file -DgroupId=com.google.protobuf -DartifactId=protoc -Dversion="${PROTOC_VERSION}" -Dclassifier=linux-riscv64 -Dpackaging=exe -Dfile=/usr/local/bin/protoc
```

接下来重新运行脚本，即可编译成功

```shell
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------------< org.apache.maven:standalone-pom >-------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] --------------------------------[ pom ]---------------------------------
[INFO] 
[INFO] --- maven-install-plugin:2.4:install-file (default-cli) @ standalone-pom ---
[INFO] Installing /usr/local/bin/protoc to /root/.m2/repository/com/google/protobuf/protoc/3.25.4/protoc-3.25.4-linux-riscv64.exe
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  18.085 s
[INFO] Finished at: 2025-05-09T00:58:43+08:00
[INFO] ------------------------------------------------------------------------
this script is finished
```

同样的，deploy_protobuf_2.sh这个脚本也要做同样的处理。

### 针对mvn构建速度过慢的探究

在deploy_hbase_thirdparty部分尝试了采用`-T`参数指定编译线程，但是相关依赖好像并不支持多线程构建

```shell
INFO] ---------< org.apache.hbase.thirdparty:hbase-shaded-protobuf >----------
[INFO] Building Apache HBase Patched and Relocated (Shaded) Protobuf 4.1.5
[INFO] --------------------------------[ jar ]---------------------------------
[WARNING] *****************************************************************
[WARNING] * Your build is requesting parallel execution, but project      *
[WARNING] * contains the following plugin(s) that have goals not marked   *
[WARNING] * as @threadSafe to support parallel building.                  *
[WARNING] * While this /may/ work fine, please look for plugin updates    *
[WARNING] * and/or request plugins be made thread-safe.                   *
[WARNING] * If reporting an issue, report it against the plugin in        *
[WARNING] * question, not against maven-core                              *
[WARNING] *****************************************************************
[WARNING] The following plugins are not marked @threadSafe in Apache HBase Patched and Relocated (Shaded) Protobuf:
[WARNING] org.apache.maven.plugins:maven-patch-plugin:1.2
[WARNING] Enable debug to see more precisely which goals are not marked @threadSafe.
[WARNING] *****************************************************************

```

随后又在deploy_hbase构建阶段指定多线程，发现支持多线程编译

```shell
[INFO] Using the MultiThreadedBuilder implementation with a thread count of 16
```

### patch问题

>如果你遇到如下字样的报错，很可能也是patch没打

```shell
[ERROR] unknown os.arch: riscv64 -> ［Help 1］
[ERROR] 

[ERROR]  To see the full stack trace of the errors,re-run Maven with the -e swit ch.
[ERROR]  Re-run Maven using the -X switch to enable full debug logging. 
[ERROR]
[ERROR]  For more information about the errors and possible solutions, please rea d the following articles：
```


运行deploy_hbase_thirdparty.sh的时候出现报错

```shell
./deploy_hbase_thirdparty.sh: line 19: /root/hbase_thirdparty_deploy//patches/hbase-thirdparty.patch: No such file or directory
```

~~但貌似并不影响编译~~

找到问题所在了，这里其实要手动添加patch文件，在部署脚本的同级目录下执行

```shell
mkdir patches
cd patches
vim hbase-thirdparty.patch
```

将以下内容复制进去

```shell
diff --git a/hbase-shaded-protobuf/pom.xml b/hbase-shaded-protobuf/pom.xml
index 8748a39..16f7ce0 100644
--- a/hbase-shaded-protobuf/pom.xml
+++ b/hbase-shaded-protobuf/pom.xml
@@ -94,7 +94,7 @@
                  <artifactItem>
                   <groupId>com.google.protobuf</groupId>
                   <artifactId>protobuf-java</artifactId>
-                  <version>${protobuf.version}</version>
+                  <version>3.25.4</version>
                   <classifier>sources</classifier>
                   <type>jar</type>
                   <overWrite>true</overWrite>

```

再次运行脚本，即可正确打上patch。

上面的patch打好之后，继续运行，又会出现一个报错

```shell
./deploy_hbase.sh: line 38: ../patches/hbase-2.5.10.patch: No such file or directory
```

在有了上述经验之后，我们可以快速得知，又是忘patch了，于是如法炮制。不过在进行后续操作时，我放弃了用脚本部署，选择了手动运行后续的脚本。

在运行deploy_hbase的时候又需要打一个patch，名字是`hbase-2.5.10.patch`，

```shell
diff --git a/hbase-protocol-shaded/pom.xml b/hbase-protocol-shaded/pom.xml
index b607d05..06792b3 100644
--- a/hbase-protocol-shaded/pom.xml
+++ b/hbase-protocol-shaded/pom.xml
@@ -34,7 +34,7 @@
     <!--Version of protobuf that hbase uses internally (we shade our pb)
          Must match what is out in hbase-thirdparty include.
     -->
-    <internal.protobuf.version>3.24.3</internal.protobuf.version>
+    <internal.protobuf.version>3.25.4</internal.protobuf.version>
   </properties>
   <dependencies>
     <!--BE CAREFUL! Any dependency added here needs to be
diff --git a/pom.xml b/pom.xml
index a58a062..39cfdef 100644
--- a/pom.xml
+++ b/pom.xml
@@ -2590,7 +2590,7 @@
       <extension>
         <groupId>kr.motd.maven</groupId>
         <artifactId>os-maven-plugin</artifactId>
-        <version>${os.maven.version}</version>
+        <version>1.7.1</version>
       </extension>
     </extensions>
   </build>

```

### 莫名报错

完成上述任务之后，又出现报错

```shell
ERROR] [ERROR] Some problems were encountered while processing the POMs:
[ERROR] Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-mapreduce of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist @ 
[ERROR] Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-protocol of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist @ 
[ERROR] Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-client of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist @ 
[ERROR] Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-examples of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist @ 
[ERROR] Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-assembly of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist @ 
[ERROR] Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-rest of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist @ 
[ERROR] Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-checkstyle of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist @ 
[ERROR] Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-rsgroup of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist @ 
[ERROR] Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-hadoop2-compat of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist @ 
 @ 
[ERROR] The build could not read 1 project -> [Help 1]
[ERROR]   
[ERROR]   The project org.apache.hbase:hbase:2.5.10 (/root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml) has 9 errors
[ERROR]     Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-mapreduce of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist
[ERROR]     Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-protocol of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist
[ERROR]     Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-client of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist
[ERROR]     Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-examples of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist
[ERROR]     Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-assembly of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist
[ERROR]     Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-rest of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist
[ERROR]     Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-checkstyle of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist
[ERROR]     Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-rsgroup of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist
[ERROR]     Child module /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-hadoop2-compat of /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/pom.xml does not exist
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/ProjectBuildingException
```

推测是更改patch为完成走完构建流程，所以再次执行完整脚本，问题解决。

### 构建过程中卡住，CPU内存都没有占用

```shell
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ hbase-protocol-shaded ---
[INFO] Compiling 38 source files to /root/hbase_2.5.10_deploy/hbase-2.5.10RC1/hbase-protocol-shaded/target/classes
```

执行到这里的时候出现的问题，未能复现，再次执行编译后问题消失。