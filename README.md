# kimi-first

A [Claude Code](https://claude.com/claude-code) skill that routes implementation work-orders to [Kimi Code CLI](https://moonshotai.github.io/kimi-code/) (Kimi k3) lanes — **Kimi types, Claude thinks and verifies.**

Claude keeps the judgment work: specs, architecture, orchestration, review, and the merge button. Kimi gets the hands-on volume: frozen-spec builds, mechanical migrations, test fills, bulk sweeps, CI fixes. Kimi's flat-rate membership makes it a cheap, high-throughput implementation lane while Claude spends its tokens on the parts that need judgment.

This is a port of [@steipete](https://github.com/steipete)'s excellent [codex-first](https://github.com/steipete/agent-scripts/blob/main/skills/codex-first/SKILL.md) skill (from [agent-scripts](https://github.com/steipete/agent-scripts)) — same delegation doctrine, swapped from the Codex CLI to the Kimi Code CLI.

## How it works

When a task reads as a **work order** (the spec is frozen, no design decisions left), Claude:

1. Writes a full-context prompt to a temp file — goal, repo paths, constraints, non-goals, expected proof, output shape. Kimi starts with zero session context, so spec quality decides success.
2. Runs `kimi -p` headless in the target repo (tool calls auto-approved, thinking to stderr, answer to stdout).
3. Reviews the full diff like a hostile PR reviewer, runs the tests itself — Kimi's claims are advisory.
4. Iterates via `kimi --continue`; after 2 failed rounds, takes over and does it directly.
5. Ships through normal repo protocol — Claude's hands on the merge button, never Kimi's.

Design work, tiny edits, prod operations, and review are **never** delegated.

## Install

Requires [Claude Code](https://claude.com/claude-code) and the [Kimi Code CLI](https://moonshotai.github.io/kimi-code/) (authenticated via `kimi login`).

```bash
mkdir -p ~/.claude/skills/kimi-first
curl -fsSL https://raw.githubusercontent.com/boringmarketer/kimi-first/main/skills/kimi-first/SKILL.md \
  -o ~/.claude/skills/kimi-first/SKILL.md
```

Or clone and symlink:

```bash
git clone https://github.com/boringmarketer/kimi-first.git
ln -s "$(pwd)/kimi-first/skills/kimi-first" ~/.claude/skills/kimi-first
```

Claude Code picks it up automatically; invoke it explicitly with `/kimi-first` or just hand Claude a work-order-shaped task.

## Verified behavior

The CLI mechanics in the skill were live-tested against Kimi Code CLI v0.26.0:

- `kimi -p` runs headless with tool calls auto-approved — answer on stdout (`• `-prefixed), thinking on stderr
- `--yolo` refuses to combine with `-p` (and isn't needed)
- `kimi --continue -p` resumes the latest session for that directory with full context
- `--continue` is cwd-filtered — never run two concurrent sessions in the same directory
- Headless mode auto-approves, so only delegate from repos without prod secrets in reach

## Credits

- [@steipete](https://github.com/steipete) for the original [codex-first](https://github.com/steipete/agent-scripts/blob/main/skills/codex-first/SKILL.md) doctrine this ports
- Built by [@boringmarketer](https://x.com/boringmarketer) · [boringmarketing.com](https://boringmarketing.com) — Search & AI visibility, in Slack

## License

[MIT](LICENSE)
