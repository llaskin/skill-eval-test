# Axiom Health Discovery Analysis — Multi-Source Evidence Report

## Problem/Feature Description

Axiom Health is a prospect we've been engaged with over several weeks. Our AEs log calls in Attio CRM, and Granola automatically records all calendar meetings. When we pulled data for Axiom Health, we found that our Attio CRM has two recorded call transcripts and Granola captured one additional meeting that isn't in Attio (an informal product walkthrough that ran outside the formal Attio-logged call sequence).

The problem: some topics came up in more than one call, across both systems. We need a clean discovery analysis that properly attributes each customer statement to the right source and date, so that the sales team can see exactly what was said — and in which call — without ambiguity. The analysis should reflect the full picture from all available recordings.

Analyze the merged call data below and produce a discovery analysis report covering Sections A and B (A1–A4, B1–B4). For each question, extract the customer answer with appropriate evidence citation. Save the output to `discovery_analysis.json`.

## Output Specification

- `discovery_analysis.json` — JSON file with one entry per question covering A1–A4 and B1–B4, each containing:
  - `question_id`: e.g. "A1"
  - `question`: the question text
  - `customer_answer`: the extracted answer with verbatim quotes where possible
  - `assessment`: GOOD / CAUTION / RED FLAG / N/A
  - `sources_used`: list of source attributions with dates

## Input Files

The following merged call data is provided. Extract the files before beginning.

=============== FILE: inputs/merged_calls.json ===============
{
  "company": "Axiom Health",
  "calls": [
    {
      "call_id": "att-rec-201",
      "source": "attio",
      "date": "2026-02-14",
      "title": "Axiom Health — Discovery intro call",
      "attendees": ["Priya Mehta (Axiom, CTO)", "Daniel Park (Axiom, Head of DevEx)", "Leo Chen (Tessl, AE)"],
      "transcript_excerpt": "Leo: Can you walk me through the AI tools your engineers are using today?\nPriya: Sure. We rolled out Claude Code to the entire engineering org last October — we have about 320 engineers. Adoption is around 70% actively using it. We also have a small group, maybe 20 engineers on the platform team, who are using Cursor as well. We haven't standardised on anything else.\nLeo: And how do engineers today find context about the codebase when they're starting a task?\nPriya: Honestly, it's a mess. We have a Confluence wiki but it's massively out of date — I'd say maybe 30% of it is current. Engineers mostly just ask Slack or go straight to the code. Some of our more senior folks have started dropping CLAUDE.md files into repos, maybe 25% of repos have one, but there's no standard format.\nLeo: How long does it take a new engineer to be productive?\nPriya: The painful truth is about 7 months for someone to feel fully independent. The first 3 months they're basically shadowing. It's something we really want to fix.\nDaniel: Yeah, and it's gotten worse as the codebase has grown. We went from 80 engineers to 320 in three years — the contextual knowledge didn't scale with headcount.\nLeo: Do you have any formal process for capturing and sharing engineering best practices or standards?\nPriya: We have an RFC process, but it's very backend-heavy. Frontend patterns are largely undocumented. We don't have anything like a shared skills library."
    },
    {
      "call_id": "att-rec-202",
      "source": "attio",
      "date": "2026-03-07",
      "title": "Axiom Health — Technical deep-dive",
      "attendees": ["Daniel Park (Axiom, Head of DevEx)", "Rania Aziz (Axiom, Staff Engineer)", "Maria Santos (Tessl, SE)"],
      "transcript_excerpt": "Maria: What's your source control setup?\nDaniel: We're on GitHub Enterprise. We have a hybrid setup — one main monorepo for the core product plus about 50 microservice repos. The monorepo has the most complexity.\nMaria: Any AI-assisted code review tools?\nDaniel: We evaluated CodeRabbit about 6 months ago but didn't adopt it. The signal-to-noise ratio wasn't good enough for our compliance-heavy codebase. We're aware of Greptile but haven't tried it. Right now our review process is manual.\nMaria: How would you describe the team's attitude toward new AI tooling?\nDaniel: Cautious but curious. Priya sets the tone — she'll give something a 4-week pilot and then make a data-driven call. We're not early adopters who try everything, but we're also not resistant.\nRania: The challenge is compliance. We're HIPAA-governed, so any new tool goes through a security review that takes 6–8 weeks. That slows experimentation.\nMaria: What about existing context files in repos — AGENTS.md, .cursorrules, anything like that?\nRania: CLAUDE.md in maybe a quarter of repos, and Rania herself wrote a RULES.md in three of the microservice repos. Nothing standardised across the org."
    },
    {
      "call_id": "grn-mtg-301",
      "source": "granola",
      "date": "2026-02-28",
      "title": "Axiom Health product walkthrough",
      "attendees": ["Priya Mehta (Axiom, CTO)", "Leo Chen (Tessl, AE)", "Sam Liu (Tessl, Product)"],
      "transcript_excerpt": "Leo: Just to fill in some gaps from our last call — you mentioned about 320 engineers using Claude Code. Is that the whole engineering org, or just certain teams?\nPriya: The whole engineering org, yes. 320 in total. We also have about 15 designers and 8 PMs who have been granted access but I don't think many of them use it actively — it's really an engineer tool for us.\nLeo: And Cursor — you mentioned maybe 20 engineers on the platform team using it?\nPriya: That's right, 20 on the platform team. It's mostly for the TypeScript frontend work.\nLeo: What's your SSO setup? We'll need to know for provisioning.\nPriya: We're on Okta. We have SAML-based SSO and SCIM provisioning set up with most of our SaaS tools already.\nLeo: Any plans to change that?\nPriya: No, Okta is our standard. All new tooling gets integrated with it.\nSam: And just for context — how active is the DevEx team in reviewing new AI tools that come up?\nPriya: Daniel's team does a quarterly AI tooling review. They evaluate anything that's gotten traction in the org and decide what to standardise on."
    }
  ],
  "dedup_log": {
    "attio_call_count": 2,
    "granola_total_count": 2,
    "granola_unique_count": 1,
    "granola_duplicates_dropped": 1,
    "notes": "grn-mtg-300 (Feb 14 discovery intro) was dropped as a duplicate of att-rec-201 — same date and overlapping attendee domain axiomhealth.com"
  }
}

