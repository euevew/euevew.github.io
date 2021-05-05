# go-ethereum 代码学习笔记（一）——整体结构

*很早之前就想深入读一下 go-ethereum 的代码了，因为我觉得要深入理解以太坊整个生态，深入去读代码是一个很好的切入方式，它可以帮助我更深入得去进入这个领域，跟着它的成长节奏一起成长。但一直以来，由于种种原因以及自己的懒惰，这件事情一直搁置至今，现在想重新拾起来，希望可以在2022年之前读完完整的代码，并完成笔记（如有错误，欢迎指正）。*

go-ethereum 的源码结构非常清晰，本系列文章将按照以下步骤对代码进行阅读分析：代码整体结构——核心模块介绍——历史重大更新——后续发展。

如下所示，是go-ethereum 的源码结构，主要代码是用golang实现的。代码模块包括：accounts，cmd，common，consensus，console，contracts，core，crypto，eth，ethclient，ethdb，ethstats，event，graphsql，internal，les，light，log，metrics，miner，mobile，node，p2p，params，rlp，rpc，signer，swarm，tests，trie。

```shell
.
├── AUTHORS
├── COPYING
├── COPYING.LESSER
├── Dockerfile
├── Dockerfile.alltools
├── Makefile
├── README.md
├── SECURITY.md
├── accounts // 账户模块
├── appveyor.yml
├── build
├── circle.yml
├── cmd // 命令行模块
├── common // 通用代码模块
├── consensus // 共识机制模块
├── console // 控制台模块
├── contracts // 合约模块
├── core // 核心模块
├── crypto // 加密模块
├── docs
├── eth // 以太坊协议模块
├── ethclient // 以太坊客户端模块
├── ethdb // 数据库模块
├── ethstats // 以太坊网络状态模块，网络统计报告服务
├── event // event事件模块
├── go.mod
├── go.sum
├── graphql // 为Ethereum节点数据提供了一个GraphQL接口
├── interfaces.go
├── internal // 内部实现模块，这个软件包主要是胶水代码，通过CLI和RPC子系统提供这些设施。
├── les // 实现了轻量级以太坊子协议
├── light // 为Ethereum Light客户端实现了能够按需检索的状态和链对象。
├── log // 日志模块
├── metrics // 监控模块
├── miner // 挖矿模块
├── mobile // 变量的包装器
├── node // 节点模块
├── oss-fuzz.sh
├── p2p // p2p网络模块
├── params // 参数模块
├── rlp // rlp实现了RLP(Recursive Linear Prefix)的序列化格式
├── rpc // rpc服务模块
├── signer // 签名模块
├── swarm // swarm 模块
├── tests // 测试模块
└── trie // 实现了 Merkle Patricia Tries
```



// TODO



从下一篇文章开始，将按照以下顺序对代码进行分析：

1. ethdb
2. crypto，signer
3. accounts
4. node
5. core
6. event，log
7. eth
8. ethclient，ethstats
9. consensus
10. p2p，rpc
11. contracts
12. node
13. miner
14. light、les，mobile
15. params，common，internal
16. trie，rlp
17. metrics，graphsql
18. cmd，console
19. swarm
20. tests

