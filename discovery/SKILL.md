---
name: discovery
description: Analyze discovery call transcripts from Attio (primary) and Granola (secondary) against the Discovery Questions Tracker template, creates or appends to a Google Sheet in the Discovery folder with full scoring, customer answers, and evidence, and posts a summary note to the Attio company record. Outputs an xlsx with tabs matching the original spreadsheet template. Use when the user wants to process a discovery call, fill out discovery questions, create a discovery doc, or run /discovery.
license: MIT
metadata:
  version: 0.4.0
---

# Discovery Call Processor

Analyze discovery call transcripts from Attio and Granola and produce a structured Google Sheet (xlsx with multiple tabs) that mirrors the Discovery Questions Tracker spreadsheet, with auto-scored maturity dimensions, customer answers, and evidence.

**Template reference**: See [Discovery Questions Tracker](https://docs.google.com/spreadsheets/d/1EA2VrQhKgGYeQ7cY0P6NmDX2ucDTKizw/edit?gid=304954826#gid=304954826) in Constants — the full question set, scoring rubrics, and follow-ups live there.

---

## Execution Flow

Skill invoked -> Phase 1 (Gather Input) -> Phase 2 (Fetch Data from Attio + Granola) -> Phase 3 (Analyze & Score) -> Phase 4 (Create or Append Sheet) -> Phase 5 (Create Attio Note) -> Phase 6 (Report Link)

---

## Phase 1 -- Gather Input

### Step 1: Ask for Customer Name

Ask the user to provide the **customer/company name** for the discovery call.

Example prompt: "Which customer is this discovery call for? (company name)"

Store: `customer_name`

### Step 2: Ask About Call Sequence

Ask the user: "Is this the first discovery call with this customer, or a follow-up (2nd, 3rd, etc.)?"

- If **first call**: will create a new doc
- If **follow-up**: will search for and append to the existing discovery doc

### Step 3: Ask for Specific Meeting (Optional)

Ask: "Do you want me to pull all recent calls with this customer, or a specific one? (If specific, paste the Granola link or Attio recording ID.)"

- If **all recent**: will search both Attio and Granola for matching calls
- If **specific**: will use the provided ID as the anchor and still search for supplementary data from the other source

---

## Phase 2 -- Fetch Data from Attio + Granola

> **Source priority**: Attio is PRIMARY; Granola fills gaps only. When the same call appears in both systems, use the Attio version.

### Step 1: Load MCP Tools

Load all required Attio and Granola MCP tools in parallel:
- Attio: `search-records`, `semantic-search-call-recordings`, `get-call-recording`, `search-notes-by-metadata`
- Granola: `list_meetings`, `get_meeting_transcript`

### Step 2: Fetch Attio Company Record

```
mcp__attio__search-records with:
  object: "companies"
  search_term: "<customer_name>"
```

Extract: official company name, website/domain, custom fields (org size, tech stack, etc.).

Store: `company_record_id`, `company_data`, `company_domain`

If not found, proceed with the customer name provided by the user.

### Step 3: Fetch Attio Call Recordings (Primary Source)

Run two semantic searches in parallel:

**Search 1 -- Discovery context:**
```
mcp__attio__semantic-search-call-recordings with:
  query: "<customer_name> discovery skills context AI tooling engineering team"
  max_results: 5
```

**Search 2 -- Technical & evaluation context:**
```
mcp__attio__semantic-search-call-recordings with:
  query: "<customer_name> evaluation POC budget timeline platform developer productivity"
  max_results: 5
```

Deduplicate by `recording_id`. Fetch the full transcript for each unique relevant recording:

```
mcp__attio__get-call-recording with:
  recording_id: "<recording_id>"
```

Store: `attio_calls` -- list of `{ recording_id, title, date, attendees, transcript }`

### Step 4: Fetch Attio Notes (Supplementary)

```
mcp__attio__search-notes-by-metadata with:
  filter: { "parent_object": "companies", "parent_record_id": "<company_record_id>" }
```

Store: `attio_notes` -- supplementary context, not a primary transcript source

### Step 5: Fetch Granola Meetings (Secondary Source)

```
mcp__granola__list_meetings with last 90 days
```

Filter for meetings where the title contains the customer name (case-insensitive). For each match:

```
mcp__granola__get_meeting_transcript with:
  meeting_id: "<meeting_id>"
```

Store: `granola_calls` -- list of `{ meeting_id, title, date, attendees, transcript }`

### Step 6: Deduplicate Calls (Attio Primary, Granola Secondary)

1. **Build Attio index**: fingerprint each Attio call as `{ date (YYYY-MM-DD), normalized_title, attendee_domains }`.
2. **Match Granola calls against index** (in priority order):
   - Same date + overlapping attendee email/domain → **DUPLICATE, discard Granola version**
   - Same date + similar title (shared keyword: "discovery", "intro", "follow-up") → **LIKELY DUPLICATE, discard Granola version**
   - No match → **UNIQUE to Granola, keep it**
3. **Merged list** = all Attio calls + Granola-only calls, sorted chronologically (oldest first).

Log: `attio_call_count`, `granola_unique_count`, `granola_duplicates_dropped`

Store: `merged_calls` with source attribution (`attio` or `granola`)

### Step 7: Extract Company Name and Metadata

Prefer Attio company record; fall back to meeting metadata.

Store: `company_name`, `call_date` (most recent call), `attendees_list` (union across all calls)

If company name is still ambiguous, ask the user.

---

## Phase 3 -- Analyze & Score

Analyze all transcripts in `merged_calls` against the Discovery Questions Tracker template (see Constants). Always include **source attribution** in evidence (e.g., `[Attio, 2026-02-15]` or `[Granola, 2026-02-10]`). When a topic appears in multiple calls, prefer Attio and note supplementary Granola context.

### Maturity Qualification (8 Dimensions)

Score each dimension 1–3 using the rubric in the template spreadsheet.

| Dimension |
|-----------|
| AI Tooling Landscape |
| Context Awareness |
| Internal Knowledge Surface |
| Knowledge Distribution Pain |
| Quality Measurement |
| Platform / DevEx Ownership |
| Scale of Developer Base |
| Contribution Readiness |

**Score guide**: 3 = High (proceed); 2 = Medium (proceed with caution); 1 = Low (consider qualifying out)

**Total score bands**: 20–24 High maturity | 14–19 Medium | 8–13 Low

For each dimension record: Score (1–3), Evidence (verbatim quote + source attribution), Notes.

### Sections A through F

For each question extract these columns (see template for full question text, Good/Caution/Red Flag answers, and follow-ups):

1. **#** (A1, A2, B1, …)
2. **Question** (from template)
3. **Why It Matters** (from template)
4. **Customer Answer** — verbatim quotes where possible; include source attribution. If covered in multiple calls, combine with per-source attribution. Write "Not discussed." if absent.
5. **Good Answer** (from template)
6. **Caution Answer** (from template)
7. **Red Flag Answer** (from template)
8. **Assessment** — GOOD / CAUTION / RED FLAG / N/A
9. **Follow-Up Used/Needed**
10. **Notes**

**Sections**:
- A — Context & Skills Today (A1–A4)
- B — Skills Creation & Mgmt (B1–B4)
- C — Skills Quality & Eval (C1–C4)
- D — Decision & Budget (D1–D3)
- E — Success & POC Scoping (E1–E3)
- F — Competitive & Alternatives (F1–F2)

### Technical Environment

Extract the following factual data points from the transcripts. For each field, record the value and source attribution. If not discussed, leave the answer blank.

| Field | Description |
|-------|-------------|
| SCM | Source control management platform (e.g., GitHub, GitLab, Bitbucket, Azure DevOps) |
| Repo Structure | Monorepo or many repos? Note any details on count or organization |
| Primary Languages | Main coding languages used across the org |
| Agent Harnesses / Tooling | AI coding agent harnesses or developer tooling in use (e.g., Claude Code, Cursor, Copilot, Cline, Aider, Windsurf) |
| Vendor Lock-in Concerns | Any lock-in to specific harnesses or AI providers? Attitude toward switching or experimenting with new tools |
| Experimentation Attitude | Openness to trying new AI/dev tools — proactive adopters vs. cautious/policy-driven |
| Dedicated AEL Team Size | Size of the dedicated AI-Enhanced Engineering / AI Platform / DevEx team |
| Overall Team Size | Total engineering team size |
| Agent User Roles | Who in the org uses coding agents? Engineers only, or also designers, CS, PMs, etc.? |
| SSO Setup | SSO provider and relevant details (e.g., Okta, Azure AD, Google Workspace) |
| Existing Repo Context | What context exists in their repos today? AGENTS.md, CLAUDE.md, .cursorrules, etc.? |
| Tessl Feature Adoption | Already using any of: Skills, Plugins, Hooks, MCP servers, Cloud agents? Note specifics |
| Skills Storage | If using skills, how are they stored and distributed? (repo, registry, shared drive, etc.) |
| AI Code Review Tooling | AI-assisted code review tools in use (e.g., CodeRabbit, Codacy, Qodo, Greptile, native Copilot review) |

For each field, record: **Value** (what they said), **Evidence** (verbatim quote + source attribution), **Notes** (any caveats or follow-up needed).

### Feature Requests & Needs

Scan all transcripts for any features, capabilities, or product enhancements the customer explicitly requests, asks about, or implies they need. These may surface anywhere in the conversation — not just in response to direct questions.

For each feature identified, record:

1. **Feature** — short descriptive name (e.g., "SAML SSO support", "skill version pinning", "audit log export")
2. **Category** — classify as one of: Platform, Skills, Agents, Integrations, Analytics, Admin, Other
3. **Context** — one-line summary of why they want it or what problem it solves
4. **Verbatim Quote** — the most relevant customer quote with source attribution
5. **Urgency** — Blocker (can't proceed without it), Important (strong preference), Nice-to-have (mentioned in passing)
6. **Notes** — any related context, workarounds they're using, or similar requests from other sections

If no feature requests are identified, note "No explicit feature requests captured in this call."

### Security & Governance Requirements

Extract any requirements, concerns, or questions the customer raises about security, compliance, governance, access control, data handling, or audit capabilities. These are critical for scoping POCs and understanding procurement blockers.

For each requirement identified, record:

1. **Requirement** — short descriptive name (e.g., "SOC 2 Type II certification", "data residency in EU", "prompt logging and audit trail")
2. **Category** — classify as one of: Compliance & Certifications, Data Privacy & Residency, Access Control & SSO, Audit & Logging, Network & Infrastructure, AI Safety & Guardrails, Other
3. **Context** — why this matters to them (regulatory, internal policy, customer requirement, etc.)
4. **Verbatim Quote** — the most relevant customer quote with source attribution
5. **Severity** — Blocker (deal-breaker if not met), Required (must have for production), Preferred (would like but can work around)
6. **Current State** — how they handle this today, if mentioned
7. **Notes** — any follow-up needed, related certifications, or timeline constraints

If no security/governance topics are raised, note "No security or governance requirements captured in this call."

### Call Checklist

For each of the 20 must-ask questions drawn from Sections A–F, mark:
- **Asked?**: Yes / No based on transcript coverage
- **Key Takeaway**: One-sentence summary (or "Not covered")

The 20 checklist items map directly to the must-ask questions in Sections A–F of the template (5 Context Today, 4 Skills Mgmt, 4 Quality & Eval, 3 Decision, 3 POC Scoping, 2 Competitive).

---

## Phase 4 -- Create or Append Sheet

Output is an xlsx workbook built with Python openpyxl, then uploaded to Google Drive as a native Google Sheet.

### Workbook Structure (11 Tabs)

> Full tab specifications (column layouts, color codes, header styles) are defined in `XLSX_STRUCTURE.md` alongside this skill. Key points:

- **Tab 1 "Maturity Qualification"**: one row per dimension, total score row (green fill if high maturity), scoring guide rows.
- **Tabs 2–7 "Sections A–F"**: 11 columns each; assessment/answer column fills: Good → `#C6F6D5`, Caution → `#FEFCBF`, Red Flag → `#FED7D7`, N/A → `#D9D9D9`.
- **Tab 8 "Technical Environment"**: 4 columns — Field | Value | Evidence | Notes. One row per field (SCM, Repo Structure, Primary Languages, Agent Harnesses / Tooling, Vendor Lock-in Concerns, Experimentation Attitude, Dedicated AEL Team Size, Overall Team Size, Agent User Roles, SSO Setup, Existing Repo Context, Tessl Feature Adoption, Skills Storage, AI Code Review Tooling). Cells with values get light blue fill `#DBEAFE`; blank/unknown cells get light gray fill `#F3F4F6`.
- **Tab 9 "Feature Requests"**: 6 columns — Feature | Category | Context | Verbatim Quote | Urgency | Notes. One row per feature. Urgency fills: Blocker → `#FED7D7`, Important → `#FEFCBF`, Nice-to-have → `#DBEAFE`. If no features identified, single row: "No explicit feature requests captured in this call."
- **Tab 10 "Security & Governance"**: 7 columns — Requirement | Category | Context | Verbatim Quote | Severity | Current State | Notes. One row per requirement. Severity fills: Blocker → `#FED7D7`, Required → `#FEFCBF`, Preferred → `#DBEAFE`. If no requirements identified, single row: "No security or governance requirements captured in this call."
- **Tab 11 "Call Checklist"**: Category | Must-Ask Question | Asked? | Key Takeaway; Asked? fills: Yes → `#C6F6D5`, Partial → `#FEFCBF`, No → `#FED7D7`.
- **Global styling**: all headers dark slate `#2D3748` / white bold; data font `#1A202C`; wrap text, top-align, thin borders; appropriate column widths.

---

### Path A: New Spreadsheet (First Call)

#### Step 1: Build xlsx with openpyxl

Create and run a Python script that constructs the workbook per the structure above, saves to `/tmp/<Company>-discovery.xlsx`.

#### Step 2: Upload to Google Drive as native Google Sheet

**IMPORTANT**: Use this Ruby one-liner — the `drive_manager.rb` upload does NOT convert xlsx. Setting `mime_type: "application/vnd.google-apps.spreadsheet"` on the file metadata triggers conversion.

```bash
/opt/homebrew/opt/ruby/bin/ruby -e '
require "google/apis/drive_v3"
require "googleauth"
require "googleauth/stores/file_token_store"
require "json"

credentials_path = File.join(Dir.home, ".claude", ".google", "client_secret.json")
token_path = File.join(Dir.home, ".claude", ".google", "token.json")

scopes = [Google::Apis::DriveV3::AUTH_DRIVE]
client_id = Google::Auth::ClientId.from_file(credentials_path)
token_store = Google::Auth::Stores::FileTokenStore.new(file: token_path)
authorizer = Google::Auth::UserAuthorizer.new(client_id, scopes, token_store)
credentials = authorizer.get_credentials("default")

drive = Google::Apis::DriveV3::DriveService.new
drive.authorization = credentials

file_metadata = Google::Apis::DriveV3::File.new(
  name: "<Company>-discovery",
  mime_type: "application/vnd.google-apps.spreadsheet",
  parents: ["1qXQHbloLqQGHVcLakuMXycLi74eqdVh6"]
)

result = drive.create_file(
  file_metadata,
  upload_source: "/tmp/<Company>-discovery.xlsx",
  content_type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
  fields: "id, name, mimeType, webViewLink, parents"
)

puts JSON.pretty_generate({
  status: "success",
  id: result.id,
  name: result.name,
  mime_type: result.mime_type,
  web_view_link: result.web_view_link
})
'
```

Save the returned `id` and `web_view_link`.

---

### Path B: Append to Existing Spreadsheet (Follow-up Call)

#### Step 1: Find Existing Sheet

```bash
/Users/Tessl-Leo/.claude/skills/google-docs/scripts/drive_manager.rb search \
  --query "'1qXQHbloLqQGHVcLakuMXycLi74eqdVh6' in parents and name contains '<Company>'"
```

If not found, ask the user to confirm the company name or create a new sheet instead.

#### Step 2: Download Existing Sheet

```bash
/Users/Tessl-Leo/.claude/skills/google-docs/scripts/drive_manager.rb download \
  --file-id <existing_file_id> \
  --output /tmp/<Company>-discovery-existing.xlsx
```

#### Step 3: Update the xlsx with openpyxl

Load the existing workbook and for each sheet:
- Replace cells containing "Not discussed in this call" with new answers from this call
- Update maturity scores if new evidence changes them
- Update the checklist with newly covered questions
- Add a new row at the top of Maturity Qualification noting `Updated: Call <N> | <date> | <attendees>`

#### Step 4: Re-upload Updated Sheet

```bash
/Users/Tessl-Leo/.claude/skills/google-docs/scripts/drive_manager.rb update \
  --file-id <existing_file_id> \
  --file /tmp/<Company>-discovery-updated.xlsx
```

---

## Phase 5 -- Create Attio Note

Attach a discovery summary note to the customer's Attio company record. This makes key findings visible directly in the CRM without opening the spreadsheet.

> **Prerequisite**: `company_record_id` must exist from Phase 2. If the company was not found in Attio, skip this phase and note the skip in the final report.

### Note Format

Attio notes support markdown headings, lists, bold, italic, highlight, and links — but **not tables or code blocks**. Use the structured list format below.

```
mcp__attio__create-note with:
  title: "<Company Name> — Discovery Summary"
  parent_object: "companies"
  parent_record_id: "<company_record_id>"
  content: <see template below>
```

**Note content template:**

```markdown
[View full Discovery Sheet](<web_view_link>)

## Overview

- **Company:** <Company Name>
- **Call:** <call_number> (<call_date>)
- **Attendees:** <attendees_list>
- **Maturity Score:** ==<total>/24 (<High/Medium/Low> maturity)==

---

## Maturity Dimensions

- **AI Tooling Landscape** — <score>/3 · <one-line evidence summary>
- **Context Awareness** — <score>/3 · <one-line evidence summary>
- **Internal Knowledge Surface** — <score>/3 · <one-line evidence summary>
- **Knowledge Distribution Pain** — <score>/3 · <one-line evidence summary>
- **Quality Measurement** — <score>/3 · <one-line evidence summary>
- **Platform / DevEx Ownership** — <score>/3 · <one-line evidence summary>
- **Scale of Developer Base** — <score>/3 · <one-line evidence summary>
- **Contribution Readiness** — <score>/3 · <one-line evidence summary>

---

## Technical Environment

- **SCM:** <value or *Not discussed*>
- **Repo Structure:** <value or *Not discussed*>
- **Primary Languages:** <value or *Not discussed*>
- **Agent Harnesses / Tooling:** <value or *Not discussed*>
- **Vendor Lock-in:** <value or *Not discussed*>
- **Experimentation Attitude:** <value or *Not discussed*>
- **Dedicated AEL Team Size:** <value or *Not discussed*>
- **Overall Team Size:** <value or *Not discussed*>
- **Agent User Roles:** <value or *Not discussed*>
- **SSO Setup:** <value or *Not discussed*>
- **Existing Repo Context:** <value or *Not discussed*>
- **Tessl Feature Adoption:** <value or *Not discussed*>
- **Skills Storage:** <value or *Not discussed*>
- **AI Code Review Tooling:** <value or *Not discussed*>

---

## Key Findings

### Questions Answered: <X> of 20

**Highlights** (top 3–5 noteworthy answers — pick GOOD or RED FLAG assessments first):

- **<Question #>** — <one-line takeaway> · ==<GOOD / CAUTION / RED FLAG>==
- **<Question #>** — <one-line takeaway> · ==<GOOD / CAUTION / RED FLAG>==
- **<Question #>** — <one-line takeaway> · ==<GOOD / CAUTION / RED FLAG>==

### Open Questions (not yet covered)

- <Question #> — <question text>
- <Question #> — <question text>

---

## Feature Requests

<If features were identified, list them. If none, write "No explicit feature requests captured in this call.">

- ==<Urgency>== **<Feature name>** — <context/why they want it> · *"<short verbatim quote>"* [<source>]
- ==<Urgency>== **<Feature name>** — <context/why they want it> · *"<short verbatim quote>"* [<source>]

---

## Security & Governance

<If requirements were identified, list them. If none, write "No security or governance requirements captured in this call.">

- ==<Severity>== **<Requirement>** — <context/why it matters> · *"<short verbatim quote>"* [<source>]
- ==<Severity>== **<Requirement>** — <context/why it matters> · *"<short verbatim quote>"* [<source>]
```

**Formatting rules for the note:**
- Use `==highlight==` for the maturity score band and assessment ratings to make them stand out
- Use `*italic*` for "Not discussed" values to visually distinguish gaps from answers
- Keep evidence summaries to one line — the full detail lives in the spreadsheet
- Highlights section: pick the 3–5 most interesting findings, prioritizing strong signals (GOOD or RED FLAG) over CAUTION
- Open Questions section: list up to 5 unanswered must-ask questions to guide the next call

### Append Behavior

If this is a **follow-up call** (Path B), search for the existing discovery note:

```
mcp__attio__search-notes-by-metadata with:
  filter: { "parent_object": "companies", "parent_record_id": "<company_record_id>" }
```

Look for a note with title containing "Discovery Summary". If found, create a **new note** titled `<Company Name> — Discovery Update (Call <N>)` rather than editing the existing one — this preserves the history of what was known after each call.

---

## Phase 6 -- Report Link

Display a summary to the user:

```
Discovery doc ready!

**Company:** <Company Name>
**Call:** <call_number> (<call_date>)
**Maturity Score:** <total>/24 (<High/Medium/Low> maturity)
**Questions Answered:** <X> of 20
**Questions Remaining:** <Y> of 20
**Feature Requests:** <count> identified
**Security & Governance:** <count> requirements identified

**Technical Environment:**
- SCM: <value or "Not discussed">
- Repo Structure: <value or "Not discussed">
- Primary Languages: <value or "Not discussed">
- Agent Harnesses / Tooling: <value or "Not discussed">
- Vendor Lock-in: <value or "Not discussed">
- Experimentation Attitude: <value or "Not discussed">
- Dedicated AEL Team Size: <value or "Not discussed">
- Overall Team Size: <value or "Not discussed">
- Agent User Roles: <value or "Not discussed">
- SSO Setup: <value or "Not discussed">
- Existing Repo Context: <value or "Not discussed">
- Tessl Feature Adoption: <value or "Not discussed">
- Skills Storage: <value or "Not discussed">
- AI Code Review Tooling: <value or "Not discussed">

**Data Sources:**
- Attio call recordings: <attio_call_count>
- Granola meetings (unique): <granola_unique_count>
- Granola duplicates dropped: <granola_duplicates_dropped>
- Attio CRM record: <Yes/No>
- Attio notes: <count>

Sheet: <web_view_link from upload>
Attio Note: <Created / Skipped (no Attio record)>
```

If this was an append operation, also show:
```
**New answers this call:** <N>
**Updated scores:** <list of changed dimensions>
Attio Note: Created as "Discovery Update (Call <N>)"
```

---

## Constants

- **Discovery folder ID**: `1qXQHbloLqQGHVcLakuMXycLi74eqdVh6`
- **Template spreadsheet**: `https://docs.google.com/spreadsheets/d/1EA2VrQhKgGYeQ7cY0P6NmDX2ucDTKizw/edit?gid=304954826#gid=304954826`
- **docs_manager.rb path**: `/Users/Tessl-Leo/.claude/skills/google-docs/scripts/docs_manager.rb`
- **drive_manager.rb path**: `/Users/Tessl-Leo/.claude/skills/google-docs/scripts/drive_manager.rb`
