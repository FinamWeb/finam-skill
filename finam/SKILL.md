---
name: finam
description: Execute trades, manage portfolios, and access real-time market data via Finam Trade API
metadata: '{"openclaw": {"emoji": "📈", "homepage": "https://tradeapi.finam.ru/", "requires": {"bins": ["curl", "jq"], "env": ["FINAM_API_KEY", "FINAM_ACCOUNT_ID"]}, "primaryEnv": "FINAM_API_KEY"}}'
---

# Finam Trade API Skill

## Setup

Obtain and store JWT token before using the API:

```shell
export FINAM_JWT_TOKEN=$(curl -sL "https://api.finam.ru/v1/sessions" \
--header "Content-Type: application/json" \
--data "{\"secret\": \"$FINAM_API_KEY\"}" | jq -r '.token')
```

**Note:** Token expires after 15 minutes. Re-run this command if you receive authentication errors.

## Usage

### General Rules

**Symbol Format:** All symbols must be in `ticker@mic` format (e.g., `SBER@MISX`)

**Available MIC Codes:**
- `MISX` - Moscow Exchange
- `XNGS` - NASDAQ/NGS
- `XNYS` - New York Stock Exchange
- `ARCX` - NYSE ARCA
- `RUSX` - RTS

---

## Account Management

### Get Account Portfolio

Retrieve portfolio information including positions, balances, and P&L:

```shell
curl -sL "https://api.finam.ru/v1/accounts/$FINAM_ACCOUNT_ID" \
  --header "Authorization: $FINAM_JWT_TOKEN" | jq
```

## Market Data

### Get Latest Quote

Retrieve current bid/ask prices and last trade:

```shell
curl -sL "https://api.finam.ru/v1/instruments/{SYMBOL}/quotes/latest" \
  --header "Authorization: $FINAM_JWT_TOKEN" | jq
```

### Get Order Book (Depth)

View current market depth with bid/ask levels:

```shell
curl -sL "https://api.finam.ru/v1/instruments/{SYMBOL}/orderbook" \
  --header "Authorization: $FINAM_JWT_TOKEN" | jq
```

### Get Recent Trades

List the most recent executed trades:

```shell
curl -sL "https://api.finam.ru/v1/instruments/{SYMBOL}/trades/latest" \
  --header "Authorization: $FINAM_JWT_TOKEN" | jq
```

### Get Historical Candles (OHLCV)

Retrieve historical price data with specified timeframe:

```shell
curl -sL "https://api.finam.ru/v1/instruments/{SYMBOL}/bars?timeframe={TIMEFRAME}&interval.startTime={START_TIME}&interval.endTime={END_TIME}" \
  --header "Authorization: $FINAM_JWT_TOKEN" | jq
```

**Available Timeframes:**
- `TIME_FRAME_M1`, `M5`, `M15`, `M30` - Minutes (1, 5, 15, 30)
- `TIME_FRAME_H1`, `H2`, `H4`, `H8` - Hours (1, 2, 4, 8)
- `TIME_FRAME_D` - Daily
- `TIME_FRAME_W` - Weekly
- `TIME_FRAME_MN` - Monthly
- `TIME_FRAME_QR` - Quarterly

**Date Format (RFC 3339):**
- Format: `YYYY-MM-DDTHH:MM:SSZ` or `YYYY-MM-DDTHH:MM:SS+HH:MM`
- `startTime` - Inclusive (interval start, included in results)
- `endTime` - Exclusive (interval end, NOT included in results)
- Examples:
  - `2024-01-15T10:30:00Z` (UTC)
  - `2024-01-15T10:30:00+03:00` (Moscow time, UTC+3)


## Order Management

### Place Order

**Order Types:**
- `ORDER_TYPE_MARKET` - Market order (executes immediately, no `limitPrice` required)
- `ORDER_TYPE_LIMIT` - Limit order (requires `limitPrice`)

```shell
curl -sL "https://api.finam.ru/v1/accounts/$FINAM_ACCOUNT_ID/orders" \
  --header "Authorization: $FINAM_JWT_TOKEN" \
  --header "Content-Type: application/json" \
  --data '{
    "symbol": "{SYMBOL}",
    "quantity": {"value": "{QUANTITY}"},
    "side": "{SIDE}",
    "type": "{ORDER_TYPE}",
    "limitPrice": {"value": "{PRICE}"}
  }' | jq
```

**Parameters:**
- `symbol` - Instrument (e.g., `SBER@MISX`)
- `quantity.value` - Number of shares/contracts
- `side` - `SIDE_BUY` or `SIDE_SELL`
- `type` - `ORDER_TYPE_MARKET` or `ORDER_TYPE_LIMIT`
- `limitPrice` - Only for `ORDER_TYPE_LIMIT` (omit for market orders)

### Get Order Status

Check the status of a specific order:

```shell
curl -sL "https://api.finam.ru/v1/accounts/$FINAM_ACCOUNT_ID/orders/{ORDER_ID}" \
  --header "Authorization: $FINAM_JWT_TOKEN" | jq
```

### Cancel Order

Cancel a pending order:

```shell
curl -sL --request DELETE "https://api.finam.ru/v1/accounts/$FINAM_ACCOUNT_ID/orders/{ORDER_ID}" \
  --header "Authorization: $FINAM_JWT_TOKEN" | jq
```

## Reference Data

### List Available Exchanges

View all supported exchanges with their MIC codes:

```shell
jq -r '.exchanges[] | "\(.mic) - \(.name)"' assets/exchanges.json
```

### Get Equities by Exchange

List stocks available on a specific exchange:

```shell
jq -r '.{MIC}[:{LIMIT}] | .[] | "\(.symbol) - \(.name)"' assets/equities.json
```

**Example - Get 10 NYSE stocks:**
```shell
jq -r '.XNYS[:10] | .[] | "\(.symbol) - \(.name)"' assets/equities.json
```

**Example - Get all MISX stocks:**
```shell
jq -r '.MISX[] | "\(.symbol) - \(.name)"' assets/equities.json
```

See [API Reference](assets/openapi.json) for full details.
