# QuantForge

Open-source quantitative finance research platform for market data engineering,
strategy development, backtesting, portfolio optimization, machine learning,
and AI-driven market analysis.

> **Status:** Phase 0 (Foundation) and Phase 1 (Data Engine) complete.
> Strategies, backtesting, portfolio optimization, ML, and AI research modules
> are planned for later phases — see [Roadmap](#roadmap).

## Why QuantForge

Most personal quant projects start as a notebook that downloads a CSV and
never becomes anything else. QuantForge is built the other way around:
a typed, tested Python package with a stable data contract, so every later
phase (features, strategies, backtesting, ML, portfolio optimization) can be
built on top of a foundation that already works.

```python
from quantforge.data import MarketData

market = MarketData()

df = market.get(
    symbol="AAPL",
    start="2024-01-01",
    end="2025-01-01",
    timeframe="1d",
)
```

The caller never needs to know which provider served the data, or whether it
came from cache or a fresh download.

## Install

```bash
git clone https://github.com/<your-username>/quantforge.git
cd quantforge

python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

pip install -e ".[dev]"
pip install yfinance         # first data provider
```

Copy `.env.example` to `.env` if you want to override any defaults (data
directories, log level, provider API keys for future providers).

## Quickstart

```bash
python examples/quickstart.py
```

or interactively:

```python
from quantforge.data import MarketData

market = MarketData()

aapl = market.get("AAPL", start="2020-01-01", end="2025-01-01", timeframe="1d")
print(aapl.head())

market.quality_report("AAPL")
market.list_datasets()

# Ad-hoc SQL over everything you've downloaded, via DuckDB:
market.query("SELECT symbol, AVG(close) FROM read_parquet('data/processed/*/*/data.parquet') GROUP BY symbol")
```

## Architecture

```
Provider  ->  Raw Data  ->  Normalizer  ->  Validator  ->  Parquet  ->  DuckDB  ->  Researcher
```

- **Provider** (`quantforge/data/providers/`) — pluggable data sources behind
  a single `DataProvider` interface. Ships with `YahooProvider`; Alpha
  Vantage, Polygon, Binance, and IBKR are designed to plug in later without
  touching the rest of the codebase.
- **Normalizer** (`quantforge/data/normalizer.py`) — maps any provider's raw
  columns onto the canonical schema: `timestamp, symbol, open, high, low,
  close, volume` (UTC, sorted, deduplicated).
- **Validator** (`quantforge/data/validators.py`) — detects (never silently
  repairs) bad data: non-positive or inconsistent OHLC prices, negative
  volume, duplicate or missing timestamps, timestamp gaps.
- **Storage** (`quantforge/data/storage.py`) — Parquet is the source of
  truth on disk (`data/processed/<symbol>/<timeframe>/data.parquet` +
  `metadata.json`); DuckDB is a queryable analytical layer on top of it.
- **Catalog** (`quantforge/data/catalog.py`) — inventory of every dataset
  QuantForge has stored locally.
- **MarketData** (`quantforge/data/market_data.py`) — the single high-level
  entry point (`get`, `download`, `validate`, `quality_report`,
  `list_datasets`, `delete`, `query`) that wires all of the above together,
  with on-disk caching so repeat research runs don't re-hit the network.

## Project layout

```
quantforge/
├── quantforge/
│   ├── config/       # Settings (pydantic-settings, .env-driven)
│   ├── data/
│   │   ├── models.py       # Canonical OHLCV schema + metadata models
│   │   ├── interfaces.py   # DataProvider / StorageBackend ABCs
│   │   ├── validators.py   # Schema / price / volume / timestamp checks
│   │   ├── normalizer.py   # Provider-specific -> canonical schema
│   │   ├── storage.py      # Parquet + DuckDB
│   │   ├── catalog.py      # Local dataset inventory
│   │   ├── market_data.py  # High-level MarketData API
│   │   └── providers/      # YahooProvider (first of several)
│   └── utils/        # Logging, timeframe/timestamp helpers
├── tests/             # pytest suite (unit + offline integration tests)
├── examples/          # Runnable usage examples
├── scripts/           # CLI helpers (e.g. quality_report.py)
├── data/
│   ├── raw/           # never committed
│   ├── processed/     # Parquet datasets, never committed
│   └── cache/
└── .github/workflows/ # CI: ruff + mypy + pytest on every push/PR
```

## Development

```bash
pytest                        # run the test suite
pytest --cov=quantforge       # with coverage
ruff check quantforge tests   # lint
mypy quantforge                # type-check
```

CI (`.github/workflows/tests.yml`) runs lint + tests on Python 3.11 and 3.12
for every push and pull request against `main`.

## Roadmap

| Phase | Module              | Status      |
|-------|----------------------|-------------|
| 0     | Architecture          | ✅ Done |
| 1     | Data Engine            | ✅ Done |
| 2     | Research / Feature Engine | Planned |
| 3     | Strategy Engine         | Planned |
| 4     | Backtesting Engine      | Planned |
| 5     | Risk Engine             | Planned |
| 6     | Portfolio Engine        | Planned |
| 7     | ML Engine               | Planned |
| 8     | AI Market Intelligence  | Planned |
| 9     | Regime Detection        | Planned |
| 10    | LOB / HFT Research      | Planned |
| 11    | Research Dashboard      | Planned |
| 12    | Experiment Management   | Planned |

## License

MIT — see [LICENSE](LICENSE).
