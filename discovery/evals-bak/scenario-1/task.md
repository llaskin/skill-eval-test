# Maturity Qualification Scoring

## Context

You are processing a discovery call transcript for **NovaSpark Technologies**, a mid-size fintech company with ~400 engineers. The transcript below covers all 8 maturity dimensions from the Discovery Questions Tracker. Your job is to score each dimension 1-3 using the rubric, extract supporting evidence with verbatim quotes, and compute the total maturity band.

## Task

Analyze the transcript excerpts below and produce a structured JSON output containing:

1. A score (1, 2, or 3) for each of the 8 maturity dimensions
2. Verbatim evidence quotes for each score with source attribution
3. Brief notes explaining the rationale
4. A total score (sum of all 8 dimensions)
5. A maturity band classification: High (20-24), Medium (14-19), Low (8-13)

## Scoring Rubric

- **3 = High** (proceed): Strong signals, mature practices, clear fit
- **2 = Medium** (proceed with caution): Some practices in place, gaps exist
- **1 = Low** (consider qualifying out): Minimal practices, significant gaps, poor fit

## The 8 Dimensions

1. AI Tooling Landscape
2. Context Awareness
3. Internal Knowledge Surface
4. Knowledge Distribution Pain
5. Quality Measurement
6. Platform / DevEx Ownership
7. Scale of Developer Base
8. Contribution Readiness

## Expected Output

```json
{
  "company": "NovaSpark Technologies",
  "dimensions": [
    {
      "dimension": "<dimension name>",
      "score": <1|2|3>,
      "evidence": "<verbatim quote>",
      "source": "<attribution>",
      "notes": "<rationale>"
    }
  ],
  "total_score": <sum>,
  "maturity_band": "<High|Medium|Low>"
}
```

## Input Transcript

=============== TRANSCRIPT: NovaSpark Discovery Call — 2026-02-18 ===============
Source: [Attio, 2026-02-18]
Attendees: Sarah Chen (NovaSpark VP Engineering), Marcus Webb (NovaSpark Staff Eng), Leo (Tessl)

---

**[AI Tooling Landscape — 00:04:12]**

LEO: Can you walk me through what AI tooling your engineering team currently uses day-to-day?

SARAH CHEN: We've rolled out GitHub Copilot across all 400 engineers about eight months ago. Adoption is strong — our telemetry shows about 72% weekly active usage. Beyond that, we have a small internal team that built a ChatGPT wrapper for our docs, but honestly it hallucinates a lot and people don't trust it much. We also have three teams experimenting with Cursor, but that's grassroots — no central governance.

MARCUS WEBB: I'd add that our platform team evaluated Cody from Sourcegraph last quarter but decided not to roll it out because the context window wasn't sufficient for our monorepo. We have a pretty sophisticated understanding of what these tools can and can't do.

---

**[Context Awareness — 00:11:45]**

LEO: When your developers are working on a task, how do they get context about the codebase, architecture decisions, or related services?

SARAH CHEN: That's our biggest pain point honestly. We have a monorepo with about 2.3 million lines of code across 18 services. Developers typically grep through code, check our internal wiki — which is about 60% outdated — or just ping someone on Slack. There's no systematic way to understand how services connect.

MARCUS WEBB: We tried building an architecture knowledge graph last year but the project stalled after the tech lead left. Right now context is basically tribal knowledge. New hires take 4-6 months to become productive, which Sarah and I both think is way too long.

---

**[Internal Knowledge Surface — 00:19:30]**

LEO: How does institutional knowledge get captured and surfaced to engineers who need it?

MARCUS WEBB: We have Confluence, but I'll be honest — it's a graveyard. Maybe 30% of our runbooks are current. We also have a Slack channel called #eng-knowledge but it's high-volume and things get buried within hours. The real knowledge lives in a handful of senior engineers' heads.

SARAH CHEN: We tried mandating ADRs — architecture decision records — about a year ago. Compliance is maybe 40%. The ones that do get written are genuinely useful, but there's no way to search across them effectively or link them to the relevant code.

---

**[Knowledge Distribution Pain — 00:27:15]**

LEO: What happens when a key engineer is unavailable or leaves? How does knowledge transfer work?

SARAH CHEN: Painfully, to be direct. We lost our payments domain expert six months ago and we're still feeling it. That team's velocity dropped about 35% and hasn't recovered. We've tried pairing sessions and recorded walkthroughs but it's ad hoc.

MARCUS WEBB: I'd estimate we have about 12 engineers across the company who are single points of failure for critical systems. We know it's a risk but haven't found a scalable solution. Every quarter we say we'll do knowledge transfer sprints but they keep getting deprioritized for feature work.

---

**[Quality Measurement — 00:34:50]**

LEO: How do you currently measure code quality and developer productivity?

SARAH CHEN: We track DORA metrics — deployment frequency, lead time, change failure rate, MTTR. We're pretty data-driven there. We also have a code review SLA dashboard. But we don't have any systematic way to measure the quality of AI-generated code specifically, or whether Copilot suggestions are actually helping versus introducing subtle bugs.

MARCUS WEBB: We had an incident last month that we traced back to a Copilot suggestion that passed review. It was a subtle off-by-one in a pagination handler. That made us realize we need better guardrails but we haven't figured out what that looks like yet.

---

**[Platform / DevEx Ownership — 00:42:20]**

LEO: Do you have a dedicated platform engineering or developer experience team?

SARAH CHEN: Yes, we have a 12-person platform engineering team. They own our CI/CD pipeline, internal tooling, and developer onboarding. They've been around for about two years and have strong executive sponsorship — our CTO considers DevEx a strategic priority.

MARCUS WEBB: The platform team has a formal intake process. They run quarterly developer surveys and their NPS has been trending up — currently at 42. They also maintain our internal CLI tool and golden path templates. It's a well-run team.

---

**[Scale of Developer Base — 00:49:10]**

LEO: Can you tell me more about the size and structure of your engineering organization?

SARAH CHEN: We have 400 engineers total, organized into 8 domains with about 50 engineers each. Each domain has 4-6 squads. We're hiring actively — plan to be at 550 by end of year. We also have about 60 contractors who rotate in and out, which makes the knowledge problem even harder.

MARCUS WEBB: We're distributed across four offices and about 30% fully remote. Time zones span US Pacific to Central Europe. Synchronous knowledge transfer is genuinely difficult.

---

**[Contribution Readiness — 00:55:40]**

LEO: If you were to adopt a new developer tool or platform, how would rollout typically work? What's the appetite for engineers contributing back — creating custom rules, templates, or configurations?

SARAH CHEN: Our engineers are generally enthusiastic about new tools — Copilot adoption was faster than we expected. We have an internal "guild" system where about 80 engineers voluntarily participate in cross-domain interest groups. The AI guild is our largest with about 45 members. They'd definitely want to contribute.

MARCUS WEBB: That said, we've learned the hard way that rollouts need to be structured. Our last three grassroots tool adoptions created fragmentation. So the platform team now gates all new tooling. They'd want a clear integration path and contribution model before approving anything org-wide.

---

END OF TRANSCRIPT
