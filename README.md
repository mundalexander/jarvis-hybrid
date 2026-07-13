# Jarvis Hybrid Trading

**Akademische Faktor-Bibliothek + Backtest-Infrastruktur + Swarm Earnings Desk**
Basierend auf [HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading) (MIT) und unserem Musterdepot.

## Architektur

```
jarvis-hybrid/
├── light/                  # Lightversion (Cron, keine LLM-Calls)
│   ├── factors/             # Akademische Faktoren (HML, SMB, BAB, Illiq, RetSkew)
│   ├── backtest/            # Backtest-Metriken (Sharpe, Sortino, Max DD)
│   ├── risk/                # Risk Parity Optimizer
│   └── analysis/            # IC/IR, Layered Backtest
├── full/                   # Vollversion (On-Demand, mit Swarm)
│   ├── swarm/               # Multi-Agent Earnings Desk
│   │   ├── fundamental_analyst.py
│   │   ├── revision_tracker.py
│   │   ├── options_analyst.py
│   │   └── earnings_strategist.py
│   └── skills/              # Angepasste Skill-Prompts
├── shared/                 # Gemeinsame Code-Basis
│   ├── factor_base.py       # Operator-Bibliothek (rank, zscore, ts_rank, etc.)
│   ├── factor_registry.py   # Faktor-Registry
│   ├── backtest_base.py     # DataLoader Protocol, Validation
│   ├── yfinance_loader.py   # yfinance Daten-Loader
│   └── shadow_storage.py   # Shadow Account Persistenz
└── examples/               # Beispiele & Tests
```

## zwei Modi

### Light (Cron-Tauglich)
- Reine Python-Funktionen, keine LLM-Calls
- Läuft in Sekunden
- Faktor-Berechnung, IC/IR, Backtest-Metriken, Risk-Parity
- Integration in bestehende `auto_stop_loss.py`, `strategy_sync.py`

### Full (On-Demand)
- LLM-gestützte Tiefenanalyse für einzelne Ticker
- 4 Agenten: Fundamental, Revision, Options, Strategist
- Läuft in ~5-10 Minuten pro Ticker
- Nutzt `qwen/qwen3-coder:free` (OpenRouter, kostenlos, 480B)

## Modelle

| Zweck | Modell | Kosten |
|------|-------|------|
| Planung/Architektur | `ollama-cloud/glm-5.2` | Cloud |
| Schreiben/Coding | `qwen/qwen3-coder:free` (OpenRouter) | Kostenlos |
| Fallback | `bigmac/qwen3.6:35b-a3b-coding-nvfp4` (Gaming PC) | Lokal |

## Lizenz

MIT (geerbt von Vibe-Trading)