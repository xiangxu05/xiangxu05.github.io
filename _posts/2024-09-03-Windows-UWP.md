---
layout: post
title: 日常记录｜Windows UWP 应用设置代理
categories: [Daily Notes]
description: 遇到的一个问题是微软商店始终无法联网，这是可行的解决方法
keywords: Windows，Microsoft
mermaid: false
sequence: false
flow: false
mathjax: true
mindmap: false
mindmap2: false
---

​	因为日常需要在谷歌学术查阅一些文献以及上Github，因此梯子总是保持默认开启的状体。于是就发现一个问题是，Windows商城始终是无法联网的，翻了翻网上大家的记录，国内按理说是可以连上的，于是进行的相关搜索，最终发现了解决方法：[如何为 Windows 10 UWP 应用设置代理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/29989157)。

![image-20240903110744735](/images/posts/2024-09-03-Windows-UWP.assets\image-20240903110744735.png)

## 原理

​	UWP 是微软在 Windows 10 中引入的新概念，由于所有 UWP 应用均运行在被称为 App Container 的虚拟沙箱环境中，其安全性及纯净度远胜于传统的 EXE 应用。但 App Container 机制同时也阻止了网络流量发送到本机（即 loopback）， 使大部分网络抓包[调试工具](https://zhida.zhihu.com/search?q=调试工具&zhida_source=entity&is_preview=1)无法对 UWP 应用进行流量分析。同样的，该机制也阻止了 UWP 应用访问 localhost，即使你在系统设置中启用了代理，也无法令 UWP 应用访问本地[代理服务器](https://zhida.zhihu.com/search?q=代理服务器&zhida_source=entity&is_preview=1)，十分恼人。

​	而梯子的原理，也就是开启了一个本地代理服务器，本机网络代理规则内的出口流量都会经过这个服务器，而微软软件的一些目标地址在规则中是需要经过本地代理服务器的，这就是Windows商店一直都无法联网的原因，文中提到了以下方法进行操作： **Windows 10 自带了一款名为 CheckNetIsolation.exe 的命令行工具可以帮助我们将 UWP 及 Windows 8 Metro 应用添加到排除列表**。但是评论区中有一个更快捷的方法。

# 解决方法

​	使用cmd输入指令：

```cmd
FOR /F "tokens=11 delims=\" %p IN ('REG QUERY "HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\CurrentVersion\AppContainer\Mappings"') DO CheckNetIsolation.exe LoopbackExempt -a -p=%p
```

​	这段代码的主要作用是通过遍历注册表中应用程序的包SID，将它们添加到允许访问本地环回地址的豁免列表中。这通常用于调试和开发环境，允许UWP（通用Windows平台）应用程序访问本地资源。

**FOR /F "tokens=11 delims=" %p**:

- `FOR /F` 是一个用于遍历命令输出或文件中的每一行的命令。

- ```cmd
  "tokens=11 delims=\"
  ```

   指定了如何解析每一行的内容：

  - `delims=\"` 将反斜杠 (`\`) 设置为分隔符。
  - `tokens=11` 表示只取第11个分段内容赋值给变量 `%p`。

**IN ('REG QUERY ...')**:

- `REG QUERY` 命令用于查询注册表项的内容。
- `REG QUERY "HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\CurrentVersion\AppContainer\Mappings"` 具体查询了该路径下的所有注册表项。
- 该路径存储了Windows应用程序的AppContainer映射信息。

**DO CheckNetIsolation.exe LoopbackExempt -a -p=%p**:

- `DO` 后面跟的是当上述命令成功时执行的动作。

- ```cmd
  CheckNetIsolation.exe LoopbackExempt -a -p=%p
  ```

   用于将应用程序添加到网络环回隔离豁免列表中：

  - `LoopbackExempt` 选项用于管理允许哪些应用程序绕过网络隔离策略，允许访问本地环回地址（如 `127.0.0.1`）。
  - `-a` 代表“添加”操作。
  - `-p=%p` 指定了应用程序的包SID， `%p` 是从上面 `FOR` 循环中提取的值。

![image-20240903110752590](/images/posts/2024-09-03-Windows-UWP.assets\image-20240903110752590.png)
