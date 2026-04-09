# Output Rules Reference — TTS-Optimized Voice Output

Output rules control the FORM of every response. They directly affect what the TTS engine receives and reads aloud. Poorly formatted output is the #1 reason voice agents sound robotic.

All output rules are written in **English** in the system prompt. Example phrases are in the **target language**.

---

## 1. Response Length

```
RESPONSE LENGTH:
- Maximum 2 sentences per turn. One sentence is often enough.
- After asking a question, STOP IMMEDIATELY. Do not add commentary,
  context, or filler after the question mark.
- If you need to convey complex information, break it into multiple
  turns with confirmation checkpoints between them.
```

### Why This Matters

Callers stop listening after ~10 seconds of continuous speech. At normal speaking rate, 2 sentences ≈ 8–12 seconds. Anything longer and the caller either interrupts, zones out, or hangs up.

### Complex Information Pattern

When you need to convey multi-part information, use the **chunk-and-confirm** pattern:

**WRONG (monologue):**
```
"Die Lizenz kostet vierhundertneunzig Euro im Monat, enthält drei
Arbeitsplätze, ein Jahr Updates und unseren Standard-Support, der montags
bis freitags von acht bis achtzehn Uhr erreichbar ist."
```

**RIGHT (chunk-and-confirm):**
```
Turn 1: "Die Lizenz kostet vierhundertneunzig Euro im Monat.
         Soll ich Ihnen erklären, was da alles enthalten ist?"
Turn 2: [Caller says yes]
Turn 3: "Enthalten sind drei Arbeitsplätze und ein Jahr lang alle Updates.
         Möchten Sie auch etwas zum Support wissen?"
Turn 4: [Caller says yes]
Turn 5: "Unser Standard-Support ist montags bis freitags von acht bis
         achtzehn Uhr erreichbar. Haben Sie noch Fragen dazu?"
```

---

## 2. Question Rules

```
QUESTION RULES:
- Ask ONE question per turn. Never stack questions.
- After asking, STOP. Do not add "or" alternatives, explanations,
  or context after the question.
- Use closed or guided questions when possible. Open questions
  invite monologues that are harder to process.
```

### Question Types by Purpose

**Binary (yes/no)** — for confirmations:
```
"Soll ich Sie mit dem Vertrieb verbinden?"
```

**Guided choice** — for routing:
```
"Geht es um ein bestehendes Produkt oder eine neue Anfrage?"
```

**Specific data** — for collection:
```
"Wie ist Ihr Nachname?"
```

**AVOID open-ended** unless necessary:
```
× "Erzählen Sie mir von Ihrem Problem."
✓ "Was genau funktioniert nicht — die Anmeldung oder eine bestimmte Funktion?"
```

---

## 3. TTS Speech Optimization

These rules transform written German into spoken German that the TTS engine reads naturally.

### 3.1 Abbreviations → Full Words

```
ABBREVIATION RULES:
Always spell out abbreviations. The TTS will attempt to read them
as written, producing awkward output.

z.B.  → zum Beispiel
d.h.  → das heißt
ca.   → etwa / ungefähr
bzgl. → bezüglich
ggf.  → falls nötig / gegebenenfalls
inkl. → inklusive
usw.  → und so weiter
bzw.  → beziehungsweise
o.Ä.  → oder Ähnliches
u.a.  → unter anderem
i.d.R. → in der Regel
Nr.   → Nummer
Tel.  → Telefon
Str.  → Straße
```

### 3.2 Numbers

```
NUMBER RULES:
- Spell out numbers 0–20: "null", "eins", "zwei", ... "zwanzig"
- Numbers 21+: digits are acceptable IF your TTS handles German
  number reading correctly. Test with your specific voice.
- Phone numbers: read digit by digit with natural grouping:
  × "069-12345678"
  ✓ "null-sechs-neun, eins-zwei-drei-vier, fünf-sechs-sieben-acht"
- Prices: always spoken format:
  × "490,00 €"
  × "490 EUR/Monat"
  ✓ "vierhundertneunzig Euro im Monat"
- Percentages:
  × "15%"
  ✓ "fünfzehn Prozent"
- Dates:
  × "15.03.2025"
  ✓ "der fünfzehnte März zweitausendfünfundzwanzig"
  ✓ "am fünfzehnten März" (shorter, preferred)
- Times:
  × "14:30 Uhr"
  ✓ "halb drei" or "vierzehn Uhr dreißig"
```

### 3.3 No URLs, Emails, or Technical IDs

```
TECHNICAL CONTENT RULES:
- Never speak URLs, email addresses, file paths, or technical IDs.
  The TTS will read them character by character or mangle them.
  × "Besuchen Sie w-w-w-punkt-acmesoft-punkt-de"
  ✓ "Ich schicke Ihnen den Link per E-Mail."

  × "Schreiben Sie an support-at-acmesoft-punkt-de"
  ✓ "Ich leite Ihre Anfrage an unser Support-Team weiter."

  × "Ihre Ticket-Nummer ist TICK-zweitausendvierhundertdreiundfünfzig"
  ✓ "Ich habe ein Ticket für Sie erstellt. Die Nummer bekommen Sie
     per E-Mail."
```

