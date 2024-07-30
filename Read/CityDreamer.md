# City Deamer

<p>&emsp;&emsp;本篇文章作者团队来自于南洋理工大学 S-Lab，其中陈昭熹是之前阅读的Scene Dreamer的一作，两篇文章具有很强的相关性。</p>
<p>&emsp;&emsp;本文创新点如下：<br>
(1)针对复杂的城市场景，提出了City Dreamer的无边界生成模型，其场景生成与场景编辑功能均达到了SOTA。<br>
(2)作者将城市场景分为两部分：背景与建筑物，分别进行生成体积渲染。使用BEV Representation进行场景表示(Scene Dreamer)，利用Neural Hash Grid处理建筑物(Scene Dreamer)<br>
(3)将房屋分为立面与房顶两部分，通过添加风格编码实现建筑物生成的细分类生成。<br>
(4)作者提供了实验使用的OSM与Google Earth数据集。</p>

## 实验工作面对难点与Pipeline
### 工作难点
<p>
(1)生成对抗模型与常用的三维卷积生成无边界场景会占用大量内存，消耗大量算力。<br>
(2)场景级生成连续性往往聚焦与有限的室内场景，空间受限，无法完成大规模城市场景的相关任务。</p>

### 本文工作的Pipeline
<p>&emsp;&emsp;CityDreamer分4步：1.无边界Layout生成器创建任意城市layout L。2.城市背景生成器生成背景图片附带相关的掩膜。3.建筑实体生成器，生成建筑实体与各自对应的掩膜。4.组成器：将渲染背景与建筑实例合并到单个内聚连续的图像中。如下图所示为文章Pipeline。

### 结果


## 模型
### City Layout
<p>&emsp;&emsp;这部分作者使用Segment Anything进行OSM数据分割，利用获取的语义图进行生成模型训练，使得模型可以在后续任务中进行无边界场景生成（利用随机初始化的噪声获取数据集随机分布）。<br>
&emsp;&emsp;作者使用BEV Representation来表示城市区域的数据。该表示形式的优越性在Scene Dreamer中已经提及。这里对BEV表示进行了一定的改进。将城市空间地物分为两大类：背景与建筑物。其中背景包含：道路、绿地、建筑工地、水域、其它。此外，稀疏区域用空来表示。<br>
&emsp;&emsp;这里作者使用VQGAN来进行BEV Representation的生成（Height Map以及Semantic Map）同时作者利用插值来提升VQVAE模型生成的多样性。<br>
&emsp;&emsp;生成过程中，利用重叠度为0.25滑动窗口进行局部生成并保证生成无边界区域的一致性。<br>
&emsp;&emsp;该训练过程的损失包含了Height MAP与Semantic MAP损失，此外作者还引入了平滑损失以强化建筑边界的锐度。</P>

### Background Generator
<p>&emsp;&emsp;这部分模型作者使用之前生成BEV Representation与真实图像作为输入训练模型，使得模型可以生成真实背景图像。<br>
&emsp;&emsp;首先是利用局部滑动窗口获取局部的BEV Representation。然后将局部的BEV信息输入到Global Encoder中，将信息映射到隐空间中（得到紧凑场景特征）。然后将隐空间的特征信息输入到Generative Hash Grid中得到Indexed Feature（紧凑场景索引）。利用该特征进行体渲染与生成对抗。<br>
&emsp;&emsp;该部分损失可以分为重建损失与生成对抗损失。即L1 Loss， perceptual Loss 以及 GAN Loss。注意：这部分只可以用于背景区域的生成。</p>

### Building Instance Generator
<p>&emsp;&emsp;这部分模型作者也是利用之前生成的局部BEV信息进行建筑场景的生成。不过这里为了效率与生成质量，将每栋建筑作为单独的Instance进行处理。<br>
&emsp;&emsp;首先根据滑动窗口获取局部的BEV，由于建筑物在语义图上标签相同，因此利用连通性检测组件来划分不同Instance。由于建筑立面与屋顶差异巨大，因此设置两个标签进行表示，同时加入风格编码来反映不同的建筑风格。<br>
&emsp;&emsp;这里主要对局部BEV特征进行如下处理：将初始化完成的BEV数据输入到Local Encoder中进行局部编码；然后再利用NERF的位置编码函数对编码结果进行处理；最后进行体渲染。其中这里体渲染引入了风格编码来使得生成的建筑实例风格多样。<br>
&emsp;&emsp;这里模型损失主要是GAN Loss，即语义图与生成结果之间的GAN Loss.</p>

### Compositor
<p>&emsp;&emsp;由于没有真实场景影像来与合成场景求损失。因此，合成器利用生成的背景与建筑影像结合它们相关的二值掩码进行统一图像合成。

## 实验设计

### 数据集
<p>&emsp;&emsp;使用OSM数据集与Google Earth数据集进行实验。<br>
&emsp;&emsp;OSM数据集来自于OpenStreetMap，栅格化语义地图。栅格化过程中将经纬度转到EPSG：3857坐标系统中。<br>
 &emsp;&emsp;GGE Dataset是在曼哈顿与布鲁克林的四百条轨迹搜集的图像，相机内外参数以及创建用于语义与建筑实体分割的自动注释。首先，为了对建筑物进行实例分割，在语义地图上进行连通性分析；然后是进行city Layout处理，最后利用摄像机参数将城市Layout投影到图像上。与其他数据集相比，GGE具备不同高度与视角信息。</p>

### 评价指标
<p> &emsp;&emsp;（1）FID与KID：在抽取的1500个帧中计算FID与KID。<br>
 &emsp;&emsp;（2）深度误差：使用depth error（DE）来衡量3D几何图形，使用预训练模型，使用累积密度来衡量伪地图深度。实际深度与预测伪深度均归一化为零均值与单位方差以消除尺度模糊性。DE计算为两个归一化深度之间的L2距离。作者对每种评估方法100帧进行深度误差估计。<br>
&emsp;&emsp;（3）相机误差：Camera Error（CE，相机误差）评估多视角连续性。CE量化了推理相机轨迹与COLMAP估计的额相机轨迹之间差异。其计算方式是重建的相机姿势与生成相机姿势之间尺度不变的L2距离。</p>

### 比较实验
 &emsp;&emsp;作者与其他工作方法进行了定性比较、定量比较以及用户体验比较。其中用户体验比较包含：图像感知质量、三维真实度、三维视觉连续度三个方面。

### 消融实验
 &emsp;&emsp;作者消融实验包含以下内容：
 &emsp;&emsp;(1)无边界布局生成器：将这部分从VQVAE换成Infini City与IPSM中的生成器。
 &emsp;&emsp;(2)建筑实例生成器：1.直接移除建筑实例生成器;2.建筑实体生成器中不加入实体标签。
 &emsp;&emsp;(3)场景参数化：背景生成参数化方法为利用Neural Hash Grid，而建筑实例生成则是利用NeRF编码。交换了两种方法后发现效果不好。




