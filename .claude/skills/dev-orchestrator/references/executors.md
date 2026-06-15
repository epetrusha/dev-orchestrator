# Executor registry — the adapter table for external-agent offload

The orchestrator's external-agent path — the `dev-external-agent` skill, invoked from §Build — names an executor by **tier** — the abstract role. This file is the ONLY place that maps a tier to a concrete command, its prereqs, and its reliability/cost. Skills state the need; this registry resolves the HOW. Add or swap an executor here, never in a skill.

> **This is a template.** The dev core ships the tier model + candidate agents; each project (or the import session) fills the concrete command + prereqs for its own machine and removes tiers it does not use. Do not hardcode one machine's paths into the shared core.

## The split: harness or not

The primary axis is **does the executor bring its own harness?** — it decides the mechanics:

- **`agent`** (harness + model) — own agentic loop + file tools; EDITS files and returns a **diff** you review. The external-agent doctrine (isolation, diff-review, gate, valve) applies in full. Earns its keep only as a **different model** than the orchestrator — same model is just `self` with extra plumbing. Model-agnostic harnesses (Aider, OpenCode) count as `agent` only when pointed at a non-orchestrator model.
- **`model`** (bare, no harness) — text in → text out, no tools, no edits. **You are the harness**: craft the prompt, capture the text, apply + own the edit under your normal discipline. No diff to review — you wrote it.
- **`self`** — the orchestrator does it inline. No offload.

Tag each `agent`/`model` **cloud** or **local** (cost/privacy); reliability is a property of the model you pick. The diff-review + own-test gate is non-negotiable for every `agent`, regardless of tag.

## `agent` candidates (headless, tool-using CLI agents — pick what you have)

All run headless (non-interactive) and edit files in place; confine edits to the repo and review the diff yourself. **Heterogeneity is the point** — pick a *different model lineage* than the orchestrator. (When the orchestrator is Claude, do NOT list Claude Code here — that's `self`.) Substitute the exact invocation your installed version uses; a "local" agent is any of these pointed at a local model. Verify the current invocation + status of any tool before relying on it — this list drifts.

- **Codex CLI** (OpenAI / GPT, cloud) — `codex exec --cd <repo> --sandbox workspace-write "<task>"` (`--sandbox read-only` for read-only; prompt may be trailing arg or piped). `--sandbox workspace-write` confines edits to `--cd`.
- **Aider** (model-agnostic; point at a non-orchestrator model) — `aider --message "<task>" --yes` (git-integrated, auto-commits each edit — fits the *dedicated-branch* isolation model).
- **A second independent-lineage agent**: **OpenCode** (open-source, model-agnostic — also your easiest *local* `agent`), **Goose** (Block), **Amazon Q Developer CLI**, or **Gemini CLI** (Google).

## `model` candidates (bare; you apply the text)

No harness — the orchestrator crafts the prompt, captures stdout, and applies the edit itself.

- **Local via Ollama** — `ollama run <model> "<self-contained prompt>"` → stdout. Scripted/headless capture: `curl -s http://localhost:11434/api/generate -d '{"model":"<model>","prompt":"<...>","stream":false}' | jq -r .response`. (e.g. a Gemma or Llama model.)
- **Local via llama.cpp server**, or **a cloud model via raw API** — same shape: text in, text out, you apply it.

## Commands template

Fill in per machine; record prereqs (auth/login, local model server, API key) next to each command.
