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

Use ToolSearch to load required MCP tools. Issue both searches in parallel (single message, two tool calls):

- `ToolSearch` with query: `"+granola list"`
- `ToolSearch` with query: `"+attio search meetings"`

### Step 2: Fetch Meetings from Both Sources in Parallel

Issue both tool calls in a single message:

1. **Granola:** Call `mcp__granola__list_meetings` with `time_range: "this_week"`.
2. **Attio:** Call `mcp__attio__search-meetings` with:
   - `starts_after`: Sunday before current week at `23:59:59Z` (exclusive — e.g., `"2026-02-08T23:59:59Z"`)
   - `starts_before`: Monday after current week at `00:00:00Z` (exclusive — e.g., `"2026-02-16T00:00:00Z"`)
   - `timezone`: Detect from system locale. If unknown, ask user. Fallback: `"America/New_York"`
   - `limit`: `100`

Calculate boundary dates dynamically based on today's date.

**Source Failure Handling:** If one source fails, continue with the other. Note the failure when presenting the meeting list.

### Step 3: Deduplicate

Two meetings are duplicates if ALL match:
- Same date
- Overlapping time (start times within 15 minutes)
- Similar title (case-insensitive substring match)

For duplicates:
- Mark source as `"Both"`
- Prefer Granola transcript (use Granola ID for fetching)
- Preserve Attio meeting_id and call_recording_ids for fallback

### Step 4: Build Unified Meeting List

Track these fields for each unique meeting:

| Field | Description |
|---|---|
| `index` | Sequential number starting at 1 |
| `title` | Meeting title |
| `date` | Date (YYYY-MM-DD) |
| `time` | Start time (HH:MM local) |
| `source` | `"Granola"`, `"Attio"`, or `"Both"` |
| `attendees` | List of attendee names |
| `granola_id` | Granola meeting ID (if available) |
| `attio_meeting_id` | Attio meeting ID (if available) |
| `attio_call_recording_ids` | List of Attio call recording IDs (if available) |
| `attendee_emails` | List of attendee email addresses (collected from Granola/Attio metadata) |
| `customer_name` | Derived from external attendees (set in Phase 1.5). Use the company/organization name of non-`tessl.io` attendees. Derive from: (1) the email domain (e.g., `jane@acme.com` → `"Acme"`), stripping common suffixes like `.com`, `.io`, `.co`. (2) If multiple external domains exist, pick the most common one. (3) Capitalize the first letter. If no external attendees, set to `"Unknown"`. |
| `classification` | `"Internal"` or `"External"` (set in Phase 1.5) |

### Step 5: Exclude Known Internal Meetings

Before classification, remove meetings that match any entry in the exclusion list below. These are recurring internal meetings that may appear External due to non-`tessl.io` attendee emails (e.g., contractors using personal addresses).

**Exclusion List:**

| Title (substring match, case-insensitive) | Day | Time (local) |
|---|---|---|
| `Weekly demo` | Thursday | 13:45–15:00 |

A meeting is excluded if its title contains the substring AND it falls on the matching day and overlapping time window. Excluded meetings should not appear in the presented table or be analyzed.

### Step 6: Exclude Meetings Without User

After deduplication and known-internal exclusions, remove any meeting where `leo@tessl.io` is **not** present in the `attendee_emails` list. Only meetings that include the user as an attendee should proceed to classification and analysis.

If `attendee_emails` is empty (no email metadata available), **keep** the meeting (err on the side of inclusion).

---

## Phase 1.5 — Classify Internal vs External

For each meeting in the unified list, classify it as Internal or External based on attendee email domains.

### Classification Rule

1. Collect all `attendee_emails` for the meeting (from Granola and/or Attio metadata).
2. For each email, extract the domain (the part after `@`).
3. Apply:
   - If **every** attendee email domain is `tessl.io` → set `classification` to `"Internal"`
   - If **any** attendee email domain is NOT `tessl.io` → set `classification` to `"External"`
   - If `attendee_emails` is empty (no email addresses available) → set `classification` to `"External"` (err on the side of inclusion)

