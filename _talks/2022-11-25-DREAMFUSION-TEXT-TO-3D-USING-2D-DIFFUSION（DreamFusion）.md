---
title: "DREAMFUSION: TEXT-TO-3D USING 2D DIFFUSION（DreamFusion）"
collection: talks
type: "Talk"
permalink: /talks/2022-11-25-talk-1
venue: "UC Berkeley"
date: 2022-11-25
location: "2022 Siggraph"
paperurl: 'https://originf.github.io/files/Diffusion/DreamFusion.pdf'

---

对于Diffusion(DreamFusion)相关的一些论文的阅读感悟

### [<u>Diffusion</u>](https://originf.github.io/files/Diffusion/Diffusion.pdf)

- 马尔可夫链$A \rightarrow B \rightarrow C$有如下特征公式：
  $$
  P(A,B,C) = P(C|B,A)P(B|A)P(A)  = P(C|B)P(B|A)P(A)\\
  P(C,B|A) = P(C|B)P(B|A)
  $$

- 满足高斯分布的两个随机变量$p \sim N(\mu_1,\sigma_1^2)$和$q \sim N(\mu_2,\sigma_2^2)$，有如下KL散度表达式：
  $$
  D_{KL}(p,q) = log(\frac{\sigma_2}{\sigma_1}) + \frac{\sigma_1^2 + (\mu_1 - \mu_2)^2}{2\sigma_2^2} - \frac{1}{2} \tag{KL}
  $$

- 重参数技巧：

  为了在$N(\mu,\sigma^2)$中采样x，先取$z \sim N(0,1)$，那么有$x = \sigma \times z + \mu$，可以对z进行采样，然后返回时对$\sigma$和$\mu$进行梯度传播。（此时z表示一个常量）。为了保证可导，先抽样，在进行变化即可。

- 两个过程：

  - 正向过程（扩散过程）：从原图$T_0$不断添加高斯噪声最终变成$T_s$完全变为高斯分布。这个过程为马尔可夫链，且添加的过程中均值和方差是不变的。
  - 逆向过程（逆扩散过程，重建过程）：从高斯分布$T_s$不断的收缩的原图的过程$T_s$

- 正向过程：确定的参数$\beta_t$和对应的状态$x_t$（其中$\beta_1 < \beta_2 < ... < \beta_t$）
  $$
  p(x_{t}|x_{t-1}) = N(x_t:\sqrt{1-\beta_t} \times x_{t-1}, \beta_t I)
  $$

- 正向过程：使用重参数技巧的公式可以推得以下两个公式。
  $$
  \begin{align}
  &\alpha_t = {1-\beta_t} \rightarrow \\
  &\bar{\alpha}_t = \prod \limits_{\tau=0}^t \alpha_t \rightarrow \\
  &x_t = \sqrt{\bar{\alpha}_t}\times x_{0} + \sqrt{1-\bar{\alpha}_t} \times z_{t-1},  z_{t-1} \sim N(0,1) \rightarrow \\
  &q(x_t|x_0) = N(x_0:\sqrt{\bar{\alpha}_t}\times x_0,\sqrt{1-\bar{\alpha}_t} \times I)
  \end{align}
  $$
  由此可以得到成为标准分布的时刻t。

- 逆向过程：确定了初始的分布从而拿到对应的联合分布的概率：
  $$
  p_{\theta}(x_{0:T}) = p(x_T) \prod_{\tau = 1}^{T} p(x_{\tau - 1}|x_{\tau}) \\
  p_{\theta}(x_{t-1}|x_t) = N(x_{t-1}:\mu_s(x_t,t),\Sigma_s(x_t,t))
  $$
  每个逆向概率的期望和方差都是x_t和t的函数，其中$\Sigma_s(x_t,t)$被论文设置为常数$\sigma_s$

- 后验概率q：
  $$
  q(x_{t-1}|x_{t},x_0)=N(x_{t-1}: :\widetilde{\mu}_t(x_t,x_0),\widetilde{\beta}_t) \\
  \widetilde{\mu}_t = \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}x_t + \frac{\sqrt{\alpha_{t-1}}\beta_t}{1-\bar{\alpha}_t}x_0 \tag{M0}\\
  \widetilde{\beta}_t = \frac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_t}x_0
  $$
  从而直接拿到对应的公式，不用在进行一些复杂的先验后验计算。

  - 根据前向传递有：
    $$
    x_0 = \frac{1}{\bar{\alpha}_t}(x_t - \sqrt{1-\bar{\alpha}_t} \times z_t)
    $$

  - 带入上式：
    $$
    \widetilde{\mu}_t = \frac{1}{\sqrt{\alpha_t}}(x_t - \frac{\beta_t}{\sqrt{1-\bar{\alpha}_t}}\times z_t) \tag{M1}\\ 
    $$

  - 从而得到:
    $$
    q(x_{t-1}|x_t)=N(x_{t-1}:\widetilde{\mu}_t(x_t),\widetilde{\beta}_t)
    $$

