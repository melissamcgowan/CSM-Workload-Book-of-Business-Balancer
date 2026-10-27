# CSM Workload & Book-of-Business Balancer

An interactive what-if tool for **resegmentations and book rebalancing**. Before anyone is handed a new book, test where accounts can go, see what it does to every CSM's load, and export the plan.

Runs entirely in the browser from a single `index.html`. No build step, no backend. All data is synthetic. Live Demo: https://melissamcgowan.github.io/CSM-Workload-Book-of-Business-Balancer/

## When to use it

- **Resegmentation.** Segment definitions are changing (for example, Enterprise now starts at $180K ARR instead of $250K). See which accounts change segment, whether each pod has the capacity to absorb them, and where the changed accounts should land.
- **Rebalancing.** A few books have crept past capacity while others have room. Get a suggested set of moves, or make them by hand and watch the numbers update.
- **Staffing events.** A CSM leaves, goes on leave, or a new hire is approved. Simulate it and see how the portfolio absorbs the change.

## What you can do

| Tab | Purpose |
|---|---|
| Scenario board | Every CSM's utilization, baseline vs scenario, with at-risk ARR, neglect exposure, pod misfits and region mismatches. |
| Account sandbox | Filter and sort every account, reassign owners, lock accounts that must not move, and click **Where can it go?** to rank every CSM as a destination. |
| Resegmentation | Slide the ARR cut-offs. See segment demand vs pod capacity, a segment shift matrix, and the list of accounts changing segment. One click routes them to the right pod. |
| Staffing events | Simulate a departure or leave, add a hire, edit capacity. Orphaned accounts are flagged until you rebalance. |
| Change log & export | Every move with its reason and a warm-handoff flag on sensitive accounts. Download CSV or JSON, or copy a one-line summary. |

## How the scoring works

**Account load** is a point value that reflects how much work the account really takes, not just that it exists:

```
load = base points for segment x (1 + complexity uplift + risk uplift + renewal uplift)
```

- Base points: Enterprise 10, Mid-Market 5, SMB 2 (adjustable)
- Complexity: +0.15 per level above 1
- Risk: scales up as health drops below 70
- Renewal: +0.25 when the renewal is inside 90 days

**Utilization** is a CSM's total load divided by their capacity points. Defaults: 115% or more is Overloaded, 100% to 115% is Stretched, below 70% is Underused. All adjustable.

**Balance score** runs 0 to 100 and measures how evenly utilization is spread across the team. 100 means everyone is at the same percentage of capacity.

**Neglect exposure** counts at-risk accounts (health under 50, or renewal within 60 days) sitting in a book that is over 105% utilized. These are the accounts most likely to get missed.

**Suggested rebalance** greedily picks the move that most improves balance, minus penalties for:
- breaking pod specialty (segment fit)
- crossing regions
- disrupting sensitive accounts (renewal inside 90 days or health under 50) and large ARR

Locked accounts never move. "Disruption aversion" and "Max account moves" control how conservative the suggestions are.

## Two-layer design

The data layer and the logic layer are separate, so a real export can replace the synthetic data without touching anything else.

Replace `loadData()` in `index.html` with a function that returns:

```js
{
  csms:  [{id, name, pod, region, share /* or cap = capacity points */}],
  accts: [{id, name, arr, health, renew /* days to renewal */, cx /* complexity 1-5 */, region, owner /* csm id */}]
}
```

If you supply `cap` on each CSM, it is used as is. Otherwise, capacity is derived from `share`, so the team sits near 97% in aggregate.

## Run it

Open `index.html` in a browser. 

## Notes

- Synthetic data only. No real customers or employees.
- The model is a decision aid. Capacity points and weights should be calibrated to your team's actual coverage model before using it for real assignments.
