> 作为一名有轻度阅读障碍并且英语渣，最近学会了一种看专业文档的方法——将文档最重要的部分选翻译出来再理解。翻译的过程首先可以帮助你专注理解文章的内容，而若非深刻理解，很难准确的翻译成中文。
>
> 原来计划先去分析最主流的DeFi项目合约代码和以太坊代码，再来分析零知识证明相关的项目，但最近因为工作涉及到了这块，所以想改变一下顺序。
>
> 首先先从zk Rollup开始，zkSync主要涉及三个部分：zk Rollup ，zinc 和 zkPorter。三个部分都将展开介绍，在开始之前我想先最MatterLabs的几篇文章进行翻译（侵删）。



# zkRollup 第一站——读 zkSync

## 1. 介绍zksync：以太坊大规模应用缺失的链接（Introducing zkSync: the missing link to mass adoption of Ethereum）

[原文链接](https://medium.com/matter-labs/introducing-zk-sync-the-missing-link-to-mass-adoption-of-ethereum-14c9cea83f58)

### 摘要

在公共区块链上扩容问题的一个成功解决方案已经不仅仅是实现了交易的高吞吐量。它还必须被定义为系统在不破坏去中心化的同时满足百万用户的需求的能力。实现大规模加密应用的前提包括高速，低成本，顺滑的UX和隐私。

在没有技术突破的情况下，现有的扩展解决方案不得不对其中一项或多个要求作出重大妥协。幸运的是，最近在[零知识证明](https://github.com/matter-labs/awesome-zero-knowledge-proofs)方面取得的进展为解决这一问题开辟了全新的可能性。



今天，我们[Matter Labs](https://matter-labs.io/) 兴奋得宣布我们在[zkSync](https://zksync.io/)上的愿景：基于 ZK Rollup 的以太坊的无信任扩展和隐私解决方案，重点是卓越的用户和开发人员体验。我们还自豪地宣布为 zkSync 推出测试网。

ZK Sync 旨在为以太坊带来每秒数千笔交易 （TPS） 的 VISA 规模吞吐量，同时保持资金与基础 L1账户一样安全，并保持高度的审查阻力。协议的另一个重要方面是其超低延迟：ZK Sync 交易将提供**即时经济终结**.

我们认同精益设计理念和逐步的协议演变，在每个步骤中逐一引入功能，为用户带来最切实的价值。这就是为什么我们从基本原理（安全性）开始，首先关注基本的可扩展性（token转账），然后是可编程性（智能合同），最后是隐私 。

> ZK Sync特性一撇
>
> * 安全性与L1相当
> * VISA级别的吞吐量
> * 亚秒级交易确认
> * 抗审查和抗DoS攻击
> * 保证隐私的智能合约



### 区块链扩容的最难问题

如今，在实践中，加密仍然主要用于投机。如果没有真正大规模使用，那么神奇的互联网货币，DeFi，Web3.0 和所有其他有前途的区块链穿衣的价值主张将很大程度上得不到实现。

可扩展性不仅仅是交易吞吐量，而是区块链系统满足数百万用户需求的整体准备。

 我们来思考一下将区块链革命扩大到大众的三个最困难的问题。

#### 挑战 #1：保持去中心

真实世界应用需要的交易吞吐量比当今最分散的区块链所能处理的交易量要高一个数量级。比特币能够做到7个TPS，以太坊是15个TPS——而仅VISA就平均处理了令人生畏的2000个TPS 。

但是比特币和以太坊的缓慢是一个特性，而非bug！几乎不可能通过减少验证节点的数量来提高速度。这两个领先的区块链网络的大量的全节点是他们引以为傲的最重要资产。

另一种流行的可扩展性方法是要求每个验证器只检查相关区块链流量的一部分，而不是全部。但这不可避免地引入了额外的信任假设，并将此类系统置于非常不稳定的游戏理论基础之上。



#### 挑战 #2: 启用隐私

大多数人会对将他们大部分财富转移到一个公开曝光的玻璃盒感到不舒服。危险地区的居民——例如委内瑞拉——如果接受者能够立即知道他们有多少钱，他们不太可能向当地商人支付加密费用。如果这些支付可能与其真实身份相关，创建或消费色情等淫秽内容（如色情内容）的人不太可能使用加密货币作为[PayPal的替代品](https://www.reuters.com/article/us-paypal-pornhub/paypal-halts-payment-support-to-pornhub-models-idUSKBN1XO2SV)。

此外，在没有链上保密的情况下，隐私法规如GDPR和CCPA，将驱使普通企业从公共区块链专项更中心化的支付和金融中心，使得我们日益无现金的社会变成监控噩梦。

隐私绝对是大规模应用的先决条件。

基于以下几个因素，在公共区块链上实现隐私尤为困难：

1. 隐私必须作为整体协议功能在默认情况下打开。引用Vitalik Buterin的话：“如果你的隐私模型有一个中等匿名集合，它实际只是一个小型的匿名集合。如果您的隐私模型是一个小型的匿名集合，它的匿名集合其实只为1。只有全局匿名集合才能真正安全。”
2. 为了实现默认隐私，隐私交易成本必须非常低，尽管这要增加大量的计算消耗。
3. 隐私模型必须支持可编程性，因为实际的使用案例需要的不仅仅是转账，它还需要账户恢复，多签，支出限制等。

#### 挑战 #3：满足用户体验期望

现实是残酷的：产品经理非常清楚，用户通常更喜欢相对容易更轻量级的体验，与替代方案相比，而他们往往（忽略[长尾风险]((https://www.goodreads.com/book/show/242472.The_Black_Swan))）。这需要一种[非凡的力量](https://medium.com/@joeantenucci/peter-thiels-4-essential-rules-for-creating-a-dominant-company-7f2f3024a43) 去哄骗用户从熟悉的东西切换到新的东西上。对有些人来说，加密货币的价值主张（激进的自我所有权，审查阻力，健全的钱）是足够的。但是这些人可能已经在船上了。我们需要数百万，如果不是数十亿的话，来达到加密货币实现它承诺的必要规模。

为了吸引数百万主流用户，我们需要为他们提供用户体验，这个不只要符合他们的期望，更要超出预期。我们必须这样做来提供全新的可能性，同时保持人们已经习惯的所有传统网络产品所剧本的方便的特性，一切都必须快速，简单，直观且耐错。



### zkSync的承诺：无信任，保密和飞快



在技术编写中，我们将探讨协议的高层架构，设计选择和属性。

#### 1. 安全性：ZK Rollup的根基

ZK Sync 是建立在 ZK Rollup的概念之上的。

简言之，ZK Rollup是一个二层扩容解决方案其中所有的资金都由主链上的一个智能合约持有，而计算和存储都在链下执行。对于每一个Rollup块，在主链合约上生成并验证状态转换零知识证明（SNARK）。此SNARK包括Rollup区块中每笔交易的有效性证明。此外，每个块的公共数据都作为便宜的calldata发布在主网上。

这个架构提供了以下的保证：

1. Rollup验证器永远不会损坏状态或窃取资金（不像侧链）。
2. 由于数据可用性，即使验证器停止合作，用户始终可以从Rollup中恢复资金（不像 Plasma）。
3. 由于有效性证明，用户和单个受信任的第三方都不需要在线监控Rollup区块来阻止作弊行为（不像欺诈证明系统，例如支付通道或者optimistic rollups）。这篇[优秀的文章](https://medium.com/starkware/validity-proofs-vs-fraud-proofs-strike-back-4d0bf90eed15)深入探讨了有效性证明在欺诈证据上的压倒性好处。

换句话说，ZK Rollup严格继承了L1的安全保证。这一点，加上以太坊社区和先有基础设施的丰富性，是我们决定专注于L2解决方案而不是试图构建我们自己的L1的决定性因素。

为了更好得了解这个概念，请参阅我们在 [ZCon1](https://www.youtube.com/watch?v=QyM9qdFKsEA) 和 [Dappcon](https://www.youtube.com/watch?v=CY-WaZ01BTw)的讨论，这是Matter Labs在零知识频道的一个[播客](https://www.zeroknowledge.fm/72?source=post_page-----6795d3226699----------------------)，还有我们早期 [技术解释](https://medium.com/matter-labs/introducing-matter-testnet-502fab5a6f17) 文章。好奇的读者还可以去[这篇博客](https://medium.com/matter-labs/optimistic-vs-zk-rollup-deep-dive-ea141e71e075) 关于ZK Rollup 和 Optimistic Rollup的不同。

继[来自以太坊基金会的赠款](https://medium.com/matter-labs/grant-from-the-ethereum-foundation-for-matter-labs-64338f3dd938)之后，Matter Labs 已经花费了过去一年的时间一直在研究ZK Rollup技术。自第一个原型推出来以后，我们已经完整地重写了架构和ZK电路。最新版本结合了我们从社区收到的反馈，并实现了各种可用性和性能改进。

> 总结：ZK Rollup
>
> * 完整无信任
> * 与底层L1（以太坊）一直的安全性保证
> * 首次确认后确定的以太坊结局



#### 2. 可用性：实时交易

我们预计ZK证明技术当前的发展将达到证明时间使得ZK Rollup块在一分钟内产生。一旦区块证明被提交到主链并被Rollup的智能合约验证，区块里的所有交易都最终确定，并且受L1重新组织的阻力保证。

但是在零售和在线支付中，计算是以太坊15秒的区块延迟也可能太长了。我们怎么做才能做得更好呢？

方法如下：在ZK Sync中，我们引入**即时交易收据**。

选择参与ZK Sync区块生产的验证器必须在主网上向 ZK Sync的智能合约提交安全保证金。验证人运行的共识为用户提供了秒以下的验证，即他们的交易将被包含在下一个 ZK Sync 块中，该块有2/3的共识参与者（按赌注加权）的绝对多数签名。

如果一个新的ZK Sync区块被生产并提交到主网上，它无法被撤回。但是，如果它没有包含承诺的交易，原始收入的签署人和新区块的签署者的交叉点的安全保证金将被削减。这个交叉口保证有超过1/3 的抵押。这保证了至少1/3的安全保证金可以被削减，只有恶意用户才会受惩罚。

部分被削减的资金将用于补偿交易接收者。其余的将被烧掉。

这个削减可以由用户自己或者签署原始交易收据的共识中的任意诚实参与者触发。后者将有一个自然的动机，称之为欺诈；如果他们参与随后的块生产，他们也可以被削减。因此，至少有一个诚实的参与者在共识中足以发现欺诈行为。

让我们检查一下这个协议的属性。我们将称一个零确认tx为一笔尚未发布到以太坊ZK Sync块的交易。

在ZK Sync上零确认交易的双花是唯一潜在问题是在非常短的时间窗口内发生直到这个块的证明被发布发到主网上。此外，恶意的验证器还必须诱使用户接受价值超过安全保证金1/6的零确认交易。

从买家和卖家的角度来看，零确认的交易是：

1. 即时的
2. 可能可逆，但仅仅在几分钟内
3. 只有在同时攻击数千名商户时才可逆，而不是逐一攻击。

这是一个在信用卡支付上的巨大的用户体验和安全改进！我们来从不同人的视角看看它：

* **有实物商品的在线商店** 能够即时向用户确认购买，但也可以免受攻击，因为它们将等待完整的确认后再发货
* **实物商店** 在处理较小金额时不易受到攻击。即使你以即时交易收据出售macbook，在不同地方也需要数千名协作的物理攻击者和大多数验证者串通才能致使亏损。

但是，我们再深入挖掘一下。为了量化风险，债券提供的经济担保可以与工作证明区块链提供的结算保证进行比较（参见[尼克·卡特的这篇伟大的著作](https://medium.com/@nic__carter/its-the-settlement-assurances-stupid-5dcd1c3f4e41)）。例如，Coinbase在考虑以太坊的最终存款结果之前需要35个交易确认。通过对从 AWS 租用的 GPU 进行 51% 的攻击 10 分钟来回退此交易的成本约为 60,000 美元。假定证券债券为数百万美元，相比而言回退ZK Sync收据的成本更昂贵。因而，即时收据通常是具备类似ETH甚至更好的经济最终属性。

需要注意的是，即时交易收据还可以防止ETH区块，因为他们的有效性是独立于以太坊的重构。以外，以太坊的结算保证与ZK Sync 的结算保证相融合。

> 总结：实时交易
>
> * 亚秒级交易确认并且与以太坊相近的经济最终属性
> * 几秒后确定的以太坊结局



#### 3. 活力：审查阻力和Dos阻力

任何扩容解决方案的必然属性是大多数的用户无法参与到验证所有的交易量上来。这就导致了需要在L2可扩展解决方案（Plasma 或者 Rollups, Lightning hubs等的验证者）中的专门角色。这就增加了对这些角色在安全性和性能要求构成了机制和审查的风险。

ZK 同步设计通过引入两个不同的角色来解决这个问题：验证器和守护者。

##### 验证者

验证者负责将交易打包成块，并为其生成零知识证明。他们参与协商，因此必须为即时交易收据贡献一部分安全保证金。他们的节点必须在具有良好互联网带宽环境下运行。或者他们可以选择在不安全的云上生成零知识证明。

验证者将获得交易费用作为奖励，这些费用可以再交易的任何令牌中支付（为最终用户提供最大便利）。

为了保持ZK Sync共识的速度，在任何时刻（30到100之间，但必须进行剖析）只允许有限数量的验证者。但是回想一下，ZK Rollup验证者是完全无信任的 。在ZK Sync 中，作恶的验证者既不能危机系统的安全性，也不能欺骗诚实的验证者降低条件。因此，与optimistic rollups不同，一小部分验证者可以经常被守护者轮换。同时，只要超过2/3的被提名验证者诚实和可操作，协商一致意见的活力就得到保证。

##### 守护者

守护者包括大多数ZK Sync的token持有人，他们持有代币份额以提名验证者。守护者的目的是监控点对点交易忘了。检测审查行为，并确保被捕获的审查人员未提名。守护者的动机是通过确保ZK Sync保持对Dos 和审查抵抗力来确保它的抵押的价值。

尽管将投票密钥保持在线状态，ZK Sync的守护者也从未面临被削减或者被盗的风险（所有权密钥可以保持在冷存储中）。他们可能也选择去监控流量的一小部分。因而，他们的节点可以被运行在普通笔记本电脑或者云服务器上，而无需专门的验证器服务。

守护者从ZK Sync原始token标记的验证者那里获得手续费的奖励。他们的收益与抵押长期锁定，以激励长期ZK Synctoken价值优先于短期回报。

> 总结：活力
>
> * 两个角色：验证者和守护者，都是由交易费激励
> * 验证者运行共识并生成证明
> * 守护者使用普通硬件去审查制度



#### 4. 可编程性和隐私：基石

实现高效的可编程性和隐私性是ZK Sync的愿景。它需要为适当的零知识证明系统和智能合约编程框架进行可靠的设计和实现。

##### 4.1 RedShift：透明通用的SNARK

实现基于零知识证明的智能合约最大的障碍（无论是透明还是隐私保护而言）是缺乏具有递归成分的有效通用 ZK 证明系统。Groth16 是当时效率最高的 ZK SNARK，需要针对应用程序的可信设置，并且在应用递归时会失去很多效率。另一方面，基于FRI的STARKs需要高度专业化的技能来构建和缺乏任意通用电路的有效递归组合。

这是我们在 [RedShift](https://eprint.iacr.org/2019/1400)上工作的主要动机之一：一个新的透明、高效和完全简洁的SNARK，源自我们基于FRI的多多项式承诺。我们目前正在整合同行评审和社区反馈，然后部署 Redshift 作为 ZK Sync。

Redshift是一种通用的SNARK，它允许我们使用它将任意程序方便地转换为可证明的 ZK 电路。异质电路（如不同的智能合约）可以递归在一个 SNARK 中。RedShift 是后量子安全可靠的，因为它完全依赖于抗碰撞的哈希函数。

> 总结：Redshift
>
> * 透明：无可信任设置需求
> * 合理的抗量子安全：基于久经沙场的密码学
> * 广泛的：适用于通用程序（与STARKs相反）



##### 4.2 Zinc：零知识证明智能合约框架

在ZK Sync可编程模型的设计中，我们致力于提交野心勃勃的正交目标的组合：

* 高可扩展性
- 支持公共和私人智能合约
- 最重要的是：平坦的学习曲线和轻松的发展

许多未完成的项目都分享了其中的一些目标，但没有一个项目同时承诺实现所有的这些目标。例如[ZkVM](https://github.com/interstellar/zkvm)为通用机密智能合同提供了一台虚拟机器，但基于Bulletproofs，不支持简洁的证明聚合。[ZEXE](https://github.com/scipr-lab/zexe) 拥有出色的隐私保护设计，但需要深入了解零知识电路的具体细节和权衡，更广泛的程序员圈子进入门槛非常高。其他更简单的 ZK 编程框架缺乏安全智能合约开发所需的表现力或功能 。

这就致使我们去创建Zinc：一个安全、简单和高效的编程框架和基于 VM 的运行时间环境，专为基于 ZKP 的智能合约而设计的。

SyncVM 的主要设计重点是安全性和开发人员友好性。定义合同的编程语言紧跟简化的 Rust 语法，智能合约编程元素借用了"Solidity"和"Libra的Move"。它并不要求开发人员深入了解 ZKP 的具体细节，以编写高效和安全的程序。事实上，Zinc可以在一天内由具有Rust, Solidity, C++或类似编程语言背景的开发人员学习。

> 总结： Zinc
>
> * 安全和可传递的编程框架
> * 零知识证明代理生成的沙盒VM
> * 专注于轻松发展：对于Solidity程序员来说可一天学会



## 2. Optimistic vs. ZK Rollup

[原文链接](https://medium.com/matter-labs/optimistic-vs-zk-rollup-deep-dive-ea141e71e075)

### TL;DR: brief summary

Optimistic Rollup is a promising technology for scaling general-purpose smart contracts on Ethereum in the near term. If built relatively quickly, it can offer an easy way to migrate existing dapps and services with a reasonable degree of security/scalability tradeoffs. This will enable ETH 1.0 to keep up with growing demand.

ZK Rollup is a more sophisticated technology. [It can be used for token transfers](http://demo.matter-labs.io/) and specialized applications today. However, it will take a little longer to implement general-purpose smart contracts, and even more research work is required to efficiently wrap EVM in zero-knowledge proofs. However, once ZK Rollup is fully developed, all existing Ethereum dapps and services will be able to smoothly migrate to it without much effort.

ZK Rollup will fix several fundamental issues with Optimistic Rollup:

- Eliminate a nasty tail risk: theft of funds from OR via intricate yet viable attack vectors;
- Reduce withdrawal times from 1–2 weeks to a few minutes;
- Enable fast tx confirmations and exits in practically unlimited volumes;
- Introduce privacy by default.

Optimistic Rollup is great news for ZK Rollup. The transition to L2 scaling requires significant changes in wallets, oracles, dapps and user habits. Optimistic Rollup can help to prepare the ecosystem for this move, bringing scale to those dapps that cannot yet be built on ZK Rollup today. This will give ZK Rollup time to mature and make its adoption completely seamless, while maintaining Ethereum’s growth momentum.

### **Rollup 101**

#### What is a Rollup?

A Rollup is a Layer-2 scaling solution similar to Plasma: a single mainchain contract holds all funds and a succinct cryptographic commitment to a larger “sidechain” state (usually a Merkle tree of accounts, balances and their states). The sidechain state is maintained by users and operators offchain, without reliance on L1 storage (which is the source of the biggest scalability win).

What differentiates Rollup from Plasma is that it solves Plasma’s huge problem — data availability — by publishing some data for each transaction via the L1 network (in Ethereum specifically tx CALLDATA is used for this purpose). Thousands of transactions can thus be bundled up (rolled up) together in a single Rollup block. While this approach grows strictly linear in costs (O(n) of the number of transactions), it provides a practical 100-fold improvement in throughput, because CALLDATA is way cheaper than L1 storage and computation.

Rollup has been repeatedly [endorsed by Vitalik Buterin as his favorite Layer-2 scaling solution](https://www.trustnodes.com/2019/08/22/vitalik-buterin-is-more-and-more-pessimistic-about-scaling-through-second-layers).

Depending on how the correctness of state transition is guaranteed, there are two Rollup flavours: ZK Rollup and Optimistic Rollup. A brief history of both solutions is nicely [presented here](https://medium.com/@kimiwu/zk-rollup-optimistic-rollup-70c01295231b).

#### What is a ZK-Rollup (ZKR)?

In a ZK-Rollup, operator(s) must generate a succinct Zero-Knowledge Proof (SNARK) for every state transition, which is verified by the Rollup contract on the mainchain. This SNARK proves that there exists a series of transactions, correctly signed by owners, which update the account balances in the correct way, and which lead from the old Merkle root to the new one. It is thus impossible for the operators to commit an invalid or manipulated state.

More technical details can be found [here](https://ethresear.ch/t/on-chain-scaling-to-potentially-500-tx-sec-through-mass-tx-validation/3477) and [here](https://medium.com/matter-labs/introducing-matter-testnet-502fab5a6f17). You can play around with the [live demo of Matter Labs’ ZK Rollup for ERC-20 token transfers](https://demo.matter-labs.io/).

#### What is an Optimistic Rollup (OR)?

In an [Optimistic Rollup](https://medium.com/plasma-group/ethereum-smart-contracts-in-l2-optimistic-rollup-2c1cef2ec537), the new state root is published by operator(s) without being checked every time by the Rollup smart contract. Instead, everybody hopes that the state transition is correct. However, if an incorrect state transition is published, other operators or users (who MUST observe what’s going on in the L1 Rollup contract, executing every single transaction) will be able to point to the invalid transaction and revert the incorrect block, slashing malicious operators.

The idea of OR was originally [introduced by John Adler](https://ethresear.ch/t/minimal-viable-merged-consensus/5617). Readers can find more details in an [AMA session on Optimistic Rollup](https://docs.google.com/document/d/17f9JeeSW_M3PrMmvMsmArTDEqOkSKEZQySNKfpewIP4/edit). Kudos to the Plasma Group for the great work!

### **Let’s compare!**

#### Flexibility: general-purpose computation

**BIG UPDATE 2021:** This section is obsolete. ZK rollups can now [support EVM-compatibility](https://medium.com/matter-labs/zksync-2-0-roadmap-update-zkevm-testnet-in-may-mainnet-in-august-379c66995021)!

**Optimistic Rollup**

Although Optimistic Rollup could be used for specialized applications, the most important innovation of the [Plasma Group](https://plasma.group/) is the OVM: [Optimistic Virtual Machine](https://medium.com/plasma-group/introducing-the-ovm-db253287af50). OVM enables the implementation of arbitrary smart contract logic. Almost anything that is possible in Ethereum is also possible in the OVM, including composability of smart contracts. It can be based on EVM, EWASM or any other virtual machine.

The nice thing about OVM is that if used with EVM, it will support writing code in Solidity. Because of this, large parts of existing codebase can be ported onto OR with little effort.

It would be ideal if OVM could directly reuse existing EVM bytecode, but it’s probably not that simple. A proper implementation will require changes of the transaction data (CALLDATA) format and sophisticated Truebit/Plasma Leap style implementation of challenge/response protocol for fraud proofs. This is likely to lead to divergence from EVM to properly handle the edge cases, meaning that some work will still be required to adapt existing contracts for OVM.

Another challenge to implementation lies in the fact that fraud proofs for large blocks can require more gas than permitted by the L1 block gas limit. These fraud proofs must then be broken down into multiple ETH transactions.

**ZK Rollup**

All existing implementations of ZK-Rollup (including the one by yours truly) have so far focused only on specialized operations such as token transfers or atomic swaps. There are several major reasons for this.

First, there was no efficient technique for succinct recursive proof composition for different zero-knowledge proofs (ZKPs), which would be required to aggregate the execution of different smart contracts in a single block. The best we had was [Groth16](https://eprint.iacr.org/2016/260.pdf) over the [cycles of elliptic curves](https://arxiv.org/abs/1803.02067) (used by Coda), which required computation over long fields and would be totally inefficient for large computations.

Second, even if we had shorter fields, Groth16 would require a separate [trusted setup ceremony](https://github.com/matter-labs/awesome-zero-knowledge-proofs#multi-party-ceremony-mpc-for-trusted-setup) for each smart contract and for each new version! Obviously, this would be absolutely unrealistic.

The only efficient ZKP technique we had without trusted setup was [FRI-based STARKs](https://github.com/matter-labs/awesome-zero-knowledge-proofs#fri-starks). However, the verifier is succinct to only a limited class of problems (expressible as succinct arithmetic circuits). A STARK verifier must execute each constraint of the computational statement being proved at least once, which means we cannot iterate over a collection of heterogeneous smart contracts.

All of this started to change with the advent of [SNORKs](https://github.com/matter-labs/awesome-zero-knowledge-proofs#snorks), a new generation of ZKPs based on a slightly different set of cryptographic primitives — most notably, polynomial commitment schemes. Pioneered by Sean Bowe in *Sonic*, it was followed in summer 2019 by *PLONK* and *Marlin*. All of them have one thing in common: while trusted setup is still required, it now would be universal and updateable. Done once, it could be reused for any number of different programs at any time.

However, the Kate polynomial commitment scheme used in these proof systems would still require efficient cycles of elliptic curves for recursion, which are currently not available. This is why we are super excited about the most recent, fully succinct and transparent (no trusted setup) [proof systems](https://github.com/matter-labs/awesome-zero-knowledge-proofs#starks), such as Halo, SuperSonic, Fractal**,** and [something exciting](https://twitter.com/the_matter_labs/status/1171698458505416704) Matter Labs team is currently working on.

**Long story short:** the barriers to building general purpose smart contracts on ZKPs have now been removed. ZK Rollup is perfectly able to support the same programming model as EVM (including seamless composability and interoperability). The first contracts will likely require specialized DSLs, although the learning curve for Solidity developers won’t exceed 1 day. Eventually, given the current pace of advances in ZKPs prover technologies, we expect all existing ETH (and even EWASM) contracts to be efficiently portable with minimum effort.

#### Scalability & transaction costs

**Optimistic Rollup**

- [According to John Adler](https://t.me/plasmacontributors/1831), the current estimate is about 4k gas per transfer tx, post EIP2028/Istanbul.
- Which translates into ~100 TPS
- With BLS signature aggregation, this number can go up to ~500 TPS (in order not to break EVM compatibility tx params will likely remain long).
- If the EVM compatibility is broken, the throughput could theoretically grow up to the limits of ZKR.

**Realistic throughput cap (token transfers): 500 TPS.**

This is probably fine for now.

**ZK Rollup**

- The cost of public data per transfer tx in Matter Testnet is currently 16 bytes, which will cost 272 gas post EIP2028/Istanbul.
- Additionally, there will be an amortized cost of the proof, estimated at something like 300k gas.
- Even if we assume a worst case scenario with a 1M gas proof cost, the estimated ceiling will still be over 2140 TPS for transfers.
- In some discussions I have heard people argue that ZKPs incur significant computational overhead and are therefore expensive. In reality, the computational cost is negligible compared to the cost of gas, which is the real bottleneck because of censorship-resistant decentralization. We also expect this factor to go down significantly with time.

**Realistic throughput cap (token transfers): over 2000 TPS — Visa-like scale**.

However, for a lot of use cases ZK Rollup will offer much more significant savings, because large pieces can be omitted from the public data (by moving them to the ZK circuit witness), which are not required to reconstruct the state transition delta. The core insight is this: while OR always requires users to publish the complete transaction input, in ZK we can flexibly choose between 1) transaction input minus witness not affecting the state transition, and 2) transaction output only. This choice can be implemented quite elegantly and without a lot of complexity.

Notable examples:

- In multisig wallets, wallets with Argent-style account abstractions or decentralized exchanges, users need to submit signatures to be verified by the contract. These sigs are not required for state delta updates and can be omitted from public data.
- Contracts like Gnosis’ Dfusion dutch DEX require large dataset inputs which do not directly affect storage, but are only used to verify the results of computations.

**Post ETH 2.0**

Since any Rollup will reside in a single shard, it is unlikely that the costs of CALLDATA (and thus Rollup transaction costs) will change much, unless bandwidth generally becomes cheaper.

#### Meta-transactions

Both Rollups are equally well suited to support meta-transactions and account abstraction.

#### Security

**Optimistic Rollup**

Unlike payment channels, all funds in a Rollup are held by a single smart contract. Since Rollup is IMHO the most promising scaling direction, we should see a large number of users moving into it and a lot value being concentrated in this one contract. With tens or hundreds of Millions (or maybe even Billions) of dollars worth of assets at stake, the Rollup contract becomes an extremely attractive honeypot for high profile hackers. Under these conditions, if an attack has good chances, it will probably be attempted no matter how intricate.

The security model of OR is based on two assumptions:

1. At least 1-of-N honest participants who execute all OR transactions and will submit fraud proof in case invalid state transition is published;
2. Strong censorship-resistance of underlying L1 network.

**1-of-N honest participants**

As for the first part, it’s realistic to expect that only the operators of the Rollup will be actually monitoring and executing transactions. Normal users will have neither incentives to do so, nor technical capabilities to process transactions at high load (if they could, where would the scaling come from?). Luckily, operators are naturally incentivized to check each other’s blocks for correctness, because creating a block on top of an invalid one is a slashing condition.

1-of-N honest operators is a reasonable assumption with enough credible participants. However, since the number of active participants is limited (hundreds?), some sophisticated attacks could include: targeting the infrastructure of all the operators (very hard but not infeasible), bribing/blackmailing Devops engineers to secretly install malicious code, targeting update distribution channels for rollup software, etc., and of course a combination thereof. These attacks are hard and should be actively protected against, but they are much more realistic than, let’s say, trying to attack Ethereum miners in the same way — especially because a successful attack on OR will go unnoticed until completion.

**Strong censorship-resistance of L1**

The second assumption is a tricky one. Indeed, the design of Ethereum provides economic mechanisms which are very effective in preventing ordinary censorship. Yet, these mechanisms stop functioning in the presence of [anti-mechanisms](https://slideslive.com/38920016/designing-smart-contracts-with-free-will). An attacker can create a [fully automated bribing mechanism to coordinate a 51% attack by miners](https://ethresear.ch/t/nearly-zero-cost-attack-scenario-on-optimistic-rollup/6336/6), which will prevent honest miners from including fraud proofs in their blocks. Interestingly, the direct cost of this attack for participating miners is zero, not counting social costs which might arise from the response of an angry community if the censorship can clearly be attributed. This part is also tricky, because the mechanism elegantly provides plausible deniability to the participants in the attack: “Given the credible commitment by the attacking majority, if I don’t participate, my blocks will be abandoned, so I must be doing this not for the sake of profit, but rather to avoid losses”.

I invite the reader to follow a [discussion of this attack](https://twitter.com/gluk64/status/1184399877146587136) and a recent [analysis of 51% censorship attacks by Vitalik Buterin](https://ethresear.ch/t/responding-to-51-attacks-in-casper-ffg/6363). Below I will share some interesting insights.

This type of attack is, unfortunately, very realistic under PoW. There is no effective way to punish anonymous miners for participation in it.

After the transition to PoS, the community will be in a position to punish censoring miners by slashing their stake, if the broad social consensus is reached on this. After all, a censorship attack like this could be considered an aggression against the entire network (although one could also argue that the miners simply honestly follow the protocol and are not obligated to behave in any way contrary to their best economic interests). However, post DAO-fork, this will be a very controversial discussion, to say the least, with an unpredictable outcome. In a recent community poll by Vitalik, [63% voted strictly against](https://twitter.com/VitalikButerin/status/1187854232398917632) any manual intervention in the immutable blockchain to bail out users, regardless of the extent of the attack. Needless to say, wiping out the stake of even one validator would be extremely difficult to push through, let alone wiping out the stake of the majority.

***Update Nov 26 2019:\*** *more research* [*on collusion*](https://vitalik.ca/general/2019/04/03/collusion.html) *has recently been published, as well as a* [*new attack on fraud proofs in PoS environment*](https://ethresear.ch/t/undetectable-censorship-attack-on-fraud-proof-based-layer2-protocols/6492)*, which demonstrate that the censorship attack risks for OR in PoS are at least as high as in PoW.*

A more realistic way to withstand this kind of attack is a quick mobilization of the community in a UASF (user-activated soft fork) to force miners to include certain transactions. This scenario is complicated from both an engineering and a social perspective, and will definitely require a relatively long challenge period window for fraud proofs — minimum 1 week, better 2. At the same, given that the [major DeFi operators are effectively in a position to decide the outcome of such a fork](https://medium.com/dragonfly-research/ethereum-is-now-unforkable-thanks-to-defi-9818b967738f), and that it is in their best interest to avoid loud disrupting events, their best bet could be to just silently comply with the attacker (which will keep Ethereum on the longest chain and produce a less controversial result than a successful soft-fork).

To summarize: the risks of fraud proof censorship are relatively low but non-negligible. With a 1–2-week fraud proof challenge period and not too much money at stake, the OR is probably fine: operator/miner collusion is not going to be worth the hassle and the risks. However, as the value in the rollup grows, the lurking Black Swan would become more and more worrisome, at least to people as paranoid as yours truly.

**ZK Rollup**

In a ZK Rollup, every state transition is verified by the Rollup smart contract before it becomes effective. It is strictly not possible for operators to steal the funds or corrupt the Rollup state. ZKR relies on the censorship-resistance of L1 only for its liveness, not for its security. There is no need for anyone to monitor the ZKR: after a block is verified, user funds are always guaranteed to be eventually retrievable even if operators refuse to cooperate.

Thus, ZKR embodies more fully the foundational ideals of crypto: achieving resilience by replacing trusted parties with cryptography and game-theoretical incentive alignment.

For completeness, however, I must mention several other potential risks specific to ZKR.

**Trusted setup**

If the ZKPs used in a ZK Rollup require a universal trusted setup, we end up with the 1-of-N honest participants assumption. This might or might not be an acceptable risk, depending on the number and quality of the participants. But safe is safe, which is why I’m very excited about the recent advances in efficient trustless SNARKs, especially [the construct](https://eprint.iacr.org/2019/1020) we at Matter Labs are currently working on.

**Cryptography**

The newest generation of SNARKs is using multiple more solid and battle-tested cryptographic primitives than Groth16. Matter Labs’ work mentioned earlier is based on FRI and is therefore even plausibly post-quantum secure. However, to be completely calm, two mitigation strategies should be applied:

- A large bounty must be deployed with much lower security parameters than the actual production version, similar to the RSA challenge. If a practical attack is ever discovered, the challenge will be broken by researchers years before breaking the production code becomes feasible.
- All state transitions must be sendable only by the operators of the ZKR, who will essentially serve as a 2-Factor protection layer.

#### Latency (time-to-verifiable-finality)

**Optimistic Rollup**

Due to the problems mentioned in the security section above, Optimistic Rollup can only be safe with a 1–2-week fraud proof challenge window. No transaction can be considered final until this time passes — neither an internal Rollup tx nor an exit.

Unfortunately, there is no quicker way for an end-user to check whether the transaction is final or not, than by executing all the transactions for the entire last challenge period. It’s important to note that users cannot rely on pure game-theoretical guarantees of block finalization, because a bug (or a hack) in a node of a single operator can still lead to reverts.

Time-to-finality (under PoW): 2 weeks.

Time-to-finality (under PoS): 1 week.

**ZK Rollup**

Currently ZKPs are quite computationally intense. At present, for a block of 1000 tx we can have 20 minute proof generation time on ordinary server hardware.

Ongoing GPU prover implementations (by [Matter Labs](https://github.com/matter-labs/belle_cuda) and [Coda](https://github.com/CodaProtocol/snark-challenge-prover-reference)) promise to increase tx speeds by at least ~10x. In the not so distant future, specialized hardware will be likely to boast a much higher computational power. Eventually, we expect to see block confirmation under 1 min.

Time-to-finality (now): 20 min.

Time-to-finality (future): under 1 min.

#### Fast confirmations for intra-Rollup transactions

In both types of Rollup, it is possible for operators to issue instant transaction confirmations to the users by putting up certain security deposits which will be slashed if the transaction is not included in the promised block. This provides an economic guarantee to finality.

This approach has several limitations. It works well for transfers of fungible tokens, but it gets difficult with **NFTs** (which might have no market value, or when the owner of such assets would not want to “sell” it immediately under any circumstances) and **generalized contract calls** (because it’s not easy to exactly quantify the monetary value if some previous transaction in the chain gets reverted; a simple example: how much of the operator’s money should be at stake for you to accept a stablecoin oracle price broadcast as final?)

#### Fast withdrawals

Fast exits are similar to fast intra-Rollup confirmations. Operators can cooperate with liquidity providers to initiate withdrawals of fungible tokens to users immediately, without waiting for the exit transaction to become final in the Rollup.

This requires a significant amount of collateral which will be proportional to the time-to-finality. Assuming realistic near-future finality times of 1 week for OR and 5 min for ZKR, OR would require 2000 times more collateral to support the same weekly withdrawal volume as ZKR.

#### Privacy

**Optimistic Rollup**

Optimistic Rollup can support any privacy solution available on L2 Ethereum (mixers, etc). Since OR itself is L2, any privacy solution implemented on it will live as L3. This might lead to even more fragmentation of privacy services, and as a result to small anonymity sets, which renders the utility of privacy very low (as we can observe even with zcash, where transactions are not shielded by default).

**ZK Rollup**

To achieve real privacy, systems must support it by default. From the technological perspective, ZKR can at some point easily support confidential transactions for token transfers at the protocol level by default, as well as differentiate between public and private smart contracts (ZK ZK Rollup style).

At the same time, building fully anonymous transactions zcash-style (i.e. hiding not only amounts, but also the participants of the transaction) would require changing the storage model of ZK Rollup from account-based to UTXO-based, which would create too many problems and is unlikely to happen.

### **Conclusion**

Optimistic Rollup is currently in the PoC stage. We will hopefully see production-grade implementation coming soon. If it turns out to be relatively easy to port existing code, projects will gradually start adopting it and building new infrastructure: L2 support will appear in wallets, oracles will start broadcasting to OR, etc.

ZK Rollup is already more mature with regard to specialized applications (such as [ERC-20 token transfers](http://demo.matter-labs.io/)), but will travel a more gradual path with fully generalized smart contracts. Eventually, it will be possible to port any EVM- and WASM-based smart contract to ZK Rollup — and at the current pace of technological development, this is not likely to take years.

Similar infrastructure changes in wallets, oracles and other smart contract components must be made for both Rollup types. This requires a significant amount of work which will be accelerated as more projects become interested in L2 scaling tech. Since Optimistic Rollup makes a promise of generalized EVM-based smart contracts earlier than ZK-Rollup, it will provide a huge boost to the community’s motivation to adopt L2.

For users and dapps, jumping from one Rollup to another will be easier than the initial migration from ETH to L2. Bridges will make this process even smoother. Because of this ease of switching, my personal take is that the solution develops a significant edge in UX will likely become the sole winner in the long term.

No matter the outcome, this is going to be a very important and exciting evolution to observe. And the ultimate winner will be the Ethereum community in any case.



## 3. zkRollup vs. Validium

DeversiFi recently [launched a new version](https://blog.deversifi.com/introducing-deversifi2-0/) of their exchange, powered by the StarkEx trading engine. This is an incredible technological achievement that raises the bar for the kind of security users can expect from crypto exchanges. It also marks a historical turning point: it’s the first ever application of [STARKs](https://github.com/matter-labs/awesome-zero-knowledge-proofs#starks) (succinct zero-knowledge proofs without trusted setup) in a production system.

For background, StarkEx is a Validium: a Layer-2 scaling solution in which the validity of all transactions is enforced using zero-knowledge proofs, while data availability is kept off-chain. This prevents the funds in the Validium from being stolen as every transfer of value from an account of a given user must be authorized by that user.

<iframe src="https://cdn.embedly.com/widgets/media.html?type=text%2Fhtml&amp;key=a19fcc184b9711e1b4764040d3dc5c07&amp;schema=twitter&amp;url=https%3A//twitter.com/VitalikButerin/status/1267455602764251138&amp;image=" allowfullscreen="" frameborder="0" height="281" width="500" title="" class="t u v je ak" scrolling="auto" style="box-sizing: inherit; position: absolute; top: 0px; left: 0px; width: 680px; height: 382.156px;"></iframe>

Validium’s mechanism is very similar to a [zkRollup](https://zksync.io/faq/tech.html#zkrollup-architecture), the only difference being that data-availability in a zkRollup is on-chain, while Validium keeps it off-chain. This permits Validium to achieve considerably higher throughput — but this comes at a price:

### Operators of a StarkEx Validium can freeze users’ funds.

> “The people who can destroy a thing, they control it.”
>
> Frank Herbert, Dune

Without zkRollup’s data availability guarantees, the operator — or to be more precise: the data availability manager(s) — of a Validium can deny any user the right to move their funds.

Here’s how it works: the operator makes a tiny change in the Merklized state without disclosing the state change to users. Lacking this information, users cannot create Merkle proofs of ownership for their accounts.

![img](https://miro.medium.com/max/60/0*XMv93TVXGQMpO0ti?q=20)

![img](https://miro.medium.com/max/1844/0*XMv93TVXGQMpO0ti)

*Illustration: if account* ***d3\*** *is changed by the operator, the owner of account* ***d1\*** *will be missing the information about the node* ***m\*** *they need in their proof in order to prove their account ownership.*

*Is there a way to prevent data withholding attacks in Validium?* This problem has been extensively discussed since the conception of Plasma in 2016, and zkRollups were born as an outcome of that research. Non-rollup attempts to trustlessly ensure data availability would result in [losing most of Validium’s competitive advantages](https://twitter.com/VitalikButerin/status/1267566253780226048).

While the problem is not entirely solvable, StarkEx mitigates it by introducing a permissioned Data Availability Committee (DAC). The DAC must acknowledge it has received the data by signing every update to the state by a quorum of its members. In StarkEx, the DAC consists of [8 participants](https://medium.com/starkware/data-availability-e5564c416424) (adding too many members will jeopardize the liveness of the system). These are well known, highly reputable organizations in established legal jurisdictions. It is very unlikely for them to ever even try to abuse their powers — or so the reasoning goes.

Paradoxically, being well known, highly reputable, and residing in a jurisdiction with a strong state is [exactly what makes them vulnerable](https://vitalik.ca/general/2019/05/09/control_as_liability.html). One plausible scenario of things going haywire: operators are required to implement KYC/AML regulations and are obliged to freeze all funds of the accounts with over $10k trading history (possibly forever).

It gets even more interesting as we dive deeper. StarkEx implements a [Verifier Contract Upgrade](https://medium.com/starkware/contract-upgradability-d3a4451877c) mechanism that permits the operator to add a new item to the chain of verifier contracts immediately, sans delay. This cannot invalidate any of the old logic — you can’t remove user signature checks for example. Rather it allows additional constraints to be added (you can think of constraints as `require()` statements, speaking in terms of Solidity).

It is a nice security feature: should any missing constraint be found in the StarkEx’ STARK circuit logic, it can be fixed quickly without introducing any new vulnerabilities. However, this feature can be abused as a **concealed** **censorship backdoor**. In a nutshell, the StarkEx operator can always deploy an extension to the contract logic that introduces a blacklist without any prior warning to users. It’s not entirely clear from their documentation, but it looks like the consent of the DAC is not required to enforce the new rules.

This doesn’t make much sense if you were to think of StarkEx as a fully decentralized exchange protocol. **Imagine Vitalik Buterin owning a switch that can instantly freeze any Ethereum account**. On the other hand, it makes perfect sense if you look at StarkEx as a security enhancement for crypto exchanges (which its creators surely do).

### Operators of a StarkEx Validium can seize users’ funds.

Let’s extend our thought experiment. For whatever hypothetical reason (most likely owing to circumstances outside the control of the operators), the assets of a number of users are now frozen. Could users’ funds in StarkEx also be confiscated?

As a matter of fact, this can happen.

StarkEx, just like many other crypto projects, implements a state-of-the-art [upgrade mechanism](https://medium.com/starkware/contract-upgradability-d3a4451877c). Users are given 28 days notice before a new version is deployed and whoever doesn’t like it can exit.

*Except* for those whose funds have been frozen.

A new contract logic can be deployed after the grace period is over that transfers the frozen funds into custody of a designated party. Unfortunately, there is nothing affected users will be able to do against it.

There are also [reasonable concerns](https://twitter.com/jadler0/status/1268920716944162816) that the upgrade notice period per se might not be sufficient to allow every user who disagrees with the changes to exit (the so-called “mass exit” scenario). But this problem is a general contract upgradeability issue not unique to Validium.

### *Update 2020–07–06: Justin Drake describes crypto-economic attack on Validium.*

In the follow-up discussion, Justin Drake [pointed out](https://twitter.com/drakefjustin/status/1269309163995303936) that data availability approach of Validium can lead to an unexpected attack vector: should the signing keys of the quorum of the Data Availability Committee be compromised (and these keys are kept online, which makes it notoriously hard to secure them), attacker can transition Validium into a state only known to them, thus freezing all assets, and then demand ransom to unlock it.

Theoretically, the contract upgrade mechanism should mitigate this attack. Validium’s operators could initiate deployment of a new version where the state is reverted to the last known one after 28 days of upgrade notice period. It would be a month of locked capital (which of course has quite significant costs), but if the DAC refused to negotiate, attacker would not get a single penny.

However, it turns out there is a way for attacker to force the operators into deciding between losing everything or allowing the attacker to make a double-spend. It can be illustrated with the following example:

Imagine that you can hack an ATM in such a way as to erase the entire bank database after your withdrawal is complete. You can only withdraw from your own account, but the details of the operation will be lost when the DB is gone.

Bank employees can go through a complicated process of restoring the database in one month. But since they don’t know who did the withdrawal, by reverting to the last checkpoint they will also restore the balance on your account—restoring the balance you have withdrawn!

Of course, this double-spend will be limited to the amount the attacker can provision on their account. However, it is trivial to construct a trustless contract and borrow the necessary capital from evil anonymous whales in the darknet. We will leave this exercise to the reader.

This attack demonstrates that the security model of Validium is relatively similar to that of a PoA network. In fact, a PoA network with 20 nodes and 51% signing threshold might be **more secure** than a Validium with 8 nodes and 100% signing threshold.

### Data Availability in zkRollup protects users’ funds from seizure, censorship and hacks—at a cost of somewhat lower throughput.

The rollup’s state is available to the zkRollup users as long as at least one Ethereum full node is online.

*How it works:* for every zkRollup block, information required to reconstruct the changes in the state must be submitted as call-data of the Ethereum transaction — otherwise the zkRollup smart contract will refuse to make the state transition. State changes on zkRollups incurs a small gas cost per transaction which grows linearly with the number of transactions.

With the Merkle tree data at hand, users who are being censored always have the ability to claim their funds directly from the zkRollup contract on the mainnet. All they need to do is to provide a Merkle proof of ownership on their account. Thus, on-chain data availability serves as a guarantee that **nobody (including zkRollup operators) can freeze nor capture users’ funds**.

On-chain storage for data availability leads to a limitation in throughput — zkRollups have a strict practical ceiling of 2k transactions per second (TPS) on today’s Ethereum, while StarkEx Validium claims 9k+ TPS. This difference will likely play an important role in determining the application areas and use-cases for both technologies. For example, zkRollup is well suited for scaling decentralized crypto payments (VISA averages 2k TPS globally) and immutable smart contracts with strict requirements for trustlessness; Validium, on the other hand, could be a better fit for traditional high-frequency trading or games with lower trust assumptions.

### Conclusion

We’ve shown that zkRollups and Validium (StarkEx) are relatively similar in how they work, with their main point of difference — whether data is available on-chain or off-chain — crucial in understanding them and where they can be used. This difference means that while zkRollup is a completely trustless decentralized scaling protocol, Validium displays more properties of a custodial PoA system— both in its throughput capacity and its risk profile — albeit with greatly improved security.

Every technological development that reduces trust and provides users with more control over their assets is a step towards empowering the individual. There are always trade-offs we need to make in order to keep moving forward.

Nevertheless, there is a growing consensus in the crypto-community that technology has passed the “don’t be evil” phase — it’s high time for “can’t be evil.” We can get there through self-custody, censorship-resistance, privacy and elimination of single points of failure. These ideas form the foundational values for the systems we’re striving to build.

The time for fully trustless scalability is arriving and the count-down for Matter Labs’ big announcement is on — [stay tuned](https://twitter.com/the_matter_labs)!

