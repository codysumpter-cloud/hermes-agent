---
name: governed-market-intelligence-loop
description: Use when running or improving a market intelligence loop that observes stocks, macro, filings, social sentiment, analyst/fundamental changes, congressional disclosures, and account risk state to produce evidence-backed proposals without executing trades.
version: 1.0.0
author: Prismtek / Hermes Skill Bridge
license: MIT
platforms: [macos, linux, windows, vps]
metadata:
  hermes:
    tags: [markets, stocks, risk, intelligence-loop, proposal-only, model-agnostic, provider-agnostic, cron, tradingagents, investskill]
    related_skills: [market-sports-trend-intelligence, polymarket-trading-advisor, sportsbook-betting-advisor]
  buddy:
    risk_class: money
    auto_executable: false
    requires_explicit_approval: true
---
# Governed Market Intelligence Loop

## Overview

This skill turns a market cron idea into a governed intelligence system. It is designed for Hermes to monitor watchlists, positions, filings, congressional disclosures, social/X sentiment, analyst/fundamental changes, market price/volume behavior, and stored investment theses across any model/provider stack.

The skill is **proposal-only**. It must not place trades, modify orders, resize exposure, change stops, deposit, withdraw, or open broker/wallet/sportsbook actions unless a separate explicit user approval and broker-specific execution skill/policy exists.

The target architecture is:

```txt
Observe -> Validate -> Score -> Propose -> Require Approval -> Log
```

Not:

```txt
Signal -> Analyze -> Report -> Execute
```

## Core Principle

Cron is only a heartbeat. The application decides whether a run is valid.

Hermes must use:

- market-calendar guards;
- source validation;
- structured signal schemas;
- divergence scoring;
- deterministic risk policy;
- idempotency/locking;
- alert tiers;
- proposal output;
- audit logs.

## Model / Provider Agnostic Design

This skill can use any model or provider as long as the provider follows the same contracts.

Supported provider roles:

- `collector`: extracts raw signals from source adapters;
- `normalizer`: converts raw observations into structured signals;
- `bull_analyst`: argues the thesis-supporting case;
- `bear_analyst`: argues the thesis-breaking case;
- `risk_reviewer`: applies deterministic risk policy and flags violations;
- `synthesizer`: produces final proposal/digest;
- `auditor`: verifies evidence, schemas, and actions-not-taken.

Allowed provider families:

- local Ollama / LM Studio models;
- OpenAI-compatible APIs;
- Claude-compatible APIs;
- Gemini-compatible APIs;
- local specialist models;
- TradingAgents / InvestSkill style multi-agent debate;
- deterministic Python/TypeScript rules for risk and scoring.

Provider rules:

- Every model output must be coerced into schemas.
- A model cannot bypass risk policy.
- A model cannot authorize execution.
- A model cannot mark an unsupported claim high confidence.
- A model cannot convert social-only or delayed-disclosure-only signals into action.
- Deterministic policy has veto power over all model agents.

## Allowed Actions

Allowed:

- load canonical state;
- read account/position/order state through read-only adapters;
- read market data through approved adapters;
- read filings, disclosures, public news, and social/X sentiment through approved read-only adapters;
- normalize signals;
- score divergence;
- create watch/proposal/human-review alerts;
- create daily/weekly digests;
- create KnowledgeVault notes;
- create content ideas from public market narratives;
- create proposal packets requiring approval.

Not allowed inside this skill:

- placing market orders;
- placing limit orders;
- cancelling orders;
- modifying orders;
- changing stops;
- tightening stops automatically;
- increasing or decreasing exposure;
- rotating into a new ticker;
- opening broker/wallet pages for action;
- deposits/withdrawals/transfers;
- signing wallet transactions;
- presenting watchlists as direct financial recommendations.

## Required State Load

Every run must load canonical state before analysis:

```txt
- watchlist
- active holdings
- queued orders
- original thesis per ticker
- active risk policy
- previous run summary
- last alert sent per ticker
- previous divergence scores
- current exposure limits
```

If critical state cannot be loaded, fail closed.

Examples:

```txt
if thesis missing -> P1 alert, no proposal
if account state unavailable -> P1 alert, no proposal
if risk policy unavailable -> P0/P1 alert, no proposal
```

## Schedule Guidance

Do not use only `0 * * * *`.

Recommended schedule, Eastern Time:

```cron
CRON_TZ=America/New_York
15 8 * * 1-5  hermes premarket-scan --proposal-only
25 9 * * 1-5  hermes open-risk-check --require-human-approval
7,22,37,52 9-15 * * 1-5 hermes market-loop --proposal-only
5 16 * * 1-5 hermes close-review --digest
30 17 * * 1-5 hermes slow-signal-scan --proposal-only
15 20 * * 1-5 hermes calibrate-thresholds --no-live-actions
```

Inside the app:

```txt
if not trading day -> exit market closed
if market not open and job requires open market -> exit market not open
if early close -> adjust thresholds/schedule
if first five minutes after open -> require human approval
if last five minutes before close -> reduce social signal weight
```

## Source Treatment

Congressional disclosures are delayed context, not real-time insider activity.

Track:

```txt
transaction_date
filing_date
detected_at
```

Rule:

