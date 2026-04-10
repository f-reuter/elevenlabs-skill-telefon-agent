# Expressive Mode — Eleven v3 Conversational & Emotional Delivery

Expressive mode enables agents to deliver speech that reflects intent, emotion, and emphasis in real time. Built on Eleven v3 Conversational — the most emotionally intelligent TTS model available in ElevenAgents.

---

## 1. What It Does

| Feature | Description |
|---|---|
| **Context-aware delivery** | Agent adapts tone based on conversation context — calm for worried callers, direct when clarity matters |
| **Expressive tags** | LLM outputs tags like `[laughs]`, `[whispers]`, `[sighs]` to control specific delivery moments |
| **Improved turn-taking** | New system using Scribe v2 Realtime — analyzes emotional cues + speech patterns for better timing |
| **70+ languages** | Expanded from ~32 in Flash models, improved expressiveness in languages like Japanese |
| **Same pricing** | $0.08/min — same as other TTS models |

---

## 2. Configuration

### Enabling Expressive Mode

1. Agent configuration → **Voice** tab
2. Select **V3 Conversational** as TTS model
3. Expressive mode is enabled by default with this model

### Critical Limitation

**Eleven v3 Conversational does NOT preserve Professional Voice Clone (PVC) characteristics.** The output may not sound like the original PVC voice.

If maintaining PVC voice identity is critical → use **Flash v2** instead.

---

## 3. Expressive Tags

The LLM can output explicit tags to control specific moments of delivery.

| Tag | Effect | Duration |
|---|---|---|
| `[laughs]` | Adds laughter | ~4-5 words |
| `[whispers]` | Lowers volume for whispering | ~4-5 words |
| `[sighs]` | Adds a sighing quality | ~4-5 words |
| `[slow]` | Slows down speech delivery | ~4-5 words |
| `[excited]` | Adds excitement | ~4-5 words |

**Each tag affects approximately 4-5 words** before returning to normal delivery.

### Using Tags in System Prompts

```
EXPRESSIVE DELIVERY RULES:
- Use [laughs] for moments of humor or when acknowledging something funny
  the caller said.
- Use [slow] when reading back important information like dates, numbers,
  or names.
- Use [whispers] sparingly — only for creating intimacy or confidentiality.
- Never use more than one expressive tag per turn.

Example output:
"Das freut mich zu hoeren! [laughs] Dann ist alles geklaert."
"Ihr Termin ist am [slow] fuenfzehnten Mai um vierzehn Uhr."
```

---

## 4. System Prompt Tone Guidance

Guide emotional delivery through natural language instructions — the model interprets these contextually.

### Broad Guidance (Recommended Start)

```
TONE AND DELIVERY:
When a caller sounds frustrated or upset, respond in a calm, reassuring tone.
When delivering good news, allow your tone to reflect genuine warmth.
When handling complaints, remain composed and solution-oriented.
Maintain a professional but approachable delivery throughout.
```

### Specific Triggers (More Control)

```
TONE GUIDELINES:
- When a caller expresses frustration → calm and empathetic tone
- When explaining technical steps → clear and measured pace
- When a caller shares good news → warmth and enthusiasm
- When handling complaints → composed and solution-oriented
- When confirming important data → deliberate and precise
- When ending a successful call → warm and encouraging
```

### Per Agent Type

| Agent Type | Tone Guidance |
|---|---|
| **Routing / Reception** | Professional, efficient, warm. Quick and clear without rushing. |
| **Sales** | Enthusiastic but not pushy. Confident about products, empathetic to needs. |
| **Support** | Patient, methodical, reassuring. Calm even when caller is frustrated. |
| **Complaint** | Empathetic, composed, solution-focused. Never defensive. |
| **Appointment Booking** | Friendly, organized, confirmatory. Deliberate when reading dates/times. |

---

## 5. Turn-Taking System

The new turn-taking system (paired with v3 Conversational) uses Scribe v2 Realtime to analyze:
- **Emotional cues** — how the caller sounds (frustrated, happy, confused)
- **Speech patterns** — whether a pause is a thinking pause or a finished thought

**Example:** "yeah" can be a complete acknowledgement or a lead-in. The system analyzes prosody (how it was said) to decide whether to wait or respond.

### Pairing with Turn Eagerness

| Setting | When to Use | Expressive Mode Benefit |
|---|---|---|
| **Eager** | Fast-paced routing, simple FAQ | Quick responses, energetic |
| **Normal** | Standard conversations | Balanced timing |
| **Patient** | Emotionally sensitive, complaints | Gives callers space to express feelings |

```
For complaint agents:
"Use patient turn eagerness. Give callers extra time to express
their frustration before responding. Do not interrupt emotional
statements."
```

---

## 6. Model Comparison for Voice Agents

| Model | Expressiveness | Latency | Languages | PVC Support | Best For |
|---|---|---|---|---|---|
| **V3 Conversational** | Highest | Ultra-low | 70+ | No | Most agents — default choice |
| **Flash v2** | Good | Very low | ~32 | Yes | When PVC voice identity is critical |
| **Flash v2.5** | Good | Lowest | ~32 | Yes | Latency-critical routing agents |
| **Multilingual v2** | Moderate | Low | ~32 | Yes | Legacy multilingual setups |

**Recommendation:** Use V3 Conversational as default for all new agents unless PVC is required.

---

## 7. Best Practices

1. **Start with broad tone guidance** in the system prompt, then refine based on conversation analysis
2. **Test across target languages** — expressiveness varies by language
3. **Pair with patient turn eagerness** for complaint and emotionally sensitive agents
4. **Use expressive tags sparingly** — one per turn maximum, for key moments only
5. **Monitor with analytics** — track how callers respond to expressive delivery
6. **Don't over-specify** — the model handles many emotional nuances naturally. Too many rules can make it sound scripted
7. **PVC check** — if using Professional Voice Clones, test V3 Conversational. If voice identity degrades, fall back to Flash v2
