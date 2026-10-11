# Crypto Trading Agent

A production-ready crypto trading agent for cryptocurrency and crypto-linked assets with backtesting, risk management, multi-exchange support, and a comprehensive REST API.

## Features

- **Multi-Exchange Support**: Binance, Bybit, Coinbase, Kraken (via CCXT)
- **Asset Classes**: Cryptocurrency and crypto-linked equities
- **Strategy Engine**: Modular signal generation (EMA, RSI, MACD, breakout, mean reversion)
- **Risk Management**: Position sizing, max drawdown, daily loss limits, leverage caps, stop-loss/take-profit
- **Execution Layer**: Order placement, modification, cancellation with retries and idempotency
- **Backtesting**: Historical replay with performance metrics
- **REST API**: FastAPI-based configuration, monitoring, and control
- **Monitoring & Alerts**: Health checks, trade logging, audit trails
- **Paper Trading**: Safe testing before live execution
- **Database**: PostgreSQL for persistence, Redis for caching and rate-limiting
- **Docker**: Full containerization for deployment

## Architecture

```
crypto-trading-agent/
├── app/
│   ├── __init__.py
│   ├── config.py              # Configuration management
│   ├── main.py                # Application entry point
│   ├── api/                   # REST API endpoints
│   │   ├── routes.py
│   │   ├── schemas.py
│   │   └── dependencies.py
│   ├── agents/                # Trading agents
│   │   ├── base_agent.py
│   │   ├── crypto_agent.py
│   │   └── equity_agent.py
│   ├── strategies/            # Strategy implementations
│   │   ├── base_strategy.py
│   │   ├── trend_strategy.py
│   │   ├── momentum_strategy.py
│   │   ├── breakout_strategy.py
│   │   └── mean_reversion.py
│   ├── risk/                  # Risk management
│   │   ├── portfolio_manager.py
│   │   ├── position_sizer.py
│   │   └── risk_limits.py
│   ├── execution/             # Order execution
│   │   ├── exchange_adapter.py
│   │   ├── order_manager.py
│   │   └── trade_reconciliation.py
│   ├── data/                  # Data layer
│   │   ├── market_data_service.py
│   │   ├── data_store.py
│   │   └── indicators.py
│   ├── monitoring/            # Monitoring & alerts
│   │   ├── logger.py
│   │   ├── health_check.py
│   │   └── alerting.py
│   ├── backtester/            # Backtesting engine
│   │   ├── backtest_engine.py
│   │   └── metrics.py
│   └── models/                # Database models
│       ├── trade.py
│       ├── order.py
│       └── market_data.py
├── tests/
│   ├── test_strategies.py
│   ├── test_risk_management.py
│   ├── test_execution.py
│   └── test_backtester.py
├── config/
│   ├── development.yaml
│   ├── production.yaml
│   └── backtest.yaml
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
├── pytest.ini
├── .env.example
└── README.md
```

## Quick Start

### Prerequisites

- Python 3.9+
- PostgreSQL 12+
- Redis 6+
- Docker & Docker Compose

### Installation

1. Clone the repository:
```bash
git clone https://github.com/Wenqin740/crypto-trading-agent.git
cd crypto-trading-agent
```

2. Create virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

4. Set up environment:
```bash
cp .env.example .env
# Edit .env with your API keys and configuration
```

5. Run with Docker Compose:
```bash
docker-compose up -d
```

6. Start the agent:
```bash
python app/main.py --config config/development.yaml --mode paper
```

## API Endpoints

- `GET /health` - Health check
- `GET /status` - Agent status and positions
- `GET /trades` - List trades
- `POST /orders` - Place manual order
- `GET /config` - Get configuration
- `PUT /config` - Update configuration
- `POST /backtest` - Run backtest
- `GET /backtest/{id}` - Get backtest results
- `POST /alerts` - Configure alerts

## Configuration

Edit `config/development.yaml`:

```yaml
trading:
  mode: paper              # paper or live
  exchanges:
    - name: binance
      api_key: ${BINANCE_API_KEY}
      secret: ${BINANCE_SECRET}
    - name: bybit
      api_key: ${BYBIT_API_KEY}
      secret: ${BYBIT_SECRET}

strategies:
  - name: trend_ema
    enabled: true
    params:
      fast_ema: 12
      slow_ema: 26
      min_confidence: 0.7

risk:
  max_position_size: 0.05    # 5% of portfolio
  max_leverage: 2.0
  max_daily_loss: 0.02       # 2% max daily loss
  max_drawdown: 0.10         # 10% max drawdown

database:
  url: postgresql://user:password@localhost:5432/crypto_agent

redis:
  url: redis://localhost:6379/0
```

## Usage Examples

### Run Paper Trading

```bash
python app/main.py --config config/development.yaml --mode paper
```

### Run Backtest

```bash
python app/main.py backtest \
  --strategy trend_ema \
  --symbol BTC/USDT \
  --start 2023-01-01 \
  --end 2024-01-01 \
  --initial-capital 10000
```

### Deploy to Production

```bash
docker-compose -f docker-compose.prod.yml up -d
```

## Risk Management

The agent enforces multiple layers of risk control:

1. **Position Sizing**: Kelly Criterion or fixed fraction of capital
2. **Stop-Loss/Take-Profit**: Per-trade risk/reward ratios
3. **Max Drawdown**: Portfolio-level daily loss limits
4. **Leverage Caps**: Maximum allowed leverage per asset
5. **Circuit Breaker**: Halts trading on volatility spikes or API issues
6. **Reconciliation**: Validates orders against exchange state every minute

## Testing

```bash
pytest -v tests/
pytest --cov=app tests/
```

## Monitoring

The agent logs all trades, orders, and signals to PostgreSQL. Monitor via:

- REST API: `/status`, `/trades`
- Database: Query `trades`, `orders`, `signals` tables
- Redis: Cache hit rates and rate-limit state
- Alerts: Email/Slack on risk violations or execution errors

## Safety Features

- **Paper Trading Mode**: Test strategies without real capital
- **Kill Switch**: Disable trading instantly via API
- **Audit Trail**: Every decision logged with timestamp and rationale
- **Order Confirmation**: Manual approval on first live trade
- **Health Checks**: Validates exchange connectivity and data freshness
- **Reconciliation**: Ensures local state matches exchange state

## Deployment

### Development

```bash
docker-compose up
# Runs: PostgreSQL, Redis, API server
# Access: http://localhost:8000/docs
```

### Production

Edit `docker-compose.prod.yml` with:
- SSL certificates
- Database backups
- Monitoring stack (Prometheus, Grafana)
- Log aggregation (ELK)

```bash
docker-compose -f docker-compose.prod.yml up -d
```

## Contributing

1. Create a feature branch: `git checkout -b feature/my-strategy`
2. Implement with tests
3. Submit a pull request

## License

MIT

## Disclaimer

This is an educational project. Use at your own risk. Crypto trading is risky. Never trade with capital you cannot afford to lose. Thoroughly backtest and paper trade before going live.

## Support

- Documentation: `/docs` (Swagger UI)
- Issues: GitHub Issues
- Discussions: GitHub Discussions
