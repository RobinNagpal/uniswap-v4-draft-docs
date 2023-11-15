## Testing on Goerli Testnet
1. **Uniswap Foundation's Slimmed Down v4 Contract**: Use the provided addresses for testing on the Goerli Testnet.

2. **Setting Up RPC-URL**: Obtain an RPC-URL from Infura and run scripts on Foundry for Goerli Testnet. For example:
   ```
   forge script script/CounterDeploy.s.sol \
   --rpc-url [your_rpc_url_here] \
   --private-key [your_private_key_on_goerli_here] \
   --broadcast
   ```

## Deploying Your Own Tokens
1. **Creating Mock Tokens**: The template includes Mock UNI and Mock USDC contracts for testing. Deploy them using:
   ```
   forge create script/mocks/mUNI.sol:MockUNI \
   --rpc-url [your_rpc_url_here] \
   --private-key [your_private_key_on_goerli_here] \

   forge create script/mocks/mUSDC.sol:MockUSDC \
   --rpc-url [your_rpc_url_here] \
   --private-key [your_private_key_on_goerli_here] \
   ```
