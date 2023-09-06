Implement Ordered Dithering opacity mask in Unreal Engine

## Ordered Dithering介绍

### 关于Dithering（颜色抖动）

在计算机成像时，得到的数字图像是对真实世界的采样。其中涉及到三个维度的量化：

**采样帧率：**即单位时间内采集的图像数量，比如常见的30FPS或者60FPS等；

**图像空间分辨率：**即我们常说的图像的宽高分别是多少，反映为图像有多少像素，比如1280*720；

**颜色位深度：**比如真彩色为24位，灰度图为8位，体现在每个像素可以表示多少种颜色。

颜色抖动则是尝试用较低的颜色位深度来获得更为丰富的视觉效果，比如用1位的位深度来尽可能得到8位的视觉效果。与颜色抖动相似的一个概念是半调（Halftone），半调是一种用尺寸不同的各个离散点来表示有连续灰度变化的图像的方式。最早被应用于印刷行业，如下图为纽约时报中的一幅图，通过控制印刷时油墨点的大小来实现视觉上的灰度变化：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-1024x594.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image.png)

随着显示技术和印刷技术的改进，半调多转变为了一种艺术风格，如经常会在网上看到这种点半调或者线条半调的艺术效果，称之为数字半调：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-1.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-1.png)

可见，上述的半调或者数字半调只包含黑白两种颜色，却能表征出连续的灰度变化，但是会牺牲空间分辨率。而颜色抖动则是在不牺牲空间分辨率情况下来实现半调。如下图所示，展示了四种技术对8位的Lenna图量化为1位后的结果：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-2.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-2.png)

均匀量化（Uniform Quantize）直接根据固定的灰度阈值将图像二值化，丢失了很多细节信息；而右下角的误差扩散（Error Diffusion）方法则得到几乎和原图一致的视觉效果。

## 关于Ordered Dithering（有序颜色抖动）

### 概述

Ordered Dithering的原理是使用一个阈值矩阵，应用在原来的图像的每个像素上，根据该像素是否超过对应的阈值，来选择最近的可用颜色。

如何理解呢？举个简单的例子，我们用下面的矩阵来代表一张4×4的灰度图，每个值代表该像素的亮度：

[0.5,0.5,0.5,0.5]  
[0.5,0.5,0.5,0.5]  
[0.5,0.5,0.5,0.5]  
[0.5,0.5,0.5,0.5]

如果我们要将这张灰度图变为一张只能使用0和1的纯黑白图，就可以用下面的2×2阈值矩阵来和原矩阵进行比较：

[ 0 , 0.5 ]  
[0.75, 0.25]

（关于这个矩阵是怎么来的，请看下面的阈值矩阵部分）

把阈值矩阵平铺在原图上：

[ 0 , 0.5 , 0 , 0.5]  
[0.75,0.25,0.75,0.25]  
[ 0 , 0.5 , 0 , 0.5]  
[0.75,0.25,0.75,0.25]

逐个进行比较，如果原图上的值超过对应的阈值，则设为1，没超过则设为0，则得到结果矩阵：

[1,0,1,0]  
[0,1,0,1]  
[0,1,0,1]

得到的这张黑白图，应该和原来的灰度图看起来的视觉效果相近。

下图是不同灰度通过4×4的阈值矩阵处理得到的效果的展示：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-8.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-8.png)

### 阈值矩阵

阈值矩阵有各种大小，通常为 2 的幂：

![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-4.png)

![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-5.png)

![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-6.png)

你可以直接使用这些矩阵，也可以通过数学方法得到这些矩阵。首先用连续的整数填充矩阵的每个位置。然后对它们重新排序，使矩阵中两个连续数字之间的平均距离尽可能大，确保表格在边缘“环绕”。对于维度为 2 的幂的阈值矩阵，可以通过以下方式递归生成：

![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-7.png)

这个函数也可以只用位算术来表示：

M(i, j) = bit_reverse(bit_interleave(bitwise_xor(i, j), i)) / n ^ 2;

### 为什么用Ordered Dithering实现半透明？

在 3D 游戏中，“半透明”永远是开发者的痛（因为半透明物体不能直接在 deferred 中完成绘制，以及排序困难）。而角色这样的非凸多面体存在透视问题——比如透过人体表面看到眼球、舌头或另一条手臂，这会严重影响体验的。因此不能通过使用半透明 shader 解决问题。

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-3.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-3.png "直接让材质半透明的效果不尽人意")

直接让材质半透明的效果不尽人意

另一种思路是使用不透明蒙版来模拟半透明的效果。我们知道不透明蒙版贴图中，只有黑和白两种颜色，只分为透明和不透明，那如何使用不透明蒙版来达到半透明的效果呢？前面我们说的Dithering就是用来解决这种问题的，即使我们只有黑白两种颜色，但我们仍能通过一些特殊的规则来模拟各种灰度，以达到在不透明蒙版中使用“灰度图”的目的。

游戏当中比较常见的就是利用Ordered Dithering来控制不透明度，比如原神中的半透明效果：

## 实现

本文将展示如何在UE中利用Ordered Dithering实现当摄像机离物体过近时有个淡出的效果。大家可以各取所需。

大致思路是，使用一个材质函数，将生成一个不透明蒙版贴图输入到材质的不透明蒙版节点中。

首先创建一个材质函数：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-9.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-9.png)

