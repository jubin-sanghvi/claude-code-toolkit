Prepare for recruiter screens (15-30 min) and hiring manager screens (45-60 min). Generates a reusable story bank, resume walkthrough scripts, red flag analysis with prepared reframes, and full Q&A sets in PAR (Problem, Action, Result) format. Rates every answer by strength so you know where to invest prep time. Saves a full prep guide to disk and prints a condensed drill sheet in the terminal.

## How to use it

```
/interview-screen-prep ~/Documents/resume.pdf
/interview-screen-prep ~/Documents/resume.pdf, Sr Engineering Manager at Meta
/interview-screen-prep ~/Documents/resume.pdf, --jd ~/Documents/job-description.txt
/interview-screen-prep ~/Documents/resume.pdf, --jd https://careers.example.com/job/12345
/interview-screen-prep ~/Documents/resume.pdf, --github jubin-sanghvi
/interview-screen-prep ~/Documents/resume.pdf, --jd ~/jd.txt, --github jubin-sanghvi
/interview-screen-prep (with resume text pasted in the same message)
```

Pass a file path (PDF or text) or paste the resume content directly. Optionally add a target role string, `--jd <path_or_url>` for targeted prep, and/or `--github <username>` for project-based questions.

---

## What it does

Analyzes a resume to extract core stories, anticipate tough questions, and generate interview-ready answers in PAR format. Covers both recruiter phone screens and hiring manager screens. The story bank output is designed to be consumed by a future `interview-behavioral-prep` skill for deeper leadership round prep.

---

## Execution steps

### Phase 0: Parse input

1. **Detect input type:**
   - If args contain a file path that exists on disk (ending in `.pdf`, `.txt`, `.md`, or `.docx`), read it.
   - Otherwise, treat the user's message content as the resume text.

