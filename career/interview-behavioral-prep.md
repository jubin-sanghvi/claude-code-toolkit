Deep behavioral and leadership round preparation. Maps your resume stories against a chosen leadership framework (Amazon LPs, Google, Meta, or generic), generates framework-specific questions, and writes both structured PAR outlines and full 2-minute rehearsal scripts for each answer. Includes a dedicated failure/growth story section. Designed to consume the story bank from `interview-screen-prep` or work standalone from a resume.

## How to use it

```
/interview-behavioral-prep ~/research/interview-prep/jubin-sanghvi-screen-prep-2026-06-22.md
/interview-behavioral-prep ~/research/interview-prep/jubin-sanghvi-screen-prep-2026-06-22.md, --framework amazon
/interview-behavioral-prep ~/Documents/resume.pdf, --framework google
/interview-behavioral-prep ~/Documents/resume.pdf, --framework meta, Sr Engineering Manager at Meta
/interview-behavioral-prep ~/Documents/resume.pdf, --framework generic, --jd ~/jd.txt
/interview-behavioral-prep (with resume text pasted in the same message)
```

**Preferred input:** The output file from `interview-screen-prep` (contains an already-extracted story bank). The skill detects this automatically by looking for a "## Story Bank" section in the input file.

**Fallback input:** A resume PDF/text. The skill will extract stories from scratch (adds one agent, slightly higher token cost).

---

## What it does

Takes your story bank (or resume) and a leadership framework, then generates behavioral interview questions mapped to each framework principle. For every question, it writes a structured PAR outline (for quick reference) and a full 2-minute rehearsal script (for practice). Also preps 2-3 dedicated failure/growth stories with positive framing.

This is the "depth" companion to `interview-screen-prep` (which provides "breadth"). Together they cover the full interview loop from first recruiter call through final behavioral panel.

---

## Frameworks available

Pass `--framework <name>` to select. Default is `generic` if omitted.

| Framework | Principles | Questions Generated | Best For |
|-----------|-----------|--------------------:|----------|
| `amazon` | 14 Leadership Principles | ~28-35 | Amazon, AWS, companies that explicitly use LP-style interviews |
| `google` | 6 hiring signals | ~12-18 | Google, Alphabet, companies that emphasize "Googleyness" style |
| `meta` | 5 cultural values | ~10-15 | Meta, Instagram, WhatsApp, companies that prize speed + impact |
| `generic` | 8 leadership competencies | ~16-24 | All other companies, startups, non-FAANG enterprise |

---

## Execution steps

### Phase 0: Parse input

1. **Detect input type:**
   - If the input file contains a `## Story Bank` section (markdown heading): treat it as screen-prep output. Extract the story bank directly. Set `mode = "from_story_bank"`.
   - If the input is a file ending in `.pdf`, `.txt`, `.md`, or `.docx` WITHOUT a Story Bank section: treat as a raw resume. Set `mode = "from_resume"`.
   - Otherwise: treat the user's message content as pasted resume text. Set `mode = "from_resume"`.

2. **Parse optional flags:**
   - `--framework <amazon|google|meta|generic>`: which LP framework to use. Default: `generic`.
   - `--jd <path_or_url>`: job description for role-specific tailoring.
   - Any remaining text = target role string.

3. **Extract candidate name** from input content.

4. **If mode = "from_story_bank":** Parse the Story Bank table and Story Details sections. You now have named stories with PAR skeletons and theme mappings.

5. **If mode = "from_resume":** Flag that Agent 1 must extract stories from scratch (adds to Phase 1a).

6. **Create output directory** if it doesn't exist: `~/research/interview-prep/`

### Phase 1a: Foundation agents (parallel)

Dispatch these agents IN PARALLEL. Use **Sonnet** model for all agents.

---

**Agent 1: Story Bank Loader/Expander**

**If mode = "from_story_bank":**

You are given an existing story bank extracted from a prior interview prep session. Your job is to EXPAND each story for behavioral interview depth:

