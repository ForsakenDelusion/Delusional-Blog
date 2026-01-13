---
citekey: ""
parent: Traffic Anomaly Prediction Based on Joint Static-Dynamic Spatio-Temporal Evolutionary Learning
title_ZH: ""
aliases: "0"
authors: Xiaoming Liu;Zhanwei Zhang;Lingjuan Lyu;Zhaohan Zhang;Shuai Xiao;Chao Shen;Philip S. Yu;
journal: IEEE Transactions on Knowledge and Data Engineering
journal_IF: []
year: "2023"
doi: https://doi.org/10.1109/TKDE.2022.3150272
url: https://ieeexplore.ieee.org/document/9711930/
zotero_link: zotero://open-pdf/0_IHX7ICYP
zotero_folder:
  - proj1
abstract: 准确的交通异常预测为在正确地点及时救助伤员提供了可能。然而，交通异常的形成过程复杂，既受多种静态因素影响，也涉及动态交互作用。近年来发展的表征学习为理解这一复杂过程提供了新思路，但仍面临数据分布不平衡与特征异质性等挑战。为此，本文提出一种名为SNIPER的时空演化模型，通过学习复杂的特征交互来预测交通异常。具体而言，我们设计了时空编码器，将时空信息映射到表征其内在关联的向量空间；提出时序动态演化嵌入方法以加强对罕见交通异常的捕捉，并构建基于注意力的多图卷积网络，从三个不同视角建模空间相互影响。采用FC-LSTM聚合考虑时空影响的异构特征。最后，设计了可缓解“过度平滑”现象并解决数据不平衡问题的损失函数。大量实验表明，在芝加哥数据集上，SNIPER在AUC-PR、AUC-ROC、F1值和准确率四项指标分别平均优于现有最佳方法3.9%、0.9%、1.9%和1.6%；在纽约市数据集上分别平均提升2.4%、0.6%、2.6%和1.3%。
tags:
  - literature-note
  - DL
summary: "**待补充**"
$version: 701
$libraryID: 1
$itemKey: NCZ3NKGE
title: 交通预测公式笔记
date: 2026-01-12 00:02:03 +08:00
filename: 2026-01-12-Traffic-Proj
categories:
  - Reasearch
dir: Reasearch/DL
share: true
archive: false
priority: 100
math: true
---

# 全部公式及其流程

