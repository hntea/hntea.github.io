树莓派3 ROS语音包开发之端点检测
------------------

　　继上篇博文，接下来讨论端点检测中的传统手段，短时过零率。何为短时过零率呢？这里再次贴出定义：
　　**短时过零率**：每帧内信号通过零值的次数。短时过零率可以在一定程度上反映信号的频率信息，因此是谱特性的一种粗略估计。对于离散信号，如果相邻的取样值改变符号则称为过零，过零率就是样本改变符号的次数！

　　**作用：**
　　可以用来粗略判断清、浊音。过零率高对应可能是清音，过零率低可能是浊音。这是由于清音的能量多数出现在较高的频率。可以利用短时过零率从背景噪声中找到语音信号，可以用来判断寂静无声段和有声段的起点和终点位置。
　　　
首先还是先贴出图片，这样可以更直白的认识：

![短时能量图](http://img.blog.csdn.net/20160823162312731)
![短时过零率](http://img.blog.csdn.net/20160823162342936)

**短时过零率的数学定义：**
$$ Z_n=1/2\sum_{m=0}^{N-1} {|sgn[x_n(m)]-sgn[x_n(m-1)]|}$$
其中sgn(x)为符号函数：
$$ sgn(x) =
\begin{cases}
 1,  & {x \ge0} \\ 
-1, & x\lt0
\end{cases}$$


有了离散数学表达式，这下好办。接下来就是如何编码测试的问题了。先来解决一个符号函数，就这么搞吧，多么简单。

```
//符号函数
int sgn(short* sample)
{
	int ret = 0;
	short tmp = *sample;
	if(tmp >= 0){
		ret = 1;
	}else{
		ret = -1;
	}
	return ret;		
}

```
对于离散信号来说，短时过零率就是相邻样本值改变符号的次数，这简单，根据上面的定义表达式可以编写如下代码：

```
/*
函数功能：计算帧缓存的短时过零率
参数说明：
	void* audioFramePtr：帧缓存指针
	int frame_per_buffer：帧缓存大小
*/
float shortTimeCrossZeroRate(short* audioFramePtr,int frame_per_buffer)
{
	//获取每一帧符号
	float rate = 0.0f;  
    for (unsigned long i = 0; i<frame_per_buffer; i++)  
    {  
        if((sgn(audioFramePtr + i + 1) - sgn(audioFramePtr + i)) < 0){
        	rate -= ( sgn(audioFramePtr+i+1) - sgn(audioFramePtr+i));
        }else{
        	rate += (sgn(audioFramePtr+i+1) - sgn(audioFramePtr+i));
        }  
    }  
  
    return (float)rate/2;  
}

```
迫不及待想来测试一下效果，赶集写个代码来实验一下，看看实验结果..

```
void ShortRatePrint(const char* file_wav)
{
	size_t size = 0;
	int  segment = 0;　　　//一个文件被分成多少段
	float rate = 0.0f;
	int i=0;
	int formate=0;　　　　　//打印格式标志
	int sum = 0	;		　//表示整个文件的过零率
	int frames = 0;
	AUDIO_INFO* info = NULL; 	
	info = audioInfoInit(8000,1,16);
	frames = getPerFrames(20,info,sizeof(short));
	short audioFrame[frames];

	
	FILE *stream = fopen(file_wav, "r+");
	if(stream == NULL){
		printf("open file err!\n");
		exit(1);
	}
	//跳过文件头
	size = fread(audioFrame, sizeof(short),44,stream);
	bzero(audioFrame,44);
	printf("Short time cross zero rate for each frames:\n");
	while(size = fread(audioFrame, sizeof(short),frames,stream))
	{
		 rate = shortTimeCrossZeroRate(audioFrame,frames);	

		 if(i==12){
		 	 i=0;
		 	printf("\n");
		 }
		 i++;
		 segment++;
		 sum+=rate;
	}
 	printf( "%6.0f ",rate);
 	printf("\n Segment is %d\n",segment);
	printf("\nSum Rate is %d\n",sum);
 	fclose(stream);
 	audioInfoFree(info);	
}
```
　　在主函数中调用，博主自己编写了录音程序，短时录音并打印分析结果，看看结果博主也是一脸懵圈，来看看这都是些什么数据：

![工作室噪声图](http://img.blog.csdn.net/20160823215952292)

![噪声过零率](http://img.blog.csdn.net/20160823221141153)

　　现在单方面噪声很难看出所以然，我们来说几句话，看看有何变化。
![人声和噪声](http://img.blog.csdn.net/20160823221828405)

![人声和噪声过零率](http://img.blog.csdn.net/20160823221902349)
　　结合一开始的概念说明，想必你已经大概知道其中的端倪在哪里！对照两幅图片，其实在过零率很低的那一帧，博主认为可以其产生的原因还是因为人发声的突然性，这样在人发声音的时候，脉冲编码器采集到的数字信号有上升的趋势并带有相关的延时特性，所以会导致过零率下降！那么问题来了，怎么取出适当的阈值，这和上篇博客说的短时能量阈值的选取一样，又是一个比较麻烦的问题，至于问题如何解决，下篇博客继续探索！