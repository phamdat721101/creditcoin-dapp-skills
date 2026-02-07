# Creditcoin Client SDK Guide

Interacting with Creditcoin's EVM layer is identical to interacting with Ethereum or Polygon. You can use standard libraries like **Ethers.js**, **Web3.js**, or **Viem**.

## Installation

```bash
npm install ethers
# or
npm install web3
```

## connecting to Creditcoin

### Using Ethers.js v6

```typescript
import { ethers } from "ethers";

// Mainnet RPC
const RPC_URL = "https://mainnet.creditcoin.network";

// Provider
const provider = new ethers.JsonRpcProvider(RPC_URL);

// Wallet (if using private key directly - Be careful in frontend!)
const privateKey = process.env.PRIVATE_KEY;
const wallet = new ethers.Wallet(privateKey, provider);

// Check balance
const balance = await provider.getBalance(wallet.address);
console.log(`Balance: ${ethers.formatEther(balance)} CTC`);
```

## Browser Integration (MetaMask)

To connect a frontend dApp to Creditcoin via MetaMask:

```typescript
async function connectWallet() {
  if (typeof window.ethereum !== 'undefined') {
    try {
      // Request account access
      await window.ethereum.request({ method: 'eth_requestAccounts' });

      // Switch to Creditcoin Mainnet
      await window.ethereum.request({
        method: 'wallet_addEthereumChain',
        params: [{
          chainId: '0x18E87', // 102031 in hex
          chainName: 'Creditcoin Mainnet',
          nativeCurrency: {
            name: 'Creditcoin',
            symbol: 'CTC',
            decimals: 18
          },
          rpcUrls: ['https://mainnet.creditcoin.network'],
          blockExplorerUrls: ['https://explorer.creditcoin.org']
        }]
      });
      
      const provider = new ethers.BrowserProvider(window.ethereum);
      const signer = await provider.getSigner();
      console.log("Connected:", await signer.getAddress());
      return signer;
      
    } catch (error) {
      console.error("User denied account access or error occurred", error);
    }
  } else {
    console.log("MetaMask is not installed!");
  }
}
```

## Interacting with Contracts

```typescript
const contractAddress = "0x...";
const abi = [...]; // Your Contract ABI

const contract = new ethers.Contract(contractAddress, abi, signer);

// Call a read function
const value = await contract.someReadFunction();

// Call a write function
const tx = await contract.someWriteFunction("arg1", "arg2");
await tx.wait(); // Wait for confirmation
console.log("Transaction confirmed:", tx.hash);
```
