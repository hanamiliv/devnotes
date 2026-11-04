---
type: concept
tags: [ai-engineering, claude, skills, mcp]
created: 2026-10-19
updated: 2026-10-19
---

# Claude App Features: Projects, Artifacts, Scheduled, Customize

> **TL;DR** — Projects hold context, Artifacts hold outputs, Scheduled runs a prompt on a timer, and Customize adds abilities: Skills teach *how*, Connectors give *access*, Plugins bundle both.

## The Problem

The Claude sidebar has many entries. Without a clear model of each one, you re-type context, lose outputs, and do repetitive work by hand.

## Mental Model

Treat Claude as a new teammate:

- **Project** = the brief folder you hand them.
- **Artifact** = finished work they put in a shared cabinet.
- **Scheduled task** = a standing instruction: "every Monday, do X".
- **Skill** = a runbook. **Connector** = a login to another system. **Plugin** = an onboarding kit with both.

## Key Points

- **Projects**: instructions and files shared by every chat in the project. Use one when you repeat the same context across chats.
- **Artifacts**: standalone outputs (docs, slides, dashboards, small apps) with a shareable link, all listed in the Artifacts tab. Templates: Design, Slides, Docs (beta, paid plans).
- **Scheduled**: a saved prompt plus a cadence (hourly, daily, weekly, weekdays, or manual). Runs in the cloud by default, so the prompt must be complete; nobody is there to answer questions.
- **Skills**: a `SKILL.md` with instructions, plus optional scripts and reference files. Claude loads it only when the task needs it.
- **Connectors**: MCP (Model Context Protocol) links to external services such as GitHub, Drive or an internal API, so Claude can read and act there.
- **Plugins**: one install that bundles skills, connectors, slash commands and sub-agents.
- Skills, connectors and plugins are saved to the account, so they follow you across web, desktop and mobile.

## Example

A weekly learning loop built from the pieces:

```text
Project    "Learning ClickHouse"   → notes + instructions as context
Skill      devnote                 → how to turn a chat into a note
Connector  GitHub                  → lets Claude push the note
Scheduled  every Friday 16:00      → "summarize this week's notes"
Artifact   weekly summary page     → the output you open and share
```

## Gotchas

- A skill gives know-how, not access. Pushing to GitHub needs a connector (or repo access) as well.
- Cloud scheduled tasks cannot see local folders. A task set to run on your computer needs it powered on with Claude open.
- Design is an artifact template for visual work (mockups, one-pagers) built with your design system.

## Sources

- [Connectors, skills, and plugins – Claude Docs](https://claude.com/docs/extend/overview)
- [What are artifacts – Claude Help Center](https://support.claude.com/en/articles/17153992-what-are-artifacts)
- [Scheduled tasks on the web – Claude Code Docs](https://code.claude.com/docs/en/web-scheduled-tasks)
