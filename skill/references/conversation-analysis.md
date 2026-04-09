# Conversation Analysis — Performance Review & Feedback Loop

Analyzing real conversations is the only reliable way to improve voice agents. This reference defines how to retrieve, assess, and act on conversation data.

---

## 1. Retrieval

### MCP Tools

- `list_conversations(agent_id, page_size, call_start_after_unix, call_start_before_unix)` → overview
- `get_conversation(conversation_id)` → full transcript + metadata

### Sampling Strategy

| Purpose | Sample Size | Selection |
|---|---|---|
| Initial quality check (new agent) | 5–10 calls | First 10 after deployment |
| Weekly performance review | 15–20 calls | Random from last 7 days |
| Issue investigation | All calls in period | Filter by duration < 30s (drop-offs) or > 10min (stuck) |
| Pre-launch audit | 20–30 calls | Mix of short, medium, long calls |

### Red Flag Filters

Pull these conversations first — they reveal the worst failures:

| Filter | Why |
|---|---|
| Duration < 30 seconds | Caller hung up immediately — first message or routing broken |
| Duration > 10 minutes | Agent couldn't resolve — stuck loop or missing knowledge |
| Multiple agent transfers (3+) | Routing failure or misclassification |
| Conversation ended by agent (not caller) | Silence timeout or max duration hit |

---

## 2. Single Conversation Assessment

For each conversation, evaluate these five dimensions:

### 2.1 Goal Completion

- Did the agent achieve its primary objective?
- Was the caller's actual question answered?
- Was the outcome positive for the caller?
- Did the conversation end with clear next steps?

**Score:** Completed / Partially Completed / Failed

### 2.2 Conversation Quality

- Response length: Were responses ≤ 2 sentences?
- Question discipline: One question per turn?
- Tone: Appropriate for the context?
- Flow: Did the agent follow its conversation flow (steps)?
- Natural: Did it sound like a real person or a script?

**Score:** Excellent / Good / Needs Improvement / Poor

### 2.3 Error Detection

| Error Type | What to Look For | Severity |
|---|---|---|
| Hallucination | Agent stated facts not in KB or prompt | Critical |
| Misclassification | Routed to wrong subagent | Critical |
| Language switch | Agent responded in English | High |
| Guardrail breach | Answered a blocked topic | High |
| Filler overuse | "Gerne!", "Das ist eine gute Frage." | Medium |
| Too verbose | > 2 sentences per turn | Medium |
| Missed confirmation | Didn't repeat back collected data | Medium |
| No transfer announcement | Transferred without telling caller | Medium |
| Wrong formality | Used "Du" instead of "Sie" | Low |

### 2.4 Missed Opportunities

- Could the agent have resolved the issue without transfer?
- Did the agent miss a chance to collect lead data?
- Was there a natural upsell/cross-sell moment?
- Could the agent have been more empathetic at a key moment?

### 2.5 Technical Issues

| Issue | Indicator in Transcript |
|---|---|
| ASR errors | Agent responds to something the caller didn't say |
| Latency | Long gaps in transcript timestamps |
| Voice quality | N/A from transcript — need audio review |
| Timeout | Conversation ended abruptly with timeout message |
| Tool failure | Agent says "Ich habe gerade leider keinen Zugriff" |

---

## 3. Aggregate Analysis (Multiple Conversations)

### Performance Metrics

Calculate across 20+ conversations:

| Metric | Formula | Target |
|---|---|---|
| **Completion Rate** | Completed / Total | > 80% |
| **Average Duration** | Sum(duration) / Total | 2–4 min (varies by type) |
| **Drop-off Rate** | Duration < 30s / Total | < 10% |
| **Transfer Rate** | Transferred to human / Total | < 30% |
| **Stuck Rate** | Duration > 10min / Total | < 5% |
| **First-Contact Resolution** | Resolved without callback / Total | > 60% |

### Benchmark by Agent Type

