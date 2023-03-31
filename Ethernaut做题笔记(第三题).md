# Ethernaut做题笔记(第三题)

## 1.题目要求

 获得以下合约的所有权来完成这一关.

 这可能有帮助

- Solidity Remix IDE



## 2.题目代码

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import 'openzeppelin-contracts-06/math/SafeMath.sol';

contract Fallout {
  
  using SafeMath for uint256;
  mapping (address => uint) allocations;
  address payable public owner;


  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

  modifier onlyOwner {
	        require(
	            msg.sender == owner,
	            "caller is not the owner"
	        );
	        _;
	    }

  function allocate() public payable {
    allocations[msg.sender] = allocations[msg.sender].add(msg.value);
  }

  function sendAllocation(address payable allocator) public {
    require(allocations[allocator] > 0);
    allocator.transfer(allocations[allocator]);
  }

  function collectAllocations() public onlyOwner {
    msg.sender.transfer(address(this).balance);
  }

  function allocatorBalance(address allocator) public view returns (uint) {
    return allocations[allocator];
  }
}
```



## 3.合约分析

该关卡的要求是获取合约的owner，我们从上面的代码中可以看到没有类似于上一关的回退函数也没有相关的owner转换函数，但是我们在这里却发现一个致命的错误————构造函数名称与合约名称不一致使其成为一个public类型的函数，即任何人都可以调用，同时在构造函数中指定了函数调用者直接为合约的owner，所以我们可以直接调用构造函数Fal1out来获取合约的ower权限。



## 4.攻击流程

先生成一个新实例

[![iw4xTo.png](https://i.328888.xyz/2023/03/31/iw4xTo.png)](https://imgloc.com/i/iw4xTo)



之后查看当前合约的owner，并调用构造函数来更换owner

``await contract.owner()`` ``await contract.Fallout()`` 

[![iw4XHc.png](https://i.328888.xyz/2023/03/31/iw4XHc.png)](https://imgloc.com/i/iw4XHc)



再次查看合约的owner，发现已经发生改变了

``await contract.owner()`` 

[![iw4puJ.png](https://i.328888.xyz/2023/03/31/iw4puJ.png)](https://imgloc.com/i/iw4puJ)



提交该实例

[![iw4geA.png](https://i.328888.xyz/2023/03/31/iw4geA.png)](https://imgloc.com/i/iw4geA)



