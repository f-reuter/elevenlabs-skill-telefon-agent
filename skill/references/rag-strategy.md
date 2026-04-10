# RAG Strategy — Knowledge Bases for Voice Agents

Voice agents retrieve from knowledge bases differently than chatbots. Callers can't scroll back, responses must be short (max 2 sentences), and retrieval latency adds to response time. Every design decision must account for these constraints.

---

## 0. Platform Basics — File Types, Limits & Upload Methods

### Supported File Types

| Format | Notes |
|---|---|
| **PDF** | Most common. Ensure text is selectable (not scanned images) |
| **TXT** | Plain text. Best for manually structured atomic facts |
| **DOCX** | Word documents. Formatting stripped on import |
| **HTML** | Web pages. Tags stripped, content extracted |
| **EPUB** | E-books. Chapter structure preserved |

### Size Limits

- **Standard accounts:** Maximum **20 MB** or **300,000 characters** per knowledge base
- **Enterprise plans:** Expanded capacity available

### Three Upload Methods

| Method | When to Use | MCP Tool / API |
|---|---|---|
| **File upload** | PDF, DOCX, HTML, EPUB, TXT from local disk | `add_knowledge_base_to_agent(input_file_path=...)` |
| **URL import** | Documentation pages, product pages, blog posts | `add_knowledge_base_to_agent(url=...)` |
| **Text entry** | Manually written atomic facts, FAQs, structured content | `add_knowledge_base_to_agent(text=...)` |

### URL Import Limitations

- Does **NOT** auto-scrape linked pages (single page only)
- Does **NOT** continuously update when the source page changes
- Verify you have permission to use imported content
- For multi-page documentation: upload each page separately or export as single document

### API / SDK Methods

**Python:**
```python
from elevenlabs import ElevenLabs
client = ElevenLabs(api_key="sk_...")

# From text
doc = client.conversational_ai.knowledge_base.create_from_text(
    name="Preisliste 2026",
    text="## Was kostet AcmeSoft?\nEine Lizenz kostet ab 490 Euro netto pro Monat."
)

# From URL
doc = client.conversational_ai.knowledge_base.create_from_url(
    name="Produktseite",
    url="https://acmesoft.de/produkt"
)

# From file
doc = client.conversational_ai.knowledge_base.create_from_file(
    name="Technische Docs",
    file=open("docs.pdf", "rb")
)

# Attach to agent
client.conversational_ai.agents.update(
    agent_id="agent_...",
    knowledge_base=[{"type": "file", "name": "Preisliste 2026", "id": doc.id}]
)
```

**JavaScript/TypeScript:**
```javascript
import { ElevenLabs } from 'elevenlabs';
const client = new ElevenLabs({ apiKey: 'sk_...' });

const doc = await client.conversationalAi.knowledgeBase.createFromText({
  name: 'Preisliste 2026',
  text: '## Was kostet AcmeSoft?\nEine Lizenz kostet ab 490 Euro netto pro Monat.'
});
```

### KB Naming Convention

Name knowledge bases with dates for traceability:
- `Preisliste-2026-04` not `Preisliste`
- `Notdienst-Q2-2026` not `Notdienst`
- `FAQ-Produkt-v3` not `FAQ`

---

## 1. When KB vs. System Prompt

| Put in System Prompt | Put in Knowledge Base |
|---|---|
| Core behavior, personality, rules | Large reference datasets (10+ items) |
| Guardrails and boundaries | FAQs (10+ questions) |
| Conversation flow (steps) | Product catalogs / price lists |
| Short critical facts (address, hours) | Policy documents, manuals |
| Escalation procedures | Content that changes frequently |
| Output rules and error recovery | Customer-specific data (via dynamic KB) |
| 5 or fewer FAQ-style items | Anything > 500 words of factual content |

**Rule of thumb:** If the agent needs it in EVERY turn (rules, behavior, flow), put it in the prompt. If the agent needs it only WHEN ASKED (facts, details, specifics), put it in the KB.

**Token economics:** System prompt is loaded on every turn and consumes context. KB content is retrieved on demand. A 3000-token prompt + 5 KB retrievals is more efficient than a 6000-token prompt with everything embedded.

---

