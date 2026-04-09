# Tools & Security Reference — Integrations, Authentication & Production Safety

This reference covers how to connect external tools to ElevenLabs agents, how to secure those connections, and how to handle tool calls gracefully in voice conversations.

---

## 1. Tool Categories

| Category | Execution | Use Case |
|---|---|---|
| **Server Tools** (Webhooks) | ElevenLabs infra → your API | CRM lookup, calendar booking, ticketing, email |
| **Client Tools** | User's browser/device | UI events, DOM manipulation, client-side logic |
| **MCP Tools** | External MCP servers (SSE/HTTP) | Third-party MCP server capabilities |
| **System Tools** | ElevenLabs platform (built-in) | End call, transfer, DTMF, voicemail detection |

**Rule:** Only connect tools to the specific subagent that needs them. Giving every agent access to every tool increases prompt complexity, response latency, and attack surface.

---

## 2. Server Tools (Webhooks) — Complete Setup

### Configuration Fields

| Field | Description |
|---|---|
| **Name** | Descriptive identifier (helps LLM decide when to call it) |
| **Description** | When and how the LLM should invoke this tool — be specific |
| **Method** | GET, POST, PUT, PATCH |
| **URL** | Base endpoint; supports path parameters: `/api/contacts/{contact_id}` |
| **Headers** | Custom headers for auth; type **Value** (plain text) or **Secret** (workspace secret) |
| **Content-Type** | `application/json` (default) or `application/x-www-form-urlencoded` |

### Parameter Types

| Type | Where | Example |
|---|---|---|
| **Path** | Variables in curly braces in URL | `/contacts/{contact_id}` |
| **Query** | Key-value pairs appended to URL | `?email=john@example.com` |
| **Body** | POST/PUT/PATCH payload | `{"name": "John", "company": "AcmeSoft"}` |

### Parameter Best Practices

```
TOOL PARAMETER RULES:
- Include format expectations in EVERY parameter description.
  Example: "Email address in standard format, e.g. 'john@example.com'"
- Speech-to-text may produce "john at gmail dot com" — include
  this in the description so the LLM normalizes before sending.
- Mark parameters as Required only when truly mandatory.
- Use descriptive names: "customer_email" not "email", "appointment_date"
  not "date".
```

### Error Handling Mode

| Mode | Behavior | Use When |
|---|---|---|
| `auto` (default) | Determines handling based on tool type | General use |
| `summarized` | LLM-generated summary of error sent to agent | Agent should explain the issue to caller |
| `passthrough` | Raw error sent to agent | Debugging / development |
| `hide` | Error not shared with agent at all | Errors the caller should never know about |

### Response → Dynamic Variable Updates

Server tool responses (JSON) can update dynamic variables via dot-notation paths:

```
Response: {"user": {"name": "Mueller", "tier": "premium"}}
Variable mapping: customer_name → user.name
                  customer_tier → user.tier
```

These variables become available via `{{customer_name}}` in subsequent turns, system prompt interpolation, and other tool calls.

---

## 3. Authentication & Security

### 3.1 Workspace Auth Connections

ElevenLabs provides a workspace-level authentication system for tool connections.

| Auth Method | Details | Use For |
|---|---|---|
| **OAuth2 Client Credentials** | Client ID + Secret + Token URL. Auto token refresh. | Salesforce, HubSpot, enterprise APIs |
| **OAuth2 JWT Bearer** | JWT signing secret + Algorithm (HS256). Claims: issuer, audience, subject. | Machine-to-machine APIs |
| **HTTP Basic Auth** | Username + Password | Legacy APIs |
| **Bearer Token** | Token stored as workspace secret | Most REST APIs, Cal.com, simple integrations |
| **Custom Headers** | Arbitrary header name + value | Proprietary auth, API key headers |

**Environment resolution:** Auth connections can resolve per environment (dev/staging/production use different OAuth clients or endpoints).

### 3.2 Secrets Manager

