---
title: vs2017+Qt创建Qtdll项目并调用
excerpt: 将一些公用的工具函数可以进行封装并生成Dll，方便组内成员直接调用，提高开发效率
index_img: https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Vs2017_Qt_Creates_Qtdll_Project_And_Calls_It/step1.png
date: 2022-01-22 14:20:54
tags:
 - Qt
 - 动态链接库
categories:
  - Qt
---

# 前言
将一些公用的工具函数可以进行封装并生成Dll，方便组内成员直接调用，提高开发效率
<!-- more -->

# 创建Qtdll项目
1. 按照如下图方式在 vs2017 中创建一个Qtdll项目
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Vs2017_Qt_Creates_Qtdll_Project_And_Calls_It/step1.png)
项目创建成功后会自动生成两个.h文件和一个.cpp文件如下图。**此外刚创建项目后include部分会报错，需要增加两行代码`#include <iostream>`;`using namespcae std;`**
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Vs2017_Qt_Creates_Qtdll_Project_And_Calls_It/step2.png)

2. 在创建的Qtdll项目中编写测试函数（求和函数）
{% codeblock %}
	class QTCLASSLIBRARY1_EXPORT QtClassLibrary1
	{
	public:
		QtClassLibrary1();
		int Add(int x, int y);
	};
	
	int QtClassLibrary1::Add(int x, int y)
	{
		return x + y;
	}
{% endcodeblock %}

3. 在vs里面的解决方案资源管理器栏内右键点击`解决方案`，选择`生成解决方案`，成功后会看到如下界面
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Vs2017_Qt_Creates_Qtdll_Project_And_Calls_It/step3.png)
同时在Qtdll项目`x64\Debug`路径下可以看到生成的文件（.dll与.Lib）
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Vs2017_Qt_Creates_Qtdll_Project_And_Calls_It/step4.png)

# 创建调用Qtdll的项目
1. 为了方便期间，创建一个QtConsoleApplication项目对生成的Qtdll进行调用
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Vs2017_Qt_Creates_Qtdll_Project_And_Calls_It/step5.png)

2. 将Qtdll项目的中的两个.h文件（`QtClassLibrary1.h`,`qtclasslibrary1_global.h`）以及.lib文件（`QtClassLibrary1.lib`）移动到QtConsoleApplication项目的如下图位置
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Vs2017_Qt_Creates_Qtdll_Project_And_Calls_It/step6.png)

3. 将Qtdll项目的中的.dll文件（`QtClassLibrary1.dll`）移动到QtConsoleApplication项目的如下图位置
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Vs2017_Qt_Creates_Qtdll_Project_And_Calls_It/step7.png)

4. 在QtConsoleApplication项目中添加头文件（`QtClassLibrary1.h`,`qtclasslibrary1_global.h`）与lib文件（`QtClassLibrary1.lib`），并调用Dll
{% codeblock %}
#include <QtCore/QCoreApplication>
#include "QtClassLibrary1.h"
#include "qtclasslibrary1_global.h"
#include "qdebug.h"

#pragma comment(lib,"QtClassLibrary1.lib")
#pragma execution_character_set("utf-8")

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
	QtClassLibrary1 dll;
	qDebug() << "加法" << dll.Add(56, 45);
    return 0;
}
{% endcodeblock %}
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/Vs2017_Qt_Creates_Qtdll_Project_And_Calls_It/step8.png)

# 补充知识
1. 导入dll库
{% codeblock %}
#pragma comment(lib,"QtClassLibrary1.lib")
{% endcodeblock %}
- **这是告诉编译器在编译形成的.obj文件和.exe文件中加一条信息，使得链接器在链接库的时候要去找QtClassLibrary1.lib这个库，而不是先去找别的库**
- **#pragma comment(lib, libname)告诉链接器将`libname`库添加到库依赖关系列表中，与添加到项目属性中的操作一样 Linker->Input->Additional dependencies**

2. 解决源代码中有中文字符无法识别问题
{% codeblock %}
#pragma execution_character_set("utf-8")
{% endcodeblock %}
- **编译器将源代码中的窄字符和窄字符串文本编码为可执行文件中UTF-8，缺少这行代码就无法识别中文**