# nove2.0
基于spring cloud F版本的2.0基础框架组件

手册说明
       此手册用于Redefine-nove 开发平台各个功能使用简介，此平台建立是为了辅助上层应用快速实现业务开发，屏蔽不必要的工作。另一方面帮助系统工具进行同一化管理（包括jar version 等），提供插件化的开发方式，为未来的服务链健康系统提供基础。



注意事项：

1、根据运维持续集成策略，开发需要拥有三个分支develop test master,其中develop 分支提交代码将自动打包部署并合并到test ，master 为手动。

2、在配置中server.port log.home 这两个参数必须写在bootstrap.properties中，不能写在远端git，不然在运维部署时将无法动态替换

3、开发项目package 请以 global.redefine.xxx  格式命名

4、redefine-nove 版本说明 《A 基础平台版本升级信息》

5、基于spring boot 2.0 升级，请查看《Spring Cloud F & Spring boot 2.0 Version Update》

一、模块简介
redefine-nove

     顶级目录，里面包含了spring boot  、cloud当前版本，定义了基础功能组件版本。

redefine-core

     核心组件，用于扩展插件化功能扩展等基础。

redefine-kafka

    提供kafka基本功能，目的是接管spring默认功能组件，为未来的个性化配置和监控作准备

redefine-rabbitmq

  . 提供rabbitmq基本功能，目的是接管spring默认功能组件，为未来的个性化配置和监控作准备

redefine-redis

    提供redis基本功能，目的是接管spring默认功能组件，为未来的个性化配置和监控作准备

redefine-jdbc 

    提供jdbc基础功能，内部使用druid连接池

使用方式

    在自己项目POM中添加 如下配置



<parent>
    <groupId>com.redefine</groupId>
    <artifactId>redefine-nove</artifactId>
    <version>1.5.0-RELEASE</version>
</parent>


<dependencies>
    <dependency>
        <groupId>com.redefine</groupId>
        <artifactId>redefine-core</artifactId>
    </dependency>
    
</dependencies>


<build>
	<!--此为项目名称-->
    <finalName>redefine-consumer</finalName>
    <resources>
        <resource>
            <directory>${basedir}/src/main/resources</directory>
            <filtering>true</filtering>
            <includes>
                <include>**/*</include>
            </includes>
        </resource>
    </resources>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
                <skip>true</skip>
                <testFailureIgnore>true</testFailureIgnore>
            </configuration>
        </plugin>
    </plugins>
</build>




二、KAFKA使用
1、添加POM依赖

<dependency>
 <groupId>com.redefine</groupId>
 <artifactId>redefine-kafka</artifactId>
</dependency>

2、在配置中心增加kafka基础配置，个性化配置自行定义

spring.kafka.consumer.group-id=consumer
spring.kafka.bootstrap-servers=localhost:5679

3、发送数据demo

package com.redefine.service;

import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

/**
 * Created by luqiang on 2018/3/13.
 */
@Service("testKafkaService")
public class TestKafkaService {

    @Resource
    private KafkaTemplate<String,String> kafkaTemplate;


    public String sendKafkaMsg(){
        kafkaTemplate.send("testqiang","测试kafka基础组件"+System.currentTimeMillis());
        return "发送消息完成";
    }
}


4、接收demo

package com.redefine.service;

import org.springframework.kafka.annotation.KafkaHandler;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

/**
 * Created by luqiang on 2018/3/13.
 * 因kafka为批量拉取同步消费，如果对性能有要求，请开启异步消费
 */
@Component
@KafkaListener(topics = "testMsg")
public class TestKafkaListener {

    @KafkaHandler
    public void getMessage(String data){
        System.out.println("接收到消息："+data);
    }
}





5、Base 包内部基础配置细节

1、kafka ack发送确认机制默认配置为 1，可使用参数 spring.kafka.producer.acks 进行定义

/ at most once: 最多一次,这个和JMS中"非持久化"消息类似.发送一次,无论成败,将不会重发. 1
// at least once: 消息至少发送一次,如果消息未能接受成功,可能会重发,直到接收成功. 2
// exactly once: 消息只会发送一次. 3


2、kakfa consumer listener concurrency 默认设置为3 ，也就是说默认三个消费者，如有需求请配置参数 spring.kafka.listener.concurrency 



