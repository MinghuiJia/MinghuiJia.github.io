---
title: Hexo写文章时引用本地图片无法显示
excerpt: 利用搭建好的博客进行文章编写过程中，文章内容需要展示图片，但引用本地图片时总显示不出来。本篇博客主要解决利用Hexo写文章引用本地图片无法显示的问题
index_img: https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo_Write_The_Article_Images_Cannot_Be_Displayed/index_img4.png
date: 2022-02-22 21:33:31
tags:
  - hexo
  - 图片无法加载
categories:
  - hexo
---

# 前言
利用搭建好的博客进行文章编写过程中，文章内容需要展示图片，但引用本地图片时总显示不出来。本篇博客主要解决利用Hexo写文章引用本地图片无法显示的问题
<!-- more -->

# 问题描述
当用户利用Hexo编写文章引入图片时，常常会出现如下情况：
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo_Write_The_Article_Images_Cannot_Be_Displayed/step1.png)

# 插件安装与配置
1. 首先需要安装一个图片路径转换的插件，叫***hexo-asset-image*** 。在Git Bash界面输入命令：
{% codeblock %}
	npm install https://github.com/CodeFalling/hexo-asset-image --save
{% endcodeblock %}

2. 然后修改myBlog/node_modules/hexo-asset-image/index.js文件中的代码，将内容更换为下面代码
{% codeblock %}
	'use strict';
	var cheerio = require('cheerio');

	// http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
	function getPosition(str, m, i) {
	  return str.split(m, i).join(m).length;
	}

	var version = String(hexo.version).split('.');
	hexo.extend.filter.register('after_post_render', function(data){
	  var config = hexo.config;
	  if(config.post_asset_folder){
			var link = data.permalink;
		if(version.length > 0 && Number(version[0]) == 3)
		   var beginPos = getPosition(link, '/', 1) + 1;
		else
		   var beginPos = getPosition(link, '/', 3) + 1;
		// In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
		var endPos = link.lastIndexOf('/') + 1;
		link = link.substring(beginPos, endPos);

		var toprocess = ['excerpt', 'more', 'content'];
		for(var i = 0; i < toprocess.length; i++){
		  var key = toprocess[i];
	 
		  var $ = cheerio.load(data[key], {
			ignoreWhitespace: false,
			xmlMode: false,
			lowerCaseTags: false,
			decodeEntities: false
		  });

		  $('img').each(function(){
			if ($(this).attr('src')){
				// For windows style path, we replace '\' to '/'.
				var src = $(this).attr('src').replace('\\', '/');
				if(!/http[s]*.*|\/\/.*/.test(src) &&
				   !/^\s*\//.test(src)) {
				  // For "about" page, the first part of "src" can't be removed.
				  // In addition, to support multi-level local directory.
				  var linkArray = link.split('/').filter(function(elem){
					return elem != '';
				  });
				  var srcArray = src.split('/').filter(function(elem){
					return elem != '' && elem != '.';
				  });
				  if(srcArray.length > 1)
					srcArray.shift();
				  src = srcArray.join('/');
				  $(this).attr('src', config.root + link + src);
				  console.info&&console.info("update link as:-->"+config.root + link + src);
				}
			}else{
				console.info&&console.info("no src attr, skipped...");
				console.info&&console.info($(this));
			}
		  });
		  data[key] = $.html();
		}
	  }
	});
{% endcodeblock %}

3. 打开myBlog文件夹下的_config.yml文件，修改下述内容
{% codeblock %}
	post_asset_folder: true
{% endcodeblock %}

# 问题解决方法
1. 当上述插件安装并完成配置后，我们在Git Bash中输入如下命令
{% codeblock %}
	hexo new post article1
	hexo new draft article2
{% endcodeblock %}
在myBlog/source/_posts 或myBlog/source/_drafts 文件夹下会创建article.md文件与article同名文件夹
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo_Write_The_Article_Images_Cannot_Be_Displayed/step2.png)

2. 将文章需要引用的本地图片，放在同名文章所对应的文件夹下
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo_Write_The_Article_Images_Cannot_Be_Displayed/step3.png)

3. 在Markdown（文章）文件中需要引入图片的地方添加如下代码：
{% codeblock %}
	{% asset_img example.jpg %}
{% endcodeblock %}

4. 当上述操作执行完成，并输入如下命令完成博客部署
{% codeblock %}
	hexo g
	hexo d
{% endcodeblock %}
在GitHub主页可以看到，html页面与图片均在同一个文件夹中，文件夹命名与文章.md同名
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo_Write_The_Article_Images_Cannot_Be_Displayed/step4.png)

5. 此时在浏览器浏览文章，可以发现显示图片的源码，在图片加载路径那里找到了与html在同一个文件夹下的图片
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Hexo_Write_The_Article_Images_Cannot_Be_Displayed/step5.png)
它会自动寻找，并补全图片的绝对路径，完成图片加载

# 图片无法加载的可能原因
1. 本地图片没有上传至GitHub仓库，导致引用无效
解决方案：安装插件

2. 本地图片没有存放在同名文件夹中
解决方案：将需要引用的本地图片存放在与文章名相同的文件夹中

3. 图片路径出错
解决方案：打开myBlog文件夹下的_config.yml修改下述内容
{% blockquote %}
	\# URL
	\#\# Set your site url here. For example, if you use GitHub Page, set url as '`https://username.github.io/project`'
	url: `http://minghuijia.cn/` 改成域名访问地址
{% endblockquote %}

4. 相对路径引用的标签插件不当
把一个 example.jpg 图片放在资源文件夹中，如果通过使用相对路径的常规 markdown 语法 \!\[\](/example.jpg) ，它将不会出现在首页上，需要采用 ***问题解决方法*** 3点中的方式

