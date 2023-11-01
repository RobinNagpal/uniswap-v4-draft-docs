# Swap Tokens
As explained in the fees section, the swap happens in a loop until the specified amount has been completely 
used or the price limit has been reached. In each iteration, the code calculates how much of the tokens can 
be swapped at the current price level.

The swap keeps iterating until either the specified amount is fully used or the square root of
the price hits the defined limit (`sqrtPriceLimitX96`).

Here is the important structs and `swap` function from IPoolManager which is used to swap tokens:

```solidity
/// @notice Emitted for swaps between currency0 and currency1
/// @param id The abi encoded hash of the pool key struct for the pool that was modified
/// @param sender The address that initiated the swap call, and that received the callback
/// @param amount0 The delta of the currency0 balance of the pool
/// @param amount1 The delta of the currency1 balance of the pool
/// @param sqrtPriceX96 The sqrt(price) of the pool after the swap, as a Q64.96
/// @param liquidity The liquidity of the pool after the swap
/// @param tick The log base 1.0001 of the price of the pool after the swap
event Swap(
    PoolId indexed id,
    address indexed sender,
    int128 amount0,
    int128 amount1,
    uint160 sqrtPriceX96,
    uint128 liquidity,
    int24 tick,
    uint24 fee
);

struct SwapParams {
    bool zeroForOne;
    int256 amountSpecified;
    uint160 sqrtPriceLimitX96;
}

/// @notice Swap against the given pool
function swap(PoolKey memory key, SwapParams memory params, bytes calldata hookData)
    external
    returns (BalanceDelta);
```

# Example
Here is an example of how to swap tokens by calling the `swap` function:
```solidity
PoolKey memory key = PoolKey({
    currency0: Currency.wrap(address(0)),
    currency1: currency1,
    fee: 3000,
    hooks: IHooks(address(0)),
    tickSpacing: 60
});

IPoolManager.SwapParams memory params =
    IPoolManager.SwapParams({zeroForOne: true, amountSpecified: 100, sqrtPriceLimitX96: SQRT_RATIO_1_2});


manager.initialize(key, SQRT_RATIO_1_1, ZERO_BYTES);

manager.swap(key, params, ZERO_BYTES);
```

# Acquiring Lock
Full detail about the locking mechanism is explained in the [Locking Mechanism](/03_Locking_Mechanism/README.md) section.

The contract that calls the `swap` must implement ILockCallback interface.

PoolSwapTest.sol has some examples of how to acquire lock and some basic checks in place.
https://github.com/Uniswap/v4-core/blob/main/contracts/test/PoolSwapTest.sol

In `PoolSwapTest` the `lockAcquired` function is triggered when a certain "lock" is acquired in the 
context of swapping assets or tokens. Once this lock is confirmed, the function handles 
the settlement based on the result of the swap. It decodes the data provided to understand the context, asks 
the manager to perform the swap, and then either 
transfers, withdraws, or mints tokens based on the balance changes resulting from the swap.

To ensure secure operations, the function checks that it's only being called by the intended `manager` contract. 
Depending on the type of swap and settings, it handles the balance adjustments for two types of currencies: 
`currency0` and `currency1`. After settling all balances, it returns the balance changes to the caller.
