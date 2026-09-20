---
name: fund-research
description: Research Tuatara-managed funds on cymetica.com for a client conversation — ranked list, one fund in detail, index history and the measurement rules. Use when an advisor asks about EventTrader, Cymetica, the Tuatara Hedge Fund, an AIB fund, the M&A Ripple or FTA sleeves, or wants fund numbers to brief a client.
---

# Fund research on cymetica.com

The `eventtrader-funds` MCP server (https://cymetica.com/mcp/v1) exposes four
read-only tools with no key. They return exactly what the public page
https://cymetica.com/fund-performance shows, and nothing it withholds.

## Workflow

1. Call `get_fund_methodology` once per session and keep its rules in view.
2. Call `list_funds` (optionally `status: live`) for the ranked roster. Funds
   are ordered best return since inception first; unknown returns last.
3. For a named fund, call `get_fund` with its `symbol` from the roster
   (AIB symbols such as `AIB-ROBOTICS`, or the sleeves `MA-RIPPLE`,
   `FTA-CRYPTOS`, `FTA-STOCKS`, `EVENT-CARD`).
4. For a chart or a trend question, call `get_fund_history` with the symbol.

## How to present the numbers

- Quote every figure **as of** the timestamp in the response. Never present
  it as current.
- `return_since_inception_pct` is the **fund** return. For a short book a
  falling index is a positive return; the sign is already corrected.
- `index_price_usd` is a price-weighted index **level**, not a share price
  or a capital figure. Capital, AUM and allocations are never published;
  give the mandate and deployment as percentages only.
- Status: `live` takes real USDC through https://cymetica.com/fund/apply;
  `paper` trades simulated and takes no investment; `standing` is live but
  not yet allocated. Say the status whenever you name a fund.
- Constituents: short books are aliased and open M&A ripple baskets are
  withheld on purpose. Say "withheld by the platform", never guess names.
- The execution venue is referred to as **Brokerage100**. Do not speculate
  about the underlying broker.
- The Tuatara Hedge Fund is described by the platform as highly
  experimental. This is information, not a recommendation; suitability is
  the advisor's call.

## Links to cite

- Fund performance: https://cymetica.com/fund-performance
- One fund: `profile_url` from `get_fund`
- How it works: https://cymetica.com/tuatara-fund/how-it-works
- Research reports: https://cymetica.com/fund-performance/research
- Apply / invest: https://cymetica.com/fund/apply
