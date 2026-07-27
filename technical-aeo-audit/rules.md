# Technical AEO rules

Evaluate **every** rule. Map Answerlint JSON/CSV findings into matching rules when attached. Group under the 10 checklist areas.

Statuses: `pass` | `partial` | `fail` | `n/a` | `unknown`  
Priorities: `High` | `Medium` | `Low`

Primary free attachment: [Answerlint](https://www.npmjs.com/package/answerlint) (`answerlint audit` JSON/CSV; optional `answerlint llms lint`).

---

## 1. AI crawler access

| ID | Priority | Rule |
|----|----------|------|
| B-01 | High | `robots.txt` exists in production / build output |
| B-02 | High | Search/citation bots are not blanket-blocked (`OAI-SearchBot`, `PerplexityBot`, `Claude-SearchBot` / `ClaudeBot` as applicable) |
| B-03 | Medium | Training vs search agents are considered deliberately (e.g. GPTBot vs OAI-SearchBot) — document intent if training is disallowed |
| B-04 | Medium | Important content paths are not Disallow’d for AI search agents |
| B-05 | Low | `Sitemap:` is declared in `robots.txt` (helps all crawlers, including AI) |

**Answerlint / attachments:** crawler-access style checks if present; otherwise verify `robots.txt` in repo.

## 2. Answer-shaped content

| ID | Priority | Rule |
|----|----------|------|
| A-01 | High | Key pages lead with a direct, self-contained answer in the opening (BLUF / first paragraph) |
| A-02 | High | Primary headings include natural-language questions users ask (where intent is informational) |
| A-03 | Medium | FAQ or Q&A blocks exist on pages that answer multiple related questions |
| A-04 | Medium | Answers are concise enough to lift (roughly 40–80 words for the lead answer) without burying the point |
| A-05 | Low | Soft marketing fluff does not precede the factual answer on money/docs pages |

**Answerlint:** direct_answer, Q&A density (and similar).

## 3. Extractable structure

| ID | Priority | Rule |
|----|----------|------|
| H-01 | High | One clear primary `h1` per page type |
| H-02 | High | Heading hierarchy is logical (H1→H2→H3; no chaotic skips sitewide) |
| H-03 | Medium | Passages are self-contained (quotable without missing antecedents) |
| H-04 | Medium | Lists/tables used where comparisons or steps are clearer than prose |
| H-05 | Low | Boilerplate repeated H2s do not drown topical headings |

**Answerlint:** readability / extractability / structure-style checks.

## 4. FAQ / HowTo schema

| ID | Priority | Rule |
|----|----------|------|
| F-01 | High | If the page has visible FAQ/Q&A: `FAQPage` JSON-LD present and valid — else `n/a` |
| F-02 | Medium | If the page is a procedure: `HowTo` JSON-LD present when steps are shown — else `n/a` |
| F-03 | Medium | Structured data matches visible Q&A / steps (no hidden-only markup) |
| F-04 | Low | Article/WebPage schema does not pretend to be FAQ when no Q&A exists |

**Answerlint:** faq_schema / HowTo-related checks. Prefer `n/a` over fail when intent is not Q&A/howto.

## 5. Entity & brand clarity

| ID | Priority | Rule |
|----|----------|------|
| N-01 | High | Brand / product / org name is clear and consistent on key templates |
| N-02 | High | `Organization` and/or `WebSite` JSON-LD (or equivalent) present sitewide |
| N-03 | Medium | Named entities in copy are explicit (expand acronyms on first use) |
| N-04 | Medium | `Person` / author entity linked where authorship is claimed |
| N-05 | Low | SameAs / official profile URLs present when brand has public profiles |

**Answerlint:** named entities / entity clarity.

## 6. E-E-A-T / trust signals

| ID | Priority | Rule |
|----|----------|------|
| T-01 | High | Author byline (or org attribution) on article/docs templates |
| T-02 | Medium | Author credentials / role visible or linked where expertise matters |
| T-03 | Medium | About / contact (or equivalent trust pages) linked from global chrome |
| T-04 | Medium | Claims that need proof are not anonymous (“studies show” without source) |
| T-05 | Low | NAP / business identity consistent when local or company entity matters — else `n/a` |

**Answerlint:** author byline / trust signals.

## 7. Freshness

| ID | Priority | Rule |
|----|----------|------|
| D-01 | High | Visible publish and/or updated date on time-sensitive content |
| D-02 | Medium | `datePublished` / `dateModified` in schema when Article/BlogPosting (or similar) is used |
| D-03 | Medium | Cornerstone pages show evidence of periodic refresh (not multi-year stale facts) |
| D-04 | Low | Evergreen pages without dates are acceptable if intentionally timeless — document as `pass` with evidence |

**Answerlint:** content freshness.

## 8. Citation readiness

| ID | Priority | Rule |
|----|----------|------|
| C-01 | High | Key claims include verifiable statistics, definitions, or concrete facts |
| C-02 | High | Outbound citations / proof links to primary or reputable sources where claims need support |
| C-03 | Medium | Quotable expert or source attributions appear where authority helps (without keyword stuffing) |
| C-04 | Medium | Original data, examples, or first-hand detail present on flagship pages |
| C-05 | Low | Citation style is consistent and link targets resolve |

**Answerlint:** citation likelihood / external citations / evidence-quality style checks. Strongest research-backed content area (GEO: statistics, quotes, cite sources).

## 9. Comparison & decision content

| ID | Priority | Rule |
|----|----------|------|
| V-01 | High | If page targets “best / vs / which” intent: clear criteria and comparison structure — else `n/a` |
| V-02 | Medium | Comparison tables or explicit trade-offs present for decision pages — else `n/a` |
| V-03 | Medium | Alternatives / competitors named fairly when the page is comparative — else `n/a` |
| V-04 | Low | Decision pages state who the recommendation is for (audience fit) |

**Answerlint:** comparison content. Prefer `n/a` on pure docs/blog that are not comparative.

## 10. Attachments & measurement

Evaluate when Answerlint / prompt-pack / `llms lint` artifacts are attached; otherwise `unknown` or `n/a` as noted.

| ID | Priority | Rule |
|----|----------|------|
| X-01 | High | If Answerlint JSON/CSV attached: fail/warn checks mapped into areas 1–9 (none left unexplained) |
| X-02 | High | If Answerlint attached: composite / AEO / GEO scores recorded in Evidence for summary context |
| X-03 | Medium | If `llms.txt` exists or was requested: `answerlint llms lint` (or equivalent) findings addressed — else `n/a` |
| X-04 | Medium | Optional prompt-pack CSV: priority prompts show mention/citation status — else `unknown` if user wanted visibility proof, else `n/a` |
| X-05 | Low | Answerlint diff / CI threshold regressions noted when before/after reports are attached |

**Answerlint:** primary machine evidence (like Screaming Frog for SEO). Prompt packs are optional outcome evidence.

---

## Counts

- Checklist areas 1–9: **42** rules (B–V)  
- Attachment extras: **5** rules (X)  
- **Total: 47 rules**

Always evaluate all rows. Use `n/a` / `unknown` rather than skipping.
