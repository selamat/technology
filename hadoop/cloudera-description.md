# cloudera常见问题
## 1. cloudera目录使用情况
### cloudera将hdfs，yarn等封装成了单个服务
>* cloudera仓库目录/opt/cloudera/parcel-repo
>* 全部服务都在 **/opt/cloudera/parcels/CDH-5.3.2-1.cdh5.3.2.p0.10/lib/** 中
>* 命令行工具 **/opt/cloudera/parcels/CDH-5.3.2-1.cdh5.3.2.p0.10/bin** 目录下，脚本内容也指向各个服务的bin目录
>* cloudera运行日志文件目录 **/data/log/cloudera**中，可通过manager配置
>* Service Monitor存储了时间序列和健康数据，Yarn应用的元数据（时间序列和健康数据比较大），默认情况下，这些数据存储在/var/lib/cloudera-service-monitor/目录下。采用leveldb做key/ 	value形式的存储。Host Monitor一样，这部分数据比较大， 规划存储空间时需要考虑
>* 采用manager更新配置文件时，需要重新部署客户端才能更新成功

### cloudera主要目录就这些，在规划的时候主要注意目录大小，否则会经常因为存储空间不足报集群健康状况存在隐患

## 2. hdfs添加高可用之后，hbase启动失败
### 异常信息
```shell
Unhandled exception. Starting shutdown.
org.apache.hadoop.hbase.TableExistsException: hbase:namespace
		at org.apache.hadoop.hbase.master.handler.CreateTableHandler.prepare(CreateTableHandler.java:133)
		at org.apache.hadoop.hbase.master.TableNamespaceManager.createNamespaceTable(TableNamespaceManager.java:232)
		at org.apache.hadoop.hbase.master.TableNamespaceManager.start(TableNamespaceManager.java:86)
		at org.apache.hadoop.hbase.master.HMaster.initNamespace(HMaster.java:1069)
		at org.apache.hadoop.hbase.master.HMaster.finishInitialization(HMaster.java:942)
		at org.apache.hadoop.hbase.master.HMaster.run(HMaster.java:613)
		at java.lang.Thread.run(Thread.java:745)


Master exiting
java.lang.RuntimeException: HMaster Aborted
		at org.apache.hadoop.hbase.master.HMasterCommandLine.startMaster(HMasterCommandLine.java:194)
		at org.apache.hadoop.hbase.master.HMasterCommandLine.run(HMasterCommandLine.java:135)
		at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:70)
		at org.apache.hadoop.hbase.util.ServerCommandLine.doMain(ServerCommandLine.java:126)
		at org.apache.hadoop.hbase.master.HMaster.main(HMaster.java:2822)
```

### 原因：hdfs添加高可用，修改了hdfs namespace，造成hbase在zookeeper配置文件失效，因此需要登录zookeeper删除原有的hbase文件。

```shell
## stop hbase

## This command is used to repair MetaData of Hbase
hbase org.apache.hadoop.hbase.util.hbck.OfflineMetaRepair 
	
## use ls / to list files on zookeeper, then use 'rmr /hbase' to delete hbase's data
/opt/cloudera/parcels/CDH-5.3.2-1.cdh5.3.2.p0.10/lib/zookeeper/bin/zkCli.sh 

## start hbase
```

## 3. hive执行异常

### 建表语句，注意input采用正则时一定注意转义，否则即便建表成功，也无法正确加载数据

```sql
create table if not exists tmp_table (
	client string,
	gateway string,
	remote_addr string,
	host string,
	server_addr string,
	access_at string,
	request string,
	request_time string,
	upstream_response_time string,
	status string,
	body_bytes_sent string,
	referer string,
	agent string
) partitioned by (dt string)
ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
	"output.format.string" = "%1$s %2$s %3$s %4$s %5$s %6$s %7$s %8$s %9$s %10$s %11$s %12$s %13$s", 
	"input.regex" = "([^\\s]+)\\s+([^\\s]+)\\s+([^\\s]+)\\s+([^\\s]+)\\s+([^\\s]+)\\s+\\[(.*)\\]\\s+\"(-|.+)\"\\s+([^\\s]+)\\s+\"(-|.+)\"\\s+([^\\s]+)\\s+([^\\s]+)\\s+\"(-|.+)\"\\s+\"(.*)\""
)
LOCATION '/hive/warehouse/tmp_table';
```

