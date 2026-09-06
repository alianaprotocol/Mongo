# Meteora 代币启动完整配置文档

> **项目名称**：Mongo
> **文档版本**：v1.0
> **最后更新**：2026-09-06
> **部署网络**：☐ Devnet ☐ Mainnet

---

## 一、总览

### 1.1 设计目标
# Meteora Token Launch — Complete Configuration Document

> **Project Name**: Mongo
> **Document Version**: v1.0
> **Last Updated**: 2026-09-06
> **Network**: ☐ Devnet ☐ Mainnet

---

## Executive Summary

This document outlines a three-phase token launch strategy designed to deliver **explosive early momentum** followed by **long-term stability**.

| Phase | Component | Objective |
|-------|-----------|-----------|
| **I** | Alpha Vault (Pro Rata) | Fair launch with significant early wealth effect |
| **II** | DBC 3-Segment Curve | Price discovery: Explosion → Expansion → Stability |
| **III** | DAMM v2 | Sustainable liquidity with **100% permanent lock** |

**Core Metrics**:

| Metric | Value |
|--------|-------|
| Total Supply | 1,000,000,000 tokens |
| DBC Supply Mode | **Dynamic Supply** (minted on demand) |
| Reserved Supply | 3% (30,000,000 tokens, unminted reserve) |
| Vault Entry Price | $0.0015 |
| Migration Price | ~$0.01 |
| Vault User Paper Gain | **~567%** |
| Vault Unlock | Day 7 → Day 30 (linear) |

---

## Phase I: Alpha Vault (Pro Rata)

### Overview

Alpha Vault is Meteora's pre-launch allocation mechanism. It collects Quote Token deposits before public trading begins, then executes buys during a protected pre-activation window — preventing sniper bots from front-running.

**Pro Rata Mode**: Total deposits may exceed `max_buying_cap`. Allocations are distributed proportionally.

### Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Vault Mode** | **Pro Rata** | Proportional allocation on oversubscription |
| **`max_buying_cap`** | **20,000 USDC** | Maximum swap amount |
| **Vault Execution Price** | **$0.0015** | Price at which Vault buys |
| **Per-Wallet Soft Cap** | **200–400 USDC** | Frontend-enforced, ensures distribution |
| **`depositing_point`** | 48–72 hrs post-announcement | Deposit window opens |
| **`last_join_point`** | `depositing_point` + 96 hrs | Deposit window closes |
| **`pre_activation_start_point`** | `last_join_point` + **1 min** | Vault begins filling |
| **`last_buying_point`** | `pre_activation_start_point` + **2 hrs** | Vault fill ends |
| **`start_vesting_point`** | **TGE + 7 days** | Linear unlock begins |
| **`end_vesting_point`** | **TGE + 30 days** | Fully unlocked |
| **`escrow_fee`** | **0.3%** | Escrow service fee |

### Timeline Diagram

```
T-4d            T           T+1m        T+2h        T+7d        T+30d
 │              │             │           │           │           │
depositing  last_join   pre_activat  last_buy   start_vest  end_vest
 │              │             │           │           │           │
 ├── Deposit ──┤             ├── Fill ──┤           ├── Unlock ──┤
 │              │             │           │           │           │
 └─ Deposit/Withdraw ─┘   └─ Vault Buys ─┘   └─ Linear (24 days) ─┘
```

### Pro Rata Allocation Formula

```
Vault Actual Swap = min(Total Deposits, max_buying_cap)
User Allocation = User Deposit / Total Deposits
User Token Receive = Vault Actual Swap × User Allocation / Vault Price
```

**Example**:
- `max_buying_cap` = 20,000 USDC
- Total deposits = 50,000 USDC (oversubscribed)
- Vault swaps only 20,000 USDC
- User A deposits 1,000 USDC → Receives = 20,000 × (1,000/50,000) / 0.0015 = 266,667 tokens
- Excess (30,000 USDC) refunded proportionally

### Per-Wallet Soft Cap Design

Pro Rata mode defaults to `u64::MAX` (no cap). **Enforce via frontend**:

| Cap | Minimum Addresses to Fill Vault |
|-----|--------------------------------|
| 200 USDC | 100 addresses |
| 400 USDC | 50 addresses |

### Vesting Schedule

Claims unlock **linearly** from `start_vesting_point` to `end_vesting_point`. Both endpoints are inclusive.

```
Total Unlock Period = end_vesting_point - start_vesting_point + 1 = 24 days
Claimable Ratio = Time Elapsed / Total Unlock Period
```

### Fill Execution Plan

Vault fills are **permissionless** — anyone can trigger during the pre-activation window.

| Item | Recommendation |
|------|----------------|
| Cranker | **Team-run** |
| Whitelist | **Add Cranker address to whitelist** (waives 0.0001 SOL per fill) |
| Fill Batches | **10–20 transactions** |
| Per-Batch Cap | **1,000–2,000 USDC** |
| Fill Interval | Every 6–12 minutes |
| `minimum_amount_out` | **0** (default) |

### Sample Initialization Code

```typescript
import { AlphaVault } from "@meteora-ag/alpha-vault-sdk";

const params = {
  pool_type: PoolType.DAMM_V2,
  quote_mint: USDC_MINT,
  base_mint: YOUR_TOKEN_MINT,
  depositing_point: new BN(depositStartTimestamp),
  start_vesting_point: new BN(tgeTimestamp + 7 * 24 * 60 * 60),
  end_vesting_point: new BN(tgeTimestamp + 30 * 24 * 60 * 60),
  max_buying_cap: new BN(20_000 * 1_000_000), // 6 decimals
  escrow_fee: new BN(30), // 0.3% in basis points
  whitelist_mode: WhitelistMode.Permissionless,
};

const tx = await AlphaVault.createCustomizableProrataVault(
  connection,
  params,
  ownerWallet
);
```

---

## Phase II: DBC (Dynamic Bonding Curve)

### Overview

DBC is a customizable bonding curve launch protocol supporting **up to 16 price-liquidity segments** to control curve shape and price behavior.

**Core Mechanics**:
- Each segment follows the constant product formula: `x × y = k`
- Virtual liquidity `L` controls price movement speed: **Lower L = faster moves**
- Migration price is **automatically derived**, not manually set

### Supply Mode: Dynamic Supply

| Mode | Behavior | Leftover |
|------|----------|----------|
| **Dynamic Supply** | Tokens minted on demand | None |
| Fixed Supply | All tokens pre-minted | Large leftover |

**This plan uses Dynamic Supply** — 97% of tokens flow naturally to users and liquidity pools.

### 3-Segment Curve Design

| Segment | Price Range | Liquidity (L) | Target Quote | Effect |
|---------|-------------|---------------|--------------|--------|
| **S1: Explosion** | $0.001 → $0.003 | **Very Low** | 0 → 15,000 USDC | 3× surge, creates FOMO |
| **S2: Expansion** | $0.003 → $0.006 | **Medium** | 15,000 → 45,000 USDC | Steady climb, attracts buyers |
| **S3: Stability** | $0.006 → $0.01 | **High** | 45,000 → 100,000 USDC | Smooth approach to migration |

