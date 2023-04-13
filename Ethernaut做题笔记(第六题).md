# Ethernaut做题笔记(第六题)

## 1.题目要求

这一关的目标是攻破下面这个基础 token 合约

你最开始有20个 token, 如果你通过某种方法可以增加你手中的 token 数量,你就可以通过这一关,当然越多越好

 这可能有帮助:

- 什么是 odometer?



## 2.题目代码

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```



## 3.合约分析

此处的映射balance代表了我们拥有的token

```solidity
mapping(address => uint) balances;
  uint public totalSupply;
```



然后通过构造函数初始化了owner的balance，但不知道是多少

```solidity
 constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }
```



下面的transfer函数的功能为转账操作，最下面的balanceOf函数功能为查询当前账户余额。
通过粗略的一遍功能查看之后我们重点来看此处的transfer()函数

```solidity
function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }
```

在该函数中最为关键第一处就是"require"校验，此处可以通过“整数下溢”来绕过检查，同时这里的balances和value都是无符号整数，所以无论如何他们相减之后值依旧大于0（在相等的条件下为0）。
那么在当前题目条件下(题目中token初始化为20)，所以当转21的时候则会发生下溢，导致数值变大其数值为2^256 - 1



## 4.攻击流程

本题参考题解：https://cyanwingsbird.blog/solidity/ethernaut/5-token/

编译合约Token_test.sol

![ig6psP.png](https://i.328888.xyz/2023/04/13/ig6psP.png)



部署合约，ENVIRONMENT填入metamask地址，Deploy填入F12中的实例地址

![ig6XyX.png](https://i.328888.xyz/2023/04/13/ig6XyX.png)



在Remix IDE中，呼叫claim函数，metamask完成两次合约交互。最后提交该实例

![ig6gIt.png](https://i.328888.xyz/2023/04/13/ig6gIt.png)





![ig6xGJ.png](https://i.328888.xyz/2023/04/13/ig6xGJ.png)

