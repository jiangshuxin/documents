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
1. ```pom.xml```中引入
```
    <dependency>
        <groupId>com.handpay</groupId>
        <artifactId>locks</artifactId>
        <version>1.0.1</version>
    </dependency>
```
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
#### 4.1.1 依赖声明
```
    <dependency>
        <groupId>com.handpay</groupId>
        <artifactId>core-common</artifactId>
        <version>1.0.0</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>com.handpay</groupId>
        <artifactId>core-interface</artifactId>
        <version>1.0.0</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.hornetq</groupId>
        <artifactId>hornetq-core-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.hornetq</groupId>
        <artifactId>hornetq-jms-client</artifactId>
    </dependency>
```
如需指定版本则使用```<version>2.2.5.FINAL</version>```
#### 4.4.2 配置文件说明
由于基于Spring-JMS开发,请严格按照**原生ConnectionFactory**->**缓存ConnectionFactory**->**JmsTemplate**三部曲进行配置。
```
<bean name="mq.targetConnectionFactory"
		  class="com.handpay.core.common.spring.hornetq.DirectHornetQCFFactoryBean">
		<property name="address" value="${hornetq.server.address}"></property>
		<property name="params">
			<map>
				<entry key="hornetq.client.retryInterval" value="2000" />
				<entry key="hornetq.client.retryIntervalMultiplier" value="1" />
				<entry key="hornetq.client.maxRetryInterval" value="60000" />

				<entry key="hornetq.client.reconnectAttempts" value="5" />
				<entry key="hornetq.client.failoverOnInitialConnection" value="true" />
				<entry key="hornetq.client.connectionTTL" value="30000" />
				<entry key="hornetq.client.clientFailureCheckPeriod" value="30000" />
			</map>
		</property>
	</bean>

	<bean id="mq.connectionFactory"
		  class="org.springframework.jms.connection.CachingConnectionFactory">
		<property name="targetConnectionFactory" ref="mq.targetConnectionFactory" />
		<property name="sessionCacheSize" value="10" />
		<property name="cacheProducers" value="false" />
	</bean>

	<bean id="mq.core_fund_monitor.jmsTemplate" class="com.handpay.core.mq.CoreRetryJmsTemplate">
		<property name="connectionFactory" ref="mq.connectionFactory" />
		<property name="defaultDestinationName" value="core_fund_monitor"></property>
	</bean>
```
消费端配置请参考:
```
<bean id="mq.listener.coreOrderDelivery"
		  class="org.springframework.jms.listener.DefaultMessageListenerContainer">
		<property name="connectionFactory" ref="mq.connectionFactory" />
		<property name="concurrency" value="1-5" />
		<property name="messageListener" ref="mq.coreOrder.delivery.CoreOrderMessageListener" />
		<property name="destinationName" value="core_order" />
	</bean>

<bean id="mq.coreOrder.delivery.CoreOrderMessageListener" class="com.handpay.core.coreorder.kernel.delivery.CoreOrderMessageListener"/>
```
注意:类com.handpay.core.coreorder.kernel.delivery.CoreOrderMessageListener需实现javax.jms.MessageListener接口
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