### hive通过正则表达式建表后，执行mapr操作异常，普通建表执行mapr操作没问题，异常堆栈
```shell
Error: java.lang.RuntimeException: Error in configuring object
	at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:109)
	at org.apache.hadoop.util.ReflectionUtils.setConf(ReflectionUtils.java:75)
	at org.apache.hadoop.util.ReflectionUtils.newInstance(ReflectionUtils.java:133)
	at org.apache.hadoop.mapred.MapTask.runOldMapper(MapTask.java:446)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:343)
	at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:168)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1642)
	at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:163)
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:106)
	... 9 more
Caused by: java.lang.RuntimeException: Error in configuring object
	at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:109)
	at org.apache.hadoop.util.ReflectionUtils.setConf(ReflectionUtils.java:75)
	at org.apache.hadoop.util.ReflectionUtils.newInstance(ReflectionUtils.java:133)
	at org.apache.hadoop.mapred.MapRunner.configure(MapRunner.java:38)
	... 14 more
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:106)
	... 17 more
Caused by: java.lang.RuntimeException: Map operator initialization failed
	at org.apache.hadoop.hive.ql.exec.mr.ExecMapper.configure(ExecMapper.java:157)
	... 22 more
Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: java.lang.ClassNotFoundException: Class org.apache.hadoop.hive.contrib.serde2.RegexSerDe not found
	at org.apache.hadoop.hive.ql.exec.MapOperator.getConvertedOI(MapOperator.java:334)
	at org.apache.hadoop.hive.ql.exec.MapOperator.setChildren(MapOperator.java:352)
	at org.apache.hadoop.hive.ql.exec.mr.ExecMapper.configure(ExecMapper.java:126)
	... 22 more
Caused by: java.lang.ClassNotFoundException: Class org.apache.hadoop.hive.contrib.serde2.RegexSerDe not found
	at org.apache.hadoop.conf.Configuration.getClassByName(Configuration.java:1953)
	at org.apache.hadoop.hive.ql.exec.MapOperator.getConvertedOI(MapOperator.java:304)
	... 24 more
```

### 产生原因，建表成功，说明建表语句中引用的serde类加载无误，但执行类似count操作，出现上述异常，说明执行jar加载异常，通过排查发现  **Class org.apache.hadoop.hive.contrib.serde2.RegexSerDe not found**  为异常根源，hive-contrib jar包已经在CLASSPATH中，但不生效，所以只有通过外部引用的方式解决。解决方式：

```shell    
## add this code to hive-site.xml
<property>
	<name>hive.aux.jars.path</name>
  	<value>file:///usr/lib/hive/lib/hive-contrib-0.7.0-CDH3B4.jar</value>
  	<description>These JAR file are available to all users for all jobs</description>
</property>

##	or add HIVE_AUX_JARS_PATH=/opt/hive-ext to hive-env.sh， 第二种方式更值得使用，使用hive时可能会引入自定义jar或python脚本，通过配置路径，可将外部程序统一管理

## 在hive中可以通过set -v查看已经加载的jar和路径等
```

## 4. agent启动失败，造成manager实例管理中，多出一台一模一样的实例，老实例无法启动
### manager重启agent实例失败，agent日志打印如下

```shell
Starting Supervisor daemon manager...
Error: Another program is already listening on a port that one of our HTTP servers is configured to use.  Shut this program down first before starting supervisord.

For help, use /usr/bin/supervisord -h
...fail!
```

### 通过查看supervisor进程信息和端口kill无用进程并释放端口，重启agent成功。
### 通过google，cloudera官方也认为这是unusual的，下次出现一定完整记录情况，避免日后正式集群出现这样的问题，造成事故
