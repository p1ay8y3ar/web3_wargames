# Privacy


## 代码
```solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Privacy {
    bool public locked = true;
    uint256 public ID = block.timestamp;
    uint8 private flattening = 10;
    uint8 private denomination = 255;
    uint16 private awkwardness = uint16(block.timestamp);
    bytes32[3] private data;

    constructor(bytes32[3] memory _data) {
        data = _data;
    }

    function unlock(bytes16 _key) public {
        require(_key == bytes16(data[2]));
        locked = false;
    }

    /*
    A bunch of super advanced solidity algorithms...

      ,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`
      .,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,
      *.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^         ,---/V\
      `*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.    ~|__(o.o)
      ^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'  UU  UU
    */
}
```


## 分析
还是和[9-Vault.md](./9-Vault.md)一样，链上无隐私。同时在[7-Delegation](./7-Delegation.md) 上也了解到了storage的layout。

需要注意的，solidity也会自动对齐。不足32个字节的，同类型的应该都会自动对齐。
因此:
bool public locked = true; -> `slot0`
uint256 public ID = block.timestamp; -> `slot1`
uint8 private flattening = 10; -> `slot2`
uint8 private denomination = 255; -> `slot2`
uint16 private awkwardness = uint16 (block.timestamp);-> `slot2`
bytes32[3] private data;

data[0]->-> `slot3`
data[1]->-> `slot4`
data[2]->-> `slot5`


因此通过读取slot=5的就可以读取到具体的值。
## pwn
不会写solidity代码，因为现在还不会solidity的会变。只能用web3js来做。

```js
await web3.eth.getStorageAt(contract.address,5)
'0x3236b54662ad442b905dc803044cf42e5ed4a49b57585810ff9eb9bd46e6707f'
```

这是32字节的，但是需要的参数是bytes16，16个字节的。

```
 await web3.eth.getStorageAt(contract.address,5).then(v=>v.slice(0,34))
'0x3236b54662ad442b905dc803044cf42e'
```
从0-34的原因，是web3返回的时候，默认给加了个`0x`，所以多了2个字节。

最后`await contract.unlock("0x3236b54662ad442b905dc803044cf42e")` 就通过了。





## ref
[How to read Ethereum contract storage](https://medium.com/@dariusdev/how-to-read-ethereum-contract-storage-44252c8af925)
