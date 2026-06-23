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
| [resume-review](career/resume-review.md) | Score and critique a resume against Google XYZ format, verb diversity, ATS readiness, recruiter-scan optimization, and defensibility. Quick mode (scorecard) or deep mode (full before/after rewrites). Parallel 6-7 agent analysis with file output. |
| [linkedin-optimize](career/linkedin-optimize.md) | Generate an optimized LinkedIn profile (About, headline, narrative experience blurbs) from a resume. Transforms bullets into story-driven content for recruiters and hiring managers. Parallel 4-agent generation with file output. |
| [interview-screen-prep](career/interview-screen-prep.md) | Prepare for recruiter and hiring manager screens. Extracts a reusable story bank, generates resume walkthrough scripts, red flag analysis with reframes, and full Q&A sets in PAR format. Rates answer strength and identifies prep gaps. Parallel 5-6 agent analysis with file output. |
| [interview-behavioral-prep](career/interview-behavioral-prep.md) | Deep behavioral/leadership round preparation. Maps stories against Amazon LPs, Google, Meta, or generic frameworks. Generates both PAR outlines and full 2-minute rehearsal scripts. Includes failure/growth story prep and a prioritized practice plan. Parallel 6-7 agent analysis with file output. |

## Structure

```
claude-code-toolkit/
  career/           # Job search, interview prep, company research
  engineering/      # (future) Code review, architecture, debugging workflows
  productivity/     # (future) Scheduling, reporting, automation
  research/         # (future) Deep-dive investigation patterns
```

## Cost Estimates

Approximate token usage per run. Actual cost varies by data availability (well-known companies produce more web results).

| Skill | Agents | Input Tokens | Output Tokens | Total | Notes |
|-------|--------|--------------|---------------|-------|-------|
| research-company (with role) | 8 Sonnet + Opus synthesis | ~175K-255K | ~13K-15K | ~200-270K | Full 8-agent sweep |
| research-company (no role) | 6 Sonnet + Opus synthesis | ~130K-190K | ~10K-12K | ~150-200K | Skips Engineering and Onboarding agents |
| resume-review (quick) | 6-7 Sonnet + Opus synthesis | ~80K-120K | ~10K-12K | ~100-150K | Scorecard + critique only |
| resume-review (deep) | 6-7 Sonnet + Opus synthesis | ~140K-200K | ~15K-20K | ~180-250K | Full rewrites for weak bullets |
| linkedin-optimize | 4 Sonnet + Opus assembly | ~60K-90K | ~8K-12K | ~80-120K | About + headline + experience blurbs |
| interview-screen-prep | 5-6 Sonnet + Opus synthesis | ~100K-160K | ~12K-18K | ~120-200K | Story bank + Q&A + red flags |
| interview-behavioral-prep | 6-7 Sonnet + Opus synthesis | ~120K-240K | ~18K-30K | ~130-300K | Varies by framework (Amazon highest) |

For context: one run is roughly equivalent to 15-20 minutes of active Claude Code usage.

## Requirements

- Claude Code CLI or IDE extension
- WebSearch and WebFetch tools (built into Claude Code by default)
- No MCP servers, API keys, or external dependencies

## License

MIT
