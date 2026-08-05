# Syllable

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Syllable is a Mountain View, California company (founded 2017) that operates an agentic platform for
building, deploying and optimizing AI voice, SMS, email, web-chat and WhatsApp agents, with a strong
focus on healthcare patient experience and contact-center automation.

- Website: https://syllable.ai/
- Documentation: https://docs.syllable.ai/
- Console / sign-up: https://syllable.cloud/sign-up
- GitHub: https://github.com/asksyllable
- Status: https://syllable.statuspage.io
- Trust center: https://trust.syllable.ai

## API surface

The **Syllable Platform SDK API** is served from `https://api.syllable.cloud` under `/api/v1`, with a
real OpenAPI 3.1.0 document (**176 operations, 107 paths, 310 component schemas**) published at
`https://spec.speakeasy.com/syllable/syllable/syllable-sdk-oas-with-code-samples` and rendered into the
documentation site. Authentication is a per-organization API token in the `Syllable-API-Key` header.

Agent-facing surfaces are unusually complete for a company this size:

| Surface | Where |
|---|---|
| OpenAPI 3.1.0 | `openapi/` (harvested verbatim, with code samples) |
| MCP server | `https://docs.syllable.ai/mcp` — anonymous `tools/list`, 3 tools, documentation scope |
| A2A agent card | `https://docs.syllable.ai/.well-known/agent-card.json` — graded *flavored* against A2A 1.0.0 |
| Provider agent skills | a Claude Code skill in the CLI repo, plus a `/.well-known/agent-skills/` skill |
| llms.txt | on both `docs.syllable.ai` and `syllable.ai` |
| SDKs | `syllable-sdk` on npm and PyPI; `@syllableai/webchat` for the embeddable widget |
| CLI | `asksyllable/syllable-cli` (Go), Homebrew cask, "built for humans and AI agents" |

## Notable gaps

- **No `security.txt`** on any host, despite a trust center carrying SOC 2 Type 2, HIPAA, HITRUST and GDPR.
- **No idempotency contract** — no `Idempotency-Key` on any of the 176 operations.
- **No documented rate limits** — no `429`, `Retry-After` or `X-RateLimit-*` in the spec.
- **No deprecation policy or `Sunset` header**, though 12 operations are marked `deprecated: true`.
- **No AsyncAPI**; the only event surface is per-campaign outbound webhooks (HMAC-signed).
- Errors are plain `application/json`, not RFC 9457.
