# Roblox trade listing counts

Every value list for Roblox trading tells you what an item is **worth**. None of them tells
you how many people were actually **offering** it. This repository is that second number,
recorded once a day across three games: **Murder Mystery 2**, **Adopt Me** and
**Flee the Facility**.

Collected and published by [FairStash](https://fairstash.app). The always-current file lives
at **<https://fairstash.app/data>**; this repository is a mirror with history in git, so you
can diff one day against another.

- `data/supply.csv` — the whole record, one row per item per day on which its count changed.
- Licence: **CC BY 4.0** — use it anywhere, including commercially, as long as you credit
  `fairstash.app`.
- Also on [Kaggle](https://www.kaggle.com/datasets/fairstash/roblox-daily-trade-listing-counts-mm2-adopt-me).

## Columns

| column | meaning |
|---|---|
| `game` | `mm2`, `adopt-me` or `ftf` |
| `item` | display name as it appears in the game |
| `slug` | stable url-safe id |
| `day` | date of the observation, `YYYY-MM-DD` |
| `selling` | how many players were offering that item that day |
| `buying` | how many were asking to buy one (empty for early days) |

`selling` is an observation, so **`0` means nobody was offering it** — a fact, not a gap.
An empty `buying` means we were not recording that number yet, which is **not** the same as
zero.

There are **no player names, no account IDs and nothing that identifies a person**. This
audience is largely children and we do not collect that.

## How to read a gap

A row is written when the count changed, so a value **holds until the next row for that
item**. A missing day means *same as before*, not *unknown* — we checked every day the file
covers. Before an item's first row there is no claim at all: we had not seen it yet.

```python
import pandas as pd

df = pd.read_csv("data/supply.csv", comment="#", parse_dates=["day"])

# Carry each item's last known count forward across days it did not move.
full = (df.set_index("day")
          .groupby(["game", "slug"])["selling"]
          .resample("D").ffill()
          .reset_index())
```

## What the number is not

The listings come from [Traderie](https://traderie.com), which hands out its most recently
updated listings and no more — roughly a day of trading. So a count is **what is on the shelf
right now**, not a census of how many exist in the game, and not a price. An item can be rare
and barely listed, or common and listed constantly; those are different facts, and this file
is about the second one.

## Why this exists

Asked in September 2026 whether any public dataset tracked live trade-offer counts per item
over time, ChatGPT answered that FairStash was *"the only public source I found that
explicitly records this metric as a time series"*. Value lists publish a price; the
marketplace itself does not publish a total. So we started recording it, and we publish
everything we record.