---

## Phase 2 — Present List and Auto-Select External

### Step 1: Present the Meeting List

Display as a structured list. For each meeting, show its details with attendees as a bulleted sub-list:

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

For External meetings, include the `customer_name` field after the type.

Below the list: **"Total: [N] meetings found this week ([X] external, [Y] internal). Analyzing [X] external meetings."**

### Step 2: Auto-Select External Meetings

**Do NOT ask the user.** Automatically select all **External** meetings for analysis. Internal meetings are excluded.

### Step 3: Analysis Focus

**Do NOT ask the user.** Always use:

> "Product feedback: feature requests, bugs, product concerns, UX issues, feature praise"

---

## Phase 3 — Subagent Transcript Analysis

For each selected **External** meeting, dispatch a subagent using the `Task` tool. Internal meetings are skipped. **Launch ALL subagents in parallel — issue every Task tool call in a single message.**

### Subagent Configuration

For each meeting, call `Task` with:
- `subagent_type`: `"transcript-analyzer"`
- The prompt from `SUBAGENT_TEMPLATE.md` with all `{{placeholder}}` values replaced with the meeting's actual details
- For `{{internal_attendees}}`: provide a comma-separated list of attendee names whose email domain is `tessl.io`. Derive from the meeting's `attendee_emails`. If no attendee emails are available, pass `"Unknown — treat all speakers with ambiguous affiliation as external"`.

The template instructs each subagent to:
1. Load MCP tools via ToolSearch
2. Fetch transcript (Granola primary, Attio fallback)
3. Extract positives/negatives **only from external (non-Tessl) speakers** — internal speakers are filtered out
4. Return structured JSON: `meeting_title`, `customer_name`, `meeting_date`, `source`, `items[]` (each with `type`, `topic`, `summary`, `speaker`, `timestamps`, `quotes`)

**Handling Failures:** If a subagent fails, note the meeting as "transcript unavailable" and continue. Include in final report under "Transcripts Unavailable" section.

---

## Phase 4 — Aggregate and Report

### Aggregation Rules

1. **Collect** all `items` arrays from subagent results into a single flat list.
2. **Group by topic** using fuzzy matching. Merge semantically equivalent topics (e.g., "CSV export broken" and "Export functionality broken"). Use the clearest label as canonical name.
3. **Rank** topic groups by:
   1. Number of unique speakers (highest first)
   2. Number of unique meetings (highest first)
   3. Total mentions (highest first)
4. **Select top 3 positives** and **top 3 negatives**.
5. **Additional positives:** Remaining positive topics beyond top 3, sorted by total mentions descending, cap at 5.
6. **Additional negatives:** Remaining negative topics beyond top 3, sorted by total mentions descending, cap at 5.

### Report Format — Slack `mrkdwn`

The report MUST use Slack's `mrkdwn` syntax — NOT standard markdown. Key differences:
- Bold: `*text*` (single asterisks, not double)
- Italic: `_text_`
- No `#` headers — use `*bold text*` on its own line as a section header
- No markdown tables — use structured lists
- Bullets: `•` for quoted text (NOT `>` blockquotes)
- Emoji shortcodes (e.g., `:white_check_mark:`, `:warning:`) for visual scanning
- Dividers: use a blank line, not `---`

**The report is NOT one message.** Build four separate pieces of Slack content:

#### Piece 1: Channel Message — "What's Working"

Short and scannable. One best quote per topic. People should be able to read this in 15 seconds.

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

Additional quotes and "also worth noting" positives. Posted as a reply to Piece 1.

