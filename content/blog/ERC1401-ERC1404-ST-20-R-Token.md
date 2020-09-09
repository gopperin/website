---
title: "Security Token标准的对比"
date: 2018-11-12T14:51:12+06:00
author: 糙米薏仁汤(引)
# image: images/blog/blog-post-3.jpg
---

> 1.概要

随着对区块链生态及数字货币市场的监管越发严格，ICO的狂野时代已经成为了过去时，数字货币的发行很可能受证券法的监管。在这一背景下，为了让企业能通过区块链向投资者提供符合法规监管的金融产品，而又不违反证券法，证券型代币发行（STO）解决方案应运而生。

与实用型代币(Utility Token)不同的是，证券型代币(Security Token)表示对资产的部分或完全所有权，公司、房地产甚至知识产权的股票，都可以用证券型代币来表示。证券型代币的好处，不仅适用于区块链融资，它还有可能改变传统的纸股范式，从而提高效率以及改善分配。例如，智能合约的很多应用，可以和证券型代币一起使用，以优化公司治理中的投票表决，提高其透明度。

然而，如果没有标准，监管机构、开发者、KYC供应商、投资者、发行商、钱包以及交易所，就无法在同一个框架中协同工作。目前，市场上已出现了多种STO标准尝试：其中有来自Polymath团队的ST-20，Harbor团队的R-Token，还有来自社区的提案——ERC1400/ERC1410及ERC1404标准等。

> 2.基本情况

代号	标准名称	作者	阶段	创建时间
ST-20	ST-20 Standard	Polymath团队	—	—
R-Token	Regulated Token Standard	Harbor团队	—	—
ERC-1404	Simple Restricted Token Standard	Ron Gierlach, James Poole, Mason Borda, Lawson Baker	Draft	2018-07-27
ERC-1400	Security Token Standard	Adam Dossa, Pablo Ruiz, Fabian Vogelsteller, Stephane Gosselin	Draft	2018-09-09

> 3.标准的主要内容与差异
我们从ST-20，R-Token，ERC1404，ERC1400标准内容切入，结合业务场景研究设计的目的，再比较各个标准之间的差异，思索在证券化代币形式标准化的过程中，所面临的难题及取舍。

3.1 代币性质
这几个标准在代币性质上，ERC-1400与其他三个的差异最大。我们先了解下Fungible Token的概念。

3.1.1 Fungible Token 和 Non-fungible Token
Fungible Token称为同质化代币，也可以叫做可互换代币，典型的如同ERC-20标准的代币，token不存在附加的解释，每个单位相同token价值完全相同，token持有者可以随意交换，拆分整合。而ERC-721标准的代币，每个单位的token均有不同的ID，不同ID可以有不同的解释，设计上认为每个token都是不同的，不能随意按单位进行交换，这就是Non-fungible Token（非同质化代币/不可互换代币）。

3.1.2 ERC-1410标准与tranche
ERC-1400标准依赖于ERC-1410标准，在设计上，将token的余额通过一个叫做tranche的属性，划分成不同的部分。可以对tranche做出不同解释，操作上进行不同的限制（例如：某些操作只限指定的tranche，某些操作优先消耗指定tranche下的token），这有些Non-fungible Token的概念，但也存在不同：tranche相同的token价值相同，是可以随意置换的。结合了Fungible Token和Non-fungible Token两者，故称为Partially-Fungible Token（部分可互换代币）。

3.1.3 分类与场景
ST-20，R-Token，ERC-1404都是直接依赖于ERC-20标准的，很明显，这三个都属于Fungible Token。ERC-1400依赖于ERC-1410，故属于Partially-Fungible Token。

实际应用场景中，同一家企业发行的证券可能是存在差异的，如限售股/非限售股，优先股/普通股，原始股/增发股，这些不同性质的证券在分红，票权，流通性上不尽相同，其性质也可能在某个阶段发生转变，不同性质的证券在投资人眼里的价值可能是不同的。

ERC-1404，ST-20，R-Token这类同质化代币，要在合约层面体现出证券性质上的差异，只能依据地址进行限制，设置维护各种规则名单（比如：KYC/AML名单，出入账限制名单，冻结名单，最小保留额等）来做限制。这种设计存在局限性：单一地址在同一时刻无法持有多种性质不同的代币。如果要在合约层面适用于上述复杂的业务，可以通过多个合约管理不同性质的代币，这在完整性上大打折扣，开发的复杂度也会加大。再看看ERC-1400，在合约内部就将持有的token划分成不同的tranche，这样不仅能对地址做限制，还能对tranche做限制，诸如配股增发，投票表决，分红除息等复杂功能都可以通过在合约层定义tranche来实现差异化和差异转变。ERC-1400的设计也使得开发者，交易所接入的难度加大，所以，根据实际的业务场景选择适合的证券型代币标准才是最应当的。

3.2 接口设计
考虑到证券相关的业务场景，证券型代币的合约标准做了对提供转账限制的定义，在执行转账前先进行限制判断，并提供对转账限制判断结果的可读解释，从而在合约层面实现诸如锁仓，KYC/AML验证，出入账冻结等功能。

3.2.1 转账限制的判断函数
在这几个证券型代币标准中都声明了一个函数，实现对转账限制的判断逻辑，返回结果的布尔值或是状态码。在执行转账操作时需要先调用判断函数，判断通过则继续执行，失败则返回相应状态码，取消转账，它允许函数的调用者知道转账失败的原因并将其报告给相关方。

