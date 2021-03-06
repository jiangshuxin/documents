# 基础服务
![public-service](https://github.com/zhengfc/redis-cluster-monitor/blob/master/doc/public-service.png?raw=true)

## 1 缓存
### 1.1 cache-client
#### 1.1.1 ```pom.xml```中引入```cache-client-1.6.1.jar```  
#### 1.1.2 在```properties```中添加```redis```服务器配置信息  

  ```
  # redis address:ip:port:pwd
  # cache_server_address=10.48.193.201:6669:test123
  cache_server_address=http://redismonitor/cache-monitor/ws/cache-server.htm
  cache_server_appCode=app123
  ```
  
#### 1.1.3 编写```spring-redis-cache.xml```文件  

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

#### 1.1.4 ```spring applicationContext```中引入配置文件

  ```
  <import resource="spring-redis-cache.xml"/>
  ```

#### 1.1.5 利用```cacheExcutor```操作```Redis```

### 1.2 Redis 3.0集群  
#### 1.2.1 依赖声明  
```
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-redis</artifactId>
  <version>1.7.4.RELEASE</version>
</dependency> 
```  
#### 1.2.2 配置文件说明  
详细指导请参考: https://github.com/spring-projects/spring-data-redis  
```
<bean id="redisClusterConfiguration" class="org.springframework.data.redis.connection.RedisClusterConfiguration">
    <property name="maxRedirects" value="${redis.maxRedirects}"></property>
    <property name="clusterNodes">
        <set>
            <bean class="org.springframework.data.redis.connection.RedisNode">
                <constructor-arg name="host" value="${redis.host1}"/>
                <constructor-arg name="port" value="${redis.port1}"/>
            </bean>
            <bean class="org.springframework.data.redis.connection.RedisNode">
                <constructor-arg name="host" value="${redis.host2}"/>
                <constructor-arg name="port" value="${redis.port2}"/>
            </bean>
            <bean class="org.springframework.data.redis.connection.RedisNode">
                <constructor-arg name="host" value="${redis.host3}"/>
                <constructor-arg name="port" value="${redis.port3}"/>
            </bean>
        </set>
    </property>
</bean>

<bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <constructor-arg name="clusterConfig" ref="redisClusterConfiguration"/>
</bean>

<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <property name="connectionFactory" ref="jedisConnectionFactory"/>
</bean>
```
#### 1.2.3 操作
```
boolean hasKey = redisTemplate.hasKey("testKey");
ClusterOperations clusterOps = redisTemplate.opsForCluster();
```

## 2 分布式锁  
公司分布锁基于```zookeeper```开发的，所以需要配置```zookeeper```连接信息；具体步骤如下：  
  
### 2.1 ```pom.xml```中引入

  ```
  <dependency>
	  <groupId>com.handpay</groupId>
	  <artifactId>locks</artifactId>
	  <version>1.0.1</version>
  </dependency>
  ```

### 2.2 在```properties```中添加```zookeeper```服务器配置信息

  ```
  locks.zookeeper.connectServer=zookeeper1:2181,zookeeper2:2182,zookeeper3:2183
  locks.zookeeper.namespace = locks
  locks.zookeeper.connectionTimeout = 30000
  locks.zookeeper.sessionTimeout = 15000
  ```
  
### 2.3 编写```spring-locks.xml```文件

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
  	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  	xmlns:context="http://www.springframework.org/schema/context"
  	xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
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
  
### 2.4 利用```distributedLockFactory```构建相应的```lock```，操作```api```

## 3 搜索
### 3.1 Elasticsearch
详细指导请参考: https://github.com/spring-projects/spring-data-elasticsearch
#### 3.1.1 依赖声明
```
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-elasticsearch</artifactId>
  <version>2.0.2.RELEASE</version>
</dependency>
```
#### 3.1.2 配置文件说明
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
       xsi:schemaLocation="http://www.springframework.org/schema/data/elasticsearch http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <elasticsearch:repositories base-package="com.handpay.xxx"/>

    <elasticsearch:transport-client id="client" cluster-nodes="ip:9300,ip:9300" cluster-name="elasticsearch" />

    <bean name="elasticsearchTemplate" class="org.springframework.data.elasticsearch.core.ElasticsearchTemplate">
        <constructor-arg name="client" ref="client"/>
    </bean>

</beans>
```
### 3.2 SolrCloud
详细指导请参考: https://github.com/spring-projects/spring-data-solr
#### 3.2.1 依赖声明
```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-solr</artifactId>
  <version>1.4.1.RELEASE</version>
</dependency>
```
#### 3.2.2 配置文件说明
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:solr="http://www.springframework.org/schema/data/solr"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/data/solr
    http://www.springframework.org/schema/data/solr/spring-solr.xsd">

  <solr:solr-client id="solrClient" url="http://locahost:8983/solr" />
  
  <bean id="solrConverter" class="org.springframework.data.solr.core.convert.MappingSolrConverter">
  	<constructor-arg>
  		<bean class="org.springframework.data.solr.core.mapping.SimpleSolrMappingContext" />
  	</constructor-arg>
  	<property name="customConversions" ref="customConversions" />
  </bean>
  
  <bean id="customConversions" class="org.springframework.data.solr.core.convert.CustomConversions">
  	<constructor-arg>
  		<list>
  			<bean class="com.acme.MyBeanToSolrInputDocumentConverter" />
  		</list>
  	</constructor-arg>
  </bean>
  
  <bean id="solrTemplate" class="org.springframework.data.solr.core.SolrTemplate">
  	<constructor-arg ref="solrClient" />
  	<property name="solrConverter" ref="solrConverter" />
  </bean>
</beans>
```
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

#### 4.1.2 配置文件说明
由于基于```Spring-JMS```开发,请严格按照**原生ConnectionFactory**->**缓存ConnectionFactory**->**JmsTemplate**三部曲进行配置。

```
<bean name="mq.targetConnectionFactory" class="com.handpay.core.common.spring.hornetq.DirectHornetQCFFactoryBean">
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
<bean id="mq.listener.coreOrderDelivery" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
	<property name="connectionFactory" ref="mq.connectionFactory" />
	<property name="concurrency" value="1-5" />
	<property name="messageListener" ref="mq.coreOrder.delivery.CoreOrderMessageListener" />
	<property name="destinationName" value="core_order" />
</bean>

<bean id="mq.coreOrder.delivery.CoreOrderMessageListener" class="com.handpay.core.coreorder.kernel.delivery.CoreOrderMessageListener"/>
```

注意:类```com.handpay.core.coreorder.kernel.delivery.CoreOrderMessageListener```需实现```javax.jms.MessageListener```接口
### 4.2 disruptor
#### 4.2.1 依赖声明
```
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.2.0</version>
</dependency>
```
#### 4.2.2 基本用法

1.定义事件
```
public class LongEvent
{
    private long value;

    public void set(long value)
    {
        this.value = value;
    }
}
```


2.定义事件工厂
```
import com.lmax.disruptor.EventFactory;

public class LongEventFactory implements EventFactory<LongEvent>
{
    public LongEvent newInstance()
    {
        return new LongEvent();
    }
}
```

3.定义事件处理
```
import com.lmax.disruptor.EventHandler;

public class LongEventHandler implements EventHandler<LongEvent>
{
    public void onEvent(LongEvent event, long sequence, boolean endOfBatch)
    {
        System.out.println("Event: " + event);
    }
}
```

4.启动disruptor
```
ExecutorService executor = Executors.newCachedThreadPool();

WaitStrategy BLOCKING_WAIT = new BlockingWaitStrategy();
WaitStrategy SLEEPING_WAIT = new SleepingWaitStrategy();
WaitStrategy YIELDING_WAIT = new YieldingWaitStrategy();

EventFactory<LongEvent> eventFactory = new LongEventFactory();
ExecutorService executor = Executors.newSingleThreadExecutor();
int ringBufferSize = 1024 * 1024; // RingBuffer 大小，必须是 2 的 N 次方；       
Disruptor<LongEvent> disruptor = new Disruptor<LongEvent>(eventFactory,
                ringBufferSize, executor, ProducerType.SINGLE,
                new YieldingWaitStrategy());
       
EventHandler<LongEvent> eventHandler = new LongEventHandler();
disruptor.handleEventsWith(eventHandler);
       
disruptor.start();
```

5.发布事件
```
//发布事件；
RingBuffer<LongEvent> ringBuffer = disruptor.getRingBuffer();
long sequence = ringBuffer.next();//请求下一个事件序号；
   
try {
    LongEvent event = ringBuffer.get(sequence);//获取该序号对应的事件对象；
    long data = getEventData();//获取要通过事件传递的业务数据；    
    event.set(data);
} finally{
    ringBuffer.publish(sequence);//发布事件；
}
```
## 5 RPC
### 5.1 dubbo
公司的服务发现是基于阿里的```dubbo```，```dubbo```根据需要主要提供下面三种配置信息。
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
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
		http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

	<!-- 邮件发送服务 -->
	<dubbo:reference id="sender" interface="com.handpay.framework.mail.spec.MailSender" retries="0" check="false" />
</beans>
```

#### 5.1.3 dubbo-server  
作为生产者，提供服务，示例如下

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd 
		http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

	<dubbo:service interface="com.handpay.framework.mail.spec.MailSender" ref="mailSender" version="1.0.0" />
	<bean id="mailSender" class="ccom.handpay.framework.mail.spec.MailSenderImpl">
		<property name="mailSenderService" ref="mailSenderService" />
	</bean>
</beans>
```

### 5.2 httpInvoker
#### 5.2.1 httpInvoker依赖
```
<dependency>
  <groupId>com.handpay</groupId>
  <artifactId>hpInterface4.0</artifactId>
  <version>1.1.0</version>
</dependency>
```

#### 5.2.2 httpInvoker Server
Server端```spring```配置文件如下:

```
<bean id="hisUrlMappingAnno" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
</bean>

<!-- Spring 容器后处理器，用于解释http invoker 注释声明 -->
<bean class="com.handpay.core.httpinvoker.HttpInvokerPostProcessor">
	<property name="httpInvokerHandleMapping">
		<ref bean="hisUrlMappingAnno" />
	</property>
	<property name="onlyClientWired">
		<value>false</value>
	</property>
	<property name="exportedServiceUrlsInfo">
		<map>
			<entry key="auth.authService" value="/authService.his" />
			<entry key="auth.authAdminService" value="/authAdminService.his" />
		</map>
	</property>
</bean>
```
对应的```Java```代码如下:
```
@HttpInvokerService(exportedRelativeUrl = "<auth.authService>", serviceInterfaceClass = IAuthService.class)
public class AuthServiceImpl implements IAuthService{...}
```
#### 5.2.3 httpInvoker Client
Client端```spring```配置文件如下:

```
<bean class="com.handpay.core.httpinvoker.HttpInvokerPostProcessor">
    <property name="clientUrlsInfo">
        <map>
            <entry key="auth.authService">
                <value>${auth.service.address}/authService.his</value>
            </entry>
            <entry key="auth.authAdminService">
                <value>${auth.service.address}/authAdminService.his</value>
            </entry>
        </map>
    </property>		
</bean>
```

注意,```auth.service.address```为服务对应的地址,一般写在属性文件中。
对应的```Java```代码如下:  

```
public class LoginAction extends BaseAction {

	private IAuthService authService;
	
	@HttpInvokerClientWired(remoteServiceUrl = "<auth.authService>")
	public void setAuthService(IAuthService authService) {
		this.authService = authService;
	}
	...
}
```

## 6 定时任务
* Cron：计划任务，是在约定的时间执行已经计划好的工作  
* Cron表达式  
想了解Cron表达式最好的方法是[Quartz的官方文档](http://www.quartz-scheduler.org/documentation/)。本节也大致介绍一下。  
Cron表达式由6~7项组成，中间用空格分开。从左到右依次是：秒、分、时、日、月、周几、年（可省略）。值可以是数字，也可以是以下符号：  
  * ```*```：所有值都匹配
  * ```?```：无所谓，不关心，通常放在“周几”里
  * ```,```：或者
  * ```/```：增量值
  * ```-```：区间  

  下面举几个例子：  
  ```0 * * * * *```：每分钟（当秒为0的时候）  
  ```0 0 * * * *```：每小时（当秒和分都为0的时候）  
  ```*/10 * * * * *```：每10秒  
  ```0 5/15 * * * *```：每小时的5分、20分、35分、50分  
  ```0 0 9,13 * * *```：每天的9点和13点  
  ```0 0 8-10 * * *```：每天的8点、9点、10点  
  ```0 0/30 8-10 * * *```：每天的8点、8点半、9点、9点半、10点  
  ```0 0 9-17 * * MON-FRI```：每周一到周五的9点、10点…直到17点（含）  
  ```0 0 0 25 12 ?```：每年12月25日圣诞节的0点0分0秒（午夜）  
  ```0 30 10 * * ? 2016```：2016年每天的10点半  
 不记得也没关系，记住[Cron Maker](http://www.cronmaker.com/)也可以，它可以在线生成cron表达式。

### 6.1 Spring TaskScheduler
* 注解的方法  
    ```
    @Scheduled(cron = "${cron_expression}")
    ```
* xml配置方法  
    ```
    <task:scheduler id="myScheduler"/>
    
    <task:scheduled-tasks scheduler="myScheduler">
        <task:scheduled ref="doSomethingTask" method="doSomething" cron="${cron_expression}"/>
    </task:scheduled-tasks>
    ```
    
* [示例](https://spring.io/guides/gs/scheduling-tasks/#_enable_scheduling)  
  * Application加入```@EnableScheduling```注解  
    ```
    @SpringBootApplication
    @EnableScheduling
    public class Application {
    
        public static void main(String[] args) throws Exception {
            SpringApplication.run(Application.class);
        }
    }
    ```
  * ScheduledTasks  
    ```
    @Component
    public class ScheduledTasks {
    
        private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);
    
        private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");
        //5秒执行一次
        @Scheduled(fixedRate = 5000)
        public void reportCurrentTime() {
            log.info("The time is now {}", dateFormat.format(new Date()));
        }
        //每天9点执行
        @Scheduled(cron = "0 0 9 * * *")
        public void doSomething() {
        	log.info("Job start ......");
        	log.info("do something ......");
        	log.info("Job End ......");
        }
    }
    ```  

### 6.2 Spring集成quartz  
* 引入```quartz```依赖 
 
	```
	<dependency>
	  <groupId>quartz</groupId>
	  <artifactId>quartz</artifactId>
	  <version>1.6.5</version>
	</dependency>
	```  

* 配置定时任务执行线程和周期  
    ```
    <task:scheduler id="myScheduler" pool-size="5" />
    
    <task:scheduled-tasks scheduler="myScheduler">
        <task:scheduled ref="doSomethingTask" method="doSomething" cron="${cron_expression}"/>
    </task:scheduled-tasks>
    ```  
* 配置定时任务执行类  
    ```
    <bean id="doSomethingTask" class="com.handpay.test.QuartzTest"></bean>
    ```  
* 编写任务  
    ```
    public class QuartzTest {
    
        private static final Logger log = LoggerFactory.getLogger(QuartzTest.class);
        
        public void doSomething() {
        	log.info("Quartz start ......");
        	log.info("do something ......");
        	log.info("Quartz End ......");
        }
    }
    ```

## 7 公共服务
### 7.1 短信服务
#### 7.1.1 调用说明
目前调用短信服务的基本都调用```downloadService```，或者其他封装的服务。实际这些提供短信服务的```service```都是对短信系统进行了简单的封装。
	
短信系统直接提供下行服务的```url```是： <http://sms.99wuxian.com:12501/sgs/front/SendSm!sendMsg.action>  
参数如下表  

形参名称 | 解释	| 实例
---|---|---
tk | token令牌，不能为空 | 373058ca7a
m | 接受信息的手机号码，不能为空 | 13812345678
smc | 发送的内容，不能为空 | Hi，您好！
amid | 应该记录的id，小于32位数字，不能为空 | 123456
dtime | 定时发送时间,14位数，不填立即发送 | 20140826152930
apptp | 应用分类，小于50个字符 | app
appara | 应用自定义参数，小于50个字符 | app1
sprio | 发送优先级0-9，默认5 | 5
frt | 失败重发次数，大于0小于3 | 1
srurl | 状态报告通知应用的url | http://app.99wuxian.com/xxx.do
vtime | 短信发送有效期，不填无失效期 | 20140826154055
ext | 运营商扩展子号码，要供应商支持 | -
sm | 拆分不是none：不拆分，auto：自动，default：按照标准拆分（59个字）| auto
tm | 信息时间处理模式immediate：立刻发送（默认），delay：延时，fixed：定时 | immediate

#### 7.1.2 配置
* 首先配置```token```，如果使用了```downloadService```发送短信，不配置```token```使用默认的```token```，不建议使用默认的token.支撑【令牌管理】可以查看。
* 选择的```token```会对应一个应用名称，一个应用会对呀一个注册公司，状态为启用。支撑【应用管理】可以查看。
* 发送的通道的注册公司要跟token对应的注册公司一致，且通道处于启用状态。支撑【通道管理】可以查看。
* 如果要获取短信发送的结果只需在发送短信的时候将srurl这个参数赋值，短信系统会在信息发送有结果了，回调此```url```。回调的参数如下表  

形参名称 | 解释 | 实例
---|---|---
sgsid | 短信系统id，sequence | 2566
amid | 应该记录的id，小于32位数字，不能为空 | 123456
resp | 短信在运营商那里的发送结果 | 03(建周发送成功状态)
desc | 描述 | 	
m | 手机号码 | 13812345678
apptp | 应用分类，小于50个字符 | app
appara | 应用自定义参数，小于50个字符 | app1
sp | 运营商扩展子号码，要供应商支持| 

* 如果需要短信的上行服务需要在支撑的【上行管理->创建新配置】中选择自己的应用配置规则。短信系统在有上行的时候会去查找所有的规则，找到通知对应的应用。应用接受的参数列表如下  

形参名称 | 解释 | 实例
---|---|---
sgsid | 短信系统id，sequence | 2563
m | 手机号码 | 13812345678
rep | 上行的内容，也就是用户回复的内容 | A123
rept | 上行到达短信系统的时间 | 20140828105125


### 7.2 邮件服务
#### 7.2.1 ```pom.xml``` 增加依赖
```
<dependency>
  <groupId>com.handpay</groupId>
  <artifactId>handpay-mail-spec</artifactId>
  <version>1.0.1</version>
</dependency>
```
#### 7.2.2 ```spring``` 的 ```dubbo```配置文件，（如果工程中没有```common``` 注册中心时）增加```common```的注册中心
```
<dubbo:registry protocol="${common.dubbo.registry.protocol}"
    address="${common.dubbo.registry.address}"
    file="${user.home}/.dubbo-cache/common-3${env.num}"
    group="dubbo-${env_path}" id="common.registry" default="false" />

common.dubbo.registry.protocol = zookeeper
common.dubbo.registry.address  = common1:2141,common2:2142,common3:2143
```
#### 7.2.3 ```dubbo reference```增加
```
<dubbo:reference id="mailSender" interface="com.handpay.framework.mail.spec.MailSender" 
    check="false" retries="0" registry="common.registry" />
```  
如果工程中已配置有```common```的注册中心，把```registry```改为已配置好的```common```注册中心的```id```值  
#### 7.2.4 邮件发送，调用```com.handpay.framework.mail.spec.MailSender```接口的```send```方法：  
```
String send(MailRequest request);
//MailRequest 是个普通的 POJO 类，发送邮件时需要填写以下参数
MailRequest request = new MailRequest();

// 邮件编码，默认为 GBK
request.setEncoding( "UTF-8" );
// 邮件标题
request.setSubject( "Mail Subject" );
// 邮件发件人，该发件人必须在邮件服务中登录过
request.setFrom( from );
// 邮件收件人，可以调用多次添加多个，也可将收件人使用 ; 分隔添加
request.addRecipient( "" );
// 邮件内容，内容默认采用 HTML 发送，若需要纯文本发送添加 request.setHtml( false ); 即可
request.setContent( "邮件内容" );

// 接口方法返回值需要在 INFO 级别的日志中输出，以便于今后问题查找
String result = mailSender.send( request );
LOG.info( "Send mail, from: {}, result: {}" , request.getFrom() , result );
```

result 的数据格式为：<邮件发送标识>:<邮件发送状态>
* 邮件发送标识：每次邮件发送的唯一标识号
* 邮件发送状态：

```
success                     //邮件服务已成功接收该邮件
param.required.<field_name> //request中的field字段为 null 或者空
encoding.unsupported.XXXX   //XXXX 字符编码不被支持
mail.from.not.exists        //发件人没有在邮件服务中登记
mail.smtp.not.exists        //邮件发送的 SMTP 服务器不存在
```

result 仅在返回 <邮件发送标识>: success 才表示邮件服务已成功接收该邮件

### 7.3 Otp服务
#### 7.3.1 Otp 管理
otp管理系统测试环境地址：<http://10.48.170.201:7000/otp-admin/login.htm>  
用户名/密码：```admin/123456```，生产环境由```sys```操作

#### 7.3.2 新增用户
1. 登录 OTP 管理系统
2. 点击【用户管理】中的【新建】按钮
3. 填写业务系统代号和业务系统的登录用户名后，点击【添加】
4. 通过【查询】查找到第 3 步添加的用户，点击【生成二维码】按钮
5. 使用手机的 Google Authenticator 应用扫描二维码  
   若手机上未装 Google Authenticator时，点击二维码下面相应的手机操作系统进行安装，目前支持 iOS 和 Android  
   (a) iOS 会跳至 itunes 下载应用  
   (b) Android 通过扫描二维码直接下载安装  
 Google Authenticator 和 Google Authenticator 所使用的 Zxing 扫码安装程序
6. 点击【验证】，输入系统登录用户名和业务系统代码，以及 Google Authenticator 生成的 6 位数字，点击【验证】  
   6.1  如果验证成功，表示当前的扫码是正确的  
   6.2  如果验证不成功  
   
    ```
    (a) 可以尝试重新点击【生成二维码】，重新扫码（重新扫码之前需要将手机 Google Authenticator 中对应用户删除）  
    (b) 校准手机的系统时间
    ```

#### 7.3.3 HTTP 接口
##### 7.3.3.1 新增用户  
* URL：<http://otpservice/otp-service/api/creator>  
* POST 参数：  

    ```
    userName: 业务系统的用户名
    issuer: 业务系统代号
    mobile: 手机号码（选填）
    name: 用户姓名（选填）
    ```

* 返回 JSON：  

    ```
	创建成功: { "message": "ok" }
	重复数据: { "message": "此条记录已存在" }
    issuer为空: { "message": "没有填写认证单位参数(issuer参数为空)" }
    userName为空: { "message": "没有填写认证单位参数(userName参数为空)" }
    ```

##### 7.3.3.2 校验口令
* URL：<http://otpservice/otp-service/api/verifier>
* POST 参数：  

    ```
    userName: 业务系统的用户名
    issuer: 业务系统代号
    authCode: 用户输入需要校验的 OTP 口令
    ```

* 返回 JSON：  

	```
	校验成功: { "passed": true, "message": "ok" }
    authCode为空或错误: { "passed": false, "message": "验证未通过" }
    issuer为空: { "passed": false, "message": "没有填写认证单位参数(issuer参数为空)" }
    userName为空: { "passed": false, "message": "没有填写用户名参数(userName参数为空)" }
    ```
