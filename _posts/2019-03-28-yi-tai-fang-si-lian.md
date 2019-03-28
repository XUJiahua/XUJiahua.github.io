---
layout: post
title: "搭建一个以太坊私链"
modified: 2019-03-28 10:18:42 +0800
category: []
tags: [ethereum,blockchain]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---


### 假设 

已安装geth，安装参考 https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum

### 正式开始

*我的工作目录为`/tmp`*

#### 创建账号

*使用--datadir指定节点数据路径，我们会在单机上部署多个节点*
*datadir会存放keystore（用户账号）、区块数据*

```
mkdir node01 node02 node03
geth --datadir ./node01 account new
// Address: {ce8e6a5072d61847073b213105882ae5c36d2094}
geth --datadir ./node02 account new
// Address: {54bcb00e6f83040d29800b804258e5133721a4e3}
geth --datadir ./node03 account new
// Address: {48f7ac407ef8ddf417da219ee87681b31e950139}
```

`account new` 生成的账号在这里：

```
node01
└── keystore
    └── UTC--2019-03-28T03-09-55.670996000Z--ce8e6a5072d61847073b213105882ae5c36d2094
node02
└── keystore
    └── UTC--2019-03-28T03-10-12.852944000Z--54bcb00e6f83040d29800b804258e5133721a4e3
node03
└── keystore
    └── UTC--2019-03-28T03-10-48.322585000Z--48f7ac407ef8ddf417da219ee87681b31e950139
```

#### 定义 genesis.json (genesis state)

```
{
  "config": {
    "chainId": 2018,
    "homesteadBlock": 0,
    "eip155Block": 0,
    "eip158Block": 0
  },

  "alloc": {
    "ce8e6a5072d61847073b213105882ae5c36d2094": {
      "balance": "1000000000000000000"
    },
    "54bcb00e6f83040d29800b804258e5133721a4e3": {
      "balance": "2000000000000000000"
    },
    "48f7ac407ef8ddf417da219ee87681b31e950139": {
      "balance": "3000000000000000000"
    }
  },
  "coinbase": "0x0000000000000000000000000000000000000000",
  "difficulty": "0x400",
  "extraData": "",
  "gasLimit": "0x2fefd8",
  "nonce": "0x0000000000000042",
  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp": "0x00"
}
```

1. alloc：预先分配以太币给一些账户（address）
2. difficulty：挖矿难度
3. 其他都是默认值

#### 节点 (node) 初始化

```
geth --datadir ./node01 init genesis.json
geth --datadir ./node02 init genesis.json
geth --datadir ./node03 init genesis.json
```

#### bootnode

起一个独立的bootnode，通过这个节点作为信息中转站，其他geth node能互相找到对方：

```
bootnode --genkey=boot.key
bootnode --nodekey=boot.key

// enode://7d166ffd1b9c312712a843e682771b5a85bf047316f93af1076910d013c1f24d44ac618e67e8a6341aca75653a399bce05ee4874b458fc6474d6a151d8c0f726@127.0.0.1:0?discport=30301
```

#### 启动

P2P服务的`--port`的默认值是30303，分配其他端口号给其他两个节点。
geth默认打开的是IPC服务(unix socket)，如果要打开HTTP服务，需要考虑分配合理的端口号。

```
geth --datadir ./node01 --bootnodes="enode://7d166ffd1b9c312712a843e682771b5a85bf047316f93af1076910d013c1f24d44ac618e67e8a6341aca75653a399bce05ee4874b458fc6474d6a151d8c0f726@127.0.0.1:0?discport=30301"
geth --datadir ./node02 --port=30304 --bootnodes="enode://7d166ffd1b9c312712a843e682771b5a85bf047316f93af1076910d013c1f24d44ac618e67e8a6341aca75653a399bce05ee4874b458fc6474d6a151d8c0f726@127.0.0.1:0?discport=30301"
geth --datadir ./node03 --port=30305 --bootnodes="enode://7d166ffd1b9c312712a843e682771b5a85bf047316f93af1076910d013c1f24d44ac618e67e8a6341aca75653a399bce05ee4874b458fc6474d6a151d8c0f726@127.0.0.1:0?discport=30301"
```

#### 操作

进入console：

```
geth --datadir ./node01 attach
```

查看peer数：

```
> net.peerCount
2
```

查看余额：

```
> eth.getBalance(eth.accounts[0])
1000000000000000000
```

转账：

```
> eth.sendTransaction({from:eth.accounts[0], to:"0x54bcb00e6f83040d29800b804258e5133721a4e3", value:100000})
"0xfdf1ebd68f68e972a4ea55a2246dc61114bee5b19e2f5928e461e958e6e4599b"
> eth.getBalance(eth.accounts[0])
1000000000000000000
```

如果报错，需要先unlock账号：

```
> personal.unlockAccount(eth.accounts[0])
Unlock account 0xce8e6a5072d61847073b213105882ae5c36d2094
Passphrase:
true
```

查看余额发现转账没有发生。因为还需要挖矿组成block。

```
> miner.start(1)
> miner.stop()
```

挖矿有收益，可看到node01的account收到了很多挖矿奖励。

#### cleanup

```
rm -rf node01 node02 node03
```

### 参考

* https://github.com/ethereum/go-ethereum#operating-a-private-network
* https://blog.ippon.tech/a-journey-into-blockchain-private-network-with-ethereum/

