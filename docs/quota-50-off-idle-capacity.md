# Why a marketplace can sell official-price quota at 50% off

*Snapshot: 2026-11-25 17:46 UTC+8 · LONG AI engineering notes · one OpenAI-compatible key across 122 live models from 63 upstream sellers.*

The cheapest text listing on the marketplace is at 0.45 折 today, and the most expensive listing is pinned at 4–5 折, the rule ceiling. Both numbers sit on the same page, in the same snapshot, for models that cost ¥0.669 and ¥66.95 respectively upstream. The question worth answering is not "who is cheap" but "why can anyone sell official-price quota at half price at all". The answer is idle capacity, and it changes how you should buy.

## The 50% ceiling is a rule, not a market price

The platform caps a seller quote at half the vendor's official price (5 折) and puts no floor underneath. That ceiling is a policy against list-price undercutting games, not a clearing price discovered by supply and demand. What it produces is predictable: sellers who want volume go as low as they can, sellers who want to keep a listing alive park at the ceiling.

In the 2026-11-25 17:46 snapshot there are 122 live models, 63 vendors, and 122 listings with a computable in-site price — a 100% coverage rate. Average discount is 1.43 折, median 0.5 折. The median being far below the mean is the fingerprint of a distribution with a hard cap and a deep floor.

## Where the discount comes from: prepaid quota with a clock

Most sellers are not reselling per-token purchases. They hold prepaid or volume-committed quota bought at or near official price, often with a monthly or contractual expiry. Once that quota is paid for, the marginal cost of serving one more request is close to zero — but only until the clock runs out, after which unused quota is worth nothing.

That asymmetry sets the price. Early in a cycle a seller has little reason to discount deeply: the quota will probably be used. Late in the cycle, unsold quota is a sunk cost, and any revenue above zero beats expiry. This is why the deepest discounts cluster late in a billing period, and why the same model can be at 1 折 on the 3rd and 0.3 折 on the 28th.

## What the distribution looks like right now

| Discount bucket (deepest quote) | Models | Share | Typical seller stance |
| --- | --- | --- | --- |
| 0.1–0.5 折 (1–5% of official) | 70 | 57.4% | Clearing idle quota before it expires |
| 0.5–1 折 (5–10%) | 21 | 17.2% | Still moving volume, small margin kept |
| 1–2 折 (10–20%) | 5 | 4.1% | Expensive upstream, thin headroom |
| 2–3 折 (20–30%) | 4 | 3.3% | Hesitant: not cheap enough to win, not safe enough to hold |
| 3–4 折 (30–40%) | 3 | 2.5% | Listed to be seen, rarely chosen |
| 4–5 折 (40–50%) | 18 | 14.8% | Parked at the rule ceiling, near official |

57.4% of models already sit at or below 0.5 折, so a half-price quote is above average on this marketplace. The deepest listing in the snapshot is GPT-6 Astra: official ¥66.95 per million input tokens, in-site ¥0.469, which works out to 0.07 折 — 0.7% of official price.

## Why the middle of the range is empty

1–4 折 holds only 12 models, 9.9% of the catalogue, while both ends together hold 72.2%. The middle is economically unstable. A seller at 2 折 is not cheap enough to win a price-driven order against a 0.5 折 neighbour, and not expensive enough to earn anything meaningful if the quota is already prepaid. There is no reason to stand there: if you are clearing idle capacity, go to the floor; if you are protecting margin, sit at the ceiling and wait for a buyer who needs that specific model.

This is also why "average discount" is a poor buying signal. The mean is dragged upward by the 18 models parked at the ceiling; the median tells you what a typical order actually pays. Both numbers are on the [model plaza](https://ai.dijinlong.com/model-plaza), which lists each model with its current deepest quote.

## The stability trade-off: idle capacity is idle until it isn't

Deep discounts carry a cost the price column does not show. A seller clearing month-end quota may stop serving the model entirely next cycle, or serve it more slowly under load. That is not malice; it is the same inventory logic that produced the discount. So the money saved on a 0.5 折 quote can be spent twice if production traffic depends on it and the listing goes quiet.

Three practical rules follow. First, split traffic: put latency-tolerant batch work on the deepest quotes and keep interactive paths on listings that have been continuously available. Second, re-check the quote before you scale rather than after; a [live price table](https://ai.dijinlong.com/pricing) is the only source that reflects a seller changing stance mid-cycle. Third, watch the unit as carefully as the discount: 93 text models here are priced per million input tokens, but 2 text models are priced per second and the 15 video models are priced per second as well — a deep discount on a per-second listing is not comparable to a per-token one without converting first.

## How to check a marketplace before you commit

Three checks, all doable from the outside. Is the quote expressed against a stated official price, so you can compute the ratio yourself? Is the deepest quote per model shown, or only a headline "up to" number? And is there a dated snapshot you can point at later, so that a dispute about "it was cheaper last week" has an answer? The [integration docs](https://ai.dijinlong.com/docs) cover how billing and quota are reported per request, which matters more than the discount once you are actually running traffic.

Idle capacity is a real source of cheap tokens, and the 50% ceiling is a real constraint that keeps the market from turning into a race to zero. Buying well means treating a deep quote as inventory with an expiry, not as a permanent price.

*Snapshot: 2026-11-25 17:46 UTC+8 · 122 live models · 63 vendors · average 1.43 折, median 0.5 折 · USD converted at 6.695 CNY. Prices move; re-check before ordering.*


---

[Live pricing](https://ai.dijinlong.com/pricing) · [Model plaza](https://ai.dijinlong.com/model-plaza) · [Integration docs](https://ai.dijinlong.com/docs)
