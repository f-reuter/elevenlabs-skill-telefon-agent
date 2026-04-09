# ElevenLabs Telefon-Agent Skill

Claude Code Skill zum automatisierten Erstellen professioneller ElevenLabs Conversational AI Telefon-Agenten.

## Was ist das?

Ein vollstaendiges Skill-Paket fuer Claude Code, mit dem sich **produktionsreife ElevenLabs Voice Agents** fuer jede Branche und jeden Use Case bauen lassen. Claude lernt damit automatisch, wie man Telefon-Agenten richtig promptet, konfiguriert und testet -- unabhaengig von Branche oder Unternehmen.

## Projektstruktur

```
elevenlabs-skill-telefon-agent/
|
|-- skill/                          # Claude Code Skill (nach ~/.claude/skills/ kopieren)
|   |-- SKILL.md                    # Haupt-Skill mit Workflow, Prinzipien, Anti-Patterns
|   |-- references/
|       |-- prompt-template.md      # 10-Sektionen Prompt-Schema (Copy-Paste Template)
|       |-- guardrails.md           # 3-Layer Guardrails (Topic, Behavioral, Data/Privacy)
|       |-- output-rules.md         # TTS-optimierte Output-Regeln (Deutsch)
|       |-- workflow-design.md      # Multi-Agent Workflow-Architekturen
|       |-- elevenlabs-config.md    # Technische Parameter (Temperature, Voice, Turn Detection)
|       |-- testing.md              # Testmethodik, Szenarien, Quality Metrics
|       |-- rag-strategy.md         # Knowledge Base Architektur & Chunking
|       |-- conversation-analysis.md # Gespraeche analysieren & Feedback-Loop
|       |-- agent-api-operations.md # PATCH API, Audit Checklist, Outbound Safety
|       |-- example-complete-prompt.md # Vollstaendig ausgefuelltes Beispiel (Sales Agent)
|
|-- mcp.json                        # ElevenLabs MCP Server Konfiguration
```

## Installation

### 1. Skill installieren

```bash
cp -r skill/ ~/.claude/skills/elevenlabs/
```

### 2. MCP Plugin einrichten

Die `mcp.json` in dein Projektverzeichnis kopieren:

```bash
cp mcp.json /dein/projektordner/.mcp.json
```

ElevenLabs API-Key setzen:

```bash
export ELEVENLABS_API_KEY="sk_dein_key_hier"
```

### 3. Testen

Claude Code starten und sagen:

> "Erstelle einen ElevenLabs Telefonagenten fuer [deine Branche/dein Unternehmen]"

Claude laedt automatisch den Skill und baut einen kompletten Agenten mit Workflow, Prompts, Guardrails und Testszenarien.

## Was der Skill abdeckt

| Bereich | Details |
|---------|---------|
| **Prompt Engineering** | 10-Sektionen Schema, englischer System-Prompt mit Zielsprach-Ausgabe |
| **Workflow Design** | Hub-and-Spoke, Linear Chain, Hybrid Topologien |
| **Guardrails** | Topic, Behavioral, Data/Privacy (DSGVO) -- 3 Layer |
| **TTS-Optimierung** | Abkuerzungen ausschreiben, keine URLs, Schachtelsaetze vermeiden |
| **Knowledge Bases** | Atomic Facts, Voice-RAG Chunking, FAQ-Format |
| **Testing** | Unit, Transition, End-to-End, Adversarial, Sprache/Dialekt |
| **Konfiguration** | Temperature, Max Tokens, Voice, Turn Detection pro Agent-Typ |
| **API Operations** | PATCH-Updates, Agent Audit Checklist, Outbound Safety |
| **Conversation Analysis** | Performance Metrics, Feedback-Loop, Continuous Improvement |

## Kern-Prinzipien

1. **Englischer Prompt, Zielsprach-Ausgabe** -- LLM-Reasoning funktioniert in Englisch zuverlaessiger
2. **Ein Subagent = Ein Job** -- Keine Multi-Responsibility Agents
3. **Voice ≠ Chat** -- Max 2 Saetze pro Turn, keine Scroll-back-Moeglichkeit
4. **Guardrails sind Pflicht** -- Topic, Behavioral, Data/Privacy fuer jeden Agent
5. **Jeder Pfad muss enden** -- Telefon-Transfer, strukturiertes Gespraechsende oder Rueckruf-Versprechen

## Output-Format bei Agent-Erstellung

Claude liefert bei jeder Agent-Erstellung:

1. Workflow-Diagramm (Mermaid oder textuell)
2. Workflow-Dokumentation (ausgefuelltes Template)
3. Komplette Prompts pro Subagent (alle 10 Sektionen)
4. First Messages in Zielsprache
5. Transition Conditions mit positiven und negativen Beispielen
6. Knowledge Base Empfehlungen
7. Konfigurationsparameter pro Subagent
8. Test-Szenarien (Happy Path + Edge Cases + Guardrail Tests)

## Lizenz

MIT
