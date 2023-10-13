

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

    
/// @notice Initialize the state for a given pool ID
function initialize(PoolKey memory key, uint160 sqrtPriceX96, bytes calldata hookData)
external
returns (int24 tick);
```
Here are the main pieces of this code:

1. **Event for Pool Initialization**:
    - `event Initialize(...)`: This is like an announcement system. Whenever a new pool is set up, this "event" will send out details about it. The details it shares include the unique identity of the pool (`id`), the two types of digital currencies in the pool (`currency0` and `currency1`), the fee charged for swapping currencies in the pool (`fee`), the space between special points called "ticks" (`tickSpacing`), and any additional rules or actions (`hooks`) that might be attached to the pool.

2. **Error Messages**:
    - `error TickSpacingTooLarge()`: This error message pops up if someone tries to set the space between "ticks" too large during the pool's setup.
    - `error TickSpacingTooSmall()`: Similarly, this error pops up if the space between "ticks" is set too small.
    - `error CurrenciesInitializedOutOfOrder()`: This message comes up if the digital currencies in the pool are not arranged in the right order.

3. **Function to Initialize the Pool**:
    - `function initialize(...)`: This is the main action to start or "initialize" a new pool. When someone wants to create a new pool, they call this function and provide some essential details. These details include a `key` that identifies the types of currencies in the pool and some initial settings like the starting price. The function then sets up the pool and returns the starting "tick."

4. **Limits on Tick Spacing**:
    - `function MAX_TICK_SPACING()`: This function tells you the maximum allowed space between "ticks" when setting up a pool.
    - `function MIN_TICK_SPACING()`: Similarly, this function provides the smallest allowed space between "ticks."

In essence, these pieces ensure that when a new pool is created, it's set up correctly, adhering to the rules, and everyone is informed about its details.

# PoolManager.sol
https://github.com/Uniswap/v4-core/blob/main/contracts/PoolManager.sol

```solidity
/// @inheritdoc IPoolManager
function initialize(PoolKey memory key, uint160 sqrtPriceX96, bytes calldata hookData)
external
override
returns (int24 tick)
{
    if (key.fee.isStaticFeeTooLarge()) revert FeeTooLarge();

    // see TickBitmap.sol for overflow conditions that can arise from tick spacing being too large
    if (key.tickSpacing > MAX_TICK_SPACING) revert TickSpacingTooLarge();
    if (key.tickSpacing < MIN_TICK_SPACING) revert TickSpacingTooSmall();
    if (key.currency0 > key.currency1) revert CurrenciesInitializedOutOfOrder();
    if (!key.hooks.isValidHookAddress(key.fee)) revert Hooks.HookAddressNotValid(address(key.hooks));

    if (key.hooks.shouldCallBeforeInitialize()) {
        if (key.hooks.beforeInitialize(msg.sender, key, sqrtPriceX96, hookData) != IHooks.beforeInitialize.selector)
        {
            revert Hooks.InvalidHookResponse();
        }
    }

    PoolId id = key.toId();
    uint24 protocolFees = _fetchProtocolFees(key);
    uint24 hookFees = _fetchHookFees(key);
    tick = pools[id].initialize(sqrtPriceX96, protocolFees, hookFees);

    if (key.hooks.shouldCallAfterInitialize()) {
        if (
            key.hooks.afterInitialize(msg.sender, key, sqrtPriceX96, tick, hookData)
                != IHooks.afterInitialize.selector
        ) {
            revert Hooks.InvalidHookResponse();
        }
    }

    emit Initialize(id, key.currency0, key.currency1, key.fee, key.tickSpacing, key.hooks);
}
```
Sure, let's explain this function in a straightforward manner.

---

function `initialize` is used to set up a new pool with specific settings.

Here's what initialize does step-by-step:

1. **Check the Fee**:
   - If the fee for using the pool is too high, then revert error called `FeeTooLarge`.

2. **Check the Tick Spacing**:
   - "Ticks" are like markers in the pool. There's a space between these markers.
   - If the space between the markers is too large, then revert with error `TickSpacingTooLarge`.
   - If it's too small, then revert with error `TickSpacingTooSmall`.

3. **Ensure the Digital Currencies are in Order**:
   - Keys should be sorted numerically
   - If they are in the wrong order, then revert with error `CurrenciesInitializedOutOfOrder`.

4. **Check the Hooks**:

5. **Set Up the Pool**:
   - Create a unique `id` for the pool.
   - Actually set up the pool with the given starting price and fees.

6. **After Setup Actions**:
   - Some hooks might have actions to do after the pool is set up. If they don't respond correctly, revert with `InvalidHookResponse`.

7. **Announce the Pool**:
   - Finally, announce to everyone that a new pool has been created, sharing all its details.

# PoolKey
```solidity
/// @notice Returns the key for identifying a pool
struct PoolKey {
    /// @notice The lower currency of the pool, sorted numerically
    Currency currency0;
    /// @notice The higher currency of the pool, sorted numerically
    Currency currency1;
    /// @notice The pool swap fee, capped at 1_000_000. The upper 4 bits determine if the hook sets any fees.
    uint24 fee;
    /// @notice Ticks that involve positions must be a multiple of tick spacing
    int24 tickSpacing;
    /// @notice The hooks of the pool
    IHooks hooks;
}
```

# Hook Deployment
```solidity
uint256 internal constant BEFORE_INITIALIZE_FLAG = 1 << 159;
uint256 internal constant AFTER_INITIALIZE_FLAG = 1 << 158;


/// @notice Utility function intended to be used in hook constructors to ensure
/// the deployed hooks address causes the intended hooks to be called
/// @param calls The hooks that are intended to be called
/// @dev calls param is memory as the function will be called from constructors
function validateHookAddress(IHooks self, Calls memory calls) internal pure {
    if (
        calls.beforeInitialize != shouldCallBeforeInitialize(self)
            || calls.afterInitialize != shouldCallAfterInitialize(self)
            || calls.beforeModifyPosition != shouldCallBeforeModifyPosition(self)
            || calls.afterModifyPosition != shouldCallAfterModifyPosition(self)
            || calls.beforeSwap != shouldCallBeforeSwap(self) || calls.afterSwap != shouldCallAfterSwap(self)
            || calls.beforeDonate != shouldCallBeforeDonate(self) || calls.afterDonate != shouldCallAfterDonate(self)
    ) {
        revert HookAddressNotValid(address(self));
    }
}

function shouldCallBeforeInitialize(IHooks self) internal pure returns (bool) {
    return uint256(uint160(address(self))) & BEFORE_INITIALIZE_FLAG != 0;
}

function shouldCallAfterInitialize(IHooks self) internal pure returns (bool) {
    return uint256(uint160(address(self))) & AFTER_INITIALIZE_FLAG != 0;
}
```



# References
https://link.excalidraw.com/l/ABfjq3uKuGl/3vqFljWwYXG 

https://uniswaphooks.com/

https://docs.uniswapfoundation.org/overview/conduit-testnet 

GitHub - kadenzipfel/uni-lbp: A capital-efficient Uniswap v4 liquidity bootstrapping pool (LBP) hooks contract
