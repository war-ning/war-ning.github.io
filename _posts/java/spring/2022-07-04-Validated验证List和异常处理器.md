---
layout: articles
title: Validated验证List和异常处理器
tags:  Valid Validated
author: Warning
key:    java-spring-head-08
aside:
  toc: true
sidebar:
nav: Java
category: [java, spring]
---

经常遇到一些参数需要验证，用@validated 的分组验证方式很棒，可以解决大量的[冗余]代码，提升美观性。但是我们平时会遇到入参为list的校验
@validated 的分组验证就无法使用了，非常麻烦，各大搜索引擎一查，总结一下比较推荐的方法：



<!--more-->



------


### 反面例子：

```java
@***Mapping("/**")
public *** apiName(@RequestBody @Validated(Add.class) List<AClass> aObject)
```

正常情况下这个例子是无法使用的，不是接口无法使用，是`@Validated无效。`这是因为你的入参实体是List，其内并没有调用AClass的@Valid，导致你的校验规则只校验的List本身，并不校验其内部实体。

###

### 解决方法：

在项目里添加一个`ValidList`类即可，此类通用，可以在全部由此需求的项目（`jdk1.8`）中添加，无需改动，有ValidList类之后只要将接口方法参数中的List改成ValidList即可：

```java
@***Mapping("/**")
public *** apiName(@RequestBody @Validated(Add.class) ValidList<Bean> aObject)
```

### `上代码：`

```java
package com.xiaobodata.biz.common.valid;

import lombok.Data;
import lombok.NoArgsConstructor;

import javax.validation.Valid;
import java.util.*;


/**
 * @author Warning
 * @since 2022-07-04 14:28
 */

@Data
@NoArgsConstructor
public class ValidList<E> implements List<E> {

    @Valid
    private List<E> list = new LinkedList<>();

    public ValidList(List<E> paramList) {
        this.list = paramList;
    }

    @Override
    public int size() {
        return list.size();
    }

    @Override
    public boolean isEmpty() {
        return list.isEmpty();
    }

    @Override
    public boolean contains(Object o) {
        return list.contains(0);
    }

    @Override
    public Iterator<E> iterator() {
        return list.iterator();
    }

    @Override
    public Object[] toArray() {
        return list.toArray();
    }

    @Override
    public <T> T[] toArray(T[] a) {
        return list.toArray(a);
    }

    @Override
    public boolean add(E e) {
        return list.add(e);
    }

    @Override
    public boolean remove(Object o) {
        return list.remove(o);
    }

    @Override
    public boolean containsAll(Collection<?> c) {
        return list.containsAll(c);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        return list.addAll(c);
    }

    @Override
    public boolean addAll(int index, Collection<? extends E> c) {
        return list.addAll(index, c);
    }

    @Override
    public boolean removeAll(Collection<?> c) {
        return list.removeAll(c);
    }

    @Override
    public boolean retainAll(Collection<?> c) {
        return list.retainAll(c);
    }

    @Override
    public void clear() {
        list.clear();
    }

    @Override
    public E get(int index) {
        return list.get(index);
    }

    @Override
    public E set(int index, E element) {
        return list.set(index, element);
    }

    @Override
    public void add(int index, E element) {
        list.add(index, element);
    }

    @Override
    public E remove(int index) {
        return list.remove(index);
    }

    @Override
    public int indexOf(Object o) {
        return list.indexOf(o);
    }

    @Override
    public int lastIndexOf(Object o) {
        return list.lastIndexOf(o);
    }

    @Override
    public ListIterator<E> listIterator() {
        return list.listIterator();
    }

    @Override
    public ListIterator<E> listIterator(int index) {
        return list.listIterator(index);
    }

    @Override
    public List<E> subList(int fromIndex, int toIndex) {
        return list.subList(fromIndex, toIndex);
    }
}
```

这样前端的代码是不需要任何改动的！

**这个时候有小伙伴说，需要捕获异常，是的，`@Validated是通过抛出异常来进行返回的。`**

### 上代码，全局异常处理：

```java
/**
 * 异常处理拦截器
 * @author Warning
 * @since 2022-07-04 14:28
 */

@RestControllerAdvice
public class GlobalException {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    /**
     * 参数为空异常处理
     *
     * @param ex ex
     * @return ReturnResult
     */
    @ExceptionHandler({
            ParamNullException.class, MethodArgumentNotValidException.class,
            ConstraintViolationException.class, HttpMessageNotReadableException.class,
            MissingServletRequestParameterException.class
    })
    public ReturnResult<?> requestMissingServletRequest(Exception ex) {
        ReturnResult<?> returnResult = new ReturnResult<>();
        returnResult.setRetCode(2);
        if (ex instanceof MissingServletRequestParameterException) {
            returnResult.setRetMsg("参数非法:" + ((MissingServletRequestParameterException) ex).getParameterName());
        } else if (ex instanceof HttpMessageNotReadableException) {
            //排除入参问题
            returnResult.setRetMsg("参数非法:" + ex.getMessage());
        } else if (ex instanceof MethodArgumentNotValidException) {
            //排除入参问题
            FieldError error = (FieldError) ((MethodArgumentNotValidException) ex).getBindingResult().getAllErrors().get(0);
            returnResult.setRetMsg("参数非法:" + error.getField() + " " + error.getDefaultMessage());
        } else if (ex instanceof ConstraintViolationException) {
            //排除入参问题
            ConstraintViolation<?> violation = ((ConstraintViolationException) ex).getConstraintViolations().iterator().next();
            returnResult.setRetMsg("参数非法:" + violation.getPropertyPath().toString().split("[.]")[1]
                    + violation.getMessage());
        }
        return returnResult;
    }

    /**
     * 其他异常
     *
     * @param request request
     * @param ex      ex
     * @return ReturnResult
     */
    @ExceptionHandler(Exception.class)
    public ReturnResult<?> resolveException(HttpServletRequest request, Exception ex) {
        ReturnResult<?> returnResult = new ReturnResult<>();

        logger.error("==============异常开始=============");
        logger.error("url: " + request.getRequestURL() + " msg: " + ex.getMessage());
        ex.printStackTrace();
        logger.info("==============异常结束=============");

        returnResult.setRetCode(1);
        returnResult.setRetMsg("GlobalException!!!");
        returnResult.setRetData(ex.toString());
        return returnResult;
    }
}
```



ReturnResult是自己定一的方法返回封装类，根据自己的项目自己封装就行



**END**


# 附录
## A 资源
## B 参考资料

