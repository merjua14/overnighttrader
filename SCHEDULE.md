# Schedule reference

## The four cron lines

All in **UTC**, weekdays only (`1-5` is Monday through Friday).

| Task | Cron | Fires at ET (summer/EDT) | Fires at ET (winter/EST) |
|---|---|---|---|
| BUY (A) | `55 19 * * 1-5` | 15:55 **acts** | 14:55 skips |
| BUY (B) | `55 20 * * 1-5` | 16:55 skips | 15:55 **acts** |
| SELL (A) | `32 13 * * 1-5` | 09:32 **acts** | 08:32 skips |
| SELL (B) | `32 14 * * 1-5` | 10:32 skips | 09:32 **acts** |

The prompt's time guard is what makes the skips happen. Buy window is 15:50 to 15:58 ET. Sell window is 09:31 to 09:45 ET. If you change the cron times, change the windows to match or the whole thing will silently stop trading.

## Why 15:55 and not 15:59:59

A market order has to actually reach the exchange before 16:00 ET, and the agent has work to do first: check the clock, pull a quote, check positions, check today's orders, run a review, then place. Five minutes is enough room for all of that.

The buy prompt also re-checks the clock immediately before placing and aborts past 15:58:30. That second check is not paranoia. **A market order that arrives after 16:00 ET does not get rejected, it queues for the next morning's open.** If that happened, the sell leg would find the position at 09:32 and close it minutes later, which gives you a same-day round trip with no overnight hold at all, plus a logged day trade. That is the exact opposite of the strategy, so it is worth a guard.

Note also that a market order at 15:55 gets you the price at 15:55, **not the official closing auction price**. Those are usually close together and occasionally are not.

## Why 09:32 and not 09:30

Because the 9:30:00 price in a backtest is not a real price. See section 2 of [RISKS.md](RISKS.md). Selling into the first few seconds of the open means paying the widest spread of the day. 9:32 is a compromise between capturing the overnight move and not donating the spread.

If you want to test the sensitivity, move the sell window later (say 09:35 to 09:50) and compare over a couple of months. This is the single knob most worth experimenting with.

## Daylight saving dates

US DST starts the second Sunday of March and ends the first Sunday of November. You do not need to do anything on those dates. The duplicate tasks and the time guards handle it, and because both transitions land on a Sunday, no trading day is ever half-shifted. That is the entire reason there are four tasks instead of two.

## Market holidays

The market guard handles these automatically. On a full holiday there is no recent trade to find, the guard sees a stale quote, and both tasks skip. You do not need a holiday calendar.

The guard is written to **fail closed**: if it cannot read the market-state fields at all, it refuses to trade rather than assuming the market is open. That is deliberate, and if you edit the prompt, keep it that way. A guard that fails open is worse than no guard, because you will believe you have one.

## Early closes

These are the days the market closes at **13:00 ET**:

- **July 3**, when July 4 falls on a Tuesday through Friday. (When July 4 falls on a weekend, the observed holiday is a full closure on the adjacent Friday or Monday, and there is no early close at all.)
- **The Friday after Thanksgiving**
- **December 24**, when it falls on a weekday

On those days the buy tasks fire at 15:55 ET, find the market long since closed, and skip. **This is intentional.** You miss up to three overnight holds a year.

If you would rather catch them, add **two** more tasks, `55 16 * * 1-5` and `55 17 * * 1-5`, with the buy window widened to accept 12:50 to 12:58. You need both for the same daylight saving reason as everything else: July 3 is EDT, but the Friday after Thanksgiving and December 24 are EST, so a single task would only ever catch one of the three.

I left this out of the default setup on purpose. Two extra tasks firing 250 times a year each, to catch three trades, is 500 extra chances for something to go wrong against three chances to make a few dollars. Bad trade.

## Verifying it is set up

Ask Claude: *"List my scheduled tasks."* You should see four, all enabled, with next-run times that make sense. After a week, ask for the run history to confirm exactly one of each pair is acting each day.
