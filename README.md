# Tradingagent

A Claude Code project that connects to the Robinhood Trading MCP server so
Claude can look up quotes/positions and place trades in a Robinhood account
on your behalf, with your confirmation.

## Setup

The MCP server is already registered for this project in `.mcp.json`. To
connect and authenticate:

1. Open this project in Claude Code.
2. Run `/mcp` inside Claude Code.
3. Select `robinhood-trading` and follow the prompts to authenticate with
   your Robinhood account.

If you need to (re-)register the server manually instead, run:

```bash
claude mcp add robinhood-trading --transport http https://agent.robinhood.com/mcp/trading
```

## Usage

Once authenticated, just ask Claude Code to check prices, positions, or
place trades. See `CLAUDE.md` for the safety rules this agent follows —
in short: every trade is described and confirmed with you before it's
placed; nothing trades autonomously.

## Status

This is an early scaffold: MCP wiring and agent guardrails only, no
additional application code yet.