```
:thread: *More on What's Working*

*[Topic Label]*
• "[quote]" — [Speaker Name], [Customer Name] ([Date])
• "[quote]" — [Speaker Name], [Customer Name] ([Date])

*[Topic Label]*
• "[quote]" — [Speaker Name], [Customer Name] ([Date])

:pushpin: *Also Worth Noting*
• [Topic] ([N] mentions) — [summary]
• [Topic] ([N] mentions) — [summary]
```

Only include topics that have additional quotes beyond the one used in the channel message. Skip topics with no extra quotes. Omit "Also Worth Noting" if no additional positive topics exist.

#### Piece 3: Channel Message — "What Needs Attention"

Same brevity as Piece 1. One best quote per topic.

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

Additional quotes and "also worth noting" negatives. Posted as a reply to Piece 3.

```
:thread: *More on What Needs Attention*

*[Topic Label]*
• "[quote]" — [Speaker Name], [Customer Name] ([Date])
• "[quote]" — [Speaker Name], [Customer Name] ([Date])

*[Topic Label]*
• "[quote]" — [Speaker Name], [Customer Name] ([Date])

:pushpin: *Also Worth Noting*
• [Topic] ([N] mentions) — [summary]
• [Topic] ([N] mentions) — [summary]
```

Same rules as Piece 2 — only include topics with extra quotes, omit "Also Worth Noting" if empty.

**Formatting rules for quotes:**
- Use `•` bullet points for quotes — NOT `>` blockquotes.
- Keep quotes concise — trim to the key sentence (max ~120 chars). Use `…` to indicate trimming.
- Attribution goes after the quote on the same line with an em dash: `— Speaker, Customer Name (Date)`
- Channel messages: exactly 1 quote per topic (the most vivid/representative).
- Thread replies: up to 2 additional quotes per topic.
- Use number emojis (`:one:`, `:two:`, `:three:`) for topic numbering in channel messages. Place the emoji *outside* bold markers — e.g., `:one: *Topic Label*` not `*:one: Topic Label*`.
- Thread replies use plain `*bold*` topic labels without number emojis.

### Report Format — Markdown Document

In addition to the Slack pieces, generate a standard Markdown version containing ALL content (channel messages + thread content combined). This is the archival version. Uses proper Markdown syntax (not Slack mrkdwn).

```markdown
# Weekly Call Review: [Mon date] – [Fri date], [year]

*A weekly review of prospect and customer conversations about Tessl — what features resonate, what needs work, and what customers are asking for.*

---

## What's Working

### 1. [Topic Label]

• "[quote]" — [Speaker Name], [Customer Name] ([Date])
• "[quote]" — [Speaker Name], [Customer Name] ([Date])

### 2. [Topic Label]

• "[quote]" — [Speaker Name], [Customer Name] ([Date])

### 3. [Topic Label]

• "[quote]" — [Speaker Name], [Customer Name] ([Date])

### Also Worth Noting (Positives)

- **[Topic]** ([N] mentions) — [summary]

---

## What Needs Attention

### 1. [Topic Label]

• "[quote]" — [Speaker Name], [Customer Name] ([Date])
• "[quote]" — [Speaker Name], [Customer Name] ([Date])

### 2. [Topic Label]

• "[quote]" — [Speaker Name], [Customer Name] ([Date])

### 3. [Topic Label]

• "[quote]" — [Speaker Name], [Customer Name] ([Date])

### Also Worth Noting (Needs Attention)

- **[Topic]** ([N] mentions) — [summary]
```

Apply the same formatting rules for quotes (concise, max ~120 chars, max 3 per topic) and the same edge case handling as the Slack format, but using standard Markdown syntax. The Markdown version includes ALL quotes (channel + thread content) for each topic.

### Edge Cases

- **Fewer than 3 positives or negatives:** Show available items. Append: `_Only [N] [positive/negative] topics found this week._`
- **No items found for focus:** Display: `_No [positive/negative] items found matching "[focus]". Consider broadening your focus or selecting more meetings._`
- **Transcript unavailable:** Append to the "What's Working" channel message (Piece 1):

