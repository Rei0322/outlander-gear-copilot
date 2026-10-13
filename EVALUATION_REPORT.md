# Evaluation Report: Outlander Gear Co. Product Data Copilot

**Author:** Samira Ray
**Course:** Udacity Azure Generative AI Engineer Nanodegree, Project 2
**Date:** October 13, 2026
**Platform:** Microsoft Foundry (new Foundry experience), France Central

---

## 1. Summary

I built a product copilot for Outlander Gear Co. that answers customer questions about prices, specs, warranties, and product comparisons using only the company's 20 product information files. The copilot is a Foundry agent running **gpt-4o** and grounded in an **Azure AI Search** knowledge base. It passed **every automated quality metric at 93% or higher** and answered **9 of 9 manually tested questions correctly**. It is deployed with an active Responses API endpoint.

| Area | Result |
|---|---|
| Automated evaluation (15 questions, 7 evaluators) | 100% pass on 5 evaluators, 93% on 2 |
| Response accuracy (Similarity vs. ground truth) | 100% pass, average **4.73 / 5** |
| Grounding in the product data | 100% pass, average **5.00 / 5** |
| Manual evaluation | **9 / 9** correct |
| Deployment | Running, endpoint active |

> **Note on Prompt Flow:** Microsoft has replaced Prompt Flow in the Foundry portal with the Microsoft Agent Framework, and the course instructions were updated to match. The agent configuration in `screenshots/04_agent_setup.png` stands in for the Prompt Flow screenshot. It shows the same pieces: model, system prompt, and the retrieval step over the indexed data.

---

## 2. Architecture

```
Customer question
      │
      ▼
outlander-product-copilot (Foundry agent, gpt-4o, version 3)
      │  tool call: knowledge_base_retrieve (1–3 query variants)
      ▼
outlander-product-index (Foundry IQ knowledge base)
      │
      ▼
Azure AI Search "genapp" → index ks-file-677-index
(20 product files → 326 chunks, embedded with text-embedding-3-small)
      │
      ▼
Retrieved product text → grounded answer with citations
```

| Component | Setting |
|---|---|
| Chat model | gpt-4o (2024-11-20), Global Standard |
| Embedding model | text-embedding-3-small, Global Standard |
| Search | Azure AI Search (Free tier), service `genapp` |
| Knowledge base | `outlander-product-index`, minimal reasoning effort, extractive output |
| Tools | Knowledge base only. Web search was removed so answers can't come from the internet. |

**System instructions:**
> You are the Outlander Gear Co. Product Data Copilot. Help customers with product details, pricing, specifications, warranties, and comparisons. Answer only using information from the attached product knowledge base. If the answer is not in the knowledge base, say you don't have that information. Keep answers concise and friendly.

---

## 3. Automated Evaluation

**Dataset:** [`evaluation/outlander_eval_dataset_query.jsonl`](evaluation/outlander_eval_dataset_query.jsonl). It has 15 questions, and every `ground_truth` answer was checked against the product files. The questions cover pricing, specs, warranties, cross-product comparisons, and one question about a product the store doesn't carry (kayaks).

**Run:** `outlander-copilot-auto-eval-v1`, judge model gpt-4o, pass threshold 3 / 5. Status Completed in 3m 26s.

**Field mapping:** `query → {{item.query}}`, `response → {{sample.output_text}}`, `ground_truth → {{item.ground_truth}}`

### Results

| Evaluator | What it measures | Pass rate | Avg score |
|---|---|---|---|
| **Similarity** | How closely answers match the ground truth (accuracy) | 15/15 (100%) | 4.73 |
| **Groundedness** | Whether answers are supported by retrieved product data | 15/15 (100%) | 5.00 |
| **TaskAdherence** | Whether the agent followed its instructions | 15/15 (100%) | pass |
| **Coherence** | Logical, well-structured answers | 15/15 (100%) | 4.07 |
| **Fluency** | Natural, readable language | 15/15 (100%) | 3.07 |
| **IntentResolution** | Whether the user's actual need was resolved | 14/15 (93%) | 4.80 |
| **Relevance** | Whether answers address the question | 14/15 (93%) | 4.13 |

