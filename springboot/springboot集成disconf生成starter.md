[TOC]

## spring-boot-starter-disconf简介

disconf是一个分布式配置管理平台，通过和springboot2.1的整合成一个starter，无需xml繁琐的配置，即可快速使用disconf，相关代码已上传到github。

## 一、starter项目地址

[https://github.com/foolishboy66/spring-boot-starter-disconf.git](https://github.com/foolishboy66/spring-boot-starter-disconf.git)



## 二、示例代码地址

[https://github.com/foolishboy66/springboot-disconf-test.git](https://github.com/foolishboy66/springboot-disconf-test.git)



## 三、disconf-web环境搭建说明

### 1、参考[disconf官网](https://disconf.readthedocs.io/zh_CN/latest/install/src/02.html)说明安装disconf-web

### 2、本人安装的相关组件如下

系统为macos

- [x] jdk1.8

- [x] mysql-5.7.28

- [x] redis-5.0.6

- [x] tomcat-9.0.27

  `server.xml配置如下`

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!--
    Licensed to the Apache Software Foundation (ASF) under one or more
    contributor license agreements.  See the NOTICE file distributed with
    this work for additional information regarding copyright ownership.
    The ASF licenses this file to You under the Apache License, Version 2.0
    (the "License"); you may not use this file except in compliance with
    the License.  You may obtain a copy of the License at
  
        http://www.apache.org/licenses/LICENSE-2.0
  
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
  -->
  <!-- Note:  A "Server" is not itself a "Container", so you may not
       define subcomponents such as "Valves" at this level.
       Documentation at /docs/config/server.html
   -->
  <Server port="8005" shutdown="SHUTDOWN">
    <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
    <!-- Security listener. Documentation at /docs/config/listeners.html
    <Listener className="org.apache.catalina.security.SecurityListener" />
    -->
    <!--APR library loader. Documentation at /docs/apr.html -->
    <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
    <!-- Prevent memory leaks due to use of particular java/javax APIs-->
    <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
    <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
    <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
  
    <!-- Global JNDI resources
         Documentation at /docs/jndi-resources-howto.html
    -->
    <GlobalNamingResources>
      <!-- Editable user database that can also be used by
           UserDatabaseRealm to authenticate users
      -->
      <Resource name="UserDatabase" auth="Container"
                type="org.apache.catalina.UserDatabase"
                description="User database that can be updated and saved"
                factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
                pathname="conf/tomcat-users.xml" />
    </GlobalNamingResources>
  
    <!-- A "Service" is a collection of one or more "Connectors" that share
         a single "Container" Note:  A "Service" is not itself a "Container",
         so you may not define subcomponents such as "Valves" at this level.
         Documentation at /docs/config/service.html
     -->
    <Service name="Catalina">
  
      <!--The connectors can use a shared executor, you can define one or more named thread pools-->
      <!--
      <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
          maxThreads="150" minSpareThreads="4"/>
      -->
  
  
      <!-- A "Connector" represents an endpoint by which requests are received
           and responses are returned. Documentation at :
           Java HTTP Connector: /docs/config/http.html
           Java AJP  Connector: /docs/config/ajp.html
           APR (HTTP/AJP) Connector: /docs/apr.html
           Define a non-SSL/TLS HTTP/1.1 Connector on port 8080
      -->
      <Connector port="8080" protocol="HTTP/1.1"
                 connectionTimeout="20000"
                 redirectPort="8443" />
      <Connector port="8015" protocol="HTTP/1.1"
                 connectionTimeout="20000"
                 redirectPort="8543" />
      <!-- A "Connector" using the shared thread pool-->
      <!--
      <Connector executor="tomcatThreadPool"
                 port="8080" protocol="HTTP/1.1"
                 connectionTimeout="20000"
                 redirectPort="8443" />
      -->
      <!-- Define an SSL/TLS HTTP/1.1 Connector on port 8443
           This connector uses the NIO implementation. The default
           SSLImplementation will depend on the presence of the APR/native
           library and the useOpenSSL attribute of the
           AprLifecycleListener.
           Either JSSE or OpenSSL style configuration may be used regardless of
           the SSLImplementation selected. JSSE style configuration is used below.
      -->
      <!--
      <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
                 maxThreads="150" SSLEnabled="true">
          <SSLHostConfig>
              <Certificate certificateKeystoreFile="conf/localhost-rsa.jks"
                           type="RSA" />
          </SSLHostConfig>
      </Connector>
      -->
      <!-- Define an SSL/TLS HTTP/1.1 Connector on port 8443 with HTTP/2
           This connector uses the APR/native implementation which always uses
           OpenSSL for TLS.
           Either JSSE or OpenSSL style configuration may be used. OpenSSL style
           configuration is used below.
      -->
      <!--
      <Connector port="8443" protocol="org.apache.coyote.http11.Http11AprProtocol"
                 maxThreads="150" SSLEnabled="true" >
          <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
          <SSLHostConfig>
              <Certificate certificateKeyFile="conf/localhost-rsa-key.pem"
                           certificateFile="conf/localhost-rsa-cert.pem"
                           certificateChainFile="conf/localhost-rsa-chain.pem"
                           type="RSA" />
          </SSLHostConfig>
      </Connector>
      -->
  
      <!-- Define an AJP 1.3 Connector on port 8009 -->
      <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
  
  
      <!-- An Engine represents the entry point (within Catalina) that processes
           every request.  The Engine implementation for Tomcat stand alone
           analyzes the HTTP headers included with the request, and passes them
           on to the appropriate Host (virtual host).
           Documentation at /docs/config/engine.html -->
  
      <!-- You should set jvmRoute to support load-balancing via AJP ie :
      <Engine name="Catalina" defaultHost="localhost" jvmRoute="jvm1">
      -->
      <Engine name="Catalina" defaultHost="localhost">
  
        <!--For clustering, please take a look at documentation at:
            /docs/cluster-howto.html  (simple how to)
            /docs/config/cluster.html (reference documentation) -->
        <!--
        <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
        -->
  
        <!-- Use the LockOutRealm to prevent attempts to guess user passwords
             via a brute-force attack -->
        <Realm className="org.apache.catalina.realm.LockOutRealm">
          <!-- This Realm uses the UserDatabase configured in the global JNDI
               resources under the key "UserDatabase".  Any edits
               that are performed against this UserDatabase are immediately
               available for use by the Realm.  -->
          <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
                 resourceName="UserDatabase"/>
        </Realm>
  
        <Host name="localhost"  appBase="webapps"
              unpackWARs="true" autoDeploy="true">
  
          <!-- SingleSignOn valve, share authentication between web applications
               Documentation at: /docs/config/valve.html -->
          <!--
          <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
          -->
  
          <!-- Access log processes all example.
               Documentation at: /docs/config/valve.html
               Note: The pattern used is equivalent to using pattern="common" -->
          <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                 prefix="localhost_access_log" suffix=".txt"
                 pattern="%h %l %u %t &quot;%r&quot; %s %b" />
          <Context path="" docBase="/Users/foolishboy/tools/disconf/war"></Context>
  
        </Host>
      </Engine>
    </Service>
  </Server>
  ```

- [x] zookeeper-3.5.6

  #### `zoo.cfg配置如下`

  ```properties
  # The number of milliseconds of each tick
  tickTime=2000
  # The number of ticks that the initial 
  # synchronization phase can take
  initLimit=10
  # The number of ticks that can pass between 
  # sending a request and getting an acknowledgement
  syncLimit=5
  # the directory where the snapshot is stored.
  # do not use /tmp for storage, /tmp here is just 
  # example sakes.
  dataDir=/Users/foolishboy/tools/zookeeper-3.5.6/data
  dataLogDir=/Users/foolishboy/tools/zookeeper-3.5.6/logs
  # the port at which the clients will connect
  clientPort=2181
  # the maximum number of client connections.
  # increase this if you need to handle more clients
  #maxClientCnxns=60
  #
  # Be sure to read the maintenance section of the 
  # administrator guide before turning on autopurge.
  #
  # http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
  #
  # The number of snapshots to retain in dataDir
  #autopurge.snapRetainCount=3
  # Purge task interval in hours
  # Set to "0" to disable auto purge feature
  #autopurge.purgeInterval=1
  ```

- [x] nginx-1.7.13

  `nginx.conf配置如下`

  ```properties
  
  #user  nobody;
  worker_processes  1;
  
  #error_log  logs/error.log;
  #error_log  logs/error.log  notice;
  #error_log  logs/error.log  info;
  
  #pid        logs/nginx.pid;
  
  
  events {
      worker_connections  1024;
  }
  
  
  http {
      include       mime.types;
      default_type  application/octet-stream;
  
      #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
      #                  '$status $body_bytes_sent "$http_referer" '
      #                  '"$http_user_agent" "$http_x_forwarded_for"';
  
      #access_log  logs/access.log  main;
  
      sendfile        on;
      #tcp_nopush     on;
  
      #keepalive_timeout  0;
      keepalive_timeout  65;
  
      #gzip  on;
  
      upstream disconf {
          server localhost:8015;
      }
  
      server {
  
          listen   8081;
          server_name disconf.com;
          access_log /Users/foolishboy/tools/disconf/logs/access.log;
          error_log /Users/foolishboy/tools/disconf/logs/error.log;
  
          location / {
              root /Users/foolishboy/tools/disconf/war/html;
              if ($query_string) {
                  expires max;
              }
          }
  
          location ~ ^/(api|export) {
              proxy_pass_header Server;
              proxy_set_header Host $http_host;
              proxy_redirect off;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Scheme $scheme;
              proxy_pass http://disconf;
          }
      }
  }online-resources相关配置如下
  ```

### 3、online-resources相关文件配置如下

- [x] application.properties

  ```properties
  domain=disconf.com
  
  EMAIL_MONITOR_ON = true
  EMAIL_HOST = smtp.163.com
  EMAIL_HOST_PASSWORD = password
  EMAIL_HOST_USER = sender@163.com
  EMAIL_PORT = 25
  DEFAULT_FROM_EMAIL = disconf@163.com
  
  CHECK_CONSISTENCY_ON= true
  ```

- [x] jdbc-mysql.properties

  ```properties
  jdbc.driverClassName=com.mysql.jdbc.Driver
  
  jdbc.db_0.url=jdbc:mysql://127.0.0.1:3306/disconf?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&rewriteBatchedStatements=false
  jdbc.db_0.username=root
  jdbc.db_0.password=root
  
  jdbc.maxPoolSize=20
  jdbc.minPoolSize=10
  jdbc.initialPoolSize=10
  jdbc.idleConnectionTestPeriod=1200
  jdbc.maxIdleTime=3600
  ```

- [x] redis-config.properties

  ```properties
  redis.group1.retry.times=2
  
  redis.group1.client1.name=localhost
  redis.group1.client1.host=127.0.0.1
  redis.group1.client1.port=6379
  redis.group1.client1.timeout=5000
  redis.group1.client1.password=
  
  redis.group1.client2.name=localhost
  redis.group1.client2.host=127.0.0.1
  redis.group1.client2.port=6379
  redis.group1.client2.timeout=5000
  redis.group1.client2.password=
  
  redis.evictor.delayCheckSeconds=300
  redis.evictor.checkPeriodSeconds=30
  redis.evictor.failedTimesToBeTickOut=6
  
  ```

- [x] zoo.properties

  ```properties
  hosts=127.0.0.1:2181
  
  # zookeeper\u7684\u524D\u7F00\u8DEF\u5F84\u540D
  zookeeper_url_prefix=/disconf
  ```

  

