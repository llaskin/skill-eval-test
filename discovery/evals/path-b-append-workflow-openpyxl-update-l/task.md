# Update Discovery Doc for Orbital Software — Follow-up Call

## Problem/Feature Description

Your company runs structured discovery processes with enterprise prospects. A first discovery call with Orbital Software was completed three weeks ago, and the discovery workbook from that session has several open questions marked as "Not discussed in this call." A follow-up call was held on 2026-04-14 and covered several of those open items — plus surfaced stronger signals on a key dimension that may warrant updating its score.

The workbook file already exists from the first call. Your task is to update it with the new information from the follow-up call, then save the revised file. You must preserve all the original content that was not addressed in the new call.

## Output Specification

Produce the following files:

1. `update_discovery.py` — A Python script that loads the existing workbook and applies all necessary updates from the follow-up call.
2. `Orbital-Software-discovery-updated.xlsx` — The updated workbook file produced by running the update script.

## Input Files

The following files are provided as inputs. Extract them before beginning.

=============== FILE: inputs/generate_base.py ===============
#!/usr/bin/env python3
"""Generates the base discovery xlsx from Call 1 with Orbital Software."""
import openpyxl
from openpyxl.styles import PatternFill, Font, Alignment, Border, Side

wb = openpyxl.Workbook()

HEADER_FILL = PatternFill("solid", fgColor="2D3748")
HEADER_FONT = Font(bold=True, color="FFFFFF")
DATA_FONT   = Font(color="1A202C")
GOOD_FILL   = PatternFill("solid", fgColor="C6F6D5")
CAUTION_FILL= PatternFill("solid", fgColor="FEFCBF")
RED_FILL    = PatternFill("solid", fgColor="FED7D7")
NA_FILL     = PatternFill("solid", fgColor="D9D9D9")
YES_FILL    = GOOD_FILL
NO_FILL     = RED_FILL
PARTIAL_FILL= CAUTION_FILL

def style_header(row_cells):
    for cell in row_cells:
        cell.fill = HEADER_FILL
        cell.font = HEADER_FONT

def style_data(row_cells):
    for cell in row_cells:
        cell.font = DATA_FONT

# ---------- Tab 1: Maturity Qualification ----------
ws_mq = wb.active
ws_mq.title = "Maturity Qualification"

mq_headers = ["Dimension", "Score", "Evidence", "Notes"]
ws_mq.append(mq_headers)
style_header(ws_mq[1])

dimensions = [
    ("AI Tooling Landscape",        2, "[Attio, 2026-03-22] 'We have Copilot but most engineers don't use it consistently'", ""),
    ("Context Awareness",            2, "[Attio, 2026-03-22] 'Engineers spend about 30% of onboarding just learning the codebase'", ""),
    ("Internal Knowledge Surface",   2, "[Attio, 2026-03-22] 'We use Confluence but it gets stale fast'", ""),
    ("Knowledge Distribution Pain",  2, "[Attio, 2026-03-22] 'We lost a senior engineer last quarter and it slowed the team'", "Moderate signal — revisit"),
    ("Quality Measurement",          2, "[Attio, 2026-03-22] 'No formal AI code quality metrics yet'", ""),
    ("Platform / DevEx Ownership",   2, "[Attio, 2026-03-22] 'Small platform team, 4 engineers'", ""),
    ("Scale of Developer Base",      1, "[Attio, 2026-03-22] '~80 engineers total'", ""),
    ("Contribution Readiness",       2, "[Attio, 2026-03-22] 'Engineers open to new tooling'", ""),
]

for dim in dimensions:
    row = ws_mq.max_row + 1
    ws_mq.append(list(dim))
    style_data(ws_mq[row])

# Total row
total_score = sum(d[1] for d in dimensions)
total_row = ws_mq.max_row + 1
ws_mq.append(["TOTAL", total_score, "", "Medium maturity (14-19)"])
for cell in ws_mq[total_row]:
    cell.fill = CAUTION_FILL
    cell.font = Font(bold=True, color="1A202C")

