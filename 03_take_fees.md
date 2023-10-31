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

For each iteration, it calculates the swap step, which includes the fee calculation:

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

The FeeAmounts struct in this Pool contract represents the amounts of fees collected from a 
liquidity operation for both the protocol and the pool's fee hook.

It contains 4 fields:

feeForProtocol0: The fee amount collected in token0 for the protocol
feeForProtocol1: The fee amount collected in token1 for the protocol
feeForHook0: The fee amount collected in token0 for the fee hook
feeForHook1: The fee amount collected in token1 for the fee hook
