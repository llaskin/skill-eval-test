# Technical Environment Documentation for CloudNova Systems

## Problem/Feature Description

The sales engineering team at your company maintains discovery records for all prospective customers using structured Excel workbooks. After a discovery call with CloudNova Systems, a cloud infrastructure startup with a fast-growing engineering org, you need to document their technical environment clearly so the account team can reference it during demos and POC planning.

A recording from an Attio call transcript is available (recorded 2026-03-12). The call covered CloudNova's developer tooling, team structure, and engineering infrastructure in detail. Your task is to extract all relevant technical environment data points from the transcript, then generate a Python script that creates a properly formatted Excel sheet capturing this information. Run the script to produce the final xlsx file.

## Output Specification

Produce the following files:

1. `tech_env_extracted.json` — A JSON file containing each technical environment field with its extracted value, a verbatim evidence quote from the transcript (with source attribution), and any notes.
2. `generate_tech_env.py` — A Python script that creates an Excel workbook with a single sheet containing all technical environment data, with appropriate formatting and styling.
3. `CloudNova-tech-env.xlsx` — The output Excel file produced by running the Python script.

## Input Files

The following files are provided as inputs. Extract them before beginning.

=============== FILE: inputs/cloudnova_transcript.txt ===============
[Attio Call Recording | CloudNova Systems | 2026-03-12]
Attendees: Sarah Chen (AE, your company), Marcus Webb (VP Engineering, CloudNova), Priya Nair (Platform Lead, CloudNova)

---

Sarah: Thanks for making time today, Marcus. Can you walk us through how the CloudNova engineering org is set up?

Marcus: Sure. We have about 300 engineers total across three product lines. The platform team — Priya's group — is 18 people, and they own everything from CI/CD to internal developer tooling. We're pretty serious about developer experience.

Priya: Yeah, we have a formal charter. Quarterly engineering surveys, OKRs tied to DORA metrics. It's not just a side project.

Sarah: How are you managing source code today?

Marcus: We run GitHub Enterprise for all production repos. We also have a handful of repos on github.com for open source contributions, but everything internal is GitHub Enterprise.

Sarah: And what's your repo structure — monorepo, many repos?

Marcus: Mix. We have three main monorepos — one per product line — and then a long tail of about 80 microservice repos. The monorepos are the main ones; the microservice repos are older stuff we haven't consolidated yet.

Sarah: Got it. What languages are your teams primarily using?

Priya: Python for the data and ML services, Go for anything performance-sensitive — our core routing engine is Go — and TypeScript on the frontend and some of the internal tooling. Python and Go are the big ones.

Sarah: And AI coding tools — what are engineers actually using day to day?

Marcus: Claude Code is the main one. We standardized on it about six months ago across most of the engineering org. A bunch of engineers also use Cursor — maybe 40% — but Claude Code is the standard. We've been pretty happy.

Sarah: Any concerns about vendor lock-in with those tools?

Marcus: Honestly, less than you'd expect. We think about it, but Claude Code's MCP support has been a big deal for us — it lets us build on top of it without being fully locked in. If something better came along we could probably migrate, but right now we're not losing sleep over it.

Sarah: What's the attitude toward trying new tools or experimenting?

Priya: Very proactive. We have a budget line specifically for developer productivity experimentation, and we encourage engineers to try new things as long as they log it in our tools registry. We adopt fast when something's clearly better.

Sarah: How big is the dedicated platform / DevEx team?

Priya: Right now it's 18 engineers on the platform team, plus 2 PMs. That's the core team. We also have a data engineering sub-team of about 12 that sits adjacent to us.

Sarah: Any context files in repos today — anything engineers maintain to help AI tools understand the codebase?

Priya: We have CLAUDE.md files in the three main monorepos. Engineers maintain them, but the quality varies a lot. The core platform monorepo has a really detailed one; the other two are pretty sparse.

Sarah: And Tessl feature adoption — are you using any skills, plugins, hooks?

Priya: We started using skills about two months ago. We have maybe a dozen skills written so far — mostly workflow automation and code review checklists. Still early but the team likes them.

Marcus: We're also exploring MCP servers for our internal tooling. Nothing in prod yet.

Sarah: Great. I think we have a solid picture for now — we ran a bit long so let's save the SSO setup, any AI code review tooling you might be using, and a few other questions for our follow-up call next week.

Marcus: Sounds good. Looking forward to it.

---
[End of transcript excerpt — SSO Setup, Agent User Roles, AI Code Review Tooling, and Skills Storage were not covered in this session]