For each story in the bank:
1. Add **behavioral angles**: list 3-4 different ways this story can be framed depending on what question asks for (e.g., the same "Platform Migration" story can answer a "technical decision" question, a "managing ambiguity" question, or a "driving alignment" question by shifting which part you emphasize).
2. Add **STAR-to-PAR bridge notes**: identify the Situation/Task split (behavioral interviewers often want you to separate "what was the context" from "what was your specific task/role").
3. Rate **versatility**: how many different LP categories can this story serve? (High = 4+, Medium = 2-3, Low = 1)
4. Flag any stories rated "Thin" in the original bank with specific notes on what details the candidate needs to recall to strengthen them.

**If mode = "from_resume":**

Read this resume and extract 6-8 core stories. For each story:
1. **Name it** (short memorable label, 2-4 words).
2. **Extract PAR skeleton**: Problem (1-2 sentences), Action (2-3 sentences focusing on YOUR contribution), Result (quantified outcome).
3. **List question themes** it can answer (leadership, conflict, technical decision, scaling, failure/learning, cross-functional influence, hiring/growing, prioritization, ambiguity, innovation, cost optimization, stakeholder management, organizational change).
4. **Rate strength**: Strong (clear PAR with hard metrics) / Medium (has PAR but soft Result) / Thin (missing Problem or Result).
5. **Add behavioral angles** (3-4 different framings per story).
6. **Rate versatility**: High/Medium/Low.

Output: expanded story bank with behavioral angles and versatility ratings. Max 800 words.

---

**Agent 2: Framework Question Generator**

Generate behavioral interview questions for the **{framework}** framework. Map each question to the specific principle/dimension it tests.

{FRAMEWORK-SPECIFIC INSTRUCTIONS - include the one matching the --framework flag:}

**If framework = amazon:**

Generate 2 questions for each of Amazon's 14 Leadership Principles (28 total). For each question:
- Name the LP it maps to.
- Write the question as a hiring manager would ask it (starting with "Tell me about a time..." or "Describe a situation where..." or "Give me an example of...").
- Note what a strong answer demonstrates (what "bar raising" looks like for this LP at the target seniority level).
- If a target role or JD is provided, make 30-40% of questions role-specific (e.g., for a platform EM role, "Ownership" questions should reference infrastructure/platform contexts).

The 14 LPs: Customer Obsession, Ownership, Invent and Simplify, Are Right A Lot, Learn and Be Curious, Hire and Develop the Best, Insist on the Highest Standards, Think Big, Bias for Action, Frugality, Earn Trust, Dive Deep, Have Backbone (Disagree and Commit), Deliver Results.

**If framework = google:**

Generate 3 questions for each of Google's 6 hiring signals (18 total):
- General Cognitive Ability (structured thinking, learning from data, problem decomposition)
- Leadership (emergent leadership, leading without authority, team outcomes over individual)
- Role-Related Knowledge (technical depth for the specific role)
- Googleyness (collaboration, comfort with ambiguity, bias to action, intellectual humility)
- Culture Add (what unique perspective you bring)
- Problem-Solving (novel approaches, first-principles thinking)

**If framework = meta:**

Generate 3 questions for each of Meta's 5 values (15 total):
- Move Fast (shipping velocity, removing blockers, pragmatic trade-offs)
- Be Bold (taking risks, proposing ambitious scope, challenging status quo)
- Focus on Impact (prioritization, saying no, measuring what matters)
- Be Open (feedback culture, transparency, admitting mistakes)
- Build Social Value (user-centricity, ethical considerations, long-term thinking)

**If framework = generic:**

