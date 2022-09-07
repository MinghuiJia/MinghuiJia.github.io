---
title: Hexo-fluid主题设置统计博客阅读量与评论
excerpt: 本篇博客基于LeanCloud统计博客页面访问次数与访问人数、及文章阅读次数，以及实现文章与友情链接评论功能
index_img: https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/index_img1.png
date: 2022-03-14 21:05:19
comment: false
tags:
 - hexo
 - 博客阅读量
 - 评论设置
categories:
 - hexo

---

# 前言
本篇博客基于LeanCloud统计博客页面访问次数与访问人数、及文章阅读次数，以及实现文章与友情链接评论功能
<!-- more -->

# 配置 LeanCloud
## LeanCloud数据库设置
1. 进入`LeanCloud官网`:https://www.leancloud.cn/ ，或`LeanCloud国际版`:https://leancloud.app/ ，注册账号并完成实名认证和邮箱验证
2. 在LeanCloud主页面按照如下3步骤，进行应用的创建（用户名随意起）
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/step1.png)
3. 创建Class，按照如下步骤，在`数据存储`->`结构化存储`创建Class
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/step2.png)
**注意：**
- 此处创建的 Class 名字必须为Counter，用来保证与NexT主题的修改相兼容，fluid没有限制要求
- ACL权限选择无限制，避免后续因为权限的问题导致次数统计显示不正常
4. 在创建的应用设置中寻找AppID与AppKey
<div align=center><img src="https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/step3.png" /></div>
<div align=center><img src="https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/step4.png" /></div>

## 修改主题配置文件中LeanCloud参数
1. 打开fluid的主题配置文件`_config.yml`，把配置 LeanCloud时的AppID与AppKey复制到如下位置，并设置`ignore_local`为`true`，保证在本地启动服务的时候不会记录访问次数
{% codeblock %}
# LeanCloud 计数统计，可用于 PV UV 展示，如果 `web_analytics: enable` 没有开启，PV UV 展示只会查询不会增加
# LeanCloud count statistics, which can be used for PV UV display. If `web_analytics: enable` is false, PV UV display will only query and not increase
leancloud:
	app_id: xxx
	app_key: xxx
    # REST API 服务器地址，国际版不填
    # Only the Chinese mainland users need to set
	server_url: 
    # 统计页面时获取路径的属性
    # Get the attribute of the page path during statistics
	path: window.location.pathname
    # 开启后不统计本地路径( localhost 与 127.0.0.1 )
    # If ture, ignore localhost & 127.0.0.1
	ignore_local: true
{% endcodeblock %}

# 博客页访问及文章阅读次数设置
1. 在fluid主题配置文件页脚部分的`enable`与`source`进行如下设置，这样可以在博客页面最下面看到访问人数与访问次数
{% codeblock %}
# 展示网站的 PV、UV 统计数
# Display website PV and UV statistics
statistics:
   enable: true

   # 统计数据来源，使用 leancloud 需要设置 `web_analytics: leancloud` 中的参数；使用 busuanzi 不需要额外设置，但是有时不稳定，另外本地运行时 busuanzi 显示统计数据很大属于正常现象，部署后会正常
   # Data source. If use leancloud, you need to set the parameter in `web_analytics: leancloud`
   # Options: busuanzi | leancloud
   source: "leancloud"

   # 页面显示的文本，{}是数字的占位符（必须包含)，下同
   # Displayed text, {} is a placeholder for numbers (must be included), the same below
   pv_format: "总访问量 {} 次"
   uv_format: "总访客数 {} 人"
{% endcodeblock %}
2. 在fluid主题配置文件中搜索`views`，将`enable`与`source`两处进行设置，可以实现对每篇博客的访问次数进行统计
{% codeblock %}
# 浏览量计数
# Number of visits
views:
  enable: true
  # 统计数据来源
  # Data Source
  # Options: busuanzi | leancloud
  source: "leancloud"
  format: "{} 次"
{% endcodeblock %}
3. **最后使用`hexo g`、`hexo d`命令重新部署博客，就可以正常使用文章阅读量统计的功能了**

