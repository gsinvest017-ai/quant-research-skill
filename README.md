# quant-research-skill

A Claude Code skill that turns Claude into a rigorous quantitative strategy researcher. One slash command triggers a full four-phase research pipeline: theory → literature → backtest → report.

## What it does

When you invoke `/quant-researcher <request>`, Claude will:

| Phase | Output |
|-------|--------|
| **1 — Theory** | Market hypothesis, mathematical signal definition, pseudo-code, risk factor analysis |
| **2 — Literature** | ≥2 verified academic citations (JF, JFE, RFS, JPM, NBER, SSRN) |
| **3 — Backtest** | Live data via `yfinance` / FinMind, ≥5-year window, 10 bps cost, no look-ahead bias; outputs CAGR, Sharpe, Max Drawdown, Calmar, volatility, win rate, equity curve chart |
| **4 — Report** | Full Traditional Chinese Markdown report saved to the working directory as `strategy_<NAME>_<YYYYMMDD>.md` |

## Installation

The skill file must be placed in `~/.claude/commands/` to be available globally across all Claude Code sessions.

```bash
# 1. Ensure the global commands directory exists
mkdir -p ~/.claude/commands

# 2. Copy the skill
cp commands/quant-researcher.md ~/.claude/commands/quant-researcher.md
```

> For project-scoped installation (current directory only), copy to `.claude/commands/` instead.  
> See [add-new-skill.md](add-new-skill.md) for a full walkthrough of how Claude Code loads custom skills.

## Usage

```
/quant-researcher <your strategy request>
```

### Examples

```
/quant-researcher generate a momentum strategy for US equities
/quant-researcher design a mean-reversion strategy for Taiwan futures
/quant-researcher is a pairs trading strategy on TSMC and Samsung effective?
/quant-researcher 幫我設計一個適合台股的低波動因子策略
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
│   └── quant-researcher/
│       └── SKILL.md          # Full skill prompt (4-phase research pipeline)
├── commands/
│   └── quant-researcher.md   # Slim entry-point for ~/.claude/commands/
├── example/                  # Sample output from a real run
│   ├── strategy_ATDF_20260507.md
│   ├── atdf_backtest_chart.png
│   └── atdf_metrics.json
├── add-new-skill.md          # Guide: how Claude Code loads custom skills
├── .gitignore                # Excludes *.py backtest scripts
└── package.json
```

## Requirements

The backtest phase installs Python dependencies automatically (`yfinance`, `pandas`, `numpy`, `matplotlib`, `scipy`). No manual setup needed — Claude handles it during execution.

## Reference

- [add-new-skill.md](add-new-skill.md) — step-by-step guide for installing custom skills in Claude Code (global vs. project scope, common mistakes)
