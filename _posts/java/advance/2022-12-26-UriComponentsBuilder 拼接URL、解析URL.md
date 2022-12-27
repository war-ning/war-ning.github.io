---
layout: articles
title: UriComponentsBuilder 拼接URL、解析URL
tags:  UriComponentsBuilder url
author: Warning
key:    java-advance-head-35
aside:
  toc: true
sidebar:
nav: Java
category: [java, advance]
---

最近在频繁的处理URL的拼接, 使用Spring提供的网址拼接工具类:UriComponentsBuilder

拼接操作会稍微轻松一点(虽然还是体力活) . . .

<!--more-->



------

https://mp.csdn.net/edit)

# UriComponentsBuilder 拼接URL、解析URL





# 前言

- 关于URI参考[这里](https://blog.csdn.net/sayyy/article/details/116723775)。
- springboot 2.1.1.RELEASE



# UriComponentsBuilder

- UriComponentsBuilder 是UriBuilder的实现。针对Servlet，还派生出来ServletUriComponentsBuilder。
- 使用过UriComponentsBuilder 的都知道，很好用



# 快速来个示例



```java
String url = UriComponentsBuilder.fromUriString("http://mydomain/api/getToken")
									.queryParam("appid", "123")
									.queryParam("appsecret", "secret123")
									.build().encode().toString();
System.out.println(url);
```

- 这段代码可以得到：`http://mydomain/api/getToken?appid=123&appsecret=secret123`

- 这段代码实际做了几件事：

  1. 实例化 UriComponentsBuilder 。`UriComponentsBuilder.fromUriString("http://mydomain/api/getToken")`

  2. 设置 UriComponentsBuilder 实例。`.queryParam("appid", "123").queryParam("appsecret", "secret123")`

  3. 创建 UriComponents 实例。`.build()`

  4. 设置 UriComponents 实例。`.encode()`

  5. UriComponents 实例转成字符串。`.toString()`



# 实例化 UriComponentsBuilder 的方法

UriComponentsBuilder 提供的实例化 UriComponentsBuilder 的方法

```java
public static UriComponentsBuilder newInstance();
public static UriComponentsBuilder fromPath(String path);
public static UriComponentsBuilder fromUri(URI uri);
public static UriComponentsBuilder fromUriString(String uri);
public static UriComponentsBuilder fromHttpUrl(String httpUrl);
public static UriComponentsBuilder fromHttpRequest(HttpRequest request);
public static UriComponentsBuilder fromOriginHeader(String origin);

```

ServletUriComponentsBuilder 提供的实例化 UriComponentsBuilder 的方法

```java
public static ServletUriComponentsBuilder fromContextPath(HttpServletRequest);
public static ServletUriComponentsBuilder fromServletMapping(HttpServletRequest);
public static ServletUriComponentsBuilder fromRequest(HttpServletRequest);
public static ServletUriComponentsBuilder fromCurrentContextPath();
public static ServletUriComponentsBuilder fromCurrentServletMapping();
public static ServletUriComponentsBuilder fromCurrentRequest();

```



# 设置 UriComponentsBuilder 实例

##### 一. 赋值

> .queryParam("appid", "123")



##### 二 . 重复赋值

> .queryParam("appid", "123")
> .queryParam("appid", "234")

- appid 会变为[“123”, “234”]

##### 三. 替换

> .queryParam("appid", "123")
> .replaceQueryParam("appid", "234")

- appid 变为234

##### 四. 路径

> .path("a")
> .path("b")
> .path("c")

- xxx/a/b/c

##### 五. 替换路径

> .path("a")
> .path("b")
> .path("c")
> .replacePath("d")

- xxx/d





# 关于urlencode

前面示例中`.build()` 中得到的是 UriComponents 派生类（`HierarchicalUriComponents`）的实例。

HierarchicalUriComponents 类的实例在`.encode()`时，使用`HierarchicalUriComponents.encodeUriComponent()` 方法进行 url 编码。其编码方式与`java.net.URLEncoder`的实现不同。

总结一下（代码参考`测试代码1`）：

| 代码                                                         | 结果                | 特征                    | 遵循协议                                                     |
| ------------------------------------------------------------ | ------------------- | ----------------------- | ------------------------------------------------------------ |
| URLEncoder.encode(“1 + 2 = 3”, “utf-8”);                     | 1+%2B+2+%3D+3       | 空格转义成+；+转义成%2B | [W3C标准规定](http://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1) |
| HierarchicalUriComponents.encodeUriComponent(“1 + 2 = 3”, “utf-8”,Type.QUERY_PARAM); | 1%20+%202%20%3D%203 | 空格转义成%20；+不变    | [RFC 3986](http://www.ietf.org/rfc/rfc3986.txtt)             |

用哪个对呢？都对！！！但，要看提供服务的应用支持哪个。我的建议是，都试试，哪个能用用哪个。

#### 如何让 URLEncoder 支持 `空格转义成%20；+不变`呢？

参考这里：https://blog.csdn.net/qq_43566496/article/details/84935364

#### 如何让 HierarchicalUriComponents 支持 `空格转义成+；+转义成%2B`呢？

##### 方法1

>  `.build(true)` 告诉 `UriComponents` 我已经做过uri编码了，你就不要再做uri编码了。

```java
String url = UriComponentsBuilder.fromUriString("http://mydomain/api/getToken")
									.queryParam("appid", "123")
									.queryParam("appsecret", "secret123")
									.queryParam("q", URLEncoder.encode("1 + 2 = 3"))
									.build(true).encode().toString();
System.out.println(url);

```



##### 方法2 :

>  去掉`.encode()` ， `UriComponents` 就不要再做uri编码了。

```java
String url = UriComponentsBuilder.fromUriString("http://mydomain/api/getToken")
									.queryParam("appid", "123")
									.queryParam("appsecret", "secret123")
									.queryParam("q", URLEncoder.encode("1 + 2 = 3"))
									.build().toString();
System.out.println(url);

```



#### spring 提供的支持 RFC 2396、RFC 3986 的类： UriUtils

https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/util/UriUtils.html



```java
String url = UriComponentsBuilder.fromUriString("http://api.fanyi.baidu.com/api/trans/vip/translate")
									.queryParam("appid", appid)
									.queryParam("q", encode(query))
									.queryParam("from", from)
									.queryParam("to", to)
									.queryParam("salt", salt)
									.queryParam("sign", sign)
									.build(true).encode().toString();
```



**END**


# 附录
## A 资源
## B 参考资料

