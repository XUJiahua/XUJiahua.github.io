---
layout: post
title: "使用truffle开发智能合约"
modified: 2019-03-29 15:37:27 +0800
category: []
tags: []
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

remix很好用，存在什么问题：

1. 必须连入互联网
1. 单元测试不方便
1. 代码版本管理不方便

这篇文章，通过一个简单示例，演示一下truffle常见用法。

## 准备工作

### 安装命令行工具

```
npm install -g truffle
```

参考：https://truffleframework.com/docs/truffle/getting-started/installation

### 提供一个以太坊节点

#### Ganache

mock以太坊节点，有可视化界面。非常方便开发使用。

参考：https://truffleframework.com/ganache

#### 私有链

略。记得开启HTTP服务，truffle需要HTTP RPC服务。比如：

```
geth --datadir ./datadir \
-rpc -rpcapi "db,personal,eth,net,web3,debug" -rpccorsdomain="*" -rpcaddr="localhost" -rpcport 8545 \
--ws --wsapi "db,personal,eth,net,web3,debug" --wsorigins="*"  --wsaddr="localhost" --wsport 8546 \
 console
```

## 从零开始一个truffle项目

主要是仿照`truffle unbox metacoin`生成的代码。

#### 新建项目

```
mkdir token0329
truffle init
```

#### truffle-config.js指定下网络，这里我使用Ganache暴露的服务

```
  networks: {
    // Useful for testing. The `development` name is special - truffle uses it by default
    // if it's defined here and no other network is specified at the command line.
    // You should run a client (like ganache-cli, geth or parity) in a separate terminal
    // tab if you use this network and you must also set the `host`, `port` and `network_id`
    // options below to some value.
    //
    development: {
      host: "127.0.0.1", // Localhost (default: none)
      port: 7545, // Standard Ethereum port (default: none)
      network_id: "*" // Any network (default: none)
    }
```

#### 主体部分，编写合约

主要功能就是set赋值, get取值。

contracts/SimpleStorage.sol

```
pragma solidity >=0.4.21 <0.6.0;

contract SimpleStorage {
    uint storedData;

    function set(uint x) public {
        storedData = x;
    }

    function get() public view returns (uint){
        return storedData;
    }
}
```

#### 准备部署脚本
*必须先准备好，不然`truffle test`不会成功的*

migrations/2_deploy_contracts.js

```
const SimpleStorage = artifacts.require("SimpleStorage");

module.exports = function(deployer) {
  deployer.deploy(SimpleStorage);
};
```

#### 使用JavaScript写测试

test/simplestorage.js

```
const SimpleStorage = artifacts.require("SimpleStorage");

contract("SimpleStorage", accounts => {
  it("should set 42 correctly", async () => {
    const simpleStorageInstance = await SimpleStorage.deployed();
    await simpleStorageInstance.set(42);
    const v = await simpleStorageInstance.get.call();
    assert.equal(v, 42, "v is not 42");
  });
});
```

测试通过：

```
> truffle test

Using network 'development'.


Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



  Contract: SimpleStorage
    ✓ should set 42 correctly (77ms)


  1 passing (119ms)
```

#### 部署合约

*注：`truffle migrate` = `truffle deploy`*

```
truffle migrate
```

#### 执行合约

```
> truffle console
truffle(development)> let simpleStorageInstance = await SimpleStorage.deployed()
undefined
truffle(development)> await simpleStorageInstance.set(42)
{ tx:
   '0x575fecbd07bf8099892b477c4b8957f44ec87ae3f47fdd3c4a7f6bd8ea2e6b09',
  receipt:
   { transactionHash:
      '0x575fecbd07bf8099892b477c4b8957f44ec87ae3f47fdd3c4a7f6bd8ea2e6b09',
     transactionIndex: 0,
     blockHash:
      '0x661af44a6ab64d410965bb7aa9e6ebcc30973382de207a4fd5fc8612a971e424',
     blockNumber: 23,
     from: '0x0461e09f1e177d34b89288b80131b2e1ab894072',
     to: '0x69b25a01d3241c5c20b0b18c5905ac49abcf951a',
     gasUsed: 41695,
     cumulativeGasUsed: 41695,
     contractAddress: null,
     logs: [],
     status: true,
     logsBloom:
      '0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
     v: '0x1c',
     r:
      '0xbd439a44b14914c606317ad9122be420c2af656fea83e12162c15f95e53ab0a9',
     s:
      '0x26bf5032fe26abd8d342fcec8e3c406c80475ce9eaf4404b416b83930e727b6a',
     rawLogs: [] },
  logs: [] }
truffle(development)> (await simpleStorageInstance.get.call()).toNumber()
42
```

### 参考

* https://truffleframework.com/docs/truffle/quickstart


