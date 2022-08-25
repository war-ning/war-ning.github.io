---
layout: articles
title: Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床
tags:  PicGO Typora
author: Warning
key:    linux-head-6
aside:
toc: true
sidebar:
nav: linux
category: [linux]
---



<!--more-->



------


# Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床



**转载** [Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_mb5fed6fc050005的技术博客_51CTO博客](https://blog.51cto.com/u_15072910/4145011)

## 软件配置

### 1. Jopin配合Typora才是最佳搭档, 配置如下

工具–>选项

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_网络](https://s2.51cto.com/images/blog/202108/12/adff285a9f52e6242e7d1896011743ba.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 2. 验证

新建笔记, 然后`ctrl + E`快捷键调用Typora编辑器来使用

编写完成后,关闭Typora创建, 还要记得关闭调用， 点击”停止“

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_git_02](https://s2.51cto.com/images/blog/202108/12/8848e5bd7595fab51faddc24444d5177.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 1. node 环境准备

下载安装

```html
https://nodejs.org/dist/v14.17.4/node-v14.17.4-x64.msi
```

## 2. 使用 node 安装 PicGo-Core

```html
// npm 命令执行速度过慢的话，我们可以使用一下淘宝的镜像
npm install -g picgo --registry=https://registry.npm.taobao.org

// 安装完成以后测试一下是否安装成功
picgo -v

```

## 3. 使用 picgo 命令安装 gitee-uploader 插件

```html
picgo install gitee-uploader
picgo install super-prefix
```

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_配置文件_03](https://s2.51cto.com/images/blog/202108/12/bdc3b51485d6990056a7a5d6a41baab8.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 4.创建 Gitee账号和令牌

建议选择国内网站,这样网速有保证
(假设已经创建了Gitee的账号)
![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_网络_04](https://s2.51cto.com/images/blog/202108/12/cb169f088a5969e3b225fb35f1cbef0f.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_网络_05](https://s2.51cto.com/images/blog/202108/12/43ef9d87b39433e2279be9941cc4e87a.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

设置私人令牌

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_上传_06](https://s2.51cto.com/images/blog/202108/12/8c91b688fc753e5731ea92a519b0e79a.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_网络_07](https://s2.51cto.com/images/blog/202108/12/17e2075008a5112c7c7f580127a1b072.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_配置文件_08](https://s2.51cto.com/images/blog/202108/12/bc614aa8ccd8d6433feb57e8338d719b.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_git_09](https://s2.51cto.com/images/blog/202108/12/1033764cb58f4080fefc266ffb6df3b3.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

> 复制保存好自己的私人令牌

## 5.Typora设置上传服务

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_网络_10](https://s2.51cto.com/images/blog/202108/12/649f15f4b51fdb5acbfbb751509fc3eb.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

点`打开配置文件`

## 6. 完整的配置文件

**以下是参照 [ PicGo-Core官方文档](https://picgo.github.io/PicGo-Core-Doc/zh/guide/config.html) 的进行的配置**

首先我们需要找到我们的配置文件，**picgo** 的默认配置文件在不同系统的目录不太一样：
linux 和 macOS 均为 `~/.picgo/config.json`
windows 则为 `C:\Users\{你的用户名}\.picgo\config.json`

```html
{
  "picBed": {
    "current": "gitee",
    "gitee": {
      "repo": "ilangel/markdown-images",
      "branch": "master",
      "token": "74068fdb49c84fccb4d92481d1e3cdb9",
      "path": "img",
      "customPath": "",
      "customUrl": "https://gitee.com/ilangel/markdown-images/raw/master/"
    },
    "uploader": "gitee",
    "transformer": "path"
  },
  "picgoPlugins": {
    "picgo-plugin-gitee-uploader": true,
	   "picgo-plugin-super-prefix": true
  },
  "picgo-plugin-gitee-uploader": {
    "lastSync": "2021-08-11 12:48:27"
  },
  "picgo-plugin-super-prefix": {
    "fileFormat": "YYYYMMDD-HHmmss"
  } //super-prefix插件配置
}
```

**注意：配置完成后可以点击 `验证图片上传选项` 来测试是否配置成功**

## 7.对本地图片上传

对位于本地的所有图片，右键点击上传图片。

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_git_11](https://s2.51cto.com/images/blog/202108/12/f4f80d41fd2d7a48d4ce5301d8e23dcc.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

当然也可以批量上传

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_git_12](https://s2.51cto.com/images/blog/202108/12/9a04b1a32c766f51b292e3db4064a5cc.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 8. 解决 `文件大于1M，登录后可见` 的问题

> 按照步骤 1-6 我们确实成功地配置了一个免费好用的 `Gitee图床`，简单使用也没有什么问题。可是当我们上传的图片大小超过 1M 后：OMG，图片无法正常显示，在浏览器中打开图片的地址，直接跳转到 Gitee 登录界面，并且出现出现了很扎心的 `文件大于1M，登录后可见` 文字的提示。关键是这个文件大小限制还没有办法解决，凉凉！！！

**凉凉？不存在的！** 俗话说：办法总比困难多。我们访问 git 仓库中文件的方式并不是只有一种，更何况它只是一些`静态的资源`文件。所以是不是只要我们想办法配置一个简单的HTTP服务就可以了。问题迎刃而解：Gitee 官方给我们提供了一种供博客 / 门户 / 开源项目网站 / 开源项目静态效果演示用途的 `Git Pages`服务。

### 8.1 开启 Git Pages 服务

1. 进入到阁下 Gitee 图床 所在仓库的页面，找到 `服务` -> `Gitee Pages`

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_git_13](https://s2.51cto.com/images/blog/202108/12/02d9f28dab002a666014f4ecc273bb99.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

1. 无需修改任何配置。直接点击 `启动`按钮，等待服务启动完毕即可。

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_配置文件_14](https://s2.51cto.com/images/blog/202108/12/eaa152b702ae34d9399844954186fee8.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 8.2 更新图片访问的路径

**当我们的 图床仓库 开启 Git Pages 服务后，就会得到一个专属的网站地址，格式为：“ `个人空间地址`.gitee.io/`仓库名`”** 。

例如：`http://zi1.gitee.io/pic`，则我们访问该图床中的静态资源文件的路径为 `http://zi1.gitee.io/pic` + *仓库中文件的可见路径*。

比如：你的仓库中的 picture 目录下的 1.jpg 的图片文件： `picture/1.jpg`，则我们访问该图片的路径为：`http://zi1.gitee.io/pic/picture/1.jpg`

![Typora+PicGo-Core(command line)+Gitee 实现上传图片到图床_配置文件_15](https://s2.51cto.com/images/blog/202108/12/87b1ff668ad1de1f348d72e1349aa0c1.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 9. 开启 Git Pages 后完整的配置文件

```html
{
  "picBed": {
    "current": "gitee",
    "gitee": {
      "repo": "ilangel/markdown-images",
      "branch": "master",
      "token": "74068fdb11114fccb4d92481d111111b9",
      "path": "img",
      "customPath": "",
      "customUrl": "https://gitee.com/ilangel/markdown-images/raw/master/"
    },
    "uploader": "gitee",
    "transformer": "path"
  },
  "picgoPlugins": {
    "picgo-plugin-gitee-uploader": true,
    "picgo-plugin-super-prefix": true
  },
  "picgo-plugin-gitee-uploader": {
    "lastSync": "2021-08-11 02:45:59"
  },
  "picgo-plugin-super-prefix": {
    "fileFormat": "YYYY-MM-DD_HH:mm:ss"
  } //super-prefix插件配置
}
```



**END**


# 附录
## A 资源
## B 参考资料

