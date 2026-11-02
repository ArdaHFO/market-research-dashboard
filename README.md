Market Research Dashboard brings company news, price charts and recorded forecasts together for easier comparison.

<h1 align="center">Market Research Dashboard</h1>

<p align="center"><strong>Read the news. Inspect the chart. Compare the evidence.</strong></p>
<p align="center">A Python market-research dashboard combining financial headlines, technical indicators and recorded directional forecasts.</p>

<p align="center">
  <strong>English</strong> · <a href="README.tr.md">Türkçe</a><br />
  <a href="#run-locally"><strong>Run locally</strong></a> ·
  <a href="#inside-the-dashboard">Explore the workflow</a> ·
  <a href="https://github.com/ArdaHFO/market-research-dashboard/issues">Report an issue</a>
</p>

---

Market Research Dashboard brings symbol search, headline analysis, interactive charts, watchlists and forecast history into one FastAPI application. Compare news-heavy and technical-heavy views, inspect different timeframes, and review recorded outcomes rather than treating a single score as a complete decision.

**This is a research application with a Turkish interface.** It generates analytical outputs and records predictions. It does not contain a brokerage or exchange order-execution integration. English and Turkish documentation are provided.

## Run locally

| Requirement | Current repository |
| --- | --- |
| Python | The supplied Dockerfile uses Python 3.12 |
| Dependencies | Install [`requirements.txt`](requirements.txt) into an isolated environment |
| Storage | SQLite by default, PostgreSQL when `DATABASE_URL` is configured |
| Market data | Internet access for Yahoo Finance data through `yfinance` |
| News analysis | A Groq API key for language-model sentiment, optional provider keys for additional news sources |
| Browser | A modern browser with access to the chart-library CDNs |

Create a fresh virtual environment. The repository currently contains a tracked `.venv` tree, which is not a portable development environment.

```bash
git clone https://github.com/ArdaHFO/market-research-dashboard.git
cd market-research-dashboard
python -m venv .venv-local
```

Activate it with `.venv-local\Scripts\Activate.ps1` in PowerShell, or `source .venv-local/bin/activate` in a POSIX shell. Then install the dependencies.

```bash
python -m pip install -r requirements.txt
```

Copy [`.env.example`](.env.example) to `.env`. For a local HTTP session, set `COOKIE_SECURE=false`. Add your own `GROQ_API_KEY` to enable news sentiment, or leave it empty to explore the technical-analysis path. `REGISTRATION_CODE` can be empty for a private local development session.

```bash
python main.py serve
```

Open `http://127.0.0.1:8000`, choose **Kaydol** to create a local account, then use **Giriş Yap** to sign in. The CLI server binds to `0.0.0.0`, so this development configuration should not be exposed as a public service.

For hosted use, keep secure cookies enabled, provide HTTPS and set a registration code. The current browser signup form does not submit that code. A protected signup must use `POST /api/register` with `username`, `password` and `invite_code`, then the existing browser login can be used. The API schema is available at `/docs`.

## Inside the dashboard

- **Search and select.** Find symbols by ticker or company name. Browse curated BIST, US and European lists, then save selected items to a personal watchlist.
- **Read the evidence.** Combine headlines from Google News and Yahoo Finance, with optional Finnhub and NewsAPI sources. Groq produces a structured news assessment when configured.
- **Inspect charts.** Explore price bars and technical indicators, including moving averages, RSI, MACD, Bollinger Bands, Supertrend, VWAP, pivot and Fibonacci levels.
- **Compare profiles.** Choose balanced, news-heavy, technical-heavy, news-only, technical-only or learned scoring. The comparison action runs the three blended profiles side by side.
- **Explore timeframes.** Use `30m`, `1h`, `1d`, `1wk` or `1mo` and compare multiple selected intervals.
- **Screen and follow up.** A technical screener ranks preset universes without calling the news language model. History, outcome summaries and CSV export support later inspection.
- **Keep personal settings.** Accounts have their own watchlists, settings and recorded analysis state.

Typical browser flow is **Analiz** → select symbols and settings → **Analiz Et** → open a result card. Use **Profilleri Karşılaştır** for profile comparison, **Çoklu Zaman Dilimi** for interval comparison, **Tarayıcı** for screening and **Panom** for history.

## Configuration

| Variable | Purpose |
| --- | --- |
| `GROQ_API_KEY` / `GROQ_MODEL` | News sentiment credentials and model selection |
| `FINNHUB_API_KEY` / `NEWSAPI_KEY` | Enable optional headline providers |
| `DATABASE_URL` | PostgreSQL DSN, when using a server database |
| `FORECASTER_DB_PATH` | SQLite path when `DATABASE_URL` is absent, defaults to `predictions.db` |
| `WATCHLIST_PATH` | CLI watchlist, defaults to `watchlist.json` |
| `MODEL_PATH` | Learned-model JSON, defaults to `model.json` |
| `REGISTRATION_CODE` | Optional code required by the registration API |
| `COOKIE_SECURE` | Defaults to secure cookies, use `false` only for local HTTP |
| `PORT` | Web-server port, defaults to `8000` |