# Scoring guide
ws_mq.append(["Scoring Guide: 3=High (proceed), 2=Medium (proceed with caution), 1=Low (consider qualifying out)", "", "", ""])
ws_mq.append(["Score Bands: 20-24 High | 14-19 Medium | 8-13 Low", "", "", ""])

# ---------- Tab 2: Section A ----------
ws_a = wb.create_sheet("Section A")
a_headers = ["#", "Question", "Why It Matters", "Customer Answer", "Good Answer", "Caution Answer", "Red Flag Answer", "Assessment", "Follow-Up Used/Needed", "Notes", "Source"]
ws_a.append(a_headers)
style_header(ws_a[1])

section_a = [
    ("A1", "How do engineers currently find context about unfamiliar parts of the codebase?",
     "Reveals how painful context-finding is today",
     "[Attio, 2026-03-22] 'Mostly grep the repo and ask a senior engineer. We have some Confluence docs but engineers don't trust them.'",
     "Rich automated context (AGENTS.md, RAG, etc.)",
     "Partial docs, ad hoc searches",
     "No documentation; everything lives in senior engineer heads",
     "CAUTION", "", "Classic knowledge silo scenario", "[Attio, 2026-03-22]"),
    ("A2", "Which AI coding tools are engineers using today?",
     "Establishes baseline adoption and openness",
     "[Attio, 2026-03-22] 'We have GitHub Copilot on enterprise license. Low adoption — maybe 30% of engineers use it regularly.'",
     "Multiple tools, high engagement",
     "One tool, low adoption",
     "None in use; no interest",
     "CAUTION", "", "", "[Attio, 2026-03-22]"),
    ("A3", "How satisfied are engineers with their current development experience?",
     "Measures pain level and urgency to change",
     "[Attio, 2026-03-22] 'We did a quick survey — about 6/10 on average. Main complaints: slow onboarding and knowledge silos.'",
     "Strong dissatisfaction (high urgency to improve)",
     "Moderate dissatisfaction",
     "Satisfied — no pressing pain",
     "GOOD", "", "", "[Attio, 2026-03-22]"),
    ("A4", "How long does it take a new engineer to become productive?",
     "Quantifies knowledge transfer pain",
     "[Attio, 2026-03-22] 'About 4-5 months before they can own a feature end-to-end. The first 6 weeks are especially rough.'",
     "4+ months onboarding time",
     "2-3 months",
     "Under 1 month",
     "GOOD", "Follow up: what % of that is codebase context vs. process?", "", "[Attio, 2026-03-22]"),
]

for row_data in section_a:
    row = ws_a.max_row + 1
    ws_a.append(list(row_data))
    style_data(ws_a[row])
    assessment = row_data[7]
    if assessment == "GOOD":     ws_a.cell(row, 8).fill = GOOD_FILL
    elif assessment == "CAUTION": ws_a.cell(row, 8).fill = CAUTION_FILL
    elif assessment == "RED FLAG": ws_a.cell(row, 8).fill = RED_FILL
    else:                         ws_a.cell(row, 8).fill = NA_FILL

# ---------- Tab 3: Section B ----------
ws_b = wb.create_sheet("Section B")
ws_b.append(a_headers)
style_header(ws_b[1])

section_b = [
    ("B1", "What does the process for creating a new skill or agent look like today?",
     "Reveals maturity of skills creation workflow",
     "[Attio, 2026-03-22] 'We don't have a formal process. Engineers write ad hoc scripts and share them informally.'",
     "Formal creation workflow with templates",
     "Informal but some tooling",
     "No process at all",
     "CAUTION", "", "", "[Attio, 2026-03-22]"),
    ("B2", "Who is responsible for maintaining shared agent configurations?",
     "Identifies ownership and governance",
     "[Attio, 2026-03-22] 'Right now it falls on whoever set it up. The platform team is trying to take ownership but bandwidth is tight.'",
     "Platform team with clear ownership",
     "Shared but unclear ownership",
     "Nobody — completely ad hoc",
     "CAUTION", "", "", "[Attio, 2026-03-22]"),
    ("B3", "Is there a process for versioning and updating shared coding standards?",
     "Tests governance maturity",
     "[Attio, 2026-03-22] 'Not really. We have a Google Doc that someone updates occasionally but there's no versioning or review process.'",
     "Formal versioning with change tracking",
     "Informal doc with occasional updates",
     "No central standards at all",
     "CAUTION", "Ask about adoption tracking in follow-up", "", "[Attio, 2026-03-22]"),
    ("B4", "How do teams share proven patterns or reusable templates today?",
     "Reveals knowledge distribution mechanism",
     "Not discussed in this call",
     "Shared registry with discovery mechanism",
     "Informal Slack sharing",
     "No sharing; each team reinvents",
     "N/A", "Ask in follow-up", "", ""),
]

