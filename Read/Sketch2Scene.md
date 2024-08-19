# Sketch2Scene: Automatic Generation of Interactive 3D Game Scenes from User’s Casual Sketches

<p>&emsp;&emsp;作者团队来自于腾讯 XR视觉实验室。

<p>&emsp;&emsp;本文主要贡献：<br>
&emsp;&emsp;（1）基于深度学习方法，提出了利用用户简单提示（草图）进行可交互式场景生成。<br>
&emsp;&emsp;（2）为解决三维数据不足的问题，利用扩散模型生成二维影像，再利用等角投影模式进行相机位姿分解并获取场景的分布。<br>
&emsp;&emsp;（3）对等角影像进行分割，分割出影像中具有语义的部分。利用语义信息与场景分布信息使用Unity等工具三维场景生成。<br>
&emsp;&emsp;（4）实验结果表明，该方法可以很好地根据用户意愿生成可交互式、无缝、高质量的三维场景。

<p>&emsp;&emsp;本文局限性：<br>
&emsp;&emsp;（1）多个生成步骤累积误差，使用时需要调试随机种子。<br>
&emsp;&emsp;（2）本文纹理与材质通过检索数据集获取，限制了多样性。<br>
&emsp;&emsp;（3）作者认为可以同时生成多个模块或者同时生成前景与背景面对（1）；利用扩散模型生成来面对问题（2）。

## 预备知识
Isometric projection：一种坐标轴布置形式：XYZ三个轴的单位长度相同，但是互相之间的夹角为120°，从而可以处理简单的遮挡关系。

## 一、Introduction
<p>&emsp;&emsp;目前三维生成领域蓬勃发展在文本、视频、音乐等领域取得了很多成就，列举了相关模型与数据集。<br>
&emsp;&emsp;而相比于上述AIGC，3D场景生成尤其是开放游戏世界场景生成还面临许多问题，这主要是因为除了少数开源的自动驾驶数据集外，缺乏大规模三维数据集。<br>
&emsp;&emsp;本文主要是基于用户草图与可选的文字提示进行符合要求的三维场景生成，并且针对缺乏三维数据集这一现状提出了利用二维扩散模型生成数据集进行训练的策略。<br>
&emsp;&emsp;本文方法流程大致如下：首先根据需要生成的三维场景生成二维说明性影像；再利用visual scene 理解模块将该影像分解为背景与前景分布图两部分；而前景的分布图输入到procedure content pipeline中进行三维场景生成，该场景可以与unity等兼容，生成开放游戏场景。<br>
&emsp;&emsp;为保证草图控制的精确与可控，使用semantic-constraint 扩散损失来约束Control Net；同时采用新的basemap develop来进行场景basemap修复。为了实现上述两种改进，作者利用unique isometric数据集进行上述模型训练。并且使用BEV图像进行精细化贴图。<br>
&emsp;&emsp;总结全文的创新点。

## 二、相关工作
&emsp;&emsp;作者根据自己工作聚焦的突破点介绍了之前基于扩散模型的三维生成相关工作以及procedural 生成。

## 三、实验方法
<p>&emsp;&emsp;本文模型主要分为三部分：基于草图的Isometric的生成器、visual场景理解模块、程序三维场景生成器。<br>
如下图所示主要的流程：首先利用Control Net 处理用户输入的草图进行基于草图指导的isometric generation获得2D isometric 影像；然后将该影像输入到视觉场景理解器中，首先提取前景对象蒙版；然后利用该蒙版生成2D isometric 影像对应的basemap（没有地物对象以及前景的高程场景背景图）；并且场景理解模块计算高程图、表面纹理、地物目标姿态。最后利用 procedural 进行三维游戏场景的生成。</p>

### 1.Sketch Guided Isometric Generation
####  1.1 2D Isometric Image Generation
<p>&emsp;&emsp;该部分旨在利用用户草图生成说明性的二维图像isometric 图像。即利用预训练的扩散模型，通过Isometric Projection model生成3D场景的倾斜视图。<br>
&emsp;&emsp;首先利用control net 处理用户草图生成说明性倾斜二维图像，训练过程中使用N个通道的one-hot 编码，其中每个通道对应一种地物。（比起RGB，one-hot训练简单且允许类别重叠）。<br>
&emsp;&emsp;这一过程中，为了能够使得基于用户草图的生成影像合理（例如在草图空白区域合理位置添加合理的地物）。为了实现该效果，需要利用不同地物组合的草图进行模型训练。具体做法为训练过程中，对草图进行类别过滤，随机丢弃某种地物外的其他地物，从而进行针对某种地物的分布数据增强。<br>
&emsp;&emsp;而上述随机丢弃的数据增强因为草图基于GT而不能直接进行。因此使用SketchAware Loss (SAL)损失函数进行。即根据草图创建掩码蒙版，然后利用该蒙版作为损失的权重矩阵从而鼓励扩散模型关注特定的区域。（该权重矩阵由草图蒙版卷积得来，因此越接近用户草图的丢弃方式权重越大）

