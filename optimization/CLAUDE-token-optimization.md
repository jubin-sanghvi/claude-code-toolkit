# Token-Optimized CLAUDE.md Template

Drop this into `~/.claude/CLAUDE.md` and customize. Works with Claude Code on any plan.

For maximum savings, install these tools first:
- [RTK](https://github.com/rtk-ai/rtk) (CLI output compression, hooks into Bash automatically)
- [Caveman](https://github.com/JuliusBrussee/caveman) (output token compression plugin)
- [Ponytail](https://github.com/DietrichGebert/ponytail) (prevents over-engineering plugin)
- [CodeGraph](https://github.com/colbymchenry/codegraph) (code knowledge graph MCP)
- [Headroom](https://github.com/headroomlabs-ai/headroom) (persistent memory MCP)

---

## Subagent Delegation Rules (ALWAYS FOLLOW)

### Always delegate to subagent:
- **Planning/exploration**: Explore agents (model: haiku) for codebase research. 1-3 parallel based on scope.
- **Git commit + push**: Haiku subagent. Compose message, stage, commit, push.
- **Code review**: Haiku subagent (or cavecrew-reviewer if Caveman plugin installed for compressed output).
- **File location** ("where is X", "what calls Y"): Haiku subagent (or cavecrew-investigator with Caveman).
- **1-2 file surgical edits**: Subagent when scope is bounded.
- **README/docs/wiki updates**: Haiku subagent.
- **Dependency updates**: Haiku subagent.
- **Test execution**: Background bash (not subagent, cheaper).

### Keep in main context:
- User conversation and decisions
- Architecture/design discussions
- Complex multi-file refactors needing coordination
- Anything requiring user approval

### Parallel execution:
When 2+ tasks are independent, ALWAYS launch parallel subagents in a single message. Never sequential when parallel is possible.

## Tiered Model Routing

| Task type | Model | Why |
|-----------|-------|-----|
| Exploration, file reads, grep | haiku | Cheapest, fast enough |
| Git ops, docs, formatting | haiku | No reasoning needed |
| Code review | haiku | Compressed output anyway |
| Default implementation | sonnet | Good balance |
| Deep architecture/reasoning | sonnet | Only in main context |

When spawning Agent, ALWAYS set model parameter:
- `model: "haiku"` for routine/mechanical tasks
- Omit model (defaults to sonnet) for implementation
- `model: "opus"` only if user explicitly requests it

## Memory Strategy

### What to save (auto-memory):
- User preferences and feedback
- Project decisions not captured in code/commits
- External references (URLs, tools, services)
- Architecture choices with reasoning

### What NOT to save:
- Code patterns (read the code)
- File paths (grep for them)
- Debugging solutions (in the commits)
- Anything already in CLAUDE.md

### Cross-session continuity:
Before starting project work, check its memory/ directory. Update or remove stale memories.

## Token Optimization Rules

1. Never re-read a file just read in this conversation
2. Use CodeGraph MCP for code exploration before falling back to grep/find
3. Use Headroom memory for large context preservation across sessions
4. Batch independent tool calls in single messages
5. Keep text responses short
6. Use /compact at phase boundaries (after exploration, after implementation), not when desperate
7. Use /clear between unrelated tasks or project switches
8. Use "ultrathink" in prompt for deep architecture reasoning
9. Use /rewind (Esc+Esc) after 2+ failed corrections instead of continuing with polluted context

## One-Shot Pattern

When given a spec or detailed requirements:
1. Enter plan mode, map spec sections to files/changes
2. Flag ambiguities, don't guess
3. Implement: types/data first, core logic, UI, polish
4. Verify against spec's success criteria

## Preferences

- No auto-commits unless explicitly asked
- Plan mode for significant changes
- Parallel agents for independent work

## CodeGraph

In repositories indexed by CodeGraph (a `.codegraph/` directory exists at the repo root), reach for it BEFORE grep/find or reading files when you need to understand or locate code:

- **MCP tools** (when available): `codegraph_explore` answers most code questions in one call. `codegraph_node` returns one symbol's source + callers, or reads a whole file with line numbers.
- **Shell** (always works): `codegraph explore "<symbol names or question>"` and `codegraph node <symbol-or-file>` print the same output.

If there is no `.codegraph/` directory, skip CodeGraph entirely.

## Headroom

Headroom MCP provides persistent memory across sessions and context optimization.
- Use `headroom memory` for storing large context blobs, architectural snapshots, codebase summaries
- Use `headroom diff` (difft) for structural diffs instead of regular diff
- Use `headroom loc` (scc) for repo shape analysis