for row_data in section_b:
    row = ws_b.max_row + 1
    ws_b.append(list(row_data))
    style_data(ws_b[row])
    assessment = row_data[7]
    if assessment == "GOOD":     ws_b.cell(row, 8).fill = GOOD_FILL
    elif assessment == "CAUTION": ws_b.cell(row, 8).fill = CAUTION_FILL
    elif assessment == "RED FLAG": ws_b.cell(row, 8).fill = RED_FILL
    else:                         ws_b.cell(row, 8).fill = NA_FILL

# ---------- Tab 4: Section C (abbreviated) ----------
ws_c = wb.create_sheet("Section C")
ws_c.append(a_headers)
style_header(ws_c[1])

section_c = [
    ("C1", "How do you measure the quality of AI-generated code?",
     "Critical gap for AI code review and governance",
     "[Attio, 2026-03-22] 'We don't have specific AI code quality metrics. We rely on standard PR review.'",
     "Automated AI code quality gates",
     "Manual review only",
     "No review process at all",
     "CAUTION", "", "", "[Attio, 2026-03-22]"),
    ("C2", "What is your process for evaluating new AI tools before adoption?",
     "Reveals experimentation governance",
     "[Attio, 2026-03-22] 'We do informal pilots — the platform team picks a tool, runs it for a few weeks, and reports back.'",
     "Structured evaluation framework",
     "Ad hoc pilots",
     "No evaluation — just adopt and see",
     "CAUTION", "", "", "[Attio, 2026-03-22]"),
    ("C3", "How do you track which engineers are using AI tools vs not?",
     "Measures adoption visibility",
     "[Attio, 2026-03-22] 'Honestly, we don't have great visibility. GitHub Copilot gives us some usage stats but it's not granular.'",
     "Real-time dashboards with per-engineer metrics",
     "Aggregate stats, limited visibility",
     "No tracking at all",
     "CAUTION", "", "", "[Attio, 2026-03-22]"),
    ("C4", "How do you identify gaps in AI tooling coverage?",
     "Tests proactive vs reactive approach",
     "[Attio, 2026-03-22] 'Mostly reactive — engineers raise issues when something is painful enough. No proactive gap analysis.'",
     "Quarterly gap analysis with structured survey",
     "Reactive feedback loop",
     "No awareness of gaps",
     "CAUTION", "", "", "[Attio, 2026-03-22]"),
]

for row_data in section_c:
    row = ws_c.max_row + 1
    ws_c.append(list(row_data))
    style_data(ws_c[row])
    assessment = row_data[7]
    if assessment == "GOOD":     ws_c.cell(row, 8).fill = GOOD_FILL
    elif assessment == "CAUTION": ws_c.cell(row, 8).fill = CAUTION_FILL
    elif assessment == "RED FLAG": ws_c.cell(row, 8).fill = RED_FILL
    else:                         ws_c.cell(row, 8).fill = NA_FILL

# ---------- Tab 5: Section D ----------
ws_d = wb.create_sheet("Section D")
ws_d.append(a_headers)
style_header(ws_d[1])

