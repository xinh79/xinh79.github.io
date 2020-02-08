---
layout:      post
title:       "Lyapunov优化理论"
subtitle:    "Lyapunov Optimization Theory"
author:      "Ashior"
header-img:  "img/post-bg-ubuntu.jpg"
catalog:     true
tags:
  - 科研
  - SDN
  - 博士
---

> 由于github无法显示显示LaTeX公式，方便起见，使用图片代替文字，但是再最后依旧会给出文字版，方便读者查阅。

----

![lyapunov](/img/Lyapunov.png)

----


## Lyapunov优化理论

----

#### 前言

[Michael J. Neely](http://www-bcf.usc.edu/~mjneely/) 教授提出来的Lyapunov Optimization，应用于网络优化理论，其**核心思想为稳定数据队列的同时，尽力最大化/最小化效用函数**。见最后的两篇主要参考文献，论文实际上有很多很多，但是笔者只挑一些列出来了，抛砖引玉，烦请诸位自行查找阅读辨别。文献[1](#参考文献)、[6](#参考文献) 本质上差不多，但是有了 [1](#参考文献) 的基础看 [6](#参考文献) 会比较轻松，都是一本书的厚度，加上全英文，一大半都是数学推导，公式请一带而过，**不要尝试公式推导！！！** 推完之后依旧会忘，怕出错请看文献 [6](#参考文献) ，文献 [1](#参考文献) 好像有一些错误。所以请读者自行分配时间。推荐入门还是先走维基百科，切记，**不要走国内的~~CSDN~~网站**，里面天花乱坠抄来抄去毫无干货，即使有也是关于**控制工程**中的原生优化理论。笔者曾经花了近一个月的闲散时间复习了线性代数，学习了控制工程相关理论知识。所以少走弯路能为你节省很多时间。除了参考文献，笔者还列出了参考网址，一定程度上可以帮助读者对Lyapunov优化理论的理解，同时了解其应用，理论基础在工科生手里，更重要的还是如何将其落地。

----

#### 网络模型

以最简单的排队网络模型为例。数据包的入队速度是$\lambda(t)$，出队速度是$\mu(t)$，目标是是怎么样调整$\lambda(t)$，使得队列**平均速率稳定**，同时达到最大的用户满意度。采用**效用函数(utility function)** $U$衡量。显而易见，请求的数据流速率$\lambda(t)$越大，用户的满意度越高，但是会导致队列长度。队列长度我们用$Q$表示。则$Q$随时间的变化可以表示为：

$$Q(t+1)=max[Q(t)-\mu(t),0]+\lambda(t)$$

我们此时获取了数据队列的长度，因此我们回到Lyapunov核心思想，稳定数据队列，那么转换为数学公式为：

$$\lim_{T \to \infty} \frac{\mathbb{E}(Q(t))}{T}=0$$

实际上，处于此状态的系统属于**强稳定系统**。此处不做深入探讨，详情请参考Neely教授论文。

那么我们之前所说的：稳定数据队列的同时，尽力最大化/最小化效用函数。就可以转换为如下表达式：

| $max:$ | $U$ |
|----:|:----|
| $subject:$ | $\lim_{T \to \infty} \frac{\mathbb{E}(Q(t))}{T}=0$ |

有了优化目标和限制条件，现在我们来做优化。我们假设$L(t)=\frac{{Q(t)}^2}{2}$，则$L$随时间变化的表达式为：$\Delta L(t)=L(t+1)-L(t)$。$\Delta L(t)$在论文中称为Lyapunov漂移(Lyapunov Drift)。

| ${Q(t+1)}^2$ | $=$ | ${\{max[Q(t)-\mu(t),0]+\lambda(t)\}}^2$ |
|----:|:----:|:----  |
|  | $\le$ | ${Q(t)}^2+{\mu(t)}^2+{\lambda(t)}^2-2\mu(t)Q(t)+2Q(t)(\lambda(t)-\mu(t))-2\lambda(t) \mu(t)$ |
|  | $\le$ | ${Q(t)}^2+{\mu(t)}^2+{\lambda(t)}^2+2Q(t)\lambda(t)$ |

$$
{Q(t+1)}^2-{Q(t)}^2 \le {\mu(t)}^2+{\lambda(t)}^2+2Q(t)\lambda(t)
$$

$$
\Delta L(t) \le B+Q(t)\lambda(t)
$$

此时变量$B$满足$B \ge \frac{\lambda(t)^2+\mu(t)^2}{2}$，此时请不要细究，以此为研究方向的同学可以从论文细究。接下来就是引出Lyapunov Drift Reward/Penalty，有说是Lyapunov奖励或者是漂移惩罚，在[文献11](#参考文献)中使用的是Lyapunov漂移加罚。表达式为：$\Delta L(t)-VU$，其中$V$为优化常量，通过此旋钮来权衡(tradeoff)系统队列与待优化的目标性能，使得总体和最优即可。这个问题就转化为根据队列的长度，调节速率的问题。

$$
\min Q(t)\lambda(t)-VU
$$

上述表达式可以求导，通过对参数$V$进行选择，从而达到对整个系统优化的目的。

对于常规的实验设计，可以采用OMNet++或者在NS3进行模拟实验。在笔者的[项目](https://github.com/xinh79/Lyapunov4OMNetpp) 中入队列速率采用指数分布(Exponential Distribution)，队列处理数据采用泊松分布(Possion Distribution)，通过在OMNet++5.4.1中对节点进行设置，然后通过过滤器(控制器)进行预设的算法，最后完成网络模型的模拟。

在[文献11](#参考文献)中通过对IoT-Edge-SDN架构模型下网络通信建模，设计了效用函数$U$与数据队列$Q$，通过OMNet++5.4.1进行仿真实验。

----

#### 参考网页

- [Ride论文的Github](https://github.com/KyleBenson/ride)
- [Drift plus penalty - Wikipedia](https://en.wikipedia.org/wiki/Drift_plus_penalty)
- [Backpressure routing - Wikipedia](https://en.wikipedia.org/wiki/Backpressure_routing)
- [Lyapunov optimization - Wikipedia](https://en.wikipedia.org/wiki/Lyapunov_optimization)
- [排队理论之性能分析 - Little Law & Utilization Law](https://blog.csdn.net/zhoucengchao/article/details/37974165)
- [算法博弈论(algorithmic game theory)](https://blog.csdn.net/golden1314521/article/details/51661281?locationnum=7)
- [建模算法(七)——排队论模型](https://www.cnblogs.com/BlueMountain-HaggenDazs/p/4270875.html)
- [一文读懂马尔科夫过程](https://blog.csdn.net/DeepOscar/article/details/81036635)

----

#### 参考文献

1. Georgiadis L , Neely M J , Tassiulas L . Resource Allocation and Cross-Layer Control in Wireless Networks[J]. Foundations and Trends? in Networking, 2005, 1(1):1-144.
2. Do N , Zhao Y , Wang S T , et al. Optimizing offline access to social network content on mobile devices[C]// Infocom, IEEE. IEEE, 2014.
3. Yang S , Adeel U , Mccann J . Backpressure meets taxes: Faithful data collection in stochastic mobile phone sensing systems[C]// IEEE INFOCOM 2015 - IEEE Conference on Computer Communications. IEEE, 2015.
4. Uddin M Y S , Setty V , Zhao Y , et al. RichNote: Adaptive Selection and Delivery of Rich Media Notifications to Mobile Users[C]// 2016 IEEE 36th International Conference on Distributed Computing Systems (ICDCS). IEEE, 2016.
5. Zhu Q , Uddin M Y S , Qin Z , et al. Upload Planning for Mobile Data Collection in Smart Community Internet-of-Things Deployments[C]// IEEE International Conference on Smart Computing. IEEE, 2016.
6. Neely M J. Stochastic network optimization with application to communication and queueing systems[J]. Synthesis Lectures on Communication Networks, 2010, 3(1): 1-211.
7. Benson K E, Wang G, Venkatasubramanian N, et al. Ride: A resilient iot data exchange middleware leveraging sdn and edge cloud resources[C]//2018 IEEE/ACM Third International Conference on Internet-of-Things Design and Implementation (IoTDI). IEEE, 2018: 72-83.
8. Zhu Q, Uddin M Y S, Venkatasubramanian N, et al. Spatiotemporal scheduling for crowd augmented urban sensing[C]//IEEE INFOCOM 2018-IEEE Conference on Computer Communications. IEEE, 2018: 1997-2005.
9. 林闯, 陈莹, 黄霁崴, et al. 服务计算中服务质量的多目标优化模型与求解研究[J]. 计算机学报, 2016, 38(10).
10. 面向多源大数据云端处理的成本最小化方法[J]. 软件学报, 2017(3).
11. Wu D, Huang X, Xie X, et al. LEDGE: Leveraging Edge Computing for Resilient Access Management of Mobile IoT[J]. IEEE Transactions on Mobile Computing, 2019.

