# USC Cross-Chain Credit Scoring

Build a unified credit score from repayment data across Ethereum, Bitcoin, Solana, and Polygon — verified trustlessly via STARK proofs. Users earn a soulbound NFT representing their credit tier.

> **Prerequisites**: Familiarity with USC basics in `usc-smart-contract.md` and cross-chain architecture in `usc-cross-chain.md`.

## Contracts Overview

| Contract | Role |
|:---------|:-----|
| `ICreditScore` | Read interface for external consumers |
| `CrossChainCreditScore` | Proof verification, score calculation, relayer management |
| `CreditTierNFT` | Soulbound ERC-721 with tier badges |

## 1. ICreditScore Interface

External contracts and dApps consume scores through this interface — keeps consumers decoupled from implementation.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

interface ICreditScore {
    enum Tier { None, Bronze, Silver, Gold, Platinum }

    function getScore(address user) external view returns (uint256);
    function getTier(address user) external view returns (Tier);
    function getRepaymentStats(address user)
        external view returns (uint256 onTime, uint256 late);
}
```

## 2. CrossChainCreditScore Contract

Handles three responsibilities: relayer access control, proof-based repayment recording, and score computation.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@gluwa/usc/UniversalSmartContract_Core.sol";
import "./ICreditScore.sol";

contract CrossChainCreditScore is UniversalSmartContract_Core, ICreditScore {

    // --- State ---

    struct RepaymentRecord {
        uint256 onTime;
        uint256 late;
        uint256 chainsUsed;   // bitmap: bit0=ETH, bit1=BTC, bit2=SOL, bit3=Polygon
    }

    mapping(address => RepaymentRecord) private records;
    mapping(address => bool) public relayers;
    address public owner;
    address public tierNFT;

    uint256 private constant BASE_SCORE = 300;
    uint256 private constant MAX_SCORE = 850;
    uint256 private constant ON_TIME_WEIGHT = 15;
    uint256 private constant LATE_PENALTY = 30;
    uint256 private constant CHAIN_BONUS = 25;

    // --- Events ---

    event RepaymentRecorded(address indexed user, uint256 chainId, bool onTime);
    event RelayerUpdated(address indexed relayer, bool active);

    // --- Modifiers ---

    modifier onlyRelayer() {
        require(relayers[msg.sender], "Not a relayer");
        _;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    // --- Constructor ---

    constructor(address _tierNFT) {
        owner = msg.sender;
        tierNFT = _tierNFT;
    }

    // --- Relayer Management ---

    function setRelayer(address relayer, bool active) external onlyOwner {
        relayers[relayer] = active;
        emit RelayerUpdated(relayer, active);
    }

    // --- Core: Record Repayment via STARK Proof ---

    function recordRepayment(
        bytes32 queryId,
        address proverContract
    ) external onlyRelayer {
        bytes memory result = _processOracleResults(proverContract, queryId);

        address borrower = _extractAddress(result, 0);
        uint256 chainId   = _extractChainId(result, 32);
        bool    onTime    = _extractUint256(result, 64) == 1;

        RepaymentRecord storage r = records[borrower];
        if (onTime) {
            r.onTime++;
        } else {
            r.late++;
        }
        r.chainsUsed |= _chainBit(chainId);

        emit RepaymentRecorded(borrower, chainId, onTime);
    }

    // --- Score Calculation ---

    function getScore(address user) external view override returns (uint256) {
        return _computeScore(records[user]);
    }

    function getTier(address user) external view override returns (Tier) {
        return _scoreToTier(_computeScore(records[user]));
    }

    function getRepaymentStats(address user)
        external view override returns (uint256 onTime, uint256 late)
    {
        RepaymentRecord storage r = records[user];
        return (r.onTime, r.late);
    }

    // --- Internal ---

    function _computeScore(RepaymentRecord storage r) internal view returns (uint256) {
        uint256 score = BASE_SCORE;
        score += r.onTime * ON_TIME_WEIGHT;

        uint256 penalty = r.late * LATE_PENALTY;
        score = penalty >= score ? 0 : score - penalty;

        score += _countBits(r.chainsUsed) * CHAIN_BONUS;

        return score > MAX_SCORE ? MAX_SCORE : score;
    }

    function _scoreToTier(uint256 score) internal pure returns (Tier) {
        if (score >= 750) return Tier.Platinum;
        if (score >= 650) return Tier.Gold;
        if (score >= 500) return Tier.Silver;
        if (score >= 300) return Tier.Bronze;
        return Tier.None;
    }

    function _chainBit(uint256 chainId) internal pure returns (uint256) {
        if (chainId == 1)      return 1;  // Ethereum
        if (chainId == 0)      return 2;  // Bitcoin
        if (chainId == 1399811149) return 4;  // Solana
        if (chainId == 137)    return 8;  // Polygon
        return 0;
    }

    function _countBits(uint256 x) internal pure returns (uint256 count) {
        while (x != 0) {
            count += x & 1;
            x >>= 1;
        }
    }
}
```

