# PART 3 — The 10 Workflows (each a separate process)

Keep PARTS 1–2 unchanged. Build `/workflows` (gallery of 10 cards) and `/workflows/:id` (the runner).

## 3.1 Workflow engine
- Each workflow is defined in `workflows.ts` as an ordered list of **stages**. Each stage = `{ id, title, specialist, skillIds: string[], gate?: "human" | null, description }`.
- Runner UI: a **vertical stepper** on the left (stage name, owning specialist monogram, status: pending / running / awaiting review / done). Main panel shows the current stage's input, output and actions.
- Stage 0 for every workflow is **Matter Intake** (facts, documents, jurisdiction, acting for). Every later stage receives the intake plus all prior stage outputs as context (truncate oldest outputs first if over the size limit).
- Each stage = one `ai` call in `workflow_step` mode using the generic skill prompt for its primary skill, plus: "You are stage {{n}} of the {{workflow}} workflow. Build on prior stage outputs; do not repeat them."
- **Human gates:** at stages marked `gate: "human"`, pause with an "Approve and continue / Edit output / Re-run stage" panel. Nothing proceeds without approval. Show a small padlock icon on gated stages.
- Final stage of every workflow = **Consolidated Work-Product**: one combined document (executive summary, issue register table, stage outputs as appendices, consolidated "Notes for counsel", verification flags) with DOCX/PDF export.
- An **audit trail** tab: timestamped log of each stage, gate decisions and edits.
- Each workflow is independent: its own route, state, demo matter, and export. Workflows can be started from: the gallery, the CLO's recommendation, or a skill output ("Continue in workflow").
- "Load demo matter" button on every workflow (realistic UK facts, see §3.3). For the three ★ hero workflows, ship **full pre-built Demo Mode outputs for every stage** so they run end-to-end with zero API calls.

## 3.2 The 10 workflows and their stages
1. ★ **Contract Review & Negotiation** — Intake → Contract summary (contract-summariser) → Risk review (contract-reviewer) → Liability & indemnity deep-dive (indemnity-liability-analyst) → 🔒 Gate → Redlines (redline-proposer) → Negotiation positions (negotiation-position-planner) → Adversarial check (adversarial-reviewer) → Consolidated.
2. **M&A Due Diligence** — Intake → Diligence checklist (m-and-a-diligence-checker) → Corporate & structure (deal-structure-analyst) → Contracts sweep (obligations-extractor) → Employment (uk-employment-law-applicability-checker) → IP (ip-portfolio-analyst) → Data protection (uk-data-protection-compliance-checker) → 🔒 Gate → Issue register & red flags → Consolidated.
3. **Litigation Preparation** — Intake → Chronology (chronology-builder) → Issues (issue-spotter) → Limitation (limitation-checker) → Pre-action compliance (england-wales-pre-action-protocol-checker) → Evidence map (evidence-organizer) → 🔒 Gate → Strategy (litigation-strategy-planner) → Particulars framework (england-wales-civil-claim-drafter) → Consolidated.
4. **Regulatory Compliance Review** — Intake → Applicability (regulatory-applicability-analyst) → Obligations register (compliance-obligations-mapper) → Gap analysis → 🔒 Gate → Remediation plan → Filing needs (regulatory-filing-preparer) → Consolidated.
5. ★ **Data-Breach Response** — Intake (what, when discovered, data types, data subjects, jurisdictions) → Containment & evidence (breach-response-planner) → Harm & risk assessment → Notification analysis (uk-data-protection-compliance-checker: ICO notification within 72 hours of becoming aware where required under UK GDPR Art. 33; data-subject notification where high risk) → 🔒 Gate → Draft ICO notification and data-subject communication → Remediation → Consolidated. Show a live **72-hour countdown** from the "became aware" time entered at intake.
6. **Internal Investigation** — Intake → Whistleblower report triage (whistleblower-report-analyst) → Investigation plan & legal hold (legal-hold-planner) → Evidence & chain of custody (chain-of-custody-documenter, digital-evidence-reviewer) → Fraud patterns (fraud-pattern-analyst) → 🔒 Gate → Report (investigation-report-drafter) → Consolidated.
7. **Commercial Dispute Lifecycle** — Intake → Notice analysis (legal-notice-analyser) → Merits & viability (litigation-viability-assessor) → Pre-action steps (england-wales-pre-action-protocol-checker) → 🔒 Gate → Letter Before Claim / response (demand-notice-drafter or notice-reply-drafter) → Settlement options (settlement-strategy-planner) → Consolidated.
8. **Arbitration Lifecycle** — Intake → Clause review (arbitration-clause-reviewer) → Notice of arbitration (arbitration-notice-drafter) → Tribunal appointment (arbitrator-appointment-advisor) → Interim relief options (arbitration-interim-relief-drafter) → 🔒 Gate → Statement of case (arbitration-pleading-drafter) → Award review & challenge (award-challenge-analyst) → Consolidated. Default seat London, Arbitration Act 1996 (as amended).
9. **Financing Transaction** — Intake → Structure (deal-structure-analyst) → Facility/loan terms review (contract-reviewer) → Security & guarantees (security-documenter, guarantee-analyst) → 🔒 Gate → Conditions precedent checklist (transaction-document-checker) → Board approvals (board-resolution-drafter) → Consolidated.
10. ★ **Dispute Viability Assessment** — Intake → Issue spotting (issue-spotter) → Limitation (limitation-checker) → Merits & evidence (litigation-viability-assessor, evidence-organizer) → Quantum (damages-quantifier) → Costs & recovery (costing-estimator, recovery-strategy-planner) → 🔒 Gate → Recommendation memo: sue / defend / settle / investigate (legal-risk-assessor) → Consolidated. Include a **viability scorecard** (merits, evidence, limitation, quantum, recoverability, costs risk — each Red/Amber/Green).

## 3.3 Demo matters (fictional, UK, ship with each workflow)
1. Northbridge SaaS agreement (reuse Sample A).
2. "Fernhill Retail Ltd" acquiring "Pemberton Kitchens Ltd" — 40 staff, customer data on a CRM, two key supplier contracts with change-of-control clauses.
3. Unpaid invoices of £86,400 owed to "Calder Joinery Ltd" by "Marlow Developments Ltd"; last invoice 14 months ago; disputed quality complaints raised late.
4. "Brightwater Lettings Ltd" — residential lettings agent; review of AML, UK GDPR and consumer obligations.
5. Ransomware on "Hollins & Webb Accountants LLP" — client tax records (names, NI numbers, bank details) of ~3,200 individuals; became aware yesterday 16:30.
6. Anonymous whistleblower alleging the procurement manager of "Tamworth Freight Ltd" approved inflated invoices from a supplier owned by a relative.
7. "Orchard Foods Ltd" received a letter claiming £240,000 for alleged breach of an exclusive supply agreement.
8. LCIA-seated dispute (London) between a UK shipbuilder and a Greek charterer over late delivery; arbitration clause names "London arbitration" without rules.
9. £3m term loan to "Ridgeway Clinics Ltd" with a debenture and personal guarantees from two directors.
10. Former employee of "Kingsmead Software Ltd" set up a competitor and took a client list; client asks whether to sue.
