# Yieldstark Architecture: Technical Design & Flow Diagrams

## System Architecture Overview

This document provides detailed architectural specifications for the Yieldstark protocol, including component interactions, data flows, and system design patterns.

## High-Level Architecture

```mermaid
graph TB
    subgraph "User Layer"
        U1[Web Interface]
        U2[Mobile App]
        U3[Wallet Integration]
    end
    
    subgraph "Application Layer"
        A1[Frontend Services]
        A2[API Gateway]
        A3[State Management]
    end
    
    subgraph "Smart Contract Layer"
        SC1[ysDOT Token Contract]
        SC2[Staking Pool Contract]
        SC3[Strategy Manager Contract]
        SC4[Rebasing Controller]
        SC5[Integration Adapters]
    end
    
    subgraph "Cross-Chain Layer"
        XCM1[XCM Message Handler]
        XCM2[Parachain Connectors]
        XCM3[Asset Transfer Module]
    end
    
    subgraph "External Parachains"
        P1[Bifrost Parachain<br/>vDOT]
        P2[Acala Parachain<br/>LDOT]
        P3[Other Parachains]
    end
    
    subgraph "Polkadot Relay Chain"
        RC1[Staking Module]
        RC2[Validator Set]
    end
    
    U1 --> A1
    U2 --> A1
    U3 --> A1
    A1 --> A2
    A2 --> SC1
    A2 --> SC2
    SC2 --> SC3
    SC3 --> SC4
    SC3 --> SC5
    SC5 --> XCM1
    XCM1 --> XCM2
    XCM2 --> P1
    XCM2 --> P2
    XCM2 --> P3
    XCM2 --> RC1
    SC4 --> SC1
```

## Core Protocol Flow

### Staking Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant ysDOT Contract
    participant Strategy Manager
    participant XCM Handler
    participant Bifrost
    participant Acala
    
    User->>Frontend: Stake 100 DOT
    Frontend->>ysDOT Contract: deposit(100 DOT)
    ysDOT Contract->>Strategy Manager: allocateFunds(100 DOT)
    
    Strategy Manager->>Strategy Manager: Calculate Allocation
    Note over Strategy Manager: 40% Bifrost, 40% Acala, 20% Reserve
    
    Strategy Manager->>XCM Handler: transferToBifrost(40 DOT)
    XCM Handler->>Bifrost: XCM Transfer + Stake
    Bifrost-->>XCM Handler: vDOT Receipt
    
    Strategy Manager->>XCM Handler: transferToAcala(40 DOT)
    XCM Handler->>Acala: XCM Transfer + Stake
    Acala-->>XCM Handler: LDOT Receipt
    
    XCM Handler-->>Strategy Manager: LST Tokens Received
    Strategy Manager->>ysDOT Contract: mint ysDOT
    ysDOT Contract-->>Frontend: 100 ysDOT minted
    Frontend-->>User: 100 ysDOT in wallet
```

### Rebasing Flow

```mermaid
sequenceDiagram
    participant Epoch Timer
    participant Rebasing Controller
    participant Strategy Manager
    participant XCM Handler
    participant Bifrost
    participant Acala
    participant ysDOT Contract
    
    Epoch Timer->>Rebasing Controller: Trigger Rebasing Epoch
    Rebasing Controller->>Strategy Manager: Query Total Rewards
    
    Strategy Manager->>XCM Handler: Query vDOT Rewards
    XCM Handler->>Bifrost: Query Exchange Rate
    Bifrost-->>XCM Handler: vDOT/DOT Rate
    
    Strategy Manager->>XCM Handler: Query LDOT Rewards
    XCM Handler->>Acala: Query Exchange Rate
    Acala-->>XCM Handler: LDOT/DOT Rate
    
    Strategy Manager-->>Rebasing Controller: Total Rewards Calculated
    Note over Rebasing Controller: Calculate new supply<br/>old_supply * (1 + reward_rate)
    
    Rebasing Controller->>ysDOT Contract: rebase(newSupply)
    ysDOT Contract->>ysDOT Contract: Update Total Supply
    ysDOT Contract->>ysDOT Contract: Update All Balances
    Note over ysDOT Contract: All user balances<br/>increase proportionally
