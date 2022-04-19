---
title: C++调用Python代码
excerpt: Python强大的第三方库有助于方法建模。而当我们需要采用C++开发软件时，没有相应的C++库或开源代码，使得我们不得不在软件开发过程中调用部分Python代码
date: 2022-03-19 21:25:38
tags:
  - C++
  - Python
  - 混合编程
categories:
  - C++
---

# 前言
Python强大的第三方库有助于方法建模。而当我们需要采用C++开发软件时，没有相应的C++库或开源代码，使得我们不得不在软件开发过程中调用部分Python代码

# 前期准备
- Python配置完成
- VS安装完成

# 实现方法
## 实现基本流程
1. 调用的Python代码都需要包含在两行代码中间，即如下形式：
{% codeblock %}
Py_Initialize(); //首。初始化Python解释器
//这里是一堆其他代码
Py_Initialize(); //尾。结束Python的工作。
{% endcodeblock %}
2. 跟Python相关的东西一般声明为PyObject指针的形式，比如一会见到的如下声明：
{% codeblock %}
PyObject * pModule = NULL; //Python模块
PyObject * pFunc = NULL; //Python函数
PyObject * pArg = NULL; //函数接受的参数
{% endcodeblock %}
3. 用下面三行代码来完成导入模块、引入函数、构建参数的工作，都是”闻名如见面“类型的函数：
{% codeblock %}
pModule = PyImport_ImportModule("test"); //导入模块
pFunc = PyObject_GetAttrString(pModule, "write_to_xlsx"); //引入操作Excel的函数
pArg = Py_BuildValue("O,O", tuple1, tuple2); //把C++类型转换为Python类型
{% endcodeblock %}
4. 最后用一句，把参数传给调用函数并实际运行：
{% codeblock %}
PyEval_CallObject(pFunc, pArg);
{% endcodeblock %}


## 创建C++项目并完成环境和库配置
- 包含目录：在创建好的C++项目，`右击项目->属性->C/C++/常规`，将安装Python路径下的`\include`目录添加进去
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Cpp_Calls_Python_Code/step1.png)
- 附加库目录：在创建好的C++项目，`右击项目->属性->链接器->常规->附加库目录`，将安装Python路径下的`\libs`目录添加进去
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Cpp_Calls_Python_Code/step2.png)
- 附加依赖项：在创建好的C++项目，`右击项目->属性->链接器->输入->附加依赖库`，在**debug**下输入`python36.lib`，在**release**下输入`python36_d.lib`（不同版本Python最后数字不一样，自行修改）
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Cpp_Calls_Python_Code/step3.png)
***注意：在Python安装路径下的`\libs`文件夹中可能只存在python36.lib而没有python36_d.lib，此时需要复制一份python36.lib，并重命名为python36_d.lib，此外安装的Python是32位与64位，要与C++项目的环境配置匹配***

## C++调用Python函数并传入参数（参数为简单类型）
C++调用Python实现成功的前提是，此Python文件可以正常运行
1. 定义Python函数，并命名为test1.py。**注意：文件命名切忌为test等冲突的名称，因为Python有内置test函数，python自身模块的优先级高于你自己定义的模块**
{% codeblock %}
def add(a,b):
    return a+b
{% endcodeblock %}

2. 编写C++代码调用Python函数
- 引入头文件
{% codeblock %}
#include <Python.h>
#include <iostream>
using namespace std;
{% endcodeblock %}

- 测试
{% codeblock %}
	Py_Initialize();//使用python之前，要调用Py_Initialize();这个函数进行初始化
	if (!Py_IsInitialized())
	{
		printf("初始化失败！");
		return 0;
	}
	// 直接执行Python语句
	PyRun_SimpleString("import sys");
	PyRun_SimpleString("import os");
	PyRun_SimpleString("sys.path.append('./')");//这一步很重要，修改Python路径
	PyRun_SimpleString("print(os.listdir())");

	// 调用Python文件中的函数
	PyObject * pModule = NULL;//声明变量
	PyObject * pFunc = NULL;
	PyObject* args = NULL;
	PyObject* pRet = NULL;

	pModule = PyImport_ImportModule("test1");//这里是要调用的文件名
	if (pModule == NULL)
	{
		cout << "没找到" << endl;
	}
	else
	{
		cout << "找到了" << endl;
	}
	pFunc = PyObject_GetAttrString(pModule, "add");//这里是要调用的函数名
	args = Py_BuildValue("(ii)", 28, 103);//给python函数参数赋值
	pRet = PyObject_CallObject(pFunc, args);//调用函数
	int res = 0;
	PyArg_Parse(pRet, "i", &res);//转换返回类型
	cout << "res:" << res << endl;//输出结果
{% endcodeblock %}
**注意：PyImport_ImportModule，该函数似乎只能用相对路径，且PyImport_ImportModule函数传入的参数不能加”.py“**