三、RabbitMQ 使用

1、添加依赖
<dependency>
    <groupId>com.redefine</groupId>
    <artifactId>redefine-rabbitmq</artifactId>
</dependency>

2、添加配置
spring.rabbitmq.addresses=localhost:5672

3、发送demo
package com.redefine.service;

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

/**
 * Created by luqiang on 2018/3/13.
 */
@Service
public class TestRabbitmqService {

    @Resource
    private RabbitTemplate rabbitTemplate;

    public String sendMQ(){

        rabbitTemplate.convertAndSend("testMQ","测试MQ基础服务");
        return "发送MQ基础测试";
    }

}




4、接收demo

package com.redefine.service;

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * Created by luqiang on 2018/3/13.
 * 因kafka为批量拉取同步消费，如果对性能有要求，请开启异步消费
 */
@Component
@RabbitListener(queues = "testMQ")
public class TestRabbitistener {

    @RabbitHandler
    public void getMessage(String data){
        System.out.println("接收到消息："+data);
    }
}


5、Base 包内部基础配置细节

1、每个线程 channelCacheSize 默认配置为50

2、MessageConverter 发送使用默认 SimpleMessageConverter 接收兼容了Jackson2JsonMessageConverter

3、默认设置CreateMessageIds 使用UUID

4、默认 ConcurrentConsumers 设置为3 ，使用spring.rabbitmq.listener.simple.concurrency 进行更改

5、默认拉去消息队列配置PrefetchCount 为50，此参数非常重要，太小有可能阻塞业务消费,使用 spring.rabbitmq.listener.simple.prefetch 修改

6、因为rabbitmq默认消费失败无限重试，为了防止也无阻塞进行失败兼容。开始失败重试后，默认失败三次的消息将记录日志并发送到监控系统以北查询。开启失败重试的业务应当注意这个点。使用参数 spring.rabbitmq.template.retry.enabled=true 开启




四、Redis 使用
1、添加POM 依赖

<dependency>
 <groupId>com.redefine</groupId>
 <artifactId>redefine-redis</artifactId>
</dependency>

2、添加基础配置，



REDIS  STAND-ALONE CONFIG

spring.redis.host=localhost
spring.redis.port=6379


REDIS CLUSTER CONFIG

spring.redis.cluster.max-redirects=@ecej.redis.maxRedirects@

spring.redis.cluster.nodes=127.0.0.1:6379, 127.0.0.1:6380

REDIS SENTINEL CONFIG

spring.redis.sentinel.master=@ecej.redis.mastername@

spring.redis.sentinel.nodes=127.0.0.1:6379, 127.0.0.1:6380


3、demo

package com.redefine.service;

import com.redefine.redis.utils.RedefineClusterUtils;
import org.springframework.stereotype.Service;

/**
 * Created by luqiang on 2018/3/13.
 */
@Service
public class TestRedisService {

	//此demo 使用封装RedefineClusterUtils ，也可以使用原生 private RedisTemplate<String, String> redisTemplate
    public String testRedis(){

        RedefineClusterUtils.saveString("testKey","对Redis数据进行测试");
        return RedefineClusterUtils.getString("testKey");
    }

}


4、可用于生产的基础配置

REDIS CLUSTER CONFIG

spring.redis.cluster.max-redirects=@ecej.redis.maxRedirects@

spring.redis.cluster.nodes=127.0.0.1:6379,127.0.0.1:6380

spring.redis.password=@ecej.redis.password@


# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=300
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=15
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=5
# 连接超时时间（毫秒）
spring.redis.timeout=10
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=1000


5、Base 包内部基础配置细节

1、兼容单机 sentinel cluster 等启动模式

2、内部转发配置默认为10

五、JDBC使用
此JDBC 内部使用druid + JPA 如有其他需求请联系

1、添加POM

<dependency>
 <groupId>com.redefine</groupId>
 <artifactId>redefine-jdbc</artifactId>
</dependency>

2、添加配置（详细配置可参考druid官方文档 https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter，内置默认监控开启）

spring.datasource.druid.url= # 或spring.datasource.url= 
spring.datasource.druid.username= # 或spring.datasource.username=
spring.datasource.druid.password= # 或spring.datasource.password=
spring.datasource.druid.driver-class-name= #或 spring.datasource.driver-class-name=


