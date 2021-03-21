# DeFi 漫游笔记（一） —— uniswap 的简洁力量

*我很喜欢坐高铁，近两年越发喜欢在高铁上做的一件事就是看/写代码，那种低头思考，抬头看苍茫大地在眼前跳跃的感觉真的非常惬意。第一次读 uniswap 的代码就是在去年十一期间的高铁上，那也是我看过的第一份 DeFi 项目的合约代码，虽然有点迟，但为时不晚。最大的感受就是如此简洁有效的设计，真是贯彻了区块链的精髓。*

Uniswap是以太坊上的一个去中心化交易协议。在Uniswap的官网有一篇文章介绍了它的发展历程（[地址链接](https://uniswap.org/blog/uniswap-history/)）。早在2017年Vitalik就写过一篇文章介绍过这个方案。
> If you want to make a market maker for existing tokens without a price cap, my favorite (credit to Martin Koppelmann) mechanism is that which maintains the invariant.
 tokenA_balance(p) * tokenB_balance(p) = k for some constant k. 

目前Uniswap已经从V0版本演化到V2版本，官网有完整的介绍[文档](https://uniswap.org/docs/v2/protocol-overview/how-uniswap-works).

> Uniswap is an automated liquidity protocol powered by a constant product formula and implemented in a system of non-upgradeable smart contracts on the Ethereum blockchain. It obviates the need for trusted intermediaries, prioritizing decentralization, censorship resistance, and security. 

Uniswap中有几个关键角色和关键词：
- LP(liquidity provider)
    LP 是指给交易池子提供流动性的提供商，在Uniswap中，任何人都可以变成池子的LP，他只需要拿等值的基础代币做抵押，来换取池子的 Token，再按照这些代币的比例分配LP份额获得收益，并且可以随时赎回资产。

- Pairs
    从Uniswap V2工厂部署的智能合约，可以在两个ERC20代币之间进行交易。Pair执行做市商的角色，交易对维持一个最简单的公式：x * y = k。它表示交易不得更改池子里Token的余额（x和y）的乘积（k）。

Uniswap的合约分成两部分：Core 和 Periphery。
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

(未完)