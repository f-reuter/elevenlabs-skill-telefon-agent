# Agent API Operations — Edit, Audit & Deploy

The ElevenLabs MCP server provides `create_agent` and `get_agent` but NO update tool. To edit an existing agent without recreating it, use the PATCH API directly via curl.

---

## 1. PATCH API — Update Agents In Place

### Endpoint & Authentication

```
PATCH https://api.elevenlabs.io/v1/convai/agents/{agent_id}
Header: xi-api-key: $ELEVENLABS_API_KEY
Content-Type: application/json
```

The PATCH endpoint **merges** changes — send only the fields you want to change. Unchanged fields remain as-is.

### Complete Nested Structure

```json
{
  "name": "string",
  "tags": ["production", "german", "healthcare"],
  "conversation_config": {
    "asr": {
      "quality": "high",
      "keywords": ["[domain-specific]", "[product-name]", "[key-terms]"]
    },
    "turn": {
      "turn_timeout": 7.0,
      "initial_wait_time": 2.5,
      "silence_end_call_timeout": 30,
      "turn_eagerness": "normal",
      "spelling_patience": "auto",
      "soft_timeout_config": {
        "timeout_seconds": 5.0,
        "message": "Sind Sie noch da?"
      }
    },
    "tts": {
      "model_id": "eleven_multilingual_v2",
      "voice_id": "zKHQdbB8oaQ7roNTiDTK",
      "stability": 0.55,
      "similarity_boost": 0.8,
      "speed": 1.0,
      "expressive_mode": true
    },
    "conversation": {
      "max_duration_seconds": 600
    },
    "agent": {
      "prompt": "System prompt here...",
      "first_message": "Greeting here...",
      "language": "de",
      "llm": "claude-3.5-sonnet",
      "temperature": 0.4,
      "max_tokens": 150,
      "max_conversation_duration_message": "Vielen Dank für Ihren Anruf. Leider ist unsere Gesprächszeit abgelaufen. Bitte rufen Sie erneut an.",
      "tools": [],
      "knowledge_base": []
    }
  }
}
```

---

## 2. Common PATCH Operations

### Update System Prompt

```bash
curl -X PATCH "https://api.elevenlabs.io/v1/convai/agents/{agent_id}" \
  -H "Content-Type: application/json" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -d '{
    "conversation_config": {
      "agent": {
        "prompt": "New system prompt..."
      }
    }
  }'
```

### Update Voice + TTS Model

```bash
curl -X PATCH "https://api.elevenlabs.io/v1/convai/agents/{agent_id}" \
  -H "Content-Type: application/json" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -d '{
    "conversation_config": {
      "tts": {
        "voice_id": "zKHQdbB8oaQ7roNTiDTK",
        "model_id": "eleven_multilingual_v2",
        "stability": 0.55,
        "similarity_boost": 0.8
      }
    }
  }'
```

### Update Turn Detection (per-agent responsiveness)

```bash
curl -X PATCH "https://api.elevenlabs.io/v1/convai/agents/{agent_id}" \
  -H "Content-Type: application/json" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -d '{
    "conversation_config": {
      "turn": {
        "turn_eagerness": "eager",
        "spelling_patience": "high",
        "turn_timeout": 5.0
      }
    }
  }'
```

### Add ASR Keywords (improve speech recognition)

```bash
curl -X PATCH "https://api.elevenlabs.io/v1/convai/agents/{agent_id}" \
  -H "Content-Type: application/json" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -d '{
    "conversation_config": {
      "asr": {
        "quality": "high",
        "keywords": ["AcmeSoft", "Dashboard", "Lizenznummer", "Kundennummer"]
      }
    }
  }'
```

### Multi-Field Update (common after conversation analysis)