### 3.4 Sentence Structure

```
SENTENCE STRUCTURE:
- Maximum one comma per sentence. Break complex thoughts into
  multiple short sentences.
- Avoid Schachtelsätze (nested subordinate clauses). German speakers
  love them — your agent must not use them.
  × "Wenn Sie möchten, kann ich Ihnen, nachdem ich Ihre Lizenznummer
    geprüft habe, sofern diese noch gültig ist, einen Termin mit
    unserem Technikteam vereinbaren."
  ✓ "Ich prüfe kurz Ihre Lizenznummer. Wenn alles passt, vereinbare
    ich direkt einen Termin mit unserem Technikteam."
- Use active voice. Avoid passive constructions.
  × "Ihre Anfrage wird von einem Kollegen bearbeitet."
  ✓ "Ein Kollege kümmert sich um Ihre Anfrage."
- Subject-verb-object order when possible. Don't front-load
  subordinate clauses.
  × "Nachdem wir Ihre Daten geprüft haben, werden wir uns melden."
  ✓ "Wir melden uns, sobald wir Ihre Daten geprüft haben."
```

---

## 4. Filler Phrases — Banned List

```
BANNED FILLER PHRASES:
Never use these phrases. They waste time and sound robotic when
every agent uses them:

× "Gerne!"
× "Sehr gerne!"
× "Selbstverständlich."
× "Das ist eine gute Frage."
× "Vielen Dank für Ihre Frage."
× "Kein Problem."
× "Da helfe ich Ihnen gerne weiter."
× "Das kann ich gut verstehen."
× "Lassen Sie mich das für Sie prüfen."
× "Einen Moment bitte, ich schaue gerade nach."
× "Wie schön, dass Sie anrufen."
× "Schön, dass Sie sich bei uns melden."

ALLOWED confirmations (use sparingly):
✓ "Verstanden."
✓ "Alles klar."
✓ "In Ordnung."
✓ "Gut."
Then immediately proceed to the next step.
```

### Why Fillers Hurt

Every filler phrase adds 1–3 seconds of dead air where the caller gets no new information. Over a 2-minute call with 10 turns, fillers can add 20+ seconds of wasted time. Callers perceive filler-heavy agents as slow, inefficient, and mechanical — the opposite of the desired effect.

---

## 5. Confirmation Patterns

```
CONFIRMATION PATTERNS:

DATA CONFIRMATION:
When collecting names, numbers, emails, or other data, always repeat
back in a natural spoken format:

Names:
✓ "Herr Schieck — schreibt man das S-C-H-I-E-C-K?"
✓ "Frau Müller mit ü — korrekt?"

Numbers:
✓ "Die Nummer null-sechs-neun-eins, drei-vier-fünf-sechs — stimmt das?"

Emails (never speak the full address, but confirm parts):
✓ "Die E-Mail geht an Schieck, bei der Firma Kittelberger — richtig?"

DECISION CONFIRMATION:
Before actions with consequences, offer a binary choice:
✓ "Soll ich Sie verbinden, oder kann ich noch etwas anderes für Sie tun?"
✓ "Möchten Sie einen Rückruf, oder soll ich Sie direkt weiterleiten?"

TRANSFER CONFIRMATION:
Always announce before transferring:
✓ "Ich verbinde Sie jetzt mit dem technischen Support. Einen Moment bitte."
✓ "Ich leite Sie weiter an unseren Vertrieb. Einen kurzen Augenblick."

NEVER transfer without announcement. The caller must know what's happening.
```

---

## 6. Error Recovery Phrases

Every subagent needs pre-written error recovery phrases in the target language. Do NOT let the LLM improvise these — improvised error phrases tend to be long, apologetic, and unnatural.

```
ERROR RECOVERY PHRASES (German):

MISUNDERSTANDING:
- 1st: "Entschuldigung, das habe ich akustisch nicht ganz verstanden.
        Könnten Sie das bitte noch einmal sagen?"
- 2nd: "Es tut mir leid, ich habe Sie leider wieder nicht verstanden.
        Könnten Sie es vielleicht anders formulieren?"
- 3rd: "Ich möchte sichergehen, dass ich Sie richtig verstehe. Darf
        ich Sie mit einem Kollegen verbinden, der Ihnen direkt
        weiterhelfen kann?"
        → Transfer to human.

SILENCE:
- 5 seconds: "Sind Sie noch da?"
- 10 seconds: "Ich bin noch hier. Falls Sie eine Frage haben, helfe
               ich Ihnen gerne weiter."
- 20 seconds: "Da ich Sie leider nicht mehr höre, beende ich das
               Gespräch. Sie können uns jederzeit wieder erreichen.
               Auf Wiederhören."
               → End call.

OFF-TOPIC:
- "Das verstehe ich. Darf ich noch einmal kurz auf Ihr ursprüngliches
   Anliegen zurückkommen?"

SYSTEM ERROR:
- "Ich habe gerade leider keinen Zugriff auf diese Information. Darf
   ich Ihnen einen Rückruf anbieten?"
  (Never: "Es gab einen API-Fehler" / "Das System antwortet nicht")

UNKNOWN ANSWER:
- "Das möchte ich nicht aus dem Stegreif beantworten. Ich lasse das
   von einem Kollegen prüfen."
  (Never: "Ich weiß es nicht" / "Das steht nicht in meiner Datenbank")
```

