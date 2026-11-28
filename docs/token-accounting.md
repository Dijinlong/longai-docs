# Token accounting when the upstream does not report usage

*Snapshot: 2026-11-28 UTC+8 · LONG AI Engineering Notes*

A gateway bills on tokens, and the token count is not always something the upstream tells you. This page is about what we do when `usage` is missing, why your number and ours will still disagree slightly, and which of those disagreements are worth chasing.

## Where the number comes from

Three sources, in order of preference:

1. **Upstream `usage` block.** When a provider returns `usage.prompt_tokens` and `usage.completion_tokens` in the final chunk or the response body, we use it verbatim. It is the vendor's own accounting, and arguing with it is pointless.
2. **Gateway tokenizer.** When `usage` is absent, we tokenize the request and the response ourselves with the tokenizer family the model uses. This is an estimate, and it is labelled as one.
3. **Character heuristic.** Last resort for models whose tokenizer we do not ship: roughly one token per four characters for Latin text, adjusted upward for CJK. Only used for monitoring, never for invoicing.

The rule that keeps this honest: **every usage record says which source it came from.** `"source": "upstream"` or `"source": "estimated"` is one field, and it is the difference between a support ticket and a five-second answer.

## Why your count and ours differ

Even with upstream usage, three systematic differences are normal:

- **System prompt and tool definitions.** They are part of the prompt tokens, but not part of what you typed. A tool-heavy request can carry several hundred tokens before your first message.
- **Tokenizer family.** The same Chinese sentence is a different number of tokens under different tokenizers. Comparing your local count to ours across models is comparing two tokenizers, not two bills.
- **Truncation.** A request cut at `max_tokens` still bills for the tokens that were generated, which may equal the limit even though the answer looks short.

If the gap is under a few percent and only appears on long prompts, it is almost certainly the first cause. If the gap scales with output length, look at the third.

## Streaming: what counts as generated

A cancelled stream is the interesting case. If a client disconnects after 40 deltas, the upstream has already generated those tokens, and most providers bill for them. Our behaviour:

- tokens received and forwarded before the disconnect are billed
- the upstream's final `usage`, when it arrives despite the disconnect, wins
- when it does not arrive, we bill the tokens we actually forwarded and mark the record `estimated` with `"cancelled": true`

That last flag matters because it is the first thing to check when a customer says "we cancelled everything and the bill went up": cancelled requests are usually cheap, not free.

## Making the number auditable

Each response carries a stable `request_id`, and the usage record is retained for the billing period. A customer who wants to understand a specific line item can ask about one id instead of one month, and the answer includes the source field, the model, the upstream seller, and whether the request was retried.

The public-facing description of what is billed, including refunds and top-ups, is in the [FAQ](https://ai.dijinlong.com/faq); the per-model prices that the counts are multiplied by are on the [price table](https://ai.dijinlong.com/pricing).
