# Citation Verification — Reference

How a research-orchestrator subagent proves EVERY reference in a paper is real (not
hallucinated) and that the cited claim is actually supported. Gold standard: **download
the real PDF and check the claim against its text.** References are guilty until verified.

---

## 1. Why this is mandatory

LLM-generated reference lists fabricate citations at high rates, and fabrications "look
legitimate at first glance" (real-sounding authors, well-formatted DOIs, real journals).

- Walters & Wilder (2023), *Scientific Reports*: **~55% of GPT-3.5 citations and ~18% of
  GPT-4 citations were fabricated**; among the *real* ones, **43% (GPT-3.5) / 24% (GPT-4)
  contained substantive errors**. Verification used multi-database lookup (Google, PubMed,
  Scopus, WorldCat, publisher sites).
  - https://pmc.ncbi.nlm.nih.gov/articles/PMC10484980/
  - https://www.nature.com/articles/s41598-023-41032-5
- Cross-study aggregate: across 6 studies, **51% of 732 citations fabricated** (range
  47–69%); a psychology study found **32.3% of 300 hallucinated**; a GPT-4o mental-health
  study found **~1 in 5 fabricated, 56% fake-or-erroneous**.
  - https://www.psypost.org/chatgpt-hallucinates-fake-but-plausible-scientific-citations-at-a-staggering-rate-study-finds/
  - https://studyfinds.org/chatgpts-hallucination-problem-fabricated-references/

**Core consequence:** a present/valid DOI is NOT sufficient — it may resolve to a real
article whose title/authors/subject differ from the claimed reference. So every reference
must pass: (a) existence, (b) fuzzy metadata match, (c) claim-vs-full-text verification.

---

## 2. The pipeline (per reference)

### 2a. RESOLVE to a canonical paper
Use the most specific identifier available; fall back to bibliographic search.

- **DOI content negotiation (doi.org)** — confirms a DOI resolves to metadata:
  `curl -LH "Accept: application/vnd.citationstyles.csl+json" https://doi.org/{DOI}` →
  CSL-JSON (title, author, issued year, container-title). Also `application/x-bibtex`,
  `application/vnd.crossref.unixsd+xml`. A 404/non-resolution = strong fabrication signal.
  - https://citation.doi.org/docs.html
- **Crossref REST** — primary bibliographic resolver. No key needed.
  - Bib search: `https://api.crossref.org/works?query.bibliographic=<terms>&rows=5`
  - DOI lookup: `https://api.crossref.org/works/{doi}`
  - Polite pool: add `&mailto=you@example.org` (or `mailto:` in `User-Agent`). Rate limits
    via `X-Rate-Limit-Limit` / `X-Rate-Limit-Interval` headers.
  - https://github.com/CrossRef/rest-api-doc
- **arXiv API** — for preprints. Be polite: **3-second delay** between calls.
  - Base: `http://export.arxiv.org/api/query`
  - Title: `?search_query=ti:<terms>&max_results=5` · Id: `?id_list=2103.00020`
  - https://info.arxiv.org/help/api/user-manual.html
- **Semantic Scholar Graph API**:
  - Best single match: `GET https://api.semanticscholar.org/graph/v1/paper/search/match?query=<title>`
  - Relevance: `GET .../graph/v1/paper/search?query=<terms>&fields=title,authors,year,externalIds,openAccessPdf&limit=5`
  - By id: `GET .../graph/v1/paper/{paper_id}?fields=title,authors,year,externalIds,openAccessPdf,abstract`
    (`paper_id` may be S2 id, `DOI:...`, `ARXIV:...`)
  - Rate limits: unauthenticated share **5,000 req / 5 min**; free key gives 1 req/s on
    search/batch/recommendations, 10 req/s otherwise.
  - https://www.semanticscholar.org/product/api/tutorial · https://api.semanticscholar.org/api-docs/graph
- **OpenAlex** — metadata search:
  - Base: `https://api.openalex.org` · Full-text: `GET /works?search=<terms>`
  - DOI as entity id: `GET /works/https://doi.org/{DOI}`; filter example `GET /works?filter=publication_year:2024`
  - Polite pool: `?mailto=you@example.org`; optional free key at https://openalex.org/settings/api
  - https://developers.openalex.org/ · https://developers.openalex.org/api-reference/introduction

**Resolution order:** DOI → doi.org content-neg + Crossref `/works/{doi}`. arXiv id →
arXiv API. Free text → Crossref `query.bibliographic` + S2 `search/match` + OpenAlex
`search`, then cross-confirm.

### 2b. EXISTENCE / METADATA gate (the anti-hallucination gate)
- **Title fuzzy-match** returned vs claimed: token-set / Levenshtein ratio (e.g.
  `rapidfuzz.fuzz.token_set_ratio`); ≥~90 = match, 70–90 = inspect, <70 = no match.
- **Author surname overlap**: at least the first author's surname should appear.
- **Year**: claimed year within ±1 of canonical `published`/`year` (preprint slack).
- **No close match in ANY source** (Crossref + OpenAlex + S2 + arXiv) → **NOT-FOUND
  (likely fabricated)** — the core fabrication detector, per Walters & Wilder.
- DOI claimed but does not resolve (2a doi.org) → fabricated DOI, even if a similarly
  titled paper exists. DOI resolves but its CSL-JSON title/authors differ → **METADATA-MISMATCH**.

