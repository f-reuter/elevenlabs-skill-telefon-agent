# Knowledge Base Zuordnung — Wianco OTT Robotics
# Version: 2026-04-07

## KB-Übersicht

| KB-Datei | Inhalt | Auth erforderlich |
|----------|--------|-------------------|
| `kb-wianco-public.md` | Produkte, FAQ, Einstieg, News, Service-Überblick | Nein |
| `kb-wianco-support-technik.md` | Technik, Troubleshooting, Lizenzen, Admin, Konfiguration | Ja |

## Zuordnung pro Agent

| Agent | KB Public | KB Support Technik | Hinweis |
|-------|-----------|-------------------|---------|
| **Sales-Hotline** | ✅ | ❌ | Nur öffentliche Infos, Lead-Qualifizierung |
| **Qualifizierung (Sophie)** | ✅ | ❌ | Branche + Automatisierungsbedarf erfragen |
| **Support-Hotline (vor Auth)** | ✅ | ❌ | Allgemeine Infos, dann Auth einleiten |
| **Support-Hotline (nach Auth)** | ✅ | ✅ | Voller Zugriff auf technische KB |
| **Outbound** | ✅ | ❌ | Nur öffentliche Infos für Follow-ups |

## Confluence Page ID Referenz

Alle IDs verweisen auf Wianco-Confluence (`{KUNDE}.atlassian.net/wiki/api/v2/pages/{ID}`).

### In KB Public verwendet:
| Confluence ID | Titel | Sektion |
|--------------|-------|---------|
| 174063629 | Erste Schritte | Produktübersicht |
| 175046664 | EMMA Produkte | Produktübersicht |
| 310247428 | EMMA Service | Produktübersicht |
| 287342593 | Release 2.7.1.300 | Produktübersicht, Neuheiten |
| 315654147 | EMMA – Neues Feature: Classifier | Produktübersicht |
| 309854254 | EMMA Configuration | Produktübersicht |
| 173375489 | Dein erster Prozess | Onboarding |
| 267780104 | Dein zweiter Prozess - ECB Exchange Rates | Onboarding |
| 302907397 | Dein dritter Prozess - Die RPA Challenge | Onboarding |
| 172392460 | Portal - Wissensdatenbank | Onboarding |
| 304775214 | Ich habe EMMA neu - was mache ich wo? | Onboarding |
| 175276033 | Service - Support | Service |
| 175308801 | News - Produktneuheiten | Neuheiten |
| 21233668 | Sending Emails Using Emma | FAQ |
| 21201218 | Using OAuth 2.0 for Email Authentication | FAQ |
| 308707426 | Wie extrahiere ich Daten aus einer PDF-Rechnung? | FAQ |
| 310116376 | Wie man einen Plan erstellt | FAQ |

### In KB Support Technik verwendet:
| Confluence ID | Titel | Sektion |
|--------------|-------|---------|
| 309067800 | Finden Schritt Überblick | Studio Features |
| 309755906 | Bild oder Objekt | Studio Features |
| 309952527 | Finden-Schritt als Textquelle | Studio Features |
| 310050829 | Sonderzeichen schreiben | Studio Features |
| 309002248 | Tastaturlayout prüfen | Studio Features |
| 310083610 | Tippen: Tipps und Tricks | Studio Features |
| 310673409 | Dateien nummerieren | Studio Features |
| 310738947 | File Operation Schritt | Studio Features |
| 309035013 | Heutiges Datum ermitteln | Studio Features |
| 309002277 | Entscheidung bei RPA-Challenge | Studio Features |
| 301629442 | Richtige Zeile in Tabelle finden | Studio Features |
| 305627171 | ABBYY OCR in EMMA | Studio Features |
| 310116376 | Plan erstellen | Pläne |
| 308903964 | Voraussetzungen automatische Ausführung | Pläne |
| 310640652 | Pläne im Rollen- und Rechtekonzept | Pläne |
| 283049994 | Installation EMMA Windows Service | Infrastruktur |
| 305299171 | Auflösung in Windows-VM | Infrastruktur |
| 233701378 | Auflösung herausfinden | Infrastruktur |
| 246448131 | Programm im Vollbildmodus | Infrastruktur |
| 329351172 | SQL Server Deployment | Infrastruktur |
| 309166113 | Automatisierte Datenbank-Backups | Infrastruktur |
| 310083645 | Manuelle Datenbank-Sicherung | Infrastruktur |
| 230948867 | SQL-DB Datenvolumen | Infrastruktur |
| 308674573 | Löschkonzept | Infrastruktur |
| 177963014 | EMMA Startet nicht | Troubleshooting |
| 228818948 | EMMA liest nicht mehr | Troubleshooting |
| 310444046 | Anzeigefehler in EMMA Studio | Troubleshooting |
| 308772866 | Wann läuft die EMMA-Lizenz ab? | Lizenz |
| 309854219 | Lizenz Restlaufzeit einsehen | Lizenz |
| 309985303 | Update EMMA | Lizenz |
| 231145476 | Wibu Error | Lizenz |
| 231735297 | Lizenz nach Umzug verloren | Lizenz |
| 310018089 | OAuth 2.0 E-Mail-Authentifizierung | Admin |
| 21233668 | Sending Emails Using Emma | Admin |
| 21201218 | OAuth 2.0 Email Authentication | Admin |
| 230850562 | Portal Access | Admin |

## Nächste Schritte: Confluence-Integration

1. API-Token von Wianco erhalten
2. Pro Confluence ID den Body abrufen: `GET /wiki/api/v2/pages/{ID}?body-format=storage`
3. HTML-Body in Atomic-Fact-Format umwandeln (max 200 Wörter pro Chunk)
4. Bestehende Beschreibungen in den KB-Dateien durch echte Inhalte ersetzen
5. KBs via `add_knowledge_base_to_agent` den Agenten zuweisen
6. Wianco validiert inhaltliche Richtigkeit
