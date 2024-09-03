---
layout: post
title: 论文｜ECDSA原理及引入SRAM PUF方法
categories: [Coding Theory]
description: 对ECDSA方法做了一个简单的介绍，并介绍了如何使用SRAM PUF来进行签名
keywords: SRAM PUF, ECDSA
mermaid: false
sequence: false
flow: false
mathjax: true
mindmap: false
mindmap2: false
---

## 一、基本概念

文章中部分基本概念，来自于文献：Daniel R. L. Brown, Elliptic Curve Cryptography Standards for Efficient Cryptography, May 21, 2009 Version 2.0。

![image-20240903112355813](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112355813.png)

![image-20240903112417264](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112417264.png)

![image-20240903112554107](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112554107.png)

![image-20240903112559289](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112559289.png)

![image-20240903112604185](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112604185.png)

![image-20240903112607546](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112607546.png)

![image-20240903112612516](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112612516.png)

![image-20240903112615914](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112615914.png)

![image-20240903112618766](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112618766.png)

![image-20240903112623861](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112623861.png)

# 二、ECDSA流程

![image-20240903112627893](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112627893.png)

![image-20240903112632134](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112632134.png)

![image-20240903112635107](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112635107.png)

![image-20240903112637750](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112637750.png)

![image-20240903112643686](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112643686.png)

![image-20240903112646536](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112646536.png)



![image-20240903112649408](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112649408.png)

![image-20240903112655307](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112655307.png)



![image-20240903112700557](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112700557.png)

# 三、引入SRAM PUF

![image-20240903112844522](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112844522.png)

![image-20240903112708292](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112708292.png)

# 四、ECDSA结合RM纠错方案

![image-20240903112911426](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112911426.png)

![image-20240903112915987](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112915987.png)

![image-20240903112918848](/images/posts/2024-09-03-ECDSA-With-SRAM-PUF.assets\image-20240903112918848.png)
