---
layout: articles
title: SprinbBoot静态资源获取和文件上传
tags:  工具类
author: Warning
key:    java-spring-head-02
aside:
  toc: true
sidebar:
nav: Java
category: [java, spring]
---



> 今天新的项目用到了传文件，记录一下
>


<!--more-->


# 配置本地资源映射路径

## addResourceHandlers

> 实现 WebMvcConfigurer，重写 addResourceHandlers(ResourceHandlerRegistry registry)方法
addResourceHandler（） 添加的是访问路径
addResourceLocations（）添加的是映射后的真实路径，映射的真实路径末尾必须加 / ,
不然映射不到，这个问题困扰了我半天, / 适用于 windows和linux
如下:




```java
@SpringBootApplication
@ConditionalOnClass(SpringfoxWebMvcConfiguration.class)
public class SwaggerBootstrapUiDemoApplication  implements WebMvcConfigurer{
  @Value("${hongsen.other-file-root-dir-path:D:/acpfiles}")
  private String tempPath;
  @Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("swagger-ui.html").addResourceLocations("classpath:/META-INF/resources/");
    registry.addResourceHandler("doc.html").addResourceLocations("classpath:/META-INF/resources/");
    registry.addResourceHandler("/webjars/**").addResourceLocations("classpath:/META-INF/resources/webjars/");
    registry.addResourceHandler("/other/file/**").addResourceLocations("file:"+tempPath);
  }
}

```

## 上传文件

### 注意：
> transferTo()方法，只能使用一次。因为因http post文件流只可以接收读取一次，传输完毕则关闭流。
> 如需要保存两次，需要先存到本地，然后对文件进行复制移动等操作
```java

    /**
     * 上传文件
     *
     * @return
     */
    private String pcUploadFile(MultipartFile file, String type) throws IOException {
        String dirFilePath = otherFileRootDirPath + File.separator;
        String typePath = "";
        List<String> pathList = hongsenProperties.getOtherFilePathList();
        for (String s : pathList) {
            String[] split = s.split(";");
            if (null != split && split[0].equals(type)) {
                typePath = split[1];
                dirFilePath += typePath;
                break;
            }
        }
        // 获取文件目录
        String dir = getDir();
        dirFilePath += File.separator + dir;
        File dirFile = new File(dirFilePath);
        if (!dirFile.exists()) {
            dirFile.mkdirs();
        }
        String originalFilename = file.getOriginalFilename();
        if (StrUtil.isNotEmpty(originalFilename) && "fysp".equals(typePath)) {
            String filePath = dirFilePath + File.separator + originalFilename;
            File f = new File(filePath);
//            if (!f.exists()) {
//                f.createNewFile();
//            }
            file.transferTo(f);
            return "/other/file/" + typePath + "/" + dir + "/" + originalFilename;
        } else {
            //非服药视频
            String postfix = originalFilename.substring(originalFilename.lastIndexOf("."));
            String fileName = IdWorker.get32UUID() + postfix;
            String filePath = dirFilePath + File.separator + fileName;
            File f = new File(filePath);
//            if (!f.exists()) {
//                f.createNewFile();
//            }
            file.transferTo(f);
            // 这里是统一同other/file做文件静态资源过滤
            return "/other/file/" + typePath + "/" + dir + "/" + fileName;
        }
    }

    /**
     * 获取目录
     *
     * @return
     */
    private String getDir() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMM");
        return sdf.format(new Date());
    }

```
# 附录
## A 资源
## B 参考资料
https://www.cnblogs.com/lianxuan1768/p/12752482.html


