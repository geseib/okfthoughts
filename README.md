# OKF Thoughts

Notes on using Google's [Open Knowledge Format](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing) (OKF) to help agents retrieve the standards and controls that apply when they build architecture as code, and a plan for testing whether it beats RAG.

| File | What it is |
| --- | --- |
| [okf-standards-retrieval.md](okf-standards-retrieval.md) | How to model standards as an OKF bundle: frontmatter, links, severity, firm/division/team layering, and the agent loop |
| [evaluation-plan.md](evaluation-plan.md) | A pre-registered plan comparing the recall and precision of OKF with RAG and GraphRAG for control retrieval in a regulated environment |
| [okf-study-page.html](okf-study-page.html) | A self-contained study page that teaches the topic from the ground up; open it in a browser |

Requirement IDs, business units and examples are illustrative. OKF is at v0.1, and no published benchmark yet compares it with RAG for this use; the evaluation plan is meant to produce one.
