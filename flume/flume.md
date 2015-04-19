# flume(write to hdfs2.0)

## 1. flume配置，采用flume自带的hdfs sink，下面给出sink部分的配置
```shell
# Description: one node flume connecting between
#              the scribe source and hdfs repository
#              more category to add...
# sink
agent.sinks.sink_hdfs.type = hdfs
agent.sinks.sink_hdfs.channel = ch_file
agent.sinks.sink_hdfs.hdfs.path = hdfs://{hdfs_host}:{port}/hive/warehouse/tmp_web_spider_log/dt=%Y-%m-%d
agent.sinks.sink_hdfs.hdfs.fileType = DataStream
agent.sinks.sink_hdfs.hdfs.writeFormat = TEXT
agent.sinks.sink_hdfs.hdfs.rollInterval = 0
agent.sinks.sink_hdfs.hdfs.rollSize = 0	agent.sinks.sink_hdfs.hdfs.rollCount = 0
agent.sinks.sink_hdfs.hdfs.idleTimeout = 300
agent.sinks.sink_hdfs.hdfs.callTimeout = 300000
agent.sinks.sink_hdfs.hdfs.rollTimerPoolSize = 1000
agent.sinks.sink_hdfs.hdfs.threadsPoolSize = 5000
agent.sinks.sink_hdfs.hdfs.filePrefix = analytics-%Y-%m-%d-%H
```

## 2. 修改conf目录下flume-env.sh，除了配置JAVAHOME之外还要配置HADOOP_HOME，否则会出现HDFS IPC VERSION does not match异常
```shell
HADOOP_HOME={HADOOP_DIR}
```

## 3. 启动测试命令，测试无误后可以使用nohup或者supervisor启动后台进程
```shell
bin/flume-ng agent --conf conf --conf-file {CONF_DIR} --name agent -Dflume.root.logger=INFO,console -Dflume.monitoring.type=http -Dflume.monitoring.port=34558
```
    
## 4. nohup启动进程
```shell
# start flume by nohup, but not enough, because if the process dead, system will not try to start the process

nohup bin/flume-ng agent --conf conf --conf-file {CONF_DIR} --name agent -Dflume.log.file={LOG_FILE} -Dflume.monitoring.type=http -Dflume.monitoring.port=34545 &
```

## 5. supervisor启动进程
### supervisor flume配置文件

```shell
[program:flume]
command=/data/server/services/flume/bin/flume-ng agent -c /data/server/services/flume/conf -f /data/server/services/flume/conf/flume-agent.conf -n agent
directory=/data/server/services/flume/bin
stopasgroup=true
stopwaitsecs=600
```

### supervisor启动
```shell
supervisorctl -c /etc/supervisor/supervisord.conf
start flume
```

