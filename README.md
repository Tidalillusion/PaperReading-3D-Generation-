
# PaperReading-3D-Generation
<p>This repository is built to record the paper I have read for me to revise.<br>  Hope that by writing and sharing, I coulde make progress.</p>
<P> 
The Table 1 is the paper that I have intensive reading, so there will be detailed information about this paper. If I have read the code, I will show my understanding about it. If it is the early work that I think it is important, I will not show the result of the expriment the author showing in the paper. I will oly show the recent work's result.<br>
The Table 2 is the paper that I have skimmed, so there will just abound with something interesting for me.<br> 
I have divide this into 4 parts: <br>(1) Use the 3DGS <br>(2) Use the NeFR <br>(3) Use the Diffusion model <br>(4) Other related papers(such as dataset, 2D generation, other 3D generation).

# Table 1(Intensive Reading) 
| Title | Author | Source | Overview |
| ---   | ---     | --- | ---|
|SceneDreamer: Unbounded 3D Scene Generation from 2D Image Collections<br> [DooDoo_xy's Note](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/SceneDreamer.md)<br>[arxiv](https://arxiv.org/abs/2302.01330#)<br>[Github](https://github.com/FrozenBurning/SceneDreamer)|Zhaoxi Chen, Guangcong Wang, and Ziwei Liu<br>S-LAB 南洋理工大学|TPAMI 2023|1.2023年来提出的首个无边界生成模型，生成结果较好地保持了风格一致性与内容连续性。<br>2.模型训练的数据为无标注的、分辨率、场景不统一的自然影像数据。<br>3.提出了BEV presentation来表示数据，并提出了Neural Hash Grid结构来处理数据，提升了模型容量、生成效率、效果。|
|CityDreamer: Compositional Generative Model of Unbounded 3D Cities<br>[DooDoo_xy's Note](https://github.com/Tidalillusion/PaperReading-3D-Generation-/blob/main/Read/CityDreamer.md)<br>[arxiv](https://arxiv.org/abs/2309.00610)<br>[Github](https://github.com/hzxie/CityDreamer)|Haozhe Xie, Zhaoxi Chen, Fangzhou Hong, Ziwei Liu <br> S-LAB 南洋理工大学| CVPR 2024 | (1)针对复杂的城市场景，提出了City Dreamer的无边界生成模型，其场景生成与场景编辑功能均达到了SOTA。<br>(2)作者将城市场景分为两部分：背景与建筑物，分别进行生成体积渲染。使用BEV Representation进行场景表示(Scene Dreamer)，利用Neural Hash Grid处理建筑物(Scene Dreamer)<br>(3)将房屋分为立面与房顶两部分，通过添加风格编码实现建筑物生成的细分类生成。<br>(4)作者提供了实验使用的OSM与Google Earth数据集。 




# table 2.1(skimming-3DGS)

# table 2.2(skimming-NeRF)

# table 2.3(skimming-Diffusion)

# tanle 2.4(skimming-others)
