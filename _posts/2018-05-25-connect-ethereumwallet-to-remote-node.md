---
layout: post
title: "EthereumWallet 连接远程节点"
tags:
    - ethereum
    - blockchain
---

EthereumWallet/Mist 是一个 GUI 应用，自带了 geth 客户端，所以，当 EthereumWallet
启动的时候，实际上是会启动本地的 geth 客户端，然后 EthereumWallet 会连接到本地的
geth 客户端。

但是，很多时候，开发工作实际上是在远程开发机上完成的，因为开发机性能更强，磁盘容量更大，
网络也更好。

所以，只在本地运行 EthereumWallet GUI，让它连接到远程的 geth 客户端就比较好了。

### 远程客户端

在远程客户端机器上，启动 geth，开启 rpc，命令如下：

```
$ geth --rinkeby --datadir . --rpc --rpcaddr 0.0.0.0 --rpcapi "db,eth,net,web3,personal"
```

上面的命令将节点连接到 rinkeby 测试网络，rinkeby 测试网络是 PoA
测试网络，不能挖矿，只有通过认证的节点才能挖矿。

如果想要获得 rinkeby 测试网络上的 Ether，可以搜索一下，这里不再介绍。

### 从 EthereumWallet 连接远程节点

启动 EthereumWallet 的时候指定 `--rpc` 参数，如下：

```
$ ethereumwallet --rpc http://<remote-ip>:<remote-port>
```

上面的 remote-port 模式是 8545，remote-ip 则是远程节点的 IP。

启动时会提示如下安全警告，确定即可。

![warning](/assets/images/remote-note-warning.png)

然后这样就连接上了。
