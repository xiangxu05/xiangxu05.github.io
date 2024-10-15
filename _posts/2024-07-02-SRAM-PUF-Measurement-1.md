---
layout: post
title: 论文｜主流SRAM PUF的评价方法-1
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

​	SRAM PUF 一直是 PUF 设备种类中，较为热门的一个方向，那么对于这一类设备来说如何对其或者说对其产生的密钥和随机数进行评价呢，以下几篇论文都对其进行了阐述。

## 单芯片微型传感器和SRAM PUF的可行性研究

​	单芯片微型传感器和SRAM PUF的可行性研究<sup>[1]</sup>,是一篇在IEEE物联网无晶体/无晶体无线电和基于系统的研究研讨会上提出的文章。探索微小型的无线传感器节点（无晶振，集成传感、计算和通信功能）是否能作为SRAM PUF使用，总结了一些衡量SRAM PUF是否合格的评价指标。其认为要作为SRAM PUF使用，需要满足以下几个要求：随机性、可靠性、不可预测性、独立性。

> ​	Physically unclonable functions (PUFs) are used as low-cost cryptographic primitives that extract key material from manufacturing variabilities of a device. All microcontrollers have on-chip SRAM; the power-up state of SRAM cells provides one way of obtaining a random output. However, not every SRAM can be used as a PUF. SRAM PUFs need to fulfill certain requirements: randomness, reliability, unpredictability

1. 随机性

   ​	文章认为对随机性评估，SRAM PUF的二进制输出应该实现平衡和无偏的比特分布。因此使用了两个指标来衡量PUF输出的非随机性和依赖关系：汉明权重（Fractional Hamming Weight）、比特间相关性（Correlation  Between  Bits）。

   ​	**汉明权重**：SRAM PUF的偏置为0或1。它构成了SRAM的上电值的平均值，用于评估SRAM PUF中比特的分布和平衡。如果SRAM PUF是完全无偏的，其平均值应该是0.5。n个上电值的平均值x<sub>i</sub>可以如下表示：
   $$
   \overline{x} = \frac{1}{n} \sum_{i=1}^{n} x_i \tag{1}
   $$
   ​	**比特相关性**：目的是主要评估生成的比特序列的独立性和随机性水平。如果输出位是高度相关的，它有可能引入可能被对手利用的模式或可预测性。距离（滞后）为j的n位之间的相关性可以用二进制序列的*无偏自相关估计器*来表示，定义为：
   $$
   r_j = \frac{1}{n - j} \sum_{i=1}^{n-j} x_i x_{i+j} \tag{2}
   $$
   ​	对于0 ≤ j ≤ (n - 1)。为了使零成为中性元素，将零位值替换为-1。因此，\(r<sub>j</sub> = 1\) 和 \(r<sub>j</sub> = -1\) 意味着位是完全相关的，\(r<sub>j</sub> = 0\) 意味着距离为 \(k\) 的位是完全不相关的，在这种情况下，\(n - j\) 应该是偶数。请注意，自动相关公式的实现可能会根据所使用库的具体实现和参数配置而略有变化。

2. 可靠性

   ​	可靠性是PUF的另一个必须研究的重要属性。它衡量的是当一些环境条件发生变化时的稳定性或一致性，如环境温度、电源电压等。由于环境变化导致的SRAM启动值的变化受到关键晶体管参数的影响，如门限电压、漏电流、延迟和其他。当PUF被用来提取密钥时，由于两个主要原因，尽量减少这些变化的影响是至关重要的。首先，该设备必须表现出固有的弹性，因为它可以自然地暴露在环境变化中。此外，攻击者不应该仅仅通过操纵温度来泄露设备的信息。因此文章确定了两个不同的测试来评估PUF的可靠性。

   ​	**片内汉明距离(Intra-Chip Hamming Distance)**：该措施评估了在有或没有环境变化的情况下从同一芯片再次重新生成时发生变化的PUF输出位的数量。它也被称为错误率，表明PUF输出的可靠性。当PUF被用来生成秘钥时，芯片内的哈明距离最好是0%。然而，在实践中，10<sup>−6</sup>的错误率被认为是足够的。片内汉明距离可以用相对于参考向量的汉明距离来测量。第一个读出模式被用作参考，同一芯片的其他测量值使用分数汉明距离与该参考进行比较。

   ​	**稳定性(Stability)**:SRAM PUF的稳定性可以从稳定的SRAM单元数和单元总数之间的分数来评估。当一个SRAM单元在大量的上电过程中总是被供电到0或1状态时，它被认为是稳定的。在一个更加量化的方法中，SRAM单元的稳定性是通过引入经验性的单概率项来评估的。SRAM内特定单元i的单概率（p<sub>i</sub>）是指该单元的响应值（X<sub>i</sub>）在多个上电实例中的概率。形式上，它的定义如下:
   $$
   p_i := \Pr(X_i = 1) \tag{3}
   $$

