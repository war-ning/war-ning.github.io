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



<!--more-->



------



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

##### 赋值

```
.queryParam("appid", "123")
```

##### 重复赋值

```
.queryParam("appid", "123")
.queryParam("appid", "234")
```

- appid 会变为[“123”, “234”]

##### 替换

```
.queryParam("appid", "123")
.replaceQueryParam("appid", "234")
```

- appid 变为234

##### 路径

```
.path("a")
.path("b")
.path("c")
```

- xxx/a/b/c

##### 替换路径

```
.path("a")
.path("b")
.path("c")
.replacePath("d")
```

- xxx/d

##### 更多

略。

# 关于urlencode

前面示例中`.build()` 中得到的是 UriComponents 派生类（`HierarchicalUriComponents`）的实例。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210519112045555.png)
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

```java
String url = UriComponentsBuilder.fromUriString("http://mydomain/api/getToken")
									.queryParam("appid", "123")
									.queryParam("appsecret", "secret123")
									.queryParam("q", URLEncoder.encode("1 + 2 = 3"))
									.build(true).encode().toString();
System.out.println(url);
```

- `.build(true)` 告诉 `UriComponents` 我已经做过uri编码了，你就不要再做uri编码了。

##### 方法2

```java
String url = UriComponentsBuilder.fromUriString("http://mydomain/api/getToken")
									.queryParam("appid", "123")
									.queryParam("appsecret", "secret123")
									.queryParam("q", URLEncoder.encode("1 + 2 = 3"))
									.build().toString();
System.out.println(url);
```

- 去掉`.encode()` ， `UriComponents` 就不要再做uri编码了。

#### spring 提供的支持 RFC 2396、RFC 3986 的类： UriUtils

https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/util/UriUtils.html

# 测试代码1

```java
import java.io.ByteArrayOutputStream;
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.nio.charset.Charset;

import org.springframework.util.Assert;
import org.springframework.util.StreamUtils;
import org.springframework.util.StringUtils;

public class Test {
	public static void main(String[] args) throws UnsupportedEncodingException {
		System.out.println(URLEncoder.encode("1 + 2 = 3", "utf-8"));
		System.out.println(encodeUriComponent("1 + 2 = 3", Charset.forName("utf-8"), Type.QUERY_PARAM));
	}

	/*HierarchicalUriComponents.encodeUriComponent() 方法的高仿实现*/
	static String encodeUriComponent(String source, Charset charset, Type type) {
		if (!StringUtils.hasLength(source)) {
			return source;
		}
		Assert.notNull(charset, "Charset must not be null");
		Assert.notNull(type, "Type must not be null");

		byte[] bytes = source.getBytes(charset);
		boolean original = true;
		for (byte b : bytes) {
			if (!type.isAllowed(b)) {
				original = false;
				break;
			}
		}
		if (original) {
			return source;
		}

		ByteArrayOutputStream baos = new ByteArrayOutputStream(bytes.length);
		for (byte b : bytes) {
			if (type.isAllowed(b)) {
				baos.write(b);
			}
			else {
				baos.write('%');
				char hex1 = Character.toUpperCase(Character.forDigit((b >> 4) & 0xF, 16));
				char hex2 = Character.toUpperCase(Character.forDigit(b & 0xF, 16));
				baos.write(hex1);
				baos.write(hex2);
			}
		}
		return StreamUtils.copyToString(baos, charset);
	}

	enum Type {

		QUERY {
			@Override
			public boolean isAllowed(int c) {
				return isPchar(c) || '/' == c || '?' == c;
			}
		},
		QUERY_PARAM {
			@Override
			public boolean isAllowed(int c) {
				if ('=' == c || '&' == c) {
					return false;
				}
				else {
					return isPchar(c) || '/' == c || '?' == c;
				}
			}
		};

		/**
		 * Indicates whether the given character is allowed in this URI component.
		 * @return {@code true} if the character is allowed; {@code false} otherwise
		 */
		public abstract boolean isAllowed(int c);

		/**
		 * Indicates whether the given character is in the {@code ALPHA} set.
		 * @see <a href="http://www.ietf.org/rfc/rfc3986.txt">RFC 3986, appendix A</a>
		 */
		protected boolean isAlpha(int c) {
			return (c >= 'a' && c <= 'z' || c >= 'A' && c <= 'Z');
		}

		/**
		 * Indicates whether the given character is in the {@code DIGIT} set.
		 * @see <a href="http://www.ietf.org/rfc/rfc3986.txt">RFC 3986, appendix A</a>
		 */
		protected boolean isDigit(int c) {
			return (c >= '0' && c <= '9');
		}

		/**
		 * Indicates whether the given character is in the {@code sub-delims} set.
		 * @see <a href="http://www.ietf.org/rfc/rfc3986.txt">RFC 3986, appendix A</a>
		 */
		protected boolean isSubDelimiter(int c) {
			return ('!' == c || '$' == c || '&' == c || '\'' == c || '(' == c || ')' == c || '*' == c || '+' == c ||
					',' == c || ';' == c || '=' == c);
		}

		/**
		 * Indicates whether the given character is in the {@code unreserved} set.
		 * @see <a href="http://www.ietf.org/rfc/rfc3986.txt">RFC 3986, appendix A</a>
		 */
		protected boolean isUnreserved(int c) {
			return (isAlpha(c) || isDigit(c) || '-' == c || '.' == c || '_' == c || '~' == c);
		}

		/**
		 * Indicates whether the given character is in the {@code pchar} set.
		 * @see <a href="http://www.ietf.org/rfc/rfc3986.txt">RFC 3986, appendix A</a>
		 */
		protected boolean isPchar(int c) {
			return (isUnreserved(c) || isSubDelimiter(c) || ':' == c || '@' == c);
		}
	}
}


```

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



# 参考

https://www.cnblogs.com/zhengxl5566/p/10783422.html
https://www.cnblogs.com/zhengxl5566/p/13492602.html
https://www.cnblogs.com/siqi/p/10070926.html




**END**


# 附录
## A 资源
## B 参考资料

