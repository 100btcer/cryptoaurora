# 权利过大的所有者

> Solidity智能合约中，一般都会设置合约所有者地址或部分敏感操作的操作者地址，一旦操作者地址权利过大，如果操作者私钥丢失或泄露，就会出现潜在的严重问题

例如下面这个合约：

```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.8.2 <0.9.0;

contract OverpoweredOwnerExample {
    address public owner;

    modifier onlyOwner {
        require(msg . sender == owner ,"Only owner can call this function.");
        _;
    }

    constructor() {
        owner = msg . sender;
    }
    function dosomething() public onlyOwner {

    }
    function doSomethingElse() public {
        require (msg.sender ==owner);
    }
}

```

只有合约所有者能够调用`doSomething()`和`doSomethingElse()`函数：前者使用`onlyOwner`修饰器， 而后者则显式执行该修饰器。这带来了严重的风险：如果所有者的私钥遭到泄露， 则攻击者可以控制该合约。
