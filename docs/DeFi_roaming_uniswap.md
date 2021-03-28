# DeFi 漫游笔记（一） —— uniswap 的简洁力量

*我很喜欢坐高铁，近两年越发喜欢在高铁上做的一件事就是看/写代码，那种低头思考，抬头看苍茫大地在眼前跳跃的感觉真的非常惬意。第一次读 uniswap 的代码就是在去年十一期间的高铁上，那也是我看过的第一份 DeFi 项目的合约代码，虽然有点迟，但为时不晚。最大的感受就是如此简洁有效的设计，真是贯彻了区块链的精髓。*

Uniswap是以太坊上的一个去中心化交易协议。在Uniswap的官网有一篇文章介绍了它的发展历程（[地址链接](https://uniswap.org/blog/uniswap-history/)）。早在2017年Vitalik就写过一篇文章介绍过这个方案。
> If you want to make a market maker for existing tokens without a price cap, my favorite (credit to Martin Koppelmann) mechanism is that which maintains the invariant.
 tokenA_balance(p) * tokenB_balance(p) = k for some constant k. 

## Uniswap V2
目前Uniswap已经从V0版本演化到V2版本，官网有完整的介绍[文档](https://uniswap.org/docs/v2/protocol-overview/how-uniswap-works).

> Uniswap is an automated liquidity protocol powered by a constant product formula and implemented in a system of non-upgradeable smart contracts on the Ethereum blockchain. It obviates the need for trusted intermediaries, prioritizing decentralization, censorship resistance, and security. 

Uniswap中有几个关键角色和关键词：
- LP(liquidity provider)
    LP 是指给交易池子提供流动性的提供商，在Uniswap中，任何人都可以变成池子的LP，他只需要拿等值的基础代币做抵押，来换取池子的 Token，再按照这些代币的比例分配LP份额获得收益，并且可以随时赎回资产。

- Pairs
    从Uniswap V2工厂部署的智能合约，可以在两个ERC20代币之间进行交易。Pair执行做市商的角色，交易对维持一个最简单的公式：x * y = k。它表示交易不得更改池子里Token的余额（x和y）的乘积（k）。

### 合约分析
Uniswap V2的合约分成两部分：Core 和 Periphery。
- 核心（Core）模板
    Core为合约的核心部分，为与Uniswap进行交互的所有各方提供了基本的安全保证。核心部分由一个工厂（Factory）合约和多个交易对（Pairs）合约组成。
    
    - Factory 合约
        Factory 合约用于创建交易对及对交易对和手续费管理
        ```js
        interface IUniswapV2Factory {
        // 创建交易对的事件
        event PairCreated(address indexed token0, address indexed token1, address pair, uint);
        // 读取收取手续费地址
        function feeTo() external view returns (address);
        // 读取可手续费收款地址设置的账户
        function feeToSetter() external view returns (address);
        // 获取指定两个token的交易对
        function getPair(address tokenA, address tokenB) external view returns (address pair);
        // 获取所有交易对
        function allPairs(uint) external view returns (address pair);
        // 获取交易对数量
        function allPairsLength() external view returns (uint);

        // 创建交易对
        function createPair(address tokenA, address tokenB) external returns (address pair);
        // 设置手续费收款地址
        function setFeeTo(address) external;
        // 设置手续费收款地址的设置地址
        function setFeeToSetter(address) external;
    }

    - Pairs 合约
        交易对合约
        ```js
        interface IUniswapV2Pair {
            // erc20 接口和事件
            event Approval(address indexed owner, address indexed spender, uint value);
            event Transfer(address indexed from, address indexed to, uint value);

            function name() external pure returns (string memory);
            function symbol() external pure returns (string memory);
            function decimals() external pure returns (uint8);
            function totalSupply() external view returns (uint);
            function balanceOf(address owner) external view returns (uint);
            function allowance(address owner, address spender) external view returns (uint);

            function approve(address spender, uint value) external returns (bool);
            function transfer(address to, uint value) external returns (bool);
            function transferFrom(address from, address to, uint value) external returns (bool);

            function DOMAIN_SEPARATOR() external view returns (bytes32);
            function PERMIT_TYPEHASH() external pure returns (bytes32);
            function nonces(address owner) external view returns (uint);

            function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external;

            // 铸币，烧币和兑换，同步的事件
            event Mint(address indexed sender, uint amount0, uint amount1);
            event Burn(address indexed sender, uint amount0, uint amount1, address indexed to);
            event Swap(
                address indexed sender,
                uint amount0In,
                uint amount1In,
                uint amount0Out,
                uint amount1Out,
                address indexed to
            );
            event Sync(uint112 reserve0, uint112 reserve1);
         
            function MINIMUM_LIQUIDITY() external pure returns (uint);
            // Factory 合约地址
            function factory() external view returns (address);
            // token0 的地址
            function token0() external view returns (address);
            // token1 的地址
            function token1() external view returns (address);
            // 池子中 token0 和 token1 的剩余 token 数量
            function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);
            function price0CumulativeLast() external view returns (uint);
            function price1CumulativeLast() external view returns (uint);
            // 获取最新的 k 值
            function kLast() external view returns (uint);

            // 铸币
            function mint(address to) external returns (uint liquidity);
            // 烧币
            function burn(address to) external returns (uint amount0, uint amount1);
            // 兑换
            function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external;
            // force balances to match reserves
            function skim(address to) external;
            // 更新reserve值 
            function sync() external;
            // 初始化
            function initialize(address, address) external;
        }

- 外围（Periphery）模板
    Periphery 为处理外围工作的合约，它要与Core部分交互。

    - router1

    ```js
        interface IUniswapV2Router01 {
            // 工厂合约地址
            function factory() external pure returns (address);
            // WETH 地址
            function WETH() external pure returns (address);
            // 增加流动性接口
            function addLiquidity(
                address tokenA,
                address tokenB,
                uint amountADesired,
                uint amountBDesired,
                uint amountAMin,
                uint amountBMin,
                address to,
                uint deadline
            ) external returns (uint amountA, uint amountB, uint liquidity);
            // 增加ETH的流动性接口
            function addLiquidityETH(
                address token,
                uint amountTokenDesired,
                uint amountTokenMin,
                uint amountETHMin,
                address to,
                uint deadline
            ) external payable returns (uint amountToken, uint amountETH, uint liquidity);
            // 移除流动性接口
            function removeLiquidity(
                address tokenA,
                address tokenB,
                uint liquidity,
                uint amountAMin,
                uint amountBMin,
                address to,
                uint deadline
            ) external returns (uint amountA, uint amountB);
            // 移除ETH的流动性接口
            function removeLiquidityETH(
                address token,
                uint liquidity,
                uint amountTokenMin,
                uint amountETHMin,
                address to,
                uint deadline
            ) external returns (uint amountToken, uint amountETH);
            // 授权下移除流动性接口
            function removeLiquidityWithPermit(
                address tokenA,
                address tokenB,
                uint liquidity,
                uint amountAMin,
                uint amountBMin,
                address to,
                uint deadline,
                bool approveMax, uint8 v, bytes32 r, bytes32 s
            ) external returns (uint amountA, uint amountB);
            // 授权下移除ETH流动性接口
            function removeLiquidityETHWithPermit(
                address token,
                uint liquidity,
                uint amountTokenMin,
                uint amountETHMin,
                address to,
                uint deadline,
                bool approveMax, uint8 v, bytes32 r, bytes32 s
            ) external returns (uint amountToken, uint amountETH);
            // 用特定数量的 token 兑换另一种 token
            function swapExactTokensForTokens(
                uint amountIn,
                uint amountOutMin,
                address[] calldata path,
                address to,
                uint deadline
            ) external returns (uint[] memory amounts);
            // 用一种 token 兑换另一种特定数量的 token
            function swapTokensForExactTokens(
                uint amountOut,
                uint amountInMax,
                address[] calldata path,
                address to,
                uint deadline
            ) external returns (uint[] memory amounts);
            // 用特定数量的 ETH 兑换另一种 token
            function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline)
                external
                payable
                returns (uint[] memory amounts);
            // 用一种 token 兑换另一种特定数量的 ETH
            function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline)
                external
                returns (uint[] memory amounts);
            // 用特定数量的 token 兑换另一种 ETH
            function swapExactETHForTokens(uint amountO
            function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline)
                external
                returns (uint[] memory amounts);
            // 用一种 ETH 兑换另一种特定数量的 token
            function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline)
                external
                payable
                returns (uint[] memory amounts);
            function quote(uint amountA, uint reserveA, uint reserveB) external pure returns (uint amountB);
            // 获取兑换出来的 Token 数量
            function getAmountOut(uint amountIn, uint reserveIn, uint reserveOut) external pure returns (uint amountOut);
            // 获取为了兑换某个 Token 所需要的 Token 数量
            function getAmountIn(uint amountOut, uint reserveIn, uint reserveOut) external pure returns (uint amountIn);
            // 获取一组兑换出来的 Token 数量
            function getAmountsOut(uint amountIn, address[] calldata path) external view returns (uint[] memory amounts);
             // 获取一组为了兑换某个 Token 所需要的 Token 数量
            function getAmountsIn(uint amountOut, address[] calldata path) external view returns (uint[] memory amounts);
        }
    ```

    - router2

    ```js
        interface IUniswapV2Router02 is IUniswapV2Router01 {
            // 移除ETH和某个token的流动性（支持 fee-on-transfer 代币）
            function removeLiquidityETHSupportingFeeOnTransferTokens(
                address token,
                uint liquidity,
                uint amountTokenMin,
                uint amountETHMin,
                address to,
                uint deadline
            ) external returns (uint amountETH);
            
            // 授权移除ETH和某个token的流动性（支持 fee-on-transfer 代币）
            function removeLiquidityETHWithPermitSupportingFeeOnTransferTokens(
                address token,
                uint liquidity,
                uint amountTokenMin,
                uint amountETHMin,
                address to,
                uint deadline,
                bool approveMax, uint8 v, bytes32 r, bytes32 s
            ) external returns (uint amountETH);
            // 用指定数量的一种 Token 兑换另一种 Token(支持 fee-on-transfer 代币）
            function swapExactTokensForTokensSupportingFeeOnTransferTokens(
                uint amountIn,
                uint amountOutMin,
                address[] calldata path,
                address to,
                uint deadline
            ) external;
            // 用指定数量的 ETH 兑换指定数量的 Token(支持 fee-on-transfer 代币）
            function swapExactETHForTokensSupportingFeeOnTransferTokens(
                uint amountOutMin,
                address[] calldata path,
                address to,
                uint deadline
            ) external payable;
            // // 用指定数量的 Token 兑换指定数量的 ETH(支持 fee-on-transfer 代币）
            function swapExactTokensForETHSupportingFeeOnTransferTokens(
                uint amountIn,
                uint amountOutMin,
                address[] calldata path,
                address to,
                uint deadline
            ) external;
        }

    ```

总结下来，就是Uniswap V2的实现思路就是：
1. Uniswap V2核心模块中有一个 Factory 合约，用于管理交易对的——创建和查询交易对合约，并设置和查询手续费获取地址。任何人都可以用Factory合约来创建交易对，并保存新的交易对信息。

2. Uniswap上可以建立任意两种ERC20 token的交易对（Pair：token0和token1），任何人都可以将存放这两个币种的token到交易对的池子里(mint)，这个就是做市，做市商可以根据它放入池子里的钱在池子总金额所占的比例赚去相应份额的手续费。reserve0 和 reserve1 表示池子中 token0 和 token1 的数量。
做市商可以随时取出放进池子的金额和赚去的手续费，取出的金额根据它当时在池子里所占比例在算以及当前池子里的总金额来算，所以做市商最终取出的金额有可能要低于投入的金额，这就是无常损失。
任何用户都可以到池子里做交易——用token0兑换token1，或者token1兑换token0。在兑换中池子里的资产要满足一个公式 reserve0 * reserve1 = k。交易时保持k不变，做市商增加或者减少做市金额时候k才会发生变化。

3. 用户到池子里做交易的时候，输入的金额的0.3% 会作为手续费，给到做市商。


## Uniswap V3
最近刚在学习Uniswap V2的代码，就 V3 版本就来了，它将在5月5号启动以太坊L1主网，不久之后还将在Optimism的L2上进行部署。V3 版本在 V2 的基础上改进了,改善了V2版本中利池子里资金利用率不高的问题，同时Uniswap V3 为 LP 提供了三个级别的手续费。

> Uniswap v3 introduces:
> - **Concentrated liquidity,** giving individual LPs granular control over what price ranges their capital is allocated to. Individual positions are aggregated together into a single pool, forming one combined curve for users to trade against
> - **Multiple fee tiers** , allowing LPs to be appropriately compensated for taking on varying degrees of risk

另外，V3 版本预言机一次调用可以获取过去约 9 天内任何的 TWAP 价格，并且降低了Gas费用。

这些改动将使Uniswap V3更加高效灵活。

> These features make Uniswap v3 **the most flexible and efficient AMM ever designed**:
> - LPs can provide liquidity with **up to 4000x capital efficiency** relative to Uniswap v2, earning **higher returns on their capital**
> - Capital efficiency paves the way for low-slippage **trade execution that can surpass both centralized exchanges and stablecoin-focused AMMs**
> - LPs can significantly **increase their exposure to preferred assets** and **reduce their downside risk**
> - LPs can sell one asset for another by adding liquidity to a price range entirely above or below the market price, approximating **a fee-earning limit order that executes along a smooth curve**

    
### 合约分析
Uniswap V3 的合约代码结构与V2基本保持一致，也基本保持了简洁的特性。
Uniswap V2的合约同样分成两部分：Core 和 Periphery。
- 核心（Core）模板
  核心部分除了由一个工厂（Factory）合约和多个交易对（Pairs）合约组成外，还包括一个UniswapV3PoolDeployer 合约。
  - Factory
    ```js
    /// @title The interface for the Uniswap V3 Factory
    /// @notice The Uniswap V3 Factory facilitates creation of Uniswap V3 pools and control over the protocol fees
    interface IUniswapV3Factory {
        /// @notice Emitted when the owner of the factory is changed
        /// @param oldOwner The owner before the owner was changed
        /// @param newOwner The owner after the owner was changed
        event OwnerChanged(address indexed oldOwner, address indexed newOwner);

        /// @notice Emitted when a pool is created
        /// @param token0 The first token of the pool by address sort order
        /// @param token1 The second token of the pool by address sort order
        /// @param fee The fee collected upon every swap in the pool, denominated in hundredths of a bip
        /// @param tickSpacing The minimum number of ticks between initialized ticks
        /// @param pool The address of the created pool
        event PoolCreated(
            address indexed token0,
            address indexed token1,
            uint24 indexed fee,
            int24 tickSpacing,
            address pool
        );

        /// @notice Emitted when a new fee amount is enabled for pool creation via the factory
        /// @param fee The enabled fee, denominated in hundredths of a bip
        /// @param tickSpacing The minimum number of ticks between initialized ticks for pools created with the given fee
        event FeeAmountEnabled(uint24 indexed fee, int24 indexed tickSpacing);

        /// @notice Returns the current owner of the factory
        /// @dev Can be changed by the current owner via setOwner
        /// @return The address of the factory owner
        function owner() external view returns (address);

        /// @notice Returns the tick spacing for a given fee amount, if enabled, or 0 if not enabled
        /// @dev A fee amount can never be removed, so this value should be hard coded or cached in the calling context
        /// @param fee The enabled fee, denominated in hundredths of a bip. Returns 0 in case of unenabled fee
        /// @return The tick spacing
        function feeAmountTickSpacing(uint24 fee) external view returns (int24);

        /// @notice Returns the pool address for a given pair of tokens and a fee, or address 0 if it does not exist
        /// @dev tokenA and tokenB may be passed in either token0/token1 or token1/token0 order
        /// @param tokenA The contract address of either token0 or token1
        /// @param tokenB The contract address of the other token
        /// @param fee The fee collected upon every swap in the pool, denominated in hundredths of a bip
        /// @return pool The pool address
        function getPool(
            address tokenA,
            address tokenB,
            uint24 fee
        ) external view returns (address pool);

        /// @notice Creates a pool for the given two tokens and fee
        /// @param tokenA One of the two tokens in the desired pool
        /// @param tokenB The other of the two tokens in the desired pool
        /// @param fee The desired fee for the pool
        /// @dev tokenA and tokenB may be passed in either order: token0/token1 or token1/token0. tickSpacing is retrieved
        /// from the fee. The call will revert if the pool already exists, the fee is invalid, or the token arguments
        /// are invalid.
        /// @return pool The address of the newly created pool
        function createPool(
            address tokenA,
            address tokenB,
            uint24 fee
        ) external returns (address pool);

        /// @notice Updates the owner of the factory
        /// @dev Must be called by the current owner
        /// @param _owner The new owner of the factory
        function setOwner(address _owner) external;

        /// @notice Enables a fee amount with the given tickSpacing
        /// @dev Fee amounts may never be removed once enabled
        /// @param fee The fee amount to enable, denominated in hundredths of a bip (i.e. 1e-6)
        /// @param tickSpacing The spacing between ticks to be enforced for all pools created with the given fee amount
        function enableFeeAmount(uint24 fee, int24 tickSpacing) external;
    }
    ```
  - UniswapV3PoolDepolyer 合约
  ```js
    /// @title An interface for a contract that is capable of deploying Uniswap V3 Pools
    /// @notice A contract that constructs a pool must implement this to pass arguments to the pool
    /// @dev This is used to avoid having constructor arguments in the pool contract, which results in the init code hash
    /// of the pool being constant allowing the CREATE2 address of the pool to be cheaply computed on-chain
    interface IUniswapV3PoolDeployer {
        /// @notice Get the parameters to be used in constructing the pool, set transiently during pool creation.
        /// @dev Called by the pool constructor to fetch the parameters of the pool
        /// Returns factory The factory address
        /// Returns token0 The first token of the pool by address sort order
        /// Returns token1 The second token of the pool by address sort order
        /// Returns fee The fee collected upon every swap in the pool, denominated in hundredths of a bip
        /// Returns tickSpacing The minimum number of ticks between initialized ticks
        function parameters()
            external
            view
            returns (
                address factory,
                address token0,
                address token1,
                uint24 fee,
                int24 tickSpacing
            );
    }
    ```

    - pool
    V3中将交易对的合约拆分成池子的多个合约。
        - UniswapV3PoolActions 合约
            ```js
            /// @title Permissionless pool actions
            /// @notice Contains pool methods that can be called by anyone
            interface IUniswapV3PoolActions {
                /// @notice Sets the initial price for the pool
                /// @dev Price is represented as a sqrt(amountToken1/amountToken0) Q64.96 value
                /// @param sqrtPriceX96 the initial sqrt price of the pool as a Q64.96
                function initialize(uint160 sqrtPriceX96) external;
                /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
                /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
                /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
                /// on tickLower, tickUpper, the amount of liquidity, and the current price.
                /// @param recipient The address for which the liquidity will be created
                /// @param tickLower The lower tick of the position in which to add liquidity
                /// @param tickUpper The upper tick of the position in which to add liquidity
                /// @param amount The amount of liquidity to mint
                /// @param data Any data that should be passed through to the callback
                /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
                /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
                function mint(
                    address recipient,
                    int24 tickLower,
                    int24 tickUpper,
                    uint128 amount,
                    bytes calldata data
                ) external returns (uint256 amount0, uint256 amount1);
                /// @notice Collects tokens owed to a position
                /// @dev Does not recompute fees earned, which must be done either via mint or burn of any amount of liquidity.
                /// Collect must be called by the position owner. To withdraw only token0 or only token1, amount0Requested or
                /// amount1Requested may be set to zero. To withdraw all tokens owed, caller may pass any value greater than the
                /// actual tokens owed, e.g. type(uint128).max. Tokens owed may be from accumulated swap fees or burned liquidity.
                /// @param recipient The address which should receive the fees collected
                /// @param tickLower The lower tick of the position for which to collect fees
                /// @param tickUpper The upper tick of the position for which to collect fees
                /// @param amount0Requested How much token0 should be withdrawn from the fees owed
                /// @param amount1Requested How much token1 should be withdrawn from the fees owed
                /// @return amount0 The amount of fees collected in token0
                /// @return amount1 The amount of fees collected in token1
                function collect(
                    address recipient,
                    int24 tickLower,
                    int24 tickUpper,
                    uint128 amount0Requested,
                    uint128 amount1Requested
                ) external returns (uint128 amount0, uint128 amount1);

                /// @notice Burn liquidity from the sender and account tokens owed for the liquidity to the position
                /// @dev Can be used to trigger a recalculation of fees owed to a position by calling with an amount of 0
                /// @dev Fees must be collected separately via a call to #collect
                /// @param tickLower The lower tick of the position for which to burn liquidity
                /// @param tickUpper The upper tick of the position for which to burn liquidity
                /// @param amount How much liquidity to burn
                /// @return amount0 The amount of token0 sent to the recipient
                /// @return amount1 The amount of token1 sent to the recipient
                function burn(
                    int24 tickLower,
                    int24 tickUpper,
                    uint128 amount
                ) external returns (uint256 amount0, uint256 amount1);

                /// @notice Swap token0 for token1, or token1 for token0
                /// @dev The caller of this method receives a callback in the form of IUniswapV3SwapCallback#uniswapV3SwapCallback
                /// @param recipient The address to receive the output of the swap
                /// @param zeroForOne The direction of the swap, true for token0 to token1, false for token1 to token0
                /// @param amountSpecified The amount of the swap, which implicitly configures the swap as exact input (positive), or exact output (negative)
                /// @param sqrtPriceLimitX96 The Q64.96 sqrt price limit. If zero for one, the price cannot be less than this
                /// value after the swap. If one for zero, the price cannot be greater than this value after the swap
                /// @param data Any data to be passed through to the callback
                /// @return amount0 The delta of the balance of token0 of the pool, exact when negative, minimum when positive
                /// @return amount1 The delta of the balance of token1 of the pool, exact when negative, minimum when positive
                function swap(
                    address recipient,
                    bool zeroForOne,
                    int256 amountSpecified,
                    uint160 sqrtPriceLimitX96,
                    bytes calldata data
                ) external returns (int256 amount0, int256 amount1);

                /// @notice Receive token0 and/or token1 and pay it back, plus a fee, in the callback
                /// @dev The caller of this method receives a callback in the form of IUniswapV3FlashCallback#uniswapV3FlashCallback
                /// @dev Can be used to donate underlying tokens pro-rata to currently in-range liquidity providers by calling
                /// with 0 amount{0,1} and sending the donation amount(s) from the callback
                /// @param recipient The address which will receive the token0 and token1 amounts
                /// @param amount0 The amount of token0 to send
                /// @param amount1 The amount of token1 to send
                /// @param data Any data to be passed through to the callback
                function flash(
                    address recipient,
                    uint256 amount0,
                    uint256 amount1,
                    bytes calldata data
                ) external;

                /// @notice Increase the maximum number of price and liquidity observations that this pool will store
                /// @dev This method is no-op if the pool already has an observationCardinalityNext greater than or equal to
                /// the input observationCardinalityNext.
                /// @param observationCardinalityNext The desired minimum number of observations for the pool to store
                function increaseObservationCardinalityNext(uint16 observationCardinalityNext) external;
            }
            ```
        - UniswapV3PoolDerivedState
        ```js
        /// @title Pool state that is not stored
        /// @notice Contains view functions to provide information about the pool that is computed rather than stored on the
        /// blockchain. The functions here may have variable gas costs.
        interface IUniswapV3PoolDerivedState {
            /// @notice Returns a relative timestamp value representing how long, in seconds, the pool has spent between
            /// tickLower and tickUpper
            /// @dev This timestamp is strictly relative. To get a useful elapsed time (i.e., duration) value, the value returned
            /// by this method should be checkpointed externally after a position is minted, and again before a position is
            /// burned. Thus the external contract must control the lifecycle of the position.
            /// @param tickLower The lower tick of the range for which to get the seconds inside
            /// @param tickUpper The upper tick of the range for which to get the seconds inside
            /// @return A relative timestamp for how long the pool spent in the tick range
            function secondsInside(int24 tickLower, int24 tickUpper) external view returns (uint32);

            /// @notice Returns the cumulative tick and liquidity as of each timestamp `secondsAgo` from the current block timestamp
            /// @dev To get a time weighted average tick or liquidity-in-range, you must call this with two values, one representing
            /// the beginning of the period and another for the end of the period. E.g., to get the last hour time-weighted average tick,
            /// you must call it with secondsAgos = [3600, 0].
            /// @dev The time weighted average tick represents the geometric time weighted average price of the pool, in
            /// log base sqrt(1.0001) of token1 / token0. The TickMath library can be used to go from a tick value to a ratio.
            /// @param secondsAgos From how long ago each cumulative tick and liquidity value should be returned
            /// @return tickCumulatives Cumulative tick values as of each `secondsAgos` from the current block timestamp
            /// @return liquidityCumulatives Cumulative liquidity-in-range value as of each `secondsAgos` from the current block
            /// timestamp
            function observe(uint32[] calldata secondsAgos)
                external
                view
                returns (int56[] memory tickCumulatives, uint160[] memory liquidityCumulatives);
        }
        ```
        - UniswapV3PoolEvents
        ```js
        /// @title Events emitted by a pool
        /// @notice Contains all events emitted by the pool
        interface IUniswapV3PoolEvents {
            /// @notice Emitted exactly once by a pool when #initialize is first called on the pool
            /// @dev Mint/Burn/Swap cannot be emitted by the pool before Initialize
            /// @param sqrtPriceX96 The initial sqrt price of the pool, as a Q64.96
            /// @param tick The initial tick of the pool, i.e. log base 1.0001 of the starting price of the pool
            event Initialize(uint160 sqrtPriceX96, int24 tick);

            /// @notice Emitted when liquidity is minted for a given position
            /// @param sender The address that minted the liquidity
            /// @param owner The owner of the position and recipient of any minted liquidity
            /// @param tickLower The lower tick of the position
            /// @param tickUpper The upper tick of the position
            /// @param amount The amount of liquidity minted to the position range
            /// @param amount0 How much token0 was required for the minted liquidity
            /// @param amount1 How much token1 was required for the minted liquidity
            event Mint(
                address sender,
                address indexed owner,
                int24 indexed tickLower,
                int24 indexed tickUpper,
                uint128 amount,
                uint256 amount0,
                uint256 amount1
            );

            /// @notice Emitted when fees are collected by the owner of a position
            /// @dev Collect events may be emitted with zero amount0 and amount1 when the caller chooses not to collect fees
            /// @param owner The owner of the position for which fees are collected
            /// @param tickLower The lower tick of the position
            /// @param tickUpper The upper tick of the position
            /// @param amount0 The amount of token0 fees collected
            /// @param amount1 The amount of token1 fees collected
            event Collect(
                address indexed owner,
                address recipient,
                int24 indexed tickLower,
                int24 indexed tickUpper,
                uint128 amount0,
                uint128 amount1
            );

            /// @notice Emitted when a position's liquidity is removed
            /// @dev Does not withdraw any fees earned by the liquidity position, which must be withdrawn via #collect
            /// @param owner The owner of the position for which liquidity is removed
            /// @param tickLower The lower tick of the position
            /// @param tickUpper The upper tick of the position
            /// @param amount The amount of liquidity to remove
            /// @param amount0 The amount of token0 withdrawn
            /// @param amount1 The amount of token1 withdrawn
            event Burn(
                address indexed owner,
                int24 indexed tickLower,
                int24 indexed tickUpper,
                uint128 amount,
                uint256 amount0,
                uint256 amount1
            );

            /// @notice Emitted by the pool for any swaps between token0 and token1
            /// @param sender The address that initiated the swap call, and that received the callback
            /// @param recipient The address that received the output of the swap
            /// @param amount0 The delta of the token0 balance of the pool
            /// @param amount1 The delta of the token1 balance of the pool
            /// @param sqrtPriceX96 The sqrt(price) of the pool after the swap, as a Q64.96
            /// @param tick The log base 1.0001 of price of the pool after the swap
            event Swap(
                address indexed sender,
                address indexed recipient,
                int256 amount0,
                int256 amount1,
                uint160 sqrtPriceX96,
                int24 tick
            );

            /// @notice Emitted by the pool for any flashes of token0/token1
            /// @param sender The address that initiated the swap call, and that received the callback
            /// @param recipient The address that received the tokens from flash
            /// @param amount0 The amount of token0 that was flashed
            /// @param amount1 The amount of token1 that was flashed
            /// @param paid0 The amount of token0 paid for the flash, which can exceed the amount0 plus the fee
            /// @param paid1 The amount of token1 paid for the flash, which can exceed the amount1 plus the fee
            event Flash(
                address indexed sender,
                address indexed recipient,
                uint256 amount0,
                uint256 amount1,
                uint256 paid0,
                uint256 paid1
            );

            /// @notice Emitted by the pool for increases to the number of observations that can be stored
            /// @dev observationCardinalityNext is not the observation cardinality until an observation is written at the index
            /// just before a mint/swap/burn.
            /// @param observationCardinalityNextOld The previous value of the next observation cardinality
            /// @param observationCardinalityNextNew The updated value of the next observation cardinality
            event IncreaseObservationCardinalityNext(
                uint16 observationCardinalityNextOld,
                uint16 observationCardinalityNextNew
            );

            /// @notice Emitted when the protocol fee is changed by the pool
            /// @param feeProtocol0Old The previous value of the token0 protocol fee
            /// @param feeProtocol1Old The previous value of the token1 protocol fee
            /// @param feeProtocol0New The updated value of the token0 protocol fee
            /// @param feeProtocol1New The updated value of the token1 protocol fee
            event SetFeeProtocol(uint8 feeProtocol0Old, uint8 feeProtocol1Old, uint8 feeProtocol0New, uint8 feeProtocol1New);

            /// @notice Emitted when the collected protocol fees are withdrawn by the factory owner
            /// @param sender The address that collects the protocol fees
            /// @param recipient The address that receives the collected protocol fees
            /// @param amount0 The amount of token0 protocol fees that is withdrawn
            /// @param amount0 The amount of token1 protocol fees that is withdrawn
            event CollectProtocol(address indexed sender, address indexed recipient, uint128 amount0, uint128 amount1);
        }
        ```
        - UniswapV3PoolImmutables
        ```js
        /// @title Pool state that never changes
        /// @notice These parameters are fixed for a pool forever, i.e., the methods will always return the same values
        interface IUniswapV3PoolImmutables {
            /// @notice The contract that deployed the pool, which must adhere to the IUniswapV3Factory interface
            /// @return The contract address
            function factory() external view returns (address);

            /// @notice The first of the two tokens of the pool, sorted by address
            /// @return The token contract address
            function token0() external view returns (address);

            /// @notice The second of the two tokens of the pool, sorted by address
            /// @return The token contract address
            function token1() external view returns (address);

            /// @notice The pool's fee in hundredths of a bip, i.e. 1e-6
            /// @return The fee
            function fee() external view returns (uint24);

            /// @notice The pool tick spacing
            /// @dev Ticks can only be used at multiples of this value, minimum of 1 and always positive
            /// e.g.: a tickSpacing of 3 means ticks can be initialized every 3rd tick, i.e., ..., -6, -3, 0, 3, 6, ...
            /// This value is an int24 to avoid casting even though it is always positive.
            /// @return The tick spacing
            function tickSpacing() external view returns (int24);

            /// @notice The maximum amount of position liquidity that can use any tick in the range
            /// @dev This parameter is enforced per tick to prevent liquidity from overflowing a uint128 at any point, and
            /// also prevents out-of-range liquidity from being used to prevent adding in-range liquidity to a pool
            /// @return The max amount of liquidity per tick
            function maxLiquidityPerTick() external view returns (uint128);
        }
        ```
        - UniswapV3PoolOwnerActions
        ```js
        /// @title Permissioned pool actions
        /// @notice Contains pool methods that may only be called by the factory owner
        interface IUniswapV3PoolOwnerActions {
            /// @notice Set the denominator of the protocol's % share of the fees
            /// @param feeProtocol0 new protocol fee for token0 of the pool
            /// @param feeProtocol1 new protocol fee for token1 of the pool
            function setFeeProtocol(uint8 feeProtocol0, uint8 feeProtocol1) external;

            /// @notice Collect the protocol fee accrued to the pool
            /// @param recipient The address to which collected protocol fees should be sent
            /// @param amount0Requested The maximum amount of token0 to send, can be 0 to collect fees in only token1
            /// @param amount1Requested The maximum amount of token1 to send, can be 0 to collect fees in only token0
            /// @return amount0 The protocol fee collected in token0
            /// @return amount1 The protocol fee collected in token1
            function collectProtocol(
                address recipient,
                uint128 amount0Requested,
                uint128 amount1Requested
            ) external returns (uint128 amount0, uint128 amount1);
        }
        ```
        - UniswapV3PoolState
        ```js
        /// @title Pool state that can change
        /// @notice These methods compose the pool's state, and can change with any frequency including multiple times
        /// per transaction
        interface IUniswapV3PoolState {
            /// @notice The 0th storage slot in the pool stores many values, and is exposed as a single method to save gas
            /// when accessed externally.
            /// @return sqrtPriceX96 The current price of the pool as a sqrt(token1/token0) Q64.96 value
            /// tick The current tick of the pool, i.e. according to the last tick transition that was run.
            /// This value may not always be equal to SqrtTickMath.getTickAtSqrtRatio(sqrtPriceX96) if the price is on a tick
            /// boundary.
            /// observationIndex The index of the last oracle observation that was written,
            /// observationCardinality The current maximum number of observations stored in the pool,
            /// observationCardinalityNext The next maximum number of observations, to be updated when the observation.
            /// feeProtocol The protocol fee for both tokens of the pool.
            /// Encoded as two 4 bit values, where the protocol fee of token1 is shifted 4 bits and the protocol fee of token0
            /// is the lower 4 bits. Used as the denominator of a fraction of the swap fee, e.g. 4 means 1/4th of the swap fee.
            /// unlocked Whether the pool is currently locked to reentrancy
            function slot0()
                external
                view
                returns (
                    uint160 sqrtPriceX96,
                    int24 tick,
                    uint16 observationIndex,
                    uint16 observationCardinality,
                    uint16 observationCardinalityNext,
                    uint8 feeProtocol,
                    bool unlocked
                );

            /// @notice The fee growth as a Q128.128 fees of token0 collected per unit of liquidity for the entire life of the pool
            /// @dev This value can overflow the uint256
            function feeGrowthGlobal0X128() external view returns (uint256);

            /// @notice The fee growth as a Q128.128 fees of token1 collected per unit of liquidity for the entire life of the pool
            /// @dev This value can overflow the uint256
            function feeGrowthGlobal1X128() external view returns (uint256);

            /// @notice The amounts of token0 and token1 that are owed to the protocol
            /// @dev Protocol fees will never exceed uint128 max in either token
            function protocolFees() external view returns (uint128 token0, uint128 token1);

            /// @notice The currently in range liquidity available to the pool
            /// @dev This value has no relationship to the total liquidity across all ticks
            function liquidity() external view returns (uint128);

            /// @notice Look up information about a specific tick in the pool
            /// @param tick The tick to look up
            /// @return liquidityGross the total amount of position liquidity that uses the pool either as tick lower or
            /// tick upper,
            /// liquidityNet how much liquidity changes when the pool price crosses the tick,
            /// feeGrowthOutside0X128 the fee growth on the other side of the tick from the current tick in token0,
            /// feeGrowthOutside1X128 the fee growth on the other side of the tick from the current tick in token1,
            /// feeGrowthOutsideX128 values can only be used if the tick is initialized,
            /// i.e. if liquidityGross is greater than 0. In addition, these values are only relative and are used to
            /// compute snapshots.
            function ticks(int24 tick)
                external
                view
                returns (
                    uint128 liquidityGross,
                    int128 liquidityNet,
                    uint256 feeGrowthOutside0X128,
                    uint256 feeGrowthOutside1X128
                );

            /// @notice Returns 256 packed tick initialized boolean values. See TickBitmap for more information
            function tickBitmap(int16 wordPosition) external view returns (uint256);

            /// @notice Returns 8 packed tick seconds outside values. See SecondsOutside for more information
            function secondsOutside(int24 wordPosition) external view returns (uint256);

            /// @notice Returns the information about a position by the position's key
            /// @param key The position's key is a hash of a preimage composed by the owner, tickLower and tickUpper
            /// @return _liquidity The amount of liquidity in the position,
            /// Returns feeGrowthInside0LastX128 fee growth of token0 inside the tick range as of the last mint/burn/poke,
            /// Returns feeGrowthInside1LastX128 fee growth of token1 inside the tick range as of the last mint/burn/poke,
            /// Returns tokensOwed0 the computed amount of token0 owed to the position as of the last mint/burn/poke,
            /// Returns tokensOwed1 the computed amount of token1 owed to the position as of the last mint/burn/poke
            function positions(bytes32 key)
                external
                view
                returns (
                    uint128 _liquidity,
                    uint256 feeGrowthInside0LastX128,
                    uint256 feeGrowthInside1LastX128,
                    uint128 tokensOwed0,
                    uint128 tokensOwed1
                );

            /// @notice Returns data about a specific observation index
            /// @param index The element of the observations array to fetch
            /// @dev You most likely want to use #observe() instead of this method to get an observation as of some amount of time
            /// ago, rather than at a specific index in the array.
            /// @return blockTimestamp The timestamp of the observation,
            /// Returns tickCumulative the current tick multiplied by seconds elapsed for the life of the pool as of the
            /// observation,
            /// Returns liquidityCumulative the current liquidity multiplied by seconds elapsed for the life of the pool as of
            /// the observation,
            /// Returns initialized whether the observation has been initialized and the values are safe to use
            function observations(uint256 index)
                external
                view
                returns (
                    uint32 blockTimestamp,
                    int56 tickCumulative,
                    uint160 liquidityCumulative,
                    bool initialized
                );
        }
        ```
(未完)