section_d = [
    ("D1", "Who makes the final decision on adopting a new platform tool?",
     "Identifies the economic buyer and decision process",
     "Not discussed in this call",
     "Named executive sponsor with approval authority",
     "Committee decision with unclear ownership",
     "No clear decision maker",
     "N/A", "Ask in follow-up", "", ""),
    ("D2", "What is the timeline for a decision on this evaluation?",
     "Establishes urgency and deal velocity",
     "[Attio, 2026-03-22] 'We're targeting Q3. Budget was approved in February but we're still in evaluation mode.'",
     "Clear deadline with signed-off budget",
     "Approximate quarter target",
     "No timeline — open-ended",
     "GOOD", "", "", "[Attio, 2026-03-22]"),
    ("D3", "Has budget been allocated for this type of tooling?",
     "Validates financial qualification",
     "[Attio, 2026-03-22] 'Yes — we have $150K earmarked for developer productivity tools this fiscal year. About $90K is still available.'",
     "Allocated budget with specific amount",
     "Informal expectation of budget",
     "No budget; would need to build a business case",
     "GOOD", "", "", "[Attio, 2026-03-22]"),
]

for row_data in section_d:
    row = ws_d.max_row + 1
    ws_d.append(list(row_data))
    style_data(ws_d[row])
    assessment = row_data[7]
    if assessment == "GOOD":     ws_d.cell(row, 8).fill = GOOD_FILL
    elif assessment == "CAUTION": ws_d.cell(row, 8).fill = CAUTION_FILL
    elif assessment == "RED FLAG": ws_d.cell(row, 8).fill = RED_FILL
    else:                         ws_d.cell(row, 8).fill = NA_FILL

# ---------- Tab 6: Section F ----------
ws_f = wb.create_sheet("Section F")
ws_f.append(a_headers)
style_header(ws_f[1])

section_f = [
    ("F1", "What other solutions are you evaluating?",
     "Identifies competitive landscape",
     "[Attio, 2026-03-22] 'We looked at Sourcegraph Cody briefly and decided it was too complex to set up. We're talking to one other vendor but I can't say who yet.'",
     "Evaluated and rejected alternatives; Tessl is primary focus",
     "Multiple active evaluations",
     "Satisfied with current state; no urgency",
     "GOOD", "Ask about the unnamed vendor in follow-up", "", "[Attio, 2026-03-22]"),
    ("F2", "What would have to be true for you to walk away and build in-house?",
     "Reveals build vs buy threshold",
     "[Attio, 2026-03-22] 'If the price is way over our budget or if the integration story with GitHub Enterprise is complicated, we'd consider it.'",
     "High bar to build in-house; clear integration requirements stated",
     "Some in-house capability already exists",
     "Actively planning in-house solution",
     "GOOD", "", "", "[Attio, 2026-03-22]"),
]

for row_data in section_f:
    row = ws_f.max_row + 1
    ws_f.append(list(row_data))
    style_data(ws_f[row])
    assessment = row_data[7]
    if assessment == "GOOD":     ws_f.cell(row, 8).fill = GOOD_FILL
    elif assessment == "CAUTION": ws_f.cell(row, 8).fill = CAUTION_FILL
    elif assessment == "RED FLAG": ws_f.cell(row, 8).fill = RED_FILL
    else:                         ws_f.cell(row, 8).fill = NA_FILL

# ---------- Tab 7: Call Checklist ----------
ws_cl = wb.create_sheet("Call Checklist")
cl_headers = ["Category", "Must-Ask Question", "Asked?", "Key Takeaway"]
ws_cl.append(cl_headers)
style_header(ws_cl[1])

