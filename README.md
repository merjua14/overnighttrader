# overnighttrader

Buy a stock near the close. Sell it near the open. Every weekday, automatically, with no code running on your machine.

This repo is a copy-paste setup for the trade in [this chart](#the-chart-that-started-this): the observation that for many stocks, almost all of the long-run gain happened between the closing bell and the next opening bell.

**One paste and you are done.** No Python, no server, no GitHub Actions, no keys in a config file.

> Read [RISKS.md](RISKS.md) before you run this. The headline number in that chart is not the number you will get, and the reasons why are specific and knowable. I would rather you understand the trade than run it.

---

## What you need

1. A Robinhood account in good standing (existing, individual, funded)
2. A desktop computer for the one-time setup (Robinhood requires desktop for this)
3. Claude with the Robinhood connector added

That is it. Total setup time is about 10 minutes, and most of it is Robinhood's onboarding screens.

---

## Step 1: Open your Robinhood Agentic account

Your agent cannot touch your main account. Robinhood makes you open a separate **Agentic account**, and that is the only account it can trade in. This is a good design and it is your main safety rail.

1. On a desktop, go to [robinhood.com/agentic-trading](https://robinhood.com/us/en/agentic-trading/)
2. Follow the onboarding to open your Agentic account
3. **When it asks for account type, choose "limited margin."** This matters. See the box below.
4. Transfer money into the Agentic account. Start with an amount you would be genuinely fine losing.
5. **Keep this account empty of your chosen ticker.** Do not transfer MU into it, do not buy MU in it by hand, and clean up after any manual test. The sell leg sells the *entire* position in that ticker, whatever its size and wherever it came from.

> ### Do not skip step 3
>
> This is the thing that breaks this strategy for most people, and you will not find out until day two.
>
> Stocks settle **T+1**, one business day. If you open the Agentic account as a **cash account**, the money from Tuesday morning's sale is not available to spend until Wednesday. So Tuesday afternoon's buy has no funds, and the agent either fails or racks up a **good faith violation**. The strategy needs to buy and sell every single day, so a cash account cannot fund it on a single day's cash.
>
> Two fixes:
> - **Choose "limited margin"** when you open the account. Unsettled sale proceeds become instantly tradable, and the cycle works. (This is not leverage. Margin borrowing is not enabled on Agentic accounts, so you cannot trade on borrowed money here.)
> - **Or, if you want a cash account,** fund it with **at least 2x your daily trade size** and it will work, because every buy is then covered by already-settled cash. $1,000 a day means fund it with $2,000 and leave it alone.

## Step 2: Connect Robinhood to Claude

Add the Robinhood MCP connector. The server URL is:

```
https://agent.robinhood.com/mcp/trading
```

**On Claude Pro or Max:** Settings, then Connectors, then "+", then "Add custom connector". Paste the URL. Click Add. Then authenticate to Robinhood when prompted.

**On Claude Team or Enterprise:** only an org Owner can add connectors. Owner goes to Organization settings, then Connectors, then Add, then hover "Custom" and pick "Web", then paste the URL. Everyone else then goes to Settings, Connectors, and clicks Connect.

**On Claude Code:** `claude mcp add --transport http robinhood https://agent.robinhood.com/mcp/trading`

Verify it worked by asking Claude: *"List my Robinhood accounts."* You want to see an account with `agentic_allowed: true`. **Copy that account number.** You need it in the next step.

## Step 3: Paste this

Change the three settings near the top of the block, then paste the whole thing into Claude in a chat where the Robinhood connector is enabled.

**All three settings are live.** `TICKER` and `DOLLARS_PER_NIGHT` are pre-filled with MU at $1,000 a night. If you paste without reading, that is what you get. Change them.

The same block is in [prompts/SETUP.txt](prompts/SETUP.txt) as a plain file if that is easier to copy. The two are identical.

```
Set up my overnight trading agent with these settings:

  TICKER: MU
  DOLLARS_PER_NIGHT: 1000
  ACCOUNT_NUMBER: <paste your agentic_allowed account number here>

Create the scheduled tasks below for me using the create_trigger tool. All of
them are weekday-only. Cron is in UTC.

Because US markets shift with daylight saving time and cron does not, create
FOUR tasks total: two for each leg, one hour apart. Each task's prompt contains
a time guard that makes the wrong one abort harmlessly, so exactly one of each
pair acts on any given day. This is intentional. Do not try to simplify it to
two tasks.

TASK 1 name: "Overnight agent - BUY (A)"   cron: 55 19 * * 1-5
TASK 2 name: "Overnight agent - BUY (B)"   cron: 55 20 * * 1-5
Both use the BUY prompt below, verbatim, with my values substituted in.

TASK 3 name: "Overnight agent - SELL (A)"  cron: 32 13 * * 1-5
TASK 4 name: "Overnight agent - SELL (B)"  cron: 32 14 * * 1-5
Both use the SELL prompt below, verbatim, with my values substituted in.

On all four tasks: set initiation to human_request, set requires_local_device
to false, and set notifications to BOTH push and email. The notifications are
not optional. A failed sell leg is the one failure mode that can actually cost
me money, and it is worthless if it reports into a session I am not watching.

=== BUY PROMPT ===

You are the buy leg of an automated overnight trading strategy. Act without
asking me anything. I will not be present. I have already authorized these
trades; do not request confirmation before placing an order.

Settings: TICKER=<TICKER>, DOLLARS=<DOLLARS_PER_NIGHT>, ACCOUNT=<ACCOUNT_NUMBER>

1. TIME GUARD. Check the current time in America/New_York. If it is not
   between 15:50 and 15:58 on a weekday, stop immediately and report
   "skipped: outside buy window" and nothing else. Do not trade. This guard
   is what makes the duplicate daylight-saving task harmless.

2. MARKET GUARD. Call get_equity_quotes for TICKER. Stop and report
   "skipped: market not open" if either of these is true:
     - the quote's state field is present and is not "active"
     - the quote's last trade timestamp is more than 3 minutes old
   FAIL CLOSED: if the response does not contain a state field and a last
   trade timestamp you can actually read, stop and report "skipped: cannot
   verify market state" and do not trade. Never treat a missing field as
   permission to proceed. This guard is the only thing standing between you
   and trading on a market holiday or after an early close (the day after
   Thanksgiving, Christmas Eve, July 3), so it must fail closed.

3. POSITION GUARD. Call get_equity_positions. If you already hold any
   quantity of TICKER, stop and report "skipped: position already open".
   Never stack a second buy on top of an unsold position.

4. DUPLICATE ORDER GUARD. Call get_equity_orders. If any order for TICKER
   was already placed today, stop and report "skipped: order already placed
   today". A pending or partially filled order is not yet a position, so
   step 3 alone will not catch a re-run or a retried task.

5. Call review_equity_order: account ACCOUNT, symbol TICKER, side buy,
   type market, dollar_amount DOLLARS, market_hours regular_hours.
   If review returns any alert about insufficient buying power, a trading
   halt, or a restricted account, stop and report the alert. Do not place
   the order.

6. LATE GUARD, then place. Re-check the America/New_York time right now. If
   it is 15:58:30 or later, stop and report "skipped: too late to place
   safely" and do not place the order. This matters more than it looks: a
   market order that arrives after 16:00 ET does not fail, it QUEUES for the
   next morning's open, and then the sell leg closes it minutes later. That
   turns the strategy inside out (a same-day round trip with no overnight
   hold at all) and logs a day trade.

   If the time is fine, call place_equity_order with the same parameters as
   step 5 and a fresh UUID as ref_id. Place it. Do not ask me first.

7. Report one line: filled quantity, price, total cost, and the time.

If any step errors, stop and report the error. Never retry a place_equity_order
call with a new ref_id, because you may double your position. Reuse the same
ref_id if and only if the call failed in transport and you do not know whether
it landed.

=== SELL PROMPT ===

You are the sell leg of an automated overnight trading strategy. Act without
asking me anything. I will not be present. I have already authorized these
trades; do not request confirmation before placing an order.

Settings: TICKER=<TICKER>, DOLLARS=<DOLLARS_PER_NIGHT>, ACCOUNT=<ACCOUNT_NUMBER>

1. TIME GUARD. Check the current time in America/New_York. If it is not
   between 09:31 and 09:45 on a weekday, stop immediately and report
   "skipped: outside sell window" and nothing else. Do not trade.

2. POSITION FIRST. Call get_equity_positions. Find TICKER. If there is no
   position, stop and report "nothing to sell". Read the sellable share
   count, which is the quantity NOT under a hold, and truncate it to at most
   6 decimal places. Also read the average cost.

   Check this before you do anything else, because if a position exists it
   needs to be closed, and a market-state check that runs first can strand it.

3. SIZE SANITY CHECK. Multiply the sellable shares by the current price. If
   that value is more than 1.5 x DOLLARS, stop and report "unexpected
   position size, not selling" with the numbers. The buy leg only ever
   creates a position of about DOLLARS. Anything much larger is a position
   this strategy did not create, and selling it would liquidate a holding I
   did not intend to trade.

4. MARKET GUARD. Call get_equity_quotes for TICKER. If the quote's state
   field is present and is not "active", stop and report "skipped: market not
   open". If the last trade timestamp is more than 3 minutes old, retry the
   quote once. If it is still stale AND a position exists AND it is past
   09:35 ET, do NOT silently skip: report "ALERT: stale quote, position still
   open, could not verify market state" so I get the notification, and stop.
   A thinly traded ticker can look closed at 09:32 when it is not, and a
   position left open every morning while the buy leg skips every evening is
   the worst state this strategy can be in.

5. Call review_equity_order: account ACCOUNT, symbol TICKER, side sell,
   type market, quantity = the sellable share count from step 2 truncated to
   6 decimals, market_hours regular_hours. Sell the whole sellable position.
   Never a dollar amount, because a dollar amount would leave a fractional
   remainder behind.

6. Call place_equity_order with the same parameters and a fresh UUID as
   ref_id. Place it. Do not ask me first.

7. Report one line: shares sold, fill price, proceeds, and the gain or loss
   against average cost times shares sold.

If any step errors, stop and report it. Never retry a place_equity_order call
with a new ref_id: you may double-sell, and in an account that cannot short,
the duplicate is likely to be rejected, which hides whether the position
actually closed. Reuse the same ref_id if and only if the call failed in
transport and you do not know whether it landed.

If a sell fails, say so loudly. An unsold position means tonight's buy will be
skipped by its position guard, and the strategy is now holding overnight risk
it was not designed to hold.
```

That is the whole setup. Claude will create the four tasks and confirm.

## Step 4: Watch the first two days

Do not walk away. Check that:

- The buy fired at about 3:55pm ET and filled
- The sell fired at about 9:32am ET the next morning and closed the whole position
- Exactly one of each duplicate pair acted, and the other reported `skipped: outside buy window` or `skipped: outside sell window`
- Your cash did not go negative or trigger a violation notice
- You actually received the push and email notifications

If the buy fills but the sell does not, you are holding a position the strategy did not intend to hold. Fix that before the next cycle.

---

## Changing the ticker or the amount

Ask Claude: *"Update my overnight agent tasks to use NVDA at $500 per night."* It will rewrite the four task prompts in place. Or edit them yourself in your scheduled tasks list.

**A word on other tickers.** MU was chosen because the chart is dramatic, not because it is optimal. The overnight effect is not a property of MU specifically; it shows up broadly across US equities and it is strongest in exactly the places where it is hardest to capture (small, illiquid, wide-spread names). Picking a thinner stock makes the backtest look better and the real result worse. See [RISKS.md](RISKS.md).

## Turning it off

*"Delete my overnight agent scheduled tasks."* Or pause them individually. Do this before any vacation where you will not be watching, and before earnings if you do not want to hold a semiconductor stock overnight through an earnings print.

## Going paper first (recommended, and it costs you nothing)

This ships ready to place real orders. If you want to watch it for a week before risking money, do this to **both** prompts:

**Delete step 6 from the BUY prompt and step 6 from the SELL prompt** (the two steps that call `place_equity_order`), and add this line right after the settings line of each:

```
DRY RUN: there is no order-placing step in this procedure. Run every other
step, including review_equity_order, then report exactly what you would have
placed and stop.
```

Delete the step rather than layering a "do not place" instruction on top of it. An instruction that says "place it, do not ask me first" sitting a few lines below one that says "do not place" is a coin flip, and the losing side of that flip is a real order.

You get a full log of every trade it would have made, with real quotes, and zero dollars at risk. I would do a week of this first, mostly to confirm the timing guards behave the way you expect through a weekend and a holiday.

---

## Why four tasks instead of two

US market hours move twice a year. Cron runs in UTC and does not. 3:55pm in New York is 19:55 UTC in summer and 20:55 UTC in winter, and the daylight saving switch happens mid-March and early November, not on month boundaries, so you cannot express it cleanly in cron.

So instead of trying to be clever, both fire, every day, and the time guard inside each prompt decides which one is real:

| | Task A fires | Task B fires |
|---|---|---|
| **Summer (EDT)** | 15:55 ET, **buys** | 16:55 ET, outside window, skips |
| **Winter (EST)** | 14:55 ET, outside window, skips | 15:55 ET, **buys** |

Same idea for the sell. It is slightly ugly and completely reliable, and you never have to remember to change anything in March.

## The chart that started this

The premise, from @wheelie and widely reposted: buy MU at every close and sell at every open since 1990 and you are up 138,330,342%. Do the opposite, open to close, and you are down 99.92%.

The decomposition is real arithmetic and it is not a trick. Overnight returns really did carry that stock. But the number is not achievable, and [RISKS.md](RISKS.md) walks through exactly where it goes, with the specific studies that measured it. The short version: that figure assumes free trading at prices that were never actually available, and it assumes you reinvested everything, which a fixed $1,000 a night does not do.

## Files

- [prompts/SETUP.txt](prompts/SETUP.txt) - the paste block from Step 3 as a plain file
- [RISKS.md](RISKS.md) - what the number really is, and the four things that eat it
- [SCHEDULE.md](SCHEDULE.md) - the cron reference, holidays, half days
- [DISCLAIMER.md](DISCLAIMER.md) - not advice, you own your trades
- [POST.md](POST.md) - draft copy if you want to share your own results
- [PUSH.md](PUSH.md) - git commands to publish your fork
- [LICENSE](LICENSE) - MIT

## License

MIT. Do what you want. It is your money and your account.
