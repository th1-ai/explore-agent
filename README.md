# Explore Agent

A pointer to the **`Explore` sub-agent built into [Claude Code](https://code.claude.com/docs)**. There is nothing to install. This repo documents when and how to use it from a hotel-operations workflow.

> **Note:** Unlike the other th1-ai agents, `Explore` is not a custom sub-agent definition. It ships with Claude Code, and Claude delegates to it when a task needs a read-only search. There is no agent file to copy, no placeholders to fill in, and no `.env`.

---

## What it does

`Explore` is a read-only search sub-agent. Typical jobs in a hotel-operations repo:

- Find files by name, glob, or path pattern ("where are the pre-arrival email templates?")
- Search for a config key, an env var name, a rate code, or any string across the repo
- Locate where something is defined or referenced
- Summarise what is in a folder tree

Two properties make it useful:

- **It cannot change files.** It runs with read-only tools; Write and Edit are denied. That makes it safe to run on a repo you do not know yet.
- **It keeps your main conversation short.** The search runs in a separate context window. Only the summary comes back, not every file it opened.

---

## What it is **not**

`Explore` is not for:

- **Code review**: use a dedicated review skill or your normal Claude Code session
- **Design or implementation plans**: use plan mode, see [`plan-agent`](https://github.com/th1-ai/plan-agent)
- **Tasks that mix research and changes**: see [`general-purpose-agent`](https://github.com/th1-ai/general-purpose-agent)
- **Research on the web** (suppliers, local businesses, competitors): see [`web-research-agent`](https://github.com/th1-ai/web-research-agent)
- **Anything that writes**: `Explore` is read-only by design

If you find yourself asking it to "also fix" or "also draft" something, you wanted a different sub-agent.

---

## When Claude Code picks it automatically

Claude delegates to `Explore` when it needs to search or understand a repo without making changes, for example:

- "Find where `<symbol>` is defined"
- "List all files matching `<glob>`"
- "Search the repo for `<pattern>`"
- "Locate the config that controls X"

You do not need to do anything for this to happen. For a one-file lookup Claude often just reads the file itself, which is fine.

---

## How to invoke it explicitly

From inside a Claude Code session, name it:

```
> Use the Explore subagent to find every file that references PMS_CLIENT_ID.
```

Or @-mention it: type `@`, pick `Explore` from the list, then write the request.

Or just ask a read-only question and let Claude decide:

```
> Where in this repo do we set up the PMS token refresh?
```

### Getting a good answer

- **Say how deep to go.** Claude gives `Explore` a thoroughness level: *quick* for a targeted lookup, *medium* for balanced exploration, *very thorough* for a full sweep. Put the word in your request ("do a very thorough search for ...") when it matters, for example before you remove a config key.
- **Put the context in the request.** `Explore` starts without your `CLAUDE.md` files, to stay fast. Rules or naming conventions that live only in `CLAUDE.md` are not known to it, so state the ones that matter ("rate codes are in `config/rates/`, ignore the `archive/` folder").
- **Ask for file paths and line numbers** if you plan to act on the result.

---

## Verify it works

1. Open Claude Code in any repo (`claude`).
2. Ask: `Use the Explore subagent to list every file that mentions "check-in" and say what each one is for.`
3. You should see a sub-agent step labelled `Explore` in the transcript, then a summary with file paths. No file is changed.

## Troubleshooting

- **Claude answers without using `Explore`.** Normal for small lookups. Name the sub-agent in your request to force it.
- **"Explore" is not available.** Check whether it was switched off: a `permissions.deny` entry of `Agent(Explore)` or `Agent` in a settings file, the `--disallowedTools "Agent(Explore)"` flag, or the environment variable `CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS=1`. Built-in sub-agents are registered by default in interactive sessions; a script or SDK run can remove them.
- **`Explore` ignores a house rule.** It does not load `CLAUDE.md`. Repeat the rule in the request.
- **`Explore` behaves differently from this README.** Look for a file with `name: Explore` in `.claude/agents/` or `~/.claude/agents/`. See the next section.

---

## Name collision: a custom agent called `Explore`

Per the Claude Code docs (checked November 2026), a user or project sub-agent named `Explore` **overrides the built-in**. Claude Code does not report an error; delegation to `Explore` simply goes to your definition.

- This repo ships **no** agent file, so cloning it or opening Claude Code in it overrides nothing.
- The override is also the documented way to tune `Explore`. For example, a custom definition keeps its own `model` field, so `model: haiku` runs exploration on a lower-cost model. By default `Explore` runs on the main conversation's model (with some exceptions listed in the docs), and the general `CLAUDE_CODE_SUBAGENT_MODEL` variable does not change that on its own.
- The docs do not say that an override inherits the built-in's prompt or tool limits. Treat an override as a full replacement: write your own instructions and set `tools` to a read-only list yourself. An agent file without `tools` inherits every tool, so a careless override can turn a read-only agent into one that writes.
- When several definitions share a name, the higher-priority location wins: managed settings, then the `--agents` flag, then the project's `.claude/agents/`, then `~/.claude/agents/`, then plugins.
- If you want a search agent with different behaviour and no surprises, give it a different `name` and leave the built-in alone.

---

## File structure

```
explore-agent/
├── README.md       # This file; it is the whole repo
├── LICENSE
└── .gitignore
```

There is no `.claude/` folder and no `.env.example`. The agent lives inside Claude Code itself.

---

## Related agents

The other built-ins documented in this org:

- [`plan-agent`](https://github.com/th1-ai/plan-agent): research and design a plan before changing anything
- [`general-purpose-agent`](https://github.com/th1-ai/general-purpose-agent): multi-step tasks that need both exploration and action

Custom agent for searches outside the repo:

- [`web-research-agent`](https://github.com/th1-ai/web-research-agent): suppliers, local businesses, products, competitors

Hotel-domain template repos where an `Explore` pass is a good first step after cloning, to see which files hold the placeholders and rules:

- [`morning-routine-agent`](https://github.com/th1-ai/morning-routine-agent)
- [`email-triage-agent`](https://github.com/th1-ai/email-triage-agent)
- [`finance-filing-agent`](https://github.com/th1-ai/finance-filing-agent)
- [`villa-analytics-agent`](https://github.com/th1-ai/villa-analytics-agent)

All 21 agents: https://th1.ai/agent-stack

---

## Further reading

- Official Claude Code sub-agents docs: https://code.claude.com/docs/en/sub-agents

---

## License

MIT. See [LICENSE](LICENSE).
