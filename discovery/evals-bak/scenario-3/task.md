# Source Deduplication

## Context

You are building the data-fetching phase of the discovery call processor. Two data sources — Attio (primary) and Granola (secondary) — have returned call records for the customer **Quantum Dynamics Corp**. Some calls appear in both systems and must be deduplicated. Attio is always the preferred source when duplicates are found.

## Task

Write a script (Python or JavaScript) that takes the two call record lists below as input and produces a single deduplicated `merged_calls` list. The script must implement the skill's three-condition deduplication logic:

### Deduplication Rules (in priority order)

1. **Same date + overlapping attendee email domain** → DUPLICATE — discard the Granola version, keep Attio
2. **Same date + similar title** (shared keyword: "discovery", "intro", "follow-up", "review", "demo") → LIKELY DUPLICATE — discard the Granola version, keep Attio
3. **No match on either condition** → UNIQUE to Granola — keep it

### Output Requirements

For each call in the merged list, include:
- `call_id`: the Attio recording_id or Granola meeting_id
- `source`: "attio" or "granola"
- `title`: call title
- `date`: call date (YYYY-MM-DD)
- `attendees`: union of attendees from both sources if duplicate, otherwise the source's attendees
- `attio_recording_id`: Attio ID or null
- `granola_meeting_id`: Granola ID or null
- `duplicate_detected`: boolean, true if a Granola call was discarded in favor of this Attio call

Sort the merged list chronologically (oldest first).

Also output a deduplication log:
```json
{
  "attio_call_count": <N>,
  "granola_total_count": <N>,
  "granola_unique_count": <N>,
  "granola_duplicates_dropped": <N>,
  "merged_total": <N>
}
```

## Expected Outputs

- `deduplicate_calls.py` (or `.js`) — the deduplication script
- `merged_calls.json` — the unified call list
- `dedup_log.json` — the deduplication summary

## Input Files

=============== FILE: inputs/attio_calls.json ===============
[
  {
    "recording_id": "att-rec-001",
    "title": "Quantum Dynamics - Discovery Call",
    "date": "2026-01-15",
    "attendees": [
      {"name": "Wei Zhang", "email": "wei.zhang@quantumdyn.com"},
      {"name": "Leo Bravo", "email": "leo@tessl.io"}
    ],
    "transcript": "Full transcript from Attio for the initial discovery call..."
  },
  {
    "recording_id": "att-rec-002",
    "title": "Quantum Dynamics - Technical Deep Dive",
    "date": "2026-01-22",
    "attendees": [
      {"name": "Wei Zhang", "email": "wei.zhang@quantumdyn.com"},
      {"name": "Raj Patel", "email": "raj.patel@quantumdyn.com"},
      {"name": "Leo Bravo", "email": "leo@tessl.io"},
      {"name": "Sophie Martin", "email": "sophie@tessl.io"}
    ],
    "transcript": "Full transcript from Attio for the technical deep dive..."
  },
  {
    "recording_id": "att-rec-003",
    "title": "Quantum Dynamics - Follow-up on POC",
    "date": "2026-02-05",
    "attendees": [
      {"name": "Wei Zhang", "email": "wei.zhang@quantumdyn.com"},
      {"name": "Leo Bravo", "email": "leo@tessl.io"}
    ],
    "transcript": "Full transcript from Attio for the POC follow-up..."
  },
  {
    "recording_id": "att-rec-004",
    "title": "Quantum Dynamics - Budget Review",
    "date": "2026-02-12",
    "attendees": [
      {"name": "Karen Liu", "email": "karen.liu@quantumdyn.com"},
      {"name": "Wei Zhang", "email": "wei.zhang@quantumdyn.com"},
      {"name": "Leo Bravo", "email": "leo@tessl.io"}
    ],
    "transcript": "Full transcript from Attio for budget review..."
  }
]

=============== FILE: inputs/granola_calls.json ===============
[
  {
    "meeting_id": "grn-mtg-101",
    "title": "Discovery call - Quantum Dynamics",
    "date": "2026-01-15",
    "attendees": [
      {"name": "Wei Zhang", "email": "w.zhang@quantumdyn.com"},
      {"name": "Leo B", "email": "leo@tessl.io"}
    ],
    "transcript": "Granola transcript of the same discovery call with slightly different formatting..."
  },
  {
    "meeting_id": "grn-mtg-102",
    "title": "Quantum Dynamics Engineering Review",
    "date": "2026-01-29",
    "attendees": [
      {"name": "Raj Patel", "email": "raj.patel@quantumdyn.com"},
      {"name": "Leo Bravo", "email": "leo@tessl.io"}
    ],
    "transcript": "Granola transcript of an engineering review that only exists in Granola..."
  },
  {
    "meeting_id": "grn-mtg-103",
    "title": "QD follow-up discussion",
    "date": "2026-02-05",
    "attendees": [
      {"name": "Wei Zhang", "email": "wei.zhang@quantumdyn.com"},
      {"name": "Leo Bravo", "email": "leo@tessl.io"}
    ],
    "transcript": "Granola transcript of the same POC follow-up call..."
  },
  {
    "meeting_id": "grn-mtg-104",
    "title": "Internal prep - Quantum Dynamics pricing",
    "date": "2026-02-12",
    "attendees": [
      {"name": "Leo Bravo", "email": "leo@tessl.io"},
      {"name": "Sophie Martin", "email": "sophie@tessl.io"}
    ],
    "transcript": "Internal Tessl meeting to prepare pricing proposal for Quantum Dynamics..."
  },
  {
    "meeting_id": "grn-mtg-105",
    "title": "Quantum Dynamics - Demo walkthrough",
    "date": "2026-02-19",
    "attendees": [
      {"name": "Wei Zhang", "email": "wei.zhang@quantumdyn.com"},
      {"name": "Raj Patel", "email": "raj.patel@quantumdyn.com"},
      {"name": "Leo Bravo", "email": "leo@tessl.io"}
    ],
    "transcript": "Granola transcript of a product demo that only exists in Granola..."
  }
]

## Key Test Points

The input data contains the following deduplication scenarios that your script must handle correctly:

- **grn-mtg-101** (Jan 15): Same date as att-rec-001, overlapping attendee domain (quantumdyn.com), title contains "discovery" → DUPLICATE, drop Granola
- **grn-mtg-102** (Jan 29): No Attio call on this date → UNIQUE, keep Granola
- **grn-mtg-103** (Feb 5): Same date as att-rec-003, overlapping attendee domain (quantumdyn.com), title contains "follow-up" → DUPLICATE, drop Granola
- **grn-mtg-104** (Feb 12): Same date as att-rec-004, but NO overlapping attendee domain (Granola attendees are all @tessl.io, Attio has @quantumdyn.com attendees) and title does NOT share a keyword with "Budget Review" → UNIQUE, keep Granola (this is an internal prep meeting, not the same call)
- **grn-mtg-105** (Feb 19): No Attio call on this date → UNIQUE, keep Granola

Expected merged list should have **7 calls**: 4 Attio + 3 Granola-unique, sorted by date.
