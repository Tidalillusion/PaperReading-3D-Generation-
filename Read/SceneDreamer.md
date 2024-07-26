# SceneDreamer


<p>&emsp;&emsp;作者团队主要来自于南洋理工大学S-LAB（后续还会follow该团队基于NeRF与3DGS的城市生成工作）</P>
<p>&emsp;&emsp;这篇工作主要是通过输入随机初始化噪声，通过风格编码与设计的相机轨迹，实现不同风格与相机轨迹的无边界场景生成。</P>
<p>&emsp;&emsp;这篇工作的主要创新点在于:<br>
&emsp;&emsp;(1) 2023年来提出的首个无边界生成模型，生成结果较好地保持了风格一致性与内容连续性。<br>
&emsp;&emsp;(2) 模型训练的数据为无标注的、分辨率、场景不统一的自然影像数据。<br>
&emsp;&emsp;(3) 提出了BEV presentation来表示数据，并提出了Neural Hash Grid结构来处理数据，提升了模型容量、生成效率、效果。</p>

<p>&emsp;&emsp;这篇工作的可以改进地方：<br>
&emsp;&emsp;(1)由于BEV Representation限制（可以理解为鸟瞰图），因此只能针对凹面进行处理而不能针对凸面进行处理。<br>
&emsp;&emsp;(2)由于使用GAN作为生成器因此随机初始化可能会造成崩塌导致训练难度较大。<br>
&emsp;&emsp;(3)该方法计算开支较大</p>

##  无边界三维生成难点与改进
### 工作难点
<p>&emsp;&emsp;(1)缺乏有效的三维场景表示方法。之前表示方法如Triplane更多面向整个场景进行建模，而无法有效进行无边界的生成。<br>
&emsp;&emsp;(2)缺乏有效的内容对齐手段（之前训练数据多为规范的数据，无法进行尺度巨大、场景丰富的结果进行生成）<br>
&emsp;&emsp;(3)重建时要面对缺乏相机位姿的情况(户外影像无规则无法准确推算位姿)</p>

###模型改进
<p>&emsp;&emsp;(1)BEV representation:由高度场与语义场组成（类似于鸟瞰的场景）。优点在于不需要相机三维位姿、可以提供足够的语义信息与几何信息、以较低开支实现二维数据进行三维重建。<br>
&emsp;&emsp;(2)Neural Hash Grid:通过输入场景特征、语义、几何信息有效对齐语义与几何信息，实现跨场景生成潜特征并且使使模型拥有足够容量与效率生成三维场景。<br>
&emsp;&emsp;(3)style-based 体渲染器:确保生成图像在不同帧之间具备几何一致性(场景三维一致性)，同时通过输入的风格编码确保语义一致性。</p>

##  模型
### BEV Representation
<p>&emsp;&emsp;BEV presentation具备如下优点：
<br>&emsp;&emsp;(1)高效：无边界场景对算力、空间开支要求高，因此需要数据Representation高效.<br>&emsp;&emsp;(2)表示性强：Representation需要方便语义信息与几何信息对齐</p>

<p>&emsp;&emsp;如前文提到，BEV Representation由Height filed 与Semantic field组成。<br>&emsp;&emsp;Height filed 反映了三维目标表面的高程信息，通过设置特定分辨率的窗口可以将Height filed 转换为Height map。<br>&emsp;&emsp;Semantic filed利用表以One-hot的形式存储表面像素对应的语义信息；同样Semantic field也可以利用特定分辨率窗口转换为对应的Semantic Map。</p>
<p>&emsp;&emsp;BEV Representation 采样过程：<br>
&emsp;&emsp;本文采用无参数的前馈生成模式，因此生成阶段噪声是基于梯度噪声网络的n维噪声函数，是Perlin噪声的改进版.该噪声由level of octave控制，high level则噪声复杂，多样性较好；low level噪声平滑但是多样性低.<br>
&emsp;&emsp;Height Field生成：利用两个不同level of octave生成两个噪声，再对两个噪声进行插值从而进行初始化噪声<br>
&emsp;&emsp;Semantic Field生成：利用不同随机种子初始化两个相同level of octave的噪声，分别代表气温与降水，将各个像素生成结果与预先设置九种地形对应，存储在表中。<br>
&emsp;&emsp;标签正则化:预先设置了十二种地物标签，并且将定义的九种地形与这十二种地物标签建立映射关系。利用Voronoi diagram对语义图中的标签进行正则化。</p>

### Neural Hash Grid
<p>&emsp;&emsp;目的：实现跨场景的三维表征学习，将语义信息与几何信息进行对齐，将特征信息映射进入隐空间便于生成对抗训练。<br>
&emsp;&emsp;由编码器（将语义图、高度图信息映射到隐空间中）、具备编码参数的Neural Hash Function（将坐标信息与编码的场景信息映射到超空间）组成。<br>
&emsp;&emsp;将语义图与高度图的信息进行编码获得场景特征；将位置坐标、语义信息、场景特征输入到Neural Hash Grid中获取超空间特征。

### Unbounded 3D Scene GAN
<p>&emsp;&emsp;目的：从没有位姿信息的户外影像数据生成三维场景。<br>
&emsp;&emsp;这里的生成器被称为体渲染生成器，包括了NeRF、Conditional Neural Field、GAN<br>
&emsp;&emsp;首先NeRF进行光线追踪、然后神经场获得特征张量、最后利用GAN进行生成.

## 训练与推理
### 训练
<p>&emsp;&emsp;影像筛选策略：剔除低熵语义标签与低平均深度的影像。<br>
&emsp;&emsp;损失：分为重建损失与生成对抗损失两部分；包含：MSE、GAN、Perceptual

### 采样
<p>&emsp;&emsp;无边界场景生成：首先生成全局的BEV，再利用滑动窗口进行局部场景的生成。<br>
&emsp;&emsp;高分场景渲染：由于BEV的紧凑表达，因此可以生成比训练数据分辨率更高的场景影像。
