Question - after running forge install, the v4-periphery needs to be updated with `forge update v4-periphery`
Answer - yeah when i update my dependencies i usually just forge remove and forge install 🥴️️️️️️


Question - For testing purpose how can we mock an address for validateHookAddress()? I assume we need to create an address that trigger the correct hook we want to execute.
Answer -  You can reference the tests in v4-periphery. You can also see this in V4 template also has example: https://github.com/uniswapfoundation/v4-template/blob/main/test/Counter.t.sol#L30-L37


Question - when I use .modifyPosition in poolmanager it show Arithmetic over/underflow. Do you know how to add liquidity
Answer -  If there's no lock acquired, the math underflows at LockDataLibrary
https://github.com/Uniswap/v4-core/blob/bc94eb0f2074e366ea3386adb1a7b9978c8d1070/contracts/libraries/LockDataLibrary.sol#L48

Question - Does anyone have a resource where we can track the most innovative application of v4 Hooks?
Answer - check out this new website one of our UF Ambassadors put together for hooks and all the ones being created so far. Not a complete list but continuing to add! https://uniswaphooks.com/

Question - since v4 is going to use ticks, is there an updated version of the tick math library? 
Answer - Yes - https://github.com/Uniswap/v4-core/blob/main/src/libraries/TickMath.sol


Question - so observe() isn't going to be available in IPoolManager, we'll have to redeploy our pools and then, i guess there's gonna be a standard impl  that has observe in it. is that right?
Answer - v3-style TWAP will be available as a hook with permanent, full-range liquidity https://github.com/Uniswap/v4-periphery/blob/main/contracts/hooks/examples/GeomeanOracle.sol#L140.  Truncated Oracle hook also gives some of the similar functionality https://blog.uniswap.org/uniswap-v4-truncated-oracle-hook

Question - Hey, any updates regarding hook licensing?
Answer - you're talking about the interfaces and types in core right? (e.g. those mentioned here: https://github.com/Uniswap/v4-core#license)




Security Risks - https://phalcon.xyz/blog/thorns-in-the-rose-exploring-security-risks-in-uniswap-v4-s-novel-hook-mechanism

Question - hey guys quick question, this is more of a v4 question. I'm using a smart contract to add liquidity into a 
uniswap pool. In the IPoolManager, I'm not understanding which function exactly does this? Are there any code examples of how to do this?
Answer - provisioning liquidity will typically be done via a periphery/router contract. these contracts will be the ones calling poolManager.modifyPosition
example: https://github.com/Uniswap/v4-core/blob/main/contracts/test/PoolModifyPositionTest.sol
inside your tests you would use this contract to provision liquidity

 
Question - - in this context what do you mean by "provisioning liquidity" does this just mean setting positions and whatnot? 

Answer - provisioning liquidity as in "minting an LP". i.e. specfiying the price range and sending the tokens. the link above is the periphery contract. example usage here: https://github.com/saucepoint/v4-template/blob/main/test/Counter.t.sol#L45C19-L50
with deployment here: https://github.com/saucepoint/v4-template/blob/main/test/utils/HookTest.sol#L46 in v3, when you LP for a pool you get an ERC721 NFT. The actual minting of the receipt token, balance accounting, and fee claiming is done via the NonfungiblePositionManager, which is entirely separate from V3Pool.sol (and the factory) same will be true for v4


Question -  I'm trying out a project to use a contract as a proxy to anonymously add liquidity to a uniswap pool. I add the pool manager directly as variables directly into my contract but seems to be that it should be that I should be using a router contract instead for that? Are there any cons to just interacting directly with the pool manager? 
Answer -  router contract would possibly be better especially if you dont need any highly customized LPing logic

its possible to interact directly with the poolmanager, you'll need to call manager.lock and define lockAcquired to call manager.modifyPosition (much like PoolModifyPositionTest)

the main con with this approach is rolling-your-own LP accounting. if you have multiple users using PrivacyProxy, it will responsible for knowing who owns what and how much

Question - I understand the purpose of the lock on the pool but I'm confused on what type of logic managed in this function. Is it to ensure the delta returns to 0?
Answer - yeah at the end of lockAcquired, the delta should be 0 (you dont owe the pool manager, and the pool manager doesnt owe you)

