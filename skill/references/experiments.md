# Experiments — A/B Testing for Voice Agents

Run controlled A/B tests on production traffic to optimize agent performance through data, not intuition. Experiments let you test changes to any aspect of agent configuration — prompts, workflows, voices, tools, knowledge bases — before rolling them out fully.

**Prerequisite:** Agent versioning must be enabled before experiments can be used.

---

## 1. How Experiments Work

### Four-Step Workflow

```
Create Variant → Route Traffic → Measure Impact → Promote Winner
```

| Step | What You Do | Key Detail |
|---|---|---|
| **1. Create Variant** | Branch from current agent config, modify what you want to test | Changes tracked as versioned configs in the **Branches** tab |
| **2. Route Traffic** | Set % of live conversations going to each variant | Must sum to exactly 100% — deployment fails otherwise |
| **3. Measure Impact** | Compare variant vs. baseline on analytics dashboard | Track CSAT, containment, conversion, latency, cost |
| **4. Promote Winner** | Increase traffic share or merge variant into main branch | Full version history preserved for rollback |

### Traffic Routing Mechanics

- Traffic splitting is **deterministic** based on conversation ID
- Ensures consistent user routing across sessions (same caller → same variant)
- By default, traffic randomizes across users
- Via API: you can direct specific cohorts to specific branches by controlling which branch config initiates each conversation

---

## 2. What You Can Test

Nearly any agent configuration aspect can vary between branches:

| Category | Examples |
|---|---|
| **System Prompt** | Tone, instructions, personality, guardrails, conversation flow |
| **Workflow** | Node structure, branching logic, escalation paths, transition conditions |
| **Voice** | Voice selection, TTS model, speed, stability |
| **Tools** | Tool configuration, server logic, MCP servers |
| **Knowledge Base** | Documents, RAG settings, which KBs are assigned |
| **LLM Parameters** | Model selection (GPT vs. Claude vs. Gemini), temperature, max tokens |
| **Evaluation Criteria** | Success metrics per branch (different criteria per variant) |
| **Language** | Language settings, multi-language configurations |

---

## 3. Metrics Tracked

The analytics dashboard compares variants on:

| Metric | What It Measures |
|---|---|
| **CSAT Scores** | Customer satisfaction (if feedback collection enabled) |
| **Containment Rate** | % of calls resolved without human transfer |
| **Conversion Metrics** | Lead capture, appointment booking, or custom conversion goals |
| **Average Handling Time** | Mean call duration per variant |
| **Median Agent Response Latency** | How fast the agent responds (LLM + TTS) |
| **Cost per Resolution** | Total cost (LLM + voice + tools) per completed call |

---

## 4. Best Practices

### Before Creating an Experiment

1. **Define a hypothesis** — "Changing X will improve Y by Z%" — before creating the variant
2. **Set up evaluation criteria** on the agent BEFORE running the experiment (see `conversation-analysis.md`)
3. **Change ONE variable at a time** — if you change prompt AND voice simultaneously, you can't isolate which caused the improvement

### During the Experiment

4. **Start conservative** — 5–10% traffic to the variant initially, scale up as confidence grows
5. **Allow sufficient volume** — don't draw conclusions from 10 conversations. Wait for 50+ per variant minimum
6. **Let it run long enough** — at least 1–2 weeks to account for day-of-week and time-of-day variations

### After the Experiment

7. **Merge or discard promptly** — don't let experiments linger. Branch drift makes merging harder over time
8. **Document findings** — save what you learned to the project's `notes.md`
9. **Version history** — all variants remain in version history. You can always roll back

---

## 5. Common Experiment Ideas for Voice Agents

### Prompt Experiments

| Test | Variant A (Control) | Variant B | Measure |
|---|---|---|---|
| Greeting style | Formal ("Guten Tag, Sie sprechen mit...") | Casual ("Hallo! Wie kann ich helfen?") | CSAT, completion rate |
| Instruction strength | "You should try to..." | "You MUST always..." | Guardrail compliance |
| Turn limit | Max 12 turns | Max 8 turns | Completion rate, handle time |
| Deflection phrasing | "Das kann ich leider nicht beantworten." | "Dazu verbinde ich Sie gerne mit einem Kollegen." | CSAT, transfer rate |

### Voice Experiments

| Test | Variant A | Variant B | Measure |
|---|---|---|---|
| Voice gender | Female voice | Male voice | CSAT, completion rate |
| Speaking speed | 1.0x | 1.1x | Handle time, completion rate |
| Voice stability | 0.5 (varied) | 0.8 (stable) | CSAT |

### Model Experiments

| Test | Variant A | Variant B | Measure |
|---|---|---|---|
| LLM model | GPT-4o | Claude Sonnet | Response quality, latency |
| Temperature | 0.3 (deterministic) | 0.5 (creative) | Consistency, naturalness |
| Fast vs. accurate | Gemini Flash (routing) | GPT-4o (routing) | Latency, routing accuracy |

### Workflow Experiments

| Test | Variant A | Variant B | Measure |
|---|---|---|---|
| Verification step | Verify before support | Skip verification | Handle time, security |
| Routing depth | Direct routing (2 agents) | Hub-and-spoke (4 agents) | Completion rate, handle time |

---

## 6. Integration with Agent Workflow

### When to Use Experiments

| Situation | Use Experiment? | Why |
|---|---|---|
| New agent deployment | No | No baseline to compare against. Deploy, collect data, then experiment |
| Prompt optimization | Yes | Compare phrasing, tone, flow changes |
| Voice change | Yes | Caller perception is subjective — measure it |
| Model upgrade | Yes | New model might be faster but less accurate |
| Workflow restructuring | Yes | Big changes need data validation |
| Bug fix | No | Just fix it. Don't A/B test broken vs. working |
| Guardrail addition | No | Safety isn't negotiable. Add it to all variants |

### Experiment + Conversation Analysis

Experiments work best when combined with evaluation criteria (see `conversation-analysis.md`):

1. **Set evaluation criteria** on the agent (success evaluation + data collection)
2. **Create experiment variant** with the change you want to test
3. **Route 10% traffic** to variant
4. **Compare evaluation scores** between control and variant on the analytics dashboard
5. **Scale up or discard** based on data

---

## 7. Anti-Patterns

| Problem | Why It Fails | Fix |
|---|---|---|
| Testing without evaluation criteria | No automated scoring → manual review of every call | Set up success evaluation BEFORE running experiment |
| Changing multiple variables | Can't isolate cause of improvement/decline | One change per variant |
| Too little traffic (1%) | Takes months to get meaningful data | Start at 5–10% |
| Running too short (<3 days) | Day-of-week effects skew results | Run at least 1–2 weeks |
| Not documenting results | Same experiment gets re-run later | Save findings to project notes |
| Letting old experiments linger | Branch drift, confusing config state | Merge or discard within 2 weeks |
| A/B testing safety features | Guardrails must always be on | Never put guardrails in only one variant |
