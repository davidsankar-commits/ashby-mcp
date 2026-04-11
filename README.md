# Ashby MCP Server

MCP server that connects Claude to [Ashby](https://www.ashbyhq.com/) (ATS). Single-file, zero config.

## What it does

23 tools covering the full Ashby API:

| Category | Tools |
|---|---|
| Candidates | `search_candidates`, `get_candidate`, `list_candidates`, `create_candidate` |
| Jobs | `search_jobs`, `get_job`, `list_jobs`, `create_job` |
| Applications | `list_applications`, `get_application`, `create_application` |
| Interviews | `list_interview_schedules`, `list_interview_events`, `list_interviews`, `list_interview_plan` |
| Feedback | `get_feedback_form_definition`, `list_feedback_form_definitions`, `list_application_feedback`, `submit_application_feedback` |
| Lookup | `list_locations`, `list_departments`, `list_users`, `list_sources` |

## Setup

### Prerequisites

- [uv](https://docs.astral.sh/uv/) (`brew install uv`)
- Python 3.11+
- An Ashby API key ([Settings > Integrations > API Keys](https://app.ashbyhq.com/admin/api-keys))

### 1. Clone this repo

```bash
git clone git@github.com:louislambert/ashby-mcp.git
```

### 2. Configure Claude Desktop

Open **Claude Desktop > Settings > Developer > Edit Config**, and add:

```json
{
  "mcpServers": {
    "ashby": {
      "command": "/opt/homebrew/bin/uv",
      "args": ["run", "--script", "/path/to/ashby-mcp/server.py"],
      "env": {
        "ASHBY_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

> **Note:** Replace `/path/to/ashby-mcp/server.py` with the actual path where you cloned the repo, and add your Ashby API key.
>
> On Linux, replace `/opt/homebrew/bin/uv` with the output of `which uv`.

### 3. Restart Claude Desktop

The Ashby server should appear with a green dot in **Settings > Developer**.

## How it works

`server.py` is a single-file Python script with [inline dependencies](https://packaging.python.org/en/latest/specifications/inline-script-metadata/) (PEP 723). When Claude Desktop runs `uv run --script server.py`, uv automatically installs `mcp` and `httpx` into an isolated environment. No `pip install`, no `venv`, no `pyproject.toml`.

All tool parameters use the exact Ashby API field names (camelCase) — arguments are passed directly to the API with no remapping.

## Submitting interview feedback

The `submit_application_feedback` tool accepts the exact Ashby API format:

```json
{
  "formDefinitionId": "uuid",
  "applicationId": "uuid",
  "interviewEventId": "uuid",
  "userId": "uuid",
  "feedbackForm": {
    "fieldSubmissions": [
      {"path": "overall_recommendation", "value": "3"},
      {"path": "feedback", "value": {"type": "PlainText", "value": "Great candidate"}},
      {"path": "some-score-field", "value": {"score": 3}}
    ]
  }
}
```

Use `get_feedback_form_definition` first to get field paths and selectable values.
