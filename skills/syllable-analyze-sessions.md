---
name: Analyze Syllable sessions and debug a bad call
description: Find the session, read the transcript, timeline, latency and tool results, label it, and roll it into an insights workflow.
api: openapi/syllable-sdk-openapi-original.yml
generated: '2026-08-05'
method: generated
source: openapi/syllable-sdk-openapi-original.yml
operations:
  - sessions_list
  - session_get_by_id
  - session_transcript_get_by_id
  - session_timeline_get_by_id
  - session_latency_get_by_id
  - session_full_summary_get_by_id
  - generate_session_recording_urls
  - get_session_data_by_session_id
  - get_session_data_by_sid
  - get_session_tool_call_result_by_id
  - session_label_create
  - session_labels_list
  - events_list
  - insights_workflow_create
  - insights_workflow_activate
  - insights_workflow_queue_work
---

# Analyze Syllable sessions and debug a bad call

Base URL `https://api.syllable.cloud/api/v1`, header `Syllable-API-Key: <token>`.

## Find the session

`sessions_list` (`GET /sessions/`) with `start_datetime` / `end_datetime` plus the
`search_fields` / `search_field_values` pair, `order_by` / `order_by_direction`, and
`page` / `limit`. Project columns with `fields`. If all you have is a carrier-side id, use
`get_session_data_by_sid`
(`GET /session_debug/sid/{channel_manager_service}/{channel_manager_sid}`) to cross the
boundary from a Twilio/SIP sid to a Syllable session.

## Read the session, in increasing depth

| Question | Operation |
|---|---|
| What is this session? | `session_get_by_id` |
| What was said? | `session_transcript_get_by_id` |
| What happened, in order? | `session_timeline_get_by_id` |
| Why was it slow? | `session_latency_get_by_id` |
| Give me the whole picture | `session_full_summary_get_by_id` |
| Let me listen | `generate_session_recording_urls` (`POST /sessions/recording/{session_id}`) |
| Raw debug payload | `get_session_data_by_session_id` |
| What did that tool actually return? | `get_session_tool_call_result_by_id` (`/session_debug/tool_result/{session_id}/{tool_call_id}`) |

`session_timeline_get_by_id` returns events of kind `transcript`, `tool` or `latency` —
that enum is the fastest way to separate "the model said the wrong thing" from "the tool
returned the wrong thing" from "the turn took too long".

## Confirm the tool call before blaming the prompt

When an agent gives a wrong answer, pull `get_session_tool_call_result_by_id` for the tool
call on the timeline before editing the prompt. A stale data source or a failing service
credential looks identical to a bad prompt from the transcript alone.

## Label it

`session_label_create` (`POST /session_labels/`) records a quality evaluation and a
description of the issue. `session_labels_list` and `session_label_get_by_id` read them
back. Labels are the human-graded ground truth that insights workflows consume.

## Roll it up

- `events_list` (`GET /events/`) is the poll-only stream of in-session events (tool calls
  and LLM prompts), keyed on `session_id` and `conversation_id`. There is no push surface
  for these — only outbound campaign webhooks push.
- `insights_workflow_create` then `insights_workflow_activate`, and
  `insights_workflow_queue_work` to run it. `insights_workflow_inactivate` pauses it.
  Supporting documents live in folders via `insights_folder_create` and
  `insights_folder_upload_file`.
- Dashboards exist (`post_sessions_dashboard`, `post_session_summary_dashboard`,
  `post_session_events_dashboard`, `post_session_transfers_dashboard`) but **all four are
  marked `deprecated: true` in the spec** — build on `sessions_list` and the insights
  workflows instead.

## Rules to follow

- **Sessions carry PHI in healthcare deployments.** Transcripts, recordings and raw debug
  payloads are patient content. Syllable publishes HIPAA and HITRUST posture at
  https://trust.syllable.ai; handle the data you pull accordingly and prefer
  `session_full_summary_get_by_id` over raw recordings when a summary answers the question.
- **Recording URLs are generated, not permanent.** `generate_session_recording_urls` is a
  `POST` that mints access; do not cache the URL as a stable identifier.
- **Pagination and errors** follow the platform conventions: `page`/`limit`, and a `422`
  body of `{"detail":[{"loc","msg","type"}]}`.
