# Promptize

> A confidence-gated intent builder, packaged as an Agent Skill — clarify and confirm *what you want* before an agent acts on it.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE) [![Agent Skills compatible](https://img.shields.io/badge/Agent%20Skills-compatible-blue)](https://agentskills.io)

## Why

As we hand more work to autonomous agentic loops, the bottleneck moves from *execution* to *intent*. A loop that runs on a misunderstood premise doesn't make one mistake — it amplifies that mistake across every step it takes.

Promptize is one small, opinionated answer to that, built on a single principle:

> **No consequential decision without informed gating.**

It applies that principle at four touchpoints:

1. **Your intent** — it won't execute on a guess. It clarifies until it is ~95% sure it has your intent right.
2. **Your technical choices** — when a decision needs domain knowledge you may not have, it gives you grounded trade-offs and a marked, contestable recommendation, not a blank "what do you want?".
3. **The agent's own forks** — if execution hits an unanticipated, consequential fork, it never guesses silently and never blocks an unattended run: in-session it asks; in auto/subagent mode it takes the safest reversible option (or defers that branch) and flags it in the final report.
4. **The agent's "done"** — before calling the job finished, it checks the result against your success criteria instead of stopping early.

The result is **governed autonomy**: the run completes, but it leaves an audit trail of its judgement calls instead of silent guesses.

The implementation here is a Claude Code skill — but the pattern, *gate intent before execution*, is ecosystem-agnostic.

## Relation to spec-driven development

Promptize shares the instinct behind **spec-driven development (SDD)** — the shift that treats *intent* as the source of truth and the output (code, a config, an answer) as something that flows from it, not the other way around. It just works at a smaller altitude: not a whole project's spec → plan → tasks, but a **single task's intent, clarified before one run**.

It also leans into the sharpest critique of SDD-in-practice — that people often spec the *solution* they already pictured, not the *problem* it has to solve. Promptize gates the intent first: it clarifies what you actually want **before** any structured artifact — a prompt, or a spec — flows from it. Think of it as lightweight, and upstream of any spec.

## What it does

You give Promptize a rough, ambiguous ask. Before anything runs, it turns that into a structured, reviewable prompt — and only then executes it (and saves it for reuse).

The structured prompt has up to nine fields:

```
Goal · Context · Inputs · Expected output · Scope (in/out)
Constraints · Success criteria · Open assumptions · Examples
```

You approve the prompt, choose how it runs (in your session, or in an isolated subagent), and Promptize executes it. Every approved prompt is saved to a versioned library for reuse.

The prompt it produces is plain Markdown — **not tied to Claude.** Paste it into any LLM (ChatGPT, Gemini, a local model, anything) and it works the same: the tool runs in Claude Code, but its output goes anywhere.

## Example

**You type:**

```
/promptize audit my network device configs for security risks
```

**Promptize clarifies** — a couple of targeted questions (*which devices, and running- or startup-configs? what counts as a "risk" — telnet, weak SNMP, default creds? read-only audit or also remediate? where do findings go?*) — until it is ~95% sure it has your intent, then **produces a prompt you approve:**

```markdown
## Goal
Audit the network device configs for security risks.

## Inputs
- ./configs/*.cfg   (Cisco IOS running-configs)

## Expected output
A findings table in chat: device · risky line · severity · why.

## Scope
IN: telnet/HTTP enabled, weak SNMP communities, default/shared creds, missing AAA, over-broad ACLs.
OUT: routing design, performance tuning.

## Constraints
Read-only — flag risks, never modify a config or push changes.

## Success criteria
Every flagged risk cites device + line + the rule it violates.

## Open assumptions
Assuming these are IOS running-configs; if any are NX-OS / Junos, ask before applying IOS-specific checks.
```

Then it executes the approved prompt (in-session by default). The number doesn't end up in the prompt — it's the *assumptions* that travel, because those are what an executor can act on.

## Two ways to run it

Once you approve the prompt, you choose how it executes:

- **In-session** (default) — runs right here in your conversation. You see every step and can correct it live.
- **Subagent** — hands the prompt to a *separate, isolated* Claude that does the whole task alone and returns one consolidated report. Use it for big jobs (long research, large audits) so the noise stays out of your main conversation.

A subagent runs blind — no conversation, can't ask you mid-task — so Promptize holds it to a **higher bar**: it requires full ≥95% confidence and a self-sufficient brief, and at any unanticipated consequential fork it takes the safe/reversible option or defers that branch and flags it — never guessing, never blocking the run.

## Every prompt is kept

Promptize isn't a one-shot. Each prompt you approve is saved to a versioned library as a dated Markdown file (`~/.claude/prompts/YYYY-MM-DD-<slug>.md`) with metadata — date, execution mode, confidence at approval, status. Over time you build a greppable history of every structured intent you've run: reopen one, adapt it for a new case, and when the same prompt keeps recurring, promote it to a dedicated skill.

## Install

Promptize is an [Agent Skill](https://agentskills.io) — a single `SKILL.md` (plus its docs) in the open Agent Skills format. It isn't tied to one tool: any host that supports the standard can run it, and the prompt it produces works in any LLM regardless of host.

**Today's host is Claude Code.** Drop the folder into your skills directory:

```bash
git clone https://github.com/netcopilot-labs/promptize.git
cp -r promptize ~/.claude/skills/promptize
```

It's then available in every project, invoked explicitly — it never activates on its own.

As more tools adopt the Agent Skills standard, the same folder installs there the same way — point it at that host's skills directory.

## Usage

```
/promptize audit my network device configs for security risks
```

Promptize will:

1. parse the ask and show a field-by-field readiness check,
2. clarify with you until confidence reaches ~95% (or you say *proceed*),
3. assemble the structured prompt for your approval,
4. execute it in your chosen mode, saving the prompt to your library.

## How it works

For the full flow, the diagrams, and the mode-aware fork-handling behaviour, see **[ARCHITECTURE.md](ARCHITECTURE.md)**.

## License

[MIT](LICENSE)
