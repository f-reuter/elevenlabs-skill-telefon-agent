# Dynamic Variables — Personalization & Runtime Data

Dynamic variables inject runtime values into system prompts, first messages, and tool parameters using `{{variable_name}}` syntax. This enables caller-specific conversations without creating separate agents.

---

## 1. System Dynamic Variables

Automatically available in every conversation — no configuration needed.

### Phone-Relevant Variables

| Variable | Description | Example Value |
|---|---|---|
| `system__caller_id` | Caller's phone number (voice calls only) | "+49 170 1234567" |
| `system__called_number` | Destination phone number (voice calls only) | "+49 30 9876543" |
| `system__call_duration_secs` | Call duration in seconds (updates live) | 120 |
| `system__time_utc` | Current UTC time (ISO format) | "2026-04-10T14:30:00Z" |
| `system__time` | Current time in specified timezone (human-readable) | "Friday, 14:30 10 April 2026" |
| `system__timezone` | User-provided timezone (must be valid tzinfo) | "Europe/Berlin" |
| `system__conversation_id` | ElevenLabs conversation ID | "conv_abc123" |
| `system__call_sid` | Twilio Call SID (Twilio calls only) | "CA1234..." |

### Agent-Tracking Variables

| Variable | Description | Resets? |
|---|---|---|
| `system__agent_id` | ID of the agent that initiated the conversation | Never (stable across transfers) |
| `system__current_agent_id` | ID of the currently active agent | On agent transfer |
| `system__agent_turns` | Total turns taken across all agents | Never |
| `system__current_agent_turns` | Turns taken by current agent | On agent transfer |
| `system__current_subagent_turns` | Turns taken by current subagent node | On workflow node transition |
| `system__is_text_only` | True if text-only mode | Never |

### Conversation History Variable

`system__conversation_history` — JSON-serialized representation of the full conversation, lazily evaluated when referenced. Useful for passing context to tools or sub-agent prompts.

```json
{
  "x-elevenlabs-history": true,
  "entries": [
    { "role": "user", "message": "Ich moechte einen Termin." },
    { "role": "agent", "message": "Gerne! Welcher Tag passt Ihnen?" },
    { "role": "agent", "tool_requests": [{"tool_name": "check_slots", "params_as_json": {"date": "2026-04-15"}}] },
    { "role": "tool", "tool_results": [{"tool_name": "check_slots", "result_value": "{\"slots\": [\"14:00\", \"15:30\"]}"}] }
  ]
}
```

Nested conversation histories in tool results are redacted to prevent recursive expansion: `[conversation_history (5 turns)]`.

**Important:** The `system__` prefix is reserved. Custom variables cannot use it.

---

## 2. Custom Dynamic Variables

Define your own variables and pass values at conversation start.

### Where to Use Them

| Location | Syntax | Example |
|---|---|---|
| System prompt | `{{customer_name}}` | "Greet the caller as {{customer_name}}." |
| First message | `{{customer_name}}` | "Guten Tag, {{customer_name}}! Wie kann ich Ihnen helfen?" |
| Tool parameters | `{{customer_id}}` | Pass as path/query/body parameter in webhook tools |
| Tool headers | `{{auth_token}}` | Bearer token for authenticated API calls |

### Supported Types

| Type | Example |
|---|---|
| String | `"Herr Mueller"` |
| Number | `1.5`, `490` |
| Boolean | `true`, `false` |

### Passing Variables at Conversation Start

**Python SDK:**
```python
from elevenlabs.conversational_ai.conversation import Conversation, ConversationInitiationData

config = ConversationInitiationData(
    dynamic_variables={
        "customer_name": "Herr Mueller",
        "customer_tier": "premium",
        "account_id": "ACC-12345"
    }
)

conversation = Conversation(client, agent_id, config=config, ...)
```

**JavaScript SDK:**
```javascript
const conversation = await Conversation.startSession({
    agentId: 'agent_id',
    dynamicVariables: {
        customer_name: 'Herr Mueller',
        customer_tier: 'premium'
    }
});
```

**Outbound Call API:**
```json
{
  "agent_id": "agent_...",
  "phone_number": "+49...",
  "dynamic_variables": {
    "customer_name": "Herr Mueller",
    "appointment_time": "14:00"
  }
}
```

---

## 3. Secret Dynamic Variables

For auth tokens or private IDs that should NEVER reach the LLM.

- Prefix with `secret__` (e.g., `secret__api_token`)
- Used ONLY in tool headers — never sent to LLM as part of prompt
- Populated the same way as normal dynamic variables

```
Tool header: Authorization: Bearer {{secret__crm_token}}
```

The LLM never sees the value. It is injected at runtime by the platform.

---

## 4. Updating Variables from Tool Responses

Server tools can CREATE or UPDATE dynamic variables when they return JSON. Use dot-notation paths to extract values.

**Tool response:**
```json
{
  "response": {
    "status": 200,
    "customer": {
      "name": "Mueller",
      "tier": "premium",
      "email": "mueller@firma.de"
    }
  }
}
```

**Variable assignments:**

| Variable Name | Dot-Notation Path | Extracted Value |
|---|---|---|
| `customer_name` | `response.customer.name` | "Mueller" |
| `customer_tier` | `response.customer.tier` | "premium" |
| `customer_email` | `response.customer.email` | "mueller@firma.de" |

For arrays: `response.users.0.email` extracts the first user's email.

These variables become available via `{{customer_name}}` in:
- Subsequent turns of the system prompt
- Other tool calls
- Agent transfer context

---

## 5. Common Patterns for Phone Agents

### Pattern A: Personalized Greeting with CRM Lookup

```
System prompt:
"Greet the caller by name using {{customer_name}} if available.
If {{customer_tier}} is 'premium', prioritize their request."

First message:
"Guten Tag, {{customer_name}}! Willkommen bei AcmeSoft. Wie kann ich Ihnen helfen?"
```

The CRM tool enriches variables after first lookup:
1. Call starts → `system__caller_id` available
2. Agent calls `search_contact` tool with `{{system__caller_id}}`
3. Tool returns customer data → variables updated via assignments
4. Agent now knows `{{customer_name}}`, `{{customer_tier}}`

### Pattern B: Outbound Call with Pre-Loaded Context

```
Dynamic variables at call initiation:
{
  "customer_name": "Frau Schmidt",
  "appointment_date": "15. Mai",
  "appointment_time": "14 Uhr",
  "doctor_name": "Dr. Meier"
}

First message:
"Guten Tag, {{customer_name}}. Hier ist die Praxis {{doctor_name}}.
Ich rufe an, um Sie an Ihren Termin am {{appointment_date}} um
{{appointment_time}} zu erinnern. Passt der Termin noch fuer Sie?"
```

### Pattern C: Turn Limit via System Variables

```
System prompt:
"You have a maximum of 15 turns. Track this using {{system__current_agent_turns}}.
When you reach turn 12, begin wrapping up the conversation.
At turn 15, offer a callback: 'Ich moechte Sie nicht laenger aufhalten.
Darf ich einen Rueckruf fuer Sie einrichten?'"
```

---

## 6. Troubleshooting

| Problem | Fix |
|---|---|
| Variables not replacing | Check exact name match (case-sensitive), double curly braces `{{ }}` |
| System variables not available | Ensure correct prefix (`system__`), check if voice-only (e.g., `caller_id`) |
| Tool assignment not working | Verify JSON response path exists, use dot-notation correctly |
| Secret variables leaking | Ensure `secret__` prefix, only use in headers (not prompt/first message) |
