

# Position
To add or remove the tokens we call the function `modifyPosition` which is defined in `Pool.sol` file


```solidity
struct ModifyPositionParams {
    // the address that owns the position
    address owner;
    // the lower and upper tick of the position
    int24 tickLower;
    int24 tickUpper;
    // any change in liquidity
    int128 liquidityDelta;
    // the spacing between ticks
    int24 tickSpacing;
}


/// @dev Effect changes to a position in a pool
/// @param params the position details and the change to the position's liquidity to effect
/// @return result the deltas of the token balances of the pool
function modifyPosition(State storage self, ModifyPositionParams memory params)
internal
returns (BalanceDelta result, FeeAmounts memory fees)
{
    // Implementation
}
```

# Tick

Ticks basically represent token  prices of a pair .

Means - smallest amount  possible by which the price can go up or down

price (i) = 1.001 ^ i  ; *( where i is our tick )*

Let’s say we have Token A and Token B

pools has initial tick equals to zero

Token A and B are equal initially so the initial price of the Token A is

price(i=0) = 1.001^0 = 1 ;

some purchase happen where they sell some token B and buy some token A which cause the price of Token A to go up and B to go down

then suppose tick will reduce i = -50 ;

price(i = -50) = 1.001 ^(-50)= 0.995012

1 token A = 0.995012 of Token B

Ticks are intergers but they’re use to find the value of assets which are fractional values as solidity cannot store float values they use ticks for these things

Now Again suppose there is a token A that can only move in increment or decrement of  1 dollar , only possible values for ticks then become 1 ,2, 3, ,4 and so on .

but if some sell and buy happens in that particular stock it doesn’t means that the tick will get adjusted to some 1 ,2,3, kind of values it can be 0.6 or 0.7 but ticks are spaced to at intervals of 1 hence

We round the actual tick to either `tickLower` or `tickHigher`  , here 0 and 1 respectively


# Liquidity Delta


The `liquidityDelta` is the difference between the current liquidity and the desired liquidity in a position. It can be positive (when you're adding liquidity) or negative (when you're removing liquidity).

To calculate `liquidityDelta` when calling `ModifyPosition`:

1. **Determine the Current Liquidity**:
    - If you have an existing position, you should know its current liquidity. If not, you can query it from the contract using the position key (which is often a combination of the user's address and tick range).

2. **Determine Desired Liquidity**:
    - This is based on how much liquidity you want to add or remove. For instance, if you want to increase your position's liquidity by `X`, then `desiredLiquidity = currentLiquidity + X`. If you want to reduce it by `Y`, then `desiredLiquidity = currentLiquidity - Y`.

3. **Calculate liquidityDelta**:
   ``` 
   liquidityDelta = desiredLiquidity - currentLiquidity 
   ```
![Liquidity Delta](/images/02_Managing_Position/LiquidityDelta.png)

![Liquidity Delta 1](/images/02_Managing_Position/LiquidityDelta_1.png)


Here are some practical steps:

a. **Find Current Position Details**:
Use the `positions` function on the Uniswap v3 NFT positions manager to get details about your current position:
   ```solidity
   (,,uint128 currentLiquidity,,,,,) = positionsManager.positions(positionKey);
   ```

b. **Determine Desired Liquidity**:
Calculate how much liquidity you want in the position after the modification. This can be based on the amount of tokens you're adding or removing.

c. **Calculate `liquidityDelta`**:
   ```solidity
   int256 liquidityDelta = int256(desiredLiquidity) - int256(currentLiquidity);
   ```

When you have `liquidityDelta`, you can use it as a parameter for the `ModifyPosition` function.


Here are the steps to calculate the `desiredLiquidity`
1) Define your price range: the tickLower and tickUpper values for your liquidity position.
2) **Compute the amount of liquidity for that price range**: Uniswap v3 provides a function to help you compute the amount of liquidity for a given amount of token and a price range. You can use the `getAmountsForLiquidity` function from the Uniswap v3 core library. This function tells you how much liquidity a certain amount of token will represent in a given price range:

    ```solidity
    (uint160 sqrtPriceX96,,,,) = pool.slot0();
    
    uint128 maxLiquidity = LiquidityAmounts.getLiquidityForAmounts(
        sqrtPriceX96,
        tickLower,
        tickUpper,
        amount0,
        amount1
    );
    ```

   In this example, `pool` is the Uniswap v3 pool contract for the token pair you're interested in. The `amountToken` is the amount of the token you want to provide. The function returns the `liquidity` value which corresponds to the amount of liquidity that your token amount will represent between the given ticks.

