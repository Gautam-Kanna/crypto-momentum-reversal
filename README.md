# Statistical Arbitrage in Crypto: Momentum and Reversal

I wanted to see whether momentum and reversal patterns in crypto prices are still profitable after realistic trading costs. I tested a range of signals on 10 large coins and ended up with a multi-horizon cross-sectional momentum strategy. I then checked it on a 2024-2025 holdout period that wasn't used for any decisions.

## Data
4-hour candles (close and dollar volume) for BTC, ETH, BNB, XRP, ADA, DOGE, SOL, DOT, LTC and LINK against USDT, pulled from Binance's public API with `python-binance` (no API key needed). Nov 2020 to Dec 2025, 11,317 bars per coin.

## Approach
- Cross-sectional, dollar-neutral rank portfolios: long the coins with the highest signal, short the lowest
- The portfolio formed at bar *t* earns bar *t+1*'s return, so there's no look-ahead
- 20 bps cost per trade as the base case, with exponential smoothing of the weights to cut turnover
- All choices made on 2021-2023; 2024-2025 kept as an out-of-sample test
- Statistics on daily returns (4h returns compounded), Sharpe annualized with √252, max drawdown from the growth-of-$1 curve, and alpha / beta / alpha t-stat from a regression on BTC

## What I found
- **Horizon:** reversal only at the 4-hour look-back. From 8 hours to 30 days it's momentum (Sharpe up to about 1.7 before costs, about 2.2 when the latest bar is skipped).
- **Volume:** reversal is concentrated in low-volume coins (Sharpe 1.70 vs -0.51 for high-volume coins), which fits the uninformed-trading story. It turns over too much to survive 20 bps, so I kept it as a research finding rather than forcing it into the portfolio.
- **Seasonality:** same-time-of-day momentum was the strongest signal before costs (Sharpe 2.94), but it has to rebuild the whole portfolio every bar.
- **Costs decide everything:** at 20 bps none of the reversal or seasonality versions made money. A slow multi-horizon momentum strategy (ranks averaged over 3/7/14/30 days, 8-day smoothing) did.

## Out-of-sample results (2024-2025)

| | Final strategy (20 bps, base case) | 7 bps (optimistic) | BTC buy & hold |
|---|---|---|---|
| Annual return | 4.7% | 6.8% | 33.3% |
| Volatility | 11.5% | 11.5% | 39.7% |
| Sharpe | 0.41 | 0.59 | 0.84 |
| Max drawdown | -13.1% | -11.7% | -32.0% |
| Beta to BTC | -0.02 | -0.02 | 1.00 |
| Alpha t-stat | 0.82 | 1.13 | - |

The out-of-sample Sharpe (0.41) was in line with training (0.36), the strategy was positive in 4 of 5 years, and it's market-neutral (correlation with BTC -0.08). The alpha is **not** statistically significant, though (t-stat 0.82 out of sample). Five years isn't enough data to separate a 0.4 Sharpe from luck.

The 7 bps column assumes limit orders always fill at the quoted price, which ignores missed fills and adverse selection, so I treat it as a sensitivity test only.

## Running it
Open `Crypto_Momentum_Reversal.ipynb` in Jupyter, run the install cell once, then Run All. The first run downloads the data and saves it to `prices_4h.csv` / `volumes_4h.csv`.

## Limitations and next steps
- Survivorship bias: the coins are ones that are large today. Next step is a point-in-time universe (top coins by trailing volume at each date, including ones later delisted).
- Only 10 coins. Expanding to 20-30+ is the natural robustness test.
- Flat cost assumption, unmodeled limit-order fills, free shorting (in practice via perpetual futures with funding), and fills at the 4h close.
