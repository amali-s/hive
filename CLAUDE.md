# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Aden Hive is a goal-driven agent framework for building self-improving AI agents. Instead of hardcoding workflows, you define goals in natural language, and the framework generates agent graphs dynamically. When agents fail, the system captures failures and evolves the agent through continuous improvement loops.

**Key packages:**
- `core/` - Core framework: agent runtime, graph executor, LLM providers
- `tools/` - MCP tools for agent capabilities (file ops, web search, PDF/CSV, etc.)
- `exports/` - User-created agent packages (gitignored)

## Common Commands

```bash
# Setup
./quickstart.sh                    # Interactive setup wizard

# Linting and formatting (uses ruff)
make lint                          # Auto-fix linting issues
make format                        # Apply ruff formatting
make check                         # Check without modifying (CI-safe)

# Testing
make test                          # Run core framework tests
cd core && pytest tests/ -v        # Same as above
cd core && pytest tests/test_file.py -v                    # Single test file
cd core && pytest tests/test_file.py::test_function -v     # Single test function

# Agent commands (after creating an agent)
PYTHONPATH=core:exports python -m agent_name validate      # Validate structure
PYTHONPATH=core:exports python -m agent_name info          # Show agent info
PYTHONPATH=core:exports python -m agent_name run --input '{...}'  # Run agent
PYTHONPATH=core:exports python -m agent_name run --mock --input '{...}'  # Mock mode (no LLM)
PYTHONPATH=core:exports python -m agent_name test          # Run agent tests
PYTHONPATH=core:exports python -m agent_name test --type constraint  # Specific test type
```

## Code Style

Configured in `core/pyproject.toml` using ruff:
- Python 3.11+ target, 100 char line length
- Double quotes for strings
- Type hints required on all function signatures
- Use `from __future__ import annotations` for modern type syntax
- Import order: stdlib → third-party → first-party (`framework`, `aden_tools`) → local
- Raise exceptions with `from` in except blocks (B904)
- Prefer comprehensions over map/filter

## Architecture

### Agent Structure

Agents are defined via JSON with:
- **Goal** - What the agent should accomplish (natural language)
- **Success Criteria** - Measurable outcomes with weights
- **Constraints** - Limitations/rules
- **Nodes** - Processing units (LLM calls, functions, routers)
- **Edges** - Connections between nodes with routing conditions

### Node Types

1. **LLM Generate** (`llm_generate`) - LLM calls without tool use
2. **LLM Tool Use** (`llm_tool_use`) - LLM calls with tool access
3. **Router** (`router`) - Decision nodes based on LLM output
4. **Function** (`function`) - Custom Python functions
5. **Human-in-the-Loop** (`hitl`) - Pause for human input

### Core Framework Components (`core/framework/`)

- `runner/` - AgentRunner loads and executes agents
- `graph/` - GraphExecutor executes node-based graphs
- `llm/` - LLM providers (Anthropic, OpenAI, LiteLLM for 100+ providers)
- `mcp/` - MCP server integration for tools
- `credentials/` - Secure credential management
- `runtime/` - Node execution environment with context
- `schemas/` - Pydantic data models

### MCP Tools (`tools/src/aden_tools/`)

Built with FastMCP. Tools available: file operations (view, write, list, replace, patch), web search/scrape, PDF read, CSV read/write, email, HubSpot.

## Agent Package Structure

```
exports/my_agent/
├── __init__.py
├── __main__.py           # CLI entry point
├── agent.json            # Agent definition (nodes, edges, goal)
├── tools.py              # Custom tools (optional)
├── mcp_servers.json      # MCP config (optional)
└── tests/
    ├── test_constraint.py
    └── test_success.py
```

## Claude Code Skills

Use these slash commands when building agents:

- `/building-agents-construction` - Step-by-step agent building guide
- `/building-agents-core` - Fundamental agent concepts
- `/building-agents-patterns` - Best practices and patterns
- `/testing-agent` - Test and validate agents
- `/agent-workflow` - End-to-end guided workflow
- `/setup-credentials` - Configure agent credentials

## Environment Variables

```bash
ANTHROPIC_API_KEY="sk-ant-..."     # Required for Claude models
OPENAI_API_KEY="sk-..."            # Optional, for OpenAI models
BRAVE_SEARCH_API_KEY="..."         # Optional, for web search tool
```

## MCP Server Configuration

MCP servers are configured in `.claude/mcp.json`:
- `agent-builder` - Agent generation tools
- `tools` - File, web, and data tools

For Cursor IDE, configuration is in `.cursor/mcp.json`.
