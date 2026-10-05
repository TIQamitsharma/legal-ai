# PART 2 — The firm: Virtual CLO, 9 Specialists, 3 Jurisdiction Counsel, Skills Library

Keep everything from PART 1 unchanged. Add the "virtual legal department" layer.

## 2.1 Data registries (create as typed TS files in `/src/data/`)
- `specialists.ts` — the 9 Specialist Virtual Lawyers.
- `counsel.ts` — the 3 Jurisdiction Counsel.
- `skills.ts` — the full skills registry.
- `workflows.ts` — created in PART 3.
- `sources.ts` — created in PART 4.
- All counts shown anywhere in the UI are computed from these registries (never hard-coded).

Each skill object:
```ts
{
  id: string;              // kebab-case, e.g. "contract-reviewer"
  name: string;            // Title Case display name
  category: string;        // practice area (see list below)
  specialist: string;      // owning specialist id
  jurisdiction: "neutral" | "uk" | "us" | "india";
  description: string;     // one line, written by you from the skill name + category
  outputType: "analysis" | "draft" | "checklist" | "table";
  inputs: Array<{ key: string; label: string; type: "text" | "textarea" | "select" | "file" | "number" | "date"; options?: string[]; required?: boolean }>;
  showcase?: boolean;      // true = has a tuned demo example + pre-built Demo Mode output
  demoInput?: Record<string, string>;
}
```
Default inputs for every skill if not specified: "Matter facts" (textarea, required), "Upload a document" (file, optional), "Jurisdiction" (select, defaults to the header jurisdiction), "Acting for" (text, optional).

## 2.2 The Virtual Chief Legal Officer ("Ask the CLO", `/clo`)
- A chat-style intake page. The user describes a matter in plain language (or uploads a document).
- The CLO makes ONE orchestration call returning JSON: `{ "matter_summary": "", "jurisdiction": "", "jurisdiction_confidence": "high|medium|low", "facts": [""], "assumptions": [""], "missing_information": [""], "specialists_assigned": [{"id": "", "why": ""}], "skills_recommended": [{"id": "", "why": "", "order": 1}], "workflow_recommended": {"id": "", "why": ""} | null, "urgent_deadlines_or_risks": [""] }`.
- Render it as a **Matter Brief card**: summary, a "staffing" row of specialist avatars (monogram circles, no stock photos), recommended skills as clickable chips (open the skill pre-filled with the matter facts), and a primary button "Run recommended workflow".
- The orchestrator may only recommend skill and workflow ids that exist in the registries (validate server-side; drop unknown ids).
- Orchestrator system prompt:
```
You are the Virtual Chief Legal Officer of a UK-facing virtual legal department. You triage a matter and staff it. You do not answer the legal question yourself.
1. Summarise the matter neutrally. Separate facts stated from assumptions.
2. Decide which legal system governs (England & Wales, Scotland, Northern Ireland, US, India, or unclear) and say how confident you are. Never apply England & Wales procedure to Scotland or Northern Ireland.
3. Assign the specialist virtual lawyers needed, recommend skills in the order they should run, and recommend one workflow if one fits.
4. Flag urgent deadlines or risks (limitation, pre-action steps, regulatory notification windows) only if the facts indicate them.
5. List the information missing before advice could be given.
Use only the skill and workflow ids provided in the catalogue below. Return ONLY JSON in the required schema.
CATALOGUE: {{injected list of skill ids + names + workflow ids + names}}
```

## 2.3 The 9 Specialist Virtual Lawyers (`/specialists` section on the Skills page)
Profile cards with monogram, title, practice summary, list of owned skills (from the registry), and "Brief this specialist" button (opens a skill picker filtered to them).
1. **Contracts Counsel** — contracts, property documents
2. **Corporate Counsel** — corporate, finance, startup
3. **Court Litigation Counsel** — litigation, consumer, criminal, family
4. **Dispute Resolution Counsel** — arbitration, conciliation & mediation
5. **Compliance Counsel** — regulatory, privacy, public law, tax
6. **Employment Counsel** — employment
7. **Intellectual Property Counsel** — IP
8. **Investigations Counsel** — investigations
9. **Legal Research Counsel** — research, verification, learning
Plus the CLO owns "advisory" and "practice" skills.

