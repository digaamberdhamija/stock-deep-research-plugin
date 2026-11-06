# Stock Deep Research Plugin

A Claude Code plugin (`trading-skills`) for news-driven stock and market research. Ask one question and the `deep-research` skill runs a set of market-news skills in parallel, writes a sourced report with scenarios and trigger levels, and ends with an order verdict: PLACE, WAIT, or DO NOT PLACE, with entry, stop, and targets. If you agree, it drafts that order for an Alpaca **paper** trading account and submits it only after you approve each ticket.

Research only, not investment advice. The bundled Alpaca server is locked to paper trading.

## What's inside

| Part | Contents |
|---|---|
| Entry point | `deep-research` — the only skill you call directly (`/trading-skills:deep-research <question>`) |
| News analysis | `market-news-analyst`, `market-environment-analysis`, `scenario-analyzer`, `news-reaction-failure-analyzer` |
| Event calendars | `earnings-calendar`, `economic-calendar-fetcher`, `fxmacrodata-calendar` |
| Catalyst analysis | `stockbee-episodic-pivot-analyzer`, `edge-hint-extractor` |
| Agents | `research-analyst` (one per research angle, read-only), `scenario-analyst` and `strategy-reviewer`, `report-writer`, `paper-trader` (the only agent that can place paper orders) |
| MCP server | `alpaca-paper` — the official Alpaca MCP server, pinned, with `ALPACA_PAPER_TRADE=true` |
| Command | `/trading-skills:scenario-analyzer` |

The news skills are marked `user-invocable: false`: they stay out of the slash-command menu, and `deep-research` (or Claude, when a request matches) runs them.

## Requirements

- [Claude Code](https://code.claude.com/docs/en/getting-started)
- [uv](https://docs.astral.sh/uv/) — skills run their scripts with `uv run --with-requirements skills/<skill>/requirements.txt`
- Optional: a [Financial Modeling Prep](https://site.financialmodelingprep.com/developer/docs) API key (`FMP_API_KEY`) for the earnings and economic calendars, news-reaction analysis, and episodic-pivot enrichment. The free tier is enough for occasional use. Without it, those angles fall back to web search or are skipped.
- Optional: an `FXMACRODATA_API_KEY` for non-USD rows in the FXMacroData calendar.
- Optional: an [Alpaca](https://alpaca.markets/) paper trading account, for the order step.

## Install

```bash
git clone https://github.com/digaamberdhamija/stock-deep-research-plugin.git
cd stock-deep-research-plugin
claude plugin marketplace add .
claude plugin install trading-skills@trading-skills
```

The plugin loads straight from the checkout, so `git pull` updates it; restart Claude Code to pick up changes.

### Alpaca paper keys

The plugin asks for two sensitive options, your paper API key ID and secret. Set them from an interactive Claude Code session:

```
/plugin configure trading-skills@trading-skills
```

Or pipe them in from a terminal as JSON: `{"alpaca_paper_api_key": "...", "alpaca_paper_secret_key": "..."}` to `claude plugin configure trading-skills@trading-skills --values-stdin`. Claude Code keeps sensitive values in its secure storage, not in settings files. Uninstalling the plugin deletes them.

## Use

```
/trading-skills:deep-research what does NVDA's guidance cut mean for semis?
```

1. It frames the question and checks which API keys are set.
2. It picks the relevant angles (recent news, macro backdrop, scenarios, market reaction, upcoming events, catalyst moves) and runs one subagent per angle in parallel.
3. A second agent reviews the scenarios, and `report-writer` writes `reports/trading_deep_research_<slug>_<date>.md`.
4. It replies with the verdict and the order verdict.
5. For a PLACE (or a WAIT that can rest at its trigger), it offers to draft the paper order. `paper-trader` returns exact tickets (sized to at most 1% of equity at risk per ticket by default) and submits only the ones you approve.

## License

MIT. Based on [claude-trading-skills](https://github.com/tradermonty/claude-trading-skills) by TraderMonty; see [LICENSE](LICENSE).