=============== FILE: inputs/sections_reference.json ===============
{
  "sections": {
    "A": [
      { "id": "A1", "question": "How do your engineers find context about the codebase today when starting a new task?", "why_it_matters": "Reveals current context-finding pain and opportunity for improvement" },
      { "id": "A2", "question": "What AI coding tools and agent harnesses are your engineers using today?", "why_it_matters": "Establishes baseline AI adoption and tool mix" },
      { "id": "A3", "question": "How satisfied are developers with the quality of context available to them?", "why_it_matters": "Quantifies the pain and urgency" },
      { "id": "A4", "question": "How long does it take a new engineer to become fully productive?", "why_it_matters": "Onboarding time is a proxy for context quality and knowledge transfer efficiency" }
    ],
    "B": [
      { "id": "B1", "question": "Do you have any formal process for capturing and distributing engineering best practices or standards?", "why_it_matters": "Reveals maturity of knowledge management and appetite for skills" },
      { "id": "B2", "question": "Who is responsible for maintaining shared context or standards across teams?", "why_it_matters": "Identifies ownership model and potential champion for Tessl" },
      { "id": "B3", "question": "How do you handle updates or versioning of standards as practices evolve?", "why_it_matters": "Surfaces distribution problem that skills solve" },
      { "id": "B4", "question": "Do you have a way to share reusable patterns or templates across engineering teams?", "why_it_matters": "Identifies existing analogs to skills and gaps" }
    ]
  }
}
