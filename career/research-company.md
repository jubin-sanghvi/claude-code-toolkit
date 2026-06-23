Research a company for a potential job opportunity. Produces a comprehensive due-diligence report covering financials, culture, team, tech stack, market position, alumni sentiment, and trajectory outlook. Saves a full markdown report to disk and prints an executive summary in the terminal.

## How to use it

```
/research-company Acme Corp, Staff Engineer
/research-company "Databricks" "Sr Engineering Manager, Data Platform"
/research-company CompanyName        (role is optional; omit for general assessment)
```

Pass the company name and optionally the role title as args. If the role is omitted, the assessment is company-wide only (no team/tech-stack deep dive).

---

## What it does

Runs a structured due-diligence sweep across 8 dimensions, then synthesizes a trajectory assessment (bullish / neutral / bearish). The research is parallelized across subagents at the cheapest model tier that preserves quality. The main thread orchestrates and synthesizes only.

---

## Execution steps

### Phase 0: Parse input

Extract `company_name` and `role_title` from args. If unclear, ask the user to clarify before proceeding.

Create the output directory if it doesn't exist: `~/research/`

### Phase 1: Parallel research (fan-out)

Dispatch ALL of the following research agents IN PARALLEL (single message, multiple Agent calls). Each agent uses web search and web fetch internally. Use **Sonnet** model for all research agents.

Every agent prompt MUST end with: "Return findings as a concise structured summary. No preamble, no hedging, no filler. Use bullet points. Max 800 words."

---

**Agent 1: Financials & Funding**
Research `{company_name}` financial health:
- Public or private? Stock ticker if public. Current stock price and 52-week trend.
- Latest valuation or market cap.
- Revenue (annual, growth rate YoY if available).
- Profitability: profitable or burning cash? Operating margins. Runway estimate if private.
- Funding history: total raised, last round date, round type, lead investors, terms if known.
- Recent financial events: earnings beats/misses, down-rounds, debt, layoffs, hiring freezes.
- Compensation philosophy: pay at/above/below market? Cash-heavy or equity-heavy?
- For private companies: liquidity path for equity (secondary sales, IPO timeline, acquisition)?
- Refresh grants and vesting structure (front-loaded or back-loaded?).

---

**Agent 2: Product, Customers & Competitors**
Research `{company_name}` product and market:
- Core product(s) and value proposition (one paragraph).
- Target customer segment (enterprise, SMB, consumer, developer).
- Known customer count or ARR bracket.
- Customer concentration risk (top customers if discoverable).
- Net revenue retention (NRR) signals: are existing customers expanding or churning?
- Customer sentiment: G2, TrustRadius, Gartner Peer Insights ratings and recent trend.
- Top 3-5 direct competitors and how this company differentiates.
- TAM size and growth trajectory for their market.
- Is this company a platform or a feature? Risk of being absorbed by a larger player.
- Support quality reputation: are customers happy or trapped by switching costs?

---

**Agent 3: Culture, Ethics & Workplace**
Research `{company_name}` as an employer:
- Glassdoor overall rating and trend over the last 2 years (improving or declining).
- Common themes in employee reviews: top 3 pros, top 3 cons.
- Leadership stability: CEO/CTO tenure, recent executive departures or additions.
- Company ethics: controversies, lawsuits, data breaches, discrimination cases, DEI stance.
- Litigation and legal risk: pending lawsuits (patent, employee, antitrust), regulatory fines.
- Remote/hybrid/in-office policy. Stated clearly or ambiguous?
- Office locations (all major ones).
- US citizenship or security clearance requirements for engineering roles.
- LinkedIn presence: follower count, posting cadence, engagement quality.
- Diversity and inclusion: public commitments, ERGs, representation data if published.
- Community giving: open-source contributions, sponsorships, scholarship programs, meetup hosting, hiring from non-traditional backgrounds.
- Decision-making style: centralized (top-down) or decentralized (team ownership)?
- Internal mobility: can you switch teams/orgs easily?

---

**Agent 4: Market Position & Perception**
Research `{company_name}` market standing:
- Industry analyst placements: Gartner Magic Quadrant, Forrester Wave, IDC positioning.
- Recent press coverage: categorize as predominantly positive, negative, or neutral. Top 3 stories.
- Brand perception in the developer/engineering community (HN, Reddit, Twitter/X sentiment).
- Partnerships and integrations (ecosystem position, who do they depend on?).
- Acquisition activity: have they acquired companies? How did integrations go (retained talent or brain drain)?
- Are THEY an acquisition target? Who might buy them?
- Regulatory exposure: AI governance, data privacy (GDPR/CCPA), FedRAMP, SOC2, HIPAA, FIPS.
- Is regulation a moat for them or a headwind?
- Security posture: history of breaches, bug bounty program, compliance certifications.

---

**Agent 5: Engineering & Tech** (only if role_title provided)
Research `{company_name}` engineering organization:
- Engineering blog posts and conference talks (topics from the last 12 months).
- Open-source projects they maintain (GitHub org, stars, contributor activity, commit recency).
- Do they contribute upstream to projects they use, or only consume?
- Tech stack: languages, frameworks, infra (from job postings, blog posts, GitHub, StackShare).
- Engineering team size estimate and hiring velocity (growing or shrinking?).
- Engineering culture signals: deploy frequency, incident transparency, blameless postmortems.
- CI/CD maturity: how fast from PR merge to production?
- Internal platform/DX investment (do they build tooling or duct-tape?).
- On-call burden: rotation frequency, compensated, how painful (Glassdoor/Blind signals).
- Tech debt posture: are they hiring to build new things or stabilize what's failing?
- Budget/headcount authority for managers at this level.

---

