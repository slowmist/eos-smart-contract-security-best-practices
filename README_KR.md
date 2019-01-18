#  EOS 스마트 컨트랙트 최고의 보안 개발 가이드

中文版请见: [此处](/README.md)
English Version: [check here](/README_EN.md)

한국어 버전 번역: [Yongsun Choi (from EXTALES)](https://github.com/ever4cys)

이 문서는 EOS 스마트 컨트랙트 개발자들에게 **보안 가이드라인**을 제공하고 **알려진 컨트랙트 취약점**에 대한 분석들을 열거하는 것이 목적입니다. 우리는 공동체가 이 문서의 수정이나 개선을 제안하도록 요청하며, 다양한 형태의 풀 리퀘스트를 환영합니다. 관련 글이나 발행된 블로그를 추가하는 것 또한 환영하며, 자료들을 [참고자료](#참고자료)에 추가해 주세요.


## 차례

* [보안 가이드라인](#보안-가이드라인)
* [알려진 취약점](#알려진-취약점)
   * [산술 오버플로우](#산술-오버플로우)
      * [취약점 예시](#취약점-예시)
      * [방어 방법](#방어-방법)
      * [실제 사례](#실제-사례)
   * [권한 확인](#권한-확인)
      * [취약점 예시](#취약점-예시-1)
      * [방어 방법](#방어-방법-1)
      * [실제 사례](#실제-사례-1)
   * [apply 확인](#apply-확인)
      * [취약점 예시](#취약점-예시-2)
      * [방어 방법](#방어-방법-2)
      * [실제 사례](#실제-사례-2)
   * [transfer 허위 통보](#transfer-허위-통보)
      * [취약점 예시](#취약점-예시-3)
      * [방어 방법](#방어-방법-3)
      * [실제 사례](#실제-사례-3)
   * [난수 사례](#난수-사례)
      * [취약점 예시](#취약점-예시-4)
      * [방어 방법](#방어-방법-4)
      * [실제 사례](#실제-사례-4)
   * [롤백 공격](#롤백-공격)
      * [취약점 예시](#취약점-예시-5)
      * [방어 방법](#방어-방법-5)
      * [실제 사례](#실제-사례-5)


* [참고자료](#참고자료)
* [감사의글](#감사의글)

## [보안 가이드라인](#보안-가이드라인)

EOS는 여전히 초기 단계에 있고 실험적인 특성을 가지고 있다. 그 결과로, 새로운 버그나 보안 취약점들이 발견되고 새로운 기능이 개발되고 있어서, 우리가 마주하는 보안 위협은 지속적으로 변하고 있다. 이 글은 안전한 스마트 컨트랙트를 만드는 개발자들에게 단지 시작일 뿐이다.

스마트 컨트랙트를 개발하는 것은 새로운 종류의 엔지니어링 사고방식을 요구하는데, 그것은 우리의 기존 프로젝트 개발과는 다른 것이다. 실수를 만드는 것에 대한 비용이 꽤 크기 때문에, 중앙집중식 소프트웨어가 하는 것처럼 패치를 통하여 그것을 만회하는 것이 어렵다. 금융 서비스의 하드웨어 프로그래밍이나 소프트웨어 개발과 마찬가지로, 웹이나 모바일 개발에 비하여 훨씬 더 큰 도전에 직면하게 된다. 그러므로, 알려진 취약점들에 대하여 방어하는 것만으로는 충분하지 않으며, 새로운 개발 개념을 배워야 한다.

- **가능한 실수에 대하여 준비하라**. 어떠한 의미있는 스마트 컨트랙트도 다소간에 잘못된 부분이 있으므로, 당신의 코드는 발생하는 버그들과 취약점들을 적절하게 다룰 수 있어야 한다.
언제나 이 규칙을 따라라:
  - 오류가 발생하면 스마트 컨트랙트를 멈춰라.
  - 계정의 위험을 관리해라, (전송) 비율 제한을 두거나 최대 (전송) 한도와 같은 것으로.
  - 버그를 수정하거나 기능을 향상시키는 효율적인 방법을 알아내라.

- **스마트 컨트랙트를 배포하는 것에 신중하라**. 스마트 컨트랙트를 공식적으로 배포하기 전에 잠재적인 버그를 찾아내고 수정하는 것에 최선을 다하라.
  - 스마트 컨트랙트를 철저하게 테스트하고 어떤 새로운 공격이 발견된 후 적절한 시간에 다시 테스트 하라. (이미 발행된 스마트 컨트랙트도 마찬가지로 테스트)
  - 검토를 위하여 전문적인 보안 감사회사에 의뢰하고 기린넷, 정글넷 또는 다른 일반 테스트넷에서 알파 릴리즈를 시작할 때 부터 버그 현상금 프로그램을 제공하라.
  - 여러 단계로 배포하고, 각 단계에서, 적합한 테스트가 있었는지 반드시 확인하라.

- **스마트 컨트랙트를 단순하게 유지하라**. 복잡도의 증가는 오류의 위험도를 높힌다.
  - 스마트 컨트랙트의 논리가 간결한지 확인하라.
  - 컨트랙트와 기능들이 모듈화 되어 있는지 확인하라.
  - 이미 넓게 채택되어 있는 컨트랙트나 툴을 이용하라. (예를 들면, 난수 발생기를 당신이 작성하지 말아라.)
  - 가능하다면 명료성이 성능보다 더 중요하다.
  - 블록체인 기술을 오직 당신 시스템의 탈중앙화된 부분에만 이용하라.

- **업데이트 상태를 유지하라**. 리소스를 공개하여 최신 보안 기술을 이용하고 있는지 확인하라.
  - 어떠한 새로운 취약점이 발견되면 당신의 스마트 컨트랙트를 확인하라.
  - 가능한 때에 라이브러리나 툴을 가능한 한 빠르게 업데이트 하라.
  - 최신의 보안 기술을 이용하라.

- **블록체인의 특징을 명확히 이해하라**. 비록 당신의 예전 프로그래밍 경험이 스마트 컨트랙트 개발에 여전히 적용 가능하더라도, 아직 주목해야 할 함정이 있다:
  - `require_recipient(account_name name)`는 알림기능을 작동시킬 것이고, (만약 `name` 계정이 컨트랙트를 이미 설치했다면) 동일한 이름으로 `name` 컨트랙트 내부의 함수를 호출한다. [공식문서 확인하기](https://developers.eos.io/eosio-cpp/v1.2.0/reference#section-require_recipient)  

## [알려진 취약점](#알려진-취약점)

### [산술 오버플로우](#산술-오버플로우)

산술 연산을 할 때, 경계값들을 확인하지 않으면 값의 오버플로우를 유발하게 되고, 사용자의 자산 손실을 초래한다.

#### [취약점 예시](#취약점-예시)

취약 코드: `batchtransfer` 일괄 전송

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

    uint64_t amount = balance * 4; //곱셈 오버플로우

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

#### [방어 방법](#방어-방법)
가능한 순간까지, 연산을 위하여 `balance`를 추출하기 보다 `asset` 구조를 이용하라.

#### [실제 사례](#실제-사례)

- [【EOS Fomo3D 게임을 플레이하지 마시오】늑대 게임은 오버플로우 공격을 당했고 죽어 간다](https://bihu.com/article/995093)

### [권한 확인](#권한-확인)

관련된 연산을 할 때, 함수로 전달된 파라미터들이 실제 호출자와 일치하는지 부디 엄격하게 확인하고, 권한 확인을 위해서 `require_auth`를 이용하라.

#### [취약점 예시](#취약점-예시-1)

취약 코드：`transfer`

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

#### [방어 방법](#방어-방법-1)

자산 전송 계정과 호출 계정이 동일한지 확인하기 위하여 `require_auth( from )` 방법을 이용하라.

#### [실제 사례](#실제-사례-1)

없음

### [apply 확인](#apply-확인)

컨트랙트 호출 과정에서, 각각의 액션과 코드가 관련 요구사항을 만족하는지 확인하라.

#### [취약점 예시](#취약점-예시-2)

취약 코드：

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

#### [방어 방법](#방어-방법-2)

아래의 코드를 사용하라:

```
if( ((code == self  && action != N(transfer) ) || (code == N(eosio.token) && action == N(transfer)) || action == N(onerror)) ) { }
```
비정상적이고 잘못된 호출을 피하기 위해서 각각의 핵심 액션과 코드를 요구사항에 맞도록 묶어라.

#### [실제 사례](#실제-사례-2)

[EOSBet 전송 해킹 설명](https://medium.com/@eosbetcasino/eosbet-transfer-hack-statement-31a3be4f5dcf)

### [transfer 허위 통보](#transfer-허위-통보)

`require_recipient`에 의하여 시작되는 알림기능을 처리할 때, `transfer.to`가 `_self`인 것을 확인하라.

#### [취약점 예시](#취약점-예시-3)

취약 코드：

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

#### [방어 방법](#방어-방법-3)

다음을 추가하라.

```
if (transfer_data.to != _self) return;
```

#### [실제 사례](#실제-사례-3)

- [EOS DApp 재충전 "오류 통보" 취약점 분석](https://mp.weixin.qq.com/s/8hg-Ykj0RmqQ69gWbVwsyg)

### [난수 사례](#난수-사례)

난수 발생 알고리즘은 조작가능하거나 예측가능한 시드를 도입해서는 안된다.

#### [취약점 예시](#취약점-예시-4)

취약 코드：

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
    //공격자는 inline_action을 통하여 total_eos을 변경할 수 있고, 결과를 제어할 수 있음
    total_eos = pool_eos + ram_eos + betdiceadmin_eos + newdexpocket_eos + chintailease_eos + eosbiggame44_eos;
    auto mixd = tapos_block_prefix() * tapos_block_num() + name + game_id - current_time() + total_eos.amount;
    const char *mixedChar = reinterpret_cast<const char *>(&mixd);

    checksum256 result;
    sha256((char *)mixedChar, sizeof(mixedChar), &result);

    uint64_t random_num = *(uint64_t *)(&result.hash[0]) + *(uint64_t *)(&result.hash[8]) + *(uint64_t *)(&result.hash[16]) + *(uint64_t *)(&result.hash[24]);
    return (uint8_t)(random_num % 100 + 1);
}
```

#### [방어 방법](#방어-방법-4)

진정한 난수는 EOS에서 생성될 수 없다. 난수 클래스 응용프로그램을 디자인 하는 경우 공식 예제를 참조하는 것을 추천한다.

- [Randomization in Contracts](https://developers.eos.io/eosio-cpp/docs/random-number-generation)


#### [실제 사례](#실제-사례-4)

- [SlowMist의 경고: 잘 알려진 DApp EOSDice가 난수 문제 때문에 또다시 해킹당합니다.](http://www.chaindd.com/nictation/3140025.html)


### [롤백 공격](#롤백-공격)

- 기법 1: 트랜잭션의 (합산된 수량, 계정 잔고, 테이블 기록, 난수 계산결과 등과 같은) 수행 결과를 감지하고, 결과가 특정 조건을 만족하면 `eosio_assert`를 호출하여, 현재의 트랜잭션이 롤백되지 못하게 한다.
- 기법 2: 수퍼-노드 블랙리스트에 올라있는 계정으로 트랜잭션을 시작하여 일반 노드가 반응하도록 기만할 수 있지만, 트랜잭션은 패키지 되지 않을 것이다.

#### [취약점 예시](#취약점-예시-5)

취약 일반 사항:

- 도박 게임에 베팅을 한 이후, 추첨은 시작되고 전달될 것이다. 악의적인 컨트랙트는 `inline_action`을 이용하여 잔고가 증가하는 것을 감지할 수 있고, 그러므로 실패한 도박의 경우 롤백 한다.
- 도박 게임에 베팅을 한 이후, 추첨의 결과가 양식에 적히게 된다. 악의적인 컨트랙트는 `inline_action`을 이용하여 잔고가 증가하는 것을 감지할 수 있고, 그러므로 실패한 도박의 경우 롤백 한다.
- 도박 게임 추첨 결과와 게임내 추첨 번호는 연관되어 있다. 악의적인 컨트랙트는 작은 베팅의 트랜잭션과 큰 베팅의 트랜잭션을 같은 시간에 발생시킬 수 있고, 작은 양의 승리가 발생했을 때 트랜잭션을 롤백하여, 승리가 가능한 추첨 번호를 큰 베팅으로 "전달"하는 목적을 달성하게 된다.
- 도박 게임 추첨이 베팅 트랜잭션과 연관되어 있지 않고, 공격자는 블랙리스트 계정이나 악의적인 컨트랙트로 베팅 트랜잭션을 롤백 할 수 있다.

#### [방어 방법](#방어-방법-5)

- 자산 및 영수증 전송에 `defer action`을 이용한다.
- 순서 의존성과 같은 노출되는 함수의 의존성을 확립하라. 노출될 때 블록체인에 기록이 존재하는지 확인하라. 노출되는 함수가 노드 서버에서 정상적으로 동작했더라도, 기록이 bp에서 롤백 되므로, 해당되는 기록도 롤백 될 것이다.

#### [실제 사례](#실제-사례-5)
- [EOS 블랙리스트 롤백 공격](https://mp.weixin.qq.com/s/WyZ4j3O68qfN5IOvjx3MOg)

## [참고자료](#참고자료)

- [保管好私钥就安全了吗？注意隐藏在EOS DAPP中的安全隐患](https://zhuanlan.zhihu.com/p/40625180)
- [漏洞详解|恶意 EOS 合约存在吞噬用户 RAM 的安全风险](https://zhuanlan.zhihu.com/p/40469719)
- [How EOSBET attacked by aabbccddeefg](https://www.reddit.com/r/eos/comments/9fpcik/how_eosbet_attacked_by_aabbccddeefg/)
- [BET被黑客攻击始末，实锤还原作案现场和攻击手段](https://github.com/ganjingcun/bet-death-causes/blob/master/README.md)
- [累计薅走数百万，EOS Dapps已成黑客提款机？](https://mp.weixin.qq.com/s/74ggygC3nbDihLkobXOW2w)

## [감사의글](#감사의글)

- [CryptoKylin Workgroup](https://github.com/cryptokylin)
- eosiofans
- Kai Jing(荆凯)
- 星魂
- 岛娘
- Yu Zhao(赵余)
- 字符
