# USC Cross-Chain Capabilities

USC provides native multichain interoperability without third-party oracles or bridges.

## Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Source Chain│ --> │ Attestation │ --> │    STARK    │ --> │     USC     │
│ (ETH/BTC/SOL)│     │    Layer    │     │   Proofs    │     │  Contract   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

### Components

| Layer | Description |
|:------|:------------|
| **Source Chains** | Any L1/L2 (Ethereum, Bitcoin, Solana, Polygon, etc.) |
| **Attestation Layer** | Decentralized network monitoring source chain events |
| **Proof Generation** | STARK proofs verifying data exists in attested history |
| **USC Layer** | Execution environment evaluating cryptographic proofs |

## Supported Source Chains

-   **Ethereum** (Mainnet, Sepolia)
-   **Bitcoin** (Mainnet, Testnet)
-   **Solana** (Mainnet, Devnet)
-   **Polygon** (Mainnet, Mumbai)

## Cross-Chain Use Cases

### 1. Unified Credit Identity

Aggregate credit history from multiple chains:

```solidity
function aggregateCreditHistory(bytes32[] memory proofIds) external {
    for (uint i = 0; i < proofIds.length; i++) {
        bytes memory result = _processOracleResults(prover, proofIds[i]);
        uint256 chainId = _extractChainId(result, 64);
        // Merge into unified profile
    }
}
```

### 2. Trustless Bridge (Burn/Mint)

```solidity
function claimBridgedAsset(bytes32 burnProofId) external {
    bytes memory proof = _processOracleResults(prover, burnProofId);
    address recipient = _extractAddress(proof, 0);
    uint256 amount = _extractUint256(proof, 32);
    
    require(recipient == msg.sender, "Invalid recipient");
    _mint(recipient, amount);
}
```

### 3. Cross-Chain Collateral

Use ETH collateral for Creditcoin loans:

```solidity
function depositCrossChainCollateral(bytes32 depositProofId) external {
    bytes memory proof = _processOracleResults(prover, depositProofId);
    uint256 ethValue = _extractUint256(proof, 32);
    
    collateral[msg.sender] += ethValue;
    emit CollateralDeposited(msg.sender, ethValue, "ETH");
}
```

## Why USC is Unique

| Feature | Traditional Bridges | USC |
|:--------|:-------------------|:----|
| Trust Model | Trusted operators | Cryptographic (STARK) |
| Latency | Minutes to hours | Block confirmation + proof |
| Security | Bridge exploits possible | Math-based verification |
| State Access | Limited | Native cross-chain reads |

## Network Endpoints

| Network | RPC URL |
|:--------|:--------|
| Testnet | `https://testnet.creditcoin.network` |
| Mainnet | `https://mainnet.creditcoin.network` |

## Explorers

-   **EVM Explorer**: `explorer.creditcoin.org`
-   **Substrate Explorer**: `subscan.io/creditcoin`
