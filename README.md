# Live College Football Trading on Kalshi and Polymarket

Noah Regal · MS Financial Mathematics, NC State (Dec 2026) · October 2026

I built a Python system that prices live college football games and trades thin in-game event contracts on
Kalshi and Polymarket with real money. This page summarizes what it does and what I learned. The code stays
private because it runs a live strategy; I'm happy to walk through it.

## What I built

- **In-game win-probability model.** Logistic model on 2013–2024 play-by-play (score, clock, possession, field
  position, 3rd/4th down, timeouts), anchored to the de-vigged pregame moneyline. Checked against 2025 Kalshi
  prices before any money went in; it beat ESPN's own win-probability model.
- **Exchange clients.** Kalshi (signed API requests) and Polymarket US: market discovery, order books, orders,
  positions and per-venue fee functions, with unit tests.
- **Live collector.** Records 1-second order book snapshots next to live game state from the ESPN feed, with
  game-to-ticker matching, change-only storage and an always-on supervisor.
- **Risk controls.** Per-order, per-game and per-day caps, restart-safe position counts, order reconciliation,
  and a kill switch I can trigger from my phone.

## What the data showed

- **Thin markets underreact to scores.** In thin FCS markets, about half of a touchdown's price move is still
  left a minute later. Prices also sit compressed toward 50¢.
- **Speed alone is not the edge.** The free ESPN scoreboard runs about 33 seconds behind the field, so
  the edge has to come from pricing the game state better.
- **Market making needs a model.** Resting orders blindly in these books lost money. Resting only behind the
  model's price earned about 2¢ per contract at five minutes, net of fees, in a pre-registered out-of-sample
  test across 31 games.
- **Pregame prices were efficient.** Against Kalshi at kickoff my pregame model found no edge after fees,
  so the system trades in-game only.

## Live results

Two live Saturdays so far: about +$1,500 net, reconciled to the cent against both exchanges. It's a small
sample and one trade drove most of the second day, so I read it as evidence that the pipeline works, not proof
of an edge. Two lessons:

- **Trust the rules.** My manual overrides cost about $570 against the system's rules in one day.
- **Check the feed before the quote.** The data feed occasionally served stale or impossible game states,
  so I added sanity checks before any order.

## How I work

Every methodology choice goes in a written decision log with its evidence. Tests are pre-registered before
out-of-sample games, and splits are by season (train through 2024, validate 2025, hold out 2026). Fees and
fills are modeled at displayed depth, with no mid-price fills.

Python · pandas · NumPy · scikit-learn · statsmodels · httpx · pytest
