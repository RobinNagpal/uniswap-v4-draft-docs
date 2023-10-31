

# Position
To add or remove the tokens we call the function `modifyPosition` which is defined in `PoolManager.sol` file

```solidity
struct ModifyPositionParams {
   // the lower and upper tick of the position
   int24 tickLower;
   int24 tickUpper;
   // how to modify the liquidity
   int256 liquidityDelta;
}

/// @notice Modify the position for the given pool
function modifyPosition(PoolKey memory key, ModifyPositionParams memory params, bytes calldata hookData)
external
returns (BalanceDelta);
```

# Important Concepts
Some of the important concepts to understand when working with Uniswap v3 or v4 positions are:

1) Tick
2) Tick Spacing
4) SquareRoot Price X96
5) Liquidity Delta

### Tick
`tick` is a measure used in this code to handle prices of two different assets (tokens) in a unique way. A `tick` 
represents a specific price ratio between these two assets, calculated using a mathematical formula.

There are minimum and maximum `tick` values defined within the code, ensuring that calculated prices are within a 
reasonable or acceptable range.

In Uniswap v3(and v4), liquidity providers can provide liquidity at specific price ranges (ticks), allowing them to 
concentrate their liquidity and potentially earn more fees.

Each tick corresponds to a specific price, and not all prices are represented due to the discrete nature of the ticks.

### Tick Spacing
The tick spacing is a parameter that determines the separation between these usable ticks, making only every 
Nth tick available for liquidity provision, where `N` is the tick spacing. This is a form of quantization of 
the price levels that liquidity can be provided at.

![Price And Ticks](/images/02_Managing_Position/PriceAndTicks.png)

### SquareRoot Price X96
In Uniswap v3(and v4), the square root price (`sqrtPriceX96`) is a key concept and a crucial part of the mathematical 
calculations. It's utilized for various calculations, including determining the amount of tokens that should be moved 
during a swap and the liquidity calculations within specific price ranges.

Here’s a breakdown of what `sqrtPriceX96` represents:

### 1. **Square Root Price:**
The price is represented as the square root of the actual price ratio of the tokens. This representation 
simplifies the math, especially when it comes to calculating the amounts to be swapped, as well as the 
liquidity calculations within tick ranges.

### 2. **X96:**
The X96 suffix refers to the fixed-point format used. Uniswap v3 utilizes a 96-bit fixed-point number format. 
In this representation, there is a convention to maintain high precision in calculations. The fixed-point 
representation means that the actual floating-point number is multiplied by \(2^{96}\) and stored as an 
integer. When reading the value, it has to be interpreted properly by dividing it by \(2^{96}\) to get the 
actual value.


![SqrtPriceX96](/images/02_Managing_Position/SqrtPriceX96.png)

### SqrtPriceX96 to Tick
Since both the `tick` and `sqrtPriceX96` are representations of the price, they can be converted from one to the other.

The Uniswap v3/v4 core library provides two functions to convert between the two representations:

https://github.com/Uniswap/v4-core/blob/main/contracts/libraries/TickMath.sol

1. The function `getSqrtRatioAtTick` takes a `tick` value as an input, and it computes the square root of the price 
ratio of the two assets at that specific `tick`. The result represents the price relationship between two 
assets in a particular state or position.

2. The `getTickAtSqrtRatio` function does the reverse—it takes a square root of a price ratio and calculates 
the corresponding `tick`. This `tick` value represents a position or state where the assets have the given 
price relationship.

![sqrtPriceX96 to Tick](/images/02_Managing_Position/sqrtPriceX96_to_tick.png)


### 4. **Updating the sqrtPriceX96:**
- The `sqrtPriceX96` is not a constant; it changes as transactions occur within a pool, reflecting the current state and price ratio of the tokens within the pool.

In conclusion, the `sqrtPriceX96` is a specific representation of the price in the Uniswap v3 protocol, designed to optimize and simplify the mathematical calculations involved in swaps and liquidity management within the protocol. It is a dynamic value that reflects the current pricing state of the pool.

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
