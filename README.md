<p align="center">
  <img src="assets/logo.png" alt="creditcoin-dapp-skill" width="200px">
</p>

<p align="center">
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square" alt="License: MIT"></a>
  <a href="https://creditcoin.org"><img src="https://img.shields.io/badge/Creditcoin-Network-orange.svg?style=flat-square" alt="Creditcoin Network"></a>
  <a href="https://docs.creditcoin.org"><img src="https://img.shields.io/badge/Solidity-EVM-blue.svg?style=flat-square" alt="Solidity"></a>
  <a href="https://www.typescriptlang.org/"><img src="https://img.shields.io/badge/TypeScript-Active-3178C6.svg?style=flat-square&logo=typescript" alt="TypeScript"></a>
</p>

<p align="center">
  <strong>A <a href="https://github.com/anthropics/skills">Claude Code skill</a> for building RWA dApps on Creditcoin Network.</strong>
  <br>
  <a href="https://docs.creditcoin.org">Creditcoin Docs</a> · <a href="#quick-start">Quick Start</a> · <a href="https://github.com/gluwa/creditcoin">Creditcoin GitHub</a>
</p>

## What it does

This skill gives Claude Code deep knowledge of the Creditcoin development stack so it can help you:

- **Scaffold** Creditcoin dApps with Solidity contracts and TypeScript frontends
- **Write** EVM-compatible smart contracts for credit history and RWA
- **Deploy** using Hardhat, Foundry, or Remix
- **Interact** with the Creditcoin ledger using Ethers.js and Web3.js
- **Record** credit performance on-chain
- **Build USC** cross-chain dApps with trustless multichain verification

## Quick Start

```bash
bash <(curl -s https://raw.githubusercontent.com/phamdat721101/creditcoin-dapp-skills/main/install.sh)
```

Then start Claude Code and ask it to build something:

```
> Create a Creditcoin contract for a loan system
```

## Stack

| Layer | Tool | Notes |
|:------|:-----|:------|
| Smart Contracts | **Solidity** | EVM-compatible language |
| Client SDK | **Ethers.js** | Interact with Creditcoin EVM |
| Wallet | **MetaMask** | Managing assets and signing transactions |
| Runtime | **Node.js 18+** | Required environment |

<details>
<summary><strong>Prerequisites</strong></summary>

<br>

- [Node.js](https://nodejs.org/) 18+
- [Git](https://git-scm.com/)
- [MetaMask](https://metamask.io/)
- VS Code (Recommended)

</details>

## Usage examples

Once installed, start a Claude Code session and try:

```
> Help me start a new Creditcoin dApp
> Write a Solidity contract for a loan offer
> Explain how credit history works in Creditcoin
> Create a script to deploy my contract
```

### USC Cross-Chain Examples

```
> Build a USC contract that aggregates credit history from Ethereum and Solana
> Create a trustless bridge contract using STARK proof verification
> Write a cross-chain collateral system where ETH backs Creditcoin loans
> Implement a unified credit identity that tracks repayments across 3 chains
```

## Skill structure

```
creditcoin-dapp-skill/
├── SKILL.md                            # Main skill definition
├── references/
│   ├── creditcoin-smart-contract.md    # Solidity guide
│   ├── creditcoin-client-sdk.md        # JS SDK guide
│   ├── creditcoin-setup.md             # Setup guide
│   ├── usc-smart-contract.md           # USC contract guide
│   └── usc-cross-chain.md              # Cross-chain patterns
├── install.sh
├── assets/
│   └── logo.png
└── README.md
```

## Resources

| Resource | Description |
|:---------|:------------|
| [Creditcoin Docs](https://docs.creditcoin.org) | Official documentation |
| [Creditcoin Explorer](https://explorer.creditcoin.org) | Block explorer |
| [Creditcoin GitHub](https://github.com/gluwa/creditcoin) | Open source tools/libs |

## License

[MIT](LICENSE)
