# 挖矿

1.去下载安装Go运行库 https://golang.google.cn/dl/ windows的选最左边的windows版本&#x20;

2.去官方github下载挖矿端 https://github.com/PowERC-20/PoWERC20Miner 点右上角绿色的code 然后Download ZIP&#x20;

3.解压压缩包 在解压出来的文件夹里打开powershell（在顶上地址栏直接输入powershell即可打开）&#x20;

4.输入指令 go get 以安装运行环境 等待安装完成&#x20;

5.输入指令 go build -o Powerc20Worker 以编译得到挖矿端程序&#x20;

6.指令执行后会在文件夹生成一个没用后缀名的文件 Powerc20Worker 把这个文件重命名并加上后缀名.exe&#x20;

7.输入挖矿指令开始挖 ./Powerc20Worker -privateKey 你的钱包私钥 -contractAddress 0xca9b78435Be8267922E7Ac5cDE70401e7502c9cc -workerCount 64 挖矿指令“你的私钥”换成钱包私钥 记得用临时钱包 出块后不会自行显示 从区块浏览器自己查 有合约交互就代表出了一次 为加快速度 可以多开几个powershell窗口 每个窗口指令相同 多开可加快速度

[https://powerc20.com](https://powerc20.com)





一个地址限制上限十张 目前成本差不多五刀一张

> \[!NOTE] 个人挖矿
>
> 我目前使用腾讯云轻量级服务器在挖矿，钱包用的是谷歌浏览器小狐狸钱包主钱包（没有资金）
