# Ethernaut做题笔记(第七题)



## 1.题目要求

这一关的目标是申明你对你创建实例的所有权.

 这可能有帮助

- 仔细看solidity文档关于 `delegatecall` 的低级函数, 他怎么运行的, 他如何将操作委托给链上库, 以及他对执行的影响.

- Fallback 方法

- 方法 ID



## 2.题目代码

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Delegate {

  address public owner;

  constructor(address _owner) {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
}
```



## 3.合约分析

在这里我们看到了两个合约，Delegate初始化时将传入的address设定为合约的owner，下面一个pwn函数也引起我们的注意，从名字也能看出挺关键的。
之后下面的Delegation合约则实例化了上面的Delegate合约，其fallback函数使用了delegatecall来调用其中的delegate合约，而这里的delegatecall就是问题的关键所在。
我们经常会使用call函数与合约进行交互，对合约发送数据，当然，call是一个较底层的接口，我们经常会把它封装在其他函数里使用，不过性质是差不多的，这里用到的delegatecall跟call主要的不同在于通过delegatecall调用的目标地址的代码要在当前合约的环境中执行，也就是说它的函数执行在被调用合约部分其实只用到了它的代码，所以这个函数主要是方便我们使用存在其他地方的函数，也是模块化代码的一种方法，然而这也很容易遭到破坏。用于调用其他合约的call类的函数，其中的区别如下：
1、call 的外部调用上下文是外部合约
2、delegatecall 的外部调用上下是调用合约上下文
3、callcode() 其实是 delegatecall() 之前的一个版本，两者都是将外部代码加载到当前上下文中进行执行，但是在 msg.sender 和 msg.value 的指向上却有差异。

在这里我们要做的就是使用delegatecall调用delegate合约的pwn函数，这里就涉及到使用call指定调用函数的操作，当你给call传入的第一个参数是四个字节时，那么合约就会默认这四个自己就是你要调用的函数，它会把这四个字节当作函数的id来寻找调用函数，而一个函数的id在以太坊的函数选择器的生成规则里就是其函数签名的sha3的前4个bytes，函数前面就是带有括号括起来的参数类型列表的函数名称。

经过上面的简要分析，问题就变很简单了，sha3我们可以直接通过web3.sha3来调用，而delegatecall在fallback函数里，我们得想办法来触发它，前面已经提到有两种方法来触发，但是这里我们需要让delegatecall使用我们发送的data，所以这里我们直接用封装好的sendTransaction来发送data，其实到了这里我也知道了前面fallback那关我们也可以使用这个方式来触发fallback函数：

```solidity
contract.sendTransaction({data:web3.utils.sha3("pwn()").slice(0,10)});
```



## 4.攻击流程

先获取一个实例，查看玩家地址与合约地址

![i7peMP.png](https://i.328888.xyz/2023/04/15/i7peMP.png)



之后通过fallback函数里的delegatecall来调用pwn函数更换owner：

再次查看合约拥有者地址，并提交该实例

![i7p6tX.png](https://i.328888.xyz/2023/04/15/i7p6tX.png)
