# Yieldstark Architecture Diagrams: Complete Visual Reference

## System Overview Diagram

```mermaid
graph TB
    subgraph "User Interface Layer"
        UI[Web Application]
        MOBILE[Mobile App]
        WALLET[Wallet Integration]
    end
    
    subgraph "Application Services"
        API[API Gateway]
        AUTH[Authentication]
        CACHE[Cache Layer]
        INDEXER[Event Indexer]
    end
    
    subgraph "Smart Contract Layer - Yieldstark Parachain"
        YSDOT[ysDOT Token Contract]
        POOL[Staking Pool Contract]
        STRAT[Strategy Manager]
        REBASE[Rebasing Controller]
        ADAPTER[Integration Adapters]
    end
    
    subgraph "Cross-Chain Communication"
        XCM[XCM Handler]
        ROUTER[XCM Router]
    end
    
    subgraph "External Parachains"
        BIFROST[Bifrost Parachain<br/>vDOT Protocol]
        ACALA[Acala Parachain<br/>LDOT Protocol]
        OTHERS[Other Parachains]
    end
    
    subgraph "Polkadot Relay Chain"
        RELAY[Relay Chain]
        VALIDATORS[Validator Set]
        STAKING[Staking Module]
    end
    
    UI --> API
    MOBILE --> API
    WALLET --> API
    API --> AUTH
    API --> CACHE
    API --> YSDOT
    API --> POOL
    
    POOL --> STRAT
    STRAT --> REBASE
    STRAT --> ADAPTER
    
    ADAPTER --> XCM
    XCM --> ROUTER
    ROUTER --> BIFROST
    ROUTER --> ACALA
    ROUTER --> OTHERS
    ROUTER --> RELAY
    
    RELAY --> VALIDATORS
    RELAY --> STAKING
    
    YSDOT --> INDEXER
    POOL --> INDEXER
    STRAT --> INDEXER
    INDEXER --> CACHE
    
    style YSDOT fill:#4CAF50
    style STRAT fill:#2196F3
    style REBASE fill:#FF9800
    style XCM fill:#9C27B0
```

## Complete Staking Flow

```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant API as API Gateway
    participant YS as ysDOT Contract
    participant SM as Strategy Manager
    participant IA as Integration Adapter
    participant XCM as XCM Handler
    participant B as Bifrost
    participant A as Acala
    participant RC as Relay Chain
    
    U->>F: Stake 1000 DOT
    F->>API: POST /stake {amount: 1000}
    API->>YS: deposit(1000 DOT)
    YS->>YS: Validate & Store
    
    YS->>SM: allocateFunds(1000 DOT)
    SM->>SM: Calculate Optimal Allocation
    Note over SM: Bifrost: 40% (400 DOT)<br/>Acala: 40% (400 DOT)<br/>Reserve: 20% (200 DOT)
    
    SM->>IA: stakeOnBifrost(400 DOT)
    IA->>XCM: Build XCM Message
    XCM->>RC: Send XCM Transfer
    RC->>B: Forward to Bifrost
    B->>B: Process Staking
    B->>B: Mint vDOT
    B-->>RC: vDOT Receipt
    RC-->>XCM: XCM Response
    XCM-->>IA: vDOT Confirmed
    
    SM->>IA: stakeOnAcala(400 DOT)
    IA->>XCM: Build XCM Message
    XCM->>RC: Send XCM Transfer
    RC->>A: Forward to Acala
    A->>A: Process Staking
    A->>A: Mint LDOT
    A-->>RC: LDOT Receipt
    RC-->>XCM: XCM Response
    XCM-->>IA: LDOT Confirmed
    
    IA-->>SM: All Allocations Complete
    SM->>YS: mint ysDOT(1000)
    YS->>YS: Mint 1000 ysDOT
    YS-->>API: Success Event
    API-->>F: Transaction Confirmed
    F-->>U: 1000 ysDOT Received
```

## Rebasing Mechanism Flow

