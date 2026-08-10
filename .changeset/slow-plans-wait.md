---
"@call-e/cli": patch
"@call-e/core": minor
---

Give `plan_call` requests a 150-second CLI default timeout while preserving the
15-second default for other MCP requests and explicit `--timeout-seconds`
overrides.
