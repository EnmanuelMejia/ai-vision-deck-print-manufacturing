# AI Implementation UltraPlan — Mid-Size Commercial Print Manufacturer

## An AI-first IT ownership model for a commercial print & packaging manufacturer

**Prepared by Enmanuel Mejia — AI implementation proposal for a mid-size commercial print manufacturer**

**Date:** September 29, 2026

**Created by Interstitium Labs** — interstitiumlabs.dev

![Interstitium Labs sigil](sigil_il.png)

---

### What this document is — and what it is not

This is a **proposal**, written in the candidate's voice. Everything architectural in it is framed as *"what I would build"* — it describes no system I have built at the company, and it claims no access I do not have. Nothing here is presented as already existing inside the company's walls.

Every claim about the company's operation carries a status tag so you can trace it:

- **[VERIFIED]** — a public source names it at the company (every one is cited).
- **[INFERRED]** — industry-standard for this stack, but no public the company source; never presented as fact.
- **[UNVERIFIED]** — no public evidence either way; listed explicitly as questions I would want to confirm on the floor.
- **[PROPOSED]** — my recommendation. All architecture, agents, phases, and estimates are PROPOSED.

I have invented no the company metrics, revenues, counts, or internal facts anywhere in this plan. Where measurement matters, the plan says *I would measure it first*.

---

### Contents

