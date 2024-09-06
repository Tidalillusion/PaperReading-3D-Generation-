# Sat2Density: Faithful Density Learning from Satellite-Ground Image Pairs
![author](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/d7-1.%E4%BD%9C%E8%80%85%E5%9B%A2%E9%98%9F.png)

![demo](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/d7-2.demo.png)

创新点：<br>
1.针对卫星图像生成3D感知的地面视图，从几何角度学习场景密度场从而实现地面全景图合成。
<br>
2.Sat2Density通过密度表示自动学习准确和真实的3D几何而无需进行深度监督。
<br>
3.研究提供了一种新的视角来理解卫星图像与地面全景图在3D空间的关系。

# 工作的遇到的挑战与改建
## 挑战
<p>&emsp;&emsp;卫星图像与地面图像之间视点差异巨大，视觉特征重叠有限、
外观变化巨大所导致了病态学习问题。具体挑战如下两点：<br>
&emsp;&emsp;1.卫星视图中天空是缺失的不可以使用显示表示；<br>
&emsp;&emsp;2.训练过程中影像间照明差异也使得几何学习具有挑战性。

## 本文改进
&emsp;&emsp;增加了两种监督信号来共同获取体渲染结果：<br>
&emsp;&emsp;1.非天空不透明度监督（使密度场专注于地面场景而非无限区域）<br>
&emsp;&emsp;2.光照注入（从天空将光照进行规范化，从而进一步规范化密度场）

# 网络架构
 &emsp;&emsp;将卫星影像输入到DensityNet编码解码器中，生成密度场的显示体积表达。
 渲染全景图深度并沿着地面视图的光线进行颜色投射，将结果输入到RenderNet中。为了确保照明一致，
 全景图中天空区域的颜色直方图被用作我们方法的条件输入。
 ![pipeline](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/d7-3.Pipeline.png)

##  补充细节
### 密度场表示
 &emsp;&emsp;将显示体积密度编码为场景几何的离散表示，然后使用编码解码器对其参数化
从而学习到密度场的概率分布。对于不存在于网格中的位置，使用三线性插值获得密度。同时对
立方体进行设定：（1）立方体外的体积密度为0；（2）将最低体积密度设置为相对较大值。
### 体渲染
 &emsp;&emsp;光线传播：与NeRF通过光线与粒子作用计算RGB不同，本文是将卫星图上颜色复制到
复制粘贴并进行双线性插值进行渲染。<br>
 &emsp;&emsp;针对体密度场，将深度、不透明度、颜色张量进行拼接作为输入张量输入到Render Net
中，进行地面视图合成。<br>
### 监督
 &emsp;&emsp;非天空不透明度监督：将天空区域视为有意义的类，进行分割，建立关于天空区域与非天空
区域的损失函数。<br>
&emsp;&emsp;从天空注入光照：为了使模型不仅可以学习到光照信息，而且更加专注于地物的几何形体，
将天空区域的RGB直方信息作为光照提示，按照固定维度进行嵌入，从而使得模型可以学习到复杂光照信息。


# 实验
## 数据集与评估指标
&emsp;&emsp;数据集：CVACT（照明控制、视屏合成） CVUSA（仅进行中心地面视图合成）  
<br>&emsp;&emsp;RMSE、SSIM、PSNR、SD进行像素级相似性评估；同时使用两个预训练CNN进行评估。

## 消融实验
&emsp;&emsp;消融实验主要内容：1.天空不透明度 2.光照插入 3.将深度与最初全景图进行拼接
<br>&emsp;&emsp;光照插入很大程度影响评价指标，透明度影响较小，而深度几乎不影响，而将透明度与光照
结合可以很好提升指标。
 ![ablation](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/d7-4.%E6%B6%88%E8%9E%8D%E5%AE%9E%E9%AA%8C.png)

## 实验结果
&emsp;&emsp;中心视图合成：定量比较与定性比较结果均较好。<br>

 ![result](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/d7-5.Baseline%E5%AF%B9%E6%AF%94.png)
 
&emsp;&emsp;可控光照：明显可以看到提供了不同光照，道路颜色发生变化，几何特征不变。<br>

 ![light_demo](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/d7-6%E5%85%89%E7%BA%BF%E6%8E%A7%E5%88%B6%E7%BB%93%E6%9E%9C.png)

&emsp;&emsp;地面视频合成：本文实验结果具备时空一致性。

 ![video](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/d7-7.%E8%A7%86%E9%A2%91%E5%90%88%E6%88%90%E7%BB%93%E6%9E%9C.png)