六、基础项目配置


此基础项目配置用于为大家展示提供最简的项目配置

1、当项目只作为消费者 不提供服务时 配置以下内容即可

1）、添加依赖

<dependency>
    <groupId>com.redefine</groupId>
    <artifactId>redefine-core</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>


2）基础配置

ureka.client.serviceUrl.defaultZone=http://localhost:8999/eureka/
eureka.instance.hostname=localhost
eureka.client.healthcheck=true

feign.hystrix.enabled=true
#断路器的超时时间需要大于ribbon的超时时间，不然不会触发重试
hystrix.command.defult.execution.isolation.thread.timeoutInMilliseconds=3000
hystrix.threadpool.default.coreSize=500

#开启服务重试机制，注意幂等性
spring.cloud.loadbalancer.retry.enabled=true
#切换实例的重试次数
redefine-provider.ribbon.MaxAutoRetriesNextServer=2
#对当前实例的重试次数
redefine-provider.ribbon.MaxAutoRetries=1
#请求连接的超时时间
redefine-provider.ribbon.ConnectTimeout=250
#请求处理的超时时间
redefine-provider.ribbon.ReadTimeout=1000
#对所有操作请求都进行重试
redefine-provider.ribbon.OkToRetryOnAllOperations=true


3）启动类需要的注解

@SpringCloudApplication
@EnableDiscoveryClient

4）Feign配置类

package com.redefine.config;

import org.springframework.cloud.netflix.feign.EnableFeignClients;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

/**
 * Created by luqiang on 2018/3/13.
 */
@Configuration
@EnableFeignClients(basePackages = "com.redefine")
public class FeignConfig {


}




5）服务接口demo

package com.redefine.service;

import com.redefine.dto.User;
import com.redefine.fallback.RedefineServiceFallback;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.*;

/**
 * Created by luqiang on 2018/3/12.
 */
@FeignClient(value = "redefine-provider")
@Component("feignConsumer")
public interface FeignConsumer {

    @RequestMapping("/provider/testP")
    public String testP();

    @RequestMapping(value = "/provider/testPR",method = RequestMethod.GET)
    public String testPR(@RequestParam("name") String name);

    @RequestMapping(value = "/provider/testPH",method = RequestMethod.GET)
    public String testPH(@RequestHeader("name") String name, @RequestHeader("age") String age);

    @RequestMapping(value = "/provider/testPB",method = RequestMethod.POST)
    public String testPB(@RequestBody User user);

    @RequestMapping(value = "/provider/testPRedis",method = RequestMethod.GET)
    public String testPRedis();

    @RequestMapping(value = "/provider/testPRabbit",method = RequestMethod.GET)
    public String testPRabbit();

}






2、当项目只作为提供者 不消费服务时 配置以下内容即可

1）添加依赖



<dependency>
    <groupId>com.redefine</groupId>
    <artifactId>redefine-core</artifactId>
</dependency>


2）添加基础配置

eureka.client.serviceUrl.defaultZone=http://localhost:8999/eureka/
eureka.client.healthcheck.enabled=true

3）启动类添加注解

@EnableEurekaClient
@SpringCloudApplication


3、如果即提供服务 也消费服务 组合一下上边的就行了



4、提供bootstrap.properties基础配置



spring.application.name=redefine-demo
server.port=8081
log.home=/tmp/redefine/logs/redefine-demo

spring.cloud.config.profile=@redefine.config.profile@
spring.cloud.config.label=@redefine.config.label@
#注册自己并声明从哪个配置中心获取配置
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.service-id=REDEFINE-CONFIG
#eureka 注册地址
eureka.client.serviceUrl.defaultZone=@redefine.eureka.serviceurl@
POM中的profile

