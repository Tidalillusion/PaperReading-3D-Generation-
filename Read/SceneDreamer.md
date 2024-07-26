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

### 模型改进
<p>&emsp;&emsp;(1)BEV representation:由高度场与语义场组成（类似于鸟瞰的场景）。优点在于不需要相机三维位姿、可以提供足够的语义信息与几何信息、以较低开支实现二维数据进行三维重建。<br>
&emsp;&emsp;(2)Neural Hash Grid:通过输入场景特征、语义、几何信息有效对齐语义与几何信息，实现跨场景生成潜特征并且使使模型拥有足够容量与效率生成三维场景。<br>
&emsp;&emsp;(3)style-based 体渲染器:确保生成图像在不同帧之间具备几何一致性(场景三维一致性)，同时通过输入的风格编码确保语义一致性。</p>

##  模型
### BEV Representation
<p>&emsp;&emsp;BEV presentation具备如下优点：
<br>&emsp;&emsp;(1)高效：无边界场景对算力、空间开支要求高，因此需要数据Representation高效.<br>&emsp;&emsp;(2)表示性强：Representation需要方便语义信息与几何信息对齐</p>

<p>&emsp;&emsp;如前文提到，BEV Representation由Height filed 与Semantic field组成。<br>

