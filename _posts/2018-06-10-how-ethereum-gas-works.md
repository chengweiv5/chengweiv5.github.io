---
layout: post
title: 理解以太坊的 Gas 相关概念
tags:
  - ethereum
  - blockchain
---

在比特币中，交易手续费市价是一个动态变化的过程，矿工总是会先打包手续费高的交易；
在以太坊中，也有类似的机制，但是，由于以太坊具有智能合约的功能，所以，以太坊中的交易和比特币的有所不同。

更准确的说，以太坊中称之为消息，而创建执行合约，调用智能合约也通过发送消息来完成。
由于引入了智能合约，为了避免恶意攻击（例如：无限循环），所以，以太坊对消息的存储和执行都会收取手续费。

而手续费在以太坊中由 Gas 和 Gas Price 的乘积构成，Gas 简单的说就是以太坊中执行或者存储任何消息需要消耗的单位，
而 Gas Price 则是对 Gas 的定价。

同样，以太坊矿工也会优先处理高价的消息，但是，和比特币不一样的是，以太坊没有固定区块大小，而是由 Block Gas Limit
来决定一个区块能够打包的交易。而 Block Gas Limit 则会根据全网的状态，动态调节大小，主要考量两个方面：

  - 避免全网拥塞
  - 调节手续费/Gas Price

对于用户来说，用户的操作或者说发送的消息决定了消耗的 Gas，Gas Price 则决定了被确认的速度，用户还可以指定一个 Gas Limit，
以便确保不会消费过多的 Gas。

通常来说，用户并不需要关心 Gas，Gas Price 和 Gas Limit，以太坊客户端会自动计算好。
全网的 Gas Price 等信息可以在 https://ethgasstation.info/ 查看，例如：

![gasstation](/assets/images/ethereum/gasstation.png)

在上图中：

  - Std Cost for Transfer，转帐的手续费（美元单位），因为转帐这个类型消耗的 gas 是固定的（从实际少量观察，以及猜测和实际转帐的数量无关，因为存储 ether 的数据类型是固定的）
  - Gas Price Std (Gwei)，标准的 gas price，1 min 左右的确认时间
  - SafeLow Cost for Transfer，较低的手续费（美元单位）
  - Gas Price SafeLow (Gwei)，较低的 gas price

拖动 Adjust confirmation time 这个 bar 可以以不同的 gas price 来估计交易确认的时间以及需要花费的手续费，
可以看到，gas price 为 8 Gwei 之后，确认时间就不再减少了，因为通常来说，至少 3 个块确认后才比较安全，而以太坊平均出块时间为 15s 左右。

前面提到过，gas price 是动态变化的，矿工总是选择价格高的交易先处理，但是由于有动态调整机制，以太坊不存在比特币的扩容问题。