<profiles>
    <profile>
        <id>local</id>
        <properties>
            <redefine.eureka.serviceurl>http://localhost:8999/eureka/</redefine.eureka.serviceurl>
            <redefine.eureka.instance.hostname>localhost</redefine.eureka.instance.hostname>
            <redefine.redefine.config.profile>dev</redefine.redefine.config.profile>
            <redefine.redefine.config.label>master</redefine.redefine.config.label>
        </properties>

    </profile>
    <profile>
        <id>dev</id>
        <properties>
            <redefine.eureka.serviceurl>http://39.104.13.81:8090/eureka/</redefine.eureka.serviceurl>
            <redefine.eureka.instance.hostname>HB-DEV-APP-SERVER-01</redefine.eureka.instance.hostname>
            <redefine.redefine.config.profile>dev</redefine.redefine.config.profile>
            <redefine.redefine.config.label>master</redefine.redefine.config.label>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>test</id>
        <properties>
            <redefine.eureka.serviceurl>http://149.129.130.68:8090/eureka/</redefine.eureka.serviceurl>
            <redefine.eureka.instance.hostname>YD-TEST-APP-SERVER-01</redefine.eureka.instance.hostname>
            <redefine.redefine.config.profile>dev</redefine.redefine.config.profile>
            <redefine.redefine.config.label>master</redefine.redefine.config.label>
        </properties>
    </profile>
</profiles>







七、项目部署方式


项目部署采用boot标准模式 ，端口和log地址由运维分配，部署命令如下：

例：nohub java jar xxx.jar -server.port=8080 --log.home=/tmp/xxx --log.stdout=0 & 

注意：为了方便运维分配端口和log地址，配置中server.port  、log.home  不允许写在远端git上，需要写在bootstrap.properties里，否则将不能动态替换



八、将项目加入配置中心


1、添加依赖，其中core里面包含了配置中心需要的依赖

<dependency>
 <groupId>com.redefine</groupId>
 <artifactId>redefine-core</artifactId>
</dependency>


2、添加注解开启注册到eureka

@EnableDiscoveryClient
3、添加配置文件

spring.application.name=redefine-consumer
spring.cloud.config.profile=dev
spring.cloud.config.label=master
#注册自己并调用SERVER
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.service-id=REDEFINE-CONFIG
eureka.client.serviceUrl.defaultZone=http://localhost:8999/eureka/


九、Swagger 配置


因为目前内外网络的问题，默认项目启动获取的为内网IP ，外网不能访问接口页面，所以暂时提供动态配置参数，下面举例说明

redefine.swagger.path=localhost:8080（配置应用的外网访问IP）
此参数配置后，在eureka链接时将使用此IP 访问swagger（此为暂时解决方案，后期将进行swagger聚合）


生产环节下swagger默认已关闭，可使用配置自行打开

redefine.swagger.open=true


十、各个节点超时时间配置选择


1、配置网关超时

官方解说：
If Zuul is using service discovery there are two timeouts you need to be concerned with, the Hystrix timeout (since all routes are wrapped in Hystrix commands by default) and the Ribbon timeout. The Hystrix timeout needs to take into account the Ribbon read and connect timeout PLUS the total number of retries that will happen for that service. By default Spring Cloud Zuul will do its best to calculate the Hystrix timeout for you UNLESS you specify the Hystrix timeout explicitly.



也就是说如果是网关行为，就得多方面考虑了。要关注两个超时Hystrix 和 Ribbon 这两个超时配置。Hystrix超时需要考虑到Ribbon读取和连接超时以及该服务将要发生的重试的总数。



  Hystrix 超时计算公式：

(ribbon.ConnectTimeout + ribbon.ReadTimeout) * (ribbon.MaxAutoRetries + 1) * (ribbon.MaxAutoRetriesNextServer + 1)


如果你设置了

hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds

任何一个参数，那就必须自己考虑好下面俩


#ribbon请求连接的超时时间- 限制1秒内必须请求到服务，并不限制服务处理的返回时间

ribbon.ConnectTimeout=1000

#请求处理的超时时间 下级服务响应最大时间,超出时间消费方（路由也是消费方）返回timeout

ribbon.ReadTimeout=1000


如果你在网关设置了路由规则，需要设置以下参数

#default 1000

 zuul.host.connect-timeout-millis=1000

#default 1000

 zuul.host.socket-timeout-millis=1000

完整配置：

hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=3000
ribbon.ConnectTimeout=1000
ribbon.ReadTimeout=1000
zuul.host.connect-timeout-millis=1000
zuul.host.socket-timeout-millis=1000







十一、开启失败重试机制


 开启失败重试机制（redefine-provider 为服务名称）