3)**The `desiredLiquidity`**:

   If you're adding liquidity, your `desiredLiquidity` will be:
   ```solidity
   desiredLiquidity = currentLiquidity + liquidity;
   ```

   If you're removing liquidity, your `desiredLiquidity` will be:
   ```solidity
   desiredLiquidity = currentLiquidity - liquidity;
   ```

   Remember that `currentLiquidity` is obtained from your current position details.

# Tick Spacing
The distance between these ticks is not uniform across all pools but is defined by a parameter called `tickSpacing`. This `tickSpacing` is determined by the pool's fee tier.

For example, for the 0.05% fee tier, the `tickSpacing` might be 10. For the 0.3% fee tier, it might be 60, and so on. This means that for a pool with a `tickSpacing` of 10, valid tick values could be ..., -20, -10, 0, 10, 20, ... and so forth. You cannot, for instance, set a tick position at 5 or 15 in this example.

When you're creating or modifying a position in Uniswap v3, you must ensure that the ticks (`tickLower` and `tickUpper`) you choose are valid given the pool's `tickSpacing`. They should be multiples of the `tickSpacing` value.


```solidity
int24 tickLower = (currentTick / tickSpacing) * tickSpacing;
int24 tickUpper = tickLower + tickSpacing;
```

Here's an example:

1. **Ensuring valid ticks when creating a position**:

   Let's say you're dealing with a pool that has a `tickSpacing` of 60 (like the 0.3% fee tier). If the current tick value you're getting is 1234, then:

   ```solidity
   int24 tickSpacing = 60;
   int24 currentTick = 1234; // This is a hypothetical value, you'd get it from the pool.
   int24 tickLower = (currentTick / tickSpacing) * tickSpacing;  // This will be 1200
   int24 tickUpper = tickLower + tickSpacing;  // This will be 1260
   ```

2. **Modifying a position with `tickSpacing`**:

   If you're modifying a position and want to adjust the tick range, ensure that the new range adheres to the `tickSpacing`:

   ```solidity
   int24 tickSpacing = 60;
   int24 newDesiredTick = 1350; // Hypothetical desired tick.
   int24 tickLower = (newDesiredTick / tickSpacing) * tickSpacing;  // This will be 1320
   int24 tickUpper = tickLower + tickSpacing;  // This will be 1380
   ```

   Now, when calling `modifyPosition` or any other function that requires tick values, you'd use `tickLower` and `tickUpper` as arguments.

# Add/Remove Liquidity

```solidity
 int24 tickLower = -600;
    int24 tickUpper = 600;
    uint256 liquidity = 1e18;
    
    lpm.modifyPosition(
        address(this),
        poolKey,
        IPoolManager.ModifyPositionParams({
            tickLower: tickLower,
            tickUpper: tickUpper,
            liquidityDelta: int256(liquidity)
        }),
        ZERO_BYTES
    );

    // recieved 1e18 LP tokens (6909)
    Position memory position = Position({poolKey: poolKey, tickLower: tickLower, tickUpper: tickUpper});
    assertEq(lpm.balanceOf(address(this), position.toTokenId()), liquidity);

```


```solidity
 // assume liquidity has been provisioned
    int24 tickLower = -600;
    int24 tickUpper = 600;
    uint256 liquidity = 1e18;

    // remove all liquidity
    lpm.modifyPosition(
        address(this),
        poolKey,
        IPoolManager.ModifyPositionParams({
            tickLower: tickLower,
            tickUpper: tickUpper,
            liquidityDelta: -int256(liquidity)
        }),
        ZERO_BYTES
    );

```
