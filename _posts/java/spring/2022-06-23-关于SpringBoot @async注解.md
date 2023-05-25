---
layout: articles
title: 关于SpringBoot @async注解
tags:  多线程
author: Warning
key:    java-spring-head-07
aside:
  toc: true
sidebar:
nav: Java
category: [java, spring]
---

SpringBoot使用异步线程池:
1. 编写线程池配置类，自定义一个线程池；
2. 定义一个异步服务；
3. 使用@Async注解指向定义的线程池；

这里以我工作中使用过的一个案例来做描述，我所在公司是医疗行业，敏感数据需要上报到某监管平台，所以有一个定时任务在流量较小时（一般是凌晨后）执行上报行为。
但特殊时期会存在一定要在工作时间大批量上报数据的情况，且要求短时间内就要完成，此时就考虑写一个专门的异步上报接口手动执行，利用线程池上报，极大提高了速度。

<!--more-->



------




### 编写线程池配置类



```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;
import java.util.concurrent.ThreadPoolExecutor;

/**
 * 类名称：ExecutorConfig
 * ********************************
 * <p>
 * 类描述：线程池配置
 *
 * @author warning
 * @date 2022-06-07 09:00
*/
@Configuration
@EnableAsync
@Slf4j
public class ExecutorConfig {
   /**
    * 定义数据上报线程池
    * @return
    */
   @Bean("dataCollectionExecutor")
   public Executor dataCollectionExecutor() {

       ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();

       // 核心线程数量：当前机器的核心数
       executor.setCorePoolSize(
               Runtime.getRuntime().availableProcessors());

       // 最大线程数
       executor.setMaxPoolSize(
               Runtime.getRuntime().availableProcessors() * 2);

       // 队列大小
       executor.setQueueCapacity(Integer.MAX_VALUE);

       // 线程池中的线程名前缀
       executor.setThreadNamePrefix("sjsb-");

       // 拒绝策略：直接拒绝
       executor.setRejectedExecutionHandler(
               new ThreadPoolExecutor.AbortPolicy());

       // 执行初始化
       executor.initialize();

       return executor;
   }

}
```

PS：

1）、需要注意，这里一定要自己定义ThreadPoolTaskExecutor线程池，否则springboot的异步注解会执行默认线程池，存在线程阻塞导致CPU飙高及内存溢出的风险。这一点可以参考阿里开发手册，线程池定义这块明确提到了这一点；

2）、在@Bean注解中定义线程池名称，后面异步注解会用到。



### 编写异步服务



```java
 /**
 * 异步方法的服务, 不影响主程序运行。
 */
 @Service
 public class AsyncService {

    private final Logger log = LoggerFactory.getLogger(AsyncService.class);

    /**
     * 发送短信
     */
    @Async("sendMsgExecutor")
    public void sendMsg(String access_token, Consult item, Map<String, String> configMap) {
        // 此处编写发送短信业务
        // 1、buildConsultData();
        // 2、sendMsg();
    }

    /**
     * 发送微信订阅消息
     */
    @Async
    public void sendSubscribeMsg(String access_token, Consult item, Map<String, String> configMap) {
        // 此处编写发送微信订阅消息业务
        // 1、buildConsultData();
        // 2、sendSubscribeMsg();
    }

    /**
     * 数据并上报
     */
    @Async("dataCollectionExecutor")
    public void buildAndPostData(String access_token, Consult item, Map<String, String> configMap) {
        // 此处编写上报业务，如拼接数据，然后执行上报。
        // 1、buildConsultData();
        // 2、postData();
    }
 }
```

PS：

1）、以上是代码片段，个人经验认为专门定义一个异步service存放各个异步方法最佳，这样可以避免编码时一些误操作比如异步方法不是void或者是private修饰，导致@Async注解失效的情况，同时可以安排每个注解指向不同的自定义线程池更加灵活；

2）、@Async注解中的名称就是上面定义的自定义线程池名称，这样业务执行时就会从指定线程池中获取异步线程。



### 异步批量上报数据



```java
 @Autowired
 private AsyncService asyncService;

 /**
  * 手动上报问诊记录，线程池方式。
  */
 public void manualUploadConsultRecordsAsync(String channel, Date startTime, Date endTime) {

     // 查询指定时间内的问诊记录
    List<Consult> consultList = consultService
        .findPaidListByChannelAndTime(channel, startTime, endTime, configMap.get("serviceId"));

    if (!CollectionUtils.isEmpty(consultList)) {

        log.debug("[SendWZDataService][manualUploadConsultRecordsAsync]>>>> 手动上报问诊记录, 一共[{}]条", consultList.size());

        consultList.forEach((item) -> {
            try {
                // 异步调用，使用线程池。
                asyncService.buildAndPostData(access_token, item, configMap);
            } catch (Exception ex) {
                log.error("[SendWZDataService][manualUploadConsultRecordsAsync]>>>> 手动上报问诊记录发生异常: ", ex);
            }
        });

    }
 }
```



### 总结

以上方式已经在生产环境运行，在工作时间内执行过很多次，一次数万条记录基本是几分钟内就全部上报完毕，而正常循环遍历时一次大概需要半个小时左右。

线程池的使用方式往往来源于业务场景，如果类似的业务不存在紧急处理的情况，大体还是以任务调度执行为主，因为更安全。如果存在紧急处理的情况，那么使用SpringBoot+线程池的方式不仅能节省非常多的时间，且不占用主线程的执行空间。



**END**


# 附录
## A 资源
## B 参考资料