Generate 3 questions for each of these 8 leadership competencies (24 total):
- Strategic Thinking (vision, long-term planning, connecting work to business outcomes)
- People Development (hiring, coaching, promoting, managing out)
- Execution Excellence (delivering on time, quality bar, managing dependencies)
- Influence Without Authority (cross-functional alignment, persuasion, stakeholder management)
- Change Management (driving org change, managing resistance, communication during transitions)
- Technical Vision (architecture decisions, build vs. buy, tech debt, modernization)
- Stakeholder Management (managing up, aligning competing interests, executive communication)
- Innovation (identifying opportunities, experimentation, calculated risks)

**For ALL frameworks, also output:**
- A "story mapping" suggestion: for each question, which story from the bank (if available) would best answer it. Use story names from Agent 1's output. Mark questions where no story fits well as "GAP - needs new story."

Output: numbered question list with LP/principle mapping, bar-raiser notes, and story suggestions. Max 800 words.

---

**Agent 3: Growth and Failure Story Prep**

Extract 2-3 failure or growth stories from this resume. These answer questions like:
- "Tell me about a time you failed."
- "What's a mistake you made and what did you learn?"
- "Describe a project that didn't go as planned."
- "What would you do differently if you could go back?"
- "Tell me about receiving tough feedback."

**For each failure/growth story:**

1. **Name it** (e.g., "The Migration That Slipped", "Hiring Mistake I Owned").
2. **Find the signal in the resume**: Look for bullets that imply a journey (before/after), a pivot, a lesson learned, or a scale challenge. Not every failure is explicit in a resume, but most career arcs contain implied struggles.
3. **Write the PAR with growth framing:**
   - Problem: what went wrong or what was the challenge? (Be specific and honest.)
   - Action: what did you do once you recognized the problem? (Show self-awareness and agency.)
   - Result: what was the outcome? (Include both the recovery/fix AND the lesson learned.)
   - Growth: one sentence on how this changed your approach going forward.
4. **Write the "safe" version**: The same story told in a way that shows vulnerability without raising red flags. Key principle: the failure should be (a) in the past, (b) something you caught and fixed, (c) something that made you measurably better.
5. **Map to framework principles**: which LP/dimension does this story demonstrate? (Usually: Learn and Be Curious, Earn Trust, Insist on Highest Standards, or generic: People Development, Execution Excellence.)

**Rules for failure stories:**
- Never pick a failure that questions your competence for the target role.
- Never pick a failure involving ethics, integrity, or interpersonal harm.
- Best failures: over-engineering, moving too fast without alignment, hiring the wrong person, under-communicating, missing a deadline because of scope creep.
- The "what I learned" must be concrete and actionable, not "I learned to communicate better" (vague) but "I now run a 15-minute alignment check with stakeholders before any project enters sprint planning" (specific).

Output: 2-3 growth stories with full PAR + growth framing + safe version. Max 600 words.

---

### Phase 1b: Answer writing agents (parallel, after Phase 1a completes)

After Phase 1a agents return, the main thread:
1. Collects the expanded story bank from Agent 1.
2. Collects the full question set from Agent 2 (with story mapping suggestions).
3. Collects growth stories from Agent 3.
4. Splits the question set into 3-4 roughly equal groups by category.
5. Dispatches answer-writing agents IN PARALLEL.

Use **Sonnet** model for all answer writers.

---

**Agents 4-6 (or 4-7): Answer Writers**

Split the question set from Agent 2 into 3-4 groups (by principle category). Each agent handles one group.

For each question assigned to you:

