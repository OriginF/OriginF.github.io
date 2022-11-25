---
title: 'Monte Carlo path tracer'
date: 2022-11-23
permalink: /posts/2022/11/23/MontoCarlo
tags:
  - Monto Carlo
  - Path Tracing


---

使用蒙塔卡洛积分对光照模型中的光线追踪的光照强度结果进行估计



### 光照模型

- 局部光照：计算反射光/折射光的模型
  - 冯模型 Phong Model
  - 微表面模型 Microfacet Model
- 全局光照模型：直接光照/间接光照等
  - 路径追踪
  - 光子映射
  - 双向路径追踪
  - 辐射度法

### 光线追踪

- 计算全局光照的模型公式为空间积分过程。

- 积分的时候可以使用蒙特卡洛积分的方法

  - PDF：概率密度函数

  - 蒙塔卡洛方法就是将没有解析解的定积分使用划分区间采样的方法对结果进行估计。

  - 其积分公式如下：
    $$
    F = \int_{\Omega}f(x)dx \sim \hat F_n = \frac{1}{N} \sum_{i=1}^{N} \frac{f(x_i)}{pdf(x_i)} 
    $$

  - $x_i$为采样点，$f(x_i)$为采样点的值，$pdf(x_i)$表示采样点的概率密度函数，就是在该点的密度值。

  - 从而可以估计出该点的光照强度（能量）