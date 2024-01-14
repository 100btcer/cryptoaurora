# 时间戳依赖

（EVM）不提供时钟时间，并且通常用于获取时间戳的`now`变量（`block.timestamp`的别名）实际上是矿工可以操纵的环境变量。

```javascript
if (timeHasCome == block.timestamp) {
	winner.transfer(amount);
}
```

由于矿工可以操纵当前的环境变量，因此只能在不等式`>`、`<`、`>=`和`<=`中使用其值。

如果你的应用需要随机性，可以参考[RANDAO合约](https://github.com/randao/randao)， 该合约基于任何人都可以参与的去中心化自治组织（DAO），是所有参与者共同生成的随机数。
