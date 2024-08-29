
# PaperReading-3D-Generation
<p>This repository is built to record the paper I have read for me to revise.<br>  Hope that by writing and sharing, I coulde make progress.</p>
<P> 
The Table 1 is the paper that I have intensive reading, so there will be detailed information about this paper. If I have read the code, I will show my understanding about it. If it is the early work that I think it is important, I will not show the result of the expriment the author showing in the paper. I will oly show the recent work's result.<br>
The Table 2 is the paper that I have skimmed, so there will just abound with something interesting for me.<br> 
I have divide this into 4 parts: <br>(1) Use the 3DGS <br>(2) Use the NeFR <br>(3) Use the Diffusion model <br>(4) Other related papers(such as dataset, 2D generation, other 3D generation).

# Table 1(Intensive Reading) 
| Title | Author | Source | Overview |
| ---   | ---     | --- | ---|
|GANcraft: Unsupervised 3D Neural Rendering of Minecraft Worlds<br>[DooDoo_xy's Note](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Dreamer0%EF%BC%88Base%EF%BC%89-GANcraft.md)<br>[aarxiv](https://arxiv.org/abs/2104.07659)<br>[Github](hhttps://github.com/NVlabs/imaginaire)|Zekun Hao,Arun Mallya,Serge Belongie, Ming-Yu Liu<br>NVIDIA&Cornell|ICCV 2021|1.提出了非监督的神经渲染框架，该框架可以用真实三维图像渲染block，从而将block场景（世界）渲染为真实场景（世界）。<br>2.提出了一种基于为地面图像进行渲染的对抗训练方式实现了三维渲染.<br>3.通过伪GT影像训练模型从而解决没有真实数据的问题。<br>4.与基线对比实验表明，GANcraft在三维block世界合成的效果很好。|
|SceneDreamer: Unbounded 3D Scene Generation from 2D Image Collections<br> [DooDoo_xy's Note](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Dreamer1%3ASceneDreamer.md)<br>[arxiv](https://arxiv.org/abs/2302.01330#)<br>[Github](https://github.com/FrozenBurning/SceneDreamer)|Zhaoxi Chen, Guangcong Wang, and Ziwei Liu<br>S-LAB 南洋理工大学|TPAMI 2023|1.2023年来提出的首个无边界生成模型，生成结果较好地保持了风格一致性与内容连续性。<br>2.模型训练的数据为无标注的、分辨率、场景不统一的自然影像数据。<br>3.提出了BEV presentation来表示数据，并提出了Neural Hash Grid结构来处理数据，提升了模型容量、生成效率、效果。|
|CityDreamer: Compositional Generative Model of Unbounded 3D Cities<br>[DooDoo_xy's Note](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Dreamer2%3ACityDreamer.md)<br>[arxiv](https://arxiv.org/abs/2309.00610)<br>[Github](https://github.com/hzxie/CityDreamer)|Haozhe Xie, Zhaoxi Chen, Fangzhou Hong, Ziwei Liu <br> S-LAB 南洋理工大学| CVPR 2024 | (1)针对复杂的城市场景，提出了City Dreamer的无边界生成模型，其场景生成与场景编辑功能均达到了SOTA。<br>(2)作者将城市场景分为两部分：背景与建筑物，分别进行生成体积渲染。使用BEV Representation进行场景表示(Scene Dreamer)，利用Neural Hash Grid处理建筑物(Scene Dreamer)<br>(3)将房屋分为立面与房顶两部分，通过添加风格编码实现建筑物生成的细分类生成。<br>(4)作者提供了实验使用的OSM与Google Earth数据集。 |
|GaussianCity: Generative Gaussian Splatting for Unbounded 3D City Generation<br>[DooDoo_xy's Note](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Dreamer3%3AGaussianCity.md)<br>[arxiv](https://arxiv.org/abs/2406.06526)<br>[Github](https://github.com/hzxie/GaussianCity)|Haozhe Xie Zhaoxi Chen Fangzhou Hong Ziwei Liu<br>S-LAB 南阳理工大学|arxiv|1.首个将3DGS应用于无边界场景生成的工作，并在Google Earth以及KITTI-360数据集达到了SOTA<br>2.提出了compact BEV-Point Representation，降低了训练时的显存与内存的开支，减少了参与优化的点数目。<br>3.提出了Spatial-aware Gaussian Attribute Decoder通过局部生成实现了3DGS RGB属性的高效、高质量生成。|
|SemCity: Semantic Scene Generation with Triplane Diffusion<br>[DooDoo_xy's Note](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Semcity.md)<br>[paper](https://arxiv.org/abs/2403.07773)<br>[GitHub](https://github.com/zoomin-lee/SemCity)|Jumin Lee & Sebin Lee & Changho Jo.et <br> 韩国国家科学院KAIST|CVPR 2024|(1)使用Triplane Diffusion model进行户外场景生成，有效应对真实户外场景生成存在的非结构性与稀疏性问题。<br>(2)无缝的Triplane Diffusion model 很好地进行下游的场景 inpainting、outpainting以及场景补全改善。<br>(3)利用Semantic KIITTI数据集实验，结果表明在场景生成与下游任务均达到了SOTA，并且模型可以很好地扩展到城市尺度。|
|Sketch2Scene: Automatic Generation of Interactive 3D Game Scenes from User’s Casual Sketches<br>[DooDoo_xy's Note](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/Sketch2Scene.md)<br>[project page](https://xrvisionlabs.github.io/Sketch2Scene/)|Yongzhi Xu & Yonhon Ng & Yifu Wang.et<br>腾讯XR视觉实验室|arxiv|（1）基于深度学习方法，提出了利用用户简单提示（草图）进行可交互式场景生成。<br>（2）为解决三维数据不足的问题，利用扩散模型生成二维影像，再利用等角投影模式进行相机位姿分解并获取场景的分布。<br>（3）对等角影像进行分割，分割出影像中具有语义的部分。利用语义信息与场景分布信息使用Unity等工具三维场景生成。<br>（4）实验结果表明，该方法可以很好地根据用户意愿生成可交互式、无缝、高质量的三维场景。|




# table 2.1(skimming-3DGS)

# table 2.2(skimming-NeRF)

# table 2.3(skimming-Diffusion)

# tanle 2.4(skimming-others)
