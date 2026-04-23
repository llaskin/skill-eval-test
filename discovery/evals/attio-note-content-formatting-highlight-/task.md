# Verano Systems Discovery Call — Attio CRM Note

## Problem/Feature Description

Our sales team relies on Attio CRM to track every customer relationship. After completing a discovery call, the account executive posts a structured note to the company record in Attio so that anyone on the team — including leadership, customer success, and the AE — can review key findings without opening the full spreadsheet.

The full analysis has already been completed for Verano Systems' first discovery call and saved to a Google Sheet. Your job is to produce the Attio note content that the AE will post to the Verano Systems company record. The note must be self-contained and readable inside Attio's note UI, which has some rendering constraints. Save the note content to a file called `attio_note.md`.

## Output Specification

- `attio_note.md` — the complete note content ready to be pasted into Attio's note editor

## Input Files

The following analysis data is provided as input. Extract the files before beginning.

=============== FILE: inputs/analysis.json ===============
{
  "company": "Verano Systems",
  "call_number": 1,
  "call_date": "2026-03-20",
  "attendees": ["Rachel Okonkwo (Verano, CTO)", "James Tate (Verano, Head of Platform)", "Leo Chen (Tessl, AE)", "Maria Santos (Tessl, SE)"],
  "web_view_link": "https://docs.google.com/spreadsheets/d/1Kz9aBcD3efGhIJKlMnOpQrStUvWxYz/edit",
  "maturity": {
    "total_score": 18,
    "band": "Medium",
    "dimensions": [
      { "name": "AI Tooling Landscape", "score": 3, "evidence": "Using Claude Code org-wide, 85% dev adoption, running 40+ custom skills [Attio, 2026-03-20]" },
      { "name": "Context Awareness", "score": 2, "evidence": "Have CLAUDE.md files in 60% of repos, no shared context standards yet [Attio, 2026-03-20]" },
      { "name": "Internal Knowledge Surface", "score": 2, "evidence": "Docs in Confluence and Notion but no centralised context layer [Attio, 2026-03-20]" },
      { "name": "Knowledge Distribution Pain", "score": 3, "evidence": "Lost 3 months velocity when senior architect left; no structured knowledge transfer process [Attio, 2026-03-20]" },
      { "name": "Quality Measurement", "score": 2, "evidence": "Track PR cycle time and defect escape rate; no AI-specific quality metrics [Attio, 2026-03-20]" },
      { "name": "Platform / DevEx Ownership", "score": 2, "evidence": "8-person DevEx team, no exec mandate for AI tooling governance [Attio, 2026-03-20]" },
      { "name": "Scale of Developer Base", "score": 2, "evidence": "150 engineers across 4 teams; growing 20% this year [Attio, 2026-03-20]" },
      { "name": "Contribution Readiness", "score": 2, "evidence": "Engineers write their own ad-hoc skills but no review or publishing workflow [Attio, 2026-03-20]" }
    ]
  },
  "sections": {
    "highlights": [
      { "question": "A2", "text": "Using Claude Code and Cursor across all 150 engineers; 85% daily active usage", "assessment": "GOOD" },
      { "question": "B3", "text": "No process for updating or versioning context standards — Rachel admitted 'we just wing it'", "assessment": "RED FLAG" },
      { "question": "C1", "text": "No AI-specific code quality metrics; they track traditional DORA metrics only", "assessment": "CAUTION" },
      { "question": "D3", "text": "$180K allocated specifically for developer context tooling in FY26 budget", "assessment": "GOOD" }
    ],
    "open_questions": ["B4", "C2", "C4", "E1", "E2", "E3"]
  },
  "technical_environment": {
    "SCM": "GitHub Enterprise",
    "Repo Structure": "Monorepo (main product) + ~30 microservice repos",
    "Primary Languages": "Python, TypeScript, Go",
    "Agent Harnesses / Tooling": "Claude Code (primary), Cursor (secondary)",
    "Vendor Lock-in Concerns": "Comfortable switching models; prefer Anthropic for now",
    "Experimentation Attitude": "Proactive — Rachel says 'we try everything for 2 weeks'",
    "Dedicated AEL Team Size": "8 engineers",
    "Overall Team Size": "150 engineers",
    "Agent User Roles": null,
    "SSO Setup": null,
    "Existing Repo Context": "CLAUDE.md in ~60% of repos; no AGENTS.md",
    "Tessl Feature Adoption": "Skills (40+ custom), Plugins (3), no MCP servers yet",
    "Skills Storage": null,
    "AI Code Review Tooling": null
  },
  "call_checklist": {
    "total_asked": 14,
    "total_not_covered": 6,
    "not_covered": ["B4", "C2", "C4", "E1", "E2", "E3"]
  },
  "data_sources": {
    "attio_call_count": 1,
    "granola_unique_count": 0,
    "granola_duplicates_dropped": 0,
    "attio_crm_record": true,
    "attio_notes_count": 0
  }
}
