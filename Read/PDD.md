# Pyramid Diffusion for Fine 3D Large Scene Generation

![author](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/PDD1author.png)
作者团队来自于西南交通大学。


## 创新点

1.通过金字塔扩散模型实现了大型户外场景由粗到细的生成<br>
2.对本文提出的模型进行了广泛实验，引入了新的指标，从多角度评价，验证生成质量较高。<br>
3.本文模型应用广泛，可以从合成数据集泛化到真实世界数据场景，还可以拓展进行无限场景创建。

![contributions](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/PDD2PDDcontributions.png)

## 本文挑战

1.三维生成方面：目前缺乏大型、充足的三维场景数据。<br>
2.扩散模型应用：扩散模型占用大量内存，训练时间较长；扩散模型训练需要大量数据集。<br>
3.指导生成挑战：目前使用的指导条件是Scene Graph与2D maps，但是这种指导不是一直有用。<br>

# 实验方法

## overview

![pipeline](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/PDD3Pipeline.png)

&emsp;&emsp;通过利用金字塔，先利用三扩散模型在低分辨率尺度上生成场景，然后将生成的场景进行上采样与高分辨率场景拼接后作为输入的原始数据进行扩散过程。这样，在各个金字塔均训练了扩散模型进行数据集概率分布的学习(可以生成各个层级对应的场景)。<br>
&emsp;&emsp;在最高分辨率的层级(本文设置为CarlaSC数据集256×256×16)，将场景划分为4个子场景，生成结束后利用融合算法进行融合。设置参数保证场景重叠度从而确保场景连续性的同时平衡融合算法对结果的影响。

## 离散扩散

1.利用独热的形式（场景大小+标签张量）表示三维场景。通过马尔科夫链与转移矩阵来计算贝叶斯，从而计算得到概率分布。<br>
2.增噪过程：不断利用马尔科夫转移矩阵对原始场景进行增噪，生成均匀离散噪声形式。<br>
3.去噪过程：利用模型预测去噪标签（利用预测的参数获取上一步的概率分布）<br>
4.损失：预测上一步与预测初始状态概率分布的两个KL散度加权和。<br>

## 金字塔扩散

&emsp;&emsp;小尺度场景（可以视作大尺度场景的有损压缩），进行大尺度场景生成（恢复场景细节）。因此执行扩散过程时需要先进行最初扩散模型生成，然后依次进行场景恢复。
PDD优点：不同尺度扩散模型独立，可以并行训练；可以利用中间尺度恢复任意分辨率的场景。

## 场景细分

&emsp;&emsp;在最高分辨率层级，可以将大分辨率场景划分成数个子场景，利用共享的扩散模型进行场景生成，然后利用融合算法进行子场景融合。<br>
&emsp;&emsp;为了保证生成结果具有连续性，添加了其他具有重叠区域的子区域作为条件进行训练（除第一个子场景由噪声初始化外，其他场景均有重叠区域作为条件）

# 实验设置

## 评估工具

1.三维语义分割：利用三维语义分割网络进行mIou以及Mas作为评估指标评估生成质量。<br>
2.参数：（1）F3D：FID的三维适应版本，用预训练的三维CNN计算生成场景与真实场景在特征域上的距离。<br>
&emsp;&emsp;&emsp;&emsp;（2）MMD（最大平均差异）：同样利用预训练的三维CNN通过统计方法度量生成场景与真实场景之间景物分布差异。

## 数据集

&emsp;&emsp;本文使用CarlaSC以及Semantic KITTI数据集。前者用以进行实验模型训练、消融实验以及主要质量对比实验。后者主要用以进行数据迁移实验。

![generation_m](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/PDD6result_generation.png)

## Baseline

&emsp;&emsp;在最大分辨率数据集场景上对原始离散扩散模型与具备VQVAE 的LDM模型训练，结果作为对比基线。

![baseline](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/PDD5quality.png)

## 实验结果

&emsp;&emsp;生成质量对比基线十分优异；同时作者将生成场景与训练场景相似性进行匹配差异对比，通过相似性说明这不是过拟合。<br>
下图展示两个数据集上条件生成以及自由生成的结果。

![CarLASC](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/PDD7CarLASC.png)

![CarLASC_con](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/PDD8CarLASCcon.png)




&emsp;&emsp;下游场景应用：无限场景生成与模型跨数据集迁移能力。

![Semantic_KITTO](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Image/PDD9SemanticKITTO.png)

## 消融实验

1.金字塔扩散：随着增加金字塔层级（即增加更多维度来扩充金字塔），视觉指标提升，但该做法具备明显的边际效应。<br>
2.子场景划分：融合算法限制与重叠区域增加连续性的trade off<br>
3.场景分辨率：随着分辨率增加，生成的细节增加，布局更加合理，场景保真度与复杂性也提高。<br>










