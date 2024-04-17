# Raydium DEX

> 这篇文章主要记录通过使用golang操作Raydium和使用Raydium的SDK demo等，学习Solana和Raydium的过程，其中会有一些踩坑经历。记录这些主要是方便以后回顾和帮助更多的人学习这门技术，使用这门技术，基于这门技术做出更多更好的产品。

前段时间，我使用golang开发了一款基于Telegram Bot的交易机器人，用来交易Solana上的代币，交易所的话用的是Raydium，但是由于一些问题，狙击功能没搞好，因为不知道怎么解析池子数据。所以后来我开始学习nodejs，使用Raydium的SDK来操作和监控Raydium的交易。

这几天我在使用Raydium的sdk-demo，这里把整个过程记录下来。

Raydium SDK：[https://github.com/raydium-io/raydium-sdk](https://github.com/raydium-io/raydium-sdk)

Raydium SDK Demo：[https://github.com/raydium-io/raydium-sdk-V1-demo](https://github.com/raydium-io/raydium-sdk-V1-demo)

本地需要先安装solana环境，安装教程：[https://docs.solanalabs.com/cli/install](https://docs.solanalabs.com/cli/install)

我们生成一个地址：

```
solana-keygen new --no-outfile
```

一些常用命令：

`solana address`查看地址

`solana config get`获取配置

`solana config set --url` [`https://api.devnet.solana.com`](https://api.devnet.solana.com) 设置RPC节点

为了测试Radium的创建market id、创建流动性池、获取流动性池、添加流动性、撤出流动性、销毁LP等。

我们需要请求一些开发链的SOL，这里我们请求3个SOL

```
solana airdrop 1
```

然后我们创建一个spl代币：

```
spl-token create-token
```

这会产生一个代币地址，例如：`9YEkKmWtDtBaKMqfBRokJXmQiMPB7t4nzwRQuDgYrB1B`

然后创建token-account

```
spl-token create-account 9YEkKmWtDtBaKMqfBRokJXmQiMPB7t4nzwRQuDgYrB1B
```

铸造代币：

```
spl-token mint 9YEkKmWtDtBaKMqfBRokJXmQiMPB7t4nzwRQuDgYrB1B 1000000000000000000000
```

将sol转成wsol

```
spl-token wrap 1
```

非首次的话，添加参数 `--create-aux-account`

下面开始测试demo，需要注意地址上一定要多弄sol，我就是因为地址sol少，每次出错也不知道是什么原因，折腾了很久。

### 一、创建market id

```
ts-node src/utilsCreateMarket.ts
```

保持地址有足够的so，否则会报奇怪的错误，交易会打印两个txid，例如：

```
txids [
  'HpKp1TAbZX5iPw1VxHtQcSLG3JLnowkN8u9Q259kKEKGwGBcpHHE6GYdqsYhk6m9n66iQcYv6TD76J2VQaDHtrg',
  '45sMSANytu1chK8pg3Hv9GDpdx9T89rijbNCvuzQ22PxgTXunyzyJkCgQ5V6GFCrtbD2Ud4KyyLhbnJhroaxYKt5'
]
```

确保两个全部成功。

### 二、创建AMM池

```
ts-node src/ammCreatePool.ts
```

执行前，需要修改targetMarketId为上一步产生的marketId，例如：

```
const targetMarketId = new PublicKey("Ai9Gc2wnzL22c5yz6KWR4DbJLpgBjz3EPBEdfUEy5spk")
```

这一步可能会出问题，例如这个交易：

{% embed url="https://solscan.io/tx/Daoowt7wLagjZpgiPAxJUTV1PWexrDVGsmQhUYFc9v2SbVcELTJBiDy9BvZByVK5SdY4jGX9SXJAuysNDxi292N?cluster=devnet" %}

成功的交易：

{% embed url="https://solscan.io/tx/3A2m9fb9dtZQD34SvB5sRtZ7r7a7n5DtJLTvxxZF4N8keq9hZicQiGTYcRh4BBCiCdTznkXkuULsgt3J85DbL3tS?cluster=devnet" %}

注意代码里面要提取pool id和池子相关数据，并保存，供后续使用，同时需要注意，**代码中设置的两个币的数量就决定了baseToken代币的初始价格。**

添加流动性：

```
ts-node src/ammAddLiquidity.ts
```

成功的交易：

{% embed url="https://solscan.io/tx/2FG1DvjXhJknHq2whDqqxUYfJBFKcFzwNiSUV2T1rjZ8QKy6udJ6PYk64WE3vas4HyBCJLqh6XbviC9VLYeFdCoj?cluster=devnet" %}

注意数量，lpDecimal，否则可能报错。

移除流动性：

```
ts-node src/ammRemoveLiquidity.ts
```

成功的交易：

{% embed url="https://solscan.io/tx/yUka4rkpkk3dEswhe3P7nRuYAce9SwcpVsGGFNFSC8fykWS7tLmDEH7qPzU86EWobLekVEcB1JBdFHLJJn98TDr?cluster=devnet" %}