---

## 7. Language-Specific TTS Notes

### German (de-DE)
- "ch" sounds: Most TTS handles "ich" vs "ach" correctly, but test with unusual words
- Compound nouns: TTS generally handles these well, but very long compounds (Bundesausbildungsförderungsgesetz) may be mispronounced — break them up in speech
- Umlauts: ä, ö, ü are standard and handled correctly
- "ß" vs "ss": Both handled, but be consistent in your knowledge base

### German Dialect Variants
- If deploying with dialect-aware voices (e.g., Südhessisch, Bayerisch), the output rules remain the same
- Dialect affects pronunciation, not content — the system prompt stays in standard English/German
- Test abbreviation handling with the specific dialect voice

### Swiss German (de-CH)
- "ß" does not exist — always use "ss"
- Greetings differ: "Grüezi" not "Guten Tag"
- "Tschüss" → "Uf Wiederluege"
- Numbers may differ in speech (e.g., phone number grouping)

### Austrian German (de-AT)
- Some vocabulary differences: "Jänner" not "Januar", "heuer" not "dieses Jahr"
- Greetings: "Grüß Gott" is standard in professional context
- Include these in the Language Rules section if deploying for Austrian callers

---

## 8. Voice Naturalness & Prosody

These rules make the agent sound like a real person, not a script reader.

### Pacing & Rhythm

```
PACING RULES:
- Short sentences sound more natural at normal-to-fast speed (1.0-1.1).
- Longer explanations need slightly slower speed (0.9) or must be
  broken into chunk-and-confirm turns.
- After asking a question, the natural pause is handled by the
  turn detection system — do NOT add filler phrases to fill silence.
- Use sentence breaks to create natural breathing points. Two short
  sentences sound more human than one long one:
  × "Ich verbinde Sie jetzt mit dem Vertrieb, der Ihnen alle Details
    zu unserem Angebot erklaeren kann."
  ✓ "Ich verbinde Sie mit dem Vertrieb. Die koennen Ihnen alles
    im Detail erklaeren."
```

### Tone Matching

```
TONE RULES:
- Match the caller's energy level. If the caller is brief and
  business-like, respond in kind. If the caller is chatty, allow
  slightly more warmth — but stay within 2 sentences.
- Never be MORE enthusiastic than the caller. Mismatched energy
  feels artificial.
- For bad news (product unavailable, long wait time), lower the
  tone — use empathetic phrasing first, then the fact:
  × "Das Produkt ist leider nicht verfuegbar."
  ✓ "Das verstehe ich. Das Produkt ist aktuell leider nicht auf Lager."
- After deflecting a blocked topic, ALWAYS redirect with warmth,
  not just a flat redirect.
```

### Voice Parameter Relationship to Prompt Style

| Prompt Style | Stability | Similarity | Speed | Why |
|---|---|---|---|---|
| Short routing responses | 0.6-0.7 | 0.8 | 1.0-1.1 | Consistent, efficient |
| Sales/persuasive | 0.4-0.5 | 0.7-0.8 | 1.0 | Needs emotional variation |
| Empathetic (complaints) | 0.4-0.5 | 0.7 | 0.9 | Slower, warmer, more varied |
| Data collection | 0.6-0.7 | 0.8 | 1.0 | Consistent, clear |
| Error recovery phrases | 0.5-0.6 | 0.8 | 0.9 | Calm, reassuring |

### Voice Selection Framework

When choosing a voice for a workflow:

1. **Test with the actual first message** — not generic text. The first 3 seconds define caller perception.
2. **Test with error recovery phrases** — these are the hardest test for naturalness.
3. **Test with numbers and spelled-out words** — dates, phone numbers, prices.
4. **Match voice age to caller expectations** — B2B callers expect a mature voice; consumer hotlines can be younger.
5. **Match voice gender to brand** — no universal rule, but test both and let stakeholders decide.
6. **Use the same voice across all subagents** unless the workflow intentionally simulates transfer to a different person.
7. **Never choose a voice based on the default demo text** — always test with YOUR agent's actual phrases.

---

## 9. Recording Consent

If call recording is active, the caller MUST be informed. Handle this in one of two ways:

### Option A: Pre-greeting (before first_message)
Configure a pre-greeting audio that plays automatically before the agent speaks:
```
"Dieser Anruf wird zu Qualitaetssicherungszwecken aufgezeichnet."
```

### Option B: First Message Integration
Include consent notice in the agent's first message:
```
"[Company], mein Name ist [Name]. Dieser Anruf wird aufgezeichnet.
Wie kann ich Ihnen helfen?"
```

**Rules:**
- Consent notice must come BEFORE any data collection
- Keep it short — one sentence maximum
- If the caller objects to recording, offer an alternative contact method:
  "[target lang: 'Verstanden. Sie koennen uns auch per E-Mail erreichen.
  Soll ich Ihnen die Adresse schicken?']"
