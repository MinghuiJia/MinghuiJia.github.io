---
title: Hexo-fluid主题设置统计博客阅读量与评论
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
1. 进入LeanCloud官网:https://www.leancloud.cn/ ，注册账号并完成实名认证和邮箱验证
2. 在LeanCloud主页面按照如下3步骤，进行应用的创建（用户名随意起）
{% asset_image step1.png %}
3. 创建Class，按照如下步骤，在`数据存储`->`结构化存储`创建Class
{% asset_image step2.png %}
**注意：**
- 此处创建的 Class 名字必须为Counter，用来保证与NexT主题的修改相兼容，fluid没有限制要求
- ACL权限选择无限制，避免后续因为权限的问题导致次数统计显示不正常
4. 在创建的应用设置中寻找AppID与AppKey
<div align=center>{% asset_img step3.png %}</div>
<div align=center>{% asset_img step4.png %}</div>

# 修改主题配置文件
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
2. 在fluid主题配置文件页脚部分的`enable`与`source`进行如下设置，这样可以在博客页面最下面看到访问人数与访问次数
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
3. 在fluid主题配置文件中搜索`views`，将`enable`与`source`两处进行设置，可以实现对每篇博客的访问次数进行统计
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
**最后使用`hexo g`、`hexo d`命令重新部署博客，就可以正常使用文章阅读量统计的功能了**

# 后台管理
当以上部分配置完成之后，我们的博客页面打开时，便会自动向服务器发送信息，在我们刚才创建的应用test的Counter表中创建数据
{% asset_image step8.png %}
***需要特别说明的是：***
记录文章访问量的唯一标识符是文章的发布日期和文章的标题，因此要确保这两个数值组合的唯一性，如果你更改了这两个数值，会造成文章阅读数值的清零重计。其中time字段的数值表示某一篇文章的访问量，其他字段的具体作用可以查阅LeanCloud官方文档，最好不要随意修改

# 评论设置
1. 当用户配置好LeanCloud后，可以在主题配置文件`_config.yml`文件中找到评论插件部分，设置`Valine`部分的`appId`与`appKey`即可。
更多参数设置可以访问https://valine.js.org/configuration.html
{% codeblock %}
#---------------------------
# 评论插件
# Comment plugins
#
# 开启评论需要先设置上方 `post: comments: enable: true`，然后根据 `type` 设置下方对应的评论插件参数
# Enable comments need to be set `post: comments: enable: true`, then set the corresponding comment plugin parameters below according to `type`
#---------------------------
# Valine
# 基于 LeanCloud
# Based on LeanCloud
# See: https://valine.js.org/
valine:
  appId: xxx
  appKey: xxx
  path: window.location.pathname
  placeholder: 欢迎评论
  avatar: 'retro'
  meta: ['nick', 'mail', 'link']
  requiredFields: []
  pageSize: 10
  lang: 'zh-CN'
  highlight: false
  recordIP: false
  serverURLs: ''
  emojiCDN:
  emojiMaps:
  enableQQ: false
{% endcodeblock %}


2. 在fluid主题中可以搜索`comments`关键词，分别对友情链接与文章页进行评论设置，分别设置`enable`为`true`，`type`为`valine`
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
    type: valine

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
    type: valine
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