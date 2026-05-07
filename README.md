# Explore Agent

A pointer to the **`Explore` sub-agent built into [Claude Code](https://claude.ai/code)**. There is nothing to install — this repo exists to document when and how to use it from a hotel-operations workflow.

> **Note:** Unlike the other th1-ai agents, `Explore` is not a custom sub-agent definition. It ships with Claude Code and is invoked automatically when the main agent decides a fast read-only search is needed. There is no `agent.md` to copy.

---

## What it does

`Explore` is a fast, read-only search sub-agent. It can:

- Find files by name, glob, or path pattern
- Grep for symbols, function names, config keys, or arbitrary strings across a repo
- Locate where something is defined or referenced
- Summarise what's in a directory tree

It runs with read-only tools (Read, Grep, Glob, basic Bash), so it cannot modify files, run destructive commands, or make API calls. That makes it cheap to invoke and safe to run speculatively.

---

## What it is **not**

`Explore` is not for:

- **Code review** — use a dedicated review skill or your normal Claude Code session
- **Design / architecture audits** — use [`plan-agent`](https://github.com/th1-ai/plan-agent) instead
- **Multi-step research with reasoning** — use [`general-purpose-agent`](https://github.com/th1-ai/general-purpose-agent) instead
- **Anything that writes** — `Explore` is read-only by design

If you find yourself asking it to "also fix" or "also draft" something, that's a signal you wanted a different sub-agent.

---

## When Claude Code picks it automatically

Claude Code routes to `Explore` (or invokes it under the hood) when the main agent's plan includes a step like:

- "Find where `<symbol>` is defined"
- "List all files matching `<glob>`"
- "Search the repo for `<pattern>`"
- "Locate the config that controls X"

You don't need to do anything for this to happen — the routing is built in.

---

## How to invoke it explicitly

From inside a Claude Code session:

```
> Use the Explore subagent to find every file that references CLOUDBEDS_CLIENT_ID.
```

Or, more naturally, just ask:

```
> Where in this repo do we set up the Cloudbeds OAuth refresh-token flow?
```

Claude Code will normally pick `Explore` itself based on the read-only nature of the question.

---

## File structure

```
explore-agent/
├── README.md       # This file — the whole repo, essentially
├── LICENSE
└── .gitignore
```

There is no `.claude/` folder, no `.env.example`, and no `agent.md`. The agent lives inside Claude Code itself.

---

## Related agents

This sub-agent is one of three built-ins documented in this org:

- [`plan-agent`](https://github.com/th1-ai/plan-agent) — design implementation plans before changing code
- [`general-purpose-agent`](https://github.com/th1-ai/general-purpose-agent) — default fallback for multi-step research and complex tasks

Hotel-domain custom agents that often delegate to `Explore` under the hood:

- [`morning-routine-agent`](https://github.com/th1-ai/morning-routine-agent)
- [`email-triage-agent`](https://github.com/th1-ai/email-triage-agent)
- [`finance-filing-agent`](https://github.com/th1-ai/finance-filing-agent)
- [`villa-analytics-agent`](https://github.com/th1-ai/villa-analytics-agent)

---

## Further reading

- Official Claude Code sub-agents docs: https://docs.claude.com/en/docs/claude-code/sub-agents

---

## License

MIT — see [LICENSE](LICENSE).
