# 算数精度问题

> 由于使用256位虚拟机（[EVM](https://learnblockchain.cn/2019/04/09/easy-evm)），Solidity的数据类型有些复杂。 Solidity 不提供浮点运算， 并且少于32个字节的数据类型将被打包到同一个32字节的槽位中

例如以下代码是存在精度问题的：

```javascript
function calculateBonus(uint amount) returns (uint) {
  return amount/DELIMITER * BONUS;
}
```

乘法之前进行了处罚，可能存在精度问题。

可以采用以下安全的苦，如uniswap代码内的：QU
