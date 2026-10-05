# PART 1 — Core app, design, Contract Review, Drafting Studio

Build a polished, production-quality demo web app called **TIQ-VirtualCLO** — an AI virtual legal department **built for UK barristers' chambers**. Tagline: **"Your virtual Chief Legal Officer — built for UK chambers."** Footer: "A TIQ Digital product". The audience is an English barrister, so legal accuracy, UK terminology and restraint matter more than flashy features.

## 1. Tech stack
- React + Vite + TypeScript + Tailwind CSS. React Router for pages.
- **Supabase Edge Function** called `ai` that makes all LLM calls. **Never expose API keys in the browser.**
- Provider-agnostic LLM call inside the edge function, controlled by secrets:
  - `LLM_PROVIDER` = `anthropic` or `gemini`
  - `ANTHROPIC_API_KEY` / `GEMINI_API_KEY`
  - `MODEL_ID` (model name string, set by me — do not hard-code a model)
- The `ai` function accepts `{ mode: "review" | "draft" | "skill" | "workflow_step" | "orchestrate" | "research_synthesis", systemPrompt?, payload }` so every later feature reuses one function.
- If no key is configured, or the call fails, fall back to **Demo Mode** (pre-built outputs) and show a small "Demo mode" badge. The app must never show a broken state in front of a client.
- Client-side file parsing: PDF via `pdfjs-dist`, DOCX via `mammoth`, plus paste-text option. Only extracted text is sent to the edge function.
- Export: DOCX via the `docx` package, PDF via `jspdf` (or print-to-PDF stylesheet).
- Guardrails for limited API credit: reject inputs over 60,000 characters with a friendly message; disable the submit button while a request is running; max **40 AI requests per browser session** (counter in memory; each workflow step counts as one) with a polite limit message.
- No document storage. State lives in memory only. State this in the UI ("Documents are processed in-session and not stored").

## 2. Design
- Editorial, understated, "Inns of Court" feel — not startup-neon.
- Palette: deep navy `#14213D`, ivory `#F7F4EC`, oxblood accent `#7A1E2C`, muted gold `#B08D57` for small details, slate greys for text.
- Fonts (Google Fonts): **EB Garamond** for headings, **Inter** for UI/body.
- Generous whitespace, thin rules, small caps labels, subtle card shadows. Fully responsive (works on an iPad, which barristers use).
- Light mode by default; tasteful dark mode toggle.
- British English everywhere (organise, licence (noun), defence, judgment), £, dates DD Month YYYY.

## 3. Pages and navigation
Top nav: **Home · Ask the CLO · Contract Review · Drafting Studio · Skills · Workflows · Research · Showcase · About**. (Pages not built in this PART should exist as tasteful "Coming next" placeholders so routing works.)

1. **Home** — hero with tagline; a stats strip showing live counts read from the registries (do NOT hard-code numbers): "1 Virtual Chief Legal Officer · 9 Specialist Virtual Lawyers · 3 Jurisdiction Counsel · N Specialist Legal Skills · 10 Workflows · N Official Research Sources". Then large tiles: "Ask the CLO", "Review a contract", "Draft a document", "Run a workflow", "Research the law". A 3-step "How it works", a "Built for English law" strip (UCTA 1977 · CPR · Pre-Action Conduct · UK GDPR), trust section ("Draft for counsel review — not legal advice", "No documents stored", "You stay in control"), CTA "Book a walkthrough" (mailto placeholder).
2. **Contract Review** (`/review`) — see §4.
3. **Drafting Studio** (`/draft`) — see §5.
4. **About** — product story, who it's for (barristers, chambers, direct access work, solicitors instructing counsel), roadmap teaser (skeleton arguments, bundle indexing) marked "Coming soon".
- Global header: serif wordmark "TIQ-VirtualCLO", nav, "Demo mode" badge when active, jurisdiction selector chip (default **England & Wales**).
- Global footer: disclaimer + "A TIQ Digital product" + "Credits" link.

## 4. Contract Review
**Input panel**
- Upload PDF/DOCX or paste text.
- Selectors: "Acting for" (Party A / Party B / Neutral — auto-fill party names after parsing if detectable), "Contract type" (Commercial services, SaaS, NDA, Supply of goods, Consultancy, Other), "Counterparty type" (Business / Consumer).
- "Try a sample" buttons loading the built-in samples (§6).