3. 运行结果
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Cpp_Calls_Python_Code/step4.png)

## C++调用Python函数并传入参数（参数为列表类型）
1. 定义Python函数，并命名为test12.py
{% codeblock %}
# -*- coding:utf-8 -*-
from openpyxl import Workbook

def fit_prophet(time_series):
    book = Workbook()
    sheet = book.active
    str1 = ()

    time_series = tuple(time_series)
    str1 = str1 + ("time",)
    row1 = str1 + time_series
    rows = (row1,)
    for row in rows:
        sheet.append(row)
        print(row)
    book.save("E:\\test.xlsx")

if __name__=="__main__":
    fit_prophet((1,2,3,4,5.5))
{% endcodeblock %}

2. 编写C++代码调用Python函数
{% codeblock %}
#include <QtCore/QCoreApplication>
#include <Python.h>
#include <iostream>
using namespace std;

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

	Py_Initialize();//使用python之前，要调用Py_Initialize();这个函数进行初始化
	if (!Py_IsInitialized())
	{
		printf("初始化失败！");
		return 0;
	}
	// 直接执行Python语句
	PyRun_SimpleString("import sys");
	PyRun_SimpleString("sys.path.append('./')");//这一步很重要，修改Python路径

	// 调用Python文件中的函数
	PyObject * pModule = NULL;//声明变量
	PyObject * pFunc = NULL;
	PyObject* args = NULL;
	PyObject* pRet = NULL;

	// 调用需要传入list数据类型的Python函数
	pModule = PyImport_ImportModule("test12");//这里是要调用的文件名hello.py
	if (pModule == NULL)
	{
		cout << "test12 没找到" << endl;
	}
	else
	{
		cout << "test12 找到了" << endl;
	}

	vector<double> normalized_ntl_list{ 840.64914699999997, 65535.000000000000, 65535.000000000000 ,65535.000000000000 ,582.03208400000005 ,529.01506400000005 ,565.42856200000006 ,
										65535.000000000000, 565.42856200000006, 563.01787400000001 };

	pFunc = PyObject_GetAttrString(pModule, "fit_prophet");//这里是要调用的函数名
	
	PyObject* pyParams = PyList_New(0);           //初始化一个列表
	for (int i = 0; i < normalized_ntl_list.size(); i++)
	{
		PyList_Append(pyParams, Py_BuildValue("d", normalized_ntl_list[i]));	//列表添加元素值浮点数，可以添加不同类型的数据
	}
	
	/*
	PyObject* tuple = PyTuple_New(normalized_ntl_list.size());	//初始化一个元组
	for (int ii = 0; ii < normalized_ntl_list.size(); ii++)
	{
		PyTuple_SetItem(tuple, ii, Py_BuildValue("d", normalized_ntl_list[ii]));	//为元组赋值
	}
	*/
	args = PyTuple_New(1);              
	PyTuple_SetItem(args, 0, pyParams);
	//PyTuple_SetItem(args, 0, tuple);
	PyEval_CallObject(pFunc, args);	//函数调用

	cout << "prophet finished..." << endl;

	Py_Finalize();//调用Py_Finalize，这个根Py_Initialize相对应的。

	return 0;
}
{% endcodeblock %}
3. 运行结果
- console输出结果展示：
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Cpp_Calls_Python_Code/step5.png)
可见在Python代码中的print可以在C++的console输出展示

