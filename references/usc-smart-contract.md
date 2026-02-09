# USC Smart Contract Development

Universal Smart Contracts (USC) enable trustless cross-chain interoperability using STARK proofs.

## Core Concepts

### Inheritance

All USC contracts inherit from `UniversalSmartContract_Core`:

```solidity
import "@gluwa/usc/UniversalSmartContract_Core.sol";

contract MyUSCContract is UniversalSmartContract_Core {
    // Your cross-chain logic
}
```

### Cross-Chain Verification

Use `_processOracleResults()` to retrieve and verify STARK proofs:

```solidity
function verifyCrossChainEvent(
    bytes32 queryId,
    address proverContract
) external {
    bytes memory result = _processOracleResults(proverContract, queryId);
    
    // Extract data from verified proof
    address sender = _extractAddress(result, 0);
    uint256 amount = _extractUint256(result, 32);
    uint256 chainId = _extractChainId(result, 64);
    
    // Execute business logic
}
```

## Helper Functions

| Function | Description |
|:---------|:------------|
| `_extractAddress(bytes, offset)` | Extract address from proof result |
| `_extractUint256(bytes, offset)` | Extract uint256 from proof result |
| `_extractChainId(bytes, offset)` | Extract source chain ID |

## RWA Credit Profile Pattern

Standard pattern for Real World Asset credit scoring:

```solidity
struct CreditProfile {
    uint256 totalBorrowed;
    uint256 totalRepaid;
    uint256 onTimePayments;
    uint256 latePayments;
}

function calculateScore(address user) public view returns (uint256) {
    CreditProfile memory p = profiles[user];
    uint256 score = 500; // Base score
    score += p.onTimePayments * 10;
    score -= p.latePayments * 20;
    return score > 850 ? 850 : score;
}
```

## Installation

```bash
npm install @gluwa/creditcoin-usc
```

## Best Practices

-   Validate all cross-chain proofs before state changes
-   Use typed extraction helpers for proof data
-   Emit events for all cross-chain verifications
-   Test with testnet attestors before mainnet deployment
