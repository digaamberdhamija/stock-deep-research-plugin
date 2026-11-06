---
name: deep-research
description: >-
  Deep, multi-angle news research on a US stock, sector or theme, macro event, or
  the overall market, built by running this plugin's news skills in parallel and
  merging their output into one sourced report with scenarios and trigger
  levels. Use when the user asks for deep research, a full research report, or a
  thorough look at what a headline, event, or catalyst means, e.g. "deep
  research on NVDA's guidance cut", "what does the Fed decision mean for
  banks?", "research the latest tariff news". Not for a quick headline lookup or
  a task one skill already covers. It ends with an order verdict (PLACE, WAIT,
  or DO NOT PLACE, with entry, stop, and targets) and can draft that order for
  the Alpaca paper account, placed only after the user approves it.
argument-hint: "<ticker, theme, event, or market question>"
---

# Trading Deep Research

Research question: $ARGUMENTS
(If empty, use the user's latest message.)

Repository root (`ROOT`): `${CLAUDE_PLUGIN_ROOT}`. If that shows as a literal
placeholder, this skill is not loaded as a plugin: use the directory two
levels above this file (it contains `skills-index.yaml`).

Subagents: this plugin's agents are named `trading-skills:<agent>`
(`research-analyst`, `scenario-analyst`, `strategy-reviewer`, `report-writer`,
`paper-trader`). If that name is not available, use the bare `<agent>` name.
Subagents cannot start other subagents, so launch every one from this
session.

Run a skill's helper scripts from `ROOT` with
`uv run --with-requirements skills/<skill-id>/requirements.txt python` in place
of `python3`, so each skill gets the packages it declares. Run other Python with
`uv run python`. If `uv` is not installed, ask the user to install it first.

## Workflow

### 1. Frame the question

State the subject (ticker, sector/theme, macro event, or market), the horizon
(default: 1–6 months), and what a useful answer looks like. Ask one clarifying
question only when no subject can be identified.

### 2. Check data access

Report which provider keys are set, without printing their values:

```bash
uv run python -c "import os; print({k: bool(os.environ.get(k)) for k in ['FMP_API_KEY', 'FXMACRODATA_API_KEY']})"
```

A skill whose required integration is missing (see its `integrations` entry in
`ROOT/skills-index.yaml`) is skipped or replaced with WebSearch data, and the
gap goes in the report.

### 3. Pick the angles

Choose the angles that fit the subject; a narrow question needs only two or
three.

| Angle | Skills | Fits |
|---|---|---|
| Recent news | market-news-analyst | every subject |
| Macro backdrop | market-environment-analysis | every subject |
| Scenarios | scenario-analyzer (its `scenario-analyst` agent) | a headline, event, or theme |
| Market reaction | news-reaction-failure-analyzer | a crowded market or ticker reacting to news |
| Upcoming events | earnings-calendar, economic-calendar-fetcher, fxmacrodata-calendar | ticker, macro event, market |
| Catalyst moves | stockbee-episodic-pivot-analyzer, edge-hint-extractor | ticker, theme |

### 4. Research the angles in parallel

Create a run directory `reports/deep-research/<slug>_<YYYY-MM-DD>/` under the
current working directory. In a single message, launch:

- one `research-analyst` per angle except Scenarios, giving it the question,
  subject, horizon, its angle and skills, the `ROOT` path, the run directory,
  and the data-access result
- for the Scenarios angle, one `scenario-analyst` with the headline or event,
  the subject, and the horizon

### 5. Challenge the findings

If the Scenarios angle ran, launch one `strategy-reviewer` with the full
`scenario-analyst` output. Then list where the angles disagree and the
strongest case against the leading view. If one gap would change the
conclusion, close it with a single targeted follow-up search.

### 6. Write the report

Launch one `report-writer` with the question, subject, and horizon; every
angle's findings; the scenario analysis and review, if any; the challenge
notes; the run directory; and the report path
`reports/trading_deep_research_<slug>_<YYYY-MM-DD>.md` under the current
working directory.

### 7. Give the order verdict

Reply with the research verdict in three to five lines, the report path, and
the report's order verdict:

- **PLACE** — symbol, side, order type and entry, stop-loss, targets with
  reward-to-risk, time in force, and the cancel-if conditions
- **WAIT** — the same levels, plus the trigger that turns it into PLACE and
  the valid-until date
- **DO NOT PLACE** — the reasons, and what would change the verdict

Present it as the research's conclusion for the paper account, not as
personal investment advice.

### 8. Offer paper orders

Offer this only for a PLACE verdict, or for a WAIT verdict that can be placed
as a resting limit or stop order at its trigger. For DO NOT PLACE, offer
nothing. Ask the user once whether to draft the order for the Alpaca paper
account, and place nothing without the user's approval in this conversation:

1. If the user says yes, launch `paper-trader` in `draft` mode with the report
   path, the order verdict, and any sizing the user gave. Show the user the
   tickets it returns exactly.
2. Ask the user which tickets to submit. Launch `paper-trader` in `submit`
   mode with only the approved tickets, copied exactly, and report the order
   IDs and statuses it returns.
3. If the user changes a ticket, draft it again before submitting.

Never pass trade ideas to `paper-trader` in `submit` mode without that
approval, and never use it for live trading.
