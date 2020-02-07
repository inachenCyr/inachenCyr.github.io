---
layout:     post
title:      GANerated Hands for Real-Time 3D Hand Tracking from Monocular RGB
subtitle:   CVPR2019
date:       2020-02-07
author:     chenyiru
header-img: img/post-2020-02-07-head.jpg
catalog: true
tags:
    - 论文阅读
    - 手势识别
	- CVPR
---

## GANerated Hands for Real-Time 3D Hand Tracking from Monocular RGB

### Content

1. Highlights
2. 核心模块
3. 效果
4. 总结

### Highlights

#### **Problem**

- 如何能直接得到手势姿态关键点的3D世界坐标？
- 如何获取精确的3Dhand pose annotation？

#### **Solutions**

- 文章提出了一种能够保持手部姿态不变的image-to-image transfer的GeoConGAN来生成训练数据；
- 文章提出的GeoConGAN不需要paired-image输入；
- 文章采用了kinematic hand model来提高有物体遮挡情况下手势姿态回归的精度；
- 文章提出的骨架匹配方法直接给出手势姿态的三维世界坐标；

#### 核心模块

##### **1. GeoConGAN**

这个GAN网络的目的是将通过软件仿真出来手转化成非常逼真的手，即给假手渲染上真实的肤色和纹理。通过这种image-to-image的translation即可减少训练数据与真实数据之间的gap。在网络设计上，作者提出了两个约束条件：

1. 训练的input image 不需要 synthetic-real 的 paired image
2. 生成的 'real image' 要和input的synthetic image 保持相同的手势姿态，即 geometrically consistent

GeoConGAN的网络架构如下图所示：

![](/Users/chenyiru/blogs/inachencyr.github.io/img/post-2020-02-07-t1.jpg)

- 从框图上能比较明显的看出GeoConGAN是基于CycleGAN的网络，采用的CycleGAN的目的就是为了满足第一个约束条件，unpaired-image input，这也是CycleGAN最重要的一个特征性。
- 为了满足第二个约束条件，作者采用了cross-entropy loss 作为生成器的loss，以保证hand pose的consistent。（这里通过这个loss为什么能实现酱紫的效果，论文并没有具体介绍，也没有给出具体公式）
- 在这个模块中，有一个细节，即生成图像和真实图像都是白底的图像，这么做一方面能使GeoConGAN网络能聚焦于手部特征和细节的学习，二方面是能够通过SilNet来计算 geometrically consistent loss，三方面也是方面于或许做背景增强。

##### **2. Kinematic Skeleton Fitting**

作者首先根据DOF手关节自由度模型建立了kinematic hand model 的表示，每个关键点表示为Θ = (t, R, θ)，其中，t是关键点的3维世界坐标，R是根节点的rotation，θ是每个关节点的自由度的链接角度。 - M(Θ) ∈ R 则表示了由21个关键点构成的手势。 - 通过遍历3个参数的所有取值组合构建了一个 kinematic tree。 - 通过公式E(Θ) = E2D(Θ)+E3D(Θ)+Elimits(Θ)+Etemp(Θ) 来计算输入的2D keypoints heatmap 和 3D relative joints 最匹配的hand model - 这个部分有一个细节是，用户需要在相机前的固定位置（论文里是45cm）完成一个初始化操作，来计算手部的scale

#### 效果

- 文章在 **STB**[1] 、 **Dexter+Object**[2] 和 **EgoDexter**[3] 数据集上评估了效果, 并主要和Z&B[4]的工作做了比较。具体比较如下，括号中是Z&B 的结果：
- STB : AUC = 0.965 (0.948)
- Dexter+object: AUC = 0.64 （0.49）
- EgoDexter: AUC = 0.54 （0.44）

#### 结论

- 文章最大的亮点是提出了GeoConGAN来解决训生成的练数据与真实数据的domain gap；
- 文章通过匹配hand model来直接计算3D姿态的世界坐标，但匹配过程和需要初始化的启动感觉略为复杂；
- 文章通过hand model匹配的方式来解决手部有物体遮挡的case，效果提升比较明显。

#### Ref

[1] 3d hand pose tracking and estimation using stereo matching

[2]  Real-time Joint Tracking of a Hand Manipulating an Object from RGB-D Input

[3] Real-time hand tracking under occlusion from an egocentric rgb-d sensor

[4] LearningtoEstimate3DHand Pose from Single RGB Images