2. **Parse optional flags:**
   - `--jd <path_or_url>`: If path exists on disk, read it. If it looks like a URL (starts with http), use WebFetch to retrieve it.
   - `--github <username>`: GitHub username for surface profile scan.
   - Any remaining text after the file path (that isn't a flag or flag value) is the `target_role` string.

3. **Check for existing company research:**
   - If a JD or target role mentions a company name, scan `~/research/` for a matching file (pattern: `{company}*.md`).
   - If found, read the Executive Summary, Trajectory Scorecard, and Interview Questions sections.
   - Store as `company_context` for agents that need it.

4. **Extract from resume:**
   - Candidate name
   - Career arc (chronological list of roles with dates)
   - Quantified achievements (dollar amounts, percentages, scale numbers, team sizes)
   - Skills and technical domain
   - Employment gaps (periods with no role listed)
   - Tenure patterns (time at each company)
   - Title progression (lateral, upward, or mixed)

5. **Create output directory** if it doesn't exist: `~/research/interview-prep/`

### Phase 1: Parallel analysis (fan-out)

Dispatch ALL of the following agents IN PARALLEL (single message, multiple Agent calls). Use **Sonnet** model for all agents.

Every agent prompt MUST receive: the full resume text, the extracted profile facts from Phase 0, the target role if provided, the JD content if provided, and the company context if available.

---

**Agent 1: Story Bank Extractor**

Read this resume and identify 5-8 core stories that represent the candidate's most significant achievements. A "story" is a distinct accomplishment that demonstrates multiple competencies and can be told in PAR (Problem, Action, Result) format.

For each story:
1. **Name it** with a short memorable label (2-4 words, e.g., "Platform Migration at Scale", "Turned Around Failing Team").
2. **Extract the PAR skeleton** from resume bullets:
   - Problem: what was broken, missing, or needed?
   - Action: what did the candidate specifically do (their contribution, not the team's)?
   - Result: quantified outcome (metrics, dollars, scale, time saved).
3. **List question themes** this story can answer. Choose from: leadership, conflict resolution, technical decision-making, scaling systems, failure/learning, cross-functional influence, hiring/growing people, prioritization, ambiguity, innovation, cost optimization, stakeholder management, organizational change.
4. **Rate strength:**
   - Strong: clear PAR with hard metrics in the Result.
   - Medium: has PAR but the Result is soft (qualitative, no numbers) or the Problem is implied.
   - Thin: missing a clear Problem OR Result from the resume text. Needs the candidate to fill in context.

Also flag: are there common question themes (from the list above) that NO story covers? List these as "uncovered themes."

Output: table of stories + detailed PAR skeletons + uncovered themes list. Max 600 words.

---

**Agent 2: Resume Walkthrough Generator**

Generate a 2-3 minute "walk me through your background" pitch for this candidate. This is the answer to the most common opening question in any screen.

**Structure:**
- **Opening (20-30 sec):** One sentence establishing credibility and domain. Not "I am a [title] with [years]." Instead: what kind of problems you solve, at what scale.
- **Career narrative (90-120 sec):** Hit 2-3 key roles in chronological order. For each: company name, your role in one phrase, 1-2 quantified achievements that connect to the target role. Show progression (IC to lead, or increasing scope, or deepening specialization).
- **Current state (20-30 sec):** What you're doing now, what you've accomplished recently, and why you're open to new opportunities. Frame positively (pulled toward something, not pushed away).
- **Bridge (10 sec):** Connect your arc to the target role/company. "Which is why [this role] is compelling to me because [specific connection]."

Generate TWO versions:
1. **Recruiter version (2 min):** Lighter on technical depth, heavier on motivation and progression. Recruiters want to know: are you qualified, are you genuinely interested, and are you a good communicator?
2. **Hiring Manager version (2.5 min):** Deeper on impact and technical judgment. HMs want to know: what did you actually own, how do you think about problems, and would I want you on my team?

If target role/JD provided, tailor the bridge and emphasis to match the role's requirements.
If company research context is available, weave in one company-specific signal in the bridge (e.g., "I've followed how [company] is approaching [challenge], and that maps directly to what I've been building at [current company]").

**Rules:**
- First person throughout.
- No em-dashes. Use commas, colons, parentheses, or periods.
- Conversational tone (this is spoken, not written). Use contractions naturally.
- Numbers should be spoken-friendly ("about 200 million" not "$200M").

Output: both walkthrough scripts, clearly labeled. Max 600 words.

---

**Agent 3: Red Flag Analyzer**

Scan this resume for patterns that would trigger tough follow-up questions from a recruiter or hiring manager. Think like a skeptical screener: what stands out as unusual, risky, or needing explanation?

**Patterns to check:**
- Employment gaps (6+ months with no role listed)
- Short tenures (under 18 months at any company)
- Lateral moves (same or lower title between companies without clear scope increase)
- Single-company long tenure (5+ years, especially 8+, may signal risk aversion or narrow experience)
- Title inflation between companies (VP at a 10-person startup to IC at a large company)
- Title deflation (demotion signal?)
- Missing skills for the target role (if JD provided)
- Overqualification signals (applying "down" a level)
- Recent role change (why leave after only X time?)
- Vague bullets that invite "tell me more" probes (no metrics, unclear ownership)
- Industry/function switches without explanation
- Education gaps or unusual credentials for the level

**For each red flag found:**
1. Describe the pattern and why it concerns a screener.
2. Generate the exact question they would ask (in quotes).
3. Draft a PAR-format answer that reframes the pattern positively. The reframe should be honest (no lying) but position the choice as intentional and growth-oriented.
4. Rate severity:
   - High: will definitely come up, must have a polished answer.
   - Medium: likely to come up, should have talking points ready.
   - Low: possible but not guaranteed, brief answer sufficient.

If no red flags found, say so explicitly (this is a valid and useful result).

Output: red flag table with severity, question, and reframe answer. Max 600 words.

---

**Agent 4: Recruiter Screen Q&A Generator**

Generate 12-15 questions a recruiter would ask this specific candidate in a 15-30 minute phone screen. Base questions on the actual resume content, not generic templates. Tailor to what THIS resume would make a recruiter curious about.

**Group questions by category:**

**Background and Motivation (4-5 Qs):**
- Walk me through your background (reference Agent 2's output for answer structure)
- Why are you looking / open to new roles?
- Why this company / role specifically? (if JD/company provided)
- What are you looking for in your next role?
- What's your ideal team/company culture?

**Logistics and Fit (3-4 Qs):**
- Compensation expectations
- Timeline / notice period / availability
- Location / remote preferences
- Other interviews in progress?

**Resume Deep-Dive Probes (3-4 Qs):**
- Questions about specific achievements that need elaboration
- Questions about transitions between roles
- Questions about scope claims that need validation

**For each question:**
- Provide a PAR-format answer draft based ONLY on what the resume supports. If the resume doesn't provide enough for a full PAR, use what's available and mark what's missing.
- Rate answer strength: Strong / Adequate / Weak
- If Weak: note "NEEDS PREP" and explain what the candidate should think through before the interview.

**Also generate: 3-4 questions the candidate should ask the recruiter**, each with a one-line note on what signal it provides (team health, timeline, role clarity, compensation philosophy, etc.).

If JD or company context available: tailor "Why this company?" and "Why this role?" with specific details.

Output: grouped Q&A with strength ratings. Max 800 words.

---

**Agent 5: Hiring Manager Screen Q&A Generator**

Generate 15-18 questions a hiring manager would ask this specific candidate in a 45-60 minute screen. These are deeper than recruiter questions: they probe judgment, leadership philosophy, technical depth, and how you handle hard situations.

**Group questions by category:**

**Leadership and People Management (4-5 Qs):**
- How do you build and develop a team?
- Tell me about growing someone on your team (promotion, skill development)
- How do you handle underperformers?
- Describe your management philosophy in 2-3 sentences
- How do you balance being hands-on technically vs. leading people?

**Technical Depth and Decision-Making (4-5 Qs):**
- Walk me through a major technical decision you drove recently
- Tell me about a time you had to choose between competing architectural approaches
- How do you stay technical as a people leader?
- What's the most complex system you've owned end-to-end?
- How do you make build vs. buy decisions?

**Conflict and Difficult Decisions (3-4 Qs):**
- Tell me about a disagreement with a peer leader and how you resolved it
- Describe a time you had to say no to a stakeholder
- Tell me about a project that failed and what you learned
- How do you handle conflicting priorities from multiple stakeholders?

**Initiative and Problem-Solving (3-4 Qs):**
- Tell me about something you built or drove that wasn't asked of you
- How do you identify what to work on vs. what to delegate?
- Describe improving a process or system significantly
- What's the biggest impact you've had in your current role?

**For each question:**
- Provide a PAR-format answer draft referencing specific resume content.
- Note which "core story" from the resume this answer draws from (use a label like "Story: Platform Migration" that maps to Agent 1's story bank).
- Rate answer strength: Strong (clear PAR with metrics) / Adequate (has PAR but soft result) / Weak (missing components)
- If Weak: note "NEEDS PREP" with guidance.

**Also generate: 4-5 questions the candidate should ask the hiring manager**, each probing a different dimension (team health, decision-making autonomy, success criteria, biggest challenge, growth ceiling). Include a one-line note on what each question reveals.

Output: grouped Q&A with story references and strength ratings. Max 800 words.

---

**Agent 6: GitHub Profile Scanner** (ONLY dispatch if `--github` flag is provided)

Scan the GitHub profile for `{username}` at a surface level. Do NOT deep-read entire repos. Focus on what a hiring manager would see in a 2-minute profile visit.

**Gather:**
- Bio and profile README (if present)
- Pinned repositories: names, descriptions, languages, star counts
- Top 5 most-starred or most-recent repos
- Contribution graph: is this person actively contributing? Consistent or sporadic?
- Any repos that signal professional projects vs. hobby/learning

**Generate:**
- 3-5 "tell me about this project" questions a technical HM might ask based on visible repos
- For each: suggest a PAR-format answer connecting the project to professional skills (e.g., "This open-source tool solved [Problem] I kept hitting at work. I built [Action] and it's been adopted by [Result].")
- Note any inconsistencies between GitHub activity and resume claims (e.g., resume emphasizes Python but GitHub is predominantly TypeScript)
- Note any positive signals (active open-source maintainer, popular tools, consistent contribution pattern)

Output: project questions with suggested answers + consistency notes. Max 400 words.

---

### Phase 2: Synthesis (main thread)

After all agents return, the main thread assembles the final prep guide. Do NOT re-derive content. Perform these editorial passes:

1. **Cross-reference story bank against Q&A:** For each answer in Agents 4-5, add a story label annotation from Agent 1's bank. If an answer maps to a named story, mark it. If it doesn't map to any story, note that (this answer needs its own preparation).

2. **Identify coverage gaps:** Collect all questions marked "NEEDS PREP" or rated "Weak" into a dedicated Preparation Gaps section. For each gap, include:
   - The question
   - What's missing from the resume
   - Guidance on what to think through (specific memories, conversations, or experiences to recall)

3. **Compute readiness score (1-10):**
   - Story coverage (2 pts): % of common question themes that have at least one Strong story
   - Red flag readiness (2 pts): all High-severity flags have prepared reframes
   - Answer quality (3 pts): % of total answers rated Strong
   - Company knowledge (1 pt): company research integration present and used (or N/A)
   - Walkthrough quality (2 pts): both versions are specific, timed, and bridge to target

4. **Company integration (if research report found):**
   - Pull "Why this company?" talking points from the research
   - Extract 2-3 risks from the research as "questions to ask" (shows research and probes real concerns)
   - Note "signals to listen for" during the screen (leadership stability, hiring velocity, tech debt signals)

5. **Assemble the full report** in the grouped-by-theme format below.

6. **Generate the flashcard appendix:** Compress each Q&A to 2-3 sentence answers (PAR in its shortest form). These are for quick drilling, not deep reading.

7. **Verify no em-dashes** slipped through any agent output.

### Phase 3: Output to file

Write the full prep guide to:
- General: `~/research/interview-prep/{name_slug}-screen-prep-{YYYY-MM-DD}.md`
- Targeted: `~/research/interview-prep/{name_slug}-{company_slug}-screen-prep-{YYYY-MM-DD}.md`

Replace spaces with hyphens, lowercase. Example: `jubin-sanghvi-meta-screen-prep-2026-06-22.md`

**Report structure:**

```markdown
# Interview Screen Prep: {Candidate Name}

**Date:** {YYYY-MM-DD}
**Target:** {role + company, or "General screen prep"}
**Mode:** {General / Targeted}
**Readiness Score:** {X}/10

---

## Story Bank

| # | Story Name | Summary | PAR Strength | Themes Covered |
|---|-----------|---------|--------------|----------------|
| 1 | ... | ... | Strong/Medium/Thin | leadership, scaling, ... |
| 2 | ... | ... | ... | ... |

### Story Details

#### Story 1: {Name}
**Problem:** {1-2 sentences}
**Action:** {2-3 sentences}
**Result:** {1-2 sentences with metrics}
**Use for:** {question themes this story answers}

(repeat for each story)

---

## Resume Walkthrough

### Recruiter Version (2 min)
{full script, conversational tone}

### Hiring Manager Version (2.5 min)
{full script, impact-focused}

---

## Red Flags and Defensive Answers

| # | Pattern | Severity | Likely Question | Prepared Reframe |
|---|---------|----------|----------------|-----------------|
| 1 | ... | High/Med/Low | "..." | {PAR answer} |

(If none found: "No significant red flags identified.")

---

## Recruiter Screen Questions

### Background and Motivation
**Q: {question}**
Story: {story name reference, or "N/A - standalone"}
**A:** {PAR answer}
**Strength:** {Strong/Adequate/Weak}

(repeat for each question in this group)

### Logistics and Fit
(same format)

### Resume Deep-Dive Probes
(same format)

### Questions to Ask the Recruiter
1. {question} - *Probes:* {what signal it provides}
2. ...

---

## Hiring Manager Screen Questions

### Leadership and People
(same Q/A format with story references and strength ratings)

### Technical Depth and Decisions
(same format)

### Conflict and Difficult Decisions
(same format)

### Initiative and Problem-Solving
(same format)

### Questions to Ask the Hiring Manager
1. {question} - *Probes:* {what signal it provides}
2. ...

---

## GitHub Insights (if scanned)

### Projects That May Come Up
| Repo | Why They'd Ask | Suggested Answer |
|------|---------------|-----------------|
| ... | ... | {PAR-format} |

### Consistency Notes
{any gaps between GitHub and resume, or "Consistent" if aligned}

---

## Preparation Gaps

Questions where the resume does not provide strong answer material. Prep these before the interview.

| # | Question | What's Missing | How to Prep |
|---|----------|---------------|-------------|
| 1 | ... | ... | {specific guidance: what memory to recall, what data to gather} |

---

## Company-Specific Notes (if research available)

### "Why This Company?" Answer
{drafted answer pulling specific facts from research report}

### Risks to Ask About
1. {risk from research} - *Ask:* "{question}"
2. ...

### Signals to Listen For
{what to watch during the screen that validates or invalidates concerns}

---

## Readiness Assessment

| Dimension | Score | Notes |
|-----------|-------|-------|
| Story coverage | X/10 | {themes covered vs. uncovered} |
| Red flag readiness | X/10 | {high-severity flags addressed?} |
| Answer quality | X/10 | {X% rated Strong, Y% Adequate, Z% Weak} |
| Company knowledge | X/10 | {or "N/A - general prep"} |
| Walkthrough quality | X/10 | {specific, timed, bridges to target?} |
| **Overall** | **X/10** | |

---

## Flashcard Appendix (Quick Drill)

Compressed answers for rapid review. Read the full sections above for context, use these to drill timing and flow.

### Recruiter Screen
**Q:** {question}
**A:** {2-3 sentence PAR, compressed}

**Q:** {question}
**A:** ...

### Hiring Manager Screen
**Q:** {question}
**A:** {2-3 sentence PAR, compressed}

**Q:** ...

### Red Flag Responses
**Q:** {likely question}
**A:** {reframe, compressed to 2 sentences}
```

### Phase 4: Terminal output

After writing the file, print a condensed prep guide to the terminal:

```
---
INTERVIEW SCREEN PREP: {Name}
Target: {role or "General"}
Readiness: {X}/10
---

STORY BANK (use these for any behavioral Q):
1. {Story Name} - {one-line summary} [{Strong/Medium/Thin}]
2. {Story Name} - {one-line summary} [{rating}]
3. ...

RED FLAGS TO PREP:
1. [{Severity}] {Pattern} - Q: "{question}" -> Reframe: {one-line}
2. ...

GAPS (no strong answer available, prep before interview):
1. {Question or theme} - {what's missing}
2. ...

RECRUITER WALKTHROUGH (opening):
"{first 2 sentences of the recruiter version}"

HM WALKTHROUGH (opening):
"{first 2 sentences of the HM version}"

Full prep guide: ~/research/interview-prep/{filename}.md
```

---

## Screen interview conventions (baked into agent prompts)

These rules inform the question selection and answer quality:

1. **PAR format for all answers:** Problem (what was broken/needed), Action (what you specifically did), Result (quantified outcome). Keep Problem brief, Action specific, Result metric-driven.
2. **"I" not "we":** Every answer must specify the candidate's personal contribution. "We shipped" is weak. "I drove the architecture decision, coordinated 3 teams, and shipped in 6 weeks" is strong.
3. **2-minute rule:** No answer should take more than 2 minutes to deliver. If it does, it's too long for a screen. Flashcard versions should be 30-45 seconds.
4. **Relevance over completeness:** Tailor answers to the target role. A platform engineering story matters more for a platform role than a people-management story, even if the people story is "better."
5. **Numbers anchor memory:** Every answer should contain at least one concrete number (team size, dollar amount, percentage improvement, timeline). Numbers are what interviewers remember and write in feedback.
6. **Bridge forward:** End each answer with a brief connection to why this experience matters for the target role. "That's why I'm excited about [this role], because the same challenge exists at larger scale."
7. **Red flags are opportunities:** A gap, short tenure, or lateral move with a prepared, honest, forward-looking explanation demonstrates self-awareness and maturity.
8. **Ask questions that demonstrate research:** "What does success look like in the first 90 days?" is generic. "I noticed you're growing the platform team from 4 to 8, how are you thinking about specialization vs. generalist hires?" shows you did homework.
9. **No em-dashes in any output.** Use commas, colons, parentheses, or periods.
10. **No emojis.**
11. **Honest about gaps:** If the resume cannot support a strong answer for a question, say so explicitly. Never fabricate. Mark as "NEEDS PREP" with guidance.
12. **Story reuse is a feature:** The best interview candidates have 5-8 polished stories that cover 80% of possible questions. The story bank makes this explicit.

---

## Token optimization rules

- All analysis agents run at **Sonnet** model tier.
- 5 agents always run (Story Bank, Walkthrough, Red Flags, Recruiter Qs, HM Qs).
- Agent 6 (GitHub Scanner) only runs if `--github` flag is provided.
- ALL agents run in PARALLEL in a single message dispatch.
- Agent word caps: 800 words for Agents 4-5 (more Q&A content), 600 words for Agents 1-3 and 6.
- The main thread only does synthesis and file writing (Phase 2-4). It never duplicates the agents' analysis.
- Estimated total tokens:
  - General (resume only, no JD, no GitHub): ~120-160K
  - Targeted (resume + JD): ~140-180K
  - Full (resume + JD + GitHub): ~160-200K

---

## Composability

This skill produces a **story bank** artifact (in the Story Bank section of the output file) that is designed to be consumed by the future `interview-behavioral-prep` skill. That skill will:
- Read the story bank from this output
- Expand each story into full-length behavioral answers
- Map stories against leadership principle frameworks (Amazon LPs, Google, Meta, etc.)
- Add more behavioral questions beyond what screens cover

The two skills compose as: screen-prep (breadth, surface-level answers) -> behavioral-prep (depth, fully rehearsed stories).

---

## Portability

This skill is a standalone `.md` file. To use on another machine:
1. Copy this file to `~/.claude/commands/interview-screen-prep.md` (or a project commands dir).
2. Requires: Read tool (for PDF/file input), WebFetch (for JD URLs), WebSearch (for GitHub surface scan if used).
3. No MCP servers, API keys, or project-specific dependencies.
4. Output goes to `~/research/interview-prep/` which is created automatically.
5. Optionally reads from `~/research/` for company research integration (output of the `research-company` skill).
