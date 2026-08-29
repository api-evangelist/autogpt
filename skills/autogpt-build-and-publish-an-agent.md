---
name: autogpt-build-and-publish-an-agent
description: >-
  Inspect the AutoGPT block catalog, create an agent graph, test a single block,
  and read the marketplace — through the AutoGPT Platform External API.
api: AutoGPT External API
base_url: https://backend.agpt.co/external-api
spec: openapi/autogpt-external-api-openapi.json
operations:
  - get_graph_blocks_v1_blocks_get
  - execute_graph_block_v1_blocks__block_id__execute_post
  - create_graph_v1_graphs_post
  - get_store_creators_v1_store_creators_get
  - get_store_creator_v1_store_creators__username__get
  - get_store_agents_v1_store_agents_get
generated: '2026-08-29'
method: generated
source: >-
  Grounded in operationIds verified present in
  openapi/autogpt-external-api-openapi.json, with the entity model from
  data-model/autogpt-data-model.yml.
---

# Build an AutoGPT agent programmatically

AutoGPT's model is short: a **block** is one unit of capability, a **graph**
wires blocks together and *is* the agent, and running a graph version produces a
**graph execution**. This skill covers the build half.

## Step 1 — Read the block catalog

```
GET /v1/blocks
```
`get_graph_blocks_v1_blocks_get` — every block available to this account, with
its input fields and its cost model. Read this before composing anything; block
ids are opaque and cannot be guessed.

## Step 2 — Test one block in isolation

```
POST /v1/blocks/{block_id}/execute
```
`execute_graph_block_v1_blocks__block_id__execute_post`

**This spends automation credits.** It is the cheapest way to check a block's
input shape before wiring it into a graph, but it is still a paid, non-idempotent
call — one execution per request, no retry safety. Confirm with a human before
looping over it.

## Step 3 — Create the graph

```
POST /v1/graphs
```
`create_graph_v1_graphs_post`

A graph carries `nodes` (each bound to a `block_id`) and `links` (edges between
node outputs and node inputs). Graphs are versioned; the version you create is
the version you must name when you execute.

Creating a graph is free. Executing it is not — see
`skills/autogpt-run-an-agent.md`.

> **No delete in this API.** The External API exposes no graph deletion. The
> platform API's `DELETE /api/graphs/{graph_id}` is documented as "Delete graph
> permanently" with no restore operation anywhere in the contract. Do not reach
> for it to clean up a mistake.

## Step 4 — Look at what others published

```
GET /v1/store/agents
GET /v1/store/agents/{username}/{agent_name}
GET /v1/store/creators
GET /v1/store/creators/{username}
```

Marketplace reads. Cheaper and faster than building from scratch — check whether
a published agent already does the job before composing one.

Note that **submitting** an agent to the marketplace is not in the External API;
it lives on the platform API (`POST /api/store/submissions`) and goes through
human review.

## Pagination

Not uniform. Store listings take `page` and `page_size`; other collections take
`limit` and `offset`, or bare `limit`. Read the parameters for the specific
operation rather than assuming. There is no cursor pagination and no `Link`
header.

## Errors

`402` on the paid execute, `422` on any schema mismatch (read `detail[].loc`),
`401`/`403` on credential and scope problems. See
`errors/autogpt-problem-types.yml`.