### 2c. DOWNLOAD the OA PDF (open-access routes only; stop at first working PDF)
1. **arXiv** (if arXiv id): `https://arxiv.org/pdf/{arxiv_id}`
2. **Unpaywall** (by DOI): `GET https://api.unpaywall.org/v2/{DOI}?email=you@example.org`
   — required `email`; response has `is_oa`, `best_oa_location.url_for_pdf`. **100,000
   calls/day**, no key. https://unpaywall.org/products/api ·
   https://support.unpaywall.org/support/solutions/articles/44002142311
3. **Semantic Scholar** `openAccessPdf` `{url, status}` via `fields=openAccessPdf`.
4. **OpenAlex** `best_oa_location.pdf_url` / `open_access.oa_url` (also `is_oa`, `oa_status`).

Validate magic bytes `%PDF` before parsing. Set a descriptive User-Agent + mailto. Never
bypass paywalls — if no OA PDF, mark **EXISTS-BUT-UNCHECKED** (never silently → VERIFIED).

### 2d. EXTRACT text
- `pdftotext paper.pdf out.txt` (Poppler) — fast for clean digital PDFs.
- PyMuPDF (`import fitz; "\n".join(pg.get_text() for pg in fitz.open(p))`) — robust, per-page.
- `pdfplumber` — when tables/positions matter.
- Empty text (scanned/image PDF) → OCR fallback (`ocrmypdf`/Tesseract), flag lower confidence.

### 2e. CLAIM-MATCH (does the paper actually say that?)
- **Keyword/section search**: pull salient terms (numbers, method/dataset names, the
  specific quantity) from the citing sentence; regex/windowed search the extracted text.
- **Semantic check**: chunk + embed text and the claim (sentence-transformers), retrieve
  top-k; optionally pass claim + chunks to an LLM with a strict "is this supported? quote
  the sentence or say NOT SUPPORTED" prompt.
- Supporting passage found → **VERIFIED**. Competent search but no support → **CLAIM-UNSUPPORTED**.

---

## 3. Failure taxonomy (exactly one label per reference)

| Label | Definition | Reached when |
|---|---|---|
| **VERIFIED** | Exists + OA PDF obtained + attributed claim supported by full text | 2e finds a supporting passage |
| **EXISTS-BUT-UNCHECKED** | Real paper, metadata matches, but no legal OA PDF (paywalled) — content not auto-verifiable; **report honestly** | 2c yields no PDF |
| **METADATA-MISMATCH** | A real paper exists but author/year/venue (or the DOI's target) differs from the claimed reference | 2b partial match / DOI resolves to a different paper |
| **CLAIM-UNSUPPORTED** | Real paper, PDF obtained, but the text does not support the attributed claim | 2e finds no supporting passage |
| **NOT-FOUND** | No match in Crossref, OpenAlex, S2, or arXiv → likely fabricated | 2b all-source miss / DOI fails to resolve |

Grounded in the hallucination literature: fabricated (→ NOT-FOUND) and erroneous-but-real
(→ METADATA-MISMATCH / CLAIM-UNSUPPORTED) are the two distinct failure classes both
quantified by Walters & Wilder (https://pmc.ncbi.nlm.nih.gov/articles/PMC10484980/).

---

## 4. Wiring to existing skills + the gap

| Step | Skill to use | Notes |
|---|---|---|
| 2c download (arXiv) | **`aris-arxiv`** | `arxiv_fetch.py download <id>`; only skill with a robust self-contained downloader + >10 KB sanity check. |
| 2b existence + metadata + claim-context | **`aris-citation-audit`** | Strongest verifier; per-use SUPPORTS/WEAK/WRONG verdict via DBLP/arXiv/ACL/OpenReview. **The only skill that checks the cited claim** — but it does so via **web lookups, NOT the downloaded PDF**. |
| 2a/2b cheap existence pre-gate | **`flonat-bib-validate --verify-doi`** | Deterministic DOI→Crossref + OpenAlex resolution to catch fabricated entries before spending reviewer budget. |

**THE GAP (orchestrator must fill it):** No existing skill closes the loop
**"download the PDF → verify the claim against THAT downloaded text."** `aris-citation-audit`
verifies claims via live web lookups, not by grounding in the actual full text — there is no
"extract PDF text → locate/quote the supporting sentence" step. The orchestrator fills this
with steps **2c–2e** above.

**Also:** non-arXiv OA download needs **Unpaywall** (2c #2) — `aris-openalex` /
`aris-semantic-scholar` only surface the OA PDF URL and never fetch it; no skill wraps
Unpaywall. (`scholar-citation-verification` is prose-only, no automation.)

---

## 5. The HARD RULE (verification ledger)

Maintain a verification ledger: one row per reference with its label from §3.

- A paper **may not be submitted**, and a claim **may not be stated**, while any
  **load-bearing** reference is **NOT-FOUND**, **METADATA-MISMATCH**, or **CLAIM-UNSUPPORTED**.
- **EXISTS-BUT-UNCHECKED** references must be **flagged to the user** (paywalled, no OA PDF —
  existence confirmed but the claim could not be auto-verified). Never silently upgrade to VERIFIED.
- Submission/claim is gated on the ledger: all load-bearing references VERIFIED, with any
  EXISTS-BUT-UNCHECKED explicitly surfaced for human sign-off.

### Engineering notes
- Add `mailto`/email to Crossref, OpenAlex, Unpaywall for polite-pool access; get free keys
  for OpenAlex and S2 for volume. Respect each API's rate limit.
- Cache by DOI/arXiv-id (responses change rarely) to stay under daily caps.
- Only ever fetch OA PDFs via the sanctioned OA URLs above — never bypass paywalls.
