# nginx+tomcat cluster

## 简介
### 
nginx+tomcat集群搭建，nginx通过设置每台tomcat权重将请求分发给各个tomcat节点，tomcat各个节点之间采用组播方式建立集群，tomcat集群各节点通过建立tcp链接来完成Session的拷贝。其中拷贝session方式有两种：同步复制和异步复制。同步复制是指：服务器对客户端响应必须等待session在各个tomcat节点拷贝完成之后。异步复制是指：服务器对于客户端响应无需等待session拷贝完成，这种方式更加高效，但同步模式可靠性更高。

## 1. 下载tomcat压缩包并解压

```
tar -zxvf apache-tomcat-6.0.35.tar.gz -C /home/trssmas/apps
```

## 2. 修改server.xml配置文件 ``` /home/trssmas/apps/apache-tomcat-6.0.35/conf ```

```
 同一台机器启动多tomcat实例时需要先修改tomcat监听端口，端口不冲突即可。
 (1)在server.xml中找到Server标签，将8005端口改为9005.
 (2)在server.xml中找到Connector标签，修改tomcat监听的8080改为9080端口。
 (3)在server.xml中找到Connector标签，修改tomcat AJP 8009改为9009端口。
 (4)找到<Engine name="Catalina" defaultHost="localhost" jvmRoute="jvm1">，并将其注释去掉；将原来的<Engine name="Catalina" defaultHost="localhost">注释掉。（jvmRoute值可以任意设置，只要各个tomcat实例不同即可）
```

添加tomcat cluster配置 ```（其中，Receiver中port如果在相同机器上则两个tomcat实例需要配置唯一端口）  ```
```

 <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
                 channelSendOptions="8">

          <Manager className="org.apache.catalina.ha.session.BackupManager"
                   expireSessionsOnShutdown="false"
                   notifyListenersOnReplication="true"
                   mapSendOptions="6"/>      
          <Channel className="org.apache.catalina.tribes.group.GroupChannel">
            <Membership className="org.apache.catalina.tribes.membership.McastService"
                        address="228.0.0.4"
                        port="45564"
                        frequency="500"
                        dropTime="3000"/>
            <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
                      address="auto"
                      port="5000"
                      selectorTimeout="100"
                      maxThreads="6"/>

            <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
              <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
            </Sender>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatch15Interceptor"/>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.ThroughputInterceptor"/>
          </Channel>

          <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"
                 filter=".*\.gif;.*\.js;.*\.jpg;.*\.png;.*\.htm;.*\.html;.*\.css;.*\.txt;"/>

          <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>
        </Cluster>
```

## 3. 修改应用web.xml文件并部署应用

```
在web.xml web-app标签中添加<distributable/>标签（含义为：告诉tomcat此应用部署在分布式环境中）
将应用打包后拷贝至各个tomcat节点webapp文件夹下
```

## 4. 启动tomcat

```
bin/startup.sh
```

## 5. server.xml集群配置参数说明

### channelSendOptions
用来控制session复制模式，默认值是8，为异步模式；4是同步模式。在异步模式下，可以通过加上拷贝确认（Acknowledge， channelSendOptions为10）来提高可靠性。


### Manager标签 
Manager用来在节点间拷贝Session，默认使用DeltaManager，DeltaManager采用的一种all-to-all的工作方式，即集群中的节点会把Session数据向所有其他节点拷贝，而不管其他节点是否部署了当前应用。当集群中的节点数量很多并且部署着不同应用时，可以使用BackupManager，BackManager仅向部署了当前应用的节点拷贝Session。

### value标签
Valve用于在节点向客户端响应前进行检测或进行某些操作，ReplicationValve就是用于用于检测当前的响应是否涉及Session数据的更新，如果是则启动Session拷贝操作，filter用于过滤请求，如客户端对图片，css，js的请求就不会涉及Session，因此不需检测，默认状态下不进行过滤，监测所有的响应。


### Channel标签
Channel负责对tomcat集群的IO层进行配置。Membership用于发现集群中的其他节点，这里的address用的是组播地址（Multicast address），使用同一个组播地址和端口的多个节点同属一个子集群，因此通过自定义组播地址和端口就可将一个大的tomcat集群分成多个子集群。Receiver用于各个节点接收其他节点发送的数据，在默认配置下tomcat会从4000-4100间依次选取一个可用的端口进行接收，自定义配置时，如果多个tomcat节点在一台物理服务器上注意要使用不同的端口。Sender用于向其他节点发送数据，具体实现通过Transport配置。PooledParallelSender是从tcp连接池中获取连接，可以实现并行发送，即集群中的多个节点可以同时向其他所有节点发送数据而互不影响。Interceptor有点类似于上面解释的Valve，起到一个阀门的作用，在数据到达目的节点前进行检测或其他操作，如TcpFailureDetector用于检测在数据的传输过程中是否发生了tcp错误。


## 6. nginx反向代理
#### (1) 此处只写简单的nginx配置，并无调优等。
#### (2) 修改nginx配置文件nginx.conf，在http标签下添加如下代码，weight为权重

```
upstream tomcatserver {
        server ip:port weight=1;
        server ip:port weight=1;
}
```
#### (3) 在server标签中找到location标签，在location标签中添加
```
proxy_pass http://tomcatserver;
```

#### (4) 启动nginx

## 7. 注意问题
#### session过期问题：对于因超时引起的Session失效tomcat1无需通知tomcat2，因为tomcat2同样知道session已经超时。因此对于tomcat集群有一点非常重要，所有节点的操作系统时间必须一致！不然会出现某个节点Session已过期而在另一节点此Session仍处于活动状态的现象。


