# Key hygiene for a gateway with many upstreams

*Snapshot: 2026-11-28 UTC+8 · LONG AI Engineering Notes*

One key for every project is convenient for exactly as long as nothing goes wrong. After that it is a forensic problem: nobody can tell which client burned 40 million tokens on Tuesday.

This page is the key model we recommend to customers, and the one we use internally.

## One key per environment, per project

A key should identify a **billing boundary**, not a person. The practical minimum:

- `prod` and `staging` never share a key
- one key per product or repository that calls the API
- one named key per CI/CD pipeline that runs unattended

Names cost nothing and answer the only question that matters during an incident: which workload produced this traffic.

## Keep the key out of the client's model context

An MCP server or agent framework that stores the key in a config file the model can read is one prompt-injection away from leaking it. The key belongs in the process environment or an OS keychain, never in a file the model is asked to open.

```json
// client config: the key is read from the environment, not written in the file
{
  "mcpServers": {
    "longai": {
      "command": "npx",
      "args": ["-y", "longai-mcp"],
      "env": { "LONGAI_API_KEY": "${LONGAI_API_KEY}" }
    }
  }
}
```

If a framework forces you to paste the key into a JSON file, that file goes in `.gitignore` before you paste it, not after.

## Rotation without downtime

Rotating a key should not require a deploy. The sequence that works:

1. create the new key, leave the old one alive
2. deploy the new key to the process environment
3. confirm traffic moved by watching usage per key
4. revoke the old key

Steps 2 and 3 are the ones people skip, and then they revoke the key that is still serving 30% of traffic.

## Detect the thing you cannot prevent

Revocation is a reaction; detection is the prevention. Three signals worth alerting on, per key:

- usage from a region or ASN you never call from
- a sudden change in model mix (a key that only used `gpt-4o-mini` now calling expensive video models)
- 401/403 spikes, which usually mean a key was rotated somewhere else and something did not pick it up

Usage records carry the key id, so these are queries rather than guesses.

## What we do on the gateway side

Every key is scoped to a project, counted separately, and shows up in the usage table by name. Revoking a key takes effect on the next request; in-flight requests are allowed to finish, because killing a generation mid-stream still bills for the generated tokens.

Details on limits, top-ups and what happens when a key is revoked mid-flight are in the [FAQ](https://ai.dijinlong.com/faq); the integration examples that read `LONGAI_API_KEY` from the environment are in the [integration docs](https://ai.dijinlong.com/docs).
