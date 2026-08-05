---
name: Build and deploy a Syllable voice agent
description: Stand up a working Syllable agent end to end — knowledge, credentials, tools, prompt, agent, then the channel target it answers on.
api: openapi/syllable-sdk-openapi-original.yml
generated: '2026-08-05'
method: generated
source: openapi/syllable-sdk-openapi-original.yml
operations:
  - data_sources_create
  - service_create
  - tool_create
  - prompt_get_supported_llms
  - prompts_create
  - agent_get_available_voices
  - agent_create
  - channels_list
  - channel_targets_create
  - send_test_message
  - agent_get_by_id
---

# Build and deploy a Syllable voice agent

Base URL `https://api.syllable.cloud`. Every request carries the header
`Syllable-API-Key: <organization API token>` (generate it in the Syllable Console under
profile → API tokens; one token per organization, shown once). All paths below are under
`/api/v1`.

## Build order matters

Syllable resources layer, and each depends on the ones below it:

```
Data Sources -> Tools + Services -> Prompts -> Agents -> Channels/Targets -> Users
```

Data sources never attach to an agent directly. You wrap a data source in a tool, put the
tool on a prompt, and attach the prompt to the agent.

## Steps

1. **Create the knowledge base.** `data_sources_create` (`POST /data_sources/`). The
   document body field is the text field — the CLI exposes it as `--text`. Read back with
   `data_sources_get_by_id`.

2. **Create the credential store.** `service_create` (`POST /services/`). A service holds
   auth (Basic, Bearer, or custom headers) once and is reused by many tools. Every tool is
   backed by a service.

3. **Create the tools.** `tool_create` (`POST /tools/`). Tools are what the agent calls
   during a live session — lookups, scheduling, transfers. Note the asymmetry in this
   resource: `tool_get_by_name` and `tool_delete` key on `{tool_name}`, while
   `tool_history` keys on `{tool_id}`. Tools are versioned; `tool_history` returns the
   version list.

4. **Pick the model.** `prompt_get_supported_llms`
   (`GET /prompts/llms/supported`) returns the LLMs you may name in a prompt. Do not
   hard-code a model string without checking this first.

5. **Create the prompt.** `prompts_create` (`POST /prompts/`). The prompt names the LLM,
   the tools available, and the behavior rules. Prompts are versioned automatically — every
   edit creates a new version and `prompts_history` (`GET /prompts/{prompt_id}/history`)
   lists them. Temperature is validated server-side; an out-of-range value returns 422.

6. **Pick a voice.** `agent_get_available_voices` (`GET /agents/voices/available`) for
   voice agents.

7. **Create the agent.** `agent_create` (`POST /agents/`) with the prompt id, agent type,
   timezone, optional opening message and session variables. Note `agent_update` is
   `PUT /agents/` — the id travels in the body, not the path — while `agent_get_by_id` and
   `agent_delete` are `/agents/{agent_id}`.

8. **Test before going live.** A test channel is auto-generated for every agent. Drive it
   with `send_test_message` (tag `agents.test`) and read the result back with
   `session_transcript_get_by_id`.

9. **Attach the real channel target.** `channels_list` to find the channel, then
   `channel_targets_create` (`POST /channels/{channel_id}/targets`) to bind the agent to a
   phone number or chat id. An agent maps one-to-one to a channel target.

## Rules to follow

- **Pagination.** Every list operation takes `page` (0-based) and `limit` (default 25),
  plus `order_by` / `order_by_direction` and `search_fields` / `search_field_values`. Use
  `fields` to project. Responses come back in a `ListResponse_<Entity>_` envelope.
- **No idempotency.** There is no `Idempotency-Key` on any Syllable operation. A retried
  `POST` creates a second resource. Read back by list-and-search before retrying a create.
- **No documented rate limits.** No `429`, `Retry-After` or `X-RateLimit-*` is declared.
  Back off conservatively rather than assuming headroom.
- **Errors.** `422` returns `{"detail": [{"loc": [...], "msg": "...", "type": "..."}]}` —
  `loc` names the exact failing field. Also expect `400`, `404`, `412` and `500`. Nothing is
  RFC 9457; the content type is always `application/json`.
- **Deprecated surfaces.** Avoid `channels_twilio_create` / `channels_twilio_update` and
  the whole `language_groups` family — all are marked `deprecated: true` in the spec. Use
  `voice_groups` instead of `language_groups`.
- **Prefer the CLI for scripted work.** `syllable <resource> <verb>` covers the same
  surface, and `--dry-run` prints the exact method, url and body it would send (plus
  `missing_required_fields`) without writing anything.
