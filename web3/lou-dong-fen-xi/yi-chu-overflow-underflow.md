# 溢出（Overflow / Underflow）



> Solidity的256位虚拟机存在上溢出和下溢出问题（译者注：由于结果超出取值范围称为溢出）， [这里](https://www.checkmarx.com/blog/checkmarx-research-solidity-and-smart-contracts-from-a-security-standpoint)有具体的分析。 在`for`循环条件中使用`uint`数据类型时，开发人员要格外小心，因为它可能导致无限循环

如下代码，可能会出现溢出的问题：

```javascript
for (uint i = border; i >= 0; i--) {
  ans += i;
}
```

在上面的示例中，当`i`的值为`0`时，下一个值为`2^256 -1`，这使条件始终为`true`。 开发人员应当尽量使用`<`、`>`、`!=`和`==`进行比较。