Latency: P50 5.7s, P95 9.5s.

Evidence: `screenshots/07_auto_eval_results_1.png`, `screenshots/07_auto_eval_results_2.png`, the full export in [`evaluation/auto_eval_results_export.jsonl`](evaluation/auto_eval_results_export.jsonl), and per-question scores in [`evaluation/auto_eval_summary.json`](evaluation/auto_eval_summary.json).

### Interpretation

- **Accuracy and grounding are the strongest results.** Every answer matched the expected answer closely and was supported by the product files. This matters most for a retail copilot, because a wrong price or warranty claim costs the business real money.
- **The only failure was the out-of-catalog question.** For *"Do you sell kayaks?"* the agent said *"I couldn't find information in the knowledge base about Outlander Gear Co. selling kayaks."* The judge scored this 2/5 for both IntentResolution and Relevance, because it didn't clearly say no and didn't suggest an alternative. The agent correctly avoided making up a product. The problem is the wording.
- **Fluency passed but scored lowest (3.07).** The answers are deliberately short ("Keep answers concise"). The judge prefers fuller prose, but short answers suit a quick product-lookup tool, so I accepted this tradeoff.

---

## 4. Manual Evaluation

The full table is in [`evaluation/manual_evaluation.md`](evaluation/manual_evaluation.md). Screenshots: `screenshots/05_samira_copilot_test_1.png` through `_4.png`.

I asked nine questions by hand in the published web app and rated each answer against the source files. Four were straightforward (including the three sample questions from the project instructions), and five were chosen to be hard for a retrieval system: counting across products, finding the cheapest item, a possible hallucination trap, a nuanced "winter use" question, and an open-ended recommendation.

**Result: 9 / 9 correct.** All supporting details, such as the TrailMaster X4's 80 sq ft floor area and gear loft, were verified against the spec sheets.

The thumbs-up/down buttons in the new Foundry playground did not keep their ratings, so the ratings and feedback are recorded in the manual evaluation file.

---

## 5. Where the Copilot Needs Improvement

| # | Issue | Evidence | Recommended fix |
|---|---|---|---|
| 1 | **Vague answers to out-of-catalog questions** | Kayak question: 2/5 on IntentResolution and Relevance in the automated run. The wording also changed between runs. | Add to the instructions: *"If a product isn't in the catalog, say clearly that Outlander Gear doesn't carry it and suggest related products we do sell."* |
| 2 | **Contradictory source data** | `product_info_6.md` (EcoFire stove) lists fuel as "specific fuel sources not provided" in its specs, but its FAQ says "twigs, leaves, and other natural biomass." | Fix the product data so each fact has one source of truth. Customers could get different answers depending on which chunk is retrieved. |
| 3 | **Short answers lower the fluency score** | Fluency average 3.07 / 5 | If a fuller tone is wanted, change "Keep answers concise" to allow one supporting detail per answer. |

---

## 6. Deployment

The agent is published and running. Evidence: `screenshots/09_deployment.png`.

| Field | Value |
|---|---|
| Status | **Running** |
| Active version | Latest (Version 3) |
| Responses protocol endpoint | **Active**. Access requires Azure RBAC permissions. |
| Endpoint URL | `https://bld-gen-ai-app.services.ai.azure.com/api/projects/proj-gen-ai/agents/outlander-product-copilot/endpoint/protocols/openai/responses` |
| Preview web app channel | **Active**. This is the customer-facing chat app used for the manual tests. |
| Web app URL | `https://ai.azure.com/nextgen/agents/8ee24372-0433-4022-963c-c2368e00f4c0/preview?version=3` |

The Responses endpoint is the API that applications call. The web app is the chat interface built on top of it. Both serve version 3.

---

## 7. Cost Notes

This was built on an Azure for Students subscription with a limited budget:
- AI Search on the **Free** tier, so there is no hourly charge.
- Both models on **Global Standard** (pay per token), so there is no charge while idle.
- The automated evaluation was trimmed from 22 auto-suggested evaluators to the 7 relevant ones. Safety evaluators and multi-tool agent evaluators were not relevant to a single-knowledge-base product catalog. Trimming cut judge-model calls by about two-thirds.
