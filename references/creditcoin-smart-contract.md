# Creditcoin Smart Contract Development (Solidity)

Creditcoin is an EVM-compatible chain built on Substrate. You can write smart contracts using **Solidity** and deploy them using standard Ethereum tools like **Hardhat**, **Foundry**, or **Remix**.

## Core Concepts

### EVM Compatibility

Creditcoin's EVM layer allows you to:
-   Deploy standard ERC-20, ERC-721, and custom Solidity contracts.
-   Use existing Ethereum tooling and libraries (OpenZeppelin, etc.).
-   Interact with contracts using MetaMask.

### Network Details

-   **Mainnet Chain ID**: `102031`
-   **Testnet Chain ID**: `102030`
-   **Native Token**: CTC
-   **RPC URL**: `https://mainnet.creditcoin.network` (Mainnet) / `https://rpc.testnet.creditcoin.network` (Testnet)

## Writing Contracts

### Basic Example

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimpleCredit {
    event LoanCreated(address borrower, uint256 amount);

    function requestLoan(uint256 amount) external {
        emit LoanCreated(msg.sender, amount);
    }
}
```

## Compilation (Hardhat)

1.  **Install Hardhat**:
    ```bash
    npm install --save-dev hardhat
    npx hardhat init
    ```

2.  **Configure `hardhat.config.ts`**:
    ```typescript
    import { HardhatUserConfig } from "hardhat/config";
    import "@nomicfoundation/hardhat-toolbox";
    require("dotenv").config();

    const config: HardhatUserConfig = {
      solidity: "0.8.19",
      networks: {
        creditcoin: {
          url: "https://mainnet.creditcoin.network",
          accounts: [process.env.PRIVATE_KEY!],
          chainId: 102031,
        },
        creditcoinTestnet: {
            url: "https://rpc.testnet.creditcoin.network",
            accounts: [process.env.PRIVATE_KEY!],
            chainId: 102030,
        },
      },
    };

    export default config;
    ```

3.  **Compile**:
    ```bash
    npx hardhat compile
    ```

## Deployment

Create a script `scripts/deploy.ts`:

```typescript
import { ethers } from "hardhat";

async function main() {
  const credit = await ethers.deployContract("SimpleCredit");
  await credit.waitForDeployment();

  console.log(`Contract deployed to ${await credit.getAddress()}`);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

Run deployment:
```bash
npx hardhat run scripts/deploy.ts --network creditcoin
```

## Best Practices

-   **Credit History**: When designing dApps, consider how to leverage Creditcoin's specific features for recording credit performance (ask for specific Substrate interaction guides if bridging is needed).
-   **Security**: Always audit your Solidity code. Use tools like Slither.
