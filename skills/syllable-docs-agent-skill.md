---
name: Actiumhealth
description: Use when building, configuring, and deploying AI voice agents; managing outbound campaigns; creating prompts and tools; setting up channels and integrations; analyzing sessions and insights; or working with the Syllable SDK to automate agent workflows.
metadata:
    mintlify-proj: actiumhealth
    version: "1.0"
---

# Syllable Skill Reference

## Product summary

Syllable is an AI agent platform for building and deploying voice, SMS, email, chat, and WhatsApp agents. Agents are powered by LLMs and configured through prompts, tools, data sources, and messages. You can build agents via the Syllable Console (no-code UI) or the Syllable SDK (TypeScript/Python). Agents handle inbound conversations through channels (voice, SMS, email, chat, WhatsApp) and outbound campaigns (bulk calling, messaging). Key resources: **Prompts** (LLM instructions), **Tools** (APIs/functions agents call), **Data Sources** (knowledge bases), **Messages** (greetings), **Agents** (the assembled component), **Channels** (communication modes), **Campaigns** (outbound batches), **Sessions** (conversation logs), **Workflows** (insight analysis). Primary docs: https://docs.syllable.ai

## When to use

Reach for this skill when:
- Building a new agent from scratch (prompt → tools → agent)
- Configuring outbound voice, SMS, or email campaigns with batches
- Creating or updating prompts, tools, data sources, or messages
- Setting up channels (Twilio voice, webchat, email, SMS, WhatsApp)
- Testing agents via voice or chat before deployment
- Analyzing sessions, transcripts, and tool calls
- Running insight workflows to extract metrics from call recordings
- Using the SDK to automate agent creation, campaign management, or batch uploads
- Debugging tool failures or agent behavior issues
- Managing API tokens and authentication

## Quick reference

### Core components

| Component | Purpose | Key config |
|-----------|---------|-----------|
| **Prompt** | LLM instructions for agent behavior | Provider (OpenAI/Google/Azure), model, temperature, seed, tools, text |
| **Tool** | API or function agent can call | Schema (endpoint, parameters), static parameters, service auth |
| **Data Source** | Knowledge base for agent lookup | Text content, linked via tool with `doc` static parameter |
| **Message** | Agent greeting at conversation start | Default text, time-based rules, variables, pauses |
| **Agent** | Assembled agent ready to deploy | Prompt, message, timezone, STT provider, voice group, tools config |
| **Channel** | Communication mode (voice, SMS, email, chat, WhatsApp) | Channel type, Twilio auth, telephony timeouts, responsive dialogue |
| **Target** | Endpoint for agent (phone number, email, chat URL) | Mode, channel, phone/email, agent, test flag |
| **Campaign** | Outbound bulk communication effort | Agent, channel, rate/hour, business hours, call filtering, voicemail detection |
| **Batch** | Set of contacts for a campaign | CSV upload (reference_id, target, custom columns), auto-run flag, expiry |
| **Session** | Individual conversation with user | Transcript, tool calls, recording (voice), labels, insights |
| **Workflow** | Automated insight extraction from sessions | Tools, sample rate, session conditions, time limits |

### API authentication

Generate a single API token per organization:
1. Console → Profile icon (top right) → "API tokens"
2. Click "Generate" → copy token immediately (not shown again)
3. Store securely; revoke and regenerate if lost
4. Use in SDK: `SyllableSDK(api_key_header="your-token")`

### LLM models available

| Provider | Models |
|----------|--------|
| **OpenAI** | gpt-4o, gpt-4o-mini, gpt-4.1, gpt-4.1-mini, gpt-5, gpt-5-mini |
| **Google** | gemini-2.0-flash, gemini-2.0-flash-lite, gemini-2.5-flash, gemini-2.5-flash-lite |
| **Azure OpenAI** | gpt-4o, gpt-4o-mini, gpt-4.1 |

### Campaign batch CSV format

Required columns:
- `reference_id` — unique identifier per row
- `target` — phone number (format: +14155551234) or email

Optional columns: any custom data (e.g., `first_name`, `account_id`) accessible in prompt/message as `{{ vars.first_name }}`

### Session status (voice campaigns)

| Status | Meaning |
|--------|---------|
| PENDING | Queued, not yet dialed |
| COMPLETED | Call placed, no voicemail detection |
| HUMAN | Voicemail detection: live person |
| MACHINE | Voicemail detection: voicemail system |
| UNKNOWN | Voicemail detection: inconclusive |
| NO-ANSWER | Timeout, no answer |
| BUSY | Busy signal |
| FAILED | Twilio reported failure |
| FILTERED_LINE_TYPE | Excluded by call filter rules |

### Prompt variables syntax

