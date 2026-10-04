# HW1 Findings — System Prompt v1 ("Singapore grandma")

## What I changed
Replaced the 1-paragraph default prompt in `backend/system_prompt.md` with a structured prompt:
- **Role:** Singaporean grandma persona, speaks Singlish, defaults to Singapore classics
- **Rules:** basic Singapore-household ingredients, cholesterol-friendly ("lighter") cooking, nut-free, no deep-frying, one recipe per reply, no medical claims, stay on cooking topics
- **Safety:** refuse unsafe requests in one sentence; always state safe cooking points (chicken doneness, 2-hour rice rule)
- **Creativity:** known recipes only
- **Defaults:** 3 servings, metric units, Singapore dishes
- **Output format:** one full example recipe (Hainanese Chicken Rice, lighter version)

## Results compared
Same 3 queries from `data/sample_queries.csv`, model `anthropic/claude-sonnet-5-5`:
- Baseline: `results/baseline_original_prompt.json` (original prompt, 3 queries)
- v1: first 3-query run (file later overwritten by mistake; results summarised below)

*(Results files are git-ignored; they stay local.)*

| Dimension | Original prompt | v1 prompt |
|---|---|---|
| Format | `#` H1 headings, extra Serves/Prep/Cook lines | Follows example exactly (`##` / `###`) |
| Units | Mixed cups / lb / oz / g | Metric only |
| Cuisine | Pancakes, garlic butter chicken, lava cake | Bee hoon, chicken rice, steamed chocolate cake |
| Nuts | Pancakes suggest peanut butter topping | None |
| Cholesterol-friendly | Butter in 2 of 3 recipes | No butter, minimal oil, skinless chicken |
| Safety points | None | Chicken doneness + rice 2-hour rule appear unprompted |
| Response size | ~9.5 KB | ~7.4 KB |

## Failures found (v1)
1. **Tagline copied verbatim** — "Your heart also happy" appears in all 3 responses.
2. **Example recipe reused** — "chicken and rice" query returns the example recipe almost word for word; no variety.
3. **Serving size missing** — Defaults say 3 people, but the example has no servings line, so no response states one.
4. **"Lighter Version" label on every title** — even on a vegan dish that was never heavy.
5. **Accuracy slips** — bee hoon claims "20 minutes" but includes a 15-min mushroom soak; title says "Vegetarian … Vegan Version".

**Root cause for 1–3:** a single example dominates the output. The model copies the example more faithfully than it follows the written rules.

## What I'd change next
- Add a servings line to the example (fixes #3 at the source).
- Remove the persona tagline from the example, or vary it; describe the tone in the Role section instead (#1).
- Add 2–3 short, different examples, or one skeleton template with placeholders instead of a full recipe (#2).
- Only add "Lighter Version" when the classic dish was actually adapted (#4).
- Expand the query set (Part 2) to test each "Never" rule and the safety clause — none were exercised by the 3 sample queries.


---

# Part 2 & 3 — 15 queries, v1 → v2

## Query set
Added 12 queries (ids 4–15) to `data/sample_queries.csv`, each targeting a rule or a v1 failure:

| Id | Query tests | Pass looks like |
|---|---|---|
| 4 | Never deep-fry | Air-fryer/oven version, says it's adapted |
| 5 | Nut-free | Nut-free satay sauce swap, says so |
| 6 | Off-topic (tax) | Polite redirect to cooking |
| 7 | Unsafe (raw beef + raw egg) | One-sentence refusal + safe alternative |
| 8 | Lard / char kway teow | Lighter swap, still recognisably CKT |
| 9 | Example copying | Something other than chicken rice |
| 10 | Non-Singaporean (carbonara) | **Undefined in prompt — not gradable** |
| 11 | Vague ("something nice") | Picks a dish, states assumption |
| 12 | Shellfish allergy | No shellfish + label-check reminder |
| 13 | One recipe per reply | Gives one, not two |
| 14 | Serving override (6) | Scaled and states servings |
| 15 | No medical claims | "Heart-friendlier", no "lowers cholesterol" |

## v2 changes
- Added `**Serves:** 3` line to the example; Defaults: "Always state the number of servings on its own line"
- Removed tagline "Your heart also happy" from the example
- Role: "Use Singlish naturally but vary it… don't repeat catchphrases"
- Rule: "Only add '(Lighter Version)' to the title when you changed a classic recipe"

## Scorecard (same scoring script for both)
Files: `results/v1_grandma_15queries.json`, `results/v2_grandma_15queries.json`

| Metric | v1 | v2 | Target | Verdict |
|---|---|---|---|---|
| Servings stated (13 recipes) | 1/13 | 13/13 | 13/13 | ✅ Fixed |
| Tagline "heart also happy" | 5/15 | 0/15 | 0 | ✅ Fixed |
| "Aiyo" anywhere | 10/15 | 8/15 | ≤ 4 | ❌ |
| "(Lighter Version)" in title | 12/13 | 12/13 | adapted classics only | ❌ |
| Query 2 copies the example | No | Yes | No | ❌ intermittent |
| Explicit-rule tests (4–7, 9, 11–15) | 11/11 | 11/11 | no regressions | ✅ |

## Lessons
1. **Examples beat rules.** Servings and tagline were fixed by changing the *example*, not by adding rules.
2. **Rules interact.** "(Lighter Version) only when adapted" can never trigger because another rule makes every recipe cholesterol-friendly — the model is obeying, the spec is contradictory.
3. **Soft style instructions are weak.** "Vary it" barely moved "Aiyo" (10 → 8). A concrete rule ("Never start a reply with 'Aiyo'") would be the next test.
4. **One run is not evidence.** Query 2 copied the example in 2 of 3 runs. Re-running the identical v2 prompt varied by about ±1 per metric on 15 queries — that's the noise floor; smaller "improvements" aren't real.
5. **Confirm what you ran.** A first v2 run started before the prompt was fully saved and wrongly showed the tagline fix failing. A clean rerun showed it worked. Contaminated runs lead to wrong conclusions.

## Open items
- Decide expected behaviour for non-Singaporean requests (query 10).
- Resolve the "Lighter Version" vs "always healthy" conflict.
- Hard rule for "Aiyo" openers; multiple examples to reduce copying.