```mermaid
graph TD
    START[Rebasing Trigger] --> CHECK{Trigger Type?}
    
    CHECK -->|Epoch Timer| EPOCH[Epoch-Based Trigger]
    CHECK -->|Reward Threshold| THRESH[Threshold-Based Trigger]
    CHECK -->|Manual| MANUAL[Manual Trigger]
    
    EPOCH --> QUERY
    THRESH --> QUERY
    MANUAL --> QUERY
    
    QUERY[Query All Protocol Rewards] --> BIF[Query Bifrost vDOT Rate]
    QUERY --> ACA[Query Acala LDOT Rate]
    QUERY --> DIRECT[Query Direct Staking Rewards]
    
    BIF --> CALC1[Calculate vDOT Rewards]
    ACA --> CALC2[Calculate LDOT Rewards]
    DIRECT --> CALC3[Calculate Direct Rewards]
    
    CALC1 --> AGG[Aggregate Total Rewards]
    CALC2 --> AGG
    CALC3 --> AGG
    
    AGG --> RATE[Calculate Reward Rate]
    RATE --> VALIDATE{Validate Calculation}
    
    VALIDATE -->|Invalid| ERROR[Log Error & Abort]
    VALIDATE -->|Valid| CALCSUP[Calculate New Supply]
    
    CALCSUP --> UPDATE[Update Total Supply]
    UPDATE --> REBASE[Rebase All Balances]
    REBASE --> EVENT[Emit Rebasing Event]
    EVENT --> INDEX[Update Off-Chain Indexers]
    INDEX --> END[Rebasing Complete]
    
    style START fill:#4CAF50
    style AGG fill:#2196F3
    style REBASE fill:#FF9800
    style END fill:#4CAF50
```

## Multi-Strategy Allocation Logic

```mermaid
graph TD
    INPUT[New Funds to Allocate] --> ANALYZE[Analyze Current State]
    
    ANALYZE --> YIELD[Query Yield Rates]
    YIELD --> BIFY[Bifrost APY: 15%]
    YIELD --> ACAY[Acala APY: 14.5%]
    YIELD --> OTHY[Other Protocols APY]
    
    BIFY --> RISK[Assess Risk Scores]
    ACAY --> RISK
    OTHY --> RISK
    
    RISK --> LIQ[Check Liquidity]
    LIQ --> BIFL[Bifrost Liquidity: High]
    LIQ --> ACAL[Acala Liquidity: High]
    LIQ --> OTHL[Other Liquidity]
    
    BIFL --> CALC[Calculate Optimal Allocation]
    ACAL --> CALC
    OTHL --> CALC
    
    CALC --> CONSTRAINTS[Apply Constraints]
    CONSTRAINTS --> MIN[Min: 10% per protocol]
    CONSTRAINTS --> MAX[Max: 50% per protocol]
    CONSTRAINTS --> DIVERS[Diversification: Min 3 protocols]
    
    MIN --> OPTIMIZE[Optimize Allocation]
    MAX --> OPTIMIZE
    DIVERS --> OPTIMIZE
    
    OPTIMIZE --> ALLOC[Final Allocation]
    ALLOC --> BIF40[40% Bifrost]
    ALLOC --> ACA40[40% Acala]
    ALLOC --> RES20[20% Reserve]
    
    BIF40 --> EXECUTE[Execute Allocation]
    ACA40 --> EXECUTE
    RES20 --> EXECUTE
    
    EXECUTE --> MONITOR[Monitor Performance]
    MONITOR --> REBAL{Need Rebalance?}
    REBAL -->|Yes| ANALYZE
    REBAL -->|No| DONE[Allocation Complete]
    
    style INPUT fill:#4CAF50
    style OPTIMIZE fill:#2196F3
    style EXECUTE fill:#FF9800
    style DONE fill:#4CAF50
```

## Unstaking/Redemption Flow

```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant YS as ysDOT Contract
    participant SM as Strategy Manager
    participant IA as Integration Adapter
    participant XCM as XCM Handler
    participant B as Bifrost
    participant A as Acala
    
    U->>F: Redeem 500 ysDOT
    F->>YS: redeem(500 ysDOT)
    YS->>YS: Validate Balance
    YS->>YS: Burn 500 ysDOT
    
    YS->>SM: requestRedemption(500 DOT equivalent)
    SM->>SM: Calculate Redemption Sources
    Note over SM: Redeem proportionally:<br/>40% from Bifrost (200 DOT)<br/>40% from Acala (200 DOT)<br/>20% from Reserve (100 DOT)
    
    SM->>IA: redeemFromBifrost(200 DOT)
    IA->>IA: Check Redemption Type
    IA->>IA: Choose: Fast/Instant/Standard
    
    alt Fast Redeem
        IA->>XCM: Fast Redeem Request
        XCM->>B: Fast Redeem
        B->>B: Queue Match
        B-->>XCM: DOT (delayed)
    else Instant Swap
        IA->>XCM: Instant Swap Request
        XCM->>B: Swap vDOT for DOT
        B->>B: Execute Swap
        B-->>XCM: DOT (instant)
    else Standard
        IA->>XCM: Standard Redeem
        XCM->>B: Unbond Request
        B-->>XCM: Unbonding Started (28 days)
    end
    
    SM->>IA: redeemFromAcala(200 DOT)
    IA->>IA: Check Redemption Type
    
    alt Instant Swap
        IA->>XCM: Instant Swap via DEX
        XCM->>A: Swap LDOT for DOT
        A->>A: Execute Swap
        A-->>XCM: DOT (instant)
    else Standard Unbonding
        IA->>XCM: Standard Redeem
        XCM->>A: Unbond Request
        A-->>XCM: Unbonding Started (28 days)
    end
    
    SM->>SM: Aggregate Received DOT
    SM->>YS: transferDOT(user, totalAmount)
    YS->>YS: Transfer DOT to User
    YS-->>F: Redemption Complete
    F-->>U: DOT Received
    
    Note over SM: If unbonding required,<br/>user receives DOT after<br/>unbonding period completes
```

