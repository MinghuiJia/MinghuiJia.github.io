---
title: Windows下Python2.7安装第三方库编码错误
date: 2022-05-10 14:24:45
index_img: https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Python_Install_Decode/python_decode_index.png
excerpt: 解决Python2.7安装第三方库过程中出现UnicodeDecodeError问题
tags:
  - Python
  - install
categories:
  - Python
---

# 前言
前两天为了使用Python的ArcPy第三方库，我重新在电脑上安装了ArcGIS10.4。由于ArcGIS自带的Python版本是2.7且只装有部分基础库，我的需求是更新Python2.7版本中的Numpy库，用于矩阵乘法的计算。但是在安装过程中出现了编码问题，特此记录一下解决方案

# 问题
- 我进入到Python2.7安装路径下有exe可执行文件的文件夹下，在cmd命令行中输入：
{% codeblock %}
python -m pip install numpy -i https://pypi.tuna.tsinghua.edu.cn/simple
{% endcodeblock %}
安装过程中报错
{% note warning %}
**UnicodeDecodeError: 'ascii' codec can't decode byte 0xcb in position 0:**
{% endnote %}

# 解决方案
- 报错的原因是与编码有关，pip把下载的临时文件放在用户临时文件中，可能路径中存在中文，导致ascii无法解码
- 找到Python2.7目录下下的`Lib`文件夹中的`ntpath.pt`文件打开，并找到`def join(path, *path):`方法，在函数下添加如下两行代码：
{% codeblock %}
reload(sys)  
sys.setdefaultencoding('gbk')
{% endcodeblock %}

# 参考资料
window下安装numpy出现UnicodeDecodeError：https://blog.csdn.net/wfei101/article/details/76166923