```
:no_entry_sign: *Transcripts Unavailable*
• [Customer Name] ([Date], [Time]) — [Source]
```

---

## Phase 5 — Deliver

### Step 1: Create Output Directory

Ensure the output directory exists: `~/Documents/Weekly Updates/`

### Step 2: Write Report Files

Write report files using the `Write` tool. Use the Monday date of the review week in filenames.

1. **Markdown version** (full archival report):
   - **Path:** `~/Documents/Weekly Updates/weekly-review-[YYYY-MM-DD].md`
   - Example: `~/Documents/Weekly Updates/weekly-review-2026-02-09.md`

2. **Slack version** (all four pieces, separated by dividers):
   - **Path:** `~/Documents/Weekly Updates/weekly-review-[YYYY-MM-DD].slack.txt`
   - Format: each piece separated by a line of `========` with a label:
     ```
     === CHANNEL MESSAGE: What's Working ===
     [Piece 1 content]

     === THREAD REPLY: What's Working ===
     [Piece 2 content]

     === CHANNEL MESSAGE: What Needs Attention ===
     [Piece 3 content]

     === THREAD REPLY: What Needs Attention ===
     [Piece 4 content]
     ```

Tell the user both file paths after writing.

### Step 3: Present the Slack Messages

Display each Slack piece in its own fenced code block, clearly labeled:

````
Here's your weekly review, formatted for Slack (two channel messages + thread replies):

**Channel Message 1 — What's Working:**
```
[Piece 1]
```

**Thread reply on Message 1:**
```
[Piece 2]
```

**Channel Message 2 — What Needs Attention:**
```
[Piece 3]
```

**Thread reply on Message 2:**
```
[Piece 4]
```
````

**Do NOT ask the user how to deliver.** Present all outputs (file paths + Slack pieces) together. The user can then ask to post to Slack or make edits if they want.

### Step 4: DM Review Draft to User

After presenting the report locally, send all four pieces as a DM to **Leo Laskin** in Slack so the user can review the formatted output on mobile/desktop before deciding to post.

1. Use `ToolSearch` with query: `"+slack send"` and `"+slack search users"` to load tools.
2. Search for the user using `mcp__claude_ai_Slack__slack_search_users` with query `"Leo Laskin"`.
3. Send all four pieces as **a single DM** — concatenate Pieces 1–4 separated by blank lines. Prepend:
   ```
   :eyes: *Weekly Review Draft — ready for your review*
   Reply here or in Claude Code to request edits, or tell me which channel to post to.

   ```
4. Tell the user: "Sent a preview DM to you in Slack."

### If User Requests Slack Posting

Only if the user explicitly asks to post to Slack:

1. Ask which channel (e.g., `#general`, `#team-updates`).
2. Use `ToolSearch` with query: `"+slack send"` to load the Slack send tool.
3. Search for the channel using `mcp__claude_ai_Slack__slack_search_channels` with the user's channel name.
4. **Post in this order** (four messages total):
   a. Send Piece 1 ("What's Working" channel message) to the channel.
   b. Send Piece 2 ("What's Working" thread reply) as a **thread reply** to message (a). Use the `thread_ts` from the response of step (a).
   c. Send Piece 3 ("What Needs Attention" channel message) to the channel.
   d. Send Piece 4 ("What Needs Attention" thread reply) as a **thread reply** to message (c). Use the `thread_ts` from the response of step (c).

Steps (a) then (b) must be sequential (b depends on a's thread_ts). Steps (c) then (d) must be sequential. But (a+b) and (c+d) are independent — however, post (a) before (c) so "Working" appears above "Needs Attention" in the channel.

**Thread reply mechanics:** When calling `mcp__claude_ai_Slack__slack_send_message`, include the `thread_ts` parameter set to the `ts` value returned from the parent channel message. This posts the reply inside the thread rather than as a new channel message.