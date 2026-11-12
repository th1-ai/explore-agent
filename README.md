# Explore Agent

A pointer to the **`Explore` sub-agent built into [Claude Code](https://code.claude.com/docs)**. There is nothing to install. This repo documents when and how to use it from a hotel-operations workflow.

> **Note:** Unlike the other th1-ai agents, `Explore` is not a custom sub-agent definition. It ships with Claude Code, and Claude delegates to it when a task needs a read-only search. There is no agent file to copy, no placeholders to fill in, and no `.env`. The repo adds two small files for unattended use: [`AUTONOMY.md`](AUTONOMY.md) and [`.claude/settings.example.json`](.claude/settings.example.json). Neither changes how Claude Code behaves when you clone the repo.

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
- **"Explore" is not available.** Check whether it was switched off: a `permissions.deny` entry of `Agent(Explore)` or `Agent` in a settings file, the `--disallowedTools "Agent(Explore)"` flag, or the environment variable `CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS=1`. In a `claude -p` run, `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS=1` removes every built-in sub-agent. Built-in sub-agents are registered by default in interactive sessions; a script or SDK run can remove them.
- **`Explore` ignores a house rule.** It does not load `CLAUDE.md`. Repeat the rule in the request.
- **`Explore` behaves differently from this README.** Look for a file with `name: Explore` in `.claude/agents/` or `~/.claude/agents/`. See the next section.

---

## Run it on its own

A scheduled hotel agent can hand its searches to `Explore`. `Explore` stays read-only; what the calling agent does with the answer is set by [`AUTONOMY.md`](AUTONOMY.md). Copy that file into the repo that runs on a schedule. The shipped default is `Level: 1`: scheduled, report only.

The mechanics below were checked against the Claude Code docs in November 2026 and the command was run once on Claude Code 2.1.323. Everything else is marked as an example.

### A scheduled, read-only health check

This is the one job where `Explore` is the whole task: a daily check that the files your guest-facing agents depend on are complete.

```bash
claude -p "Read AUTONOMY.md and obey it. Then use the Explore subagent, very thorough, to search this repo (skip .claude/) for: text in double curly braces that is still an unfilled placeholder, and markdown links to files that do not exist. Give file paths and line numbers. Change nothing. Start the report with a section 'Needs you'; if it is empty write 'Nothing needs you'." \
  --settings .claude/settings.example.json \
  --permission-mode dontAsk \
  --max-turns 20 \
  --output-format json < /dev/null
```

What each part does, per the docs:

- `-p` runs Claude Code without a session to type in. Built-in sub-agents such as `Explore` are available unless a deny rule or an environment variable removes them (see Troubleshooting).
- `--settings` loads the permission file for this run only. Nothing is installed.
- `--permission-mode dontAsk` denies every tool call that would otherwise ask for approval. File reads inside the working directory still run.
- `--max-turns` stops a run that does not finish. `--max-budget-usd` is a second cap if you pay per token; it works from an estimate, so leave headroom.
- `--output-format json` returns one JSON object. The report is in `result` and the estimated cost in `total_cost_usd` (extract with `jq -r '.result'`). With the default text output, a run that started `Explore` in the background prints an interim line first ("still running") and the report after it.
- `< /dev/null` tells Claude Code not to wait for piped input.

Do not add `--bare` to this command. Bare mode does not use a subscription login and skips the discovery of hooks, skills and custom sub-agents. We did not test `Explore` in bare mode. The docs say bare mode will become the default for `-p` in a future release; when that happens, check this command again.

### The permission file

[`.claude/settings.example.json`](.claude/settings.example.json) is an example of least privilege for a search-only run. It is named `.example` so that cloning never applies it. Pass it with `--settings`, or copy its rules into your own `.claude/settings.json`.

- It sets `defaultMode` to `dontAsk`.
- It denies `Bash`, `PowerShell`, `Edit`, `Write`, `NotebookEdit`, `WebFetch`, `WebSearch` and every MCP tool (`mcp__*`). A sub-agent inherits the parent's permission rules, so `Explore` is bound by the same list. This matters: the docs say `Explore` has "read-only tools; Write and Edit are denied", and do not say it is without shell or MCP tools. The deny rules are what keep a scheduled search away from your mail and PMS tools.
- It denies reading `.env` files. Add a `Read(...)` rule for every folder that holds guest data the search does not need.
- With `Bash` removed, Claude Code searches with its `Glob` and `Grep` tools.

When the file was tested, a request to create a file and to read `.env` was refused on both counts, and the search above ran through `Explore` with only `Glob`, `Grep` and `Read`.

If your hotel agent needs its own CLIs (a PMS wrapper, a mail wrapper), do not add them to this file. Keep this file for search-only runs and give the hotel agent its own settings, as described in that agent's repo.

### Scheduler example (cron, macOS or Linux)

An example, not run as a cron job by us. The Claude Code docs do not describe system cron; they list cloud scheduled tasks, Desktop scheduled tasks and GitHub Actions as their own ways to run on a schedule. Put the `claude -p` command above into a script, for example `health-check.sh`, then:

```cron
# Every day at 06:30. Runs only if AUTONOMY.md has exactly one Level line and it says 1, 2 or 3.
30 6 * * * cd /path/to/your/hotel-repo && [ "$(grep -c '^Level:' AUTONOMY.md)" = 1 ] && grep -Eq '^Level: [123]$' AUTONOMY.md && ./health-check.sh >> health-check.out 2>&1
```

The two `grep` checks are the kill switch. They were tested: `Level: 1` and `Level: 3` run; `Level: 0`, `Level: PAUSED`, an unknown value, two `Level:` lines, or a missing `AUTONOMY.md` does not. Add `health-check.out` to your `.gitignore`. Cron has a minimal environment, so use the full path to `claude` if the command is not found.

### When another agent delegates to Explore

Inside a scheduled run of one of your hotel agents, delegation to `Explore` needs no setup. Three facts from the docs shape how to use it:

- **`Explore` starts without your `CLAUDE.md` files**, and it does not read `AUTONOMY.md`. The calling agent must put the limits in the request: which folders, which to skip, how thorough.
- **`Explore` is one-shot.** It returns one answer and cannot be resumed. For work that continues, use [`general-purpose-agent`](https://github.com/th1-ai/general-purpose-agent) or a custom sub-agent.
- **A custom agent run as the main session can be limited to `Explore`.** When you start an agent with `claude --agent <name>`, a `tools` line such as `tools: Agent(Explore), Read` lets it spawn `Explore` and no other sub-agent. The docs say this allowlist works only for the main-session agent, not inside a sub-agent.

Never delegate to `Explore`: anything that writes or sends, a lookup in a live system, a decision, web research, or proof that something is absent before a step that cannot be undone. The full list and the reasons are in [`AUTONOMY.md`](AUTONOMY.md).

Handoffs: `Explore` returns its answer to the agent that called it and to nothing else. That agent puts findings a human must see in the "Needs you" section of its own run report. If you use [`morning-routine-agent`](https://github.com/th1-ai/morning-routine-agent) as the daily orchestrator, the health check report is one more input for it to read.

### Cost

- Each sub-agent sends its own requests on top of the main conversation's, and they count toward your usage.
- `Explore` runs on the main conversation's model (the docs list exceptions). Start the scheduled run with `--model sonnet` or `--model haiku` to lower the cost of both, or use a custom override as described under "Name collision".
- A personal Pro or Max plan used headless is subject to Anthropic's usage policy and plan limits; for a heavy or business-critical schedule, use an API key or a team plan.

### Moving up a level

1. Run at `Level: 1` for 7 days. Read each report.
2. If the reports were correct and nothing surprised you, change the one line in `AUTONOMY.md`.
3. For `Explore` a higher level adds nothing by itself. It lets the calling agent prepare work from the findings, under that agent's own `AUTONOMY.md`.
4. To stop at any time, change the line to `Level: PAUSED` (or `Level: 0`).

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
├── README.md                       # This file
├── AUTONOMY.md                     # Rules for a scheduled agent that delegates to Explore
├── .claude/
│   └── settings.example.json       # Example permissions for a search-only run (not applied on clone)
├── LICENSE
└── .gitignore
```

There is no `.claude/agents/` folder and no `.env.example`. The agent lives inside Claude Code itself.

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
- Running Claude Code without a session (`claude -p`): https://code.claude.com/docs/en/headless
- Permission rules and modes: https://code.claude.com/docs/en/permissions

---

## License

MIT. See [LICENSE](LICENSE).