#开启服务重试机制，注意幂等性
spring.cloud.loadbalancer.retry.enabled=true
#切换实例的重试次数
redefine-provider.ribbon.MaxAutoRetriesNextServer=1
#对当前实例的重试次数
redefine-provider.ribbon.MaxAutoRetries=1
#请求连接的超时时间
redefine-provider.ribbon.ConnectTimeout=1000
#请求处理的超时时间
redefine-provider.ribbon.ReadTimeout=1000
#对所有操作请求都进行重试
redefine-provider.ribbon.OkToRetryOnAllOperations=true


十二、系统监控


地址：http://watchdog.welike.in/

配置：

此系统根据eureka自动获取应用信息，只需关闭auth权限验证即可

#关闭监控数据权限验证
management.security.enabled=false

1、流量监控



如果点击http://172.31.47.170:8703/hystrix 将进入配置页面

图解

http://localhost:8089/turbine.stream


01-开发 > A 基础平台开发手册 > 20171013115833819.png



01-开发 > A 基础平台开发手册 > 20171013115932013.png



2、应用监控

添加cloud admin监控系统 ，引用原文来简介下此系统作用：

http://codecentric.github.io/spring-boot-admin/current/#_what_is_spring_boot_admin



What is Spring Boot Admin?

codecentric’s Spring Boot Admin is a community project to manage and monitor your Spring Boot ® applications. The applications register with our Spring Boot Admin Client (via HTTP) or are discovered using Spring Cloud ® (e.g. Eureka, Consul). The UI is just an AngularJs application on top of the Spring Boot Actuator endpoints



3、调用链监控

应用关闭调用链监控： spring.zipkin.enabled=false

配置：

#监控服务器地址

pring.zipkin.base-url=http://localhost:8086
spring.sleuth.sampler.percentage=1.0
#转化率

十三、统一日志配置


1、项目体系中统一使用logback日志模块

2、日志配置路径支允许写在bootstrap.properties 中，key为 log.home 。不允许写在远程配置仓库，以免运维统一动态参数失效。

3、请使用统一日志样式方便日志收集系统进行分析，样式如下：

- | [%d{yyyyMMdd HH:mm:ss.SSS}] | [${APPNAME}]| [${IP}] | [${HOSTNAME}] | [%level] | [%X{X-B3-TraceId}] | [%X{X-B3-SpanId}] | [%X{X-B3-ParentSpanId}] | [%thread] | [%logger{40}] | [--> %msg] |%n
4、给出统一logback-spring.xml 配置文件，不需要额外任何配置
《logback-spring.xml》



十四、代码开发合并流程
01-开发 > A 基础平台开发手册 > QxVmJ.png

十五、服务调用链监控（暂缓）
1、添加依赖实现内置监控

<!-- log monitor -->
<dependency>
 <groupId>com.redefine</groupId>
 <artifactId>redefine-monitor</artifactId>
</dependency>
注意：使用此监控依赖前提是使用了远程配置仓库，因为很多特定配置由仓库提供


十六、添加动态配置推送


在动态字段 添加类注解

@RefreshScope

修改后执行刷新命令

http://host:port/bus/refresh



十七、网关添加统一异常拦截


1、首先定义统一异常拦截POST，用于拦截下游服务异常

package global.redefine.welike.gateway.filter;

import com.alibaba.fastjson.JSON;
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.netflix.zuul.util.ZuulRuntimeException;
import org.springframework.util.StreamUtils;

import javax.annotation.PostConstruct;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.Charset;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

/**
 * AccessTokenFilter
 *
 * @author zhangjunjie | 2018-03-15
 */
public class TestPostFilter extends ZuulFilter {



    @Override
    public String filterType() {
        return "post";
    }

    @Override
    public int filterOrder() {
        return 999;
    }

    @Override
    public boolean shouldFilter() {
        RequestContext context = RequestContext.getCurrentContext();
        InputStream stream = context.getResponseDataStream();
        try {
            String body = StreamUtils.copyToString(stream, Charset.forName("UTF-8"));
            Map<String,Object> map = (Map<String, Object>) JSON.parse(body);
            context.setResponseBody(body);
            if(500 == (Integer) map.get("status")){
                return true;
            }
        } catch (IOException e) {

        }
        return false;
    }

    @Override
    public Object run() {
        RequestContext context = RequestContext.getCurrentContext();
		//TODO
        return null;
    }

