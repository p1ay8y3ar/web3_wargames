
# FAL1OUT

## 提示:
```text
获得以下合约的所有权来完成这一关.

  这可能有帮助

Solidity Remix IDE
```


源代码:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "openzeppelin-contracts-06/math/SafeMath.sol";

contract Fallout {
    using SafeMath for uint256;

    mapping(address => uint256) allocations;
    address payable public owner;

    /* constructor */
    function Fal1out() public payable {
        owner = msg.sender;
        allocations[owner] = msg.value;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "caller is not the owner");
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

    function allocatorBalance(address allocator) public view returns (uint256) {
        return allocations[allocator];
    }
}
```



## 知识点
现在的solidity，添加了一个`constructor` 关键字用来创建构造函数。但是，在0.4.22之前，这个关键字是没有的。之前的构造函数和类名同名。
这样就在历史上发生了一件事情，就是合约改了，构造函数的名字没有改。而且一般的构造函数权限又没改，可以直接调用，erc20也没有所有权，可能通过这种方式获取合约的所有权。

##  solution

`await contract.Fal1out({value:1})`