####  1.2  2D Empty Terrain Extraction
<p>&emsp;&emsp;该模块旨在提取不含前景的terrain map，需要对之前仍存在部分遮挡的2D Isometric image进行处理。这类似一个复杂的inpainting task但目前SOTA的扩散模型无法很好完成该工作。<br>
&emsp;&emsp;为解决问题，使用自己创建的数据集在SDXL Inpaint上微调LORA。作者创建的数据集由三部分组成：1.具备前景的basemap（即2D影像） 2.terrain map的透视图像 3. terrain 表面纹理图像。其中对数据1，要求前景蒙版中前景不能重叠；对于其他两种任务蒙版则是来自于随机形状相交的其他前景蒙版。<br>
&emsp;&emsp;因此这部分微调损失可以优化为简单向数据集（数据集经过前景图像与等距图像合并）中添加噪声学习背景图像。这一过程中三种数据均经过随机洗牌与抽取。<br>
&emsp;&emsp;在这一过程训练与推理过程导致了概率偏移。原因：1.训练过程屏蔽背景而推理过程则需要屏蔽前景；2.获取的蒙版与GT存在差异。因此在训练后期使用SUD（该方法只在接近正确时有效）。SUD算法如下图所示。

### 2.visual scene understanding
<p>&emsp;&emsp;这里将三维场景分解为三个部分：1.高度图 2.纹理splatmap 3.前景。 分别控制场景地形形状、场景地形纹理以及地形上对象的种类、位置、姿态。

#### 2.1 高度图
<p>&emsp;&emsp;由于之前inpaint basemap后仍然存在部分遮挡关系，因此利用之前创建的terrain map重建三维场景的mesh。与之前依赖增量式场景重建的方法不同，本文提出的方法，利用Isometric 视角最小化遮挡的影响。即利用单张影像获取场景未遮挡部分粗略颜色与场景深度。<br>
&emsp;&emsp;深度获取：使用depth-anything方法来获取深度，然后将RGB影像映射获取点云，最后利用泊松重建技术进行三维重建。重建完成后，由于是Isometric 视角，因此经过简单的旋转便可以得到BEV视角，利用BEV视角沿着重力方向可以得到深度，再利用深度计算高度。<br>
&emsp;&emsp;粗略颜色获取：针对水区域，不仅根据粗略颜色添加水区域，还设置其Height。

#### 2.2 纹理splatmap
<p>&emsp;&emsp;仅利用之前获取的mesh来设置颜色会导致结果质量低，为了与主流引擎兼容，设置N个texture tiles与N个Channels splat。首先利用Segment everything在获取mesh上的RGB影像计算得到texture tiles；然后利用Osprey获取各个分割类别的mask语义种类。从texture tiles列表中获取texture tiles以及其对应的种类，然后将结果分配给对应的mesh。这样使得近距离观察纹理依然清晰。<br>

#### 2.3 前景目标
<p>&emsp;&emsp;使用Sam进行地物分割获取每个前景地物的2D 掩膜从帮助进行三维姿态估计。<br>
&emsp;&emsp;由于物体以Isometric形式排列（有45°夹角），因此设计了下面方法估计其foot print。首先通过homography来warp前景分割图像.利用homography得到的BB以及之前SAM得到的分割目标信息来获取旋转后的footprint。由于Isometric 投影的特性，将footprint变回Isometric Image并且估计了高程信息。利用估计的高程信息，将foot print信息转换为三维位置信息。

### 3.程序三维生成
<p>&emsp;&emsp;通过之前模块生成的语义与几何信息可以利用程序三维生成技术生成三维场景，并且直接与下游引擎兼容。本文实验使用的引擎为unity。<br>
&emsp;&emsp;根据给定的Height map、splatmap、titles可以方便使用引擎进行三维生成，并且根据给定tiles的类别，可以贴上高分辨率的三维纹理。<br>
&emsp;&emsp;对于较大尺度的前景地物，使用检索或生成的方式进行场景重建。（1）检索:通过对比clip分数检索到与目标地物最为接近的Objaverse dataset中的实例；（2）生成：使用最新的2D-2-3D生成技术生成三维实例，再根据姿态将三维实例构建场景。

### 4.实验结果
#### 1.训练与推理细节
<p>&emsp;&emsp;优化器、lr、训练各部分模块以及推理的GPU、step、time。数据集生成。

#### 2.草图生成Isometric 影像结果
<p>&emsp;&emsp;如图所示为control net生成的结果。可以发现模型很好的满足多样性生成的要求（a、b、c三种风格），并且内绘可以得到单纯的basemap。同时本文利用草图随意性、随机添加额外对象以及拓展补丁区域来trade off 草图与文字之间的关系。

#### 3.内绘结果比较
<p>&emsp;&emsp;可以发现与Baseline（保留了人造地物）相比，本文内绘方法可以获得更加纯净的地形信息。

#### 4.视觉场景理解模块
<p>&emsp;&emsp;展示了生成的高度图、BEV三维实例放置示意图以及参考三维实例图。

#### 5.程序化三维生成结果
<p>&emsp;&emsp;下图a为使用数据库检索的场景重建结果，b、c为生成结果。