    ZuulException findZuulException(Throwable throwable) {
        if (throwable.getCause() instanceof ZuulRuntimeException) {
            // this was a failure initiated by one of the local filters
            return (ZuulException) throwable.getCause().getCause();
        }

        if (throwable.getCause() instanceof ZuulException) {
            // wrapped zuul exception
            return (ZuulException) throwable.getCause();
        }

        if (throwable instanceof ZuulException) {
            // exception thrown by zuul lifecycle
            return (ZuulException) throwable;
        }

        // fallback, should never get here
        return new ZuulException(throwable, HttpServletResponse.SC_INTERNAL_SERVER_ERROR, null);
    }
}




2、如有特殊异常处理逻辑可添加自定义error filter

3、添加异常重定向，用于拦截系统内部异常

package global.redefine.welike.gateway.controller;

import org.springframework.boot.autoconfigure.web.ErrorController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Created by luqiang on 2018/4/3.
 */
@RestController
public class ErrorHandlerController implements ErrorController {

    /**
     * 出异常后进入该方法，交由下面的方法处理
     */
    @Override
    public String getErrorPath() {
        return "/error";
    }

    @RequestMapping("/error")
    public String error(HttpServletRequest request, HttpServletResponse response) {
        System.out.println(request.getAttribute("javax.servlet.error.exception"));
        System.out.println(request.getAttribute("javax.servlet.error.status_code"));
        System.out.println(request.getAttribute("javax.servlet.error.message"));
        return "出现异常";
    }
}


十八、使用MongoDB
1、添加依赖

<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>


2、添加配置文件



spring.data.mongodb.uri=mongodb://userm:xxx@10.xx.96.xx:27017/monitor
多个IP集群可以采用以下配置：

spring.data.mongodb.uri=mongodb://user:pwd@ip1:port1,ip2:port2/database

3、注入模版

@Resource
private MongoTemplate mongoTemplate;
十九、优雅停机
使用方式

POST http://ip:port/shutdown



二十、基于RabbitMQ的延迟队列
前言：

    对于延迟队列有两种实现模式，一种就是rabbitmq_delayed_message_exchange 插件，另一种就是基于TTL+DLX 实现，因为rabbitmq_delayed_message_exchange是非官方的，缺乏大面积生产实践，并且出现了一些小问题，所有我们此次选用后一种模式。



下面举例： 比我我要发送消息到consumer 队列 ，设定超时时间 5S

增加配置：

<dependency>
 <groupId>com.redefine</groupId>
 <artifactId>redefine-rabbitmq</artifactId>
</dependency>
配置文件：
spring.rabbitmq.addresses=localhost:5672


1、实现思路

   具体用了RabbitMQ的两个特性，一个是Time-To-Live Extensions，另一个是Dead Letter Exchanges。



Time-To-Live Extensions



RabbitMQ允许我们为消息或者队列设置TTL（time to live），也就是过期时间。TTL表明了一条消息可在队列中存活的最大时间，单位为毫秒。也就是说，当某条消息被设置了TTL或者当某条消息进入了设置了TTL的队列时，这条消息会在经过TTL秒后“死亡”，成为Dead Letter。如果既配置了消息的TTL，又配置了队列的TTL，那么较小的那个值会被取用。



Dead Letter Exchange



刚才提到了，被设置了TTL的消息在过期后会成为Dead Letter。其实在RabbitMQ中，一共有三种消息的“死亡”形式：



消息被拒绝。通过调用basic.reject或者basic.nack并且设置的requeue参数为false。
消息因为设置了TTL而过期。
消息进入了一条已经达到最大长度的队列。


如果队列设置了Dead Letter Exchange（DLX），那么这些Dead Letter就会被重新publish到Dead Letter Exchange，通过Dead Letter Exchange路由到其他队列。



2、流程图





3、上代码

首先定义一下配置：

package global.redefine.config;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Created by luqiang on 2018/5/11.
 */
@Configuration
public class RabbitConfig {


    /**
     * 实际消费队列
     * @return
     */
    @Bean
    public Queue queue() {
        return new Queue("consumer");
    }


    /**
     * DLX交换机
     * @return
     */
    @Bean
    DirectExchange delayExchange() {
        return new DirectExchange("delayExchange");
    }


