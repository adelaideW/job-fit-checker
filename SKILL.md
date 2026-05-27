---
name: job-fit-checker
description: >
  Researches a company and evaluates open job roles against a user-provided candidate profile.
  Trigger whenever the user shares a company website URL and wants to know if it's worth applying to,
  check for open roles, or evaluate their fit. Also triggers on: "check this company", "look into [URL]",
  "any jobs at [company]?", "is [company] worth applying to?", "find roles at [company]", or when the
  user pastes a URL with no explanation (assume they want the full analysis). After fetching company info,
  always prompt the user to share their background materials before scoring fit — never skip this step.
---

# Job Fit Checker Skill

Researches a company, finds open roles, then scores candidate fit based on materials the user provides
(resume, portfolio, GitHub, LinkedIn, or a plain-text summary of their background).

---

## Workflow

### Step 1 — Company Analysis

Fetch the company's homepage plus any "About", "Team", or investor-facing pages. Summarize:

- **What they do** — product/service in plain language
- **Stage & size** — startup / growth / public, headcount estimate
- **Funding** — last round, total raised, key investors (search Crunchbase or news if not on site)
- **Focus area** — industry vertical, technology focus (AI, fintech, developer tools, etc.)
- **Business model** — B2B / B2C / marketplace / other
- **Momentum signals** — recent launches, press, notable customers, hiring trends
- **Overall verdict** — 2–3 sentence take on whether this seems like a compelling place to work

### Step 2 — Find the Careers Page

Try common paths: `/careers`, `/jobs`, `/work-with-us`, `/about/careers`. If not found,
search `site:[domain] careers` or look for a link in the footer or nav.

If the company uses an ATS (Greenhouse, Lever, Ashby, Workday, etc.), follow the link and load listings.

### Step 3 — Scan for Open Roles

Pull **all** open roles. Do not pre-filter by job type yet — the user will tell you their target function
after sharing their background. Show the full role list as a compact table so the user can orient.

If no roles are found at all, say so clearly and stop here.

### Step 4 — Prompt for Candidate Materials

**Before scoring anything**, pause and ask the user to share their background. Use this exact prompt pattern:

---

> **Got the company info. Now tell me about yourself so I can score your fit.**
>
> Share any of the following (one or more is fine):
> - 📄 **Resume** — paste the text, or describe your experience
> - 🌐 **Portfolio** — drop a URL (personal site, Behance, Dribbble, etc.)
> - 💻 **GitHub** — link to your profile or a specific repo
> - 🔗 **LinkedIn** — your profile URL
> - ✍️ **Plain summary** — just tell me your role, background, and what you're looking for
>
> You can also tell me which role(s) you're most interested in, or I'll evaluate all relevant ones.

---

Wait for the user's response before continuing.

### Step 5 — Build Candidate Profile

From whatever the user shares, extract or infer:

| Attribute | How to extract |
|---|---|
| **Current role / title** | Resume headline, LinkedIn summary, or stated |
| **Past employers & roles** | Work history in resume / LinkedIn |
| **Domain expertise** | Industries and product types they've worked in |
| **Key strengths** | Recurring themes in their work / portfolio |
| **AI / technical skills** | Tools, languages, AI work mentioned |
| **Seniority level** | Years of experience, title progression |
| **Location** | If stated or inferable from profile |
| **Salary expectations** | Only if the user mentions it |
| **Work style preferences** | Remote / hybrid / in-office, if mentioned |

If a portfolio or GitHub URL is provided, fetch it and summarize relevant work.
If a LinkedIn URL is provided, note you can't log in — ask the user to paste the key sections instead.

Echo back a brief **Candidate Snapshot** (3–5 bullet points) so the user can confirm you understood them correctly before scoring.

### Step 6 — Match Roles & Score

Filter the full role list to roles relevant to the candidate's background. For each relevant role:

1. **Title & team** — exact job title and product area
2. **Requirements summary** — key qualifications, tools, domain, years of experience
3. **Fit assessment** — honest comparison of the candidate's background vs. the role requirements.
   Call out genuine strengths AND real gaps. Don't oversell the fit.
4. **Skills match score (1–10)** — how well their demonstrated skills match the role's requirements
   - 9–10: Near-perfect match, few or no gaps
   - 6–8: Strong overlap, some gaps that are bridgeable
   - 3–5: Partial match, notable gaps
   - 1–2: Weak match; role requires very different expertise
5. **Worth applying score (1–10)** — overall recommendation, weighted by:
   - Skills fit (primary)
   - Role-to-background narrative (can they tell a compelling story?)
   - Seniority alignment
   - Location / remote fit (only factor in if the user mentioned preferences or if role has hard constraints)
   - Salary fit (only factor in if the user mentioned expectations)
   - If a hard filter the user cares about is failed, cap at 5/10 and flag it
6. **Application link** — direct URL

---

## Output Format

```
## [Company Name]

### Company Overview

| Factor | Details |
|---|---|
| Stage / Size | e.g. Series B, ~120 employees |
| Funding | e.g. $85M raised, last round: Series B $60M (2024) |
| Key Investors | e.g. Sequoia, Benchmark |
| Focus Area | e.g. AI-powered legal tech |
| Business Model | e.g. B2B SaaS |
| Momentum | e.g. 3× YoY growth, recently launched enterprise tier |

**Verdict:** [2–3 sentence take on this employer]

---

### All Open Roles

| Title | Team / Area | Location |
|---|---|---|
| [Role 1] | [Team] | [Remote / City] |
| [Role 2] | [Team] | [Remote / City] |
...

---

### Your Fit — Relevant Roles

[For each matched role:]

| Field | Details |
|---|---|
| **Role** | [Exact title] |
| **Team / Area** | [Product area] |
| **Requirements** | [Key qualifications] |
| **Fit Assessment** | [Honest strengths + gaps vs. candidate profile] |
| **Skills Match** | X/10 — [1-line reason] |
| **Worth Applying** | X/10 — [1-line reason; flag failed hard filters if any] |
| **Salary** | [Listed range or "Not disclosed"] |
| **Location / Office** | [City + in-office policy if known] |
| **Apply** | [URL] |

---
### No relevant roles found
[if applicable — suggest checking back or reaching out directly]
```

---

## Notes & Edge Cases

- **No background materials yet:** Never score fit without Step 4. If the user skips ahead and asks for fit scores without sharing their background, repeat the prompt from Step 4.
- **Partial materials:** Work with what you get. If only a job title is given, use general knowledge of that role to estimate fit, but be explicit that the scoring is approximate.
- **LinkedIn walls:** You cannot access LinkedIn directly. Ask the user to paste their summary, headline, or work history instead.
- **Salary not listed:** Note "Not disclosed" — don't penalize unless the user has stated a salary requirement.
- **Remote roles:** Pass any location filter the user mentioned.
- **Many matching roles:** If 5+, surface the top 3 by fit score and note the rest exist.
- **No relevant roles:** Clearly say so. Optionally suggest adjacent roles or similar companies.
- **Stealth companies / no careers page:** Note no public openings were found and suggest reaching out directly or checking back.
- **General-purpose:** This skill works for any job function — engineering, design, product, marketing, operations, etc. Do not assume a specific field; derive it from the candidate's materials.
