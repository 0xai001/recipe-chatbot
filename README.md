# AI Evals in Practice — Recipe Chatbot Labs

Coursework for **[AI Evals For Engineers & PMs](https://maven.com/parlance-labs/evals)** (Hamel Husain & Shreya Shankar), worked end to end and written up through a product lens.

This is homework on a toy bot, and I treat it that way. The point is to build first-hand judgment about how evaluation drives product decisions for LLM systems: what "good" means, which failures matter, and when something is ready to ship.

Forked from the course's [recipe-chatbot](https://github.com/ai-evals-course/recipe-chatbot) starter. The bot scaffold, homework briefs and walkthroughs are the course's; the prompts, test sets, analysis and write-ups are mine.

---

## Why evals matter to me as a PM

LLM products fail quietly. They don't throw errors — they drift, contradict their own instructions, or regress when someone "improves" a prompt. Without evals, launch decisions rest on a few hand-picked demos, and quality is whatever the last person to edit the prompt believed it was.

Evals are how a product team turns "it feels better" into a decision it can defend: a definition of quality, a severity model for failures, and a release bar that every change has to clear.

---

## The product under test

A recipe assistant with a deliberate persona and constraints, chosen to create realistic tensions:

| Requirement | Why it makes evaluation interesting |
|---|---|
| Singaporean-grandma persona, Singlish | Brand voice vs repetitive tics |
| Cholesterol-friendly by default | Conflicts with authenticity of classic dishes |
| Nut-free, allergy-aware | Safety-critical — zero tolerance |
| One recipe per reply, fixed format | Instruction-following under pressure |

Bot: Claude Sonnet via LiteLLM · Judge (from HW3): Claude Haiku

---

## Progress

| # | Topic | Status | Write-up |
|---|---|---|---|
| HW1 | Prompt design, test set, baseline vs iteration | ✅ | [findings](homeworks/hw1/findings.md) |
| HW2 | Error analysis & failure taxonomy | ⬜ | |
| HW3 | LLM-as-Judge, judge calibration | ⬜ | |
| HW4 | Retrieval evaluation (BM25, Recall@k, MRR) | ⬜ | |
| HW5 | Agent failure analysis | ⬜ | |
| Capstone | Evals for an agentic harness at enterprise scale | 🗓 Planned | |

---

## HW1 — PM lens

> **Decision this eval informs:** Is prompt v2 safe to ship, and what is it costing us?
>
> **Severity model**
> | Tier | Failure types | Release rule |
> |---|---|---|
> | **Blocker** | Unsafe food guidance, allergen slips, medical claims | Must be 100% — any failure blocks release |
> | **Major** | Ignored explicit instructions (servings, one recipe, cooking method, off-topic handling) | ≥ 95%, regressions block release |
> | **Minor** | Persona tics, labelling, low variety | Tracked, fixed opportunistically |
>
> **Result (v2, 15 targeted queries)**
> - Blockers: **4/4 pass** — refused raw beef & egg with a safe alternative; nut-free and shellfish-free swaps with label reminders; no medical claims.
> - Major: **pass** — servings stated 13/13 (up from 1/13 in v1), one recipe per reply, no deep-frying, off-topic redirected.
> - Minor: **open** — "Aiyo" opener 8/15; "(Lighter Version)" label on 12/13 titles; example recipe intermittently copied.
>
> **Ship call:** Shippable with known minor issues. **Confidence is low** — 15 queries, single runs, and a measured run-to-run noise floor of about ±1 per metric. Before a real launch I'd want a larger, independently sourced test set and repeated runs to report rates, not counts.
>
> **Trade-off surfaced:** The v2 system prompt is ~8× longer than the original (≈1,050 vs ≈125 tokens), added to every call. Responses got ~⅓ shorter, which roughly offsets the input cost since output tokens are priced higher — but the long in-prompt example is also the main driver of copying. Next lever: shorter, varied examples.
>
> **Root causes worth remembering:** in-prompt examples outweigh written rules; two sensible rules can make a third impossible to trigger; soft style guidance ("vary it") barely moves behaviour.

---

## How I'd run this in a product team

What these labs are rehearsing, at team scale:

- **Quality is defined before it's measured.** Every AI feature ships with a written quality bar and severity model, agreed by product, engineering and domain owners — not inferred from whatever the metrics happen to show.
- **Golden datasets are owned assets.** Test sets are curated with domain experts, versioned like code, and continuously refreshed from real production traces, especially failures.
- **Error analysis comes before automation.** People read outputs and name failure modes first; only then do we build automated checks or LLM judges — and judges are calibrated against human labels before anyone trusts them.
- **Evals gate releases.** Prompt, model and retrieval changes run the suite in CI. Blocker regressions stop the release; changes within the noise floor aren't treated as improvements.
- **Every result is traceable.** Each result is tied to a prompt version, model version and dataset version, so any number can be reproduced and challenged.
- **Cost and latency are first-class metrics,** reported alongside quality — a better answer that doubles cost is a product decision, not an engineering detail.
- **Ownership is explicit.** A named owner for the eval suite, a review cadence, and a path for anyone on the team to add a failing case.

---

## Working method

1. Baseline before any change.
2. One change at a time, same test set, same scoring script.
3. Decide the pass bar **before** looking at results.
4. Record what improved, what regressed, and what remains open.
5. Compare with the course walkthrough only after my own attempt is committed.

---

## Run it

Requires Python 3.10+ and [uv](https://docs.astral.sh/uv/).

```bash
git clone https://github.com/0xai001/recipe-chatbot.git
cd recipe-chatbot
uv sync
cp env.example .env        # model names + API key
uv run uvicorn backend.main:app --reload --reload-include '*.md'   # chat UI at http://127.0.0.1:8000
uv run python scripts/bulk_test.py                                  # run the test set
```

Full course setup and homework briefs: [upstream README](https://github.com/ai-evals-course/recipe-chatbot#readme).

## Credits & licence

Course and starter code by Hamel Husain, Shreya Shankar and contributors at [ai-evals-course/recipe-chatbot](https://github.com/ai-evals-course/recipe-chatbot). GPL-3.0 (see [LICENSE](LICENSE)), as is this fork.