    /**
     * 组建前置队列，注意consumer-pre 为前置队列名称，delayExchange 为交换机名称，
     * consumer 为实际队列名称，5000为超时时间
     * @return
     */
    @Bean
    Queue delayQueuePerQueueTTL() {
        return QueueBuilder.durable("consumer-pre")
                .withArgument("x-dead-letter-exchange", "delayExchange")
                .withArgument("x-dead-letter-routing-key", "consumer")
                .withArgument("x-message-ttl", 5000)
                .build();
    }


    /**
     * 绑定交换机和队列
     * @return
     */
    @Bean
    public Binding dlxBinding() {
        return BindingBuilder.bind(queue()).to(delayExchange()).with("consumer");
    }
}




4、发送

rabbitTemplate.convertAndSend("consumer-pre", "this is test data TTL 5")



5、消费者

@RabbitListener(queues = "consumer")
@Component
public class RabbitmqService {

    @RabbitHandler
    public void consumer(String data){
        System.out.println(data);
    }

}

二十一、添加自定义参数到服务追踪
 nove version 1.4.4

注入接口

@Resource
private Tracer tracer;
tracer.addTag("key","value");
二十二、spring task 多线程配置
问题：

因为@Scheduled默认为单线程执行模式，在某些情况下不满足业务需求，会导致卡死无法执行定时任务

解决：

进行如下配置即可：

@Configuration
public class ScheduleConfig implements SchedulingConfigurer {
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(Executors.newScheduledThreadPool(5));
 }
}
二十三、Mongo分页样例
很常用的模式，记录在此，方便使用

package global.redefine.watchdog.po;

import java.io.Serializable;
import java.util.List;

/**
 * Created by luqiang on 2018/5/28.
 */
public class PageModel<T> implements Serializable {
    private static final long serialVersionUID = 8776788184840922942L;
    private List<T> datas;
    private int rowCount;
    private int pageSize = 100;
    private int pageNo = 1;
    private int skip = 0;

    public PageModel() {
    }

    public int getTotalPages() {
        return (this.rowCount + this.pageSize - 1) / this.pageSize;
    }

    public List<T> getDatas() {
        return this.datas;
    }

    public void setDatas(List<T> datas) {
        this.datas = datas;
    }

    public int getRowCount() {
        return this.rowCount;
    }

    public void setRowCount(int rowCount) {
        this.rowCount = rowCount;
    }

    public int getPageSize() {
        return this.pageSize;
    }

    public void setPageSize(int pageSize) {
        this.pageSize = pageSize;
    }

    public int getSkip() {
        this.skip = (this.pageNo - 1) * this.pageSize;
        return this.skip;
    }

    public void setSkip(int skip) {
        this.skip = skip;
    }

    public int getPageNo() {
        return this.pageNo;
    }

    public void setPageNo(int pageNo) {
        this.pageNo = pageNo;
    }
}




/**
 * 获取应用接口列表详情统计数据
 *
 * @return
 */
@Override
public PageModel<UltronSpan> getInterfaceDatasByName(String serviceName,String interfaceName, int pageNo) {


    PageModel<UltronSpan> pageModel = new PageModel<>();
    pageModel.setPageNo(pageNo);

    Criteria criteria = Criteria.where("localEndpoint.serviceName").is(serviceName).and("name").is(interfaceName);
    Query query = new Query(criteria);
    int count = (int) mongoTemplate.count(query, "redefine-trace");
    pageModel.setRowCount(count);
    query.with(new Sort(Sort.Direction.DESC, "timestamp"));
    query.skip(pageModel.getSkip()).limit(pageModel.getPageSize());
    List<UltronSpan> result = mongoTemplate.find(query, UltronSpan.class, "redefine-trace");
    pageModel.setDatas(result);

    return pageModel;
}


简单易懂



二十四、多版本控制
1、添加依赖

<dependency>
 <groupId>com.redefine</groupId>
 <artifactId>redefine-version</artifactId>
</dependency>

2、配置版本要求

注意：只需要在提供者端配置，不配置的话默认允许所以版本

eureka.instance.metadata-map.versions=1


3、权重配置

eureka.instance.metadata-map.weight=2（默认为10 ，区间为1～10）


如果有特殊参数获取需求，比如我需要从cookie session 等获取参数的需求,请自行实现 RequestVersionExtractor接口进行扩展。