checklist = [
    ("Context Today",  "How do engineers find context in the codebase?",            "Yes",     "Mostly grep + ask senior engineers; Confluence not trusted"),
    ("Context Today",  "What AI coding tools are in use?",                           "Yes",     "GitHub Copilot enterprise, ~30% adoption"),
    ("Context Today",  "How satisfied are engineers with DX today?",                  "Yes",     "~6/10; main pain is onboarding and knowledge silos"),
    ("Context Today",  "How long does onboarding take?",                             "Yes",     "4-5 months to full productivity; first 6 weeks hardest"),
    ("Context Today",  "What is the overall engineering org size and structure?",     "Yes",     "~80 engineers; small platform team of 4"),
    ("Skills Mgmt",    "What is the process for creating skills/agents?",             "Yes",     "No formal process; ad hoc scripting"),
    ("Skills Mgmt",    "Who owns shared agent configurations?",                       "Yes",     "Platform team trying to own but bandwidth-constrained"),
    ("Skills Mgmt",    "Is there versioning for shared coding standards?",            "Yes",     "Informal Google Doc; no versioning"),
    ("Skills Mgmt",    "How are proven patterns/templates shared across teams?",      "No",      "Not covered"),
    ("Quality & Eval", "How is AI code quality measured?",                            "Yes",     "No specific metrics; standard PR review only"),
    ("Quality & Eval", "What is the tool evaluation process?",                        "Yes",     "Ad hoc platform team pilots"),
    ("Quality & Eval", "How is AI tool adoption tracked?",                            "Yes",     "Limited; Copilot usage stats only"),
    ("Quality & Eval", "How are AI tooling gaps identified?",                         "Yes",     "Reactive; no proactive gap analysis"),
    ("Decision",       "Who is the final decision maker?",                            "No",      "Not covered"),
    ("Decision",       "What is the timeline for a decision?",                        "Yes",     "Q3 target; budget approved in February"),
    ("Decision",       "Has budget been allocated?",                                  "Yes",     "$150K earmarked; ~$90K available"),
    ("POC Scoping",    "What does success look like for a POC?",                      "Partial", "Mentioned Q3 decision but no success criteria defined"),
    ("POC Scoping",    "What is the ideal team for a POC?",                           "Partial", "Platform team mentioned as likely; not confirmed"),
    ("POC Scoping",    "What is the evaluation timeline?",                            "Partial", "Q3 decision window but no evaluation milestones set"),
    ("Competitive",    "What other solutions are being evaluated?",                   "Yes",     "Sourcegraph Cody rejected; one unnamed vendor in consideration"),
]

fill_map = {"Yes": YES_FILL, "No": NO_FILL, "Partial": PARTIAL_FILL}
for row_data in checklist:
    row = ws_cl.max_row + 1
    ws_cl.append(list(row_data))
    style_data(ws_cl[row])
    ws_cl.cell(row, 3).fill = fill_map.get(row_data[2], NA_FILL)

wb.save("/tmp/Orbital-Software-discovery.xlsx")
print("Saved: /tmp/Orbital-Software-discovery.xlsx")

=============== FILE: inputs/followup_transcript.txt ===============
[Attio Call Recording | Orbital Software | 2026-04-14]
Attendees: Leo Morrow (AE, your company), Dana Khalil (SE, your company), Marcus Webb (VP Engineering, Orbital Software), Sandra Liu (Platform Lead, Orbital Software)

---

Leo: Thanks for the follow-up, Marcus. We wanted to circle back on a few questions we didn't get to last time. Let's start with how teams share patterns and templates — is there a mechanism for that today?

Marcus: Yeah, that's actually something Sandra has been working on. Sandra?

Sandra: We started a kind of internal template library last quarter. It's just a GitHub repo right now — engineers can submit PRs with reusable prompt templates, Copilot config snippets, that kind of thing. The adoption is spotty but the concept is there.

Marcus: I'd say about a third of the platform team uses it regularly. The rest don't know it exists, honestly. Discovery is the main problem — there's no easy way to find what's in there.

Leo: Got it. That's useful context. The other open item was around the decision process — who ultimately signs off on adopting a tool like this?

Marcus: That's me, in consultation with our CTO, Farida Hossain. She approves any new vendor spend over $50K. This would fall in that range, so she'd need to sign off, but she's already in the loop. Farida attended our internal review last week and gave a green light to proceed to a final evaluation.

Leo: That's great to hear. How many engineers have left in the last year, and what was the impact?

Marcus: We lost four senior engineers in the past 14 months. Each departure took about 6-8 weeks of recovery time — slower velocity, lots of urgent documentation sprints, senior engineers context-dumping into shared docs that nobody reads afterward. It's been genuinely painful. Two of those engineers owned entire subsystems with no backup.

Leo: That's a significant impact. Anything else come up since our last call?

Marcus: One thing I wanted to flag — we had a situation last month where a junior engineer merged an AI-generated change that introduced a performance regression in our payments processing path. It took us two days to track down because we couldn't tell which parts of the PR were AI-written. That incident has created a lot of urgency internally around AI code governance. Farida brought it up in the leadership review.

Leo: Thanks — that's really valuable. Let's pick up the POC conversation next week.

---
[End of transcript]
