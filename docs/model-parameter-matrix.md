# Parameter matrix: what each model actually accepts

Two models at the same price per million tokens can differ on everything that matters at runtime: context window, streaming, tool calling, structured output, vision, and whether the provider bills cached input differently.

- Context length sets your worst-case cost per call. A 128k model at a low unit price can still cost more per request than a 32k model at a higher one.
- Streaming support decides whether you can build a chat UI at all, and whether a long generation can be cancelled to save tokens.
- Tool calling and JSON mode decide whether you need a wrapper layer in your own code.
- Vision decides whether an image pipeline is possible on that route.
- Cached input pricing decides the cost of long system prompts, which is where most of the bill usually hides.

The [model reference](https://ai.dijinlong.com/model-docs) keeps a per-model row for each of these. The [model plaza](https://ai.dijinlong.com/model-plaza) carries the price and the upstream channel on the same row, so parameter support and cost can be read together instead of from two different pages.

## How to use the matrix

1. Filter by the capability you need (tool calling, vision, JSON mode).
2. Sort by unit price on that filtered set, not across the whole catalogue.
3. Check the upstream on the row: the same model on a first-party channel and through a reseller chain will behave differently under load.
4. Re-test monthly. Parameter support changes without a price change.

Start at [ai.dijinlong.com](https://ai.dijinlong.com) or read the [pricing table](https://ai.dijinlong.com/pricing).