## XCM Message Flow

```mermaid
sequenceDiagram
    participant SC as Smart Contract
    participant XCM as XCM Handler
    participant RC as Relay Chain
    participant TP as Target Parachain
    participant PROTO as Target Protocol
    
    SC->>XCM: Request Operation
    Note over XCM: Build XCM Message
    
    XCM->>XCM: Construct XCM::Transact
    XCM->>XCM: Encode Call Data
    XCM->>XCM: Set Weight Limits
    XCM->>XCM: Add Fee Payment
    
    XCM->>RC: Send XCM Message
    RC->>RC: Validate Message
    RC->>RC: Route to Target
    
    RC->>TP: Forward XCM Message
    TP->>TP: Validate & Execute
    
    TP->>PROTO: Execute Call
    PROTO->>PROTO: Process Operation
    
    PROTO-->>TP: Operation Result
    TP-->>RC: XCM Response
    
    alt Success
        RC-->>XCM: Success Response
        XCM-->>SC: Operation Successful
    else Error
        RC-->>XCM: Error Response
        XCM-->>SC: Operation Failed
        XCM->>XCM: Handle Error
        XCM->>XCM: Retry or Fallback
    end
```

## State Management Architecture

```mermaid
graph TB
    subgraph "On-Chain State"
        OC1[Token Balances<br/>Base Balance + Rebase Index]
        OC2[Allocation State<br/>Protocol Distribution]
        OC3[Strategy Parameters<br/>Min/Max Allocations]
        OC4[Protocol Configs<br/>Addresses & Settings]
        OC5[Rebase History<br/>Event Logs]
    end
    
    subgraph "Off-Chain Indexing"
        OF1[Event Indexer<br/>Listen to Chain Events]
        OF2[State Cache<br/>Redis/Memcached]
        OF3[Analytics DB<br/>PostgreSQL/MongoDB]
        OF4[Time Series DB<br/>InfluxDB/TimescaleDB]
    end
    
    subgraph "External Data Sources"
        EX1[Yield Oracles<br/>Protocol APY Data]
        EX2[Price Feeds<br/>DOT/LST Prices]
        EX3[Protocol APIs<br/>Bifrost/Acala APIs]
        EX4[Block Explorers<br/>Subscan/Polkascan]
    end
    
    OC1 --> OF1
    OC2 --> OF1
    OC3 --> OF1
    OC4 --> OF1
    OC5 --> OF1
    
    OF1 --> OF2
    OF1 --> OF3
    OF1 --> OF4
    
    EX1 --> OF3
    EX2 --> OF3
    EX3 --> OF3
    EX4 --> OF3
    
    OF2 --> API[API Gateway]
    OF3 --> API
    OF4 --> API
    
    style OC1 fill:#4CAF50
    style OF1 fill:#2196F3
    style OF3 fill:#FF9800
```

## Security Architecture

```mermaid
graph TD
    subgraph "Access Control Layer"
        AC1[Role-Based Access Control]
        AC2[Multi-Sig Governance]
        AC3[Time-Locked Upgrades]
        AC4[Emergency Pause]
    end
    
    subgraph "Validation Layer"
        VL1[Input Validation]
        VL2[State Validation]
        VL3[Rate Limiting]
        VL4[Circuit Breakers]
    end
    
    subgraph "Monitoring Layer"
        ML1[Event Logging]
        ML2[Anomaly Detection]
        ML3[Real-Time Alerts]
        ML4[Audit Trails]
    end
    
    subgraph "Risk Management"
        RM1[Insurance Fund]
        RM2[Slashing Protection]
        RM3[Diversification Rules]
        RM4[Liquidity Management]
    end
    
    AC1 --> VL1
    AC2 --> VL2
    AC3 --> VL3
    AC4 --> VL4
    
    VL1 --> ML1
    VL2 --> ML2
    VL3 --> ML3
    VL4 --> ML4
    
    ML2 --> RM1
    ML3 --> RM2
    RM1 --> RM3
    RM2 --> RM4
    
    style AC1 fill:#4CAF50
    style VL1 fill:#2196F3
    style ML1 fill:#FF9800
    style RM1 fill:#F44336
```

