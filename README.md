# NEMO Protocol SDK Documentation

## Table of Contents

1. [Overview](#overview)
2. [Installation](#installation)
3. [Core Features](#core-features)
4. [API Reference](#api-reference)
5. [Type Definitions](#type-definitions)

---

## Overview

The NEMO Protocol SDK provides core functionality for querying user positions, calculating market values, and managing yield distributions in the NEMO protocol.

### Key Features

- **Position Query**: Query all positions for any Sui address
- **Holder Statistics**: Get PT, YT, LP holder counts and position quantities
- **Market Value Calculation**: Calculate current market values in SUI or underlying assets
- **Yield Management**: Query and claim available yield
- **Pool Information**: Get comprehensive pool statistics and market data
- **Liquidity Management**: Add and remove liquidity positions with transaction execution

---

## Installation

### Package Installation

```bash
npm install @nemoprotocol/contract-sdk
# or
yarn add @nemoprotocol/contract-sdk
# or
pnpm add @nemoprotocol/contract-sdk
```

### Environment Configuration

```bash
# Build-time configuration
API_BASE_URL=https://api.nemoprotocol.com npm run build

# Development configuration (.env file)
API_BASE_URL=https://api.nemoprotocol.com
```

### SDK Initialization

```typescript
import { PositionQuery, PoolQuery, AddLiquidityAction } from '@nemoprotocol/contract-sdk'
import { SuiClient, getFullnodeUrl } from '@mysten/sui/client'

// Initialize query classes
const positionQuery = new PositionQuery({
  network: 'mainnet', // or 'testnet', 'devnet', 'localnet'
  rpcUrl: 'https://sui-mainnet.mystenlabs.com' // optional
})

const poolQuery = new PoolQuery()

// Initialize action class for transaction execution
const suiClient = new SuiClient({ url: getFullnodeUrl('mainnet') })
const liquidityAction = new AddLiquidityAction({
  suiClient,
  privateKeyHex: 'your-private-key-hex' // without 0x prefix or with it
})
```

---

## Core Features

### 1. Position Query

Query all positions for a specific Sui address in the NEMO protocol.

#### LP Position Query

```typescript
const lpPositions = await positionQuery.queryLpPositions({
  address: '0x...', // user address
  positionTypes: [
    '0x...::market::LpPosition<0x...::coin::SUI>',
    '0x...::market::LpPosition<0x...::coin::USDC>'
  ],
  maturity: '2024-12-31', // optional filter
  marketStateId: '0x...' // optional filter
})

// Response format:
// [
//   {
//     id: { id: '0x...' },
//     name: 'LP-SUI-2024-12-31',
//     expiry: '2024-12-31',
//     lpAmount: '1000000000',
//     description: 'LP position for SUI',
//     marketStateId: '0x...'
//   }
// ]
```

#### PY Position Query

```typescript
const pyPositions = await positionQuery.queryPyPositions({
  address: '0x...', // user address
  positionTypes: [
    '0x...::py_state::PyState<0x...::coin::SUI>',
    '0x...::py_state::PyState<0x...::coin::USDC>'
  ],
  maturity: '2024-12-31', // optional filter
  pyStateId: '0x...' // optional filter
})

// Response format:
// [
//   {
//     id: '0x...',
//     maturity: '2024-12-31',
//     ptBalance: '500000000',
//     ytBalance: '300000000',
//     pyStateId: '0x...'
//   }
// ]
```

### 2. Holder Statistics

Get users holding different asset types (PT, YT, LP) and their position quantities.

#### PT/YT Holder Count

```typescript
const holdersCount = await positionQuery.queryPyPositionHoldersCount({
  positionTypes: [
    '0x...::py_state::PyState<0x...::coin::SUI>',
    '0x...::py_state::PyState<0x...::coin::USDC>'
  ],
  maturity: '2024-12-31',
  pyStateId: '0x...',
  pageSize: 50 // optional
})

console.log('PT Holders:', holdersCount.ptHolders)
console.log('YT Holders:', holdersCount.ytHolders)
console.log('Total Holders:', holdersCount.totalHolders)
console.log('Holders by Type:', holdersCount.holdersByType)
console.log('Total Positions:', holdersCount.totalPositions)
```

#### LP Holder Count

```typescript
const lpHoldersCount = await positionQuery.queryLpPositionHoldersCount({
  positionTypes: [
    '0x...::market::LpPosition<0x...::coin::SUI>',
    '0x...::market::LpPosition<0x...::coin::USDC>'
  ],
  maturity: '2024-12-31',
  marketStateId: '0x...',
  pageSize: 50 // optional
})

console.log('LP Holders:', lpHoldersCount.totalHolders)
console.log('Holders by Type:', lpHoldersCount.holdersByType)
console.log('Total Positions:', lpHoldersCount.totalPositions)
```

### 3. Market Value Calculation

Calculate and display current market values of user positions (priced in SUI or underlying assets).

#### Yield Query

```typescript
const yieldResult = await positionQuery.queryYield({
  address: '0x...',
  ytBalance: '1000000000',
  pyPositions: [
    {
      id: '0x...',
      maturity: '2024-12-31',
      ptBalance: '500000000',
      ytBalance: '300000000',
      pyStateId: '0x...'
    }
  ],
  receivingType: 'sy', // or 'underlying'
  config: {
    nemoContractId: '0x...',
    version: '1.0.0',
    coinType: '0x...::coin::SUI',
    pyStateId: '0x...',
    syCoinType: '0x...::sy_coin::SyCoin<0x...::coin::SUI>',
    yieldFactoryConfigId: '0x...',
    marketStateId: '0x...',
    underlyingCoinType: '0x...::coin::SUI',
    underlyingProtocol: 'Scallop',
    priceOracleConfigId: '0x...',
    oraclePackageId: '0x...',
    oracleTicket: '0x...',
    oracleVoucherPackageId: '0x...'
  }
})

console.log('Output Value:', yieldResult.outputValue)
console.log('Output Amount:', yieldResult.outputAmount)
```

### 4. Rewards query

Query and claim available yield rewards in the NEMO protocol.

```typescript
// Query rewards for multiple reward metrics
const rewards = await positionQuery.queryRewards({
  address: '0x...', // user address
  config: {
    nemoContractId: '0x...',
    version: '0x...',
    marketStateId: '0x...',
    // other required config fields
  },
  lpPositions: [
    {
      id: { id: '0x...' },
      name: 'LP-SUI-2024-12-31',
      expiry: '2024-12-31',
      lpAmount: '1000000000',
      description: 'LP position for SUI',
      marketStateId: '0x...'
    }
  ],
  rewardMetrics: [
    {
      tokenType: '0x...::coin::SUI',
      syCoinType: '0x...::sy_coin::SyCoin<0x...::coin::SUI>',
      tokenLogo: 'https://...',
      dailyEmission: '1000000000',
      tokenPrice: '1.23',
      tokenName: 'SUI',
      decimal: '9'
    },
    {
      tokenType: '0x...::coin::USDC',
      syCoinType: '0x...::sy_coin::SyCoin<0x...::coin::USDC>',
      tokenLogo: 'https://...',
      dailyEmission: '500000000',
      tokenPrice: '1.00',
      tokenName: 'USDC',
      decimal: '6'
    }
  ]
});

// Response format:
// [
//   {
//     coinType: '0x...::coin::SUI',
//     coinName: 'SUI',
//     amount: '1.234567890'
//   },
//   {
//     coinType: '0x...::coin::USDC',
//     coinName: 'USDC',
//     amount: '100.123456'
//   }
// ]
```

### 5. Pool Information

Query basic information and statistics for different pools.

#### Pool Query

```typescript
const pools = await poolQuery.queryPools()

// Response format:
// [
//   {
//     id: '0x...',
//     tvl: '1000000000000',
//     tvlRateChange: '5.2',
//     coinLogo: 'https://...',
//     maturity: '2024-12-31',
//     startTime: '2024-01-01',
//     coinName: 'SUI',
//     coinType: '0x...::coin::SUI',
//     ptTokenType: '0x...::pt_coin::PtCoin<0x...::coin::SUI>',
//     nemoContractId: '0x...',
//     boost: '1.5',
//     provider: 'Scallop',
//     providerLogo: 'https://...',
//     cap: '10000000000000',
//     marketStateId: '0x...',
//     syCoinType: '0x...::sy_coin::SyCoin<0x...::coin::SUI>',
//     underlyingCoinType: '0x...::coin::SUI',
//     providerMarket: '0x...',
//     providerVersion: '1.0.0',
//     priceOracleConfigId: '0x...',
//     decimal: '9',
//     underlyingApy: '8.5',
//     coinPrice: '1.2',
//     underlyingPrice: '1.2',
//     pyStateId: '0x...',
//     syStateId: '0x...',
//     conversionRate: '1.0',
//     marketFactoryConfigId: '0x...',
//     swapFeeForLpHolder: '0.003',
//     underlyingCoinName: 'SUI',
//     underlyingCoinLogo: 'https://...',
//     version: '1.0.0',
//     perPoints: '1000000',
//     oraclePackageId: '0x...',
//     oracleTicket: '0x...',
//     oracleVoucherPackageId: '0x...',
//     yieldTokenType: '0x...::yt_coin::YtCoin<0x...::coin::SUI>',
//     tokenRegistryState: '0x...',
//     ptPrice: '0.95',
//     ptTvl: '500000000000',
//     syTvl: '500000000000',
//     marketState: {
//       marketCap: '1000000000000',
//       totalSy: '500000000000',
//       lpSupply: '1000000000000',
//       totalPt: '500000000000',
//       rewardMetrics: [...]
//     },
//     scaledPtApy: '12.5',
//     scaledUnderlyingApy: '8.5',
//     feeApy: '2.0',
//     sevenAvgUnderlyingApy: '8.2',
//     sevenAvgUnderlyingApyRateChange: '0.3',
//     ytPrice: '0.15',
//     lpPrice: '1.0',
//     ytTokenLogo: 'https://...',
//     ptTokenLogo: 'https://...',
//     lpTokenLogo: 'https://...',
//     ytReward: '150000000',
//     underlyingProtocol: 'Scallop',
//     yieldFactoryConfigId: '0x...',
//     pyPositionTypeList: ['0x...::py_state::PyState<0x...::coin::SUI>'],
//     marketPositionTypeList: ['0x...::market::LpPosition<0x...::coin::SUI>'],
//     lpPriceRateChange: '2.1',
//     ptPriceRateChange: '-1.5',
//     ytPriceRateChange: '5.2',
//     incentiveApy: '3.0',
//     incentives: [...],
//     poolApy: '15.5',
//     tradeStatus: 'active'
//   }
// ]
```

### 6. Liquidity Management

Execute liquidity-related transactions including adding and removing liquidity positions.

#### Add Liquidity

```typescript
// Add liquidity to a pool
const addResult = await liquidityAction.addLiquidity({
  decimal: 9,
  addType: 'SY', // 'SY' | 'TOKEN' | 'PT'
  slippage: '0.01', // 1% slippage
  lpValue: '1000000000', // Amount in base units
  coinType: '0x...::coin::SUI',
  conversionRate: '1.0',
  addValue: '1000000000',
  tokenType: 0, // 0 for SY, 1 for TOKEN, 2 for PT
  action: 'mint', // 'mint' | 'swap'
  
  // Configuration objects
  coinConfig: {
    nemoContractId: '0x...',
    version: '1.0.0',
    coinType: '0x...::coin::SUI',
    pyStateId: '0x...',
    syCoinType: '0x...::sy_coin::SyCoin<0x...::coin::SUI>',
    yieldFactoryConfigId: '0x...',
    marketStateId: '0x...',
    underlyingCoinType: '0x...::coin::SUI',
    underlyingProtocol: 'Scallop',
    priceOracleConfigId: '0x...',
    oraclePackageId: '0x...',
    oracleTicket: '0x...',
    oracleVoucherPackageId: '0x...'
  },
  
  marketStateData: {
    // Market state object from pool query
  },
  
  coinData: [
    {
      coinType: '0x...::coin::SUI',
      balance: '1000000000',
      coinObjectId: '0x...'
    }
  ],
  
  pyPositionData: {
    // PY position data object
  },
  
  lpPositions: [
    {
      id: { id: '0x...' },
      name: 'LP-SUI-2024-12-31',
      expiry: '2024-12-31',
      lpAmount: '1000000000',
      description: 'LP position for SUI',
      marketStateId: '0x...'
    }
  ],
  
  // Optional parameters
  vaultId: '0x...', // optional
  insufficientBalance: false
})

// Check result
if (addResult.success) {
  console.log('Transaction successful:', addResult.transactionHash)
  console.log('Transaction data:', addResult.data)
} else {
  console.error('Transaction failed:', addResult.error)
}
```

#### Remove Liquidity

```typescript
// Remove liquidity from a position
const removeResult = await liquidityAction.removeLiquidity({
  lpAmount: '500000000', // Amount to remove
  slippage: '0.01', // 1% slippage
  ytBalance: '300000000',
  action: 'swap', // 'swap' | 'redeem'
  receivingType: 'underlying', // 'underlying' | 'sy'
  
  // Configuration objects
  coinConfig: {
    nemoContractId: '0x...',
    version: '1.0.0',
    coinType: '0x...::coin::SUI',
    pyStateId: '0x...',
    syCoinType: '0x...::sy_coin::SyCoin<0x...::coin::SUI>',
    yieldFactoryConfigId: '0x...',
    marketStateId: '0x...',
    underlyingCoinType: '0x...::coin::SUI',
    underlyingProtocol: 'Scallop',
    priceOracleConfigId: '0x...',
    oraclePackageId: '0x...',
    oracleTicket: '0x...',
    oracleVoucherPackageId: '0x...'
  },
  
  lpPositions: [
    {
      id: { id: '0x...' },
      name: 'LP-SUI-2024-12-31',
      expiry: '2024-12-31',
      lpAmount: '1000000000',
      description: 'LP position for SUI',
      marketStateId: '0x...'
    }
  ],
  
  pyPositions: [
    {
      id: '0x...',
      maturity: '2024-12-31',
      ptBalance: '500000000',
      ytBalance: '300000000',
      pyStateId: '0x...'
    }
  ],
  
  marketState: {
    marketCap: '1000000000000',
    totalSy: '500000000000',
    lpSupply: '1000000000000',
    totalPt: '500000000000',
    rewardMetrics: []
  },
  
  // Optional parameters
  vaultId: '0x...', // optional
  minSyOut: '480000000', // minimum SY output
  ptCoins: [
    {
      coinType: '0x...::pt_coin::PtCoin<0x...::coin::SUI>',
      balance: '500000000',
      coinObjectId: '0x...'
    }
  ],
  minValue: '480000000',
  isSwapPt: false
})

// Check result
if (removeResult.success) {
  console.log('Transaction successful:', removeResult.transactionHash)
  console.log('Transaction data:', removeResult.data)
} else {
  console.error('Transaction failed:', removeResult.error)
}
```

#### Error Handling

```typescript
try {
  const result = await liquidityAction.addLiquidity(params)
  
  if (!result.success) {
    // Handle business logic errors
    console.error('Business error:', result.error)
    
    // Common error scenarios:
    // - Insufficient balance
    // - Slippage too high
    // - Invalid parameters
    // - Market not active
  }
} catch (error) {
  // Handle unexpected errors
  console.error('Unexpected error:', error)
}
```

#### Utility Methods

```typescript
// Get current wallet address
const walletAddress = liquidityAction.getAddress()
console.log('Wallet address:', walletAddress)

// Get Sui client instance
const client = liquidityAction.getSuiClient()
```

---

## API Reference

### PositionQuery Class

#### Constructor

```typescript
new PositionQuery(config: PositionQueryConfig)
```

**Parameters:**
- `config.network`: Network type ('mainnet' | 'testnet' | 'devnet' | 'localnet')
- `config.rpcUrl`: Custom RPC URL (optional)

#### Methods

##### queryLpPositions()

```typescript
async queryLpPositions(options: {
  address: string;
  positionTypes: string[];
  maturity?: string;
  marketStateId?: string;
}): Promise<LpPosition[]>
```

##### queryPyPositions()

```typescript
async queryPyPositions(options: {
  address: string;
  positionTypes: string[];
  maturity?: string;
  pyStateId?: string;
}): Promise<PyPosition[]>
```

##### queryPyPositionHoldersCount()

```typescript
async queryPyPositionHoldersCount(options: {
  positionTypes: string[];
  maturity?: string;
  pyStateId?: string;
  pageSize?: number;
}): Promise<{
  ptHolders: number;
  ytHolders: number;
  totalHolders: number;
  holdersByType: Record<string, { ptHolders: number; ytHolders: number }>;
  totalPositions: number;
}>
```

##### queryLpPositionHoldersCount()

```typescript
async queryLpPositionHoldersCount(options: {
  positionTypes: string[];
  maturity?: string;
  marketStateId?: string;
  pageSize?: number;
}): Promise<{
  totalHolders: number;
  holdersByType: Record<string, number>;
  totalPositions: number;
}>
```

##### queryYield()

```typescript
async queryYield(params: QueryYieldParams): Promise<{
  outputValue: string;
  outputAmount: string;
}>
```

### PoolQuery Class

#### Constructor

```typescript
new PoolQuery()
```

#### Methods

##### queryPools()

```typescript
async queryPools(): Promise<PortfolioItem[]>
```

### AddLiquidityAction Class

#### Constructor

```typescript
new AddLiquidityAction(config: AddLiquidityActionConfig)
```

**Parameters:**
- `config.suiClient`: SuiClient instance for blockchain interaction
- `config.privateKeyHex`: Private key in hexadecimal format (with or without 0x prefix)

#### Methods

##### addLiquidity()

```typescript
async addLiquidity(params: AddLiquidityActionParams): Promise<AddLiquidityActionResult>
```

**Parameters:**
- `params.decimal`: Number of decimal places for the token
- `params.addType`: Type of asset to add ('SY' | 'TOKEN' | 'PT')
- `params.slippage`: Maximum slippage tolerance (e.g., '0.01' for 1%)
- `params.lpValue`: LP value amount in base units
- `params.coinType`: Coin type identifier
- `params.conversionRate`: Conversion rate for the operation
- `params.addValue`: Amount to add in base units
- `params.tokenType`: Token type (0 for SY, 1 for TOKEN, 2 for PT)
- `params.action`: Action type ('mint' | 'swap')
- `params.coinConfig`: Coin configuration object
- `params.marketStateData`: Market state data
- `params.coinData`: Array of coin data objects
- `params.pyPositionData`: PY position data
- `params.lpPositions`: Array of LP positions
- `params.vaultId`: Optional vault identifier
- `params.insufficientBalance`: Optional insufficient balance flag

##### removeLiquidity()

```typescript
async removeLiquidity(params: RemoveLiquidityActionParams): Promise<AddLiquidityActionResult>
```

**Parameters:**
- `params.lpAmount`: Amount of LP tokens to remove
- `params.slippage`: Maximum slippage tolerance
- `params.ytBalance`: YT balance amount
- `params.action`: Action type ('swap' | 'redeem')
- `params.receivingType`: Type of asset to receive ('underlying' | 'sy')
- `params.coinConfig`: Coin configuration object
- `params.lpPositions`: Array of LP positions
- `params.pyPositions`: Array of PY positions
- `params.marketState`: Market state object
- `params.vaultId`: Optional vault identifier
- `params.minSyOut`: Optional minimum SY output
- `params.ptCoins`: Optional PT coins array
- `params.minValue`: Optional minimum value
- `params.isSwapPt`: Optional PT swap flag

##### execute()

```typescript
async execute(params: AddLiquidityActionParams): Promise<AddLiquidityActionResult>
```

Alias for `addLiquidity()` method for backward compatibility.

##### getAddress()

```typescript
getAddress(): string
```

Returns the wallet address associated with the private key.

##### getSuiClient()

```typescript
getSuiClient(): SuiClient
```

Returns the SuiClient instance used for blockchain interactions.

---

## Type Definitions

### PositionQueryConfig

```typescript
interface PositionQueryConfig {
  network?: 'mainnet' | 'testnet' | 'devnet' | 'localnet';
  rpcUrl?: string;
}
```

### LpPosition

```typescript
interface LpPosition {
  id: { id: string };
  name: string;
  expiry: string;
  lpAmount: string;
  description: string;
  marketStateId: string;
}
```

### PyPosition

```typescript
interface PyPosition {
  id: string;
  maturity: string;
  ptBalance: string;
  ytBalance: string;
  pyStateId: string;
}
```

### PortfolioItem

```typescript
interface PortfolioItem {
  id: string;
  tvl: string;
  tvlRateChange: string;
  coinLogo: string;
  maturity: string;
  startTime: string;
  coinName: string;
  coinType: string;
  ptTokenType: string;
  nemoContractId: string;
  boost: string;
  provider: string;
  providerLogo: string;
  cap: string;
  marketStateId: string;
  syCoinType: string;
  underlyingCoinType: string;
  providerMarket: string;
  providerVersion: string;
  priceOracleConfigId: string;
  decimal: string;
  underlyingApy: string;
  coinPrice: string;
  underlyingPrice: string;
  pyStateId: string;
  syStateId: string;
  conversionRate: string;
  marketFactoryConfigId: string;
  swapFeeForLpHolder: string;
  underlyingCoinName: string;
  underlyingCoinLogo: string;
  version: string;
  perPoints: string;
  oraclePackageId: string;
  oracleTicket: string;
  oracleVoucherPackageId: string;
  yieldTokenType: string;
  tokenRegistryState: string;
  ptPrice: string;
  ptTvl: string;
  syTvl: string;
  marketState: MarketState;
  scaledPtApy: string;
  scaledUnderlyingApy: string;
  feeApy: string;
  sevenAvgUnderlyingApy: string;
  sevenAvgUnderlyingApyRateChange: string;
  ytPrice: string;
  lpPrice: string;
  ytTokenLogo: string;
  ptTokenLogo: string;
  lpTokenLogo: string;
  ytReward: string;
  underlyingProtocol: string;
  yieldFactoryConfigId: string;
  pyPositionTypeList: string[];
  marketPositionTypeList: string[];
  lpPriceRateChange: string;
  ptPriceRateChange: string;
  ytPriceRateChange: string;
  incentiveApy: string;
  incentives: Incentive[];
  poolApy: string;
  tradeStatus: string;
}
```

### AddLiquidityActionConfig

```typescript
interface AddLiquidityActionConfig {
  suiClient: SuiClient;
  privateKeyHex: string; // hexadecimal private key (with or without 0x prefix)
}
```

### AddLiquidityActionParams

```typescript
interface AddLiquidityActionParams {
  // Basic parameters
  decimal: number;
  addType: string;
  slippage: string;
  lpValue: string;
  coinType: string;
  conversionRate: string;
  addValue: string;
  tokenType: number;
  action: string; // "mint" | "swap"
  
  // Configuration and data
  coinConfig: CoinConfig;
  marketStateData: any;
  coinData: CoinData[];
  pyPositionData: any;
  lpPositions: LpPosition[];
  
  // Optional parameters
  vaultId?: string;
  insufficientBalance?: boolean;
}
```

### RemoveLiquidityActionParams

```typescript
interface RemoveLiquidityActionParams {
  lpAmount: string;
  slippage: string;
  vaultId?: string;
  minSyOut?: string;
  ytBalance: string;
  ptCoins?: CoinData[];
  coinConfig: CoinConfig;
  action: "swap" | "redeem";
  lpPositions: LpPosition[];
  pyPositions: any[];
  minValue?: string | number;
  isSwapPt?: boolean;
  receivingType?: "underlying" | "sy";
  marketState: MarketState;
}
```

### AddLiquidityActionResult

```typescript
interface AddLiquidityActionResult {
  success: boolean;
  transactionHash?: string;
  error?: string;
  data?: any;
}
```

### CoinConfig

```typescript
interface CoinConfig {
  nemoContractId: string;
  version: string;
  coinType: string;
  pyStateId: string;
  syCoinType: string;
  yieldFactoryConfigId: string;
  marketStateId: string;
  underlyingCoinType: string;
  underlyingProtocol: string;
  priceOracleConfigId: string;
  oraclePackageId: string;
  oracleTicket: string;
  oracleVoucherPackageId: string;
}
```

### CoinData

```typescript
interface CoinData {
  coinType: string;
  balance: string;
  coinObjectId: string;
}
```

### MarketState

```typescript
interface MarketState {
  marketCap: string;
  totalSy: string;
  lpSupply: string;
  totalPt: string;
  rewardMetrics: any[];
}
```

---

**Document Version**: 1.1

**Last Updated**: [Date] 