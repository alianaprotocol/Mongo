# 🚀 Mongo — Meteora Token Launch Configuration

> **Project**: Mongo  
> **Version**: v1.0  
> **Last Updated**: September 6, 2026  
> **Network**: ☐ Devnet ☐ Mainnet

---

## 📋 TL;DR — Executive Summary

| Metric | Value |
|--------|-------|
| **Total Supply** | 1,000,000,000 $MONGO |
| **Public Launch Price** | ~$0.01 |
| **Vault Entry Price** | **$0.0015** (6.7× Discount) |
| **Vault User Paper Gain** | **~567%** |
| **Unlock Schedule** | Day 7 → Day 30 (Linear) |
| **Liquidity Lock** | **100% Permanent** |
| **Migration Threshold** | 100,000 USDC |

**Three‑Phase Strategy**:
```
Alpha Vault (Fair Launch) → DBC 3‑Segment Curve (Price Discovery) → DAMM v2 (Permanent Liquidity)
```

**[⬇ Download PDF](#) | [🔗 View on GitHub](#)**

---

## 📑 Table of Contents

- [Executive Summary](#-executive-summary)
- [Phase I: Alpha Vault (Pro Rata)](#-phase-i-alpha-vault-pro-rata)
- [Phase II: DBC (Dynamic Bonding Curve)](#-phase-ii-dbc-dynamic-bonding-curve)
- [Phase III: DAMM v2 (Liquidity Migration)](#-phase-iii-damm-v2-liquidity-migration)
- [Price Path & Expected Returns](#-price-path--expected-returns)
- [Risk Management](#-risk-management)
- [Deployment Checklist](#-deployment-checklist)
- [References](#-references)

---

## 📊 Executive Summary

### The Big Picture

Mongo Launches via a **Three‑Phase Mechanism** Designed for **Explosive Early Momentum** and **Long‑Term Stability**:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  PHASE 1               PHASE 2                PHASE 3                       │
│  ┌──────────────┐     ┌──────────────┐       ┌──────────────┐             │
│  │   Alpha      │     │     DBC      │       │    DAMM      │             │
│  │   Vault      │ ──► │    Curve     │ ──►   │     v2       │             │
│  │              │     │              │       │              │             │
│  │  Fair        │     │  Price       │       │  100%        │             │
│  │  Launch      │     │  Discovery   │       │  Locked      │             │
│  └──────────────┘     └──────────────┘       └──────────────┘             │
│                                                                             │
│  Entry: $0.0015       3‑Segment             Permanent Liquidity            │
│  Unlock: D7→D30       Explosion→Stability   Never Withdrawable             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Core Metrics

| Metric | Value |
|--------|-------|
| **Total Supply** | 1,000,000,000 $MONGO |
| **DBC Supply Mode** | **Dynamic Supply** (Minted on Demand) |
| **Reserved Supply** | 3% (30,000,000 — Unminted Reserve) |
| **Vault Entry Price** | **$0.0015** |
| **Migration Price** | **~$0.01** |
| **Vault User Paper Gain** | **~567%** |
| **Unlock Schedule** | Day 7 → Day 30 (Linear, 24‑Day Period) |
| **Liquidity Lock** | **100% Permanent** |
| **Migration Threshold** | **100,000 USDC** |

---

## 🏦 Phase I: Alpha Vault (Pro Rata)

### Overview

Alpha Vault Is Meteora's **Pre‑Launch Allocation Mechanism**. It Collects Deposits Before Public Trading, Then Executes Buys During a **Protected Window** — Preventing Sniper Bots from Front‑Running.

**Pro Rata Mode**: Deposits May Exceed `max_buying_cap`; Allocations Are **Proportional**.

### Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Vault Mode** | **Pro Rata** | Proportional on Oversubscription |
| **`max_buying_cap`** | **20,000 USDC** | Maximum Swap Amount |
| **Vault Execution Price** | **$0.0015** | Entry Price for Early Supporters |
| **Per‑Wallet Soft Cap** | **200–400 USDC** | Frontend‑Enforced; Ensures Fair Distribution |
| **Deposit Window** | **4 Days** | `depositing_point` → `last_join_point` |
| **Fill Window** | **2 Hours** | 10–20 Batches |
| **`start_vesting_point`** | **TGE + 7 Days** | Unlock Begins |
| **`end_vesting_point`** | **TGE + 30 Days** | Fully Unlocked |
| **`escrow_fee`** | **0.3%** | Protocol Fee |

### Timeline

```
T-4d        T          T+1m       T+2h        T+7d        T+30d
 │           │            │          │           │           │
 ├─ Deposit ─┤            ├─ Fill ──┤           ├─ Unlock ──┤
 │           │            │          │           │           │
└─ Deposit / Withdraw ────┘          └─ Linear (24 Days) ──┘
```

### Pro Rata Allocation Formula

```
Vault Actual Swap = min(Total Deposits, max_buying_cap)
User Token Allocation = Vault Actual Swap × (User Deposit / Total Deposits) / $0.0015
```

**Example**:  
You Deposit 1,000 USDC into a 50,000 USDC Pool (20,000 USDC Cap).  
→ You Receive `20,000 × (1,000 / 50,000) / 0.0015 = 266,667 $MONGO`.  
→ Excess 30,000 USDC Is Refunded Proportionally.

### Fill Execution Plan

| Item | Recommendation |
|------|----------------|
| **Cranker** | Team‑Run |
| **Whitelist** | Add Cranker Address to Whitelist (Waives 0.0001 SOL per Fill) |
| **Batches** | 10–20 Transactions |
| **Per‑Batch Cap** | 1,000–2,000 USDC |
| **Interval** | Every 6–12 Minutes |
| **`minimum_amount_out`** | 0 (Default) |

---

## 📈 Phase II: DBC (Dynamic Bonding Curve)

### Overview

DBC Is a **Customizable Bonding Curve** Supporting **Up to 16 Price‑Liquidity Segments**. Virtual Liquidity (`L`) Controls Price Movement Speed — **Lower `L` = Faster Moves**.

### Supply Mode: Dynamic Supply

| Mode | Behavior | Leftover |
|------|----------|----------|
| **Dynamic Supply** ✅ | Minted on Demand | None |
| Fixed Supply | Pre‑Minted All | Large Leftover |

✅ **This Plan Uses Dynamic Supply** — 97% of Tokens Flow Naturally to Users and Liquidity Pools.

### 3‑Segment Curve Design

```
Price
$0.01  ────────────────────★ Migration (100K USDC)
       │                  ╱
$0.006 ──────────────╱───  Segment 3: High L (Stable)
       │            ╱
$0.003 ────────╱───  Segment 2: Medium L (Expansion)
       │      ╱
$0.001 ──╱───  Segment 1: Very Low L (EXPLOSION!)
       │
       0    15K   45K   100K  → Cumulative Quote (USDC)
```

| Segment | Price Range | Liquidity | Target Quote | Effect |
|---------|-------------|-----------|--------------|--------|
| **S1: Explosion** | $0.001 → $0.003 | **Very Low** | 0 → 15,000 USDC | **3× Surge** — Creates FOMO |
| **S2: Expansion** | $0.003 → $0.006 | **Medium** | 15K → 45K USDC | Steady Climb — Attracts Buyers |
| **S3: Stability** | $0.006 → $0.01 | **High** | 45K → 100K USDC | Smooth Approach to Migration |

### Core Formulas

**Single Segment**:
```
Base Amount = L × (1/√P_lower - 1/√P_upper)
Quote Amount = L × (√P_upper - √P_lower)
```

**Migration Price** (Automatically Derived):
```
P_migration² = Migration Threshold × (1 - Fee) / Migration Amount
```

### DBC Parameters

| Parameter | Value |
|-----------|-------|
| **Supply Mode** | **Dynamic Supply** |
| **Quote Mint** | USDC (6 Decimals) |
| **Base Mint** | $MONGO (9 Decimals) |
| **Activation Type** | Timestamp |
| **Migration Target** | **DAMM v2** |
| **Migration Threshold** | **100,000 USDC** |
| **Pool Fee** | **0.3%** (30 bps) |
| **Migration Fee** | **10%** |
| **Creator Fee Share** | **50%** |

---

## 🔒 Phase III: DAMM v2 (Liquidity Migration)

### Migration Flow

When `quote_reserve` Reaches `100,000 USDC`:

```
PreBondingCurve → PostBondingCurve → LockedVesting → CreatedPool
   (Trading)        (Complete)        (Lock Setup)   (DAMM v2 Live)
```

### Liquidity Distribution: 100% Permanent Lock

**DBC Requirement**: Total Distribution Must **= 100%**, with at Least **10%** Locked on Day One.

**This Plan: 100% Permanent Lock**

| Allocation | Percentage |
|------------|------------|
| **Partner Permanent Lock** | **50%** |
| **Creator Permanent Lock** | **50%** |
| Unlocked / Vesting | **0%** |
| **Total** | **100%** |

> **Permanent Lock** = Liquidity Can **Never** Be Withdrawn. LP Positions Still Earn Fees and Rewards.

### DAMM v2 Pool Configuration

| Parameter | Value |
|-----------|-------|
| **Pool Type** | **Concentrated Liquidity** |
| **Price Range** | **$0.005 – $0.02** |
| **Initial Price** | **$0.01** |
| **Pool Fee** | **0.3%** (30 bps) |
| **Dynamic Fee** | **Enabled** |
| **Vesting Lock Cap** | **2 Years** |

---

## 💰 Price Path & Expected Returns

### Price by Stage

| Stage | Cumulative Quote | Price | Event |
|-------|------------------|-------|-------|
| **Alpha Vault** | — | **$0.0015** | Vault Entry |
| **DBC Launch** | 0 | **$0.001** | Curve Opens |
| **Explosion End** | 15,000 USDC | **$0.003** | 3× Surge |
| **Expansion End** | 45,000 USDC | **$0.006** | 2× Additional |
| **Migration Trigger** | **100,000 USDC** | **$0.01** | Graduation to DAMM v2 |
| **DAMM v2 Open** | — | **$0.01** | Initial Trading Price |

### Vault User Returns

```
Entry Price:     $0.0015
Exit Price:      $0.01
Paper Gain:      ~567% (6.7×)
Unlock Schedule: Day 7 → Day 30 (Linear)
```

---

## ⚠️ Risk Management

### Risk Register

| Risk | Mitigation |
|------|------------|
| **Insufficient Public Buying** | Prepare **20,000–30,000 USDC** Bootstrap Capital |
| **Sniper Bots** | Anti‑Sniper Suite (Fee Scheduler, Rate Limiter) |
| **Vault User Concentrated Selling** | 24‑Day Linear Unlock + DAMM v2 Depth |
| **Curve Design Errors** | **Devnet Full Testing** |
| **100% Lock Irreversible** | Confirm as Final Decision |

### Testing Plan

| Step | Network | Scope |
|------|---------|-------|
| **1** | **Localnet** | Meteora Invent Local Validator |
| **2** | **Devnet** | Full Vault → DBC → Migration Flow |
| **3** | **Devnet** | Validate Price Path, Vesting |
| **4** | **Mainnet** | Deploy After Verification |

---

## ✅ Deployment Checklist

### Pre‑Launch

- [ ] Token Mint Created (SPL or Token 2022)
- [ ] Token Decimals: 6–9
- [ ] Quote Mint Selected (USDC)
- [ ] Activation Type Confirmed (Timestamp)
- [ ] Vault Parameters Validated on Devnet
- [ ] 3‑Segment DBC Curve Validated on Devnet
- [ ] Migration Flow Validated on Devnet
- [ ] 100% Permanent Lock Confirmed
- [ ] **All Parameters Disclosed to Community**

### Launch

- [ ] Create DBC Config
- [ ] Create DBC Virtual Pool
- [ ] Initialize Alpha Vault (Link to DBC Pool)
- [ ] Open Deposit Window
- [ ] Execute Fills (10–20 Batches)
- [ ] Monitor DBC Public Trading
- [ ] Migration Threshold Reached → Auto‑Migrate

### Post‑Launch

- [ ] Confirm DAMM v2 Pool Created
- [ ] Confirm 100% Liquidity Permanently Locked
- [ ] Confirm Position NFTs Created
- [ ] Monitor Vault Vesting Unlocks (from Day 7)
- [ ] Monitor DAMM v2 Trading Depth & Stability

---

## 📚 References

- [Meteora Documentation](https://docs.meteora.ag)
- [Alpha Vault Docs](https://docs.meteora.ag/helper-products/alpha-vault)
- [DBC Docs](https://docs.meteora.ag/core-products/dbc)
- [Meteora Invent (Launch Toolkit)](https://github.com/MeteoraAg/meteora-invent)
- [DBC TypeScript SDK](https://www.npmjs.com/package/@meteora-ag/dynamic-bonding-curve-sdk)

---

---

# 🚀 Mongo — Meteora 代币启动配置文档

> **项目名称**：Mongo  
> **版本**：v1.0  
> **最后更新**：2026年9月6日  
> **网络**：☐ Devnet ☐ Mainnet

---

## 📋 执行摘要

| 指标 | 数值 |
|------|------|
| **总供应量** | 1,000,000,000 $MONGO |
| **公开发行价** | ~$0.01 |
| **Vault 入场价** | **$0.0015**（6.7 倍折扣） |
| **Vault 用户浮盈** | **~567%** |
| **解锁时间** | 第 7 天 → 第 30 天（线性） |
| **流动性锁定** | **100% 永久** |
| **迁移阈值** | 100,000 USDC |

**三阶段策略**：
```
Alpha Vault（公平启动）→ DBC 三段式曲线（价格发现）→ DAMM v2（永久流动性）
```

---

## 📑 目录

- [执行摘要](#-执行摘要)
- [阶段一：Alpha Vault（Pro Rata）](#-阶段一alpha-vault-pro-rata)
- [阶段二：DBC（动态绑定曲线）](#-阶段二dbc-dynamic-bonding-curve)
- [阶段三：DAMM v2（流动性迁移）](#-阶段三damm-v2-流动性迁移)
- [价格路径与预期收益](#-价格路径与预期收益)
- [风险管理](#-风险管理)
- [部署检查清单](#-部署检查清单)
- [参考链接](#-参考链接)

---

## 📊 执行摘要

### 整体设计

Mongo 通过 **三阶段机制** 启动，兼顾 **前期爆发性** 和 **长期稳定性**：

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  阶段 1              阶段 2               阶段 3                            │
│  ┌──────────────┐     ┌──────────────┐       ┌──────────────┐             │
│  │   Alpha      │     │     DBC      │       │    DAMM      │             │
│  │   Vault      │ ──► │    曲线      │ ──►   │     v2       │             │
│  │              │     │              │       │              │             │
│  │  公平启动    │     │  价格发现    │       │  100%        │             │
│  │              │     │              │       │  永久锁定    │             │
│  └──────────────┘     └──────────────┘       └──────────────┘             │
│                                                                             │
│  入场价 $0.0015      三段式曲线             流动性永不撤出                  │
│  解锁 D7→D30        爆发→扩张→稳定                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 核心数据

| 指标 | 数值 |
|------|------|
| **总供应量** | 1,000,000,000 $MONGO |
| **DBC 供应模式** | **Dynamic Supply**（按需铸造） |
| **预留供应量** | 3%（3,000万枚，未铸造储备） |
| **Vault 入场价** | **$0.0015** |
| **迁移价格** | **~$0.01** |
| **Vault 用户浮盈** | **~567%** |
| **解锁时间** | 第 7 天 → 第 30 天（线性，共 24 天） |
| **流动性锁定** | **100% 永久** |
| **迁移阈值** | **100,000 USDC** |

---

## 🏦 阶段一：Alpha Vault（Pro Rata）

### 概述

Alpha Vault 是 Meteora 的 **预启动分配机制**。它在公开交易前收集存款，并在 **受保护窗口** 内执行买入——防止机器人抢跑。

**Pro Rata 模式**：存款可超过 `max_buying_cap`，按 **比例分配**。

### 参数

| 参数 | 数值 | 说明 |
|------|------|------|
| **Vault 模式** | **Pro Rata** | 超额认购时按比例分配 |
| **`max_buying_cap`** | **20,000 USDC** | Vault 最大 Swap 额度 |
| **Vault 执行价** | **$0.0015** | 早期支持者入场价 |
| **单钱包软上限** | **200–400 USDC** | 前端强制执行，确保公平分散 |
| **存款窗口** | **4 天** | `depositing_point` → `last_join_point` |
| **Fill 窗口** | **2 小时** | 分 10–20 笔执行 |
| **`start_vesting_point`** | **TGE + 7 天** | 解锁开始 |
| **`end_vesting_point`** | **TGE + 30 天** | 完全解锁 |
| **`escrow_fee`** | **0.3%** | 协议费用 |

### 时间线

```
T-4天       T          T+1分钟    T+2小时     T+7天        T+30天
 │           │            │          │           │           │
 ├─ 存款 ────┤            ├─ Fill ──┤           ├─ 解锁 ────┤
 │           │            │          │           │           │
└─ 存入/撤出 ─────────────┘          └─ 线性解锁（24天）───┘
```

### Pro Rata 分配公式

```
Vault 实际 Swap = min(总存款, max_buying_cap)
用户获得代币 = Vault 实际 Swap × (用户存款 / 总存款) / $0.0015
```

**示例**：  
你存入 1,000 USDC，总池 50,000 USDC（上限 20,000）。  
→ 获得 `20,000 × (1,000 / 50,000) / 0.0015 = 266,667 $MONGO`。  
→ 超额部分（30,000 USDC）按比例退还。

### Fill 执行计划

| 项目 | 建议 |
|------|------|
| **Cranker** | 团队自运行 |
| **白名单** | 将 Cranker 地址加入白名单（免交每次 0.0001 SOL 费用） |
| **批次数** | 10–20 笔 |
| **单笔上限** | 1,000–2,000 USDC |
| **间隔** | 每 6–12 分钟一笔 |
| **`minimum_amount_out`** | 0（默认值） |

---

## 📈 阶段二：DBC（动态绑定曲线）

### 概述

DBC 是 **可自定义的绑定曲线**，支持 **最多 16 个价格-流动性分段**。虚拟流动性 `L` 控制价格移动速度 — **`L` 越低，价格变动越快**。

### 供应模式：Dynamic Supply

| 模式 | 行为 | Leftover |
|------|------|----------|
| **Dynamic Supply** ✅ | 按需铸造 | 无 |
| Fixed Supply | 预先全部铸造 | 大量 Leftover |

✅ **本方案使用 Dynamic Supply** — 97% 代币自然流向用户和流动性池。

### 三段式曲线设计

```
价格
$0.01  ────────────────────★ 迁移点（100K USDC）
       │                  ╱
$0.006 ──────────────╱───  分段3：高流动性（稳定）
       │            ╱
$0.003 ────────╱───  分段2：中等流动性（扩张）
       │      ╱
$0.001 ──╱───  分段1：极低流动性（爆发！）
       │
       0    15K   45K   100K  → 累计交易额（USDC）
```

| 分段 | 价格区间 | 流动性 | 目标交易额 | 效果 |
|------|---------|--------|-----------|------|
| **S1：爆发** | $0.001 → $0.003 | **极低** | 0 → 15,000 USDC | **3 倍暴涨** — 制造 FOMO |
| **S2：扩张** | $0.003 → $0.006 | **中等** | 15K → 45K USDC | 稳步上涨 — 吸引买家 |
| **S3：稳定** | $0.006 → $0.01 | **高** | 45K → 100K USDC | 平稳走向迁移 |

### 核心公式

**单个分段**：
```
Base 数量 = L × (1/√P_lower - 1/√P_upper)
Quote 数量 = L × (√P_upper - √P_lower)
```

**迁移价格**（自动推导）：
```
P_migration² = 迁移阈值 × (1 - 费率) / 迁移数量
```

### DBC 参数汇总

| 参数 | 数值 |
|------|------|
| **供应模式** | **Dynamic Supply** |
| **Quote Mint** | USDC（6 位小数） |
| **Base Mint** | $MONGO（9 位小数） |
| **激活类型** | Timestamp |
| **迁移目标** | **DAMM v2** |
| **迁移阈值** | **100,000 USDC** |
| **池费率** | **0.3%**（30 bps） |
| **迁移费用** | **10%** |
| **Creator 费用分成** | **50%** |

---

## 🔒 阶段三：DAMM v2（流动性迁移）

### 迁移流程

当 `quote_reserve` 达到 **100,000 USDC** 时：

```
PreBondingCurve → PostBondingCurve → LockedVesting → CreatedPool
   （交易中）        （完成）          （锁仓设置）    （DAMM v2 上线）
```

### 流动性分配：100% 永久锁定

**DBC 要求**：分配总和必须 **= 100%**，至少 **10%** 在第一天保持锁定。

**本方案：100% 永久锁定**

| 分配项 | 比例 |
|--------|------|
| **Partner 永久锁定** | **50%** |
| **Creator 永久锁定** | **50%** |
| 解锁 / Vesting | **0%** |
| **总计** | **100%** |

> **永久锁定** = 流动性 **永远无法撤出**。LP 头寸仍持续赚取手续费和奖励。

### DAMM v2 池配置

| 参数 | 数值 |
|------|------|
| **池类型** | **集中流动性** |
| **价格范围** | **$0.005 – $0.02** |
| **初始价格** | **$0.01** |
| **池费率** | **0.3%**（30 bps） |
| **动态费率** | **启用** |
| **Vesting 锁仓上限** | **2 年** |

---

## 💰 价格路径与预期收益

### 各阶段价格

| 阶段 | 累计交易额 | 价格 | 事件 |
|------|-----------|------|------|
| **Alpha Vault** | — | **$0.0015** | Vault 入场 |
| **DBC 启动** | 0 | **$0.001** | 曲线开启 |
| **爆发段结束** | 15,000 USDC | **$0.003** | 3 倍涨幅 |
| **扩张段结束** | 45,000 USDC | **$0.006** | 再涨 2 倍 |
| **迁移触发** | **100,000 USDC** | **$0.01** | 毕业到 DAMM v2 |
| **DAMM v2 开盘** | — | **$0.01** | 初始交易价格 |

### Vault 用户收益

```
入场价：     $0.0015
退出价：     $0.01
浮盈：       ~567%（6.7 倍）
解锁时间：   第 7 天 → 第 30 天（线性）
```

---

## ⚠️ 风险管理

### 风险清单

| 风险 | 对冲方案 |
|------|---------|
| **公开市场买盘不足** | 准备 **20,000–30,000 USDC** 引导资金 |
| **狙击机器人** | Anti‑Sniper Suite（费率调度器、速率限制器） |
| **Vault 用户集中抛售** | 24 天线性解锁 + DAMM v2 深度 |
| **曲线设计错误** | **Devnet 完整测试** |
| **100% 锁定不可逆** | 确保此为最终决策 |

### 测试计划

| 步骤 | 网络 | 内容 |
|------|------|------|
| **1** | **Localnet** | 本地验证器测试 |
| **2** | **Devnet** | 完整跑通 Vault → DBC → 迁移流程 |
| **3** | **Devnet** | 验证价格路径、解锁 |
| **4** | **Mainnet** | 确认无误后部署 |

---

## ✅ 部署检查清单

### 部署前

- [ ] 项目代币已创建（SPL 或 Token 2022）
- [ ] 代币小数位数 6–9
- [ ] Quote Mint 已确定（USDC）
- [ ] 激活类型已确认（Timestamp）
- [ ] Vault 参数已在 Devnet 验证
- [ ] DBC 三段式曲线已在 Devnet 验证
- [ ] 迁移流程已在 Devnet 验证
- [ ] 100% 永久锁定已确认
- [ ] **所有参数已向社区公示**

### 部署中

- [ ] 创建 DBC Config
- [ ] 创建 DBC 虚拟池
- [ ] 初始化 Alpha Vault（关联 DBC 池）
- [ ] 开启存款窗口
- [ ] 执行 Fill（10–20 笔）
- [ ] 监控 DBC 公开交易
- [ ] 达到迁移阈值 → 自动迁移

### 部署后

- [ ] 确认 DAMM v2 池已创建
- [ ] 确认 100% 流动性已永久锁定
- [ ] 确认 Position NFTs 已创建
- [ ] 监控 Vault 用户解锁（第 7 天起）
- [ ] 监控 DAMM v2 交易深度与稳定性

---

## 📚 参考链接

- [Meteora 官方文档](https://docs.meteora.ag)
- [Alpha Vault 文档](https://docs.meteora.ag/helper-products/alpha-vault)
- [DBC 文档](https://docs.meteora.ag/core-products/dbc)
- [Meteora Invent（启动工具包）](https://github.com/MeteoraAg/meteora-invent)
- [DBC TypeScript SDK](https://www.npmjs.com/package/@meteora-ag/dynamic-bonding-curve-sdk)

---

*Built with ❤️ on Solana*
