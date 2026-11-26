# 429s are a scheduling problem, not a capacity problem

*Snapshot: 2026-11-26 19:02 UTC+8 · LONG AI Engineering Notes*

A 429 is not a sign that the upstream is broken. In a marketplace it is the normal output of a scheduler that has more requests than slots, and the fix is almost never "retry harder" — it is "retry in a shape the scheduler can absorb".

This page covers the scheduling side of rate limits: why 429s cluster, what `Retry-After` actually tells you, and how to write a retry loop that does not amplify mild congestion into a full outage.

## Why 429s arrive in bursts, not evenly

Every upstream seller behind a gateway has its own concurrency window: a number of in-flight requests it will accept before it starts shedding load. Below that window you see a steady stream of 200s. Above it, the queue does not degrade gently — it flips. Requests that would have taken 2 seconds start taking 12, the seller's own timeout fires, and the gateway sees a burst of failures that all look like "too many requests".

That is why one slow endpoint can poison an unrelated one: both compete for the same concurrency tokens. Treat 429 as a **per-seller, per-model** signal, and log which upstream produced it, not just the model you asked for.

## Read Retry-After before you read the body

Most client libraries retry on a fixed schedule and ignore the header. That is the most common cause of a retry storm: the server says "come back in 3 seconds", the client comes back in 200 ms three times, then escalates.

- `Retry-After: 3` — wait the whole 3 seconds. Not 2.9, not "3 plus jitter on the first attempt".
- An HTTP-date value — compute the delta against your own clock, and clamp it so a skewed upstream clock cannot park you for an hour.
- No header at all — fall back to exponential backoff with full jitter, capped around 8–10 seconds for interactive traffic.

## A retry loop that does not amplify congestion

The loop below is deliberately boring. It caps attempts, honours the header when present, and adds full jitter only when the server gives no guidance. The `deadline_s` argument matters more than the attempt count: an interactive request has a budget, and a retry that runs past the caller's budget is just a slower failure.

```python
import random, time, httpx

RETRY_STATUS = {429, 500, 502, 503, 504}

def call_with_budget(client, payload, deadline_s=25.0, max_attempts=4):
    start = time.monotonic()
    for attempt in range(1, max_attempts + 1):
        try:
            r = client.post("/v1/chat/completions", json=payload, timeout=20.0)
        except (httpx.ConnectError, httpx.ReadTimeout):
            if attempt == max_attempts:
                raise
            delay = random.uniform(0, min(8.0, 0.5 * 2 ** (attempt - 1)))
        else:
            if r.status_code not in RETRY_STATUS:
                r.raise_for_status()
                return r.json()

            header = r.headers.get("retry-after")
            if header and header.isdigit():
                delay = min(float(header), 30.0)          # trust the server
            else:
                delay = random.uniform(0, min(8.0, 0.5 * 2 ** (attempt - 1)))  # full jitter

        remaining = deadline_s - (time.monotonic() - start)
        if remaining <= 1.0 or attempt == max_attempts:
            return {"error": "budget_exhausted", "status": r.status_code}
        time.sleep(min(delay, remaining))
    raise RuntimeError("unreachable")
```

Two details are easy to miss. The sleep is clamped to the remaining budget, so the last attempt never overshoots the caller's deadline. And `Retry-After` is capped at 30 seconds: sellers occasionally return a very large value during maintenance, and honouring it literally is worse than failing fast.

## Spread the load instead of retrying it

Retries buy time; routing buys throughput. If a model has several sellers behind it, a 429 from one seller should not be retried against the same seller — it should be re-routed. That is the part a client cannot do alone, and it is why the seller count on a listing matters: with one seller there is nowhere to route, and the only options are waiting or degrading the request.

In the current snapshot 58 of 109 live listings have a single seller, so this asymmetry is worth planning for: pick multi-seller listings for anything latency-sensitive, and put single-seller ones behind a queue that can absorb a 30-second wait without the caller noticing.

## What to watch on the status side

Three numbers, sampled per seller, cover most of it: 429 share of total requests, p95 latency, and the retry-to-success ratio. The first tells you whether you are pushing against the window; the second whether you are about to; the third whether your retry policy is helping or hurting. If 429 share rises while p95 stays flat, you are being throttled, not congested — the fix is usually concurrency, not backoff.

Live availability per upstream is published on the [status page](https://ai.dijinlong.com/status), the per-model seller counts that decide whether re-routing is possible are on the [model plaza](https://ai.dijinlong.com/model-plaza), and current prices for the models you are re-routing to are on the [pricing page](https://ai.dijinlong.com/pricing).

## The short version

1. 429 is a scheduling signal from one upstream, not a verdict on the gateway.
2. Obey `Retry-After` exactly; only invent jitter when the header is absent.
3. Cap retries by deadline, not by attempt count.
4. Re-route before you re-retry, and only where a second seller exists.
5. Track 429 share, p95 and retry-to-success together — they diagnose different problems.
