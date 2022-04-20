---
title: GDAL分块读写影像
excerpt: 在使用GDAL读取GeoTiff影像时针对特大的影像，有时为了减少内存消耗，对图像进行分块读取很有必要
index_img: https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/GDAL_Read_Write_Images_In_Block/index_img.jpg
date: 2022-01-20 10:31:56
tags:
 - GDAL读写
 - C++
categories:
 - GDAL
---

# 前言
在使用GDAL读取GeoTiff影像时针对特大的影像，有时为了减少内存消耗，对图像进行分块读取很有必要
<!-- more -->

# 代码实现
{% codeblock %}
// gdal分块并行处理tif影像
void GDALBlockReadWrite(string openFilePath, string saveFilePath, int nBlockSize)
{
	//打开影像
	GDALDataset *poSrcDS = (GDALDataset*)GDALOpen(openFilePath.c_str(), GA_ReadOnly);
	if (poSrcDS == NULL)	//影像打开失败
		return ;

	//获取图像的高宽,夜光影像只有一个波段
	int nXSize = poSrcDS->GetRasterXSize();	//列
	int nYSize = poSrcDS->GetRasterYSize();	//行
	int nBands = poSrcDS->GetRasterCount();

	//获取输入影像仿射变换参数
	double adfGeotransform[6] = { 0 };
	poSrcDS->GetGeoTransform(adfGeotransform);

	//获取输入影像空间参考
	const char* pszProj = poSrcDS->GetProjectionRef();

	//获取波段测试
	GDALRasterBand *poBand = poSrcDS->GetRasterBand(1);
	if (poBand == NULL)
	{
		GDALClose((GDALDatasetH)poSrcDS);
		return ;
	}
	int dataType = poBand->GetRasterDataType();	//获取波段1数据的类型

	//创建tif驱动和数据集
	string format = "GTiff";
	GDALDriver *saveDriver = GetGDALDriverManager()->GetDriverByName(format.c_str());
	GDALDataset *saveDataset = saveDriver->Create(saveFilePath.c_str(), nXSize, nYSize, nBands, GDT_Float32, NULL);
	saveDataset->SetGeoTransform(adfGeotransform);
	saveDataset->SetProjection(pszProj);

	//分配分块读取数据缓存
	float *pafScan = new float[nBlockSize*nBlockSize*nBands];

	//定义读取影像波段顺序
	int *pBandMaps = new int[nBands];
	for (int i = 0; i < nBands; i++)
		pBandMaps[i] = i + 1;

	//循环分块处理
	for (int r = 0; r < nYSize; r += nBlockSize)
	{
		for (int c = 0; c < nXSize; c += nBlockSize)
		{
			//两个变量记录分块大小
			int nXBlock = nBlockSize;
			int nYBlock = nBlockSize;

			//处理最后尺寸不足nBlockSize
			if (r + nBlockSize > nYSize)
				nYBlock = nYSize - r;
			if (c + nBlockSize > nXSize)
				nXBlock = nXSize - c;

			//读取原始影像块
			poSrcDS->RasterIO(GF_Read, c, r, nXBlock, nYBlock, pafScan, nXBlock, nYBlock, GDT_Float32, nBands, pBandMaps, 0, 0, 0, NULL);
			
			//分块读取的内容写入影像
			saveDataset->RasterIO(GF_Write, c, r, nXBlock, nYBlock, pafScan, nXBlock, nYBlock, GDT_Float32, nBands, 0, 0, 0, 0);
			GDALFlushCache(saveDataset);
		}
	}

	delete[]pBandMaps;
	delete[]pafScan;
	GDALClose((GDALDatasetH)poSrcDS);
	//delete[]poSrcDS;
}
{% endcodeblock %}
