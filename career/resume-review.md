Review and score a resume against industry best practices (Google XYZ format, verb diversity, ATS readiness, recruiter-scan optimization, defensibility). Accepts a PDF path or pasted text, optionally a target role or job description. Two modes: quick (scorecard + critique) and deep (scorecard + before/after rewrites for every weak bullet). Saves a full markdown report to disk and prints an executive summary in the terminal.

## How to use it

```
/resume-review ~/Documents/resume.pdf
/resume-review ~/Documents/resume.pdf, Sr Engineering Manager at FAANG
/resume-review ~/Documents/resume.pdf, --jd ~/Documents/job-description.txt
/resume-review ~/Documents/resume.pdf, Staff SWE at AI labs, --deep
/resume-review (with resume text pasted in the same message)
```

Pass a file path (PDF or text) or paste the resume content directly. Optionally add a target role string or `--jd <path>` for role-fit analysis. Add `--deep` for full before/after rewrites (default is quick mode: score + critique only).

---

## What it does

Runs a structured resume audit across 6-7 dimensions using parallel subagents, then synthesizes a scorecard, prioritized improvement list, and (in deep mode) rewritten bullets. Based on Google's XYZ formula, FAANG resume conventions, and ATS parsing requirements.

---

## Execution steps

### Phase 0: Parse input

1. **Detect input type:**
   - If args contain a file path that exists on disk (ending in `.pdf`, `.txt`, `.md`, or `.docx`), read it.
   - Otherwise, treat the user's message content as the resume text.