1. [Executive Summary](#1-executive-summary)
2. [Baseline: Their Operation as Verified](#2-baseline-their-operation-as-verified)
3. [End-to-End Systems Map](#3-end-to-end-systems-map)
4. [The Hybrid Architecture](#4-the-hybrid-architecture)
5. [The Agents](#5-the-agents)
6. [Phase 0–4 UltraPlan](#6-phase-04-ultraplan)
7. [Risk Register](#7-risk-register)
8. [Interview Q&A Appendix](#8-interview-qa-appendix)
9. [Sources](#9-sources)

---

## 1. Executive Summary

**Who this is for.** A ~100–150-person commercial print and flexible-packaging manufacturer operating three adjacent production buildings ("three plants under one roof"). You are not a typical mid-size printer: you have a public, leadership-driven AI program already — a published AI blog, an internal program connecting core systems and standardizing data, and a CEO who champions AI adoption across the organization. You are ahead of your peer set. This plan is about keeping you there.

**The core thesis.** I would run IT at the company as an **AI-first ownership model** — not "AI as a side project," but AI embedded in how the shop is monitored, troubleshot, documented, and run — built on a **hybrid architecture**:

- **Tier 1 — On-prem open-weights models**, running on the company hardware, on the shop network. Customer artwork, job tickets, Tharstern MIS data, and color profiles for Fortune 500 packaging work never leave the building. [PROPOSED]
- **Tier 2 — Frontier LLMs via API**, for heavy reasoning, code generation, and troubleshooting assistance — used only on data that is safe to send. [PROPOSED]
- **Tier 3 — Jev, TypeSafe's "System One" decision model (launched September 15, 2026), as the decision layer between them.** [PROPOSED — pilot-phase recommendation, not deployed]

The operating principle, in one line: **LLMs generate, Jev decides, software governs.** Generative models draft and explain; the decision model makes calibrated, typed judgments (route this order here, score this anomaly, verify this field) with explicit probabilities and human-escalation thresholds; and ordinary, auditable software enforces guardrails, audit logging, and rollback on every action. No AI system touches a press, a job ticket, or a customer file without a logged, reversible path and a human gate where it matters.

**Why this shape.** Three reasons, all grounded in your operation:

1. **You already have the rules-based spine.** Enfocus Switch routes files by rules; Aleyant Pressero does AI-driven job routing; Prinect PAT benchmarks you against 13,000+ connected presses ([VERIFIED](#2-baseline-their-operation-as-verified), the company AI blog). A calibrated decision layer slots *between* those rules and your people — it adds judgment where rules are brittle and escalates where humans are needed. That is exactly the gap your internal program ("connects core systems, standardizes data, puts practical AI tools into the hands of teams across operations, sales, and finance" — company LinkedIn post, Aug 2026) is built to fill.
2. **Your 2026 growth changed the data surface.** An HP Indigo 200K, a second pouching line, a Hudson-Sharp Ares 400-SUP, a Versafire LV, a third POLAR cutter, a 15,000-sq-ft folding-carton building, and an inline carton camera — all announced in 2026 ([VERIFIED](#2-baseline-their-operation-as-verified)). More equipment, more telemetry, more job tickets, more files. Manual monitoring and tribal-knowledge troubleshooting do not scale with that; an instrumented, AI-assisted operation does.
3. **Confidentiality is the constraint that picks the architecture.** You print for national brands, including Fortune 500 companies ([VERIFIED](#2-baseline-their-operation-as-verified), 2026 PRs). Unreleased packaging artwork cannot go to a public chatbot. So the architecture is data-residency-first: sensitive work stays on-prem by construction, and anything that leaves the building does so through an explicit, logged gateway decision.

**What the plan delivers in year one.** Five measurable outcomes, each with success criteria in the phase plan: (1) repetitive IT and production-floor tasks automated, with time saved tracked against a Phase 0 baseline; (2) AI embedded in daily troubleshooting, documentation, and monitoring — used by operators and CSRs, not just IT; (3) a smooth mixed environment — office IT and OT/production segmented, monitored, and independently recoverable; (4) every user trained on responsible AI use, with a written policy; (5) current, living documentation of every system and automation.

**What I am not claiming.** I have not seen your network, your servers, or your internal AI program's actual state. Every [UNVERIFIED] cell in this plan is a question I want to ask on the floor — including, on Friday, confirming the 2026 additions face-to-face. The Jev layer is an *evaluated recommendation*: I have researched it independently (details in §4), I can defend the fit, and I would run it as a measured pilot with a defined fallback — an open-weights classifier — if access, latency, or cost does not fit.

**Effort language.** Where I size work, I use t-shirt sizes in *owner-weeks* (one owner = me, as your IT Systems Administrator): **S** = 0.5–1 week, **M** = 2–4 weeks, **L** = 5–8 weeks, **XL** = 9–16 weeks. All estimates are labeled as estimates — order-of-magnitude planning aids, not commitments.

---

## 2. Baseline: Their Operation as Verified

*Everything in this section describes the company as public sources describe it. Status tags and sources are part of every claim. Where the research corrected an earlier premise, the correction is stated explicitly.*

### 2.1 Company

- **Founded 2006/2007** by its founders **[VERIFIED]** — company history; Trade & Industry Development, Jan 2026.
- **100–150 employees** **[VERIFIED]** — Enterprise League company profile. Public sources range from "over 70" (company site) to 100–150; I use the verified bracket.
- **Three adjacent production buildings, described as "three plants under one roof"** **[VERIFIED]** — Trade & Industry Development, Jan 2026.
- **31 hires in the 12 months before January 2026**, including an additional shift in flexible packaging **[VERIFIED]** — Trade & Industry Development, Jan 2026.
- **Services:** commercial/digital/wide-format print, folding cartons, flexible packaging, labels, direct mail, fulfillment, mailing, creative/design, bindery/finishing, trade-show support, promotional items **[VERIFIED]** — same source; equipment list, Mar 2025.
- **Client verticals:** restaurants, healthcare, financial services, entertainment, vacation ownership **[VERIFIED]** — PIWorld; **"national brands, including a number of Fortune 500 companies"** **[VERIFIED]** — the company 2026 press releases; food/CPG via flexible packaging **[VERIFIED]** — pouch press releases. **Revenue figure: UNVERIFIED** — no public source found.

### 2.2 The 2026 equipment wave

All five announcements below are **[VERIFIED]** with sources. I would want to confirm each on the floor on Friday.

| # | Announcement | Date | What it is | Source |
|---|---|---|---|---|
| 1 | **HP Indigo 200K digital press** | **April 17, 2026** | Dedicated to the packaging division: custom pouches, flexible wraps, pressure-sensitive labels at higher volume and speed. the company: *"For product marketers, packaging is as important as the product itself."* | blog · Packaging Gateway |
| 2 | **Heidelberg Versafire LV + third POLAR cutter** | **April 3, 2026** | Five-color digital with white/clear/neon toners. Prinect Digital Frontend unifies the Speedmaster CD 102 and Versafire into one workflow for routing, scheduling, color management, reporting, and automation. the company: *"minimizing operator touchpoints, improving quality, and reducing waste"*; *"improved scheduling and greater data visibility."* | PR Newswire via Morningstar · PostPress Mag |
| 3 | **POUCH³** | **April 30, 2026** | Sun Centre-patented cuboid pouch using 30–40% less material than a typical stand-up pouch; printable on all six sides. | blog |
| 4 | **Sun Centre XL-DR pouching machine** | **May 21, 2026** | Stand-up pouches, registered gussets, inserted/terminated side-gusset bags, bag-in-bag, closures. Flexible-packaging capability at the company dates to Feb 2020. | blog · The Packman |
| 5 | **Hudson-Sharp Ares 400-SUP with custom dual registration** | **~June 2026** | Front-to-back registered inserted-gusset pouches; up to three films (clear front, metallic back, paper bottom). the company: *"we instantly have expanded our pouch capabilities."* | Manufacturing Today |
| 6 | **Third production facility** | **Announced January 30, 2026** | New 15,000-sq-ft building for folding cartons, finishing, mailing, offices/meeting space; houses the folding-carton operation with **new inline carton-camera equipment**. | PR via Millis Medway News |

**Correction stated explicitly:** a "June 2026" date for the third building appears in some summaries, but the announcement I could verify is dated **January 30, 2026** — the June date is **UNVERIFIED** and I do not use it.

**Pre-existing fleet** (March 2025 published list) **[VERIFIED]** — Equipment List 2025 (PDF): Heidelberg Speedmaster **CD 102-6+L** (40″, inline aqueous coater, Image Control), Printmaster QM 46-2, **Suprasetter** CtP, HP 25K, HP Indigo 7R/6R, Ricoh Pro C9100/C9210, HP Latex 2700/R2000, **Tharstern MIS**, **Heidelberg Prinect Production**, **Enfocus Switch / PitStop Server**, HP SmartStream, Adobe Creative Suite. (2023 list also showed X-Rite IntelliTrax AT240SM — **VERIFIED historical**.)

### 2.3 Awards — with the corrections stated

- **Florida's Best Printer 2026** — Florida Print Awards (Florida Graphics Alliance), August 21, 2026; **Golden Flamingo for the fourth time**; 58 Best of Category, 45 Awards of Excellence, 28 Judges Awards; three special trophies (Best Invitation — Dr Phillips Gala Box; Best Art Reproduction — "Elephant Swimming Laps"; Best Digital Printing — ORL28 Olympic Qualifier Bid Book) **[VERIFIED]** — PR Newswire via Morningstar.
- **2026 Flexible Packaging Association Achievement Awards:** the company won **Silver for Shelf Impact** (stand-up pouches for a pet nutrition brand) **[VERIFIED]** — blog. **Correction:** POUCH³'s **Highest Achievement + two Gold awards** (Expanding Use of Flexible Packaging; Packaging Excellence) belong to **Sun Centre**, the equipment maker — not to the company. I keep them separate.
- **Inc. 5000: 2016 and 2019 only.** 2016: 18% year-over-year growth; 2019: 62% three-year growth, top-50 regional **[VERIFIED]** — blog (2016) · blog (2019). **Correction: no 2025/2026 Inc. 5000 listing was found — I do not claim one.**
- Also: ADDY (Direct Marketing/Direct Mail), Best in Show + Best Packaging at Florida Print Awards **[VERIFIED]** — Trade & Industry Development. **SGP certification: inaugural 2010** — first SGP-certified printer in Florida **[VERIFIED]** — SGP Partnership.

### 2.4 Their AI program — the anchor of this plan

the company's public AI posture is unusually forward for a mid-size printer, and this plan is built as an **extension of what they started**, not a repair of something broken.

**The company blog: "Human Creativity, Computer Efficiency: How We Use AI at the company" (~January 2026)** **[VERIFIED]** — usa.com. Five pillars, in their own words:

1. **Intelligent printing:** Heidelberg Color Assistant Pro ("intelligent ink control and color matching"); HP presses' built-in AI/ML ("automatically optimizes calibration, detects and corrects defects in real time").
2. **Streamlined prepress:** Signa Station — "automation and AI logic… catch potential errors… fewer touchpoints."
3. **Planned workflow:** Prinect Performance Advisor Technology (PAT) — "AI and data from over 13,000 connected presses worldwide… benchmarks our performance, identifies bottlenecks"; Enfocus Switch — "rules-based intelligence to route, process, and check files automatically."
4. **Client tools:** Aleyant Pressero web-to-print with "AI-driven job routing."

**The internal program** — company LinkedIn post, August 2026 **[VERIFIED]**: *"The CEO has led the company's adoption of artificial intelligence across the organization, building an internal AI program that **connects core systems, standardizes data, and puts practical AI tools into the hands of teams across operations, sales, and finance.**"*

**"Leading Beyond the Algorithm" panel, August 25, 2026, ** **[VERIFIED]** — same LinkedIn post. the CEO's 2026 PR quotes consistently frame investment as lean manufacturing: less operator touch, less waste, more data visibility **[VERIFIED]** — see §2.2 sources; bio: long-tenured CEO — team page. the company employs an "AI & Digital Marketing Manager" (2× Forbes-featured GenAI author) **[VERIFIED]** — VDP blog.

**What is UNVERIFIED about the internal program:** no public detail on *which* core systems are connected or the data-standardization schema, and no long-form interview transcript of the CEO on AI was found. Those are the open questions (§8).

**The honest framing, which I would say out loud:** most press-OEM "AI" is embedded automation plus statistical models, not large language models — Heidelberg's PAT is AI consulting over anonymized benchmarks; HP's onboard AI/ML is calibration and defect correction. That is not a criticism; it is the industry state of the art, and the company's own blog already draws the line between human creativity and computer efficiency. My plan adds the *next* layer — language-model reasoning, calibrated decision models, and software governance — on top of what those systems already do.

---

## 3. End-to-End Systems Map

*One architecture, three confidence levels. I would validate every [INFERRED] and [UNVERIFIED] cell in my first weeks; the [UNVERIFIED] IT-backbone cells are the most important interview topics.*

| # | Domain / system | Status | Evidence / source |
|---|---|---|---|
| 1 | Customer intake — web-to-print portals | **[VERIFIED]** — Aleyant Pressero | AI blog ("AI-driven job routing") — usa.com |
| 2 | Customer intake — FTP file intake | **[INFERRED]** | No public the company source |
| 3 | Customer intake — EDI / API order ingestion | **[INFERRED]** | No public the company source |
| 4 | CRM | **[INFERRED]** | No CRM named publicly (may live in Pressero/Tharstern modules) |
| 5 | Estimating + MIS/ERP | **[VERIFIED]** — Tharstern | Mar 2025 equipment list (PDF) |
| 6 | Prepress workflow automation | **[VERIFIED]** — Enfocus Switch | Equipment list + AI blog ("rules-based intelligence to route, process, and check files automatically") |
| 7 | Preflight | **[VERIFIED]** — Enfocus PitStop Server | Equipment list |
| 8 | Imposition | **[VERIFIED]** — Heidelberg Signa Station | AI blog ("automation and AI logic… catch potential errors… fewer touchpoints") |
| 9 | Proofing (contract / digital) | **[INFERRED]** | No public the company source |
| 10 | RIP farm | **[VERIFIED product family]** — Heidelberg Prinect Production; exact topology [INFERRED] | Equipment list |
| 11 | CtP | **[VERIFIED]** — Heidelberg Suprasetter | Equipment list (an older the company doc misspells "Super Setter") |
| 12 | JDF job tickets | **[INFERRED — strong]** | Prinect Production + Tharstern are both JDF-capable and verified present; explicit JDF use at the company not publicly stated |
| 13 | JMF status messaging | **[INFERRED]** | Prinect Remote Service eCall verified as Heidelberg capability; the company-specific use not stated |
| 14 | XJDF (current CIP4 standard, v2.x) | **[INFERRED]** | No public the company source |
| 15 | Offset press console — Speedmaster CD 102-6+L | **[VERIFIED]** | Equipment list; Heidelberg Image Control **[VERIFIED]** on same list |
| 16 | Digital press — HP Indigo 200K / 7R / 6R / 25K | **[VERIFIED]** | Equipment list + April 2026 Indigo 200K announcement; HP onboard AI/ML per AI blog; PrintOS Nio use [INFERRED] |
| 17 | Wide-format — HP Latex 2700 / R2000 | **[VERIFIED]** | Equipment list; AI-assisted features [INFERRED] |
| 18 | Toner digital — Ricoh Pro C9100/C9210, Versafire LV | **[VERIFIED]** | Equipment list + April 3, 2026 Versafire LV announcement (Prinect DFE unifying CD 102 + Versafire) |
| 19 | Color — Image Control / IntelliTrax | **[VERIFIED]** — Image Control (2025 list); **[VERIFIED historical]** — X-Rite IntelliTrax AT240SM (2023 list) | Equipment PDFs; current closed-loop workflow [INFERRED] |
| 20 | Postpress — POLAR cutters | **[VERIFIED]** — third POLAR cutter, April 2026 | April 2026 PRs (§2.2) |
| 21 | Postpress — folding cartons / finishing | **[VERIFIED]** | Jan 30, 2026 facility announcement; specific bindery machines [INFERRED] |
| 22 | Pouch / flexible packaging line | **[VERIFIED]** — XL-DR (May 2026), Hudson-Sharp Ares 400-SUP w/ dual registration (~June 2026) | §2.2 sources |
| 23 | Vision / quality inspection | **[VERIFIED]** — inline carton camera equipment | Facility announcement: "specialized inline camera system for folding cartons"; AI defect classification [INFERRED] |
| 24 | Warehouse / inventory (WMS) | **[INFERRED]** | Fulfillment services [VERIFIED]; no WMS named publicly |
| 25 | Shipping / logistics (TMS) | **[INFERRED]** | Direct mail + fulfillment [VERIFIED] as services; no TMS named publicly |
| 26 | Billing / AR | **[INFERRED]** | Tharstern billing module is standard; not specifically confirmed |
| 27 | IT backbone: AD/LDAP, VLANs, backup/DR, monitoring | **[UNVERIFIED]** | No public sources — **prime October 2 interview discovery topics** |
| 28 | "JEV" | **RESOLVED — Jev is TypeSafe's System One decision model (launched Sept 15, 2026); NOT a print-industry system** | See §4.3; no print/packaging "JEV" standard exists |

**Three facts from the research that shape the plan's architecture:**

- **The integration spine is Tharstern ↔ Prinect ↔ Switch ↔ Pressero.** These four systems are all [VERIFIED] present, all JDF-capable where it counts, and the CEO's internal program is explicitly about *connecting* core systems. That is where the monitoring and decision agents live (§5).
- **JDF/JMF ticket flow is the highest-leverage unknown.** If job tickets and status messages already flow from MIS to press consoles, my monitoring agent reads that stream; if tickets are still manual, Phase 0's first project is instrumenting it. Either way the plan starts there.
- **The IT backbone is a blank cell — deliberately flagged, not guessed.** I would not walk in assuming I know the domain, the backup posture, or the OT segmentation. That is what Friday is for.

---

## 4. The Hybrid Architecture

*All of this is [PROPOSED] — what I would build: one recommendation per component, with effort sizes, the data each piece touches, and why it exists. Effort sizes are owner-weeks (S = 0.5–1, M = 2–4, L = 5–8, XL = 9–16) and are labeled estimates, not commitments. §4g surveys how competitors deploy AI, then this section's recommendation stands as the single system I would build.*

### Design principles

1. **Data residency is decided by the data, not by convenience.** Customer artwork, job tickets, MIS data, and color profiles for Fortune 500 packaging work never leave the building. Only data classified as safe-to-send reaches an external API, and only through the gateway.
2. **Calibrated autonomy, not blind autonomy.** Every AI judgment carries a probability; thresholds decide whether the system acts, drafts-for-review, or escalates to a human. The decision layer exists to make those thresholds trustworthy.
3. **Software governs.** Guardrails, audit logging, cost tracking, and evals are not add-ons — they are the harness every model and agent runs inside. If it isn't logged, it didn't happen; if it can't be rolled back, it doesn't ship.
4. **Build on their spine, don't bypass it.** The agents read and extend Tharstern, Prinect, Switch, and Pressero — they do not replace them. The vendors' embedded AI (PAT, PrintOS, Color Assistant Pro) keeps doing what it does; this architecture adds the layer above it: vendor AI stays at the machine layer, and the fabric learns from their data (JDF/JMF events, job histories) and orchestrates *across* vendors — the layer no vendor covers (Heidelberg's Touch Free explicitly does not replace an MIS; HP's AI is Indigo-locked).
5. **Tharstern remains the system of record.** AI reads and writes via Tharstern's API and the existing JDF/XJDF + JMF exchange. No MIS rip-and-replace: agents propose, humans and approvals execute anything that moves the business.
6. **OT-safe by construction.** Agents act through pre-approved action sets; human approval gates on every machine actuation; deterministic rules (PitStop-style) remain the hard guardrails beneath ML; every agent action is audit-logged (the Johnson Controls OBI pattern).

### 4a. Tier 1 — On-prem open-weights inference

**What it is.** A GPU-equipped inference host (or small cluster) on the shop network, running open-weights models that the company fully controls — no weights, prompts, or customer data ever leave the premises. It serves the workloads where confidentiality is non-negotiable and where latency to the floor matters.

**The recommendation.** One workstation-class GPU inference server (current-generation data-center or prosumer GPUs) running an open-source serving stack (vLLM-class) with open-weight models (Llama/Mistral-class) — redundant power, UPS, locked rack in the existing server room, on a segmented production VLAN. [PROPOSED] Expansion is phasing, not an option menu: a second node joins in Phase 4 for redundancy when the agent fleet goes production-wide. Precedent: ANZ Bank and France's DINUM run open-weight models self-hosted for regulated data — the same sovereignty logic, sized for a printer.

**Effort:** M (2–4 owner-weeks, estimate) for procurement, install, serving stack, baseline benchmarks, and backup integration. A second node is a further S–M when justified.

**Data it touches.** Customer artwork files, job tickets, Tharstern MIS extracts, color profiles and calibration curves, internal documentation, press-console logs, Switch/PitStop outputs — i.e., everything the business would not want on someone else's server.

**Why it exists.** Two reasons. First, the confidentiality constraint: unreleased packaging artwork for national brands cannot transit a third-party API, full stop. Second, operational independence: prepress and monitoring workloads keep running when the internet doesn't — a print shop's uptime cannot depend on a SaaS status page.

### 4b. Tier 2 — Frontier LLMs via API

**What it is.** API access to one or two frontier large language models, used for the workloads where reasoning depth beats data sensitivity: heavy troubleshooting analysis, code generation (automation scripts, Switch flows, integration glue), runbook drafting, and natural-language analytics over *sanitized* data.

**The recommendation.** One commercial API subscription with enterprise/data-processing terms; the router (§4d) abstracts the provider so models are swappable. The gateway routes open-weight-local by default and escalates to the frontier API only on low local confidence, hard-reasoning tasks, or safe-to-transmit data — the task-typed routing pattern Goldman Sachs runs across OpenAI, Gemini, and Llama for 46,500+ employees (CNBC, Jan 2025), and the cost logic of published routing research (RouteLLM/FrugalGPT: ~95% of GPT-4 quality calling GPT-4 ~14% of the time — research claims, not deployments). A second provider joins in Phase 3 for redundancy and price leverage. [PROPOSED]

**Effort:** S (0.5–1 owner-week, estimate) for account setup, key management in the vault, and gateway wiring; ongoing cost tracked per use case (§4e).

**Data it touches.** Only data classified safe-to-send: anonymized log excerpts, sanitized error traces, generated code, public documentation, internal runbooks with customer identifiers stripped. The gateway enforces the classification — the model never sees raw artwork or MIS records with customer data.

**Why it exists.** Open-weights models are excellent and improving, but frontier models still lead on hard reasoning, long-context troubleshooting, and code generation. Using them *selectively* — through a gateway, on safe data — buys capability without surrendering the data-residency posture. Honest note: vendor ROI numbers in this industry are self-reported and directional; I would measure our own cost-per-resolution before scaling any API workload.

### 4c. Tier 3 — Jev, the decision layer (evaluated recommendation — pilot, not deployed)

**What it is, verified.** Jev is TypeSafe AI's "System One" decision model, **publicly launched September 15, 2026** — two weeks before this writing. It is **a decision model, not a generative LLM**: it answers *typed questions* about a text "state" — `choice` (pick one of up to 255 options), `score` (place on an ordered 2–10-level rubric), `noul` (probability a statement is true). **It writes no text** — it cannot chat, extract values, summarize, or read images/PDFs (text-only input). API: `POST /v1/systemone` at `api.typesafe.ai`; route name `jev-latest`; Python SDK (`typesafe_sdk`, `TypeSafeClient`) verified in use. Pricing: **$0.042 per million input tokens, output free** (~$0.0004/decision). Access: waitlist, OpenRouter, Vercel AI Gateway. Company: $40M seed led by DCVC. Founders: Diogo Almeida (ex-OpenAI), Erik Gafni, Sasha Sheng (ex-Meta/FAIR). (Sources: [typesafe.ai](https://typesafe.ai/); [independent review, dev.to, Sept 23, 2026](https://dev.to/gde/jev-after-eight-days-of-independent-tests-level-with-mid-price-llms-behind-the-frontier-1kln); [Unstract applied analysis, Sept 2026](https://unstract.com/blog/jev-and-where-a-system-one-model-fits-in-document-processing/).)

**What independent testing actually shows — the honest version.**
- **Accuracy:** level with mid-price LLMs; **6.5–11.5 points behind frontier models** in the cleanest comparisons (largest study: 7,977 human-labelled items). It is not a frontier replacement.
- **Calibration:** out of the box, the best-calibrated of the models measured on familiar English tasks; drifts off-distribution (overconfident on some public sets, under-confident on others) — a temperature fit on **50–300 of our own labeled items** per workload fixes most of it. Probabilities are rounded to 0.01; `noul` clamped to 0.01–0.98.
- **"0% hallucination" means schema compliance:** zero invalid answers across 23,703 decision-model calls in the largest study. **It can still confidently pick the wrong option** — that is why escalation thresholds exist.
- **Repeatability:** ~1–2% of answers change between identical passes. Fine for triage; accounted for in thresholds.
- **Speed/cost:** TypeSafe's own "193.6× faster, 444.6× cheaper" comes from four *in-house* workflow evals (TypeSafe discloses possible bias; comparator unnamed) — self-tested against model-averaged answers, not a ground-truth benchmark. Closest independent datapoint: OpenRouter Banking77 head-to-head, Jev 81.0% vs. Claude Opus 5 84.4%, 13× faster median, ~1/22 the cost. Published pricing: $0.042/M input tokens, output free (~$0.0004/decision); field latency p50 ~110–175 ms.
- **Breaks on:** wording/phrasing of options (one-line descriptions fix most), language shift, literal reading, arithmetic/counting, dates, long irrelevant input; most confidently wrong where the answer isn't in the input.
- **How new:** two weeks old; cited arXiv preprints are days old and unrefereed; most studies small, synthetic, or LLM-graded. Pricing longevity is an open question — TypeSafe's own FAQ addresses "are these prices temporary or subsidized?" with *"we can't prove it isn't subsidized."*
- **What doesn't exist yet:** no named enterprise pilot customers, no public production adoption, no independent benchmark reproduction, no published enterprise SLA. Zero-retention enterprise tier is mentioned in practitioner docs only — artwork stays on-prem regardless.

**Privacy — stated plainly.** Vendor privacy claims I have seen repeated — no training on customer data, zero data retention for enterprise tiers — **did NOT appear on TypeSafe's primary sources in my research; they are UNVERIFIED.** A specific "hosted on US West Coast" claim is likewise **UNVERIFIED**. Therefore: **artwork and customer-identifiable data stay on-prem regardless**, and any Jev workload touching production data runs through the gateway with explicit data classification, exactly as the API tier does. I would require written data-processing terms before any pilot touches real job data — and the fallback below exists precisely so the plan does not depend on that outcome.

**Where it fits at the company — as the judgment layer between rules and humans:**
- **Triage** (`choice`): classify incoming order emails/files — invoice vs. PO vs. artwork vs. change-order — before expensive processing.
- **Routing** (`choice`): pick the right workflow/press route for an order (offset vs. digital vs. wide-format), with confidence gating to a human dispatcher.
- **Anomaly scoring** (`score`): rate job-cost variance, waste outliers, or preflight risk on an ordered rubric; flag the tails for review.
- **Verification / guardrails** (`noul`): "does this order match the PO terms?" / "is this artwork change material?" — verify every field, escalate the doubtful ones.
- Text-only means an OCR/text-extraction step sits upstream for scanned documents; it cannot extract values (no invoice numbers, no dimensions) — extraction stays with LLMs or rules, and Jev *judges*.

**Why it fits this stack specifically.** the company already has rules-based routing (Switch, Pressero). Jev adds *calibrated* judgment with explicit escalate-vs-act thresholds — the missing middle between "the rule fired" and "a human decided." That maps directly onto the internal program's pillars: connect core systems, standardize data, put practical tools in operations/sales/finance.

**Pilot terms [PROPOSED].** One workload, one rubric, 50–300 labeled items, 30 days: order-email triage (`choice`) is the candidate pilot. Success = measured precision/recall against human labels, calibrated probabilities after refit, and a documented escalation rate — before anything touches production routing. **Fallback:** an open-weights classifier (fine-tuned small model on Tier 1) if Jev access is not granted, or if latency/cost doesn't fit. The plan works either way; Jev is an accelerant, not a dependency.

**Effort:** S–M (1–3 owner-weeks, estimate) for the pilot including labeling, calibration, and the fallback classifier.

### 4d. The model router / gateway

**What it is.** A single internal service that every AI call passes through. It decides, per request: which tier handles it (on-prem, frontier API, or Jev), based on explicit, logged policy. [PROPOSED]

**The trade-off matrix it enforces:**

| Request class | Route | Why | Example |
|---|---|---|---|
| Customer artwork, MIS records with customer data, color profiles | Tier 1 (on-prem) only | Confidentiality; Fortune 500 packaging work never leaves the building | Preflight risk scoring on a new pouch dieline |
| Structured judgment on text (triage, routing, verification) | Jev (pilot) → Tier 1 fallback | Speed, cost, calibrated probabilities; text-only constraint respected | "Is this email a change-order?" with confidence |
| Heavy reasoning / code / long-context troubleshooting | Tier 2 (frontier API) | Capability; only on sanitized data | "Why did this Switch flow fail? Here's the anonymized log." |
| Anything unclassified | Tier 1 (on-prem) | Fail closed: default-deny to the building | — |

**The recommendation.** Built in-house as a thin service (I would write it — it's integration code, not research): policy engine + provider adapters + the audit/cost hooks from §4e. No vendor product required — building is the whole point of the gateway: it is the one component that must be ours for policy, logging, and cost control to be trustworthy. [PROPOSED]

**Effort:** M (2–4 owner-weeks, estimate) for the gateway, policy engine, and provider adapters.

**Why it exists.** Without a gateway, "hybrid" is just three disconnected tools and data classification is a hope. With it, every AI decision is a *policy* decision — logged, costed, and reversible — and models become swappable commodities instead of lock-in.

### 4e. The harness — what every model and agent runs inside

**What it is.** The operational scaffolding that makes AI safe to run in a production print shop. Six components, all [PROPOSED]:

1. **Local inference serving** — the Tier 1 serving stack: model versioning, health checks, GPU utilization monitoring, automatic failover between served models.
2. **Prompt & version management** — every prompt, rubric, and threshold versioned like code; no "tweak it in the chat window and forget." Changes go through the eval harness first.
3. **Guardrails + content filters** — input/output filtering (PII detection on the way out, injection screening on the way in), hard blocks on disallowed actions (see §5 approval gates), and the data-classification enforcement the gateway depends on.
4. **Audit logging of every AI action** — who/what/which-model/which-version/input-hash/decision/confidence/timestamp, immutable and retained per policy. If an auditor, a customer, or leadership asks "why did the system do that?" — the answer is a log query, not a shrug.
5. **Cost tracking per use case** — tokens, API spend, GPU-hours, and (for pilots) human-time-saved, attributed per agent and per workload. This is how Phase 2 pilots prove themselves.
6. **Eval harness** — regression-tests every automation before production: labeled golden sets per workload, precision/recall/latency budgets, calibration checks for Jev workloads, and a promotion gate (dev → shadow → production). Nothing ships on vibes.

**Effort:** L (5–8 owner-weeks, estimate) across Phase 1–2 — the harness is the single biggest build, and deliberately so: it is what makes everything else trustworthy.

**Why it exists.** A print shop runs on accountability — to customers, to auditors, to the next shift. AI without a harness is a liability; AI inside one is an asset you can defend in a customer audit. This is also the honest differentiator I would bring: most "AI plans" are model demos; this one is infrastructure.

### 4f. Edge vision on the floor (the BMW AIQX + Foxconn NxVAE pattern)

**What it is.** Camera-based vision on the CD 102 delivery and key finishing lines, running on the Tier 1 fabric, with unsupervised anomaly detection: good prints are the training set — no labeled-defect dataset required. [PROPOSED]

**What it is not.** A replacement for the Indigo 200K's on-press inspection (stays as-is) or for vendor inline color (Color Assistant Pro and IQ-501-class closed-loop control keep doing their jobs). It is the *cross-press* quality layer the vendors don't sell — one vision system spanning offset and digital, finishing and delivery.

**Precedent.** BMW's in-house AIQX platform runs 1,000+ camera/sensor units at its Landshut plant alone, live at Dingolfing, Regensburg, and Debrecen (multi-year, multi-site, company engineers on record in trade press). Foxconn's NxVAE detects surface defects without labeled golden samples (vendor claims). The print-shop version is smaller and cheaper — achievable on the same GPU server as Tier 1.

**Effort:** M–L (4–8 owner-weeks, estimate) across Phase 3: camera mounting on one line first, anomaly baselines learned from good production, decision-layer scoring of anomalies, human review of every flag before any automated divert.

**Why it exists.** The 2026 equipment wave (Indigo 200K, Versafire LV, the folding-carton building with its inline carton camera) made visual QC the bottleneck that scales with growth. Cameras are cheap; reprints and customer complaints are not. And it feeds the same decision layer (§4c) and audit harness (§4e) as everything else — one system, not another point tool.

**Human-approval gates.** Every anomaly flag goes to a human review queue in Phase 3; automated divert/reprint is a Phase 4 conversation only after the false-positive rate is measured and signed off by production supervision.

### 4g. Competitive benchmark — survey, then decide

*Ten competitors and near-neighbors, graded on what they actually deploy — not what their marketing says. Evidence grades: **Strong** = independent/trade press + named company + documented outcomes; **Moderate** = vendor case study or named-company announcement; **Weak** = vendor launch/marketing only, no named customer. Every competitor metric below is the claimant's claim — never an independently audited fact — and marketing language is tagged as such. Nothing here is invented. Honorable mentions follow the table.*

| Competitor | What they deploy | Evidence | How the company's proposal matches or exceeds it |
|---|---|---|---|
| Heidelberg | Intellistart 3 AI make-ready (vendor claim: setup under 30 seconds); Color Assistant Pro self-learning color; Portal NL natural-language production analytics (3,000+ shops); Prinect Touch Free AI job routing — first beta ~July 2026, explicitly does not replace an MIS | Strong (shipping automation documented via customer cases); Moderate (AI labels — ML specifics undisclosed) | Keep Heidelberg's machine layer; build the cross-vendor learning/orchestration layer it leaves untouched by design |
| HP (PrintOS / Indigo, incl. 200K) | AAA 2.0 ML inline defect detection with auto-divert/reprint (Elanders case: vendor-reported 1 hr saved per 80k impressions, −5–7% complaints); Print Mode Preflight ML (shipped July 2025); Nio v1.2 PrintOS AI chatbot (Sept 2026); AI Order Intake + AI Upscale; Indigo 200K on-press inspection + Spot Master, 400+ installs | Strong (shipping features, named customers, 2024–2026 trade press; figures are vendor/customer quotes, not audits) | The strongest press-floor AI story — but Indigo-locked and starting at order intake; the proposal integrates Indigo as one device class under a vendor-neutral fabric |
| Konica Minolta IQ-501 | Inline spectrophotometer/scanner with auto registration, density, and gray-balance correction during the run (vendor claim: profiling 30→5 min); FastPrint UK case: halved turnarounds, 10,000+ jobs without manual checking (Sept 2026) | Strong for the product; Moderate for the AI framing ("intelligent" is closed-loop control, not ML) | Extends inline-QC thinking with camera vision + learning across ALL presses (§4f) — a press accessory is not a workflow brain |
| Enfocus (PitStop Server / Switch) | Ubiquitous deterministic rules-based preflight and workflow automation; "AI-powered PitStop" is 2026 show-floor language — no neural preflight in shipping release notes | Strong (automation is real); Weak (ML content — rules engines, not learning systems) | Keep Enfocus as the trusted guardrail base; add ML *above* it — learning from the shop's own prepress history (which files fail, which fixes work), which nobody ships |
| BMW (AIQX) | In-house vision platform: cheap cameras + acoustic sensors along conveyors, cloud-trained models; 1,000+ units at Landshut; live at Dingolfing, Regensburg, Debrecen | Strong (multi-year, multi-site, company engineers on record in trade press) | The template for the edge-vision layer (§4f): the same pattern, smaller and cheaper — achievable on one GPU server |
| GE Appliances | 800+ AI agents on Google Cloud Gemini Enterprise across factories, warehouses, and supply chain (Sept 3, 2026); one agent cut back orders 25% across 600+ suppliers (company-reported) | Strong for the deployment; outcome figures company-reported via trade press | Proves agents orchestrate manufacturing work at scale; the company's agents are smaller and scoped (prepress, MIS monitor, documentation) with human approval gates — same pattern, instrumented for a three-building printer |
| Goldman Sachs | Internal assistant for 46,500+ employees routing across OpenAI, Google Gemini, and Meta Llama by task type (CNBC, Jan 2025) | Moderate-Strong (CNBC-reported, named, dated, quantified) | The exact precedent for the task-typed model gateway (§4d): a router that picks the right model class per task |
| Meta (Llama Guard) | Llama Guard 4 (12B) and Prompt Guard 2 (86M injection classifier) in front of/behind generative LLMs, backing Meta's production Llama Moderations API; NVIDIA NeMo Guardrails as the open-source analog | Strong (production product, Meta-published) | Proves "small model decides, big model generates" is production-standard, not novel — the Jev-style typed decision layer is the next step, not the first |
| ANZ Bank / Omada Health / France DINUM | ANZ runs Llama on its Ensayo platform (on-prem/cloud choice for compliance); Omada Health fine-tunes Llama inside HIPAA environments (4.5 months start to launch); France runs Llama/Mistral in sovereign SecNumCloud for 10,000 agents across 8 ministries | Moderate (vendor case studies, independent evidence brief; figures self-declared) | The sovereignty argument for self-hosting at a printer: customer artwork, job tickets, and mailing lists never leave the building — the same logic banks and governments use for regulated data |
| Jev (TypeSafe "System One") | Typed decisions (Choice/Score/Noul) with calibrated probabilities; launched Sept 15, 2026; $40M DCVC seed; early-access waitlisted API; ~$0.0004/decision; field latency p50 ~110–175 ms | Weak (launch coverage and community demos only; no named enterprise pilots, no production adoption, no independent benchmark reproduction; "193.6×/444.6×" is vendor self-tested) | Evaluated as a plug-in, not a dependency: the decision layer runs on Llama Guard/NeMo or a fine-tuned small open model today, and Jev is piloted behind confidence thresholds with LLM/human escalation. |

**Honorable mentions (short list).** Foxconn NxVAE unsupervised defect detection — good prints are the training set (moderate evidence, vendor claims). Siemens Senseye predictive maintenance — $45M single-site savings, vendor claim from sponsored content, cited only as a vendor-reported benchmark. GE Vernova APM — 1,000+ power plants under predictive monitoring (vendor-reported scale). Johnson Controls OBI — agentic assistant executing pre-approved workflows under human supervision; the template for OT-safe agent actuation. Toyota NA agentic supply chain — +20% forecast accuracy replacing 70+ spreadsheets (partner-reported). RouteLLM/FrugalGPT (research) — 95% of GPT-4 quality calling GPT-4 ~14% of the time (research claims, not deployments).

**Survey, then decide: the single recommendation — SOVEREIGN HYBRID AI FABRIC.** On-prem open-weight models as the default workhorse behind a task-typed routing gateway, frontier APIs for hard-reasoning overflow, a typed decision layer (Jev evaluated, open fallback) for fast routing/scoring/guardrail gates, edge vision on the presses, OT-segmented and MIS-integrated — vendor AI kept at the machine layer, learning orchestrated across vendors. Fit for the company's stack: Tharstern MIS (system of record), Prinect + Suprasetter CtP, Heidelberg CD 102 (6-color + inline coater) plus new 2026 Heidelberg equipment, HP Indigo 200K, Enfocus Switch/PitStop prepress, JDF/XJDF + JMF status exchange, mixed Windows/Mac/Linux estate with AD/LDAP, three production buildings. The layers above (§4a–4f plus the design principles) are the build — one answer, no menu.

*Every major vendor automated their own box — make-ready, inline color, file intake — but nobody ships a learning layer spanning estimating to finishing across vendors, or one that gets smarter from the shop's own history. That is precisely the space this architecture claims.*

*Dashboard note: KPI values shown in the deck's dashboard mock are illustrative targets pending Phase 0 baselines; competitor metrics quoted in this section are claimant claims with named sources, never facts.*

---

## 5. The Agents

*Four concrete agents, each mapped to the company's floor. All [PROPOSED]. For each: purpose, inputs, the tools it may touch, human-approval gates, and rollback. None of these exists today — the baselines they would beat are established in Phase 0, not invented here.*

### Agent 1 — Prepress preflight agent

**Purpose.** Catch file problems *before* they cost press time — before the CD 102-6+L or the Indigo 200K. Bad incoming files are the leading cause of prepress bottlenecks industry-wide; the company's own blog credits Signa Station's automation with catching errors and cutting touchpoints.

**Inputs.** Incoming customer files from Pressero, FTP drops, and sales uploads; PitStop Server preflight reports; Signa Station imposition logs; historical rework/reprint records (once Phase 0 baselines them).

**Tools it can touch (read-first).** Read: file shares, Switch flow logs, PitStop reports, Tharstern job records. Write (gated): annotated preflight reports back into the job folder; CSR notification drafts. It does **not** modify customer artwork — ever. It flags; humans fix.

**How the hybrid stack serves it.** Tier 1 (on-prem) scores preflight risk on the actual file content (artwork never leaves the building); Jev (`score` on an ordered risk rubric, `noul` on "is this file print-ready?") provides the calibrated judgment with an escalation threshold; Tier 2 drafts the plain-English explanation for the CSR when the data is sanitized.

**Human-approval gates.** (1) Any file it scores above the risk threshold goes to a prepress operator queue — it cannot hold or reroute a job on its own. (2) Threshold changes require IT approval and pass the eval harness. (3) Customer-facing messages are drafts until a human sends them.

**Rollback.** Disable the agent's write path in the gateway (one config flag); prepress reverts to the current manual review instantly. All scored files and decisions remain in the audit log for review.

### Agent 2 — JDF/MIS monitoring agent

**Purpose.** Watch the Tharstern ↔ Prinect integration spine — the automation backbone the whole shop depends on — and catch silent failures before production goes blind: tickets not reaching the Prinect JDF hotfolder, actuals not returning to the MIS, hotfolder filling, JMF endpoint going quiet.

**Inputs.** Prinect JDF hotfolder state, JMF receiver health, Tharstern job-ticket flow, MIS↔Prinect sync logs, Switch flow status. (If Phase 0 finds JDF/JMF isn't actually flowing — the [INFERRED] cell in §3 — this agent's first job is instrumenting the flow, not monitoring it.)

**Tools it can touch.** Read: hotfolders, JMF endpoints, MIS database (read replica or API), log shares. Write (gated): alerts to IT and production supervision; a daily spine-health digest. It does **not** restart services or requeue jobs autonomously.

**How the hybrid stack serves it.** Mostly deterministic software with Jev (`noul`: "is the ticket flow healthy?") as the anomaly-judgment layer over log patterns, and Tier 1 summarizing the overnight digest. This agent is 80% monitoring engineering, 20% AI — and that's the point.

**Human-approval gates.** (1) Alerts only — remediation actions (restart a service, clear a hotfolder, requeue) are proposed, never executed, until Phase 3 at the earliest, and only for a documented safe-action list. (2) Any new monitored endpoint needs change control.

**Rollback.** Alerts can be silenced per-endpoint in one step; the underlying monitoring config is versioned, so a bad change reverts cleanly.

### Agent 3 — Troubleshooting co-pilot

**Purpose.** Shrink mean-time-to-resolution for IT and production issues: ingest logs and symptoms, suggest runbook-matched fixes, and draft the incident record. The audience is the next shift as much as IT — tribal knowledge, written down and searchable.

**Inputs.** Sanitized log excerpts (Switch, Prinect, press-console exports, network monitoring), the living runbook library (Agent 4's output), vendor documentation, and the technician's symptom description.

**Tools it can touch.** Read: log shares, runbook library, monitoring dashboards, vendor knowledge bases. Write (gated): draft incident notes and suggested next steps into the ticket. It does **not** execute commands on production systems — suggestions only.

**How the hybrid stack serves it.** Tier 2 (frontier API) does the heavy reasoning over sanitized logs; Tier 1 handles anything with customer or internal identifiers; Jev (`choice`: "which runbook matches these symptoms?") ranks candidate runbooks with calibrated confidence, escalating low-confidence matches to a human.

**Human-approval gates.** (1) Every suggestion is labeled with confidence and source; low-confidence suggestions are visually flagged. (2) Any action on a production system (restart, reconfig) requires explicit human execution — the co-pilot never gets credentials to production hosts.

**Rollback.** Remove the co-pilot from the ticket workflow; runbooks remain usable standalone. No production system was ever in its control path, so there is nothing to unwind.

### Agent 4 — Documentation agent

**Purpose.** Turn ticket resolutions into living runbooks. Every solved incident becomes a searchable, versioned procedure — so the shop stops solving the same problem twice and onboarding stops depending on whoever happens to be on shift.

**Inputs.** Resolved tickets from the shop's ticketing system (whatever it is — see note below), technician notes, the co-pilot's incident drafts, and existing documentation.

**Tools it can touch.** Read: ticketing system, shared documentation. Write (gated): draft runbooks into a review queue; approved runbooks publish to the library with version history.

**How the hybrid stack serves it.** Tier 1 drafts from internal tickets (customer data stays on-prem); Tier 2 polishes structure on sanitized content; Jev (`noul`: "does this draft accurately reflect the resolution?") acts as a second reviewer before human approval.

**Human-approval gates.** (1) No runbook publishes without a human reviewer who worked the incident or owns the system. (2) Runbooks touching OT/production systems require an additional reviewer from production supervision.

**Rollback.** Versioned library — revert any runbook to its prior version in one step; publishing can be paused globally.

**Note on "JIRA":** the specific ticketing system at the company is **[UNVERIFIED]** — I use "ticketing system" deliberately. If it's JIRA, ServiceNow, or a Tharstern module, the agent's ticket adapter is written to that API in Phase 1. The architecture doesn't care which one.

### The fallback, stated up front

If Jev access is not granted (waitlist), or if measured latency/cost doesn't fit a workload, the decision-layer role is filled by a **fine-tuned open-weights classifier running on Tier 1** — same typed-question interface (`choice`/`score`/`noul`-style outputs), same escalation thresholds, same eval harness, calibrated on the same 50–300 labeled items. The agents don't change; only the judgment engine behind the gateway does. I would build the fallback interface first and treat Jev as a swappable upgrade — that way the plan never depends on a two-week-old vendor's waitlist.

---

## 6. Phase 0–4 UltraPlan

*All [PROPOSED]. Owner on every phase is me, as your IT Systems Administrator — I own the outcome, coordinate the vendors and the floor, and report progress against the success criteria. Timeframes are estimates from start date. The five year-one outcomes from the executive summary are the scoreboard; each phase lists which ones it advances.*

Year-one scoreboard (referenced below as O1–O5):
- **O1** — Repetitive IT and production-floor tasks automated, with time saved tracked against a measured baseline.
- **O2** — AI embedded in daily troubleshooting, documentation, and monitoring — used by operators and CSRs, not just IT.
- **O3** — A smooth mixed environment: office IT and OT/production segmented, monitored, and independently recoverable.
- **O4** — Every user trained on responsible AI use, with a written policy.
- **O5** — Current, living documentation of every system and automation.

### Phase 0 — Audit & baseline (weeks 1–4)

**What I do.** Map every repetitive task across IT and the production floor; baseline what it costs in time today; inventory the actual IT backbone (filling the §3 [UNVERIFIED] cells); and document the current state of the internal AI program — which core systems are connected, what data was standardized — so this plan extends it instead of duplicating it.

- Shadow prepress, CSRs, and press crews for one full cycle each; log every task repeated daily/weekly with measured durations.
- Inventory: domain/identity, VLANs, backup/DR posture, monitoring stack, patching cadence, vendor remote-access paths (Heidelberg eCall/Remote Service, HP, others).
- Verify the JDF/JMF flow end to end (Tharstern → Prinect hotfolder → press console → actuals back); document where data is still re-keyed by hand.
- Baseline the numbers the pilots will be judged against: prepress touches per job, rework/reprint rate, ticket-flow failure frequency, mean-time-to-resolution for common IT issues — plus waste %, makeready time, prepress rework rate, first-pass yield, and helpdesk ticket volume. I invent none of these — I measure them. Nothing is AI-enabled until it is measured.

**Owner:** me. **Dependencies:** floor access and introductions (week 1); read-only access to MIS, Prinect, and Switch logs.

**Risks + mitigations.**
- *Risk:* audit fatigue — "another IT guy asking questions." *Mitigation:* time-boxed shadowing, share-back of findings to each team, and I fix one small pain point per team during the audit (goodwill is infrastructure).
- *Risk:* discovering the backbone is undocumented. *Mitigation:* that *is* the deliverable — the documentation I produce in Phase 0 becomes O5's foundation.

**Success criteria (advances O1, O5).** A written task inventory with measured time costs; a verified systems map with no [UNVERIFIED] IT-backbone cells remaining; baseline metrics signed off by production supervision; one quick win shipped per team.

### Phase 1 — Foundation (weeks 5–12)

**What I do.** Build the ground the agents stand on: Tier 1 on-prem inference host online; the model router/gateway with data-classification policy; audit logging and cost tracking live; OT network segmentation for the presses; the ticketing-system adapter; and the eval harness skeleton. No agents ship in this phase — foundation only.

- Procure, rack, and harden the Tier 1 inference host; serving stack with health checks and backup integration. **(M, estimate)**
- Build the gateway: policy engine, provider adapters, classification enforcement, audit + cost hooks. **(M, estimate)**
- Segment production/OT from office IT: dedicated production VLAN, firewall ACLs permitting only what each host needs (JMF to the Prinect server, SMB to file shares, internal NTP/DNS) — following Heidelberg's own hardening guidance (disable RDP, strip unneeded services on Prinect servers). **(M, estimate)**
- Establish the single authoritative internal NTP source and consistent internal DNS for all production hosts — Prinect requires matching NTP across press and server for automation and Kerberos; DNS changes break JMF messaging. **(S, estimate)**
- Stand up the eval harness with the first golden sets (from Phase 0 baselines). **(part of the L harness build, estimate)**

**Owner:** me. **Dependencies:** Phase 0 inventory complete; procurement approval for the inference host; maintenance windows coordinated with production for any network changes (presses don't stop for IT).

**Risks + mitigations.**
- *Risk:* network changes during production hours. *Mitigation:* all OT changes in scheduled, vendor-coordinated windows; rollback configs pre-staged; no change without a tested revert path.
- *Risk:* procurement delay on the inference host. *Mitigation:* gateway and harness development proceeds on existing hardware; Tier 1 workloads run in shadow on a workstation-class machine until the server lands.

**Success criteria (advances O3, O5).** Tier 1 serving with uptime SLO met for 30 days; gateway routing 100% of AI calls with classification enforced (zero unclassified external calls); OT/office segmentation verified by scan; NTP/DNS uniform across production hosts; eval harness running nightly.

### Phase 2 — Pilots (weeks 13–24)

**What I do.** Ship exactly two contained pilots with measured before/after against the Phase 0 baselines. (a) The prepress agent on Enfocus Switch hotfolders — file triage, auto-fix proposals, decision-layer verdicts, human review queue. (b) The Tharstern MIS monitor — JDF/JMF exception detection, late-job prediction, scheduling assist. (The §4g benchmark is why these two: prepress triage is where Enfocus leaves the learning layer unclaimed, and the MIS monitor watches the spine no vendor covers.) The Jev pilot runs here: one workload (order-email triage, `choice`), 50–300 labeled items, 30 days, with the open-weights fallback built in parallel.

- Each pilot: shadow mode (decisions logged, humans act) → assisted mode (drafts/suggestions) → production with gates. Promotion requires passing the eval harness: precision/recall/latency budgets met, calibration checked, escalation rate documented.
- Cost tracking per use case goes live: tokens, API spend, GPU-hours vs. human-time-saved.
- Responsible-AI policy drafted and first training delivered to pilot teams (O4 starts here, not later).

**Owner:** me. **Dependencies:** Phase 1 foundation stable; labeled data from the teams (I do the labeling legwork — I don't ask operators to do my homework); Jev access or the fallback classifier ready.

**Risks + mitigations.**
- *Risk:* a pilot underperforms its baseline. *Mitigation:* that's what pilots are for — kill criteria are written *before* launch; a killed pilot still leaves the baseline data and the harness behind.
- *Risk:* Jev waitlist or privacy terms stall. *Mitigation:* the fallback classifier is the default path; Jev is evaluated only when access and written data terms exist. The plan never blocks on it.
- *Risk:* operators route around the agents. *Mitigation:* agents are built with the floor, not for it — weekly feedback during pilots, and every agent must save its user measurable time within 30 days or it gets redesigned.

**Success criteria (advances O1, O2, O4).** 2–3 pilots in production with before/after metrics published (time saved, escalation rates, cost per use case); Jev pilot verdict documented (adopt / fallback / retry-later) with evidence; responsible-AI policy v1 published and pilot teams trained; kill criteria honored publicly if a pilot fails — credibility compounds.

### Phase 3 — Rollout (months 7–10)

**What I do.** Take what the pilots proved and run it fleet-wide: the prepress agent across all Switch hotfolders and press lines; the MIS monitor across the full Tharstern ↔ Prinect ↔ Switch ↔ Pressero spine; the troubleshooting co-pilot available to every shift. Add the second frontier-API provider for redundancy and price leverage. New in this phase: edge vision on the CD 102 delivery and finishing lines (§4f) with human review of every flag; predictive maintenance starting with vibration/temperature trending on motors and HVAC — models only when the data supports them; and the documentation agent converting the year's ticket resolutions into runbooks.

- Roll out per line/building with the same shadow → assisted → production promotion, reusing each pilot's golden sets plus new line-specific labels.
- Extend monitoring to the 2026 additions: Indigo 200K telemetry, the inline carton camera feed (human review vs. automated reject — a Phase 0 question — determines the agent's role), pouch-line status.
- Begin predictive maintenance with trending, not models: vibration and temperature sensors on press motors and HVAC, baselined for a quarter before any anomaly model is trained — the Siemens Senseye / GE Vernova pattern, sized for a print shop.
- Harden backup/DR to cover the new AI infrastructure: Tier 1 host, gateway configs, prompt/version store, audit logs — all in the 3-2-1 posture with tested restores.

**Owner:** me. **Dependencies:** Phase 2 pilots meeting their success criteria; production scheduling cooperation for line-by-line rollout windows; second API provider's enterprise terms.

**Risks + mitigations.**
- *Risk:* scaling breaks what pilots proved (different lines, different failure modes). *Mitigation:* per-line calibration and golden sets; no fleet-wide flip — line-by-line promotion gates.
- *Risk:* Tharstern's acquisition by ePS (~Sept 2026) changes the MIS roadmap mid-rollout. *Mitigation:* the monitoring agent talks to documented interfaces (JDF/JMF, APIs), not to Tharstern internals; interface-level integration survives vendor roadmap changes. I would open a direct line to the ePS account team in Phase 1.
- *Risk:* change fatigue on the floor. *Mitigation:* one line at a time, visible time-savings reported back to each crew, and nothing ships that makes an operator's day worse.

**Success criteria (advances O1, O2, O3).** All four agents in production across the fleet with per-line metrics; second API provider live; AI infrastructure covered by tested backup/DR; year-to-date time-saved ledger published.

### Phase 4 — Continuous improvement (months 11–12, then ongoing)

**What I do.** Turn the build into a practice: eval-driven model refresh cadence, quarterly calibration refits, user training for every new hire and every new AI capability, and a yearly architecture review.

- **Evals:** golden sets refreshed quarterly; Jev workloads refit on 50–300 new labels per workload per quarter (calibration drifts — the research is explicit); open-weights models evaluated for upgrade on a fixed cadence.
- **Model refresh:** Tier 1 models and Tier 2 providers reviewed against cost/quality/latency every quarter; the gateway makes swaps cheap — exercise that.
- **Training:** responsible-AI training for all users (O4 complete); "AI office hours" for operators and CSRs; new-hire onboarding includes the runbook library and the co-pilot.
- **Coverage tracking:** which lines, systems, and data types the AI layer actually covers — the map that shows what's instrumented and what still runs on tribal knowledge.
- **Documentation:** the runbook library and systems map are living documents with named owners and review dates (O5 sustained).

**Owner:** me. **Dependencies:** a full year of metrics; management sign-off on the ongoing training time.

**Risks + mitigations.**
- *Risk:* the program decays into unmaintained automation. *Mitigation:* evals and refits are scheduled work with owners, not good intentions — they go on the same calendar as patching and backups.
- *Risk:* model/provider churn. *Mitigation:* the gateway abstraction and the harness mean a swap is a measured migration, not a rewrite.

**Success criteria (all five outcomes).** O1: year-one time-saved ledger vs. Phase 0 baselines, published. O2: agent usage and escalation metrics showing daily operational use. O3: segmented, monitored, independently recoverable — demonstrated by drill, not by claim. O4: 100% of users trained, policy acknowledged. O5: documentation current with review dates — the next IT hire (or the next auditor) can run the shop from it.

---

## 7. Risk Register

*All [PROPOSED] assessments. Likelihood/impact are rough planning judgments, labeled as such — not measured data.*

| # | Risk | L × I (rough) | Mitigation | Residual risk |
|---|---|---|---|---|
| 1 | **OT safety — an agent actuates a press or production system** | Low × Critical | **No agent actuates a press without a human gate — ever.** Agents alert, draft, and suggest; humans execute. Production-system credentials are never issued to any agent; the co-pilot's action path ends at "suggested next step." OT changes follow the existing maintenance-window discipline. | Low. The residual is human error under the gate — addressed by training and by keeping the gate list short and explicit. |
| 2 | **Data leakage — customer artwork or PII reaches an external model** | Medium × Critical | **Data residency by construction:** Tier 1 handles all artwork/MIS/customer data on-prem; the gateway's default route for unclassified data is on-prem (fail closed); Jev and frontier-API workloads run only on classified safe-to-send data through the gateway. **Jev's privacy claims (no training on customer data, zero retention) are UNVERIFIED on primary sources — so artwork stays on-prem regardless of what the vendor asserts**, until written data-processing terms say otherwise. Audit logging makes any leak attributable. | Low–Medium. The residual is misclassification of a new data type — addressed by the fail-closed default and quarterly classification reviews. |
| 3 | **Vendor lock-in — Tharstern/ePS, Heidelberg, or a model provider changes terms or roadmap** | Medium × Medium | **Interface-level integration, not internals:** agents talk to JDF/JMF, documented APIs, and file/log interfaces — Tharstern's acquisition by ePS (~Sept 2026, [VERIFIED] — [PennBizReport](https://pennbizreport.com/news/26181-pittsburgh-technology-company-purchases-software-provider/)) is exactly why. The gateway makes model providers swappable; evals make swaps measurable. Dual frontier-API providers from Phase 3. | Low–Medium. The residual is a vendor deprecating an interface we depend on — addressed by the quarterly architecture review and maintaining the fallback classifier on Tier 1. |
| 4 | **Model drift — Jev calibration or open-weights behavior degrades on new data** | Medium × Medium | **Scheduled refits, not hope:** 50–300 new labeled items per Jev workload per quarter; golden sets refreshed quarterly; eval harness gates every promotion; escalation-rate monitoring as a drift canary (rising escalations = degrading judgment). | Low. The residual is a sudden distribution shift (new product line, new file types) — addressed by the canary metrics and per-line calibration in Phase 3. |
| 5 | **Legacy console OS constraints — press consoles and CtP PCs run vendor-pinned Windows builds IT may not patch** | High × Medium | **Compensating controls instead of patches:** OT segmentation (no internet, no office-LAN path to consoles), Heidelberg's own hardening guidance (disable RDP, strip unneeded services), allow-listed firewall rules, and vendor-coordinated maintenance windows for qualified updates. The monitoring agent watches these hosts' health without requiring agents installed on them. | Medium. The residual is a vendor-disclosed vulnerability with no qualified patch — accepted explicitly, documented, and re-reviewed quarterly. This is standard print-shop reality, not a plan defect. |
| 6 | **NTP/DNS fragility — clock drift or DNS changes break Prinect automation, Kerberos, and JMF messaging** | Medium × High | **Single authoritative internal NTP source; static reservations and internal DNS for all production hosts** (Phase 1); the monitoring agent watches NTP offset and DNS resolution as first-class health signals; any DNS/NTP change goes through change control with a Prinect-impact check. | Low. The residual is a misconfigured change slipping through — addressed by the monitoring canary and the change-control gate. |
| 7 | **Jev-specific — waitlist access, pricing change, or vendor viability** (two-week-old company, $40M seed; pricing longevity openly questioned by the vendor itself) | Medium × Low | **The plan never depends on Jev:** the open-weights fallback classifier is built first, on the same interface and thresholds; Jev is a pilot-phase evaluated upgrade. If pricing moves or access stalls, the fallback is already running. | Low. The residual is sunk pilot effort — bounded to S–M owner-weeks by design. |
| 8 | **JDF integration brittleness — the MIS↔Prinect link silently breaks** (cert expiry, share permission change, full hotfolder) | Medium × High | The monitoring agent (Agent 2) exists for this: hotfolder depth, JMF endpoint health, and ticket-flow lag are monitored like a production service with alerting — turning today's silent failure into a paged event. | Low–Medium. The residual is alert fatigue — addressed by tuned thresholds from Phase 0 baselines, not defaults. |
| 9 | **Adoption failure — the floor routes around the agents** | Medium × High | Build with the floor: weekly feedback in pilots, 30-day measurable-time-saved bar per agent, kill criteria honored publicly. An agent that doesn't save its users time gets redesigned or killed — credibility is the deployment strategy. | Low–Medium. The residual is cultural, not technical — addressed by the same mechanism that builds it: demonstrated, measured wins. |
| 10 | **Single-owner risk — the plan's owner is one IT administrator** | Medium × Medium | **Documentation as the deliverable (O5):** every system, automation, threshold, and runbook is written down, versioned, and owned — the shop must be runnable from the documentation, not from my memory. Cross-train one operator or supervisor per critical system in Phase 3. | Low–Medium. The residual is genuine single-point-of-human-failure during absence — addressed by the runbook library and the co-pilot, which exist precisely to make knowledge transferable. |

**The one-line version I would give leadership:** the plan's risks are ordinary, named, and mitigated — the biggest risk isn't in this table, it's *not* instrumenting a shop that just added five major systems in one year.

---

## 8. Interview Q&A Appendix

### 8a. Questions they will likely ask — and my crisp answers

**"You've read our AI blog. What did you take from it?"**
Your five pillars are real systems doing real work — Color Assistant Pro, HP's onboard AI/ML, Signa Station's automation, PAT benchmarking against 13,000+ presses, Switch's rules-based routing, Pressero's AI job routing. What I took from it is the *gap between the pillars*: the rules route, the models optimize locally, but the judgment between them — triage this order, score this anomaly, verify this field — is still human. That's the layer I'd build, and your internal program's three goals (connect core systems, standardize data, practical tools in ops/sales/finance) are exactly the foundation it needs.

**"The CEO has led our internal AI program. How would you plug into that instead of duplicating it?"**
Phase 0 of my plan is explicitly *not* building — it's mapping what the program already connects and what data was standardized, then extending it. If the core systems are already connected, my monitoring and decision agents read that integration instead of rebuilding it. I'd want him to tell me where the program is and where it stalled; the plan starts where his left off.

**"We just added a lot of equipment in 2026. How does IT keep up?"**
By instrumenting instead of chasing. The Indigo 200K, the XL-DR, the Ares 400-SUP, the Versafire LV, the third POLAR cutter, the folding-carton building with its inline camera — every one of them is a new telemetry and ticket-flow source. My monitoring agent watches the Tharstern↔Prinect spine they all hang off, so growth shows up as data before it shows up as downtime. And I'd love to confirm the 2026 additions on the floor Friday — see them with my own eyes.

**"Our clients include Fortune 500 brands. How do you handle their data with AI?"**
It never leaves the building. My architecture has an on-prem tier specifically so customer artwork, job tickets, and MIS data are processed on the company hardware — no third-party API ever sees unreleased packaging. Anything that goes to an external model is classified safe-to-send first, through a gateway that logs every call. If a customer's NDA says no external AI, the answer is already built in: the on-prem tier handles it.

**"What's your view on the Tharstern acquisition by ePS?"**
I read about it — Tharstern's roadmap now sits under ePS's cloud strategy. My answer is interface-level integration: the monitoring I build talks to JDF/JMF and documented APIs, not to Tharstern internals, so a vendor roadmap change doesn't break the shop's automation. And I'd open a direct line to the ePS account team early — you manage vendors, you don't just consume them.

**"Heidelberg says disable RDP and pin our console builds. How do you patch a shop like ours?"**
Two cadences: aggressive on office IT, scheduled and vendor-coordinated on production. Console and CtP PCs run the builds Heidelberg qualifies — I don't freelance-patch those; I segment them, harden per Heidelberg's own guidance, and put compensating controls where patches can't go. A bad patch that breaks the Prinect link costs press-hours; that's the trade I manage.

**"What would you do in your first 30 days?"**
Audit before I build. Shadow prepress, CSRs, and press crews; map every repetitive task with measured time costs; inventory the actual IT backbone — domain, VLANs, backup/DR, monitoring; verify the JDF/JMF flow end to end; and document the current state of the internal AI program. I'd fix one small pain point per team during the audit, and I'd come out of the month with baselines every later project is measured against.

**"Why the company?"**
Because you're the rare mid-size manufacturer that already treats AI as infrastructure — a public program, a CEO championing it, an AI marketing manager on staff — and you just made the biggest equipment investment in your history. That's the exact moment an AI-first IT ownership model pays off, and it's the work I want to do.

### 8b. Questions I ask them — filling the [UNVERIFIED] cells

**IT backbone (the blank cells in §3):**
1. What does the IT backbone look like today — domain/identity, VLANs across the three buildings, backup/DR posture, monitoring stack?
2. How is the production/OT network segmented from office IT right now — and where do the press consoles sit?
3. What's the backup and disaster-recovery story for the Prinect server, the Tharstern database, and the prepress file shares — and when was the last tested restore?

**The integration spine:**
4. Which systems are actually integrated today — Tharstern ↔ Prinect ↔ Switch ↔ Pressero — and where is data still re-keyed by hand?
5. Do JDF/JMF tickets actually flow from the MIS to the press consoles, or are job tickets manual?
6. Who owns Prinect day-to-day — IT, prepress, or shared — and who owns the Heidelberg remote-service relationship?

**The internal AI program:**
7. Which core systems does the internal AI program already connect, and what was standardized in the data work?
8. How do you measure the AI program today — what does success look like to you, in numbers?

**The floor (Friday):**
9. I'd love to confirm the 2026 additions on the floor Friday — the Indigo 200K, the pouch lines, the Versafire LV, the folding-carton building and its inline camera. And: what does the carton camera feed into today — human review or automated reject?
10. What's the single most painful recurring IT/production issue right now — the one that, if I fixed it in my first 90 days, would make the floor's life tangibly better?

---

## 9. Sources

*Every the company factual claim in this plan traces to one of these. Vendor and industry claims are included with the same rule: cited, and flagged where self-reported.*

**the company — company, growth, equipment**
- — company history
- — expansion, 31 hires, three plants
- — 100–150 employees (verified profile)
- — March 2025 equipment list
- — HP Indigo 200K (Apr 17, 2026)
- — Indigo 200K trade coverage
- — Versafire LV + third POLAR cutter (Apr 3, 2026)
- — Versafire LV trade coverage
- — POUCH³ (Apr 30, 2026); FPA 2026 awards
- — XL-DR pouching machine (May 21, 2026)
- — XL-DR trade coverage
- — Hudson-Sharp Ares 400-SUP (~June 2026)
- — third production facility (announced Jan 30, 2026)

**the company — AI program and leadership**
- — "How We Use AI at the company" (five pillars)
- Company LinkedIn post on the internal AI program (Aug 2026)
- — the CEO bio
- — the AI & Digital Marketing Manager, AI & Digital Marketing Manager

**the company — awards**
- — Florida's Best Printer 2026
- — Inc. 5000 (2016)
- — Inc. 5000 (2019)
- — SGP certification (2010, first in Florida)

**Industry / vendors**
- https://pennbizreport.com/news/26181-pittsburgh-technology-company-purchases-software-provider/ — Tharstern acquired by ePS (~Sept 2026)
- — client verticals

**Jev / TypeSafe**
- https://typesafe.ai/ — TypeSafe homepage (product, pricing, API)
- https://dev.to/gde/jev-after-eight-days-of-independent-tests-level-with-mid-price-llms-behind-the-frontier-1kln — independent review (Sept 23, 2026): accuracy, calibration, speed/cost, weaknesses
- https://unstract.com/blog/jev-and-where-a-system-one-model-fits-in-document-processing/ — applied analysis: triage/routing/verification fit

---

*End of plan. Prepared September 29, 2026. Created by Interstitium Labs — interstitiumlabs.dev.*
