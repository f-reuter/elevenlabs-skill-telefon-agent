================================================================
WORKFLOW: Wianco OTT Robotics — Telefonagenten
COMPANY: Wianco OTT Robotics GmbH
PURPOSE: KI-Telefonagenten für Sales, Support und Outbound
LANGUAGE: Deutsch (Hochdeutsch, Sie-Form) + Englisch (auto-detect)
VERSION: 2026-04-07
================================================================

ARCHITECTURE: Hub-and-Spoke (Hybrid)

```
                                ┌─── Sales-Hotline (UC-01)
                                │    Produkt-FAQ, Lead-Qualifizierung
                                │
Inbound ──→ Sophie ─────────────┼─── Support-Hotline (UC-02)
             (Qualifizierung)   │    Auth → Technik, Lizenzen, EMMA Studio
             Hub-Agent          │
                                └─── [Telefonanlage] Mitarbeiter-Transfer
                                     Kritische Fälle, Eskalation

Outbound ──→ Outbound-Agent (UC-03)
              Zufriedenheit, Follow-ups
              (separat via Webhook, nicht über Sophie)
```

ENTRY POINT:
- Agent: Sophie (Qualifizierung)
- Alle Inbound-Anrufe (Sales + Support) landen bei Sophie
- Sophie qualifiziert und routet an den richtigen Subagenten

SUBAGENTS:
| # | Name | Agent ID | Role | Temp | KB | Voice |
|---|------|----------|------|------|----|-------|
| 1 | Sophie | agent_1501kmzy7rmf | Qualifizierung & Routing (Hub) | 0.3 | Public | NE7AIW5D |
| 2 | Sales-Hotline | agent_6501kmzy8k8 | Produktinfos, Lead-Qualifizierung | 0.4 | Public | cjVigY5q |
| 3 | Support-Hotline | agent_7401kmzy9gyk | Auth + Technik-Support | 0.2 | Public + Support-Technik | FGY2WhTY |
| 4 | Outbound | agent_9601kmzya5j8 | Umfragen, Follow-ups | 0.4 | Public | cjVigY5q |

TRANSITIONS:
| From | To | Condition | Context Passed |
|------|----|-----------|----------------|
| Sophie | Sales-Hotline | Interesse an EMMA, Produktinfos, Demo, Preisanfrage | caller_name, caller_company, caller_intent |
| Sophie | Support-Hotline | Technisches Problem, Lizenzfrage, bestehender Kunde | caller_name, caller_company, caller_intent |
| Sophie | Telefonanlage | Expliziter Mitarbeiterwunsch, unklares Anliegen nach 3 Versuchen | caller_name, caller_intent |
| Sales-Hotline | Sophie | Themenwechsel zu Support | caller_name, new_intent |
| Support-Hotline | Telefonanlage | Auth fehlgeschlagen 3x, Lizenzumzug, komplexe Konfiguration | caller_name, auth_status, issue_summary |

FALLBACK MATRIX:
| Agent | Fallback Trigger | Fallback Destination |
|-------|-----------------|---------------------|
| Sophie | Intent unklar nach 3 Nachfragen | Telefonanlage (Mitarbeiter) |
| Sophie | Loop-Detection (2x zurück) | Telefonanlage (Mitarbeiter) |
| Sales-Hotline | Turn-Limit erreicht (15) | Lead-Daten erfassen → Rückruf |
| Sales-Hotline | Preisverhandlung, Vertragsfragen | Telefonanlage (Vertrieb) |
| Support-Hotline | Auth fehlgeschlagen 3x | Sperre + Verweis alternativer Kontaktweg |
| Support-Hotline | Turn-Limit erreicht (15) | Ticket erstellen → Rückruf |
| Support-Hotline | Nicht lösbar | Telefonanlage (Ops-Mitarbeiter, wöchentlich wechselnd) |
| Outbound | Anrufer will nicht sprechen | Gespräch beenden |
| Outbound | Komplexes Support-Thema | Verweis auf Support-Hotline |

EXIT POINTS:
| Exit Type | From Agent | Condition |
|-----------|-----------|-----------|
| Phone Transfer | Sophie, Support, Sales | Kritischer Fall, Mitarbeiterwunsch |
| Call End | Sales | Lead-Daten erfasst, Rückruf versprochen |
| Call End | Support | Problem gelöst oder Ticket erstellt |
| Call End | Outbound | Umfrage abgeschlossen oder Follow-up erledigt |
| Call End (Sperre) | Support | Auth 3x fehlgeschlagen |

ESCALATION CONTACTS:
| Department | Verfügbarkeit | Hinweis |
|-----------|--------------|---------|
| Ops-Mitarbeiter (Support) | Wöchentlich wechselnd | Über Telefonanlage |
| Vertrieb | [GESCHÄFTSZEITEN] | Über Telefonanlage |
| Datenschutz | [KONTAKT] | Für DSGVO-Anfragen |

KNOWLEDGE BASES:
| KB Name | Assigned To | Content | Auth |
|---------|------------|---------|------|
| kb-wianco-public | Sophie, Sales, Support (pre-auth), Outbound | Produkte, FAQ, Onboarding, News | Nein |
| kb-wianco-support-technik | Support (post-auth) | Technik, Troubleshooting, Lizenzen, Admin | Ja |

VERSION HISTORY:
| Date | Change | Author |
|------|--------|--------|
| 2026-04-07 | Initial workflow design | NeoPhone |
