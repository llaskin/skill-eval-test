# XLSX Structure Validation

## Context

You have already completed the analysis phase for a discovery call with **Meridian Aerospace**. All maturity scores, section answers, and checklist data are provided below as structured JSON. Your job is to generate a Python script using openpyxl that creates the correctly structured 8-tab xlsx workbook with proper column layouts, styling, and conditional color coding.

## Task

Write a Python script (`build_workbook.py`) that:

1. Takes the pre-analyzed data below as input (embedded in the script or loaded from a JSON file)
2. Creates an xlsx workbook with exactly 8 tabs in the correct order
3. Applies the correct column layouts, header styles, and conditional color fills
4. Saves to `/tmp/Meridian-Aerospace-discovery.xlsx`

### Workbook Structure Requirements

**Tab 1: "Maturity Qualification"**
- Columns: Dimension | Score | Evidence | Notes
- One row per dimension (8 data rows)
- A total score row at the bottom with the sum and maturity band
- Scoring guide rows below the total (explaining 1/2/3 and band thresholds)
- Total score row: green fill (`#C6F6D5`) if High maturity (20-24), yellow fill (`#FEFCBF`) if Medium (14-19), red fill (`#FED7D7`) if Low (8-13)

**Tabs 2-7: "Section A" through "Section F"**
- Columns (11 total): # | Question | Why It Matters | Customer Answer | Good Answer | Caution Answer | Red Flag Answer | Assessment | Follow-Up Used/Needed | Notes | Source
- One row per question in the section
- Assessment column conditional fills: GOOD → `#C6F6D5`, CAUTION → `#FEFCBF`, RED FLAG → `#FED7D7`, N/A → `#D9D9D9`

**Tab 8: "Call Checklist"**
- Columns: Category | Must-Ask Question | Asked? | Key Takeaway
- 20 rows (one per must-ask question)
- Asked? column conditional fills: Yes → `#C6F6D5`, Partial → `#FEFCBF`, No → `#FED7D7`

### Global Styling Requirements

- All header rows: dark slate background `#2D3748`, white bold text
- Data cells: font color `#1A202C`
- All cells: wrap text enabled, top-aligned
- Thin borders on all cells
- Reasonable column widths (Question/Answer/Evidence columns wider than ID/Score columns)

## Expected Output

- `build_workbook.py` — the Python script
- When run, it produces `/tmp/Meridian-Aerospace-discovery.xlsx`

## Pre-Analyzed Input Data

