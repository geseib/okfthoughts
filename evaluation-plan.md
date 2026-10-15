# Evaluation Plan: OKF vs. RAG for Control Retrieval in Regulated Environments

Oct 10, 2026 · George Seib

This plan compares the recall and precision of Google's Open Knowledge Format (OKF) with retrieval-augmented generation (RAG) approaches for identifying the standards and controls that apply to a proposed architecture in a regulated environment. For the results to be credible, the study needs an expert-verified answer key of the controls that apply to each architecture, built independently of every system under test, a comparison against the strongest alternative rather than a naive baseline, and paired statistical tests with stated limitations.

## Research question and hypotheses

For a given architecture component, which retrieval approach returns the complete set of applicable controls with the fewest false positives? The primary measure is recall of mandatory controls; precision is secondary.

The following hypotheses will be committed to the repository before any system is run:

- **H1:** OKF metadata filtering with link traversal achieves higher recall of mandatory controls than chunked vector RAG.
- **H2:** OKF does not outperform vector RAG with equivalent metadata filters on recall, because both rely on the same tags. Stating this in advance shows the study is designed to test the approach, not promote it.
- **H3:** Explicit links improve completeness by retrieving the approved reference pattern and enforcement rule alongside each control.
- **H4:** OKF's advantage degrades as metadata tagging accuracy declines.

## Corpus

The corpus combines public control catalogs with a synthetic set that models firm-specific layering. The synthetic set is identified as such in all results.

| Source | Contribution | Format |
| --- | --- | --- |
| [FINOS Common Cloud Controls](https://ccc.finos.org/architecture) | Financial-services controls mapped to cloud service types, each with testable assessment requirements | YAML |
| [NIST SP 800-53 Rev. 5 (OSCAL)](https://pages.nist.gov/OSCAL/learn/concepts/layer/control/catalog/sp800-53rev5-example), from [usnistgov/oscal-content](https://github.com/usnistgov/oscal-content) | Authoritative control catalog in machine-readable form; a subset of families (SC, AU, AC) keeps labeling tractable | JSON |
| Synthetic enterprise layer | Approximately 40 firm, division and team requirements, including superseded versions, refinements and time-bound exceptions not represented in public catalogs | Markdown |

All configurations ingest identical source text. The OKF configuration also receives frontmatter metadata, and the effort required to author it is reported as a cost of the approach.

## Evaluation set and answer key

The evaluation set comprises 60 to 100 reference architectures expressed as FINOS CALM models, varied by component type, data classification, jurisdiction and owning business unit. It deliberately includes:

- Controls that apply but share no vocabulary with the architecture description
- Near-miss components, such as object storage holding only public data
- Superseded controls that remain textually similar to their replacements
- Division- and team-level refinements and approved exceptions

Each architecture needs an answer key: the list of controls that actually apply to it, verified by experts. Every system is scored against this key, and a flawed key is the most common reason published comparisons fail. The key is built as follows:

- Two subject-matter experts independently identify the applicable controls for each architecture from the source standards, before any system output is available.
- Inter-rater agreement is reported as Cohen's kappa, with a target of 0.7 or higher; a third reviewer adjudicates disagreements.
- The key is derived from the control text, not from OKF metadata. Deriving it from the same tags OKF filters on would bias the results in its favor.

## Systems under test and ablations

All configurations use the same language model, CALM input and token budget.

| ID | Configuration | Purpose |
| --- | --- | --- |
| A | Chunked vector RAG, top-k | Baseline |
| B | Hybrid BM25 and vector retrieval with reranking | Strong text-retrieval baseline |
| C | Vector RAG with metadata filters equivalent to OKF tags | Strongest alternative |
| D | GraphRAG over source text (Microsoft GraphRAG or LightRAG) | LLM-extracted knowledge graph |
| E | OKF metadata filtering and link traversal over a SQLite index | Curated knowledge graph |
| F | Configuration E with semantic fallback | Proposed architecture |

Ablation studies isolate the contribution of each component:

- OKF without link traversal
- OKF with unreviewed, LLM-generated metadata, quantifying the value of expert review
- Metadata with 5%, 10% and 20% injected tagging errors
- LLM-classified CALM components in place of manually classified ones, measuring sensitivity to classification error

## Metrics and statistics

| Category | Measure |
| --- | --- |
| Primary | Recall of mandatory controls; proportion of architectures with at least one missed mandatory control, the measure most relevant to risk and audit stakeholders |
| Secondary | Precision; retrieval of superseded controls; correct application of division and team refinements; co-retrieval of reference pattern and enforcement rule; consistency across five repeated runs (Jaccard similarity) |
| Cost | Query latency; token consumption; metadata authoring effort per 100 controls |
| End-to-end (optional) | Policy-check failures, evaluated with OPA or Cedar, when each configuration's output is used to generate the CALM model |

Statistical method:

- Paired comparisons per architecture, using the Wilcoxon signed-rank test for recall and McNemar's test for the binary missed-control outcome
- 95% bootstrap confidence intervals for all reported metrics
- Repeated runs for non-deterministic configurations, with variance reported
- Holm correction for multiple comparisons

## Blog structure

1. Problem statement, illustrated with a concrete missed control
2. Pre-registered hypotheses, including H2
3. Methodology: corpus, evaluation set, labeling protocol and inter-rater agreement
4. Results: recall with confidence intervals, missed-control analysis and cost comparison
5. Cases where OKF underperformed or matched the alternatives
6. Threats to validity: synthetic data, sample size, author-produced labels, single-model evaluation
7. Reproducibility: repository, data, pinned dependency versions and a single-command run

If OKF matches configuration C on recall but leads on co-retrieval and auditability, the write-up should say so. A balanced result is more credible to a regulated audience than a clean sweep.

## Next steps

- [ ] Build the evaluation repository: corpus loaders, CALM architecture template, labeling workbook, configurations A, C and E, and scoring harness
- [ ] Commit the hypotheses before the first run
- [ ] Engage a second subject-matter expert for labeling

## Sources

- [FINOS Common Cloud Controls architecture](https://ccc.finos.org/architecture)
- [OSCAL SP 800-53 Rev. 5 catalog example](https://pages.nist.gov/OSCAL/learn/concepts/layer/control/catalog/sp800-53rev5-example)
- [usnistgov/oscal-content](https://github.com/usnistgov/oscal-content)
