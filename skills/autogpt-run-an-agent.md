---
name: autogpt-run-an-agent
description: >-
  Find a published AutoGPT agent, run it against the AutoGPT Platform External
  API, and poll for its result — with the credit, retry and reversal rules that
  actually apply.
api: AutoGPT External API
base_url: https://backend.agpt.co/external-api
spec: openapi/autogpt-external-api-openapi.json
operations:
  - find_agent_v1_tools_find_agent_post
  - run_agent_v1_tools_run_agent_post
  - execute_graph_v1_graphs__graph_id__execute__graph_version__post
  - get_graph_execution_results_v1_graphs__graph_id__executions__graph_exec_id__results_get
  - get_user_info_v1_me_get
generated: '2026-08-29'
method: generated
source: >-
  Grounded in operationIds verified present in
  openapi/autogpt-external-api-openapi.json, plus
  https://agpt.co/docs/platform/api-and-integrations/api-guide.md.
---

# Run an AutoGPT agent

## Before you start

Authenticate with **either**:

- `X-API-Key: <key>` — an account API key created in AutoGPT Platform settings, or
- `Authorization: Bearer agpt_xt_...` — an OAuth 2.0 access token.

With OAuth you need the `EXECUTE_GRAPH` scope to run and `READ_GRAPH` to read
results. `USE_TOOLS` covers the `/v1/tools/*` shortcuts.

Confirm the credential works and learn who you are acting as:

```
GET /v1/me
```
`get_user_info_v1_me_get` — returns user id, e-mail and timezone. Requires the
`IDENTITY` scope on an OAuth token.

## Step 1 — Find the agent

```
POST /v1/tools/find-agent
```
`find_agent_v1_tools_find_agent_post` — describe the job in natural language and
get back matching agents from the caller's library and the marketplace. Use this
rather than guessing a `graph_id`.

To browse instead of search:

```
GET /v1/store/agents
GET /v1/store/agents/{username}/{agent_name}
```

## Step 2 — Run it

The direct route:

```
POST /v1/tools/run-agent
```
`run_agent_v1_tools_run_agent_post`

The explicit route, when you already know the graph and the version you want
pinned:

```
POST /v1/graphs/{graph_id}/execute/{graph_version}
```
`execute_graph_v1_graphs__graph_id__execute__graph_version__post`

**STOP AND CHECK WITH A HUMAN BEFORE THIS CALL.** It:

- spends automation credits, and
- starts an autonomous agent that may act in connected third-party systems
  (Slack, GitHub, Google, Stripe and 40+ others), and
- **is not idempotent.** AutoGPT accepts no `Idempotency-Key`. If the request
  times out and you retry, you start a **second run** and pay twice.

If a call times out, do **not** blind-retry. List executions for the graph and
check whether a run already started.

## Step 3 — Poll for the result

Runs are asynchronous and AutoGPT delivers **no completion webhook**. Poll:

```
GET /v1/graphs/{graph_id}/executions/{graph_exec_id}/results
```
`get_graph_execution_results_v1_graphs__graph_id__executions__graph_exec_id__results_get`

Back off between polls. There are no published rate limits and no
`RateLimit-*` headers to steer by, so use a conservative interval (start at 5s,
back off to 30s).

## Stopping a run

The External API has no stop operation. The platform API does:

```
POST /api/graphs/{graph_id}/executions/{graph_exec_id}/stop
```

Credits already consumed by blocks that have run are **not** described as
recoverable, and AutoGPT publishes no refund window. Treat a started run as
partially unrecoverable.

## Error handling

| Status | Meaning | What to do |
|---|---|---|
| 401 | Missing or invalid credential | `{"detail":"Missing authentication. Provide API key or access token."}` — re-issue the token at `POST /api/oauth/token` |
| 402 | Out of automation credits, or NO_TIER paywall | **Do not retry.** Tell the human to top up the wallet |
| 403 | Token lacks the required scope | Re-authorize with the scope named in `scopes/autogpt-scopes.yml` |
| 422 | Schema validation failed | Read `detail[].loc` for the field and `detail[].msg` for the reason; `hint` carries the remediation |
| 429 | Frequency or concurrency cap | Back off; honour `Retry-After` if present |
| 503 | Dependency degraded | Honour `Retry-After` before retrying |

Errors are **not** RFC 9457 and carry no stable error code. Branch on the HTTP
status, not on the message text.

See `errors/autogpt-problem-types.yml` and
`conventions/autogpt-conventions.yml`.
