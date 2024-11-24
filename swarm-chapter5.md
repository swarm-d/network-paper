# Chapter 5: Token Economics

## 5.1 Token Overview

The SWARM token (SWARM) serves as the core utility and governance token of the SWARM network. It implements advanced tokenomics designed to ensure network stability, fair resource allocation, and sustainable economic growth.

### 5.1.1 Token Fundamentals

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SWARMToken is ERC20Votes, ReentrancyGuard {
    // Token parameters
    uint256 public constant MAX_SUPPLY = 1_000_000_000 * 10**18; // 1 billion tokens
    uint256 public constant INITIAL_SUPPLY = 100_000_000 * 10**18; // 100 million tokens
    
    // Emission schedule
    uint256 public constant EMISSION_RATE = 2; // 2% annual emission
    uint256 public constant EMISSION_DECAY = 5; // 5% annual decay in emission rate
    
    // Resource pricing
    mapping(bytes32 => uint256) public resourcePrices;
    uint256 public constant PRICE_UPDATE_INTERVAL = 1 hours;
    
    constructor() ERC20("SWARM Network Token", "SWARM") ERC20Permit("SWARM") {
        _mint(msg.sender, INITIAL_SUPPLY);
    }
}
```

### 5.1.2 Token Utility

The SWARM token provides multiple utilities within the network:

1. **Resource Allocation**
   - Compute credit purchasing
   - Storage allocation
   - Network bandwidth
   - Priority access

2. **Governance Rights**
   - Protocol upgrades
   - Parameter adjustments
   - Resource pricing
   - Feature proposals

3. **Staking Benefits**
   - Resource discounts
   - Priority scheduling
   - Enhanced SLAs
   - Network rewards

## 5.2 Economic Model

### 5.2.1 Dynamic Resource Pricing

```typescript
interface ResourcePricing {
    basePrice: BigNumber;
    demandMultiplier: number;
    utilizationFactor: number;
    minimumPrice: BigNumber;
}

class PricingEngine {
    private oracle: PriceOracle;
    private metrics: NetworkMetrics;

    async calculatePrice(resource: Resource): Promise<BigNumber> {
        // Get base price from oracle
        const basePrice = await this.oracle.getBasePrice(resource);
        
        // Calculate demand multiplier
        const demand = await this.metrics.getResourceDemand(resource);
        const multiplier = this.calculateDemandMultiplier(demand);
        
        // Apply utilization factor
        const utilization = await this.metrics.getUtilization(resource);
        const utilizationFactor = this.calculateUtilizationFactor(utilization);
        
        // Calculate final price
        return basePrice
            .mul(multiplier)
            .mul(utilizationFactor)
            .div(PRICE_PRECISION);
    }

    private calculateDemandMultiplier(demand: number): number {
        // Implement dynamic pricing curve based on demand
        return Math.max(1, 1 + Math.log10(demand));
    }
}
```

### 5.2.2 Staking Mechanism

```solidity
contract SWARMStaking is ReentrancyGuard {
    struct Stake {
        uint256 amount;
        uint256 timestamp;
        uint256 lockPeriod;
        uint256 rewardMultiplier;
    }
    
    mapping(address => Stake) public stakes;
    
    // Staking parameters
    uint256 public constant MIN_STAKE = 1000 * 10**18; // 1000 SWARM
    uint256 public constant MAX_STAKE = 1000000 * 10**18; // 1M SWARM
    uint256 public constant BASE_REWARD_RATE = 500; // 5% APY
    
    function stake(uint256 amount, uint256 lockPeriod) external nonReentrant {
        require(amount >= MIN_STAKE, "Below minimum stake");
        require(amount <= MAX_STAKE, "Above maximum stake");
        
        // Calculate reward multiplier based on lock period
        uint256 multiplier = calculateMultiplier(lockPeriod);
        
        // Transfer tokens
        require(swarmToken.transferFrom(msg.sender, address(this), amount),
            "Transfer failed");
        
        // Create stake
        stakes[msg.sender] = Stake({
            amount: amount,
            timestamp: block.timestamp,
            lockPeriod: lockPeriod,
            rewardMultiplier: multiplier
        });
        
        emit Staked(msg.sender, amount, lockPeriod);
    }
}
```

### 5.2.3 Incentive Structure

The network implements multiple incentive mechanisms:

```typescript
interface IncentiveStructure {
    // Resource provider incentives
    provider: {
        computeRewards: BigNumber;
        storageRewards: BigNumber;
        networkRewards: BigNumber;
        performanceBonus: BigNumber;
    };
    
    // User incentives
    user: {
        stakingRewards: BigNumber;
        referralRewards: BigNumber;
        loyaltyBonus: BigNumber;
    };
    
    // Network incentives
    network: {
        governanceRewards: BigNumber;
        developmentFund: BigNumber;
        ecosystemGrowth: BigNumber;
    };
}

class IncentiveManager {
    private rewardPool: RewardPool;
    private metrics: NetworkMetrics;

    async distributeRewards(): Promise<void> {
        // Calculate total rewards
        const totalRewards = await this.calculateTotalRewards();
        
        // Distribute to providers
        await this.distributeProviderRewards(totalRewards.provider);
        
        // Distribute to users
        await this.distributeUserRewards(totalRewards.user);
        
        // Allocate network rewards
        await this.allocateNetworkRewards(totalRewards.network);
    }
}
```

## 5.3 Resource Credit System

### 5.3.1 Credit Generation

```typescript
interface ResourceCredit {
    creditAmount: BigNumber;
    resourceType: ResourceType;
    validityPeriod: number;
    constraints: ResourceConstraints;
}

