# Jarvis Hybrid — Build-Plan

## Ziel
Zwei Versionen bauen:
1. **Light** (Cron-tauglich, keine LLM-Calls) — Faktor-Analyse + Backtest-Metriken + Risk-Parity
2. **Full** (On-Demand, mit Swarm) — Earnings Research Desk mit 4 LLM-Agenten

## Modelle
- **Planung:** `ollama-cloud/glm-5.2` (Cloud, wir gerade am Laufen)
- **Schreiben:** `nvidia/nemotron-3-ultra-550b-a55b:free` (OpenRouter, kostenlos, 550B, 1M ctx)
- **Schreiben Fallback:** `qwen/qwen3-coder:free` (OpenRouter, kostenlos, 480B)

## Phase 1: Light-Version

### 1.1 Backtest-Metriken (`light/backtest/metrics.py`)
- Sharpe Ratio (annualisiert)
- Sortino Ratio
- Max Drawdown
- Win Rate, Profit Factor
- Annualisierung für verschiedene Intervalle (Tage/Stunden/Minuten)
- Quelle: `shared/backtest_base.py` + eigenes Wissen

### 1.2 Faktor-Analyse (`light/analysis/factor_analysis.py`)
- IC (Information Coefficient) — Spearman Rank Korrelation Faktor vs Returns
- IR (Information Ratio) — mean(IC) / std(IC)
- Layered Backtest — Quantil-Portfolios (Top/Bottom Gruppen)
- Quelle: `shared/factor_analysis.py` (bereits extrahiert)

### 1.3 Risk-Parity Optimizer (`light/risk/risk_parity.py`)
- Spinu (2013) Style: Inverse-Vol Seed + Newton-Refinement
- Equal-Risk-Contribution Weights
- Quelle: Vibe-Trading `backtest/optimizers/risk_parity.py`

### 1.4 Faktor-Integration (`light/factors/__init__.py`)
- Wrapper für `shared/` Faktoren (HML, SMB, BAB, Illiq, RetSkew, Alpha101)
- Anpassung an unser Depot-Format (Dict von DataFrames, nicht Panel)

### 1.5 Depot-Adapter (`light/depot_adapter.py`)
- Liest Positionen aus SQLite (`musterdepot.db`)
- Liefert Preise als wide DataFrame (index=Datum, columns=Ticker)
- Berechnet Returns

## Phase 2: Full-Version

### 2.1 Earnings Research Desk (`full/swarm/`)
4 Agenten als Python-Funktionen (kein DAG-Framework, simples async/await):

- **fundamental_analyst.py** — yfinance Fundamentaldaten, Bilanzqualität, Accrual Ratio, FCF
- **revision_tracker.py** — Analyst Upgrades/Downgrades, Consensus-Schätzungen
- **options_analyst.py** — Implied Move, IV Crush (yfinance options chain)
- **earnings_strategist.py** — Synthese → Trade-Empfehlung

### 2.2 Orchestrator (`full/orchestrator.py`)
- `analyze_earnings(symbol)` — ruft 4 Agenten parallel auf, wartet auf alle, synthetisiert
- LLM-Calls via OpenRouter (nemotron-3-ultra:free)
- Output: Strukturiertes Dict (Signal, Confidence, Entry, Stop, Target, Size)

### 2.3 Telegram-Integration (`full/telegram_report.py`)
- Formatierter Report für Telegram
- Auf-Demand via Sascha-Befehl: "analyse NVDA"

## Phase 3: Tests & Beispiele
- `examples/test_light.py` — Faktor-Analyse für unsere 32 Ticker
- `examples/test_full.py` — Earnings-Analyse für 1 Ticker
- `examples/test_risk_parity.py` — Risk-Parity für Depot 2

## Ausführung
1. OpenCode mit `nvidia/nemotron-3-ultra-550b-a55b:free` starten
2. Task: "Implementiere Phase 1.1 bis 1.5 gemäss BUILD_PLAN.md"
3. Commits direkt auf master
4. Ich (Jarvis/GLM-5.2) reviewe und merge

## Konfiguration OpenCode

```bash
cd /home/sascha/.openclaw/workspace/jarvis-hybrid

# OpenCode mit nemotron starten
opencode run -m openrouter/nvidia/nemotron-3-ultra-550b-a55b:free \
  --title "Jarvis Hybrid Phase 1" \
  "Lies BUILD_PLAN.md und implementiere Phase 1.1 bis 1.5. Commits auf master."
```