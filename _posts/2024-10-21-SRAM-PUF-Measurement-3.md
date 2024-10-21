---
layout: post
title: 论文｜主流SRAM PUF的评价方法-3
categories: [Coding Theory]
description: 几篇较新论文中SRAM PUF的评价方法。
keywords: SRAM PUF, Embedded
mermaid: false
sequence: false
flow: false
mathjax: true
mindmap: false
mindmap2: false
---

​	SRAM PUF 一直是 PUF 设备种类中，较为热门的一个方向，那么对于这一类设备来说如何对其或者说对其产生的密钥和随机数进行评价呢，以下几篇论文都对其进行了阐述。（续接上文-2）

## Determining the Quality Metrics for PUFs and Performance Evaluation of Two RO-PUFs

​	10th IEEE international NEWCAS conference，2012.	

​	物理不可克隆函数（PUF）是一种电路基元，根据制造过程中存在的不可控元件，产生芯片的特定签名。认证、密钥生成和IP保护是PUF电路的三个重要使用领域。除了不可克隆性之外，唯一性和稳健性是每个PUF应该提供的主要属性。尽管文献中提出了许多PUF类型，但这些特性或测试方法的标准和令人满意的性能评估指标还没有被提出。在这项工作中，为了对PUF进行公平和详细的性能评估，已经开发了一套完整的质量指标。其次，提出了一种测试方法，并在PUF评估中采用了置信区间和置信度概念，以保持结果的可靠性。我们在FPGA上实现了两个基于环形振荡器的PUF电路，并根据提出的质量指标详细评估了它们在不同环境条件下的性能。

​	创新点：尽管存在相当多的不同的PUF结构，并且在文献中提出了结果，但没有对鲁棒性和唯一性进行详细的评估。Hori等人和Maiti等人定义的质量指标，用直接的方法评估了鲁棒性和唯一性。这使得我们无法比较PUF并为特定的应用选择最佳的拟合结构。同样，也没有定义PUF电路的测试方法，每个PUF都用不同的参数集进行测试，这也阻碍了对电路的有意义的比较。

#### 唯一性

​	唯一性体现PUF的芯片间差异，在理想情况下，来自不同芯片的所有PUF输出应该是均匀分布和统计上独立的。

1. 汉明距离(U_QM1)

   ​	如果这组测量值在统计上是独立的，它们的汉明距离（HD）将是50%。

   ![image-20241021230320498](D:\Jobs\xiangxu05.github.io\_posts\2024-10-21-SRAM-PUF-Measurement-3.assets\image-20241021230320498.png)

   公式解释：

   - U_QM1 表示唯一性度量值。
   - k表示总的响应序列数。
   - n表示每个响应序列的比特长度。
   - HD(Ri,Rj)表示响应序列 Ri和 Rj之间的汉明距离。
   - 两层求和符号代表对所有可能的响应序列对（Ri和 Rj，其中 i<j）进行累加。

   计算方法：

   1. 计算汉明距离：汉明距离是两个相同长度的二进制序列之间不同位的个数。例如，序列 1010和 1001的汉明距离为 2（第2位和第4位不同）。

   2. 求平均值

      ：通过两两序列之间的汉明距离的平均值衡量唯一性。

      - 首先计算每一对响应序列之间的汉明距离 HD(Ri,Rj)。
      - 然后对所有序列对的汉明距离求和，除以响应序列总数对的数量 k(k−1)\2。
      - 最后除以比特长度 n，并乘以 100%，得到百分比形式的唯一性。

   ​	即使上述质量指标U_QM1给出了关于系统性能的信息，它也不能保证均匀分布，因为非均匀数据也可能产生50%的HD。如果将第一个质量指标作为唯一的性能参数，两个性能不同的PUF可以被评估为相同。

2. 汉明距离的高斯程度(U_QM2)

   ​	文章认为PUF数据的HD分布在理想状态下应该与高斯分布相关联，因此通过计算理想分布与实际分布之间的相关系数，结果越接近于1，分布就越高斯；因此电路在唯一性方面表现更好。

   ![image-20241021230850690](D:\Jobs\xiangxu05.github.io\_posts\2024-10-21-SRAM-PUF-Measurement-3.assets\image-20241021230850690.png)

   具体计算方法：

   1. **计算离散汉明距离 (DIS_HD)**：
      - 对PUF响应序列进行两两比较，计算汉明距离（HD），这给出了实际观测的汉明距离分布。
   2. **计算PUF响应的平均汉明距离**：
      - 计算所有响应序列之间的汉明距离的均值 Mn(HD_PUF)和标准差 σ，得到这组数据的分布特性。
   3. **拟合高斯分布**：
      - 基于 Mn(HD_PUF)和 σ参数，生成一个理论上的高斯分布，作为PUF响应序列的理想分布。
   4. **计算相关性**：
      - 使用相关性函数 Corr(x,y)，对离散汉明距离分布 DIS_HD和理论高斯分布 Gaus(Mn(HD_PUF),σ)进行相关性分析。
      - 相关性值反映了实际观测汉明距离分布和理论高斯分布的吻合程度。如果相关性较高，则说明PUF响应序列的实际汉明距离分布和理论高斯分布接近。

3. Gilbert-Varshamov  bound(U_QM3)

   ​	Gilbert-Varshamov bound（GVB）被用来确定PUF输出对穷举搜索攻击的安全性。这是通过计算均匀分布的输出集合中两个随机输出之间的最小汉明距离来实现的。这个约束也可以用来确定结构的唯一性。

   ​	在收集了一定数量的数据后，计算出它们之间的最小距离dHm，并通过（3）和（4）确定R'。理想的R是通过（5）使用测量次数M和PUF长度N来计算的。如(6)所示，R与理想R的比例也可以作为唯一性的质量指标U QM3。如果输出是均匀分布的，意味着唯一性是理想的，那么最小HD与GVB兼容，U_QM3趋于统一。否则，最小HD比约束状态差，U_QM3小于1。

   ![image-20241021231917786](D:\Jobs\xiangxu05.github.io\_posts\2024-10-21-SRAM-PUF-Measurement-3.assets\image-20241021231917786.png)

   ​	

4. 3

## 参考文献

[1] Kietzmann P, Schmidt T C, Wählisch M. Puf for the commons: Enhancing embedded security on the os level[J]. IEEE Transactions on Dependable and Secure Computing, 2023.