## 2.4 The 3 Jurisdiction Counsel
Cards for **UK Counsel**, **US Counsel**, **India Counsel**, each showing: what it checks before applying local law, its skills, and its official sources (PART 4).
- **Jurisdiction overlay:** whenever a skill runs, append the active counsel's rules to the system prompt:
  - UK Counsel: "Confirm UK law governs. Identify whether England & Wales, Scotland or Northern Ireland applies, or a UK-wide rule. Check territorial extent, devolution, authority hierarchy and effective dates. Never apply England & Wales procedure (CPR, Pre-Action Protocols) to Scotland or Northern Ireland. Flag limitation and procedural deadlines."
  - US Counsel: "Confirm US law governs. Separate federal from State issues, identify the relevant State and forum, check controlling authority and effective dates. Do not imply coverage of every State."
  - India Counsel: "Confirm Indian law governs. Identify the relevant State and forum, check authority hierarchy and the applicable legal version, apply limitation and filing rules."
- **Default jurisdiction is England & Wales.** The UI should feel UK-first; US and India are visible as capability, not the focus.

## 2.5 Skills Library (`/skills`)
- Searchable, filterable catalogue: filter by practice area, specialist, jurisdiction, output type. Card grid with name, one-line description, specialist badge, jurisdiction flag. A "Showcase" ribbon on skills with `showcase: true`.
- **Skill runner page** (`/skills/:id`): dynamic form from `inputs`, a "Load example" button when `demoInput` exists, Run, then a rendered markdown output with sections, an "AI-generated draft — verify before use" label, "Notes for counsel" box, export DOCX/PDF, and "Send to another skill" (pipe the output into the next skill's facts box).
- **Generic skill system prompt** (built per run):
```
You are {{specialist name}}, a specialist in a virtual legal department, performing the skill "{{skill name}}": {{skill description}}.
Jurisdiction rules: {{jurisdiction overlay}}
Produce work-product for a qualified lawyer to review, not advice to a lay client.
- Use only the facts and documents supplied. Mark assumptions clearly. Insert [PLACEHOLDERS] for missing facts in drafts.
- Never invent case names, citations, statute sections or deadlines. If an authority is needed but not supplied by the research step, describe the principle and mark "[authority to be verified]".
- British English for UK matters. Structured headings. Concise.
End with "Notes for counsel" (max 5 bullets: assumptions, gaps, points to verify).
Return ONLY JSON: {"title": "", "output_markdown": "", "notes_for_counsel": [""], "verification_flags": [""]}
```
- Mark these as `showcase: true`, write a realistic UK `demoInput` for each, and ship a pre-built Demo Mode output: contract-reviewer, redline-proposer, negotiation-position-planner, legal-opinion-drafter, litigation-viability-assessor, limitation-checker, chronology-builder, written-submissions-drafter, cross-examination-planner, settlement-evaluator, breach-response-planner, issue-spotter, citation-integrity-checker, adversarial-reviewer, england-wales-pre-action-protocol-checker, england-wales-civil-claim-drafter, uk-data-protection-compliance-checker, uk-employment-law-applicability-checker.

## 2.6 Full skills list (create ALL of these in `skills.ts`)
Write the `description` for each yourself from its name and category. Keep ids exactly as given.

**Jurisdiction-neutral skills (131)**

- **advisory** (CLO): client-intake, client-update-drafter, demand-notice-drafter, engagement-letter-drafter, legal-explainer, legal-notice-analyser, legal-opinion-drafter, legal-risk-assessor, notice-reply-drafter
- **arbitration** (Dispute Resolution): arbitral-award-analyst, arbitration-clause-reviewer, arbitration-interim-relief-drafter, arbitration-notice-drafter, arbitration-pleading-drafter, arbitrator-appointment-advisor, award-challenge-analyst, procedural-order-drafter
- **conciliation & mediation** (Dispute Resolution): adr-brief-drafter, caucus-strategy-planner, conciliation-proposal-drafter, mediation-opening-drafter, party-interest-analyst, settlement-documenter, settlement-evaluator, settlement-strategy-planner
- **consumer** (Court Litigation): compensation-quantifier, consumer-pleading-drafter, deficiency-analyst, product-liability-analyst
- **contracts** (Contracts): clause-comparator, contract-drafter, contract-reviewer, contract-summariser, indemnity-liability-analyst, negotiation-position-planner, obligations-extractor, redline-proposer, termination-analyst
- **corporate** (Corporate): board-resolution-drafter, deal-structure-analyst, investment-and-shareholder-agreement-reviewer, m-and-a-diligence-checker, minutes-drafter, restructuring-documenter, transaction-document-checker
- **criminal** (Court Litigation): defence-strategy-planner, sentencing-analyst
- **employment** (Employment): disciplinary-documenter, employment-contract-drafter, handbook-drafter, separation-documenter
- **family** (Court Litigation): maintenance-calculator, settlement-deed-drafter, will-drafter
- **finance** (Corporate): guarantee-analyst, recovery-strategy-planner, security-documenter
- **investigations** (Investigations): chain-of-custody-documenter, digital-evidence-reviewer, fraud-pattern-analyst, investigation-report-drafter, osint-collector, transaction-tracer, whistleblower-report-analyst
- **ip** (IP): cease-desist-drafter, infringement-analyst, ip-assignment-drafter, ip-portfolio-analyst
- **learning** (Legal Research): learn-law, legal-exam-prep
- **litigation** (Court Litigation): appeal-grounds-drafter, case-law-analyst, chronology-builder, court-order-compliance-checker, cross-examination-planner, damages-quantifier, disclosure-request-drafter, document-review-protocol-builder, evidence-organizer, interim-application-drafter, legal-hold-planner, limitation-checker, litigation-viability-assessor, litigation-strategy-planner, pleadings-analyst, privilege-log-builder, production-set-checker, redaction-reviewer, witness-statement-drafter, written-submissions-drafter
- **practice** (CLO): brief-to-counsel-drafter, closure-report-drafter, conflict-checker, costing-estimator, legal-document-producer, time-narrative-drafter
- **privacy** (Compliance): breach-response-planner, cross-border-transfer-analyst, data-processing-agreement-reviewer, dpia-documenter, privacy-policy-drafter
- **property** (Contracts): development-agreement-reviewer, sale-deed-drafter
- **public** (Compliance): government-contract-reviewer, policy-note-drafter, tender-compliance-checker
- **regulatory** (Compliance): compliance-obligations-mapper, examination-response-drafter, licence-application-drafter, regulatory-applicability-analyst, regulatory-change-monitor, regulatory-filing-preparer, sanctions-screening-documenter
- **research** (Legal Research): comparative-analyst, forum-jurisdiction-analyst, issue-spotter, legal-research-planner, legislative-history-analyst, precedent-mapper, research-synthesiser, statutory-interpreter
- **startup** (Corporate): cap-table-analyst, esop-scheme-drafter, founders-agreement-drafter
- **tax** (Compliance): transfer-pricing-documenter, treaty-analyst
- **verify** (Legal Research): adversarial-reviewer, assumption-flagger, authority-validator, citation-integrity-checker, consistency-checker

**UK Counsel skills (6)**
uk-companies-act-compliance-checker (company records, approvals and filings) · uk-data-protection-compliance-checker (UK GDPR, DPA 2018 and PECR) · uk-employment-law-applicability-checker (employment coverage across UK jurisdictions) · england-wales-pre-action-protocol-checker (protocol and Practice Direction compliance) · england-wales-civil-claim-drafter (claim form and particulars framework) · uk-insolvency-applicability-checker (corporate insolvency and restructuring routes)

**US Counsel skills (6)**
us-federal-state-issue-mapper · us-state-law-research-planner · us-federal-civil-procedure-checker · us-sec-reporting-checker · us-employment-law-applicability-checker · us-privacy-law-applicability-checker

**India Counsel skills (42)**
- Corporate: listing-obligations-checker, related-party-analyst, secretarial-compliance-checker, securities-compliance-checker, startup-compliance-checker
- Courts & disputes: arbitral-award-enforcement-advisor, bail-advisor-and-drafter, chargesheet-analyst, cheque-dishonour-complaint-drafter, cheque-dishonour-notice-drafter, commercial-suit-filing-checker, decree-execution-and-enforcement-drafter, matrimonial-petition-drafter, india-legal-notice-response-strategist, pil-drafter, plaint-drafter, pre-institution-mediation-advisor, quashing-petition-drafter, rti-appeal-drafter, rti-application-drafter, succession-advisor, written-statement-drafter
- Employment: labour-compliance-checker, posh-compliance-advisor
- Finance & recovery: loan-agreement-reviewer, sarfaesi-advisor
- Insolvency: avoidance-transaction-analyst, cirp-timeline-checker, claim-verification-analyst, liquidation-documenter, operational-creditor-application-drafter, resolution-plan-reviewer
- IP & privacy: trademark-opposition-drafter, dpdp-compliance-checker
- Property: encumbrance-analyst, rera-compliance-checker, stamp-duty-analyst, title-diligence-analyst
- Tax & cross-border: fema-analyst, gst-compliance-analyst, tax-appeal-grounds-drafter, tax-assessment-reply-drafter

(Total = 131 + 6 + 6 + 42 = 185. The Home page count must read from the registry.)

## 2.7 Credits page (`/credits`)
"TIQ-VirtualCLO's skill library is adapted from vCLO by Rohas Nagpal (github.com/rohasnagpal/legal-ai-skills), used under the MIT License." Include the full MIT licence text with "Copyright (c) Rohas Nagpal". Also credit data sources (PART 4).
