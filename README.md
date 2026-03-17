# Trading Dashboard

A full-stack trading dashboard built with Go, React, PostgreSQL, and Docker.

## Quick Start

```bash
docker-compose up --build
```

Then open **http://localhost:3000**

**Demo login:**
- Email: `demo@trading.com`
- Password: `password123`

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   React (Nginx) │────▶│   Go REST API   │────▶│   PostgreSQL    │
│   Port: 3000    │     │   Port: 8080    │     │   Port: 5432    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Tech Stack

| Layer         | Technology                          |
|---------------|-------------------------------------|
| Frontend      | React 18, Recharts, React Router v6 |
| Backend       | Go 1.21, Gin framework              |
| Auth          | JWT (HS256, 24h expiry)             |
| Database      | PostgreSQL 15                       |
| Containers    | Docker + Docker Compose             |

## Features

- **JWT Authentication** — Secure login with bcrypt password hashing
- **Market Overview** — Watchlist of stocks and mutual funds
- **Search** — Real-time symbol & name search across all tracked instruments
- **Price Charts** — Area chart with 1-day and 30-day trend view
- **Price Table** — Sortable OHLCV table with change metrics
- **Data Pipeline** — Backend fetches market data (Alpha Vantage or simulated) and stores in PostgreSQL; runs hourly via cron

## API Endpoints

| Method | Endpoint                        | Auth | Description                     |
|--------|---------------------------------|------|---------------------------------|
| POST   | `/api/auth/login`               | No   | Login, returns JWT token        |
| GET    | `/api/health`                   | No   | Health check                    |
| GET    | `/api/market/symbols`           | Yes  | List all tracked symbols        |
| GET    | `/api/market/search?q=AAPL`     | Yes  | Search symbols by name/ticker   |
| GET    | `/api/market/history?symbol=AAPL&days=30` | Yes | Price history |
| POST   | `/api/market/refresh`           | Yes  | Trigger manual data refresh     |

## Database Schema

```sql
-- Users table
CREATE TABLE users (
  id            SERIAL PRIMARY KEY,
  email         VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name          VARCHAR(255) NOT NULL,
  created_at    TIMESTAMP DEFAULT NOW()
);

-- Market prices table (OHLCV daily data)
CREATE TABLE market_prices (
  id         SERIAL PRIMARY KEY,
  symbol     VARCHAR(20) NOT NULL,
  name       VARCHAR(255) NOT NULL,
  asset_type VARCHAR(50) NOT NULL,   -- 'stock' or 'mutual_fund'
  price      DECIMAL(18,4) NOT NULL, -- closing price
  open       DECIMAL(18,4),
  high       DECIMAL(18,4),
  low        DECIMAL(18,4),
  volume     BIGINT,
  change     DECIMAL(18,4),
  change_pct DECIMAL(10,4),
  date       DATE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(symbol, date)
);
```

## Tracked Instruments

**Stocks:** AAPL, GOOGL, MSFT, AMZN, TSLA, NVDA, META, JPM, V, JNJ

**Mutual Funds:** VFINX, FXAIX, VTSAX, AGTHX, PRGFX

## Design Decisions

1. **Data Pipeline**: Backend attempts to fetch from Alpha Vantage API first; falls back to deterministic simulated data if the API is unavailable (so the app always works offline/demo)
2. **Upsert pattern**: Market data uses `ON CONFLICT DO UPDATE` to handle re-runs gracefully
3. **Cron scheduling**: Data refresh runs hourly using `robfig/cron`
4. **Security**: Passwords hashed with bcrypt; JWT tokens expire in 24h; CORS configured explicitly; timing-safe auth error responses
5. **Frontend proxy**: Nginx proxies `/api/` to the Go backend — frontend never calls external APIs directly
