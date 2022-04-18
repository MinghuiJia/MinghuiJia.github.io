---
title: 基于jsDelivr的CDN图床搭建
date: 2022-04-18 21:27:21
tags:
  - jsDelivr
  - CDN图床
categories:
  - 教程
---

# 前言
我们将个人博客托管在GitHub平台时，网站的访问和图片加载较为缓慢，给读者带来较差的阅读体验。本篇博客**基于图床**的方式，解决了原始的将**本地图片一并上传到GitHub博客的对应仓库**，造成博客**图片加载缓慢**的问题
<!-- more -->

# jsDelivr介绍
jsDelivr 是一款免费、开源的加速 CDN 公共服务。`CDN` (Content Delivery Network)全称为内容分发网络，它是构建在现有网络基础之上的智能虚拟网络，依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率

# 配置过程
 配置过程主要包括两个步骤：
 - GitHub图片存储仓库创建及本地图片上传
 - 基于jsDelivr CDN的图片资源引用

## 创建存储图片的GitHub仓库并上传本地图片
1. 在GitHub上创建一个名为 `CDN-source` 的仓库用于存储图片等资源。仓库名称可以自行定义，下图由于我已经创建了名为 `CDN-source` 的仓库，提示我该仓库已存在：

![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/CDN_Based_On_jsDelivr/step1.png)

2. 想刚创建好的 `CDN-source` 仓库上传本地的图片
- 将需要上传的所有图片放在一个文件夹中，如图中的 `D:\myBlog\img` ：

![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/CDN_Based_On_jsDelivr/step2.png)

- 打开 **Git Bash** 进入 `D:\myBlog\img` 文件夹，执行如下命令完成图片上传：
	{% codeblock %}
	git init							// 进入项目目录下进行git初始化（可以看到文件夹中生成了.git目录）
	git remote add origin + 仓库地址 	// 添加远程仓库
	git add .							// 添加提交的所有文件
	git commit -m "注释"				// 提交代码
	git push -u origin master -f 		// 推送至仓库
	{% endcodeblock %}

- 执行完上述命令后即可在GitHub仓库看到自己上传的本地图片：

![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/CDN_Based_On_jsDelivr/step3.png)

## 基于jsDelivr CDN的图片资源引用

只需要将访问的资源地址改为如下形式即可：
{% codeblock %}
https://cdn.jsdelivr.net/gh/你的用户名/你的仓库名/文件夹名/文件名
{% endcodeblock %}

已我当前创建的图片仓库及上传的图片为例，访问链接为：
{% codeblock %}
https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/themes_pic/index_pic.jpg
{% endcodeblock %}

# 额外补充
发布不同版本的图片仓库有助于图片资源的管理，下面简单说一下**如何发布以及引用仓库发布版本的图片资源**

## 仓库发布
1. 在图片仓库中找到 `Draft a new release` 进入发布资源页面

![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/CDN_Based_On_jsDelivr/step4.png)

2. 填写相关信息：

![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/CDN_Based_On_jsDelivr/step5.png)
其中发布版本号①必填，其余②③选填，最后点击 `Publish release` 完成发布
***注意：版本号可自定义，但是十分重要（资源引用地址填写时有用）***

3. 构建基于jsDelivr的访问资源地址：

{% codeblock %}
https://cdn.jsdelivr.net/gh/你的用户名/你的仓库名@发布的版本号/文件夹名/文件名
{% endcodeblock %}
已我当前创建的图片仓库及上传的图片为例，访问链接为：
{% codeblock %}
https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source@1.0/themes_pic/index_pic.jpg
{% endcodeblock %}

**发布版本有助于区分新旧数据、便于资源管理，每次更新图片仓库后都需要发布**

# 参考
- [基于 jsDelivr + Github 搭建免费图床教程](https://blog.douchen.life/%E5%9F%BA%E4%BA%8EjsDelivr-Github%E6%90%AD%E5%BB%BA%E5%85%8D%E8%B4%B9%E5%9B%BE%E5%BA%8A%E6%95%99%E7%A8%8B/)
- [5步搭建免费图床（CDN图床）再也不用担心网站网速与内存了](https://blog.51cto.com/u_13409958/3669893)