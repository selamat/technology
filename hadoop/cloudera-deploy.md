# cloudera deploy(ubuntu)
## 1. download cloudera

#### cloudera manager下载链接（注意版本选择要与机器一致）：[http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cm_vd.html#cmvd_topic_1](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cm_vd.html#cmvd_topic_1)

#### CDH安装包下载链接：[http://archive.cloudera.com/cdh5/parcels/latest/](http://archive.cloudera.com/cdh5/parcels/latest/)
需要下载下述文件：
>* CDH-5.3.2-1.cdh5.3.2.p0.10-el5.parcel
>* CDH-5.3.2-1.cdh5.3.2.p0.10-el5.parcel.sha1
>* manifest.json

## 2. 配置各个节点HOST
>* 修改hostname
>* 修改host映射

## 3. 打通免密码登陆，root最好，其他也可

```shell
$ ssh-keygen -t rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys
```

## 4. 为所有节点安装JDK

#### （1）cloudera安装时会用root或sudo账号创建其他用户，所以JAVA_HOME配置时一定是全局，不能采用局部
#### （2）部署JDK时一定采用oracle jdk，不可以使用openjdk

## 5. 为master节点安装mysql服务

#### cloudera采用mysql存储集群配置
```shell
$ sudo apt-get install mysql-server
$ /etc/init.d/mysql start
$ mysqladmin -u root password 'xxxx'
$ create database hive DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
$ create database amon DEFAULT CHARSET utf8 COLLATE utf8_general_ci;

$ grant all privileges on *.* to 'root'@'{master-host}' identified by 'xxxx' with grant option;
$ flush privileges;
```

## 6. 关闭防火墙和SELinux

#### cloudera所需端口过多，安装时可以先关闭防火墙，安装好之后再根据需要配置防火墙

#### CentOS默认打开SELinux，ubuntu没有，不过‘检查关闭’还是必须的

## 7. 为全部节点配置NTP服务

#### cloudera中，agent节点时间都需要从master中同步，所以需要将master节点配置ntp广播服务（即便是有运维同学已经配置了各个节点时间服务，仍需要配置master节点的ntp广播服务，否则cloudera启动服务状态会出现clock offset异常）

```shell
## master配置
$ sudo apt-get install ntp
$ vim /etc/ntp.conf
$ << EOF
	# If you want to provide time to your local subnet, change the next line.
	# (Again,the address is an example only.)
	broadcast {master-host}
	EOF
$ sudo /etc/init.d/ntp start 

## agent配置crontab即可（其他方式不再赘述）
```

## 8. 安装cloudera manager

```shell
$ tar xzvf cloudera-manager*.tar.gz
$ sudo mv cm-5.3.2 /opt/
$ sudo mkdir /opt/cloudera/parcel-repo

## 将CDH-5.3.2-1.cdh5.3.2.p0.10-el5.parcel
## CDH-5.3.2-1.cdh5.3.2.p0.10-el5.parcel.sha1
## manifest.json拷贝到parcel-repo，注意一定要进行这个步骤，否则manager还会重新下载安装包
$ cd /opt/parcel-repo/
$ mv CDH-5.3.2-1.cdh5.3.2.p0.10-el5.parcel.sha1 CDH-5.3.2-1.cdh5.3.2.p0.10-el5.parcel.sha

## MySql的官网下载JDBC驱动，http://dev.mysql.com/downloads/connector/j/，解压后，找到mysql-connector-java-5.1.33-bin.jar，放到/opt/cm-5.3.2/share/cmf/lib/中，否则在测试链接mysql过程中会出现jdbc driver exception

$ cd /opt/cm-5.3.2

## 初始化cloudera数据库
$ /opt/cm-5.3.2/share/cmf/schema/scm_prepare_database.sh mysql cm -hlocalhost -uroot -pxxxx --scm-host localhost scm scm scm
$ sudo vim ./etc/cloudera-scm-agent/config.ini
$ << EOF
	# Hostname of the CM server.
	server_host={master-host}
	EOF
$ scp cm-5.3.2 {other}

## 启动cloudera manager（无需sudo启动，否则会报错）
$ /opt/cm-5.3.2/etc/init.d/cloudera-scm-server start

## 启动{other}节点上agent（必须以sudo启动，否则会出现文件无法写入情况）
$ sudo /opt/cm-5.3.2/etc/init.d/cloudera-scm-agent start
```
#### cloudera manager webui默认端口为7180，在配置文件中不可修改，只能更新数据库CONFIGS表中内容
```sql
USE cm;
INSERT INTO `CONFIGS` VALUES (4, NULL, 'http_port', '8180', NULL, NULL, 2, 2, NULL);
```

#### 接下来登录cloudera manager webui，用户名和密码默认均为admin，按照提示安装即可，安装过程不再赘述