class CreditGenerator {
    private tokenContract: SWARMToken;
    private pricingEngine: PricingEngine;

    async generateCredits(
        tokenAmount: BigNumber,
        resourceType: ResourceType
    ): Promise<ResourceCredit> {
        // Calculate credit amount based on token amount and current price
        const price = await this.pricingEngine.calculatePrice(resourceType);
        const creditAmount = tokenAmount.mul(CREDIT_MULTIPLIER).div(price);
        
        // Generate credit object
        return {
            creditAmount,
            resourceType,
            validityPeriod: this.calculateValidityPeriod(tokenAmount),
            constraints: this.determineConstraints(creditAmount)
        };
    }
}
```

### 5.3.2 Credit Management

```typescript
class CreditManager {
    private credits: Map<string, ResourceCredit>;
    private usage: Map<string, BigNumber>;

    async consumeCredits(
        creditId: string,
        amount: BigNumber
    ): Promise<boolean> {
        // Get credit balance
        const credit = this.credits.get(creditId);
        if (!credit) return false;
        
        // Check validity
        if (this.isExpired(credit)) return false;
        
        // Check constraints
        if (!this.meetsConstraints(credit, amount)) return false;
        
        // Update usage
        const currentUsage = this.usage.get(creditId) || BigNumber.from(0);
        this.usage.set(creditId, currentUsage.add(amount));
        
        return true;
    }
}
```

## 5.4 Economic Security

### 5.4.1 Price Stability Mechanisms

```typescript
class StabilityManager {
    private treasury: Treasury;
    private marketMaker: AutomatedMarketMaker;

    async stabilizePrice(currentPrice: BigNumber): Promise<void> {
        // Calculate target price band
        const targetPrice = await this.calculateTargetPrice();
        const allowedDeviation = targetPrice.mul(5).div(100); // 5% deviation
        
        if (currentPrice.gt(targetPrice.add(allowedDeviation))) {
            // Price too high - increase supply
            await this.increaseSupply(this.calculateSupplyIncrease(currentPrice));
        } else if (currentPrice.lt(targetPrice.sub(allowedDeviation))) {
            // Price too low - decrease supply
            await this.decreaseSupply(this.calculateSupplyDecrease(currentPrice));
        }
    }
}
```

### 5.4.2 Sybil Resistance

```typescript
interface SybilResistance {
    minimumStake: BigNumber;
    reputationScore: number;
    activityMetrics: ActivityMetrics;
    validationRules: ValidationRule[];
}

class SybilResistanceManager {
    private stakeManager: StakeManager;
    private reputationSystem: ReputationSystem;

    async validateParticipant(
        address: string
    ): Promise<ValidationResult> {
        // Check stake
        const stake = await this.stakeManager.getStake(address);
        if (stake.lt(MINIMUM_STAKE)) return false;
        
        // Check reputation
        const reputation = await this.reputationSystem.getScore(address);
        if (reputation.lt(MINIMUM_REPUTATION)) return false;
        
        // Validate activity patterns
        return this.validateActivityPatterns(address);
    }
}
```

## 5.5 Governance

### 5.5.1 Proposal System

```solidity
contract SWARMGovernance {
    struct Proposal {
        uint256 id;
        address proposer;
        string description;
        bytes callData;
        uint256 votingStart;
        uint256 votingEnd;
        uint256 forVotes;
        uint256 againstVotes;
        bool executed;
    }
    
    mapping(uint256 => Proposal) public proposals;
    
    function propose(
        string calldata description,
        bytes calldata callData
    ) external returns (uint256) {
        require(token.getVotes(msg.sender) >= PROPOSAL_THRESHOLD,
            "Insufficient voting power");
        
        uint256 proposalId = _generateProposalId();
        
        proposals[proposalId] = Proposal({
            id: proposalId,
            proposer: msg.sender,
            description: description,
            callData: callData,
            votingStart: block.timestamp + VOTING_DELAY,
            votingEnd: block.timestamp + VOTING_DELAY + VOTING_PERIOD,
            forVotes: 0,
            againstVotes: 0,
            executed: false
        });
        
        emit ProposalCreated(proposalId, msg.sender, description);
        return proposalId;
    }
}
```

### 5.5.2 Voting Mechanism

```typescript
interface VotingPower {
    baseVotes: BigNumber;
    stakingMultiplier: number;
    timeMultiplier: number;
    reputationBonus: number;
}

class VotingSystem {
    private tokenContract: SWARMToken;
    private staking: StakingContract;

    async calculateVotingPower(address: string): Promise<BigNumber> {
        // Get token balance
        const balance = await this.tokenContract.balanceOf(address);
        
        // Get staking multiplier
        const stake = await this.staking.getStake(address);
        const stakingMultiplier = this.calculateStakingMultiplier(stake);
        
        // Calculate time multiplier
        const timeMultiplier = await this.calculateTimeMultiplier(address);
        
        // Apply multipliers
        return balance
            .mul(stakingMultiplier)
            .mul(timeMultiplier)
            .div(PRECISION);
    }
}
```

This chapter provides a comprehensive overview of the SWARM token economics, including detailed implementations of key mechanisms that ensure network stability, fair resource allocation, and sustainable growth. The system is designed to be both flexible and robust, with multiple safeguards against potential economic attacks or market manipulation.