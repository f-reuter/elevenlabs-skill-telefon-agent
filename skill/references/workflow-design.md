# Workflow Design Reference — Multi-Agent Architectures

---

## 1. Workflow Topologies

### Hub-and-Spoke (Default)

The most common and recommended topology for voice agent workflows.

```
                    ┌─── Sales Agent
                    │
Caller → Routing ───┼─── Support Agent
         Agent      │
                    ├─── Complaint Agent
                    │
                    └─── Lead Capture
```

**When to use:** Most customer-facing phone lines. One entry point, multiple specialized handlers.

**Rules:**
- All calls enter through the routing/reception hub
- Each spoke handles exactly one type of request
- Spokes can return to the hub (e.g., if the caller changes topic) or terminate
- The hub never solves problems — it only routes

### Linear Chain

Sequential processing where each agent completes its task before handing off.

```
Caller → Verification → Support → Resolution Summary
```

**When to use:** When steps must happen in order (verify before supporting, collect before processing).

**Rules:**
- Each agent completes fully before transitioning
- No backward transitions (if verification fails, don't send back — escalate to human)
- Keep chains to max 3 agents — longer chains feel like IVR hell

### Hybrid (Hub + Chains)

Combines hub-and-spoke with linear chains within spokes.

```
                    ┌─── Verification → Support
                    │
Caller → Routing ───┼─── Sales → Lead Capture
                    │
                    └─── Complaint → [Phone Transfer]
```

**When to use:** When some paths need sequential steps (verify → support) but the entry point is a hub.

---

## 2. Transition Design

### Transition Conditions

Every transition needs a condition that is **semantically clear and mutually exclusive** from other conditions on the same agent.

**Condition format:**

```
TRANSITION: Route to [destination agent]
CONDITION: [English description of when to trigger]

POSITIVE EXAMPLES (caller says or implies):
- "[target language example 1]"
- "[target language example 2]"
- "[target language example 3]"

NEGATIVE EXAMPLES (do NOT trigger — route elsewhere):
- "[target language example 1]" → Route to [other agent] instead
- "[target language example 2]" → Route to [other agent] instead
```

**Rules:**
- Minimum 3 positive examples, 2 negative examples per condition
- Positive examples should cover different phrasings of the same intent
- Negative examples should cover the most likely confusion cases
- Write examples in the target language (the language callers speak)
- Write the condition description in English
- Order conditions from most specific to least specific — first match wins

### Disambiguation

When two conditions could both match, the routing agent needs a disambiguation step in its conversation flow:

```
Step 2: If the caller's intent is ambiguous between Sales and Support,
ask: "[target lang: 'Haben Sie bereits ein Produkt bei uns, oder
möchten Sie sich über unsere Angebote informieren?']"
```

### Context Handoff

Every transition must pass context to the receiving agent. Define the payload:

```
CONTEXT PAYLOAD:
- caller_name: [string or null]
- caller_company: [string or null]
- caller_intent: [one-sentence summary of why they're being transferred]
- key_info: [any specific details already gathered]
- previous_agent: [name of the transferring agent]
- transfer_reason: [why this specific agent was chosen]
```

The receiving agent's prompt (Section 5: KNOWLEDGE & CONTEXT) must include instructions for using this context to avoid re-asking the caller.

### Transfer Announcement

Before every transition, the current agent must announce it:

```
TRANSFER ANNOUNCEMENT TEMPLATE:
"[target lang: 'Ich verbinde Sie jetzt mit [destination description].
[Optional: brief reason]. Einen Moment bitte.']"

Examples:
- "Ich verbinde Sie mit unserem Vertriebsteam. Einen Moment bitte."
- "Ich leite Sie an den technischen Support weiter, der kann Ihnen
  bei Ihrem EMMA-Problem direkt helfen. Einen Augenblick."
- "Für Ihr Anliegen verbinde ich Sie mit einem Kollegen. Einen
  Moment bitte."
```

Never transfer silently. The caller must know what's happening.

---

## 3. Mandatory Workflow Elements

Every workflow MUST include these elements. Check each one before deployment.

### Entry Point
- One routing/reception agent
- Greets the caller, identifies intent, routes
- Never solves problems — only classifies and forwards

### Exit Points (at least one per spoke)
Every workflow path must terminate. Valid terminations:
1. **Phone Transfer** — Hand off to a human agent, department, or external number
2. **Structured End** — Summary of what was discussed + next steps + goodbye
3. **Callback Promise** — Data captured, promise to call back, goodbye

### Fallback Paths
Every subagent needs a "cannot handle this" escape:
- Support can't solve → Phone Transfer to specialist
- Sales can't answer pricing → Lead Capture → callback promise
- Verification fails → Phone Transfer or back to routing

### Human Escalation
At least one path in the entire workflow must lead to a real human. This is non-negotiable — there will always be cases the AI cannot handle.

### Loop Detection
If a caller is routed back to the routing agent more than twice, something is wrong. After 2 returns to routing, offer human transfer directly:

```
LOOP DETECTION:
If you receive a caller who has already been transferred back to you
[2+] times, do not attempt further routing. Instead:
"[target lang: 'Ich merke, dass wir Ihr Anliegen so nicht optimal
klären können. Ich verbinde Sie direkt mit einem Mitarbeiter, der
sich persönlich darum kümmert.']"
→ Transfer to human.
```

### Timeout Handling
Define globally for the workflow what happens when a caller is silent too long. See `references/output-rules.md` Section 6 for standard silence handling phrases.

---

## 4. Workflow Documentation Template

Maintain this documentation for every deployed workflow.

```
================================================================
WORKFLOW: [Name]
COMPANY: [Company name]
PURPOSE: [One sentence]
LANGUAGE: [Target language + formality level]
VERSION: [YYYY-MM-DD]
================================================================

ENTRY POINT:
- Agent: [Name and role]
- Phone number / channel: [how callers reach this]

SUBAGENTS:
| # | Name | Role | Temperature | Max Tokens | KB | Voice |
|---|------|------|-------------|------------|-----|-------|
| 1 | [Name] | [Role] | [0.X] | [N] | [KB name] | [Voice ID] |
| 2 | [Name] | [Role] | [0.X] | [N] | [KB name] | [Voice ID] |
| ... |

TRANSITIONS:
| From | To | Condition | Context Passed |
|------|----|-----------|----------------|
| [Agent A] | [Agent B] | [When X] | caller_name, intent, key_info |
| [Agent A] | [Agent C] | [When Y] | caller_name, intent |
| [Agent B] | [Phone Transfer] | [When Z] | full conversation summary |
| ... |

FALLBACK MATRIX:
| Agent | Fallback Trigger | Fallback Destination |
|-------|-----------------|---------------------|
| [Agent A] | Intent unclear after 3 attempts | Human transfer |
| [Agent B] | Cannot resolve, turn limit reached | Human transfer |
| [Agent C] | Verification failed | [Agent D] or human |
| ... |

EXIT POINTS:
| Exit Type | From Agent | Condition |
|-----------|-----------|-----------|
| Phone Transfer | [Agent X] | [When completed/escalated] |
| Call End | [Agent Y] | [When resolved] |
| Callback Promise | [Agent Z] | [When data captured] |

ESCALATION CONTACTS:
| Department | Phone / Queue | Availability |
|-----------|--------------|-------------|
| Technical Support | [number] | Mon-Fri 8-18 |
| Sales | [number] | Mon-Fri 9-17 |
| Emergency | [number] | 24/7 |

KNOWLEDGE BASES:
| KB Name | Assigned To | Content | Update Frequency |
|---------|------------|---------|-----------------|
| [Name] | [Agents] | [Description] | [Weekly/Monthly] |

VERSION HISTORY:
| Date | Change | Author |
|------|--------|--------|
| [Date] | [Description] | [Name] |
```

---

## 5. Workflow Design Checklist

Before deploying, verify every item:

**Architecture:**
- [ ] Every subagent has exactly one task
- [ ] Topology is appropriate (hub-spoke / linear / hybrid)
- [ ] Every path terminates (no dead ends)
- [ ] Human escalation exists in every path
- [ ] Loop detection is implemented (max 2 returns to routing)
- [ ] Timeout/silence handling is defined for all agents

**Transitions:**
- [ ] Every condition has 3+ positive and 2+ negative examples
- [ ] Conditions are mutually exclusive (no ambiguous overlaps)
- [ ] Context payload is defined for every transition
- [ ] Transfer announcements are defined for every transition
- [ ] Fallback transitions exist for every agent

**Documentation:**
- [ ] Workflow documentation is complete using the template above
- [ ] Escalation contacts are documented with availability
- [ ] Knowledge base assignments are documented
- [ ] Version history is initialized