1. **Select the best story** from the expanded bank (use Agent 2's mapping suggestion as a starting point, but override if a better fit exists).

2. **Write the PAR Outline:**
   ```
   Story: {story name}
   Principle: {LP/dimension name}

   Problem (2 sentences):
   - What was the situation? What was at stake?

   Action (3-4 bullet points):
   - What specifically did YOU do?
   - What was your reasoning/approach?
   - Who did you influence/collaborate with?
   - What trade-offs did you navigate?

   Result (2 sentences):
   - Quantified outcome (metric, dollar, timeline, scale)
   - Broader impact or follow-on effect
   ```

3. **Write the Full Rehearsal Script (2 minutes, ~300-350 words):**
   Write this as if the candidate is speaking it aloud in an interview. Conversational first person, natural pacing. Structure:
   - Context setting (15 sec): "When I was at [Company] leading [team], we faced [problem]..."
   - The challenge (15 sec): why this was hard, what was at stake
   - Your actions (60 sec): what you specifically did, step by step, with reasoning
   - The result (20 sec): quantified outcome, what changed
   - The reflection (10 sec): what this taught you or how it applies to this role

   Rules for scripts:
   - Use "I" not "we" for your contributions. Use "we" only when describing team outcomes you led.
   - Include at least one concrete number (team size, dollar amount, percentage, timeline).
   - End with a brief bridge to the target role if relevant.
   - No em-dashes. Use commas, colons, parentheses, or periods.
   - Conversational tone with contractions ("I'd", "we're", "didn't").
   - Stay under 350 words (pacing for 2 minutes spoken).

4. **Rate the answer:**
   - Strong: story is a perfect fit, PAR is complete with hard metrics, script flows naturally.
   - Adequate: story fits but isn't ideal, or Result is soft, or the script feels forced.
   - Weak: no good story available, answer is thin, candidate needs to find a better example.

5. **For questions marked "GAP - needs new story":** Write a brief note on what kind of experience the candidate should recall (not from their resume) to answer this. Don't fabricate, just guide.

Output: all assigned questions with outline + script + rating. Max 1200 words per agent.

---

### Phase 2: Synthesis (main thread)

After all agents return:

1. **Assemble the full question/answer set** in framework order (by principle, not by agent split).

2. **Build the Story Coverage Matrix:** A table showing which stories map to which principles. Identify:
   - Well-covered principles (2+ strong answers)
   - Adequately covered (1 strong or 2 adequate)
   - Gaps (no strong answer, or questions marked Weak/GAP)

3. **Compute readiness score (1-10):**
   - Story versatility (2 pts): do stories cover the framework broadly, or are they clustered?
   - Answer quality (3 pts): % rated Strong
   - Gap coverage (2 pts): are gaps identified with prep guidance?
   - Failure stories (1 pt): 2+ growth stories prepped with safe framing?
   - Script quality (2 pts): scripts are 2-min paced, have numbers, bridge to target?

4. **Verify no em-dashes** in any output.

5. **Generate the practice plan:** Recommend which 5-8 answers to rehearse first (based on: High priority = Strong story + frequently tested principle; drill these for fluency. Medium = Adequate story + common principle; strengthen these. Low = rare principle or already natural.)

### Phase 3: Output to file

Write to: `~/research/interview-prep/{name_slug}-behavioral-{framework}-{YYYY-MM-DD}.md`

If company/role provided: `~/research/interview-prep/{name_slug}-{company_slug}-behavioral-{framework}-{YYYY-MM-DD}.md`

**Report structure:**

```markdown
# Behavioral Interview Prep: {Candidate Name}

**Date:** {YYYY-MM-DD}
**Target:** {role + company, or "General behavioral prep"}
**Framework:** {Amazon LPs / Google / Meta / Generic}
**Questions Prepped:** {N}
**Readiness Score:** {X}/10

---

## Story Bank (Expanded)

| # | Story Name | Versatility | Principles Covered | Strength |
|---|-----------|-------------|-------------------|----------|
| 1 | ... | High/Med/Low | LP1, LP2, LP3... | Strong/Medium/Thin |
| 2 | ... | ... | ... | ... |

### Expanded Story Details

#### Story 1: {Name}
**PAR Skeleton:**
- Problem: ...
- Action: ...
- Result: ...

**Behavioral Angles:**
1. For "{principle A}" questions: emphasize {aspect}
2. For "{principle B}" questions: emphasize {aspect}
3. For "{principle C}" questions: emphasize {aspect}

(repeat for each story)

---

## Story Coverage Matrix

| Principle/Dimension | Story 1 | Story 2 | Story 3 | ... | Coverage |
|--------------------|---------|---------|---------|-----|----------|
| {LP name} | X | | X | | Well-covered |
| {LP name} | | X | | | Adequate |
| {LP name} | | | | | GAP |

---

## Growth and Failure Stories

### Story: {Name}
**Context:** {when/where this happened}

**PAR (honest version):**
- Problem: ...
- Action: ...
- Result: ...
- Growth: ...

**Safe Interview Version (script):**
{full 90-second script for telling this story in an interview}

**Maps to:** {principles this demonstrates}

(repeat for 2-3 stories)

---

## Behavioral Questions and Answers

### {Principle/Dimension 1: Name}

#### Q1: "{question text}"
**Principle:** {specific LP or dimension}
**Bar-raiser note:** {what a strong answer demonstrates at this level}

**PAR Outline:**
- Story: {name}
- Problem: ...
- Action: ...
  - ...
  - ...
- Result: ...

**Rehearsal Script (2 min):**
> {full conversational script}

**Rating:** {Strong/Adequate/Weak}
{If Weak: "PREP NOTE: {guidance on what to recall or prepare}"}

---

#### Q2: "{question text}"
(same format)

---

### {Principle/Dimension 2: Name}
(same format, repeat for all principles)

---

## Preparation Gaps

Principles or questions where no strong story exists:

| # | Principle | Question | What's Missing | Prep Guidance |
|---|-----------|----------|---------------|---------------|
| 1 | ... | ... | ... | {what kind of experience to recall} |

---

## Practice Plan

**Priority 1 (drill for fluency, these WILL come up):**
1. {Question} - Story: {name} - Principle: {LP}
2. ...
3. ...

**Priority 2 (strengthen, likely to come up):**
1. ...
2. ...
3. ...

**Priority 3 (polish if time permits):**
1. ...
2. ...

**Daily practice routine:**
- Pick 2 Priority 1 answers. Set a 2-minute timer. Speak the answer aloud.
- Record yourself once per day. Listen for: filler words, vague "we" language, missing numbers, going over 2 minutes.
- Rotate through all Priority 1 answers over 3-4 days before adding Priority 2.

---

## Readiness Assessment

| Dimension | Score | Notes |
|-----------|-------|-------|
| Story versatility | X/10 | {coverage breadth across principles} |
| Answer quality | X/10 | {X% Strong, Y% Adequate, Z% Weak} |
| Gap coverage | X/10 | {gaps identified with prep guidance?} |
| Failure stories | X/10 | {2+ prepped with safe framing?} |
| Script quality | X/10 | {pacing, numbers, bridges present?} |
| **Overall** | **X/10** | |

---

## Flashcard Appendix (Quick Drill)

Compressed answers for rapid review. Use the full scripts above for actual practice.

### {Principle 1}
**Q:** {question}
**A:** {3-4 sentence PAR, compressed. Story: {name}}

**Q:** {question}
**A:** ...

### {Principle 2}
(same format)

### Growth/Failure
**Q:** Tell me about a time you failed.
**A:** {compressed safe version}
```

### Phase 4: Terminal output

Print condensed prep summary:

```
---
BEHAVIORAL PREP: {Name}
Framework: {Amazon LPs / Google / Meta / Generic}
Target: {role or "General"}
Questions Prepped: {N}
Readiness: {X}/10
---

STORY BANK ({N} stories):
1. {Name} [{Versatility}] - covers: {principles list}
2. ...

COVERAGE GAPS (no strong story):
- {Principle}: {what's needed}
- ...

GROWTH STORIES PREPPED:
1. {Name} - {one-line, safe framing}
2. ...

PRACTICE PLAN (start here):
Priority 1: {Q1 short form}, {Q2 short form}, {Q3 short form}
Priority 2: {Q4 short form}, {Q5 short form}

Full prep guide: ~/research/interview-prep/{filename}.md
```

---

## Behavioral interview conventions (baked into agent prompts)

1. **PAR + Growth for failures:** Problem, Action, Result, Growth. The fourth element (what changed in your approach) is what separates senior candidates from junior ones.
2. **2-minute rule for scripts:** Every rehearsal script must fit in 2 minutes spoken (~300-350 words). Interviewers lose focus after 2 minutes. Practice with a timer.
3. **"I" language enforced:** The interviewer is evaluating YOU, not your team. "I decided," "I built," "I influenced." Use "we" only for team outcomes you clearly led.
4. **One number minimum:** Every answer must contain at least one concrete number. If the resume doesn't have one for that story, note it as a prep gap (candidate should find the real number).
5. **Bridge to target:** End strong answers with a 1-sentence bridge: "That experience is directly relevant here because [connection to target role/company]."
6. **Principle-level depth over breadth:** For Amazon interviews, they test ALL 14 LPs. For Google/Meta, fewer dimensions but deeper probing (2-3 follow-up questions per story). Scripts should anticipate follow-ups.
7. **The "so what" test:** After every Result, ask "so what?" If the answer is "it saved money" or "it shipped faster," push deeper. The real result is often: "which unlocked [next thing], enabling [business outcome]."
8. **Story versatility is king:** A great behavioral candidate has 6-8 stories that flex across 3-4 principles each. Prep fewer stories deeply rather than many stories shallowly.
9. **Failure stories must be past tense:** The failure happened, you fixed it, you grew. Never tell a story about an ongoing struggle. The resolution and growth are mandatory.
10. **No em-dashes in any output.** Use commas, colons, parentheses, or periods.
11. **No emojis.**
12. **Honest about gaps:** If no story covers a principle, say so. Don't stretch a story beyond its natural fit. The prep guidance helps the candidate recall experiences not on their resume.

---

## Token optimization rules

- All agents run at **Sonnet** model tier.
- Phase 1a: 3 agents in parallel (Story Bank, Framework Questions, Growth Stories).
- Phase 1b: 3-4 agents in parallel (Answer Writers, split by principle category).
- Total: 6-7 agents across two parallel waves.
- Agent word caps: 800 words for Agents 1-2, 600 words for Agent 3, 1200 words for each Answer Writer agent.
- Main thread only does synthesis, scoring, and assembly (Phase 2-4).
- Estimated total tokens:
  - Amazon (28-35 questions, both formats): ~220-300K
  - Google (12-18 questions, both formats): ~150-200K
  - Meta (10-15 questions, both formats): ~130-180K
  - Generic (16-24 questions, both formats): ~170-240K
- From story bank input: subtract ~20-30K (no extraction needed).

---

## Composability

**Reads from:** `interview-screen-prep` output (story bank section). If that file exists, pass it as input for the most efficient workflow.

**Recommended workflow:**
1. Run `interview-screen-prep` first (covers recruiter + HM screens, generates story bank).
2. Run `interview-behavioral-prep` with the screen-prep output file + target framework.
3. Together: complete interview loop coverage from first recruiter call through final behavioral panel.

**Story bank format (what this skill reads from screen-prep output):**
```markdown
## Story Bank

| # | Story Name | Summary | PAR Strength | Themes Covered |
|---|-----------|---------|--------------|----------------|

### Story Details

#### Story 1: {Name}
**Problem:** ...
**Action:** ...
**Result:** ...
**Use for:** ...
```

---

## Portability

This skill is a standalone `.md` file. To use on another machine:
1. Copy this file to `~/.claude/commands/interview-behavioral-prep.md` (or a project commands dir).
2. Requires: Read tool (for PDF/file input). WebFetch only if --jd is a URL.
3. No MCP servers, API keys, or project-specific dependencies.
4. Output goes to `~/research/interview-prep/` which is created automatically.
5. Works best when paired with `interview-screen-prep` (reads its story bank output) but can run standalone with just a resume.
