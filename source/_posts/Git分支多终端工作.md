---
title: Git分支多终端工作
date: 2022-02-26 21:50:52
tags:
  - Git
  - 多终端
categories:
  - hexo
---

# 前言
由于hexo d将个人博客部署在GitHub仓库前需要进行编译，将Markdown等文件进行编译后生成网页并上传，并不包含源文件。因此，当在某个电脑环境下配置好Hexo+GitHub个人博客并进行了一段时间写作后，需要多终端切换工作，这时就需要在Git上进行上传分支
<!-- more -->

# 简介
本篇博客实现了简单在新电脑上配置，并从GitHub上把文件Clone下来就可以实现多终端无缝切换

# 原理
采用hexo d对博客内容上传部署到GitHub的过程，实际上是hexo编译后的文件上传，没有源文件。在上传的GitHub仓库可以看到，上传的文件其实是本地目录中.deploy_git文件夹里面的文件，而source、themes、node_modules等文件都没有同步上传到GitHub。利用git的分支管理可以将源文件上传到GitHub

# 分支上传
## 创建分支
1. 首先需要登录自己的GitHub主页，找到对应的repositories仓库。然后在按照如下图步骤创建一个hexo分支：
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Cpp_Calls_Python_Code/step1.png)

2. 然后在仓库的setting中，设置新创建的hexo分支为默认分支，如下图操作：
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Cpp_Calls_Python_Code/step2.png)
设置为默认分支的目的是为了方便同步，不用指定分支即可更新源码文件

## 本地克隆
1. 分支创建并设置好后，在本地任意目录下，通过Git Bash，输入如下指令：
{% codeblock %}
	git clone git@github.com:MinghuiJia/minghuijia.github.io.git
{% endcodeblock %}
将GitHub仓库中的文件克隆到本地。此时由于已经设置了默认分支是hexo，所以只克隆hexo分支下的文件

2. 然后在本地已经克隆好的文件夹（minghuijia.github.io）中删除***\.git文件夹***以外的所有文件，如下图红色框起来的文件均需要删除
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Cpp_Calls_Python_Code/step3.png)

3. 将本地之前写博客的源文件***除.deploy_git文件夹***外全部拷贝过来。***注意：拷贝到clone文件夹（minghuijia.github.io）中的源文件应包含一个.gitignore文件，用于忽略一些不需要的文件，如果没有该文件的话，自己创建一个并粘贴如下文字***：
{% codeblock %}
	.DS_Store
	Thumbs.db
	db.json
	*.log
	node_modules/
	public/
	.deploy*/
{% endcodeblock %}
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Cpp_Calls_Python_Code/step4.png)
另外需要注意，***如果theme是克隆来的，应该将主题文件夹中的.git文件夹删掉***。因为git不能嵌套上传，会导致上传时报错，无法上传主题文件，导致配置在其余终端不能用
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Cpp_Calls_Python_Code/step5.png)

## 上传分支
上述操作完成后，Git Bash在当前文件夹下输入如下指令上传源文件：
{% codeblock %}
	git add .
	git commit -m "add branch"
	git push
{% endcodeblock %}
上传结束后可以在GitHub仓库的hexo分支查看，可以发现node_modules、public、db.json被忽略而未上传（这些文件夹会在新电脑配置时自动创建）
***注意:node_modules文件夹中的文件会自动生成，但是图片加载的hexo-asset-image/index.js文件也是默认创建，如果不按照本地图片无法显示教程中对js文件修改会导致图片路径错误而无法加载图片**

# 更换电脑操作
## 基本环境配置
1. 安装Git
2. [安装Node.js](https://www.runoob.com/nodejs/nodejs-install-setup.html)[npm安装教程](https://www.cnblogs.com/quwaner/p/11541445.html)
3. 设置Git全局邮箱和用户名
{% codeblock %}
	git config --global user.name "xxx"
	git config --global user.email "xxx"
{% endcodeblock %}
4. 设置ssh key
{% codeblock %}
	ssh-keygen -t rsa -C "youremail"
{% endcodeblock %}
rsa生成后填到GitHubSSH中，并验证是否成功
{% codeblock %}
	ssh -T git@github.com
{% endcodeblock %}
5. 安装hexo，输入如下指令：
{% codeblock %}
	npm install hexo-cli -g
{% endcodeblock %}
6. 由于博客框架已经搭建好，当前目的只是更换终端设备，因此不需要初始化

## 文件克隆
1. 在新的终端选择任意文件夹，打开Git Bash，并输入如下指令完成克隆：
{% codeblock %}
	git clone git@xxxx
{% endcodeblock %}
2. 进入到克隆下来的文件夹，并完成如下指令安装：
{% codeblock %}
	cd xxx.github.io
	npm install		#执行之后会创建node_modules文件夹
	npm install hexo-deployer-git --save	#执行后会创建.deploy_git文件夹
{% endcodeblock %}
3. 然后使用hexo命令生成和部署：
{% codeblock %}
	hexo g		#执行之后会创建public文件夹
	hexo d	#执行后会将新的文件部署到GitHubmaster分支
{% endcodeblock %}
4. 在写文章之前，记得检查node_modules文件夹中hexo-asset-image/index.js文件，需要对该文件的代码进行修改（具体参考本地图片加载博客教程），保证图片加载时路径正确
5. 最后即可进行新的博客撰写
{% codeblock %}
	hexo new newpage
{% endcodeblock %}

## 博客写完后的操作
1. 每次文章写完后，要把源文件上传到hexo分支
{% codeblock %}
	git add .
	git commit -m "xxx
	git push
{% endcodeblock %}
2. 如果终端已经执行过1操作一次，本地电脑上clone文件夹，后续更新源文件即可使用如下指令和远端同步即可：
{% codeblock %}
	git pull
{% endcodeblock %}
