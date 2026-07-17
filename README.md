# prompt-forge

A Claude Code plugin that turns vague ideas into prompts that actually work —
and doesn't call a prompt done until it has survived real inputs.

Most prompt tools rewrite your prompt and wish you luck. prompt-forge runs
the full loop:

1. **Forge** — a four-question interview (task, inputs, output contract,
   feared failure modes), then a structured prompt built on proven patterns:
   XML-fenced data slots, escape hatches against hallucination, few-shot
   examples, exact output contracts.
2. **Test** — runs the prompt against your real sample inputs, including one
   ugly one. Where it breaks, it fixes the prompt and shows before/after.
3. **Keep** — saves the winner to your project's prompt library, or converts
   it into a personal slash command so next time it's just `/your-prompt`.

It also teaches as it goes: every forge and every critique explains *why* —
so you gradually stop needing it.

## Install

```
/plugin marketplace add plumbgoat/prompt-forge
/plugin install prompt-forge@prompt-forge-marketplace
```

## Use

```
/forge summarize customer support tickets into a daily digest for my team lead
/forge <paste a prompt that keeps producing garbage>
/forge test
```

Or just say "help me write a prompt for…" — the skill triggers on its own.

## Example run

You run:

```
/forge summarize customer support tickets into a daily digest for my team lead
```

**1. Forge** — it asks four quick questions, then builds a structured prompt:

```
› What does a single input look like?     a raw ticket thread (JSON)
› What should the output contain?         3–5 bullet digest + a "needs attention" flag
› Worst thing it could get wrong?         inventing counts, or flagging calm tickets as urgent

Forged prompt:
  <role>You are a support-ops analyst writing a daily digest.</role>
  <tickets>{{TICKETS}}</tickets>
  Summarize into 3–5 bullets. Mark "⚠ needs attention" ONLY when a ticket
  contains an explicit escalation, refund demand, or churn signal.
  If a field is missing, write "unknown" — never estimate a number.
  <example>…one worked example…</example>
```

**2. Test** — it runs the prompt on your samples, including a deliberately ugly one:

```
✔ normal ticket batch      → clean 4-bullet digest
✔ empty/garbled thread     → "unknown", no invented data
✗ calm 'thanks!' ticket    → flagged ⚠ (false positive)

Fix: tightened the flag rule to require an explicit escalation phrase.
Re-test → calm ticket no longer flagged. ✔
```

**3. Keep** — save the winner:

```
Saved to .claude/prompts/support-digest.md — or turn it into /support-digest?
```

Next time it's just `/support-digest`.

## What makes it different

- **Tests, not vibes** — a prompt isn't delivered until it survives sample
  inputs, including an adversarial one.
- **Explained fixes** — critiques map each weakness to the symptom it causes
  (checklist of 10 failure patterns + symptom→cause table).
- **Prompt → slash command** — one step from "this works" to a reusable
  `/command` in your workflow.

## License

MIT
