# Meridian Logistics — Second Discovery Call Attio Note

## Problem/Feature Description

Meridian Logistics is an existing prospect we've been running a structured discovery process with. After our first call three weeks ago, an account executive posted a discovery summary note to their company record in Attio CRM. That note is now the canonical reference for everything learned in Call 1.

We've just completed a second call with Meridian Logistics that surfaced new information — previously unanswered questions were covered, and one maturity dimension score should be revised upward based on new evidence. Our discovery Google Sheet has been updated with the new data.

We maintain a strict policy of preserving the history of what was known after each call — so we never edit or replace existing notes. Instead, each follow-up call gets its own note. Your job is to produce the correct Attio note content for this second call and save it to `attio_note.md`.

## Output Specification

- `attio_note.md` — the complete note content, including the note title on the first line

## Input Files

The following data is provided. Extract the files before beginning.

=============== FILE: inputs/existing_note.json ===============
{
  "found_in_attio": true,
  "existing_note_title": "Meridian Logistics — Discovery Summary",
  "existing_note_call_number": 1,
  "existing_note_date": "2026-03-18"
}

=============== FILE: inputs/call2_analysis.json ===============
{
  "company": "Meridian Logistics",
  "call_number": 2,
  "call_date": "2026-04-10",
  "attendees": ["Marcus Webb (Meridian, VP Engineering)", "Sophie Laurent (Meridian, Platform Lead)", "Leo Chen (Tessl, AE)"],
  "web_view_link": "https://docs.google.com/spreadsheets/d/1NpQrStUvWxYzAbCdEfGhIJ/edit",
  "maturity": {
    "total_score": 19,
    "band": "Medium",
    "dimensions": [
      { "name": "AI Tooling Landscape", "score": 3, "evidence": "Full Claude Code rollout completed in February, 92% daily active rate [Attio, 2026-04-10]" },
      { "name": "Context Awareness", "score": 2, "evidence": "CLAUDE.md in 70% of repos; no centralized template yet [Attio, 2026-04-10]" },
      { "name": "Internal Knowledge Surface", "score": 2, "evidence": "Engineering wiki exists but 40% stale; Confluence migration in progress [Attio, 2026-04-10]" },
      { "name": "Knowledge Distribution Pain", "score": 3, "evidence": "Call 2 revealed 4 critical knowledge holders with no documented handoff process; Sophie described two near-misses in Q1 [Attio, 2026-04-10]", "previous_score": 2, "changed": true },
      { "name": "Quality Measurement", "score": 2, "evidence": "DORA metrics tracked; piloting CodeRabbit for AI code review since January [Attio, 2026-04-10]" },
      { "name": "Platform / DevEx Ownership", "score": 3, "evidence": "Sophie's team of 10 has explicit charter for AI tooling governance approved by CTO [Attio, 2026-04-10]" },
      { "name": "Scale of Developer Base", "score": 2, "evidence": "210 engineers; targeting 260 by end of year [Attio, 2026-04-10]" },
      { "name": "Contribution Readiness", "score": 2, "evidence": "Engineers contributing skills ad hoc; no review or registry yet [Attio, 2026-04-10]" }
    ]
  },
  "updated_scores": [
    { "dimension": "Knowledge Distribution Pain", "from": 2, "to": 3 }
  ],
  "newly_answered_this_call": ["E1", "E2", "D2"],
  "sections": {
    "highlights": [
      { "question": "E1", "text": "Success defined as 30% reduction in onboarding time for new engineers within 90 days", "assessment": "GOOD" },
      { "question": "E2", "text": "Pilot team: Platform squad (12 engineers) led by Sophie — already using Claude Code daily", "assessment": "GOOD" },
      { "question": "D2", "text": "Marcus is the economic buyer; approved $240K for developer context tooling in FY26", "assessment": "GOOD" },
      { "question": "B3", "text": "No versioning or distribution process for skill updates — skills pushed to a shared Drive folder with no access tracking", "assessment": "RED FLAG" }
    ],
    "open_questions_after_call2": ["B4", "C2", "C4", "F2"]
  },
  "technical_environment": {
    "SCM": "GitLab Cloud",
    "Repo Structure": "Polyrepo (~80 service repos)",
    "Primary Languages": "Java, Python, TypeScript",
    "Agent Harnesses / Tooling": "Claude Code (all engineers), Cursor (25% of engineers)",
    "Vendor Lock-in Concerns": "Prefer provider-agnostic where possible",
    "Experimentation Attitude": "Proactive — Sophie described monthly AI tooling retrospectives",
    "Dedicated AEL Team Size": "10 engineers",
    "Overall Team Size": "210 engineers",
    "Agent User Roles": null,
    "SSO Setup": "Okta (confirmed in Call 1)",
    "Existing Repo Context": "CLAUDE.md in 70% of repos",
    "Tessl Feature Adoption": "Skills (30+ custom), Hooks (3)",
    "Skills Storage": "Google Drive shared folder (unversioned)",
    "AI Code Review Tooling": null
  },
  "data_sources": {
    "attio_call_count": 2,
    "granola_unique_count": 1,
    "granola_duplicates_dropped": 1,
    "attio_crm_record": true,
    "attio_notes_count": 1
  }
}
