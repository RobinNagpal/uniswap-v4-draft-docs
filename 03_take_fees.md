# Background on Fees
In a liquidity pool, users can provide liquidity by depositing tokens. These positions can be modified 
by either adding or removing liquidity. Fees are also involved in these modifications - collected from 
traders who swap tokens in the pool and distributed to liquidity providers.

![Fees](/images/03_fee_calculation/FeeCalculation.png)

# Swap - Step by Step

The swap happens in a loop until the specified amount has been completely used or the price limit 
has been reached. In each iteration, the code calculates how much of the tokens can be swapped at 
the current price level.

The swap keeps iterating until either the specified amount is fully used or the square root of 
the price hits the defined limit (`sqrtPriceLimitX96`).

# Fees in Uniswap v3
In Uniswap v3, following struct and properties are used to represent fees:

https://github.com/Uniswap/v3-core/blob/main/contracts/UniswapV3Pool.sol
```solidity
  // accumulated protocol fees in token0/token1 units
    struct ProtocolFees {
        uint128 token0;
        uint128 token1;
    }
    /// @inheritdoc IUniswapV3PoolState
    ProtocolFees public override protocolFees;
```

A fee is calculated for each swap iteration. It is determined based on the liquidity and the amount being 
swapped in that iteration.

Each swapping loop iteration starts a new swap step. As part of each iteration following properties are being calculated:
- `step.feeAmount`: This is the fee calculated in each swap step, and it's dependent on the liquidity and price movement within that step.
- `state.protocolFee`: This accumulates the total protocol fees during the entire swap.
- `feeGrowthGlobalX128`: This keeps track of the global fee growth, which will be used for liquidity providers to calculate their earned fees.


# Fees in Uniswap v4
In Uniswap v4, in addition to the protocol fees, there can also fees for the hooks. 

The FeeAmounts struct in this Pool library represents the amounts of fees collected from a 
liquidity operation for both the protocol and the pool's hook.

```solidity
    struct FeeAmounts {
        uint256 feeForProtocol0;
        uint256 feeForProtocol1;
        uint256 feeForHook0;
        uint256 feeForHook1;
    }

```
Here is a breakdown of each field:

`feeForProtocol0` and `feeForProtocol1`: These are fees that are directed towards the protocol. This is similar to the fees in Uniswap v3.

`feeForHook0` and `feeForHook1`: These are fees related to a specific operation or integration referred to as 
a "hook" in this code. The fees collected from these hooks may be directed towards the entity that provides the hook 
functionality. Similar to the protocol fees, they are specific to two different assets/tokens in the pool.


If you look at the swapping code in v4, the fees are calculated for each swap step just like in Uniswap v3
https://github.com/Uniswap/v4-core/blob/main/contracts/libraries/Pool.sol#L378


# IHookFeeManager
The `IHookFeeManager` interface is used to manage the fees for the hooks. It's defined as follows:

```solidity
/// @notice The interface for setting a fee on swap or fee on withdraw to the hook
/// @dev This callback is only made if the Fee.HOOK_SWAP_FEE_FLAG or Fee.HOOK_WITHDRAW_FEE_FLAG in set in the pool's key.fee.
interface IHookFeeManager {
    /// @notice Gets the fee a hook can take at swap/withdraw. Upper bits used for swap and lower bits for withdraw.
    /// @param key The pool key
    /// @return The hook fees for swapping (upper bits set) and withdrawing (lower bits set).
    function getHookFees(PoolKey calldata key) external view returns (uint24);
}
```
