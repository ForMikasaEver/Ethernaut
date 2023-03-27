# Ethernaut题解第二题

## 1.题目要求
仔细看下面的合约代码.

通过这关你需要

1. 获得这个合约的所有权
2. 把他的余额减到0

这可能有帮助

* 如何通过与ABI互动发送ether
* 如何在ABI之外发送ether
* 转换 wei/ether 单位 (参见 help() 命令)
* Fallback 方法

## 2.题目代码
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {

  mapping(address => uint) public contributions;
  address public owner;

//通过构造函数初始化贡献者的值为1000ETH
  constructor() {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }


  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }

// 将合约所属者移交给贡献最高的人，这也意味着你必须要贡献1000ETH以上才有可能成为合约的owner
  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

//获取请求者的贡献值
  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

//取款函数，且使用onlyOwner修饰，只能被合约的owner调用
  function withdraw() public onlyOwner {
    payable(owner).transfer(address(this).balance);
  }

//fallback函数，用于接收用户向合约发送的代币
  receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);// 判断了一下转入的钱和贡献者在合约中贡献的钱是否大于0
    owner = msg.sender;
  }
}
```

## 3.解题步骤
### 3.1合约分析
通过源代码我们可以了解到要想改变合约的owner可以通过两种方法实现：
1、贡献1000ETH成为合约的owner(虽然在测试网络中我们可以不断的申请测试eth，但由于每次贡献数量需要小于0.001，完成需要1000/0.001次，这显然很不现实~)
2、通过调用回退函数fallback()来实现
显然我们这里需要通过第二种方法来获取合约的owner，而触发fallback()函数也有下面两种方式：

- 没有其他函数与给定函数标识符匹配
- 合约接收没有数据的纯ether(例如：转账函数)

因此我们可以调用转账函数"await contract.sendTransaction({value:1})"或者使用matemask的转账功能(注意转账地址是合约地址也就是说instance的地址)来触发fallback()函数。
那么分析到这里我们从理论上就可以获取合约的owner了，那么我们如何转走合约中的eth呢？很明显，答案就是——调用withdraw()函数来实现。

### 3.2攻击流程
先点击生成新实例

之后开始交互，首先查看合约地址的资产总量，并向其转1wei
![inH64C.png](https://i.328888.xyz/2023/03/27/inH64C.png)

等交易完成后再次获取balance发现成功改变：
![inHevE.png](https://i.328888.xyz/2023/03/27/inHevE.png)

通过调用sendTransaction函数来触发fallback函数并获取合约的owner：
![inHFIP.png](https://i.328888.xyz/2023/03/27/inHFIP.png)

之后等交易完成后再次查看合约的owner，发现成功变为我们自己的地址：
![inHGdX.png](https://i.328888.xyz/2023/03/27/inHGdX.png)

之后调用withdraw来转走合约的所有代币
![inHdut.png](https://i.328888.xyz/2023/03/27/inHdut.png)

再点击提交该实例即可完成闯关
![inH52J.png](https://i.328888.xyz/2023/03/27/inH52J.png)
![inHvWo.png](https://i.328888.xyz/2023/03/27/inHvWo.png)
