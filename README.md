# job-fit-checker

A Claude skill that researches a company and scores open job roles against **your** background — works with any combination of resume, portfolio, GitHub, LinkedIn, or a plain-text summary.

## What it does

1. **Company research** — fetches the homepage, about pages, and funding info to give you a quick company overview (stage, size, investors, momentum)
2. **Careers scan** — finds the careers page (including third-party ATS like Greenhouse, Lever, Ashby) and lists all open roles
3. **Candidate intake** — prompts you to share your background materials before scoring anything
4. **Fit scoring** — matches relevant roles to your profile and scores each one on:
   - Skills match (1–10)
   - Worth applying (1–10), with honest strengths + gaps

## Supported input formats

| Format | How to share |
|---|---|
| Resume | Paste the text |
| Portfolio | Drop a URL |
| GitHub | Link to your profile or a repo |
| LinkedIn | Share your profile URL (Claude will ask you to paste key sections if it can't log in) |
| Plain summary | Just describe your role and background in a few sentences |

## How to trigger it

Just paste a company URL and say something like:
- _"Check this company for me"_
- _"Any jobs at [company]?"_
- _"Is [company] worth applying to?"_

Claude will research the company first, then ask for your materials before scoring your fit.

## Installation

Download `SKILL.md` and install it in your Claude skills folder.

---

Built with Claude · generalized from [company-checker](https://github.com/adelaideW)
