# 重入攻击

## 本文主要包括

* 什么是重入攻击
* 重入攻击的原理
* 单函数重入攻击
* 跨函数重入攻击
* 重入攻击解决方案新引入的漏洞
* 防止被永久锁定

## 什么是重入攻击

> 重入攻击是 Solidity 智能合约中最具破坏性的攻击之一。当一个函数对另一个不受信任的合约进行外部调用时，就会发生重入攻击。然后，不受信任的合约递归回调原始函数，试图耗尽资金。

当合约在发送资金之前未能更新其状态时，攻击者可以不断调用 withdraw 函数来耗尽合约的资金。现实世界中一个著名的重入攻击是造成 6000 万美元损失的 DAO 攻击。

### 重入攻击的原理

当智能合约向不受信任的智能合约发送以太币时，不受信任的智能合约的`receive`函数或`fallback`函数将会被调用（前者不存在时将调用后者），不受信任的智能合约可以在这两个函数中反向调用发送方智能合约的函数，来实现重入攻击。

### 以太币转移和回退功能 <a href="#yi-tai-bi-zhuan-yi-he-hui-tui-gong-neng" id="yi-tai-bi-zhuan-yi-he-hui-tui-gong-neng"></a>

以太坊智能合约中重入攻击的可能性来自以太坊上处理价值转移的方式。以太坊对用户和智能合约账户一视同仁，因此两者都可以调用智能合约或接收以太币的转账。

如果以太币被转移到一个包含智能合约代码的地址，那么该智能合约就有机会运行一些代码。这种“回退功能”可用于根据转账更新内部状态或采取存款触发的其他行动（例如发行一些代币作为响应）。

## 重入漏洞分析 <a href="#san-zhong-ru-lou-dong-fen-xi" id="san-zhong-ru-lou-dong-fen-xi"></a>

### 单函数重入攻击

这个漏洞的第一个版本涉及可以在函数的第一次调用完成之前重复调用的函数。这可能会导致函数的不同调用以破坏性方式进行交互。

```javascript
// INSECURE
mapping (address => uint) private userBalances;

function withdrawBalance() public {
    uint amountToWithdraw = userBalances[msg.sender];
    (bool success, ) = msg.sender.call.value(amountToWithdraw)(""); // At this point, the caller's code is executed, and can call withdrawBalance again
    require(success);
    userBalances[msg.sender] = 0;
}

```

由于用户的余额直到函数的最后才设置为 0，第二次（和以后的）调用仍然会成功，并且会一遍又一遍地提取余额。\
2016 年 6 月 17日，The DAO遭到黑客攻击，360 万以太币（5000 万美元）在第一次重入攻击中被盗。以太坊基金会发布了一个重要更新来回滚黑客攻击。这导致以太坊被分叉为以太坊经典和以太坊。

在给出的例子中，防止这种攻击的最好方法是确保在完成所有需要做的内部工作之前不要调用外部函数：

```solidity
mapping (address => uint) private userBalances;

function withdrawBalance() public {
    uint amountToWithdraw = userBalances[msg.sender];
    userBalances[msg.sender] = 0;
    (bool success, ) = msg.sender.call.value(amountToWithdraw)(""); // The user's balance is already 0, so future invocations won't withdraw anything
    require(success);
}

```

请注意，如果您有另一个调用 的函数withdrawBalance()，它可能会受到相同的攻击，因此您必须将调用不受信任合约的任何函数视为本身不受信任。有关潜在解决方案的进一步讨论，请参见下文。

### 跨函数重入 <a href="#kua-han-shu-zhong-ru" id="kua-han-shu-zhong-ru"></a>

攻击者还可以使用共享相同状态的两个不同函数进行类似的攻击。

```solidity
// INSECURE
mapping (address => uint) private userBalances;

function transfer(address to, uint amount) {
    if (userBalances[msg.sender] >= amount) {
       userBalances[to] += amount;
       userBalances[msg.sender] -= amount;
    }
}

function withdrawBalance() public {
    uint amountToWithdraw = userBalances[msg.sender];
    (bool success, ) = msg.sender.call.value(amountToWithdraw)(""); // At this point, the caller's code is executed, and can call transfer()
    require(success);
    userBalances[msg.sender] = 0;
}

```

在这种情况下，攻击者在他们的代码中外部调用执行withdrawBalance的时候调用transfer()，由于他们的余额尚未设置为 0，因此即使他们已经收到提款，他们也可以转移代币。该漏洞也被用于 DAO 攻击。 相同的解决方案将起作用，但有相同的注意事项。另请注意，在此示例中，两个函数都是同一个合约的一部分。但是，如果多个合约共享状态，则相同的错误可能会出现在多个合约中。

