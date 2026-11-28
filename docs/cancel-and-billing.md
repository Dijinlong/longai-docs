# Cancelling a stream: what gets billed and what does not

*Snapshot: 2026-11-28 UTC+8 · LONG AI Engineering Notes*

Cancellation looks like a client-side concern and turns out to be a billing and reliability concern. This page covers what happens on our side when a caller aborts a streaming response, and how to cancel in a way that costs you the least.

## Three ways a stream ends early

| How it ends | Who closed it | What the upstream sees | What is billed |
|---|---|---|---|
| Client abort | caller closes the connection | context cancelled, generation stopped | tokens already generated |
| Client timeout | caller's HTTP library gives up | same as abort | tokens already generated |
| Upstream error mid-stream | provider drops the stream | vendor-side failure | tokens generated before the drop |

In all three cases the request has already been paid for upstream. The only way to avoid paying for generation is to not ask for it.

## Cancel early, and cancel locally

If the user closed the tab, the money is already spent. What is still avoidable is the *tail*: a generation that continues to 4,000 tokens after nobody is listening.

- Abort the HTTP request as soon as the consumer stops reading. An SDK that keeps a stream open "just in case" keeps paying.
- Do not retry a cancelled request automatically. A retry doubles the bill for an answer nobody wanted.
- For long generations, set `max_tokens` to the length you would actually display. It is a budget, not a suggestion.

```python
with requests.get(url, stream=True, timeout=(10, 60)) as r:
    for line in r.iter_lines():
        if stop_requested:
            r.close()          # abort: stops upstream generation
            break
        handle(line)
```

## The `[DONE]` frame is not a receipt

Clients often treat `data: [DONE]` as "everything succeeded". It means "the generator finished its turn". A stream can end with `[DONE]` and still be truncated by a token limit, and it can end without `[DONE]` because the connection died after a complete answer. Check `finish_reason` on the last chunk, not the sentinel.

## Why cancelled requests still show up

Billing systems record what was generated, not what was consumed. A cancelled request therefore appears as a small line item with `cancelled: true`. That is not an error - it is the honest record. The only genuine anomaly is a cancelled request billed at the full `max_tokens`, which usually means the cancellation never reached the upstream and generation ran to completion.

If you see that pattern repeatedly, the abort is happening at the wrong layer: closing a UI component without closing the socket.

## What to monitor

- **cancel rate** per client, per model: a sudden rise is usually a UI regression, not a traffic change
- **cancelled-but-full-length** count: should be near zero
- **time to abort** after first token: a proxy for how quickly the client reacts

The per-model status and incident history for these signals are published on the [status page](https://ai.dijinlong.com/status). Definitions for refunds and partial charges are in the [FAQ](https://ai.dijinlong.com/faq), and the price each token multiplies against is on the [price table](https://ai.dijinlong.com/pricing).
