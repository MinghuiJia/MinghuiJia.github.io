---
title: Qt读取Excel数据
date: 2022-05-10 12:40:35
excerpt: 作者写本篇博客的原因是，在前段时间基于Qt进行GUI程序开发过程中需要读取经济面板数据并将数据传入已写好的计量经济学模型中。因此我在网上查找了基于Qt读取Excel的教程，在这里做一个总结和归纳，方便以后需要相同需求时随时查看
index_img: https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/QtExcel/qtexcel_index.png
tags:
  - Qt
  - 数据读取
categories:
  - Qt
---

# 前言
作者写本篇博客的原因是，在前段时间基于Qt进行GUI程序开发过程中需要读取经济面板数据并将数据传入已写好的计量经济学模型中。因此我在网上查找了基于Qt读取Excel的教程，在这里做一个总结和归纳，方便以后需要相同需求时随时查看

# Excel读取库
- ExcelEngine.h
{% codeblock %}
#ifndef EXCELENGINE_H
#define EXCELENGINE_H
#include <ActiveQt/QAxObject>
#include <Windows.h>
#include <QFile>
#include <QStringList>
#include <QString>

class ExcelEngine
{
public:
    ExcelEngine(void);
    ExcelEngine(const QString file_Name);
    ~ExcelEngine(void);
    
    // 打开Excel
    bool open(bool visible = false,bool display_Alerts = false);    

    // 通过Index获取sheet工作表
    QAxObject *getWorkSheet(int Sheet_Index);

    // 通过SheetName获取sheet工作表
    QAxObject *getWorkSheet(QString sheet_Name);

    // 通过索引值获取工作表（workSheet）的表明
    QString getWorkSheetNameByIndex(int inedx);

    // 获取WorkSheets工作表名列表
    QStringList getWorkSheetNameList(void);

    // 获取单元格内容
    QString getCellString(int workSheet_Index,int row,int column);

    // 获取单元格内容
    QString getCellString(QString workSheet_Name,int row,int column);

    // 获取workSheet的行数
    int getWorkSheetRows(QString workSheet_Name);

    // 获取workSheet的行数
    int getWorkSheetRows(int  workSheet_Index);

    // 获取workSheet的列数
    int getWorkSheetColumns(QString workSheet_Name);

    // 获取workSheet的列数
    int getWorkSheetColumns(int workSheet_Index);

    // 保存（写操作时需要要保存）
    bool save(void);

    // 关闭资源（释放资源）
    void close(void);

private:
    // 设置窗体是否可见
    bool setVisible(bool visible = false);

    // 释放资源
    void release(void);

private:
    bool m_IsOpened;

    // EXCEL程序指针
    QAxObject *m_Excel;

    // 工作簿集
    QAxObject *m_WorkBooks;

    // 当前活动的工作簿
    QAxObject *m_WorkBook;

    // 当前活动工作簿的工作表集（即所有的sheet表）
    QAxObject *m_WorkSheets;

    // 工作簿（m_WorkBook）中工作表（workSheet）的个数
    int m_WorkSheetCount;

    // 文件名
    QString m_FileName;

};

#endif // EXCELENGINE_H
{% endcodeblock %}

- ExcelEngine.cpp
{% codeblock %}
#include "ExcelEngine.h"

ExcelEngine::ExcelEngine(void):
    m_IsOpened(false),
    m_Excel(nullptr),
    m_WorkBooks(nullptr),
    m_WorkBook(nullptr),
    m_WorkSheets(nullptr),
    m_WorkSheetCount(0),
    m_FileName()
{
    // 打开当前进程的COM并释放相关资源
    OleInitialize(0);
}

ExcelEngine::ExcelEngine(const QString file_Name):
    m_IsOpened(false),
    m_Excel(nullptr),
    m_WorkBooks(nullptr),
    m_WorkBook(nullptr),
    m_WorkSheets(nullptr),
    m_WorkSheetCount(0),
    m_FileName(file_Name)
{
    // 打开当前进程的COM并释放相关资源
    OleInitialize(0);
}