| Agent Type | Avg Duration | Completion Target | Transfer Target |
|---|---|---|---|
| Routing / Reception | 0.5–1.5 min | 90% | 15% |
| Appointment Scheduling | 2–3 min | 80% | 20% |
| Damage / Intake Reports | 3–5 min | 75% | 30% |
| Sales / Lead Qualification | 3–6 min | 70% | 40% |
| Technical Support | 4–8 min | 65% | 35% |
| Complaint Handling | 3–5 min | 60% | 45% |

### Pattern Recognition

Look for recurring patterns across conversations:

**Common routing failures:**
- Intent X consistently misrouted to Agent Y → fix transition condition
- Ambiguous intents (caller says "Ich habe eine Frage" — too vague) → add disambiguation step

**Common knowledge gaps:**
- Same question unanswered 3+ times → add to KB
- Agent deflects with "Das weiß ich leider nicht" for an answerable question → KB missing entry

**Common prompt failures:**
- Agent gives long monologues → output rules not enforced, reinforce in prompt
- Agent asks multiple questions in one turn → question rules not enforced
- Agent uses filler phrases → banned fillers not listed or not strong enough

**Common guardrail failures:**
- Agent discusses competitors → topic guardrail missing or phrased too loosely
- Agent improvises facts → hallucination guardrail needs reinforcement
- Agent reveals being AI unprompted → identity protection too weak

---

## 4. The Feedback Loop — From Analysis to Prompt Fix

This is the critical step most people skip. Every finding must map to a specific change.

### Feedback Action Matrix

| Finding | Action | Where to Change |
|---|---|---|
| Agent too verbose | Add/reinforce "Max 2 sentences per turn" | OUTPUT RULES section |
| Agent hallucinates about X | Add explicit "Never discuss X" + deflection | TOPIC GUARDRAILS |
| Agent misroutes intent Y | Add negative example to transition condition | Workflow transition |
| Agent doesn't know answer Z | Add atomic fact to KB | Knowledge Base |
| Agent ignores caller name | Add "Use caller_name if available" | INCOMING CONTEXT section |
| Agent doesn't confirm data | Add "Always repeat back: name, number, date" | OUTPUT RULES |
| Callers ask Q frequently | Add FAQ entry to KB | Knowledge Base |
| Agent sounds robotic | Reduce stability, add personality traits | IDENTITY + voice config |
| Long silence during tool call | Add filler phrase instruction | TASK or CONVERSATION FLOW |
| Caller drops after first message | Rewrite first_message | First message |
| Agent stuck in loop | Add turn limit, strengthen exit conditions | CONVERSATION FLOW |

### Change Implementation

1. **Identify** the finding from conversation analysis
2. **Locate** the exact section/line to change (using the matrix above)
3. **Draft** the specific prompt change (English, exact phrasing)
4. **Apply** via PATCH API (see `agent-api-operations.md`)
5. **Test** with 3–5 test conversations
6. **Verify** improvement in next analysis cycle

### Change Prioritization

| Priority | Criteria | Timeline |
|---|---|---|
| P1 — Critical | Hallucination, misrouting, guardrail breach, privacy violation | Fix within hours |
| P2 — Important | Verbosity, missed confirmations, knowledge gaps, wrong tone | Fix within 1 week |
| P3 — Polish | Filler phrases, minor flow issues, edge case handling | Next iteration cycle |

---

## 5. Reporting Template

After each analysis cycle, document:

```
================================================================
CONVERSATION ANALYSIS REPORT
Agent: [Name]
Period: [Date range]
Conversations Analyzed: [N]
================================================================

METRICS:
- Completion Rate: [X%] (target: [Y%])
- Average Duration: [X min]
- Drop-off Rate: [X%]
- Transfer Rate: [X%]

TOP 3 ISSUES:
1. [Issue] — [X occurrences] — [Impact] — [Recommended fix]
2. [Issue] — [X occurrences] — [Impact] — [Recommended fix]
3. [Issue] — [X occurrences] — [Impact] — [Recommended fix]

CHANGES APPLIED:
- [Change 1] — Applied via PATCH on [date]
- [Change 2] — Applied via PATCH on [date]

NEXT REVIEW: [Date]
================================================================
```

