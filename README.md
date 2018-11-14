# EOS 智能合约最佳安全开发指南

English Version: [check here](/README_EN.md)

这篇文档旨在为 EOS 智能合约开发人员提供一些智能合约的**安全准则**及**已知漏洞分析**。我们邀请社区对该文档提出修改或完善建议，欢迎各种合并请求(Pull Request)。若有相关的文章或博客的发表，也请将其加入到[参考文献](#参考文献)中。

## 目录

* [安全准则](#安全准则)
* [已知漏洞](#已知漏洞)
   * [数值溢出](#数值溢出)
      * [漏洞示例](#漏洞示例)
      * [防御方法](#防御方法)
      * [真实案例](#真实案例)
   * [权限校验](#权限校验)
      * [漏洞示例](#漏洞示例-1)
      * [防御方法](#防御方法-1)
      * [真实案例](#真实案例-1)
   * [apply 校验](#apply-校验)
      * [漏洞示例](#漏洞示例-2)
      * [防御方法](#防御方法-2)
      * [真实案例](#真实案例-2)
   * [transfer 假通知](#transfer-假通知)
      * [漏洞示例](#漏洞示例-3)
      * [防御方法](#防御方法-3)
      * [真实案例](#真实案例-3)
   * [随机数实践](#随机数实践)
      * [漏洞示例](#漏洞示例-4)
      * [防御方法](#防御方法-4)
      * [真实案例](#真实案例-4)
* [参考文献](#参考文献)
* [致谢](#致谢)

## 安全准则

EOS 处于早期阶段并且有很强的实验性质。因此，随着新的 bug 和安全漏洞被发现，新的功能不断被开发出来，其面临的安全威胁也是不断变化的。这篇文章对于开发人员编写安全的智能合约来说只是个开始。

开发智能合约需要一个全新的工程思维，它不同于我们以往项目的开发。因为它犯错的代价是巨大的，很难像中心化类型的软件那样，打上补丁就可以弥补损失。就像直接给硬件编程或金融服务类软件开发，相比于 Web 开发和移动开发都有更大的挑战。因此，仅仅防范已知的漏洞是不够的，还需要学习新的开发理念：

- **对可能的错误有所准备**。任何有意义的智能合约或多或少都存在错误，因此你的代码必须能够正确的处理出现的 bug 和漏洞。需始终保证以下规则：
	- 当智能合约出现错误时，停止合约
	- 管理账户的资金风险，如限制（转账）速率、最大（转账）额度
	- 有效的途径来进行 bug 修复和功能提升
- **谨慎发布智能合约**。 尽量在正式发布智能合约之前发现并修复可能的 bug。
	- 对智能合约进行彻底的测试，并在任何新的攻击手法被发现后及时的测试（包括已经发布的合约）
	- 从 alpha 版本在麒麟测试网(CryptoKylin-Testnet)上发布开始便邀请专业安全审计机构进行审计，并提供漏洞赏金计划(Bug Bounty)
	- 阶段性发布，每个阶段都提供足够的测试
- **保持智能合约的简洁**。复杂会增加出错的风险。
	- 确保智能合约逻辑简洁
	- 确保合约和函数模块化
	- 使用已经被广泛使用的合约或工具（比如，不要自己写一个随机数生成器）
	- 条件允许的话，清晰明了比性能更重要
	- 只在你系统的去中心化部分使用区块链
- **保持更新**。通过公开资源来确保获取到最新的安全进展。
	- 在任何新的漏洞被发现时检查你的智能合约
	- 尽可能快的将使用到的库或者工具更新到最新
	- 使用最新的安全技术
- **清楚区块链的特性**。尽管你先前所拥有的编程经验同样适用于智能合约开发，但这里仍然有些陷阱你需要留意：
	- `require_recipient(account_name name)` 可触发通知，如果账户`name`下有合约，会调用`name`合约中的同名函数，[官方文档](https://developers.eos.io/eosio-cpp/v1.2.0/reference#section-require_recipient)

## 已知漏洞

### 数值溢出

在进行算术运算时，未进行边界检查可能导致数值上下溢，引起智能合约用户资产受损。

#### 漏洞示例

存在缺陷的代码：`batchtransfer` 批量转账

```c++
typedef struct acnts {
    account_name name0;
    account_name name1;
    account_name name2;
    account_name name3;
} account_names;

void batchtransfer(symbol_name symbol, account_name from, account_names to, uint64_t balance)
{
    require_auth(from);
    account fromaccount;

    require_recipient(from);
    require_recipient(to.name0);
    require_recipient(to.name1);
    require_recipient(to.name2);
    require_recipient(to.name3);

    eosio_assert(is_balance_within_range(balance), "invalid balance");
    eosio_assert(balance > 0, "must transfer positive balance");

    uint64_t amount = balance * 4; //乘法溢出

    int itr = db_find_i64(_self, symbol, N(table), from);
    eosio_assert(itr >= 0, "Sub-- wrong name");
    db_get_i64(itr, &fromaccount, (account));
    eosio_assert(fromaccount.balance >= amount, "overdrawn balance");

    sub_balance(symbol, from, amount);

    add_balance(symbol, to.name0, balance);
    add_balance(symbol, to.name1, balance);
    add_balance(symbol, to.name2, balance);
    add_balance(symbol, to.name3, balance);
}
```

#### 防御方法

尽可能使用 asset 结构体进行运算，而不是把 balance 提取出来进行运算。

#### 真实案例

- [【EOS Fomo3D你千万别玩】狼人杀遭到溢出攻击, 已经凉凉](https://bihu.com/article/995093)

### 权限校验

在进行相关操作时，应严格判断函数入参和实际调用者是否一致，使用`require_auth`进行校验。

#### 漏洞示例

存在缺陷的代码：`transfer` 转账

```c++
void token::transfer( account_name from,
                      account_name to,
                      asset        quantity,
                      string       memo )
{
    eosio_assert( from != to, "cannot transfer to self" );
    eosio_assert( is_account( to ), "to account does not exist");
    auto sym = quantity.symbol.name();
    stats statstable( _self, sym );
    const auto& st = statstable.get( sym );

    require_recipient( from );
    require_recipient( to );

    eosio_assert( quantity.is_valid(), "invalid quantity" );
    eosio_assert( quantity.amount > 0, "must transfer positive quantity" );
    eosio_assert( quantity.symbol == st.supply.symbol, "symbol precision mismatch" );
    eosio_assert( memo.size() <= 256, "memo has more than 256 bytes" );

    auto payer = has_auth( to ) ? to : from;

    sub_balance( from, quantity );
    add_balance( to, quantity, payer );
}
```

#### 防御方法

使用`require_auth( from )`校验资产转出账户与调用账户是否一致。

#### 真实案例

暂无

### apply 校验

在处理合约调用时，应确保每个 action 与 code 均满足关联要求。

#### 漏洞示例

存在缺陷的代码：

```c++
// extend from EOSIO_ABI
#define EOSIO_ABI_EX( TYPE, MEMBERS ) \
extern "C" { \
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) { \
      auto self = receiver; \
      if( action == N(onerror)) { \
         /* onerror is only valid if it is for the "eosio" code account and authorized by "eosio"'s "active permission */ \
         eosio_assert(code == N(eosio), "onerror action's are only valid from the \"eosio\" system account"); \
      } \
      if( code == self || code == N(eosio.token) || action == N(onerror) ) { \
         TYPE thiscontract( self ); \
         switch( action ) { \
            EOSIO_API( TYPE, MEMBERS ) \
         } \
         /* does not allow destructor of thiscontract to run: eosio_exit(0); */ \
      } \
   } \
}

EOSIO_ABI_EX(eosio::charity, (hi)(transfer))
```

#### 防御方法

使用

```
if( ((code == self  && action != N(transfer) ) || (code == N(eosio.token) && action == N(transfer)) || action == N(onerror)) ) { }
```

绑定每个关键 action 与 code 是否满足要求，避免异常调用。

#### 真实案例

- [EOSBet 黑客攻击事件复盘](https://medium.com/@eosbetcasino/eosbet-%E9%BB%91%E5%AE%A2%E6%94%BB%E5%87%BB%E4%BA%8B%E4%BB%B6%E5%A4%8D%E7%9B%98-13663d8f3f1)

### transfer 假通知

在处理 `require_recipient` 触发的通知时，应确保 `transfer.to` 为 `_self`。

#### 漏洞示例

存在缺陷的代码：

```c++
// source code: https://gitlab.com/EOSBetCasino/eosbetdice_public/blob/master/EOSBetDice.cpp#L115
void transfer(uint64_t sender, uint64_t receiver) {

	auto transfer_data = unpack_action_data<st_transfer>();

	if (transfer_data.from == _self || transfer_data.from == N(eosbetcasino)){
		return;
	}

	eosio_assert( transfer_data.quantity.is_valid(), "Invalid asset");
}
```

#### 防御方法

增加

```
if (transfer_data.to != _self) return;
```

#### 真实案例

- [EOS DApp 充值“假通知”漏洞分析](https://mp.weixin.qq.com/s/8hg-Ykj0RmqQ69gWbVwsyg)

### 随机数实践

随机数生成算法不要引入可控或者可预测的种子

#### 漏洞示例

存在缺陷的代码：

```c++
// source code: https://github.com/loveblockchain/eosdice/blob/3c6f9bac570cac236302e94b62432b73f6e74c3b/eosbocai2222.hpp#L174
    uint8_t random(account_name name, uint64_t game_id)
    {
        auto eos_token = eosio::token(N(eosio.token));
        asset pool_eos = eos_token.get_balance(_self, symbol_type(S(4, EOS)).name());
        asset ram_eos = eos_token.get_balance(N(eosio.ram), symbol_type(S(4, EOS)).name());
        asset betdiceadmin_eos = eos_token.get_balance(N(betdiceadmin), symbol_type(S(4, EOS)).name());
        asset newdexpocket_eos = eos_token.get_balance(N(newdexpocket), symbol_type(S(4, EOS)).name());
        asset chintailease_eos = eos_token.get_balance(N(chintailease), symbol_type(S(4, EOS)).name());
        asset eosbiggame44_eos = eos_token.get_balance(N(eosbiggame44), symbol_type(S(4, EOS)).name());
        asset total_eos = asset(0, EOS_SYMBOL);
	//攻击者可通过inline_action改变余额，从而控制结果
        total_eos = pool_eos + ram_eos + betdiceadmin_eos + newdexpocket_eos + chintailease_eos + eosbiggame44_eos;
        auto mixd = tapos_block_prefix() * tapos_block_num() + name + game_id - current_time() + total_eos.amount;
        const char *mixedChar = reinterpret_cast<const char *>(&mixd);

        checksum256 result;
        sha256((char *)mixedChar, sizeof(mixedChar), &result);

        uint64_t random_num = *(uint64_t *)(&result.hash[0]) + *(uint64_t *)(&result.hash[8]) + *(uint64_t *)(&result.hash[16]) + *(uint64_t *)(&result.hash[24]);
        return (uint8_t)(random_num % 100 + 1);
    }
```

#### 防御方法

EOS链上不能生成真随机数，在设计随机算法时建议参考官方的示例

```
// source code: https://developers.eos.io/eosio-cpp/docs/random-number-generation
```

#### 真实案例

- [EOS DApp 充值“假通知”漏洞分析](https://mp.weixin.qq.com/s/8hg-Ykj0RmqQ69gWbVwsyg)

## 参考文献

- [保管好私钥就安全了吗？注意隐藏在EOS DAPP中的安全隐患](https://zhuanlan.zhihu.com/p/40625180)
- [漏洞详解|恶意 EOS 合约存在吞噬用户 RAM 的安全风险](https://zhuanlan.zhihu.com/p/40469719)
- [How EOSBET attacked by aabbccddeefg](https://www.reddit.com/r/eos/comments/9fpcik/how_eosbet_attacked_by_aabbccddeefg/)
- [BET被黑客攻击始末，实锤还原作案现场和攻击手段](https://github.com/ganjingcun/bet-death-causes/blob/master/README.md)
- [累计薅走数百万，EOS Dapps已成黑客提款机？](https://mp.weixin.qq.com/s/74ggygC3nbDihLkobXOW2w)

## 致谢

- [麒麟工作组](https://github.com/cryptokylin)
- eosiofans
- 荆凯(EOS42)
- 星魂
- 岛娘
- 赵余(EOSLaoMao)
- 字符
