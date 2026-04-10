# Privacy & Data Retention — DSGVO, PII Redaction & Audio Saving

Controls how long conversation data is stored, whether audio is saved, and how sensitive information is redacted. Critical for German/EU phone agents under DSGVO (GDPR).

---

## 1. Retention Settings

### Default: 2 years

ElevenLabs retains conversation data for 2 years by default. You can change this per agent.

| Setting | Value | Effect |
|---|---|---|
| Custom days | e.g., 30, 90, 365 | Data deleted after N days |
| Unlimited | -1 | Data kept indefinitely |
| Immediate deletion | 0 | Data deleted immediately after call (maximum privacy) |

Retention applies separately to:
- **Conversation transcripts** — text records of all interactions
- **Audio recordings** — voice recordings from both user and agent

### Location

Agent settings → **Advanced** tab → **Data Retention** section.

### Applying to Existing Data

When reducing retention period, you can choose:
- Apply to existing data (may delete older conversations immediately)
- Apply only to new conversations going forward

**Warning:** Reducing retention with "apply to existing" is irreversible.

---

## 2. Audio Saving

### Default: Enabled

Audio recordings are saved by default. Toggle per agent.

| Setting | Effect |
|---|---|
| **Enabled** | Call audio available for review in call history |
| **Disabled** | No audio recordings stored (transcripts still available) |

### Location

Agent settings → **Advanced** tab → **Privacy Settings** section.

**Note:** Disabling audio saving only affects NEW calls. Existing recordings remain until deleted via retention settings.

---

## 3. Conversation History Redaction (Enterprise)

Automatically detects and removes sensitive information from stored conversations BEFORE storage.

**Availability:** Enterprise plans only.

### How It Works

1. Conversation ends
2. Post-processing scans transcript and audio
3. Detected entities are replaced:
   - **Transcripts:** Entity replaced with `[ENTITY_NAME]` (e.g., `[NAME]`, `[EMAIL_ADDRESS]`)
   - **Audio:** Entity replaced with bleep sound
   - **Webhooks:** Also receive redacted data
4. Redacted conversation stored

**Note:** Short delay before conversation appears in history (post-processing time).

### Configuration

Agent settings → **Advanced** tab → **Privacy** section.

| Field | Type | Description |
|---|---|---|
| `enabled` | Boolean | Turn redaction on/off |
| `entities` | List | Which entity types to redact (see Section 4) |

---

## 4. Supported Redaction Entities

Enable ONLY the entities you need — fewer types = more accurate detection.

### Names

| Entity | What It Redacts | Placeholder |
|---|---|---|
| `name` | All name types (given, family, other) | `[NAME]` |
| `name.name_given` | First/middle names | `[NAME_GIVEN]` |
| `name.name_family` | Last names | `[NAME_FAMILY]` |
| `name.name_other` | Titles, nicknames, initials | `[NAME_OTHER]` |

### Contact

| Entity | What It Redacts | Placeholder |
|---|---|---|
| `email_address` | Email addresses (incl. spelled-out) | `[EMAIL_ADDRESS]` |
| `contact_number` | Phone/fax/mobile numbers | `[CONTACT_NUMBER]` |

### Personal Information

| Entity | What It Redacts | Placeholder |
|---|---|---|
| `dob` | Birth dates | `[DOB]` |
| `age` | Age numbers | `[AGE]` |
| `occupation` | Job titles | `[OCCUPATION]` |
| `religious_belief` | Religious affiliation | `[RELIGIOUS_BELIEF]` |
| `political_opinion` | Political affiliation | `[POLITICAL_OPINION]` |

### Financial Identifiers

| Entity | What It Redacts | Placeholder |
|---|---|---|
| `financial_id` | ALL financial IDs (parent) | `[FINANCIAL_ID]` |
| `financial_id.payment_card.payment_card_number` | Credit/debit card numbers | `[PAYMENT_CARD_NUMBER]` |
| `financial_id.payment_card.payment_card_cvv` | CVV codes | `[PAYMENT_CARD_CVV]` |
| `financial_id.payment_card.payment_card_expiration_date` | Card expiry dates | `[PAYMENT_CARD_EXPIRATION_DATE]` |
| `financial_id.bank_account.bank_account_number` | Bank account / IBAN | `[BANK_ACCOUNT_NUMBER]` |
| `financial_id.bank_account.bank_routing_number` | Routing numbers | `[BANK_ROUTING_NUMBER]` |
| `financial_id.bank_account.swift_bic_code` | SWIFT/BIC codes | `[SWIFT_BIC_CODE]` |

