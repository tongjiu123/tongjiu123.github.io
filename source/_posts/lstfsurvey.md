---
title: LSTF survey
date: 2023-07-22
categories:
  - Paper
tags:
  - LSTF
---

Time Series Forecasting blend in Online Interval Join

<!-- more -->

## 背景Background

流式处理系统中，一般使用key-partitioned based join 算法（key-OIJ）所处理的信息主要为：金融、风控、推荐等领域的毫秒级实时流式特征计算

Scalable Online Interval Join on Modern Multicore Processors in OpenMLDB中提出的scale-OIJ解决了1.work load不平衡数据倾斜； 2.重叠窗口带来的重复计算； 3.数据无序带来的数据扫描

## 动机Motivation

通过时序预测的方法

这个RP的主要目标是通过引入时序预测技术来降低OIJ的delay。我们的研究关注于使用时序预测方法来预测时间间隔数据的发展趋势，并利用这些预测结果来提前进行连接操作，从而减少实际数据到达所需的等待时间。

openMLDB需要较低的时延以满足需求

scale-OIJ是将两个数据流进行聚合操作。因此我们可以对数据流可以使用time series forecasting进行提前预测，提前进行连接操作来降低时延

## 先前的研究Related work

在时序预测方面，基于统计模型的方法被广泛应用于预测时间序列数据。例如，ARIMA模型结合自回归、差分和移动平均等技术来建模时间序列的趋势和季节性。指数平滑法（ETS）则利用加权平均和趋势调整来对时间序列数据进行预测。

基于深度学习的时序预测方法在近年来得到了广泛关注。深度学习模型如循环神经网络（RNN）、长短期记忆网络（LSTM）和Transformer等，具有捕捉时间序列复杂模式和长期依赖关系的能力。

近三年，使用MLP模型来进行时序预测的文章层出不穷，但是其是否绝对比传统方法效果好仍存在质疑。

## 模型选择 Model

在本研究中，我们将选择适合于时间序列数据预测的模型。我们将考虑传统的统计模型如ARIMA、指数平滑法（ETS）以及基于深度学习的模型如RNN、LSTM和Transformer等。

分别选出传统统计模型中的"轻"方案和使用机器学习的"重"方案。

### 传统·统计方法

#### 1.ARIMA 差分自回归移动平均模型

ARIMA 是用于单变量时间序列数据预测的最广泛使用方法之一

优点：模型十分简单，只需要内生变量而不需要借助其他外生变量

缺点：要求时序数据是稳定的；本质上只能捕捉线性关系，不能捕捉非线性关系

#### 2.ETS指数平滑法

基本原理：指数平滑法是移动平均法中的一种，其特点在于给过去的观测值不一样的权重，即较近期观测值的权数比较远期观测值的权数要大。

指数平滑法的基本公式：St=a*yt+(1-a)*St-1

### 深度学习方法

#### FEDformer

