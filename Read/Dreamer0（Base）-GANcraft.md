# GANcraft: Unsupervised 3D Neural Rendering of Minecraft Worlds

![author](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/GANcraft1_writers.png)

作者团队来自于康奈尔大学与英伟达。
<p>&emsp;&emsp;创新点：<br>
&emsp;&emsp;1.提出了非监督的神经渲染框架，该框架可以用真实三维图像渲染block，从而将block场景（世界）渲染为真实场景（世界）。<br>
&emsp;&emsp;2.提出了一种基于为地面图像进行渲染的对抗训练方式实现了三维渲染<br>
&emsp;&emsp;3.通过伪GT影像训练模型从而解决没有真实数据的问题。<br>
&emsp;&emsp;4.与基线对比实验表明，GANcraft在三维block世界合成的效果很好。</p>

<p>&emsp;&emsp;讨论：在输入几何形状粗糙的情况下，学习使用点云或mesh等非体素进行输入。

## 场景生成工作遇到的挑战与改进
### 工作难点
<p>&emsp;&emsp;1.需要在三维场景中影像与标签一一对应的数据，而缺乏这样的数据集；<br>
&emsp;&emsp;2.生成模型生成的影像难以确保其连续性。

### 本文改进
<p>&emsp;&emsp;1.在大规模数据集上预训练生成模型并微调模型后，设置相机采样参数，生成伪GT影像作为数据集进行训练；<br>
&emsp;&emsp;2.结合风格特征向量，对场景中每个体素均进行两步渲染，渲染结果既具备泛化性，质量也超过Baseline。<br>
下图为生成伪GT影像示例：
  
![SPADE](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/GANcraft2_Pretrain.png)

## 模型结构
### overview
<p>&emsp;&emsp;如下图所示为GAN craft的模型结构。<br>

![pipeline](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/GANcraft3_pipeline.png)
  
&emsp;&emsp;该模型可以分为两部分：1.预训练生成模型数据生成架构； 2.两步渲染架构。<br>
&emsp;&emsp;1.预训练生成：对体素场景进行随机相机采样，获得体素场景的分割图；根据分割图，预训练模型（SPADE）生成伪GT影像。为GT影像输入到SPADE相同风格编码器中获得风格编码，再输入到风格网络中获得风格特征向量。<br>
&emsp;&emsp;2.两步渲染过程：对体素场景中每个体素设置八个可学习特征向量（位于八个顶点位置），进行三线性插值后获得位置编码，
根据位置编码特征、标签特征、风格特征进行体渲染，获得对应的密度与颜色特征向量；利用风格特征向量进行天空渲染得到特征向量。将上述特征向量与风格特征向量结合输入到CNN渲染器，将二维特征转化为RGB图像进行渲染。
</p>

### 训练与推理
<p>&emsp;&emsp;筛选策略：剔除低熵语义标签与低平均深度影像。<br>
&emsp;&emsp;损失：主要分为重建损失与GAN损失。重建损失包含：L1、L2损失与VGG的视知觉损失；

## 实验
### 训练数据集
&emsp;&emsp;训练数据集由1M张在互联网上搜集的图像获得。将其中5000张作为测试集

### Baseline
&emsp;&emsp;MUNIT、SPADE、wc-vid2vid、NSVF-W（NSVF-W为作者改进的方法与本文方法类似，基本可以理解为缺少CNN渲染）<br>
与Baseline比较的采样结果如下图所示：

![Baseline_sample](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/GANcraft4_quality.png)

### 评估准则：
&emsp;&emsp;同样是分为定性与定量评估。定量准则包含FID与KID；定性标准则是让用户根据连续性与质量对生成结果进行评分。
<br>
定性与定量评估参数结果如下表所示：

![FID&KID](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/GANcraft5_metrics_.png)

![user](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/GANcraft6_meterics_users.png)

### 消融实验
&emsp;&emsp;1.不使用伪GT，训练过程中仅仅使用GAN loss，生成结果不切实际。<br>
&emsp;&emsp;2.仅仅使用体渲染不使用CNN渲染结果缺乏精致细节。<br>
&emsp;&emsp;3.不使用GAN 损失生成结果暗淡、模糊。<br>
&emsp;&emsp;4.不使用伪GT影像，虽然质量指标最优，但是生成器会为了指标生成不合理的影像。

![Ablation](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/GANcraft7_Ablation.png)


## 工程实现上的理解
### per-sample MLP and blending
<p>(1)根据参数设置决定对光线方向处理（不变、设置为空、进行位置编码）；<br>
  （2）根据光线传播的体素ID创建最后击中天空与直接命中天空的蒙版。<br>
  （3）根据参数设置采样的光线数量<br>
  （4）获取光线沿途采样的深度值、采样点之间的距离、采样点在深度对应的索引<br>
  （5）将分割图中的标签进行简化（按照论文中，将实际标签映射到标准COCO标签中）<br>
  （6）根据采样点在深度对应的索引，制作蒙版选择索引到分割图中。<br>
  （7）创建独热编码蒙版，将分割图设置为0与1两个部分。<br>
  （8）将体素块特征、光线点世界坐标、光线传输方向、风格编码以及独热蒙版输入到MLP渲染器正向传播函数中，获得密度与颜色特征。<br>
  （9）
