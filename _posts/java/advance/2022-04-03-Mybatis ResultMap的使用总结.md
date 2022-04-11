---
layout: articles
title: Mybatis ResultMap的使用总结
tags:  Mybatis
author: Warning
key:    java-advance-head-37
aside:
  toc: true
sidebar:
nav: Java
category: [java, advance]
---

其实本身不是很喜欢用到ResultMap去映射
感觉这玩意让代码的可读性明显下降
但有些特殊场景不得不说用着真香啊


<!--more-->


resultMap是Mybatis最强大的元素，它可以将查询到的复杂数据（比如\**查询到\**几个表中数据）**映射**到一个结果集当中。

**resultMap包含的元素：**



```xml
<!--column不做限制，可以为任意表的字段，而property须为type 定义的pojo属性-->
<resultMap id="唯一的标识" type="映射的pojo对象">
  <id column="表的主键字段，或者可以为查询语句中的别名字段" jdbcType="字段类型" property="映射pojo对象的主键属性" />
  <result column="表的一个字段（可以为任意表的一个字段）" jdbcType="字段类型" property="映射到pojo对象的一个属性（须为type定义的pojo对象中的一个属性）"/>
  <association property="pojo的一个对象属性" javaType="pojo关联的pojo对象">
    <id column="关联pojo对象对应表的主键字段" jdbcType="字段类型" property="关联pojo对象的主席属性"/>
    <result  column="任意表的字段" jdbcType="字段类型" property="关联pojo对象的属性"/>
  </association>
  <!-- 集合中的property须为oftype定义的pojo对象的属性-->
  <collection property="pojo的集合属性" ofType="集合中的pojo对象">
    <id column="集合中pojo对象对应的表的主键字段" jdbcType="字段类型" property="集合中pojo对象的主键属性" />
    <result column="可以为任意表的字段" jdbcType="字段类型" property="集合中的pojo对象的属性" />
  </collection>
</resultMap>
```



**如果collection标签是使用嵌套查询，格式如下：**

```xml
 <collection column="传递给嵌套查询语句的字段参数" property="pojo对象中集合属性" ofType="集合属性中的pojo对象" select="嵌套的查询语句" >
 </collection>
```

**注意：<collection>标签中的column：要传递给select查询语句的参数，如果传递多个参数，格式为column= ” {参数名1=表字段1,参数名2=表字段2} ；**



**以下以实例介绍resultMap的用法：**



### 一、简单需求：一个商品的结果映射；

1、创建商品pojo对象：



```java
public class TShopSku  {
    /**
     * 主键ID
     */
    private Long id;

    /**
     * 商品名
     */
    private String skuName;

    /**
     * 分类ID
     */
    private Long categoryId;


    /**
     * 主键ID
     * @return ID
     */
    public Long getId() {
        return id;
    }

    /**
     * 主键ID,
     * @param id
     */
    public void setId(Long id) {
        this.id = id;
    }

    /**
     * 商品名
     * @return SKU_NAME 商品名
     */
    public String getSkuName() {
        return skuName;
    }

    /**
     * 商品名
     * @param skuName 商品名
     */
    public void setSkuName(String skuName) {
        this.skuName = skuName == null ? null : skuName.trim();
    }

    /**
     * 分类ID
     * @return CATEGORY_ID 分类ID
     */
    public Long getCategoryId() {
        return categoryId;
    }

    /**
     * 分类ID
     * @param categoryId 分类ID
     */
    public void setCategoryId(Long categoryId) {
        this.categoryId = categoryId;
    }
```



**对应的resultMap：**

```java
<resultMap id="BaseResultMap" type="com.meikai.shop.entity.TShopSku">
    <id column="ID" jdbcType="BIGINT" property="id" />
    <result column="SKU_NAME" jdbcType="VARCHAR" property="skuName" />
    <result column="CATEGORY_ID" jdbcType="BIGINT" property="categoryId" />
</resultMap>
```

### 二、商品pojo类添加属性集合：

