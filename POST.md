# Draft X copy

## Main post

> Saw the $MU overnight chart going around and thought: this should take 10 minutes to actually set up, not a weekend.
>
> So I built it. One paste into Claude and you have an agent that buys at the close and sells at the open, every weekday, on autopilot.
>
> No code. No server. Works on any ticker, any amount.
>
> Repo below 👇

## First reply

> github.com/merjua14/overnighttrader
>
> Steps:
> 1. Open a Robinhood Agentic account (separate from your main, it's the safety rail)
> 2. Add the Robinhood connector to Claude
> 3. Paste one block, fill in ticker + $ amount
>
> That's it. It builds its own scheduled tasks.

## Second reply (do not skip this one)

> One thing I want to be straight about, because the repo is:
>
> That 138,330,342% is a fully-reinvested number. Buying a fixed $1,000/night is a SUM of nightly returns, not a product. Same data, ~+1,415% instead. Compounding is the whole gap.
>
> There's a RISKS.md that shows the math.

## Third reply

> Also worth knowing before you run it:
>
> Alpha Architect tested the S&P version 1993-2020. +717% gross, **-32% after spreads and commissions**. ~250 round trips/yr multiplies every basis point of cost by 250.
>
> Robinhood being commission-free is the only reason this is even arguable. Past ~20 bps per round trip it's negative.

## Fourth reply

> And the one that would've broken it for most people:
>
> Stocks settle T+1. If you open the Agentic account as CASH, Tuesday's sale proceeds aren't spendable until Wednesday, so Tuesday's buy fails or you get a good faith violation.
>
> Pick "limited margin" at signup. Repo tells you where.

## Fifth reply (optional, if the thread has legs)

> It ships ready to trade live, but there's a paper mode in the README that's two edits and gives you a full log of every trade it would've made with real quotes and $0 at risk.
>
> Run that for a week first. Mostly to watch the timing guards survive a weekend and a holiday.

## Notes on the thread

The honest replies are not a hedge, they are the reason the thread is worth posting. Every quant on that timeline already knows the 138 million percent is a compounding artifact, and the ones who do not know will find out in the replies. Getting there first is the difference between "useful build" and "guy who shipped a footgun."

Keep the caveats in replies, not the main post. The main post sells the build. The replies establish that you understand it.
