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

	#include <iostream>
	using namespace std;
	#include "./gdal/gdal_priv.h"
	#pragma comment(lib, "gdal_i.lib")
	int main()
	{	
		//注册驱动
		GDALAllRegister();
		//输入图像
		GDALDataset* poSrcDS_MUL;
		GDALDataset* poSrcDS_PAN;
		//输出图像
		GDALDataset* poDstDS_RES;
		//图像的宽度和高度 
		int imgXlen, imgYlen;
		//输入图像的路径
		char* srcPath_MUL = "American_MUL.bmp";
		char* srcPath_PAN = "American_PAN.bmp";
		//输出图像的路径
		char* dstPath = "result.tif";
		//图像的内存存储 
		float *DataBuff_R;
	    float *DataBuff_G;
	    float *DataBuff_B;
	    float *DataBuff_P;
	    float *DataBuff_I;
	    float *DataBuff_H;
	    float *DataBuff_S;
	    float *DataBuff;
		DataBuff_R = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
		DataBuff_G = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
		DataBuff_B = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
		DataBuff_P = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
		DataBuff_I = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
		DataBuff_H = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
		DataBuff_S = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
		DataBuff = (float*)CPLMalloc(imgXlen*imgYlen*sizeof(float));
		//图像的波段数；
		int i,bandNum;
		//打开图像
		poSrcDS_MUL = (GDALDataset*)GDALOpenShared(srcPath_MUL, GA_ReadOnly);
		poSrcDS_PAN = (GDALDataset*)GDALOpenShared(srcPath_PAN, GA_ReadOnly);
		//获取图像的宽度，高度和波段数量
		imgXlen = poSrcDS_MUL->GetRasterXSize();
		imgYlen = poSrcDS_MUL->GetRasterYSize();
		bandNum = poSrcDS_MUL->GetRasterCount();
		
		poDstDS_RES = (GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL));
		//读取MUL图的RGB
			poSrcDS_MUL->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, DataBuff_R, imgXlen, imgYlen, GDT_Float32, 0,0);
			poSrcDS_MUL->GetRasterBand(2)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, DataBuff_G, imgXlen, imgYlen, GDT_Float32, 0,0);
			poSrcDS_MUL->GetRasterBand(3)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, DataBuff_B, imgXlen, imgYlen, GDT_Float32, 0,0);
			poSrcDS_PAN->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, DataBuff, imgXlen, imgYlen, GDT_Float32, 0,0);
	
		//图像的处理
		for (i = 0; i < imgXlen*imgYlen; i++)
	    {
				//RGB转换为HIS
				DataBuff_H[i] = -sqrt(2.0f)/6.0f*DataBuff_R[i]-sqrt(2.0f)/6.0f*DataBuff_G[i]+sqrt(2.0f)/3.0f*DataBuff_B[i];
			    DataBuff_S[i] = 1.0f/sqrt(2.0f)*DataBuff_R[i]-1/sqrt(2.0f)*DataBuff_G[i];
	            //HIS转换为RGB
				DataBuff_R[i] = DataBuff[i]-1.0f/sqrt(2.0f)*DataBuff_H[i]+1.0f/sqrt(2.0f)*DataBuff_S[i];
			    DataBuff_G[i] = DataBuff[i]-1.0f/sqrt(2.0f)*DataBuff_H[i]-1.0f/sqrt(2.0f)*DataBuff_S[i];
				DataBuff_B[i] = DataBuff[i]+sqrt(2.0f)*DataBuff_H[i];
		}
	       	//将IHS影像的数据写入文件中
	        poDstDS_RES->GetRasterBand(1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, DataBuff_R, imgXlen, imgYlen, GDT_Float32, 0, 0);
	        poDstDS_RES->GetRasterBand(2)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, DataBuff_G, imgXlen, imgYlen, GDT_Float32, 0, 0);
	        poDstDS_RES->GetRasterBand(3)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, DataBuff_B, imgXlen, imgYlen, GDT_Float32, 0, 0);
	
		CPLFree(DataBuff_R);
		CPLFree(DataBuff_G);
		CPLFree(DataBuff_B);
		CPLFree(DataBuff_H);
		CPLFree(DataBuff_I);
		CPLFree(DataBuff_S);
		CPLFree(DataBuff);
		GDALClose(poDstDS_RES);
		GDALClose(poSrcDS_PAN);
		GDALClose(poSrcDS_MUL);
		GDALClose(poSrcDS_MUL);
		system("PAUSE");
		return 0;
	}
	

四、实验结果

从图像融合结果可以看出，HIS图像融合方法，保留了多光谱图像的光谱特征信息的同时，也增强了图像的清晰度

结果图片链接：http://ww1.sinaimg.cn/large/006uTyYQly1fxmcri0grzj30sg0sg1kz.jpg