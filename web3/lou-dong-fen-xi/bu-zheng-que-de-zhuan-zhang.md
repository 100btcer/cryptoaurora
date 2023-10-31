# 不正确的转账

该问题和[未检查外部调用](wei-jian-cha-wai-bu-tiao-yong.md)类似。

在合约之间进行以太币转账有多种方法。虽然官方推荐使用`addr.transfer(x)`函数，但我们仍然找到了还在使用`send()`函数的智能合约：

```javascript
if(!addr.send(1)) {
	revert()
}
```

请注意，如果转账不成功，则`addr.transfer(x)`会自动引发异常，同样减轻第一个未检查外部调用的问题。