Additional controls for news lookback, article limits, blend weights and concurrency-related symbol limits are defined in [`config.py`](src/forecaster/config.py). Keep local credentials and database files out of commits. The application sends selected news material to Groq when sentiment analysis is enabled.

## Command-line workflow

Run commands from the repository root with the environment activated.

| Command | Effect |
| --- | --- |
| `python main.py serve` | Start the web interface and API |
| `python main.py run` | Analyze the JSON watchlist once and persist predictions |
| `python main.py backfill` | Fetch prices and resolve eligible pending predictions |
| `python main.py report --days 7` | Read recent resolved outcomes from the database |
| `python main.py train --universe us --timeframe 1d --horizon 60` | Fetch historical data and train/evaluate a candidate model |

Analysis, screening, backfill and training can make external requests. `run` is a one-shot command, not an installed scheduler. The sample watchlist contains AAPL, MSFT, NVDA, TSLA and GOOGL and can be replaced with your own research universe.

## Models and interpretation

The standard profiles blend news and technical scores. If news is unavailable, blended profiles transfer its weight to technical analysis. A news-only run with no usable news stays neutral. A requested learned profile falls back to the balanced blend when no usable model is available.

The optional learned model is a logistic regression implementation. Training uses per-symbol chronological splits, a horizon-sized gap between segments, and validation-based regularization selection. A candidate is saved when it meets the code's trend-baseline accuracy and AUC conditions. Historical training examples contain technical and market features, with the news feature set to zero.

The included [`model.json`](model.json) contains a historical training summary and a 60-bar horizon. Its metadata is not a fresh validation result. The application's outcome recorder grades predictions against a subsequent price bar, so those dashboard hit rates and the model's 60-bar evaluation describe different horizons and must not be compared as equivalent results. Directional scores and historical accuracy do not establish future returns.

## Development and validation

FastAPI serves both JSON endpoints and the embedded HTML/CSS/JavaScript interface in [`webapp.py`](src/forecaster/webapp.py). The browser loads Chart.js and Lightweight Charts from CDNs. There is no separate frontend build step.

The repository includes pytest coverage for indicators, fusion, sentiment parsing, deduplication, pipeline caching/timeframes, model utilities, chart levels, storage and watchlist behavior.

```bash
python -m pytest -q
```

The 2 November 2026 source review corrected request-state handling, model fallback labels, multilingual headline deduplication, database compatibility, prediction provenance, resolved-only performance summaries and authentication/interface issues. Applications and tests were not run during this review, so the changes still require runtime validation. The existing test sources above are provided for that purpose. Dependencies are currently unpinned, so a new installation may differ from the original environment. External data coverage, rate limits and provider availability affect results.

For a container-based launch, the supplied [`Dockerfile`](Dockerfile) installs the same requirements and runs `python main.py serve`. Configure persistence separately for SQLite or use PostgreSQL. A container definition alone does not establish a validated public deployment.

## Repository map

| Path | Purpose |
| --- | --- |
| [`main.py`](main.py) | CLI entry points |
| [`src/forecaster/webapp.py`](src/forecaster/webapp.py) | API, authentication and embedded dashboard |
| [`src/forecaster/pipeline.py`](src/forecaster/pipeline.py) | News/technical orchestration and recording |
| [`src/forecaster/news/`](src/forecaster/news/) | Headline fetching and Groq sentiment |
| [`src/forecaster/technical/`](src/forecaster/technical/) | Price data, indicators and scoring |
| [`src/forecaster/fusion/`](src/forecaster/fusion/) | Profile-specific score combination |
| [`src/forecaster/learning/`](src/forecaster/learning/) | Features, logistic model, training and evaluation |
| [`src/forecaster/storage/`](src/forecaster/storage/) | SQLite/PostgreSQL persistence and outcome resolution |
| [`tests/`](tests/) | Existing automated checks |

## Contributing

Include the affected symbol, timeframe, profile, expected behavior and sanitized reproduction steps in an issue. For calculation changes, add a small deterministic fixture and state the evaluation horizon. Keep credentials, personal records and local environments out of contributions. Distinguish offline tests, mocked provider responses and live-service checks in validation reports.

## License and credits

No project-level license file is present. This documentation does not assign a new license. Python packages and browser chart libraries retain their own licenses, and data providers retain their own service terms. The existing application interface is labelled TradeView. The project is not presented as an official product of those providers.