### Why This Design

- **Bitmap for chains**: Gas-efficient way to track multi-chain diversity without arrays.
- **Constants for weights**: Easy to audit scoring formula — no hidden logic.
- **Interface separation**: Other contracts depend on `ICreditScore`, not the implementation.

## 3. CreditTierNFT (Soulbound)

Non-transferable ERC-721 that represents a user's credit tier. Only the score contract can mint or update tiers.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract CreditTierNFT is ERC721 {

    address public scoreContract;
    mapping(address => uint256) public tokenOf;  // user → tokenId
    mapping(uint256 => uint8) public tierOf;     // tokenId → tier
    uint256 private nextId = 1;

    modifier onlyScoreContract() {
        require(msg.sender == scoreContract, "Not authorized");
        _;
    }

    constructor(address _scoreContract) ERC721("CreditTier", "CTIER") {
        scoreContract = _scoreContract;
    }

    /// @notice Mint or update tier badge for a user
    function updateTier(address user, uint8 tier) external onlyScoreContract {
        uint256 tokenId = tokenOf[user];
        if (tokenId == 0) {
            tokenId = nextId++;
            tokenOf[user] = tokenId;
            _mint(user, tokenId);
        }
        tierOf[tokenId] = tier;
    }

    /// @notice Soulbound: block all transfers
    function _update(address to, uint256 tokenId, address auth)
        internal override returns (address)
    {
        address from = _ownerOf(tokenId);
        require(from == address(0), "Soulbound: non-transferable");
        return super._update(to, tokenId, auth);
    }
}
```

### Soulbound Mechanic

The `_update` override only allows minting (`from == address(0)`). Any transfer attempt reverts — making the NFT permanently bound to the original recipient.

## 4. Deployment

Uses Hardhat — network config is in `creditcoin-smart-contract.md`.

```typescript
// scripts/deploy-credit-score.ts
import { ethers } from "hardhat";

async function main() {
  // 1. Deploy score contract with placeholder NFT address
  const Score = await ethers.getContractFactory("CrossChainCreditScore");
  const score = await Score.deploy(ethers.ZeroAddress);
  await score.waitForDeployment();
  const scoreAddr = await score.getAddress();

  // 2. Deploy NFT pointing to score contract
  const NFT = await ethers.getContractFactory("CreditTierNFT");
  const nft = await NFT.deploy(scoreAddr);
  await nft.waitForDeployment();
  const nftAddr = await nft.getAddress();

  console.log("CrossChainCreditScore:", scoreAddr);
  console.log("CreditTierNFT:", nftAddr);
}

main().catch((e) => { console.error(e); process.exitCode = 1; });
```

## 5. Client Integration

Querying a user's cross-chain credit score from a frontend.

```typescript
import { ethers } from "ethers";
import ScoreABI from "./abis/CrossChainCreditScore.json";

const SCORE_ADDRESS = "0x...";
const TIERS = ["None", "Bronze", "Silver", "Gold", "Platinum"];

async function getUserCredit(provider: ethers.Provider, user: string) {
  const score = new ethers.Contract(SCORE_ADDRESS, ScoreABI, provider);

  const [creditScore, tier, stats] = await Promise.all([
    score.getScore(user),
    score.getTier(user),
    score.getRepaymentStats(user),
  ]);

  return {
    score: Number(creditScore),
    tier: TIERS[Number(tier)],
    onTime: Number(stats.onTime),
    late: Number(stats.late),
  };
}
```

## Scoring Formula

```
score = 300 (base)
      + onTimeRepayments × 15
      - lateRepayments × 30
      + uniqueChains × 25     (max 4 chains = +100)

Capped at 850. Minimum 0.
```

| Tier | Score Range |
|:-----|:-----------|
| Platinum | 750 – 850 |
| Gold | 650 – 749 |
| Silver | 500 – 649 |
| Bronze | 300 – 499 |
| None | < 300 |

## What Makes This Unique to USC

- **No oracles or bridges** — repayment proofs are cryptographic (STARK), not trust-based.
- **Four-chain aggregation** — a single score from ETH + BTC + SOL + Polygon activity.
- **Chain diversity bonus** — incentivizes multi-chain participation, unique to USC's native multichain design.
- **Soulbound identity** — credit tier is non-transferable, creating real on-chain reputation.