## 2. KB Architecture per Workflow

### Principle: One KB Per Domain, Assigned Per Subagent

Never give every subagent access to every KB. A sales agent with access to the technical support KB will answer support questions instead of routing to the support agent.

### Assignment Matrix Template

| KB Name | Content | Assigned To | NOT Assigned To |
|---|---|---|---|
| Öffnungszeiten & Kontakt | Hours, address, parking, directions | Empfang | Support, Sales |
| Produkt & Preise | Features, tiers, pricing, comparisons | Sales, Empfang (basic) | Support |
| Technische Docs | Setup, troubleshooting, known issues | Support | Sales, Empfang |
| SLA & Policies | Service levels, refund, escalation rules | Complaint, Support | Sales |
| FAQ | Common questions, general answers | Empfang, Sales | — |

---

## 3. Content Structuring for Voice RAG

### The Voice RAG Problem

Standard RAG retrieves text chunks and the LLM paraphrases them. For voice agents, this creates two issues:
1. **Chunk too long** → LLM generates a long response → caller zones out
2. **Chunk too vague** → LLM hallucinates to fill gaps → wrong information

### Solution: Atomic Fact Format

Structure KB content as self-contained atomic facts. Each entry should be answerable in 1–2 sentences.

**WRONG (paragraph format):**
```
AcmeSoft ist eine Projektmanagement-Software, die Teams bei der Planung
und Durchfuehrung komplexer Projekte unterstuetzt. Sie bietet Gantt-Charts,
Kanban-Boards, Zeiterfassung und Team-Kommunikation in einer Plattform,
laeuft als Cloud- oder On-Premise-Loesung, ist DSGVO-konform und kann nach
einer einstuendigen Einrichtung sofort genutzt werden. Lizenzen starten
ab vierhundertneunzig Euro netto pro Monat.
```

**RIGHT (atomic facts):**
```
## Was ist AcmeSoft?
AcmeSoft ist eine Projektmanagement-Software, mit der Teams komplexe
Projekte planen und durchfuehren koennen.

## Welche Funktionen bietet AcmeSoft?
AcmeSoft bietet Gantt-Charts, Kanban-Boards, Zeiterfassung und
Team-Kommunikation in einer Plattform.

## Wo laeuft AcmeSoft?
AcmeSoft gibt es als Cloud-Loesung oder als On-Premise-Installation.
Beide Varianten sind DSGVO-konform.

## Was kostet eine AcmeSoft-Lizenz?
Eine AcmeSoft-Lizenz kostet ab vierhundertneunzig Euro netto pro Monat.

## Wie schnell ist AcmeSoft einsatzbereit?
Nach einer einstuendigen Einrichtung kann Ihr Team sofort loslegen.
Keine IT-Kenntnisse noetig.
```

### Why This Works Better

- LLM retrieves ONE atomic fact → generates 1–2 sentence response
- No hallucination needed to fill gaps
- Caller gets a precise, short answer
- If caller wants more, they ask → next atomic fact is retrieved

---

## 4. FAQ Format for Voice

### Structure

```
## [Question in natural caller language]
[Answer in 1-2 sentences, max 50 words]
[Optional: Follow-up action the agent should offer]
```

### Example (Apotheke)

```
## Haben Sie heute geöffnet?
Ja, wir haben heute von acht bis achtzehn Uhr dreißig für Sie geöffnet.
→ Agent soll fragen: "Kann ich Ihnen noch bei etwas anderem helfen?"

## Kann ich ein Rezept telefonisch bestellen?
Ja, Sie können ein Folgerezept telefonisch vorbestellen. Ich brauche
dafür Ihren Namen und den Medikamentennamen.
→ Agent soll zur Rezeptannahme weiterleiten.

## Haben Sie Ibuprofen 400 vorrätig?
[NICHT BEANTWORTBAR — Verfügbarkeit ändert sich stündlich]
→ Agent soll sagen: "Das prüfe ich gerne für Sie. Einen Moment bitte."
→ Dann Tool-Call oder Rückruf anbieten.

## Wie sind Ihre Notdienst-Zeiten?
Unsere regulären Notdienst-Zeiten finden Sie auf aponet.de. Ich kann
Ihnen sagen, ob wir heute Notdienst haben.
→ Agent prüft Notdienst-KB.
```