### Curve Formulas

**Single Segment**:

```
Base Amount = L × (1/√P_lower - 1/√P_upper)
Quote Amount = L × (√P_upper - √P_lower)
```

**Full Curve**:

```
Total Base = Σ L_i × (1/P_{i-1} - 1/P_i)
Migration Threshold = Σ L_i × (P_i - P_{i-1})
```

**Migration Price**:

```
P_migration² = Migration Threshold × (1 - Migration Fee) / Migration Amount
```

### SDK Sample Code

```typescript
import {
  DynamicBondingCurveClient,
  buildCurveWithCustomSqrtPrices,
  createSqrtPrices,
  TokenDecimal,
  ActivationType,
  MigrationOption,
  MigrationFeeOption,
  DammV2BaseFeeMode,
  DammV2DynamicFeeMode,
  BaseFeeMode,
  CollectFeeMode,
} from "@meteora-ag/dynamic-bonding-curve-sdk";

// 1. Define 4 price checkpoints = 3 segments
const sqrtPrices = createSqrtPrices(
  [0.001, 0.003, 0.006, 0.01],
  TokenDecimal.SIX,   // USDC: 6 decimals
  TokenDecimal.NINE   // Project token: 9 decimals
);

// 2. Liquidity weights per segment
// Segment 1: Very low (weight 1) → Explosion
// Segment 2: Medium (weight 3) → Expansion
// Segment 3: High (weight 6) → Stability
const liquidityWeights = [1, 3, 6];

// 3. Build curve configuration
const curveConfig = buildCurveWithCustomSqrtPrices({
  token: {
    tokenType: TokenType.SPLToken,
    tokenBaseDecimal: TokenDecimal.SIX,
    tokenQuoteDecimal: TokenDecimal.NINE,
    tokenAuthorityOption: TokenAuthorityOption.PartnerUpdateAuthority,
    totalTokenSupply: 1_000_000_000,
    leftover: 0, // Dynamic Supply = no leftover
  },
  fee: {
    baseFeeParams: {
      baseFeeMode: BaseFeeMode.FeeSchedulerExponential,
      feeSchedulerParam: {
        startingFeeBps: 9000,
        endingFeeBps: 120,
        numberOfPeriod: 60,
        totalDuration: 60,
      },
    },
    dynamicFeeEnabled: true,
    collectFeeMode: CollectFeeMode.QuoteToken,
    creatorTradingFeePercentage: 0,
    poolCreationFee: 1,
    enableFirstSwapWithMinFee: false,
  },
  migration: {
    migrationOption: MigrationOption.MET_DAMM_V2,
    migrationFeeOption: MigrationFeeOption.Customizable,
    migrationFee: {
      feePercentage: 10,
      creatorFeePercentage: 50,
    },
    migratedPoolFee: {
      collectFeeMode: MigratedCollectFeeMode.QuoteToken,
      dynamicFee: DammV2DynamicFeeMode.Enabled,
      poolFeeBps: 30, // 0.3%
      baseFeeMode: DammV2BaseFeeMode.FeeTimeSchedulerLinear,
    },
  },
  liquidityDistribution: {
    // 100% Permanent Lock — see Phase III
    partnerLiquidityPercentage: 0,
    partnerPermanentLockedLiquidityPercentage: 50,
    creatorLiquidityPercentage: 0,
    creatorPermanentLockedLiquidityPercentage: 50,
  },
  lockedVesting: {
    totalLockedVestingAmount: 0,
    numberOfVestingPeriod: 0,
    cliffUnlockAmount: 0,
    totalVestingDuration: 0,
    cliffDurationFromMigrationTime: 0,
  },
  activationType: ActivationType.Timestamp,
  sqrtPrices,
  liquidityWeights,
});

const client = DynamicBondingCurveClient.create(connection, "confirmed");
const config = await client.partner.createConfig({
  payer: wallet.publicKey,
  partner: partnerKeypair,
  params: curveConfig,
});
```

### Curve Constraints

| Constraint | Requirement |
|------------|-------------|
| Segments | **1 – 16** |
| Prices | Must be strictly increasing |
| Liquidity | Positive for each segment |
| Migration Threshold | Must be reachable |
| Starting sqrt Price | **≥ 4,295,048,016** |

### DBC Parameter Summary

| Parameter | Value |
|-----------|-------|
| Supply Mode | **Dynamic Supply** |
| Quote Mint | USDC (6 decimals) |
| Base Mint | Your token (9 decimals) |
| Activation Type | Timestamp |
| Migration Target | **DAMM v2** |
| Migration Threshold | **100,000 USDC** |
| Pool Fee | **0.3%** (30 bps) |
| Migration Fee | **10%** |
| Creator Fee Share | **50%** |

---

## Phase III: DAMM v2 (Liquidity Migration)

### Migration Flow

When DBC's `quote_reserve` reaches `migration_quote_threshold`, the curve stops trading and enters migration.

**State Machine**:

```
PreBondingCurve → PostBondingCurve → LockedVesting → CreatedPool
  (Trading)        (Complete)        (Lock setup)   (DAMM v2 live)
```

**Migration Steps**:
1. Create DAMM v2 pool
2. Create migrated liquidity Position NFTs
3. Apply configured permanent locks or vesting

### Liquidity Distribution: 100% Permanent Lock

**DBC Requirement**: Total distribution must **= 100%**, with at least **10%** locked on day one.

**This Plan: 100% Permanent Lock**

| Allocation | Percentage | Description |
|------------|------------|-------------|
| Partner Permanent Lock | **50%** | Protocol share — permanently locked |
| Creator Permanent Lock | **50%** | Project share — permanently locked |
| Partner Unlocked | **0%** | — |
| Partner Vesting | **0%** | — |
| Creator Unlocked | **0%** | — |
| Creator Vesting | **0%** | — |
| **Total** | **100%** | Must sum exactly to 100% |

> **Permanent Lock Meaning**: Liquidity can never be withdrawn. LP positions still earn trading fees and yield rewards.

### DAMM v2 Pool Configuration

| Parameter | Value | Description |
|-----------|-------|-------------|
| Pool Type | **Concentrated Liquidity** | Position NFTs represent LP positions |
| Price Range | **$0.005 – $0.02** | ±100% around $0.01 migration price |
| Initial Price | **$0.01** | Derived from DBC migration |
| Pool Fee | **0.3%** (30 bps) | Must be between 0.1%–10% |
| Dynamic Fee | **Enabled** | Auto-adjusts to market volatility |
| Vesting Lock Cap | **2 years** | Maximum vesting duration |

### Liquidity Distribution Code

```typescript
const liquidityDistribution = {
  partnerLiquidityPercentage: 0,
  partnerPermanentLockedLiquidityPercentage: 50,
  partnerVestingLiquidityPercentage: 0,
  creatorLiquidityPercentage: 0,
  creatorPermanentLockedLiquidityPercentage: 50,
  creatorVestingLiquidityPercentage: 0,
  // Sum = 100%
};
```

