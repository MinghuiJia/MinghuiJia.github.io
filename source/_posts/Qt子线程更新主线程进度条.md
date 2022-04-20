---
title: Qt子线程更新主线程进度条
excerpt: 在Qt多线程编程过程中，常常会遇到不同线程之间数据传递的需求。此博客则是通过一个简单的例子——子线程任务处理过程中更新主线程进度条，来介绍线程间数据传递的解决方法
index_img: https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Qt_Child_Thread_Updates_The_Main_Thread_Progressbar/step1.png
date: 2022-03-13 21:46:45
tags:
  - Qt多线程数据传递
categories:
  - Qt
---

# 前言
在Qt多线程编程过程中，常常会遇到不同线程之间数据传递的需求。此博客则是通过一个简单的例子——子线程任务处理过程中更新主线程进度条，来介绍线程间数据传递的解决方法
<!-- more -->

# 任务需求
由于程序存在耗时的操作，因此将此耗时的操作放在一个新的线程中从而避免界面“假死”。但是在耗时操作执行过程中，为了给用户友好的交互体验，需要利用进度条（ProgressBar）给用户展示程序实时处理的进度，这就涉及到在子线程中耗时操作的处理进度变量如何传递到界面主线程中

# 解决方案
## 定义信号函数
在子线程对应的类中定义信号函数，用于传递子线程任务进度变量
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
		void updateProgress(const int nNum);	//子线程信号函数（传递数据）
	};
{% endcodeblock %}

## 定义相应的槽函数
在主线程类中定义更新进度条的槽函数，其中更新所需的变量信息通过上一步子线程的信号函数进行传递
{% codeblock %}
	class QtWidgetsApplication1 : public QWidget
	{
		Q_OBJECT

	public:
		QtWidgetsApplication1(QWidget *parent = Q_NULLPTR);
		void updateProgress(int nNum);	//主线程槽函数（根据信号传递来的数据更新进度条）

	private slots:
		void on_pushButton_clicked();

	private:
		Ui::QtWidgetsApplication1Class ui;
	};
{% endcodeblock %}

{% codeblock %}
	void QtWidgetsApplication1::updateProgress(int nNum)
	{
		ui.progressBar->setValue(nNum);
	}
{% endcodeblock %}

## 建立信号与槽的联系
在主线程中建立信号与槽的联系
{% codeblock %}
	void QtWidgetsApplication1::on_pushButton_clicked()
	{
		QThread* m_workThread = new QThread();
		workThread* worker = new workThread();
		worker->moveToThread(m_workThread);

		//建立信号槽连接
		connect(worker, &workThread::updateProgress, this, &QtWidgetsApplication1::updateProgress); //建立子线程对象与主线程对象之间的信号与槽的连接
		connect(m_workThread, &QThread::started, worker, &workThread::start1);
		connect(worker, &workThread::workFinished, worker, &workThread::deleteLater);
		connect(worker, &workThread::workFinished, m_workThread, &QThread::quit);
		connect(m_workThread, &QThread::finished, m_workThread, &QThread::deleteLater);
		worker->m_bStart = true;
		m_workThread->start();
		worker->m_bStart = false;
	}
{% endcodeblock %}

## 发送信号
在子线程处理耗时（循环）任务时，不断发送信号，将数据传递给主线程用于更新进度条
{% codeblock %}
	void workThread::doWork()
	{
		for (int i = 0; i < 1000000; i++)
		{
			qDebug() << i;
			emit updateProgress(i + 1);		//发送信号（传递数据）
		}
		emit workFinished();	//发送信号，表明线程结束工作
	}
{% endcodeblock %}

## 初始化进度条的属性
{% codeblock %}
QtWidgetsApplication1::QtWidgetsApplication1(QWidget *parent)
    : QWidget(parent)
{
    ui.setupUi(this);
	ui.progressBar->setRange(0, 100);		//设置进度条显示的范围
	ui.progressBar->setMinimum(0);			//设置程序任务的最小数量，对应range的最小值
	ui.progressBar->setMaximum(1000000);	//设置程序任务的最大数量，对应range的最大值
}
{% endcodeblock %}

## 结果展示
程序运行截图如下：
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/Qt_Child_Thread_Updates_The_Main_Thread_Progressbar/step1.png)
此时，程序在执行1000000次循环输出，可以实时更新处理进度

# 辅助知识
connect函数的五个参数表示的意义依次为：`sender＊`，`signal`，`receiver＊`，`slot`，`connectionTpye`

其中槽可以是receiver的成员函数，或者是任意可访问的静态函数。