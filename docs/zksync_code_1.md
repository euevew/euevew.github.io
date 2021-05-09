# zkRollup 第一站——zkSync 合约代码

## 1. 概述

[zkSync](https://github.com/matter-labs/zksync) 的代码主要有两个部分，contracts 和 core。其中 contracts 部分为合约代码（语言为 Solidity），core 部分为后端合约代码（主要语言为 Rust）。

本文我们先来读一下合约部分代码，下一篇文章将开始读core部分代码。contracts 部分的代码结构如下：
```shell
├── Bytes.sol
├── Config.sol
├── DeployFactory.sol
├── Events.sol
├── ForcedExit.sol
├── Governance.sol
├── IERC20.sol
├── Operations.sol
├── Ownable.sol
├── PlonkCore.sol
├── Proxy.sol
├── ReentrancyGuard.sol
├── SafeCast.sol
├── SafeMath.sol
├── SafeMathUInt128.sol
├── Storage.sol
├── TokenInit.sol
├── UpgradeGatekeeper.sol
├── Upgradeable.sol
├── UpgradeableMaster.sol
├── Utils.sol
├── Verifier.sol
├── ZkSync.sol
└── dev-contracts
    ├── AccountMock.sol
    ├── BytesTest.sol
    ├── DummyTarget.sol
    ├── EIP1271.sol
    ├── IEIP1271.sol
    ├── Multicall.sol
    ├── OperationsTest.sol
    ├── RevertReceiveAccount.sol
    ├── RevertTransferERC20.sol
    ├── SelfDestruct.sol
    ├── TestnetERC20Token.sol
    ├── ZKSyncSignatureUnitTest.sol
    ├── ZkSyncProcessOpUnitTest.sol
    ├── ZkSyncWithdrawalUnitTest.sol
    └── tokens
        ├── ContextTest.sol
        ├── MintableERC20FeeAndDividendsTest.sol
        ├── MintableERC20NoTransferReturnValueTest.sol
        ├── MintableIERC20NoTransferReturnValueTest.sol
        └── MintableIERC20Test.sol

```
这个目录分成两部分，直接的 solidity 合约代码和dev_contracts部分为测试用的 solidity 合约代码。
*  直接的 solidity 合约代码包括合约的主要代码，以ERC20 代币为基础的基本操作
* dev_contracts 目录下为测试代码。

## 2. 主要合约代码

### 2.1 基础合约
 0. Config 合约为静态配置项，如下：
    ```js
    contract Config {
        /// @dev ERC20 tokens and ETH withdrawals gas limit, used only for complete withdrawals
        uint256 constant WITHDRAWAL_GAS_LIMIT = 100000;

        /// @dev Bytes in one chunk
        uint8 constant CHUNK_BYTES = 9;

        /// @dev zkSync address length
        uint8 constant ADDRESS_BYTES = 20;

        uint8 constant PUBKEY_HASH_BYTES = 20;

        /// @dev Public key bytes length
        uint8 constant PUBKEY_BYTES = 32;

        /// @dev Ethereum signature r/s bytes length
        uint8 constant ETH_SIGN_RS_BYTES = 32;

        /// @dev Success flag bytes length
        uint8 constant SUCCESS_FLAG_BYTES = 1;

        /// @dev Max amount of tokens registered in the network (excluding ETH, which is hardcoded as tokenId = 0)
        uint16 constant MAX_AMOUNT_OF_REGISTERED_TOKENS = $(MAX_AMOUNT_OF_REGISTERED_TOKENS);

        /// @dev Max account id that could be registered in the network
        uint32 constant MAX_ACCOUNT_ID = (2**24) - 1;

        /// @dev Expected average period of block creation
        uint256 constant BLOCK_PERIOD = 15 seconds;

        /// @dev ETH blocks verification expectation
        /// @dev Blocks can be reverted if they are not verified for at least EXPECT_VERIFICATION_IN.
        /// @dev If set to 0 validator can revert blocks at any time.
        uint256 constant EXPECT_VERIFICATION_IN = 0 hours / BLOCK_PERIOD;

        uint256 constant NOOP_BYTES = 1 * CHUNK_BYTES;
        uint256 constant DEPOSIT_BYTES = 6 * CHUNK_BYTES;
        uint256 constant TRANSFER_TO_NEW_BYTES = 6 * CHUNK_BYTES;
        uint256 constant PARTIAL_EXIT_BYTES = 6 * CHUNK_BYTES;
        uint256 constant TRANSFER_BYTES = 2 * CHUNK_BYTES;
        uint256 constant FORCED_EXIT_BYTES = 6 * CHUNK_BYTES;

        /// @dev Full exit operation length
        uint256 constant FULL_EXIT_BYTES = 6 * CHUNK_BYTES;

        /// @dev ChangePubKey operation length
        uint256 constant CHANGE_PUBKEY_BYTES = 6 * CHUNK_BYTES;

        /// @dev Expiration delta for priority request to be satisfied (in seconds)
        /// @dev NOTE: Priority expiration should be > (EXPECT_VERIFICATION_IN * BLOCK_PERIOD)
        /// @dev otherwise incorrect block with priority op could not be reverted.
        uint256 constant PRIORITY_EXPIRATION_PERIOD = 3 days;

        /// @dev Expiration delta for priority request to be satisfied (in ETH blocks)
        uint256 constant PRIORITY_EXPIRATION =
            $(defined(PRIORITY_EXPIRATION) ? PRIORITY_EXPIRATION : PRIORITY_EXPIRATION_PERIOD / BLOCK_PERIOD);

        /// @dev Maximum number of priority request to clear during verifying the block
        /// @dev Cause deleting storage slots cost 5k gas per each slot it's unprofitable to clear too many slots
        /// @dev Value based on the assumption of ~750k gas cost of verifying and 5 used storage slots per PriorityOperation structure
        uint64 constant MAX_PRIORITY_REQUESTS_TO_DELETE_IN_VERIFY = 6;

        /// @dev Reserved time for users to send full exit priority operation in case of an upgrade (in seconds)
        uint256 constant MASS_FULL_EXIT_PERIOD = 9 days;

        /// @dev Reserved time for users to withdraw funds from full exit priority operation in case of an upgrade (in seconds)
        uint256 constant TIME_TO_WITHDRAW_FUNDS_FROM_FULL_EXIT = 2 days;

        /// @dev Notice period before activation preparation status of upgrade mode (in seconds)
        /// @dev NOTE: we must reserve for users enough time to send full exit operation, wait maximum time for processing this operation and withdraw funds from it.
        uint256 constant UPGRADE_NOTICE_PERIOD =
            $(
                defined(UPGRADE_NOTICE_PERIOD)
                    ? UPGRADE_NOTICE_PERIOD
                    : MASS_FULL_EXIT_PERIOD + PRIORITY_EXPIRATION_PERIOD + TIME_TO_WITHDRAW_FUNDS_FROM_FULL_EXIT
            );

        /// @dev Timestamp - seconds since unix epoch
        uint256 constant COMMIT_TIMESTAMP_NOT_OLDER = 24 hours;

        /// @dev Maximum available error between real commit block timestamp and analog used in the verifier (in seconds)
        /// @dev Must be used cause miner's `block.timestamp` value can differ on some small value (as we know - 15 seconds)
        uint256 constant COMMIT_TIMESTAMP_APPROXIMATION_DELTA = 15 minutes;

        /// @dev Bit mask to apply for verifier public input before verifying.
        uint256 constant INPUT_MASK = $$(~uint256(0) >> 3);

        /// @dev Auth fact reset timelock
        uint256 constant AUTH_FACT_RESET_TIMELOCK = 1 days;
    }
    ```
 1. SafeMath 合约为增加了溢出检查的算术操作（加，减，乘，除，取模），在默认的uint256类型的数字上进行的运算
 2. SafeMathUInt128 合约与SafeMath类似，只是数据类型变成了 uint128
 3. SafeCast 合约在将 uint256 转换为 uint8, uint16, uint32, uint64, uint128 时增加了溢出检查。
 4. Bytes 合约是字节转换操作的函数集合：各个长度的整形数字，地址，Byte20与byte数组之间互相转换等等
 5. Utils 合约为工具类合约，主要是一些工具类的函数接口：去两个数的最小值，erc20 合约转账，从以太账户签名中恢复地址，生成合约地址哈希值
 6. Operations 合约为电路操作及pub data相关的一些操作函数
 7. IERC20 合约为ERC20 合约接口
 8. Ownable 合约为管理员合约，包括与管理员master相关的一些操作函数。
 9. Upgradeable 合约为升级合约接口，将合约升级到目标地址
 10. UpgradeableMaster 合约为升级 master合约的升级合约接口，包含以下几个函数：
    ```js
     /// @notice Notice period before activation preparation status of upgrade mode
    function getNoticePeriod() external returns (uint256);

    /// @notice Notifies contract that notice period started
    function upgradeNoticePeriodStarted() external;

    /// @notice Notifies contract that upgrade preparation status is activated
    function upgradePreparationStarted() external;

    /// @notice Notifies contract that upgrade canceled
    function upgradeCanceled() external;

    /// @notice Notifies contract that upgrade finishes
    function upgradeFinishes() external;

    /// @notice Checks that contract is ready for upgrade
    /// @return bool flag indicating that contract is ready for upgrade
    function isReadyForUpgrade() external returns (bool);
    ```
 11. Proxy 合约为代理合约实现了合约升级，master合约升级等等操作
 12. Events 合约为事件接口
 13. UpgradeGatekeeper 合约为更新门卫的合约，
 14. Governance 为治理合约，包含跟治理相关的一些函数
 15. TokenInit 合约是初始化部署新 token 合约的合约，获取 token 合约地址。
 16. ReentrancyGuard 合约放重入攻击模块
 17. PlonkCore 合约为 Plonk 验证合约，主要分为以下几个部分：
    ```js
    // BN254 的 pairing操作
    library PairingsBn254 {}
    library TranscriptLibrary {}
    contract Plonk4VerifierWithAccessToDNext {}
    contract VerifierWithDeserialize is Plonk4VerifierWithAccessToDNext {}
    contract Plonk4VerifierWithAccessToDNextOld {}
    contract VerifierWithDeserializeOld is Plonk4VerifierWithAccessToDNextOld {}
    ```
 18. ForcedExit 合约为紧急赎回二层资金的合约
 19. Verifier 合约为验证合约，主要包括验证聚合区块和验证退出证明
 20. DeployFactory 合约为部署工厂合约，用来部署 Proxy 合约
 21. Storage 合约，包括存储PendingBalance，Pending withdrawals，block，优先操作容器，将地址和令牌ID打包成一个词，作为余额映射的一个关键，block信息。
 22. ZkSync 合约为 zkSync 的主合约。执行链上的操作，主要的函数如下：
    ```js
    function getNoticePeriod()
    function upgradeNoticePeriodStarted()
    function upgradePreparationStarted()
    function upgradeCanceled()
    function upgradeFinishes()
    function isReadyForUpgrade()
    function initialize(bytes calldata initializationParameters) 
    function upgrade(bytes calldata upgradeParameters)
    function _transferERC20{}
    function cancelOutstandingDepositsForExodusMode(uint64 _n, bytes[] memory _depositsPubdata)
    function depositETH(address _zkSyncAddress)
    function depositERC20(IERC20 _token, uint104 _amount, address _zkSyncAddress) 
    function getPendingBalance(address _address, address _token) 
    function getBalanceToWithdraw(address _address, uint16 _tokenId) 
    function withdrawPendingBalance(address payable _owner, address _token, uint128 _amount)
    function withdrawERC20(IERC20 _token, uint128 _amount) 
    function withdrawETH(uint128 _amount) 
    function requestFullExit(uint32 _accountId, address _token)
    function fullExit(uint32 _accountId, address _token) 
    function commitOneBlock(StoredBlockInfo memory _previousBlock, CommitBlockInfo memory _newBlock)
    function commitBlocks(StoredBlockInfo memory _lastCommittedBlockData, CommitBlockInfo[] memory _newBlocksData) 
    function withdrawOrStore(uint16 _tokenId, address _recipient, uint128 _amount) 
    function executeOneBlock(ExecuteBlockInfo memory _blockExecuteData, uint32 _executedBlockIdx)
    function executeBlocks(ExecuteBlockInfo[] memory _blocksData) 
    function proveBlocks(StoredBlockInfo[] memory _committedBlocks, ProofInput memory _proof)
    function revertBlocks(StoredBlockInfo[] memory _blocksToRevert) 
    function activateExodusMode() 
    function performExodus(StoredBlockInfo memory _storedBlockInfo, address _owner, uint32 _accountId, uint16 _tokenId, uint128 _amount, uint256[] memory _proof)
    function setAuthPubkeyHash(bytes calldata _pubkey_hash, uint32 _nonce) 
    function registerDeposit(uint16 _tokenId, uint128 _amount, address _owner)
    function registerWithdrawal(uint16 _token, uint128 _amount, address payable _to)
    function collectOnchainOps(CommitBlockInfo memory _newBlockData)
    function verifyChangePubkey(bytes memory _ethWitness, OperationsChangePubKey memory _changePk)
    function verifyChangePubkeyECRECOVER(bytes memory _ethWitness, Operations.ChangePubKey memory _changePk)
    function verifyChangePubkeyOldECRECOVER(bytes memory _ethWitness, Operations.ChangePubKey memory _changePk)
    function verifyChangePubkeyCREATE2(bytes memory _ethWitness, Operations.ChangePubKey memory _changePk)
    function createBlockCommitment(StoredBlockInfo memory _previousBlock, CommitBlockInfo memory _newBlockData, bytes memory _offsetCommitment)
    function checkPriorityOperation(Operations.Deposit memory _deposit, uint64 _priorityRequestId)
    function checkPriorityOperation(Operations.FullExit memory _fullExit, uint64 _priorityRequestId)
    function requireActive()
    function addPriorityRequest(Operations.OpType _opType, bytes memory _pubData)
    function deleteRequests(uint64 _number)
    function increaseBalanceToWithdraw(bytes22 _packedBalanceKey, uint128 _amount)
    function sendETHNoRevert(address payable _to, uint256 _amount)
    ```

### 2.2 dev_contracts

 1. BytesTest 合约为 Bytes 合约的测试合约
 2. OperationsTest 合约为 Operations 合约的测试合约
 1. Multicall 合约，将多个只读函数调用的结果汇总起来，
 2. IEIP1271 是用来验证签名的合约接口，EIP1271 为 IEIP1271的实现
 3. AccountMock 合约为账户Mock合约，实现了签名验证
 4. DummyTarget 为假测试接口
 5. TestnetERC20Token 合约在原有的 ERC20 的基础上增加了 mint操作
 6. RevertReceiveAccount 合约，为一个根据flag恢复接收资金的账户,用于测试zkSync智能合约的失败提款
 7. RevertTransferERC20 合约， 为一个ERC20代币合约，可以根据一个flag来恢复转账，用于测试zkSync智能合约中失败的ERC-20提款。 
 8. SelfDestruct 合约为增加了自毁功能的合约
 9. ZkSyncProcessOpUnitTest 合约为 ZkSync 处理 op 操作的测试合约
 10. ZKSyncSignatureUnitTest 合约为 ZkSync 处理前面的测试合约
 11. ZkSyncWithdrawalUnitTest 合约为 ZkSync 处理赎回的测试合约
 12. ContextTest 合约为提供关于当前执行环境的信息的合约，包括 msg.sender 和 msg.data
 13. MintableIERC20Test 合约为 ERC20 合约铸币测试合约
 14. MintableERC20FeeAndDividendsTest 合约为包含铸币和烧币的ERC20 测试合约
 15. MintableIERC20NoTransferReturnValueTest 合约为包含铸币功能的ERC20 合约接口；MintableERC20NoTransferReturnValueTest 为MintableIERC20NoTransferReturnValueTest 合约接口的实现

## 3. 总结

总体看来，zksync 的合约代码虽然代码量很多，但是结构还是比较清晰的，功能也不算特别复杂难懂，但由于本人目前还不太了解 plonk，所以验证操作的细节还不是很懂。
而对于 zkrollup 来说，链上完成的功能也主要就是register，deposit，withdraw和 layer2提交的block的验证处理。

本文简单的梳理了一些zksync合约代码的结构，后面有时间的话就再继续对合约的实现展开深入的介绍。