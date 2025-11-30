# StockWise Backend API

Production-ready Express.js backend for the StockWise investment dashboard.

## Assumptions

- **Default User**: User ID defaults to `1` if no `user-id` header is provided
- **Paper Trading**: Default paper trading balance is $10,000
- **Stock API Behavior**:
  - If `ALPHA_VANTAGE_API_KEY` is configured: Fetches real data from Alpha Vantage `TIME_SERIES_DAILY_ADJUSTED`, extracts the last 45 market days of closing prices, caches results for 1 hour, and enforces rate limiting (5 calls per 60 seconds). If rate limit is exceeded, returns cached data (even if expired) if available, otherwise returns HTTP 429.
  - If `ALPHA_VANTAGE_API_KEY` is missing: Returns mock data with identical schema to maintain frontend compatibility
  - Stock symbols are validated, sanitized, and uppercased
- **Caching**: Stock data cached for 1 hour TTL in JSON database under `stock_cache[symbol]`
- **Rate Limiting**: In-memory rate limiter prevents exceeding Alpha Vantage free tier (5 calls/minute)

## Quick Start

### 1. Install Dependencies

```bash
npm install
```

### 2. Configure Environment

Copy `.env.example` to `.env` and update values:

```bash
cp .env.example .env
```

Edit `.env`:
```env
PORT=3001
NODE_ENV=development
DATA_FILE=./data/stockwise.json
ALPHA_VANTAGE_API_KEY=your_key_here  # Optional - if missing, uses mock data
```

### 3. Initialize Database

```bash
npm run init-db
```

This creates `data/stockwise.json` with default data.

### 4. Start Server

Development (with auto-reload):
```bash
npm run dev
```

Production:
```bash
npm start
```

Server runs on `http://localhost:3001`

## Frontend Integration

### Next.js Configuration

Add rewrites to `next.config.mjs`:

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'http://localhost:3001/api/:path*',
      },
    ];
  },
};

export default nextConfig;
```

### Environment Variables

For production deployment, set:
- `NEXT_PUBLIC_API_URL=http://your-backend-url/api`

Then update frontend fetch calls to use `process.env.NEXT_PUBLIC_API_URL` or the rewrite above.

## API Endpoints

### Health
- `GET /health` - Health check

### Stock Data
- `GET /api/stock/:symbol` - Get stock data (historical, predicted, analysis)
  - Validates and sanitizes symbol parameter
  - Returns last 45 market days of historical data
  - Caches results for 1 hour
  - Rate limited to 5 calls per minute

### Portfolio
- `GET /api/portfolio` - Get portfolio summary
- `GET /api/portfolio/holdings` - List holdings
- `POST /api/portfolio/holdings` - Add holding
- `DELETE /api/portfolio/holdings/:id` - Remove holding
- `GET /api/portfolio/transactions` - Transaction history

### User
- `GET /api/user` - Get user profile
- `PUT /api/user` - Update user profile

### Market
- `GET /api/market/stocks` - Get market stocks list

## Testing

```bash
npm test
```

Tests cover:
- Health endpoint
- Market stocks endpoint
- Stock endpoint (mock data, API calls, rate limiting, validation)

## Alpha Vantage Integration

1. Get free API key: https://www.alphavantage.co/support/#api-key
2. Add to `.env`: `ALPHA_VANTAGE_API_KEY=your_key`
3. Rate limits: 5 calls/minute, 500 calls/day (free tier)

**Behavior**:
- If API key present: Fetches real data, caches for 1 hour
- If rate limited: Returns cached data (even if expired) if available, otherwise HTTP 429
- If API key missing: Returns mock data with same schema
- Always returns HTTP 200 (unless rate limited with no cache, then 429)

## Data Storage

Data is stored in JSON file at `./data/stockwise.json`. Structure:
- `users` - User profiles
- `portfolio_holdings` - Investment holdings
- `transactions` - Transaction history
- `stock_cache` - Cached stock data (1 hour TTL)
- `market_stocks` - Market stocks list

## Error Handling

All errors return JSON:
```json
{
  "error": "Error message"
}
```

Status codes:
- `200` - Success
- `201` - Created
- `204` - No Content (delete)
- `400` - Bad Request
- `404` - Not Found
- `429` - Too Many Requests (rate limit exceeded)
- `500` - Internal Server Error