Use `{{ variable }}` in prompts, messages, and tool descriptions:
- `{{ vars.session.id }}` — session ID
- `{{ vars.session.datetime }}` — current timestamp
- `{{ vars.session.language }}` — active language
- `{{ vars.agent.name }}` — agent name
- `{{ vars.custom_column }}` — CSV column from batch (e.g., `{{ vars.first_name }}`)

### Tool schema structure

```json
{
  "type": "endpoint",
  "endpoint": {
    "url": "https://api.example.com/endpoint",
    "method": "get|post|put|patch",
    "argumentLocation": "query|body"
  },
  "tool": {
    "type": "function",
    "function": {
      "name": "tool_name",
      "description": "What the tool does",
      "parameters": {
        "type": "object",
        "properties": { "param1": { "type": "string" } },
        "required": ["param1"]
      }
    }
  },
  "staticParameters": [
    {
      "name": "api_key",
      "type": "string",
      "default": "value",
      "required": true
    }
  ]
}
```

## Decision guidance

### When to use Console vs SDK

| Task | Console | SDK |
|------|---------|-----|
| Build first agent | ✓ (faster, visual) | ✓ (code-driven) |
| Create prompt/tools/messages | ✓ (UI editor) | ✓ (JSON/code) |
| Test agent voice/chat | ✓ (built-in test) | ✗ (use Console) |
| Bulk create agents | ✗ | ✓ (automation) |
| Automate campaign uploads | ✗ | ✓ (batch API) |
| Manage API integrations | ✓ (basic) | ✓ (full control) |

### When to use data source vs prompt text

| Scenario | Use data source | Use prompt text |
|----------|-----------------|-----------------|
| Static knowledge (FAQ, docs) | ✓ (cheaper, faster) | ✗ |
| Agent behavior instructions | ✗ | ✓ (always) |
| Large reference material (>1000 words) | ✓ (reduces tokens) | ✗ |
| Different agents, same prompt, different knowledge | ✓ (override per agent) | ✗ |
| Real-time data | ✗ | ✗ (use tool instead) |

### When to use voicemail detection v1 vs v2

| Feature | v1 | v2 (beta) |
|---------|----|----|
| Detects live person vs voicemail | ✓ | ✓ |
| Handles automated call screeners | ✗ | ✓ |
| Responds to keypad menus | ✗ | ✓ |
| Requires agent greeting | ✗ | ✓ |
| Configurable timeouts | ✓ | ✗ |

### When to use call filtering

Use call filtering when you need to restrict outreach by line type (mobile, landline, VoIP) or carrier. Example: dial only landlines and fixed VoIP, skip mobile. Incurs per-number lookup cost via Twilio Line Type Intelligence.

## Workflow

### Build and deploy a new agent

1. **Create prompt**: Console → Prompts → New prompt
   - Select LLM provider/model
   - Write instructions (role, behavior, guidelines)
   - Add tools (reference by name in instructions)
   - Save

2. **Create tools** (if needed): Console → Tools → New tool
   - Define endpoint URL, method, parameters (JSON Schema)
   - Add static parameters (defaults, auth)
   - Link to service (auth credentials)
   - Save

3. **Create data source** (if needed): Console → Data Sources → New data source
   - Paste knowledge base text
   - Create tool with `doc` static parameter pointing to data source
   - Save

4. **Create message**: Console → Messages → New message
   - Write default greeting
   - Add time-based rules (optional)
   - Use variables for personalization
   - Save

5. **Create agent**: Console → Agents → New agent
   - Name, description
   - Select prompt, message
   - Set timezone, STT provider (Google STT V1/V2)
   - Select voice group (if multilingual)
   - Configure tool overrides (static parameters per agent)
   - Save

6. **Test agent**: Click "Start" on Voice or Chat test card
   - Verify greeting, responses, tool calls
   - Check debug panel for tool arguments/results
   - Test edge cases, time-based rules

7. **Deploy**: Agent is live immediately on save
   - Create channel (Twilio voice, webchat, etc.)
   - Create target (phone number, email, chat URL)
   - Assign agent to target
   - Monitor sessions in Sessions workspace

### Create and run an outbound campaign

1. **Create campaign**: Console → Campaigns → Create campaign
   - Name, description, labels
   - Mode (Voice, SMS, Email)
   - Select channel source (must exist)
   - Set display ID (caller ID or sender)
   - Set rate per hour
   - Set campaign hours (days, times)
   - Configure voicemail detection (v1, v2, or disabled)
   - Configure call filtering (optional)
   - Save

2. **Create batch**: Click campaign → Batches → Create batch
   - Set expiry date (optional)
   - Toggle auto-run (on = start immediately, off = manual start)
   - Save