- 生成Excel展示
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Cpp_Calls_Python_Code/step6.png)
**重点：C++的vector数据转换成Python的元组或列表形式，首先需要创建对应的Object（PyList_New、PyTuple_New），然后循环赋值（PyList_Append、PyTuple_SetItem），再调用函数之前，需要再用一个元组将数据包装起来，才能成功**

## C++调用Python函数传入列表参数并返回列表参数

1. 定义Python函数，并命名为test13.py
{% codeblock %}
def listFuncTest(list1, list2, list3):
    print(list1)
    print(list2)
    print(list3)

    return list1, list3

if __name__=="__main__":
    lizt1 = [1,2,3]
    lizt2 = [4,5,6]
    lizt3 = [6,6,6]
    new_lizt1, new_lizt2 =listFuncTest(lizt1, lizt2, lizt3)
    print(new_lizt1)
    print(new_lizt2)
{% endcodeblock %}

2. 编写C++代码调用Python函数并获取返回值
{% codeblock %}
#include <QtCore/QCoreApplication>
#include <Python.h>
#include <iostream>
using namespace std;

// C++vector转PyObject函数（python list类型）
PyObject* createList(vector<int> list)
{
	PyObject* pyParams = PyList_New(0);           //初始化一个列表
	for (int i = 0; i < list.size(); i++)
	{
		PyList_Append(pyParams, Py_BuildValue("i", list[i]));	//列表添加元素值整数，可以添加不同类型的数据
	}
	return pyParams;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

	// 用于测试的vector数据
	vector<int> nums1({ 1,2,3 });
	vector<int> nums2({ 9,8,7 });
	vector<int> nums3({ -1,2,-4 });

	Py_Initialize();	//首。初始化Python解释器
	if (!Py_IsInitialized())
	{
		printf("初始化失败！");
		return 0;
	}
	// 直接执行Python语句
	PyRun_SimpleString("import sys");
	PyRun_SimpleString("sys.path.append('./')");//这一步很重要，修改Python路径

	// 调用Python文件中的函数
	PyObject * pModule = NULL;	//Python模块
	PyObject * pFunc = NULL;	//Python函数
	PyObject* args = NULL;		//函数接受的参数
	PyObject* pRet = NULL;		//函数返回的参数

	pModule = PyImport_ImportModule("test13");					//这里是要调用的文件名
	pFunc = PyObject_GetAttrString(pModule, "listFuncTest");	//这里是要调用的函数名

	// 将C++的vector转成PyObject类型
	PyObject* list1 = createList(nums1);
	PyObject* list2 = createList(nums2);
	PyObject* list3 = createList(nums3);

	// 多个参数传递到Python需要在外面再加一层tuple包装
	args = PyTuple_New(3);
	PyTuple_SetItem(args, 0, list1);
	PyTuple_SetItem(args, 1, list2);
	PyTuple_SetItem(args, 2, list3);

	// 调用Python函数并得到函数返回值（PyObject类型返回值）
	PyObject* pyReturnValue = PyEval_CallObject(pFunc, args);

	// 解析返回的PyObject类型为C++vector类型
	int size_params = PyTuple_Size(pyReturnValue);
	for (int i = 0; i < size_params; i++)
	{
		PyObject* list_value = PyTuple_GetItem(pyReturnValue, i);
		int size_list = PyList_Size(list_value);
		vector<int> result;
		for (int j = 0; j < size_list; j++)
		{
			int res = 0;
			PyArg_Parse(PyList_GetItem(list_value, j), "i", &res);
			result.push_back(res);
		}
		for (int j = 0; j < result.size(); j++)
		{
			cout << result[j] << ",";
		}
		cout << endl;
	}
	return 0;
}
{% endcodeblock %}

3. 运行结果
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Cpp_Calls_Python_Code/step7.png)

- 上图红色框输出结果为C++ vector数据在调用时作为参数传递到函数后输出的结果
- 上图黄色框输出结果为Python函数返回的两个list类型数据，在C++中解析后输出的结果

更多C++调用Python：[各种参数传递与接收示例](https://blog.csdn.net/dcx_dcx/article/details/104388718?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-4.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-4.pc_relevant_default&utm_relevant_index=8)
解析参数和构造值官方文档链接：https://docs.python.org/3/c-api/arg.html
Python官方C API手册：https://docs.python.org/zh-cn/3/c-api/concrete.html