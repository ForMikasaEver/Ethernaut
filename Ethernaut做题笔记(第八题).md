# Ethernaut做题笔记(第八题)



## 1.题目要求

有些合约就是拒绝你的付款,就是这么任性 `¯\_(ツ)_/¯`

这一关的目标是使合约的余额大于0

 这可能有帮助:

- Fallback 方法
- 有时候攻击一个合约最好的方法是使用另一个合约.
- 阅读上方的帮助页面, ["Beyond the console"](https://ethernaut.openzeppelin.com/help) 部分



## 2.题目代码

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Force {/*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/}
```



## 3.合约分析

经过查看资料，发现在以太坊里我们是可以强制给一个合约发送eth的，不管它要不要它都得收下，这是通过selfdestruct函数来实现的，如它的名字所显示的，这是一个自毁函数，当你调用它的时候，它会使该合约无效化并删除该地址的字节码，然后它会把合约里剩余的资金发送给参数所指定的地址，比较特殊的是这笔资金的发送将无视合约的fallback函数，因为我们之前也提到了当合约直接收到一笔不知如何处理的eth时会触发fallback函数，然而selfdestruct的发送将无视这一点，这里确实是比较有趣了。
那么接下来就非常简单了，我们只需要创建一个合约并存点eth进去然后调用selfdestruct将合约里的eth发送给我们的目标合约就行了。



## 4.攻击流程

先获取一个新实例，再查看合约地址

![i7Qaxk.png](https://i.328888.xyz/2023/04/15/i7Qaxk.png)



在remix里面创建一个合约

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Force_test {
    constructor(address payable target) public payable {
        require(msg.value > 0);
        selfdestruct(target);
    }
}
```

[![i7YV8y.png](https://i.328888.xyz/2023/04/15/i7YV8y.png)](https://imgloc.com/i/i7YV8y)



编译并部署该合约。

ENVIRONMENT改为metamask地址。VALUE改为1，表示给目的地址传入1wei

![i7Q1LL.png](https://i.328888.xyz/2023/04/15/i7Q1LL.png)



交易成功后，在Console里使用命令``getBalance(instance)``查看合约的额度为“非零”

提交该实例

![i7QYnp.png](https://i.328888.xyz/2023/04/15/i7QYnp.png)
