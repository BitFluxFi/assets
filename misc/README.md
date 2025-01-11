## 1. Account Position and Pool Data
### Use this query to fetch account-specific positions and current pool state:

```graphql
query GetAccountPoolPosition {
  # Get the swap (pool) information
  swap(id: "POOL_ADDRESS") {
    id
    balances
    virtualPrice
    tokens {
      symbol
      decimals
      address
    }
  }
  
  # Get add liquidity events
  addLiquidityEvents: addLiquidityEvents(
    where: {
      provider: "ACCOUNT_ADDRESS",
      swap: "POOL_ADDRESS"
    },
    orderBy: timestamp,
    orderDirection: desc
  ) {
    provider
    tokenAmounts
    lpTokenSupply
    timestamp
  }
  
  # Get remove liquidity events
  removeLiquidityEvents: removeLiquidityEvents(
    where: {
      provider: "ACCOUNT_ADDRESS",
      swap: "POOL_ADDRESS"
    },
    orderBy: timestamp,
    orderDirection: desc
  ) {
    provider
    tokenAmounts
    lpTokenSupply
    timestamp
  }
  
  # Get latest daily balance
  dailyBalances(
    first: 1,
    where: { swap: "POOL_ADDRESS" },
    orderBy: timestamp,
    orderDirection: desc
  ) {
    timestamp
    balances
  }
}
```

## 2. Historical TVL and Volume Data
### Use this query to fetch historical pool metrics:

```graphql
query GetPoolMetrics {
  # Daily volumes and balances
  dailyVolumes(
    where: { 
      swap: "POOL_ADDRESS",
      # timestamp_gte: $startTime  # Optional: Add time filter
    },
    orderBy: timestamp,
    orderDirection: asc
  ) {
    timestamp
    volume
  }
  
  dailyBalances(
    where: { 
      swap: "POOL_ADDRESS",
      # timestamp_gte: $startTime  # Optional: Add time filter
    },
    orderBy: timestamp,
    orderDirection: asc
  ) {
    timestamp
    balances
  }
}
```

## Calculation Guide

### Account's Pool Share
1. Track LP Token Position:
   - Sum all deposits from `addLiquidityEvents`
   - Subtract all withdrawals from `removeLiquidityEvents`
   - Compare against current `lpTokenSupply`

### Pool TVL
1. Current TVL:
   - Use current balances from pool data
   - Adjust using token decimals
   - Multiply by token prices (external source needed)

2. Historical TVL:
   - Use `dailyBalances` for time series data
   - Apply same calculation as current TVL

### Account's Value in Pool
- Calculate share percentage from LP tokens
- Multiply total pool TVL by share percentage

## Important Considerations

### Token Prices
- Not provided by subgraph
- External price source required
- Should match balance timestamps

### Precision Handling
- Consider token decimals
- Use appropriate precision for calculations
- Recommended to use basis points for percentages

### Historical Data
- Available through daily snapshots
- Useful for:
  - TVL tracking
  - Volume analysis
  - Time-weighted calculations

### Virtual Price
- Indicates pool growth
- Includes accumulated fees
- Used for share value calculations

## External Requirements
- Token price feed
- Decimal math handling
- Time series data processing (for historical analysis)

## Note
Replace `POOL_ADDRESS` and `ACCOUNT_ADDRESS` with actual addresses when using the queries.