---

## 6. Continuous Improvement Cycle

```
Deploy Agent
    ↓
Collect 20+ Conversations (1–2 weeks)
    ↓
Run Analysis (this framework)
    ↓
Identify Top 3 Issues
    ↓
Draft Prompt/KB Changes
    ↓
Apply via PATCH API
    ↓
Test (3–5 conversations)
    ↓
Verify in Next Cycle
    ↓
(repeat)
```

Target cadence: Every 2 weeks for the first 2 months, then monthly.

---

## 7. Post-Call Webhooks — Detailed Reference

### Three Webhook Types

| Type | Payload | Trigger |
|---|---|---|
| `post_call_transcription` | Full conversation data + analysis | Every completed call |
| `post_call_audio` | Base64-encoded MP3 audio | Every completed call (if enabled) |
| `call_initiation_failure` | Failure reason + metadata | Outbound call fails to connect |

### Transcription Webhook Payload Structure

```json
{
  "type": "post_call_transcription",
  "data": {
    "agent_id": "...",
    "conversation_id": "...",
    "status": "done",
    "transcript": [
      {
        "role": "agent",
        "message": "Willkommen bei AcmeSoft...",
        "start_time_ms": 0,
        "end_time_ms": 3200
      },
      {
        "role": "user",
        "message": "Ich habe eine Frage...",
        "start_time_ms": 3500,
        "end_time_ms": 6800
      }
    ],
    "metadata": {
      "start_time_unix": 1712345678,
      "end_time_unix": 1712345800,
      "call_duration_secs": 122,
      "cost": { "total_cost_usd": 0.16 },
      "phone": { "caller_id": "+49...", "called_number": "+49..." },
      "feedback": { "score": null, "text": null }
    },
    "analysis": {
      "evaluation_results": {},
      "data_collection_results": {},
      "call_successful": "yes",
      "transcript_summary": "Caller asked about pricing..."
    },
    "conversation_initiation_client_data": {
      "dynamic_variables": { "customer_name": "Herr Mueller" }
    }
  }
}
```

### Call Failure Webhook

```json
{
  "type": "call_initiation_failure",
  "data": {
    "failure_reason": "busy | no-answer | unknown",
    "metadata": { "sip_status_code": 486 }
  }
}
```

### HMAC Signature Verification

Webhooks include an `elevenlabs-signature` header. Verify with your shared secret:

**Python:**
```python
from elevenlabs import ElevenLabs
event = ElevenLabs.construct_event(payload, signature, secret)
```

**JavaScript:**
```javascript
import { ElevenLabs } from 'elevenlabs';
const event = ElevenLabs.constructEvent(payload, signature, secret);
```

### Retry Behavior

- Must return HTTP 200
- Auto-disabled after 10+ consecutive failures if last success > 7 days ago or never succeeded
- HIPAA-compliant: failed webhooks cannot be retried

### IP Whitelisting (for webhook security)

| Region | IPs |
|---|---|
| US | 34.67.146.145, 34.59.11.47 |
| EU | 35.204.38.71, 34.147.113.54 |
| Asia | 35.185.187.110, 35.247.157.189 |
| EU Data Residency | 34.77.234.246, 34.140.184.144 |
| India Data Residency | 34.93.26.174, 34.93.252.69 |

### Audio Webhooks

Delivered via chunked transfer encoding for large files. Configurable independently at workspace and agent levels.

---

## 8. Smart Search — Semantic Conversation Search

Find conversations by keyword OR meaning across all conversation history (semantic search). Available in the ElevenLabs Dashboard and via API. Use this to find patterns that keyword search misses — e.g., search for "frustrated caller" finds conversations where the caller expressed frustration even without using that exact word.
