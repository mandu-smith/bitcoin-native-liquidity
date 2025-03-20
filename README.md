# Bitcoin-Native-Liquidity Protocol Documentation

## Overview

Bitcoin-Native-Liquidity is an institutional-grade Automated Market Maker (AMM) protocol enabling decentralized trading of Bitcoin-native assets on Stacks Layer 2. The protocol combines capital efficiency with Bitcoin's security model, offering advanced DeFi primitives optimized for both retail and institutional participants.

## Key Features

### Core Innovations

- **Dynamic Liquidity Pools**  
  Multi-asset pools with adaptive fee structures (0.01-1%) and concentrated liquidity positions
- **Atomic Arbitrage Engine**  
  Flash loan-enabled arbitrage system with sub-block settlement (≤30s)
- **Enterprise-Grade TWAP Oracles**  
  Time-Weighted Average Prices with ±0.5% precision and 25-minute validity windows
- **Bitcoin-First Asset Support**  
  Native integration for SATS (1e-8 BTC), RBTC, and STX with BTC-settled transactions

### Performance Characteristics

- ≤100ms trade execution latency
- Zero price impact for trades ≤0.5% of pool liquidity
- MEV-resistant pool architecture with batch settlements
- Cross-chain liquidity bridges (BTC/L2) with 2-way pegs

## Technical Specifications

### Protocol Parameters

| Parameter              | Value               | Description                                |
| ---------------------- | ------------------- | ------------------------------------------ |
| `MAX_PRICE_IMPACT`     | 2%                  | Maximum allowable price movement per trade |
| `FLASH_LOAN_FEE`       | 0.1%                | Fixed fee for flash loan utilization       |
| `PROTOCOL_FEE`         | 0.5%                | Revenue share from trading fees            |
| `ORACLE_VALIDITY`      | 150 blocks (~25min) | TWAP update frequency requirement          |
| `GOVERNANCE_THRESHOLD` | 1M STX              | Minimum stake for proposal creation        |

### Core Data Structures

```clarity
;; Liquidity Pool Configuration
{
    token-x: principal,
    token-y: principal,
    reserve-x: uint,
    reserve-y: uint,
    fee-rate: uint,       ;; Basis points (30 = 0.3%)
    twap: uint,           ;; Time-weighted average price
    price-cumulative: uint
}

;; Liquidity Provider Position
{
    shares: uint,
    staked-amount: uint,
    unclaimed-fees: (uint, uint),
    reward-multiplier: uint
}

;; Flash Loan Record
{
    amount: uint,
    due-block: uint,
    collateral-ratio: uint
}
```

## Core Functionality

### Pool Management

**Create New Pool**

```clarity
(create-pool token-x token-y initial-x initial-y)
```

- Initializes pair with minimum 1,000,000 satoshi liquidity
- Mints initial LP tokens at 1:1 USD value ratio

**Add Liquidity**

```clarity
(add-liquidity pool-id amount-x amount-y min-shares)
```

- Proportional deposit system with 0.05% slippage tolerance
- Auto-rebalancing against current pool ratios

### Trading Engine

**Fixed-Input Swap**

```clarity
(swap-exact-x-for-y pool-id amount-x min-y)
```

- Guaranteed output with 0.3% base fee + protocol fee
- Reverts if price impact >2% or oracle staleness >25min

**Multi-Hop Routing**

```clarity
(multi-hop-swap [pool-1 pool-2 pool-3] amount-in min-out)
```

- Atomic cross-pool execution with optimized routing
- Integrated slippage control across all hops

### Advanced Features

**Flash Loans**

```clarity
(flash-swap pool-id amount callback-contract)
```

- 0.1% fixed fee with same-block repayment
- Requires overcollateralization of 125% for cross-protocol calls

**Yield Farming**

```clarity
(stake-in-farm pool-id amount)
```

- Auto-compounding rewards with 7-day vesting period
- Impermanent loss protection after 90-day staking

## Governance System

### Voting Mechanics

- **STX-weighted voting** with quadratic decay over time
- **Delegation** system with revocable mandates
- **Proposal Threshold**: 1M STX equivalent

### Governance Actions

1. Parameter Adjustments

   - Fee structures
   - Oracle configurations
   - Risk parameters

2. Protocol Upgrades

   - Smart contract migrations
   - Feature enablement/disablement

3. Treasury Management
   - Fee distribution
   - Grant allocations

## Security Architecture

### Risk Mitigations

- **Circuit Breakers**:  
  Automatic trading suspension if 5%+ price deviation in <10 blocks
- **Collateral Safeguards**:  
  200% overcollateralization required for cross-chain operations
- **Oracle Defenses**:  
  Three-tiered price validation with fallback to Chainlink BTC/USD

### Audit Trail

- All trades recorded to Bitcoin L1 via Stacks blocks
- Immutable transaction logs with SPV proofs
- Real-time reserve attestations

## Developer Resources

### Interaction Examples

**Create BTC/STX Pool**

```bash
clarinet contract call bitcoin-native-liquidity create-pool \
  --token-x SP3X...BTC --token-y SP2H...STX \
  --initial-x 100000000 --initial-y 500000000000
```

**Execute Flash Arbitrage**

```clarity
(flash-swap 42 1000000 'SP3K...arb-contract)
```

**Query TWAP Oracle**

```clarity
(get-twap-price 42) ;; Returns (ok u8500) for 1 BTC = 8500 STX
```

### Testing Guidelines

1. Verify contract interactions
   ```bash
   clarinet console
   >> (contract-call? .bitcoin-native-liquidity get-pool-details u42)
   ```

## References

1. [Stacks Blockchain Documentation](https://docs.stacks.co)
2. [Clarity Language Reference](https://clarity-lang.org)
