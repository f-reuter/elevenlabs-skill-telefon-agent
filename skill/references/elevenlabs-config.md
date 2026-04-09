# ElevenLabs Configuration Reference — Technical Parameters

---

## 1. Core Configuration Parameters

### System Prompt (`system_prompt`)
- Contains the complete 10-section prompt (see `references/prompt-template.md`)
- Written in **English**
- Maximum recommended length: ~3000 tokens (longer prompts slow response time)
- If prompt exceeds 3000 tokens, move detailed knowledge into RAG/knowledge base

### First Message (`first_message`)
- Written in the **target language**
- Maximum 3 sentences
- Must end with a question
- See `references/prompt-template.md` Section "First Message" for templates

### Language (`language`)
- Set to the caller's language code: `de` (German), `en` (English), etc.
- This controls STT (speech-to-text) and TTS (text-to-speech) language
- The system prompt language (English) is independent of this setting

---

## 2. Temperature Settings

Temperature controls LLM creativity/randomness. Lower = more deterministic and reliable. Higher = more natural and varied but less predictable.

| Agent Type | Temperature | Rationale |
|-----------|-------------|-----------|
| Routing / Reception | 0.2–0.3 | Must classify reliably. No creativity needed. |
| Customer Verification | 0.2–0.3 | Data validation. Deterministic. |
| Lead Capture | 0.2–0.3 | Data collection. Structured and predictable. |
| Technical Support | 0.3–0.4 | Accuracy critical. Minimal improvisation. |
| Complaint Handling | 0.4–0.5 | Needs empathy and natural phrasing but must stay on script. |
| Sales / Pricing | 0.5–0.6 | Conversational flexibility needed. Still factual. |

**Rule of thumb:** If the agent GIVES information, lower temperature (accuracy matters). If the agent COLLECTS information, lowest temperature (structure matters). If the agent needs to be PERSUASIVE or EMPATHETIC, moderate temperature (naturalness matters).

---

## 3. Max Tokens

Controls maximum response length per turn. For voice agents, shorter is always better.

| Agent Type | Max Tokens | Rationale |
|-----------|------------|-----------|
| Routing | 100–150 | Short routing responses only |
| Verification | 100–150 | Yes/no confirmations, brief questions |
| Lead Capture | 100–150 | One question at a time |
| Support | 150–200 | May need slightly longer explanations |
| Sales | 150–200 | Product descriptions need some room |
| Complaint | 150–200 | Empathetic responses need some room |

**Never exceed 250 tokens for voice agents.** At ~150 words per minute speaking rate, 250 tokens ≈ 30+ seconds of uninterrupted speech — far too long.

---

## 4. Voice Configuration

### Voice Selection

- **Consistency:** Use the same voice for all subagents in a workflow UNLESS the workflow simulates transferring to a different person
- **Gender and age:** Match to brand expectations and caller demographics
- **Language support:** Verify the selected voice supports the target language natively (not just through multilingual mode)

### Voice ID

- Each ElevenLabs voice has a unique `voice_id`
- Document the voice_id in the workflow documentation
- When switching between subagents, the voice changes only if a different voice_id is configured

### Voice Settings (fine-tuning)

| Parameter | Range | Recommendation |
|-----------|-------|----------------|
| Stability | 0.0–1.0 | 0.5–0.7 for natural speech. Higher = more consistent but robotic. |
| Similarity Boost | 0.0–1.0 | 0.7–0.85 for clarity. Higher = closer to original voice. |
| Style | 0.0–1.0 | 0.0–0.3 for professional. Higher = more expressive (less predictable). |
| Speed | 0.7–1.3 | 1.0 default. 0.9 for complex info, 1.1 for simple routing. |

---

## 5. Tool & API Integration

### When to Connect Tools

Only connect tools to the specific subagent that needs them. Giving every agent access to every tool increases prompt complexity and response time.