2. **Parse optional parameters:**
   - Look for `--jd <path>` and read that file as the job description.
   - Look for `--deep` flag. If present, mode = deep. Otherwise mode = quick.
   - Any remaining text after the file path (that isn't a flag) is the `target_role` string (e.g., "Sr EM at FAANG").

3. **Extract candidate name** from the resume content (usually the first line or header).

4. **Create output directory** if it doesn't exist: `~/research/resume-reviews/`

### Phase 1: Parallel analysis (fan-out)

Dispatch ALL of the following analysis agents IN PARALLEL (single message, multiple Agent calls). Use **Sonnet** model for all analysis agents.

Every agent prompt MUST:
- Receive the full resume text.
- Receive the target role and/or JD content if provided.
- End with: "Return findings as a concise structured summary. No preamble, no hedging, no filler. Use bullet points and tables where helpful. Max 600 words."

---

**Agent 1: XYZ Format Audit**
Analyze every bullet point in this resume against Google's XYZ format. For each bullet, identify:
- X (accomplishment): what was achieved?
- Y (quantified metric): how was it measured? Is the metric hard (dollar, percentage, count, time) or soft (vague, qualitative)?
- Z (mechanism): how was it done?

For each bullet, rate completeness: Complete (all three present with hard Y), Partial (missing one component or soft Y), or Weak (missing two or more).

Output:
- Table of all bullets with XYZ breakdown and completeness rating.
- Overall score: % of bullets rated Complete.
- Top 3 worst bullets (most in need of rewrite).

---

**Agent 2: Verb Diversity + Strength**
Extract the lead verb from every bullet point in this resume. Analyze:
- List all lead verbs in order.
- Flag any duplicates (same verb used more than once across the entire resume).
- Flag weak/generic verbs: Led, Helped, Worked on, Managed, Responsible for, Assisted, Participated, Supported, Handled, Involved in.
- For each weak or duplicated verb, suggest 2-3 stronger alternatives from this bank: Spearheaded, Orchestrated, Pioneered, Catalyzed, Drove, Captured, Doubled, Tripled, Scaled, Accelerated, Engineered, Compressed, Delivered, Shipped, Cut, Boosted, Expanded, Built, Designed, Architected, Launched.

Output:
- Full verb list with flags.
- Score: (unique strong verbs) / (total bullets). 1.0 = perfect diversity, no repeats, no weak verbs.
- Duplicate count and weak verb count.

---

**Agent 3: ATS + Keyword Readiness**
Evaluate this resume for Applicant Tracking System parsing and keyword optimization:

**Structural checks:**
- Section headers: are they standard ("Experience", "Skills", "Education") or creative/non-standard?
- Date format: consistent and parseable? (Acceptable: "Jan 2025 - Present", "January 2025 - Present")
- Does the skills section exist? Is it keyword-dense?
- Are job titles standard and searchable?
- Any formatting that might break ATS parsing? (tables, columns, headers in images, special characters)

**Keyword checks (if target role or JD provided):**
- Extract top 20 keywords from the target role/JD.
- Check which appear in the resume (exact match or close synonym).
- Calculate keyword match percentage.
- List missing high-value keywords that should be added.

**Jargon check:**
- Flag internal acronyms or jargon that an external reader wouldn't understand.
- Suggest spelled-out alternatives.

Output:
- Structural compliance score (out of 10).
- Keyword match % (if role/JD provided).
- List of missing keywords.
- Jargon flags.

---

**Agent 4: Recruiter-Scan Test (6-second test)**
Simulate a recruiter spending 6 seconds on this resume. In that time, can they identify:

1. Current role and company (is it immediately obvious?)
2. Biggest quantified impact (dollar amount, scale number, percentage)
3. Team size or scope of responsibility
4. Technical domain (what kind of engineer/leader is this person?)
5. Career progression (is there a visible growth arc?)

**Visual hierarchy assessment:**
- Are numbers and metrics the most visually prominent elements?
- Is the strongest bullet first in each role?
- Does the summary (if present) work as a "billboard" (3 concrete claims in 3 sentences)?
- Are dollar amounts, percentages, and scale numbers bolded consistently?
- How many "eye-catch" numbers appear in the first half of page 1?

**Bullet ordering assessment:**
- For each role: is the strongest bullet (biggest impact, hardest metric) listed first?
- Are the weakest bullets (soft metrics, qualitative outcomes) last?

Output:
- 6-second scan result: what a recruiter would take away.
- Eye-catch count (numbers that jump out on first page).
- Bullet ordering issues (roles where strongest isn't leading).
- Score: 1-10 based on how well the resume performs in a quick scan.

---

**Agent 5: Honesty + Defensibility Audit**
Evaluate each bullet for potential overclaims or defensibility risks in a technical interview:

**Flag these patterns:**
- "Architected" or "Spearheaded" for work that sounds collaborative (multiple teams, "with" language).
- Revenue or bookings claims without scoping the individual's contribution vs. broader org.
- Unverifiable superlatives ("first ever", "industry-leading", "revolutionary").
- Passive voice hiding attribution ("was delivered", "was achieved").
- Projected numbers stated as realized ("projecting $X" vs. "drove $X").
- Math inconsistencies (do breakdowns add up to totals?).
- Scale claims that seem implausible for the role level.

**For each flagged bullet, assign risk:**
- Low: defensible with a one-sentence explanation.
- Medium: needs a prepared 30-second story to defend.
- High: likely to be challenged and hard to defend without hedging.

Output:
- Risk register: table of flagged bullets with risk level and specific concern.
- Suggested softer phrasing where overclaim detected.
- Math-check results (any totals that don't match breakdowns).
- Overall defensibility score: 1-10 (10 = bulletproof, 1 = will crumble under pressure).

---

**Agent 6: Structure + Layout**
Evaluate resume structure and formatting conventions:

**Document structure:**
- Total page count. Is it appropriate for experience level? (1 page for <5yr, 2 for 5-15yr, 3 only for 15+ or academia)
- Section order: is it logical? (Summary > Skills > Experience > Education is standard for senior)
- Reverse chronological: strict? Any ordering violations?
- Education placement: at bottom for experienced professionals?

**Per-role structure:**
- Bullets per role: are any roles over-represented (7+ bullets) or under-represented (1 bullet)?
- Do bullet counts decrease with role age? (Recent roles should have more bullets)

**Formatting consistency:**
- Bolding pattern: is it consistent? (Bold metrics only, never verbs, never 4+ consecutive words)
- Punctuation: any em-dashes (flag them)? Consistent use of separators?
- Bullet symbols: consistent throughout?
- Date format: consistent across all roles?
- Page breaks: any orphaned bullets (single bullet alone on next page)? Section headers stranded without content?

**Skills section:**
- Is it grouped by category? (Languages, Platforms, Cloud, AI Tooling, etc.)
- Keyword density: does it reinforce the bullets and target role?

Output:
- Structure issues list.
- Formatting inconsistencies.
- Score: 1-10 for overall structural quality.

---

**Agent 7: Role-Fit Gap Analysis** (ONLY dispatch if target_role or JD is provided)
Compare this resume against the target role requirements:

**If JD provided:**
- Extract all required qualifications, preferred qualifications, and responsibilities.
- Map each to evidence in the resume (present, partially present, or missing).
- Calculate fit percentage: (matched requirements) / (total requirements).

**If target role string only (no JD):**
- Infer typical requirements for that role level at that company type.
- Assess whether the resume signals readiness for that level.

**Analysis:**
- Top 3 strengths: where the resume exceeds requirements.
- Top 3 gaps: required experience not evidenced in the resume.
- Positioning advice: which bullets to lead with for this specific role.
- Level-match assessment: does the resume signal the right seniority? (Under-leveled, matched, over-leveled)

Output:
- Fit score: percentage.
- Strengths and gaps tables.
- Positioning recommendations.
- Level-match verdict.

---

### Phase 2: Synthesis

After all agents return, the main thread synthesizes. Do NOT delegate this step. Read all agent reports and produce:

#### Quick mode output:

**Scorecard table:**

| Dimension | Score | Key Finding |
|-----------|-------|-------------|
| XYZ Format | _/10 | (one line) |
| Verb Diversity | _/10 | (one line) |
| ATS Readiness | _/10 | (one line) |
| Recruiter Scan | _/10 | (one line) |
| Defensibility | _/10 | (one line) |
| Structure | _/10 | (one line) |
| Role Fit | _/10 | (one line, or "N/A" if no role provided) |

**Overall Grade:** A / B / C / D with one-line rationale.
- A (8.5+ avg): Interview-ready. Minor polish only.
- B (7.0-8.4): Strong foundation, 3-5 targeted fixes will elevate it.
- C (5.5-6.9): Significant issues. Needs a rewrite pass.
- D (below 5.5): Fundamental structural or content problems.

**Top 5 Most Impactful Improvements:** Prioritized by impact-to-effort ratio. Each item: what to fix, which bullet(s), and why it matters.

**Per-Bullet Severity Flags:** Mark each bullet as Green (no issues), Yellow (minor fix), or Red (needs rewrite).

**Role-Fit Summary (if applicable):** Fit score, top 3 gaps to address, which bullets to lead with.

#### Deep mode output (everything above PLUS):

**Before/After Rewrites:** For every bullet flagged Yellow or Red, provide:
- The original bullet (before).
- The rewritten bullet (after), applying XYZ format, strong verb, hard metric, active gerund mechanism.
- A one-line explanation of what changed and why.

**Summary Rewrite:** If the current summary scores below 7/10, provide a rewritten version using the "billboard" format (3 concrete claims, IC credibility tail if EM/Director).

**Skills Section Restructure:** If skills aren't grouped by category or are missing obvious keywords from the resume content, provide a restructured version.

**Bullet Reordering:** For each role where the strongest bullet isn't leading, provide the suggested order.

**Verb Substitution Map:** Table of current verb -> suggested replacement for all duplicates and weak verbs.

---

### Phase 3: Output to file

Write the full report to: `~/research/resume-reviews/{name_slug}-{YYYY-MM-DD}.md`

Replace spaces in the candidate name with hyphens, lowercase. Example: `jubin-sanghvi-2026-06-13.md`

**Report structure:**

```markdown
# Resume Review: {Candidate Name}

**Date:** {YYYY-MM-DD}
**Target:** {role/JD description, or "General review"}
**Mode:** {Quick / Deep}

---

## Executive Summary

{Overall grade, 2-3 sentence assessment, single most important thing to fix}

---

## Scorecard

| Dimension | Score | Key Finding |
|-----------|-------|-------------|
| XYZ Format | X/10 | ... |
| Verb Diversity | X/10 | ... |
| ATS Readiness | X/10 | ... |
| Recruiter Scan | X/10 | ... |
| Defensibility | X/10 | ... |
| Structure | X/10 | ... |
| Role Fit | X/10 | ... |

**Overall: {Grade}** - {one-line rationale}

---

## Top 5 Improvements

1. ...
2. ...
3. ...
4. ...
5. ...

---

## Detailed Findings

### XYZ Format Audit
{condensed from Agent 1}

### Verb Analysis
{condensed from Agent 2}

### ATS Readiness
{condensed from Agent 3}

### Recruiter-Scan Test
{condensed from Agent 4}

### Defensibility Audit
{condensed from Agent 5}

### Structure + Layout
{condensed from Agent 6}

### Role-Fit Analysis (if applicable)
{condensed from Agent 7}

---

## Risk Register

| Bullet | Risk Level | Concern | Suggested Fix |
|--------|-----------|---------|---------------|
| ... | High/Med/Low | ... | ... |

---

## Rewrites (deep mode only)

| # | Before | After | Why |
|---|--------|-------|-----|
| 1 | "..." | "..." | ... |
| 2 | "..." | "..." | ... |

---

## Per-Bullet Status

| Role | Bullet (first 8 words) | Status | Primary Issue |
|------|------------------------|--------|--------------|
| ... | ... | Green/Yellow/Red | ... |

---

## Gap Analysis (if role/JD provided)

**Fit Score:** X%

**Strengths:**
- ...

**Gaps:**
- ...

**Positioning:**
- Lead with: ...
- De-emphasize: ...
```

---

### Phase 4: Terminal summary

After writing the file, print a condensed summary to the terminal:

```
## Resume Review: {Name}
## Grade: {A/B/C/D} | Mode: {Quick/Deep} | Target: {role or "General"}

| XYZ | Verbs | ATS | Scan | Defense | Structure | Fit |
|-----|-------|-----|------|---------|-----------|-----|
| X   | X     | X   | X    | X       | X         | X   |

Top 3 quick wins:
1. ...
2. ...
3. ...

{If deep mode: "Rewrote X/Y bullets. See full report for before/after."}

Full report: ~/research/resume-reviews/{filename}.md
```

---

## Resume quality principles (baked into agent prompts)

These rules inform the scoring criteria. Each agent should evaluate against the relevant subset:

1. **Google XYZ format:** Every bullet needs Accomplishment (X), Quantified Metric (Y), and Mechanism (Z).
2. **Verb diversity:** Each lead verb should appear at most once across the entire resume.
3. **Honesty calibration:** Verb must match ownership scope. "Architected" only if you owned the design. Use "Built", "Shipped", "Co-architected" when scope was shared.
4. **Active gerunds beat passive noun phrases:** "by migrating to Parquet, redesigning partitions" beats "via Parquet conversion, partition redesign."
5. **One bullet, one idea:** Don't combine cost savings and architecture and adoption into a single bullet.
6. **Math check:** Totals must match breakdowns (e.g., "4 engineers: 3 mid to senior + 2 senior to tech lead" = 5, not 4).
7. **No em-dashes:** Flag any em-dash usage (the long dash character). Replace with commas, colons, parentheses, or periods.
8. **Bold only metrics:** Never bold verbs, never bold 4+ consecutive words. Aim for 1 bold element per bullet.
9. **Strongest bullet leads:** For EM/Director, lead with biggest financial number. For IC, lead with architectural/scale flagship.
10. **Spell out jargon:** Internal acronyms should be spelled out for external readers.
11. **Summary as billboard:** For senior roles (8+ years), summary should be 3 sentences with concrete claims, not soft adjectives.
12. **Skills grouped by category:** Languages, Platforms, Cloud, AI Tooling, etc. for ATS keyword density.

---

## Token optimization rules

- All analysis agents run at **Sonnet** model tier.
- ALL agents (6-7) run in PARALLEL in a single message dispatch.
- Each agent prompt ends with the word-cap instruction (600 words max).
- The main thread only does synthesis and file writing (Phase 2-4). It never duplicates the agents' analysis.
- Agent 7 (Role-Fit) is only dispatched if a target role or JD is provided (6 agents for general review, 7 for targeted review).
- Quick mode target: ~100-150K total tokens.
- Deep mode target: ~180-250K total tokens (extra synthesis for rewrites).

---

## Portability

This skill is a standalone `.md` file. To use on another machine:
1. Copy this file to `~/.claude/commands/resume-review.md` (or a project commands dir).
2. Requires: Read tool (for PDF/file input). WebSearch only needed if JD keywords require industry context.
3. No MCP servers, API keys, or project-specific dependencies.
4. Output goes to `~/research/resume-reviews/` which is created automatically.
