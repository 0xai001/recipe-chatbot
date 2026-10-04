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
- Baseline: `results/baseline_original_prompt.json` (original prompt)
- v1: `results/v1_grandma_prompt.json`

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
