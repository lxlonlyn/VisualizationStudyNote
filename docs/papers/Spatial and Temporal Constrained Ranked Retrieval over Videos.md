# Spatial and Temporal Constrained Ranked Retrieval over Videos

## 相关文件 

[原文 PDF](origin/Spatial%20and%20Temporal%20Constrained%20Ranked%20Retrieval%20over%20Videos.pdf)

[讲解 PPT](ppt/2022.10.18%20STAR%20Retrievals.pptx)

## 问题定义

一部视频可以看做是由很多帧组成的序列，表示为 $\textit{video}=<f_1,f_2,\dots,f_n>$；

对于某一帧来说，其中有很多对象，给予每个对象一个特殊的 $\textit{ID}$；同时会有很多的标签，表示为 $\textit{L}$；

**Object Graph 对象图** 由帧抽象得到的图，保留以下三个信息：

- 结点（对象 ID）
- 结点属性（对象属性）
- 边属性（结点间的空间关系）

**Query Graph 查询图** 给定查询的单个帧相对应的对象图

**Data Graph 数据图** 由视频中帧构造而来的对象图

**Query Graph Matching 查询图匹配** 给定查询图 $P=(V_P,E_P)$ 和数据图 $G=(V_G,E_G)$，查询图匹配返回 $G$ 的一个子图 $M=(V_M,E_M)$，该子图满足：

存在双射函数 $h:V_P\to V_M$，使得：

- 子图结点与查询图结点一一对应，且每个子图结点包含对应查询图结点的所有标签： $\forall v\in V_P, L_P(v)\in L_M(h(v))$
- 子图边与查询图边一一对应： $\forall (u,v,\theta,d)\in E_P, (h(u),h(v),\theta,d) \in E_M$

在 $h$ 函数条件下，$M$ 是 $P$ 的一个匹配，可以表示为 $P \underset {h}{\sim} M$ 。

在 $h$ 函数条件下，存在 $G$ 的一个子图 $M$，满足 $M$ 是 $P$ 的一个匹配，则可以表示为 $P \underset {h}{\sim} G$。 

**Query Graph Sequence Matching 查询图序列匹配** 