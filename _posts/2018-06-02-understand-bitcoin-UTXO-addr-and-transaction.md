---
layout: post
title: 理解比特币的 UTXO、地址和交易
tags:
  - blockchain
  - bitcoin
---

UTXO（Unspent Transaction Output）在比特币中类似于以太坊中的账户模型。
在比特币种，一笔交易的每一条输入和输出实际上都是 UTXO，输入 UTXO 就是以前交易剩下的，
更准确的说是以前交易的输出 UTXO。

除了 coinbase 交易（挖矿奖励）没有输入 UTXO 之外，其它交易都有输入和输出，都可以为多个。

下图来自[《白话区块链》一书](https://www.amazon.cn/dp/B077NZVFJ5/ref=tmm_kin_swatch_0?_encoding=UTF8&qid=1527937615&sr=8-1)

![transactions](/assets/images/btc/bitcoin-transaction.png)

上图中有 3 笔交易：

  1. 第一笔交易来自挖矿，没有输入，输出为 Alice 地址，可以简单理解为一个 UTXO
  2. 第二笔交易是 Alice 转账给 Bob，输入是 Alice 的地址，输出为 Bob 地址和 Alice 地址，因为有余额
  3. 第三笔交易是 Bob 转账给 Lily，和上一笔交易类似

联系到现实中的使用，用户可以创建任意多的地址，可以通过地址接收转账，或者转账给他人。
那么，地址和 UTXO 是什么关系呢？可以看到，地址是可以反复使用的，每一次使用，要么作为输入，要么作为输出，
如果是作为输入，未使用的余额会自动输出到该地址，也就是找零的过程。

每一次交易的输入输出都是 UTXO，输出产生 UTXO，输入使用 UTXO。

例如：Alice 有一个地址，分别有 Bob 和 Lily 转账给 Alice 的这个地址，转账之后，Alice 实际上就有 2
个未使用的 UTXO 可以使用，而非一个，比特币永远不会合并一个地址里的 UTXO，每一个 UTXO 都是可以追踪的，
知道从矿工挖出的 coinbase 交易。

虽然，我们在使用钱包或者交易所的时候，在地址下看到的该地址的所有余额。

那么，看了上面的示例图，是否会觉得所有的交易都会连成一条链呢？

我们再看下比特币白皮书中的交易是怎样链接起来的。

> We define an electronic coin as a chain of digital signatures.
> Each owner transfers the coin to the next by digitally signing a hash of
> the previous transaction and the public key of the next owner and adding
> these to the end of the coin. A payee can verify the signatures to verify the chain of ownership.

![transaction chain](/assets/images/btc/bitcoin-transaction-chain.png)

所以，看了白皮书中的图之后，进一步会觉得所有的交易都会连成一条链。

但是仔细一想就会觉得不对，比特币的产生是每个块都会产生，产生比特币的 coinbase 交易，
显然是没有上一笔交易的，所以，很显然每个区块链的第一笔交易，即 coinbase 交易，
会是一条交易链的一个头（只所以说是一个头，因为一笔交易中可能有多个输入）。

下图取自比特币白皮书

![multi input](/assets/images/btc/bitcoin-multi-input.png)

看了下图或许更清晰一点：

![tx chains](/assets/images/btc/bitcoin-transaction-chains.png)

下面我们来看看比特币的地址
  1. 比特币地址是通过公钥处理而来的，并且是不可逆的，所以，不能通过地址提取公钥
  2. 地址是公开的，谁都可以查到地址上有多少比特币
  3. 但是，只有拥有生成地址的公钥对应的私钥的人才能使用地址中的比特币

然后，再来真真切切的看一个交易的内容：

![tx content](/assets/images/btc/bitcoin-transaction-content.png)

input，output 中可以有多个输入或者输出，上面的例子中只有一个，previous tx 表示输入 UTXO 是从哪比交易中产生的，
并且产生的序号是多少（Index），所以，这样子就定位了一个以前的输出 UTXO 作为这次的输入。

scriptSig 中包含两部分内容，对交易中某些字段的签名以及他的公钥，
签名用来验证消费 UTXO 的人真正有对应的私钥；公钥用来验证这个输入 UTXO 的地址是否属于这个公钥。

这里有必要再解释一下：你需要花费一个
UTXO，那么需要提供签名和公钥，验证你对 UTXO 有所有权的过程如下：

  1. 通过公钥计算出地址，然后和 UTXO 的地址想比，确保一致，这样证明你提供确实拥有该地址
  2. 通过公钥来验证签名，如果通过，则证明你拥有对应的私钥

这样，通过验证之后，就能证明你对该 UTXO 的所有权了，就能花费该 UTXO 了。

在 output 中，则对应每个输出 UTXO，包含输出的比特币量，以及 scriptPubKey，即收款人的地址，
这里不要以为是收款人的公钥，收款人的公钥发送人是不知道的。

总结下来，如果要花费某个地址中的余额，实际上可能会花费多个 UTXO，而这些细节对于用来说都是隐藏的。
