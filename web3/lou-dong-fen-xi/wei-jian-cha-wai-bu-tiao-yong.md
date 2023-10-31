# 未检查外部调用

> Solidity中，如果调用的函数抛出异常，则会继续冒泡传播，但如果调用的函数在出现异常的时候，返回bool类型的值，那么调用方应当检查返回值，如不检查返回值，将导致业务逻辑出现漏洞。

Solidity 底层调用方法，(例如 `address.call()`) 不会抛出异常。而是在遇到错误，返回`false`。

而如果使用合约调用`ExternalContract.doSomething()`时，如果 `doSomething()`抛出异常，则异常会继续“冒泡”传播。

应该通过检查返回值来显式处理不成功的情况，以下使用`addr.send()`进行以太币转账是一个很好的例子，这对于其他外部调用也有效

```javascript
if(!addr.send(1)) {
	revert()
}
```