```

### Unstaking/Redemption Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant ysDOT Contract
    participant Strategy Manager
    participant XCM Handler
    participant Bifrost
    participant Acala
    
    User->>Frontend: Redeem 100 ysDOT
    Frontend->>ysDOT Contract: redeem(100 ysDOT)
    ysDOT Contract->>ysDOT Contract: burn(100 ysDOT)
    
    ysDOT Contract->>Strategy Manager: requestRedemption(100 DOT equivalent)
    
    Strategy Manager->>Strategy Manager: Calculate Redemption Sources
    Note over Strategy Manager: Redeem from multiple<br/>protocols proportionally
    
    Strategy Manager->>XCM Handler: redeemFromBifrost(40 DOT worth)
    XCM Handler->>Bifrost: Redeem vDOT
    Bifrost-->>XCM Handler: DOT (may be delayed)
    
    Strategy Manager->>XCM Handler: redeemFromAcala(40 DOT worth)
    XCM Handler->>Acala: Redeem LDOT
    Acala-->>XCM Handler: DOT (instant or delayed)
    
    XCM Handler-->>Strategy Manager: DOT Received
    Strategy Manager->>ysDOT Contract: transferDOT(user, amount)
    ysDOT Contract-->>Frontend: DOT transferred
    Frontend-->>User: DOT in wallet
```

## Component Architecture

### 1. ysDOT Token Contract

```mermaid
graph LR
    subgraph "ysDOT Token Contract"
        T1[Minting Module]
        T2[Burning Module]
        T3[Rebasing Module]
        T4[Balance Tracking]
        T5[Access Control]
    end
    
    T1 --> T4
    T2 --> T4
    T3 --> T4
    T5 --> T1
    T5 --> T2
    T5 --> T3
```

**Responsibilities:**
- Token minting and burning
- Balance management with rebasing
- Access control for authorized contracts
- Event emission for off-chain tracking

**Key Functions:**
- `mint(address, amount)`: Mint ysDOT tokens
- `burn(address, amount)`: Burn ysDOT tokens
- `rebase(uint256 newSupply)`: Adjust total supply
- `balanceOf(address)`: Get rebased balance
- `totalSupply()`: Get current total supply

### 2. Strategy Manager Contract

```mermaid
graph TD
    subgraph "Strategy Manager"
        S1[Allocation Engine]
        S2[Yield Calculator]
        S3[Rebalancing Logic]
        S4[Protocol Adapters]
        S5[Risk Manager]
    end
    
    S1 --> S2
    S2 --> S3
    S3 --> S4
    S5 --> S1
    S5 --> S3
```

**Responsibilities:**
- Calculate optimal allocation across protocols
- Monitor yield rates
- Execute rebalancing when needed
- Manage protocol integrations
- Assess and manage risks

**Key Functions:**
- `allocateFunds(uint256 amount)`: Distribute funds
- `calculateAllocation()`: Determine optimal split
- `rebalance()`: Adjust allocations
- `getYieldRates()`: Query current yields
- `updateStrategy()`: Modify strategy parameters

### 3. Integration Adapters

```mermaid
graph LR
    subgraph "Integration Layer"
        I1[Bifrost Adapter]
        I2[Acala Adapter]
        I3[Generic Adapter]
    end
    
    subgraph "XCM Layer"
        X1[Message Builder]
        X2[Message Executor]
        X3[Response Handler]
    end
    
    I1 --> X1
    I2 --> X1
    I3 --> X1
    X1 --> X2
    X2 --> X3
    X3 --> I1
    X3 --> I2
    X3 --> I3
```

**Responsibilities:**
- Protocol-specific integration logic
- XCM message construction
- Response parsing
- Error handling and retries

**Adapter Interface:**
- `stake(uint256 amount)`: Stake assets
- `unstake(uint256 amount)`: Unstake assets
- `getBalance()`: Query staked balance
- `getExchangeRate()`: Get current rate
- `getYieldRate()`: Calculate APY