---

## Price Path Overview

### Price by Stage

| Stage | Cumulative Quote | Price | Event |
|-------|------------------|-------|-------|
| Alpha Vault | — | **$0.0015** | Vault entry |
| DBC Launch | 0 | **$0.001** | Curve opens |
| Explosion End | 15,000 USDC | **$0.003** | 3× surge |
| Expansion End | 45,000 USDC | **$0.006** | 2× additional |
| **Migration Trigger** | **100,000 USDC** | **$0.01** | Graduation to DAMM v2 |
| DAMM v2 Open | — | **$0.01** | Initial trading price |

### Vault User Returns

| Metric | Value |
|--------|-------|
| Vault Entry | **$0.0015** |
| Migration Price | **$0.01** |
| Paper Gain | **~567%** |
| Unlock Starts | TGE + **7 days** |
| Fully Unlocked | TGE + **30 days** |

---

## Risk Management

### Risk Register

| Risk | Impact | Mitigation |
|------|--------|------------|
| Insufficient public buying | Migration not triggered | Prepare **20,000–30,000 USDC** bootstrap capital |
| Sniper bots front-running | Fairness compromised | Anti-Sniper Suite (fee scheduler, rate limiter) |
| Vault user concentrated selling | Price crash | 24-day linear unlock + DAMM v2 depth |
| Curve design失控 | Extreme volatility | **Devnet full testing** |
| 100% permanent lock irreversible | Loss of LP control | Confirm as final decision |

### Testing Plan

| Step | Network | Scope |
|------|---------|-------|
| 1 | **Localnet** | Meteora Invent local validator |
| 2 | **Devnet** | Full Vault → DBC → Migration flow |
| 3 | **Devnet** | Validate price path, vesting, leftovers |
| 4 | **Mainnet** | Deploy after verification |

---

## Deployment Checklist

### Pre-Launch

- [ ] Token mint created (SPL or Token 2022)
- [ ] Token decimals: 6–9
- [ ] Quote mint selected (USDC/SOL)
- [ ] Activation type confirmed (Slot / Timestamp)
- [ ] Vault parameters validated on Devnet
- [ ] 3-segment DBC curve validated on Devnet
- [ ] Migration flow validated on Devnet
- [ ] 100% permanent lock confirmed
- [ ] All parameters disclosed to community

### Launch

- [ ] Create DBC Config
- [ ] Create DBC virtual pool
- [ ] Initialize Alpha Vault (link to DBC pool)
- [ ] Open deposit window
- [ ] Execute fills (10–20 batches)
- [ ] Monitor DBC public trading
- [ ] Migration threshold reached → auto-migrate to DAMM v2

### Post-Launch

- [ ] Confirm DAMM v2 pool created
- [ ] Confirm 100% liquidity permanently locked
- [ ] Confirm Position NFTs created
- [ ] Monitor Vault vesting unlocks (from TGE+7d)
- [ ] Monitor DAMM v2 trading depth and stability

---

## Summary

| Phase | Component | Core Parameters | User Experience |
|-------|-----------|-----------------|-----------------|
| **Fair Launch** | Alpha Vault Pro Rata | 20K USDC cap, $0.0015 entry | 567% paper gain |
| **Price Discovery** | DBC 3-Segment Curve | 100K USDC threshold | Explosion → Expansion → Stability |
| **Long-Term Stability** | DAMM v2 | 100% permanent lock | Deep, sustainable liquidity |

---

## References