ExcelEngine::~ExcelEngine(void)
{
    release();
    // 关闭当前进程的COM并释放相关资源
    OleUninitialize();
}

// 打开Excel，如果不存在则创建
bool ExcelEngine::open(bool visible ,bool display_Alerts )
{
    // 检测文件名是否为空
    if (m_FileName.isEmpty())
    {
        // 返回异常（文件名不存在）
        return false;
    }

    QFile file(m_FileName);
		
    if (file.exists())
    {
        // 文件存在，但没有以只读状态打开成功
        if (!file.open(QIODevice::ReadOnly))
        {
            // 返回异常（文件存在但是未能以只读状态打开）
            return false;
        }

        file.close();
    }
    else
    {
        // 返回异常（文件不存在）
        return false;
    }

    // Excel的程序对象指针
    m_Excel = new QAxObject;

    // 链接Excel控件（用Office的Excel打开Excel文档）
    bool isOpened = m_Excel->setControl("Excel.Application");

    if (!isOpened)
    {
        // 用WPS打开Excel文件
        isOpened = m_Excel->setControl("ket.Application");

        // 如果Office和WPS都无法打开则返回false
        if (!isOpened)
        {
            // 返回异常（Excel和WPS打开excel文件均未成功）
            return false;
        }
    }

    // 不显示窗体
    m_Excel->dynamicCall("SetVisible(bool)", visible);

    // 不显示警告
    m_Excel->setProperty("DisplayAlerts", display_Alerts);

    // 获取Excel工作簿集合对象
    m_WorkBooks = m_Excel->querySubObject("WorkBooks");

    // 打开Excel文件
    m_WorkBooks->dynamicCall("Open(const QString &)",m_FileName);

    // 当前活动工作簿
    m_WorkBook = m_Excel->querySubObject("ActiveWorkBook");

    if (m_WorkBook == nullptr)
    {
        // 返回异常（当前活动工作簿打开失败）
        return false;
    }

    // 工作表集
    m_WorkSheets = m_WorkBook->querySubObject("WorkSheets");

    // 工作表的个数
    m_WorkSheetCount = m_WorkSheets->dynamicCall("Count").toInt();


    if(m_WorkSheets == nullptr)
    {
        // 返回异常（当前表格集WorkSheets不存在，打开失败!）
        return false;
    }

    m_IsOpened = true;
    return m_IsOpened;
}

// 设置窗体是否可见
bool ExcelEngine::setVisible(bool visible )
{
    if (m_Excel == nullptr)
    {
        return false;
    }
    else
    {
        m_Excel->dynamicCall("SetVisible(bool)", visible);
        return true;
    }
}

// 获取索引出的sheet（第一个就是1，不是0）
QAxObject *ExcelEngine::getWorkSheet(int sheet_Index)
{
    QAxObject *workSheet = m_WorkBook->querySubObject("Worksheets(int)",sheet_Index);
    return workSheet;
}

// 通过SheetName获取sheet
QAxObject *ExcelEngine::getWorkSheet(QString sheet_Name)
{
    QAxObject *workSheet = m_WorkBook->querySubObject("Worksheets(QString)",sheet_Name);
    return workSheet;
}

// 获取单元格内容
QString ExcelEngine::getCellString(int workSheet_Index,int row,int column)
{
    QAxObject *workSheet = getWorkSheet(workSheet_Index);

    if(workSheet == nullptr)
    {
        return "";
    }

    QAxObject *range = workSheet->querySubObject("Cell(int,int)",row,column);

    if(range == nullptr)
    {
        return "";
    }

    QString cellTdext = range->dynamicCall("Value2()").toString();

    return cellTdext;
}

// 获取单元格内容
QString ExcelEngine::getCellString(QString workSheet_Name,int row,int column)
{
    QAxObject *workSheet = getWorkSheet(workSheet_Name);

    if(workSheet == nullptr)
    {
        return "";
    }

    QAxObject *range = workSheet->querySubObject("Cells(int,int)",row,column);

    if(range == nullptr)
    {
        return "";
    }

    QString cellTdext = range->dynamicCall("Value2()").toString();

    return cellTdext;
}

