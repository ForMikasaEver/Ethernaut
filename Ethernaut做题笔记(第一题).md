# Ethernaut做题笔记
参考答案：https://xz.aliyun.com/t/7173
         https://xz.aliyun.com/t/7174
***

## 1.Hello Ethernaut
### 1.配置MetaMask
安装metamask，测试网络为Goerli

### 2.打开浏览器控制台
在浏览器按F12，之后选择Console模块打开浏览器控制台，并查看相关信息。

### 3.使用控制台协助
控制台中输入"player"时就看到玩家的地址信息，输入getBlance(player)当前玩家的eth余额，查看控制台中的其他实用功能可以输入"help"进行查看
![第1，2，3步](https://i.328888.xyz/2023/03/22/1hHrV.png)

### 4.ethernaut 合约
控制台中输入"Ethernaut"即可查看当前以太坊合约所有可用函数。
![第4步](https://i.328888.xyz/2023/03/22/1h2AN.png)

### 5.和ABI互动
通过加"."可以实现对各个函数的引用(这里也可以把ethernaut当作一个对象实例)

### 6.获取测试以太币
全网找不到goerli测试币，找同学借了点

### 7.获得这个关卡的实例
点击最下的生成新实例

![第7步](https://i.328888.xyz/2023/03/22/1o8OC.png)

通过metamask转账后生成实例地址

### 8.检查合约
在控制台输入contract
![第8步](https://i.328888.xyz/2023/03/22/1Spqv.png)

### 9.和这个合约互动来完成关卡
在控制台输入所有对应指令后，弹出关卡实例链接，在metamask里发送。
完成后，点击页面的提交实例，又需要在metamask里发送，如果成功通过，则页面弹出源代码
![1vSHU.png](https://i.328888.xyz/2023/03/22/1vSHU.png)
![1vgEb.png](https://i.328888.xyz/2023/03/22/1vgEb.png)

源代码
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Instance {

  string public password;
  uint8 public infoNum = 42;
  string public theMethodName = 'The method name is method7123949.';
  bool private cleared = false;

  // constructor
  constructor(string memory _password) {
    password = _password;
  }

  function info() public pure returns (string memory) {
    return 'You will find what you need in info1().';
  }

  function info1() public pure returns (string memory) {
    return 'Try info2(), but with "hello" as a parameter.';
  }

  function info2(string memory param) public pure returns (string memory) {
    if(keccak256(abi.encodePacked(param)) == keccak256(abi.encodePacked('hello'))) {
      return 'The property infoNum holds the number of the next info method to call.';
    }
    return 'Wrong parameter.';
  }

  function info42() public pure returns (string memory) {
    return 'theMethodName is the name of the next method.';
  }

  function method7123949() public pure returns (string memory) {
    return 'If you know the password, submit it to authenticate().';
  }

  function authenticate(string memory passkey) public {
    if(keccak256(abi.encodePacked(passkey)) == keccak256(abi.encodePacked(password))) {
      cleared = true;
    }
  }

  function getCleared() public view returns (bool) {
    return cleared;
  }
}
```