**Output (render from structured JSON)**
- Header card: overall risk rating (Low / Medium / High / Critical) with a 0–100 score dial, one-paragraph executive summary, governing law & jurisdiction detected.
- **Issues table**, sortable, filterable by severity: Severity (🔴 Critical / 🟠 High / 🟡 Medium / 🟢 Low) · Clause ref · Short quote from the contract (max 30 words) · Issue · English-law point · Suggested redline · Negotiation fallback.
- **Missing clauses** list.
- "Questions for the client" list.
- Buttons: Export review as DOCX/PDF, Copy summary, **"Continue in Contract Review & Negotiation workflow"** (links to the workflow in PART 3 with this document preloaded).
- Collapsible "Show reasoning basis" listing the statutes/principles relied on.

**Edge function — review system prompt (use verbatim):**
```
You are a senior commercial practitioner in England and Wales assisting a barrister. You review contracts governed (or likely governed) by English law and produce work-product for counsel to check. You are not giving legal advice to a lay client.

Method:
1. Identify parties, contract type, B2B vs B2C, governing law and jurisdiction. If the governing law is not English law, say so prominently and limit analysis to flagging the issue.
2. Review from the perspective of the party specified ("acting for").
3. Check at minimum: exclusion and limitation of liability (Unfair Contract Terms Act 1977 – reasonableness test, s.2(1) bar on excluding liability for death/personal injury caused by negligence; Consumer Rights Act 2015 if a consumer is party); liquidated damages and the penalty rule (Cavendish Square Holding BV v Makdessi [2015] UKSC 67 – legitimate interest test); indemnities (scope, caps, conduct of claims); payment terms and late payment (Late Payment of Commercial Debts (Interest) Act 1998); termination rights and consequences; IP ownership and licences; confidentiality; data protection (UK GDPR and Data Protection Act 2018); force majeure; assignment and subcontracting; entire agreement and non-reliance; third-party rights (Contracts (Rights of Third Parties) Act 1999); dispute resolution, governing law and jurisdiction; notices; variation; limitation periods (Limitation Act 1980: 6 years simple contract, 12 years deed) where relevant.
4. For each issue give: severity, clause reference, a SHORT verbatim quote (max 30 words), the problem, the English-law point, suggested replacement wording, and a fallback position.

Rules:
- Use British English and UK legal terminology.
- Only cite authorities from this list: UCTA 1977, Consumer Rights Act 2015, Cavendish v Makdessi [2015] UKSC 67, Late Payment of Commercial Debts (Interest) Act 1998, Contracts (Rights of Third Parties) Act 1999, UK GDPR, Data Protection Act 2018, Limitation Act 1980, Civil Procedure Rules. For any other point, state the principle without inventing a case name or citation. Never fabricate citations.
- Separate facts in the document from assumptions. List questions where facts are missing.
- Be concise and practical. No generic disclaimers inside the analysis.

Return ONLY valid JSON matching this schema:
{
  "parties": [{"name": "", "role": ""}],
  "contract_type": "",
  "b2b_or_b2c": "B2B|B2C|Unclear",
  "governing_law": "",
  "jurisdiction": "",
  "overall_risk": "Low|Medium|High|Critical",
  "risk_score": 0,
  "executive_summary": "",
  "issues": [{"severity": "Critical|High|Medium|Low", "clause_ref": "", "quote": "", "issue": "", "law_point": "", "suggested_redline": "", "fallback": ""}],
  "missing_clauses": [""],
  "client_questions": [""],
  "authorities_relied_on": [""]
}
```
- Validate the JSON in the edge function; if invalid, retry once asking the model to return valid JSON only; if still invalid, fall back to Demo Mode output for that sample or show a graceful error.

## 5. Drafting Studio
Left: document type picker + dynamic form. Right: live, editable draft (rich-text area) with Export DOCX / PDF / Copy. A "Regenerate" button and a "Tone" toggle (Firm / Conciliatory).