**Agent 6: Growth & Expansion**
Research `{company_name}` growth trajectory:
- Employee count and 12-month growth trend (LinkedIn headcount, press releases).
- New office openings or closures in the last 18 months.
- Geographic expansion (new countries/markets).
- New product lines, pivots, or sunset products in the last 18 months.
- Hiring by function: where are they investing headcount? (Eng, sales, marketing, support?)
- Are they expanding the engineering org specifically?

---

**Agent 7: Alumni & Interview Intel**
Research what former and prospective employees say about `{company_name}`:
- Where do ex-employees go after leaving? (If alumni land at top companies, it builds your resume.)
- Average engineering tenure on LinkedIn (< 18 months median = churn signal).
- What do ex-employees say on Glassdoor/Blind about WHY they left?
- Is there an active alumni network or community?
- Interview process: how many rounds? Timeline from first contact to offer?
- Interview format: Leetcode, system design, take-home, behavioral, culture fit, presentation?
- Candidate experience reviews (Glassdoor interviews): fairness, communication, ghosting?
- Do they give feedback after rejection?
- Offer negotiation culture: do they negotiate? Exploding offers? Match competing?
- Levels.fyi compensation data for this level/role if available.
- Benefits highlights: parental leave, sabbatical, education budget, 401k match, PTO policy.

---

**Agent 8: Onboarding & New Hire Experience** (only if role_title provided)
Research `{company_name}` new hire experience:
- What do recent hires say about their first 90 days? (Glassdoor, Blind, Reddit.)
- Structured onboarding program or "figure it out" culture?
- Time to first meaningful contribution for senior hires.
- Mentorship/buddy programs.
- How are new managers set up for success? (Existing team or building from scratch?)
- Training budget and professional development support.

---

### Phase 2: Synthesize trajectory assessment

After all agents return, the main thread synthesizes. Do NOT delegate this step. Read all reports and produce:

**Trajectory Scorecard** (rate each 1-5, where 5 = strong positive signal):

| Dimension | Score | Key Signal |
|-----------|-------|------------|
| Financial health | _ | (one line) |
| Market position | _ | (one line) |
| Leadership stability | _ | (one line) |
| Product maturity | _ | (one line) |
| Engineering investment | _ | (one line) |
| Culture & ethics | _ | (one line) |
| Growth momentum | _ | (one line) |
| Talent retention | _ | (one line) |

**Overall Trajectory:** Bullish / Neutral / Bearish

**Confidence:** High / Medium / Low (based on data availability)

**Reasoning:** 3-5 sentences explaining the call. Reference specific findings.

**Key Risks:** Top 5 risks for someone joining at this level.

**Key Opportunities:** Top 5 reasons this could be a great move.

**Comparison to Current Role:** (If the user's profile is known) How does this compare on scope, scale, tech depth, growth ceiling?

### Phase 3: Generate interview questions

Based on the research, generate 12-15 questions tailored to this specific company and role:

**For the Recruiter (4-5 questions):**
- Timeline, process, level calibration, team structure, why the role is open.

**For the Hiring Manager (4-5 questions):**
- Team health, scope, what success looks like, decision-making autonomy, biggest challenge.

**For Technical Rounds (3-4 questions):**
- Architecture decisions, tech debt, on-call, deployment practices, what they'd change.

**For Leadership/Skip-Level (3-4 questions):**
- Company direction, how this team fits the strategy, investment signals, what keeps them up at night.

Each question should include a one-line note on WHY you're asking (what risk or opportunity it probes).

### Phase 4: Output to file

Write the full report to: `~/research/{company_name_slug}-{YYYY-MM-DD}.md`

The file should contain:
1. **Header:** Company name, role title, date researched.
2. **Executive Summary:** 5-line paragraph (what they do, scale, trajectory call, top risk, top opportunity).
3. **Trajectory Scorecard:** The table and overall call with reasoning.
4. **Detailed Findings:** Each dimension as an H2 section, condensed from agent output (no raw dumps).
5. **Risk Matrix:** All identified risks ranked by likelihood x impact.
6. **Opportunity Matrix:** All identified opportunities ranked by probability x upside.
7. **Interview Questions:** Grouped by audience with the "why I'm asking" notes.
8. **Unverifiable Claims:** Things research surfaced but couldn't confirm. Flag for in-person conversations.
9. **Sources:** Key URLs referenced (for later verification).

### Phase 5: Terminal summary

After writing the file, print a condensed executive summary to the terminal:

```
## {Company Name} - Trajectory: {Bullish/Neutral/Bearish} (Confidence: {H/M/L})

Score: {avg}/5 | Financial {n} | Market {n} | Leadership {n} | Product {n} | Eng {n} | Culture {n} | Growth {n} | Retention {n}

Top risks:
1. ...
2. ...
3. ...

Top opportunities:
1. ...
2. ...
3. ...

Full report: ~/research/{filename}.md
```

---

## Token optimization rules

- All research agents run at **Sonnet** model tier. Moderate analysis, not deep reasoning.
- ALL agents (up to 8) run in PARALLEL in a single message dispatch. This is the maximum parallelism available.
- Each agent prompt ends with the word-cap instruction (800 words max).
- The main thread only does synthesis and file writing (Phase 2-5). It never does raw web searches itself.
- If a research agent returns nothing useful for a dimension, note "insufficient public data" rather than re-running or re-researching.
- Agents 5 and 8 are only dispatched if a role_title is provided (6 agents for company-only, 8 for company+role).
- Total target: under 80K output tokens for a full 8-agent run.

---

## Portability

This skill is a standalone `.md` file. To use on another machine:
1. Copy this file to `~/.claude/commands/research-company.md` (or the project commands dir).
2. Requires: WebSearch and WebFetch tools (available in Claude Code by default).
3. No MCP servers, API keys, or project-specific dependencies.
4. Output goes to `~/research/` which is created automatically.
