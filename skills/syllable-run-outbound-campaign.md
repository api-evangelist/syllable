---
name: Run a Syllable outbound campaign
description: Create an outbound campaign with webhooks, load a batch of contacts, run it, and reconcile the results.
api: openapi/syllable-sdk-openapi-original.yml
generated: '2026-08-05'
method: generated
source: openapi/syllable-sdk-openapi-original.yml
operations:
  - outbound_campaign_create
  - outbound_campaign_list
  - outbound_campaign_get_by_id
  - outbound_campaign_update
  - outbound_batch_create
  - outbound_batch_upload
  - outbound_batch_add
  - outbound_batch_remove
  - outbound_batch_get_by_id
  - outbound_batch_results
  - sessions_list
  - session_transcript_get_by_id
---

# Run a Syllable outbound campaign

Base URL `https://api.syllable.cloud/api/v1`, header `Syllable-API-Key: <token>`.
An agent and a channel target must already exist — see
`syllable-build-and-deploy-agent.md`.

## Steps

1. **Create the campaign.** `outbound_campaign_create`
   (`POST /outbound/campaigns`). The campaign body carries a `webhooks[]` array — this is
   the only event surface Syllable exposes, so register it here rather than looking for a
   separate subscription endpoint. Each entry needs:
   - `url` — an HTTPS endpoint of yours
   - `request_method` — `POST`, `PUT` or `PATCH`
   - `trigger_statuses` — which `ChannelManagerStatus` values fire the call
   - `auth_values.hmac_secret` — optional, standard Base64 (RFC 4648) decoding to 32–512
     bytes of key material. Set it. Reads return only `auth_value_keys`; the secret is never
     echoed back.

   On update, leave a secret's value `null` to keep the stored one; omit the key entirely
   and the stored value is removed.

2. **Create the batch.** `outbound_batch_create` (`POST /outbound/batches`).

3. **Load contacts.** Either `outbound_batch_upload`
   (`POST /outbound/batches/{batch_id}/upload_batch`) for a file, or `outbound_batch_add`
   (`POST /outbound/batches/{batch_id}/requests`) to append requests. Remove with
   `outbound_batch_remove` (`POST /outbound/batches/{batch_id}/remove-requests`) — note it
   is a `POST`, not a `DELETE`.

4. **Watch it run.** `outbound_batch_get_by_id` for batch state and
   `outbound_batch_results` (`GET /outbound/batches/{batch_id}/results`) for per-request
   outcomes. Your webhook fires on the `trigger_statuses` you registered.

5. **Reconcile.** `sessions_list` filtered with `start_datetime` / `end_datetime` and the
   `search_fields` / `search_field_values` pair, then `session_transcript_get_by_id` for the
   transcript of any session you need to inspect. `session_full_summary_get_by_id` and
   `session_timeline_get_by_id` give the richer views.

## Status vocabulary

`trigger_statuses` uses the `ChannelManagerStatus` enum (39 values), covering the whole
voice/SMS/email delivery lifecycle:

- Queueing and dispatch: `PENDING`, `PROCESSED`, `QUEUED`, `SENDING`, `SENT`, `ACCEPTED`
- Terminal success: `DELIVERED`, `COMPLETED`, `OPENED`, `CLICKED`, `HUMAN`, `MACHINE`
- Terminal failure: `FAILED`, `UNDELIVERED`, `DELIVERY_FAILED`, `BOUNCED`, `DROPPED`,
  `DECLINED`, `BUSY`, `NO-ANSWER`, `CANCELED`, `INVALID`, `UNEXPECTED_ERROR`
- Suppression: `UNSUBSCRIBED`, `SPAM_REPORT`, `PRIOR_UNSUBSCRIBED`, `PRIOR_SPAM_REPORT`,
  `PRIOR_DROPPED`, `PRIOR_BOUNCED`, `DUPLICATE`, `FILTERED_LINE_TYPE`
- SIP: `SIP_NOT_FOUND`, `SIP_TEMPORARILY_UNAVAILABLE`, `SIP_LOOP_DETECTED`,
  `SIP_DOES_NOT_EXIST_ANYWHERE`
- Indeterminate: `IN-PROGRESS`, `DEFERRED`, `DELIVERY_UNKNOWN`, `UNKNOWN`

Treat `MACHINE` and `HUMAN` as answer-classification, not delivery failure. Treat every
`PRIOR_*` value as a suppression that predates this campaign.

## Rules to follow

- **No idempotency key.** Re-POSTing a campaign or batch creates a duplicate. List and
  search first.
- **Verify webhook signatures.** The HMAC secret is the only authenticity signal; the
  callback URL is otherwise unauthenticated.
- **Compliance is on you.** Outbound calling and messaging carry TCPA/A2P obligations.
  `channels_twilio_numbers_a2p_compliance_check` exists for the A2P side; the suppression
  statuses above are the signal you must honor.
- **Errors.** `422` returns `{"detail":[{"loc","msg","type"}]}`; `loc` names the failing
  field.
