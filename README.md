# Statistical Arbitrage in Crypto: Momentum and Reversal

I wanted to see whether momentum and reversal patterns in crypto prices are still profitable after realistic trading costs. I tested a range of signals on 10 large coins, built a market-neutral portfolio from what survived, and checked it on a 2024-2025 holdout period that wasn't used for any decisions.

## Data
4-hour candles (close and dollar volume) for BTC, ETH, BNB, XRP, ADA, DOGE, SOL, DOT, LTC and LINK against USDT, pulled from Binance's public API with `python-binance` (no API key needed). Nov 2020 to Dec 2025, 11,317 bars per coin.

## Approach
- Cross-sectional, dollar-neutral rank portfolios: long the coins with the highest signal, short the lowest
- The portfolio formed at bar *t* earns bar *t+1*'s return, so there's no look-ahead
- 20 bps cost per trade (7 bps for limit orders), with exponential smoothing of the weights to cut turnover
- All choices made on 2021-2023; 2024-2025 kept as an out-of-sample test

## What I found
- **Horizon:** reversal only at the 4-hour look-back. From 8 hours to 30 days it's momentum (Sharpe up to about 2.1 before costs).
- **Volume:** reversal is concentrated in low-volume coins (Sharpe 2.01 vs -0.68 for high-volume coins).
- **Seasonality:** same-time-of-day momentum was the strongest signal before costs (Sharpe 3.39), but it has to rebuild the whole portfolio every bar.
- **Costs:** at 20 bps none of the reversal or seasonality versions made money. A slow multi-horizon momentum strategy (ranks averaged over 3/7/14/30 days, 8-day smoothing) did.
- **Combining:** before costs, reversal and momentum are uncorrelated and a 50/50 mix has a Sharpe around 2.8. After costs, all the weight goes to momentum.

## Out-of-sample results (2024-2025, after costs)

| | Final strategy (20 bps) | Limit orders (7 bps) | BTC buy & hold |
|---|---|---|---|
| Annual return | 6.8% | 9.8% | 48.5% |
| Volatility | 13.1% | 13.1% | 48.2% |
| Sharpe | 0.51 | 0.75 | 1.01 |
| Max drawdown | -13.9% | -12.3% | -40.6% |
| Beta to BTC | -0.02 | -0.02 | 1.00 |

Market-neutral (correlation with BTC -0.06) and profitable in 4 of 5 years. 2022 was the only losing year.

## Running it
Open `Crypto_Momentum_Reversal.ipynb` in Jupyter, run the install cell once, then Run All. The first run downloads the data and saves it to `prices_4h.csv` / `volumes_4h.csv`.

## Limitations
Survivorship bias in the coin list, a flat cost assumption, free shorting (in practice via perpetual futures with funding costs), and fills at the 4h close.
