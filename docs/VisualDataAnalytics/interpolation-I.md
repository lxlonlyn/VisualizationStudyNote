# Data Reconstruction and Interpolation I

## Data reconstruction 数据重构

### 描述

将离散的样本转化为连续的表示。

## Continuous representation 连续表示

### 任务说明

给定分散的点的集合 $p_i$，以及其标量值 $f_i$。构造连续函数 $f$ 使得 $f$ 能大体表示 $(p_i,f_i)$ 的情况。

### 光滑：Radial Basis Functions 径向基函数

#### 基本表示与特性

$$f(\pmb{x}) = \sum\limits_{i=1}^{N}w_i\varphi(||p_i-\pmb{x}||)$$

- 每一对 $(p_i,f_i)$ 都会对 $f(\pmb{x})$ 造成影响，取决于欧氏距离 $r=||p_i-\pmb{x}||$，点距离越近造成的影响越大；
- $\varphi(r)$ 函数：
    - $\varphi(r)=\varphi(||p_i-\pmb{x}||)$ 相对 $p_i$ 对称，且随着 $r$ 的增大，$\varphi(r)$ 下降非常迅速；
    - 教材给出的一种方案是 $\varphi(r)=e^{-r^2}$；
- $w_i$ 量：
    - $w_i$ 使得 $f(\pmb{x})$ 经过所有的 $(p_i,f_i)$，即**插值**；
    - $w_i$ 可以这样计算，列出下列公式后，等式左侧同乘矩阵的逆即可解决。也可以通过展开列出方程组解决。在解决这样的计算时，如果 $\varphi(r)$ 非常小则可以近似为 $0$：

$$
\left[
\begin{matrix}
 \varphi(||p_1-p_1||)      & \varphi(||p_2-p_1||)      & \cdots & \varphi(||p_N-p_1||)      \\
 \varphi(||p_1-p_2||)      & \varphi(||p_2-p_1||)      & \cdots & \varphi(||p_N-p_2||)      \\
 \vdots & \vdots & \ddots & \vdots \\
 \varphi(||p_1-p_N||)      & \varphi(||p_2-p_N||)      & \cdots & \varphi(||p_N-p_N||)      \\
\end{matrix}
\right]
\left[
\begin{matrix}
 w_1      \\
 w_2      \\
 \vdots \\
 w_N      \\
\end{matrix}
\right]
=
\left[
\begin{matrix}
 f_1      \\
 f_2      \\
 \vdots \\
 f_N      \\
\end{matrix}
\right]
$$

#### 缺点

- 每个采样点都会影响整个系统；
- 每次新增结点，都会重新计算整个系统；
- 计算复杂度很高（需要解线性方程）；

#### 改进：选择不同的径向函数 $\varphi(r)$

$$\varphi(r)=\frac{\frac{1}{r^2}}{\sum\limits_{i=1}^{N}{\frac{1}{d_i}}}$$

其中 $d_i=||p_i-\pmb{x}||$，这样可以得出：

$$f(\pmb{x})=\sum\limits_{i=1}^{N}f_i\varphi(||p_i-\pmb{x}||)=\frac{\sum\limits_{i=1}^{N}\frac{f_i}{||p_i-\pmb{x}||^2}}{\sum\limits_{i=1}^{N}\frac{1}{||p_i-\pmb{x}||^2}}$$

##### 插值性质的证明

对于特定的某项 $f_t\varphi(||p_t-\pmb{x}||)$ 可以表示为：

$$f_t\varphi(||p_t-\pmb{x}||)=\frac{\frac{f_t}{||p_t-\pmb{x}||^2}}{\sum\limits_{i=1}^{N}{\frac{1}{||p_i-\pmb{x}||}}}$$

当 $x\not=t$ 时，根据 $\varphi(r)$ 单调迅速递减的性质，其值近似于 0。当 $x=t$ 时，函数趋近于 $f_t$。

#### 改进后的优缺点分析

- 优点：省略计算步骤（不用求方程组）；
- 缺点：每个点仍对整个函数造成影响；

### 不光滑：triangulation 三角剖分

#### 理想的三角剖分

1. 三角形有良好的横纵比，尽量“圆”，避免过长；
2. 最大化最小的角度；
3. 最大化内接圆半径与外接圆半径比；
4. Delaunay 三角剖分是一种最佳方案；

#### Delaunay 三角剖分

##### 准则

如果要满足 Delaunay 三角剖分的定义，需要符合两个准则：

1. 空圆特性：在 Delaunay 三角形网中任一三角形外接圆内不会存在其他点；
2. 最大化最小角：在点集形成的所有三角剖分中，Delaunay 三角剖分形成的三角形的最小角最大；

##### 特性

1. 最接近：以最近的三个点形成三角形，各个线段互不相交；
2. 唯一性：无论从哪个区域开始构建，最终结果相同；
3. 最优性：任意两个相邻三角形组成的凸四边形的对角线如可以互换，则两个三角形六个内角中最小角不会变大；
4. 最规则：如果将三角网中每个三角形的最小角升序排列，则 Delaunay 三角剖分得到的数值越大（排名靠后）；
5. 区域性：新增、删除、移动某顶点时只会影响临近的三角形；
6. 凸多边形外围：三角网最外层边界围成一个凸多边形。