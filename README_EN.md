#  EOS Smart Contract Development Security Best Practices

中文版请见: [此处](/README.md)

English version translated by: [Kai Jing(from EOS42)](https://t.me/shuke0327)

This document aims to provide some **security guidelines** for developers of EOS smart contracts and list analysis of some **known contract vulnerabilities**. We invite the community to suggest for modifications or improvements of this document and welcome various kinds of pull requests.  You are also welcomed to add relevant articles or blogs published, please add them to [reference](#Reference).


## Directories

* [Security Guidelines](#security-guidelines)
* [Known Vulnerabilities](#known-vulnerabilities)
   * [Numerical Overflow](#numerical-overflow)
      * [Vulnerability Sample](#vulnerability-sample)
      * [Defense Method](#defense-method)
      * [The Real Case](#the-real-case)
   * [Authorization Check](#authorization-check)
      * [Vulnerability Sample](#vulnerability-sample-1)
      * [Defense Method](#defense-method-1)
      * [The Real Case](#the-real-case-1)
   * [Apply Check](#apply-check)
      * [Vulnerability Sample](#vulnerability-sample-2)
      * [Defense Method](#defense-method-2)
      * [The Real Case](#the-real-case-2)
* [Reference](#reference)
* [Acknowledgement](#acknowledgement)

## [Security Guidelines](#security-guidelines)

EOS is still in its early stages and has some experimental characteristics. As a result, as new bugs and security vulnerabilities are discovered and new features are developed, the security threats we face are constantly changing. This article is just the start for developers  to create secure smart contracts.

Developing smart contracts requires a new kind of engineering mindset, which is different from the development of our previous projects. Because the cost of making mistakes are so high, it's so hard to make up for it by patching,  as centralized software does. As with hardware programming or software development for financial services, there are greater challenges to face than Web development or mobile development. Therefore, it is not enough to guard against the known vulnerabilities, but to learn new development concepts:

- **Be prepared for possible mistakes**. Any meaningful smart contract is more or less wrong, so your code must be able to properly handle the bugs and vulnerabilities that arise. 
Always follow these rules:
- stop the smart contract when an error occurs
- manage the risk of the account, such as limit (transfer) rate and set maximum (transfer) limit
- figure out effective ways to fix bugs and improve functionality

- **Be prudent of releasing smart contracts**.Try  your best to find out and fix the potential bugs before the smart contract is officially released.
   - Test smart contracts thoroughly and  retest them in time after any new attacks are discovered (testing those contracts that have been issued as well)
   - Invite professional security audit firms for audition and provide Bug Bounty Program from the start of the alpha release on the cryptokylin-testnet, Jungle-testnet, or other public test nets.
  - Release in several phases, at each phrase, ensure adequate testings are provided.

- **Keep the smart contracts simply**。Increased complexity will increase the risk of error.

  - Ensure the logic of the smart contracts are concise
  - Ensure that contracts and functions are modular
  - Use contracts or tools that are already widely adapted(For example, don't write a random number generator yourself)
  - Clarity is more important than performance when allows
  - Use blockchain tech only for the decentralized part of your system

- **Keep updated**. Ensure access to the latest security developments by disclosing resources.
  - Check your smart contract when any new vulnerabilities are discovered
  - Update the library or tool as quickly as possible when possible
  - Use the latest security technologies

- **Get clear understanding of blockchain features**。Although your previous programming experience is also applicable to smart contract development, there are still pitfalls to keep an eye out for：
  - `require_recipient(account_name name)` will trigger notification，and call the function with the same name within `name` contract(if account `name` already deployed contract)，[see official doc here](https://developers.eos.io/eosio-cpp/v1.2.0/reference#section-require_recipient)

## [Known Vulnerabilities](#known-vulnerabilities)

### [Numerical Overflow](#numerical-overflow)

When doing arithmetic operations, failing to check the boundaries may cause the values to overflow, causing loss of  users assets.

#### [Vulnerability Sample](#vulnerability-sample)

codes with vulnerability：`batchtransfer` batch transfer

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

    uint64_t amount = balance * 4; //Multiplication overflow

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

#### [Defense Method](#defense-method)
As far as possible, use the `asset` structure for operations, rather than extract `balance` for operations.

#### [The Real Case](#the-real-case)

- [【Don't play EOS Fomo3D Game】the Wolf game is under overflow attack and go die](https://bihu.com/article/995093)

### [Authorization Check](#authorization-check)

When making relevant operations, please do strictly determine whether the parameters passed into the function are consistent with the actual caller , use `require_auth` for authorization check.

#### [Vulnerability Sample](#vulnerability-sample-1)

codes with vulnerability：`transfer` 

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

#### [Defense Method](#defense-method-1)

Use `require_auth( from )` method to check whether the asset transfer account is consistent with the calling account

#### [The Real Case](#the-real-case-1)

None

### [Apply Check](#apply-check)

When processing contract calls, ensure that each action and codes meet the associated requirements.

#### [Vulnerability Sample](#vulnerability-sample-2)

codes with vulnerability：

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

#### [Defense Method](#defense-method-2)

Use the codes below:

```
if( ((code == self  && action != N(transfer) ) || (code == N(eosio.token) && action == N(transfer)) || action == N(onerror)) ) { }
```
Bind each key action and code to meet the requirements, in order to avoid abnormal and illegal calls.

#### [The Real Case](#the-real-case-2)

[EOSBet Transfer Hack Statement](https://medium.com/@eosbetcasino/eosbet-transfer-hack-statement-31a3be4f5dcf)

## [Reference](#reference)

- [保管好私钥就安全了吗？注意隐藏在EOS DAPP中的安全隐患](https://zhuanlan.zhihu.com/p/40625180)
- [漏洞详解|恶意 EOS 合约存在吞噬用户 RAM 的安全风险](https://zhuanlan.zhihu.com/p/40469719)
- [How EOSBET attacked by aabbccddeefg](https://www.reddit.com/r/eos/comments/9fpcik/how_eosbet_attacked_by_aabbccddeefg/)
- [BET被黑客攻击始末，实锤还原作案现场和攻击手段](https://github.com/ganjingcun/bet-death-causes/blob/master/README.md)
- [累计薅走数百万，EOS Dapps已成黑客提款机？](https://mp.weixin.qq.com/s/74ggygC3nbDihLkobXOW2w)

## [Acknowledgement](#acknowledgement)

- [CryptoKylin Workgroup](https://github.com/cryptokylin)
- eosiofans
- Kai Jing(荆凯)
- 星魂
- 岛娘
- Yu Zhao(赵余)
- 字符
