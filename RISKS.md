# What that 138,330,342% actually is

The chart is not fake. The arithmetic is real and the effect it points at is real and well documented in academic finance. But the number on the right edge is not a number anyone could have earned, and the gap is not small. It is roughly five orders of magnitude.

Here is where it goes, in order of how much it costs you.

---

## 1. The number assumes you reinvested everything. A fixed $1,000 a night does not.

This is the biggest single gap and it has nothing to do with fees or slippage. It is just compounding.

That +138,330,342% is a **fully reinvested** figure. Every night's winnings go back in the next night. It is a geometric product of roughly 9,000 nightly returns.

Buying **$1,000 every night** is a fixed stake. You sell each morning and put the same $1,000 back in that afternoon. Your profit is the **sum** of those nightly returns, not the product.

Working backwards from the chart, the average nightly return implied is about **0.157%**. Over ~9,000 nights:

| | Result on the strategy |
|---|---|
| Reinvest everything (the chart) | +138,330,342% |
| Fixed $1,000 every night (the plan) | **about +1,415%**, so roughly **$14,000 of total profit** |

Both come from the identical set of nightly returns. Compounding is the entire difference.

Now, $14,000 of profit on $1,000 at risk over 36 years is about **$393 a year on $1,000**, which is still a ~39% annual return on capital deployed, and that would be extraordinary if you kept it. You will not keep most of it. Keep reading.

## 2. You cannot trade at the closing price or the opening price

Backtests use the official OHLC open and close. Those are not prices you can transact at.

The **close** is roughly fine. The official closing price comes out of the closing auction, and a market-on-close order genuinely gets it.

The **open** is the problem. The reported opening price is **the first trade of the day**, not the opening auction print, and it often happens on tiny volume at a price nobody could get size at. QuantPedia tested exactly this on GDX:

| Execution assumption | Annualized overnight return |
|---|---|
| Official OHLC open (what backtests use) | **30%** |
| Actual fill at 9:31am | **8.58%** |

Same strategy, same data, **71% of the return was an artifact of the price stamp.** This repo sells at 9:31 to 9:45 rather than 9:30:00 precisely because the 9:30:00 print is not real, and you should understand that you are giving up part of the theoretical edge on purpose, because that part of the edge was never there.

## 3. Trading costs, multiplied by 250 round trips a year

This is what actually kills it, and it is arithmetic you can do yourself.

You are making about **250 round trips a year**. Whatever each round trip costs you in bid-ask spread and slippage gets multiplied by 250.

Against that ~$390 a year of gross profit per $1,000:

| Cost per round trip | Annual drag on $1,000 | Net profit left |
|---|---|---|
| 2 bps | $50 | $343 |
| 5 bps | $125 | $268 |
| 10 bps | $250 | $143 |
| **15 bps** | **$375** | **$18** |
| 20 bps | $500 | **negative** |

15 basis points per round trip is 0.15%. That is **not a lot**. On a $918 stock that is about $1.38 of round-trip cost. Half a penny of spread on each side plus a little slippage at the open gets you most of the way there.

Robinhood is commission-free, which is the only reason this is even arguable. Add a penny a share of commission and it is over immediately. Alpha Architect ran the S&P 500 version of this from 1993 to 2020 with realistic spreads and $0.01/share:

| S&P 500 overnight strategy, 1993-2020 | Cumulative return |
|---|---|
| Gross, before costs | **+717%** |
| **After spreads and $0.01/share** | **-32%** |

That is a 749 percentage point swing from trading costs alone, on the same set of trades. The gross edge was real. It was also entirely consumed. Their paired t-test on the daily overnight-vs-intraday difference came back **not statistically distinguishable from zero**, p = 0.06.

(Their paper also reports buy-and-hold and intraday-only figures for the same window. I am leaving them out because I could not get them to reconcile against the decomposition identity in section 4, and I would rather quote the two numbers that carry the argument than four I cannot check. Read the source if you want the full table.)

The lesson is not "the anomaly is fake." It is **"the anomaly is smaller than the cost of harvesting it 250 times a year."**

## 4. Buy and hold got most of it with one trade