### Ordered Dithering实现

我们需要使用一个Custom节点，来使用我们自定的Ordered Dithering算法，custom节点的介绍请看：[虚幻引擎自定义材质表达式 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/custom-material-expressions-in-unreal-engine/#custom)。

创建Custom节点：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-10.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-10.png)

将以下代码添加到Custom节点的细节面板的代码中：

```C++
//阈值矩阵
float threshold[8][8]={
	{0.,32.,8.,40.,2.,34.,10.,42.},
	{48.,16.,56.,24.,50.,18.,58.,26.},
	{12.,44.,4.,36.,14.,46.,6.,38.},
	{60.,28.,52.,20.,62.,30.,54.,22.},
	{3.,35.,11.,43.,1.,33.,9.,41.},
	{51.,19.,59.,27.,49.,17.,57.,25.},
	{15.,47.,7.,39.,13.,45.,5.,37.},
	{63.,31.,55.,23.,61.,29.,53.,21.}
};
//通过输入的所需的透明度，对像素进行处理
if(v <= threshold[x % 8][y % 8] / 64)
{
	return 0.;
}
else
{
	return 1.;
}
```

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-11.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-11.png)

这段代码很简单，注释都写清楚了，就不多做介绍。

描述当中我们将节点起名为GetOrderedDitheringOpacityMask。并在输入中添加3个成员：x,y,v，这三个参数分别代表像素的坐标x值，像素的坐标y值，需要的不透明度。v的值介于0到1之间。

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-12.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-12.png)

我们需要把屏幕上的像素位置输入给函数，使用ScreenPosition节点和BreakOutFloat2Component节点，按照下图连接：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-13.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-13.png)

### 根据摄像机离物体的距离进行过渡

我们还需要输入一个透明度，这里的透明度我们通过计算物体与摄像机的之间的距离来得到。

使用ActorPositionWS节点和CameraPositionWS节点可获得物体的位置向量和摄像机的位置向量，再用Distance节点可以得到两点之间的距离：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-14.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-14.png)

然后我们还需要规定一个过渡的范围，即摄像机在这一距离范围内透明度从0到1进行变化。

这里我直接使用常量来指示范围，你也可以使用ScalarParameter节点，让材质实例可以设置各自的虚化距离范围。

（如何添加常量？按住大键盘的1，然后在蓝图上左键点击即可创建一个常量，同理，按住2和3还可以创建二维和三维向量）

设置摄像机在200到600的范围内进行过渡：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-15.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-15.png)

通过一些数学计算，可以得到需要的透明度Alpha值：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-17.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-17.png)

稍作解析，上面一个Subtract通过摄像机距离减去200，举个例子，摄像机目前和物体的距离是500，距离200位置的差就是300，下面一个Subtract得到的是600和200之间的距离差，用这两个值相除，就可以得到摄像机所处位置在这整一段距离中的“进度”百分比，他们之间的关系可以理解成一个进度条：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/%E6%91%84%E5%83%8F%E6%9C%BA%E8%B7%9D%E7%A6%BB.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/%E6%91%84%E5%83%8F%E6%9C%BA%E8%B7%9D%E7%A6%BB.png)

300在400中的占比是75%，在这个位置我们需要的不透明度就是75%。使用Saturate节点可以将值限定在0到1之间，输入值大于1时输出值等于1，输入值小于0时输出值等于0，输入值在0到1之间则输出值就是输入值。

我们把得到的值输入进我们自己写的GetOrderedDitheringOpacity节点的v参数中，并把GetOrderedDitheringOpacity的输出值连到Output Result节点上，就已经可以在左侧的预览窗口中看到效果了：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-18.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-18.png)

### 将材质函数应用到材质

这个材质函数已经能获得我们的不透明蒙版贴图了，让我们把这个材质函数加入到我们的材质当中。

先将我们的材质函数公开到库以供其他材质使用：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-22.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-22.png)

用UE自带的方块举例（仅用作教学用途，尽量不要修改引擎自带的材质，如果你有自己的需要透明化的网格体，请在你自己的网格体的材质上进行这些操作），向场景中加入一个方块：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-19.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-19.png)

在内容浏览器中打开材质：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-20.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-20.png)

将材质的混合模式改为已遮罩以使用不透明蒙版：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-21.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-21.png)

右键并输入函数名来加入函数（我的函数起名fade），并连入不透明蒙版：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-23.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-23.png)

此时在左侧已经可以预览到效果：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-24-1024x619.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-24.png)

点击左上角的保存，即可应用到世界的物体上，此时进入场景查看原来的方块，已经有了摄像机靠近时的透明效果：

[![](https://www.zeromes.cn/wp-content/uploads/2023/02/image-25.png)](https://www.zeromes.cn/wp-content/uploads/2023/02/image-25.png)

## 参考文章

1.[关于颜色抖动（dithering） – 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/510251407)  
2.[Ordered dithering (shadertoy.com)](https://www.shadertoy.com/view/4lcyzn#)  
3.[有序抖动 – 维基百科 (wikipedia.org)](https://en.wikipedia.org/wiki/Ordered_dithering)  
4.[让角色半透明：从 Ordered Dithering 说起（一） | indienova 独立游戏](https://indienova.com/u/patheagames/blogread/4673)

 #Shader #UE #计算机图形学