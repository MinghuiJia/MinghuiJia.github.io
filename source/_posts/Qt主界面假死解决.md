---
title: Qt主界面假死解决
excerpt: 在利用Qt进行可视化编程时，某个过于耗时的操作会造成界面卡死现象。例如用户点击界面的Button控件，程序会执行clicked槽函数中的操作，但是槽函数中的操作特别耗时，此时界面处于卡死状态（用户无法进行任何操作，包括拖动界面）。为了解决这一“假死”现象，需要用到多线程
date: 2022-03-13 20:26:06
tags:
  - Qt可视化编程
  - 界面假死
categories:
  - Qt
---

# 前言
在利用Qt进行可视化编程时，某个过于耗时的操作会造成界面卡死现象。例如用户点击界面的Button控件，程序会执行clicked槽函数中的操作，但是槽函数中的操作特别耗时，此时界面处于卡死状态（用户无法进行任何操作，包括拖动界面）。为了解决这一“假死”现象，需要用到**多线程**。
<!-- more -->

# 多线程解决Qt主界面假死的原理
当前我们编写的程序大多都是单线程执行，并且Qt的可视化界面逻辑代码也与执行耗时操作的代码处在同一个线程中。因此遇到耗时操作时，程序需要等到耗时操作执行完成后再继续执行后续代码，界面中的交互功能也不能使用，呈现出“假死”状态。而采用多线程编程，将耗时操作放在新开辟的线程中可以避免上述问题。下面将详细介绍如何解决Qt主界面假死问题

# 构造新的线程类workThread
Qt线程的创建有两种方式：
- 继承QThread的方式，然后重写run，但是这种方式官方已经不推荐了
- 继承QObject，构建新的类然后move到新的线程中

构建新的类中，重点需要关注信号（signals）和槽（slots）的定义与实现
- signals包括：任务开始（workStart）与任务结束（workFinished）
- slots包括：线程开始（start1）与操作执行（doWork）
1. 在Widget.h文件中定义如下线程类：
{% codeblock %}
	class workThread : public QObject
	{
		Q_OBJECT
	public:
		workThread(QObject* parent = nullptr);
		~workThread();
		bool m_bStart;

	public slots:
		void start1();
		void doWork();

	signals:
		void workFinished();
		void workStart();
	};
{% endcodeblock %}
2. 在Widget.cpp文件中完善workThread类中的方法
{% codeblock %}
	workThread::workThread(QObject* parent) : QObject(parent)
	{
		this->m_bStart = false;
	}

	workThread::~workThread()
	{

	}
	void workThread::start1()
	{
		emit workStart();	//发送信号，表明线程开始工作
		doWork();			//开始执行线程内的操作
	}
	void workThread::doWork()
	{
		for (int i = 0; i < 1000000; i++)
		{
			qDebug() << i;
		}
		emit workFinished();	//发送信号，表明线程结束工作
	}
{% endcodeblock %}

# 给Button绑定clicked事件测试
- 给Button绑定clicked事件
- 在clicked槽函数中创建一个新的线程m_workThread对象
- 创建定义好的workThread对象worker，move到新线程中
- 完成相应的信号与槽的连接
{% codeblock %}
	void QtWidgetsApplication1::on_pushButton_clicked()
	{
		QThread* m_workThread = new QThread();
		workThread* worker = new workThread();
		worker->moveToThread(m_workThread);
		
		//建立信号槽连接
		connect(m_workThread, &QThread::started, worker, &workThread::start1);
		connect(worker, &workThread::workFinished, worker, &workThread::deleteLater);
		connect(worker, &workThread::workFinished, m_workThread, &QThread::quit);
		connect(m_workThread, &QThread::finished, m_workThread, &QThread::deleteLater);
		worker->m_bStart = true;
		m_workThread->start();
		worker->m_bStart = false;
	}
{% endcodeblock %}

程序运行截图如下：
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Qt_Main_Interface_Resolved_By_Feigning_Death/step1.png)
此时，程序在执行1000000次循环输出，而可视化界面不会卡死

***注意：因为此时界面和耗时程序执行操作在两个线程，如果后续需要传递数据的话，可以将数据通过信号槽的方式传递***