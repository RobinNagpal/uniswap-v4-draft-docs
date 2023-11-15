# Developer Guide for Using the Uniswap v4 Template Repository

#### Overview
The Uniswap v4 Template repository is designed to help developers write hooks for the Uniswap v4 protocol. It includes an example hook (`Counter.sol`), a test template, and scripts for deploying hooks and managing liquidity pools.

#### Getting Started
1. **Installation**: The template requires Foundry. Install dependencies using:
   ```
   forge install
   ```

2. **Testing**: Run tests with:
   ```
   forge test
   ```

#### Local Development with Anvil
1. **Start Anvil**: Anvil is used for local development due to Ethereum's bytecode limit and business licensing of v4. To start Anvil:
   ```
   anvil --code-size-limit 30000
   ```

2. **Deploying Scripts on Anvil**: In a new terminal, deploy your script with:
   ```
   forge script script/Anvil.s.sol \
       --rpc-url http://localhost:8545 \
       --private-key [your_private_key] \
       --code-size-limit 30000 \
       --broadcast
   ```

#### Testing on Goerli Testnet
1. **Uniswap Foundation's Slimmed Down v4 Contract**: Use the provided addresses for testing on the Goerli Testnet.

2. **Setting Up RPC-URL**: Obtain an RPC-URL from Infura and run scripts on Foundry for Goerli Testnet. For example:
   ```
   forge script script/CounterDeploy.s.sol \
   --rpc-url [your_rpc_url_here] \
   --private-key [your_private_key_on_goerli_here] \
   --broadcast
   ```

#### Deploying Your Own Tokens
1. **Creating Mock Tokens**: The template includes Mock UNI and Mock USDC contracts for testing. Deploy them using:
   ```
   forge create script/mocks/mUNI.sol:MockUNI \
   --rpc-url [your_rpc_url_here] \
   --private-key [your_private_key_on_goerli_here] \

   forge create script/mocks/mUSDC.sol:MockUSDC \
   --rpc-url [your_rpc_url_here] \
   --private-key [your_private_key_on_goerli_here] \
   ```

#### Troubleshooting
1. **Permission Denied**: Resolve by configuring SSH keys as detailed in the README.
2. **Hook Deployment Failures**: Ensure correct flags and salt mining. If problems persist, consult the README for detailed troubleshooting steps.

#### Additional Resources
- **v4-periphery**: For advanced hook implementations.
- **v4-core**: The core repository for Uniswap v4.

This template is a comprehensive toolkit for Uniswap v4 hook development, streamlining the process from local development to deployment on testnets.
