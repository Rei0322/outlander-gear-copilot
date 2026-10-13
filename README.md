# Outlander Gear Co. Product Data Copilot

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23346733.svg)](https://doi.org/10.5281/zenodo.23346733)

An AI copilot that answers customer questions about Outlander Gear Co.'s outdoor products: prices, specs, warranties, and comparisons. It's built entirely in **Microsoft Foundry**. A gpt-4o agent answers questions, and every answer is grounded in the product catalog through an **Azure AI Search** knowledge base.

*Udacity Azure Generative AI Engineer Nanodegree, Project 2*

## Results at a glance

| | |
|---|---|
| Automated evaluation | 7 evaluators × 15 questions, **100% pass** on Similarity, Groundedness, TaskAdherence, Coherence, Fluency; **93%** on IntentResolution and Relevance |
| Answer accuracy (Similarity) | **4.73 / 5** average |
| Grounding | **5.00 / 5** average |
| Manual testing | **9 / 9** correct |
| Deployment | Running, Responses endpoint **Active** |

**Full write-up: [EVALUATION_REPORT.md](EVALUATION_REPORT.md)**

## How it works

1. **Data:** 20 product files (`product_info_1.md` to `product_info_20.md`) from the [Azure-Samples RAG repo](https://github.com/Azure-Samples/rag-data-openai-python-promptflow/tree/main/tutorial/data).
2. **Index:** Uploaded to a Foundry IQ knowledge base backed by Azure AI Search. The files were split into 326 chunks and embedded with `text-embedding-3-small`.
3. **Agent:** `outlander-product-copilot` (gpt-4o) has instructions to answer only from the knowledge base and to say when it doesn't know. Web search is turned off.
4. **Evaluate:** Automated evaluation in Foundry on a 15-question JSONL dataset, plus a 9-question manual review.
5. **Deploy:** Published with an active Responses API endpoint and a preview web app.

> Microsoft has replaced Prompt Flow with the Microsoft Agent Framework in the Foundry portal. The agent configuration screenshot stands in for the Prompt Flow screenshot.

## Deliverables

| Rubric item | Where to find it |
|---|---|
| Base model deployed | `screenshots/01_model_deployments.png` |
| Data uploaded and managed | `screenshots/02_knowledge_base.png` |
| AI Search index | `screenshots/03_ai_search_index.png` (326 docs in `ks-file-677-index`) |
| Copilot setup (Prompt Flow replacement) | `screenshots/04_agent_setup.png` |
| Copilot tested with relevant questions | `screenshots/05_samira_copilot_test_1.png` to `_4.png` |
| Evaluation configuration | `screenshots/06_eval_criteria.png` |
| Automated evaluation metrics | `screenshots/07_auto_eval_results_1.png`, `_2.png`, `evaluation/auto_eval_results_export.jsonl` |
| Evaluation dataset (JSONL) | `evaluation/outlander_eval_dataset_query.jsonl` (used in Foundry), `evaluation/outlander_eval_dataset.jsonl` (original course field names) |
| Manual evaluation with feedback | `evaluation/manual_evaluation.md` |
| Deployment with endpoint | `screenshots/09_deployment.png` |

## Repo layout

```
├── README.md
├── EVALUATION_REPORT.md
├── evaluation/
│   ├── outlander_eval_dataset_query.jsonl   # dataset used in Foundry (query / ground_truth)
│   ├── outlander_eval_dataset.jsonl         # same questions, course format (chat_input / truth)
│   ├── auto_eval_results_export.jsonl       # raw export from Foundry
│   ├── auto_eval_summary.json               # per-question scores
│   └── manual_evaluation.md
└── screenshots/
```

## Tech stack

Microsoft Foundry · Azure OpenAI (gpt-4o, text-embedding-3-small) · Azure AI Search · Foundry IQ knowledge bases · Foundry evaluations
