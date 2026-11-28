# Model aliases: the layer that keeps a 109-model catalog stable

*Snapshot: 2026-11-28 UTC+8 · LONG AI Engineering Notes*

An OpenAI-compatible gateway has one job that no single upstream provider has to do: keep a hundred-odd model names working while the vendors behind them rename, deprecate and re-tier their own.

This page explains the three-name scheme we use, why the `/v1/models` endpoint is treated as a contract rather than a convenience, and what happens when an upstream retires a snapshot.

## Why renames are the expensive failure

A 404 from an upstream is loud. A rename is quiet: the health check still returns 200, the status page is green, and the only symptom is a `400 model_not_found` appearing in somebody else's production logs at 2am.

The cost is not one request. Any client that hardcoded an upstream identifier now has to ship a release on the vendor's schedule instead of its own.

## Three names per model

| Layer | Example | Who it is for | May it change? |
|---|---|---|---|
| Canonical id | `gpt-4o-mini` | integrators, our API | no, once published |
| Upstream id | `gpt-4o-mini-2024-07-18` | the vendor being called | yes, at any time |
| Display name | `GPT-4o mini` | humans on price pages | yes, freely |

Only the middle layer is allowed to move. A rename upstream becomes a one-line mapping edit, and the canonical id keeps resolving.

```python
ALIASES = {
    "gpt-4o-mini": ["gpt-4o-mini", "gpt-4o-mini-2024-07-18"],
    "kling-v2":    ["kling-v2", "kling-v2-master"],
}

def resolve(model: str) -> str:
    for canonical, names in ALIASES.items():
        if model in names:
            return canonical
    raise ValueError(f"unknown model: {model}")
```

The mapping is one-way on purpose. A request that arrives with an upstream id is normalised to the canonical form before routing, logging and billing, so the price table and the usage record never disagree about which model a token belonged to.

## `/v1/models` is a contract

Listing models is the cheapest call an integrator makes, and the easiest place to be careless. Our rule is that the list endpoint is the source of truth:

- every model in the list must be routable, or the entry is removed
- every routable model must appear in the list, or integrators will not find it
- the fields carry enough to be useful without a second call: context window, streaming support, tool-call support, and the price actually charged per million tokens

The public version of that table lives on the [per-model reference pages](https://ai.dijinlong.com/model-docs), one page per model, so a disagreement about price can be settled by reading the same page both parties are looking at.

## Retiring a snapshot without breaking anyone

When an upstream stops accepting a dated id we do not delete it. The canonical id is re-pointed at the successor for a stated deprecation window, callers are told which upstream id is now serving them, and the mapping page records the change. After the window closes the alias is removed and the canonical id is retired on our own schedule.

The result is a catalog that is monotonic from the client's point of view: a model that worked yesterday still resolves today, even if the bytes are now answered by a different upstream seller.

Browse the current catalog on the [model plaza](https://ai.dijinlong.com/model-plaza), or check the per-model detail in the [model docs](https://ai.dijinlong.com/model-docs).
