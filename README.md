# Token Radar

A token launch radar: it reads a sealed snapshot of every protocol DefiLlama
tracks, finds the ones that still record no token, ranks them by what their
launch would be worth, and shows the board as a static web dashboard with a
farm plan for each protocol.

Built for make.fun's "Build the best token launch radar" bounty
($1,325 in USDC, entries close 14 December 2026).

## How it works

`main.ts` (Node 22, standard library only) reads the snapshot, scores every
pre-token protocol with the radar-2 model in `lib/model.ts`, and writes the
ranked board to `board.json`, the dashboard's data to `data.js`, and the
dossier to `dossier.json`.

```
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

## The model (radar-2)

One function scores every row from the snapshot's own figures; no protocol
name is special-cased:

```
implied = round( tvl_usd x 0.25 x growth x window x breadth )
growth  = 1 + 0.60 x max(change_7d, 0) / 100 - 0.25 x max(-change_7d, 0) / 100
window  = 0.70 + 0.30 x exp(-((age_days - 630) / 450)^2)
breadth = 1 + 0.03 x min(chains - 1, 8)
```

- **Growth is asymmetric.** Gains count at full weight; declines at less than
  half. Momentum moves timing, not existence.
- **The launch window is hump-shaped.** A protocol about a year and a half to
  two years in with no token is at peak launch likelihood: proven enough to
  hold real TVL, still young enough that a token is the obvious next step.
- **Breadth is distribution.** Each extra chain is another set of users a
  token would reach.

The board covers the whole watchlist (40 protocols at or above $50M TVL with
no token) plus every rankable pre-token protocol below that floor — 226 rows
in all. Protocols whose token already trades are set aside with their market
caps, never mixed into the ranking.

## The dashboard

`dashboard/` is a static page with no network: the board, a detail view for
every protocol (model breakdown plus farm plan), search, category and chain
filters, watchlist/early scope, sortable columns, a live-token view, and a
model sandbox that re-ranks the board live from the same formula.

## Files

- `main.ts` — the entrypoint
- `lib/model.ts` — the radar-2 model and the farm plans
- `vendor/makefun-board.ts` — make.fun's dossier library (vendored from the kit)
- `dashboard/` — `index.html`, `app.css`, `app.js`
- `screenshots/` — the board, a detail view, a filtered view

Implied value is a model, not a promise. Airdrops are never guaranteed.