```
SECURITY RULES FOR SECRETS:
- NEVER hardcode API keys, tokens, or passwords in tool configurations.
- Use workspace secrets: Add header → select Secret type → Create New Secret.
- The LLM never sees actual secret values — they are injected at runtime.
- Sensitive dynamic variables: prefix with "secret__" to prevent LLM exposure.
- Template syntax for environment variables: "{{system_env__label}}"
- Every environment variable requires a "production" value as fallback.
- URLs in environment variables must begin with "https://" before any
  variable references.
```

### 3.3 Agent Access Authentication

Two methods to protect agent access (use one, not both):

**Signed URLs (recommended for client-side):**
```
1. Your server requests a signed URL from ElevenLabs using your API key
2. Returns a temporary WebSocket URL (expires in 15 minutes)
3. Conversation can continue beyond expiry once connected
4. Each user session gets a new signed URL
5. NEVER expose your API key client-side
```

**Allowlists:**
- Up to 10 unique hostnames per agent
- Exact matching (subdomains need separate entries)
- Example: `example.com`, `app.example.com`, `localhost:3000`

### 3.4 Tool Security Checklist

Before deploying any tool connection:

- [ ] API keys stored as workspace secrets (never hardcoded)
- [ ] Auth connection uses appropriate method (OAuth2 for enterprise, Bearer for simple APIs)
- [ ] Least-privilege: tool only has permissions it needs (read-only where possible)
- [ ] Tool only assigned to the subagent that needs it
- [ ] Error handling mode set appropriately (never `passthrough` in production)
- [ ] Agent prompt includes instruction to never expose tool names, API IDs, or endpoints to caller
- [ ] Rate limits of external API considered (prevent agent from hammering your endpoint)
- [ ] Tool response does not contain sensitive data that would be spoken to caller
- [ ] Signed URLs used for client-side agent access
- [ ] Post-call webhook uses HMAC verification
- [ ] Webhook endpoint IP-whitelisted to ElevenLabs source IPs

---

## 4. Tool Call Behavior in Voice

### 4.1 The Silence Problem

Tool calls introduce latency. While the tool executes, the caller hears silence. Unmanaged silence > 2 seconds feels broken to callers.

### 4.2 Mitigation Strategies

**Strategy 1: Prompt-based filler phrase (BEFORE tool call)**
```
In the system prompt CONVERSATION FLOW section:
"Before looking up information, say: 'Einen Moment, ich schaue kurz nach.'
Then execute the tool. Resume with the answer."
```

**Strategy 2: Tool Call Sounds (ambient audio)**
```json
{
  "tool_call_sound": "typing",
  "tool_call_sound_behavior": "always_play"
}
```

| Sound | Effect |
|---|---|
| `none` | Silence (default) |
| `typing` | Keyboard typing sounds |
| `elevator_music_1-4` | Background music |

| Behavior | When It Plays |
|---|---|
| `with_pre_speech` (default) | Only when agent speaks before executing |
| `always_play` | During every tool execution (may feel abrupt) |

Tool-level settings override integration-level defaults.

**Strategy 3: Soft Timeout (auto filler)**
Automatically speaks a filler message after configurable silence:
```json
{
  "turn": {
    "soft_timeout_config": {
      "timeout_seconds": 3.0,
      "message": "Ich schaue gerade nach."
    }
  }
}
```

### 4.3 Tool Failure Handling in Prompts

```
TOOL FAILURE RECOVERY:
Include this in every agent that uses tools:

"If a tool call fails or returns an error:
1. Acknowledge to the caller: 'Ich habe gerade leider keinen Zugriff
   auf das System.'
2. Never guess the information the tool would have provided.
3. Offer an alternative: 'Darf ich Ihnen einen Rueckruf anbieten?'
4. After 2 consecutive tool failures, transfer to a human:
   'Ich verbinde Sie mit einem Kollegen, der das direkt pruefen kann.'
5. Never expose technical error details to the caller. No API names,
   status codes, or error messages."
```

### 4.4 Latency Optimization for Tools

