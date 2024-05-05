# 

## 
```
msg.sender.transfer(withdraw_amount);
```
- msg 是所有合约可以访问的输入之一。代表触发此合约的交易
- sender是发件人的地址
- transfer是一个内置函数，它将ether从合约地址到调用它的地址。

这句话要从后往前读: transfer wei的数量是withdraw_amount,到触发此合约执行的msg的sdner.

# ERC
## ERC20 同质化代币标准

[ERC-20标准,eip-20](https://eips.ethereum.org/EIPS/eip-20)
历史：
应该就是这个库的第20个问题。所以叫做erc20.
[ERC: Token standard #20](https://github.com/ethereum/EIPs/issues/20)
### optional 可选实现

### name
返回token的名字

```solidity
function name() public view returns (string)
```

### symbol
返回token的简称
```solidity
function symbol() public view returns (string)
```
### decimals 
合约使用的最小位数。`1/10^返回值`.比如eth就是18,代表eth的最小单位是`1^18=1 wei`.


```solidity
function decimals() public view returns (uint8)

```

## 必须实现
### totalSupply
返回代币的总供应量

```solidity
function totalSupply() public view returns (uint256)
```

### balanceOf
返回一个addrees名下的余额。

```solidity
function balanceOf(address _owner) public view returns (uint256 balance)
```

### transfer

```solidity
function transfer(address _to, uint256 _value) public returns (bool success)
```
将`_value`数量的代币转移到`_to`.`_value=0`也要视为正常的传输，虽然不知道为啥。

### transferFrom

```solidity
function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)
```

将`_value`的代币，从`_from`这个地址，转移到`_to`这个地址。

一般都是`withdraw`函数中使用这个比较多。

### approve
```solidity
function approve(address _spender, uint256 _value) public returns (bool success)
```

当前`msg.sender`可以让`_spender`提款/消费最多`_value`个代币。

就是类似于"亲情付".
如果同一个地址多次使用，那么`_value`会被覆盖。不过具体也得看那个合约自己的实现。

### allowance


```solidity
function allowance(address _owner, address _spender) public view returns (uint256 remaining)
```
返回 `_spender`仍然可以从`_owner`提现的金额。

## 事件
主要是作为日志打印使用。

### Transfer
在进行发现代币和提现的出后，必须触发。
```solidity
event Transfer(address indexed _from, address indexed _to, uint256 _value)
```

### Approval

调用 `approve`成功的时候必须触发。
```solidity
event Approval(address indexed _owner, address indexed _spender, uint256 _value)
```

### 总结

- ERC20也是一种智能合约的标准。主要是定义了代币合约的基本功能
- ERC20代币合约可以被创建和部署，用于发行代币。比如ETH，USDT都符合ERC20
- ERC20的每个token都不具有唯一性

## ERC165 合约接口检测标准

[ERC-165: Standard Interface Detection](https://eips.ethereum.org/EIPS/eip-165)

创建一种标准方法，用于发布和检测智能合约实现了哪些接口。

这个标准主要解决了：
- 如何识别接口
- 合约如何发布其实现的接口
- 如何检测合约是否实现了ERC165
- 如何检测合同是否实现了任何给定接口


## 规范

### 如何识别接口

solidity中使用`interface`关键字。这个关键字定义了一堆函数选择器的子集，主要是函数原型。
比如下面的这个例子：
```solidity
pragma solidity ^0.4.20;

interface Solidity101 {
    function hello() external pure;
    function world(int) external pure;
}


contract Selector {
    function calculateSelector() public pure returns (bytes4) {
        Solidity101 i;
        return i.hello.selector ^ i.world.selector;
    }
}
```



### 合约如何发布它实现的接口

符合ERC165的合约应该实现以下接口:
```solidity
pragma solidity ^0.4.20;

interface ERC165 {
    /// @notice Query if a contract implements an interface
    /// @param interfaceID The interface identifier, as specified in ERC-165
    /// @dev Interface identification is specified in ERC-165. This function
    ///  uses less than 30,000 gas.
    /// @return `true` if the contract implements `interfaceID` and
    ///  `interfaceID` is not 0xffffffff, `false` otherwise
    function supportsInterface(bytes4 interfaceID) external view returns (bool);
}
```
每个接口都有自己的interfaceid,就是一个哈希值。

比如，上面的接口:`bytes4(keccak256('supportsInterface(bytes4)'));` = `0x01ffc9a7`.

因此，实现 ERC165的合约必须:

`supportsInterface`返回

- true
  - 当`interfaceID`=`interfaceID`
    - 也就是EIP165的接口
  - 或者此合约实现的其他`interfaceID`
- false
  - `interfaceID`=`0xffffffff`
  - 或者其他的`interfaceID`
  
并且这个函数，要被限制最多使用`30 000`gas.


### 检测合约是否使用了ERC-165

- 源合约使用`STATICCALL`发送`0x01ffc9a701ffc9a700000000000000000000000000000000000000000000000000000000`到目的合约。然后gas limit设置`30 000`。这个data就代表了`contract.supportsInterface(0x01ffc9a7)`

- 如果直接调用失败，或者返回false，那么就是没实现ERC165.
- 如果返回true。那就就要发送第二次的call,`0x01ffc9a7ffffffff00000000000000000000000000000000000000000000000000000000`
- 如果调用失败，或者返回true，那么这个合约没有实现ERC-165.因为`0xffffffff`是返回false的。
- 其他的情况就说明实现了ERC-165.

### 如何检测合约是否实现任何给定的接口

- 按照上面的步骤检测是否实现了ERC-165
- 如果没实现，那么就要看看它是不是用了什么别的方法
- 如果实现了，那么直接使用`supportsInterface(interfaceID)`


### 解释

#### 检测一个合约有没有实现ERC165的代码
```solidity
pragma solidity ^0.4.20;

contract ERC165Query {
    bytes4 constant InvalidID = 0xffffffff;
    bytes4 constant ERC165ID = 0x01ffc9a7;

    function doesContractImplementInterface(address _contract, bytes4 _interfaceId) external view returns (bool) {
        uint256 success;
        uint256 result;

        (success, result) = noThrowCall(_contract, ERC165ID);
        if ((success==0)||(result==0)) {
            return false;
        }

        (success, result) = noThrowCall(_contract, InvalidID);
        if ((success==0)||(result!=0)) {
            return false;
        }

        (success, result) = noThrowCall(_contract, _interfaceId);
        if ((success==1)&&(result==1)) {
            return true;
        }
        return false;
    }

    function noThrowCall(address _contract, bytes4 _interfaceId) constant internal returns (uint256 success, uint256 result) {
        bytes4 erc165ID = ERC165ID;

        assembly {
                let x := mload(0x40)               // Find empty storage location using "free memory pointer"
                mstore(x, erc165ID)                // Place signature at beginning of empty storage
                mstore(add(x, 0x04), _interfaceId) // Place first argument directly next to signature

                success := staticcall(
                                    30000,         // 30k gas
                                    _contract,     // To addr
                                    x,             // Inputs are stored at location x
                                    0x24,          // Inputs are 36 bytes long
                                    x,             // Store output over input (saves space)
                                    0x20)          // Outputs are 32 bytes long

                result := mload(x)                 // Load the result
        }
    }
}
```


实现:
```solidity
pragma solidity ^0.4.20;

import "./ERC165.sol";

contract ERC165MappingImplementation is ERC165 {
    /// @dev You must not set element 0xffffffff to true
    mapping(bytes4 => bool) internal supportedInterfaces;

    function ERC165MappingImplementation() internal {
        supportedInterfaces[this.supportsInterface.selector] = true;
    }

    function supportsInterface(bytes4 interfaceID) external view returns (bool) {
        return supportedInterfaces[interfaceID];
    }
}

interface Simpson {
    function is2D() external returns (bool);
    function skinColor() external returns (string);
}

contract Lisa is ERC165MappingImplementation, Simpson {
    function Lisa() public {
        supportedInterfaces[this.is2D.selector ^ this.skinColor.selector] = true;
    }

    function is2D() external returns (bool){}
    function skinColor() external returns (string){}
}
```
没看懂这个

GPT解释的:
例如，ERC721 (NFT 标准) 的接口 ID 是 0x80ac58cd。如果一个合约实现了 ERC721 的所有函数，那么它应该在调用 supportsInterface(0x80ac58cd) 时返回 true

## ERC-173 合约所有权标准

本规范定义了拥有或控制合同的标准功能。

运行几个函数:

`owner() returns (address)`返回当前的合约所有者。

`transferOwnership(address newOwner)`转移合约所有者。

`OwnershipTransferred(address indexed previousOwner, address indexed newOwner)` 所有权转移的时候的时候的事件。

这位合约的所有权行为提供了规范。

下面是ERC-173的一个接口，而且还应该给ERC173实现ERC165

```solidity

/// @title ERC-173 Contract Ownership Standard
///  Note: the ERC-165 identifier for this interface is 0x7f5828d0
interface ERC173 /* is ERC165 */ {
    /// @dev This emits when ownership of a contract changes.    
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    /// @notice Get the address of the owner    
    /// @return The address of the owner.
    function owner() view external returns(address);
	
    /// @notice Set the address of the new owner of the contract
    /// @dev Set _newOwner to address(0) to renounce any ownership.
    /// @param _newOwner The address of the new owner of the contract    
    function transferOwnership(address _newOwner) external;	
}

interface ERC165 {
    /// @notice Query if a contract implements an interface
    /// @param interfaceID The interface identifier, as specified in ERC-165
    /// @dev Interface identification is specified in ERC-165. 
    /// @return `true` if the contract implements `interfaceID` and
    ///  `interfaceID` is not 0xffffffff, `false` otherwise
    function supportsInterface(bytes4 interfaceID) external view returns (bool);
}
```

`owner`函数必须以`pure`或者`view`的方式实现。

ps:pure的话不就是写死了一个地址吗。

`transferOwnership(address _newOwner)`必须是`public`或者`external`.

要放弃合约的所有权，直接0地址`transferOwnership(address(0)).`.


## ERC-191 数据签名标准
对以太坊中数据的签名的规范。
主要解决数据的预签名问题。
###  这个标准要解决的问题

背景:
多重签名钱包: 是一种需要多个用户签名才能执行交易的钱包，通常用于提高资金安全性或实现共同管理资金的目的。
预签名交易: 指用户提前对交易数据进行签名，然后将签名后的数据提交给钱包，等待其他用户签名以完成交易。
问题:
语义不明确: 目前没有对预签名交易的 signed_data 格式进行明确的规范，导致不同的钱包实现可能会有不同的解释方式，造成兼容性问题。
标准交易可重用: 由于缺乏约束，标准的以太坊交易数据可以作为预签名交易提交，这意味着攻击者可以将用户在其他钱包中签署的交易重放到新的钱包中，从而盗取资金。
缺乏验证者绑定: 预签名交易没有与特定的钱包或验证者绑定，这意味着攻击者可以将用户在一个钱包中签署的交易重放到另一个钱包中，即使这两个钱包的成员并不完全相同。
示例:
假设用户 A、B 和 C 共享一个 2/3 多重签名钱包 X，用户 A、B 和 D 共享另一个 2/3 多重签名钱包 Y。
用户 A 和 B 提前对钱包 X 的交易进行了签名，并将签名数据提交给钱包 X。
攻击者可以获取这些签名数据，并将其提交给钱包 Y。由于钱包 Y 也需要 2 个签名才能执行交易，攻击者只需要再获得 D 的签名即可完成交易，即使 D 并不知道这笔交易的具体内容。
潜在风险:
资金被盗：攻击者可以利用预签名交易的漏洞，将用户资金转移到自己的账户中。
交易被篡改：攻击者可以修改预签名交易的内容，例如更改收款地址或交易金额，从而损害用户的利益。



在以太坊中，`signed_data`, 进行拆包之后:

```
RLP<nonce, gasPrice, startGas, to, value, data>
```
目前以太坊中的所有transaction使用的都是RLP。
> RLP (Recursive Length Prefix encoding) 是一种用于编码任意嵌套结构化数据的序列化方法

### 规范

建议使用的格式:
```
0x19 <1 byte version> <version specific data> <data to sign>.
```

这里`0x19`的意思，就是确定当前的`singed_data`不是一个有效的RLP编码，因为如果是RLP的话，攻击者就可以构造一个与用户签名匹配的恶意交易数据，并将其提交到多重签名钱包中执行。


这就说名任何符合EIP-191 的`signed_data`都不会是Ethereum的transaction。

`version`的版本:
- 0x0 EIP-191
- 0x01 [EIP-721](https://eips.ethereum.org/EIPS/eip-712)
- 0x45 EIP-191

#### 0x00的格式

```
0x19 0x00 <intended validator address> <data to sign>
```

`<intended validator address>`:预期验证者地址，即多重签名钱包的地址。
- 在多重签名钱包中，预期验证者地址是钱包自身的地址。
- 将钱包地址包含在预签名交易数据中，可以确保该交易只能由指定的钱包进行验证和执行，从而防止交易被重放到其他钱包中。

`<data to sign>`:要签名的数据。


### 0x01的格式

01和00的区别就是，01是格式化数据，具体还是去看[EIP-721](https://eips.ethereum.org/EIPS/eip-712)

一个简单的实现:
```solidity
function signatureBasedExecution(address target, uint256 nonce, bytes memory payload, uint8 v, bytes32 r, bytes32 s) public payable {
        
    // Arguments when calculating hash to validate
    // 1: byte(0x19) - the initial 0x19 byte
    // 2: byte(0) - the version byte
    // 3: address(this) - the validator address
    // 4-6 : Application specific data

    bytes32 hash = keccak256(abi.encodePacked(byte(0x19), byte(0), address(this), msg.value, nonce, payload));

    // recovering the signer from the hash and the signature
    addressRecovered = ecrecover(hash, v, r, s);
   
    // logic of the wallet
    // if (addressRecovered == owner) executeOnTarget(target, payload);
}
```


### 0x45的格式
```
0x19 <0x45 (E)> <thereum Signed Message:\n" + len(message)> <data to sign>

```
E就是0x45的字节码。多了一个特定的字符



