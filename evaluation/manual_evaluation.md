# Manual Prompt Evaluation

**Evaluator:** Samira Ray
**Agent:** `outlander-product-copilot` (version 3, gpt-4o, knowledge base `outlander-product-index`)
**Date:** October 13, 2026
**Method:** I asked each question by hand in the published web app preview and compared every answer against the source product files. Each answer got a rating and a short note.

> **Note on the interface:** The thumbs-up/down buttons in the new Microsoft Foundry playground did not keep their ratings when clicked, so this manual review is written up here instead. The screenshots in `screenshots/05_samira_copilot_test_1.png` through `_4.png` show every question and the agent's answer.

## Results

| # | Question | Agent response (summary) | Expected answer (from product data) | Rating | Feedback |
|---|---|---|---|---|---|
| 1 | How much do the TrailWalker Hiking Shoes cost? | $110, with citation | $110 | 👍 Correct | Exact match, cited the product file. |
| 2 | Which tent is the most waterproof? | Alpine Explorer Tent, 3000mm rainfly rating | Alpine Explorer Tent, 3000mm | 👍 Correct | Compared across tents correctly. |
| 3 | Can the warranty for TrailBlaze pants be transferred? | No, non-transferable, original purchaser only | Non-transferable | 👍 Correct | Clear and grounded. |
| 4 | Do you sell kayaks? | Does not currently sell kayaks | Not in catalog | 👍 Correct | Did not invent a product. See the improvement note below. |
| 5 | How many different tents do you sell? | 3: Alpine Explorer, TrailMaster X4, SkyView 2-Person | 3 tents | 👍 Correct | Counting across files worked even though retrieval only returns a few chunks. |
| 6 | What is the cheapest product you sell? | TrekMaster Camping Chair, $50 | TrekMaster Camping Chair, $50 | 👍 Correct | Correct catalog-wide comparison. |
| 7 | What type of fuel does the EcoFire Camping Stove use? | Twigs, leaves, and other natural biomass | FAQ: twigs, leaves, biomass. Spec section: "specific fuel sources not provided" | 👍 Correct, with a data issue | The answer matches the FAQ, but the product file contradicts itself. See Finding 1. |
| 8 | Can I use the CozyNights Sleeping Bag for winter camping? | 3-season, 20°F–60°F, unsuitable for extreme winter | 3-season, 20°F–60°F | 👍 Correct | Gave the reason, not just yes or no. |
| 9 | Which tent is best for a family of four? | TrailMaster X4: 4-person, 80 sq ft, interior pockets, gear loft | TrailMaster X4 (4-person) | 👍 Correct | All supporting details were checked against the spec sheet. Nothing was invented. |

**Manual score: 9 / 9 correct**

## Where it needs improvement

### Finding 1: Contradictory source data (EcoFire stove)
`product_info_6.md` says *"Compatible with various fuel sources (specific fuel sources not provided)"* in its specifications. The FAQ in the same file says it *"uses small twigs, leaves, and other natural biomass."* The agent's answer depends on which chunk retrieval returns, so two customers could get different answers. **Fix:** clean up the product data so one source of truth is used. This is a data-quality problem, not an agent problem.

### Finding 2: Out-of-catalog answers are inconsistent (kayaks)
In the manual test the agent answered "does not currently sell kayaks." During the automated evaluation, the same question produced *"I couldn't find information in the knowledge base about Outlander Gear Co. selling kayaks."* The judge model scored that answer 2/5 for both intent resolution and relevance, because it neither said no clearly nor offered an alternative. **Fix:** add a line to the agent instructions, for example *"If a product is not in the catalog, say clearly that Outlander Gear does not carry it and suggest related products we do sell."*
