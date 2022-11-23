---
title: "Representing Scenes as Neural Radiance Fields for View Synthesis（NeRF）"
collection: talks
type: "Talk"
permalink: /talks/2012-03-01-talk-1
venue: "UC Berkeley"
date: 2022-03-01
location: "2022 CSCV"
paperurl: 'https://originf.github.io/files/NeRF.pdf'
---

对于NeRF相关的一些论文的阅读感悟



### [NeRF_Survey](https://originf.github.io/files/NeRF Survey.pdf)

- 。。。

### [NeRF](https://originf.github.io/files/NeRF.pdf)

- 方法比较粗暴，直接采用了两个8层MLP神经网络（coarse，fine）

- 首次coarse采用均匀采样，fine采用的是重要性采样。

- 网络的结构如下：

  ![截屏2022-11-23 17.04.55](NeRF.assets/%E6%88%AA%E5%B1%8F2022-11-23%2017.04.55.png)

- NDC下的归一化空间位置$\gamma(x)=(x,y,z)$

- 世界坐标系下的沿视线距离$\gamma(d)=d$

- 输出为空间点$(x,y,z)$的密度$\sigma \in [0,1]$,（这里的密度和透明度$\alpha$和为1）

- 输出为空间点$(x,y,z)$的颜色$(r,g,b)$

- 一些数据集的数据中，内参矩阵中y和z都是负的，需要正则化

- 输入的数据x和d均是通过了位置编码（Position Encoding）传入的，可以减少高频图像的敏感度

### [InstantNeRF(5s)](https://originf.github.io/files/5s_NeRF.pdf)

### [LLFF](https://originf.github.io/files/LLFF.pdf)

