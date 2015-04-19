### HDFS

#### 1. HDFS基本命令：

>* 在HDFS中建立一个名为testdir的目录

```shell
bin/hadoop dfs -mkdir {dir}
```
>* 把本地文件large.zip拷贝到HDFS的根目录/user/hadoop/下，文件名为testfile.zip

```shell
$bin/hadoop dfs -put /home/hadoop/test.zip {dir}/{filename}
```
>* 查看现有文件

```shell
$bin/hadoop dfs -ls {dir}
```
>* 查看HDFS上文件

```shell
$hadoop dfs -cat {dir}/{filename}
```
>* 删除HDFS上文件

```shell
$hadoop dfs -rmr out
```

>* 从Hadoop分布式系统复制到本地文件系统

```shell
$bin/hadoop dfs -get {dir}/{filename} /home/hadoop/output
```

>* 显示目录中所有文件的大小，或者当紧指定一个文件时，显示该文件大小

```shell
$bin/hadoop dfs -du URI[URI ...]
```

#### 2. 管理与更新

>* 报告HDFS的基本统计信息

```shell
$hadoop dfsadmin -report
```
>* 退出安全模式

```shell
$hadoop dfsadmin -safemode leave
```
>* 进入安全模式

```shell
$hadoop dfsadmin -safemode enter
```
>* 负载均衡，平衡各个节点的分布

```shell
$bin/start-balancer.sh
```
>* 返回是否安全模式

```shell
$hadoop dfsadmin -safemode get
```
>* 查看块信息

```shell
$hadoop fsck / -files -blocks
```
>* 并行复制

```shell
$hadoop distcp -help hdfs://namenode1/foo hdfs://namenode2/bar
```
