# FILPAYSCOPE ANALYTICS SPECS

A comprehensive analytics layer providing protocol intelligence, financial analytics, user behavior insights, and risk analysis for the Filecoin Pay ecosystem.

---

## Table of Contents

1. [Summary](#1-summary)
2. [Positioning & Differentiation](#2-positioning--differentiation)
3. [Data Architecture](#3-data-architecture)
4. [KPIs & Formulas](#4-kpis--formulas)
5. [Dashboard Specifications](#5-dashboard-specifications)
6. [Cohort Analysis](#6-cohort-analysis)
7. [API Specification](#7-api-specification)
8. [Alerting System](#8-alerting-system)

---

## 1. Summary

**FilPayScope.xyz** is a decision-grade analytics platform for the Filecoin Pay protocol. While the official Filecoin Pay dashboard serves as an explorer answering "what exists right now?", FilPayScope answers the deeper questions: "How is the protocol used, by whom, and with what risks?"

**Key Value Propositions:**

- Real-time protocol health monitoring
- User behavior and retention analytics
- Operator performance tracking
- Risk exposure and early warning alerts
- Cohort-based growth analysis
- Public API for integrators

**Target Users:**

- Filecoin Pay protocol team
- Operators seeking performance insights
- DAOs and governance participants
- Researchers and analysts
- Integrators building on Filecoin Pay

---

## 2. Positioning & Differentiation

### Capability Comparison

| Capability                           | Filecoin Pay Dashboard | FilPayScope |
| ------------------------------------ | ---------------------- | ----------- |
| Explorer (rails, accounts)           | ✅                     | ✅          |
| Basic protocol stats                 | ✅                     | ✅          |
| Time-series analytics                | ❌                     | ✅          |
| Financial analytics (fees, net flow) | ⚠️ Limited             | ✅          |
| User behavior analysis               | ❌                     | ✅          |
| Cohort retention analysis            | ❌                     | ✅          |
| Operator performance metrics         | ⚠️ Surface-level       | ✅          |
| Risk & lockup exposure               | ❌                     | ✅          |
| Derived KPIs (churn, take rate)      | ❌                     | ✅          |
| Alerting & anomaly detection         | ❌                     | ✅          |
| Public API                           | ❌                     | ✅          |
| Data export (CSV/JSON)               | ❌                     | ✅          |

### Analogy

This is comparable to the difference between **Etherscan** (explorer) and **Dune/Token Terminal** (analytics).

---

## 3. Data Architecture

### Pipeline Overview (WIP)

```
┌─────────────────────────────────────────────────────────────────┐
│                     DATA ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │  Filecoin   │    │   Goldsky   │    │ ClickHouse  │         │
│  │  Network    │───▶│  Subgraph   │───▶│  (Analytics)│         │
│  │             │    │             │    │             │         │
│  └─────────────┘    └─────────────┘    └──────┬──────┘         │
│                            │                  │                 │
│                            │                  ▼                 │
│                            │           ┌─────────────┐         │
│                            │           │ Materialized│         │
│                            │           │   Views     │         │
│                            │           └──────┬──────┘         │
│                            │                  │                 │
│                            ▼                  ▼                 │
│                     ┌─────────────┐    ┌─────────────┐         │
│                     │  Supabase   │◀───│   Rollups   │         │
│                     │  (API/Auth) │    │ (Daily/Wkly)│         │
│                     └──────┬──────┘    └─────────────┘         │
│                            │                                    │
│                            ▼                                    │
│                     ┌─────────────┐                             │
│                     │  Frontend   │                             │
│                     │  Dashboard  │                             │
│                     └─────────────┘                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Data Granularity by Dashboard

| Dashboard            | Granularity    | Refresh Rate |
| -------------------- | -------------- | ------------ |
| Protocol Overview    | Daily          | Every 15 min |
| Cash Flow            | Hourly         | Real-time    |
| User Analytics       | Weekly cohorts | Daily        |
| Operator Performance | Daily          | Every 15 min |
| Risk Exposure        | Real-time      | Every 5 min  |
| Alerts               | Real-time      | Continuous   |

### Subgraph Entities Used

| Entity                      | Primary Use                    |
| --------------------------- | ------------------------------ |
| RailCreated                 | Rail creation, user onboarding |
| RailSettled                 | Volume, fees, settlements      |
| RailTerminated              | Churn, lifecycle analysis      |
| RailFinalized               | Completion metrics             |
| RailRateModified            | Configuration changes          |
| RailLockupModified          | Lockup parameter tracking      |
| RailOneTimePaymentProcessed | One-time payment analytics     |
| DepositRecorded             | Inflow tracking                |
| WithdrawRecorded            | Outflow tracking               |
| AccountLockupSettled        | Lockup risk analysis           |
| OperatorApprovalUpdated     | Permission monitoring          |

---

## 4. KPIs & Formulas

### Protocol Health KPIs

| KPI                       | Formula                                                                | Entity Source                     |
| ------------------------- | ---------------------------------------------------------------------- | --------------------------------- |
| Total Value Settled (TVS) | `sum(totalSettledAmount)`                                              | RailSettled                       |
| Total Rails Created       | `count(railCreateds)`                                                  | RailCreated                       |
| Active Rails              | `count(railCreateds) - count(railTerminateds) - count(railFinalizeds)` | Multiple                          |
| Protocol Take Rate        | `sum(networkFee) / sum(totalSettledAmount) * 100`                      | RailSettled                       |
| Operator Take Rate        | `sum(operatorCommission) / sum(totalSettledAmount) * 100`              | RailSettled                       |
| Net Protocol Flow         | `sum(deposits.amount) - sum(withdrawals.amount)`                       | DepositRecorded, WithdrawRecorded |

### User Behavior KPIs

| KPI                       | Formula                                                   | Entity Source             |
| ------------------------- | --------------------------------------------------------- | ------------------------- |
| Unique Payers             | `count(distinct payer)`                                   | RailCreated               |
| Unique Payees             | `count(distinct payee)`                                   | RailCreated               |
| Repeat Payer Ratio        | `count(payers with >1 rail) / count(unique payers) * 100` | RailCreated               |
| Avg Rails per Payer       | `count(rails) / count(unique payers)`                     | RailCreated               |
| Payer Concentration (HHI) | `sum((payerVolume / totalVolume)^2)`                      | RailSettled + RailCreated |
| Top 10 Payer Share        | `sum(top10PayerVolume) / totalVolume * 100`               | RailSettled + RailCreated |

### Rail Lifecycle KPIs

| KPI                      | Formula                                                      | Entity Source               |
| ------------------------ | ------------------------------------------------------------ | --------------------------- |
| Rail Survival Rate       | `1 - (count(terminated) / count(created)) * 100`             | RailCreated, RailTerminated |
| Avg Rail Lifespan        | `avg(terminatedTimestamp - createdTimestamp)`                | RailCreated, RailTerminated |
| Time to First Settlement | `avg(firstSettlementTimestamp - createdTimestamp)`           | RailCreated, RailSettled    |
| Settlement Velocity      | `count(settlements) / count(activeRails) / days`             | RailSettled                 |
| Termination Rate         | `count(terminated in period) / count(active at start) * 100` | RailTerminated              |

### Financial KPIs

| KPI                         | Formula                                        | Entity Source                            |
| --------------------------- | ---------------------------------------------- | ---------------------------------------- |
| Gross Settlement Volume     | `sum(totalSettledAmount)`                      | RailSettled                              |
| Net Payee Volume            | `sum(totalNetPayeeAmount)`                     | RailSettled                              |
| Total Network Fees          | `sum(networkFee)`                              | RailSettled                              |
| Total Operator Commissions  | `sum(operatorCommission)`                      | RailSettled                              |
| Avg Settlement Size         | `sum(totalSettledAmount) / count(settlements)` | RailSettled                              |
| One-time vs Streaming Ratio | `count(oneTimePayments) / count(settlements)`  | RailOneTimePaymentProcessed, RailSettled |

### Risk KPIs

| KPI                        | Formula                                     | Signal                |
| -------------------------- | ------------------------------------------- | --------------------- |
| Lockup Utilization         | `currentLockup / lockupAllowance * 100`     | >80% = warning        |
| Settlement Lag             | `now - lastSettlementTimestamp`             | Growing = risk        |
| Counterparty Concentration | `top5PayerVolume / totalVolume * 100`       | >50% = risk           |
| Operator Dependency        | `railsPerTopOperator / totalRails * 100`    | >30% = centralization |
| Dormant Rails              | `count(rails with no settlement > 30 days)` | High = concern        |

---

## 5. Dashboard Specifications

### 5.1 Protocol Overview & Health Dashboard

**Purpose:** High-level understanding of protocol adoption and growth.

**Metrics:**

- Total rails created (cumulative + time series)
- Active rails vs terminated rails
- Total value settled (TVS)
- One-time payments volume
- Protocol take rate
- Net protocol flow

**Entities Used:**

- RailCreated
- RailTerminated
- RailFinalized
- RailSettled
- RailOneTimePaymentProcessed

**Visualizations:**

- KPI tiles (TVS, active rails, unique users)
- Line chart: Rails created per day/week
- Area chart: Active vs terminated over time
- Bar chart: Daily settlement volume

---

### 5.2 Payments & Cash Flow Dashboard

**Purpose:** Track money movement through the protocol.

**Metrics:**

- Deposits per token (volume + count)
- Withdrawals per token (volume + count)
- Net protocol flow by token
- Gross vs net settlements
- Fee breakdown (operator vs network)

**Entities Used:**

- DepositRecorded
- WithdrawRecorded
- RailSettled
- RailOneTimePaymentProcessed

**Visualizations:**

- Stacked bar chart: Deposits vs withdrawals by token
- Pie chart: Fee breakdown
- Line chart: Daily net flow
- Sankey diagram: Token flow visualization

---

### 5.3 Payer / Payee Analytics Dashboard

**Purpose:** Understand user behavior and concentration.

**Payer Metrics:**

- Top payers by total settled amount
- Number of rails created per payer
- Average rail lifetime per payer
- Payer retention rate

**Payee Metrics:**

- Top payees by net received amount
- Revenue concentration (Top 10 share)
- Streaming vs one-time income ratio
- Payee growth over time

**Entities Used:**

- RailCreated
- RailSettled
- RailOneTimePaymentProcessed

**Visualizations:**

- Leaderboard tables (top payers, top payees)
- Pie chart: Concentration analysis
- Line chart: Unique payers/payees over time

---

### 5.4 Operator & Validator Performance Dashboard

**Purpose:** Measure intermediary performance and incentives.

**Operator Metrics:**

- Total commission earned
- Number of rails operated
- Average commission rate (bps)
- Volume processed
- Operator market share

**Validator Metrics:**

- Rails validated
- Settlement frequency
- Validation success rate

**Entities Used:**

- RailCreated
- RailSettled
- OperatorApprovalUpdated

**Visualizations:**

- Operator leaderboard table
- Bar chart: Commission earned by operator
- Pie chart: Market share distribution
- Line chart: Operator growth over time

---

### 5.5 Rail Lifecycle & Configuration Dashboard

**Purpose:** Track how payment rails evolve over time.

**Lifecycle Metrics:**

- Time-to-first-settlement distribution
- Settlement frequency histogram
- Rail duration analysis
- Termination vs finalization rate
- Termination reasons (by address)

**Configuration Changes:**

- Rate changes (old → new) timeline
- Lockup parameter changes
- Modification frequency

**Entities Used:**

- RailCreated
- RailRateModified
- RailLockupModified
- RailSettled
- RailTerminated
- RailFinalized

**Visualizations:**

- Histogram: Rail lifespan distribution
- Timeline: Configuration changes per rail
- Funnel: Created → Active → Terminated/Finalized

---

### 5.6 Lockups & Risk Exposure Dashboard

**Purpose:** Understand capital constraints and protocol risk.

**Lockup Metrics:**

- Current lockup per account
- Lockup rate trends
- Lockup utilization vs allowance
- Accounts near max lockup period

**Risk Signals:**

- High lockup + low settlement accounts
- Repeated rate reductions (distress signal)
- High termination probability indicators
- Settlement lag warnings

**Risk Scoring:**

| Risk Level | Criteria                                              |
| ---------- | ----------------------------------------------------- |
| 🟢 Low     | Lockup <50%, regular settlements, stable rate         |
| 🟡 Medium  | Lockup 50-80%, settlement lag <14 days                |
| 🔴 High    | Lockup >80%, settlement lag >30 days, rate reductions |

**Entities Used:**

- AccountLockupSettled
- OperatorApprovalUpdated
- RailRateModified
- RailSettled

**Visualizations:**

- Heat map: Lockup utilization by account
- Gauge charts: Risk indicators
- Table: High-risk accounts list

---

### 5.7 Permissions & Compliance Dashboard

**Purpose:** Monitor operator permissions and limits.

**Metrics:**

- Approved vs revoked operators
- Rate allowance usage
- Lockup allowance usage
- Clients per operator
- Permission change history

**Entities Used:**

- OperatorApprovalUpdated

**Visualizations:**

- Table: Current operator approvals
- Timeline: Permission changes
- Bar chart: Operators by client count

---

## 6. Cohort Analysis

Cohort analysis groups users by a shared characteristic (typically their first action date) and tracks their behavior over time. This reveals retention, growth quality, and user lifecycle patterns.

### 6.1 Payer Cohorts

**Definition:** Group payers by the month they created their first rail.

**Track Over Time:**

- Retention rate (% still active each month)
- Rails created per cohort
- Volume settled per cohort
- Avg rails per payer

**Why It Matters:**

- Shows if user acquisition quality is improving
- Identifies retention issues early

---

### 6.2 Payee Cohorts

**Definition:** Group payees by the month they received their first payment.

**Track Over Time:**

- Revenue per cohort (cumulative)
- Active payees per month
- Avg revenue per payee
- Revenue growth rate

**Why It Matters:**

- Shows if payees grow their revenue over time
- Identifies most valuable payee segments
- Proves protocol delivers ongoing value

---

### 6.3 Volume Cohorts

**Definition:** Group all users by their first interaction month, track total volume contribution.

**Track Over Time:**

- Cumulative volume per cohort
- % of total volume by cohort
- Volume per user per cohort

**Why It Matters:**

- Shows if early users are power users
- Identifies volume concentration risks
- Tracks new user contribution over time

---

### 6.4 Operator Cohorts

**Definition:** Group operators by the month they first operated a rail.

**Track Over Time:**

- Rails managed per cohort
- Commission earned per cohort
- Volume processed per cohort
- Operator retention rate

**Why It Matters:**

- Identifies token adoption trends
- Shows protocol versatility
- Informs token integration priorities

---

### 6.6 Depositor Cohorts

**Definition:** Group users by their first deposit date.

**Track Over Time:**

- Conversion to payer (% who create a rail)
- Time to first rail creation
- Total deposits per cohort
- Deposit frequency

**Why It Matters:**

- Shows conversion funnel health
- Identifies onboarding friction
- Tracks user activation rate

---

### 6.7 Rail Size Cohorts

**Definition:** Group rails by initial payment rate tier.

**Tiers:**

- Small: <$100/month rate
- Medium: $100-$1,000/month rate
- Large: >$1,000/month rate

**Track Over Time:**

- Survival rate by tier
- Settlement frequency by tier
- Avg lifespan by tier
- Termination reasons by tier

**Why It Matters:**

- Shows if larger rails are more stable
- Identifies at-risk segments
- Informs pricing/incentive strategies

---

## 7. API Specification

WIP

---

## 8. Alerting System

### Protocol Health Alerts

| Alert             | Trigger                                   | Severity    |
| ----------------- | ----------------------------------------- | ----------- |
| Volume Drop       | Settlement volume drops >50% day-over-day | 🔴 Critical |
| Termination Spike | Terminations >2 std dev above average     | 🟡 Warning  |
| Whale Movement    | Single deposit/withdrawal >$100K          | 🟡 Info     |
| Indexing Error    | Subgraph reports indexing errors          | 🔴 Critical |

### Risk Alerts

| Alert                  | Trigger                           | Severity   |
| ---------------------- | --------------------------------- | ---------- |
| High Lockup            | Account lockup utilization >80%   | 🟡 Warning |
| Dormant Rail           | No settlement for >30 days        | 🟡 Warning |
| Rate Reduction Pattern | 3+ rate reductions in 30 days     | 🟡 Warning |
| Operator Concentration | Single operator >40% market share | 🟡 Warning |

### User Alerts (Subscribed)

| Alert               | Trigger                              |
| ------------------- | ------------------------------------ |
| Rail Created        | New rail with user as payer/payee    |
| Settlement Received | Settlement processed on user's rails |
| Rail Terminated     | User's rail terminated               |
| Approval Changed    | Operator approval status changed     |

### Alert Channels

- Webhook integrations
- API polling

---