## 重入解决方案中的陷阱 <a href="#si-zhong-ru-jie-jue-fang-an-zhong-de-xian-jing" id="si-zhong-ru-jie-jue-fang-an-zhong-de-xian-jing"></a>

由于重入可能发生在多个函数甚至多个合约中，任何旨在防止单个函数重入的解决方案都是不够的。

相反，我们建议首先完成所有内部工作（即状态更改），然后才调用外部函数。如果仔细遵守此规则，您将可以避免由于重入而导致的漏洞。但是，您不仅要避免过早调用外部函数，还要避免调用调用外部函数的函数。例如，以下是不安全的：

```solidity
// INSECURE
mapping (address => uint) private userBalances;
mapping (address => bool) private claimedBonus;
mapping (address => uint) private rewardsForA;

function withdrawReward(address recipient) public {
    uint amountToWithdraw = rewardsForA[recipient];
    rewardsForA[recipient] = 0;
    (bool success, ) = recipient.call.value(amountToWithdraw)("");
    require(success);
}

function getFirstWithdrawalBonus(address recipient) public {
    require(!claimedBonus[recipient]); // Each recipient should only be able to claim the bonus once

    rewardsForA[recipient] += 100;
    withdrawReward(recipient); // At this point, the caller will be able to execute getFirstWithdrawalBonus again.
    claimedBonus[recipient] = true;
}

```

即使getFirstWithdrawalBonus()不直接调用外部合约，调用 withdrawReward()也足以使其容易受到重入的影响。因此，您需要将其 withdrawReward()视为不受信任。

```solidity
mapping (address => uint) private userBalances;
mapping (address => bool) private claimedBonus;
mapping (address => uint) private rewardsForA;

function untrustedWithdrawReward(address recipient) public {
    uint amountToWithdraw = rewardsForA[recipient];
    rewardsForA[recipient] = 0;
    (bool success, ) = recipient.call.value(amountToWithdraw)("");
    require(success);
}

function untrustedGetFirstWithdrawalBonus(address recipient) public {
    require(!claimedBonus[recipient]); // Each recipient should only be able to claim the bonus once

    claimedBonus[recipient] = true;
    rewardsForA[recipient] += 100;
    untrustedWithdrawReward(recipient); // claimedBonus has been set to true, so reentry is impossible
}

```

除了使重新进入变得不可能的修复之外， 还标记了不受信任的功能。同样的模式在每个级别重复：因为untrustedGetFirstWithdrawalBonus()调用 untrustedWithdrawReward()，它调用外部合约，所以您也必须将 untrustedGetFirstWithdrawalBonus()视为不安全的。 通常建议的另一种解决方案是mutex。这允许您“锁定”某些状态，因此它只能由锁的所有者更改。一个简单的示例可能如下所示：

```solidity
// Note: This is a rudimentary example, and mutexes are particularly useful where there is substantial logic and/or shared state
mapping (address => uint) private balances;
bool private lockBalances;

function deposit() payable public returns (bool) {
    require(!lockBalances);
    lockBalances = true;
    balances[msg.sender] += msg.value;
    lockBalances = false;
    return true;
}

function withdraw(uint amount) payable public returns (bool) {
    require(!lockBalances && amount > 0 && balances[msg.sender] >= amount);
    lockBalances = true;

    (bool success, ) = msg.sender.call.value(amount)("");

    if (success) { // Normally insecure, but the mutex saves it
      balances[msg.sender] -= amount;
    }

    lockBalances = false;
    return true;
}

```

如果用户在第一次调用完成之前尝试再次调用withdraw()，锁将阻止它产生任何效果。这可能是一种有效的模式，但当您有多个合约时，它会变得棘手。以下是不安全的：

```solidity
// INSECURE
contract StateHolder {
    uint private n;
    address private lockHolder;

    function getLock() {
        require(lockHolder == address(0));
        lockHolder = msg.sender;
    }

    function releaseLock() {
        require(msg.sender == lockHolder);
        lockHolder = address(0);
    }

    function set(uint newState) {
        require(msg.sender == lockHolder);
        n = newState;
    }
}

```

攻击者可以调用getLock()，然后永远不会调用releaseLock()。如果他们这样做，那么合同将永远被锁定，并且无法进行进一步的更改。如果您使用互斥体来防止重入，您将需要仔细确保没有任何方法可以声明锁并且永远不会释放锁。

## 总结 <a href="#wu-zong-jie" id="wu-zong-jie"></a>

DeFi 智能合约容易遭受重入攻击的一个共同点是，它们在易受攻击的代码启动之前没有经过安全审计。重入漏洞是一种众所周知的威胁，应该作为安全审计的一部分进行识别和修复。 保护web3世界的安全我们义不容辞！

参考文档：[https://learnblockchain.cn/article/5260](https://learnblockchain.cn/article/5260)
