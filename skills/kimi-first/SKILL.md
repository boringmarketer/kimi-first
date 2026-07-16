---
name: kimi-first
description: "Route implementation work-orders to Kimi k3 lanes via the Kimi Code CLI; Claude specs, orchestrates, reviews, verifies. Use when a task is a frozen-spec build, mechanical migration, test fill, bulk sweep, or CI fix — anything that reads as a work order rather than a design problem."
---

# Kimi First

Claude Code sessions only. Rationale: Kimi k3 lanes run flat-rate on the Kimi
membership and are strong at implementation throughput; Claude's edge is judgment,
specs, orchestration, review. Kimi types, Claude thinks and verifies. (Adapted from
[codex-first](https://github.com/steipete/agent-scripts/blob/main/skills/codex-first/SKILL.md)
in steipete/agent-scripts.)

## Route

**Delegate to Kimi** (default for hands-on volume):
- implementation from a frozen spec; refactors; mechanical migrations/sweeps
- bug fixes with a known repro; test writing; coverage fills
- CI fixes, dependency bumps, scripts/tooling
- bulk exploration where raw reading ≫ the answer

**Keep in Claude (never delegate):**
- design, architecture, naming, product judgment — tasks where writing the spec IS the work
- tiny edits (<~20 lines, single obvious change) — delegation overhead loses
- anything needing session tools: MCP servers, browser, analytics, memory
- **prod operations, always**: prod DB (psql), Railway env/deploys, Stripe, DNS, customer
  emails, anything reading `.env`/secrets. Kimi never touches prod credentials or runs
  with them in reach.
- destructive/irreversible ops, releases, pushes, merges — Claude-side per repo protocol
- review of Kimi output — never delegated, never skipped

Heuristic: prompt reads as a work order → delegate; writing it forces decisions → design,
keep it. Mixed task: Claude designs, freezes spec, delegates build-out.

## Model routing

- Default: omit `-m`, uses `default_model` from `~/.kimi-code/config.toml`
  (currently `kimi-code/k3`)
- Latency-sensitive orders: `-m kimi-code/kimi-for-coding-highspeed`
- Escalate on failure, never on importance.

## Invoke

Prompt via temp file in the scratchpad, never inline quoting. `kimi` has no `-C`/`-o`
flags — `cd` into the repo and redirect stdout yourself:

```bash
P=$(mktemp "$SCRATCHPAD/kimi-XXXXXX"); cat >"$P" <<'EOF'
<goal · repo + key paths · constraints ("don't touch X") · non-goals ·
 proof expected (exact test command) · output shape ("report files changed + test output")>
EOF
(cd <repo> && kimi -p "$(cat "$P")" > "$SCRATCHPAD/kimi-last.md" 2>/dev/null)
```

- `-p` is headless: no approvals asked, tool calls run under auto permission; static
  deny rules still apply. `--yolo`/`--auto` cannot combine with `-p` — and aren't needed.
- stdout is assistant text with a `• ` transcript prefix (strip if parsing); thinking and
  tool progress go to stderr — keep `2>/dev/null`, drop only to debug a failing run
- run long jobs with Bash `run_in_background`; read the output file on completion
- only delegate from repos without prod secrets in reach — headless mode auto-approves
  regular tool calls, there is no more-restricted non-interactive tier
- follow-ups: `(cd <repo> && kimi --continue -p "$(cat "$P2")" > "$SCRATCHPAD/kimi-last.md" 2>/dev/null)`
  — resumes the latest session in that dir, keeps context (verified: remembers prior run)
- parallel independent orders OK: separate repos/dirs, separate output files.
  `--continue` is cwd-filtered, so never run two concurrent sessions in the same dir —
  they'd race on the same "latest session"

## Prompt contract

Kimi starts with ZERO session context. Every prompt: goal, exact repo/paths, constraints,
non-goals, proof expected, output shape. Spec quality decides success.

## Verify (Claude, always)

- `git status -sb` + read the FULL diff; judge like a hostile contributor-PR review
- run the focused tests yourself or demand proof output; Kimi claims are advisory
- iterate via `--continue`; after 2 failed rounds, take over and do it directly
- ship through normal repo protocol (commit/create-pr skills, CI watch) — Claude's hands
  on the merge button, never Kimi's
- substantive results feed a cross-vendor adversarial review (Claude attacks
  Kimi-built work) before merge
