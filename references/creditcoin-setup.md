# Creditcoin Environment Setup

## 1. Prerequisites

-   **Node.js**: v18 or cleaner.
-   **Package Manager**: `npm` or `yarn`.
-   **Wallet**: MetaMask (Browser Extension).

## 2. Wallet Setup (MetaMask)

To develop on Creditcoin, you need to configure your EVM wallet.

### Mainnet Configuration

-   **Network Name**: Creditcoin Mainnet
-   **RPC URL**: `https://mainnet.creditcoin.network`
-   **Chain ID**: `102031`
-   **Currency Symbol**: `CTC`
-   **Block Explorer**: `https://explorer.creditcoin.org`

### Testnet Configuration

-   **Network Name**: Creditcoin Testnet
-   **RPC URL**: `https://rpc.testnet.creditcoin.network`
-   **Chain ID**: `102030`
-   **Currency Symbol**: `CTC` (Testnet)
-   **Block Explorer**: `https://explorer.testnet.creditcoin.org` (!verify url)

## 3. Getting Funds (Testnet)

1.  Join the [Creditcoin Discord](https://discord.gg/creditcoin).
2.  Go to the `#faucet` channel.
3.  Request testnet CTC: `/faucet <your-address>`.

## 4. Development Environment

### VS Code Extensions

-   **Solidity**: Nomic Foundation's solidity extension.
-   **Prettier**: For code formatting.

### Initializing a Project

```bash
mkdir my-dapp
cd my-dapp
npm init -y
npm install --save-dev hardhat
npx hardhat init
```