![\<img alt="" data-attachment-key="PT4WDLBT" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%225W4LAFS4%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225359%22%2C%22position%22%3A%7B%22pageIndex%22%3A3%2C%22rects%22%3A%5B%5B64.913%2C384.305%2C213.413%2C406.805%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225359%22%7D%7D" width="248" height="38" src="attachments/PT4WDLBT.png" ztype="zimage"> | 248](../../../assets/images/PT4WDLBT.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225359%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5359</a></span>)</span>

### Part1 时空编码

![\<img alt="" data-attachment-key="5YMZSNHK" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22UWICGC9R%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225359%22%2C%22position%22%3A%7B%22pageIndex%22%3A3%2C%22rects%22%3A%5B%5B284.663%2C82.05500228881837%2C548.663%2C259.805%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225359%22%7D%7D" width="440" height="296" src="attachments/5YMZSNHK.png" ztype="zimage"> | 440](../../../assets/images/5YMZSNHK.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225359%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5359</a></span>)</span>

![\<img alt="" data-attachment-key="7P2BVKXD" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22H7RJB846%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225360%22%2C%22position%22%3A%7B%22pageIndex%22%3A4%2C%22rects%22%3A%5B%5B17.663%2C499.88%2C280.913%2C638.63%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225360%22%7D%7D" width="439" height="231" src="attachments/7P2BVKXD.png" ztype="zimage"> | 439](../../../assets/images/7P2BVKXD.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225360%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5360</a></span>)</span>

ZE(t) 是一个由不同频率的正弦 ($\sin$) 和余弦 ($\cos$) 函数组成的向量。这里的下标 $1, \dots, k, \dots, D$ 代表不同的**频段，注意！这里的D是人为设定的纬度，可能和数据集中的D不一样**

点积（Dot Product）对应位置相乘，然后把结果加起来”

所以上面的式子的结果为$\cos(t+\delta)\cos(t) + \sin(t+\delta)\sin(t)$

***

![\<img alt="" data-attachment-key="3DSYEHFG" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22WJ86A4KX%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225360%22%2C%22position%22%3A%7B%22pageIndex%22%3A4%2C%22rects%22%3A%5B%5B25.163%2C293.63%2C280.913%2C325.88%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225360%22%7D%7D" width="426" height="54" src="attachments/3DSYEHFG.png" ztype="zimage"> | 426](../../../assets/images/3DSYEHFG.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225360%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5360</a></span>)</span>

差异特征，表示当前状态与“常态”$O_t^l$的偏离程度，后面减去的nor的含义是，取最近的五个正常时间，nor表示正常标签

![\<img alt="" data-attachment-key="GKDPRHKE" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22CIVSKMJA%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225360%22%2C%22position%22%3A%7B%22pageIndex%22%3A4%2C%22rects%22%3A%5B%5B19.163%2C148.13%2C283.163%2C217.13%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225360%22%7D%7D" width="440" height="115" src="attachments/GKDPRHKE.png" ztype="zimage"> | 440](../../../assets/images/GKDPRHKE.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225360%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5360</a></span>)</span>

在t时间，第l个网格的静态特征构建表达式，注意这里可能要经过融合操作，因为$O$

和$D$都是纬度为D的向量，拼在一起变成2D

## Part2 动态演化过程

最开始所有的网格动态嵌入是要初始化为0的,$$X_{t=0}^d \in \mathbb{R}^{N \times 2D}$$

update-decay机制，如果该时刻无异常，就decay更新，有异常就update更新。

![\<img alt="" data-attachment-key="9XL4V4P3" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%229RBD7SZW%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225360%22%2C%22position%22%3A%7B%22pageIndex%22%3A4%2C%22rects%22%3A%5B%5B292.913%2C394.88%2C547.163%2C427.88%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225360%22%7D%7D" width="424" height="55" src="attachments/9XL4V4P3.png" ztype="zimage"> | 424](../../../assets/images/9XL4V4P3.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225360%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5360</a></span>)</span>

$$X_{t=0}^d \in \mathbb{R}^{N \times 2D}$$，上述的$W_1 \in \mathbb{R}^{4D \times 2D}$

![\<img alt="" data-attachment-key="TVS8TBUT" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22E7E2PPJD%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225360%22%2C%22position%22%3A%7B%22pageIndex%22%3A4%2C%22rects%22%3A%5B%5B290.663%2C213.38%2C543.413%2C250.88%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225360%22%7D%7D" width="421" height="62" src="attachments/TVS8TBUT.png" ztype="zimage"> | 421](../../../assets/images/TVS8TBUT.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225360%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5360</a></span>)</span>

![\<img alt="" data-attachment-key="4V335Z5D" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22LAY6AEI5%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225360%22%2C%22position%22%3A%7B%22pageIndex%22%3A4%2C%22rects%22%3A%5B%5B286.163%2C35.63%2C547.913%2C84.38%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225360%22%7D%7D" width="436" height="81" src="attachments/4V335Z5D.png" ztype="zimage"> | 436](../../../assets/images/4V335Z5D.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225360%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5360</a></span>)</span>

*   Update 操作（触发条件：网格  $l$  在  $t$  时刻有碰撞异常，标签  $y=1$  ） ，这里略微和论文有点不同，论文上的方法是利用一个大矩阵，可以减少一些时间，但是本质是一样的，如下

$$
\underbrace{[X^d, X^s]}_{1 \times 4D} \times \underbrace{\begin{bmatrix} W_{up} \\ W_{down} \end{bmatrix}}_{4D \times 2D}= X^d \times W_{up} + X^s \times W_{down}
$$

计算逻辑：$$X_{(t,l)}^d = \sigma(X_{(t,l)}^s \cdot W_1 + X_{(t-1,l)}^d \cdot W_2)$$

其中：$\sigma$ 为 sigmoid 激活函数（控制输出在 $[0,1]$），$W_1 \in \mathbb{R}^{2D \times 2D}$、$W_2 \in \mathbb{R}^{2D \times 2D}$ *为可训练权重矩阵；*

> 如果出事了，这个动态特征就会记住这个“创伤” 可训练矩阵的意思是，**这个参数是会变化的，通过反向传播来改变**

*   Decay 操作（触发条件：网格  $l$  在  $t$  时刻无异常，标签  $y=0$  ）

计算逻辑：$$X_{(t,l)}^d = X_{(t-1,l)}^d \cdot e^{-\phi \cdot (t-t_0)}$$

这里的公式和论文上略有一些不同，但是论文上的是此刻的值由$t_\theta$时刻的值来更新。但上述公式是此刻的值由上一时刻更新。即，论文中给出的是通项公式，但是代码里面是递推式。这里应该是数学公式和代码推导式子的差异。

***

## Part3 多空间图构建过程

物理上挨得近并不代表关系密切，我们要找“功能”上相似的“邻居”，为了让模型学到更好的空间关系，构建了**三张不同的图（Graph）**：

将地点分为如下三种

*   **$\mathcal{G}_F$(Functionality - 功能图)：** 基于 POI（兴趣点）。比如，“两个地方周围都有很多餐馆和电影院”，那它们在功能上是相似的（都是商业区），即使它们物理距离很远。
*   **$\mathcal{G}_A$(Collision - 事故图)：** 基于历史事故记录。比如，“这两个路口都经常发生追尾”，那它们也是相似的。
*   **$\mathcal{G}_T$(Traffic - 交通设施图)：** 基于道路设施（公交站台或者道路类型）。

### 建图过程

1.  **向量化：** 把每个地点的 POI 分布变成一个向量。
2.  **归一化：** 使用 Max-Min Normalization 把数值缩放到一定范围。
3.  **算距离：** 计算两个地点向量之间的**欧几里得距离**（Euclidean distance）。
4.  **选邻居 (Top-k)：** 对于每一个地点，只选跟它**最相似的前$k$个地点**作为邻居。

### 以下是邻接矩阵构建式

![\<img alt="" data-attachment-key="GHY9V2QW" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22UXBEA6HV%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225361%22%2C%22position%22%3A%7B%22pageIndex%22%3A5%2C%22rects%22%3A%5B%5B25.163%2C196.955%2C282.413%2C238.205%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225361%22%7D%7D" width="429" height="69" src="attachments/GHY9V2QW.png" ztype="zimage"> | 429](../../../assets/images/GHY9V2QW.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225361%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5361</a></span>)</span>

1.  如果地点  $j$  是地点  $i$  的“前  $k$  个最相似的朋友”之一（或者  $j$  就是  $i$  自己），那么它们之间连一条线（值为 1）。

2.  否则，没有连接（值为 0）。

## Part4 注意力机制多图卷积（Multi-GCN with Attention）

![\<img alt="" data-attachment-key="XNZPKDRX" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22MPDGPDIC%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225361%22%2C%22position%22%3A%7B%22pageIndex%22%3A5%2C%22rects%22%3A%5B%5B283.913%2C509.55039149180897%2C541.913%2C730.955%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225361%22%7D%7D" width="430" height="369" src="attachments/XNZPKDRX.png" ztype="zimage"> | 430](../../../assets/images/XNZPKDRX.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225361%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5361</a></span>)</span>

![\<img alt="" data-attachment-key="MTJ3XWGH" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22LDZQEGK6%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225361%22%2C%22position%22%3A%7B%22pageIndex%22%3A5%2C%22rects%22%3A%5B%5B284.5%2C442.997%2C542.5%2C501.497%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225361%22%7D%7D" width="430" height="98" src="attachments/MTJ3XWGH.png" ztype="zimage"> | 430](../../../assets/images/MTJ3XWGH.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225361%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5361</a></span>)</span>

先形成Xs和Xd嵌入，对于每一个X，$\text{Shape} = (T \times N \times 2D)$，T为时间步长N为网格数(T=5 为历史时间切片数)，2D是之前设置的特征纬度

### 通过注意力机制生成权重矩阵（谁和自己像）

![\<img alt="" data-attachment-key="UZFURBP7" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22D3NMSTAW%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225361%22%2C%22position%22%3A%7B%22pageIndex%22%3A5%2C%22rects%22%3A%5B%5B289.163%2C221.99665649414067%2C542.663%2C295.205%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225361%22%7D%7D" width="423" height="122" src="attachments/UZFURBP7.png" ztype="zimage"> | 423](../../../assets/images/UZFURBP7.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225361%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5361</a></span>)</span>

*   $$Q_d = X_d W_{dq}$$  代表“中心节点”。它拿着自己的特征去询问周围的邻居：“你们谁跟现在的我最像/最相关？

*   $$K_d = X_d^{(k+1)} W_{dk}$$  代表“环境上下文”。它们展示自己的特征，供中心节点  $Q$  来计算匹配度（点积）

*   $V_d = K_d$  (值与键来源一致，简化计算)

> Q1:为什么K和V设计能一致（邻居动态键和动态值设置的相等，但是不怎么影响效果？）两个都可以直接拿数值表示。
>
> 注意，V和K通常是不一样的，K表示表情，V表示内容，但有可能K和V表达的意思比较相近，比如一本书的封面可以判断一本书的大概内容，但是有可能书就只有一页封面，那么内容不就和封面一样了，也就是可以看作V=K

$$
A_d = \text{Softmax}(\frac{Q_d \cdot K_d^T}{\sqrt{2D}}) \in \mathbb{R}^{T \times N \times 1 \times (k+1)}
$$

> Q2:为什么两个地方越像，注意力越大？
>
> 这句话是错的，因为取决于你对数据的训练方式！在本次任务中，两个地方的特点越近似，我们就越需要注意！

### 注意力权重矩阵点积特征，得到xxx（不知道这个词的术语是什么，领据特征权重聚合矩阵？）

$$
X_d' = A_d \cdot V_d \in \mathbb{R}^{T \times N \times 2D}
$$

邻居特征聚合的动态嵌入（即利用注意力机制生成了一个权重矩阵$A_d$，就是谁和自己像，谁的权重就高，最后得到的就是xxx（领据特征权重聚合矩阵？）$X_d'$了）

### 残差链接

![\<img alt="" data-attachment-key="ZCD7EIXJ" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22FTUGKWGF%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225361%22%2C%22position%22%3A%7B%22pageIndex%22%3A5%2C%22rects%22%3A%5B%5B293.663%2C151.955%2C545.663%2C184.205%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225361%22%7D%7D" width="420" height="54" src="attachments/ZCD7EIXJ.png" ztype="zimage"> | 420](../../../assets/images/ZCD7EIXJ.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225361%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5361</a></span>)</span>

1.  **残差连接**：$$(Q_{d} + X_{d}^{\prime})$$

    *   $Q_d$ ：当前节点自己的特征。

    *   $X_d'$ ：聚合来的邻居特征。

    *   这一步把“自己”和“邻居”融合在了一起。

2.  **特征变换**

    *   把上一步相加的结果，作为一个整体，乘以权重矩阵  $$w_{ds}$$

> 注意上面$X$的上角标1,这代表着第1轮SP Block机制得到的结果,再次回顾一下整个SP Block的构建流程,这时候会发现上一次的输出可以用于下一层的输入,本质上是一个多轮迭代邻居的过程.比如,第一轮当前块Cur只能吸收到起邻居B的特征并与其聚合,同时!B也会吸收其邻居C并与其聚合,这时候的B就也有了C的信息.在第二轮的时候A再次吸收B的信息,注意这时候B里面就又有了C的信息,所以第二轮聚合的时候,Cur实际上也有C的信息了,Gemini给出的更严谨解释如下
>
> 1.  **并行更新 (Parallel Update)：** 在每一层  $r$  中，图中的所有节点（如 Cur, B, C）同时聚合其直接邻居的信息。
>
> 2.  **信息传递 (Message Passing)：**
>
>     *   在 **Layer 1**， $B$  聚合了  $C$  的原始特征，生成了包含一阶邻居信息的表征  $B^{(1)}$ 。此时  $\text{Cur}$  仅聚合了  $B$  的原始特征。
>
>     *   在 **Layer 2**， $\text{Cur}$  再次聚合  $B$ ，但此时输入的是更新后的  $B^{(1)}$ （其中已隐含  $C$  的信息）。
>
> 3.  **感受野扩张 (Receptive Field Expansion)：** 通过这种方式，经过  $r$  轮迭代， $\text{Cur}$  能够有效地捕获距离为  $r$ -hops 的高阶邻居（如  $C$ ）的空间依赖关系，实现从局部到全局的特征融合。
>
> ###
>
> ### **GNN 的感受野扩展机制**

### 基于多图卷积网络(Multi-GCN)的联合表征学习

![\<img alt="" data-attachment-key="9EJI99NP" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22RS2ANUHG%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225362%22%2C%22position%22%3A%7B%22pageIndex%22%3A6%2C%22rects%22%3A%5B%5B21.413%2C691.28%2C279.413%2C732.53%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%7D" width="430" height="69" src="attachments/9EJI99NP.png" ztype="zimage"> | 430](../../../assets/images/9EJI99NP.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5362</a></span>)</span>

这一步是对不同的视角下的SP Block进行加权求和再激活的过程.

$$ X_{(f, \mathcal{G}_F)}^r$$ 是表示**在「功能相似图」的纬度**$\mathcal{G}_F$\*\* 上，<span style="color: rgb(14, 14, 14)">经过 r 轮 SP block 后得到的特征</span>\*\*

$\odot$表示哈达玛积，也就是简单的对应位置的元素相乘的操作，这是在学习来自哪张图的预测信息对最终预测更重要

## SNIPER模型

![\<img alt="" data-attachment-key="Y87WBCK7" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22HEPF3VVR%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225362%22%2C%22position%22%3A%7B%22pageIndex%22%3A6%2C%22rects%22%3A%5B%5B21.413%2C555.53%2C279.413%2C601.28%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%7D" width="430" height="76" src="attachments/Y87WBCK7.png" ztype="zimage"> | 430](../../../assets/images/Y87WBCK7.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5362</a></span>)</span>

> Q:这里的r表示SP块的个数，但是之前在上面计算的时候，r不是代表SP Block的迭代轮次吗？
>
> 在这里，**“SP Block 的个数”** 和 **“迭代轮次”** 指的是**同一件事**，只是从不同的角度去描述。
>
> 当我们说“$r$ 是 SP Block 的个数”时，是在描述模型的**架构深度**。
>
> *   **就像搭积木：** 模型在构建时，物理上实例化了  $r$  个 SP Block 模块，像汉堡包一样一层层叠起来。
>
> 当我们说“$r$ 代表邻居聚合的轮次”时，是在描述数据的**流动过程**。
>
> *   **数据流向：** 数据必须先流过第 1 个块，处理完的结果变成第 2 个块的输入，以此类推。
> *   **代码视角：** 这是前向传播过程（`forward`）。

![\<img alt="" data-attachment-key="A7FLMLX5" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22LGSPCRZJ%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225362%22%2C%22position%22%3A%7B%22pageIndex%22%3A6%2C%22rects%22%3A%5B%5B21.413%2C164.78%2C280.913%2C557.03%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%7D" width="433" height="655" src="attachments/A7FLMLX5.png" ztype="zimage"> | 433](../../../assets/images/A7FLMLX5.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5362</a></span>)</span>

###

![\<img alt="" data-attachment-key="M5TZT2MK" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22PELIAMJU%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225362%22%2C%22position%22%3A%7B%22pageIndex%22%3A6%2C%22rects%22%3A%5B%5B289.913%2C666.53%2C544.913%2C689.03%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%7D" width="425" height="38" src="attachments/M5TZT2MK.png" ztype="zimage"> | 425](../../../assets/images/M5TZT2MK.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5362</a></span>)</span>

1.  **降维 (FC):** 你的  $\hat{X}_f$  是  $4D$  维的，太大了。先通过一个全连接层（Fully Connected）把它压扁一点，提取最精华的信息。

2.  **时序分析 (LSTM):** 把压扁后的序列喂给 LSTM，让它从  $t-T$  时刻一直看到  $t$  时刻。

3.  **预测:** 拿出 LSTM 在最后时刻吐出来的那个向量，映射成 0\~1 的概率。

![\<img alt="" data-attachment-key="QXJAYQ7H" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22R577PDNF%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225362%22%2C%22position%22%3A%7B%22pageIndex%22%3A6%2C%22rects%22%3A%5B%5B286.913%2C519.53%2C544.913%2C555.53%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%7D" width="430" height="60" src="attachments/QXJAYQ7H.png" ztype="zimage"> | 430](../../../assets/images/QXJAYQ7H.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5362</a></span>)</span>

![\<img alt="" data-attachment-key="ERIBH5WC" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22FJKKEP6W%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225362%22%2C%22position%22%3A%7B%22pageIndex%22%3A6%2C%22rects%22%3A%5B%5B286.913%2C434.78%2C542.663%2C470.03%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%7D" width="426" height="59" src="attachments/ERIBH5WC.png" ztype="zimage"> | 426](../../../assets/images/ERIBH5WC.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5362</a></span>)</span>

![\<img alt="" data-attachment-key="ZHFM2WYG" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%22CYBSV4T8%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225362%22%2C%22position%22%3A%7B%22pageIndex%22%3A6%2C%22rects%22%3A%5B%5B289.913%2C348.53%2C547.163%2C371.78%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%7D" width="429" height="39" src="attachments/ZHFM2WYG.png" ztype="zimage"> | 429](../../../assets/images/ZHFM2WYG.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5362</a></span>)</span>

![\<img alt="" data-attachment-key="3CKZKTDM" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FIHX7ICYP%22%2C%22annotationKey%22%3A%228326JMLC%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%225362%22%2C%22position%22%3A%7B%22pageIndex%22%3A6%2C%22rects%22%3A%5B%5B289.163%2C115.28%2C544.913%2C154.28%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%7D" width="426" height="65" src="attachments/3CKZKTDM.png" ztype="zimage"> | 426](../../../assets/images/3CKZKTDM.png)\
<span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16302842%2Fitems%2FRU6UNQWP%22%5D%2C%22locator%22%3A%225362%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/RU6UNQWP">Liu 等, 2023, p. 5362</a></span>)</span>
