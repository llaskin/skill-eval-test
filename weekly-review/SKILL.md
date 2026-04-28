---
name: weekly-review
description: Extracts and compiles a ranked pros-and-cons report from meeting transcripts across Granola and Attio, with full speaker attribution and verbatim quotes. Output is formatted in Slack mrkdwn, ready to paste or post directly into a company-wide Slack channel. Use when the user wants a weekly review, call recap, meeting summary, weekly report, or needs to analyze transcripts for trends, feature requests, bugs, or concerns.
license: MIT
metadata:
  version: 0.1.0
---

# Weekly Review

Analyze this week's meetings and produce a concise weekly review as two short Slack channel messages (What's Working + What Needs Attention) with additional quotes in threads.

---

## Execution Flow

Skill invoked → Phase 1 (Fetch) → Phase 1.5 (Classify Internal/External) → Phase 2 (Present + Auto-Select External) → Phase 3 (Subagent Analysis) → Phase 4 (Aggregate + Report) → Phase 5 (Deliver)

---

## Phase 1 — Fetch Meetings

### Step 1: Load MCP Tools

Use ToolSearch to load required MCP tools. Issue both searches in parallel:

- `ToolSearch` query: `"+granola list"`
- `ToolSearch` query: `"+attio search meetings"`

### Step 2: Fetch Meetings from Both Sources in Parallel

1. **Granola:** `mcp__granola__list_meetings` with `time_range: "this_week"`
2. **Attio:** `mcp__attio__search-meetings` with:
   - `starts_after`: Sunday before current week at `23:59:59Z`
   - `starts_before`: Monday after current week at `00:00:00Z`
   - `timezone`: Detect from system locale; ask user if unknown; fallback `"America/New_York"`
   - `limit`: `100`

Calculate boundary dates dynamically. If one source fails, continue with the other and note the failure.

### Step 3: Deduplicate

Two meetings are duplicates if same date, overlapping start times (within 15 min), and similar title. For duplicates: mark source as `"Both"`, prefer Granola transcript, preserve Attio IDs for fallback.

### Step 4: Build Unified Meeting List

| Field | Description |
|---|---|
| `index` | Sequential number starting at 1 |
| `title` | Meeting title |
| `date` | YYYY-MM-DD |
| `time` | HH:MM local |
| `source` | `"Granola"`, `"Attio"`, or `"Both"` |
| `attendees` | Attendee names |
| `granola_id` | Granola meeting ID (if available) |
| `attio_meeting_id` | Attio meeting ID (if available) |
| `attio_call_recording_ids` | Attio recording IDs (if available) |
| `attendee_emails` | Attendee emails from metadata |
| `customer_name` | Company name of non-`tessl.io` attendees (derive from email domain; capitalize; pick most common if multiple). `"Unknown"` if none. |
| `classification` | `"Internal"` or `"External"` (set in Phase 1.5) |

### Step 5: Exclude Known Internal Meetings

Remove meetings matching the exclusion list before classification:

| Title (substring, case-insensitive) | Day | Time (local) |
|---|---|---|
| `Weekly demo` | Thursday | 13:45–15:00 |

### Step 6: Exclude Meetings Without User

Remove any meeting where `leo@tessl.io` is not in `attendee_emails`. If `attendee_emails` is empty, keep the meeting.

---

## Phase 1.5 — Classify Internal vs External

**Rule:** External if any attendee email domain ≠ `tessl.io`; Internal if all domains are `tessl.io`; default to External if no emails available.

---

## Phase 2 — Present List and Auto-Select External

### Step 1: Present the Meeting List

```
*1. Weekly Sync* — 2026-02-09 10:00 · Both · Internal
  Attendees:
  • Alice
  • Bob

*2. Customer Call - Acme* — 2026-02-10 14:30 · Granola · External · Customer: Acme
  Attendees:
  • Carol
  • Dave
```

Below the list: **"Total: [N] meetings found this week ([X] external, [Y] internal). Analyzing [X] external meetings."**

### Step 2: Auto-Select External Meetings

Automatically select all External meetings. Do not ask the user.

### Step 3: Analysis Focus

Always use: `"Product feedback: feature requests, bugs, product concerns, UX issues, feature praise"`. Do not ask the user.

---

## Phase 3 — Subagent Transcript Analysis

For each External meeting, dispatch a subagent via the `Task` tool. **Launch all subagents in parallel — single message.**

For each meeting, call `Task` with:
- `subagent_type`: `"transcript-analyzer"`
- The prompt from `SUBAGENT_TEMPLATE.md` with all `{{placeholder}}` values replaced
- `{{internal_attendees}}`: comma-separated names whose email domain is `tessl.io`. If no emails available: `"Unknown — treat all speakers with ambiguous affiliation as external"`

The template instructs each subagent to fetch the transcript (Granola primary, Attio fallback), extract positives/negatives **only from external (non-Tessl) speakers**, and return structured JSON: `meeting_title`, `customer_name`, `meeting_date`, `source`, `items[]` (each with `type`, `topic`, `summary`, `speaker`, `timestamps`, `quotes`).

If a subagent fails, mark the meeting as "transcript unavailable" and include it in a "Transcripts Unavailable" section of the final report.

---

## Phase 4 — Aggregate and Report

### Aggregation Rules

1. Collect all `items` arrays into a flat list.
2. Group by topic using fuzzy matching; merge semantically equivalent topics; use the clearest label as canonical.
3. Rank groups by: (1) unique speakers, (2) unique meetings, (3) total mentions.
4. Select top 3 positives and top 3 negatives.
5. Additional positives/negatives beyond top 3: sort by mentions descending, cap at 5 each.