// 获取workSheet的行数
int ExcelEngine::getWorkSheetRows(QString workSheet_Name)
{
    QAxObject *workSheet = getWorkSheet(workSheet_Name);

    if(workSheet == nullptr)
    {
        return 0;
    }

    // 获取权限
    QAxObject *usedrange = workSheet->querySubObject("Usedrange");
    int rows = usedrange->querySubObject("Rows")->property("Count").toInt();
    return rows;
}

// 获取workSheet的行数
int ExcelEngine::getWorkSheetRows(int  workSheet_Index)
{
    QAxObject *workSheet = getWorkSheet(workSheet_Index);

    if(workSheet == nullptr)
    {
        return 0;
    }

    // 获取workSheet表中的有效区域
    QAxObject *usedrange = workSheet->querySubObject("Usedrange");
    int rows = usedrange->querySubObject("Rows")->property("Count").toInt();
    return rows;
}

// 获取workSheet的列数
int ExcelEngine::getWorkSheetColumns(QString workSheet_Name)
{
    QAxObject *workSheet = getWorkSheet(workSheet_Name);

    if(workSheet == nullptr)
    {
        return 0;
    }

    // 获取workSheet表中的有效区域
    QAxObject *usedrange = workSheet->querySubObject("Usedrange");
    int columns = usedrange->querySubObject("Columns")->property("Count").toInt();
    return columns;
}

// 获取workSheet的列数
int ExcelEngine::getWorkSheetColumns(int workSheet_Index)
{
    QAxObject *workSheet = getWorkSheet(workSheet_Index);

    if(workSheet == nullptr)
    {
        return 0;
    }

    // 获取workSheet表中的有效区域
    QAxObject *usedrange = workSheet->querySubObject("Usedrange");
    int columns = usedrange->querySubObject("Columns")->property("Count").toInt();
    return columns;
}


// 通过索引值获取工作表（workSheet）的表名
QString ExcelEngine::getWorkSheetNameByIndex(int workSheet_Inedx)
{
    QAxObject *workSheet = getWorkSheet(workSheet_Inedx);

    if(workSheet == nullptr)
    {
        return "";
    }

    QString sheetName = workSheet->dynamicCall("Name").toString();
    return sheetName;
}

// 获取WorkSheets工作表的表名列表
QStringList ExcelEngine::getWorkSheetNameList(void)
{
    QStringList workSheetsList;

    for(int i = 1;i<=m_WorkSheetCount;i++)
    {
        QAxObject *workSheet = getWorkSheet(i);
        QString sheetName = workSheet->dynamicCall("Name").toString();

        workSheetsList.append(sheetName);
    }

    return workSheetsList;
}

// 保存（用于写操作）
bool ExcelEngine::save(void)
{
    if(!m_IsOpened)
    {
        return false;
    }

    QFile file(m_FileName);

    if(file.exists())
    {
        m_WorkBook->dynamicCall("Save()");
    }else
    {
        return false;
    }

    return true;
}

// 关闭资源（释放资源）
void ExcelEngine::close(void)
{
    release();   
}

// 释放资源
void ExcelEngine::release(void)
{
    m_IsOpened = false;

    if(m_WorkBook != nullptr)
    {
        m_WorkBook->dynamicCall("Close(QVariant)",0);
        delete m_WorkBook;
        m_WorkBook = nullptr;
    }

    if(m_WorkBooks != nullptr)
    {
        m_WorkBooks->dynamicCall("Close(QVariant)",0); // 报错（QAxBase: Error calling IDispatch member Close: Bad parameter count）
        delete m_WorkBooks;
        m_WorkBooks = nullptr;
    }

    if(m_Excel != nullptr)
    {
        m_Excel->setProperty("DisplayAlerts", false);
        m_Excel->dynamicCall("Quit()");
        delete m_Excel;
        m_Excel = nullptr;
    }
}
{% endcodeblock %}

