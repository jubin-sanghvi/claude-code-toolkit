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
- **Planning/exploration**: Explore agents (model: haiku) for codebase research. 1-5 parallel based on scope.
- **Git commit + push**: Haiku subagent. Compose message, stage, commit, push.
- **Code review**: Haiku subagent (or cavecrew-reviewer if Caveman plugin installed for compressed output).
- **File location** ("where is X", "what calls Y"): Haiku subagent (or cavecrew-investigator with Caveman).
- **1-2 file surgical edits**: Subagent when scope is bounded.
- **README/docs/wiki updates**: Haiku subagent.
- **Dependency updates**: Haiku subagent.
- **Test execution**: Background bash (not subagent, cheaper).

### Keep in main context (ONLY these):
- User conversation and decisions
- Multi-file refactors where files have circular dependencies
- Architecture pivots requiring user approval mid-stream
- Everything else: delegate. When in doubt, delegate.

### Default: always delegate
Every task goes to a subagent, even single tasks. Only exception: work requiring user interaction mid-stream (questions, approvals, architecture pivots). If unsure whether to delegate, delegate.

### Main thread protocol
Main thread does ONLY:
- Receive user messages and respond with summaries
- Spawn and coordinate subagents
- Ask user questions when blocked
- Report completion with what changed

Main thread NEVER:
- Reads files (delegate to Explore agent)
- Edits files (delegate to implementation agent)
- Runs bash commands except: git status (pre-commit check), launching background tasks
- Writes new files (delegate to builder agent)
- Runs grep/find (delegate to Explore agent)

If you catch yourself about to call Read, Edit, Write, Grep, or Bash for anything other than agent coordination, STOP. Describe the task in a prompt and ship it to a subagent.

### Parallel execution:
When 2+ tasks are independent, ALWAYS launch parallel subagents in a single message. Never sequential when parallel is possible.
- Writing 3 components? 3 parallel agents.
- Building backend + frontend? Parallel agents.
- 4 articles/pages? 4 parallel agents.
- Exploring 3 directories? 3 Explore agents.
- Count of parallel agents limited only by task independence, not by caution.

## Plan Orchestration (REQUIRED in every plan)

Every plan MUST end with an **Orchestration Strategy** section containing:

### 1. Execution batches
Group tasks into parallel batches. Independent tasks in same batch run as parallel subagents in single message. Format:

| Batch | Agent Type | Model | Task | Files |
|-------|-----------|-------|------|-------|
| 1 | Explore (x2) | haiku | Research types + existing patterns | src/types.ts, src/utils/ |
| 1 | Explore | haiku | Find test patterns | tests/ |
| 2 | Subagent | haiku | Add new types | src/types.ts |
| 2 | Subagent | haiku | Add API route | src/routes.ts |
| 3 | Main thread | sonnet | Core logic (multi-file coordination) | src/service.ts, src/handler.ts |
| 4 | Subagent | haiku | Review all changes | diff |

### 2. Main context budget
List ONLY what stays in main context (coordination, user decisions, complex multi-file work). Everything else goes to subagents.

### Rules
- Default haiku for all mechanical/bounded work
- Sonnet only for multi-file coordination or architecture decisions
- Never run sequentially what can run in parallel
- Each batch completes before next starts
- Exploration always batch 1 (haiku Explore agents)
- Review/verification always last batch (haiku)
- Git operations always delegated (haiku subagent)

### 3. Execution enforcement
When executing a plan with an orchestration table:
- Launch ALL agents in a batch as parallel Agent calls in a SINGLE message
- Do NOT fall back to inline work if an agent could do it
- Do NOT launch batch agents sequentially, even if "it's just one quick thing"
- After each batch completes, summarize results in 1-2 sentences, then launch next batch
- Main thread between batches: ONLY coordination text + next batch spawn

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
10. Prefer subagent over inline work: if a task can be described in a prompt, ship it to a subagent instead of doing it in main context
11. Main thread token budget: aim for <5% of session tokens in main thread. If main thread is doing mechanical work, you are doing it wrong.

## Agent Permissions & Autonomy

- Grant agents full read/write/execute access within your home directory
- Agents should NOT ask for permission to edit files, create directories, run builds, install deps
- Agents handle errors autonomously: retry, fix, or report failure in summary
- Only escalate to user for: architecture decisions, ambiguous requirements, destructive operations outside home directory

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
