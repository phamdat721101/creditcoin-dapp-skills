---
name: creditcoin-dapp-skill
description: Guide for building Real World Asset (RWA) dApps on Creditcoin Network using Solidity and EVM tools. Covers smart contract development, credit history recording, and frontend integration. Use when starting a new Creditcoin project, writing Solidity contracts, or building a frontend that interacts with the Creditcoin ledger.
---

# Creditcoin dApp Development

## Stack

| Layer | Tool | Notes |
|-------|------|-------|
| Smart Contracts | **Solidity** | EVM-compatible smart contracts (Hardhat, Foundry). |
| Client SDK | **Ethers.js** / **Web3.js** | TypeScript/JS SDKs for interacting with EVM contracts. |
| Wallet | **MetaMask** / **Polkadot.js** | MetaMask for EVM interactions; Polkadot.js for native Substrate features. |
| Network | **Creditcoin Mainnet/Testnet** | EVM Chain ID: 102031 (Mainnet). |
| Runtime | Node.js 18+ | Required for the JS client and toolchain. |

## Decision Flow

When starting a new Creditcoin project:

1.  **Define Credit Needs**: Identify if you need to record credit history/performance on-chain.
2.  **Write Contract**: Use **Solidity** (`.sol` files) to implement the logic.
3.  **Deploy**: Use `Hardhat` or `Remix` to deploy to the Creditcoin EVM.
4.  **Build Frontend**: Use **Ethers.js** to interact with your deployed contracts.

## Project Scaffolding

### Recommended Structure

```
my-creditcoin-dapp/
├── contracts/
│   └── MyContract.sol         # Solidity contract logic
├── src/
│   ├── abis/                  # Contract ABIs
│   └── components/            # Frontend components
├── test/                      # Contract tests
├── hardhat.config.ts          # Hardhat config
└── package.json
```

### Bootstrap steps

```bash
# 1. Initialize project
mkdir my-creditcoin-dapp && cd my-creditcoin-dapp
npm init -y

# 2. Install dependencies
npm install --save-dev hardhat
npm install ethers dotenv

# 3. Create contract project
npx hardhat init
```

## Core Workflow

### 1. Write Contract (Solidity)

```solidity
// contracts/CreditRecord.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CreditRecord {
    struct Loan {
        address borrower;
        uint256 amount;
        bool repaid;
    }

    mapping(uint256 => Loan) public loans;
    uint256 public loanCount;

    function createLoan(uint256 _amount) public {
        loanCount++;
        loans[loanCount] = Loan(msg.sender, _amount, false);
    }

    function repayLoan(uint256 _id) public {
        require(msg.sender == loans[_id].borrower, "Not the borrower");
        loans[_id].repaid = true;
    }
}
```

### 2. Compile

```bash
npx hardhat compile
```

### 3. Deploy & Interact (Ethers.js)

```typescript
import { ethers } from "ethers";

// Connect to Creditcoin EVM
const provider = new ethers.JsonRpcProvider("https://mainnet.creditcoin.network");
const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider);

// Deploy or Connect
const factory = new ethers.ContractFactory(abi, bytecode, wallet);
const contract = await factory.deploy();
await contract.waitForDeployment();

console.log("Contract deployed at:", await contract.getAddress());
```

## Principles

-   **Credit History**: Leverage Creditcoin's transparent ledger to build verifiable credit history.
-   **RWA Integration**: Design for Real World Assets to be recorded on-chain.
-   **EVM Compatibility**: existing Ethereum tools work out of the box.

## References

-   `references/creditcoin-smart-contract.md` — Guide to Solidity on Creditcoin.
-   `references/creditcoin-client-sdk.md` — Using Ethers.js / Web3.js.
-   `references/creditcoin-setup.md` — Environment setup (MetaMask, RPC).
