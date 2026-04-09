---
name: elevenlabs
description: "Use this skill whenever the user wants to create, design, prompt, or configure ElevenLabs Conversational AI voice agents. Triggers: 'ElevenLabs', 'voice agent', 'phone agent', 'Conversational AI', 'subagent workflow', 'IVR replacement', 'AI-Telefonagent', 'Sprachagent', or requests to build, prompt, or optimize voice-based AI agents. Covers: prompt engineering for voice agents, workflow architecture, multi-agent routing, transition conditions, guardrail creation, output rule optimization for TTS, first message design, knowledge base setup, testing strategy, and agent configuration. Use for ANY ElevenLabs-related task across the full lifecycle from architecture through deployment. Also use when writing system prompts for phone bots or optimizing German TTS output. Do NOT use for general chatbot design, text-only agents, or non-ElevenLabs platforms unless concepts transfer directly."
---

# ElevenLabs Conversational AI — Agent Design & Prompt Engineering

## Overview

This skill covers the complete lifecycle of designing, prompting, and deploying ElevenLabs Conversational AI voice agents — from architecture decisions through prompt engineering, guardrails, output optimization, and testing.

ElevenLabs Conversational AI uses LLMs (Claude, GPT) as the reasoning engine, with STT (speech-to-text) and TTS (text-to-speech) handling the voice layer. The prompt controls the LLM; the output rules control what the TTS receives.

## Quick Reference

| Task | Reference File |
|------|---------------|
| Write a subagent prompt (10-section schema) | `references/prompt-template.md` |
| Design the first message | `references/prompt-template.md` (First Message Design section) |
| See a complete worked example | `references/example-complete-prompt.md` |
| Design guardrails (topic, behavioral, privacy) | `references/guardrails.md` |
| Optimize output for TTS / voice naturalness | `references/output-rules.md` |
| Voice selection & prosody tuning | `references/output-rules.md` (Sections 8-9) |
| Design a multi-agent workflow | `references/workflow-design.md` |
| Configure ElevenLabs parameters & MCP tools | `references/elevenlabs-config.md` |
| Test and iterate | `references/testing.md` |
| Edit an agent via PATCH API / audit before deploy | `references/agent-api-operations.md` |
| Design and structure knowledge bases (RAG) | `references/rag-strategy.md` |
| Analyze conversations and improve agents | `references/conversation-analysis.md` |
| Connect tools (CRM, Calendar, Ticketing) & secure them | `references/tools-and-security.md` |

**Before writing any prompt or workflow, read the relevant reference files.** They contain the exact schemas, templates, anti-patterns, and examples you need.

---

## Core Principles

### 1. English Prompts, Target Language Output

Always write system prompts, guardrails, output rules, and transition conditions in **English**. The LLM reasoning layer processes English instructions with significantly higher accuracy — especially negations, conditionals, and complex routing logic.

Elements in the **target language** (e.g., German):
- `first_message` (spoken to the caller)
- Knowledge base / RAG documents
- Verbatim phrases the agent should say (deflections, confirmations, error messages)

Every prompt includes a `LANGUAGE RULES` block that enforces the target language for all spoken output. See `references/prompt-template.md` Section 2.

### 2. One Subagent = One Job

Never combine multiple responsibilities into a single agent. The LLM reasoning degrades when a prompt covers routing, sales, and support simultaneously.

**Decomposition test:**
- Can you describe this agent's job in one sentence? → If not, split.
- Does it need different knowledge than other agents? → Separate agent.
- Does it have a different conversation structure? → Separate agent.

### 3. Voice ≠ Chat

Voice agents operate under constraints chat agents don't have:
- **No scroll-back** — the caller can't re-read. Every turn must be self-contained.
- **Time pressure** — silence > 3 seconds feels broken. Responses must be short.
- **Linear flow** — the agent must guide the conversation step by step.
- **TTS artifacts** — abbreviations, URLs, markdown, and written-style German sound wrong when read aloud.

Every design decision must account for these constraints. The `references/output-rules.md` file contains the complete TTS optimization rules.

### 4. Guardrails Are Not Optional

Every subagent needs three layers of guardrails:
1. **Topic guardrails** — what the agent discusses (and what it blocks)
2. **Behavioral guardrails** — how the agent behaves (hallucination prevention, manipulation defense, identity protection)
3. **Data/privacy guardrails** — how the agent handles sensitive information (DSGVO, payment data, cross-customer data)

See `references/guardrails.md` for complete templates per layer.

### 5. Every Path Must Terminate

Every workflow path must end in one of:
- Phone transfer to a human
- Structured call end (summary + next steps)
- Callback promise with data captured

Dead-end agents (no defined exit) are the #1 workflow failure. Check every subagent for exit paths before deployment.

