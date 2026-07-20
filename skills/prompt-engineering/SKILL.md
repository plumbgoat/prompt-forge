---
name: prompt-engineering
description: Build a high-quality prompt from a vague idea, improve an existing prompt, or test a prompt against sample inputs. Use when the user says "help me write a prompt", "why isn't my prompt working", "improve this prompt", "build a prompt for X", or wants to turn a working prompt into a reusable template or slash command.
---

# Prompt Forge

Three modes. Detect which one the user needs; when unclear, assume BUILD.

- **BUILD** — they describe a task and want a prompt for it.
- **IMPROVE** — they paste an existing prompt that underperforms.
- **TEST** — they have a prompt and want to see it survive real inputs.

Whatever the mode, the deliverable is a prompt the user can copy, plus the
option to save it (Step 4). Patterns and the critique checklist live in
`references/patterns.md` — read it before forging.

## Step 1 — Interview (BUILD mode; keep it short)

Ask only what you cannot infer, in ONE compact round of questions — not an
interrogation. The four things that shape everything:

1. **Task & audience**: what should the output accomplish, and who consumes
   it (a human reading? code parsing the output? another AI step?).
2. **Inputs**: what variable content goes in each time (paste an example if
   they have one — a real example beats three paragraphs of description).
3. **Output contract**: exact format wanted — prose, JSON schema, markdown
   table, code only. If code parses it, the format IS the requirement.
4. **Failure modes they fear**: what has gone wrong before, or what must
   never happen (hallucinated facts, verbosity, invented fields, tone).

If they already gave some of these in their message, don't re-ask.

## Step 2 — Forge

Draft the prompt using the structure from `references/patterns.md`:
role → context → task → constraints → examples → input slot → output format.
Rules of thumb:

- Mark variable slots clearly: `{{input_document}}`, `{{user_question}}`.
- Wrap injected content in XML tags (`<document>…</document>`) so
  instructions and data cannot bleed into each other.
- One or two few-shot examples if the format is nontrivial — examples
  outperform paragraphs of format description.
- State what to do, not only what to avoid; give the escape hatch ("if the
  answer isn't in the document, say so") — this is the single highest-value
  line for factuality.
- Match prompt length to task complexity. A short prompt that's precise
  beats a long one that hedges.
- Mind the per-call token cost. Few-shot examples are the main weight, and
  they're re-sent on every run — so when a prompt is example-heavy or headed
  for reuse (a slash command, or a high-volume API call), flag that in the
  hand-off and offer the leaner version: fewer/smaller examples, or zero-shot
  if the format is simple. Mention that API prompt caching can absorb a static
  example prefix so it isn't paid in full each call. Don't sacrifice the
  examples' value for a trivial saving — flag the trade, let the user choose.

Present the forged prompt in a code block, followed by a 3–5 bullet
"why it's built this way" — teach, don't just deliver.

## Step 3 — Test it (offer always; insist when stakes are high)

A prompt that hasn't met real input is a hypothesis. Offer to test:

1. Get 2–3 realistic sample inputs from the user — including one ugly one
   (edge case, adversarial, or malformed). If they have none, generate
   plausible samples and say you did.
2. Run the prompt on each sample yourself (you are the model — execute the
   prompt faithfully as written, not charitably as intended).
3. Judge each output against the contract from Step 1: format exactly right?
   failure modes avoided? Would the consuming code/human accept it?
4. Where it fails, fix the prompt (not the judgment), explain the fix, and
   re-run the failing sample. Show before/after.

Two clean iterations is normal. If it survives the ugly input unchanged,
say so — that's the signal it's ready.

## Step 4 — Save the winner

Offer both, do whichever they pick:

- **Prompt library**: append to `.claude/prompts/<slug>.md` in the project
  (create the `.claude/prompts/` directory if absent — it sits alongside the
  `.claude/commands/` output below) with a one-line header: purpose, variables,
  date. Check the file doesn't already exist; if it does, version the slug.
- **Slash command**: write `.claude/commands/<slug>.md` with frontmatter
  (`description`, `argument-hint`) and the prompt body using `$ARGUMENTS`
  for the variable slot — so next time it's `/slug <input>` instead of
  re-pasting. Tell them it's available next session (or via reload).

## IMPROVE mode

Run the critique checklist from `references/patterns.md` against their
prompt. Report findings as: **what's weak → why it causes the symptom they
see → the fix**, then deliver the rewritten prompt in full. Never rewrite
without explaining — the explanation is what stops the next bad prompt.
If they didn't describe the symptom, ask for one example of bad output
first; critiquing blind wastes a round.

## Anti-patterns

- The 20-question interview. Four answers forge 90% of prompts.
- Cargo-cult additions: "You are a world-class expert" flourishes, threats,
  tips, ALL-CAPS begging. Structure and examples do the work.
- Over-templating a simple task — "summarize this email" does not need
  seven XML sections.
- Declaring a prompt done without testing it against at least one input.
