实验5、IHS遥感图像融合

一、实验过程

1、 将多光谱影像重采样到与全色影像具有相同的分辨率；

由于老师给的实验材料照片是处理过的，此步骤省略掉；

2、将多光谱图像的Ｒ、Ｇ、Ｂ三个波段转换到IHS空间，得到Ｉ、Ｈ、Ｓ三个分量；

读取各通道像素值，使用公式进行变换；

3、 以Ｉ分量为参考，对全色影像进行直方图匹配；

4、 用全色影像替代Ｉ分量，然后同Ｈ、Ｓ分量一起逆变换到RGB空间，从而得到融合图像；

二、实验中遇到的问题

1、实验问题，一开始用了Gyte型，计算精度不够，在听老师讲过后使用了GDT_Float32。

2、在写代码之前，阅读MATLAB程序，对于进行变换和逆变换时的矩阵运算没有看懂，误以为reshape函数是按行读取，在转化成C++时转化运算出错。

解决：就是按照前几个实验那样对每个像素值 进行相应运算；

三、实验代码

	include "stdafx.h"
	
	include <iostream>
	
	using namespace std;
	
	include "./gdal/gdal_priv.h"
	
	pragma comment(lib, "gdal_i.lib")
	
	int main()
	
	{
	
		GDALDataset* poMulDS;
	
		GDALDataset* poPanDS;
	
		GDALDataset* poFusDS;
	
	int imgXlen,imgYlen;
	
	char* mulPath = "American_Mul.bmp";
	char* panPath = "American_Pan.bmp";
	char* fusPath = "American_Fus.tif";
	
	int i,bandNum;
	float *bandR, *bandG, *bandB;
	float *bandI, *bandH, *bandS;
	float *bandP;
	
	GDALAllRegister();
	
	poMulDS = (GDALDataset*)GDALOpenShared(mulPath,GA_ReadOnly);
	poPanDS = (GDALDataset*)GDALOpenShared(panPath,GA_ReadOnly);
	
	//求取图像的像素值和通道数
	imgXlen = poMulDS->GetRasterXSize();
	imgYlen = poMulDS->GetRasterYSize();
	bandNum = poMulDS->GetRasterCount();
	
	poFusDS = (GetGDALDriverManager()->GetDriverByName("GTiff")->Create(fusPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL));
	
	//根据图像的宽度和高度分配内存
	bandR = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
	bandG = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
	bandB = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
	bandP = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
	bandI = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
	bandH = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
	bandS = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
	
	poMulDS->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen,bandR, imgXlen, imgYlen, GDT_Float32, 0, 0);
	poMulDS->GetRasterBand(2)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen,bandG, imgXlen, imgYlen, GDT_Float32, 0, 0);
	poMulDS->GetRasterBand(3)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen,bandB, imgXlen, imgYlen, GDT_Float32, 0, 0);
	poPanDS->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen,bandP, imgXlen, imgYlen, GDT_Float32, 0, 0);
	
	//将多光谱图像的Ｒ、Ｇ、Ｂ三个波段转换到IHS空间，得到Ｉ、Ｈ、Ｓ三个分量；
	for (i = 0; i < imgXlen*imgYlen; i++)
	{
	    bandH[i] = -sqrt(2.0f)/6.0f*bandR[i]-sqrt(2.0f)/6.0f*bandG[i]+sqrt(2.0f)/3.0f*bandB[i];
	    bandS[i] = 1.0f/sqrt(2.0f)*bandR[i]-1/sqrt(2.0f)*bandG[i];
		//用全色影像替代Ｉ分量，然后同Ｈ、Ｓ分量一起逆变换到RGB空间，从而得到融合图像。
	    bandR[i] = bandP[i]-1.0f/sqrt(2.0f)*bandH[i]+1.0f/sqrt(2.0f)*bandS[i];
	    bandG[i] = bandP[i]-1.0f/sqrt(2.0f)*bandH[i]-1.0f/sqrt(2.0f)*bandS[i];
	    bandB[i] = bandP[i]+sqrt(2.0f)*bandH[i];
	}
	
	poFusDS->GetRasterBand(1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen,bandR, imgXlen, imgYlen, GDT_Float32, 0, 0);
	poFusDS->GetRasterBand(2)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen,bandG, imgXlen, imgYlen, GDT_Float32, 0, 0);
	poFusDS->GetRasterBand(3)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen,bandB, imgXlen, imgYlen, GDT_Float32, 0, 0);
	
	CPLFree(bandR);
	CPLFree(bandG);
	CPLFree(bandB);
	CPLFree(bandI);
	CPLFree(bandH);
	CPLFree(bandS);
	CPLFree(bandP);
	
	GDALClose(poMulDS);
	GDALClose(poPanDS);
	GDALClose(poFusDS);
	return 0;
	}

四、实验结果

从图像融合结果可以看出，HIS图像融合方法，保留了多光谱图像的光谱特征信息的同时，也增强了图像的清晰度

结果图片链接：http://ww1.sinaimg.cn/large/006uTyYQly1fxmcri0grzj30sg0sg1kz.jpg