---

## How Templates Map to the 10-Section Prompt Schema

The reference files contain reusable template blocks. Here is exactly where each block goes when you assemble a complete subagent prompt:

| Prompt Section | Source | What to Customize |
|----------------|--------|-------------------|
| 1. IDENTITY | Write fresh per agent | Name, role, company, personality traits |
| 2. LANGUAGE RULES | `references/prompt-template.md` Sec. 2 | Target language, formal/informal, preserved terms |
| 3. TASK | Write fresh per agent | Objective, method, exclusions, exit condition |
| 4. CONVERSATION FLOW | Write fresh per agent | Steps, turn limit |
| 5. KNOWLEDGE & CONTEXT | Write fresh per agent | KB name, allowed/excluded topics, incoming context fields |
| 6. TOPIC GUARDRAILS | `references/guardrails.md` Layer 1 | Allowed topics (per agent), blocked topics (reusable) |
| 7. BEHAVIORAL GUARDRAILS | `references/guardrails.md` Layer 2 | Universal — paste as-is, only customize deflection language |
| 8. DATA & PRIVACY GUARDRAILS | `references/guardrails.md` Layer 3 | Universal — paste as-is, add industry-specific rules if needed |
| 9. OUTPUT RULES | `references/output-rules.md` | Universal — paste as-is for German TTS agents |
| 10. ERROR RECOVERY | `references/output-rules.md` Sec. 6 | Customize phrases per target language |

**Sections 1, 3, 4, 5 are always written fresh** — they define what makes this agent unique.
**Sections 6–10 start from templates** and get customized only where needed.

For a complete worked example showing all 10 sections assembled into one production-ready prompt, see `references/example-complete-prompt.md`.

---

## Workflow: Building a New Agent System

Follow this sequence when the user asks to build an ElevenLabs agent or workflow:

### Step 1: Understand the Use Case

Gather before writing anything:
- **Company & product context** — What does the company do? What product/service does the agent support?
- **Caller profile** — Who calls? (customers, prospects, partners) What do they typically want?
- **Current process** — How are calls handled today? (IVR, receptionist, voicemail)
- **Languages & formality** — Which language(s)? Formal/informal address?
- **Integration requirements** — CRM, ticketing, calendar, knowledge base?
- **Escalation paths** — Who are the human fallbacks? Phone numbers? Departments?

### Step 2: Design the Workflow Architecture

Read `references/workflow-design.md`, then:
1. Map caller intents to subagents (one intent = one subagent)
2. Choose topology: hub-and-spoke (most common), linear chain, or hybrid
3. Define all transitions with conditions, context payloads, and fallbacks
4. Verify: every path terminates, every agent has a fallback, no dead ends
5. Document the workflow using the template in `references/workflow-design.md`

### Step 3: Write Subagent Prompts

Read `references/prompt-template.md`. For each subagent:
1. Fill in all 10 sections of the prompt template (never skip sections)
2. Write the `first_message` in the target language
3. Define transition conditions with positive AND negative examples
4. Assign knowledge bases per subagent

### Step 4: Add Guardrails

Read `references/guardrails.md`. For each subagent:
1. Define topic boundaries (allowed + blocked with deflection phrases)
2. Add behavioral guardrails (hallucination, identity, manipulation, abuse)
3. Add data/privacy guardrails (sensitive data, DSGVO, cross-customer)

### Step 5: Optimize Output Rules

Read `references/output-rules.md`. Apply to every subagent:
1. Response length limits (max 2 sentences per turn)
2. TTS speech optimization (spell out abbreviations, no URLs, no markdown)
3. Filler phrase bans
4. Confirmation patterns
5. Error recovery phrases

### Step 6: Connect Tools & Integrations

Read `references/tools-and-security.md`. For each subagent:
1. Identify which tools this agent needs (CRM, calendar, ticketing, email)
2. Configure server tools with proper authentication (OAuth2, Bearer, etc.)
3. Store all API keys as workspace secrets — never hardcode
4. Set error handling mode (`summarized` for production, never `passthrough`)
5. Add tool usage rules to the system prompt (filler phrases, error recovery, no raw data)
6. Configure tool call sounds to mask latency
7. Run the tool security checklist before deployment

### Step 7: Configure ElevenLabs Parameters

Read `references/elevenlabs-config.md`. Set per subagent:
- Temperature (0.2–0.3 for routing/data collection, 0.4–0.6 for conversational)
- Max tokens (150–200 for voice)
- Voice ID (consistent across workflow unless intentionally varied)
- Turn detection (eagerness, spelling patience, soft timeout per agent type)
- LLM model (fast model for routing, accurate model for support)

### Step 8: Test

