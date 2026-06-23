# claude-code-toolkit

Reusable Claude Code command files for career research, engineering workflows, and productivity automation. Drop into `~/.claude/commands/` and invoke with `/command-name`.

## Usage

1. Copy any `.md` file into `~/.claude/commands/` (or a project-level `.claude/commands/` directory).
2. Invoke it in Claude Code with `/filename` (without the `.md` extension).

## Skills

### Career

| Skill | Description |
|-------|-------------|
| [research-company](career/research-company.md) | Full due-diligence report on a company: financials, culture, tech stack, market position, alumni sentiment, trajectory outlook. Parallel 8-agent sweep with file output. |

## Structure

```
claude-code-toolkit/
  career/           # Job search, interview prep, company research
  engineering/      # (future) Code review, architecture, debugging workflows
  productivity/     # (future) Scheduling, reporting, automation
  research/         # (future) Deep-dive investigation patterns
```

## Requirements

- Claude Code CLI or IDE extension
- WebSearch and WebFetch tools (built into Claude Code by default)
- No MCP servers, API keys, or external dependencies

## License

MIT
