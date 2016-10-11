# 基础服务
![public-service](https://github.com/zhengfc/redis-cluster-monitor/blob/master/doc/public-service.png?raw=true)
## 1 缓存
### 1.1 cache-client
1. ```pom.xml```中引入```cache-client-1.6.1.jar```  
2. 在```properties```中添加```redis```服务器配置信息  
  ```
  # redis address:ip:port:pwd
  # cache_server_address=10.48.193.201:6669:test123
  cache_server_address=http://redismonitor/cache-monitor/ws/cache-server.htm
  cache_server_appCode=app123
  ```
2. 编写```spring-redis-cache.xml```文件  

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
  	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
  	xmlns:tx="http://www.springframework.org/schema/tx" xmlns:util="http://www.springframework.org/schema/util"
  	xmlns:hpcache="http://www.handpay.com.cn/schema/cache"
  	xsi:schemaLocation="
  			http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
  			http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
  			http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd
  			http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-2.5.xsd
  			http://www.handpay.com.cn/schema/cache http://www.handpay.com.cn/schema/cache/handpay-cache.xsd">

  	<!--redis常用缓存客户端定义-->
      <!--server ip <hpcache:local address="10.48.193.201:6379"/> handpay/handpay-->
      <hpcache:cache-executor appcode="${cache_server_appCode}" id="cacheExecutor">
          <hpcache:server-address>
          	<hpcache:http endpoint="${cache_server_address}" />
          </hpcache:server-address>
          <hpcache:server>
              <!-- ===================================================== -->
              <!--                                             DEFAULT   -->
              <!--  PROPERTY NAME                              VALUE     -->
              <!-- ..................................................... -->
              <!--  redis.database                              0        -->
              <!--  redis.timeout                               2000     -->
              <!--  redis.usePool                               true     -->
              <!--  jedis.pool.maxActive                        8        -->
              <!--  jedis.pool.maxIdle                          8        -->
              <!--  jedis.pool.maxWait                          -1       -->
              <!--  jedis.pool.minEvictableIdleTimeMillis       60000    -->
              <!--  jedis.pool.minIdle                          0        -->
              <!--  jedis.pool.numTestsPerEvictionRun           -1       -->
              <!--  jedis.pool.softMinEvictableIdleTimeMillis   -1       -->
              <!--  jedis.pool.testOnBorrow                     false    -->
              <!--  jedis.pool.testOnReturn                     false    -->
              <!--  jedis.pool.testWhileIdle                    true     -->
              <!--  jedis.pool.timeBetweenEvictionRunsMillis    30000    -->
              <!--  jedis.pool.whenExhaustedAction              1        -->
              <!-- ===================================================== -->
              <props>
                  <prop key="jedis.config.timeout">10000</prop>
                  <prop key="jedis.pool.maxActive">60</prop>
                  <prop key="jedis.pool.maxIdle">5</prop>
                  <prop key="jedis.pool.maxWait">10000</prop>
              </props>
          </hpcache:server>
      </hpcache:cache-executor>

  </beans>
  ```
3. ```spring applicationContext```中引入配置文件
```
<import resource="spring-redis-cache.xml"/>
```
4. 利用```cacheExcutor```操作Redis

### 1.2 Redis 3.0分片

## 2 分布式锁  
公司分布锁基于zk开发的，所以需要配置zk连接信息；具体步骤如下：  
1. ```pom.xml```中引入```locks-1.0.1.jar```
2. 在```properties```中添加```zk```服务器配置信息
  ```
  locks.zookeeper.connectServer=zookeeper1:2181,zookeeper2:2182,zookeeper3:2183
  locks.zookeeper.namespace = locks
  locks.zookeeper.connectionTimeout = 30000
  locks.zookeeper.sessionTimeout = 15000
  ```
3. 编写```spring-locks.xml```文件

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
  	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  	xmlns:context="http://www.springframework.org/schema/context"
  	xsi:schemaLocation="
  			http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
  			http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

  	<!-- XML方式，最简配置 -->
  	<bean id="distributedLockFactory"
  		class="com.handpay.framework.locks.zookeeper.ZookeeperLockFactory"
  		init-method="init" destroy-method="destroy">
  		<property name="namespace" value="${locks.zookeeper.namespace}" />
  		<property name="connectionTimeout" value="${locks.zookeeper.connectionTimeout}" />
  		<property name="sessionTimeout" value="${locks.zookeeper.sessionTimeout}" />
  		<property name="connectServer" value="${locks.zookeeper.connectServer}" />
  	</bean>
  </beans>
  ```
4. 利用```distributedLockFactory```构建相应的```lock```，操作```api```

## 3 搜索
### 3.1 Elasticsearch
### 3.2 SolrCloud

## 4 消息
### 4.1 HornetQ
### 4.2 disruptor

## 5 RPC
### 5.1 dubbo
公司的服务发现是基于阿里的dubbo，dubbo根据需要主要提供下面三种配置信息。
#### 5.1.1 dubbo-config  
用于配置服务发现注册中心地址及应用信息计算依赖关系，示例如下：  
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="
			http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
			http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息，用于计算依赖关系 -->
	<dubbo:application name="app123" />

	<dubbo:registry protocol="${dubbo.registry.protocol}"
		address="${dubbo.registry.address}" file="${user.home}/.dubbo-cache/service-app123-3${env.num}79"
		group="dubbo-${env_path}" check="false" />

	<dubbo:protocol name="${dubbo.protocol.name}" port="3${env.num}79" />

	<dubbo:consumer check="false" timeout="${dubbo.consumer.timeout}" />
</beans>
```
#### 5.1.2 dubbo-client    
作为消费者，消费服务信息，示例如下：  
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="
			http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
			http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

	<!-- 邮件发送服务 -->
	<dubbo:reference id="sender"
		interface="com.handpay.framework.mail.spec.MailSender" retries="0"
		check="false" />
</beans>
```
#### 5.1.3 dubbo-server  
作为生产者，提供服务，示例如下
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="
			http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
			http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

	<dubbo:service
		interface="com.handpay.framework.mail.spec.MailSender"
		ref="mailSender" version="1.0.0" />
	<bean id="mailSender"
		class="ccom.handpay.framework.mail.spec.MailSenderImpl">
		<property name="mailSenderService" ref="mailSenderService" />
	</bean>

</beans>
```

### 5.2 httpInvoker

## 6 定时任务
### 6.1 spring集成quartz
### 6.2 spring TaskExecutor和TaskScheduler

## 7 公共服务
### 7.1 短信服务
### 7.2 邮件服务
### 7.3 Otp服务
### 7.4 统一session