/* ST-20 */
function verifyTransfer(address _from, address _to, uint256 _amount) public view returns (bool success);

/* R-Token */
function check(address _token, address _spender, address _from, address _to, uint256 _amount) public returns (uint8);

/* ERC-1404 */
function detectTransferRestriction (address from, address to, uint256 value) public view returns (uint8);

/* ERC-1400 */
function canSend(address _from, address _to, bytes32 _tranche, uint256 _amount, bytes _data) external view returns (byte, bytes32, bytes32);
3.2.2 判断结果的解释函数
用于对转账限制的判断函数返回的状态码做出可读的解释，通过标准化的信息，使相关应用的开发者能有效地向用户报错。

/* ST-20标准的设计中，转账限制的判断函数仅返回布尔值，故缺失对状态码的解释函数 */

/* R-Token */
function messageForReason(uint8 _reason) public view returns (string);

/* ERC-1404 */
function messageForTransferRestriction (uint8 restrictionCode) public view returns (string);

/* ERC-1400，判断函数canSend返回一个遵循ERC-1066标准的ESC (Ethereum Status Code，以太坊状态码)，
还有一个可以用于定义程序指定原因码及额外详细信息（例如，执行发送操作的转账限制无效）的 bytes32 参数。
所以ERC-1400还依赖于ERC-1066标准 */
3.2.3 转账函数
ERC-1404，ST-20，R-Token继承自ERC-20，所以转账函数与ERC-20中相同——transfer和transferFrom, 实现上进行重写即可（加入对转账限制的判断函数的调用）。

实现示例:

/* implement in the instance of R-Token */
  function transfer(address _to, uint256 _value) public returns (bool) {
    if (_check(msg.sender, _to, _value)) {
      return super.transfer(_to, _value);
    } else {
      return false;
    }
  }

  function transferFrom(address _from, address _to, uint256 _value) public returns (bool) {
    if (_check(_from, _to, _value)) {
      return super.transferFrom(_from, _to, _value);
    } else {
      return false;
    }
  }
  
  
/* implement in the instance of ERC-1404 */ 
    modifier notRestricted (address from, address to, uint256 value) {
        uint8 restrictionCode = detectTransferRestriction(from, to, value);
        require(restrictionCode == SUCCESS_CODE, messageForTransferRestriction(restrictionCode));
        _;
    }

    function transfer (address to, uint256 value)
        public
        notRestricted(msg.sender, to, value)
        returns (bool success)
    {
        success = super.transfer(to, value);
    }
    
    function transferFrom (address from, address to, uint256 value)
        public
        notRestricted(from, to, value)
        returns (bool success)
    {
        success = super.transferFrom(from, to, value);
    }
对于ERC-1400而言，因为有tranche的存在，不光是转账，整个合约的接口设计上要做出大的改变。这里只示例transfer对应的函数定义：

    // 定义在ERC-1410标准中
    function sendByTranche(bytes32 _tranche, address _to, uint256 _amount, bytes _data) external returns (bytes32);
    function sendByTranches(bytes32[] _tranches, address[] _tos, uint256[] _amounts, bytes _data) external returns (bytes32[]);
3.3 兼容性
因为ERC-20标准在行业里最为通用，这里的兼容性我们主要考虑各个证券型代币标准对ERC-20的向后兼容。

ERC-1404，ST-20，R-Token在接口设计上派生于ERC-20，其合约本身就要对ERC-20标准具有的transfer和transferFrom函数进行重写，可以说是向后完全兼容ERC-20。而ERC-1400会更麻烦一些，ERC1400继承自ERC777标准，其中的定义的转账函数并不是ERC-20标准中的transfer和transferFrom，如果要向后兼容ERC-20所以必须要额外实现ERC-20标准中具有的函数，且来自两个标准的状态变更函数在实现上最好进行解耦，彼此独立地操作。此外，相应event也要做变动。兼容ERC-20标准后，便可快速接入交易所，钱包等应用。

3.4 其他差异
ERC-1404，ST-20，R-Token在设计上大体相同，唯一明显不同的是R-Token将转账限制的判断函数和判断结果的解释函数放在了Regulator Service合约中，便于升级版本，同时使用Service Registry合约作为注册器，保存当前最新版本的Regulator Service地址。主体合约R-token先要访问注册器合约Service Registry拿到Regulator Service的地址，再调用最新版本的Regulator Service中的转账限制的判断函数。整个过程如下图所示：
R-Token.png
ERC-1400除了因tranche的引入而在接口设计上与另外三种标准明显不同之外，因其设计思路上主张尽量在合约层对证券相关的业务提供支持，比如ERC-1400考虑到实际的证券流通需要链上和链外的参与者之间更复杂的交互，因此该标准要具有能力进行强制转移，用于法律诉讼，资金追回等。

> 4.总结
文章提到的四个证券化代币标准中，ST-20和R-Token分别是Polymath和Harbor自行设计，主要依靠自身进行推广。如果能打造出证券代币化的成功案例，势必会获得认可。ERC-1404和ERC-1400则是通过EIP提议，广泛采纳社区意见，共同制定标准。但目前都处于Draft阶段。一项提案从起草提议，到进入最终调用（Last Call）阶段，通常需要经历很长的开发、编辑、实现以及漏洞修复周期。这些提案可能只是证券代币化之路的一个开始，随着行业各方陆续参与，一定出现更多的的标准。

作者：糙米薏仁汤
链接：https://www.jianshu.com/p/da778c4b0031
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。