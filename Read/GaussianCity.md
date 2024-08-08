# GaussianCity: Generative Gaussian Splatting for Unbounded 3D City Generation

<p>&emsp;&emsp; 本篇文章文章作者团队来自于南洋理工大学S-Lab，是Scene Dreamer以及Dreamer City的续作。<br>
&emsp;&emsp;本篇文章旨在使用3DGS技术进行无边界三维城市模型生成。本文创新点如下：<br>
(1)目前首个利用3DGS技术进行无边界场景生成的模型。<br>
(2)本文通过提出紧凑3D 场景表示方法——BEV-point确保了生成无边界场景的连续性并降低了场景生成过程中随尺度增加的内存开支<br>
(3)本文提出Spatial-aware Gaussian Attribute Decoder来利用局部结构化信息从而生成三维高斯的属性<br>
(4)实验结果表明，在GGE以及KITTI-360两个数据集上的生成效果达到了SOTA；同时相较于NeRF显著提高了生成效率<br>


## 模型
&emsp;&emsp;如下下图所示，本文模型主要分为三个部分，分别是BEV-point初始化，BEV-Point特征生成，BEV-Point解码。

### 1.BEV点初始化
<p>&emsp;&emsp;同样采取之前Scene Dreamer提出的局部框生成的思想以及BEV Representation的思想。这里初始的BEV-MAP由Semantic Map以及Height Map组成，并通过局部生成。<br>
&emsp;&emsp;为减少参与优化的点数目，采用了下面两种改进策略：密度图＆可见性判断<br>
&emsp;&emsp; （1）引入density map:原理在于反映不同特征的地物需要的点树木不同，即简单纹理地物较复杂纹理地物表示所需要的点数目较少。因此利用density map（密度图）来调整不同地物表征的点数目<br>
&emsp;&emsp;（2）引入可见性筛选：原理在于只需要表面可见点便可以反映地物特征，因此引入可见性图，通过筛选可见点，只对符合要求的可见点进行计算。<br>                     
&emsp;&emsp;只处理可见点，不仅可以减少参与优化点的数量，而且避免随着生成尺度增加，VRAM开支变大。

### 2.BEV点特征生成

#### 概述
<p>BEV点特征包含：<br>
（1）实例Attributes:实例的标签、尺寸以及中心坐标<br>
（2）BEV点Attributes:反映实例表面特征<br>
（3）风格查找表：利用表格存储不同实例之间的风格

#### 实例特征：
<p>&emsp;&emsp;输入语义图，利用联通性判断，获取分割不同Instance的Instance map。接下来利用世界坐标信息，获取每个Instance对应的坐标点信息。

#### BEV点属性
<p>&emsp;&emsp;关于点的信息均使用绝对的世界坐标系进行定义。这里引入相对坐标系，坐标原点位于每一个实例的中心，从而将每个Instance中的点进行归一化建模。<br>
&emsp;&emsp;同时利用编码器对高度图与语义图进行编码，从而获取各个绝对坐标下的点位对应的场景特征。

#### 风格查询表
&emsp;&emsp;为了降低内存开支，使用编码器对不同实例进行编码，利用获取的隐向量进行风格存储。

### 3.BEV解码器
&emsp;&emsp;BEV点解码器利用BEV特征生成高斯属性。该部分包含以下五个模块：（1）位置编码器(positional encoder)；（2）点顺序器(point serializer)；（3）点transformer(point transformer)；（4）调制化MLP(modulated MLP)；（5）高斯光栅化(Gaussian rasterizer)。

#### 位置编码器
&emsp;&emsp;通过点编码器将点位编码成为高维度的点位嵌入层。具体做法为：首先将连接绝对坐标信息与场景特征；输入连接结果到位置编码函数中。该函数类似于NeRF嵌入层函数。

#### 点顺序器
<p>&emsp;&emsp;BEV点与3D Gaussians由于点云分布特征保持非结构化与非顺序化。因此若使用MLP进行位置嵌入则忽略了点云及其附近点云语义与结构特征。因此需要将点进行结构化<br>
&emsp;&emsp;首先将BEV点的坐标转化为有序整数(利用每个网格尺寸进行二次项拟合，从而使得点依据空间位置进行排列，邻近位置更加靠近)。从而利用有序整数坐标获取场景特征。

#### 点transformer
&emsp;&emsp;利用transformer处理顺序化之后的场景特征，获得新的Transformer特征。

#### 调制化MLP
&emsp;&emsp;这里主要是利用MLP获取三维高斯属性。首先拼接Transformer特征以及对应的点顺序特征。然后将拼接特征、风格编码以及Instance的标签信息输入到MLP中生成三维高斯属性。

#### 高斯光栅化
&emsp;&emsp;使用相机内外参数进行高斯光栅化。这一过程中若3D Gaussian没有在上一步生成对应属性，那么则使用默认参数进行光栅化。

### 4.损失函数
&emsp;&emsp;这一重建过程中，主要包含了重建损失以及生成对抗损失。包括L1损失、VGG损失、GAN损失三部分（类似于之前两个工作）


## 实验
### 1.实验数据集
<p>&emsp;&emsp;实验使用数据集包含了Google Earth以及KITTI-360两个数据集。其中，Google Earth数据集在City Dreamer中已经使用。<br>
&emsp;&emsp;而KITTI-360则是街景数据集，路程长度共73.7km，提供了包含道路、车辆、建筑物等物体的三维标注。该数据集中的数据包含了丰富的高亮与阴影变换，反映了物体表面丰富的变换。

### 2.评估手段
&emsp;&emsp;这里评估手段主要包含：FID＆KID，Depth Error，Camera Error、Runtime。

### 3.相关细节＆结果
<p>&emsp;&emsp;作者提供了训练数据细节以及超参数的设置信息。<br>
&emsp;&emsp;作者利用不同模型在上述两个模型上进行测试，并进行了用户反馈研究

### 4.消融实验
<p>&emsp;&emsp;本文的消融实验主要是针对BEV-Point Representation以及BEV-Point decoder进行的。</p>
<p>&emsp;&emsp;BEV-Point Representation:通过光线交汇的可视性判断，BEV-Point Representation可以使得随着场景尺寸增加的内存开支基本保持为均值。通过两种方法点的数量与占用的显存关系图以及点的数量与占用内存的关系图来反映。<br>
&emsp;&emsp;此外作者还测试了基于场景或者基于实例的点过滤方法，结果表明还是可见性过滤方案的效果最佳。</p>

SUM:作者消融实验的思路：首先消除组件来判断组件对实验结果的影响；其次更换组件来证明组件的优越性。

<p>&emsp;&emsp;BEV-Decoder:由于BEV-Point Decoder由Point Serializer以及Point Transformer组成，因此分别对这两部分进行消融实验，通过FID、KID、CE、DE来反映两部分组件的作用。<br>
&emsp;&emsp;此外作者还将Serializer的方式替换为公式13以及希尔伯特曲线来进行实验，通过上述四种参数反映本文Serializer的重要作用。

## 局限性
&emsp;&emsp;本文局限主要在于通过根据将BEV点初始化高度抬升的假设，默认了建筑符合曼哈顿假设，无法表示空心结构。其次，解码器没有充分利用3DGS的能力，（仅仅使用了其RGB预测结果），然而预测过多属性会破坏训练效果。