```bash
curl -X PATCH "https://api.elevenlabs.io/v1/convai/agents/{agent_id}" \
  -H "Content-Type: application/json" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -d '{
    "name": "AcmeSoft Empfang v2",
    "tags": ["production", "pharmacy", "german"],
    "conversation_config": {
      "agent": {
        "prompt": "Updated prompt...",
        "temperature": 0.3
      },
      "turn": {
        "turn_eagerness": "eager"
      },
      "asr": {
        "keywords": ["Ibuprofen", "Rezept", "Bestellung", "PZN"]
      }
    }
  }'
```

---

## 3. Turn Detection Parameters — When to Use What

| Parameter | Values | Effect |
|---|---|---|
| `turn_eagerness` | `eager` / `normal` / `relaxed` | How quickly the agent starts speaking after the caller stops |
| `spelling_patience` | `auto` / `low` / `medium` / `high` | How long the agent waits when caller spells words |
| `turn_timeout` | seconds (float) | Time to wait for caller to start speaking |
| `silence_end_call_timeout` | seconds (-1 = off) | End call after N seconds of silence |
| `initial_wait_time` | seconds (float) | Silence before conversation starts |

### Recommendations per Subagent Type

| Subagent | Eagerness | Spelling Patience | Turn Timeout | Silence End |
|---|---|---|---|---|
| Routing / Empfang | normal | auto | 7s | 30s |
| Sales / Vertrieb | eager | auto | 5s | 30s |
| Verification (Name, ID) | relaxed | high | 10s | 30s |
| Damage Report / Schadensmeldung | relaxed | medium | 10s | 30s |
| Support / Technical | normal | medium | 7s | 45s |
| Lead Capture | normal | high | 7s | 30s |
| Complaint | relaxed | auto | 10s | 60s |

**Key insight:** Subagents within the same workflow need different turn settings. The routing agent should respond quickly (eager). The verification agent must wait patiently while the caller looks up their ID number (relaxed + high spelling patience). Set these per-node in the workflow JSON, not globally.

### Soft Timeout (Silence Filler)

Plays a soft prompt when the caller is silent, BEFORE triggering the full silence end-call.

```json
{
  "conversation_config": {
    "turn": {
      "soft_timeout_config": {
        "timeout_seconds": 3.0,
        "message": "Sind Sie noch da?"
      }
    }
  }
}
```

| Parameter | Range | Default | Notes |
|---|---|---|---|
| `timeout_seconds` | 0.5–8.0 | -1 (disabled) | Recommended: 3.0s |
| `message` | 1–200 chars | "Hhmmmm...yeah." | Use target-language prompt |

- Triggers once per turn maximum
- Can use LLM-generated messages (optional, context window up to 4 messages / 1000 chars)
- Language overrides supported for multilingual workflows

### Turn Model Versions

| Version | Behavior |
|---|---|
| `turn_v2` | Standard turn detection |
| `turn_v3` | Improved detection — better at distinguishing thinking pauses from end-of-turn |

Set via `turn_model` parameter. Use `turn_v3` for production German agents where callers often pause mid-sentence.

---

## 4. TTS Model Selection

| Model | Latency | Quality | Languages | Use When |
|---|---|---|---|---|
| `eleven_turbo_v2` | Fastest | Good | English only | English-only agents, speed-critical |
| `eleven_flash_v2_5` | Very fast | Good | 32 | Testing, low-budget deployments |
| `eleven_turbo_v2_5` | Fast | Very good | 32 | Balance of speed and quality |
| `eleven_multilingual_v2` | Moderate | Excellent | 29 | **Default for German production** |
| `eleven_v3` | Variable | Best | 80+ | Premium quality, experimental |

**Recommended default:** `eleven_multilingual_v2` — best German pronunciation quality. Accept the latency tradeoff.

---

## 5. Voice Parameter Tuning

### Stability (0.0–1.0)

| Range | Effect | Use For |
|---|---|---|
| 0.2–0.3 | Very expressive, emotional variation | Entertainment, storytelling |
| 0.4–0.5 | Balanced expressiveness | Sales, conversational agents |
| 0.5–0.6 | Consistent, professional | **Customer service, healthcare** |
| 0.7–1.0 | Very stable, minimal variation | Announcements, IVR |

