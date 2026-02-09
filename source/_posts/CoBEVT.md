---
title: CoBEVT
date: 2023-07-22
categories:
  - Paper
tags:
  - Auto_vehicle
---

# CoBEVT：稀疏Transformer做协作BEV语义分割

<!-- more -->

UCLA大学在2022年发表在CoRL

Paper：[https://arxiv.org/abs/2207.02202](https://link.zhihu.com/?target=https://arxiv.org/abs/2207.02202)

- 场景：复杂交通场景中
- 问题：基于单智能体相机的系统对于复杂交通场景中的遮挡和检测远处的目标性能较差
- 动机：使用通用多智能体多相机感知框架（车与车( Vehicle-to-Vehicle，V2V )通信技术可以使无人驾驶车辆能够共享感知信息），提高感知性能和范围
- paper发表于CoRL 2022
- 多智体多摄像机感知框架
- 轴向注意力（fused axial attention，FAX）模块
- 数据集：V2V 感知数据集OPV2V；nuScenes

### Abstract

鸟瞰图( BEV )语义分割在自动驾驶的空间感知中起着至关重要的作用。尽管最近的文献在BEV地图理解方面取得了重大进展，但它们都是基于单智能体相机的系统。这些解决方案有时在处理复杂交通场景中的遮挡或检测远处物体时存在困难。车与车( Vehicle-to-Vehicle，V2V )通信技术使无人驾驶车辆能够共享感知信息，与单智能体系统相比，显著提高了感知性能和范围。

在本文中，我们提出了CoBEVT，这是第一个能够协同生成BEV地图预测的通用多智能体多相机感知框架。为了在Transformer底层架构中高效地融合来自多视图和多智能体数据的相机特征，我们设计了一个融合的轴向注意力模块( FAX )，该模块能够稀疏地捕获视图和智能体之间的局部和全局空间交互。

### CoBEVT

第一个使用多智能体多相机传感器通过稀疏视觉转换器协作生成BEV分割地图的框架

每个AV通过SinBEVT Transformer从其相机机架计算出自己的BEV，并在压缩后传输给其他AV。接收端将接收到的BEV特征变换到其坐标系下，并使用提出的FuseBEVT进行BEV级聚合。

![CoBEVT结构图](v2-f34e07352a28b095403c224b5331a71e_720w.webp)

CoBEVT结构图

### 前提

- 一个V2V通信系统，其中所有AV都可以与其他AV交换感知信息
- 所有智能体的pose都是准确的，并且传输的消息是同步的

### FAX — Fused Axial Attention

Sin BEVT和Fuse BEVT的核心组件

可以在局部和全局上有效地聚合跨代理或相机视图的特征。

具有很大的通用性，在不同的模态上表现出对多个感知任务的有效性，包括基于多视角相机的协同/单智能体BEV分割和协同3D LiDAR目标检测。

![FAX模块](v2-c450b0e4bbbf9763b37df7f81681c6f3_720w.webp)

### SinBEVT for Single-agent BEV Feature Computation

通过SinBEVT Transformer从其相机机架计算出自己的BEV

### FuseBEVT for Multi-agent BEV Feature Fusion

一个简单的1x1卷积自动编码器来压缩和解压BEV特征，以降低传输数据大小

FuseBEVT是一个3-D视觉transformer，可以专注地融合来自多智体的BEV特征信息

### 实验

摄影机追踪结果

LiDAR-track results.

nuScenes vehicle map-view segmentation

Effect of compression rate
