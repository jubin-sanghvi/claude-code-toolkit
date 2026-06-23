Generate an optimized LinkedIn profile (About section, headline, and experience blurbs) from a resume. Transforms resume bullets into narrative-style LinkedIn content that reads naturally for recruiters, hiring managers, and VPs scanning profiles. Saves output to disk and prints the full profile to the terminal.

## How to use it

```
/linkedin-optimize ~/Documents/resume.pdf
/linkedin-optimize ~/Documents/resume.pdf, Sr Engineering Manager at FAANG
/linkedin-optimize ~/Documents/resume.pdf, Staff SWE at AI startups
/linkedin-optimize (with resume text pasted in the same message)
```

Pass a file path (PDF or text) or paste the resume content directly. Optionally add a target role string to tailor positioning and keywords.

---

## What it does

Analyzes the resume, then generates three LinkedIn components in parallel: an About section using a benefit-hook framework, headline options, and narrative experience blurbs (NOT bullet-point copy-paste from the resume). The result is a complete LinkedIn profile ready to paste in.

---

## Execution steps

### Phase 0: Parse input

1. **Detect input type:**
   - If args contain a file path that exists on disk (ending in `.pdf`, `.txt`, `.md`, or `.docx`), read it.
   - Otherwise, treat the user's message content as the resume text.