- **paper**：Frequency Enhanced Decomposed Transformer for Long-term Series Forecasting
- **作者**: [阿里巴巴达摩院]
- **会议/期刊**: [ICML]
- **年份**: [2022]
- **code**：[https://github.com/MAZiqing/FEDformer](https://github.com/MAZiqing/FEDformer)
- **摘要**: FEDformer模型在多个真实世界数据集上都表现出了很好的性能，具有较低的计算复杂度和内存复杂度，适用于大规模时间序列预测任务。
- **贡献**：
  1. 提出了一种基于频域增强的分解Transformer架构，采用专家混合技术进行季节性趋势分解
  2. 提出了傅里叶增强块和小波增强块
  3. 通过随机选择一定数量的傅里叶分量，该模型实现了线性计算复杂度和内存成本
  4. 在6个基准数据集上进行了广泛的实验，分别提高了14.8%和22.6%

FEDformer( Frequency Enhanced Decomposed Transformer)是一种针对长期序列预测的新型Transformer模型。融合transformer和经典信号处理方法。

###### 模型

![FEDformer模型](4a86682f91355cefbb7d3631a0a45a7.png)

Encoder部分：输入经过两个MOE Decomp层，每层将信号分解为S和T，S被传递给接下来的层学习，并最终传给解码器。T被舍弃

Decoder部分：Encoder的输入经过三个MOE Decomp层分解为S和T，S传递给接下来的层进行学习，通过频域Attention层对编码器和解码器的S项进行频域关联性学习，T分量则进行累加最终加回给S项以还原原始序列

###### 模块

频域学习模块 FEB：采用一个全连接层R作为可学习的参数。

频域注意力模块 FEA：将来自编码器和解码器的信号进行cross attention操作。

周期-趋势分解模块 MOE Decomp：将序列分解为S和T，并且分解不止一次，反复分解。

前向传播模块 Feed Forward。

#### TIDE

- Long-term Forecasting with TiDE: Time-series Dense Encoder
- **作者**: [GOOGLE]
- **年份**: [2023]
- **摘要**: 提出TiDE模型（新型多层感知器（MLP）编码器-解码器模型），整个模型没有任何注意力机制、RNN或CNN，完全由全连接组成。
- **code**：[https://github.com/google-research/google-research/tree/master/tide](https://github.com/google-research/google-research/tree/master/tide)
- **comment**：TIDE相对于其他模型，其没有使用自注意力机制，而且准确率很高，且速度快

![TiDE模型](v2-082a1616973b0539cda4e63e08893840_b.jpg)

###### 模型结构

TiDE整个模型结构全部由MLP组成，重点解决之前线性模型无法建模预测窗口与历史窗口非线性关系、无法有效建模外部变量等问题。

模型的核心基础组件是Residual Block。模型整体可以分为**Feature Projection、Dense Encoder、Dense Decoder、Temporal Decoder**四个部分。

#### FiLM:

- Frequency improved Legendre Memory Model for Long-term Time Series Forecasting
- **作者**: [Yifan Guo, Zhiwen Yu, Jianxin Li, Shenghua Liu]
- **会议/期刊**: [NeurIPS]
- **年份**: [2022]
- **code**：[https://github.com/tianzhou2011/FiLM/](https://github.com/tianzhou2011/FiLM/)
- **comment**：使用勒让德多项式投影，在高维空间中寻找其特征，增加准确率，对于LSTF效果不错

###### 贡献

1. 提出了一种FiLM模型，它采用专家混合的方法进行多尺度时间序列特征提取。
2. 重新设计了Legendre Projection Unit (LPU)。
3. 提出了一种Frequency Enhanced Layers (FEL)方法。
4. 在多个领域的六个基准数据集上进行了大量实验，提高了19.2%和26.1%的性能。

#### DeepTime

- Deep Time-Index Meta-Learning for Non-Stationary Time-Series Forecasting
- **作者**: [Gerald Woo ，Chenghao Liu ，Doyen Sahoo ， Akshat Kumar ，Steven Hoi ]
- **年份**: [2022]
- **code**：[https://github.com/salesforce/DeepTime](https://github.com/salesforce/DeepTime)
- **comment**：使用元数据学习方法，不需要进行内部梯度下降，因此可以避免梯度消失和梯度爆炸等问题

#### Autoformer

- Decomposition Transformers with Auto-Correlation for Long-Term Series Forecasting
- **作者**: [Haixu Wu, Jiehui Xu, Jianmin Wang, Mingsheng Long ]
- **年份**: [2022]
- **code**：[https://github.com/thuml/Autoformer](https://github.com/thuml/Autoformer)
- **comment**：自相关机制取代自注意力机制。Autoformer通过渐进式分解和序列级连接，应对复杂时间模式以及信息利用瓶颈，大幅提高了长时预测效果。

![Autoformer](v2-56dd4ca871cb71611561c2bf2bd1ab24_b.jpg)

##### AUTOformer创新

分解架构：突破将时序分解作为预处理的传统方法，设计序列分解单元以嵌入深度模型，实现渐进式地预测。

**自相关（Auto-Correlation）机制**：基于随机过程理论，丢弃点向连接的自注意力机制，实现序列级连接的自相关机制。

在能源、交通、经济、气象、疾病五大领域取得了38%的大幅效果提升。

#### Informer

- Beyond Efficient Transformer for Long Sequence Time-Series Forecasting
- **作者**: [Haoyi Zhou, Shanghang Zhang, Jieqi Peng, Shuai Zhang, Jianxin Li, Hui Xiong, Wancai Zhang]
- **会议/期刊**: [AAAI]
- **年份**: [2021]
- **code**：[https://github.com/zhouhaoyi/Informer2020](https://github.com/zhouhaoyi/Informer2020?utm_source=catalyzex.com)
- **贡献**:
  1. 提出了ProbSparse自注意机制，实现了O(L log L)的时间复杂度
  2. 提出了自注意力蒸馏操作
  3. 提出了生成式解码器，只需一步即可获得长序列输出
- **comment**：主要是改善了transformer在LSTF上表现得不足，并且具有优异的时间和空间复杂度。

##### methodology

**1.高效self-attention机制**

**2.编码器：允许在内存使用限制下处理更长的输入序列**

**3.解码器：通过一个前向过程生成长序列输出**

![Informer模型](v2-03ba0f2b93e5756396cb38ac474883b0_b.jpg)

#### Pyraformer

- Low-Complexity Pyramidal Attention for Long-Range Time Series Modeling and Forecasting
- **作者**: [Shizhan Liu, Hang Yu, Cong Liao, Jianguo Li, Weiyao Lin, Alex X. Liu, and Schahram Dustdar]
- **会议/期刊**: [ICLR]
- **年份**: [2022]
- **code**：[https://github.com/ant-research/Pyraformer](https://github.com/ant-research/Pyraformer)
- **comment**：Pyraformer的时间和空间复杂度较低，同时可以同时捕捉不同范围的时间依赖关系。

#### N-HiTS

- Neural Hierarchical Interpolation for Time Series Forecasting
- **作者**: [Cristian Challu, Kin G. Olivares, Boris N. Oreshkin, Federico Garza, Max Mergenthaler-Canseco, Artur Dubrawski]
- **会议/期刊**: [AAAI]
- **年份**: [2023]
- **code**：[https://github.com/Nixtla/neuralforecast](https://github.com/Nixtla/neuralforecast)
- **comment**：NBEATS模型主要是对于LSTF有着重的提升

###### Multi-Rate Data Sampling

用下采样（时域上的最大池化）将时间序列采样为多种粒度的序列。

###### Hierarchical Interpolation

和下采样是对应的，在预测结果上又做了个上采样。**所以每个stack实际上都是负责不同尺度的预测，最后把不同尺度的预测序列插值到相同粒度然后相加即可**。

### 传统统计方法vs深度学习方法

#### Are Transformers Effective for Time Series Forecasting

- **作者**: [Ailing Zeng, Muxi Chen, Lei Zhang, Qiang Xu]
- **会议/期刊**: [AAAI]
- **年份**: [2023]
- **摘要**: 引入了一组名为LTSF-Linear的简单的单层线性模型进行比较。在九个真实数据集上的实验结果表明，LTSF-Linear在大部分情况下都出人意料地优于现有的复杂Transformer-based LTSF模型。

### Transformer vs RNN

#### Enhancing the Locality and Breaking the Memory Bottleneck of Transformer on Time Series Forecasting

- **作者**: [Xiaoyong Jin、Yao Xuan、Xiyou Zhou、Wenhu Chen、Yu-Xiang Wang和Xifeng Yan]
- **会议/期刊**: [NIPS]
- **年份**: [2020]
- **摘要**: 提出了卷积自注意力和LogSparse Transformer来解决传统Transformer在时间序列中的局部性和内存瓶颈问题。
- **comment**：相较于传统的RNN，基于transformer的模型可以更好地处理时间序列中的局部性和长期依赖性。
