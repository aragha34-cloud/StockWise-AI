# StockWise API - cURL Examples

## Health Check

```bash
curl http://localhost:3001/health
```

## Stock Data

```bash
# Get AAPL stock data
curl http://localhost:3001/api/stock/AAPL
```

## Portfolio

```bash
# Get portfolio summary
curl http://localhost:3001/api/portfolio \
  -H "user-id: 1"

# Get holdings
curl http://localhost:3001/api/portfolio/holdings \
  -H "user-id: 1"

# Add holding
curl -X POST http://localhost:3001/api/portfolio/holdings \
  -H "Content-Type: application/json" \
  -H "user-id: 1" \
  -d '{
    "symbol": "MSFT",
    "companyName": "Microsoft",
    "shares": 5,
    "pricePerShare": 374.23,
    "purchaseDate": "2025-11-06"
  }'

# Delete holding
curl -X DELETE http://localhost:3001/api/portfolio/holdings/1 \
  -H "user-id: 1"

# Get transactions
curl http://localhost:3001/api/portfolio/transactions \
  -H "user-id: 1"
```

## User

```bash
# Get user profile
curl http://localhost:3001/api/user \
  -H "user-id: 1"

# Update user profile
curl -X PUT http://localhost:3001/api/user \
  -H "Content-Type: application/json" \
  -H "user-id: 1" \
  -d '{
    "name": "Updated Name",
    "experienceLevel": "Intermediate"
  }'
```

## Market

```bash
# Get market stocks
curl http://localhost:3001/api/market/stocks
```

