# Token Radar

[![make.fun](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/Herman-234/token-radar/rank.svg)](https://make.fun/bounty/token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/Herman-234/token-radar/reward.svg)](https://make.fun/bounty/token-radar-01)

Entry for the make.fun bounty: https://make.fun/bounty/token-radar-01

A radar for protocols that record no token yet, ranked by what their launch would be
worth, each with the steps to farm it.

`main.ts` reads the sealed DefiLlama snapshot (`input/snapshot.json`, frozen 14 Nov 2026),
scores the whole clean pre-token universe with one model, and writes `board.json`,
`data.js`, `dashboard/` and `dossier.json` for the make.fun token-radar-01 bounty.

## The board

- **198 rows**: the 40 protocols on the house watchlist, plus 158 more clean pre-token
  protocols the watchlist floor leaves out (138 below the $50M floor, 20 above it that the
  house list left out for reasons the snapshot does not carry, such as a parent protocol).
  The watchlist is the floor, never the ceiling.
- A protocol is on the board when it records no token (no market cap, no CoinGecko id, no
  symbol) and is not a wrapped, staked or bridged product of a token that already trades:
  not a Bridge / Canonical Bridge / Liquid Staking / Liquid Restaking / Restaked BTC / CEX
  category, no version marker in its name, no whole word wrapped / staked / staking, and
  no exchange's name at the start.
- Live-token protocols are kept apart in the dashboard's Live token view, shown with their
  real market caps, never ranked as if they were pre-token.

## The model (radar-2)

One function scores every row, from the snapshot's own figures only:

```
implied = round( tvl_usd x 0.25
  x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
  x (0.6 + min(age_days, 1095) / 1095 x 0.4)
  x (1 + 0.02 x min(chains, 15))
  x category )
```

- **TVL x 0.25** — the size of the launch: a token priced at roughly a quarter of TVL.
- **Seven-day growth** — its direction, clamped to -60%..+120%, half-weighted.
- **Age** — how long the protocol has run with no token, the best record of a launch
  getting closer; an unlisted age uses 365 days.
- **Chain breadth** — 2% per chain, capped at 15 chains: a protocol whose value reaches
  more ecosystems is worth more.
- **Category** — a small, stated tilt: 1.10 for token-native categories (Dexs, Lending,
  Derivatives, Yield, RWA, Prediction Market, Yield Aggregator, CDP, Basis Trading,
  Leveraged Farming), 1.00 for everything else.

No protocol name is special-cased. The dashboard recomputes every row from the same
figures and shows whether its number matches `board.json`.

## Farm plans

Each row carries six steps derived from its own record (category, chains, TVL, age):
the action, the capital, the hold time, and the cost and time of each step. Nothing here
promises an airdrop: a protocol records no token, and the plan is what to do if one ever
launches. No step ever asks for payment to claim anything.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json
```

Node 22.18 or later, standard library only, no install. The run writes `out/board.json`,
`out/data.js`, `out/dashboard/`, `out/screenshots/` and `out/dossier.json`. Run it twice:
both runs write the same bytes.

## The dashboard

A static page, no network: `dashboard/index.html` + `app.css` + `app.js` render
`data.js` beside the page. It has a top-five spotlight, search, category and chain
filters, a watchlist-only toggle, a sortable table, a detail view for every protocol
(model breakdown + farm plan), the live-token set apart, the formula in full, and an
honest-limits section.

## Honest limits

- The snapshot is frozen at 14 Nov 2026: a protocol that launched a token after that
  date still shows here as pre-token.
- "No token" means no token on the snapshot's record, never proof that none exists.
- Implied value is an estimate from TVL, growth, age, chain breadth and category. It is
  not a price, not a prediction, and not a promise that any token launches or any
  airdrop pays.

## Layout

```
main.ts                  entrypoint: snapshot in, board + dashboard + dossier out
lib/model.ts             the model, the farm plans, the snapshot parser, the gate checks
vendor/makefun-board.ts  make.fun's dossier library, byte for byte the kit's copy
dashboard/               the static page (index.html, app.css, app.js)
screenshots/             the board, a detail view and a filtered view of the dashboard
LICENSE                  MIT
```