### 4. Rebasing Controller

```mermaid
graph TD
    subgraph "Rebasing Controller"
        R1[Epoch Manager]
        R2[Reward Aggregator]
        R3[Supply Calculator]
        R4[Execution Module]
    end
    
    R1 --> R2
    R2 --> R3
    R3 --> R4
```

**Responsibilities:**
- Trigger rebasing at appropriate intervals
- Aggregate rewards from all strategies
- Calculate new supply
- Execute rebasing on token contract

**Key Functions:**
- `triggerRebase()`: Initiate rebasing process
- `calculateRewards()`: Sum rewards from all sources
- `calculateNewSupply()`: Determine new total supply
- `executeRebase()`: Update token supply

## Data Flow Architecture

### State Management

```mermaid
graph TD
    subgraph "On-Chain State"
        OS1[Token Balances]
        OS2[Allocation State]
        OS3[Strategy Parameters]
        OS4[Protocol Configs]
    end
    
    subgraph "Off-Chain Indexing"
        OC1[Event Indexer]
        OC2[State Cache]
        OC3[Analytics DB]
    end
    
    subgraph "External Data"
        ED1[Yield Oracles]
        ED2[Price Feeds]
        ED3[Protocol APIs]
    end
    
    OS1 --> OC1
    OS2 --> OC1
    OS3 --> OC1
    OC1 --> OC2
    OC2 --> OC3
    ED1 --> OC3
    ED2 --> OC3
    ED3 --> OC3
```

## Security Architecture

### Multi-Layer Security

```mermaid
graph TD
    subgraph "Access Control"
        AC1[Role-Based Access]
        AC2[Multi-Sig Governance]
        AC3[Time Locks]
    end
    
    subgraph "Validation Layer"
        VL1[Input Validation]
        VL2[State Validation]
        VL3[Rate Limiting]
    end
    
    subgraph "Monitoring"
        M1[Event Logging]
        M2[Anomaly Detection]
        M3[Circuit Breakers]
    end
    
    AC1 --> VL1
    AC2 --> VL2
    AC3 --> VL3
    VL1 --> M1
    VL2 --> M2
    VL3 --> M3
```

## Cross-Chain Communication Architecture

### XCM Message Flow

```mermaid
sequenceDiagram
    participant Contract
    participant XCM Handler
    participant Relay Chain
    participant Target Parachain
    participant Target Protocol
    
    Contract->>XCM Handler: Request Cross-Chain Operation
    XCM Handler->>XCM Handler: Build XCM Message
    Note over XCM Handler: Construct XCM::Transact<br/>with encoded call
    
    XCM Handler->>Relay Chain: Send XCM Message
    Relay Chain->>Target Parachain: Forward Message
    
    Target Parachain->>Target Protocol: Execute Call
    Target Protocol->>Target Protocol: Process Operation
    
    Target Protocol-->>Target Parachain: Result
    Target Parachain-->>Relay Chain: Response
    Relay Chain-->>XCM Handler: XCM Response
    XCM Handler-->>Contract: Operation Result
```

## Scalability Considerations

### Horizontal Scaling

- **Stateless Services**: Frontend and API services can scale horizontally
- **Database Sharding**: Partition data by user or time
- **Caching Layer**: Redis for frequently accessed data
- **Load Balancing**: Distribute requests across instances

### Vertical Scaling

- **Batch Operations**: Group multiple operations
- **Gas Optimization**: Efficient contract design
- **State Compression**: Minimize on-chain storage
- **Lazy Evaluation**: Compute only when needed

## Monitoring & Observability

### Key Metrics

1. **Protocol Metrics**
   - TVL (Total Value Locked)
   - Number of stakers
   - Average stake size
   - Protocol APY

2. **Technical Metrics**
   - Transaction success rate
   - XCM message success rate
   - Gas usage
   - Block confirmation times

3. **Financial Metrics**
   - Yield distribution
   - Fee collection
   - Treasury balance
   - Insurance fund status

### Alerting

- Unusual withdrawal patterns
- XCM message failures
- Significant yield drops
- Contract errors
- Unauthorized access attempts