### Explicit Non-Answers

Mark topics the agent should NOT answer from this KB:

```
## NICHT BEANTWORTEN: Medizinische Beratung
Medizinische Fragen wie Dosierung, Wechselwirkungen oder
Nebenwirkungen darf der Agent NICHT beantworten.
→ Deflection: "Dazu sprechen Sie am besten mit Ihrem Arzt oder
   unserem Apotheker persönlich."
```

---

## 5. Chunking Strategy

### Optimal Chunk Size for Voice RAG

| Chunk Type | Size | Why |
|---|---|---|
| Atomic facts | 30–80 words | One fact = one answer |
| FAQ entries | 50–100 words | Question + short answer + action |
| Process descriptions | 100–150 words | Step-by-step, max 3 steps per chunk |
| Policy excerpts | 80–120 words | One rule per chunk |

**Never exceed 200 words per chunk.** The LLM will try to paraphrase 200 words into a spoken response — that's always too long for voice.

### Chunking Rules

1. **One topic per chunk** — never combine opening hours and pricing
2. **Include the question in the chunk** — "Was kostet [Produkt]?" as heading helps retrieval
3. **Use the caller's vocabulary** — "Rezept vorbestellen" not "Medikationsvororder"
4. **Duplicate key terms** — If callers say both "Termin" and "Behandlung" for the same thing, include both
5. **No cross-references** — Every chunk must be self-contained. Don't write "Siehe auch Preisliste"

---

## 6. KB Updates & Maintenance

### Update Triggers

| Trigger | Action | Frequency |
|---|---|---|
| Price change | Update Produkt & Preise KB | As needed |
| New product/service | Add entries to relevant KB | As needed |
| Seasonal changes | Update Öffnungszeiten, Notdienst | Monthly |
| Conversation analysis shows gap | Add missing FAQ entries | After every analysis cycle |
| Staff/contact changes | Update Kanzlei/Praxis-Info | As needed |

### Update Process

1. **Export** existing KB content (or maintain as files)
2. **Edit** the source file
3. **Re-upload** via `add_knowledge_base_to_agent` (creates a new KB version)
4. **Delete** the old KB version to avoid duplicate/conflicting retrievals
5. **Test** with 3 questions that should retrieve updated content

---

## 7. Multi-KB Retrieval Behavior

When an agent has multiple KBs assigned, ElevenLabs searches across ALL of them and returns the most relevant chunks. This means:

- **Conflicting information across KBs** → LLM picks one unpredictably. Avoid contradictions.
- **Overlapping topics across KBs** → LLM may mix chunks from different KBs. Keep domains separate.
- **Too many KBs (5+)** → Retrieval quality degrades. Consolidate related content.

### Maximum Recommended KBs per Agent

| Agent Type | Max KBs | Why |
|---|---|---|
| Routing / Empfang | 2–3 | Needs general info only |
| Sales / Vertrieb | 2–3 | Products, pricing, maybe FAQ |
| Support | 2–3 | Technical docs, known issues, FAQ |
| Specialist (Scheduling, Verification) | 1 | One domain, one KB |

---

## 8. Connecting External KBs (HubSpot, Confluence)

### HubSpot as KB Source

Export relevant CRM data as structured text for KB upload:
- **Contact records** → NOT as KB (privacy risk). Use dynamic variables or tool calls instead.
- **Knowledge Base articles** → Export as markdown, structure as atomic facts, upload as text KB.
- **Product catalog** → Export, restructure per Section 3, upload.

### Confluence as KB Source

- **Export single pages** as HTML → upload via `input_file_path`
- **Export page trees** → merge into one structured document, upload as text
- **Keep Confluence as source of truth** → re-export and re-upload on changes
- **Don't upload entire spaces** — too much irrelevant content degrades retrieval

### Dynamic KB via API Tools

For data that changes per call (customer status, order tracking):
- Don't put it in KB (stale immediately)
- Configure as a **tool** on the agent: webhook to your API
- Agent calls the tool during conversation, gets real-time data
- Requires the ElevenLabs tool/webhook configuration (see `elevenlabs-config.md` Section 5)
