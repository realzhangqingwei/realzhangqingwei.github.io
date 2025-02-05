---
layout:     post
title:     Vscode两步搭建图床
subtitle:   
date:       2022-04-17
author:     张庆伟
header-img: img/gje.jpg
catalog: true
tags:
    - Vscode
    - 扩展
---
> [https://mp.weixin.qq.com/s/2HnFrcwxHr5g2VvRB2N_JQ]()

## 一、最终效果

VS Code中，可以 **实现图片的一键上传和引用返回** 。免费图床，简单好用～

## 二、写在前面

之前用的某个图床( **就不说名字了** )，今天支持明天不支持，写的博客直接废了，让博主痛不欲生。

有的网站，强制给本地上传的图片加水印，丑死了。

痛定思痛决定搭建个github的图床，没想到这么简单就搞定了～

### 1. VScode插件PicGo

需要用到Picgo这个插件，直接在vscode中搜索安装就行。

### 2. PicGo的Github配置

打开设置，找到extensions中的Picgo的设置

![图片](https://mmbiz.qpic.cn/mmbiz_png/KxvDktg1OnYrV4ia3Uxo26AwBCbL6tFmlKC9PIGWnyGFpV18LGJNtewA2nRWCn323sGIpXNzyTVlt216rIqU87Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

往下拖找到github，进行配置:

![图片](https://mmbiz.qpic.cn/mmbiz_png/KxvDktg1OnYrV4ia3Uxo26AwBCbL6tFmlnzohrdyQge2ibaJFKWfLBeEzRtlnv5s6wYbjB8TxorASEMe2MobBxgw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

首先是PicGo-Core设为

```
github
```

Branch自己用git命令

```
`git branch -a`
```

查看自己的branch，我就一个master，所以填的master

Path要和Repo搭配使用，到时候上传的图片就会到**Repo+Path**目录下，比如我的会传到：

```
`wyl6/wyl6.github.io/imgs_for_blogs/`
```

下面，所以并不需要单独开个repo，加个目录就行。

注意，Path后面加"/"，不然后面一部分会当成图片名称合并进去。比如Path设为：

```
`imgs_for_blogs/hello`
```

那么上传world.png时图片会成为：

```
`imgs_for_blogs/helloworld.png`
```

最后一步的token，从github上重新生成一个就行(token只出现一次，之前的几乎找不到)。首先从头像那打开设置:

![图片](https://mmbiz.qpic.cn/mmbiz_png/KxvDktg1OnYrV4ia3Uxo26AwBCbL6tFml7T3sVWS3rL22SvWgTe1Qab9FnUhg71yEhYWx0GaKRzBR4wuuSL3W1g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

找到最下面的Developer Settings:

![图片](https://mmbiz.qpic.cn/mmbiz_png/KxvDktg1OnYrV4ia3Uxo26AwBCbL6tFmlBKkSuKqZDdoY9SVlePM1eUpy8DQYepTUnwkUuBibRMLIGhxDzY9gsNA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

从Personal Access里生成一个token，可以备份下次使用：![图片](https://mmbiz.qpic.cn/mmbiz_png/KxvDktg1OnYrV4ia3Uxo26AwBCbL6tFmlGoOAd8LfCZdux42AjYZSNVfydKcTBOP5U7qWKqomT1xYZRHUju5Uxg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

最后把token复制到PicGo设置里的Github Token里就行。然后就可以在vscode里使用图床了。

### 3.图床命令

三个简要命令：

#### (1)、 `ctrl+alt+e`

从文件目录手动插入图片

#### (2)、`ctrl+alt+u`


从剪贴板插入图片

#### (3)、`ctrl+alt+o`

从输入目录插入图片，相对目录和绝对目录都行

#### (4)、 常用快捷键如下

> 从命令看，说一键不太准确，也就两三键吧。
> ![20220417103853](https://raw.githubusercontent.com/realzhangqingwei/realzhangqingwei.github.io/master/imgs_for_blogs/20220417103853.png)

### 四、  参考链接

[[https://mp.weixin.qq.com/s/2HnFrcwxHr5g2VvRB2N_JQ]()]()