**Document types and form fields**
1. **Letter Before Claim** (Practice Direction – Pre-Action Conduct and Protocols; where a business claims a debt from an individual, use the Pre-Action Protocol for Debt Claims and note the enclosures it requires). Fields: sender, recipient (business/individual), claim type (debt / breach of contract / other), amount claimed (£), key facts, documents relied on, remedy sought, response deadline (default 14 days; 30 days for Debt Protocol), ADR offered (yes/no).
2. **Non-Disclosure Agreement** — mutual / one-way, parties, purpose, term, governing law (default England and Wales), jurisdiction.
3. **Commercial Settlement Agreement** — parties, dispute summary, settlement sum (£), payment terms, full and final settlement scope, confidentiality, no admission of liability, governing law. (Show a note: employment settlement agreements need independent legal advice under s.203 Employment Rights Act 1996 — out of scope for this template.)
4. **Part 36 Offer Letter** (CPR Part 36) — offeror (claimant/defendant), offeree, claim reference, whole or part of claim, counterclaim taken into account (yes/no), amount (£), relevant period (minimum 21 days). The letter must state it is made pursuant to Part 36, state the relevant period, and say whether it relates to the whole claim or part, and whether it takes account of any counterclaim.
5. **Services Agreement (short form)** — supplier, customer, services, fees, payment terms, term & termination, liability cap, IP, governing law.
- Add a link "More drafting skills →" to the Skills page filtered to drafting skills (PART 2).

**Edge function — drafting system prompt (use verbatim, append the form data as JSON):**
```
You draft documents for use in England and Wales, for review by a barrister or solicitor before use. Produce a complete, professionally formatted draft in British English using UK legal terminology and conventions (e.g. "Letter Before Claim", "without prejudice save as to costs" where appropriate, £ amounts, dates as DD Month YYYY).

Rules:
- Use only the facts supplied. Where a needed fact is missing, insert a bracketed placeholder like [DATE OF CONTRACT] rather than inventing it.
- Follow the procedural requirements for the document type (Practice Direction – Pre-Action Conduct and Protocols, Pre-Action Protocol for Debt Claims, CPR Part 36) where relevant.
- Only cite: Civil Procedure Rules, Practice Direction – Pre-Action Conduct and Protocols, Pre-Action Protocol for Debt Claims, UCTA 1977, Late Payment of Commercial Debts (Interest) Act 1998, Contracts (Rights of Third Parties) Act 1999, UK GDPR, Data Protection Act 2018. Never invent case law or citations.
- Clear numbered clauses for agreements; standard letter structure for letters.
- After the draft, add a short section "Notes for counsel" (max 5 bullets) listing assumptions, placeholders to complete and points to verify.

Return ONLY JSON: {"title": "", "draft_markdown": "", "notes_for_counsel": [""]}
```

## 6. Demo Mode & samples (build these in)
Create two fictional sample contracts as text files bundled in the app (realistic, 1,200–2,000 words each, English law, fictional company names with "Ltd"):
- **Sample A — SaaS Services Agreement** between "Northbridge Analytics Ltd" (supplier) and "Harlow & Pike Logistics Ltd" (customer). Deliberately include: a blanket exclusion of all liability including negligence; liability cap of £500 regardless of fees; a £10,000-per-day late-delivery "liquidated damages" clause; supplier owns all customer data outputs; auto-renewal with 180-day notice; unilateral price variation; no data processing terms; governing law England and Wales but exclusive jurisdiction of the courts of New York.
- **Sample B — Mutual NDA** between "Ashcombe Biotech Ltd" and "Kestrel Ventures LLP". Deliberately include: confidentiality obligations with no time limit and no standard exclusions (public domain, prior knowledge, required by law); a one-sided injunction clause; a non-solicit hidden in the boilerplate; no return/destruction clause.
- For each sample, ship a **pre-built review JSON** (matching the §4 schema, 8–10 issues, accurate English-law points) and use it in Demo Mode so the demo runs instantly with zero API calls.
- Also ship one pre-built draft per document type in §5 for Demo Mode.

## 7. Compliance & trust copy (show in UI)
- Persistent footer disclaimer: "Outputs are draft work-product for review by a qualified legal professional. TIQ-VirtualCLO does not provide legal advice and does not create a lawyer–client relationship."
- Above every AI output: small label "AI-generated draft — verify before use".
- Privacy note on upload: "Text is processed in-session to generate your review and is not stored."

## 8. Quality bar
- No lorem ipsum anywhere. Real, sensible UK copy throughout.
- Loading states with a subtle progress message sequence ("Reading the contract…", "Checking liability provisions…", "Preparing redlines…").
- Empty states, error states and mobile layouts all designed.
- Clean component structure; put all system prompts in `/supabase/functions/ai/prompts.ts` so I can edit them later.
- After building, list the exact secrets I need to set in Supabase and how to switch between Anthropic and Gemini.
