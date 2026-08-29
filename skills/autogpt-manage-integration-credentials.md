---
name: autogpt-manage-integration-credentials
description: >-
  List, connect and remove the third-party credentials an AutoGPT agent uses,
  through the AutoGPT Platform External API — including which of these
  operations are irreversible.
api: AutoGPT External API
base_url: https://backend.agpt.co/external-api
spec: openapi/autogpt-external-api-openapi.json
operations:
  - list_providers_v1_integrations_providers_get
  - list_credentials_v1_integrations_credentials_get
  - list_credentials_by_provider_v1_integrations__provider__credentials_get
  - initiate_oauth_v1_integrations__provider__oauth_initiate_post
  - complete_oauth_v1_integrations__provider__oauth_complete_post
  - create_credential_v1_integrations__provider__credentials_post
  - delete_credential_v1_integrations__provider__credentials__cred_id__delete
generated: '2026-08-29'
method: generated
source: >-
  Grounded in operationIds verified present in
  openapi/autogpt-external-api-openapi.json.
---

# Manage AutoGPT integration credentials

AutoGPT agents act in other people's systems. Those actions run on stored
credentials, and this is the surface that manages them. Treat every write here
as security-sensitive.

## Discover what can be connected

```
GET /v1/integrations/providers
```
`list_providers_v1_integrations_providers_get` — the providers this account can
hold credentials for.

## See what is already connected

```
GET /v1/integrations/credentials
GET /v1/integrations/{provider}/credentials
```
`list_credentials_v1_integrations_credentials_get`,
`list_credentials_by_provider_v1_integrations__provider__credentials_get`

Do this **first**, always. It is the cheapest way to avoid creating a duplicate
credential, and it is a read with no consequence.

## Connect via OAuth (preferred)

```
POST /v1/integrations/{provider}/oauth/initiate
POST /v1/integrations/{provider}/oauth/complete
```
`initiate_oauth_v1_integrations__provider__oauth_initiate_post` returns a URL a
**human** must open and approve. An agent cannot complete this step alone —
hand the URL to the user, then call `complete` with what comes back.

## Connect with a supplied secret

```
POST /v1/integrations/{provider}/credentials
```
`create_credential_v1_integrations__provider__credentials_post`

Never invent, guess or reuse a secret here. If the user has not given you the
credential in this conversation, ask for it — do not read it from a file or an
environment you were not pointed at.

## Remove a credential — IRREVERSIBLE

```
DELETE /v1/integrations/{provider}/credentials/{cred_id}
```
`delete_credential_v1_integrations__provider__credentials__cred_id__delete`

**Confirm with a human before calling this.** There is no restore, no undelete
and no trash in the AutoGPT API. Deleting a credential silently breaks every
agent and every schedule that depends on it, and the only recovery is to
re-authorize the provider from scratch.

Required OAuth scopes: `READ_INTEGRATIONS` to list, `MANAGE_INTEGRATIONS` to
create, `DELETE_INTEGRATIONS` to remove. See `scopes/autogpt-scopes.yml`.

## Errors

`401` missing credential, `403` missing scope, `404` unknown provider or
credential id, `422` schema mismatch. Envelopes and remediation are in
`errors/autogpt-problem-types.yml`.