## Integration Adapter Pattern

```mermaid
graph TB
    subgraph "Strategy Manager"
        SM[Strategy Manager Contract]
    end
    
    subgraph "Adapter Interface"
        AI[LiquidStakingAdapter<br/>Interface]
    end
    
    subgraph "Concrete Adapters"
        BA[Bifrost Adapter]
        AA[Acala Adapter]
        OA[Other Adapters]
    end
    
    subgraph "XCM Layer"
        XCM[XCM Handler]
        MSG[Message Builder]
        EXEC[Message Executor]
    end
    
    subgraph "External Protocols"
        BP[Bifrost Protocol]
        AP[Acala Protocol]
        OP[Other Protocols]
    end
    
    SM --> AI
    AI --> BA
    AI --> AA
    AI --> OA
    
    BA --> XCM
    AA --> XCM
    OA --> XCM
    
    XCM --> MSG
    MSG --> EXEC
    EXEC --> BP
    EXEC --> AP
    EXEC --> OP
    
    BP --> EXEC
    AP --> EXEC
    OP --> EXEC
    
    EXEC --> XCM
    XCM --> BA
    XCM --> AA
    XCM --> OA
    
    style AI fill:#4CAF50
    style XCM fill:#2196F3
    style EXEC fill:#FF9800
```

## Governance Flow

```mermaid
graph TD
    START[Proposal Created] --> SUBMIT[Submit to Governance]
    SUBMIT --> REVIEW[Review Period]
    
    REVIEW --> VOTE{Voting Period}
    VOTE --> FOR[Vote For]
    VOTE --> AGAINST[Vote Against]
    VOTE --> ABSTAIN[Abstain]
    
    FOR --> COUNT[Count Votes]
    AGAINST --> COUNT
    ABSTAIN --> COUNT
    
    COUNT --> THRESHOLD{Threshold Met?}
    THRESHOLD -->|No| REJECT[Proposal Rejected]
    THRESHOLD -->|Yes| DELAY[Enactment Delay]
    
    DELAY --> EXECUTE[Execute Proposal]
    EXECUTE --> TYPE{Proposal Type?}
    
    TYPE -->|Parameter Change| PARAM[Update Parameters]
    TYPE -->|Strategy Update| STRAT[Update Strategy]
    TYPE -->|Integration| INTEG[Add/Remove Protocol]
    TYPE -->|Emergency| EMERG[Emergency Action]
    
    PARAM --> APPLY[Apply Changes]
    STRAT --> APPLY
    INTEG --> APPLY
    EMERG --> APPLY
    
    APPLY --> VERIFY[Verify Changes]
    VERIFY --> COMPLETE[Proposal Complete]
    
    style START fill:#4CAF50
    style VOTE fill:#2196F3
    style EXECUTE fill:#FF9800
    style COMPLETE fill:#4CAF50
```

## Data Flow: Reward Aggregation

```mermaid
graph LR
    subgraph "Reward Sources"
        RS1[Bifrost vDOT<br/>Rewards]
        RS2[Acala LDOT<br/>Rewards]
        RS3[Direct Staking<br/>Rewards]
    end
    
    subgraph "Reward Collection"
        RC1[Query Exchange Rates]
        RC2[Calculate Rate Changes]
        RC3[Compute Rewards]
    end
    
    subgraph "Reward Aggregation"
        RA1[Sum All Rewards]
        RA2[Calculate Total APY]
        RA3[Apply Protocol Fees]
    end
    
    subgraph "Rebasing Execution"
        RE1[Calculate New Supply]
        RE2[Update Total Supply]
        RE3[Rebase All Balances]
    end
    
    RS1 --> RC1
    RS2 --> RC1
    RS3 --> RC1
    
    RC1 --> RC2
    RC2 --> RC3
    
    RC3 --> RA1
    RA1 --> RA2
    RA2 --> RA3
    
    RA3 --> RE1
    RE1 --> RE2
    RE2 --> RE3
    
    style RS1 fill:#4CAF50
    style RA1 fill:#2196F3
    style RE1 fill:#FF9800
```

