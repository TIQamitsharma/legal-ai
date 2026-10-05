# PART 4 — Legal Research Sources & Connectors

Keep PARTS 1–3 unchanged. Build `/research` and `/connectors`, plus a Supabase Edge Function `research` that proxies all external calls (no keys or CORS issues in the browser).

## 4.1 Sources (define in `sources.ts`; status shown on `/connectors`)
**United Kingdom (live)**
1. **legislation.gov.uk** — UK statutes and statutory instruments. Public, no key. Use its documented search and data feeds (e.g. `https://www.legislation.gov.uk/{type}/{year}/{number}/data.xml`, Atom feeds for search). Verify endpoints against the site's API documentation.
2. **The National Archives — Find Case Law** — judgments of the UK courts and tribunals. Public, no key. Use its documented Atom search feed (`https://caselaw.nationalarchives.gov.uk/atom.xml?query=...`); verify the endpoint and respect the Open Justice Licence (link to it on /connectors; note that bulk computational analysis may need a separate licence).
3. **Companies House** — company profiles, officers, filings, PSCs, charges. Free API key (secret `COMPANIES_HOUSE_API_KEY`, HTTP Basic auth with the key as username). Base: `https://api.company-information.service.gov.uk`.

**Global**
4. **GLEIF** — Legal Entity Identifier records. Public, no key. `https://api.gleif.org/api/v1/lei-records`.

**United States**
5. **CourtListener** — case law and dockets. Free token (secret `COURTLISTENER_TOKEN`). `https://www.courtlistener.com/api/rest/v4/search/`.
6. **GovInfo** — federal legislative and regulatory publications. Free key (secret `GOVINFO_API_KEY`).
7. **Federal Register** — federal rules and notices. Public. `https://www.federalregister.gov/api/v1/documents.json`.
8. **SEC EDGAR** — company filings. Public; requests must send a descriptive `User-Agent` header with a contact email (secret `SEC_USER_AGENT`).
9. **Regulations.gov** — rulemaking dockets and comments. Free key (secret `REGULATIONS_GOV_API_KEY`).

**India (routing, link-out only)**
10. India Code, eGazette, Supreme Court of India, High Courts and eCourts — no API calls; show as "Official source routing" that opens the official site search in a new tab.

- `/connectors` page: a card per source with region flag, what it covers, auth needed, status pill (**Connected** = key present or no key needed and a health-check call succeeded; **Needs key**; **Link-out only**), and a "Test connection" button. The count on Home reads from this registry.
- If a source fails or has no key, the app must degrade gracefully ("Source unavailable — result not verified against this source") and never pretend a check happened.

## 4.2 Research page (`/research`)
- Inputs: research question, jurisdiction, source checkboxes (pre-ticked by jurisdiction: UK → legislation.gov.uk + Find Case Law + Companies House), optional company name.
- Flow: (1) `ai` plans 2–4 search queries from the question (legal-research-planner); (2) `research` function runs the queries against the selected sources and returns titles, citations/neutral citations, dates and URLs; (3) `ai` in `research_synthesis` mode writes a memo **grounded only in the retrieved results**.
- Synthesis system prompt:
```
You are Legal Research Counsel. Write a short research memo answering the question using ONLY the retrieved sources listed below. Cite each proposition with the source's title, neutral citation or legislation reference, and URL, exactly as provided. If the retrieved sources do not answer a point, say "Not found in retrieved sources — verify independently" rather than relying on memory. Never invent citations. British English for UK matters.
Return ONLY JSON: {"answer_summary": "", "memo_markdown": "", "sources_used": [{"title": "", "citation": "", "url": ""}], "gaps": [""]}
```
- Output: memo, a **Sources panel** (clickable cards opening the official page), a "Gaps / verify independently" box, export DOCX/PDF.
- **Company check widget**: name → Companies House (UK) + GLEIF results: status, incorporation date, registered office, officers, PSCs, charges count. Usable inside workflows 2, 6 and 9 as an extra "Registry check" step.
- **Citation check**: paste a draft → extract citations → look each up on Find Case Law / legislation.gov.uk → mark Found / Not found / Partial (feeds citation-integrity-checker).
- Skills that need authority (case-law-analyst, precedent-mapper, statutory-interpreter, authority-validator, legal-opinion-drafter, written-submissions-drafter) get a toggle **"Ground with live sources"** that runs the research flow first and injects results into the skill prompt.
- Demo Mode: ship pre-recorded research results for the three showcase research prompts in PART 5 so they work offline.
