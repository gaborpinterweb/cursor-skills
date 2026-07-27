# Checklist overview (10 areas)

The expandable **rules** live in [rules.md](rules.md).  
These 10 areas are the taxonomy used in evidence and handoff mapping.

1. AI crawler access  
2. Answer-shaped content  
3. Extractable structure  
4. FAQ / HowTo schema  
5. Entity & brand clarity  
6. E-E-A-T / trust signals  
7. Freshness  
8. Citation readiness  
9. Comparison & decision content  
10. Attachments & measurement  

Area 10 is attachment-driven (Answerlint JSON/CSV, optional `llms lint`, optional prompt-pack CSV). Areas 4 and 9 are often `n/a` when page intent does not match.

## Answerlint blind spots (always verify in code/repo)

- AI bot `robots.txt` rules beyond what Answerlint surfaces  
- Multi-page `llms.txt` coverage and sitemap alignment  
- JS-rendered content (audit built HTML / SSG output)  
- Live citation outcomes without a prompt pack  