- [Meteora Documentation](https://docs.meteora.ag)
- [Alpha Vault Docs](https://docs.meteora.ag/helper-products/alpha-vault)
- [DBC Docs](https://docs.meteora.ag/core-products/dbc)
- [Meteora Invent (Launch Toolkit)](https://github.com/MeteoraAg/meteora-invent)
- [DBC TypeScript SDK](https://www.npmjs.com/package/@meteora-ag/dynamic-bonding-curve-sdk)

---

---

# Meteora 代币启动 — 完整配置文档

> **项目名称**：________
> **文档版本**：v1.0
> **最后更新**：________
> **网络**：☐ Devnet ☐ Mainnet

---

## 执行摘要

本文档概述了一个三阶段代币启动策略，旨在实现 **前期爆发性增长** 和 **长期稳定性**。

| 阶段 | 组件 | 目标 |
|------|------|------|
| **一** | Alpha Vault（Pro Rata） | 公平启动，制造早期财富效应 |
| **二** | DBC 三段式曲线 | 价格发现：爆发 → 扩张 → 稳定 |
| **三** | DAMM v2 | 可持续流动性，**100% 永久锁定** |

**核心数据**：

| 参数 | 数值 |
|------|------|
| 总供应量 | 1,000,000,000 枚 |
| DBC 供应模式 | **Dynamic Supply**（按需铸造） |
| 预留供应量 | 3%（30,000,000 枚，未铸造储备） |
| Vault 入场价 | $0.0015 |
| 迁移价格 | ~$0.01 |
| Vault 用户浮盈 | **~567%** |
| Vault 解锁 | 第 7 天 → 第 30 天（线性） |

---

## 阶段一：Alpha Vault（Pro Rata）

### 概述

Alpha Vault 是 Meteora 的预启动分配机制。它在公开交易前收集 Quote Token 存款，并在受保护的预激活窗口内执行买入——防止狙击机器人抢跑。

**Pro Rata 模式**：总存款可能超过 `max_buying_cap`，按比例分配。

### 参数

| 参数 | 建议值 | 说明 |
|------|--------|------|
| **Vault 模式** | **Pro Rata** | 超额认购时按比例分配 |
| **`max_buying_cap`** | **20,000 USDC** | Vault 最大 swap 额度 |
| **Vault 执行价** | **$0.0015** | Vault 买入价格 |
| **单钱包软上限** | **200–400 USDC** | 前端强制执行，确保分散 |
| **`depositing_point`** | 公告后 48–72 小时 | 存款窗口开启 |
| **`last_join_point`** | `depositing_point` + 96 小时 | 存款窗口关闭 |
| **`pre_activation_start_point`** | `last_join_point` + **1 分钟** | Vault 开始 Fill |
| **`last_buying_point`** | `pre_activation_start_point` + **2 小时** | Vault Fill 结束 |
| **`start_vesting_point`** | **TGE + 7 天** | 线性解锁开始 |
| **`end_vesting_point`** | **TGE + 30 天** | 完全解锁 |
| **`escrow_fee`** | **0.3%** | 托管费用 |

### 时间线图解

```
T-4天          T           T+1分钟      T+2小时       T+7天         T+30天
 │              │             │            │             │             │
depositing   last_join   pre_activat  last_buying  start_vest   end_vest
 │              │             │            │             │             │
 ├── 存款窗口 ──┤             ├── Fill ────┤             ├── 解锁期 ──┤
 │              │             │            │             │             │
 └─ 用户可存入/撤出 ─┘    └─ Vault买入 ──┘    └─ 线性解锁（24天）─┘
```

### Pro Rata 分配公式

```
Vault 实际 swap = min(总存款, max_buying_cap)
用户分配比例 = 用户存款 / 总存款
用户获得代币 = Vault 实际 swap × 用户分配比例 / Vault 执行价
```

**示例**：
- `max_buying_cap` = 20,000 USDC
- 总存款 = 50,000 USDC（超额认购）
- Vault 只 swap 20,000 USDC
- 用户 A 存款 1,000 USDC → 获得 = 20,000 × (1,000/50,000) / 0.0015 = 266,667 枚
- 超出部分（30,000 USDC）按比例退还

### 单钱包软上限设计

Pro Rata 模式默认无上限。**通过前端强制执行**：

| 上限 | 填满 Vault 所需最少地址数 |
|------|-------------------------|
| 200 USDC | 100 个地址 |
| 400 USDC | 50 个地址 |

### Vesting 解锁

从 `start_vesting_point` 到 `end_vesting_point` **线性解锁**，两端点均包含。

```
总解锁周期 = end_vesting_point - start_vesting_point + 1 = 24 天
可领取比例 = 已过时间 / 总解锁周期
```

### Fill 执行计划

Vault fills 是 **无需许可的**，任何人在预激活窗口内都可触发。

| 项目 | 建议 |
|------|------|
| Cranker | **团队自运行** |
| 白名单 | **将 Cranker 地址加入白名单**（免交每次 0.0001 SOL 费用） |
| Fill 批次数 | **10–20 笔** |
| 单笔上限 | **1,000–2,000 USDC** |
| Fill 间隔 | 每 6–12 分钟一笔 |
| `minimum_amount_out` | **0**（默认值） |

### 初始化代码示例

```typescript
import { AlphaVault } from "@meteora-ag/alpha-vault-sdk";

const params = {
  pool_type: PoolType.DAMM_V2,
  quote_mint: USDC_MINT,
  base_mint: YOUR_TOKEN_MINT,
  depositing_point: new BN(depositStartTimestamp),
  start_vesting_point: new BN(tgeTimestamp + 7 * 24 * 60 * 60),
  end_vesting_point: new BN(tgeTimestamp + 30 * 24 * 60 * 60),
  max_buying_cap: new BN(20_000 * 1_000_000), // 6 位小数
  escrow_fee: new BN(30), // 0.3%，单位基点
  whitelist_mode: WhitelistMode.Permissionless,
};

const tx = await AlphaVault.createCustomizableProrataVault(
  connection,
  params,
  ownerWallet
);
```

---

## 阶段二：DBC（动态绑定曲线）

### 概述

DBC 是一个可自定义的绑定曲线启动协议，支持配置**最多 16 个价格-流动性分段**来控制曲线形态和价格行为。

**核心机制**：
- 每个分段遵循恒定乘积公式：`x × y = k`
- 虚拟流动性 `L` 控制价格移动速度：**L 越低，价格移动越快**
- 迁移价格由曲线**自动推导**，非手动设置

### 供应模式：Dynamic Supply

| 模式 | 行为 | Leftover |
|------|------|----------|
| **Dynamic Supply** | 按需铸造 | 无 |
| Fixed Supply | 预先全部铸造 | 大量 Leftover |

**本方案使用 Dynamic Supply** — 97% 代币自然流向用户和流动性池。

### 三段式曲线设计

| 分段 | 价格区间 | 流动性 (L) | 目标 Quote | 效果 |
|------|---------|-----------|-----------|------|
| **S1：爆发段** | $0.001 → $0.003 | **极低** | 0 → 15,000 USDC | 3 倍暴涨，制造 FOMO |
| **S2：扩张段** | $0.003 → $0.006 | **中等** | 15,000 → 45,000 USDC | 稳步上涨，吸引买家 |
| **S3：稳定段** | $0.006 → $0.01 | **高** | 45,000 → 100,000 USDC | 平稳走向迁移 |

### 曲线公式

**单个分段**：

```
Base 数量 = L × (1/√P_lower - 1/√P_upper)
Quote 数量 = L × (√P_upper - √P_lower)
```

**全曲线**：

```
总 Base = Σ L_i × (1/P_{i-1} - 1/P_i)
迁移阈值 = Σ L_i × (P_i - P_{i-1})
```

**迁移价格**：

```
P_migration² = 迁移阈值 × (1 - 迁移费率) / 迁移数量
```

### SDK 代码示例

```typescript
import {
  DynamicBondingCurveClient,
  buildCurveWithCustomSqrtPrices,
  createSqrtPrices,
  TokenDecimal,
  ActivationType,
  MigrationOption,
  MigrationFeeOption,
  DammV2BaseFeeMode,
  DammV2DynamicFeeMode,
  BaseFeeMode,
  CollectFeeMode,
} from "@meteora-ag/dynamic-bonding-curve-sdk";

// 1. 定义 4 个价格检查点 = 3 个分段
const sqrtPrices = createSqrtPrices(
  [0.001, 0.003, 0.006, 0.01],
  TokenDecimal.SIX,   // USDC: 6 位小数
  TokenDecimal.NINE   // 项目代币: 9 位小数
);

// 2. 各分段流动性权重
// 分段1: 极低（权重1）→ 爆发
// 分段2: 中等（权重3）→ 扩张
// 分段3: 高（权重6）→ 稳定
const liquidityWeights = [1, 3, 6];

// 3. 构建曲线配置
const curveConfig = buildCurveWithCustomSqrtPrices({
  token: {
    tokenType: TokenType.SPLToken,
    tokenBaseDecimal: TokenDecimal.SIX,
    tokenQuoteDecimal: TokenDecimal.NINE,
    tokenAuthorityOption: TokenAuthorityOption.PartnerUpdateAuthority,
    totalTokenSupply: 1_000_000_000,
    leftover: 0, // Dynamic Supply = 无 Leftover
  },
  fee: {
    baseFeeParams: {
      baseFeeMode: BaseFeeMode.FeeSchedulerExponential,
      feeSchedulerParam: {
        startingFeeBps: 9000,
        endingFeeBps: 120,
        numberOfPeriod: 60,
        totalDuration: 60,
      },
    },
    dynamicFeeEnabled: true,
    collectFeeMode: CollectFeeMode.QuoteToken,
    creatorTradingFeePercentage: 0,
    poolCreationFee: 1,
    enableFirstSwapWithMinFee: false,
  },
  migration: {
    migrationOption: MigrationOption.MET_DAMM_V2,
    migrationFeeOption: MigrationFeeOption.Customizable,
    migrationFee: {
      feePercentage: 10,
      creatorFeePercentage: 50,
    },
    migratedPoolFee: {
      collectFeeMode: MigratedCollectFeeMode.QuoteToken,
      dynamicFee: DammV2DynamicFeeMode.Enabled,
      poolFeeBps: 30, // 0.3%
      baseFeeMode: DammV2BaseFeeMode.FeeTimeSchedulerLinear,
    },
  },
  liquidityDistribution: {
    // 100% 永久锁定 — 详见阶段三
    partnerLiquidityPercentage: 0,
    partnerPermanentLockedLiquidityPercentage: 50,
    creatorLiquidityPercentage: 0,
    creatorPermanentLockedLiquidityPercentage: 50,
  },
  lockedVesting: {
    totalLockedVestingAmount: 0,
    numberOfVestingPeriod: 0,
    cliffUnlockAmount: 0,
    totalVestingDuration: 0,
    cliffDurationFromMigrationTime: 0,
  },
  activationType: ActivationType.Timestamp,
  sqrtPrices,
  liquidityWeights,
});

const client = DynamicBondingCurveClient.create(connection, "confirmed");
const config = await client.partner.createConfig({
  payer: wallet.publicKey,
  partner: partnerKeypair,
  params: curveConfig,
});
```

### 曲线约束

| 约束 | 要求 |
|------|------|
| 分段数 | **1–16 段** |
| 价格 | 必须严格递增 |
| 流动性 | 每个分段必须为正 |
| 迁移阈值 | 必须可达 |
| 起始平方根价格 | **≥ 4,295,048,016** |

### DBC 参数汇总

| 参数 | 数值 |
|------|------|
| 供应模式 | **Dynamic Supply** |
| Quote Mint | USDC（6 位小数） |
| Base Mint | 项目代币（9 位小数） |
| 激活类型 | Timestamp |
| 迁移目标 | **DAMM v2** |
| 迁移阈值 | **100,000 USDC** |
| 池费率 | **0.3%**（30 bps） |
| 迁移费用 | **10%** |
| Creator 费用分成 | **50%** |

---

## 阶段三：DAMM v2（流动性迁移）

### 迁移流程

当 DBC 的 `quote_reserve` 达到 `migration_quote_threshold` 时，曲线停止交易，进入迁移流程。

**状态机**：

```
PreBondingCurve → PostBondingCurve → LockedVesting → CreatedPool
   （交易中）        （完成）          （锁仓设置）    （DAMM v2 上线）
```

**迁移步骤**：
1. 创建 DAMM v2 池
2. 创建迁移流动性的 Position NFTs
3. 应用配置的永久锁定或 Vesting 锁定

### 流动性分配：100% 永久锁定

**DBC 要求**：分配总和必须 **= 100%**，至少 **10%** 在第一天保持锁定。

**本方案：100% 永久锁定**

| 分配项 | 比例 | 说明 |
|--------|------|------|
| Partner 永久锁定 | **50%** | 协议方份额 — 永久锁仓 |
| Creator 永久锁定 | **50%** | 项目方份额 — 永久锁仓 |
| Partner 解锁 | **0%** | — |
| Partner Vesting | **0%** | — |
| Creator 解锁 | **0%** | — |
| Creator Vesting | **0%** | — |
| **总计** | **100%** | 必须精确等于 100% |

> **永久锁定的含义**：流动性永远无法撤出。LP 头寸仍持续赚取交易手续费和挖矿奖励。

### DAMM v2 池配置

| 参数 | 数值 | 说明 |
|------|------|------|
| 池类型 | **集中流动性** | Position NFT 表示 LP 头寸 |
| 价格范围 | **$0.005–$0.02** | 围绕迁移价 $0.01 提供 ±100% 范围 |
| 初始价格 | **$0.01** | 由 DBC 迁移价格决定 |
| 池费率 | **0.3%**（30 bps） | 必须在 0.1%–10% 之间 |
| 动态费率 | **启用** | 根据市场波动自动调整 |
| Vesting 锁仓上限 | **2 年** | 最长锁仓期限 |

### 流动性分配代码

```typescript
const liquidityDistribution = {
  partnerLiquidityPercentage: 0,
  partnerPermanentLockedLiquidityPercentage: 50,
  partnerVestingLiquidityPercentage: 0,
  creatorLiquidityPercentage: 0,
  creatorPermanentLockedLiquidityPercentage: 50,
  creatorVestingLiquidityPercentage: 0,
  // 总和 = 100%
};
```

---

## 完整价格路径

### 各阶段价格

| 阶段 | 累积 Quote | 价格 | 事件 |
|------|-----------|------|------|
| Alpha Vault | — | **$0.0015** | Vault 用户入场 |
| DBC 启动 | 0 | **$0.001** | 曲线最低价 |
| 爆发段结束 | 15,000 USDC | **$0.003** | 3 倍涨幅 |
| 扩张段结束 | 45,000 USDC | **$0.006** | 再涨 2 倍 |
| **迁移触发** | **100,000 USDC** | **$0.01** | 毕业到 DAMM v2 |
| DAMM v2 开盘 | — | **$0.01** | 初始交易价格 |

### Vault 用户收益

| 指标 | 数值 |
|------|------|
| Vault 买入价 | **$0.0015** |
| 迁移价 | **$0.01** |
| 账面浮盈 | **~567%** |
| 解锁开始 | TGE + **7 天** |
| 完全解锁 | TGE + **30 天** |

---

## 风险管理

### 风险清单

| 风险 | 影响 | 对冲方案 |
|------|------|---------|
| 公开市场买盘不足 | 无法达到迁移阈值 | 准备 **20,000–30,000 USDC** 引导资金 |
| 狙击机器人抢跑 | 破坏公平性 | 使用 Anti-Sniper Suite（费率调度器、速率限制器） |
| Vault 用户集中抛售 | 价格暴跌 | 24 天线性解锁 + DAMM v2 深度 |
| 曲线设计失控 | 价格剧烈波动 | **Devnet 完整测试** |
| 100% 永久锁定不可逆 | 失去流动性控制权 | 确保此为最终决策 |

### 测试计划

| 步骤 | 网络 | 内容 |
|------|------|------|
| 1 | **Localnet** | 使用 Meteora Invent 本地验证器 |
| 2 | **Devnet** | 完整跑通 Vault → DBC → 迁移全流程 |
| 3 | **Devnet** | 验证价格路径、Vesting |
| 4 | **Mainnet** | 确认无误后部署 |

---

## 部署检查清单

### 部署前

- [ ] 项目代币已创建（SPL 或 Token 2022）
- [ ] 代币小数位数 6–9
- [ ] Quote Mint 已确定（USDC/SOL）
- [ ] 激活类型已确认（Slot/Timestamp）
- [ ] Vault 参数已在 Devnet 验证
- [ ] DBC 三段式曲线已在 Devnet 验证
- [ ] 迁移流程已在 Devnet 验证
- [ ] 100% 永久锁定已确认
- [ ] 所有参数已向社区公示

### 部署中

- [ ] 创建 DBC Config
- [ ] 创建 DBC 虚拟池
- [ ] 初始化 Alpha Vault（关联 DBC 池）
- [ ] 开启存款窗口
- [ ] 执行 Fill（10–20 笔）
- [ ] 监控 DBC 公开交易
- [ ] 达到迁移阈值 → 自动迁移到 DAMM v2

### 部署后

- [ ] 确认 DAMM v2 池已创建
- [ ] 确认 100% 流动性已永久锁定
- [ ] 确认 Position NFTs 已创建
- [ ] 监控 Vault 用户 Vesting 解锁（TGE+7天起）
- [ ] 监控 DAMM v2 交易深度和价格稳定性

---

## 总结

| 阶段 | 组件 | 核心参数 | 用户体验 |
|------|------|---------|---------|
| **公平启动** | Alpha Vault Pro Rata | 20K USDC cap，$0.0015 入场 | 567% 浮盈预期 |
| **价格发现** | DBC 三段式曲线 | 100K USDC 迁移阈值 | 爆发 → 扩张 → 稳定 |
| **长期稳定** | DAMM v2 | 100% 永久锁定 | 深度充足，可持续 |

---

## 参考链接

- [Meteora 官方文档](https://docs.meteora.ag)
- [Alpha Vault 文档](https://docs.meteora.ag/helper-products/alpha-vault)
- [DBC 文档](https://docs.meteora.ag/core-products/dbc)
- [Meteora Invent（启动工具包）](https://github.com/MeteoraAg/meteora-invent)
- [DBC TypeScript SDK](https://www.npmjs.com/package/@meteora-ag/dynamic-bonding-curve-sdk)

---

*Built with ❤️ on Solana*
本方案通过三个阶段实现“**前期爆发性 + 后期稳定性**”的代币启动：

| 阶段 | 组件 | 核心目标 |
|------|------|---------|
| **阶段一** | Alpha Vault (Pro Rata) | 公平启动，制造早期财富效应 |
| **阶段二** | DBC 三段式曲线 | 价格发现：爆发→扩张→稳定 |
| **阶段三** | DAMM v2 | 长期稳定流动性，100% 永久锁定 |

### 1.2 核心数据

| 参数 | 数值 |
|------|------|
| 项目总供应量 | 1,000,000,000 枚 |
| DBC 供应模式 | **Dynamic Supply**（代币按需铸造） |
| 预留比例 | 3%（30,000,000 枚，作为未铸造储备） |

---

## 二、阶段一：Alpha Vault（Pro Rata）

### 2.1 什么是 Alpha Vault

Alpha Vault 是 Meteora 的预启动分配机制，在公开交易前收集 Quote Token 存款，并在受保护的预激活窗口内从连接的 Launch Pool 买入代币，防止狙击机器人抢跑。

**Pro Rata 模式**：当需求可能超过 Vault 购买力时使用，总存款可以超过 `max_buying_cap`，最终按比例分配。

### 2.2 参数配置

| 参数 | 建议值 | 说明 |
|------|--------|------|
| **Vault 模式** | **Pro Rata** | 支持超额认购，按比例分配 |
| **`max_buying_cap`** | **20,000 USDC** | Vault 最大可 swap 额度 |
| **Vault 执行价** | **$0.0015** | Vault 买入时的曲线价格 |
| **单钱包软上限** | **200~400 USDC** | 通过前端施加，强制分散持仓 |
| **`depositing_point`** | 公告后 48~72 小时 | 存款窗口开启 |
| **`last_join_point`** | `depositing_point` + 96 小时 | 存款截止 |
| **`pre_activation_start_point`** | `last_join_point` + **1 分钟** | Vault 开始 Fill |
| **`last_buying_point`** | `pre_activation_start_point` + **2 小时** | Vault Fill 截止 |
| **`start_vesting_point`** | **TGE + 7 天** | 开始线性解锁 |
| **`end_vesting_point`** | **TGE + 30 天** | 完全解锁 |
| **`escrow_fee`** | **0.3%** | 托管费用 |

### 2.3 初始化指令

使用 `initialize_prorata_vault` 指令创建 Pro Rata Vault：

```typescript
import { AlphaVault } from "@meteora-ag/alpha-vault-sdk";

const params = {
  pool_type: PoolType.DAMM_V2,
  quote_mint: USDC_MINT,
  base_mint: YOUR_TOKEN_MINT,
  depositing_point: new BN(depositStartTimestamp),
  start_vesting_point: new BN(tgeTimestamp + 7 * 24 * 60 * 60),
  end_vesting_point: new BN(tgeTimestamp + 30 * 24 * 60 * 60),
  max_buying_cap: new BN(20_000 * 1_000_000), // 20,000 USDC (6 decimals)
  escrow_fee: new BN(30), // 0.3% (basis points)
  whitelist_mode: WhitelistMode.Permissionless,
};

const tx = await AlphaVault.createCustomizableProrataVault(
  connection,
  params,
  ownerWallet
);
```

> **时间格式**：支持 Slot-based（3,000 slots）或 Timestamp-based（1,200 秒）两种激活类型。

### 2.4 时间线图解

```
T-4天          T           T+1分钟      T+2小时       T+7天         T+30天
  │              │             │            │             │             │
depositing   last_join   pre_activat  last_buying  start_vest   end_vest
  │              │             │            │             │             │
  ├── 存款窗口 ──┤             ├── Fill ────┤             ├── 解锁期 ──┤
  │              │             │            │             │             │
  └─ 用户可存入/撤出 ─┘    └─ Vault买入 ──┘    └─ 线性解锁（24天）─┘
```

### 2.5 Pro Rata 分配机制

**核心公式**：

```
Vault 实际 swap 金额 = min(总存款, max_buying_cap)
用户分配比例 = 用户存款 / 总存款
用户获得代币 = Vault 实际 swap 金额 × 用户分配比例 / Vault 执行价
```

**示例**：
- `max_buying_cap` = 20,000 USDC
- 总存款 = 50,000 USDC（超募）
- Vault 只 swap 20,000 USDC
- 用户 A 存款 1,000 USDC → 获得代币 = 20,000 × (1,000/50,000) / 0.0015 = 266,667 枚
- 超出部分（30,000 USDC）按比例退还

### 2.6 单钱包上限设计

Pro Rata 模式下程序默认 `individual_depositing_cap` 为 `u64::MAX`（无上限）。**通过前端施加软上限**：

- 上限 200 USDC：至少需 **100 个地址** 填满 Vault
- 上限 400 USDC：至少需 **50 个地址** 填满 Vault

### 2.7 Vesting 解锁

Claims 从 `start_vesting_point` 到 `end_vesting_point` **线性解锁**，两端点均包含在内。

```
总解锁周期 = end_vesting_point - start_vesting_point + 1 = 24 天
可领取比例 = 已过时间 / 总解锁周期
```

### 2.8 Fill（买入）执行计划

Vault fills 是 **无需许可的**，任何人都可以在预激活窗口内触发。

| 配置项 | 建议值 |
|--------|--------|
| Cranker | **团队自运行** |
| 白名单 | **将 Cranker 地址加入白名单**（免交每次 0.0001 SOL 费用） |
| Fill 笔数 | **10~20 笔** |
| 单笔上限 | **1,000~2,000 USDC** |
| Fill 间隔 | 每 6~12 分钟一笔 |
| `minimum_amount_out` | **0**（Fill 指令默认值） |

---

## 三、阶段二：DBC（动态绑定曲线）

### 3.1 什么是 DBC

DBC（Dynamic Bonding Curve）是一个可自定义的绑定曲线启动协议，允许配置**最多 16 个价格-流动性分段**来控制曲线形状和价格行为。

**核心机制**：
- 每个分段遵循恒定乘积公式：`x × y = k`
- 虚拟流动性 `L` 控制价格移动速度：**L 越低，价格移动越快**；**L 越高，价格移动越慢**
- 迁移价格由曲线和迁移阈值**自动推导**，非手动设置

### 3.2 供应模式：Dynamic Supply

| 供应模式 | 行为 | Leftover |
|----------|------|----------|
| **Dynamic Supply** | 代币按需铸造 | 几乎没有 |
| Fixed Supply | 代币预先全部铸造 | 产生大量 Leftover |

**本方案选择 Dynamic Supply**，确保 97% 的代币按需铸造并自然流向用户和流动性池。

### 3.3 三段式曲线设计

| 分段 | 价格区间 | 流动性 (L) | 目标 Quote 累积 | 效果 |
|------|---------|-----------|----------------|------|
| **Segment 1（爆发段）** | $0.001 → $0.003 | **极低** | 0 → 15,000 USDC | 3 倍暴涨，制造 FOMO |
| **Segment 2（扩张段）** | $0.003 → $0.006 | **中等** | 15,000 → 45,000 USDC | 稳步上涨，吸引更多参与者 |
| **Segment 3（稳定段）** | $0.006 → $0.01 | **高** | 45,000 → 100,000 USDC | 平稳走向迁移 |

### 3.4 曲线公式

**单个分段**：

```
Base Amount = L × (1/√P_lower - 1/√P_upper)
Quote Amount = L × (√P_upper - √P_lower)
```

**总曲线**：

```
总 Base 数量 = Σ L_i × (1/P_{i-1} - 1/P_i)
迁移阈值 = Σ L_i × (P_i - P_{i-1})
```

**迁移价格计算**：

```
P_migration² = 迁移阈值 × (1 - 迁移费率) / 迁移数量
```

### 3.5 使用 SDK 构建三段式曲线

```typescript
import {
  DynamicBondingCurveClient,
  buildCurveWithCustomSqrtPrices,
  createSqrtPrices,
  TokenDecimal,
  ActivationType,
  MigrationOption,
  MigrationFeeOption,
  DammV2BaseFeeMode,
  DammV2DynamicFeeMode,
  BaseFeeMode,
  CollectFeeMode,
} from "@meteora-ag/dynamic-bonding-curve-sdk";

// 1. 定义三个价格检查点
const sqrtPrices = createSqrtPrices(
  [0.001, 0.003, 0.006, 0.01],  // 4 个价格点 = 3 个分段
  TokenDecimal.SIX,   // USDC 6 decimals
  TokenDecimal.NINE   // 项目代币 9 decimals
);

// 2. 定义各分段的流动性权重
// 分段1: 极低流动性 (权重1) → 爆发
// 分段2: 中等流动性 (权重3) → 扩张
// 分段3: 高流动性 (权重6) → 稳定
const liquidityWeights = [1, 3, 6];

// 3. 构建曲线配置
const curveConfig = buildCurveWithCustomSqrtPrices({
  token: {
    tokenType: TokenType.SPLToken,
    tokenBaseDecimal: TokenDecimal.SIX,
    tokenQuoteDecimal: TokenDecimal.NINE,
    tokenAuthorityOption: TokenAuthorityOption.PartnerUpdateAuthority,
    totalTokenSupply: 1_000_000_000,
    leftover: 0, // Dynamic Supply 下为 0
  },
  fee: {
    baseFeeParams: {
      baseFeeMode: BaseFeeMode.FeeSchedulerExponential,
      feeSchedulerParam: {
        startingFeeBps: 9000,
        endingFeeBps: 120,
        numberOfPeriod: 60,
        totalDuration: 60,
      },
    },
    dynamicFeeEnabled: true,
    collectFeeMode: CollectFeeMode.QuoteToken,
    creatorTradingFeePercentage: 0,
    poolCreationFee: 1,
    enableFirstSwapWithMinFee: false,
  },
  migration: {
    migrationOption: MigrationOption.MET_DAMM_V2,
    migrationFeeOption: MigrationFeeOption.Customizable,
    migrationFee: {
      feePercentage: 10,
      creatorFeePercentage: 50,
    },
    migratedPoolFee: {
      collectFeeMode: MigratedCollectFeeMode.QuoteToken,
      dynamicFee: DammV2DynamicFeeMode.Enabled,
      poolFeeBps: 30, // 0.3%
      baseFeeMode: DammV2BaseFeeMode.FeeTimeSchedulerLinear,
    },
  },
  liquidityDistribution: {
    // 100% 永久锁定 (详见第四部分)
    partnerLiquidityPercentage: 0,
    partnerPermanentLockedLiquidityPercentage: 50,
    creatorLiquidityPercentage: 0,
    creatorPermanentLockedLiquidityPercentage: 50,
  },
  lockedVesting: {
    totalLockedVestingAmount: 0,
    numberOfVestingPeriod: 0,
    cliffUnlockAmount: 0,
    totalVestingDuration: 0,
    cliffDurationFromMigrationTime: 0,
  },
  activationType: ActivationType.Timestamp,
  sqrtPrices,
  liquidityWeights,
});

// 4. 创建 Config 和 Pool
const client = DynamicBondingCurveClient.create(connection, "confirmed");
const config = await client.partner.createConfig({
  payer: wallet.publicKey,
  partner: partnerKeypair,
  params: curveConfig,
});
```

### 3.6 曲线约束

| 约束 | 要求 |
|------|------|
| 曲线分段数 | **1 ~ 16 段** |
| 价格必须递增 | 每个价格点必须高于前一个 |
| 流动性必须为正 | 每个分段都需要流动性 |
| 迁移阈值必须可达 | 曲线需有足够流动性达到阈值 |
| 起始平方根价格下限 | **≥ 4,295,048,016** |

### 3.7 DBC 完整参数清单

| 参数 | 数值 |
|------|------|
| 供应模式 | **Dynamic Supply** |
| Quote Mint | USDC（6 decimals） |
| Base Mint | 你的项目代币（9 decimals） |
| 激活类型 | Timestamp |
| 迁移目标 | **DAMM v2** |
| 迁移阈值 | **100,000 USDC** |
| 迁移池费率 | **0.3%**（30 bps） |
| 迁移费用百分比 | **10%** |
| Creator 迁移费比例 | **50%** |

---

## 四、阶段三：DAMM v2（流动性迁移）

### 4.1 迁移流程

当 DBC 曲线的 `quote_reserve` 达到 `migration_quote_threshold` 时，曲线停止交易，进入迁移流程。

**迁移状态机**：

```
PreBondingCurve → PostBondingCurve → LockedVesting → CreatedPool
     (曲线交易中)    (曲线完成)      (锁仓设置)    (DAMM池已创建)
```

**迁移步骤**：
1. 创建 DAMM v2 池
2. 创建迁移的流动性 Position NFTs
3. 应用配置的永久锁定或 Vesting 锁定

### 4.2 流动性分配（100% 永久锁定）

**DBC 要求**：迁移的流动性分配总和必须 **= 100%**，且至少 **10%** 的流动性在迁移后第一天保持锁定。

**本方案：100% 永久锁定**

| 分配项 | 比例 | 说明 |
|--------|------|------|
| Partner 永久锁定流动性 | **50%** | 协议方份额全部永久锁仓 |
| Creator 永久锁定流动性 | **50%** | 项目方份额全部永久锁仓 |
| Partner 解锁流动性 | **0%** | — |
| Partner Vesting 流动性 | **0%** | — |
| Creator 解锁流动性 | **0%** | — |
| Creator Vesting 流动性 | **0%** | — |
| **总计** | **100%** | 必须精确等于 100% |

> **永久锁定的含义**：流动性永远无法撤出，但仍持续赚取交易手续费和流动性挖矿奖励。

### 4.3 DAMM v2 池配置

| 参数 | 数值 | 说明 |
|------|------|------|
| 池类型 | **集中流动性** | Position NFT 表示 LP 头寸 |
| 价格范围 | **$0.005 ~ $0.02** | 围绕迁移价 $0.01 提供 ±100% 范围 |
| 初始价格 | **$0.01** | 由 DBC 迁移价格自动决定 |
| 池费率 | **0.3%**（30 bps） | 必须在 0.1%~10% 之间 |
| 动态费率 | **启用** | 可根据市场波动自动调整 |
| Vesting 锁仓上限 | **2 年** |

### 4.4 迁移后流动性分配代码

```typescript
// 在 DBC 配置中设置 liquidityDistribution
const liquidityDistribution = {
  // 100% 永久锁定
  partnerLiquidityPercentage: 0,
  partnerPermanentLockedLiquidityPercentage: 50,
  partnerVestingLiquidityPercentage: 0,
  creatorLiquidityPercentage: 0,
  creatorPermanentLockedLiquidityPercentage: 50,
  creatorVestingLiquidityPercentage: 0,
  // 总和 = 100%
};
```

---

## 五、完整价格路径

### 5.1 各阶段价格一览

| 阶段 | 累积 Quote | 价格 | 事件 |
|------|-----------|------|------|
| Alpha Vault | — | **$0.0015** | Vault 用户买入 |
| DBC 启动 | 0 | **$0.001** | 曲线最低价 |
| 爆发段结束 | 15,000 USDC | **$0.003** | 3 倍涨幅 |
| 扩张段结束 | 45,000 USDC | **$0.006** | 再涨 2 倍 |
| **迁移触发** | **100,000 USDC** | **$0.01** | 毕业到 DAMM v2 |
| DAMM v2 开盘 | — | **$0.01** | 初始交易价格 |

### 5.2 Vault 用户收益

| 指标 | 数值 |
|------|------|
| Vault 买入价 | **$0.0015** |
| 迁移价 | **$0.01** |
| 账面浮盈 | **~567%** |
| 解锁开始 | TGE + **7 天** |
| 完全解锁 | TGE + **30 天** |

---

## 六、风险与对冲

### 6.1 风险清单

| 风险 | 影响 | 对冲方案 |
|------|------|---------|
| 公开市场买盘不足 | 无法达到迁移阈值 | 准备 **20,000~30,000 USDC** 引导资金 |
| 狙击机器人抢跑 | 破坏公平性 | 使用 Anti-Sniper Suite（费率调度器、速率限制器） |
| Vault 用户集中抛售 | 价格暴跌 | 24 天线性解锁平滑抛压 + DAMM v2 深度 |
| 曲线设计失控 | 价格剧烈波动 | **Devnet 完整测试** |
| 100% 永久锁定不可逆 | 失去流动性控制权 | 确保此为最终决策 |

### 6.2 测试计划

| 步骤 | 网络 | 内容 |
|------|------|------|
| 1 | **Localnet** | 使用 Meteora Invent 启动本地验证器测试 |
| 2 | **Devnet** | 完整跑通 Vault + DBC + 迁移全流程 |
| 3 | **Devnet** | 验证价格路径、Leftover、Vesting |
| 4 | **Mainnet** | 确认无误后部署 |

---

## 七、部署检查清单

### 7.1 部署前

- [ ] 项目代币已铸造（SPL Token 或 Token 2022）
- [ ] 代币 decimals 在 6~9 之间
- [ ] 确定 Quote Mint（USDC/SOL）
- [ ] 确定激活类型（Slot / Timestamp）
- [ ] Vault 参数已在 Devnet 验证
- [ ] DBC 三段式曲线已在 Devnet 验证
- [ ] 迁移流程已在 Devnet 验证
- [ ] 100% 永久锁定已确认
- [ ] 所有参数已向社区公示

### 7.2 部署中

- [ ] 创建 DBC Config
- [ ] 创建 DBC 虚拟池
- [ ] 初始化 Alpha Vault（关联 DBC 池）
- [ ] 开启存款窗口
- [ ] 执行 Fill（10~20 笔）
- [ ] 监控 DBC 公开交易
- [ ] 达到迁移阈值 → 自动迁移到 DAMM v2

### 7.3 部署后

- [ ] 确认 DAMM v2 池已创建
- [ ] 确认 100% 流动性已永久锁定
- [ ] 确认 Position NFTs 已创建
- [ ] 监控 Vault 用户 Vesting 解锁（TGE+7天起）
- [ ] 监控 DAMM v2 交易深度和价格稳定性

---

## 八、总结

| 阶段 | 组件 | 核心参数 | 用户感受 |
|------|------|---------|---------|
| **公平启动** | Alpha Vault Pro Rata | 20K USDC cap, $0.0015 执行价 | 567% 浮盈预期 |
| **价格发现** | DBC 三段式曲线 | 100K USDC 迁移阈值 | 爆发→扩张→稳定 |
| **长期稳定** | DAMM v2 | 100% 永久锁定 | 深度充足，可持续 |

---

## 九、参考链接

- [Meteora 官方文档](https://docs.meteora.ag)
- [Alpha Vault 文档](https://docs.meteora.ag/helper-products/alpha-vault)
- [DBC 文档](https://docs.meteora.ag/core-products/dbc)
- [Meteora Invent (启动工具包)](https://github.com/MeteoraAg/meteora-invent)
- [DBC TypeScript SDK](https://www.npmjs.com/package/@meteora-ag/dynamic-bonding-curve-sdk)
