# 7 checks before you buy LLM API credits from a third-party marketplace

Third-party LLM API marketplaces all advertise one number: price per million tokens. That number is real, but it is the smallest part of what you are actually buying. Here are the seven things worth checking before you top up an account — each one is a question you can answer from a public page in under a minute, and each one has cost somebody a month of budget.

## 1. Is the price per unit, or per "call"?

Text models bill per million tokens, split into input and output at different rates. Image models bill per picture and the resolution tier changes the unit price. Video bills per second of output. If a listing shows one number with no unit, assume it is the input price of the cheapest tier — and check the actual table before you plan a workload.

## 2. Is the discount applied to input, output, or both?

A flat "50% off official price" is normal in this market: sellers list idle quota and compete on the discount. A discount that only touches input pricing looks identical on the headline and is worth much less in practice, because output tokens usually cost several times more than input. Multiply your expected output volume by the output rate, not by the headline.

## 3. Can you see which upstream channel serves each model?

Two listings with the same price can behave very differently if one routes to a first-party channel and the other through a chain of resellers. Look for a per-model page that names the upstream, the billing unit, and the seller quote — not a sitewide average. On [the LONG AI model plaza](https://ai.dijinlong.com/model-plaza) every model row carries its own quotes and the channel behind them, and [the model reference](https://ai.dijinlong.com/model-docs) archives billing units and tiers per model.

## 4. What happens when a request fails — are you billed?

Failed upstream requests should not appear on your invoice. Retries you issue yourself are a different story: a client that silently retries three times made three billable requests. Check the failure policy in writing, then check your own retry settings.

## 5. What is the 429 rate, measured rather than promised?

Rate limiting is the tax nobody budgets for. A marketplace that publishes its own rate-limit and failover behaviour is telling you something useful: on LONG AI the share of requests rejected with 429 fell from 15.98% to 2.74% after the scheduler started spreading load across 51 upstream sellers instead of retrying the same channel. Ask any provider for that number. Most cannot produce it.

## 6. Which currency is displayed, and which one settles?

Display and settlement are separate facts. The [English site](https://ai.dijinlong.com/en) quotes in USD and settles in CNY with the rate stated on the page; the Chinese site quotes in CNY. If the two are mixed silently, your monthly reconciliation will drift by the exchange rate and you will blame the invoice.

## 7. What does the price page promise about availability?

Idle capacity is the product, so availability moves. A page that shows live seller quotes and says "prices and stock change live, the in-site table is authoritative" is being honest; a static PDF price list that has not changed in six months is not describing this market. Budget for a routing fallback, not for a fixed rate.

## The short version

Unit, discount scope, upstream transparency, failure billing, measured 429 rate, settlement currency, and availability policy. Seven questions, all answerable before you pay, all of them cheaper than the ticket you would otherwise open in month two. The live table behind [LONG AI pricing](https://ai.dijinlong.com/pricing) shows the same seven fields per model, and the [FAQ](https://ai.dijinlong.com/faq) covers the edge cases.

## Where to check the numbers yourself

The catalogue behind every number above is public:

- [Model plaza](https://ai.dijinlong.com/model-plaza) — 100+ models, per-model price and upstream seller
- [Model docs](https://ai.dijinlong.com/model-docs) — parameter support and compatibility notes for each model
- [Pricing](https://ai.dijinlong.com/pricing) — what you actually pay per million tokens, USD and CNY
- [FAQ](https://ai.dijinlong.com/faq) — top-up, refund and rate-limit questions
- [English site](https://ai.dijinlong.com/en) — the same catalogue in English
