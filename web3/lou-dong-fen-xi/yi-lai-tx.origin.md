# 依赖tx.origin

> 智能合约不应依赖于`tx.origin`进行身份验证，因为恶意合约可能会进行中间人攻击，耗尽所有资金。 建议改用`msg.sender`

如下代码，可能被出现被中间人攻击，从而耗尽合约内的所有资金：

```javascript
function transferTo(address dest, uint amount) {
	require(tx.origin == owner) {
	   dest.transfer(amount);
	}
}
```

`tx.origin`始终是合约调用链中的最初的发起者帐户，而`msg.sender`代表直接调用者。

如果链中的最后一个合约依赖于`tx.origin`进行身份验证，那么调用链中间环节的合约将能够榨干被调用合约的资金，因为身份验证没有检查究竟是谁（`msg.sender`）进行了调用。