3. **Upload contacts**: Click batch → Save and Upload
   - Download CSV template
   - Fill: `reference_id`, `target` (phone/email), custom columns
   - Upload CSV
   - Verify status (Active, Paused, Pending, Failed)

4. **Test**: Add single test number to batch
   - Observe call respects campaign hours and rate
   - Check session transcript and tool calls

5. **Monitor**: View batch results
   - Status breakdown (COMPLETED, HUMAN, MACHINE, FAILED, etc.)
   - Fetch results via API or UI
   - Subscribe to webhooks for real-time status updates

### Extract insights from sessions

1. **Create insight tool**: Console → Tools → New Tool (Insights tab)
   - Name, description
   - Select LLM provider/model
   - Write prompt to extract insights
   - Define outputs (name, data type, return values)
   - Save

2. **Create workflow**: Console → Workflows → New workflow
   - Name, description
   - Type: Agent, Transfer, or Folder
   - Set session conditions (sample rate, duration, agents, prompts)
   - Set time limits (start, end)
   - Add tools (sequential order)
   - Save

3. **Activate workflow**: Click Activate
   - Review estimated cost
   - Workflow processes matching sessions
   - View results in Dashboards → Insights

## Common gotchas

- **Prompt token cost**: Every word in prompt counts. Move large static data to data sources instead.
- **Tool parameter names must match API**: If tool schema says `latitude`, API endpoint must accept `latitude` (case-sensitive).
- **Static parameters require `doc` name for data sources**: Must be named exactly `"doc"` to access search API.
- **API token shown once**: Copy immediately after generation; cannot be retrieved later. Revoke and regenerate if lost.
- **Voicemail Detection v2 requires agent greeting**: Campaign will not save without greeting configured.
- **Batch CSV must have reference_id and target**: Missing columns cause upload failure.
- **Campaign hours are in agent timezone**: Set agent timezone before creating campaign.
- **Tool changes affect active workflows**: Updating tool while workflow is active updates all future sessions; does not rerun past sessions.
- **Prompt versions are immutable**: Restoring old version creates new version; does not overwrite.
- **Variables in prompts use double braces**: `{{ vars.name }}` not `{ vars.name }` or `[[ vars.name ]]`.
- **Pauses in messages are voice-only**: `[[Pause, 1s]]` syntax has no effect in chat/email.
- **Call filtering incurs per-number lookup cost**: Each number in batch is looked up via Twilio; no cost if filter has no rules.
- **Responsive Dialogue requires explicit configuration**: Not enabled by default; must toggle in channel telephony settings.
- **Session labels are immutable**: Once added, cannot be updated or deleted.
- **Webhook authentication is optional**: If `auth_values` not provided, webhook sends unauthenticated POST/PUT/PATCH.

## Verification checklist

Before deploying or running a campaign:

- [ ] Prompt references all tools by exact name (no typos)
- [ ] All required tool parameters are defined in schema
- [ ] Static parameters with defaults match tool endpoint parameter names
- [ ] Data source tool has `doc` static parameter with correct data source names
- [ ] Agent has prompt, message, timezone, and STT provider set
- [ ] Message uses correct variable syntax: `{{ vars.column_name }}`
- [ ] Campaign has channel, display ID, rate, and hours configured
- [ ] Batch CSV has `reference_id` and `target` columns
- [ ] Batch CSV phone numbers are in format +14155551234
- [ ] Voicemail Detection v2 campaign has agent greeting configured
- [ ] Test agent via Voice or Chat before deploying to live channel
- [ ] Test campaign with single number before bulk upload
- [ ] Webhook URL is HTTPS and responds with 2xx status
- [ ] API token is stored securely and not committed to version control
- [ ] Session labels and insights are reviewed for accuracy

## Resources

**Comprehensive page navigation**: https://docs.syllable.ai/llms.txt

**Critical documentation**:
- [How Syllable Works](https://docs.syllable.ai/introsyllable/HowSyllableWorks) — platform architecture and core concepts
- [Agents](https://docs.syllable.ai/workspaces/Agents) — agent creation, configuration, testing
- [Prompts](https://docs.syllable.ai/resources/Prompts) — prompt engineering, LLM models, variables, versioning
- [Tools](https://docs.syllable.ai/resources/Tools) — tool schema, services, authentication, step tools
- [Campaigns](https://docs.syllable.ai/workspaces/Campaigns) — outbound campaigns, batches, voicemail detection, webhooks
- [Channels](https://docs.syllable.ai/resources/Channels) — channel setup, Twilio integration, responsive dialogue
- [SDK Overview](https://docs.syllable.ai/sdk-guides/Overview) — SDK setup and API key management

---

> For additional documentation and navigation, see: https://docs.syllable.ai/llms.txt