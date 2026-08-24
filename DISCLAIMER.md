# Disclaimer

This is not investment advice. I am not your financial advisor, and nothing in this repo is a recommendation to buy or sell any security.

This repository automates a trading strategy that places **real orders with real money** in **your** brokerage account. You are solely responsible for every trade it makes, including trades it makes incorrectly, at the wrong time, in the wrong size, or not at all.

Specific things you are accepting:

- **You can lose money.** Concentrated single-ticker overnight exposure can and will produce losing nights, losing months, and overnight gaps that no stop can protect you from, because the position is held while the market is closed.
- **The historical chart that inspired this is not a projection.** [RISKS.md](RISKS.md) explains in detail why the advertised return is not attainable. Read it.
- **Automation fails.** Scheduled tasks can miss, connectors can disconnect, brokers can have outages, and an AI agent can misread a situation. A buy without its matching sell leaves you holding risk the strategy did not intend.
- **Tax consequences are real and unfavorable.** Roughly 250 round trips a year means short-term ordinary-income treatment and continuous wash sales. Talk to a tax professional.
- **Verify before you trust.** Watch the first several cycles yourself. Do not fund this with meaningful money before you have seen it work end to end.

Use money you would be genuinely fine losing entirely. Provided as-is under the MIT License, without warranty of any kind.
