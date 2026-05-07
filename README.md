# quant-research-skill

A Claude Code skill pack for quantitative strategy work. Two complementary slash commands cover the full research lifecycle: **generate** a strategy from scratch, then **audit** it with institutional-grade rigour before deployment.

---

## Skills

### `/quant-researcher` — Strategy Generator

Turns Claude into a rigorous quantitative strategy researcher. One slash command triggers a full four-phase research pipeline: theory → literature → backtest → report.

When you invoke `/quant-researcher <request>`, Claude will:

| Phase | Output |
|-------|--------|
| **1 — Theory** | Market hypothesis, mathematical signal definition, pseudo-code, risk factor analysis |
| **2 — Literature** | ≥2 verified academic citations (JF, JFE, RFS, JPM, NBER, SSRN) |
| **3 — Backtest** | Live data via `yfinance` / FinMind, ≥5-year window, 10 bps cost, no look-ahead bias; outputs CAGR, Sharpe, Max Drawdown, Calmar, volatility, win rate, equity curve chart |
| **4 — Report** | Full Traditional Chinese Markdown report saved to the working directory as `strategy_<NAME>_<YYYYMMDD>.md` |

---

### `/review-strategy` — Strategy Auditor

Turns Claude into a Jane Street-level strategy auditor. Feed it any strategy specification (Markdown format) and it runs a five-phase adversarial validation pipeline, exposing every logic loophole, statistical trap, and implementation flaw — with mathematical proof and executable Python for each finding. Output is a full Traditional Chinese audit report with a binary deployment verdict.

When you invoke `/review-strategy <strategy spec or file path>`, Claude will:

| Phase | Output |
|-------|--------|
| **1 — Deconstruction** | Formal logic chain `P₁ → S → E → α → OOS`, signal re-formalization in math notation, exhaustive hidden-assumption inventory |
| **2 — Loophole Identification** | 9 named checks, each with math formula + executable Python validator: data snooping (Bonferroni), look-ahead bias, survivorship bias, overfitting (BDR), signal decay (IC curve), capacity limits (Almgren impact model), cost sensitivity (break-even cost), tail risk (Cornish-Fisher VaR), regime dependency |
| **3 — Statistical Validation** | Block bootstrap Sharpe confidence interval, permutation test against zero-predictability null, walk-forward IS vs OOS analysis |
| **4 — Improvement Suggestions** | Per-loophole fix with formula + pseudocode + expected performance gain, ranked P0 → P3 priority matrix |
| **5 — Audit Report** | Full Traditional Chinese Markdown report → `review_<NAME>_<YYYYMMDD>.md` with verdict: **PASS / CONDITIONAL PASS / FAIL** |

#### Validation logic chain

Every claim in the audit follows this chain — no qualitative assertions without quantitative backing:

```
Hypothesis → Math formalization → Python validation code → Measured result → Verdict
```

#### Audit standards

| Check | Method | Failure threshold |
|-------|--------|------------------|
| Data snooping | Bonferroni-corrected p-value across all tested parameter combos | p > 0.05 after correction |
| Look-ahead bias | Time-stamp audit + future-data substitution test | Any signal change on substitution |
| Overfitting | Backtest Degradation Ratio (BDR = 1 − Sharpe_OOS / Sharpe_IS) | BDR > 0.5 |
| Statistical significance | Block bootstrap 95% CI + permutation test | CI includes 0 or p > 0.05 |
| Regime dependency | Separate Sharpe by bull / bear / sideways | Sharpe < 0 in any regime |
| Tail risk | Cornish-Fisher VaR vs normal VaR gap | CF-VaR > 1.5× normal VaR |

---

## Installation

Both skills must be placed in `~/.claude/commands/` to be available globally across all Claude Code sessions.

```bash
# 1. Ensure the global commands directory exists
mkdir -p ~/.claude/commands

# 2. Copy both skills
cp commands/quant-researcher.md ~/.claude/commands/quant-researcher.md
cp commands/review-strategy.md  ~/.claude/commands/review-strategy.md
```

> For project-scoped installation (current directory only), copy to `.claude/commands/` instead.  
> See [add-new-skill.md](add-new-skill.md) for a full walkthrough of how Claude Code loads custom skills.

---

## Usage

### `/quant-researcher`

```
/quant-researcher <your strategy request>
```

**Examples**

```
/quant-researcher generate a momentum strategy for US equities
/quant-researcher design a mean-reversion strategy for Taiwan futures
/quant-researcher is a pairs trading strategy on TSMC and Samsung effective?
/quant-researcher 幫我設計一個適合台股的低波動因子策略
```

### `/review-strategy`

```
/review-strategy <path to strategy markdown file>
/review-strategy <paste strategy spec inline>
```

**Examples**

```
/review-strategy example/strategy_ATDF_20260507.md
/review-strategy 以下是一個雙均線+ADX策略，請審查其統計顯著性與邏輯漏洞：[貼上規格]
```

**Recommended workflow** — use both skills together:

```
/quant-researcher generate a breakout strategy for Taiwan futures
     ↓  (saves strategy_XXX_YYYYMMDD.md)
/review-strategy strategy_XXX_YYYYMMDD.md
     ↓  (saves review_XXX_YYYYMMDD.md with PASS/FAIL verdict)
```

## Example output

The [`example/`](example/) folder contains a sample run for a Taiwan futures trend-following strategy (ATDF):

| File | Description |
|------|-------------|
| [`strategy_ATDF_20260507.md`](example/strategy_ATDF_20260507.md) | Full research report (Traditional Chinese) |
| [`atdf_backtest_chart.png`](example/atdf_backtest_chart.png) | Equity curve, drawdown, annual returns, price + EMA, ADX charts |
| [`atdf_metrics.json`](example/atdf_metrics.json) | Machine-readable performance metrics |

**ATDF strategy headline numbers** (EMA 10/200 + ADX≥15, Long-Only, 10 bps/side, 2015–2026):

| Metric | ATDF Strategy | Buy & Hold ^TWII |
|--------|--------------|-----------------|
| CAGR | **15.90%** | 13.47% |
| Sharpe Ratio | **1.294** | 0.844 |
| Max Drawdown | **-18.69%** | -31.63% |
| Calmar Ratio | **0.851** | 0.426 |

## Repository structure

```
quant-research-skill/
├── skills/
│   ├── quant-researcher/
│   │   └── SKILL.md          # Full 4-phase research pipeline prompt
│   └── review-strategy/
│       └── SKILL.md          # Full 5-phase audit pipeline prompt
├── commands/
│   ├── quant-researcher.md   # Slim entry-point for ~/.claude/commands/
│   └── review-strategy.md    # Slim entry-point for ~/.claude/commands/
├── example/                  # Sample output from a real /quant-researcher run
│   ├── strategy_ATDF_20260507.md
│   ├── atdf_backtest_chart.png
│   └── atdf_metrics.json
├── add-new-skill.md          # Guide: how Claude Code loads custom skills
├── .gitignore                # Excludes *.py backtest scripts
└── package.json
```

## Requirements

The backtest phase (`/quant-researcher`) and statistical validation phase (`/review-strategy`) install Python dependencies automatically (`yfinance`, `pandas`, `numpy`, `matplotlib`, `scipy`). No manual setup needed — Claude handles it during execution.

## Reference

- [add-new-skill.md](add-new-skill.md) — step-by-step guide for installing custom skills in Claude Code (global vs. project scope, common mistakes)
