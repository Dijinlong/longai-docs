# Refunds, top-ups and rate limits

Three things break the arithmetic of a cheap LLM API bill: how you top up, what happens when a call fails, and how the rate limit is actually enforced. All three are answerable before you pay.

## Top-up

Minimum top-up and expiry are the two facts people forget. A channel that sells at 0.6x but requires a 500 CNY minimum with a 90-day expiry is not cheaper than one that takes 20 CNY with no expiry, unless you know you will spend it. The currency you pay in is stated on the [pricing page](https://ai.dijinlong.com/pricing); the [FAQ](https://ai.dijinlong.com/faq) covers minimums and expiry.

## Failure billing

Timeouts, upstream 5xx and empty completions should not be billed at list price. Ask what the channel does when the upstream fails mid-stream: refund, partial charge, or full charge with a retry. A marketplace that bills only successful tokens passes the upstream risk to itself; one that bills attempts passes it to you.

## Measured rate limits

The advertised RPM is a ceiling, not a promise. What you want is the measured 429 rate at your concurrency. The [model reference](https://ai.dijinlong.com/model-docs) lists per-model limits, and the [model plaza](https://ai.dijinlong.com/model-plaza) shows which upstream a model actually routes to, which is what determines the real ceiling.

## Currency and settlement

Display and settlement are separate facts. The [English site](https://ai.dijinlong.com/en) quotes in USD and settles in CNY at a rate stated on the page; the Chinese site quotes in CNY. Mix the two silently and your monthly reconciliation drifts by the exchange rate.

Back to [LONG AI Docs](index.md) or [ai.dijinlong.com](https://ai.dijinlong.com).