# no-sycophancy

[![tests](https://github.com/waitdeadai/no-sycophancy/actions/workflows/test.yml/badge.svg)](https://github.com/waitdeadai/no-sycophancy/actions/workflows/test.yml)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-hook-orange)](https://code.claude.com/docs/en/hooks)

> A Claude Code Stop hook that blocks praise-spam at the opening of an assistant turn — *"Great question!"*, *"You're absolutely right!"*, *"That's a brilliant observation!"* — so the model leads with the substantive answer instead of validation theater.

`no-sycophancy` is one bash file (~70 lines, depends only on `jq`) wired into Claude Code's `Stop` and `SubagentStop` events. It inspects the first 240 characters of every outgoing assistant message and pattern-matches the praise-spam vocabulary that LLMs default to at turn-open. When matched, it blocks with a repair-guidance template that tells the model to lead with the answer, not the compliment.

If the operator explicitly asked for praise or encouragement this turn, the hook stays out of the way — it looks for an allow-clause where the model is restating the request (*"you asked for encouragement — here are…"*).

## Why this exists

Sycophancy is the most-prevalent dark pattern in current LLMs:

- The [DarkBench benchmark](https://www.emergentmind.com/topics/darkbench) finds dark patterns in **48% of LLM conversations**, with sycophancy as a primary category.
- The [AAAI 2026 co-creativity study](https://arxiv.org/html/2604.04735v1) measures sycophancy at **91.7% prevalence** across the conversations it sampled.
- Anthropic's own [personal-guidance research](https://www.anthropic.com/research/claude-personal-guidance) measures sycophancy at 9% of guidance-seeking chats — non-zero by Anthropic's own admission.
- Sean Goedecke called sycophancy [the first LLM "dark pattern"](https://www.seangoedecke.com/ai-sycophancy/) — a name that has stuck in the field.
- OpenAI [rolled back GPT-4o in April 2025](https://news.northeastern.edu/2026/02/23/llm-sycophancy-ai-chatbots/) for being excessively flattering and obsequious.

In-context defenses (system prompts saying *"do not be sycophantic"*) can drift over long sessions. The model should not be the only judge of its own sycophancy, so this hook moves the verdict path outside the model context.

## Differentiation from existing anti-sycophancy tools

| Tool | Mechanism | Limitation |
|---|---|---|
| [FutureSpeakAI/anti-sycophancy](https://github.com/FutureSpeakAI/anti-sycophancy) | Runtime circuit breaker + system-prompt calibration | Lives in-context; model can drift past it |
| [0xcjl/anti-sycophancy](https://github.com/0xcjl/anti-sycophancy) | Three-layer Claude Code skill | Skill-based — depends on the model invoking the skill |
| **no-sycophancy** | **Stop hook (bash, out-of-band)** | **Catches the configured linguistic signature at turn-end; no LLM call decides the verdict** |

The three approaches are complementary, not competitive. Run them all if you want defense-in-depth.

## Install (30 seconds)

```bash
mkdir -p .claude/hooks
curl -fsSL https://raw.githubusercontent.com/waitdeadai/no-sycophancy/main/no-sycophancy.sh \
  -o .claude/hooks/no-sycophancy.sh
chmod +x .claude/hooks/no-sycophancy.sh
```

Then merge the hook entries from [`settings.example.json`](settings.example.json) into your `.claude/settings.json`.

Requires `jq`.

## What gets blocked

Praise-spam vocabulary at message open (first 240 chars), including:

- *Great/Excellent/Brilliant/Perfect/Amazing/Fantastic/Wonderful question[!.]*
- *That's a (great/excellent/perfect/brilliant/wonderful/fantastic/amazing/fascinating/really good/very good/terrific) question*
- *You're absolutely right[!.,]*, *You're correct[!.,]*
- *Absolutely[!.,]*, *Of course[!.,]*
- *Excellent (point/observation/catch)[!.,]*
- *Great (point/catch/observation/insight/thinking)[!.,]*
- *Perfect[!.,]*, *Brilliant[!.,]*, *Wonderful[!.,]*, *Amazing[!.,]*, *Fantastic[!.,]*
- *What an (excellent/amazing/insightful/thoughtful) question*
- *Glad you asked*, *Happy to help with that*

The full regex is in [`no-sycophancy.sh`](no-sycophancy.sh) — search for `SYCOPHANCY=`.

## What stays allowed

- The substantive use of any of those words *not* at message open. The hook only inspects the first 240 chars.
- Operator-requested praise — when the message restates a request like *"you asked for encouragement"* or *"since you wanted feedback,"* the allow-clause fires and the hook stays silent.
- Any message that opens with the actual answer, even if it later contains praise vocabulary in a substantive context (*"Brilliant is the right adjective for that approach because…"*).

## Physics-backed engine

This standalone hook remains the simplest install path. For users who want the
benchmark-backed, rule-pack-hashed engine version, the same closeout mechanic is
also available in [AgentCloseoutBench](https://github.com/waitdeadai/agent-closeout-bench):

```bash
git clone https://github.com/waitdeadai/agent-closeout-bench
cd agent-closeout-bench
bash adapters/claude-code/install.sh /path/to/your/project no-sycophancy
bash scripts/hook-smoke.sh
```

The physics-backed adapter maps `no-sycophancy` to the `sycophancy` category
engine and can be used for daily enforcement, fixtures, benchmark evaluation,
and opt-in content-free collaboration telemetry.

## Sister tools

Part of the [LLM Dark Patterns Hooks](https://github.com/waitdeadai/llm-dark-patterns) suite — single-purpose Stop hooks that suppress LLM dark-pattern defaults so power-user operators can actually work.

- [no-vibes](https://github.com/waitdeadai/no-vibes) — false-success closeouts
- [time-anchor](https://github.com/waitdeadai/time-anchor) — training-cutoff date confusion
- [no-curfew](https://github.com/waitdeadai/no-curfew) — unsolicited rest/wellness paternalism
- [no-cliffhanger](https://github.com/waitdeadai/no-cliffhanger) — dangling permission-loop endings
- [honest-eta](https://github.com/waitdeadai/honest-eta) — vibe time estimates and linear-scaling parallelism claims.
- [no-fake-recall](https://github.com/waitdeadai/no-fake-recall) — false-memory recall claims without quoted prior content.
- [no-fake-stats](https://github.com/waitdeadai/no-fake-stats) — fabricated percentages and amounts without source.
- [no-fake-cite](https://github.com/waitdeadai/no-fake-cite) — academic citation patterns without verifiable URL.
- [minmaxing](https://github.com/waitdeadai/minmaxing) — the parent governance harness

## License

Apache-2.0. See [LICENSE](LICENSE).
