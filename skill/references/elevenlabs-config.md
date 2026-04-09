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
# Product: EMMA Standard License
## Price: 490 EUR/month (net)
## Included:
- 3 workstations
- 1 year updates
- Standard support (Mon-Fri 8-18)
## NOT included:
- On-site support
- Custom integrations
- Weekend/holiday support
## Upsell path: EMMA Premium (890 EUR/month) adds 24/7 support + custom integrations
## Common objection: "Too expensive" → Compare to cost of one FTE (40-50k/year)
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
