# CALL-E Troubleshooting

Use this guide when CALL-E installation, authentication, or tool verification
fails in a local agent environment.

## Cursor agent shell returns `CONNECT tunnel failed, response 403`

### Symptoms

In Cursor, CALL-E setup may fail even though the local machine has normal network
access. Common symptoms include:

- `npx skills add https://github.com/CALLE-AI/call-e-integrations --skill calle -g`
  fails because sandboxed Git cannot write hooks.
- `calle auth login` fails with a generic `fetch failed` error.
- A direct network check fails with:

  ```text
  curl: (56) CONNECT tunnel failed, response 403
  ```

- The agent environment reports limited or allowlist-only network access.

### Cause

The `403` is returned by Cursor's sandbox network gate before the request reaches
CALL-E. It is not a CALL-E authentication failure and does not mean the CALL-E
server rejected the user.

The sandboxed agent shell may block outbound HTTPS connections to
`https://seleven-mcp-sg.airudder.com`, which prevents the CLI from starting the
brokered OAuth flow.

### Fix

Change the local Cursor agent execution mode so commands are not running in the
restricted sandbox:

1. Open Cursor Settings.
2. Go to **Agents**.
3. In **Auto-Run**, open **Auto-Run Mode**.
4. Select **Run Everything (Unsandboxed)**.

After switching to a non-sandboxed mode, retry the CALL-E login or setup check.

### Cursor Setting Screenshot

![Cursor Auto-Run Mode setting showing Run Everything Unsandboxed](../assets/troubleshooting/cursor-agent-execution-mode.png)

### Verify

Run these commands outside the restricted sandbox:

```bash
calle auth login
calle auth status --json
calle mcp tools
```

Confirm that authentication is usable and that the tool list includes:

```text
plan_call
run_call
get_call_run
```

If the same URL works from the user's terminal but fails only inside the Cursor
agent shell, the issue is the Cursor sandbox policy rather than CALL-E service
availability.

## `calle call plan` fails with `MCP request timed out for tools/call`

### Symptoms

`calle call plan` exits with this error while authentication and MCP tool
discovery still work:

```text
MCP request timed out for tools/call
```

### Cause

The `plan_call` request took longer than the CLI's effective request timeout.
Current CLI releases allow 150 seconds for `plan_call` by default while keeping
the shared 15-second default for other MCP requests. Older CLI releases used
the shared 15-second timeout for planning too. An explicit
`--timeout-seconds` value overrides either default.

This is a client-side timeout, not an authentication error. Planning does not
place a call, so it is safe to retry after adjusting the timeout.

### Fix

Check the defaults reported by the installed CLI:

```bash
calle call plan --help
```

If the command used an explicit timeout shorter than planning needs, remove the
flag to use the current planning default or retry with a longer value:

```bash
calle call plan \
  --to-phone +15551234567 \
  --goal "Confirm the appointment" \
  --timeout-seconds 300
```

If help reports only the 15-second shared default for planning, update the CLI
and retry:

```bash
npm install -g @call-e/cli
```

See the [CLI reference](../../packages/cli/docs/cli-reference.md#common-options)
for the canonical timeout defaults and option behavior.

### Verify

A successful retry prints the planned call payload as JSON. Review the plan and
continue through the normal confirmation flow; do not treat a successful plan
as evidence that a call has already started.
