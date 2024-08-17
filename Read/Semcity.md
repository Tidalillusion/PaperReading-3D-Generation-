# SemCity: Semantic Scene Generation with Triplane Diffusion


<p>&emsp;&emsp;作者团队主要来自于KAIST（韩国国家科学院）。

<p>&emsp;&emsp;创新点：<br>
&emsp;&emsp;1.使用Triplane Diffusion model进行户外场景生成，有效应对真实户外场景生成存在的非结构性与稀疏性问题。<br>
&emsp;&emsp;2.无缝的Triplane Diffusion model 很好地进行下游的场景 inpainting、outpainting以及场景补全改善。<br>
&emsp;&emsp;3.利用Semantic KIITTI数据集实验，结果表明在场景生成与下游任务均达到了SOTA，并且模型可以很好地扩展到城市尺度。

## 实验模型
### overview
<p>&emsp;&emsp;本文模型Pipeline如下图所示。生成过程主要由两个部分组成：Triplane learning（编码解码器）以及Triplane diffusion（Triplane生成器）两个部分组成。<br>
&emsp;&emsp;其中编码解码器部分主要是利用MLP将体素编码成Triplane Representation然后再解码成体素；生成部分则是利用DDPM进行Triplane的训练。

### 模型
#### Representing a semantic scene with Triplane
<p>&emsp;&emsp;上一部分提到三维场景的自动编码器由将三维体素映射到Triplane的编码器以及将Triplane解码到三维场景的MLP解码器组成。</p>
<p>&emsp;&emsp;编码器:输入的空间分辨率为X×Y×Z，通过三维卷积层，将输入的三维体素映射为对应的三个平面向量，特征形状为为（C×X×Y）、（C×X×Z）以及（C×Y×Z），其中C为特征维度。这样给定任意一个空间的三维坐标可以映射为三个平面特征向量的和。</p>
<p>&emsp;&emsp;解码器：旨在通过MLP根据输入的Triplane重建三维场景并预测各个点的语义信息。具体做法为，将每个点位的Triplane以及高频sincos位置嵌入向量进行拼接得到特征向量，利用拼接后的特征向量（位置嵌入获取了高频的位置特征）输入到MLP中进行重建。</p>
<p>&emsp;&emsp;全过程损失：整个过程的自动编码器损失由两部分组成：（1）预测概率与标签的cross entropy；（2）预测概率与标签的Lovasz-softmax并使用参数平衡两个损失权重。（旨在进行不平衡场景预测）</p>

#### Triplane Diffusion
<p>&emsp;&emsp;扩散模型旨在通过学习编码得到的Triplane的概率分布，据此进行新的概率分布随机化，从而获得新的Triplane，再利用解码器生成新的场景。具体的增加噪声与去除噪声以及采样的原理与DDPM相同这里不再赘述。<br>
&emsp;&emsp;值得注意的是：这里同样需要利用点位坐标信息进行嵌入与Triplane特征向量进行拼接，利用拼接后的特征向量进行场景重建。</p>

### 实验
#### 实验细节
<p>&emsp;&emsp;1. 实验数据集：本次实验使用Semantic KITTI数据集与CarlaSC数据集。<br>
&emsp;&emsp;Semantic KITTI数据集：为真实采集的户外数据集提供具有20个类别真实户外的语义场景。每个场景由体素(256,256,32)体积表示，等效于真实场景距离中心点各边的长度为（51.2m，51.2m，6.4m），此外数据集包含了物体运动轨迹信息，用以建立密集地面实况。<br>
&emsp;&emsp;CarlaSC数据集则不含轨迹信息。每个网格体素距离（128,128,8），真实场景的距离为（25.6,25.6,3）。</p>

<p>&emsp;&emsp;2.实验细节：主要是作者实验设备、超参数设置（编码器获取特征向量的特征维度、解码器重建的空间分辨率、损失权重、扩散过程学习率变化、训练步数）</p>

<p>&emsp;&emsp;
3.评估：利用三维场景的保真度（IS）与多样性（召回率）来评估生成场景的质量。使用FID与KID来反映两者的综合。下游的场景完善工作使用IoU与mIoU作为指标来评估相关表现

#### 实验结果——场景生成
<p>&emsp;&emsp;如图为评价指标。

<br>&emsp;&emsp;可以发现在CARlaSC数据集上，SSD（Baseline）的表现比较好，但是在Semantic KITTI上表现明显不如本文提出的Diffusion Triplane。这是因为SSD基于体素进行三维Representation 不能很好面对真实场景更为普遍的非结构以及稀疏分布问题，尤其反映在生成道路与建筑物的边界问题上与更为精细化场景上（例如树干分布）。<br>
&emsp;&emsp;对比之下，本文提出的方法在保真性与多样性均优于Baseline，同时由于采用隐式的生成，因此生成结果的分辨率并不固定。

#### 实验结果——下游应用
<p>&emsp;&emsp;本文提出的方法在inpainting、outpainting以及Completion Refinement上均表现优于Baseline，同时可以直接利用Control Net进行真实场景绘制。

#### 消融实验
<p>&emsp;&emsp;消融实验主要从两个部件考虑：三维重建时的位置嵌入层、Triplane Representation。<br>

&emsp;&emsp;嵌入层：通过原结果与不进行操作结果实验，得到上表。根据上表评价指标所示，嵌入层通过提供位置的高频信息，提高了生成结果的保真性（高频特征尤其促进了生成场景的细节）<br>
&emsp;&emsp;Triplane Representation：这里主要讲Triplane Representation 替换为XY-plane与体素进行实验。实验结果表明Triplane可以明显提高生成结果的质量。

#### 实验不足
模型局限性在于生成的场景取决于训练数据的分布。此外还有如下局限性：
（1）视角局限性，传感器无法呈现的区域无法建模；
（2）数据集从驾驶视角捕捉，无法建模高度信息；
（3）数据集包含运动帧数信息，因此建模也会有运动痕迹。<br>
作者认为之后工作可以聚焦于使用先验知识。
