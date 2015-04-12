# Hadoop 2.2.0 集群部署（以CentOS 6.4为例）

## 1.创建用户和用户组

## 2.安装JDK，配置JAVA_HOME

## 3.修改/etc/hosts文件

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.9.200.106 cluster-01 cluster-01.domain
192.9.200.107 cluster-02 cluster-02.domain
192.9.200.108 cluster-03 cluster-03.domain
```

## 4.SSH免密码
（注意不要用root）
### a).需要两个服务：ssh和rsync
查询方法：

```
rpm –qa | grep openssh  
rpm –qa | grep rsync
```

若不存在通过sudo yum install xxx 进行安装

### b).在cluster-01(master)上生成公钥私钥

```
ssh-keygen –t rsa –P ''
```

### c).将公钥写入信任文件中

```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys  
```

### d).修改权限

```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys  
```

### e).配置sshd服务 ```sudo vim /etc/ssh/sshd_config```

```
RSAAuthentication yes # 启用 RSA 认证  
PubkeyAuthentication yes # 启用公钥私钥配对认证方式  
AuthorizedKeysFile .ssh/authorized_keys # 公钥文件路径（和上面生成的文件同） 
```

修改后重启生效 ```sudo service sshd restart ```

### f).本地验证免登录

```
ssh localhost
```

### g).使用scp，将master节点的id_rsa.pub拷贝到slave节点后将该文件的内容追加到slave节点的authorized_keys文件，重启slave节点的sshd服务

## 5. 下载Hadoop 2.2.0 二进制包并解压

## 6. 创建目录

   * 在每个节点上创建数据存储目录/home/hadoop/apps/hadoop-2.2.0/hdfs，用来存放集群数据
   * 在主节点m1上创建目录/home/hadoop/apps/hadoop-2.2.0/hdfs/name，用来存放文件系统元数据
   * 在每个从节点上创建目录/home/hadoop/apps/hadoop-2.2.0/hdfs/data，用来存放真正的数据
   * 所有节点上的日志目录为/home/hadoop/apps/hadoop-2.2.0/logs
   * 所有节点上的临时目录为/home/hadoop/apps/hadoop-2.2.0/tmp

## 7. 修改系统配置

```
export HADOOP_HOME=/home/hadoop/apps/hadoop-2.2.0
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

## 8. 修改Hadoop配置文件 ``` /home/hadoop/apps/etc/hadoop ```

  * core-site.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>

        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://cluster-01:9000</value>
                <final>true</final>
        </property>

        <property>
                <name>hadoop.tmp.dir</name>
                <value>/home/hadoop/apps/hadoop-2.2.0/tmp</value>
        </property>

        <property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>

</configuration>
```
  * hdfs-site.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>

        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:///home/hadoop/apps/hadoop-2.2.0/hdfs/name</value>
        </property>

        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:///home/hadoop/apps/hadoop-2.2.0/hdfs/data</value>
        </property>

        <property>
                <name>dfs.permissions</name>
                <value>false</value>
        </property>

        <property>
                <name>dfs.webhdfs.enabled</name>
                <value>true</value>
        </property>
</configuration>
```

  * hadoop-env.sh、yarn-env.sh、mapred-env.sh 加入 JAVA_HOME

## 启停
  * HDFS

```
# start
./start-dfs.sh
# stop
./stop-dfs.sh
```

## 监控（ganglia）
http://118.192.103.2/ganglia/

部署

 * 给每个节点配置源

```
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
sudo rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm
```

 * master节点

```
yum install ganglia ganglia-devel ganglia-gmetad ganglia-gmond ganglia-web ganglia-gmond-python
```

 * slave节点

```
yum install ganglia ganglia-gmond
```

* 配置
每个节点

```
vim /etc/ganglia/gmond.conf
cluster {
  name = "hadoop"    //只需要更改这一行，设置为gmetad中data_source指定的名称即可
  owner = " unspecified"
  latlong = "unspecified"
  url = "unspecified"
}
```

 * 主节点

```
vim /etc/ganglia/gmetad.conf
data_source "hadoop" cluster-01 cluster-02 cluster-03

vim /etc/httpd/conf.d/ganglia.conf
Alias /ganglia /usr/share/ganglia

<Location /ganglia>
Order deny,allow
# Deny from all
# Allow from 127.0.0.1
# Allow from ::1
Allow from all
# Allow from .example.com
</Location>
```

 * 启动
master

```
sudo service gmond start
sudo service httpd start
sudo service gmetad start
```

slave

```
sudo service gmond start
```

## 特殊说明
## Hadoop 2.2 默认二进制包为32位版，启动时会报如下警告

```
WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
```

:exclamation: 重新编译过程如下

* 1.下载Hadoop 2.2.0 源码包，并解压

```
$ wget http://mirrors.hust.edu.cn/apache/hadoop/common/hadoop-2.2.0/hadoop-2.2.0-src.tar.gz
$ tar zxf hadoop-2.2.0-src.tar.gz
```