Read `references/testing.md`. Follow this order:
1. Unit test each subagent in isolation
2. Test each transition pair
3. End-to-end scenario tests
4. Adversarial / guardrail tests
5. Language / dialect / edge case tests

---

## Output Format

When creating agents for the user, always deliver ALL of the following. Do not skip items — incomplete delivery is the most common failure mode.

1. **Workflow diagram** — Visual overview of the agent topology (use Mermaid or describe textually). Must show all agents, transitions, and exit points.
2. **Workflow documentation** — Fill in the complete workflow documentation template from `references/workflow-design.md` Section 4. This includes the subagent table, transition matrix, fallback matrix, exit points, escalation contacts, and KB assignments. Do not just reference the template — actually fill it out.
3. **Complete prompts per subagent** — All 10 sections, English, ready to paste into ElevenLabs. Use the template from `references/prompt-template.md`. For a worked example, see `references/example-complete-prompt.md`.
4. **First messages** — In target language, per subagent. Max 3 sentences, ends with a question.
5. **Transition conditions** — With 3+ positive and 2+ negative examples per condition. Include context payload and announcement phrase.
6. **Knowledge base recommendations** — What content each agent needs access to, and what to explicitly exclude.
7. **Configuration parameters** — Temperature, max_tokens, voice recommendations per subagent.
8. **Test scenarios** — At minimum: happy path, 3 edge cases, and 3 guardrail tests per subagent.

---

## Anti-Patterns — Catch These Before Delivery

Before finalizing any agent or workflow, verify against these common failures:

### Prompt Anti-Patterns
| Problem | Fix |
|---------|-----|
| German system prompt | Rewrite in English, keep output in German |
| Multi-responsibility agent | Split into separate subagents |
| No negative examples in conditions | Add 2+ negative examples per condition |
| Improvised deflection phrases | Write exact target-language phrases |
| No turn limit | Set max turns (typically 8–15) |
| "You should" / "Try to" language | Use "Always", "Never", "You must" |
| Capabilities list instead of task | Define one task, not a menu |

### Workflow Anti-Patterns
| Problem | Fix |
|---------|-----|
| Dead-end subagent | Add exit path (transfer, end, callback) |
| Silent transfer | Agent must announce transfers |
| No human fallback | Add escalation path to every workflow |
| Circular routing without detection | Add loop counter, offer human after 2 returns |
| Missing context on handoff | Define context payload for every transition |

### Output Anti-Patterns
| Problem | Fix |
|---------|-----|
| Long monologues (>2 sentences) | Enforce max 2 sentences in output rules |
| Multiple questions per turn | One question, then stop |
| Written German ("z.B.", "€/Monat") | Spell out for TTS |
| Filler phrases ("Gerne!", "Gute Frage") | Ban explicitly in output rules |
| No data confirmation | Always repeat back names, numbers |
| Timeline promises ("bis morgen") | Never promise timelines unless in KB |
| Stacked apologies | One acknowledgment, then action |
| No recording consent | Add consent notice to first message or pre-greeting |

---

## Deployment Checklist — End to End

After building all agents and testing, follow this sequence to go live:

### Phase 1: Create Agents
- [ ] Create each subagent via `create_agent` MCP tool or ElevenLabs Dashboard
- [ ] Verify each agent's config with `get_agent` and the audit checklist in `references/agent-api-operations.md`
- [ ] Upload knowledge bases via `add_knowledge_base_to_agent`
- [ ] Assign correct KBs to correct agents (never give every agent every KB)

### Phase 2: Configure Workflow
- [ ] Connect subagents into workflow (transitions, conditions, context payloads)
- [ ] Set `prevent_subagent_loops = true` in workflow JSON
- [ ] Verify every path terminates (no dead ends)
- [ ] Verify human escalation exists in every path
- [ ] Configure ASR keywords per subagent (domain-specific terms)
- [ ] Set turn detection per subagent type (eager/normal/relaxed)

### Phase 3: Voice & Recording
- [ ] Select and test voice with actual first message and error recovery phrases
- [ ] Configure voice parameters (stability, similarity, speed) per subagent type
- [ ] Configure recording consent (pre-greeting or first message)
- [ ] Enable `record_voice = true` for quality review

### Phase 4: Test (in this order)
- [ ] Unit test each subagent in isolation
- [ ] Transition test each agent pair
- [ ] End-to-end scenario tests (every workflow path)
- [ ] Adversarial tests (manipulation, abuse, data exfiltration)
- [ ] Test with different speaking speeds, accents, background noise

### Phase 5: Go Live
- [ ] Assign phone number to workflow entry point
- [ ] Place test call to the live number from an external phone
- [ ] Verify call recording and webhook logging work
- [ ] Set up conversation analysis schedule (weekly for first 2 months)
- [ ] Document escalation contacts with availability hours
