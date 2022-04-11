---
layout: articles
title: Mybatis-Plus实现真正的批量新增更新
tags:  Mybatis
author: Warning
key:    java-advance-head-36
aside:
  toc: true
sidebar:
nav: Java
category: [java, advance]
---

项目需要按每一天存一条数据,一次配置一年的
mybatis-plus的saveBatch实际上是循环插入
令人发指. . .

<!--more-->

这里重新添加方法,拼凑foreach循环,实现批量插入

### 1.添加InsertBatchMethod和UpdateBatchMethod类

```java

import com.baomidou.mybatisplus.core.injector.AbstractMethod;
import com.baomidou.mybatisplus.core.metadata.TableInfo;
import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.executor.keygen.NoKeyGenerator;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.mapping.SqlSource;

/**
 * 批量插入方法实现
 *
 * @author Warning
 * @since 2022-04-02 15:26
 */

@Slf4j
public class InsertBatchMethod extends AbstractMethod {
  @Override
  public MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {
    final String sql = "<script>insert into %s %s values %s</script>";
    final String fieldSql = prepareFieldSql(tableInfo);
    final String valueSql = prepareValuesSql(tableInfo);
    final String sqlResult = String.format(sql, tableInfo.getTableName(), fieldSql, valueSql);
    //log.debug("sqlResult----->{}", sqlResult);
    SqlSource sqlSource = languageDriver.createSqlSource(configuration, sqlResult, modelClass);
    return this.addInsertMappedStatement(mapperClass, modelClass, "insertBatch", sqlSource, new NoKeyGenerator(), null, null);
  }

  private String prepareFieldSql(TableInfo tableInfo) {
    StringBuilder fieldSql = new StringBuilder();
    fieldSql.append(tableInfo.getKeyColumn()).append(",");
    tableInfo.getFieldList().forEach(x -> fieldSql.append(x.getColumn()).append(","));
    fieldSql.delete(fieldSql.length() - 1, fieldSql.length());
    fieldSql.insert(0, "(");
    fieldSql.append(")");
    return fieldSql.toString();
  }

  private String prepareValuesSql(TableInfo tableInfo) {
    final StringBuilder valueSql = new StringBuilder();
    valueSql.append("<foreach collection=\"list\" item=\"item\" index=\"index\" open=\"(\" separator=\"),(\" close=\")\">");
    valueSql.append("#{item.").append(tableInfo.getKeyProperty()).append("},");
    tableInfo.getFieldList().forEach(x -> valueSql.append("#{item.").append(x.getProperty()).append("},"));
    valueSql.delete(valueSql.length() - 1, valueSql.length());
    valueSql.append("</foreach>");
    return valueSql.toString();
  }
}
```

```java
import com.baomidou.mybatisplus.core.injector.AbstractMethod;
import com.baomidou.mybatisplus.core.metadata.TableInfo;
import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.mapping.SqlSource;

/**
 * 批量更新方法实现,条件为主键,选择性更新
 *
 * @author Warning
 * @since 2022-04-02 15:27
 */

@Slf4j
public class UpdateBatchMethod extends AbstractMethod {
  @Override
  public MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {
    String sql = "<script>\n<foreach collection=\"list\" item=\"item\" separator=\";\">\nupdate %s %s where %s=#{%s} %s\n</foreach>\n</script>";
    String additional = tableInfo.isWithVersion() ? tableInfo.getVersionFieldInfo().getVersionOli("item", "item.") : "" + tableInfo.getLogicDeleteSql(true, true);
    String setSql = sqlSet(tableInfo.isWithLogicDelete(), false, tableInfo, false, "item", "item.");
    String sqlResult = String.format(sql, tableInfo.getTableName(), setSql, tableInfo.getKeyColumn(), "item." + tableInfo.getKeyProperty(), additional);
    //log.debug("sqlResult----->{}", sqlResult);
    SqlSource sqlSource = languageDriver.createSqlSource(configuration, sqlResult, modelClass);
    // 第三个参数必须和RootMapper的自定义方法名一致
    return this.addUpdateMappedStatement(mapperClass, modelClass, "updateBatch", sqlSource);
  }
}
```

### 2.添加自定义方法SQL注入器

```java
import com.baomidou.mybatisplus.core.injector.AbstractMethod;
import com.baomidou.mybatisplus.core.injector.DefaultSqlInjector;
import com.baomidou.mybatisplus.core.metadata.TableInfo;

import java.util.List;

/**
 * 方法SQL注入器
 *
 * @author Warning
 * @since 2022-04-02 15:28
 */

public class CustomizedSqlInjector extends DefaultSqlInjector {
  /**
   * 如果只需增加方法，保留mybatis plus自带方法，
   * 可以先获取super.getMethodList()，再添加add
   */
  @Override
  public List<AbstractMethod> getMethodList(Class<?> mapperClass, TableInfo tableInfo) {
    List<AbstractMethod> methodList = super.getMethodList(mapperClass, tableInfo);
    methodList.add(new InsertBatchMethod());
    methodList.add(new UpdateBatchMethod());
    return methodList;
  }
}
```

### 3.注入配置

```java
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * MybatisPlus配置类
 *
 * @author Warning
 * @since 2022-04-02 15:30
 */
//com/xiaobodata/biz/modules/**/xml/*Mapper.xml
@MapperScan("com.xiaobodata.biz.modules.**.mapper")
@Configuration
public class MyBatisPlusConfig {
  @Bean
  public CustomizedSqlInjector customizedSqlInjector() {
    return new CustomizedSqlInjector();
  }
}
```

### 4.添加通用mapper

```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;

/**
 * 根Mapper，给表Mapper继承用的，可以自定义通用方法
 *
 * @author Warning
 * @since 2022-04-02 15:33
 */

public interface RootMapper<T> extends BaseMapper<T> {
  /**
   * 自定义批量插入
   * 如果要自动填充，@Param(xx) xx参数名必须是 list/collection/array 3个的其中之一
   */
  int insertBatch(@Param("list") List<T> list);

  /**
   * 自定义批量更新，条件为主键
   * 如果要自动填充，@Param(xx) xx参数名必须是 list/collection/array 3个的其中之一
   */
  int updateBatch(@Param("list") List<T> list);
}

```

### 5.如何使用

```java
@Repository
public interface UserInfoMapper extends RootMapper<UserInfo> {
}

public interface UserInfoService extends IService<UserInfo> {
  int saveAll();
  int updateAll();
}

  @Service
  public class UserInfoServiceImpl extends ServiceImpl<UserInfoMapper, UserInfo> implements UserInfoService{

    @Override
    public int saveAll() {
      List<UserInfo> list = new ArrayList<>();
      for (int i = 0; i < 20; i++) {
        UserInfo userInfo = new UserInfo();
        userInfo.setUserName("厉害" + i);
        userInfo.setSalt(RandomStringUtils.randomAlphabetic(6));
        userInfo.setPassword(SecureUtil.sha256("123456" + userInfo.getSalt()));
        userInfo.setSex(0);
        userInfo.setAvatar(LoginServiceImpl.AVATAR);
        list.add(userInfo);
      }
      return baseMapper.insertBatch(list);
    }

    @Override
    public int updateAll() {
      List<UserInfo> userInfos = baseMapper.selectList(Wrappers.<UserInfo>lambdaQuery().between(BaseEntity::getId, 43, 62));
      userInfos.forEach(userInfo -> {
        userInfo.setUserName("更新了" + IdUtil.simpleUUID());
      });
      return baseMapper.updateBatch(userInfos);
    }
  }
```



# 附录
## A 资源
## B 参考资料