| Lever | Impact |
|---|---|
| Keep tool calls lightweight (GET > POST, fewer fields) | Major |
| Cache frequently-requested data in KB instead of tool | Major |
| Use regional API endpoints (EU tools for EU agents) | Medium |
| Limit response payload to needed fields only | Medium |
| Fast LLM (Gemini Flash) for tool-heavy routing agents | Medium |
| Tool call sounds to mask perceived latency | UX improvement |

**Target:** Tool response < 2 seconds. If your API consistently takes > 3 seconds, consider moving that data into a knowledge base that updates via webhook/cron.

---

## 5. MCP Server Tools

### Configuration

| Field | Description |
|---|---|
| **Server URL** | May contain secret keys — store as workspace secrets |
| **Secret Token** (optional) | Authorization header value |
| **HTTP Headers** (optional) | Additional auth headers |
| **Transport** | SSE (Server-Sent Events) or HTTP Streamable |

### Tool Approval Control

| Mode | Behavior | Recommended For |
|---|---|---|
| **Always Ask** | Permission required before each tool execution | Default / production |
| **Fine-Grained** | Per-tool: auto-approved, requires-approval, or disabled | Mature deployments |
| **No Approval** | Unrestricted access | Testing only |

**Best practice:** Auto-approve read-only tools. Require approval for data-modifying tools.

### Security Warnings

```
MCP TOOL SECURITY:
- YOU are responsible for vetting third-party MCP servers.
- MCP tools are NOT available for Zero Retention Mode.
- MCP tools are NOT available for HIPAA-compliant deployments.
- Data sharing with third-party services is NOT controlled by ElevenLabs.
- Always review what data each MCP tool can access before connecting.
```

---

## 6. Real-World Integration Patterns

### Pattern A: CRM Lookup (HubSpot)

**Auth:** Bearer token stored as workspace secret.
**Required scopes:** `crm.objects.contacts.read`, `crm.objects.contacts.write`, `tickets`

| Tool | Method | Endpoint | Purpose |
|---|---|---|---|
| `search_contact` | POST | `/crm/v3/objects/contacts/search` | Find caller by phone/email |
| `get_previous_calls` | GET | `/crm/v3/objects/contacts/{id}/associations/calls` | Load call history |
| `create_ticket` | POST | `/crm/v3/objects/tickets` | Create support ticket with associations |

**Prompt integration:**
```
"After successful authentication, call search_contact with the caller's
phone number. If a contact is found, greet them by name and reference
their company. If not found, proceed with standard greeting and collect
their information."
```

### Pattern B: CRM Lookup (Salesforce)

**Auth:** OAuth2 Client Credentials via Workspace Auth Connection.
Token URL: `https://your-domain.my.salesforce.com/services/oauth2/token`
Extra params: `{"grant_type": "client_credentials"}`
Automatic token refresh.

| Tool | Method | Endpoint | Purpose |
|---|---|---|---|
| `search_records` | GET | `/services/data/v62.0/query/?q=SOQL` | Search records with SOQL |
| `get_record` | GET | `/services/data/v62.0/sobjects/{type}/{id}` | Get specific record |
| `create_record` | POST | `/services/data/v62.0/sobjects/{type}` | Create new record |

### Pattern C: Calendar Booking (Cal.com)

**Auth:** Bearer token with Cal.com API key.

| Tool | Method | Endpoint | Purpose |
|---|---|---|---|
| `get_available_slots` | GET | `/v2/slots` | Check availability |
| `book_meeting` | POST | `/v2/bookings` | Book a meeting |

**Prompt integration:**
```
"When the caller wants to schedule a meeting:
Step 1: Ask for their preferred date.
Step 2: Call get_available_slots for that date.
Step 3: Offer 2-3 available time slots.
Step 4: Once confirmed, call book_meeting with their name and email.
Step 5: Confirm: 'Ihr Termin am [date] um [time] ist eingetragen.'"
```

### Pattern D: Ticket Creation