3、举例

请求：

http://localhost:8998/rc/consumer/testFeign?version=1

配置：

//此配置代表允请求中携带版本号为1/2/3的request可以路由到此服务提供者
eureka.instance.metadata-map.versions=1,2,3


4、动态配置

http://10.1.2.136:8090/eureka/apps/REDEFINE-ULTRON/ip-10-1-2-251.ap-southeast-1.compute.internal:redefine-ultron:8702/metadata?weight=5



二十五、应用开启HTTPS 配置
原理：

参考：http://tomcat.apache.org/tomcat-6.0-doc/api/org/apache/catalina/valves/RemoteIpValve.html



01-开发 > A 基础平台开发手册 > image2018-6-19_21-41-48.png

1、配置nginx 

server {
    listen 80;
    listen 443 ssl;
    server_name localhost;
 
    ssl_certificate server.crt;
    ssl_certificate_key server.key;
 
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
    }
}



2、服务器配置

server.tomcat.remote_ip_header=x-forwarded-for
server.use-forward-headers=true

二十六、覆盖远程配置
1、默认配置是远程覆盖的，也就是说远程git仓库配置优先级最高，如果需要使用本地配置避免被覆盖，请配置以下属性：

spring.cloud.config.overrideNone=true 允许本地覆盖远程任何属性
-spring.cloud.config.overrideSystemProperties=false 仅仅覆盖系统属性和环境变量 



二十七、Hive 模块支持
1、添加依赖

<dependency>
<groupId>com.redefine</groupId>
<artifactId>redefine-hive</artifactId>
</dependency>



2、添加配置

redefine.hive.driver=org.apache.hive.jdbc.HiveDriver
redefine.hive.url=jdbc:hive2://172.31.46.220:10000/default

其他配置



redefine.hive.maxActive
redefine.hive.initialSize





3、添加注入

@Autowired
@Qualifier("hiveJdbcTemplate")
JdbcTemplate hiveJdbcTemplate;



4、使用

hiveJdbcTemplate.execute("drop table if exists consumer_table");
hiveJdbcTemplate.execute("CREATE table consumer_table (id string,name1 string)");
二十八、数据库XML 代码生成器
地址：http://10.1.2.167:8706/

用途：帮助开发生成mybatis 配置文件

01-开发 > A 基础平台开发手册 > image2018-7-19_21-5-3.png

二十九、动态抓取请求参数
需求：为了进行实时数据统计，针对服务请求参数动态抓取

1、配置

@Resource
private RedefineTracer redefineTracer;

redefineTracer.addTag("myname","test12345");
redefineTracer.addTag("myname1","test12345");
redefineTracer.addTag("myname2","test12345");
redefineTracer.addTag("myname3","test12345");
redefineTracer.addTag("myname4","test12345");
三十、基础版本检测
目的：时刻检测应用版本

在application.properties 或 bootstrap.properties 中增加如下基础配置

######VERSION CHECK######
info.versioncheck.version=@version@
info.versioncheck.parent=@parent@



三十一、服务端A/B TEST
redefine.abtest.enable=true
使用

当前使用hash(gaid+factor)%100 

添加配置

abtest.params 为固定前缀   

percent 为固定分桶百分比后缀

data 为固定分桶数据后缀

比如发post 功能，定义一个功能关键字 post

abtest.params.post.percent.A =30代表 post 功能 分桶百分比为30%





abtest.factor=L1 //因子

abtest.params.post.percent.A=30

abtest.params.post.percent.B=30

abtest.params.post.percent.C=40

abtest.params.post.data.A={\"test22\":\"123\"}

abtest.params.post.data.B={\"test22\":\"344\"}

abtest.params.post.data.C={\"test22\":\"55544\"}

此配置需要在配置平台进行配置，添加完成可使用动态刷新生效

使用

引入 nove version 大于 1.6.x

在方法上使用注解

@NoveTest(name = "post")  post 为功能标识

获取数据属性

RandomBucketUtils.getProperties("name");

添加自定义打点日志            

RedefineSpanContextHolder.addTag("hashBucket",result);



在服务端默认使用userId 做为分桶id,如需要特殊字段请进行配置：

例：使用 gid 做为参数

abtest.idName=gid