lockAcquired will have different implementations because that's when manager.modifyPosition or manager.swap is callable

you can chain as many operations as you want (single swaps, multihop swaps, swap-n-add, rebalance, flashloans, etc)


Question - Hey everyone, does anyone have any good examples of how swaps are implemented in V4? Is it similar to making a swap in V3? I'm trying to make a swap on a pool from a treasury contract
Answer - if you're looking to do a simple swap, here's how to use PoolSwapTest from v4-core
https://github.com/saucepoint/v4-template/blob/main/test/utils/HookTest.sol#L59C10-L69

but if you're looking for actual implementation details and tick math behind a swap:
https://github.com/Uniswap/v4-core/blob/main/contracts/libraries/Pool.sol#L378


Question - Is there any in depth exlpanation on the LockDataLibarary, i want to understand the potential use cases of lockers and how it works, from natspec it's not much clear yet.
Answer - Is there any in depth exlpanation on the LockDataLibarary, i want to understand the potential use cases of lockers and how it works, from natspec it's not much clear yet.

Question - when a user swaps, how does it gets decided which before and and after swap hooks should be triggered?. Virtually there could be 1000s of such hooks on a pool. A swapper wouldn't certainly have to choose this. Is there a router for this?
Answer - trades will probably route through pools that offer the best prices (accounting for gas). swappers wont choose, and itll be uniswap (or aggregators) to do routing. at the moment there is no offchain router im aware of
the term router is probably overloaded lol
so there's offchain routing -- i.e. an aggregator finding the best pools to use, or searching for multi-hop swaps
and then theres periphery routers. smart contracts which handle the actual token transfers. i.e. the smart contracts that interact with the pool manager

Question - Hello, I am trying to implement a Uniswap v4 hook (simple profit taking hook for now) and followed a guide. As of now my tests are failing at one point: https://github.com/Spurrya/UniLasso/blob/fa21ebe83de15e346b24d45f4e3a23308e02ad79/test/TakeProfitsHook.t.sol#L228 and failing with [FAIL. Reason: HookNotImplemented()]. I am seeing online examples and they seem to be passing ZERO_BYTES but it is failing on my end - any pointers?
Answer -  i think you need to add override to your afterSwap function

https://github.com/Spurrya/UniLasso/blob/fa21ebe83de15e346b24d45f4e3a23308e02ad79/src/TakeProfitsHook.sol#L178C26-L178C26

regarding the ZERO_BYTES stuff -- a few weeks ago, labs introduced a change where swap and modifyPosition can pass arbitrary bytes to the hook (enabling flexibility)

its possible the guide is out of date with the latest changes, but i do see your poolManager.swap providing empty bytes 😉


Question - Can the liquidity be added by EOAs? If not why?
Answer - liquidity is added via a periphery contract PoolModifyPositionTest (0x092D53306f9Df9eeD35efec24c31Ca32000033BC)

you cant directly call PoolManager.modifyPosition because EOAs/wallets cant "lock" with the pool manager
it’s bc EOAs can’t handle callbacks which is how lock accounting solvency is enforced.

Question - Is lockAquired where the logic from swap occurs?
Answer - yes, but lockAcquired is also used for LP creation/deletion. lockAcquired is where token transfers can happen (i.e. swaps, LP, and flashloans)

the main functions poolManager.swap and poolManager.modifyPosition require a lock, so the callback lockAcquired is typically where operations will happen


Question -  Wanted to understand, if i'm just building a liquidity provision contract on top of v4-core aka it's not a hook, during the modifyPosition operation for token transfers for adding liquidity, do i compulsory need to call lock and implement ILockCallback?

Also if any pool creator doesn't decide to integrate a hook, what do they pass in the hookData arguemnt of initialize function

Answer - Yeap, any interaction with PoolManager requires calling lock and handling callbck, therefore you cannot do it directly on PM from EOA. if you dont need arbitrary hookData on pool initialization, you can provide new bytes(0) in solidity or "0x0" (string) in typescript