### Location

| Entity | What It Redacts | Placeholder |
|---|---|---|
| `location` | ALL location types (parent) | `[LOCATION]` |
| `location.location_address` | Street addresses | `[LOCATION_ADDRESS]` |
| `location.location_city` | City names | `[LOCATION_CITY]` |
| `location.location_postal_code` | Postal/zip codes | `[LOCATION_POSTAL_CODE]` |

### Credentials

| Entity | What It Redacts | Placeholder |
|---|---|---|
| `username` | Usernames, login names | `[USERNAME]` |
| `password` | Passwords, PINs, access keys | `[PASSWORD]` |

### Unique Identifiers

| Entity | What It Redacts | Placeholder |
|---|---|---|
| `unique_id.government_issued_id` | SSN, passport, driver's license | `[GOVERNMENT_ISSUED_ID]` |
| `unique_id.healthcare_number` | Medical record numbers, health plan IDs | `[HEALTHCARE_NUMBER]` |
| `unique_id.account_number` | Customer account numbers | `[ACCOUNT_NUMBER]` |

### Medical

| Entity | What It Redacts | Placeholder |
|---|---|---|
| `medical` | ALL medical types (parent) | `[MEDICAL]` |
| `medical.medical_condition` | Diseases, symptoms, diagnoses | `[MEDICAL_CONDITION]` |
| `medical.medication` | Drug names, dosages | `[MEDICATION]` |
| `medical.medical_procedure` | Surgeries, treatments | `[MEDICAL_PROCEDURE]` |

### Parent/Child Hierarchy

Entities use dot notation. Enabling a parent covers all children:
- `name` → redacts `name.name_given`, `name.name_family`, `name.name_other`
- `financial_id` → redacts ALL payment card and bank account fields

If both parent and child are configured, the child is ignored (parent already covers it).

---

## 5. Recommended Configurations

### Maximum Privacy (DSGVO-Strict)

```
Retention: 0 days (immediate deletion)
Audio saving: Disabled
Redaction: Not needed (nothing stored)
```

Use when: Legal requires no data storage. Debugging via post-call webhooks only.

### Balanced (Standard German Business)

```
Retention: 90 days
Audio saving: Enabled
Redaction: name, email_address, contact_number, financial_id
```

Use when: Need to review calls for quality, but minimize PII exposure.

### Compliance (HIPAA / Healthcare)

```
Retention: 2190 days (6 years minimum)
Audio saving: Enabled
Redaction: name, medical, unique_id.healthcare_number, financial_id
```

### Debugging-Friendly

```
Retention: 30 days
Audio saving: Enabled
Redaction: financial_id, password, unique_id.government_issued_id
```

Use when: Active development phase, need full access for iteration.

---

## 6. Zero Retention Mode vs. Redaction

| | Redaction | Zero Retention Mode (ZRM) |
|---|---|---|
| How it works | Detects and replaces specific entities | Discards ALL conversation data |
| Detection | High but not 100% | 100% (nothing stored) |
| HIPAA compliance | Not guaranteed alone | Can enable compliance |
| Debugging access | Yes (redacted transcripts available) | No stored data |
| Webhooks | Receive redacted data | Must set up own HTTP endpoint |
| ElevenLabs admin access | May access internal logs | Cannot access data |

**Both can be enabled simultaneously:** ZRM prevents storage, webhooks receive redacted data.

**Fallback behavior:** If redaction fails (detection error), system falls back to ZRM behavior — no data stored.

---

## 7. Integration with Agent Workflow

### Recording Consent

Every phone agent MUST address recording consent. Options:

1. **Pre-greeting** (before first message): Automated legal notice
2. **First message**: Include consent notice as part of greeting
3. **Dynamic variable**: Check `{{recording_consent}}` if pre-collected

### DSGVO Checklist for Phone Agents

- [ ] Retention period set (not default 2 years unless justified)
- [ ] Audio saving decision documented
- [ ] Recording consent implemented (pre-greeting or first message)
- [ ] Redaction entities configured (if enterprise)
- [ ] Webhook endpoints secured (HMAC verification, IP whitelisting)
- [ ] Data/privacy guardrails in every agent prompt (see `guardrails.md`)
- [ ] DSGVO Art. 15 handling (data subject requests → transfer to DPO)
- [ ] Privacy configuration documented in project `notes.md`
