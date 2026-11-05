# Event-Market Graph

A Neo4j graph that links geopolitical events to market moves, with an AI agent you can ask "why did the VIX move in June?" and that answers with the graph paths behind it, always as a hypothesis, never as a cause.

https://github.com/user-attachments/assets/77c34c42-513b-4418-ad08-64d093e82495

<p align="center"><sub>10-minute project demo: my part, the agent, starts at 3:57 · <a href="https://event-market-graph-ten.vercel.app">Live demo (recorded runs)</a></sub></p>

<p align="center">
  <img src="docs/screenshots/agent-answer.png" width="700" alt="Agent answer with confidence, tool calls and graph evidence">
</p>

## Why I built it

This was the final project of the Graph Database course in my BSc AIDAMS (ESSEC × CentraleSupélec). Markets react to geopolitical news, but in a spreadsheet you can't see which events came just before a move. A graph puts both on the same trading day, so the connection becomes a query.

Group project of five, delivered as a relay: Adrien built the graph and data pipeline, Oscar the analytics, Victor the infrastructure, and Louis the quality and final evidence. **I built stage 3: the FastAPI analytics API and the LangGraph agent with its guardrails.**

## What it does

- **One graph for events and prices:** about 4,500 GDELT events and 880 FRED price points (Brent, WTI, EURUSD, USDJPY, US 10-year yield, VIX) from February to July 2026, joined on `TradingDay` nodes.
- **Analytics API:** correlations between event themes and assets, the events that preceded the largest moves, shortest paths and PageRank.
- **Natural-language agent:** a local LLM (Ollama) answers questions by calling four tools on the graph, and shows the Cypher it ran and the paths it used.
- **Demo UI:** a React page that animates the agent walking through the graph, step by step.

## How it works

The agent is a LangGraph workflow with four tools: a read-only Cypher query, correlation, timeline and "explain this move". Every answer is structured and labelled `verified` or `hypothesis`, with its confidence, tool calls and graph paths.

Three decisions shaped my part:
- **Generated Cypher is never trusted.** It is checked against the graph contract (labels, relationship directions), bounded in depth and `LIMIT`, and run in a read-only session with a timeout. Asked to "delete all the events", the agent's query is rejected before it reaches Neo4j.
- **No causal claims.** A filter rewrites causal wording, and every correlation carries a disclaimer. Requests for trading advice are refused.
- **The API contract was frozen before the agent was built,** so the analytics, the agent and the UI could be developed in parallel without breaking each other.

Results: on 10 fixed questions, the agent picked the right tool and answer type every time, and all 5 unsafe prompts were rejected. The test suite has 135 tests with 94% coverage.

**Stack:** Python, FastAPI, LangGraph, Ollama, Neo4j + Graph Data Science, Docker Compose, Nginx, React, pytest.

## Run it

```bash
cp .env.example .env    # set NEO4J_PASSWORD and API_KEY
make docker-up && make seed
open http://localhost:8080/docs
```

Full documentation (API examples, graph schema, architecture, datasets) is in [docs/FULL_README.md](docs/FULL_README.md).

## Limitations

- The local model is slow on CPU (about 2 minutes per answer) and sometimes leaves numbers out of its prose, so the read-only endpoints remain the reliable source for exact figures.
- GDELT events are machine-coded from news and can contain duplicates or location errors.
- Correlations are descriptive only: the project does not identify causes.

*This fork highlights my part. The original team repository is [adrienlabergere-creator/graphdb-event-market](https://github.com/adrienlabergere-creator/graphdb-event-market).*
