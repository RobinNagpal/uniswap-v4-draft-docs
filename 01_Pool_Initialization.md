

# Architecture
![image](https://github.com/RobinNagpal/uniswap-v4-draft-docs/assets/745748/2c9568c1-33c6-4655-a31f-a0488446d86a)

# IPoolManager.sol
https://github.com/Uniswap/v4-core/blob/main/contracts/interfaces/IPoolManager.sol 

```solidity
    /// @notice Emitted when a new pool is initialized
    /// @param id The abi encoded hash of the pool key struct for the new pool
    /// @param currency0 The first currency of the pool by address sort order
    /// @param currency1 The second currency of the pool by address sort order
    /// @param fee The fee collected upon every swap in the pool, denominated in hundredths of a bip
    /// @param tickSpacing The minimum number of ticks between initialized ticks
    /// @param hooks The hooks contract address for the pool, or address(0) if none
    event Initialize(
        PoolId indexed id,
        Currency indexed currency0,
        Currency indexed currency1,
        uint24 fee,
        int24 tickSpacing,
        IHooks hooks
    );
```



# References
https://link.excalidraw.com/l/ABfjq3uKuGl/3vqFljWwYXG 

https://uniswaphooks.com/

https://docs.uniswapfoundation.org/overview/conduit-testnet 

GitHub - kadenzipfel/uni-lbp: A capital-efficient Uniswap v4 liquidity bootstrapping pool (LBP) hooks contract
