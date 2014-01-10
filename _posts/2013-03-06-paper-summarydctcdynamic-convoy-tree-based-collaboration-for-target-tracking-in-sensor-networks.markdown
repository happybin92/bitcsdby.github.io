---
author: bydiao
comments: true
date: 2013-03-06 08:27:28+00:00
layout: post
slug: paper-summarydctcdynamic-convoy-tree-based-collaboration-for-target-tracking-in-sensor-networks
title: '[Paper Summary]DCTC:Dynamic Convoy Tree-Based Collaboration for Target Tracking
  in Sensor Networks'
wordpress_id: 136
categories:
- WSN
tags:[Wireless Sensor Network]
---

_  
Authors:Wensheng Zhang,Guohong Cao
From:IEEE Transactions on Wireless Communications
Key word:Convoy tree,prediction,reconfiguration,WSN,target tracking.
Cition:349
Abstract:

背景介绍。我们提出了一种Convoy Tree-Based协同的Tracking Scheme。提出了一种在理想情况下的算法，并针对实际情况对算法做了改进。并简单介绍了实验结论。
Introduction:

介绍传感器的进展，大规模传感器的应用场景。
引出WSN Tracking的应用场景，**节点一般会有两种动作：存储感知信息，等待query或主动转发感知信息到sink**，并指出，Tracking中多传感器协同，对提高精度的重要性。
简述DCTC模型的工作流程，引入o-DCTC，一个DCTC的理想情况。作为后期模拟优化的参考值。
System Model

将所有节点建模为一个图G，并用以下公式建模能量，其中a,c为常量。
\(u_i=a \times d^2_i+c\)
该文章将TT建模成以下模型：
Target进入WSN范围时刻为\(T_s\)，离开时刻为\(T_e\)，在每一个单位时间内，应该参与Tracking的节点集合为\(S_t\)，其中\(S_t\)认为是以Target为圆心，\(d_s\)为半径的圆内的所有节点。
整个工作流程为:
(1)初始化DCTC
(2)Root collect and process data 
(3)tree prune and expansion
(4)Reconfigure the tree and go back to (2)                  
参数1：Tree Coverage定义为 :\(\frac{V_t \bigcap S_t}{S_t}\)
参数2：树的能耗定义为：\(E^d(T_t(R))=r \times \sum_{i \in V_t} e(i,R)\)
o-DCTC Scheme
在所有节点都已知整个网络拓扑的情况下，同时已知物体的运动轨迹，来算出一个能量最低的DCTC树序列。
主要用动态规划算出最小的能耗，用Dijkstra算法实现树的prune和expansion
但这种理想模型是不实用的，只能对模拟起到一定的参考作用。
Desine and Implementation of DCTC
(1)建树，这里提到了跟选择算法，分布式计算导论上有很多leader选择算法，有时间要看一下。本文用到的算法是类似FTSP的最小ID算法。
(2)剪枝和扩展，分为保护模式和预测模式
保护模式是将以traget为中心的\(V_t+\beta+d_s\)内的点全都参与tracking，这样可以达到很高的精度，但每个节点的能耗会很大。
预测模式是利用数据融合算法，只将下一时刻会出现在\(d_s\)范围内的点加入。由于预测算法的不准确，会导致coverage的降低。但很节能。
(3)重配
基于阈值启发的重配，当root与target的距离大于\(d_m+\alpha\times v_t\)时，root发出reconf包，触发树重配。
重配的基本步骤:
a.选根，root为一个grid header
b.从根开始，向相邻grid header发root内所有节点的location和与root通信cost。
c.root相邻grid header将信息转发到grid中的节点，所有节点计算自己加入哪个节点，可以与root的通信cost最小。
d.以此类推，直到超过\(d_s\)
这种方法在文中成为Sequential Reconfiguration
同时提出了一种改进方法：Localized Reconfiguration
由于Seq Reconf的通信开销太大，可以简化开销。首先，cost可以通过通信测距得到。其次，可以cache其他grid节点的位置信息，并cache自己已经将位置信息发给果哪些grid header，这就减少了很多重复的位置信息传递。进而大大降低通信开销。
Performance Evaluation
选用了ns2进行模拟仿真，结论与预测结论一致，在节点密度高和低的情况下，预测模式的能耗都要低于保护模式，局部重配的能耗要低于顺序重配模式。
Ideas...
（1）优化节点的加入与剪枝模式：DCTC算法是节点加入一个与自己距离最近的树内节点，但显然这是局部最优，不是全局最优。
     要想达到全局最优，只需要每个节点交互的事自己到root的距离即可。新节点可以根据能量模型选择自己到root的最有距离点加入。
