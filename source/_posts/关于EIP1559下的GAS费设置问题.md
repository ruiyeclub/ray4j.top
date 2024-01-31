---
title: 关于EIP1559下的GAS费设置问题
date: 2024-01-31 15:16:55
tags:
---

> **EIP-1559**是一项以太坊改进提案，旨在优化以太坊区块链的费用市场系统。 EIP-1559 引入了一个新的收费系统，其中包含三个关键要素：**基本费用(base fee)**、**优先小费(priority fee)**和**费用上限(max fee)**。



## 一、EIP-1559特点

- 基础费用:eip1559引入了一种新的收费机制，即每个区块都有一个由协议根据区块需求动态调整的基础费用(base fee)，表示在该区块中进行交易时需要支付的最低手续费。基础费用的目标是让每个区块的使用率维持在50%，从而避免过度拥堵或浪费。基础费用以ETH代币支付，并且被销毁(burn)，从而减少ETH的流通供应。
- 优先费用:除了基础费用外，用户还可以为自己的交易设置一个优先费用(priority fee)，表示他们愿意额外支付给矿工/验证者的激励，以便让自己的交易更快被确认。优先费用也以ETH代币支付，并且被矿工/验证者保留。
- 费用上限:用户还可以为自己的交易设置一个费用上限(max fee)，表示他们愿意为一笔交易支付的最高手续费。费用上限包括基础费用和优先费用，当交易被确认时，用户会收到一笔退款，由实际支付金额和费用上限之间的差额组成。这个机制确保了用户永远不会超出他们所设定的预算。
- 双重交易类型:eip1559兼容两种交易类型，即新型交易类型和传统交易类型。新型交易类型遵循eip1559的规则，即指定基础费用、优先费用和费用上限。传统交易类型遵循原有的规则，即指定一个固定的出价(gasprice)。为了让两种交易类型能够共存，传统交易类型会被转换成新型交易类型，即将出价作为费用上限，并将出价减去基础费用作为优先费用。 



## 二、正确设置GAS

**GAS fee** = **Base fee**(完成操作需要付的基本费用) + **Priority fee**(小费：小费越高，越容易抢到)

总费用 = (区块基础费 + 实际优先费) * Gas使用数量

也就是总费用 = (base fee + priority fee) * gas limit(用户允许这笔交易最大可消耗的 Gas 数量)

![image-20240131135302973](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240131135302973.png)

待设置：

**Max Priority Fee**：你最多愿意付给矿工的优先费(小费)，单位为GWEI
**Max Fee**：基本费用 + 优先费

实际：**Priority Fee = Min（Max Priority Fee, Max Fee - Base Fee）**



## 三、设置技巧 

如果我们要给100GWEI的小费，我们需要设置：

1.Max Priority Fee(GWEI) = 100

2.Max Fee(GWEI) = (Base Fee * 2)  + Max Priority Fee(100)

注意：base fee代表基础费用会有波动，且多余base fee会返还所以*2

GAS参考网站：https://www.blocknative.com/gas-estimator

![image-20240131130128416](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240131130128416.png)

假如我们想要99%的打包成功率，可以设置Max Priority Fee为0.1，设置max fee为26。



参考文章：[MetaMask钱包已支持EIP-1559，一次了解调整优先费、进阶设置](https://www.wwsww.cn/qianbao/9482.html)
参考文章：[Web3.0学习树|3.区块链通识 Ⅲ](https://mirror.xyz/looplove.eth/SOhMBCF_Z4vIWFHu4r8fpdBaLMhRTBCvNj6i5d4pNMU)

内容来源：[油管](https://www.youtube.com/watch?v=Qpq00RGT5Bc)
