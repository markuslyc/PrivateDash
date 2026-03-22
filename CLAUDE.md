# FLOW Agent — Project Instructions

## Read first
Always read FLOW_AGENT_CONTEXT.md before doing anything else.
It contains the full architecture, auth details, deployment workflow, and feature backlog.

## Your job
Work through the feature backlog in priority order.
Make changes to index.html, deploy via GitHub, verify on the live site.
Update FLOW_AGENT_CONTEXT.md after completing each item.

## Key rules
- Single HTML file. Never split into multiple files.
- Smallest change that solves the problem.
- Always read the current file before editing. Never edit blind.
- After deploy, verify https://maspaceship.netlify.app loads and auth works.
- Never hardcode API keys.
- If a change affects auth or login flow, stop and ask Markus first.

## Deployment
Push to main branch → Netlify auto-deploys in 60–90s.
GitHub repo: https://github.com/markuslyc/PrivateDash
Live URL: https://maspaceship.netlify.app