* 2.安装下面的软件

```
$ sudo yum install lzo-devel  zlib-devel  gcc autoconf automake libtool   ncurses-devel openssl-deve cmake openssl-devel
```

* 3.安装Maven（Hadoop 2.2 只能用Maven 3.0.5 否则会有兼容性问题）

```
$ wget http://mirror.esocc.com/apache/maven/maven-3/3.0.5/binaries/apache-maven-3.0.5-bin.tar.gz
$ sudo tar zxf apache-maven-3.0.5-bin.tar.gz -C /home/hadoop/apps
$ sudo vim /etc/profile
export MAVEN_HOME=/opt/apache-maven-3.0.5
export PATH=$PATH:$MAVEN_HOME/bin
```

* 4.安装Ant

```
$ wget http://mirrors.koehn.com/apache//ant/binaries/apache-ant-1.9.3-bin.tar.gz
$ sudo tar zxf apache-ant-1.9.3-bin.tar.gz -C /home/hadoop/apps
$ sudo vim /etc/profile
export ANT_HOME=/opt/apache-ant-1.9.3
export PATH=$PATH:$ANT_HOME/bin
```

* 5.安装Findbugs

```
$ wget http://prdownloads.sourceforge.net/findbugs/findbugs-2.0.3.tar.gz?download
$ sudo tar zxf findbugs-2.0.3.tar.gz -C /home/hadoop/apps
$ sudo vim /etc/profile
export FINDBUGS_HOME=/opt/findbugs-2.0.3
export PATH=$PATH:$FINDBUGS_HOME/bin
```

* 6.安装protobuf（需要gcc-c++， ```yum install gcc-c++``` ）

```
$ wget https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz
$ tar zxf protobuf-2.5.0.tar.gz
$ cd protobuf-2.5.0
$ ./configure
$ make
$ sudo make install
```

* 7.Hadoop2.2源码有一个bug，需要打一个patch

```
Index: hadoop-common-project/hadoop-auth/pom.xml
--- hadoop-common-project/hadoop-auth/pom.xml   (revision 1543124)
+++ hadoop-common-project/hadoop-auth/pom.xml   (working copy)@@ -54,6 +54,11 @@     
</dependency>     
      <dependency>       
           <groupId>org.mortbay.jetty</groupId>
+         <artifactId>jetty-util</artifactId>
+         <scope>test</scope>
+    </dependency>
+    <dependency>
+         <groupId>org.mortbay.jetty</groupId>
           <artifactId>jetty</artifactId>
           <scope>test</scope>
     </dependency>cd 
```

* 8.编译 Hadoop

```
cd hadoop-2.2.0-src
mvn package -DskipTests -Pdist,native -Dtar
```

* 9.替换32位的native库

```
rm -rf ~/local/opt/hadoop-2.2.0/lib/native
cp ./hadoop-dist/target/hadoop-2.2.0/lib/native ~/local/opt/hadoop-2.2.0/lib/
```


# Hadoop 2.5.1 集群部署（以CentOS 6.5为例）
## 8. 修改Hadoop配置文件 ``` {HADOOP_HOME}/etc/hadoop ```

  * core-site.xml

```
configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://hadoop-master:9010</value>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>io.file.buffer.size</name>
    <value>131072</value>
   </property>
   <property>
     <name>hadoop.tmp.dir</name>
     <value>/home/zhanghu/hadoop/hadoop-2.5.1/tmp</value>
   </property>
   <property>
     <name>hadoop.proxyuser.hduser.hosts</name>
     <value>*</value>
   </property>
   <property>
     <name>hadoop.proxyuser.hduser.groups</name>
     <value>*</value>
   </property>
</configuration>
```
  * hdfs-site.xml

```
<configuration>
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop-master:9011</value>
  </property>
  <property>
    <name>dfs.namenode.checkpoint.dir</name>
    <value>/home/zhanghu/hadoop/hadoop-2.5.1/dfs/namesecondary</value>
  </property>
  <property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>3</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/home/zhanghu/hadoop/hadoop-2.5.1/dfs/name</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/home/zhanghu/hadoop/hadoop-2.5.1/dfs/data</value>
  </property>
</configuration>
```
* yarn-site.xml

```
<configuration>

<!-- Site specific YARN configuration properties -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop-master</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>${yarn.resourcemanager.hostname}:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>${yarn.resourcemanager.hostname}:8031</value>
  </property>
 <property>
    <name>yarn.resourcemanager.address</name>
    <value>${yarn.resourcemanager.hostname}:8032</value>
  </property>
  <property>
    <name>yarn.resourcemanager.admin.address</name>
    <value>${yarn.resourcemanager.hostname}:8033</value>
  </property>
  <property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>${yarn.resourcemanager.hostname}:8088</value>
  </property>
</configuration>
```



