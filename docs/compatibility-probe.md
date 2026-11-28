# Four probes that test an "OpenAI-compatible" endpoint

*Snapshot: 2026-11-28 UTC+8 · LONG AI Engineering Notes*

"OpenAI-compatible" is a description of a URL path, not of behaviour. This page defines the four requests we run against any base URL — ours or somebody else's — before treating it as interchangeable.

## Probe 1 — the catalog

```bash
curl -s "$BASE/v1/models" -H "Authorization: Bearer $KEY"
```

Three questions, in this order:

1. Does it return a list at all, in the `{"object": "list", "data": [...]}` shape?
2. Does the list contain the model you actually intend to call?
3. Does it contain models it would refuse to serve?

A catalog that advertises more than it can answer is worse than a short catalog, because it forces every integrator to maintain a private allowlist.

## Probe 2 — a non-streaming completion with usage

```bash
curl -s "$BASE/v1/chat/completions" \
  -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" \
  -d '{"model":"MODEL","messages":[{"role":"user","content":"say ok"}],"max_tokens":16}'
```

Assert on the response body, not on the status code: `choices[0].message.content` present, `finish_reason` populated, and both `usage.prompt_tokens` and `usage.completion_tokens` returned as integers.

A missing `usage` block passes every smoke test and quietly turns cost accounting into estimation. If the upstream genuinely does not report usage, that has to be a documented property of the model, not a surprise.

## Probe 3 — the same call with `stream: true`

Streaming is where compatible-looking endpoints diverge most. Check four properties:

- frames are `data: {...}` lines, terminated by `data: [DONE]`
- the text arrives in `choices[0].delta.content`, not in a bespoke field
- at least one frame arrives well before the final one
- the whole body is not held and flushed at the end

The fourth is the important one, and it is invisible to functional tests: HTTP 200, correct text, and a user watching a spinner for ten seconds. Time to first token is the only measurement that catches it.

```python
import json, time, urllib.request

req = urllib.request.Request(
    f"{BASE}/v1/chat/completions",
    data=json.dumps({"model": MODEL, "stream": True,
                     "messages": [{"role": "user", "content": "count to five"}]}).encode(),
    headers={"Authorization": f"Bearer {KEY}", "Content-Type": "application/json"},
)

t0 = time.perf_counter()
first = None
with urllib.request.urlopen(req, timeout=60) as r:
    for line in r:
        if not line.startswith(b"data:") or b"[DONE]" in line:
            continue
        delta = json.loads(line[6:])["choices"][0]["delta"].get("content")
        if delta and first is None:
            first = time.perf_counter() - t0
print(f"ttft={first:.2f}s")
```

## Probe 4 — deliberate failure

Send an unknown model, then a malformed body. What matters is the envelope:

- `{"error": {"message": ..., "type": ..., "code": ...}}` with a meaningful HTTP status — good
- a 200 with an empty body, or an HTML error page — the endpoint cannot be automated against
- a 500 for what is clearly a 400-class problem — your retry logic will loop forever on a permanent error

Retry classification keys off exactly this shape, which is why we test it explicitly rather than assuming it.

## Running all four as one check

The four probes are combined in one command in the LONG AI MCP server (`check_compatibility`), so the same test that a customer runs is the test we run. Field notes on interpreting each result, including which failures are upstream-specific and which are gateway bugs, are in the [integration docs](https://ai.dijinlong.com/docs); common failure symptoms and their fixes are in the [FAQ](https://ai.dijinlong.com/faq).