| Tool Type | Assign To |
|-----------|----------|
| CRM lookup (customer data) | Verification agent, Support agent |
| Calendar/scheduling API | Support agent (for callbacks), Sales agent |
| Ticketing system | Support agent, Complaint agent |
| Knowledge base / RAG | Each agent gets its own relevant KB |
| Phone transfer API | Any agent that can escalate to human |
| Email sending API | Lead Capture (confirmation), Support (ticket reference) |

### Tool Call Behavior in Voice

Tool calls introduce latency. While the tool executes, the caller hears silence. Mitigate with:

1. **Filler before tool call:** "Einen Moment, ich schaue kurz nach."
2. **Keep tool calls fast:** Use lightweight API calls, not complex queries
3. **Fallback if tool fails:** "Ich habe gerade leider keinen Zugriff. Darf ich Ihnen einen Rückruf anbieten?"

Never let the caller wait more than 5 seconds in silence during a tool call.

---

## 6. Knowledge Base Configuration

### Structure

- One knowledge base per domain, assigned to relevant subagents
- Content in the **target language** (the agent retrieves and paraphrases in the caller's language)
- Chunk documents into logical sections with clear headings
- Keep individual chunks under 500 words for reliable retrieval

### Document Format for RAG

Use structured markdown:

```markdown
# Product: [Product Name] Standard License
## Price: [X] EUR/month (net)
## Included:
- [Feature 1]
- [Feature 2]
- [Support level] (Mon-Fri [hours])
## NOT included:
- [Excluded feature 1]
- [Excluded feature 2]
## Upsell path: Premium ([Y] EUR/month) adds [additional features]
## Common objection: "Too expensive" → [Counter-argument from sales playbook]
```

### Knowledge Base Assignment Matrix

| KB Name | Content | Agents |
|---------|---------|--------|
| Product Catalog | Features, pricing, tiers, comparison | Sales, Routing (basic) |
| Technical Docs | Setup guides, troubleshooting, known issues | Support |
| SLA & Policies | Service levels, escalation procedures, refund policy | Complaint, Support |
| FAQ | Common questions and answers | Routing, Sales, Support |
| Company Info | Opening hours, address, team contacts | Routing |

### "Do Not Know" Markers

Include explicit markers in the knowledge base for topics the agent should NOT answer from this KB:

```markdown
# PRICING NOTE
Enterprise pricing is negotiated individually. NEVER quote enterprise
prices. Always say: "Enterprise-Preise erstellen wir individuell.
Darf ich einen Termin mit unserem Vertriebsleiter für Sie vereinbaren?"
```

---

## 7. Webhook & Event Configuration

### Useful Webhooks

| Event | Use Case |
|-------|----------|
| `conversation.started` | Log call start, initialize CRM record |
| `conversation.ended` | Log call end, trigger follow-up workflows |
| `agent.transferred` | Log transitions between subagents |
| `tool.called` | Monitor tool usage and errors |

### Post-Call Processing

After call ends, use webhooks to:
1. Save conversation transcript
2. Create/update CRM record
3. Send follow-up email (if promised during call)
4. Create support ticket (if issue was logged)
5. Trigger lead nurture sequence (if lead was captured)

---

## 8. Configuration Checklist

Before deploying each subagent:

- [ ] `system_prompt` is in English, follows 10-section schema
- [ ] `first_message` is in target language, max 3 sentences, ends with question
- [ ] `language` matches caller's language
- [ ] `temperature` is set appropriately for agent type
- [ ] `max_tokens` is 150–200 (never >250 for voice)
- [ ] `voice_id` is set and consistent with workflow voice strategy
- [ ] Voice stability/similarity/style are tuned for professional context
- [ ] Tools are connected only where needed (no global tool access)
- [ ] Knowledge base is assigned with relevant content only
- [ ] Webhooks are configured for logging and post-call processing

---

## 9. MCP Server Configuration

The ElevenLabs MCP server (`elevenlabs-mcp`) connects Claude Code directly to the ElevenLabs API.

### Environment Variables

| Variable | Default | Description |
|---|---|---|
| `ELEVENLABS_API_KEY` | (required) | Your ElevenLabs API key |
| `ELEVENLABS_MCP_OUTPUT_MODE` | `files` | How generated files are returned: `files` (save to disk), `resources` (return as MCP resources), `both` |
| `ELEVENLABS_MCP_BASE_PATH` | `~/Desktop` | Base directory for file output |
| `ELEVENLABS_API_RESIDENCY` | `us` | Data residency region (enterprise only) |

### Available MCP Tools for Conversational AI

| Tool | Purpose | Cost |
|---|---|---|
| `create_agent` | Create a new Conversational AI agent | Yes |
| `get_agent` | Retrieve agent configuration | No |
| `list_agents` | List all agents | No |
| `add_knowledge_base_to_agent` | Attach KB (file, URL, or text) | Yes |
| `list_conversations` | List agent conversations | No |
| `get_conversation` | Get full transcript + metadata | No |
| `search_voices` | Search user's voice library | No |
| `search_voice_library` | Search entire ElevenLabs voice library | No |
| `get_voice` | Get voice details | No |
| `text_to_speech` | Generate speech from text | Yes |
| `text_to_voice` | Create voice from description | Yes |
| `voice_clone` | Clone voice from audio | Yes |
| `make_outbound_call` | Place outbound call via agent | Yes |
| `list_phone_numbers` | List available phone numbers | No |

### Agent Creation via MCP — Conversation Config Structure

When using `create_agent`, the MCP server internally builds this nested config:

```json
{
  "conversation_config": {
    "agent": {
      "language": "de",
      "prompt": {
        "prompt": "System prompt here...",
        "llm": "claude-3.5-sonnet",
        "tools": [{"type": "system", "name": "end_call", "description": ""}],
        "knowledge_base": [],
        "temperature": 0.4,
        "max_tokens": 150
      },
      "first_message": "Greeting here...",
      "dynamic_variables": {"dynamic_variable_placeholders": {}}
    },
    "asr": {
      "quality": "high",
      "provider": "elevenlabs",
      "keywords": ["domain-specific", "terms"]
    },
    "tts": {
      "voice_id": "voice_id_here",
      "model_id": "eleven_multilingual_v2",
      "stability": 0.55,
      "similarity_boost": 0.8,
      "optimize_streaming_latency": 3
    },
    "turn": {
      "turn_timeout": 7
    },
    "conversation": {
      "max_duration_seconds": 600
    }
  }
}
```

**Note:** The MCP `create_agent` tool does NOT support PATCH updates. To modify an existing agent, use the PATCH API directly via curl (see `agent-api-operations.md`).

---

## 10. Dynamic Variables

Dynamic variables allow you to personalize agent behavior per call. They are injected at conversation start and accessible in the system prompt via `{{variable_name}}`.

### System Variables (automatically available)

| Variable | Purpose |
|---|---|
| `system__agent_id` | Unique ID of the initiating agent (stable throughout) |
| `system__current_agent_id` | Active agent ID (changes after transfers) |
| `system__caller_id` | Caller's phone number (voice only) |
| `system__called_number` | Destination phone number (voice only) |
| `system__call_duration_secs` | Current call duration in seconds |
| `system__time_utc` | Current time in ISO format |
| `system__time` | Current time in specified timezone (human-readable) |
| `system__timezone` | User-provided timezone |
| `system__conversation_id` | Platform unique conversation identifier |
| `system__call_sid` | Twilio Call SID (Twilio only) |
| `system__agent_turns` | Total conversation turns for agent |
| `system__current_agent_turns` | Agent turns (resets on transfer) |
| `system__current_subagent_turns` | Subagent turns (resets on workflow transition) |
| `system__is_text_only` | Boolean for text-only mode |
| `system__conversation_history` | JSON-serialized full conversation history |

### Custom Variables

Define in agent config and reference in prompts:
```
System prompt: "The caller's name is {{customer_name}} from {{company}}."
```

### Secret Variables

Prefix with `secret__` to ensure they are ONLY used in dynamic variable headers and NEVER sent to the LLM provider. Critical for auth tokens and private IDs.

### URL Parameter Passing (for widget/embed)

Two methods:
- Base64 JSON: `?vars=eyJ1c2VyX25hbWUiOiJKb2huIn0=`
- Individual: `?var_user_name=John&var_account_type=premium`

### Batch Calling Variables

Upload CSV/XLS with mandatory `phone_number` column. Additional columns become dynamic variables per call (e.g., `user_name`, `company`).

**Supported data types:** String, number, and boolean only.

---

## 11. System Tools Reference

| Tool | Purpose | When to Use |
|---|---|---|
| `end_call` | Terminate the conversation | After resolution or max turns |
| `transfer_to_agent` | Transfer to another subagent in workflow | Intent routing, escalation |
| `transfer_to_number` | Transfer to external phone number | Human escalation |
| `play_keypad_touch_tone` | Generate DTMF tones | IVR navigation, extension dialing |
| `skip_turn` | Skip agent response without generating output | When caller is still speaking |
| `language_detection` | Detect caller's language automatically | Multilingual entry points |

### DTMF (play_keypad_touch_tone)

Parameters: `dtmf_tones` (string: 0-9, *, #, w=0.5s pause, W=1s pause), `reason` (optional). Works only on Twilio or SIP trunk calls. Use case: navigating third-party IVR systems.

### Transfer to Number — Three Modes

| Mode | Behavior | Provider |
|---|---|---|
| `conference` (default) | Agent joins conference, then exits | All |
| `blind` | Direct transfer without warm message | Twilio native only |
| `sip_refer` | SIP REFER protocol transfer | SIP trunk only |

Parameters: `transfer_number`, `client_message` (spoken to caller), `agent_message` (context for receiving agent), `reason`, `post_dial_digits` (DTMF for extensions).

---

## 12. LLM Model Selection for Voice Agents

| Model | Latency | Accuracy | Best For |
|---|---|---|---|
| Gemini 2.5 Flash Lite | Ultra-low | Good | Simple routing, FAQ, high-volume |
| GPT-4o | Low-medium | Very good | Balanced latency + accuracy |
| GLM 4.5 Air | Low | Good | Cost-effective general use |
| Claude Sonnet 4/4.5 | Medium | Excellent | Complex reasoning, nuanced judgment |
| Claude Opus | Higher | Best | Enterprise logic, compliance-critical |

**Rule of thumb:** Use the fastest model that can handle the task. Routing agents need speed, not reasoning power. Support agents with complex troubleshooting need accuracy.

---

## 13. Phone Number Setup

### Twilio Native Integration

1. Navigate to Phone Numbers tab in ElevenAgents
2. Provide: Label, Phone Number, Twilio Account SID, Twilio Auth Token
3. System auto-detects capabilities: purchased numbers = inbound + outbound; verified caller IDs = outbound only
4. Assign agent for inbound calls from dropdown

### SIP Trunk Integration

Supported providers: Twilio Elastic SIP Trunking, Telnyx, DIDWW, standard SIP.
TLS + SRTP encryption supported.

### Batch Calling API

```
POST /v1/convai/batch-calling
```
Upload CSV with `phone_number` column + custom variable columns. Can send immediately or schedule.

---

## 14. Enterprise Features

### Data Residency
Regions: US, EU, India. Enterprise-only. Set via `ELEVENLABS_API_RESIDENCY` env variable.

### Zero Retention Mode
Set `enable_logging=false`. PII is automatically redacted before storage. Gemini and Claude LLMs support Zero Retention.

### PII Redaction (Enterprise)
13 entity categories detected: Name, Contact, Personal, Credentials, Web, Organization, Financial, Location, Date, Unique IDs, Medical, plus specialized types. Transcripts: entities replaced with `[ENTITY_NAME]`; Audio: replaced with bleep sound.

### Compliance
- SOC2 certified
- GDPR compliant
- HIPAA BAAs available for qualifying healthcare enterprises
- SSO/RBAC: Okta, Azure AD, Google Workspace integration

### Burst Pricing
Enable per-agent to handle up to 3x normal concurrency during peak demand. Excess calls charged at 2x standard rate.