- Loss:
  $$
  L_{VLB} = E_{q(x_{0:T})}[log\frac{q(x_{1:T}|x_0)}{p_{\theta}(x_{0:T})}] \geq \{H(p_{\theta},q) = E_{q(x_0)}(log ~p_{\theta}(x_0))\}
  $$
  其中$H$表示交叉熵，为两个分布之间的差别的量度，最小化$L_{VLB}$就可以最小化$H$。

  而$L_{VLB}$可以化简为：
  $$
  \begin{align}
  L_{VLB} =&\underbrace{E_q[D_{KL}(q(x_T|x_0)||p_{\theta}(x_T))]}_{L_T}+ \tag{L1}\\ &\underbrace{E_q[\sum_{\tau=2}^TD_{KL}(q(x_{t-1}|x_t,x_0)||p_{\theta}(x_{t-1}|x_t))]}_{L_{t-1}}+ \tag{L2}\\ &\underbrace{E_q[-log~p_{\theta}(x_0|x_1)]}_{L_0} \tag{L3}
  \end{align}
  $$

- 首先是$L_{t-1}$，L2带入(KL)并将常数部分记为C
  $$
  \begin{align}
  &L_{t-1}= E_q[\frac{1}{2\sigma_t^2}||\widetilde\mu_t(x_t,x_0),\mu_{\theta(x_t,t)}||^2] + C \\
  \rightarrow~~~~&... \\
  \rightarrow~~~~&L_{simple}(\theta) = E_{t,x_0,\epsilon}[||\epsilon - \epsilon_{\theta}(\sqrt{\bar\alpha_t \times \epsilon, t})||^2]
  \end{align}
  $$

- 所以训练过程就相当于优化一个网络$\epsilon_{\theta}$

- 代码过程：

  - 根据上述公式有如下训练过程Training:

  $$
  \begin{align}
  &1: repeat \\
  &2: x_0 \sim q(x_0) ~~~从输入的样本（训练集）空间中采样出一个x_0 \\
  &3: t \sim Uniform({1,...,T}) ~~~从T中随机插值出t个数据 \\
  &4: \epsilon \sim N(0,1) ~~~从正态分布中采样出一个随机值（产生随机性）\\
  &5: Take~gradient~descent~step~as: \\
  &~~~~~~~~~~~~~~~~\Delta_{\theta}||\epsilon - \epsilon_{\theta}(\sqrt{\bar\alpha_t}\times x_0+\sqrt{1-\bar\alpha_t}\times \epsilon,t)||^2  ~~~\epsilon_{\theta}为一个训练出的网络\\ 
  &6:until~converged
  \end{align}
  $$

  - 反向过程（采样过程）Sampling:

  $$
  \begin{align}
  &1:x_T \sim N((0,I) ~~~~从高斯分布中采样出一个点 \\
  &2:for~T,...,1~do 	~~~~从T到1依次迭代 \\
  &3:	~~~~z \sim N(0,I) ~if~t > 1, else~z = 0 ~~~~从正态分布中拿到一个z\\
  &4: ~~~~x_{t-1} = \frac{1}{\sqrt{\alpha_t}}(x_t - \frac{1-\alpha_t}{\sqrt{1-\bar\alpha_t}}\times \epsilon_{\theta}(x_t,t)) + \sigma_t\times z \\
  &5:end~for\\
  &6:return~x_0
  \end{align}
  $$

- 牛逼啊，一些优化主要是针对Sampling过程，减少采样计算的。

### [<u>Diffusion on Image Synthesis</u>](https://originf.github.io/files/Diffusion/DiffusionImage.pdf)

### **[DreamFusion](https://originf.github.io/files/Diffusion/DreamFusion.pdf)**

#### 	[pytorch实现库](https://github.com/OriginF/stable-dreamfusion)

- 很厉害的一篇文章，开了一个大坑。

- CLIP实现了一种从图像转文本技术应用到3D场景中的方法。

- KL散度：
  $$
  D_{KL} = -\sum_i P(i) ln \frac{Q(i)}{P(i)}
  $$

- 基于Diffusion的一个后续优化（*Ho et al., 2020; Kingma et al., 2021*）
  $$
  L_{Diff}(\phi,x) = E_{t \sim U(0,1),\epsilon \sim N(0,I)}[w(t)||\epsilon_{\phi}(\alpha_t x+\sigma_t\epsilon;t)-\epsilon||^2]
  $$
  等于是为每个t添加一个新的权重。

- 向$\epsilon_{\theta}$中添加参数y，表示一段话，从而产生text-to-image的效果。

- “frozen”的含义为：首先存在一个浅层的数据（预训练模型，可能已经使用A数据集训练好了），然后再训练（使用新的C数据集进行训练）的时候不再改变这些浅层的数据。这种方法称为frozen。

- 产生一个新的损失函数：
  $$
  \nabla_{\theta}L_{SDS}(\phi,x=g(\theta)) = \nabla_{\theta}E_t[\frac{\sigma_t}{\alpha_t}\omega(t)KL(q(z_t|g(\theta);y,t)||p_{\phi}(z_t;y,t))]
  $$
  ​		其中$\nabla_{\theta}$表示对$\theta$求偏导。$\theta$表示的是可微生成器g的参数，x为生成的图像。

- 本文不需要训练diffusion，仅仅是将其当作frozen使用。

- 采用分数函数替代了密度函数，用来绘制图像。

- 保留了Diffusion的过程，Diffusion每次根据对应的一句文本产生一个图像，然后对图像进行NeRF的一个L最小化迭代，最终使得整个结果的图像趋向于稳定即可。

![DreamF](2022-11-25-DREAMFUSION-TEXT-TO-3D-USING-2D-DIFFUSION%EF%BC%88DreamFusion%EF%BC%89.assets/DreamF.png)

可以看到，这里的这个loss：g就符合上述的说法了。

