---
layout: articles
title: 使用RestTemplate的@LoadBalanced注解使用报错
tags:  微服务
author: WarNing
key:    java-advance-head-33
aside:
  toc: true
sidebar:
nav: Java
category: [java, advance]
---

记一次使用RestTemplate时报错java.lang.IllegalStateException: No instances available for 127.0.0.1

<!--more-->

## 报错原因


在RestTemplate的配置类里使用了 @LoadBalanced

```java
@Component
public class RestTemplateConfig {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}

```

或者

![](https://gitee.com/war-ning/picture/raw/master/blog/20220318100515.png)



再调用

> @Autowired
>
> private RestTemplate restTemplate;



必须使用应用名作为代替ip:端口，
`http://127.0.0.1:8080/user/get`改成`http://应用名/user/get`
不然会报错



使用RestTemplate时报错java.lang.IllegalStateException: No instances available for 127.0.0.1

1. 不要使用ip+port的方式访问，取而代之的是应用名
2. 这种方式发送的请求都会被ribbon拦截，ribbon从eureka注册中心获取服务列表，然后采用均衡策略进行访问


---

下面贴另一篇对于@LoadBalanced注解的帖子

源帖地址 > [深入理解@LoadBalanced注解的实现原理与客户端负载均衡](https://copyfuture.com/blogs-details/20191114194436253rmip8gwhn7ut0ix)


## @LoadBalanced注解概述


在使用springcloud ribbon客户端负载均衡的时候，可以给RestTemplate bean 加一个@LoadBalanced注解，就能让这个RestTemplate在请求时拥有客户端负载均衡的能力,先前有细嚼过但是没有做过笔记,刚好处理此类问题记录下

### @LoadBalanced

```java
/**
* 注释将RestTemplate bean标记为配置为使用LoadBalancerClient。
*/
@Target({ ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Qualifier
public @interface LoadBalanced {
}
```

通过源码可以发现这是一个`LoadBalanced`标记注解并且标记了`@Qualifier`(基于Spring Boot的自动配置机制),我们可以溯源到`LoadBalancerAutoConfiguration`

#### LoadBalancerAutoConfiguration

```java
  /**
         * 功能区的自动配置（客户端负载平衡）
         */
        @Configuration
        @ConditionalOnClass(RestTemplate.class)
        @ConditionalOnBean(LoadBalancerClient.class)
        @EnableConfigurationProperties(LoadBalancerRetryProperties.class)
        public class LoadBalancerAutoConfiguration {
            @LoadBalanced
            @Autowired(required = false)
            private List<RestTemplate> restTemplates = Collections.emptyList(); //这里持有@LoadBalanced标记的RestTemplate实例
            @Autowired(required = false)
            private List<LoadBalancerRequestTransformer> transformers = Collections.emptyList();

            @Bean
            public SmartInitializingSingleton loadBalancedRestTemplateInitializerDeprecated(final ObjectProvider<List<RestTemplateCustomizer>> restTemplateCustomizers) {
                return () -> restTemplateCustomizers.ifAvailable(customizers -> {
                    for (RestTemplate restTemplate : LoadBalancerAutoConfiguration.this.restTemplates) {
                        for (RestTemplateCustomizer customizer : customizers) {
                            //为restTemplate添加定制
                            customizer.customize(restTemplate);
                        }
                    }
                });
            }
            // ...

            /**
             * 以下针对classpath存在RetryTemplate.class的情况配置，先忽略
             */
            @Configuration
            @ConditionalOnClass(RetryTemplate.class)
            public static class RetryAutoConfiguration {
                @Bean
                @ConditionalOnMissingBean
                public LoadBalancedRetryFactory loadBalancedRetryFactory() {
                    return new LoadBalancedRetryFactory() {
                    };
                }
            }
            // ...
        }
```

`@LoadBalanced`和`@Autowried`结合使用,意思就是这里注入的`RestTempate` Bean是所有加有`@LoadBalanced`注解标记的(持有`@LoadBalanced`标记的RestTemplate实例)

这段自动装配的代码的含义不难理解，就是利用了RestTempllate的拦截器，使用RestTemplateCustomizer对所有标注了@LoadBalanced的RestTemplate Bean添加了一个LoadBalancerInterceptor拦截器，而这个拦截器的作用就是对请求的URI进行转换获取到具体应该请求哪个服务实例ServiceInstance。

**关键问下自己：为什么？**

- RestTemplate实例是怎么被收集的？
- 怎样通过负载均衡规则获取具体的具体的server？

继续扒看源码>
上面可以看出，会`LoadBalancerAutoConfiguration类`对我们加上`@LoadBalanced`注解的bean 添加`loadBalancerInterceptor`拦截器

#### LoadBalancerInterceptor

```java
    /**
         * 功能区的自动配置（客户端负载平衡）。
         */
        public class LoadBalancerInterceptor implements ClientHttpRequestInterceptor {
            private LoadBalancerClient loadBalancer;
            private LoadBalancerRequestFactory requestFactory;
            public LoadBalancerInterceptor(LoadBalancerClient loadBalancer,
                                           LoadBalancerRequestFactory requestFactory) {
                this.loadBalancer = loadBalancer;
                this.requestFactory = requestFactory;
            }
            public LoadBalancerInterceptor(LoadBalancerClient loadBalancer) {
// for backwards compatibility
                this(loadBalancer, new LoadBalancerRequestFactory(loadBalancer));
            }
            @Override
            public ClientHttpResponse intercept(final HttpRequest request, final byte[] body,
                                                final ClientHttpRequestExecution execution) throws IOException {
                final URI originalUri = request.getURI();
                String serviceName = originalUri.getHost();
                Assert.state(serviceName != null,
                        "Request URI does not contain a valid hostname: " + originalUri);
                return this.loadBalancer.execute(serviceName,
                        this.requestFactory.createRequest(request, body, execution));
            }
        }
```

重点看intercept方法 当我们restTemplate执行请求操作时，就会被拦截器拦截进入intercept方法,而loadBalancer是LoadBalancerClient的具体实现

#### RibbonLoadBalancerClient

```java
 public <T> T execute(String serviceId, LoadBalancerRequest<T> request, Object hint)
throws IOException {
            ILoadBalancer loadBalancer = getLoadBalancer(serviceId);
            Server server = getServer(loadBalancer, hint);
            if (server == null) {
                throw new IllegalStateException("No instances available for " + serviceId);
            }
            RibbonServer ribbonServer = new RibbonServer(serviceId, server,
                    isSecure(server, serviceId),
                    serverIntrospector(serviceId).getMetadata(server));
            return execute(serviceId, ribbonServer, request);
        }
```

看到这里相信都遇到过类似的错误，恍然大悟

```
No instances available for xxxxx
```

### 总结

- 1.根据serviceId 获取对应的loadBalancer
- 2.根据loadBalancer获取具体的server（这里根据负载均衡规则，获取到具体的服务实例）
- 3.创建RibbonServer
- 4.执行具体请求

这里

> 注意: @LoadBalanced 标记注解获取到最后通过负载均衡规则获取具体的具体的server来发起请求

### 案例

```java
/**
         * 服务注册中心配置
         *
         * @author <a href="mailto:shangzhi.ibyte@gmail.com">iByte</a>
         * @since 1.0.1
         */
        @Configuration
        @EnableConfigurationProperties(ModuleMappingHelper.class)
        public class DiscoveryConfig {
            @Autowired
            Environment environment;
            /**
             * DiscoveryHeaderHelper默认bean
             * @return
             */
            @Bean
            public DiscoveryHeaderHelper discoveryHeaderHelper() {
                DiscoveryHeaderHelper discoveryHeaderHelper = new DiscoveryHeaderHelper(environment);
                DiscoveryHeaderHelper.INSTANCE = discoveryHeaderHelper;
                return discoveryHeaderHelper;
            }
            /**
             * resttemplate构建
             */
            @Resource
            private RestTemplateBuilder restTemplateBuilder;
            /**
             * resttemplate请求bean,更改系统本身的builder
             * @return
             */
            @Bean
            @LoadBalanced
            public RestTemplate restTemplate() {
                RestTemplate restTemplate = restTemplateBuilder.configure(new RestTemplate());
//RestTemplate interceptors 远程调用请求增加头部信息处理
                restTemplate.getInterceptors().add(new RestApiHeaderInterceptor());
//RestTemplate Set the error handler 错误处理
                restTemplate.setErrorHandler(new RestResponseErrorHandler());
                return restTemplate;
            }
            @Bean
            public DiscoveryClient.DiscoveryClientOptionalArgs discoveryClientOptionalArgs() {
                DiscoveryClient.DiscoveryClientOptionalArgs discoveryClientOptionalArgs = new DiscoveryClient.DiscoveryClientOptionalArgs();
                discoveryClientOptionalArgs.setAdditionalFilters(Collections.singletonList(new DiscoveryHeaderClientFilter()));
                discoveryClientOptionalArgs.setEventListeners(Collections.singleton(new EurekaClientEventListener()));
                return discoveryClientOptionalArgs;
            }
        }
```

源码地址 > [DiscoveryConfig](https://gitee.com/ibyte/M-Pass/blob/master/framework/framework-discovery-api/src/main/java/com/ibyte/framework/discovery/DiscoveryConfig.java)


# 附录
## A 资源
## B 参考资料

