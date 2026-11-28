# Streaming and time to first token

*Snapshot: 2026-11-28 UTC+8 · LONG AI Engineering Notes*

Every gateway claims streaming support. What actually differs is where the stream stops being a stream — and that difference never shows up in a demo, because a demo runs on a warm upstream with nothing queued.

This page covers the four shapes a "streaming" endpoint can have, how to measure time to first token honestly, and the specific way routing layers break it.

## Four shapes, three of them fake

1. **Genuine streaming.** The upstream emits deltas as it generates and the gateway forwards each frame as it arrives. Time to first token equals the upstream's own TTFT plus routing overhead.
2. **Buffered streaming.** The upstream streams, the gateway accumulates the full answer, then replays it as a stream of chunks. Text, framing and `data: [DONE]` are all correct; TTFT equals total generation time.
3. **Fake streaming.** With `stream: true` the endpoint returns a single `message` object at the end. Clients that wait for the first chunk show a spinner until the answer is finished.
4. **Stream that dies mid-flight.** Early frames arrive, then the connection closes without `[DONE]`, or with a truncated frame. A client that treats end-of-file as completion shows half an answer and reports success.

Only the first is what a chat UI assumes. The other three all return HTTP 200.

## Measure TTFT at the socket, not in the SDK

SDK-observed latency includes connection setup and client buffering. Measure from the moment the request is written to the first parsed delta, and repeat the measurement — a single sample is dominated by cold starts.

```python
def ttft(base, key, model, n=5):
    samples = []
    for _ in range(n):
        t0 = time.perf_counter()
        with urllib.request.urlopen(build_stream_request(base, key, model), timeout=60) as r:
            for line in r:
                if line.startswith(b"data:") and b"[DONE]" not in line:
                    samples.append(time.perf_counter() - t0)
                    break
        time.sleep(1)
    samples.sort()
    return round(samples[0], 3), round(samples[len(samples) // 2], 3)
```

Report the minimum and the median separately. The minimum separates network noise from real behaviour; the median tells you what users routinely get. A wide gap between them means queueing somewhere in the path, not a slow model.

## Why routing layers break it

Anything that wants to look at a response body has to hold it: token counting, content moderation, structured logging, retry-on-error. Each is trivial on a complete response and awkward on a stream, so the easy implementation silently converts genuine streaming into buffered streaming.

The rule that survives contact with production is blunt:

- nothing on the streaming path may accumulate the whole body
- usage comes from the upstream's final chunk when it reports one, and is estimated afterwards when it does not
- retries happen before the first byte only, never after

That trades a little metric accuracy for a TTFT that is genuinely the upstream's TTFT.

## Keep the number per model

A 109-model catalog contains models that answer in 300 ms and models that deliberately think for four seconds before their first token. One averaged figure describes neither. Per-model series, plus upstream incident history, is what makes a latency complaint answerable in minutes instead of a week.

Current per-model numbers and upstream incident history are published on the [status page](https://ai.dijinlong.com/status). The English site, including the streaming examples in every SDK snippet, is at [ai.dijinlong.com/en](https://ai.dijinlong.com/en).