Run the chart's own numbers. Overnight multiple times intraday multiple has to equal buy and hold:

```
1,383,304x  x  0.0008  =  about 1,107x
```

So **MU buy and hold over that window was about +110,000%**, from two trades, zero slippage, zero wash sales, and long-term capital gains treatment.

The overnight strategy's paper advantage over buy and hold is real on the page. But you are paying for it with ~18,000 executions, ~9,000 taxable short-term events, and the requirement that your fills be nearly free. Buy and hold demanded none of that.

---

## Things that will bite you specifically

**Every gain is short-term.** ~250 round trips a year means every dollar of profit is taxed as ordinary income, not long-term capital gains. If you are in a 32% bracket, that alone takes about a third of whatever is left after the table above.

**Wash sales, constantly.** Buying the same ticker within 30 days of a loss on it, roughly every single day, generates a permanent tangle of wash-sale adjustments. It is not a tax dodge and it does not cost you money in the end, but your 1099-B will be a horror show and your broker's cost-basis reporting may confuse you badly.

**Earnings.** MU is a semiconductor stock. It reports quarterly, after the close. This strategy is, four times a year, **specifically holding a volatile semi through its earnings print overnight**, which is the single highest-variance thing you can do with it. Note today's move as a live example: MU went from $966.78 to $918.63, down about 5% in one session. An overnight gap of that size against you erases many months of accumulated edge in one night. If you do not want that exposure, pause the tasks around earnings dates.

**The overnight effect is strongest where it is least harvestable.** In the literature, overnight drift concentrates in smaller, less liquid, wider-spread stocks. That is the same axis along which trading costs rise. If you go hunting for a ticker with a prettier backtest than MU, you will systematically find names where the costs in section 3 are larger. The good-looking backtest and the bad execution are the same phenomenon.

**About one hold in five is a weekend.** A Friday close to Monday open hold carries roughly three calendar days of risk, not one, and the Wednesday before Thanksgiving carries into Friday. The strategy is described as an overnight hold, and 20% of the time it is not. Same edge, more calendar exposure.

**Nothing here is diversified.** One ticker, one direction, every night.

**The automation can fail in an asymmetric way.** A buy that fires without its matching sell leaves you long overnight and into the next day. Guards are in place for that, but a missed sell is the failure mode to watch for, and it is the one that can actually hurt.

---

## So should you run it?

As an experiment, with money you are fine losing, to learn how agentic trading infrastructure behaves: sure, that is a good reason, and it is the reason the original poster gave.

As a way to make money because the chart says 138 million percent: no. The 138 million percent is a compounding artifact of a fixed-stake plan, measured at prices that were not tradable, before costs that get charged 250 times a year, against a benchmark that would have handed you 1,107x for doing nothing.

If it makes money for you, the table in section 3 is the honest range: somewhere between a few hundred basis points and a few dozen percent a year on the capital you deploy, entirely determined by your round-trip execution cost, and negative if that cost reaches about 20 bps. Whatever is left is taxed as ordinary income. And it will take years of data before you can tell any of it apart from luck. Size it like that.

## Sources

- [Trading Costs Wipe Out the Overnight Return Anomaly](https://alphaarchitect.com/trading-costs-wipe-out-the-overnight-return-anomaly/) - Alpha Architect. The +717% gross to -32% net result.
- [Dangers of Relying on OHLC Prices, the Case of Overnight Drift in GDX](https://quantpedia.com/dangers-of-relying-on-ohlc-prices-the-case-of-overnight-drift-in-gdx-etf/) - QuantPedia. The 30% to 8.58% result.
- [Night Moves: Is the Overnight Drift the Grandmother of All Market Anomalies?](https://elmwealth.com/night-moves-overnight-drift/) - Elm Wealth. Good overview of why the effect exists at all.
- [Night trading: Lower risk but higher returns?](https://onlinelibrary.wiley.com/doi/full/10.1002/rfe.1180) - Lachance, Review of Financial Economics 2023.
- [Settlement and buying power](https://robinhood.com/us/en/support/articles/settlement-and-buying-power) - Robinhood. T+1 and the cash vs margin distinction.