```txt
if source == congressional_disclosure:
  latency_class = delayed
  max_action = proposal_only
  execution_allowed = false
```

Social/X sentiment is a reflexivity/noise layer, not a sole decision source.

No single X post, analyst opinion, or delayed disclosure should trigger a pivot by itself.

## Signal Schema

Every signal must become structured data:

```txt
Signal
- ticker
- source: price | volume | congressional_disclosure | insider_filing | social | analyst | earnings | sec_filing | fundamental | account | macro
- direction: bullish | bearish | neutral
- confidence: 0.0-1.0
- severity: low | medium | high | critical
- freshness_minutes
- latency_class: real_time | near_real_time | delayed | stale | unknown
- evidence_url
- evidence_hash
- summary
- limitations
```

No evidence URL or evidence hash means no high-confidence alert.

## Divergence Scoring

A divergence report should be structured:

```txt
DivergenceScore
- ticker
- price_trend_score
- sentiment_score
- fundamental_score
- insider_score
- disclosure_score
- risk_score
- total
- action: silent | watch | proposal | human_review
```

Thresholds:

```txt
< 0.35 -> silent
0.35-0.54 -> watch
0.55-0.74 -> proposal
>= 0.75 -> human_review
```

High-confidence alert requires:

```txt
total >= 0.75
verified_source_count >= 2
has_primary_source_evidence == true
```

## Risk Policy

Split into two loops:

### Market Intelligence Loop

Asks:

- What changed outside the account?
- Does it affect the stored thesis?
- What evidence supports that?

### Account Risk Loop

Asks:

- What is current exposure?
- Are queued orders/fills expected?
- Is drawdown within policy?
- Are guardrails intact?

Account risk rules must be deterministic.

Examples:

```txt
portfolio exposure > max -> P0
single ticker exposure > max single name -> P0
unexpected filled order -> P0
unrealized loss > max probe loss -> P1
cannot verify account state -> P1 and no proposal
```

## Alert Tiers

Use alert tiers to prevent noise.

```txt
P0 — security issue, broken guardrail, unexpected order, exposure breach
P1 — high-confidence thesis break, human review required
P2 — material divergence, proposal generated
P3 — watchlist note, include in digest
P4 — noise, log only
```

Stable runs should be logged only unless a digest is due.

## Proposal Format

When divergence is found, output:

```md
# Risk Proposal: {TICKER}

## Alert Tier
P1 / P2 / P3

## Current Position
- Shares:
- Market value:
- Portfolio exposure:
- Unrealized P&L:

## Trigger
What changed?

## Evidence
- Source 1:
- Source 2:
- Source 3:

## Signal Classification
- Source type:
- Freshness:
- Confidence:
- Latency class:
- Primary-source verified: yes/no

## Thesis Impact
- Original thesis:
- New contradiction:
- Severity:

## Recommended Action
One of:
- Hold
- Watch
- Reduce proposed exposure
- Tighten proposed risk limit
- Pause new buys
- Human review required

## What Hermes Is Not Doing
- No automatic trade placed
- No order changed
- No stop changed
- No exposure changed

## Approval Required
Yes / No
```

## Failure Policy

Degrade honestly.

Examples:

```txt
social scan failed + price works -> degraded mode, no sentiment confidence
account read failed -> P1, no proposal
thesis store failed -> P1, no proposal
primary sources unavailable -> downgrade confidence
```

Do not report `stable` if critical tools failed.

## Permissions

Least-privilege defaults:

```yaml
web:
  enabled: true
  mode: read_only
market_data:
  enabled: true
  mode: read_only
account_info:
  enabled: true
  mode: read_only
positions:
  enabled: true
  mode: read_only
orders:
  enabled: true
  mode: read_only
trading:
  enabled: false
terminal:
  enabled: false
file:
  mode: read_only
  allowed_paths:
    - KnowledgeVault/50 - Content/market-watchlists/
    - KnowledgeVault/50 - Content/market-sports-digests/
    - KnowledgeVault/99-System/Risk Policy/
    - KnowledgeVault/99-System/Cron Jobs/
knowledge_vault:
  mode: read_only_by_default
x_insights:
  enabled: true
  mode: read_only
```

## Output Requirements

Every run must include:

```txt
- run ID
- timestamp
- active provider/model roles used
- tickers checked
- data sources used
- degraded tools, if any
- signals detected
- divergence score per ticker
- alert tier
- proposal, if any
- approval requirement
- actions explicitly not taken
- audit log path
```

## Content Reuse

Market narratives may be reused for Prismtek content only when framed as education/risk/process, not as recommendations.

Good content angles:

- why agents need deterministic risk vetoes;
- why social sentiment is not enough;
- why delayed disclosures are context, not execution triggers;
- how to build model-agnostic financial reasoning safely;
- why proposal-first agents beat autonomous trading bots.

## Verification Checklist

Before declaring success:

- No execution route was used.
- Model/provider outputs were schema-normalized.
- Risk policy had veto authority.
- Congressional disclosures were tagged delayed.
- No high-confidence alert came from one source.
- Run lock/idempotency was used or recommended.
- Stable runs were logged, not spammed.
- Receipts/audit logs were written.
- Any proposal clearly states approval requirements.