### Report Format — Slack `mrkdwn`

Use Slack mrkdwn syntax: `*bold*`, `_italic_`, `•` bullets (not `>`), emoji shortcodes, no `#` headers, no markdown tables. Build four separate pieces:

#### Piece 1: Channel Message — "What's Working"

```
:white_check_mark: *Weekly Call Review: What's Working* ([Mon date] – [Fri date])

:one: *[Topic Label]*
• "[best quote]" — [Speaker Name], [Customer Name] ([Date])

:two: *[Topic Label]*
• "[best quote]" — [Speaker Name], [Customer Name] ([Date])

:three: *[Topic Label]*
• "[best quote]" — [Speaker Name], [Customer Name] ([Date])

_[N] external calls analyzed · More quotes in thread :thread:_
```

#### Piece 2: Thread Reply on "What's Working"

```
:thread: *More on What's Working*

*[Topic Label]*
• "[quote]" — [Speaker Name], [Customer Name] ([Date])
• "[quote]" — [Speaker Name], [Customer Name] ([Date])

:pushpin: *Also Worth Noting*
• [Topic] ([N] mentions) — [summary]
```

Only include topics with additional quotes beyond the one used in Piece 1. Omit "Also Worth Noting" if no additional positive topics.

#### Piece 3: Channel Message — "What Needs Attention"

```
:warning: *Weekly Call Review: What Needs Attention* ([Mon date] – [Fri date])

:one: *[Topic Label]*
• "[best quote]" — [Speaker Name], [Customer Name] ([Date])

:two: *[Topic Label]*
• "[best quote]" — [Speaker Name], [Customer Name] ([Date])

:three: *[Topic Label]*
• "[best quote]" — [Speaker Name], [Customer Name] ([Date])

_More quotes in thread :thread:_
```

#### Piece 4: Thread Reply on "What Needs Attention"

Same structure as Piece 2 but for negatives.

**Quote formatting rules:**
- `•` bullets, not `>` blockquotes
- Trim to key sentence (max ~120 chars); use `…` for trimming
- Attribution: `— Speaker, Customer Name (Date)` on same line
- Channel messages: 1 quote per topic (most vivid). Thread replies: up to 2 additional quotes per topic.
- Number emojis outside bold: `:one: *Topic Label*`

### Report Format — Markdown Document

Generate a full archival Markdown version combining all four pieces. Uses standard Markdown syntax.

```markdown
# Weekly Call Review: [Mon date] – [Fri date], [year]

*A weekly review of prospect and customer conversations about Tessl — what features resonate, what needs work, and what customers are asking for.*

---

## What's Working

### 1. [Topic Label]

• "[quote]" — [Speaker Name], [Customer Name] ([Date])
• "[quote]" — [Speaker Name], [Customer Name] ([Date])

### Also Worth Noting (Positives)

- **[Topic]** ([N] mentions) — [summary]

---

## What Needs Attention

### 1. [Topic Label]

• "[quote]" — [Speaker Name], [Customer Name] ([Date])

### Also Worth Noting (Needs Attention)

- **[Topic]** ([N] mentions) — [summary]
```

Includes all quotes (channel + thread) per topic, max 3 per topic.

### Edge Cases

- **Fewer than 3 positives/negatives:** Show available items; append `_Only [N] [positive/negative] topics found this week._`
- **No items found:** Display `_No [positive/negative] items found matching "[focus]". Consider broadening your focus or selecting more meetings._`
- **Transcript unavailable:** Append to Piece 1:

```
:no_entry_sign: *Transcripts Unavailable*
• [Customer Name] ([Date], [Time]) — [Source]
```

---

## Phase 5 — Deliver

### Step 1: Write Report Files

Ensure `~/Documents/Weekly Updates/` exists, then write:

1. **Markdown:** `~/Documents/Weekly Updates/weekly-review-[YYYY-MM-DD].md` (Monday date)
2. **Slack:** `~/Documents/Weekly Updates/weekly-review-[YYYY-MM-DD].slack.txt` — four pieces separated by labeled dividers:

```
=== CHANNEL MESSAGE: What's Working ===
[Piece 1]

=== THREAD REPLY: What's Working ===
[Piece 2]

=== CHANNEL MESSAGE: What Needs Attention ===
[Piece 3]

=== THREAD REPLY: What Needs Attention ===
[Piece 4]
```

Tell the user both file paths.

### Step 2: Present Slack Messages

Display each piece in its own labeled fenced code block. Do not ask how to deliver — present all outputs together.

### Step 3: DM Review Draft to User

1. `ToolSearch` queries `"+slack send"` and `"+slack search users"` in parallel.
2. Find **Leo Laskin** via `mcp__claude_ai_Slack__slack_search_users`.
3. Send a single DM with all four pieces (blank lines between), prepended by:
   ```
   :eyes: *Weekly Review Draft — ready for your review*
   Reply here or in Claude Code to request edits, or tell me which channel to post to.
   ```
4. Tell the user: "Sent a preview DM to you in Slack."

### If User Requests Slack Posting

Only if explicitly asked:

1. Ask which channel.
2. `ToolSearch` query `"+slack send"`, then find channel via `mcp__claude_ai_Slack__slack_search_channels`.
3. Post in order:
   a. Piece 1 → channel
   b. Piece 2 → thread reply on (a), using `thread_ts` from (a)'s response
   c. Piece 3 → channel
   d. Piece 4 → thread reply on (c), using `thread_ts` from (c)'s response

Post (a) before (c) so "Working" appears above "Needs Attention".