# 评论设置
为了方便对文章感兴趣的小伙伴可以在博客上与博主互动，我特地增加了评论功能。这里我选择了一款叫做`Waline`的带后端评论系统。该系统从`Valine`衍生而来，具有免费部署、轻量易用等特点。最重要的是它具备：**邮件通知、评论管理、社交登录支持、置顶评论等**功能
{% note warning %}
由于Leancloud国内版(https://www.leancloud.cn/) 需要为应用额外绑定**已备案**的域名，同时购买独立IP并完成备案接入。因此我们切换到国际版(https://leancloud.app/)
{% endnote %}

## 配置 LeanCloud 数据库
本文已讲解如何配置 LeanCloud，因此该步骤略过，具体操作可返回上文查看：[配置 LeanCloud](#配置-LeanCloud)

## 配置 Vercel 服务端
1. 通过点击 [Vercel](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Fwalinejs%2Fwaline%2Ftree%2Fmain%2Fexample) 跳转至`Vercel`进行`Server`端部署
{% note info %}
**注意：如果未登录过，Vercel会让你注册或登录，可使用GitHub账户进行快捷登录**
{% endnote %}
2. 输入一个自定义`Vercel`项目名称并点击`Create`创建项目
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/vercel1.png)
3. 此时`Vercel`会基于Waline模板新建并初始化仓库到GitHub，仓库名为之前输入的项目名称。项目部署会等待一小会儿
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/vercel2.png)
4. 项目部署成功后，点击`Go to Dashboard`跳转至应用控制台。然后点击顶部的`Settings`-`Environment Variables`进入环境变量配置页，并配置三个环境变量名称为：`LEAN_ID`、`LEAN_KEY`、`LEAN_MASTER_KEY`，它们的值分别对应`LeanCloud`中得到的`APP ID`、`APP KEY`、`Master Key`
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/vercel3.png)
{% note info %}
使用LeanCloud国内版，需要额外配置`LEAN_SERVER`环境变量，值为绑定好的域名
{% endnote %}
5. 环境变量配置好之后，点击顶部的`Deployments`，然后点击顶部最新的一次部署右侧的`Redeploy`按钮进行重新部署，使得刚设置的环境变量生效
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/vercel4.png)
6. 等待几分钟后，部署项目的状态`STATUS`会变成`Ready`。此时点击页面中的`Visit`即可跳转到刚部署好的网站地址，该地址就是今后评论系统的服务端地址
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/vercel5.png)
7. 接下来点击顶部的`Settings`-`Domains`进入域名配置页，输入需要绑定的域名并点击Add
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/vercel6.png)
{% note info %}
绑定域名为**example.yourdomain.com**，其中`example`可以自定义修改，`yourdomain.com`则是自己的域名
{% endnote %}
8. 添加完绑定域名之后，要在域名服务器商处添加新的`CNAME`解析记录
|    Type     |     Name     |            Value          |
|:-----------:|:------------:|:-------------------------:|
|   CNAME     |   example    |    cname.vercel-dns.com   |
9. 最后等待生效，看到如下图则表明绑定成功
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/vercel7.png)
{% note info %}
- 评论系统：example.yourdomain.com
- 评论管理：example.yourdomain.com/ui
{% endnote %}