### Similarity Boost (0.0–1.0)

| Range | Effect | Use For |
|---|---|---|
| 0.3–0.5 | Loose match, more variation | Low importance of exact voice |
| 0.6–0.8 | Good balance | **Most production agents** |
| 0.8–1.0 | Very close to original | Cloned voices, brand consistency |

### Speed (0.7–1.2)

| Range | Effect | Use For |
|---|---|---|
| 0.8–0.9 | Slower, clearer | Elderly callers, complex info |
| 1.0 | Normal | **Default** |
| 1.1–1.2 | Faster | Simple routing, young audience |

### Production Defaults

For a German production agent:
- Stability: **0.55**
- Similarity Boost: **0.80**
- Speed: **1.0**
- Optimize Streaming Latency: **3**
- Expressive Mode: **true**

---

## 6. Agent Audit Checklist

Run this against every agent before deployment. Uses `get_agent` output and checks against the 10-Section schema.

### Prompt Audit

- [ ] Prompt is in **English** (not German)
- [ ] All 10 sections present (IDENTITY through ERROR RECOVERY)
- [ ] IDENTITY uses a human name, not "AI Assistant"
- [ ] LANGUAGE RULES block present with formal/informal specification
- [ ] TASK has one clear objective with explicit exclusions and exit condition
- [ ] CONVERSATION FLOW has numbered steps with turn limit
- [ ] KNOWLEDGE block lists allowed AND excluded topics
- [ ] TOPIC GUARDRAILS include deflection phrases in target language
- [ ] BEHAVIORAL GUARDRAILS cover: hallucination, identity, manipulation, abuse, stuck
- [ ] DATA & PRIVACY GUARDRAILS present with industry-specific rules
- [ ] OUTPUT RULES enforce max 2 sentences, 1 question per turn, no fillers
- [ ] ERROR RECOVERY covers: misunderstand (3 levels), silence, off-topic, system error
- [ ] First message is in target language, max 3 sentences, ends with question
- [ ] No filler phrases in first message ("Gerne!", "Herzlich willkommen!")

### Configuration Audit

- [ ] `language` = "de" (not "en")
- [ ] `llm` = "claude-3.5-sonnet" (not "gemini-2.0-flash-001")
- [ ] `model_id` = "eleven_multilingual_v2" (not "eleven_turbo_v2")
- [ ] `temperature` matches agent type (see Section 2 of elevenlabs-config.md)
- [ ] `max_tokens` ≤ 250 for voice agents
- [ ] `max_duration_seconds` ≥ 600 (10 min)
- [ ] `asr_quality` = "high"
- [ ] `record_voice` = true
- [ ] ASR keywords include domain-specific terms
- [ ] Turn detection settings match subagent type (see Section 3 above)
- [ ] Voice ID matches documented workflow voice

### Workflow Audit

- [ ] Every subagent has exactly one task
- [ ] Every path terminates (no dead ends)
- [ ] Human escalation exists in every path
- [ ] Loop detection implemented (max 2 returns to routing)
- [ ] Transition conditions have 3+ positive, 2+ negative examples
- [ ] Context payload defined for every transition
- [ ] Transfer announcements defined in target language
- [ ] Fallback transitions exist for every subagent
- [ ] `prevent_subagent_loops` = true in workflow JSON

---

## 7. Outbound Call Safety Rules

When using `make_outbound_call`:

1. **Always validate E.164 format** before calling (DE: +49XXXXXXXXXX)
2. **Always warn about costs** — outbound calls consume credits
3. **Always confirm before calling** — show agent name, from-number, to-number
4. **Check agent suitability** — inbound prompts don't work outbound (greeting mismatch)
5. **Suggest test call to own number first** for new agents
6. **After call:** retrieve conversation via `list_conversations` + `get_conversation` for review
