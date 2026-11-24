# What one prompt costs on 109 models: a same-input price walk-through

> Snapshot: 2026-11-24 19:01 UTC+8. All prices are in-site effective prices, CNY, 1 USD = 6.695 CNY.

The cheapest and the most expensive text model on the marketplace differ by 356x on the same input. That is not a typo, and it is not a marketing claim about "quality tiers" — it is the arithmetic you get when you read 109 live listings and normalise them to one unit. Below is the walk-through, using the price snapshot generated at 2026-11-24 19:01 UTC+8, so that anyone can repeat it later and check whether the shape still holds.

## Step 1: pick one prompt, one unit, and hold everything else still

The only useful comparison is a fixed prompt measured in one unit. Take a 1,200-token input — a modest system prompt plus a two-turn conversation — and compare what it costs per model, using input price per million tokens. Ignore output for the moment; output is where prices spread even further, and we come back to it.

Across the 80 text models that had a clearly stated input price, the lowest in-site price was ¥0.0301 per million tokens and the highest ¥10.71. Multiply both by 0.0012 million tokens and a single 1,200-token prompt costs ¥0.000036 at the cheap end and ¥0.012852 at the expensive end. One prompt is noise either way. The number that matters is the one you multiply by volume: 1,000 such prompts are 1.2 million input tokens, which is ¥0.036 versus ¥12.85. At 1 million prompts the spread is a car.

That is the whole point of this exercise. Per-request costs look uniformly trivial; per-month costs do not. If you only ever evaluate models one call at a time, you will pick a model that is 300x more expensive than necessary and never notice.

## Step 2: separate official price from the discount applied to it

Marketplace listings have two numbers stacked on top of each other: the vendor's official price and the seller's discount. This site caps a seller quote at 50% off official (a "5 折" ceiling) and lets sellers go as deep as they want below that. So the effective price is official × discount, and the same official price can show up at wildly different in-site prices depending on which seller is filling the order.

| Discount bucket | Models | Share | What it usually means |
|---|---|---|
| 0.1–0.5 折 (1–5% of official) | 52 | 47.7% | Seller clearing idle quota; genuine concession |
| 0.5–1 折 (5–10%) | 22 | 20.2% | Concession with a small margin left |
| 1–2 折 (10–20%) | 5 | 4.6% | Rare; usually an expensive upstream |
| 2–3 折 (20–30%) | 8 | 7.3% | The hesitant band |
| 3–4 折 (30–40%) | 3 | 2.8% | Listed, not really selling |
| 4–5 折 (40–50%) | 18 | 16.5% | At the rule ceiling — close to official |

Average discount across the catalogue is 1.62 折, but the median is 0.7 折. The gap between the two tells you the average is being pulled upward by the 18 models parked at the ceiling, while most listings sit near the deepest end. When someone quotes you a catalogue-wide average, ask for the median.

## Step 3: output price decides the bill, not the headline

Input is the cheap half of a text bill. Output tokens normally cost several times more, and a listing that only advertises one number is almost always advertising input. For budgeting, multiply expected output volume by the output rate for the model you actually plan to run, then add input on top. A model with a ¥10.71 input price and a cheap output price can be a better deal than one with a ¥1 input price and an expensive output price — and price pages that hide output make this impossible to see.

Official input prices across 82 quoted text models fall into five bands: 0 models under ¥0.1 per million, 13 between ¥0.1 and ¥1, 22 between ¥1 and ¥5, 29 between ¥5 and ¥20, and 18 above ¥20. That distribution is the reason the same discount rate produces very different absolute costs: 0.5 折 off a ¥66.95 model is still ¥33.48 per million tokens, while 5 折 off a ¥0.669 model is ¥0.33.

## Step 4: run the walk-through yourself in twenty lines

The repeatable version of this exercise is a script that takes official price, discount, and expected monthly volume, and prints a comparable cost. It should never be a spreadsheet you keep by hand, because listings change weekly and a stale sheet quietly becomes fiction.

```python
# -*- coding: utf-8 -*-
USD_CNY = 6.695

def in_site_price(official_cny, discount_tenths):
    """discount_tenths: 0.5 means 0.5 折 = 5% of official"""
    return official_cny * discount_tenths / 10.0

def monthly(official_in, official_out, discount_tenths,
            in_mtok, out_mtok):
    return (in_site_price(official_in, discount_tenths) * in_mtok
            + in_site_price(official_out, discount_tenths) * out_mtok)

# 1,000 prompts of 1,200 input tokens and 800 output tokens
for name, o_in, o_out, d in [
    ("cheap end",   0.669, 2.68,  0.5),
    ("ceiling end", 66.95, 268.0, 5.0),
]:
    print(name, round(monthly(o_in, o_out, d, 1.2, 0.8), 4), "CNY")

```

Two lines of output, one order of magnitude apart on a thousand requests and three orders apart at production volume. Note that the script applies the discount to both input and output; if a listing discounts only input, this model flatters it.

## Step 5: four things to verify before trusting any number

1. The snapshot time. A price without a timestamp is an anecdote. Prices move with seller quotes, so the page you read should say when it was generated — the monthly report states 2026-11-24 19:01 for exactly this reason, and [the live price table](https://ai.dijinlong.com/pricing) is the authority for the minute you order.

1. The unit. Tokens, images, seconds. Text at ¥0.03 per million tokens and video at ¥0.0335 per second are not comparable; convert to cost-per-task first.

1. The tier. Long-context variants, resolution tiers and batch tiers change the unit price for the same model name. [The per-model reference pages](https://ai.dijinlong.com/model-docs) record the billing unit and the tier so a quote can be pinned to one.

1. Whether the deepest quote has stock. The cheapest row is the deepest quote available, not a guaranteed fill. Budget at the second or third tier, and check [the status page](https://ai.dijinlong.com/status) when a model you rely on starts routing elsewhere.

## The conclusion in one sentence

Normalise to cost-per-task, apply the discount to both sides of the price, timestamp your snapshot, and re-run the script monthly — do that and the 356x spread stops being trivia and starts being the difference between a ¥0.04 and a ¥12.85 line on your bill.
