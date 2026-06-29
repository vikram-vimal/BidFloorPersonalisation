# Problem Definition

Mobile gaming studios are leaving a measurable and recoverable portion of their ad revenue on the table. The root cause is not a lack of ad inventory — it is a lack of intelligence in how that inventory is priced and when it is presented.

This page defines the revenue gap that bid floor personalisation is designed to close.

{% hint style="info" %}
**Prerequisite:** Familiarity with [bid floors](../ads-concept/ad-in-games.md#how-an-auction-works), first-price auctions, and fill rate. See [Core Concepts](../ads-concept/ad-in-games.md) if you need the baseline.
{% endhint %}

---

## At a glance

| | Today's default | The cost |
| --- | --- | --- |
| **Primary goal** | Maximise fill rate | Inventory is priced for availability, not value |
| **Floor strategy** | Static, set low globally or per country | High-value impressions sold at low-value prices |
| **Auction model** | First-price (e.g. AppLovin MAX) | DSPs learn to shade bids below true valuation |
| **Outcome over time** | Stable fill, declining eCPM baseline | Gap between earned and earnable revenue widens |

---

## The core problem

Studios configure mediation with one overriding priority: **fill every ad request**. The default instinct is to set bid floors low enough that virtually every request returns a filled impression.

That approach works for delivery metrics. It fails for revenue optimisation.

A bid floor is not just a price gate — it signals inventory value to demand-side algorithms. When a studio sets a static low floor, fill rate stays high, but DSPs begin shading bids downward in response. Over time, observed eCPM falls. The studio reads that as market softness and lowers floors further to protect fill. DSPs shade even more, and the cycle repeats — eroding the publisher's long-term earning baseline while delivery metrics look healthy.

---

## Where revenue is being lost

The revenue gap is structural, not operational. Three mechanisms drive it.

### 1. Bid shading in first-price auctions

In first-price auction environments (which AppLovin and most major mediation platforms operate), Demand-Side Platforms deliberately bid below their true valuation — a strategy known as **bid shading**.

When DSPs observe consistently low floors from a publisher, their algorithms learn to set even lower bids. The studio's long-term **eCPM baseline erodes** even when fill rate stays high.

### 2. High-value inventory undersold

Inventory value varies by context — geography, time of day, session depth, ad format — but static floors treat every impression the same.

| Context | Market value | Static floor ($0.50) | Dynamic floor ($5–6) |
| --- | --- | --- | --- |
| US user, Friday 9 PM (peak demand) | $8–12 CPM | Captures little of true value | Captures most value; marginal fill reduction |

A low static floor leaves the majority of peak-demand revenue on the table.

### 3. Compounding decay over time

Low floors create a **race to the bottom**:

1. DSPs shade bids in response to low publisher floors.
2. Observed eCPM drops; the studio interprets this as market softness.
3. Floors are lowered further to protect fill rate.
4. The cycle repeats — inventory reputation and earning potential both decline.

Floors must be **actively managed** to maintain publisher inventory reputation and prevent long-term eCPM decay.

---

## What is at stake

| Metric | Risk with static low floors |
| --- | --- |
| **eCPM** | Baseline erodes as DSPs adapt to low price signals |
| **Fill rate** | Stays high — masking the underlying revenue problem |
| **ARPDAU** | Underperforms relative to inventory's true market value |
| **Recoverable revenue** | Measurable gap between what is earned and what is earnable |

{% hint style="warning" %}
High fill rate alone is not a monetisation success metric. A filled impression at $0.50 CPM earns less than an unfilled request would have if the next bid at $6.00 would have won at a $5.00 floor.
{% endhint %}

---

## The opportunity

The revenue left on the table is **recoverable**. The inventory exists; demand exists. What is missing is pricing intelligence — setting the right floor for the right impression at the right moment, rather than one static value for all contexts.

Bid floor personalisation addresses this gap by replacing static configuration with dynamic, context-aware floor setting.

---

## What's next

{% content-ref url="data-requirements.md" %}
[data-requirements.md](data-requirements.md)
{% endcontent-ref %}

{% content-ref url="bid-floor-integration.md" %}
[bid-floor-integration.md](bid-floor-integration.md)
{% endcontent-ref %}

{% content-ref url="bid-floor-risks-and-challenges.md" %}
[bid-floor-risks-and-challenges.md](bid-floor-risks-and-challenges.md)
{% endcontent-ref %}
