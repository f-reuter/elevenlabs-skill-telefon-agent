# Pronunciation Dictionary — Custom TTS Pronunciation

Pronunciation dictionaries customize how the TTS engine pronounces specific words or phrases. Essential for company names, product names, technical terms, and German-specific terms that TTS engines mispronounce.

---

## 1. When to Use

| Problem | Solution |
|---|---|
| Company name mispronounced ("AcmeSoft" read letter by letter) | Alias: "AcmeSoft" → "Äckmisoft" |
| Abbreviation not expanded ("GmbH" read as letters) | Alias: "GmbH" → "Gesellschaft mit beschraenkter Haftung" |
| Product name wrong stress ("Kubernetes") | IPA phoneme tag |
| Foreign name in German context ("Thierry") | Alias: "Thierry" → "Tierri" |
| Medical/legal term mispronounced | Alias to phonetic German spelling |

---

## 2. Configuration

Location: **Voice tab** in agent configuration.

**Important constraints:**
- Phoneme tags (IPA/CMU) ONLY work with `eleven_flash_v2` model — other models silently skip them
- Phoneme tags ONLY work for English — for German and other languages, use **alias tags**
- Multiple dictionaries can be uploaded simultaneously

**For German phone agents:** Always use **alias tags**. They work across all models and languages.

---

## 3. Dictionary File Format

XML-based `.pls` files using the W3C Pronunciation Lexicon specification.

### Basic Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<lexicon version="1.0"
      xmlns="http://www.w3.org/2005/01/pronunciation-lexicon"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.w3.org/2005/01/pronunciation-lexicon
        http://www.w3.org/TR/2007/CR-pronunciation-lexicon-20071212/pls.xsd"
      alphabet="ipa" xml:lang="de">

  <!-- Alias: replace spelling for TTS -->
  <lexeme>
    <grapheme>GmbH</grapheme>
    <alias>Gesellschaft mit beschraenkter Haftung</alias>
  </lexeme>

  <lexeme>
    <grapheme>AcmeSoft</grapheme>
    <alias>Aeckmisoft</alias>
  </lexeme>

  <!-- IPA phoneme (English only, Flash v2 only) -->
  <lexeme>
    <grapheme>nginx</grapheme>
    <phoneme>ˈɛndʒɪnˈɛks</phoneme>
  </lexeme>
</lexicon>
```

---

## 4. Two Notation Systems

| System | How It Works | Language Support | Model Support |
|---|---|---|---|
| **Alias** | Substitutes entire spelling (e.g., "UN" → "United Nations") | ALL languages | ALL models |
| **IPA** | Precise phonetic notation (e.g., `/ˈɛndʒɪnˈɛks/`) | English only | `eleven_flash_v2` only |
| **CMU** | ASCII-based phonetics (e.g., "T AH M EY T OW") | English only | `eleven_flash_v2` only |

**Recommendation for German agents:** Use alias tags exclusively. They are the most reliable approach.

---

## 5. Common German Alias Examples

```xml
<!-- Company & Brand Names -->
<lexeme>
  <grapheme>AcmeSoft</grapheme>
  <alias>Aeckmisoft</alias>
</lexeme>

<!-- Abbreviations -->
<lexeme>
  <grapheme>GmbH</grapheme>
  <alias>Gesellschaft mit beschraenkter Haftung</alias>
</lexeme>

<lexeme>
  <grapheme>AG</grapheme>
  <alias>Aktiengesellschaft</alias>
</lexeme>

<lexeme>
  <grapheme>e.V.</grapheme>
  <alias>eingetragener Verein</alias>
</lexeme>

<!-- Technical Terms -->
<lexeme>
  <grapheme>API</grapheme>
  <alias>A P I</alias>
</lexeme>

<lexeme>
  <grapheme>CRM</grapheme>
  <alias>C R M</alias>
</lexeme>

<lexeme>
  <grapheme>SaaS</grapheme>
  <alias>Saas</alias>
</lexeme>

<!-- Foreign Names -->
<lexeme>
  <grapheme>Thierry</grapheme>
  <alias>Tierri</alias>
</lexeme>

<lexeme>
  <grapheme>LinkedIn</grapheme>
  <alias>Linkt-in</alias>
</lexeme>

<!-- Medical/Legal -->
<lexeme>
  <grapheme>Ibuprofen</grapheme>
  <alias>Ibuprofen</alias>
</lexeme>

<lexeme>
  <grapheme>DSGVO</grapheme>
  <alias>Datenschutz Grundverordnung</alias>
</lexeme>
```

---

## 6. Best Practices

1. **Case sensitivity:** Create separate entries for capitalized and lowercase versions if both appear
2. **Test with your voice:** Always test pronunciations with the actual voice and model selected for the agent
3. **Focus on critical words:** Only add words that are frequently mispronounced or brand-critical
4. **Maintain and document:** Keep dictionaries organized — add comments explaining each entry
5. **Per-agent dictionaries:** Different agents may need different dictionaries (e.g., medical terms only for support agent)
6. **Use AI for IPA:** Ask Claude or ChatGPT to generate IPA notation for complex words

---

## 7. Integration with Agent Workflow

### Where in the Build Process

Add pronunciation dictionaries in **Step 7: Configure ElevenLabs Parameters** (see `SKILL.md`), after selecting the voice but before testing.

### Per-Project Recommendation

Save pronunciation dictionaries to the project folder:
```
~/.claude/elevenlabs-projects/{project}/knowledge/pronunciation-{lang}.pls
```

Include pronunciation recommendations in the agent delivery output (Section 8 of `SKILL.md` Output Format).