一个商品会有一些属性，现在需要将查询出的商品属性添加到商品对象中，首先需要在原商品pojo类的基础上中添加属性的集合：



```java
    /**
     * 属性集合
     */
    private List<TShopAttribute> attributes;

    /**
     * 获得属性集合
     */
    public List<TShopAttribute> getAttributes() {
        return attributes;
    }

    /**
     * 设置属性集合
     * @param attributes
     */
     public void setAttributes(List<TShopAttribute> attributes) {
        this.attributes = attributes;
     }
```



**将Collection标签添加到resultMap中，这里有两种方式：**

**1、嵌套结果：**

**对应的resultMap：**



```xml
<resultMap id="BasePlusResultMap" type="com.meikai.shop.entity.TShopSku">
    <id column="ID" jdbcType="BIGINT" property="id" />
    <result column="SKU_NAME" jdbcType="VARCHAR" property="skuName" />
    <result column="CATEGORY_ID" jdbcType="BIGINT" property="categoryId" />
    <collection property="attributes" ofType="com.meikai.shop.entity.TShopAttribute" >
        <id column="AttributeID" jdbcType="BIGINT" property="id" />
        <result column="attribute_NAME" jdbcType="VARCHAR" property="attributeName" />
    </collection>
</resultMap>
```



**查询语句：**

```xml
<select id="getById"  resultMap="basePlusResultMap">
    select s.ID,s.SKU_NAME,s.CATEGORY_ID,a.ID,a.ATTRIBUTE_NAME
    from t_shop_sku s,t_shop_attribute a
    where s.ID =a.SKU_ID and s.ID = #{id,jdbcType =BIGINT};
</select>
```



**2、多层resultMap的嵌套：**

`<collection>`标签中,用resultMap去关联另外一个resultMap

```xml

    <resultMap id="factoryTreeModel" type="com.xiaobodata.biz.modules.canio.config.meter.entity.vo.FactoryTreeVO">
        <id column="factoryId" property="factoryId"/>
        <result column="factoryName" property="factoryName"/>
        <collection property="children" resultMap="processTreeModel"/>
    </resultMap>
    <resultMap id="processTreeModel" type="com.xiaobodata.biz.modules.canio.config.meter.entity.vo.ProcessTreeVO">
        <id column="processId" property="processId"/>
        <result column="processName" property="processName"/>
        <collection property="children" resultMap="deviceTreeModel"/>
    </resultMap>
    <resultMap id="deviceTreeModel" type="com.xiaobodata.biz.modules.canio.config.meter.entity.vo.DeviceTreeVO">
        <id column="deviceId" property="deviceId"/>
        <result column="deviceName" property="deviceName"/>
    </resultMap>
    <select id="deviceTree" resultMap="factoryTreeModel">
        SELECT
            factory.item_value AS factoryId,
            factory.item_text AS factoryName,
            process.processId AS processId,
            process.processName AS processName,
            device.id AS deviceId,
            device.device_name AS deviceName
        FROM
            sys_dict AS factoryDict
        LEFT JOIN sys_dict_item AS factory ON factoryDict.id = factory.dict_id
        LEFT JOIN device ON device.plant_name = factory.item_value
        LEFT JOIN (
            <!--查询工序字典-->
            SELECT
                item.item_value AS processId,
                item.item_text AS processName
            FROM sys_dict AS dict
                LEFT JOIN sys_dict_item AS item ON dict.id = item.dict_id
            WHERE dict.dict_code = 'PROCESS_STAGES'
                 ) AS process ON process.processId = device.process_stage
        WHERE factoryDict.dict_code = 'FACTORY'
            AND device.device_type = 'DEVICE_TYPE_1'
            AND device.process_stage IS NOT NULL
            AND device.plant_name IS NOT NULL
    </select>

```



[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)





# 附录
## A 资源
## B 参考资料
[Mybatis：resultMap的使用总结 - 偷酒的猫 - 博客园 (cnblogs.com)](https://www.cnblogs.com/kenhome/p/7764398.html)