## 修改主题配置文件中评论插件参数
1. 当用户配置好LeanCloud后，可以在主题配置文件`_config.yml`文件中找到评论插件部分，设置`Waline`部分
{% codeblock %}
#---------------------------
# 评论插件
# Comment plugins
#
# 开启评论需要先设置上方 `post: comments: enable: true`，然后根据 `type` 设置下方对应的评论插件参数
# Enable comments need to be set `post: comments: enable: true`, then set the corresponding comment plugin parameters below according to `type`
#---------------------------
# Waline
# 从 Valine 衍生而来，额外增加了服务端和多种功能
# Derived from Valine, with self-hosted service and new features
# See: https://waline.js.org/
waline:
  serverURL: 'https://waline.minghuijia.cn/'
  path: window.location.pathname
  placeholder: 欢迎评论
  meta: ['nick', 'mail', 'link']
  requiredMeta: ['nick']
  lang: 'zh-CN'
  emoji: ['https://gcore.jsdelivr.net/gh/walinejs/emojis/weibo']
  dark: 'html[data-user-color-scheme="dark"]'
  avatar: 'retro'
  avatarCDN: 'https://seccdn.libravatar.org/avatar/'
  avatarForce: false
  wordLimit: 0
  pageSize: 10
  highlight: true
  login: 'force'
  visitor: true
{% endcodeblock %}
{% note info %}
在主题配置文件`_config.yml`中可能存在部分前端配置参数没有填写，用户通过查看[**帮助手册**](https://waline.js.org/reference/client.html#el)自行添加即可
{% endnote %}

2. 在fluid主题中可以搜索`comments`关键词，分别对友情链接与文章页进行评论设置，分别设置`enable`为`true`，`type`为`waline`
{% codeblock %}
#---------------------------
# 文章页
# Post Page
#---------------------------
# 评论插件
  # Comment plugin
  comments:
    enable: true
    # 指定的插件，需要同时设置对应插件的必要参数
    # The specified plugin needs to set the necessary parameters at the same time
    # Options: utterances | disqus | gitalk | valine | waline | changyan | livere | remark42 | twikoo | cusdis
    type: waline

#---------------------------
# 友链页
# Links Page
#---------------------------
# 评论插件
  # Comment plugin
  comments:
    enable: true
    # 指定的插件，需要同时设置对应插件的必要参数
    # The specified plugin needs to set the necessary parameters at the same time
    # Options: utterances | disqus | gitalk | valine | waline | changyan | livere | remark42 | twikoo | cusdis
    type: waline
{% endcodeblock %}

3. 此时访问友情链接页与任意博客文章时，在页面底部均可以看到评论区域。如果希望某篇博客关闭评论，可以通过在 `Front-matter` 设置 `comment: bool` 来控制评论开关，或者通过 `comment: 'type'` 来开启指定的评论插件
{% codeblock %}
---
title: Hexo-fluid主题设置统计博客阅读量
date: 2022-03-14 21:05:19
comment: false
tags:
 - hexo
 - 博客阅读量
categories:
 - hexo

---
{% endcodeblock %}

## 评论通知
当网站有用户发布评论或者用户回复评论时，`Waline`支持对博主和回复评论作者进行通知。博主通知支持多种方式，包括 QQ、微信、邮件等，回复评论作者仅支持邮件通知
下面讲介绍邮件通知功能的配置过程
1. 在`Vercel`服务端添加环境变量。在部署项目顶部的`Settings`-`Environment Variables`进入环境变量配置页下列环境变量：
	- `SMTP_SERVICE`：SMTP 邮件发送服务提供商
	- `SMTP_USER`：SMTP 邮件发送服务的用户名，一般为登录邮箱
	- `SMTP_PASS`：SMTP 邮件发送服务的密码，一般为邮箱登录密码，部分邮箱(例如 163)是单独的 SMTP 密码
	- `AUTHOR_EMAIL`：博主邮箱，用来接收新评论通知。如果是博主发布的评论则不进行提醒通知

![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/email1.png)
{% note success %}
`SMTP_SERVICE`所支持的运营商可以在[**这里**](https://github.com/nodemailer/nodemailer/blob/master/lib/well-known/services.json)查看
如果运营商不受支持，则必须填写`SMTP_HOST`与`SMTP_PORT`。
- `SMTP_HOST`：SMTP 服务器地址，一般可以在邮箱的设置中找到
- `SMTP_PORT`：SMTP 服务器端口，一般可以在邮箱的设置中找到
{% endnote %}
{% note warning %}
用户注册和评论的邮件通知都会用到邮件服务。配置邮件服务相关变量后，用户注册会增加邮箱验证码确认相关的操作，用来防止恶意的注册。
{% endnote %}
2. 环境变量配置好之后，点击顶部的`Deployments`，然后点击顶部最新的一次部署右侧的`Redeploy`按钮进行重新部署，使得刚设置的环境变量生效
3. 在LeanCloud应用的`设置`-`安全中心`中，将`推送服务`打开
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/email2.png)


# 后台管理
当以上部分配置完成之后，我们的博客页面打开时，便会自动向服务器发送信息
1. 在我们刚才创建的应用waline-test的Counter表中，可以看到创建了每篇文章阅读的次数，以及用户访问博客的次数及人数
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/step8.png)
***需要特别说明的是：***
记录文章访问量的唯一标识符是文章的发布日期和文章的标题，因此要确保这两个数值组合的唯一性，如果你更改了这两个数值，会造成文章阅读数值的清零重计。其中time字段的数值表示某一篇文章的访问量，其他字段的具体作用可以查阅LeanCloud官方文档，最好不要随意修改
2. 在Comment表中可以看到在博客中的留言信息
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/comment1.png)
3. 在Users表中可以看到所有注册过的用户信息
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo-fluid_Theme_Setting_Counts_Blog_Reads/comment2.png)
{% note info %}
当使用评论的登录功能时，第一个注册的用户默认为评论管理系统的管理员。因此用户配置评论系统后需及时申请管理员账号
{% endnote %}
# 更多Fluid主题自定义教程（持续更新）
- Hexo's Fluid 主题私人定制链接：https://www.erenship.com/posts/40222.html

# 参考资料
Waline配置帮助手册：https://waline.js.org/