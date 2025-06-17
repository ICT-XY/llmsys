---
title: >-
  Improving Automatic Parallel Training via Balanced  Memory Workload
  Optimization
date: 2025-06-17 17:31:41
tags: 
  - 自动化并行
  - 三维并行
  - 大模型
  - 并行
categories: paper-reading
---

# Improving Automatic Parallel Training via Balanced  Memory Workload Optimization



# 创新点：
- 并行策略空间比较大 （DP,SDP,TP, PP, CKPT）
- 不仅考虑自动化并行，还兼顾了当模型层不规整的时候，流水线切分时的耗时/内存负载均衡问题。


# 并行策略空间：
DP, SDP, PP, CKPT
注意，CKPT将搜索空间扩展了一倍。
因为CKPT在与其他并行方法相结合时，会产生额外的通信开销。比如与TP相结合，因为CKPT会引入额外的一个forward，进而会引入额外的激活值同步开销。因此，TP+ ckpt和单纯的TP是两种截然不同的策略，有着截然不同的通信、内存和计算开销。
e:\PaperNotes\Notes\Attach\Pasted image 20250617150730.png
![](</source/images/Pasted image 20250617150730.png>)
# 优化算法
## 问题形式化：
输入：一个模型M（L层），N个设备（可用内存E）。
目标：最大化吞吐量
返回：最高吞吐量对应的并行策略。

下图是一个基准流程，假设pipeline的切分策略已知：
![](</source/images/Pasted image 20250617170627.png>)
## 阶段内并行策略优化：
动态规划算法：
注：为了简化，
C为耗时，E为内存开销。
![](</source/images/Pasted image 20250617155234.png>)

## 阶段间切分算法：
- 核心理念：
	- 兼顾时间开销的均衡和内存开销的均衡
- 定义均衡指数如下：
  ![](</source/images/Pasted image 20250617164451.png>)
	- C为时间开销，O为内存开销，P为阶段数目。
	- 指数越大，开销越均衡，性能越理想。

- 实例1：
	- ![](</source/images/Pasted image 20250617165129.png>)
	- 只优化内存均衡：时间不均衡，吞吐量低。
	- 只优化耗时均衡：个别阶段内存开销极高，超出可用内存，切分策略失效。
- 方法：
	- 优化目标：
		- ![](</source/images/Pasted image 20250617170038.png>)
		- 其中pm是只优化内存的切分策略。
		- pt是只优化时间的切分策略。
		- 这个也很好理解，如果上式有一个不成立，那我们不如干脆用pt/pm算了。
	- 双目标优化流程：
    - ![](</source/images/Pasted image 20250617170652.png>)
		- 相对于基准流程，加了一个pipeline切分的迭代。
		- 迭代以pm为起点
		- 迭代方式：取当前耗时最长的一个stage，把它的边缘的层挪到相邻stage上去。
		- 对每个新的p‘，先进行如下验证：
			- 1、最大耗时不能超过之前的最大耗时。
			- 2、最大内存开销不能超过pt下的内存开销。
			- 3、内存开销小于可用内存。
# 系统实现
![](</source/images/Pasted image 20250617141420.png>)

## 评估器
- 内存：for the memory consumption, we use the shape of a tensor and its data type to calculate its memory.
- 计算耗时：batch size乘以per sample computation time，后者可以直接在单块GPU上profile获取。 反向直接用前向时间x2.
- 通信耗时：通信量除以带宽
- 还考虑了通信和计算折叠的影响。
- 对于使用了CKPT的，再加一个foward开销。

# 内存开销分析
![](</source/images/Pasted image 20250617143104.png>)
- 其中，
	- msi为model state，是参数相关的存储（参数、参数梯度、）
	- fk为第k层的前向激活值
	- bi为第i层的反向激活值。
- 总的来说，上述式子的意思是
	- 第i层的内存开销，等于从0~i层的前向激活值、加上本层的反向激活值，加上总的参数相关存储。
	- 求所有层的开销，取最大值。

## checkpoint技术对内存开销的影响：
对于使用了ckpt的层，
c(fi) = c(bndi), 并且c(bi) = c(inti)
其中，
- bnd为两层中间的向量，比如一层的输入向量或者一层的输出向量。
- int为一层的中间向量。

对于不使用ckpt的层，
c(fi) = c(bndi) + c(inti)
c(bi) = 0
- 也就是说，他认为，不使用ckpt的时候，反向传播不会产生新的中间向量。
  
> [!attention] 存疑
在前向传播或反向传播时，**是否**会产生一些中间向量，而这个中间向量不会被保存，只是计算所用。用完就丢弃，导致计算内存开销产生尖锐峰值。