```json
{
  "company": "Meridian Aerospace",
  "call_date": "2026-02-20",
  "call_number": 1,
  "attendees": ["Diana Reeves (Meridian CTO)", "Kenji Watanabe (Meridian Head of Platform)", "Leo (Tessl)"],

  "maturity_scores": [
    {"dimension": "AI Tooling Landscape", "score": 2, "evidence": "Using Copilot with 60% adoption, no other tools evaluated. [Attio, 2026-02-20]", "notes": "Moderate AI tooling presence but no governance or multi-tool strategy"},
    {"dimension": "Context Awareness", "score": 1, "evidence": "Engineers rely entirely on Slack and outdated wiki, no code intelligence tools. [Attio, 2026-02-20]", "notes": "No systematic context-finding, heavy reliance on tribal knowledge"},
    {"dimension": "Internal Knowledge Surface", "score": 1, "evidence": "Wiki has 80% stale content, no ADRs, no knowledge graph. [Attio, 2026-02-20]", "notes": "Knowledge management is essentially absent"},
    {"dimension": "Knowledge Distribution Pain", "score": 3, "evidence": "Lost 3 key engineers in past year, 2 teams still recovering, bus factor of 1 on avionics middleware. [Attio, 2026-02-20]", "notes": "Extreme pain — high fit signal"},
    {"dimension": "Quality Measurement", "score": 2, "evidence": "Track DORA metrics but no AI-specific quality gates, had one escaped defect from Copilot suggestion. [Attio, 2026-02-20]", "notes": "Basic metrics in place but AI quality gap"},
    {"dimension": "Platform / DevEx Ownership", "score": 3, "evidence": "Dedicated 10-person platform team with CTO sponsorship, run quarterly surveys, NPS 38. [Attio, 2026-02-20]", "notes": "Strong platform function, ready to absorb new tooling"},
    {"dimension": "Scale of Developer Base", "score": 2, "evidence": "300 engineers across 5 divisions, planning to grow to 400 by year end. [Attio, 2026-02-20]", "notes": "Good scale, growing — multiplier effect for tooling"},
    {"dimension": "Contribution Readiness", "score": 2, "evidence": "Engineers interested but previous tool rollouts were ad hoc, platform team now gates approvals. [Attio, 2026-02-20]", "notes": "Willingness exists but process needs structure"}
  ],
  "total_score": 16,
  "maturity_band": "Medium",

  "sections": {
    "A": [
      {"id": "A1", "question": "How do your developers currently get context about the codebase when starting a new task?", "why_it_matters": "Reveals current pain points and baseline for improvement", "customer_answer": "Engineers use Slack, git blame, and a largely outdated Confluence wiki. No code intelligence or search tooling. [Attio, 2026-02-20]", "good_answer": "Dedicated tools/processes for context-finding", "caution_answer": "Informal processes, some documentation", "red_flag_answer": "No processes, pure tribal knowledge", "assessment": "CAUTION", "follow_up": "How much time per day do engineers estimate they spend searching for context?", "notes": "Some processes exist but ad hoc", "source": "[Attio, 2026-02-20]"},
      {"id": "A2", "question": "What AI coding tools are currently in use across the organization?", "why_it_matters": "Shows AI maturity and readiness to adopt", "customer_answer": "GitHub Copilot Business rolled out 5 months ago, ~60% weekly active usage. No other tools. [Attio, 2026-02-20]", "good_answer": "Multiple AI tools with governed rollout", "caution_answer": "One tool with moderate adoption", "red_flag_answer": "No AI tools in use", "assessment": "CAUTION", "follow_up": "Are there teams that have opted out of Copilot? Why?", "notes": "Single tool, moderate adoption, no governance", "source": "[Attio, 2026-02-20]"},
      {"id": "A3", "question": "How satisfied are developers with existing context/knowledge tools?", "why_it_matters": "Indicates urgency and willingness to adopt alternatives", "customer_answer": "Developer survey shows context-finding satisfaction at 1.8/5 — lowest category. Engineers describe current state as 'frustrating'. [Attio, 2026-02-20]", "good_answer": "Data-driven satisfaction tracking showing clear gaps", "caution_answer": "Anecdotal complaints but no data", "red_flag_answer": "Team claims everything is fine", "assessment": "GOOD", "follow_up": null, "notes": "Strong quantitative signal of pain", "source": "[Attio, 2026-02-20]"},
      {"id": "A4", "question": "What's the onboarding time for new engineers to become productive?", "why_it_matters": "Long onboarding = high knowledge distribution pain", "customer_answer": "Avionics team: 7 months. Ground systems: 4 months. Internal tools: 6 weeks. Average ~5 months. [Attio, 2026-02-20]", "good_answer": "Specific data with desire to improve", "caution_answer": "Rough estimates, no formal tracking", "red_flag_answer": "Claims onboarding is already fast / not a priority", "assessment": "GOOD", "follow_up": null, "notes": "Specific numbers, clearly a pain point", "source": "[Attio, 2026-02-20]"}
    ],
    "B": [
      {"id": "B1", "question": "Who creates and maintains coding standards and architectural guidelines?", "why_it_matters": "Shows organizational maturity around engineering governance", "customer_answer": "Architecture board of 4 senior engineers. Standards published in GitHub repo but 'loosely followed' — adoption varies by team. [Attio, 2026-02-20]", "good_answer": "Dedicated team/board with enforced standards", "caution_answer": "Standards exist but inconsistently followed", "red_flag_answer": "No formal standards", "assessment": "CAUTION", "follow_up": "What's the biggest barrier to consistent standards adoption?", "notes": "Structure exists but enforcement is weak", "source": "[Attio, 2026-02-20]"},
      {"id": "B2", "question": "How are code review standards enforced today?", "why_it_matters": "Reveals quality gates and potential for AI-assisted review", "customer_answer": "Two approvals required. Basic linting. Review quality inconsistent — senior engineers thorough but bottlenecked, junior reviewers less rigorous. [Attio, 2026-02-20]", "good_answer": "Automated checks + consistent human review", "caution_answer": "Manual review with inconsistent quality", "red_flag_answer": "No formal review process", "assessment": "CAUTION", "follow_up": "What's the average PR review turnaround time?", "notes": "Process exists but quality varies", "source": "[Attio, 2026-02-20]"},
      {"id": "B3", "question": "Is there a formal process for updating and versioning internal engineering standards?", "why_it_matters": "Versioned standards are a prerequisite for AI skills alignment", "customer_answer": "Not discussed.", "good_answer": "Versioned standards with changelog", "caution_answer": "Ad hoc updates announced informally", "red_flag_answer": "No process", "assessment": "N/A", "follow_up": "Need to ask: How do you version and communicate changes to engineering standards?", "notes": "Not covered in this call", "source": "N/A"},
      {"id": "B4", "question": "How do teams share reusable patterns or templates across the organization?", "why_it_matters": "Pattern sharing reduces duplication and improves consistency", "customer_answer": "Shared internal packages exist (~10 libraries) but adoption is optional. No golden path templates. Teams frequently rebuild common patterns. [Attio, 2026-02-20]", "good_answer": "Mandated shared libraries with golden paths", "caution_answer": "Optional shared libraries", "red_flag_answer": "No sharing mechanism", "assessment": "CAUTION", "follow_up": "Which patterns get reinvented most often?", "notes": "Libraries exist but not widely adopted", "source": "[Attio, 2026-02-20]"}
    ],
    "C": [
      {"id": "C1", "question": "How do you measure whether AI-generated code meets your quality bar?", "why_it_matters": "AI quality gates are critical for trust and safety", "customer_answer": "No systematic measurement. One escaped defect from Copilot suggestion in avionics code — caused a 2-day incident. [Attio, 2026-02-20]", "good_answer": "AI-specific quality metrics and gates", "caution_answer": "Rely on existing review process", "red_flag_answer": "No measurement at all", "assessment": "RED FLAG", "follow_up": "What changes did you make after the escaped defect incident?", "notes": "Safety-critical domain with no AI quality gates is a serious risk", "source": "[Attio, 2026-02-20]"},
      {"id": "C2", "question": "What's your process for evaluating new developer tools before org-wide rollout?", "why_it_matters": "Structured eval process predicts successful adoption", "customer_answer": "Platform team runs 4-week pilots with volunteer teams. Security review required. Clear go/no-go criteria. [Attio, 2026-02-20]", "good_answer": "Structured pilot with measurable criteria", "caution_answer": "Informal evaluation", "red_flag_answer": "No evaluation process", "assessment": "GOOD", "follow_up": null, "notes": "Well-structured eval process", "source": "[Attio, 2026-02-20]"},
      {"id": "C3", "question": "Do you have metrics on developer productivity that you track regularly?", "why_it_matters": "Existing metrics enable before/after comparison", "customer_answer": "Track DORA metrics at team level. Deployment frequency, lead time, change failure rate. No SPACE framework yet. [Attio, 2026-02-20]", "good_answer": "Multiple productivity frameworks tracked", "caution_answer": "Basic metrics (DORA only)", "red_flag_answer": "No productivity metrics", "assessment": "CAUTION", "follow_up": "Would you be open to expanding to SPACE framework dimensions?", "notes": "Good foundation but could be broader", "source": "[Attio, 2026-02-20]"},
      {"id": "C4", "question": "How do you identify and address gaps in AI tool effectiveness?", "why_it_matters": "Continuous improvement mindset predicts long-term success", "customer_answer": "Not discussed.", "good_answer": "Instrumented telemetry with feedback loops", "caution_answer": "Survey-based feedback", "red_flag_answer": "No process for identifying gaps", "assessment": "N/A", "follow_up": "Need to ask: How do you currently identify where Copilot is falling short?", "notes": "Not covered in this call", "source": "N/A"}
    ],
    "D": [
      {"id": "D1", "question": "What's the budget approval process for developer tooling?", "why_it_matters": "Understanding procurement path predicts deal velocity", "customer_answer": "CTO approves up to $200K. Above that requires CEO and board. Procurement is straightforward for established categories. [Attio, 2026-02-20]", "good_answer": "Clear budget authority with defined thresholds", "caution_answer": "Budget exists but approval path unclear", "red_flag_answer": "No budget or complex approval chain", "assessment": "GOOD", "follow_up": null, "notes": "Clean approval path with reasonable thresholds", "source": "[Attio, 2026-02-20]"},
      {"id": "D2", "question": "Who are the key decision-makers for platform/tooling investments?", "why_it_matters": "Identifies champion, economic buyer, and blockers", "customer_answer": "Diana Reeves (CTO — economic buyer), Kenji Watanabe (Head of Platform — champion), plus CISO for security review. [Attio, 2026-02-20]", "good_answer": "Clear champion + economic buyer identified", "caution_answer": "Multiple stakeholders with unclear authority", "red_flag_answer": "No clear decision-maker or champion", "assessment": "GOOD", "follow_up": null, "notes": "Both champion and economic buyer in the room", "source": "[Attio, 2026-02-20]"},
      {"id": "D3", "question": "Is there existing budget allocated for AI developer tooling this fiscal year?", "why_it_matters": "Allocated budget dramatically shortens sales cycle", "customer_answer": "General tooling budget exists with ~$150K unallocated. AI tooling is 'a priority line item' but not formally earmarked. [Attio, 2026-02-20]", "good_answer": "Specifically earmarked AI tooling budget", "caution_answer": "General budget that could be allocated", "red_flag_answer": "No budget available", "assessment": "CAUTION", "follow_up": "Can the unallocated $150K be directed to AI tooling with your approval?", "notes": "Budget available but needs to be formally directed", "source": "[Attio, 2026-02-20]"}
    ],
    "E": [
      {"id": "E1", "question": "What would success look like if you solved the context/knowledge problem?", "why_it_matters": "Defines measurable success criteria for POC", "customer_answer": "50% reduction in onboarding time, engineers spending <30min/day on context-finding, zero AI-related safety incidents. [Attio, 2026-02-20]", "good_answer": "Specific measurable outcomes", "caution_answer": "Vague improvement goals", "red_flag_answer": "Can't articulate success criteria", "assessment": "GOOD", "follow_up": null, "notes": "Clear quantified success criteria", "source": "[Attio, 2026-02-20]"},
      {"id": "E2", "question": "Which team or use case would be ideal for a proof of concept?", "why_it_matters": "Right POC team predicts expansion success", "customer_answer": "Ground systems team — 60 engineers, complex domain, enthusiastic tech lead, representative of org challenges. [Attio, 2026-02-20]", "good_answer": "Specific team identified with champion", "caution_answer": "Vague team selection", "red_flag_answer": "No team willing to pilot", "assessment": "GOOD", "follow_up": null, "notes": "Strong POC candidate identified", "source": "[Attio, 2026-02-20]"},
      {"id": "E3", "question": "What's a realistic timeline for evaluating a new tool?", "why_it_matters": "Sets expectations for POC duration and decision point", "customer_answer": "6-8 weeks for pilot plus 2 weeks for security review. Could start in March 2026. [Attio, 2026-02-20]", "good_answer": "Specific timeline with defined start date", "caution_answer": "Vague timeline", "red_flag_answer": "No urgency or indefinite timeline", "assessment": "GOOD", "follow_up": null, "notes": "Concrete timeline with near-term start", "source": "[Attio, 2026-02-20]"}
    ],
    "F": [
      {"id": "F1", "question": "What other solutions have you evaluated or are currently evaluating?", "why_it_matters": "Competitive landscape informs positioning", "customer_answer": "Evaluated Sourcegraph Cody (rejected — context window too small for monorepo). Currently talking to Tabnine (on-prem appeals to CISO). Considered in-house RAG build but lacks bandwidth. [Attio, 2026-02-20]", "good_answer": "Specific alternatives evaluated with clear reasons", "caution_answer": "Vague awareness of alternatives", "red_flag_answer": "Committed to a competitor", "assessment": "GOOD", "follow_up": "Where are you in the Tabnine evaluation process?", "notes": "Competitive but not committed — Tabnine is the main competitor", "source": "[Attio, 2026-02-20]"},
      {"id": "F2", "question": "What would make you choose one solution over another?", "why_it_matters": "Reveals decision criteria and positioning opportunities", "customer_answer": "Safety certification compatibility (aerospace domain), code understanding depth for proprietary frameworks, integration with existing GitHub workflow, and developer adoption speed. [Attio, 2026-02-20]", "good_answer": "Clear criteria aligned with our strengths", "caution_answer": "Price is the only criterion", "red_flag_answer": "Already committed to competitor's approach", "assessment": "GOOD", "follow_up": null, "notes": "Criteria favor deep code understanding — strong for Tessl positioning", "source": "[Attio, 2026-02-20]"}
    ]
  },

  "checklist": [
    {"category": "Context Today", "question_id": "A1", "must_ask_question": "How do your developers currently get context about the codebase when starting a new task?", "asked": "Yes", "key_takeaway": "Engineers rely on Slack, git blame, and outdated Confluence — no systematic tooling"},
    {"category": "Context Today", "question_id": "A2", "must_ask_question": "What AI coding tools are currently in use across the organization?", "asked": "Yes", "key_takeaway": "Copilot Business at 60% adoption, no other tools evaluated"},
    {"category": "Context Today", "question_id": "A3", "must_ask_question": "How satisfied are developers with existing context/knowledge tools?", "asked": "Yes", "key_takeaway": "Context-finding satisfaction is 1.8/5 — lowest survey category"},
    {"category": "Context Today", "question_id": "A4", "must_ask_question": "What's the onboarding time for new engineers to become productive?", "asked": "Yes", "key_takeaway": "Average ~5 months, avionics team as high as 7 months"},
    {"category": "Context Today", "question_id": "A-extra", "must_ask_question": "How large is the engineering organization and how is it structured?", "asked": "Yes", "key_takeaway": "300 engineers in 5 divisions, growing to 400, 10-person platform team"},
    {"category": "Skills Mgmt", "question_id": "B1", "must_ask_question": "Who creates and maintains coding standards and architectural guidelines?", "asked": "Yes", "key_takeaway": "Architecture board of 4 senior engineers, standards loosely followed"},
    {"category": "Skills Mgmt", "question_id": "B2", "must_ask_question": "How are code review standards enforced today?", "asked": "Yes", "key_takeaway": "Two approvals required, basic linting, inconsistent review quality"},
    {"category": "Skills Mgmt", "question_id": "B3", "must_ask_question": "Is there a formal process for updating and versioning internal engineering standards?", "asked": "No", "key_takeaway": "Not covered"},
    {"category": "Skills Mgmt", "question_id": "B4", "must_ask_question": "How do teams share reusable patterns or templates across the organization?", "asked": "Yes", "key_takeaway": "~10 shared libraries exist but adoption is optional, no golden paths"},
    {"category": "Quality & Eval", "question_id": "C1", "must_ask_question": "How do you measure whether AI-generated code meets your quality bar?", "asked": "Yes", "key_takeaway": "No systematic measurement — one escaped defect in avionics caused 2-day incident"},
    {"category": "Quality & Eval", "question_id": "C2", "must_ask_question": "What's your process for evaluating new developer tools before org-wide rollout?", "asked": "Yes", "key_takeaway": "4-week pilot with volunteer teams plus security review"},
    {"category": "Quality & Eval", "question_id": "C3", "must_ask_question": "Do you have metrics on developer productivity that you track regularly?", "asked": "Yes", "key_takeaway": "DORA metrics tracked at team level, no SPACE framework yet"},
    {"category": "Quality & Eval", "question_id": "C4", "must_ask_question": "How do you identify and address gaps in AI tool effectiveness?", "asked": "No", "key_takeaway": "Not covered"},
    {"category": "Decision", "question_id": "D1", "must_ask_question": "What's the budget approval process for developer tooling?", "asked": "Yes", "key_takeaway": "CTO approves up to $200K, above requires CEO and board"},
    {"category": "Decision", "question_id": "D2", "must_ask_question": "Who are the key decision-makers for platform/tooling investments?", "asked": "Yes", "key_takeaway": "CTO (economic buyer), Head of Platform (champion), plus CISO"},
    {"category": "Decision", "question_id": "D3", "must_ask_question": "Is there existing budget allocated for AI developer tooling this fiscal year?", "asked": "Yes", "key_takeaway": "~$150K unallocated in general tooling budget, AI is priority but not earmarked"},
    {"category": "POC Scoping", "question_id": "E1", "must_ask_question": "What would success look like if you solved the context/knowledge problem?", "asked": "Yes", "key_takeaway": "50% onboarding reduction, <30min/day context-finding, zero AI safety incidents"},
    {"category": "POC Scoping", "question_id": "E2", "must_ask_question": "Which team or use case would be ideal for a proof of concept?", "asked": "Yes", "key_takeaway": "Ground systems team — 60 engineers with enthusiastic tech lead"},
    {"category": "POC Scoping", "question_id": "E3", "must_ask_question": "What's a realistic timeline for evaluating a new tool?", "asked": "Yes", "key_takeaway": "6-8 weeks pilot + 2 weeks security review, starting March 2026"},
    {"category": "Competitive", "question_id": "F1", "must_ask_question": "What other solutions have you evaluated or are currently evaluating?", "asked": "Yes", "key_takeaway": "Cody rejected, Tabnine in active talks, in-house RAG considered but no bandwidth"}
  ]
}
```

## Verification Checklist

After generating the script and running it, verify:

1. The workbook has exactly 8 tabs in order: Maturity Qualification, Section A, Section B, Section C, Section D, Section E, Section F, Call Checklist
2. Tab 1 has a yellow-filled total row (score 16 = Medium band)
3. Section tabs have correct conditional fills (C1 RED FLAG = red fill `#FED7D7`)
4. Checklist tab has correct Asked? fills (B3 No = red, C4 No = red, all Yes = green)
5. All headers have dark slate background with white bold text
6. Column widths are reasonable (not all default 8.43)
