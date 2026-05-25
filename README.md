# Financial Analysis & Automation

Ingests financial news, enriches it with LLM-extracted metadata, and delivers sentiment analysis + event classification via REST and WebSocket.

## Stack

Kafka · FastAPI · PostgreSQL · Redis · Claude (Anthropic) · Docker

## Getting Started

**Prerequisites:** Docker, Python 3.12+, TheNewsAPI token, Anthropic API key

```bash
git clone https://github.com/your-org/financial-analysis-automation.git
cd financial-analysis-automation
cp .env.example .env   # fill in THENEWSAPI_KEY and ANTHROPIC_API_KEY
make up                # start Kafka, Redis, Postgres
make install && make migrate
docker compose up      # or run services individually (see below)
```

**Verify:** `curl http://localhost:8000/health` · Swagger: `http://localhost:8000/docs`

### Running services individually

```bash
make dev          # API on :8000
make run-ingest   # ingestion workers
make run-process  # LLM processor
make run-analysis # market analysis
```

## Key Environment Variables

| Variable | Description |
|---|---|
| `THENEWSAPI_KEY` | TheNewsAPI token (required) |
| `ANTHROPIC_API_KEY` | Anthropic API key (required) |
| `DATABASE_URL` | Async PostgreSQL DSN |
| `REDIS_URL` | Redis URL |
| `KAFKA_BOOTSTRAP_SERVERS` | Kafka brokers |
| `API_KEYS` | Comma-separated valid API keys |
| `LLM_MODEL` | Model ID (default: `claude-sonnet-4-6`) |

See [.env.example](.env.example) for all options.

## API Endpoints

All endpoints except `/health` require `X-API-Key`.

| Method | Path | Description |
|---|---|---|
| `GET` | `/health` | Health check |
| `POST/GET` | `/articles` | Ingest or list articles |
| `WS` | `/subscribe` | Real-time article stream |
| `GET` | `/analysis/articles/{id}` | Event type + sentiment |
| `GET` | `/analysis/companies/{ticker}/sentiment` | Sentiment feed |
| `GET` | `/analysis/events` | Browse events by type |
| `WS` | `/analysis/stream` | Live analysis stream |
| `GET` | `/stocks/search?q=...` | Natural-language stock search |
| `GET` | `/stocks/{ticker}` | Stock detail + metrics |

## Adding a Feed Source

```python
# src/ingestion/my_source.py
from src.ingestion.base import FeedAdapter

class MySourceAdapter(FeedAdapter):
    source_name = "my-source"

    async def fetch(self) -> list[RawArticle]:
        ...
```

Then add it to `adapters = [..., MySourceAdapter()]` in `src/ingestion/runner.py`.

## Development

```bash
make install    # install dependencies
make test       # pytest
make lint       # ruff check
make load-test  # locust load test
```

## License

MIT