3. 不可预测性

   ​	不可预测性比随机性有更广泛的意义，一个不可预测的序列可能是，也可能不是真正的随机性。这一特性指的是无法根据以前的观察来预见SRAM PUF的未来结果，当SRAM PUF被用来生成随机数时，它尤其重要。一个SRAM的PUF输出会受到噪声的影响，因此在一个SRAM上对启动值的两次测量会呈现出几个不同的比特，即使在相同的外部条件下。为了利用这种现象作为随机性的来源，在测量之间应该有足够数量的比特变化，而且，改变的比特应该是难以预测的。为了评估不可预测性，使用了熵度量。PUF熵的下限，称为最小熵，量化了描述随机性来源所需的最少信息量。

   ​	**片内最小熵(Intra-Chip Min. Entropy)**:该指标用于评估同一芯片上多个SRAM PUF测量的芯片内变化。不稳定的单元对噪声敏感，因此会对这种熵有贡献，因此该指标也被称为噪声熵。在安全性方面，具有高最小熵的SRAM PUF对于TRNG是理想的，确保生成的数字难以猜测或复制。给定一个具有概率 \(p<sub>0</sub>\) 和 \(p<sub>1</sub>\) 产生‘0’和‘1’的二进制源，该二进制源的最小熵为：
   $$
   H_{min} = -\log_2(\max(p_0, p_1)) \tag{4}
   $$
   ​	假设SRAM PUF的所有位是独立的，则每个单元位置可以看作一个独立的二进制源。为了估计一个比特单元的噪声最小熵，我们使用在公式 (3) 中定义的单概率 \(p<sub>i</sub>\)，并在同一芯片上通过多次上电SRAM PUF获得。理想概率 \(p<sub>i</sub> = 0.5\)（非常不稳定的单元）使最小熵最大化为 \(H<sub>min, noise</sub> = 1\)。因此，n 比特上单个SRAM PUF噪声的平均最小熵为：
   $$
   (H_{min, noise})_{avg} = \frac{1}{n} \sum_{i=1}^{n} -\log_2(\max(p_i, 1 - p_i)) \tag{5}
   $$

4. 独立性

   ​	由于固有的制造变异性，每个芯片应该是独一无二的，这意味着两个芯片的PUF响应接近对方的概率可以忽略不计。这种独特性对于加密密钥生成、安全设备认证和防伪措施等应用至关重要，因为它增强了安全性，防止了未经授权的访问，并促进了对单个芯片的识别和验证。PUF之间的不相似性可以用汉明距离和定理来评估，如下所示：

   ​	**片内汉明距离(Intra-Chip Hamming Distance)**：假定对PUF进行了n次取值，X为每次取出的值，可以以第一次的结果为标准，之后的二进制序列与第一次取值进行按位异或，并累加计算不同的位数，并将结果求平均。
   $$
   d_n = \frac{1}{n}\sum_{i=1}^{n} (\sum_{j=1}^{n}X_{1j} \oplus X_{ij}) \tag{6}
   $$
   ​	**片内最小熵(Intra-Chip Min. Entropy)**:
   $$
   H_{min} = -\log_2(\max(p_0, p_1)) \tag{4}
   $$

## 基于CNTFET的物联网设备的超低功率SRAM-PUF