2. **Parse optional target role** (any text after the file path that isn't a flag).

3. **Extract from resume:**
   - Candidate name
   - Current title and company
   - Career arc (progression of roles)
   - Top 3-5 quantified achievements (dollar amounts, scale numbers, team growth)
   - Technical domain and stack
   - Years of experience
   - Number of roles

4. **Create output directory** if it doesn't exist: `~/research/linkedin/`

### Phase 1: Parallel generation (fan-out)

Dispatch ALL of the following agents IN PARALLEL (single message, multiple Agent calls). Use **Sonnet** model for all generation agents.

Every agent prompt MUST receive: the full resume text, the extracted profile facts from Phase 0, and the target role if provided.

---

**Agent 1: About Section Generator**

Write a LinkedIn About section for this person based on their resume. Follow these rules strictly:

**Structure (4 paragraphs, ~800-950 characters total):**

1. **Opening line (benefit hook):** Frame the person as the solution to a problem the reader has. NOT "I am a [title] with [years] experience." Instead: what do you build, enable, or solve for the teams/orgs you serve? This line must survive LinkedIn's ~265 character "see more" cutoff on mobile. It should make a VP, director, or recruiter want to click.

2. **Proof paragraph:** Stacked evidence from the resume. Lead with scale and impact numbers. Include: scope of infrastructure/products owned, team size and growth, geographic reach, dollar impact. Weave these into 2-3 sentences (not a bullet list). Mention the company by name.

3. **Credibility + differentiation paragraph:** IC-to-leader arc (if applicable), what makes this person different from other candidates at this level, technical depth signal, any community/guild/open-source work. 2-3 sentences. End with something that shows range: "equally comfortable going deep into [technical thing] and giving Senior Leadership the 10,000-foot view."

4. **CTA (closing line):** Frame as solving the reader's problem, not "I am looking for." Example: "If you need a [type of leader] who can [value prop], let's talk." One sentence only.

**Rules:**
- First person throughout.
- No em-dashes (use commas, colons, parentheses, or periods instead).
- No emojis.
- Capitalize functional group names (Product, Engineering, ML/AI, Data Science, Senior Leadership).
- Tone: confident and warm, not boastful or corporate.
- Do NOT include the AI Guild adoption metric (43% to 89%) in the About if it appears in the resume, as this metric may not be externally defensible. Reference the Guild qualitatively only.
- Numbers from the resume are fine to include (dollar amounts, team size, scale).

Return: the complete About section text (ready to paste into LinkedIn) plus a character count.

---

**Agent 2: Headline Generator**

Generate 5 LinkedIn headline options for this person. The headline structure is: `Title @ Company | Domain keywords | Tagline`

**Rules:**
- LinkedIn headline limit: 220 characters (but shorter is punchier, aim for 100-160).
- The tagline should frame the person's value from the reader's perspective (what you do FOR others, not what you ARE).
- Domain keywords should use middle-dot separators for visual clarity: `Data . AI . Platforms`
- Do NOT repeat a word between the keyword section and the tagline. If "platforms" is in the keywords, don't use "platforms" in the tagline (use "foundations," "infrastructure," "systems" instead).
- No em-dashes.
- Provide variety: one numbers-forward, one punchy/short, one technically specific, one people-focused, one balanced.

For each headline, add a one-line note on who it appeals to most (recruiter, VP, peer engineer, etc.).

Return: 5 headline options with notes, plus character count for each.

---

**Agent 3: Experience Blurbs Generator**

Convert each role from the resume into a narrative-style LinkedIn experience blurb. These are NOT bullet points copied from the resume. They are short paragraphs that tell the story of what you did in that role.

**Format for each role:**

```
{Role Title} at {Company}
{Date range}

{Role context line: 1 sentence explaining what you owned/led and who depended on it.}

{Impact paragraph: 2-4 sentences weaving your top 3-4 outcomes into a narrative. Lead with the most impressive, close with scale or adoption. Use "I" naturally.}

{Optional: 1 sentence connecting this role to the next one in the arc, e.g., "This is the system I later scaled as an EM."}
```

**Rules:**
- NO bullet points. Paragraphs only. This is what differentiates LinkedIn from a resume.
- First person ("I built", "I grew", "I drove").
- Drop the weakest bullet from each role (LinkedIn is your highlight reel, not your complete record).
- Drop parenthetical tech stacks that clutter sentences. Mention technologies naturally where they add credibility.
- For IC roles (early career, 5+ years ago): keep it to 3-4 sentences total. Don't over-invest space.
- For EM/leadership roles (recent): 2 short paragraphs (scope + impact).
- The role context line is the "hook" that appears before LinkedIn's "see more" cutoff on each experience entry. Make it count.
- No em-dashes.
- For the most recent role, the context line should signal what you OWN, not what you DO. Example: "I lead AI and Data Infrastructure for Duo Security. My team owns the platforms that ML scoring, AI Assistant, and security analytics run on."

Return: complete blurbs for all roles, ready to paste into LinkedIn.

---

**Agent 4: Profile Strategy Notes**

Analyze the resume and provide strategic LinkedIn optimization notes:

1. **Keyword strategy:** Top 10 keywords that should appear across About + Experience + Skills for search visibility against the target role (or general seniority level if no target).

2. **Arc narrative:** In 2 sentences, what story should the profile tell when read top-to-bottom? (This helps the user verify consistency.)

3. **What to leave off LinkedIn that's on the resume:** Any bullets that are weak, unverifiable, or add noise rather than signal in a networking context.

4. **Featured section suggestion:** 1-2 things from the resume that could become LinkedIn Featured items (a blog post, a project, a talk, a key metric graphic).

5. **Activity recommendations:** 2-3 types of posts or engagement that would reinforce this person's positioning.

Max 400 words. Structured bullets.

---

### Phase 2: Synthesis (main thread)

After all agents return, the main thread assembles the final profile. Do NOT re-derive content. Assemble agent outputs into the final report with light editorial:

- Check the About for consistency with the headline (same framing/thesis).
- Verify no em-dashes slipped through.
- Verify character counts are within LinkedIn limits (About: 2,600 max; Headline: 220 max).
- Ensure experience blurbs don't repeat exact phrasing from the About.
- Pick a "recommended" headline from the 5 options (the one that best matches the About's thesis) and mark it.

### Phase 3: Output to file

Write the full profile to: `~/research/linkedin/{name_slug}-linkedin-{YYYY-MM-DD}.md`

**Report structure:**

```markdown
# LinkedIn Profile: {Candidate Name}

**Date:** {YYYY-MM-DD}
**Target:** {role or "General optimization"}
**Based on:** {resume file path or "pasted content"}

---

## Headline

**Recommended:**
> {headline}

**Other options:**
1. {headline} ({char count}) - {who it appeals to}
2. {headline} ({char count}) - {who it appeals to}
3. {headline} ({char count}) - {who it appeals to}
4. {headline} ({char count}) - {who it appeals to}
5. {headline} ({char count}) - {who it appeals to}

---

## About

{full About text, ready to copy-paste}

**Character count:** {N}/2,600
**"See more" preview (first 265 chars):** {first 265 chars}

---

## Experience

### {Role 1 - most recent}
{narrative blurb}

### {Role 2}
{narrative blurb}

### {Role 3}
{narrative blurb}

(etc.)

---

## Strategy Notes

### Keywords to weave in
{top 10 keywords}

### Profile arc
{2-sentence narrative summary}

### What to leave off
{bullets to omit from LinkedIn}

### Featured section
{suggestions}

### Activity recommendations
{post types to reinforce positioning}
```

### Phase 4: Terminal output

Print the complete profile content to the terminal (not just a summary, since the user will copy-paste directly):

```
---
LINKEDIN PROFILE: {Name}
Target: {role or "General"}
---

HEADLINE (recommended):
{headline}

---

ABOUT:
{full About text}

({N} chars | "see more" preview: first 265 shown)

---

EXPERIENCE:

{Role 1}
{blurb}

{Role 2}
{blurb}

(etc.)

---

STRATEGY NOTES:
{condensed notes}

---

Full report saved: ~/research/linkedin/{filename}.md
```

---

## LinkedIn-specific rules (baked into agent prompts)

1. **LinkedIn is NOT a resume copy-paste.** Experience uses narrative paragraphs, not bullets. About tells a story, not a list. Headline sells value, not job title.
2. **First person everywhere.** LinkedIn is a personal brand, not a formal application.
3. **"See more" optimization.** First ~265 characters of the About and first ~3 lines of each experience entry are visible before the fold. Hook must land above this line.
4. **No em-dashes.** Use commas, colons, parentheses, or periods.
5. **No emojis** (weakens Director/VP signal at senior levels).
6. **Bold does NOT reliably render** in LinkedIn experience sections (mobile strips it). Don't rely on formatting in experience blurbs.
7. **Capitalize functional group names** when referring to departments: Product, Engineering, ML/AI, Data Science, Senior Leadership.
8. **Benefit-hook framework for About opener:** Frame yourself as the solution to the reader's problem, not a biography of yourself.
9. **Headline: no word repetition** between keyword cluster and tagline.
10. **Experience blurbs should decrease in length** with role age. Most recent role gets 2 paragraphs. Oldest role gets 3-4 sentences.
11. **Role context line = the hook.** It appears before "see more" on each experience. Make it specific: what you own, who depends on it, at what scale.
12. **Undefensible metrics stay off LinkedIn.** The About is public and permanent. If a metric requires a 30-second caveat to defend, keep it on the resume (private, submitted per-application) and off LinkedIn.

---

## Token optimization rules

- All generation agents run at **Sonnet** model tier.
- ALL 4 agents run in PARALLEL in a single message dispatch.
- The main thread only assembles and does editorial checks (Phase 2-4).
- Target: ~80-120K total tokens per run.

---

## Portability

This skill is a standalone `.md` file. To use on another machine:
1. Copy this file to `~/.claude/commands/linkedin-optimize.md` (or a project commands dir).
2. Requires: Read tool (for PDF/file input). No web search needed.
3. No MCP servers, API keys, or project-specific dependencies.
4. Output goes to `~/research/linkedin/` which is created automatically.