# 开发环境配置
1. 如果仅仅将[Excel读取库](#Excel读取库)中的`.h`与`.cpp`文件放到自己的项目中，然后调用库文件操作Excel文件
{% codeblock %}
#include <QtCore/QCoreApplication>
#include "ExcelEngine.h"
#include <qstring>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

	QString excel_path = "D:\\Qt_Code\\GDP_Code_GUI\\disorder_data.xlsx";
	ExcelEngine excelOpt(excel_path);
	excelOpt.open();
	return 0;
}
{% endcodeblock %}
执行上述代码，编译器会报错***无法解析外部符号QAxObject***，此时需要执行以下两步
	- 右键点击项目->属性->c/c++->常规->附加包含目录 下添加`$(QTDIR)\include\ActiveQt`
	- 右键点击项目->属性->链接器->常规->附加库目录 下添加`D:\Qt\Qt5.12.2\5.12.2\msvc2017_64\lib`（此路径是用户安装Qt时的路径，找到安装路径下的lib文件夹即可）
	- 右键点击项目->属性->链接器->输入->附加依赖库 下添加Debug版本：`Qt5AxContainerd.lib`、`Qt5AxBased.lib`；Release版本：`Qt5AxContainer.lib`、`Qt5AxBase.lib`
	
2. 步骤1配置完成再执行上述代码时，编译器又会报错***无法打开Qt5AxContainerd.lib***，此时需要进行如下配置
	- 右键点击项目->Qt Project Settings->General->Qt Modules 中勾选`ActiveQt Container`与`ActiveQt server`

![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/QtExcel/qtexcel1.png)
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/QtExcel/qtexcel2.png)

# 快速读取Excel
## 读取Excel慢的原因
网上教程中用Qt操作Excel的方法采用如下代码：
{% codeblock %}
QVariant ExcelBase::read(int row, int col)
{
    QVariant ret;
    if (this->sheet != NULL && ! this->sheet->isNull())
    {
        QAxObject* range = this->sheet->querySubObject("Cells(int, int)", row, col);
        //ret = range->property("Value");
        ret = range->dynamicCall("Value()");
        delete range;
    }
    return ret;
}
{% endcodeblock %}
读取慢的根源在`sheet->querySubObject("Cells(int, int)", row, col)`，每次读取的是表中的一个单元格，并且`querySubObject`产生的`QAxObject*`最好手动删除，原因是虽然父级的`QAxObject`会管理内存，但是父级不析构，子对象就不会析构

## 快速读取Excel方法
- 一次调用`querySubObject`把所有数据读取到内存
- 采用`UsedRange`把所有用到的单元格范围返回，并使用属性`Value`把单元格的值获取到并存储在`QVariant`
- `QVariant`实际上是`QList<QList<QVariant>>`，表示一个表，需要将`QVariant`使用`.toList()`方法转换为`QList<QVariant>`
完整读取Excel表中范围内的数据代码如下：
{% codeblock %}
//获取工作表
QAxObject* sheet1 = excelOpt.getWorkSheet(1);	//sheet索引是从1开始的
QAxObject *usedRange = sheet1->querySubObject("UsedRange");

// 获取所有数据
QVector<QVector<QString>> vecDatas;//获取所有数据
QVariant var = usedRange->dynamicCall("Value");
foreach(QVariant varRow, var.toList()) {
	QVector<QString> vecDataRow;
	foreach(QVariant var, varRow.toList()) {
		vecDataRow.push_back(var.toString());
	}
	vecDatas.push_back(vecDataRow);
}
{% endcodeblock %}

# 参考链接
Qt读取解析Excel（可以直接使用）：https://blog.csdn.net/m0_37545861/article/details/120528790
QT界面开发-QAxObject 解析 excel 时报错error LNK2019: 无法解析的外部符号：https://blog.51cto.com/u_15127621/4238345
学习记录：vs2017+qt5关于QAxObject读取excel中数据问题：https://www.likecs.com/show-204016757.html
Qt Windows 下快速读写Excel指南：https://www.cnblogs.com/ybqjymy/p/14306196.html?ivk_sa=1024320u
