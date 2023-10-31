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
PoolManger while initialization calls Hooks library to check if the hooks are deployed at the proper addresses

Hooks library checks the starting (or leading) bits of the hook contract's address. For instance, if a hook contract
is deployed at address 0x9000000000000000000000000000000000000000, its starting bits are '1001'. This means two
specific hooks ('before initialize' and 'after modify position') will be triggered.

To generate valid hook addresses based on the code provided, we'll focus on the leading bits that indicate which hooks are invoked. Each flag corresponds to specific leading bits in the address, as indicated by the constants provided.

Here are some example addresses based on the flags:

1. **Just BEFORE_INITIALIZE_FLAG**
   Address: `0x8000000000000000000000000000000000000000`
   Leading bits: '1000...'
   Hooks: 'before initialize'

3. **Just BEFORE_MODIFY_POSITION_FLAG**
   Address: `0x2000000000000000000000000000000000000000`
   Leading bits: '0010...'
   Hooks: 'before modify position'

5. **BEFORE_INITIALIZE_FLAG and AFTER_INITIALIZE_FLAG**
   Address: `0xC000000000000000000000000000000000000000`
   Leading bits: '1100...'
   Hooks: 'before initialize' and 'after initialize'

6. **All Flags Activated**
   Address: `0xFF00000000000000000000000000000000000000`
   Leading bits: '11111111...'
   Hooks: 'before initialize', 'after initialize', 'before modify position', 'after modify position', 'before swap', 'after swap', 'before donate', and 'after donate'.

![Hooks Address](/images/01_Pool_Initialization/hooks_address.png)

# CREATE2
Ethereum blockchain allows you to create contracts (think of them as mini-programs). There are two ways to create these contracts:

1. **CREATE**: This is the regular way. Every time you create a contract using this, it gets a new, unique address (like a house getting a unique postal address). You can't fully predict this address in advance.

2. **CREATE2**: This is a special way. Here, you use some ingredients (your address, a `salt` which is a unique number you choose, and the contract's code called `bytecode`) to create the contract. The magic of `CREATE2` is that if you use the same ingredients, you'll get the same contract address every time.

So, why use `CREATE2`? Because sometimes you want to know exactly where your contract will be, even before you create it. Like in your hooks system, where certain addresses trigger certain actions, using `CREATE2` helps ensure the contract is deployed to the exact right address.

Here's a small code to show how `CREATE2` works:
```solidity
bytes32 salt = keccak256(abi.encodePacked(someData));
address predictedAddress = address(uint(keccak256(abi.encodePacked(
    byte(0xff),
    deployerAddress,
    salt,
    keccak256(bytecode)
))));
```
This code predicts the address where a contract will be deployed using `CREATE2` before actually deploying it.
# References
https://link.excalidraw.com/l/ABfjq3uKuGl/3vqFljWwYXG

https://uniswaphooks.com/

https://docs.uniswapfoundation.org/overview/conduit-testnet

GitHub - kadenzipfel/uni-lbp: A capital-efficient Uniswap v4 liquidity bootstrapping pool (LBP) hooks contract



https://github.com/Uniswap/v4-periphery/issues/59#issuecomment-1716379675

not much documentation related to it

but here’s the source

https://github.com/Arachnid/deterministic-deployment-proxy