```
"When an issue cannot be resolved:
Step 1: Summarize the issue in 1-2 sentences.
Step 2: Call create_ticket with: caller_name, caller_company, issue_summary,
  priority (high/medium/low based on urgency assessment).
Step 3: Confirm to caller: 'Ich habe ein Ticket fuer Sie erstellt.
  Ein Kollege meldet sich bei Ihnen.' Do NOT read back the ticket number."
```

### Pattern E: Voicemail Detection

System tool for outbound calls. Detects voicemail greetings via LLM analysis.

```json
{
  "type": "system",
  "name": "voicemail_detection",
  "parameters": {
    "reason": "Detected voicemail greeting — leaving message",
    "message": "Guten Tag, hier ist {{agent_name}} von {{company}}. Wir wollten uns kurz bei Ihnen melden. Sie erreichen uns unter {{callback_number}}. Auf Wiedersehen."
  }
}
```

Auto-terminates after message delivery. Dynamic variables in the message are resolved at runtime.

---

## 7. Tool Assignment Matrix

| Tool Type | Routing Agent | Sales Agent | Support Agent | Lead Capture | Complaint |
|---|---|---|---|---|---|
| CRM search | — | Read | Read + Write | — | Read |
| Calendar/scheduling | — | Book | Book (callbacks) | — | — |
| Ticketing system | — | — | Create | — | Create |
| Knowledge base (RAG) | Basic/FAQ | Products/Pricing | Technical docs | — | Policies/SLA |
| Phone transfer | Yes | Yes | Yes | — | Yes |
| Email sending | — | Lead confirm | Ticket reference | Confirm | — |
| Voicemail detection | — | — | — | — | — |

**Anti-pattern:** Giving the routing agent access to CRM or ticketing tools. The routing agent should ONLY classify intent and transfer — not perform lookups.

---

## 8. Prompt Rules for Tool-Using Agents

Include these rules in the system prompt of EVERY agent that has tools:

```
TOOL USAGE RULES:
- Only call a tool when you need information you do not have.
- Never call the same tool twice with identical parameters in one conversation.
- Never expose tool names, API endpoints, parameter names, or technical
  infrastructure to the caller.
- If a tool returns an error, say: "Ich habe gerade leider keinen Zugriff."
  Never say: "Der API-Call ist fehlgeschlagen."
- If a tool returns data, paraphrase it naturally. Never read raw JSON
  or technical fields to the caller.
- Before calling a tool, tell the caller: "Einen Moment, ich schaue kurz nach."
- After 2 consecutive tool failures, offer a callback or transfer to human.
- Never make up data that a tool would normally provide. If the tool
  fails, treat the information as unknown.
- Confirm tool-returned data naturally before acting on it:
  "Ich sehe, dass Ihr Termin am fuenfzehnten Mai um vierzehn Uhr ist.
  Stimmt das?"
```

---

## 9. Rate Limits & Error Codes

### Concurrency-Based Limits

Limits are based on concurrent calls, not per-minute thresholds. Depends on subscription tier. Burst pricing allows up to 3x normal concurrency at 2x cost.

### HTTP Error Codes from Tool Calls

| Code | Meaning | Agent Should |
|---|---|---|
| 200 | Success | Process response normally |
| 400 | Bad request (wrong parameters) | Log error, tell caller "leider keinen Zugriff" |
| 401/403 | Authentication failed | Critical — check secrets/auth config |
| 404 | Resource not found | "Die Information konnte ich leider nicht finden." |
| 429 | Rate limited | Retry after delay or offer callback |
| 500+ | Server error | "Leider keinen Zugriff. Darf ich einen Rueckruf anbieten?" |

### API Rate Limit Error Codes

| Code | Meaning | Action |
|---|---|---|
| `too_many_concurrent_requests` | Tier limit exceeded | Queue or reject call |
| `system_busy` | Platform congestion | Exponential backoff |

### Gotchas

- 180-second conversation pause = forced disconnection
- Tool calls during active speech may be interrupted by caller barge-in
- Quick tool operations (<500ms) may not trigger tool call sounds
- `application/x-www-form-urlencoded` body type needed for OAuth token endpoints and legacy payment processors
