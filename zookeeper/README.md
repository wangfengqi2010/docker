## zookeeper

### 环境配置
* 镜像
~~~
docker pull zookeeper:3.5.5
~~~
* zkEnv.sh
~~~
SERVER_JVMFLAGS="-Dzookeeper.serverCnxnFactory=org.apache.zookeeper.server.NettyServerCnxnFactory \
-Dzookeeper.ssl.keyStore.location=$ZOOCFGDIR/certs/zaozuo-zk-store.jks \
-Dzookeeper.ssl.keyStore.password=zaozuo \
-Dzookeeper.ssl.trustStore.location=$ZOOCFGDIR/certs/zaozuo-zk-trust.jks \
-Dzookeeper.ssl.trustStore.password=zaozuo \
-Djdk.tls.rejectClientInitiatedRenegotiation=true"

CLIENT_JVMFLAGS="-Dzookeeper.clientCnxnSocket=org.apache.zookeeper.ClientCnxnSocketNetty \
-Dzookeeper.ssl.keyStore.location=$ZOOCFGDIR/certs/zaozuo-zk-store.jks \
-Dzookeeper.ssl.keyStore.password=zaozuo \
-Dzookeeper.ssl.trustStore.location=$ZOOCFGDIR/certs/zaozuo-zk-trust.jks \
-Dzookeeper.ssl.trustStore.password=zaozuo \
-Dzookeeper.client.secure=true"
~~~
* zoo.cfg
~~~
clientPort=2181
secureClientPort=12181
admin.enableServer=false
server.1=zk-node-01:2888:3888:participant
server.2=zk-node-02:2888:3888:participant
server.3=zk-node-03:2888:3888:participant
~~~
* jks证书
~~~
store.jks
trust.jks
~~~
* 数据日志目录
~~~
zookeeper/nodes/node-01/data
zookeeper/nodes/node-01/datalog
zookeeper/nodes/node-01/logs
zookeeper/nodes/node-02/data
zookeeper/nodes/node-02/datalog
zookeeper/nodes/node-02/logs
zookeeper/nodes/node-03/data
zookeeper/nodes/node-03/datalog
zookeeper/nodes/node-03/logs
~~~
* docker-compose.yml
~~~
注意端口映射和主机名
~~~
### 集群运行
~~~
docker-compose up -d
~~~
### 客户端连接
* zkCli.sh
~~~
docker exec 连接容器连接验证
~~~
* Curator

    * pom.xml
    ~~~
    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
        <dependency>
          <groupId>org.apache.curator</groupId>
          <artifactId>curator-framework</artifactId>
          <version>4.2.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
        <dependency>
          <groupId>org.apache.curator</groupId>
          <artifactId>curator-recipes</artifactId>
          <version>4.2.0</version>
          <exclusions>
            <exclusion>
              <groupId>org.apache.zookeeper</groupId>
              <artifactId>zookeeper</artifactId>
            </exclusion>
          </exclusions>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
        <dependency>
          <groupId>org.apache.zookeeper</groupId>
          <artifactId>zookeeper</artifactId>
          <version>3.5.5</version>
        </dependency>
      </dependencies>
    ~~~
  * Main.java
  ~~~
  package com.zaozuo.zweb;
  
  import java.util.concurrent.TimeUnit;
  import org.apache.curator.RetryPolicy;
  import org.apache.curator.framework.CuratorFramework;
  import org.apache.curator.framework.CuratorFrameworkFactory;
  import org.apache.curator.retry.RetryOneTime;
  
  /**
   * @author fengqi
   */
  public class Main {
  
      public static void main(String[] args) throws Exception {
          CuratorFramework curatorFramework = getCuratorFramework();
          String s = curatorFramework.create().forPath("/hello2");
          System.out.println(s);
      }
  
      private static CuratorFramework getCuratorFramework() throws InterruptedException {
          String zkHost = "localhost:12181";
          int connectionTimeoutMs = 3000;
          int sessionTimeoutMs = 5000;
          String chRoot = "lock";
          RetryPolicy retryPolicy = new RetryOneTime(5000);
          CuratorFramework framework =
              CuratorFrameworkFactory.builder()
                  .connectString(zkHost)
                  .sessionTimeoutMs(sessionTimeoutMs)
                  .connectionTimeoutMs(connectionTimeoutMs)
                  .retryPolicy(retryPolicy)
                  .namespace(chRoot)
                  .build();
          framework.start();
          framework.blockUntilConnected(connectionTimeoutMs, TimeUnit.MILLISECONDS);
          return framework;
      }
  }
  
  JVMARGS
  -Dzookeeper.clientCnxnSocket=org.apache.zookeeper.ClientCnxnSocketNetty
  -Dzookeeper.ssl.keyStore.location=D:\code\docker\zookeeper\conf\certs\zaozuo-zk-store.jks
  -Dzookeeper.ssl.keyStore.password=zaozuo
  -Dzookeeper.ssl.trustStore.location=D:\code\docker\zookeeper\conf\certs\zaozuo-zk-trust.jks
  -Dzookeeper.ssl.trustStore.password=zaozuo
  -Dzookeeper.client.secure=true
  ~~~