​	这篇文章使用了一种带有碳纳米管场效应晶体管（CNTFETs）的绝热逻辑，并仿照SRAM单元结构来实现SRAM PUF，其认为引入了该种材料作为SRAM单元在嵌入式设备中可以大大降低能耗<sup>[2]</sup>，是一篇在国际微电子学会提出的文章。文中衡量这一设备的性能，考虑了三个方面：功率耗散、独立性、可靠性。功率方面这边不考虑，只介绍另外两种。

1. 独立性

   ​	PUF使用相同的挑战C将一个特定的集成电路（IC）与其他具有相同结构的电路区分开来的能力是通过其唯一性来衡量的。当两个芯片，i和j（其中i≠j），收到相同的挑战C，它们的响应被表示为R<sub>i</sub>和R<sub>j</sub>，平均设备间唯一性可以表示如下:
   $$
   \text{Uniqueness} = \frac{2}{d(d + 1)} \sum_{i=1}^{d-1} \sum_{j=i+1}^{d} \frac{HD(R_i, R_j)}{n} \times 100\% \tag{7}
   $$
   ​	其中，d表示被比较的设备（ICs）的数量，\(n\) 表示PUF响应的位长度。\(HD(R<sub>i</sub>, R<sub>j</sub>)\) 表示两个不同PUF响应（R<sub>i</sub> 和 R<sub>j</sub>）之间的汉明距离。唯一性的理想值是50%，这表明每个PUF响应与其他响应完全不同，从而在各个IC之间具有完美的区分能力。更高的唯一性百分比反映了PUF在区分不同设备方面的更强能力，从而增强了其在认证和防伪等应用中的有效性和安全性。

   > ​	Where d represents the number of devices (ICs) being compared. n denotes the bit length of the PUF responses. HD (Ri, Rj) signifies the Hamming Distance between the responses of the two distinct PUFs (Ri and Rj) [7]. 
   >
   > ​	The ideal value for uniqueness is 50%, indicating that each PUF response is entirely different from the others, resulting in perfect discrimination capability among individual ICs. A higher uniqueness percentage reflects a stronger ability of the PUF to distinguish between different devices, adding to its effectiveness and security in applications such as authentication and anti-counterfeiting measures.

2. 可靠性

   ​	PUF设计有望表现出在遇到相同的挑战位C时，即使在存在不同的环境条件（如电源电压和温度）的情况下，也能持续重现相同的响应位R的能力。PUF的可靠性可以通过计算设备内的平均汉明距离（HD）来量化，使用以下公式：
   $$
   HD_{\text{intra}} = \frac{1}{d} \sum_{t=1}^{d} \frac{HD(R_i, R_i', t)}{n} \times 100\% \tag{8}
   $$
   ​	其中，R<sub>i</sub>表示在标称工作条件下测量的芯片 i的响应。R<sub>i', t</sub>表示在不同供电电压和温度条件下提取的响应 R<sub>i</sub>的第 t个样本。n表示PUF响应的位长度（在本研究中，n = 4。d 是用于分析的设备（芯片）数量（本研究中，d = 30）。此外，我们分析中考虑的温度范围从-40°C到100°C。

   $$
   \text{Reliability} = 100\% - HD_{\text{intra}} \tag{9}
   $$

​	**这边与前文不同的计算方式，是考虑了同种实现方式的多个不同设备来计算独立性，因此引入了一个d。而为什么没有随机性的计算，可能是这篇文章中只做出了4位的最基本单元，计算随机性没什么意义。**

![image-20240702162659183](/images/posts/2024-07-02-SRAM-PUF-Measurement.assets/image-20240702162659183.png)

## 参考文献

[1] Faour S, Korecic B, Vučinić M, et al. Single-Chip Motes and SRAM PUF: Feasibility Study[C]//2024 IEEE Workshop on Crystal-Free/-Less Radio and System-Based Research for IoT (CrystalFreeIoT). 2024.

[2]Shafiei A, Monajati M. Ultra-Low Power SRAM-PUF for IoT Devices Based on CNTFETs[C]//2023 5th Iranian International Conference on Microelectronics (IICM). IEEE, 2023: 86-90.
